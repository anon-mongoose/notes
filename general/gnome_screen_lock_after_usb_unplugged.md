# How to make UDEV rules on Ubuntu 22.04 (with GNOME as desktop environment) to lock the user's screen when a Yubikey or Nitrokey Pro 2 are unplugged

## Requirements
- Ubunttu 22.04.
- GNOME as desktop environment.
- A Yubikey, Nitrokey Pro 2 or both (actually any USB device should be could).
- Be able to plug/unplug USB device on the target system.


## 1. Create both UDEV rules

### Yubikey rule
Create and edit a new rule file:
```bash
sudo vim /etc/udev/rules.d/85-yubikey.rules
```

Paste the following:
```
ACTION=="remove", ENV{PRODUCT}=="1050/407/543" RUN+="/usr/local/bin/gnome-screensaver-lock"
```

*Note: you may want to update `ENV{PRODUCT}` value in order to match the one of your device.*

### Nitrokey Pro 2 rule
Create and edit a new rule file:
```bash
sudo vim /etc/udev/rules.d/85-nitrokey.rules
```

Paste the following:
```
ACTION=="remove", ENV{PRODUCT}=="20a0/4108/101" RUN+="/usr/local/bin/gnome-screensaver-lock"
```

*Note: you may want to update `ENV{PRODUCT}` value in order to match the one of your device.*

### Load these new rules
```bash
sudo udevadm control --reload-rules && sudo udevadm trigger
```

## 2. Create the bash script to run in order to lock the screen
Create a the bash script with the same name as referenced above:
```bash
sudo vim /usr/local/bin/gnome-screensaver-lock
```

Change the script's permissions:
```bash
sudo chmod 700 /usr/local/bin/gnome-screensaver-lock
```

Paste the following lines in it:
```bash
#!/bin/bash
user=`ps axo user:30,comm | egrep "gdm-(wayland|x)" | awk '{print $1}'`

if [ -n ${user} ]; then
        systemd-run --quiet --pipe --wait --user --machine ${user}@.host /usr/bin/dbus-send --type=method_call --dest=org.gnome.ScreenSaver /org/gnome/ScreenSaver org.gnome.ScreenSaver.Lock
fi
```

*Note: the first line `#!/bin/bash` is quiet  important, do not remove it.*

Finally, test the script to ensure it works properly:
```bash
sudo /usr/local/bin/gnome-screensaver-lock
```


## 3. Test the rules
Simply plug then unplug the keys that will trigger the rules above. If it does not work, consider reading the [troubleshooting section](#troubleshooting).



## Troubleshooting

### How to find the PRODUCT value to specify in UDEV rules?
The best way is to check the website/documentation of the key's manufacturer. If you do not find it (nor on the Internet), you can try to find it with the process below:

1. Open a terminal and run the command below in order to print some related to the keys you will plug:
    ```bash
    sudo dmesg --follow-new
    ```

2. Plug in your machine the key you want to trigger one of the UDEV rules defined above. You should see the following lines printed by the command above:
    For (my) Yubikey:
    ```
    [14683.285225] usb 3-2: new full-speed USB device number 57 using xhci_hcd
    [14683.504280] usb 3-2: New USB device found, idVendor=1050, idProduct=0407, bcdDevice= 5.43
    [14683.504295] usb 3-2: New USB device strings: Mfr=1, Product=2, SerialNumber=0
    [14683.504300] usb 3-2: Product: YubiKey OTP+FIDO+CCID
    [14683.504304] usb 3-2: Manufacturer: Yubico
    [14683.509352] input: Yubico YubiKey OTP+FIDO+CCID as /devices/pci0000:00/0000:00:14.0/usb3/3-2/3-2:1.0/0003:1050:0407.0039/input/input36
    [14683.570437] hid-generic 0003:1050:0407.0039: input,hidraw1: USB HID v1.10 Keyboard [Yubico YubiKey OTP+FIDO+CCID] on usb-0000:00:14.0-2/input0
    [14683.572353] hid-generic 0003:1050:0407.003A: hiddev0,hidraw2: USB HID v1.10 Device [Yubico YubiKey OTP+FIDO+CCID] on usb-0000:00:14.0-2/input1
    ```
    The info we are looking for is located on the last line: `hidraw2`.


3. Print info related to the key by specifying the path `/dev/XXXXX` where XXXXX refers to the value found previously.
    For (my) Yubikey:
    ```
    sudo udevadm info -q all -a /dev/hidraw2
    ```

    *Note: if the digit in `/dev/hidrawX* (where X is the digit) does not seem to exist, try another one and check the printed info to find to which device it refers to until you find the good one.*

    Look for the following terms:
    - `idVendor`: in my case, it is equal to `1050`.
    - `idProduct`: in my case, it is equal to `0407`.
    - `bcdDevice`: in my case, it is equal to `0543`.

    Remove the leading `O` for each, then join them in this specific order with `/` as a separator to build the PRODUCT value: `1050/407/543`.


### The test calls the the script, but it does not do anything in practice!

1. Find the path of the device ([see above](#how-to-find-the-product-value-to-specify-in-udev-rules)) by running the following command:
    ```bash
    # Note: in our example, the device path is /dev/hidraw2. You should specify the one linked to your own device.

    sudo udevadm info -q all -a /dev/hidraw2
    ```

    Below is my truncated output:
    ```
    Udevadm info starts with the device specified by the devpath and then
    walks up the chain of parent devices. It prints for every device
    found, all possible attributes in the udev rules key format.
    A rule to match, can be composed by the attributes of the device
    and the attributes from one single parent device.

      looking at device '/devices/pci0000:00/0000:00:14.0/usb3/3-2/3-2:1.1/0003:1050:0407.003D/hidraw/hidraw2':
        KERNEL=="hidraw2"
        SUBSYSTEM=="hidraw"
        DRIVER==""
        ATTR{power/async}=="disabled"
        ATTR{power/control}=="auto"
        ATTR{power/runtime_active_kids}=="0"
        ATTR{power/runtime_active_time}=="0"
        ATTR{power/runtime_enabled}=="disabled"
        ATTR{power/runtime_status}=="unsupported"
        ATTR{power/runtime_suspended_time}=="0"
        ATTR{power/runtime_usage}=="0"

    .......
    ```

    Take note of the first device path, in my case `/devices/pci0000:00/0000:00:14.0/usb3/3-2/3-2:1.1/0003:1050:0407.003D/hidraw/hidraw2`.



2. Test the device rules when removing it:
    You actually does not have to unplug your device, the test command allows you to simulate this behavior with the `--action=remove` option.

    Run the command `sudo udevadm test --action=remove /path/to/device` where `/path/to/device` is the device path found at the previous step. It should prints this line somewhere at the end of the output.
    ```
    run: '/usr/local/bin/gnome-screensaver-lock'
    ```

    It means the rule works (= was called) as expected. But if the screen does not lock when unplugging the keys, then the script itself may have an error.


## Links
- https://docs.nitrokey.com/pro/linux/automatic-screen-lock
- https://stackoverflow.com/questions/6496847/access-another-users-d-bus-session
- https://askubuntu.com/questions/675310/unlock-gnome-screensaver-instead-of-deactivating
