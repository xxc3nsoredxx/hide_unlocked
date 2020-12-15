# Hiding the unlocked bootloader warning on Android?
I can't find instructions for removing the unlocked bootloader warning on Oxygen OS (10.0.15.HD65AA).
Some people say it's impossible.

## Requirements
* ADB and fastboot (and related drivers).
  * Gentoo: `dev-util/android-tools`
  * You may have to add a `udev` rule (and possibly add your user to the `plugdev` group) to be able to see the device.
* Rooted Android
* USB debugging enabled in developer options
  * Click "Allways allow from this device" to enable access in the event that you are unable to get an authorization promp (such as being stuck in the boot animation).

## Testing ADB and fastboot
If `adb devices` doesn't show the device, add the following rule (for example into `/etc/udev/rules.d/99-android.rules`):

```udev
# Comment, such as the manufacturer's name
SUBSYSTEM=="usb", ATTR{idVendor}=="0x[id]", MODE="0664", GROUP="plugdev"
```

To get the vendor ID, run `lsusb` (or optionally check `dmesg` for a notification of a plugged in device).
You will get a line such as:
```
Bus 004 Device 004: ID 2a70:4ee7 OnePlus Technology (Shenzhen) Co., Ltd. ONEPLUS A3010 [OnePlus 3T] / A5010 [OnePlus 5T] / A6003 [OnePlus 6] (Charging + USB debugging modes)
```
The first half of the ID is the vendor ID.
I would enter `0x2a70` in the `udev` rule.

Check if your user is in the `plugdev` group by running `groups`.
If it doesn't show up, run (as root) `usermod -a -G plugdev user` (substituting in your user).
If the group doesn't exist, run (as root) `groupadd plugdev` and run the previous command again.
Logout and log back in to have the new group.
Un-plugging and re-plugging the device should make it show up in `adb devices`.

Run `adb reboot bootloader` to go into fastboot.
Once it's booted into fastboot, run `fastboot devices`.
If you get "no permission", run `lsusb` to get the vendor ID (which has likely changed) and add a second `udev` rule for that.
Then un-plug and re-plug the device.
