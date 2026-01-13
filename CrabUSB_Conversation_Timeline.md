# CrabUSB Development Conversation Timeline

This document consolidates all OpenCode, Claude Code, and Codex conversations related to CrabUSB USB driver development for Orange Pi 5 Plus (RK3588).

---

## Overview

**Project**: CrabUSB - Rust xHCI USB Host Driver for RK3588  
**Target Hardware**: Orange Pi 5 Plus (Rockchip RK3588)  
**Goal**: Enable USB ports without relying on U-Boot `usb start` command

### Key Hardware Components
- **Port 0 (USB3_0)**: Type-C connector at `0xFC000000`
- **Port 1 (USB3_1)**: Connected to GL3523 USB Hub at `0xFC400000`
- **USBDP Combo PHY**: USB3/DisplayPort combo PHY
- **DWC3 Controller**: Synopsys DesignWare USB3 controller

---

## Session Summary

| Source | Sessions | Date Range |
|--------|----------|------------|
| **Codex** | 30+ sessions | Nov 21 - Dec 23, 2025 |
| **Claude Code** | 17 sessions | Dec 22, 2025 - Jan 9, 2026 |
| **OpenCode** | 15+ sessions | Jan 8-13, 2026 |

---

## Timeline

### November 21, 2025 (Codex)

#### Session: 019aa3fd-4eb3-7500-86ab-df96c1f13bba (01:18 UTC)
**Topic**: Initial CrabUSB debugging  
**First Message**: `find out why when run cargo test -p crab-usb --test test --target aarch64-unknown-none-softfloat -- --show-output, it output: kernel panic: no xhci found`

#### Session: 019aa5a2-d98e-75c2-8860-d4b120dfb2c9 (08:59 UTC)
**Topic**: Build system setup  
**First Message**: `based on the making uimg logic in arceos, add a shell script in CrabUSB/ to make a uimg`

---

### November 22, 2025 (Codex)

#### Session: 019aaa64-8556-7161-b056-e9520bcc9bb6 (07:08-07:24 UTC)
**Topic**: Kernel boot debugging on Orange Pi 5 Plus  
**Key Work**: Debugging kernel boot issues, FDT loading, memory mapping

#### Session: 019aaac3-f100-7e01-ad0e-cdff59bd5c26 (08:52 UTC - Nov 23)
**Topic**: QEMU vs Hardware comparison  
**First Message**: `在CrabUSB中，在qemu上测试通过；在搭载rk3588的orangepi5plus上测试失败`  
**Key Work**: Comparing QEMU success vs hardware failure

---

### November 23, 2025 (Codex)

#### Session: 019aafed-9c38-7b42-bfad-405343b08c29 (08:56 UTC)
**Topic**: Continue hardware debugging

#### Session: 019ab0e3-c054-77d3-9215-0e2ddb98ef08 (13:26-14:20 UTC)
**Topic**: Hardware boot debugging  
**Key Work**: Kernel startup sequence analysis

---

### November 24, 2025 (Codex) - Major Development Day

#### Session: 019ab4cb-5ee1-7b01-bb2e-daa2baf13d0d (07:37 UTC)
**Topic**: Initial investigation

#### Session: 019ab4cc-0fda-7b70-8eba-332e44f0ed95 (07:37-08:13 UTC)
**Topic**: QEMU vs Hardware comparison  
**Key Work**: Fixing compilation errors, analyzing test results

#### Session: 019ab50c-8c70-7c70-ab28-b850b5c3aedc (08:48-11:25 UTC)
**Topic**: FDT parsing fixes  
**Key Work**: 
- Fixed `error[E0599]: no method named 'parent' found for struct 'Node'`
- Fixed `error[E0616]: field '0' of struct 'Phandle' is private`

#### Session: 019ab5a3-7a8d-71a1-a1cd-e11c5cb7c9a1 (11:33-12:55 UTC)
**Topic**: Iterator and clone fixes  
**Key Work**: Fixed `error[E0599]: no method named 'clone' found for opaque type`

#### Session: 019ab624-8990-73e1-80af-ac30781fae83 (13:54-14:04 UTC)
**Topic**: rockchip-pm integration  
**Key Work**:
- Copied rockchip-pm into CrabUSB/module-local
- Analyzed port status: `Port 0: Enabled: true, Connected: true, Speed 3`

#### Session: 019ab634-542b-77d0-82b0-e9190740aaad (14:11-14:19 UTC)
**Topic**: Fix stuck problem on Orange Pi 5 Plus  
**Key Work**: Fixed `error[E0609]: no field 'cfgs' on type 'Option<IrqInfo>'`

---

### November 25, 2025 (Codex)

#### Session: 019ab8dc-ab97-79a0-8e76-815b62c99e8f (02:37 UTC)
**Topic**: Git commit organization  
**Key Work**: Split commits from rockchip-pm

#### Session: 019ab9f5-8788-72d2-8095-86505cef14a9 (07:42-08:48 UTC)
**Topic**: rockchip-pm version update  
**Key Work**: 
- Updated to use new rockchip-pm API
- Switched to GitHub dependency: `https://github.com/drivercraft/rockchip-pm.git`

#### Session: 019aba2a-8597-7032-8f42-a0ce7a79b883 (08:39-08:40 UTC)
**Topic**: Board panic debugging  
**Key Work**: Investigating kernel panic on Orange Pi 5 Plus

#### Session: 019aba41-5f01-7f32-ad35-f4a1244a0ebe (09:05-09:10 UTC)
**Topic**: rockchip-pm update

#### Session: 019aba52-3770-77b0-b666-7bbd837faa91 (09:29 UTC)
**Topic**: Fix regression  
**Key Work**: Restored `if domain != PD_USB.0 as u32` check

#### Session: 019abaab-fdbd-7863-8191-6e764a853a30 (11:03 UTC)
**Topic**: rockchip-pm API investigation

#### Session: 019abaaf-575d-75b3-b474-7b43902d53be (11:04-11:14 UTC)
**Topic**: Refactor rockchip-pm usage  
**Key Work**: Pinned rockchip-pm to version 0.3.3

---

### November 26, 2025 (Codex)

#### Session: 019abfdf-1c4f-73e1-a193-08613a46814d (11:15 UTC)
**Topic**: New issue from latest commit

#### Session: 019abfea-1fba-7fd2-b1a6-9398c85fb705 (11:26-11:33 UTC)
**Topic**: USB start dependency analysis  
**First Message**: `目前，在uboot启动命令中加入usb start后，CrabUSB的测试可顺利连接USB`  
**Key Finding**: USB works WITH `usb start`, fails WITHOUT it

#### Session: 019abffd-bcf6-7301-ba98-6175e5a10293 (11:47 UTC)
**Topic**: Continue USB start investigation

---

### November 27, 2025 (Codex)

#### Session: 019ac31f-d053-7c83-890b-8a39a8972214 (02:23 UTC)
**Topic**: Progress update  
**First Message**: `We're making progress!`

---

### December 3, 2025 (Codex)

#### Session: 019ae40d-1eee-7251-84cd-e60543a3c8fb (11:51 UTC)
**Topic**: USB driver specification  
**First Message**: `read usb driver related files in u-boot/ and linux-orangepi and rk3588_doc_md, write a usb driver spec for orangepi5plus`

#### Session: 019ae425-09e6-7312-b28e-0333d6e0fdd3 (12:29-12:40 UTC)
**Topic**: Git commit and rdrive investigation  
**Key Work**: Investigating rdrive initialization issues

---

### December 5, 2025 (Codex) - OpenSpec Implementation

#### Session: 019aee40-2da8-74e2-b8fc-274e6056e3ff (11:23-12:07 UTC)
**Topic**: Apply OpenSpec change  
**First Message**: `apply the OpenSpec change of openspec/changes/remove-uboot-usb-start-dependency to CrabUSB`  
**Key Work**: Multiple hardware test iterations with log analysis

#### Session: 019aee7c-e889-78b0-8be8-b2d49cd7731e (12:30-12:38 UTC)
**Topic**: Hardware log analysis  
**Key Work**: Analyzing `log/20251205_202722_crabusb.log` and subsequent logs

#### Session: 019aee8d-0547-7a22-a76e-6e624fea7747 (12:47-13:05 UTC)
**Topic**: Hardware debugging  
**Key Work**: 
- Analyzed multiple hardware logs
- **SUCCESS**: `it works, read 20251205_210252_crabusb.log`

#### Session: 019aeea2-5669-7962-8f8f-73f66586e1c5 (13:10-13:28 UTC)
**Topic**: Continue OpenSpec implementation  
**Key Work**: VBUS/clock diagnostics verification

#### Session: 019aeee0-463f-7c60-b2b3-56a9d5e5b6ae (14:26-14:29 UTC)
**Topic**: Git commit and progress update

---

### December 6, 2025 (Codex) - Intensive Hardware Debugging

#### Session: 019aeeed-1eb7-7093-81b6-4a42eff02db2 (02:34 UTC)
**Topic**: Hardware log analysis  
**Key Work**: Analyzing `log/20251206_103358_crabusb.log`

#### Session: 019af197-c55a-7570-a479-6b2d6157669e (03:18-03:27 UTC)
**Topic**: Ports disconnected without `usb start`  
**Key Work**: Analyzing `log/20251206_112524_crabusb.log`

#### Session: 019af1b5-7c81-75d0-9504-4c45e4b2aaa6 (04:35-04:49 UTC)
**Topic**: Hardware debugging  
**Key Work**: Multiple log analyses (123336, 124744)

#### Session: 019af1ff-a9ed-70a1-93dd-a71e00e973dc (05:09-05:24 UTC)
**Topic**: Hardware debugging  
**Key Work**: Multiple log analyses (130912, 131713, 132210)

#### Session: 019af222-60f2-76b0-b01f-7a0a093f9859 (05:48 UTC)
**Topic**: Hardware log analysis

#### Session: 019af245-c9ea-7362-b844-597b7311176b (06:07 UTC)
**Topic**: Fix stuck problem  
**Key Work**: Fixing issue from commit `eb1302f22708a8179f27119433f5410a2cb7f3a7`

#### Session: 019af28f-659d-7eb1-8d64-f8cb3f6a8d3c (08:09-08:21 UTC)
**Topic**: Hardware debugging  
**Key Work**: Multiple log analyses (160858, 161636, 162115)

#### Session: 019af372-3abe-73d3-bc86-f927c8d4fef2 (12:09 UTC)
**Topic**: Hardware log analysis  
**Key Work**: Analyzing `log/20251206_200819_crabusb.log`

#### Session: 019af3a2-4922-7680-a4a1-482725615dec (12:28 UTC)
**Topic**: Bug fix

---

### December 7, 2025 (Codex)

#### Session: 019af467-c3fa-72e1-b26e-88296e60f4bf (01:57 UTC)
**Topic**: Git commit

#### Session: 019af771-7353-7922-bf49-b3f58b2a1d3c (06:14-08:09 UTC)
**Topic**: Focus on USB3 (xHCI) and Port 0  
**Key Work**: Created handoff plan for session continuation

#### Session: 019af98e-6d99-7fd1-957f-1f91e59cd1ce (16:05 UTC)
**Topic**: U-Boot analysis  
**First Message**: `explain in uboot what does xhci_dwc3_reset_init(dev, plat) do?`

---

### December 8, 2025 (Codex)

#### Session: 019afb81-e0e5-76d0-b9dd-70b32a80ef74 (01:12-04:50 UTC)
**Topic**: Hardware debugging  
**Key Work**: 
- Multiple hardware log analyses (122214, 122927, 124113)
- Created handoff plan

#### Session: 019afc5a-b8fb-7d40-a9fc-e9cc01157d07 (05:06-05:56 UTC)
**Topic**: Resume from handoff  
**Key Work**: 
- Hardware log analysis (132233, 135351)
- DTS configuration investigation

---

### December 22, 2025 (Claude Code + Codex)

#### Claude Code Session: 5fb0ba38 (08:17 UTC)
**Topic**: Initial setup  
**First Message**: `fix uv run ./start.py`

#### Claude Code Session: a30d61c3 (12:54 UTC) - CrabUSB Directory
**Topic**: Test command explanation  
**First Message**: `how does cargo test -p crab-usb --test test --target aarch64-unknown-none-softfloat -- --show-output work?`

#### Claude Code Session: 684803b8 (13:13 UTC) - CrabUSB Directory
**Topic**: Serial port fix  
**First Message**: `write a shell script to fix OSError: [Errno 16] Device or resource busy: '/dev/ttyUSB0'`

#### Claude Code Session: 8428ecee (14:05 UTC)
**Topic**: Task initiation  
**First Message**: `do tasks in @status.md`

#### Claude Code Session: a3218387 (14:40 UTC) - Major Session
**Topic**: USB driver migration from U-Boot  
**First Message**: `do task in @status.md`

**Key Work**:
- Started migrating U-Boot `usb start` XHCI logic to CrabUSB
- Analyzed `xhci-dwc3.c` probe sequence
- Identified 11-step initialization process

#### Codex Session: 019b4935-4f5b-7c22-a2bc-8adcc3fb53f4 (Dec 23, 03:35 UTC)
**Topic**: Kernel panic debugging  
**First Message**: `result of script/run_crabusb_uboot.sh: kernel panic`

---

### December 23, 2025 (Claude Code)

#### Session: 928ee09a (10:40 UTC)
**Topic**: Continued development

---

### January 6, 2026 (Claude Code)

#### Session: d6263692 (14:32 UTC)
**Topic**: CrabUSB development

#### Session: 4a34f1e1 (14:33 UTC)
**Topic**: CrabUSB development

#### Session: ce3283eb (14:42 UTC)
**Topic**: CrabUSB development

---

### January 7, 2026 (Claude Code)

#### Session: c53f8e17 (01:28 UTC)
**Topic**: Pickup/continuation

#### Session: 8876b70f (01:36 UTC) - Major Session
**Topic**: Pickup/continuation  
**Duration**: Extended session (until 19:21)

---

### January 8, 2026 (OpenCode)

#### Session: ses_463afc187ffenfr1umWMv2OxDD (06:34-15:54 UTC)
**Messages**: 1247  
**Duration**: ~9 hours  
**Agents**: Sisyphus, compaction, general, Planner-Sisyphus

**Topic**: Finish task to enable USB ports without U-Boot `usb start`

**Key Work**:
- Analyzed U-Boot xhci-dwc3.c probe sequence
- Created DWC3 module for CrabUSB
- Implemented DWC3 core initialization
- Added `init_dwc3_if_needed` method to XHCI backend

**Key Findings**:
- DWC3 registers at offset 0xC100 from XHCI base
- Need to handle PRTCAPDIR bits for host/device mode
- USB2 PHY quirks: dis_enblslpm, dis-u2-freeclk-exists, dis_u2_susphy

#### Session: ses_461958239ffe774zg5eGxTdA2J (16:22 UTC - Jan 9 09:55)
**Messages**: 106  
**Duration**: ~17 hours (overnight)  
**Agents**: Sisyphus

**Topic**: Continue work on Hub Discovery & USBDP PHY

**Key Work**:
- Fixed broken `enable_usb_bvalid()` function
- Restored truncated code from bad merge
- Investigated PCS initialization

**Progress Notes**:
- GL3523 USB Hub detected on USB 2.0 (Port 2)
- Port 1 (USB 3.0) remains in RxDetect (CCS=0)
- PHY Init runs successfully (LCPLL Locked, CDR Locked)

---

### January 9, 2026 (OpenCode + Claude Code)

#### OpenCode Session: ses_45f11dc08ffeS6dTkID687WyLa (04:05-05:00 UTC)
**Messages**: 384  
**Duration**: ~1 hour  
**Agents**: Sisyphus, compaction

**Topic**: Code Cleanup & USB3 Investigation

**Commits Made**:
1. `9b6344c` - fix: restore bvalid enable and remove broken UsbDpPhy reference
2. `f37a680` - phy: try flip mode (0x30) for USBDP lane mux
3. `491761f` - dwc3: clear PHY soft reset after XHCI HCRST

#### OpenCode Session: ses_45d36f01fffeKvtsnvgGUwTB9Y (12:44 UTC - Jan 11 02:27)
**Messages**: 2655  
**Duration**: ~38 hours (multi-day)  
**Agents**: Sisyphus, compaction, general

**Topic**: Enable Port 1 (USB3_1) without U-Boot `usb start`

**Key Work**:
- Created dedicated USBDP PHY module for RK3588
- Implemented full PHY initialization sequence
- Integrated PHY init into XHCI init flow

**Hardware Addresses Identified**:
- XHCI Controller: `0xFC400000`
- USBDP PHY1 PMA: `0xFED98000`
- USBDPPHY1_GRF: `0xFD5CC000`
- USB2PHY1_GRF: `0xFD5D4000`
- USB_GRF: `0xFD5AC000`

#### Claude Code Sessions (08:37-17:09 UTC)
Multiple short test sessions for continuation

---

### January 11, 2026 (OpenCode)

#### OpenCode Session: ses_4527e796cffe2cMl0gHWQ9Pj4d (CrabUSB Directory)
**Messages**: 34  
**Agents**: Sisyphus

**Topic**: Dead code cleanup  
**First Message**: `review latest three commits, remove dead code`

**Key Work**:
- Analyzed dead code in dwc3.rs, rk3588_phy.rs, root.rs
- Removed unused `Dwc3` struct, `Dwc3Regs`, `Dwc3Quirks`, `Dwc3Mode`
- Kept only `is_dwc3_xhci()` function
- Cleaned up rk3588_phy.rs to keep only VBUS toggle functions

#### OpenCode Session: ses_452884ac2ffeAdMUPUM4PnYOCj (CrabUSB Directory)
**Messages**: 27  
**Agents**: Sisyphus

**Topic**: Git commit organization  
**First Message**: `从 786d04cf 到 a7ccd1c2, commits 太多了，整理commits`

**Key Work**:
- Reorganized 38 commits into 3 logical commits:
  1. `feat(rk3588): add DWC3 core and USBDP/USB2 PHY initialization`
  2. `feat(rk3588): add VBUS GPIO toggle for GL3523 hub workaround`
  3. `test(rk3588): add PHY initialization and VBUS toggle tests`

#### OpenCode Session: ses_4526e8238ffenVRVqpP1w0LR42 (CrabUSB Directory)
**Messages**: 3  
**Agents**: Sisyphus

**Topic**: PR title generation  
**First Message**: `based on the latest 4 commits, give me a pr title in chinese`  
**Result**: `feat(rk3588): 添加 DWC3 控制器及 USB PHY 初始化支持`

#### Codex Session: 019bac9a-5ec2-75f3-b4e5-3bdac8190ea3 (10:30 UTC)
**Topic**: Test code cleanup  
**First Message**: `clean up CrabUSB/usb-host/tests/test.rs, make it production-ready`

#### Multiple OpenCode Sessions
- ses_45397eb3dffeDFvZ0OKDDfJRHN (172 messages)
- ses_453d7e4c9ffez3qyPGfnNqUzRT (107 messages)
- ses_453e8f76dffeTln1YW4qIpJCcd (42 messages)
- ses_4542e9023ffevmErDkQi9IiJAJ (61 messages)
- ses_454409e33ffeTug7UjGxHDKwBT (24 messages)
- ses_454e87389ffev31O8sbybuoYI9 (34 messages)
- ses_4536a25ddffe8Hpb2TkOxGyGiY (16 messages)

---

### January 13, 2026 (OpenCode)

#### Session: ses_4490c4739ffe40Ck4QFfL2LiWz (Current)
**Agents**: Sisyphus

**Topic**: Consolidate conversation history

---

## Key Technical Discoveries

### 1. DWC3 Controller Initialization
The DWC3 (DesignWare USB3) controller requires specific initialization:
- Core soft reset via GCTL register
- PHY soft reset sequence
- Mode configuration (Host/Device/OTG)
- USB2 PHY quirks from device tree

### 2. USBDP Combo PHY
RK3588 uses USB3/DP combo PHY requiring:
- PMA (Physical Medium Attachment) initialization
- PCS (Physical Coding Sublayer) - initially thought missing but U-Boot doesn't use it
- Lane mux configuration (Normal: 0xC0, Flip: 0x30)
- LCPLL and CDR lock verification

### 3. GRF (General Register Files) Configuration
Multiple GRF blocks control USB:
- `USB_GRF` (0xFD5AC000): USB controller configuration
- `USBDPPHY_GRF` (0xFD5CC000): PHY configuration
- `USB2PHY_GRF` (0xFD5D4000): USB2 PHY configuration
- `PHP_GRF` (0xFD5B0000): PCIe/USB/SATA hybrid PHY

### 4. BVALID Override
For host mode without Type-C controller:
- Need to force VBUS valid through GRF registers
- `usbctrl-grf` bit14 indicates host status
- `u2phy-grf` CON2/CON4 for VBUS override

### 5. Port Status Interpretation
- PLS=4 (Disabled): Port powered but no link
- PLS=5 (RxDetect): Waiting for device
- PLS=0 (U0): Link active
- CCS=0: No device connected
- CCS=1: Device connected

---

## Files Modified

### CrabUSB Core
- `usb-host/src/backend/xhci/mod.rs` - XHCI backend with DWC3 init
- `usb-host/src/backend/xhci/dwc3.rs` - DWC3 controller module
- `usb-host/src/backend/xhci/root.rs` - Root hub port handling
- `usb-host/src/backend/xhci/rk3588_phy.rs` - USBDP PHY driver

### Test Files
- `usb-host/tests/test.rs` - Hardware test with PHY init

### Documentation
- `domain/usb_driver_spec_orangepi5plus.md` - Driver specification
- `domain/claude-progress.txt` - Session progress notes

---

## Current Status (as of January 11, 2026)

### Working
- ✅ USB 2.0 detection (GL3523 Hub detected)
- ✅ PHY initialization (LCPLL/CDR lock)
- ✅ DWC3 core initialization
- ✅ VBUS GPIO control
- ✅ Power domain enable

### In Progress
- 🔄 USB 3.0 link training (Port 1 stays in RxDetect)
- 🔄 Full device enumeration without U-Boot

### Not Started
- ❌ Port 0 Type-C support (needs FUSB302 I2C driver)
- ❌ Hub downstream port enumeration

---

## References

- RK3588 Technical Reference Manual
- Orange Pi 5 Plus Schematics (V1.3.1)
- U-Boot source: `drivers/usb/host/xhci-dwc3.c`
- U-Boot source: `drivers/phy/rockchip/phy-rockchip-usbdp.c`
- Linux source: `drivers/phy/rockchip/phy-rockchip-usbdp.c`
- CrabUSB source code

---

## Data Sources

### OpenCode Sessions
Location: `~/.local/share/opencode/sessions/`  
Query tool: `session_list`, `session_read`, `session_search`

### Claude Code Sessions
Location: `~/.claude/projects/-home-seven-Workspace-opi-workspace-misc*/`  
Format: JSONL files with session UUIDs

### Codex Sessions
Location: `~/.codex/history.jsonl` and `~/.codex/sessions/`  
Format: JSONL with session_id and timestamp
