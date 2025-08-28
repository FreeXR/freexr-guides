## How to switch to a PrivateQuest setup

(this guide is a work in progress!)

## 1. Back up your tokens
- Go to a site like <https://secure.oculus.com/> and log in
- Open your browser's development tools and find the cookies tab
- Then, grab these:
  - DeviceKey
    - Get your `oc_www_at` cookie
    - Run this in a terminal: `curl https://graph.oculus.com/graphql -H "Content-Type: application/x-www-form-urlencoded" --data-urlencode "access_token=xxx" --data-urlencode "doc_id=7400628006644133"`, where (xxx) is your access token 
    - It'll spit some json back at you containing your DeviceKey. Keep it somewhere safe.
  - Meta token & Oculus Token: 
    - Either use a rooted phone, or install an android emulator (such as waydroid) on a PC to get this done
    - If you want to use waydroid:
      - Install waydroid using the guides available for your platform (windows/mac/linux/whatever)
      - Initialize waydroid with `waydroid init -s GAPPS` (you need google play services for the meta horizon app to work)
      - Register your device id at [google's official page](https://www.google.com/android/uncertified)
      - After registering, wait a few minutes, then restart waydroid 
      - Install these using [this script](https://github.com/casualsnek/waydroid_script):
        - ARM translation layer, so we can run the meta horizon app. Use either libhoudini or libndk (libndk crashed the meta horizon app in my case, libhoudini worked)
        - magisk (we need root)
    - Then using either your rooted phone or waydroid:
      - Use either the play store, apkmirror, aurora store or whatever other method you prefer to install the Meta Horizon app
      - Log into it
      - Use root to copy  `/data/data/com.oculus.twilight/databases/prefs_db` to somewhere you can easily read it
      - `prefs_db` is an sqlite file. You can use something like [sqlitebrowser](https://sqlitebrowser.org/) to read it's contents.
			 - You're looking for `MetaProfileGenericAuthMap` in the `preferences` table within `prefs_db`. If using sqlitebrowser, go to the Browse Data Tab, select the `preferences` table, and find `MetaProfileGenericAuthMap`. It should be a bunch of JSON code. Look for `id` and `token`, and get their values. Those are what you need!
          - You can also easily get it using this SQL code: `SELECT value FROM preferences WHERE key=='MetaProfileGenericAuthMap'`

## 2. Download and install PrivateQuest
On your phone, download and install PrivateQuest from here: <https://xdaforums.com/t/app-5-0-private-quest-vr-headset-management-tool.4695491/>

## 3. DISABLE YOUR HOME INTERNET
Either use your phone to start a hotspot without internet, or use your router to turn off the internet while keeping WiFi on. Either way, **don't let your quest reach the internet during setup!**

To use a PC, laptop or other non-android device:

  - Go to your router or modem's admin panel and find a switch that toggles the internet connection
  - Use the toggle to switch your home internet off while keeping WiFi enabled
  - Verify your WiFi-connected devices can no longer access the internet

To use a phone:

  - Install Termux on your phone (you can get it on F-Droid)
  - Open Termux and type `pkg install android-tools`. It will ask you to confirm by typing Y and hitting enter - do so.
  - **Turn off mobile data** on your phone and then start a hotspot.

## 4. PrivateQuest initial setup

> [!NOTE]
> PrivateQuest cannot find the headset if it's already set up, because it will already have been assigned a DeviceKey.

> [!TIP]
> If all you want to do is control your existing setup using PrivateQuest instead of the Horizon app, and don't need to reinstall or factory reset, you can set the DeviceKey to the one you gathered while backing up tokens. This will let you control a quest that has already been set up, but it won't be the same as a debloated install.

- Factory reset your headset
- Put on your headset and turn it on. Let it sit on the initial screen, **don't let it connect to wi-fi!**
- Once it's in first time setup, connect to your headset using PrivateQuest.
    - Make sure the PrivateQuest app has access to bluetooth, location, and finding nearby devices
- Open PrivateQuest and connect to the headset (tap on where it says your headset model) - it will say ‘connecting’ and then ‘connected’ at the bottom of the screen
- In the Init tab, tap ‘Set DeviceKey’
- In the Config tab, tap 'Set Time'

# 5. Disable quest system software updates
**If you don't do this, as soon as your quest goes online, it will force upgrade itself to the latest (non-rootable) version of the system software!**

- Beside 'OTA update', tap 'Get' (Note: This will just check to see OS updates are enabled, it won't actually update the headset). 
- The switch (beside OTA update) will now show you that updates are indeed enabled (they were already enabled - the app just didn't know this yet. 'Get' doesn't change any settings).
- Now tap that switch to turn off updates. Tap 'Get' again to confirm that they are switched off.

## 6. Choose your setup
Now decide if you want to use your tokens to log your headset back in, or use an accountless setup..

You'll want a logged in setup if you want to play the games you bought from the quest store! If not, it's better to go accountless, because then your headset won't be tied to a meta account.

> [!NOTE]
> *v72 and lower can't use the logged-in setup!*. In that case, it'll throw an error about it needing to be a retail device during the Skip NUX step. Please use the accountless setup on v72 or lower.

## Logged-in setup
You'll need root to skip the first time setup screen (NUX). For rooting to work, you'll need to be on a vulnerable version of horizonOS. See [the exploit github page](https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1) for information on which versions are vulnerable.

- In PrivateQuest, press the three dots in the top right and go to settings 
- Where it says DeviceKey, change it to the one you saved from earlier 
- Tap Set Key, press yes when it asks again, then scroll to the bottom and hit Back
- Connect to your quest using adb over TCP
	- Run `adb shell pm disable-user --user 0 com.oculus.updater`
	- Use the [root exploit](<https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1>) to get root
	- In the root shell, `oculussetting --set first_time_nux_ota_state rebooting && reboot`
 	- After reboot, gain a root shell again. Then, in the root shell:
		- `oculussetting --set first_time_nux_ota_state notify_endpoint`
		- In the Config tab, Set the Oculus and Meta token to the ones you got from `MetaProfileGenericAuthMap`.
		- Set UserId underneath both the Oculus and Meta token to the `UserId` you got from `MetaProfileGenericAuthMap`
		<!-- - In privatequest over on your phone, use *Set Tokens* (*????? try not doing that, see if it works - rose*) -->
		- `oculussetting --set first_time_nux_ota_state COMPLETE`
		- `reboot`
- Verify the updater is disabled using `adb shell pm list packages -d`. It should show the updater in the list, which is the list of disabled packages.
- Proceed with [Post-setup](#post-setup) for extra security against your quest getting forcibly updated

<!--
Note: *In the horizon app, if it's still asking for a pairing code even though you don't have one, you can fix this, but it will require root.*

- Force enable ADB through PQ
- gain root and run `oculuspreferences --getc hmd_pairing_code` (fun fact: you can also set it to whatever you want with `oculuspreferences --setc hmd_pairing_code 12345`)
- input that code into your horizon app
-->

## Accountless setup
An accountless setup is easy and doesn't require root.

- In Private Quest:
	- Control -> Developer mode -> Get
	- Control -> Developer mode -> Switch to ON position
	- Control -> ADB -> Get
	- Control -> ADB -> Switch to ON position
	- Init -> Set Oculus token
	- Init -> Set Meta token
	- Init -> Skip NUX

<!--
old, may not be needed!

- In the Control tab of PrivateQuest, tap Start (to enable ADB over TCP)
- Put on the headset and navigate through the various setup panels until it asks you to connect to wifi. Connect to your hotspot or your wifi, depending on what you did before. **ensure the wifi can't reach the internet**
- In PrivateQuest, on the 'Control' tab, tap on 'Start'
- Put the headset back on and accept the connection (don't choose 'Always accept - it causes issues)
- In PrivateQuest tap, on the 'Control' tab, tap on 'Info'. This will show you two commands 'adb pair ...' and 'adb connect ....'
- Copy the pair command and paste it into your chosen terminal (such as termux on your phone, or a terminal on your pPC). It will say 'Successfully paired'. 
- Copy the connect command and paste it into your terminal too. Hopefully it will successfully connect. If it does, great.
  - If not, you need to go to the 'Wifi' tab in PrivateQuest and toggle the wifi switch a few time to disable and then re-enable wifi, ending with it disabled. Then tap 'Scan' (still on the 'Wifi' tab and reconnect to the hotspot - you will need to input your hotspot password again here in the app). You'll need to repeat the pairing process again.
- Run `adb shell pm disable-user --user 0 com.oculus.updater`
- Use the [root exploit](https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1) to get root
- In the root shell, `oculussetting --set first_time_nux_ota_state rebooting && reboot`
- Now using adb over usb:
    - `adb shell`
    - `su`
    - `oculussetting --set first_time_nux_ota_state notify_endpoint`
    - In privatequest over on your phone, use *Set Tokens* and just set them to be empty.
    - `oculussetting --set first_time_nux_ota_state COMPLETE`
    - `reboot`
-->

## Post-setup
- If you haven't already, install [Event Horizon](https://github.com/veygax/eventhorizon). This will root your quest with a simple button press! You can also set it to always root your quest on boot. You could also use the raw [root exploit](https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1) to get root.
	- Event Horizon installs Magisk onto your quest. **NEVER** press the Install or Uninstall buttons inside Magisk. That will hard-brick your quest!
- Enable root
- Use adb to connect to your quest, either via USB or wifi
- Run `adb shell`
- Now in adb shell:
	- run `pm disable-user --user 0 com.oculus.updater`
	- run `su`. If the exploit worked, you'll be in a root shell.
	- In the root shell, run `chmod 000 /data/data/com.oculus.updater`. This will completely block the updater app from being written to.
