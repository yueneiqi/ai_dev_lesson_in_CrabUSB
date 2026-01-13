## test

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
