<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

USB driver spec: ./domain/usb_driver_spec_orangepi5plus.md

Start by orienting yourself:

```bash
# 1. See your working directory
pwd

# 2. List files to understand project structure
ls -la

# 3. Read the project specification to understand what you're building
cat ./domain/usb_driver_spec_orangepi5plus.md

# 4. Read the current status
cat status.md

# 5. Read progress notes from previous sessions
cat claude-progress.txt

# 6. Check recent git history
cd CrabUSB && git log --oneline -20
```

Understanding the USB driver spec is critical - it contains the full requirements
for the driver you're building.

track progress in `./claude-progress.txt`
Update `claude-progress.txt` with:
- What you accomplished this session
- Which test(s) you completed
- Any issues discovered or fixed
- What should be worked on next
- Current completion status

可使用本地的`rdrive`
依赖的PMU驱动在`rockchip-pm`
可使用的 clk 驱动在`rk3588-clk`:
可使用的 CRU/复位/GRF 实现参考 `rockchip-soc`:
- `rockchip-soc/src/variants/rk3588/cru/mod.rs`: `Cru` 提供 `clk_enable/disable/get_rate/set_rate` + `reset_assert/deassert`，`init()` 只做 u-boot 配置校验不改寄存器
- `rockchip-soc/src/variants/rk3588/cru/clock/mod.rs`: USB 相关 `ClkId`（`ACLK_USB_ROOT`, `HCLK_USB_ROOT`, `CLK_UTMI_OTG2`, `ACLK_USB3OTG0/1/2`, `SUSPEND_CLK_USB3OTG0/1/2`, `REF_CLK_USB3OTG0/1/2`, `HCLK_HOST0/1`, `HCLK_HOST_ARB0/1`, `PCLK_USBDPPHY0/1`, `USBDP_PHY0/1_IMMORTAL` 等）
- `rockchip-soc/src/variants/rk3588/cru/gate.rs`: USB gate 表（`ACLK_USB_ROOT` reg_idx=42 bit0，`HCLK_USB_ROOT` reg_idx=42 bit1，`ACLK_USB3OTG0` reg_idx=42 bit4，`ACLK_USB3OTG1` reg_idx=42 bit7，`ACLK_USB3OTG2` reg_idx=35 bit7；`PCLK_USBDPPHY0` reg_idx=72 bit2，`PCLK_USBDPPHY1` reg_idx=72 bit4，`USBDP_PHY0/1_IMMORTAL` reg_idx=2 bit8/15）
- `rockchip-soc/src/variants/rk3588/syscon.rs`: `grf_mmio` 包含 `USB_GRF`, `USB2PHY*_GRF`, `USBDPPHY*_GRF`, `PHP_GRF` 等 GRF base

你可以参考:
- 适配orangepi5plus的linux源码：`linux-orangepi`
- uboot源码：`u-boot`
- 设备树：`orangepi5plus.dts`
- GL3523的相关资料：`gl3523/GL3523_Product_Info.md`:
- rk3588的相关资料：`rk3588_doc_md`:
```
➜ tree rk3588_doc_md
.
├── rk3588_doc1
│   ├── Chapter_10_Timer.md
│   ├── Chapter_11_GIC600.md
│   ├── Chapter_12_DMA_Controller_DMAC.md
│   ├── Chapter_13_SDMMC_BUFFER.md
│   ├── Chapter_14_Temperature-Sensor_ADC_TS-ADC.md
│   ├── Chapter_15_Debug.md
│   ├── Chapter_16_Mailbox.md
│   ├── Chapter_17_Watchdog_TimerWDT.md
│   ├── Chapter_18_Pulse_Width_Modulation_PWM.md
│   ├── Chapter_19_UART.md
│   ├── Chapter_1_System_Overview.md
│   ├── Chapter_20_GPIO.md
│   ├── Chapter_21_I2C_Interface.md
│   ├── Chapter_22_I2S.md
│   ├── Chapter_23_SPDIF_Transmitter.md
│   ├── Chapter_24_SPDIF_Receiver.md
│   ├── Chapter_25_Gigabit_Media_Access_Controller_GMAC.md
│   ├── Chapter_26_SAR-ADC.md
│   ├── Chapter_27_Digital_Audio_Codec.md
│   ├── Chapter_28_Voice_Activity_Detect_VAD.md
│   ├── Chapter_29_CAN.md
│   ├── Chapter_2_CRU.md
│   ├── Chapter_30_Flexible_Serial_Peripheral_Interface_FSPI_.md
│   ├── Chapter_31_Serial_Peripheral_Interface_SPI.md
│   ├── Chapter_32_SPINLOCK.md
│   ├── Chapter_33_KEYLAD.md
│   ├── Chapter_34_Share_Memory.md
│   ├── Chapter_35_DECOM.md
│   ├── Chapter_36_RKNN.md
│   ├── Chapter_37_Video_Capture_VICAP.md
│   ├── Chapter_38_FishEye_CorrectionFEC.md
│   ├── Chapter_39_Pulse_Density_Modulation_Interface_Controller.md
│   ├── Chapter_3_CPU.md
│   ├── Chapter_4_Graphics_Process_Unit_GPU.md
│   ├── Chapter_5_VPU.md
│   ├── Chapter_6_General_Register_Files_GRF.md
│   ├── Chapter_7_By_Section
│   │   ├── Chapter_7_1_Overview.md
│   │   ├── Chapter_7_2_Block_Diagram.md
│   │   ├── Chapter_7_3_Function_Description.md
│   │   ├── Chapter_7_4_Register_Description.md
│   │   ├── Chapter_7_5_1_Low_Power_Mode.md
│   │   ├── Chapter_7_5_2_Nested_Power_Domain.md
│   │   ├── Chapter_7_5_3_Software_Power_Down_and_Power_Up_flow.md
│   │   ├── Chapter_7_5_4_Memory_Power_Control.md
│   │   ├── Chapter_7_5_5_Power_Management_IO_Usage.md
│   │   ├── Chapter_7_5_6_Debug_Information.md
│   │   ├── Chapter_7_5_7_System_Register.md
│   │   └── Chapter_7_5_Application_Notes.md
│   ├── Chapter_7_Power_Management_Unit_PMU.md
│   ├── Chapter_8_MMU600.md
│   └── Chapter_9_MCU_Subsystem.md
└── rk3588_doc2
    ├── 01_Chapter_1_Interconnect.md
    ├── 02_Chapter_2_Dynamic_Memory_Interface_(DMC).md
    ├── 03_Chapter_3_Mobile_Storage_Host_Controller.md
    ├── 04_Chapter_4_eMMC_Host_Controller.md
    ├── 05_Chapter_5_Raster_Graphic_Acceleration_3_(RGA3).md
    ├── 06_Chapter_6_Raster_Graphic_Acceleration(RGA2).md
    ├── 07_Chapter_7_Video_Output_Processor_(VOP).md
    ├── 08_Chapter_8_Image_Enhancement_Processor（IEP).md
    ├── 09_Chapter_9_DisplayPort_Transmitter_Controller_(DPTX).md
    ├── 10_Chapter_10_CRYPTO.md
    ├── 11_Chapter_11_PCIe_Controller.md
    ├── 12_Chapter_12_USB2.0_Host_&_PHY.md
    ├── 13_Chapter_13_USB3_Controller.md
    ├── 14_Chapter_14_USBDP_COMBO_PHY.md
    ├── 15_Chapter_15_SATA_Host.md
    ├── 16_Chapter_16_Multiple_Protocol_PHY_Subsystem.md
    ├── 17_Chapter_17_Process-Voltage-Temperature_Monitor_(PVTM).md
    ├── 18_Chapter_18_PVTPLL.md
    ├── 19_Chapter_19_MIPI_CSI_HOST.md
    ├── 20_Chapter_20_MIPI_CSI_DPHY.md
    ├── 21_Chapter_21_MIPI_DSI-2_Host_Controller.md
    ├── 22_Chapter_22_MIPI_D-PHY___C-PHY_Combo_PHY.md
    ├── 23_Chapter_23_EDP_TX_Controller.md
    ├── 24_Chapter_24_HDMI_TX_Controller.md
    ├── 25_Chapter_25_HDMI_RX_Controller.md
    ├── 26_Chapter_26_HDCP2.3_Controller.md
    ├── 27_Chapter_27_HDMI_TX_eDP_Combo_PHY.md
    └── 28_Chapter_28_HDMI_RX_PHY.md
```

## Requirements

### test

qemu test
```
bash $SCRIPT_PATH/run_crabusb_qemu.sh | tee ./log/$(date +%Y%m%d_%H%M%S)_qemu.log
```

broad test command:
```bash
# no `usb start` uboot cmd
bash $SCRIPT_PATH/run_crabusb_uboot.sh | tee $LOG_PATH/$(date +%Y%m%d_%H%M%S)_crabusb.log

# with `usb start` uboot cmd
bash $SCRIPT_PATH/run_crabusb_uboot_usb_start.sh | tee $LOG_PATH/$(date +%Y%m%d_%H%M%S)_crabusb_usb_start.log
```

**Do NOT run test command via cargo** 

### Session

Before context fills up:
1. Commit all working code in CrabUSB
2. Update claude-progress.txt
3. Ensure no uncommitted changes in CrabUSB
4. Leave driver in working state (no broken features)
5. Commit all the changes in domain
