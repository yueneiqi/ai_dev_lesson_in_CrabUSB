# 使用 AI 完成 CrabUSB 驱动任务的经验分享

## 背景

### 个人背景
- 驱动开发基础较弱
	- 参加过 OpenCamp 的 StarryOS 驱动比赛，完成了简单驱动，了解 Arceos 的开发流程和嵌入式板卡烧录的开发流程

### 任务

在香橙派上开启 usb 所需的电源域，使摄像头设备进入正确连接状态

相关PR：
- https://github.com/drivercraft/CrabUSB/pull/40
- https://github.com/drivercraft/CrabUSB/pull/43

## 前置知识

- CLI AI 辅助编程：
	- [Standord CS146S: The Modern Software Developer - fa25](https://themodernsoftware.dev/)
		- Week 4: Coding Agent Patterns
	- 或者，查看 [DataWhale 对 CLI AI 的讲解](https://github.com/datawhalechina/easy-vibe/blob/bce0cfa56fb5084a46850133c5d33024ec9f1777/docs/stage-2/backend/2.6-modern-cli/extra7/extra7-cli-ai-coding-tools-and-the-principles-of-test-driven-development.md)（最新内容查阅[完整教程](https://github.com/datawhalechina/easy-vibe)）
	- 我的总结：[对 AI 辅助编程的思考和摘录](https://yueneiqi.github.io/posts/ai/)

## 如何配置，让AI自己长时间运行

- 工作区设置
	- 思路参考：[anthropics/claude-quickstarts/autonomous-coding](https://github.com/anthropics/claude-quickstarts/tree/main/autonomous-coding)
- 减少人工介入的工作流
	- `ostool`
	- [修改 `uboot`](https://github.com/starry-os-orangepi5p/u-boot)，开启看门狗`wdt`和`saveenv`
	- `CLAUDE.md` & `progress.txt`

## 中间的卡点和解决方法或教训

### 卡点

- [fdt-parser](https://github.com/drivercraft/fdt-parser)/[sparreal-os](https://github.com/drivercraft/sparreal-os)/[rdrive](https://github.com/drivercraft/rdrive) 反复报错
	- vendor
	- fdt-parser/sparreal-os 版本更新
    - Opus
- 所有 USB/PHY 寄存器现在与 U-Boot 完全匹配的情况下，端口停留在 **RxDetect 状态 (PLS=5)，CCS=0**（未检测到设备）
	- 补充gl3523的资料
	- gemini-cli 理解原理图，补充 spec
	- OpenCode + oh-my-opencode（GPT-5.2） 长思考 debug

### 经验总结/教训

- 好的 Context：
	- [Getting AI to Work in Complex Codebases](https://github.com/humanlayer/advanced-context-engineering-for-coding-agents/blob/main/ace-fca.md)![[Pasted image 20260112132024.png]]
	- 学习基础
        - 手册
		- [Embedded driver: Build Drivers and Control Dino Game](https://www.udemy.com/course/embedded-driver-build-drivers-and-control-dino-game/?couponCode=CP250105G1)
		- 韦东山 Linux 驱动大全 - USB 驱动专题
		- USB 社区  
			- CherryUSB
	- 缺少必要资料
		- 转换成分章节的 Markdown
		- 图片的理解-> gemini -> 更新文档
	- 缺少对上游的理解
		- vendor（教训：及时沟通）
- 切换模型 GPT-5.2 长思考

## 可以改进的点

- 使用 skill 改进 context management
- 进一步得自动化
	- 示例： https://github.com/yueneiqi/auto-rs

## 使用的session/prompt记录

- [Session Timeline](./CrabUSB_Conversation_Timeline.md)
- [Prompt Patterns](./CrabUSB_Prompt_Patterns.md)
