# 使用 AI 完成 CrabUSB 驱动任务的经验分享

这篇文档把一次“硬件 + 驱动 + 长周期 Debug”的 PR 经历，整理成一个更容易复用的工作流教程：如何把 Coding Agent（Codex / Claude Code / OpenCode 等）用在**复杂仓库**、**需要反复上板**、**会话跨度长**的任务上。

## 这次做了什么（目标与成果）

**任务目标**：在 Orange Pi 5 Plus（RK3588）上开启 USB 所需的电源域/相关初始化，使摄像头等 USB 设备进入正确连接状态；并尽量做到不依赖 U-Boot 的 `usb start`。

相关 PR：
- https://github.com/drivercraft/CrabUSB/pull/40
- https://github.com/drivercraft/CrabUSB/pull/43

## 适合谁读

- 你要做的是“真实工程任务”，不是 5 分钟 Demo：需要反复跑测试、读日志、查手册、改代码。
- 你对驱动/硬件不熟，但愿意用 AI + 工程化流程把任务推进。
- 你希望 AI 能连续跑几个小时甚至过夜，而不是每 5 分钟就得人工接管。

## 0. 先补一点前置（把 AI 当成工程工具）

建议先看一份系统性材料，理解 Coding Agent 的工作方式（尤其是：上下文、反馈回路、验证）。

- CLI AI 辅助编程：
  - [Standord CS146S: The Modern Software Developer - fa25](https://themodernsoftware.dev/)（Week 4: Coding Agent Patterns）
  - 或者 [DataWhale 对 CLI AI 的讲解](https://github.com/datawhalechina/easy-vibe/blob/bce0cfa56fb5084a46850133c5d33024ec9f1777/docs/stage-2/backend/2.6-modern-cli/extra7/extra7-cli-ai-coding-tools-and-the-principles-of-test-driven-development.md)（最新内容查阅[完整教程](https://github.com/datawhalechina/easy-vibe)）
  - 我的补充笔记：[对 AI 辅助编程的思考和摘录](https://yueneiqi.github.io/posts/ai/)

## 1. 工作区怎么配：让 AI 能“跑得久”

长任务最怕两件事：
1) **上下文丢失**（换会话后不知道做到哪）
2) **人工介入过多**（每次上板测试都要你手动重启/跑命令/粘日志）

核心思路：把“持续任务”拆成两个层次：
- **长期约束**：规格/硬件事实/关键命令（写进 `CLAUDE.md` 或类似文件）
- **短期记忆**：当前进展、下一步、已验证结论（写进 `progress.txt`）

### 1.1 最小工作区结构（推荐从这里开始）

下面是一个最小可用结构（思路参考 [anthropics 的 autonomous-coding](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding)）：

```
minimal structure of the workspace:
├── AGENTS.md -> ./CLAUDE.md                 # AI 代理配置文件的符号链接
├── CLAUDE.md                                # AI 代理主配置（上下文 + 指令）
├── claude-progress.txt -> domain/claude-progress.txt  # 进度跟踪（短期记忆）
├── CrabUSB                                  # CrabUSB 驱动主项目目录
└── domain                                   # 领域知识（手册拆分/规格/笔记）
```

为什么要 `AGENTS.md -> CLAUDE.md`？
- 不同工具对“默认配置文件名”的支持不一样。
- 用软链接可以让多个 coding agent 共享同一套上下文入口。

### 1.2 随着开发推进，工作区会自然膨胀

实际开发中会不断加东西（脚本、日志、参考仓库、手册转换后的 Markdown 等）。一个更“现实”的结构可能长这样：

```
add misc files
├── AGENTS.md -> ./CLAUDE.md
├── CLAUDE.md
├── claude-progress.txt -> domain/claude-progress.txt
├── config                                   # 配置文件目录（串口/板卡/测试相关配置）
├── CrabUSB
├── domain
├── log                                      # 每次跑板子的日志都落盘，便于 diff
├── orangepi5plus.dts                        # 设备树源文件（非常重要的事实来源）
├── reference -> ../../reference/orangepi     # 参考资料目录（软链接）
├── rk3588-clk                               # 3rd: RK3588 时钟驱动相关代码
├── rk3588_doc_md                            # RK3588 芯片手册（Markdown 格式，按章节拆分）
├── rkbin                                    # 3rd: Rockchip 二进制固件文件
├── rkdeveloptool                            # 3rd: Rockchip 开发烧录工具
├── rockchip-pm                              # 3rd: Rockchip 电源管理相关代码
├── script                                   # 辅助脚本（跑测试/重启/抓日志）
└── u-boot                                   # 重要参考：U-Boot 源码（对齐寄存器/初始化流程）
```

这里的文件组织还比较杂乱，但两点比较重要：
- **AI 能快速定位**：哪些是主项目、哪些是脚本、哪些是资料。
- **人能回放历史**：日志和关键上下文长期保存，避免重复踩坑。

## 2. 自动化：把“上板测试”变成一条命令

对驱动任务来说，真正的效率来自于把循环跑起来：

> 修改代码 → 编译/打包 → 上板/重启 → 运行测试 → 收集日志 → 对比预期

CrabUSB 的开发中，`ostool` 很好用（能把“开发-测试”的很多繁琐动作包装起来）：
- https://github.com/drivercraft/ostool

### 2.1 关键点：减少人工干预（重启 + 运行测试）

在我的流程里，有两块组合起来效果很好：

1) **串口重启脚本**（示例名：`start.py`）
- 通过串口给板子发 reset，让板子进入可预测的状态

2) **U-Boot 侧可控**（看门狗 + saveenv）
- 我使用了一份修改过的 U-Boot，打开 `wdt` 和 `saveenv`
- https://github.com/starry-os-orangepi5p/u-boot

这样做的目的：
- 让板子重启后停在 U-Boot 命令行等待，而不是直接启动系统跑飞
- 用看门狗兜底：如果命令卡住，超时自动重启，避免你半夜起来按电源键

### 2.2 命令模板：每次测试都落盘日志

下面这组命令在 `docs/CLAUDE.workspace.md` 和 `docs/CLAUDE.CrabUSB.local.md` 里有例子（重点是：**不要直接 cargo test**，而是用项目包装好的脚本）。

```bash
# QEMU test
bash $SCRIPT_PATH/run_crabusb_qemu.sh | tee ./log/$(date +%Y%m%d_%H%M%S)_qemu.log

# no `usb start` uboot cmd
bash $SCRIPT_PATH/run_crabusb_uboot.sh | tee $LOG_PATH/$(date +%Y%m%d_%H%M%S)_crabusb.log

# with `usb start` uboot cmd
bash $SCRIPT_PATH/run_crabusb_uboot_usb_start.sh | tee $LOG_PATH/$(date +%Y%m%d_%H%M%S)_crabusb_usb_start.log
```

注意：这些脚本在“完整开发工作区”里通常会存在，但在当前这个分享目录里未必都能直接跑；这里更想传达的是**模式**：
- 固定入口脚本（避免 AI 或人随手敲错命令）
- 每次运行都保存日志（便于对比/回滚/复盘）

## 3. 用 `direnv` 解决路径/环境变量混乱

当你同时有：主仓库、脚本仓库、手册仓库、第三方依赖、各种工具链时，环境变量会很乱。

我使用 `direnv` 来自动加载工作区环境：
- https://github.com/direnv/direnv

收益：
- 进入任何子目录都能得到一致的 `$PWD` 语义和一组约定好的路径变量（如 `$SCRIPT_PATH`、`$LOG_PATH` 等）
- 对 AI 特别友好：减少“相对路径处理不好导致跑错命令/找错文件”的概率

## 4. 上下文管理：把最重要的东西写进 `CLAUDE.md`

驱动 Debug 的难点不是“写代码”，而是“别被错误的上下文带跑”。这张图非常形象：

![Impact Hierarchy for Coding Agents：从下到上依次是 Command/CLAUDE.md、Specification、Research、Plan、Code。越靠下的错误（例如一条错误的命令/上下文约束）会放大成指数级的错误代码量；因此人类应把精力优先放在最高杠杆的环节。](./imgs/impact_hierarchy.png)

图里表达的核心：
- `1 Bad Line of Code == 1 Bad Line of Code`
- `1 Bad Line of Plan == 10-100 Bad Lines of Code`
- `1 Bad Line of Research == 1000+ Bad Lines of Code`
- `1 Bad Line of Specification == 10000+ Bad Lines of Code`
- `1 Bad Line of Command/CLAUDE.md == 100000+ Bad Lines of Code`

我的经验是：
- 真正影响产出的不是“你让 AI 写了多少代码”，而是**你给的约束是否正确**、**验证回路是否可靠**。

### 4.1 `CLAUDE.md` 建议包含哪些内容

推荐把这些东西写成“每个会话开头强制执行的 checklist”，让 AI 每次都先对齐事实：

- 工作区定位：`pwd`、`ls -la`
- 读规格：例如 `domain/usb_driver_spec_orangepi5plus.md`
- 读当前状态：`status.md`
- 读短期记忆：`claude-progress.txt`
- 看最近提交：`git log --oneline -20`
- 固定测试入口：强调“不要 cargo test，跑脚本并落盘日志”

（示例见：`docs/CLAUDE.workspace.md`）

### 4.2 `progress.txt`（短期记忆）怎么用

`progress.txt`（例如 `domain/claude-progress.txt`）是给“跨会话”准备的：
- 今天 AI 做了什么（已经验证了什么）
- 哪些假设被证伪了
- 下一步最小行动是什么
- 当前阻塞点是什么

一个小技巧：
- 一个会话尽量只解决一个“小任务”。做完就把该任务从 progress 清掉，避免越滚越大。

## 5. 我踩过的两个大坑（以及怎么走出来）

### 5.1 依赖库反复报错：不要让 AI 走到“vendor 地狱”

中间很长一段时间，AI 一改就报错，集中在：
- `fdt-parser` / `sparreal-os` / `rdrive`

常见错误的应对是：AI 试图把依赖库全部拷进仓库里改（vendor），短期看似能推进，长期会让代码膨胀、方向跑偏。

对我帮助最大的转折点之一是：
- 这些上游库后来有版本更新
- 更新后，很多“反复报错/接口对不上”的问题自然消失

教训：
- 依赖问题优先确认“是不是上游已经修了/接口变了”。
- 如果你已经卡住很久，尽快把信息同步给上游/维护者，不要一个人死磕。

### 5.2 寄存器对齐 U-Boot 仍然不工作：要补齐“缺失资料”

当 USB/PHY 寄存器看起来已经与 U-Boot 完全匹配，但端口仍停留在：
- **RxDetect (PLS=5)，CCS=0**（未检测到设备）

这类问题很容易让 AI 在“寄存器微调”里打转。

最终有效的动作是：
- 补齐 GL3523 Hub 资料（把 PDF/图片转成按章节 Markdown，便于定位）
- 用多模态模型（如 Gemini）理解原理图，把结论写回 spec
- 在确实卡住时，切到更强的“长思考/规划模式”（例如 GPT-5.2）让模型先重建因果链，再给一个完整方案

## 6. 一份可复用的“AI 驱动任务工作流”

如果你想把这套方法复用到别的驱动/硬件任务上，可以照下面走：

1) **把事实写进 spec**：板卡、地址、关键外设、拓扑、已知限制
2) **把命令写进 CLAUDE.md**：每次会话先对齐，再动手
3) **把验证脚本化**：一条命令跑完并落盘日志
4) **每次只改一件事**：改完立刻验证，不要堆改动
5) **卡住就补资料**：缺手册/原理图结论时，AI 只能猜
6) **及时沟通上游**：依赖库/接口变更不要一个人硬扛

## 7. 还能怎么改进

- 把上下文管理做成更细粒度的 skill（按“任务类型/模块”动态加载，而不是每次全量塞给模型）
- 自动化再进一步：参考 https://github.com/yueneiqi/auto-rs

## 附录：这次项目的记录

- 会话时间线：[`docs/CrabUSB_Conversation_Timeline.md`](./docs/CrabUSB_Conversation_Timeline.md)
- Prompt 模式总结：[`docs/CrabUSB_Prompt_Patterns.md`](./docs/CrabUSB_Prompt_Patterns.md)
- Workspace 级别的 `CLAUDE.md` 示例：[`docs/CLAUDE.workspace.md`](./docs/CLAUDE.workspace.md)
- CrabUSB 子目录的 `CLAUDE.local.md` 示例：[`docs/CLAUDE.CrabUSB.local.md`](./docs/CLAUDE.CrabUSB.local.md)