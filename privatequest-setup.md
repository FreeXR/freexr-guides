> [!WARNING]
> This guide is a work in progress! We're still figuring out the best ways to set up a Quest in a way that bypasses meta. Use this guide at your own risk! If you need help, join the [freeXR discord server](https://discord.gg/ABCXxDyqrH). Your questions can help us improve this guide.

## What is PrivateQuest and why do I need it?
PrivateQuest is a third party android application that's an alternative to the Meta Horizon android app, which implements most of the BLE (BluetoothLowEnergy) calls needed to remotely control the headset.

Using PrivateQuest is highly recommended, if not mandatory, for root users, as it enables you to skip the forced update to the latest quest system software after a factory reset. This is essential for recovering from softbricks.

> [!NOTE]
> The Meta Horizon android app enables Meta to interact with the Companion Server service on your quest, which has Android Admin API priviledges. This means that even with your quest fully blocked off from the internet, Meta can essentially use this as a backdoor to access your quest.
> Therefore, once you've set your quest up using PrivateQuest, we strongly recommend not attempting to connect the Meta Horizon app to your Quest again.

## 1. Block your Quest from accessing the internet
> [!CAUTION]
> This step is essential if you want to continue using root!

Before we do anything else, you need to prevent your quest from accessing any of meta's servers. It's best to just completely block it off from the internet altogether during setup.

But, in order to connect to our headset after privatequest has control of it, we do need a working wifi network! Just with no internet access.

There are many ways you can do this. Either:
- Shut off your home internet completely while you do setup
- Block your quest's MAC address within your home router (using something like a parental control setting). Please note that a quest by default will use a randomized MAC address, and this is a per-network setting. When connecting your quest to a wifi network, make sure to set that to `use device MAC address` before connecting!
- Use a pi-hole within your home network to block all of meta's servers
- Use a mobile hotspot on your phone, and make sure to turn off mobile data so that your hotspot can't access the internet

Whichever one you use, make sure your quest absolutely cannot access the internet! This is **crucial**. Check to make sure it absolutely cannot.

Once you're ready, proceed with step 2!

## 2. Back up essential data
### DeviceKey
Your DeviceKey is the key that the Meta Horizon app has assigned to your headset in order to remotely control it. It's recommended to make a backup of this, just in case you want to return to meta's official setup.

- Go to a site like <https://secure.oculus.com/> and log in
- Open your browser's development tools and find the cookies tab
- Find your `oc_www_at` cookie and get its value.
- Run this in a terminal. Replace (TOKEN_HERE) with the value of the `oc_www_at` cookie:
```console
curl https://graph.oculus.com/graphql -H "Content-Type: application/x-www-form-urlencoded" --data-urlencode "access_token=TOKEN_HERE" --data-urlencode "doc_id=7400628006644133" # Dump the document that contains the deviceKey
```
- It'll spit some json back at you containing your DeviceKey. Keep it somewhere safe.

### Meta/Oculus Access Tokens
Your Meta/Oculus access token is the token that is used to log in to your meta and oculus accounts. This is needed in case you ever want to log into your meta account whilst using privatequest! (for example to download the games you own on the meta store)

> [!CAUTION]
> When using the meta horizon app, **never connect it to your headset**. We're only trying to fetch your account token, connecting to the headset is not needed.
> If you connect to the headset, the companion server might force upgrade you, causing you to lose the root exploit.

Either use a rooted phone, or install an android emulator (such as waydroid) on a PC to get this done..

If you want to use waydroid:
- Install waydroid using the guides available for your platform (windows/mac/linux/whatever)
- Initialize waydroid with `waydroid init -s GAPPS` (you need google play services for the meta horizon app to work)
- Register your device id at [google's official page](https://www.google.com/android/uncertified)
- After registering, wait a few minutes, then restart waydroid 
- Install these using [this script](https://github.com/casualsnek/waydroid_script):
    - ARM translation layer, so we can run the meta horizon app. Use either libhoudini or libndk (libndk crashed the meta horizon app in my case, libhoudini worked)
    - magisk (we need root)
 
Then using either your rooted phone or waydroid:

- Use either the play store, apkmirror, aurora store or whatever other method you prefer to install the Meta Horizon app
- Log into it
- **Do not connect to your headset, as this will let meta backdoor into your quest.**
- Use root to copy  `/data/data/com.oculus.twilight/databases/prefs_db` to somewhere you can easily read it

`prefs_db` is an sqlite file. You can use something like [sqlitebrowser](https://sqlitebrowser.org/) to read it's contents.

If using sqlitebrowser, go to the Browse Data Tab, select the `preferences` table. Then:
- `MetaProfileGenericAuthMap` is your META and Oculus Access Token. It should be a bunch of JSON code. The values of `userID` and `token` are what you need.
- `MetaUserID` is your User ID for both tokens

Make a backup of those, save them somewhere safe. We recommend also just keeping a copy of the `prefs_db` file around. Done!

## 3. PrivateQuest setup
Please first make sure your quest can't access the internet. Got that sorted? Good! Time to get started..

- On your android phone, download and install PrivateQuest from here: <https://xdaforums.com/t/app-5-0-private-quest-vr-headset-management-tool.4695491/>
- Make sure that the app has access to Bluetooth, Location and Finding Nearby Devices.
- Perform a Factory Reset on your headset, let it finish, then once it reboots, let it sit on the initial setup screen. **Do not continue the setup, as that will start attempting to update your headset to a non-vulnerable version.**
- Open up PrivateQuest. Then, in PrivateQuest:
	- Let PrivateQuest scan for your headset. If you've properly factory reset your headset, it *should* show up in the list here. If it does not, join our discord and ask for help.
   	- Connect to the headset.
   	- In the settings, go to DeviceKey, copy whatever's in there and paste it somewhere safe just in case.
   	- Then, press `Generate`, confirm your action, then press `Set`, and confirm again.
   	- Back out of the settings page
	- Now in the init tab, press `Set DeviceKey`
 	- Doing all this has made it so that PrivateQuest is the only authority with control over your headset! Meta's phone app can no longer control it.
   	- In the Config tab, press `Set time`
	- In the Config tab, next to OTA Update, press Get. This will move the toggle to ON position.
  	- Now slide the OTA Update toggle to **OFF**
  	- Now press Get again to confirm that ota update is off!

Now if you want a meta-less setup, in the Init tab, press `Set combined token`.

If you want a logged in setup, use this instead:

- In the Init tab:
	- Set Oculus and Meta Access tokens to the value of `MetaProfileGenericAuthMap` above
	- Set Oculus and Meta User ID to the value of `METAUserID`

Finally:

- Init -> `Skip NUX`

Let it load the system UI. Once it's loaded, proceed with the next step!

## 4. Post-setup
Okay, now, if you used a meta-less setup, more than likely you'll be unable to see or open any apps! Not good.

So we're going to need to use ADB to do some sorcery and bypass things.

Remember that we blocked off the internet but not the wifi network? That comes into play now.

Connect both your headset and your phone to your home wifi network **and once again make sure any devices on your wifi network cannot access the internet**.

In PrivateQuest:

- Next to ADB, press `Get`. It should slide to ON.
- Next to ADB over TCP, press `Start`. Put on your headset and choose `Always allow over this network`. Take your headset off again or use passthrough.
- Next to ADB over TCP, press Info. This will give you the commands needed to connect to your quest!
- Using either a laptop, pc, phone, or whatever other device you can use, connect to your quest using the commands it has given you. You can also use [Termux](https://play.google.com/store/apps/details?id=com.termux) on your phone if you don't have access to a computer!

Once you are connected, run:
```console
adb shell pm disable-user --user 0 com.oculus.updater
```
This will disable the quest system software updater service. Note however that this is not foolproof, and there have been documented cases of a quest still being upgraded despite applying this command. So you'll still need to block meta!

Now:

- Download [Lightning Launcher](https://github.com/threethan/LightningLauncher/releases)
- Install lightning launcher like so:
```console
adb install LightningLauncher.apk
```
- Run this command. This will start lightning launcher:
	```console
	adb shell am start com.threethan.launcher
 	```
- Once lightning launcher is open, look for the camera icon on your dock. Drag it down off the dock, and wait until the text "unpin" appears, then let go.
- Now grab the lightning launcher icon in the dock and put it where the camera icon used to be
- Congrats! You now have a way to launch your apps and games.
- In addition, you can also a shortcut service so that lightning launcher will open when you hover over your quest dock library icon
	- To do so, inside lightning launcher, go to Settings, then Shortcuts, and pick whether you're using Dock or Navigator (in a fresh setup you're more than likely using Dock)
	- Then choose to install the app library shortcut, then follow the instructions to activate it

## 5. Root the headset
### Install root
Download [Event Horizon](https://github.com/veygax/eventhorizon). Once again, using adb, install it like so:
```
adb install eventhorizon.apk
```

Once it's installed, find it in lightning launcher, open it up, and enable `Root on Boot`, then press the `Root Now` button.

Let it perform the rooting process, then wait for it to restart the system UI. Congrats, you're rooted!

### Block meta from ever upgrading you again
> [!CAUTION]
> This step is essential if you want to continue using root!

Open event horizon again. Now head into `AiO tweaks`.
Inside Event Horizon's AiO tweaks, find the Meta Domain Blocker setting. Enable `Enable on Boot`, and enable the blocker. From now on, this will block your Quest from accessing meta's servers.

It's probably self explanatory, but you should never turn off this setting. If that setting is off and you let your quest connect to the internet, it will most likely start to upgrade itself! Meta is very aggressive with its forced upgrades, and once you've been upgraded, you cannot downgrade due to e-fuses in the hardware. So don't let that happen!

Still inside `AiO Tweaks`, turn on wireless ADB (and wireless ADB on boot if you prefer). No more need to use privatequest for that!

Now connect to your quest using wireless adb. Then:

```console
adb shell
su
chmod 000 /data/data/com.oculus.updater
exit
exit
```
This will neutralize the updater's data directory so that nothing but root has permissions to interact with it. This is yet more security against being upgraded.

## 6. Cautiously re-enable internet access
If you have verified that eventhorizon's domain blocker is working flawlessly, it should now be safe to unblock your Quest from accessing the internet.

If you want to be extra safe, you can use something like a pi-hole to block your quest from accessing meta's domains whilst still allowing access to the wider internet.

This guide won't go into detail on how to set up pihole, but you need to block these subdomains and any and all of their subdomains (use wildcards to be extra-safe):
```
meta.com
oculus.com
facebook.com
fb.com
facebook-hardware.com
messenger.com
msngr.com
m.me
fbcdn.net
oculuscdn.com
gvt2.com
fbsbx.com
oculusrift.com
facebookvirtualassistant.com
```

### 7. Optional: FreeXR Hijack
Meta Quest 3 devices are surveillance devices, made by people who don't have your best interests at heart, which gather data about you that they can use against you. Family IT Guy explains it here: https://www.youtube.com/watch?v=ooy7aLDrY0E

To mitigate this, we created [FreeXR Hijack](https://github.com/FreeXR/FreeXR-Hijack). It acts as a sort-of pseudo custom ROM that replaces all Meta applications with open-source alternatives and sets up the device to be more functional and usable beyond just being a gaming console. We would like to create a full custom ROM, however the bootloader needs to be unlocked for that first, so we have settled on this for now. It is still in development and might in the future be merged into Event Horizon, but if you want to use it right now, then you can:
  - Download the `init` branch of the [repository](https://github.com/FreeXR/FreeXR-Hijack) on your computer
  - Make sure that your computer is connected to the headset over ADB
  - FreeXR Hijack
    ```console
    ./src/main.sh --HIJACK
    ```

This will debloat the headset and tries to get rid of all tracking, telemetry, and bloat, except for the tracking required for essential functionality (if you choose to have the Meta Store). Please let us know if the hijack fails for whatever reason, so that we can fix it!

## Communities

* Discord: https://discord.gg/ABCXxDyqrH
* Matrix: Pending..
* IRC: Pending..

<!--
# Experimental setup: Log your quest in to your meta account whilst using private-quest
> [!DANGER]
> The Meta Horizon android app enables Meta to interact with the Companion Server on your quest, which has Android Admin API priviledges. This will then act as a backdoor to the device.
> **This means they can, and probably __WILL__, force upgrade your device and make you lose root.
> This is therefore strongly recommended against.

- **Method 2 (The "Rooted Android App"-way):**
  - TODO: The key should be extractable from a directory somewhere in `/data/data/com.oculus.twilight`
  - TODO: Technically possible via emulator as well


- **Method 3 (The "JailBroken iOS App"-way):**
  - TODO: Technically possible, but none tried it yet, generally not recommended.

#### 2.2.2. Logged-in Setup

  - Headset -> Init
    - Set Oculus and Meta Access tokens to the value of `MetaProfileGenericAuthMap` above
    - Set Oculus and Meta User ID to the value of `METAUserID`
    - `Set combined Token`
    - `Skip NUX`
-->

<!--
## Logged-in setup
> [!NOTE]
> You'll need root to skip the first time setup screen (NUX). For rooting to work, you'll need to be on a vulnerable version of horizonOS. See [the exploit github page](https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1) for information on which versions are vulnerable.

- Connect to your quest using adb over TCP
- Run `adb shell pm disable-user --user 0 com.oculus.updater`
- Use the [root exploit](<https://github.com/FreeXR/eureka_panther-adreno-gpu-exploit-1>) to get root
- In the root shell, `oculussetting --set first_time_nux_ota_state rebooting && reboot`
- After reboot, gain a root shell again. Then, in the root shell:
    - `oculussetting --set first_time_nux_ota_state notify_endpoint`
    - In the Config tab, Set the Oculus and Meta token to the ones you got from `MetaProfileGenericAuthMap`.
	- Set UserId underneath both the Oculus and Meta token to the `UserId` you got from `MetaProfileGenericAuthMap`
		In privatequest over on your phone, use *Set Tokens* (*????? try not doing that, see if it works - rose*)
	- `oculussetting --set first_time_nux_ota_state COMPLETE`
	- `reboot`
- Verify the updater is disabled using `adb shell pm list packages -d`. It should show the updater in the list, which is the list of disabled packages.
- Proceed with [Post-setup](#post-setup) for extra security against your quest getting forcibly updated


Note: *In the horizon app, if it's still asking for a pairing code even though you don't have one, you can fix this, but it will require root.*

- Force enable ADB through PQ
- gain root and run `oculuspreferences --getc hmd_pairing_code` (fun fact: you can also set it to whatever you want with `oculuspreferences --setc hmd_pairing_code 12345`)
- input that code into your horizon app
-->

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
