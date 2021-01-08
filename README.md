## Using Bluetooth 5.0 USB Dongle with RTL8761B chipset on Linux kernel 5.10

### Environment
This is the environment that I used for testing:
- Linux kernel 5.10.5 (stock kernel from kernel.org)
- Ubuntu 18.04.5 (Bionic Beaver)

### Firmware
You need the following firmware files from this [Android repository](https://github.com/Realtek-OpenSource/android_hardware_realtek/tree/rtk1395/bt/rtkbt/Firmware/BT):
| Filename | Renamed to |
| -------- | ---------- |
| ```rtl8761b_config``` | ```rtl8761b_config.bin``` |
| ```rtl8761b_fw``` | ```rtl8761b_fw.bin``` |

As of Jan-07-2020, these files are not yet available in [linux-firmware git repository](https://git.kernel.org/pub/scm/linux/kernel/git/firmware/linux-firmware.git/).


These files are also in this repository under [```firmware```](https://github.com/linuxonly1993/rtl8761b_bt_5_linux/tree/main/firmware) directory.

Download these files and put them under directory ```/lib/firmware/rtl_bt/```

Check the SHA256sum of these files:

```sha256sum rtl8761b_config.bin rtl8761b_fw.bin```

```
aa86a092ee58e96256331d5c28c199ceaadec434460e98e7dea20e411e1aa570  rtl8761b_config.bin
0b59a1f2422c006837c4b5e46b59d49bfdbca1defb958adbbc0d57ebdc19cc82  rtl8761b_fw.bin
```
| SHA256 sum | Filename |
| ---------- | -------- |
| ```0b59a1f2422c006837c4b5e46b59d49bfdbca1defb958adbbc0d57ebdc19cc82``` | ```rtl8761b_fw.bin``` |
| ```aa86a092ee58e96256331d5c28c199ceaadec434460e98e7dea20e411e1aa570``` | ```rtl8761b_config.bin``` |

### Problem faced
When using ```hciconfig hci0 up```, I got the following error:

```Can't init device hci0: Operation not supported (93)```

When running ```btmon``` WHILE running ```hciconfig hci0 up```, I saw an error that looks like:

```Status: Unsupported Remote Feature / Unsupported LMP Feature (0x1a)```

Found this commit in [bluetooth-next.git dated 2020-11-25](https://git.kernel.org/pub/scm/linux/kernel/git/bluetooth/bluetooth-next.git/commit/?id=7c66018139629bfd16fe09b982916cc6c814c8d6)

### Kernel patch required (as of kernel 5.10.5)
```c
diff --git a/net/bluetooth/hci_core.c b/net/bluetooth/hci_core.c
index 502552d6e9aff..c4aa2cbb92697 100644
--- a/net/bluetooth/hci_core.c
+++ b/net/bluetooth/hci_core.c
@@ -763,7 +763,7 @@ static int hci_init3_req(struct hci_request *req, unsigned long opt)
 			hci_req_add(req, HCI_OP_LE_CLEAR_RESOLV_LIST, 0, NULL);
 		}
 
-		if (hdev->commands[35] & 0x40) {
+		if (hdev->commands[35] & 0x04) {
 			__le16 rpa_timeout = cpu_to_le16(hdev->rpa_timeout);
 
 			/* Set RPA timeout */
```
This patch is also in this repository under [```patches```](https://github.com/linuxonly1993/rtl8761b_bt_5_linux/tree/main/patches) directory.

I use [kernel_build](https://github.com/sundarnagarajan/kernel_build) to build my kernels.

### dmesg when firmware is loaded correctly
```
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: new full-speed USB device number 9 using xhci_hcd
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: New USB device found, idVendor=0bda, idProduct=8771, bcdDevice= 2.00
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: New USB device strings: Mfr=1, Product=2, SerialNumber=3
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: Product: Bluetooth Radio
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: Manufacturer: Realtek
Jan 07 14:36:35 smaug kernel: usb 3-4.3.2: SerialNumber: 00E04C239987
Jan 07 14:36:35 smaug systemd[1]: Starting Load/Save RF Kill Switch Status...
Jan 07 14:36:35 smaug kernel: Bluetooth: hci0: RTL: examining hci_ver=0a hci_rev=000b lmp_ver=0a lmp_subver=8761
Jan 07 14:36:35 smaug kernel: Bluetooth: hci0: RTL: rom_version status=0 version=1
Jan 07 14:36:35 smaug kernel: Bluetooth: hci0: RTL: loading rtl_bt/rtl8761b_fw.bin
Jan 07 14:36:35 smaug kernel: Bluetooth: hci0: RTL: loading rtl_bt/rtl8761b_config.bin
Jan 07 14:36:35 smaug kernel: Bluetooth: hci0: RTL: cfg_sz 14, total sz 11678
Jan 07 14:36:35 smaug systemd[1]: Started Load/Save RF Kill Switch Status.
Jan 07 14:36:35 smaug systemd[1]: Reached target Bluetooth.
```
      
