> [!WARNING]
> This guide is a work in progress! We're still figuring out the best ways to set up a Quest in a way that bypasses meta. Use this guide at your own risk! If you are not sure about what you are doing then join our community and ask questions so that we can improve this guide.

## How to switch to a PrivateQuest setup

private-quest is a 3rd party android application that's used as an alternative to the META Horizon app which implements most of the BLE calls needed to remotely control the headset.

Using private-quest is mandatory for root users as it enables you to skip the forced update after factory reset to fully recover from soft-bricking your headset as allowing the headset to update patches the vulnerability used for root elevation.

# 0. Pre-Requirements

## 0.1. Make a backup of your deviceKey

**WARNING:** This is **NOT** recommended as Meta Horizon enables Meta to interact with the Companion Server which has Android Admin API priviledges that then acts as a backdoor to the device. Instead consider generating a random deviceKey in private-quest.

If you want to be able to use Meta Horizon and Private Quest side by side you need to make a backup of your deviceKey.

- **Method 1 (The "GraphQL"-way):**
  - Go to a site like <https://secure.oculus.com/> and log in
  - Open your browser's development tools and find the cookies tab
  - Find your `oc_www_at` cookie and get its value.
  - Run this in a terminal. Replace (TOKEN_HERE) with the value of the `oc_www_at` cookie:
  ```console
  $ curl https://graph.oculus.com/graphql -H "Content-Type: application/x-www-form-urlencoded" --data-urlencode "access_token=TOKEN_HERE" --data-urlencode "doc_id=7400628006644133" # Dump the document that contains the deviceKey
  ```
  - It'll spit some json back at you containing your DeviceKey. Keep it somewhere safe.

- **Method 2 (The "Rooted Android App"-way):**
  - TODO: The key should be extractable from a directory somewhere in `/data/data/com.oculus.twilight`
  - TODO: Technically possible via emulator as well


- **Method 3 (The "JailBroken iOS App"-way):**
  - TODO: Technically possible, but none tried it yet, generally not recommended.

## 0.2. Collect your META/Oculus Access Tokens

- **Method 1 (The "Command-line" Way):**
- Download the Meta Horizon App from apkmirror, aurora store or if desperate from the play store and log-in
  - Make a copy of `/data/data/com.oculis.twilight/databases/prefs_db` to your computer
  - Invoke `SELECT value FROM preferences WHERE key=='MetaProfileGenericAuthMap` via the `sqlite` command

- **Method 1 (The "Rooted Android"-way):**
  - Download the Meta Horizon App from apkmirror, aurora store or if desperate from the play store and log-in
  - Make a copy of `/data/data/com.oculis.twilight/databases/prefs_db` to your computer
  - Open the `prefs_db` in [sqlitebrowser](https://sqlitebrowser.org)
    - In `Preferences` Table
      - `MetaProfileGenericAuthMap` is your META and Oculus Access Token.
      - `METAUserID` is your User ID for both tokens

## 1. Download and install PrivateQuest
- On your android phone, download and install PrivateQuest from here: <https://xdaforums.com/t/app-5-0-private-quest-vr-headset-management-tool.4695491/>
- Make sure that the app has access to Bluetooth, Location and Finding Nearby Devices.
- Set your `deviceKey` inside app's settings for the headset you want to set up.

## 2. PrivateQuest initial setup
**WARNING:** Do not let your headset connect to the internet until you disabled the `com.oculus.updater` application!

  - Private Quest
    - Perform Factory Reset on your headset and turn it on so that it sit on the initial setup screen. **DO NOT CONTINUE THE SETUP!!**
    - Connect to the headset
    - Headset -> `init` -> `Set DeviceKey` -- This Claims the Headset so that private quest is the only authority that can control it unless you re-used the deviceKey for META Horizon

### 2.1. Disable System Software Updates
> [!CAUTION]
> **If you don't do this, as soon as your quest goes online, it will force upgrade itself to the latest (non-rootable) version of the system software!**

- Private Quest
  - Headset -> Config -> OTA Update -> Get -- This will move the toggle to ON position
  - Headset -> Config -> OTA Update -> Toggle **OFF**
  - Headset -> Config -> OTA Update -> Get -- To confirm it's set OFF
  - Headset -> `Control` -> Developer -> `Dev Mode` -> Get
  - Headset -> `Control` -> Developer -> `Dev Mode` -> Toggle ON
  - Headset -> `Control` -> Developer -> `Dev Mode` -> Get -- To confirm it's ON
  - Headset -> `Control` -> Developer -> `ADB` -> Get
  - Headset -> `Control` -> Developer -> `ADB` -> Toggle ON
  - Headset -> `Control` -> Developer -> `ADB` -> Get -- To confirm it's ON
- Android Debug Bridge ("adb")
  ```console
  $ adb shell pm disable-user --user 0 com.oculus.updater # Disable updates on META Quest Devices
  ```

### 2.2. Skip First Time Setup
> [!CAUTION]
> **If you don't do this, as soon as your quest gets to the update page it will force a software update, patching the vulnerability used for elevating root**

#### 2.2.1. Accountless Setup

It's possible to now set the headset without META and Oculus Access Tokens which will result in fully de-metafied headset and minimum UI by:
  - Private Quest
    - Headset -> Init -> `Set Combined Token`
    - Headset -> Init -> `Skip NUX`

#### 2.2.2. Logged-in Setup

  - Headset -> Init
    - Set Oculus and Meta Access tokens to the value of `MetaProfileGenericAuthMap` above
    - Set Oculus and Meta User ID to the value of `METAUserID`
    - `Set combined Token`
    - `Skip NUX`

### 3. Root the headset

If you are on vulnerable firmware version install [Event Horizon](https://github.com/veygax/eventhorizon) on the headset and configure it to your liking.

It's recommended to set the permission of `000` on the data directory of `com.oculus.updater` for extra safety, this requires root.

```console
# chmod 000 /data/data/com.oculus.updater # Neutralizes the updater's data directory so that nothing but root has permissions to interact with it
```

### 4. Post-Setup

META Quest 3 devices are abusive surveillence devices made by vicious people who don't care about your well being and will do anything to use any collected information about you to use against you as explained by Family IT Guy: https://www.youtube.com/watch?v=ooy7aLDrY0E

To mitigate this we created https://github.com/FreeXR/FreeXR-Hijack tool which acts as a "pseudo-ROM" that replaces all META applications with open-source alternatives and sets up the device to be more functional and useble beyond just niche gaming console, it is still in development and might be in the future merged into Event Horizon, but if you want to use it then you can:
  - Download the `init` branch of the repository on your computer
  - Make sure that the headset is authentificated over ADB
  - FreeXR Hijack
    ```console
    $ ./src/main.sh --HIJACK # Perform the Hijack
    ```

This will debloat the headset and tries to get rid of all, but functional tracking (if you choose to have META Store) from the device. Please let us know if the hijack fails for whatever reason so that we can fix it.

## Communities

* Discord: https://discord.gg/ABCXxDyqrH
* Matrix: Pending..
* IRC: Pending..

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