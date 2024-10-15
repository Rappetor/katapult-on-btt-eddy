# Katapult on BTT Eddy
Flash Katapult on BTT Eddy for more convenient Klipper firmware updates (no more pressing the boot button).

This is a quick guide on how to flash Katapult on the BTT Eddy. **What:** Katapult is a configurable bootloader for Klipper. **Why:** When using Katapult you can flash Klipper much more easy onto your BTT Eddy, no longer do you need to power off and on the BTT Eddy while holding the little boot button. **How:** With Katapult you can enter the flashing mode by a simple console/SSH command and flash Klipper without having to touch your device. Even remotely if you wish to do so..

Originally made to go together with the [Sovol SV08 Mainline guide](https://github.com/Rappetor/Sovol-SV08-Mainline) and the [Automatic MCU script updater](https://github.com/Rappetor/Sovol-SV08-Mainline/tree/main/Automatic%20MCU%20script%20update). But this will work just as well with other Klipper printers with a BTT Eddy.

All this is made with the BTT Eddy *USB version* in mind. Use SSH to your printer to run the commands.


## INSTALL KATAPULT
First, obviously, we have to install *Katapult* if you haven't done so already.
```bash
cd ~ && git clone https://github.com/Arksine/katapult
```
Install *pyserial* (we need this later to flash the firmware)
```bash
pip3 install pyserial
```

## PUT BTT EDDY IN BOOT/DFU MODE
Now we are going to need to put the BTT Eddy into boot/DFU mode (otherwise we cannot flash Katapult on it).
1. Power off your BTT Eddy (e.g. pulling the USB plug, turning off the printer etc.)
2. Press and *hold* the little boot button on top of the BTT Eddy and power up the BTT Eddy (e.g. plugging the USB cable back in or turning the printer back on etc.)
3. Once powered on wait a few seconds and you can release the button again, it should now be in boot/DFU mode.

Check if BTT Eddy is in boot mode with:
```bash
lsusb
```
It should look something like this:
![Katapult makemenu config settings](/images/lsusb.png)

Take note of that `2e8a:0003` address, we will need it later. So copy the address if it's different and use it in a later step (*you know it when you see it*).

## CREATE KATAPULT FIRMWARE & FLASH
Now we are going to configure, build and flash the Katapult firmware for the BTT Eddy.

Configure the Katapult firmware as shown in the screenshot. Press [Q] to quit and save changes.
```bash
cd ~/katapult && make menuconfig KCONFIG_CONFIG=eddy
```
![Katapult makemenu config settings](/images/Eddy_Katapult_Menuconfig.jpg)

We shall now build and flash the Katapult firmware to our BTT Eddy. Remember that address? We need it here.
```bash
make clean KCONFIG_CONFIG=eddy
make flash FLASH_DEVICE=2e8a:0003 KCONFIG_CONFIG=eddy
```
Check if Katapult has been flashed succesfully (it should show a `usb-katapult_rp2040` device with serial) and please copy the serial (e.g. `usb-katapult_rp2040_45474E621A862D4A-if00`) and save this for the next step.
```bash
ls /dev/serial/by-id/*
```
![BTT Eddy Serial](/images/BTT_Eddy_Katapult_Serial.jpg)<br>

> [!NOTE]
> Since we *just* flashed Katapult to our BTT Eddy it's already in Katapult bootloader mode. When the serial starts with `usb-katapult_rp2040_` it's in Katapult bootloader mode, ready to receive a Klipper firmware, and when the serial starts with `usb-Klipper_rp2040_` it's in normal working mode. This `usb-Klipper_rp2040_` (e.g. `usb-Klipper_rp2040_45474E621A862D4A-if00`) is the serial you will use in Klipper.

We have flashed Katapult onto our BTT Eddy and it's ready to receive a Klipper firmware! You can now use the [Automatic MCU script updater](https://github.com/Rappetor/Sovol-SV08-Mainline/tree/main/Automatic%20MCU%20script%20update) to flash the Klipper firmware to the BTT Eddy (choose menu option 3). Or do it manually in the steps below.

## BUILD & FLASH KLIPPER
Let's build us a Klipper firmware for the BTT Eddy and flash this with Katapult.

Configure and create the Klipper firmware (config) as shown in the screenshot. Press [Q] to quit and save changes.
```bash
cd ~/klipper
make clean KCONFIG_CONFIG=eddy.mcu
make menuconfig KCONFIG_CONFIG=eddy.mcu
make KCONFIG_CONFIG=eddy.mcu -j4
```
![Klipper makemenu config settings](/images/Eddy_Klipper_Menuconfig.jpg)
> [!IMPORTANT]
> Take note of the 16KiB bootloader offset! The BTT Eddy Klipper firmware now *needs* a bootloader offset otherwise it will overwrite Katapult and you will have to re-do the Katapult flashing part.

After the Klipper firmware has been created we can flash it with Katapult!
First put the BTT Eddy in Katapult bootloader mode if it isn't already (replace the serial with your serial):
```bash
cd ~/klipper/scripts/ && python3 -c 'import flash_usb as u; u.enter_bootloader("/dev/serial/by-id/usb-Klipper_rp2040_45474E621A862D4A-if00")'
```
And flash the just configured and made `klipper.bin` with Katapult (replace the serial with your serial):
```bash
cd ~/katapult/scripts && python3 flashtool.py -d /dev/serial/by-id/usb-katapult_rp2040_45474E621A862D4A-if00
```
If it was flashed succesfully you can check and see it's Klipper serial, it will look like:
```bash
ls /dev/serial/by-id/*
```
![BTT Eddy Serial](/images/BTT_Eddy_Klipper_Serial.jpg)<br>
> [!NOTE]
> Use this serial that starts with `usb-Klipper_rp2040_` in Klipper, this is the correct one you need.

🥳 Yes! You have flashed Klipper onto your BTT Eddy using Katapult! In the future you only need to repeat these steps: 
- Build the klipper firmware (with bootloader offset)
- Put the BTT Eddy into Katapult bootloader mode
- Flash the Klipper firmware with Katapult, **done**

> [!TIP]
> Save the commands, with the correct serials, somewhere in a seperate text file so you can use (and find) them more easy later!

## CONTRIBUTE & BUGS
Got something to share? Got tips? Something ain't right in the guide? Something else? Please share! Create an issue and explain in there or even better; make a pull request with the fix/addition etc!
