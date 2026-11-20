# Razr Fold: a clean main profile, Google only in a dedicated profile (English version)

> **Languages:** **English** (this document) · [Русский](README.ru.md) · [中文](README.zh.md)
>
> **License:** [CC BY-ND 4.0](LICENSE) — you may copy, share, fork and print this guide for free, but only in full and unchanged, keeping the author and the contact details (GitHub and Telegram links) intact. Editing the text is not allowed.

Nov 20, 2026 · @No

**Primary source of this guide:** [github.com/sikkatu/debloat-Motorola-Razr-Fold](https://github.com/sikkatu/debloat-Motorola-Razr-Fold)

**Primary source on social media (Telegram):** channel [t.me/motorolarazrfold](https://t.me/motorolarazrfold), chat [t.me/motorolafold](https://t.me/motorolafold)

## What you get, and at what cost

The main profile will be clean: no Google and no Moto junk. Google will live only in a second profile, “Google”: you start it manually and stop it with one command.

**How it works.** The command `pm uninstall -k --user 0 <package>` does not erase the app; it hides it from the main profile: the file stays on the system partition, the data is kept, and you can bring the app back with `cmd package install-existing`. Other profiles are not affected: in the “Google” profile the services stay in place, and the Shelter work profile, if you have one, is cleaned separately (Step 11). That is why the order is this: first we create the Google profile and check that it works, and only then remove Google from the main one.

**What survives updates.** The hidden state is stored in `/data/system/users/0/package-restrictions.xml`. Flashing a kernel (`fastboot flash boot`) does not touch this file, and neither does a regular A/B OTA. But after a major update some apps may come back or new ones may appear: Step 10 finds them and removes them again. A factory reset brings everything back, and you will have to go through the steps again.

**What will stop working in the main profile:**

- push notifications through Google: apps without their own connection will receive notifications late or not at all;
- Play Store and updates from it: install apps from F-Droid and from the developers' websites;
- location by Wi-Fi and cell towers: only GPS remains, and the first fix takes longer;
- Google Pay and Wallet, “Find My Device”, Android Auto, sign-in with Google;
- Moto features from Step 5: the Moto AI and Qira AI assistants, Moto Unplugged, Family Space, Moto's “Games”, “Voice Recorder” and “Translator”, Moto's interactive wallpapers, the Motorola Notifications messaging client, and the preinstalled Clip Studio Paint; if you also remove the apps from items 4–5, then also Moto gestures, Moto Secure, Smart Connect (together with its entry in the USB mode menu) and Quick Clip for the stylus;
- Moto AI Services and Moto AI Core (Step 5, item 6): according to their descriptions, Moto apps take AI models from them for “Notes”, “Stylization”, optimized charging and adaptive brightness; I did not check what exactly will stop working;
- internet access for the regular camera (Step 12): it will shoot as before, but camera features that need a network will stop working; I did not check which ones your camera has; the same applies to any package whose internet access you block in Step 12a;
- leftovers from Step 5a: the Smart Feed to the left of the home screen, Moto's “Create with AI” and Journal, Android AICore (the apps' AI features on Gemini Nano), ARCore (augmented reality in apps), voice training for “OK Google”, carrier stubs (`carriers.txt`, needed only with SIM cards of those carriers); I did not check which of these you will notice on your phone;
- banking apps that require Play Integrity: on a phone with an unlocked bootloader and root, many of them do not work anyway.

Calls, SMS, camera, Wi-Fi, Bluetooth and GPS keep working. The guide checks the “Emergency call” button on the lock screen separately (Steps 6 and 8). We do not touch Motorola updates.

## Before you start

Prepare a PC with adb and make backups: after the Google account is removed from the phone, everything that was stored only in it will be lost.

**What you need**

- A PC running Linux, macOS or Windows (on Windows, run the commands from this guide in Git Bash: in WSL, adb usually does not see the phone).
- A USB cable and adb from the Android Platform Tools package. fastboot comes in the same package, so you most likely already have it.
- Battery charge of at least 50%.
- The stock `boot.img` and the `fastboot flash boot` command at hand, to roll back the kernel in case the phone stops booting.
- Root via APatch is already in place. It is needed only in Steps 9, 11, 12 and 12a: to turn on the auto-stop of the “Google” profile once and make the optional buttons on the home screen (Step 9), for the Shelter work profile, and for blocking internet access for the camera and other packages. Everything else is done without root.

**Backups (do them before the first step)**

- Contacts: the Contacts app → Settings → Export → a `.vcf` file; copy it to the PC. Contacts stored in the Google account will disappear from the phone together with the account. Later, import the `.vcf` into the local “Phone” account.
- Two-factor codes: if you use Google Authenticator, move the accounts to another app (in Authenticator: menu → “Transfer accounts”). Without this you can lose access to services.
- Photos, videos and files that are stored only in the Google cloud: download them via Google Takeout.
- Calendar and tasks, if they live only in Google: export them from the web version.
- Backups of messengers that are stored in Google Drive (for example, WhatsApp): make a local copy.
- Passwords and passkeys from Google Password Manager: export the passwords. Passkeys cannot be exported, so check that important services have a backup way to sign in.

Do not delete the Google account in the main profile yet: that is done in Step 7.

## Step 1. Connect the phone to the PC

Turn on USB debugging, connect the phone with a cable and make `adb devices` show the device with the status `device`.

All commands in this guide are typed in a terminal on the PC. If a command has to be typed on the phone itself, I say so separately.

**If the answer is not what is written here.** Stop and do not go further. In such places the guide says “show the output in the chat from the header”: this is the [Telegram chat](https://t.me/motorolafold) from the header of the document. Write which step and which item you stopped at, and attach the whole output of the command.

**How to reboot the phone.** When the guide says “reboot the phone” after commands that remove or restore apps, do it like this (on the PC):

```
sleep 20
adb shell svc power reboot
```

Do not use `adb reboot`: Android writes the “package removed” mark to disk about 10 seconds later, and with a fast reboot the mark is lost and the removed apps come back (this is what happened on the Razr Fold; details in Step 4, item 4). If `adb shell svc power reboot` does not work, press the power button and choose “Restart”, but also not earlier than 20 seconds after the command. If the `debloat.sh` script prints “adb reboot” at the end, it is an old version: fix that line following Step 4, item 1.

**On the phone**

1. Open Settings → About phone and tap “Build number” seven times. A message will appear saying that you are now a developer.
2. Open Settings → System → Developer options and turn on “USB debugging”. In some firmware the section is called “Developer settings”.
3. Connect the phone to the PC with the cable. The phone screen will show the prompt “Allow USB debugging?”. Tick “Always allow from this computer” and tap “Allow”.

**On the PC**

Open a terminal and type the command:

```
adb devices
```

The correct answer looks like this (your serial number will be different):

```
List of devices attached
ZY22XXXXXXXX    device
```

If something is wrong:

- Instead of `device` it says `unauthorized`: unlock the phone and confirm the prompt on the screen again.
- The list is empty: change the cable or the USB port. In the USB notification on the phone choose “File transfer”.
- The list is empty on Windows with any cable: install the Google or Motorola USB driver.

If the terminal says that the `adb` command is not found, add the Platform Tools folder to PATH. If you unpacked it into your home folder, type `export PATH="$PATH:$HOME/platform-tools"`. The command works only in this terminal window. The script from Step 4 calls `adb` without a path, so without this it will not work.

When the status is `device`, check that the commands go to the right phone:

```
adb shell getprop ro.product.device
adb shell getprop ro.build.version.release
```

The first command prints the phone's codename (on the Razr Fold it is `blanc_g`, possibly without a suffix). The second prints `16`. If the answer is completely different, the wrong device is connected to the PC: disconnect the other devices and repeat.

## Step 1a. Remove the Shelter work profile

If you installed Shelter, the phone has its work profile (usually number 10). If you do not need it, remove it before the other steps: [according to Shelter users' reports](https://discuss.privacyguides.net/t/why-shelters-work-profile-comes-pre-loaded-with-some-google-services/14153), Google apps appear in such a profile right after it is created, and the commands from Steps 4–7 hide them only in the main profile. If you need the profile, skip this step: the guide will work, but Google and Moto junk will remain in that profile. How to remove them from there, and from a profile you create anew, is described in Step 11.

**Removal erases everything inside the profile:** the apps, their data and account sign-ins. This cannot be undone. First open Shelter, look at which apps are installed there, and save what you need using the apps' own tools.

**1. Remove the profile (on the phone)**

Settings → Passwords & accounts → the “Work” tab → “Remove work profile” → confirm. The names on Moto may differ: look for “work profile” in the accounts section. The order “first the profile, then the app” is taken from the [Shelter instructions](https://github.com/PeterCxy/Shelter?tab=readme-ov-file#uninstalling).

**2. If the profile cannot be removed in Settings (on the PC)**

First look at the list of profiles:

```
adb shell pm list users
```

Make sure that the line with number 10 is the Shelter work profile and not another profile you need, and remove it. All the profile's data is deleted ([source of the command](https://bayton.org/android/android-enterprise-faq/enrolment-failed-delete-wp/)):

```
adb shell pm remove-user 10
```

**3. Check that the profile is gone (on the PC)**

```
adb shell pm list users
```

The line with number 10 must not be there. If it is still there, wait a minute and repeat the command.

**4. Remove the Shelter app (on the phone)**

Press and hold the Shelter icon and choose “Uninstall”. If the system does not let you remove it, first turn it off in Settings → Security → Advanced → Device admin apps (the name of the item on Moto may differ).

After that, go to Step 2: it is better to take the “before” snapshot already without this profile.

## Step 2. The “before” snapshot and checking roles

Save the list of all packages on the PC and write down which apps are currently responsible for calls, SMS and the keyboard. From this snapshot you will later roll back the changes and find the junk that came back.

**Package snapshot (on the PC)**

Create a working folder. Put all the files from this guide into it.

```
mkdir -p ~/razr-debloat && cd ~/razr-debloat
```

Take three lists. The characters `|` and `>` are handled by your PC, so the files appear on the PC, not on the phone.

```
adb shell pm list packages -f -s -u --user 0 | sort > baseline_before.txt
adb shell pm list users > users_before.txt
adb shell pm list packages -3 --user 0 | sort > userapps_before.txt
```

`baseline_before.txt` will contain all system packages with the paths to their APKs, including those already hidden. This is the main file. Do not delete it, and make a copy on another disk.

**If instead of the list you got the error “Shell does not have permission to access user 10”**

This means that the command reached another profile of the phone (for the author it was profile 10, the Shelter work profile), and the adb shell is not allowed into it: debugging is prohibited there. This check exists in the Android source code ([a copy in the /e/OS mirror](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/8cd28b57ed732656d002d97879e15c5695b54fff)). Why `pm list packages` without `--user` reaches another profile, I do not know for sure. It is fixed by an explicit `--user 0`: then the command looks only at the main profile. In this guide it is already in all such commands. If the error remains, find out which profiles exist and which one is on the screen now (on the PC):

```
adb shell pm list users
adb shell am get-current-user
```

The second command prints the number of the profile that is on the screen now. If it is not `0`, return to the main profile, wait five seconds and repeat the snapshot:

```
adb shell am switch-user 0
```

Do not touch the Shelter work profile here: what to do with it is written in Step 1a.

**Who is currently responsible for calls, SMS and the keyboard (on the PC)**

```
adb shell cmd role get-role-holders --user 0 android.app.role.DIALER
adb shell cmd role get-role-holders --user 0 android.app.role.SMS
adb shell cmd role get-role-holders --user 0 android.app.role.HOME
adb shell cmd role get-role-holders --user 0 android.app.role.BROWSER
adb shell cmd role get-role-holders --user 0 android.app.role.ASSISTANT
adb shell settings get secure default_input_method
```

Each command prints a package name: the dialer, SMS, launcher, browser, assistant and keyboard, in that order. For the keyboard, after the package name comes a slash and a class name: it is not needed. Write down the answers, they will be needed in Step 6. If a line is empty, the role is not assigned or the command did not work. Then open Settings → Apps → Default apps on the phone and look there.

**What junk is on your phone (on the PC)**

```
adb shell pm list packages --user 0 | grep -E 'motorola|lenovo|google|facebook|netflix|amazon|microsoft|spotify' | sort | tee present_now.txt
```

The list will be long. Most of the `com.google.android.*` lines are Android system modules, and we do not touch them (see the section “What you must not remove”). The list is needed to compare it with the lists in Steps 4 and 5.

## Step 3. Create the “Google” profile and check it

Create a separate user “Google”, install the Google services into it, sign in to your account and make sure Play Store works while the main profile is still untouched.

Do this before removing the Google services from the main profile. This way the profile is created on a fully working system, and you immediately see that Google lives in it.

**About the profile number.** The number of the “Google” profile depends on which profiles are already on the phone: it may be 10, 11 and so on. So that you do not have to substitute the number by hand, the commands below use `$GU`. The first line of every block finds the profile named “Google” in the list by itself and remembers its number. Copy the blocks whole, together with this line.

**1. Check that the system allows several users (on the PC)**

```
adb shell pm get-max-users
adb shell pm list users
```

An answer like `Maximum supported users: 4` means everything is fine. You need the number 2 or more. If it says `1`, stop. Nothing has been removed and the phone is untouched, but without multi-user mode this plan will not work.

The second command prints the profiles that already exist: lines like `UserInfo{0:Owner:c13} running`. If one of them is already a profile named `Google`, stop: you do not need to create a second one. If it is empty and left over from a previous attempt, remove it with the command from the “Rollback” section (the last block) and start this step again.

**2. Create the profile (on the PC)**

```
adb shell pm create-user Google
```

The answer will be a line like `Success: created user id 11`. Your number may be anything, for example 10 or 11. If an error is printed instead of `Success`, stop: the profile is not created, the phone is untouched. Check that the number is found (on the PC):

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
echo Google profile number: $GU
```

The same number as in the `Success` line should be printed. If no number is printed, stop and show the output of the `adb shell pm list users` command in the chat from the header.

**3. Start the profile and install the Google services into it (on the PC)**

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am start-user -w $GU
adb shell cmd package install-existing --user $GU com.google.android.gms
adb shell cmd package install-existing --user $GU com.android.vending
adb shell cmd package install-existing --user $GU com.google.android.gsf
```

Each of the last three commands answers with a line like `Package ... installed for user: 11` (your number will be different). This is success, even if the package was already installed. Check the result:

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell pm list packages --user $GU | grep -E 'google.android.gms|android.vending|google.android.gsf'
```

There should be lines `com.google.android.gms`, `com.android.vending` and `com.google.android.gsf`. If there are others too, for example `com.google.android.gms.location.history` or `com.google.android.gms.supervision`, that is normal: the search catches any package with `google.android.gms` in its name, and the system puts these two packages into a new profile itself.

If one of the commands printed `Shell does not have permission to access user` and the number of the “Google” profile, then debugging is prohibited in it too, as in the Shelter work profile. Try to lift the prohibition only from our profile (on the PC):

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell pm set-user-restriction --user $GU no_debugging_features 0
```

Then repeat the block with `install-existing` and the check. This is based on the Android source code; I did not check it on your phone. If the command printed the same error, stop: then the prohibition can be lifted only through root, and the guide does not cover this case. Show the error in the chat from the header.

**4. Sign in to the profile and to the Google account (on the phone)**

Switch the screen to the new profile with the command:

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

The first-run wizard may open. Connect to Wi-Fi, sign in to your Google account, and skip the rest. Then open Play Store and install any small app. If it installed, Google works in the profile.

**5. Return to the main profile and turn off “Google” (on the PC)**

Paste the block whole, it will wait five seconds after the switch by itself:

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user 0
sleep 5
adb shell am stop-user -f $GU
adb shell pm list users
```

The last command prints roughly this (the numbers in braces will be different for you):

```
Users:
    UserInfo{0:Owner:c13} running
    UserInfo{11:Google:10}
```

One thing matters: the line with `Google` does not have the word `running`. This is what a turned-off profile looks like. Do not touch the other profiles in the list.

In the “Google” profile, install only what does not work without Google: banks, special-purpose apps. Keep everything else in the main profile. Steps 4–7 clean the main profile, and the “Google” profile remains untouched.

## What you must not remove

Remove only the packages named in this guide: any other package may turn out to be the one without which the phone will not boot or will not make calls.

If a package looks like junk but is not in the lists below, do not touch it. First find out what the package is, and only then decide.

| What | Example names | What will break |
| --- | --- | --- |
| Android core | everything that starts with `com.android.`, and the `android` package itself. Exceptions: `com.android.vending`, `com.android.chrome` and two `com.android.hotwordenrollment.*` packages (voice training for “OK Google”); they are named in the lists | Booting, calls, Bluetooth, NFC, settings, the interface |
| Google Mainline modules | any `com.google.android.*` that are not in your lists: `permissioncontroller`, `packageinstaller`, `documentsui`, `ext.services`, `networkstack`, `cellbroadcastreceiver`, `tethering`, `conscrypt` | Permissions, installing apps, files, network, emergency alerts |
| WebView | `com.google.android.webview`, `com.android.webview`, `com.google.android.trichromelibrary` | Pages inside apps will stop opening |
| Qualcomm | `com.qualcomm.*`, `com.qti.*`, `vendor.qti.*`, `org.codeaurora.*` | Modem, GPS, Wi-Fi, sound |
| Overlays | any name containing the word `overlay` | Themes, screen sizes, carrier settings |
| Updates and the Moto core | `com.motorola.ccc.*` (including `com.motorola.ccc.ota`; the only exception is `com.motorola.ccc.notification`, which can be removed), `com.motorola.launcher3`, `com.motorola.camera3`, `com.motorola.camera5`, `com.motorola.launcher.secondarydisplay` | Updates, the home screen, the camera, the outer display |
| Razr Fold screens and hinge | names containing the words `razr`, `fold`, `cover`, `hinge`, `flip`, `display`, `sensor` | Folding, the inner and outer displays |
| Moto settings | `com.motorola.android.providers.*`, `com.motorola.android.coresettingsext*`, `com.lc.motorola.coresettingsext` | The Settings menu and the settings storage |

The `debloat.sh` script from Step 4 itself skips some of the protected packages. This is a safety net, not a replacement for the rule. Any removal can be undone as described in the “Rollback” section. One exception to `com.motorola.ccc.*` is made deliberately: `com.motorola.ccc.notification` is not considered protected: it is in the Moto list from Step 5 and is removed like an ordinary package.

## Step 4. Remove the Google apps

Create the `debloat.sh` script and the `google.txt` list, run the removal and reboot the phone: this hides the Google apps that are not responsible for calls, SMS and the keyboard.

The dialer, SMS, contacts, keyboard, clock, calendar, calculator, browser and photos are not in this list. They are removed in Step 6, after the replacement.

**1. Create the script (on the PC)**

The script reads a list of packages and prints the result for each one. It skips packages that are not on the phone. It does not touch some of the protected packages. Copy the block whole and paste it into a terminal opened in the working folder `~/razr-debloat`:

```
cat > debloat.sh <<'EOF'
#!/usr/bin/env bash
# Usage:
#   ./debloat.sh remove list.txt   hide the packages from the list
#   ./debloat.sh restore list.txt  bring the packages from the list back
mode="$1"
list="$2"
if [ "$mode" != "remove" ] && [ "$mode" != "restore" ]; then
  echo "First argument: remove or restore"; exit 1
fi
if [ ! -f "$list" ]; then
  echo "List file not found: $list"; exit 1
fi

installed=$(adb shell pm list packages --user 0 | tr -d '\r' | sed 's/^package://')
if [ -z "$installed" ]; then
  echo "I do not see the phone. Check the adb devices command"; exit 1
fi

grep -Ev '^[[:space:]]*(#|$)' "$list" | tr -d '\r' | while read -r pkg _; do
  case "$pkg" in
    com.motorola.ccc.notification) ;;  # exception: this package can be removed (Step 5)
    android|com.android.systemui|com.android.settings|com.android.phone|com.android.server.telecom|com.android.providers.*|com.qualcomm.*|com.motorola.ccc.*|*overlay*|com.google.android.webview|com.google.android.trichromelibrary)
      echo "$pkg: protected package, skipping"; continue ;;
  esac
  if [ "$mode" = "remove" ]; then
    if printf '%s\n' "$installed" | grep -Fqx "$pkg"; then
      printf '%s: ' "$pkg"
      adb shell pm uninstall -k --user 0 "$pkg" </dev/null | tr -d '\r'
    else
      echo "$pkg: not on the phone or already removed, skipping"
    fi
  else
    printf '%s: ' "$pkg"
    adb shell cmd package install-existing --user 0 "$pkg" </dev/null | tr -d '\r'
  fi
done
echo "Done. Wait 20 seconds and reboot the phone: adb shell svc power reboot"
EOF
chmod +x debloat.sh
```

**If you already created debloat.sh from an old version of the guide**, at the end it prints “Done. Reboot the phone: adb reboot”. You must not reboot that way, the reason is in item 4 below. Fix only this line (on the PC, in the `~/razr-debloat` folder):

```
sed 's|^echo ".*adb reboot"$|echo "Done. Wait 20 seconds and reboot the phone: adb shell svc power reboot"|' debloat.sh > debloat.sh.new && mv debloat.sh.new debloat.sh && chmod +x debloat.sh
grep -c 'adb reboot' debloat.sh
tail -n 1 debloat.sh
```

The first check should print `0`, the second a line with `adb shell svc power reboot`. If you already have the new version of the script, the command will change nothing.

**2. Create the list (on the PC)**

The list is compiled from two public articles ([Motorola on droidwin](https://droidwin.com/?p=19403) and [inatagan/android-debloater](https://github.com/inatagan/android-debloater)) and from my own knowledge. Lines with an asterisk in the comment are ones I know from general lists; they are not in those articles. I did not check the list against your phone, and the script will skip extra lines. All lines without an asterisk, except nbu.files, ar.lens and apps.adm, are in droidwin's “safe to remove” list. It does not call the first two safe, and apps.adm (“Find My Device”) was added from the list of apps on the Razr Fold: it is not in the article. If in doubt, delete these lines from the list before running.

```
cat > google.txt <<'EOF'
# Google apps. Line: package  # name. Asterisk: I know the package from general lists.
com.google.android.youtube  # YouTube
com.google.android.apps.youtube.music  # YouTube Music
com.google.android.apps.maps  # Google Maps
com.google.android.gm  # Gmail
com.google.android.apps.docs  # Google Drive
com.google.android.videos  # Google TV (formerly Play Movies)
com.google.android.music  # Play Music, the old app
com.google.android.apps.tachyon  # Google Meet (formerly Duo)
com.google.android.apps.subscriptions.red  # Google One
com.google.android.apps.walletnfcrel  # Google Wallet and Pay
com.google.android.apps.googleassistant  # Google Assistant
com.google.android.apps.nbu.files  # Files by Google
com.google.android.googlequicksearchbox  # The Google app and the search bar on the home screen
com.google.android.printservice.recommendation  # Print recommendations
com.google.android.syncadapters.calendar  # Google Calendar sync
com.google.android.syncadapters.contacts  # Google Contacts sync
com.google.ar.lens  # Google Lens
com.google.android.apps.adm  # Find My Device (Find Hub)
com.google.android.apps.bard  # Gemini *
com.google.android.apps.magazines  # Google News *
com.google.android.apps.chromecast.app  # Google Home *
com.google.android.projection.gearhead  # Android Auto *
com.google.android.setupwizard  # Android Setup: the “Finish setting up your device” notification *
EOF
```

The line `com.google.android.setupwizard` (Android Setup) was added to get rid of the “Finish setting up your device” notification. The notification switch of this app is inactive in Settings, so we hide the app itself. According to general descriptions, it is needed at the phone's first start and after a factory reset; after a reset the data is wiped and all the hidden packages come back by themselves, so the initial setup will not be lost. I know the package from general lists, and I did not check it on your phone.

If `google.txt` is already created, add this line to it and run the removal again (on the PC):

```
echo 'com.google.android.setupwizard  # Android Setup: the “Finish setting up your device” notification *' >> google.txt
./debloat.sh remove google.txt
sleep 20
adb shell svc power reboot
```

The other lines will say “not on the phone or already removed, skipping”: this is normal, they are already removed. The `com.google.android.setupwizard` line should say `Success`; if it says `Failure`, show that line in full in the chat from the header. After the reboot the “Finish setting up your device” notification should be gone.

If after the reboot the notification remains, or the script wrote “not on the phone or already removed, skipping”, it is shown by another package. The list of packages that have notifications (on the PC):

```
adb shell dumpsys notification | grep -o 'pkg=[^ ]*' | sort -u
```

Find among them a package with the word `setup` in its name and show the output in the chat from the header.

**3. Run the removal (on the PC)**

```
./debloat.sh remove google.txt
```

Each line will get one of the answers. `Success` means that the package is removed. `Failure [...]` means that the system did not allow the removal: leave the package as it is. `Failure: package is non-disable` means that Motorola protected the package from being hidden; the analysis for Gboard is in Step 6. The phrase “not on the phone or already removed” means that the line was skipped.

**4. Reboot and check**

```
sleep 20
adb shell svc power reboot
```

Do not reboot the phone with `adb reboot` right after the script. Android writes the “package removed” mark to disk not at once but about 10 seconds later ([this is by design in the system itself](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/a83500e548584005e35a7836b7362333460c8c99)). If the phone is turned off earlier, the mark is lost and the removed apps come back. That is why here and further on there is first the pause `sleep 20`, then the normal reboot `svc power reboot`: on a normal shutdown Android has time to save everything. If the command does not work, restart the phone with the power button (the “Restart” item). That is exactly what happened on the Razr Fold: after `adb reboot` right after the script, all the packages from Step 7 came back, and after the pause and `svc power reboot` they stayed removed.

Unlock the phone and wait a minute. If the home screen, the notification shade and Settings open, the step is passed. If something broke, restore everything with the command `./debloat.sh restore google.txt`.

**Optional**

- Digital Wellbeing (`com.google.android.apps.wellbeing`): app timers and screen statistics will disappear.
- Google Text-to-speech (`com.google.android.tts`): reading text aloud in navigation apps and readers will disappear. First install another text-to-speech engine.

For each package the command is the same: `adb shell pm uninstall -k --user 0 package`.

**What will disappear after this step**

The Google search bar on the home screen, as well as YouTube, Maps, Gmail, Drive and the other apps from the list. In the “Google” profile they stay or are installed from Play Store.

## Step 5. Remove Moto junk and partner preinstalls

Create the `moto.txt` list with Motorola and partner apps, run the same script and reboot the phone (items 1–3). Then deal separately with the apps that have a price for removing (items 4–6). If you already went through items 1–3 earlier and the list has grown since then, create the file again (item 1) and repeat item 2: the script will skip the packages that are already removed. The remaining packages from the first list (Google leftovers, small Moto packages, partner stubs, carrier packages) are covered in Step 5a.

The list is taken from the same public articles and from the [bayton system apps database](https://sysapps.bayton.org). The database confirms that the packages exist and what they do, but does not declare them safe to remove. droidwin calls only genie, fmradio, fmplayer, moto and the four Facebook packages safe. The other lines are there only in the general list of installed packages or are missing altogether: if in doubt, delete the line from the list before running. Lines with an asterisk are from my own knowledge. The home screen, camera, updates and Razr Fold screens are not in the list, and they must not be touched.

Seven lines were added from the list of apps on the Razr Fold: `com.motorola.gamemode`, `com.motorola.audiorecorder`, `com.motorola.spaces`, `com.motorola.aitranslation`, `com.motorola.om`, `com.motorola.uxcore` and `com.lenovo.qira`. Nobody calls them safe to remove in the droidwin article: `gamemode` is there only in the general list of installed packages, and the others are not there at all. What each one does is written in the comments of the list, based on open descriptions. I could not check them on the phone itself: if some line raises doubt, delete it before running.

The line `jp.co.celsys.clipstudiopaint.googleplay` (Clip Studio Paint) was added at the author's decision: the app is paid, and it was removed right away. As far as I know, this package has nothing to do with the stylus, but item 5 checks the stylus after the removal.

The line `com.motorola.ccc.notification` (Motorola Notifications) was added at the author's decision: the package had been removed before, and it was removed. Earlier `debloat.sh` skipped it as protected, because all `com.motorola.ccc.*` packages are related to updates; now the script makes an exception for it, and the script still does not touch the other `ccc.*`. If you created `debloat.sh` earlier, create it again (Step 4, item 1): the old version will print “protected package, skipping”. One risk I could not check: in the general database this package has a permission to contact the update service (`ccc.ota`), so they may be related. That is why before a system update you should bring it back, as described in Step 10.

The line `com.motorola.livewallpaper3` (Moto Interactive Wallpaper, Moto's interactive wallpapers) was also added at the author's decision. If the home screen or the lock screen currently uses exactly this wallpaper, first go to Settings → Wallpaper & style and choose an ordinary picture (the name of the section on Moto may differ). As far as I know, when the wallpaper disappears the system sets the default wallpaper by itself, but I did not check this on your phone.

**1. Create the list (on the PC)**

```
cat > moto.txt <<'EOF'
# Moto and partners. Line: package  # name. Asterisk: I know the package from general lists.
com.motorola.genie  # Moto Help
com.motorola.help  # Moto: feedback
com.motorola.demo  # Demo mode for store displays
com.motorola.timeweatherwidget  # Moto's “Time and weather” widget
# com.motorola.gesture  # not included: judging by the APK name it is a navigation-gesture tutorial, the gain from removing it is small
com.motorola.android.fmradio  # FM radio
com.motorola.fmplayer  # FM radio
com.dti.motorola  # Moto App Manager: installer of recommended apps
com.aura.oobe.motorola  # MotoApps: recommended apps during setup
com.motorola.gamemode  # Moto Games: game mode
com.motorola.audiorecorder  # Moto Voice Recorder
com.motorola.spaces  # Family Space: children's spaces
com.motorola.aitranslation  # Translate from Moto (by the package name, AI translation)
com.motorola.om  # Moto Unplugged: a mode that limits apps and notifications
com.motorola.uxcore  # Moto AI
com.lenovo.qira  # Motorola Qira: Lenovo's AI assistant, partly works in the cloud
jp.co.celsys.clipstudiopaint.googleplay  # Clip Studio Paint: a paid drawing app
com.motorola.ccc.notification  # Motorola Notifications: Motorola's messaging client
com.motorola.livewallpaper3  # Moto Interactive Wallpaper: interactive wallpapers
com.motorola.mototour  # Introduction to Moto (tutorial) *
com.facebook.katana  # Facebook
com.facebook.appmanager  # Facebook App Manager
com.facebook.services  # Facebook Services
com.facebook.system  # Facebook System
com.netflix.mediaclient  # Netflix *
com.amazon.mShop.android.shopping  # Amazon *
com.spotify.music  # Spotify *
EOF
```

**2. Run the removal (on the PC)**

```
./debloat.sh remove moto.txt
```

The answers are the same as in Step 4. On the Razr Fold some packages may be missing: the script will skip them.

**3. Reboot and check**

```
sleep 20
adb shell svc power reboot
```

The pause in `sleep 20` and the normal reboot are needed for the same reason as in Step 4: otherwise Android may not save what was removed.

After booting, check the home screen, the camera, the notification shade, the “back” and “home” gestures, folding the phone and the outer display. If something broke, restore everything with the command `./debloat.sh restore moto.txt`.

**4. Apps that have a price for removing (on the PC)**

I did not put these three apps into `moto.txt`: each one has a noticeable function, and it is up to you. Read the descriptions. If you want to keep one, put `#` at the start of its line in the list below.

**Moto** (`com.motorola.moto`). This is not the initial setup wizard. According to the carrier's help, Moto gestures are opened through this app (Moto → Gestures), and Motorola's personalization settings are there too. The `com.motorola.actions` package (Moto Actions) is separate, and I do not touch it. A reader in a comment under the droidwin article writes that without this app, adaptive performance, optimized charging and overcharge protection disappear. I did not check this. If you use Moto gestures or these functions, keep it.

**Moto Secure** (`com.motorola.securityhub`). Motorola describes it as a common page of Android security and privacy settings plus its own functions: warnings about insecure Wi-Fi, Wi-Fi blocking, a scrambled keyboard for the PIN code and a protected folder. The standard Android settings do not go anywhere: they are in Settings → Security & privacy. In the general system apps database it has permissions marked for internet, background location, camera and the list of all installed apps. I found no data that it sends anything, but the code is closed, and it is impossible to prove that it does not send. If you do not use the protected folder, remove it. If you do, first move the files out of it. According to the author's check, the package has internet access, and the author removed it, so the line stayed in the list below.

**Dolby Atmos** (`com.motorola.dolby.dolbyui`). As far as I know, this is the Dolby sound settings window. According to the author's check (the command below), it has no direct internet access, so it was kept, and it is not in the removal lists. Do not touch the `com.dolby.daxservice` package, which does the sound processing itself, either. If you already removed `com.motorola.dolby.dolbyui`, bring it back with the command `adb shell cmd package install-existing --user 0 com.motorola.dolby.dolbyui`, then wait 20 seconds and reboot the phone with the command `adb shell svc power reboot`.

**Smart Connect** (`com.motorola.mobiledesktop`). This is the former Ready For: connecting the phone to a PC and a desktop mode on a monitor or TV. According to the description in Google Play, it also manages Moto Buds headphones and the Moto Tag tracker. Open alternatives exist: scrcpy shows the phone screen on the PC and lets you control it, KDE Connect links the phone to the PC (notifications, files, clipboard). In the general database it has permissions for internet, reading and sending SMS, contacts, audio recording and camera. According to the author's check, the package has internet access, and the author removed it, so the line stayed in the list below. After the removal, the item “Smart Connect” disappears from the USB mode menu (it appears when you connect the phone to a PC with a cable): that is how it turned out for you, the other items stayed. This means the item is created by the app itself. If you do not need the desktop mode on a monitor or TV, nothing has to be restored.

Internet access check (on the PC). Substitute the name of the package you need:

```
adb shell dumpsys package com.motorola.dolby.dolbyui | grep -iE 'permission.INTERNET|sharedUser'
```

If a line with `android.permission.INTERNET` is printed, the package may reach the network directly. If nothing is printed, there is no direct access. If only a `sharedUser=` line is printed, the package shares permissions with other system packages, and the check proves nothing. This check also will not show data transfer through another app that does have internet.

Create the list (on the PC):

```
cat > moto-extra.txt <<'EOF'
# Moto apps that have a price for removing. Whatever you want to keep, put # at the start of the line.
com.motorola.moto  # Moto: gestures and Motorola features
com.motorola.securityhub  # Moto Secure
com.motorola.mobiledesktop  # Smart Connect (formerly Ready For)
EOF
```

Run the removal and reboot the phone (on the PC):

```
./debloat.sh remove moto-extra.txt
sleep 20
adb shell svc power reboot
```

If a line says `Failure: package is non-disable` instead of `Success`, the package is protected by the system. Skip it and show the whole output in the chat from the header: such packages need the same trick as Gboard in Step 6.

After the reboot, check on the phone:

- Settings → Security & privacy opens, and the phone unlocks with a fingerprint or face.
- Moto gestures work, if you used them.
- Folding and the outer display work, as in Step 8.
- No “… has stopped” windows pop up anywhere.

If something broke, bring back all three apps (on the PC):

```
./debloat.sh restore moto-extra.txt
sleep 20
adb shell svc power reboot
```

One package is brought back with the command `adb shell cmd package install-existing --user 0 PACKAGE_NAME`, then again a pause and a reboot.

For Smart Connect (on the PC): `adb shell cmd package install-existing --user 0 com.motorola.mobiledesktop`. After the pause and the reboot, its item in the USB mode menu will appear again.

**5. Stylus (on the PC)**

The question is whether the stylus will work without `com.motorola.stylus` (in the app list it is labeled “Notes”). I do not have an exact answer: I did not find a source that says which package holds the logic of the stylus button and connection. What is known: the Moto Pen Ultra connects over Bluetooth, works on both Razr Fold screens and supports pressure, tilt and palm rejection. The pen button is configurable, and the Quick Clip function sends text to the Motorola notes app. So remove this package last and check right away. Clip Studio was removed right away: it is already in the moto.txt list (items 1–3), so we check pressure without it.

According to the author's check, `com.motorola.stylus` has internet access. Freezing (`pm disable-user`) would most likely change nothing for the stylus: the app stops working in the same way, and only the command for bringing it back changes (`pm enable`). So removal remains, like for the other packages: the check after an update for it is already written in Step 10.

Remove the package (on the PC):

```
adb shell pm uninstall -k --user 0 com.motorola.stylus
sleep 20
adb shell svc power reboot
```

After the reboot, check on the phone:

- Settings → Connected devices: the stylus is shown as connected.
- Check the pressure. If you have a drawing app, draw a line while changing the pressure: it should become thinner and thicker. If there is no such app, turn on Settings → System → Developer options → “Pointer location” (the name on Moto may differ) and move the stylus over the screen while changing the pressure: the pressure number in the top bar should change (usually labeled Prs). Then turn this item off.
- Press the side button of the stylus: it should do the same as before.
- Quick Clip and the “Notes” app will most likely disappear together with the package. If you need them, bring the package back.

If the stylus works worse than before, bring the package back (on the PC):

```
adb shell cmd package install-existing --user 0 com.motorola.stylus
sleep 20
adb shell svc power reboot
```

If you had to bring the package back, it still has internet access. You can block its network access with the method from Step 12: add `com.motorola.stylus` to `block-net.list` (first check the uid number, as in Step 12a).

If Clip Studio is the preinstalled one, `restore moto.txt` will bring it back together with the other packages of the list, and separately it can be brought back with the command `adb shell cmd package install-existing --user 0 jp.co.celsys.clipstudiopaint.googleplay` (then a pause and a reboot, as above). If it could not be brought back this way, you need its APK.

**6. Moto AI models (on the PC)**

These are the packages from which Moto apps take AI models: `com.motorola.aiservices` (“Moto AI Services”) and `com.motorola.aicore` (“Moto AI Core”). The Moto AI and Qira apps themselves were removed in items 1–3, and the models stay on the phone until you remove them. This is not Google AICore: that one has a different package, `com.google.android.aicore`; it is removed separately, in Step 5a.

What is known. According to the [description on APKMirror](https://apkmirror.com/apk/motorola-mobility-llc/moto-ai-services/moto-ai-services-1-1-149-13-release/moto-ai-services-1-1-149-13-android-apk-download), Moto AI Services supplies models for Moto's “Notes”, “Stylization”, optimized charging, adaptive brightness and other features. In the [system apps database](https://sysapps.bayton.org/packages/com.motorola.aiservices) it has marked location access and receiving push messages. The “Moto” app and Moto Actions, judging by their permissions, connect to these services. What exactly will stop working without them, I did not check. If you keep “Notes” for the stylus (item 5), the AI features in it may not work without these packages.

Create the list (on the PC):

```
cat > moto-ai.txt <<'EOF'
# Moto AI models. The Moto AI and Qira apps themselves were removed in items 1–3.
com.motorola.aiservices  # Moto AI Services: AI models for Moto apps
com.motorola.aicore  # Moto AI Core: AI models for Moto apps
EOF
```

Run the removal and reboot the phone (on the PC):

```
./debloat.sh remove moto-ai.txt
sleep 20
adb shell svc power reboot
```

After the reboot, check that the packages are removed (on the PC). The command should print nothing:

```
adb shell pm list packages --user 0 | grep -E 'com.motorola.aiservices|com.motorola.aicore'
```

If something was printed, repeat the removal from the previous block and reboot the phone again with the pause. If a line says `Failure: package is non-disable` instead of `Success`, show the output in the chat from the header, as in item 4.

Check on the phone:

- Settings → Display: adaptive brightness (it may be called “Auto brightness”) is in place. Move the phone from light into shade: the brightness should change.
- The phone charges, and Settings → Battery opens.
- The camera opens and takes pictures.
- Moto gestures work, if you used them, and the “Moto” app (if you kept it in item 4) opens.
- No “… has stopped” windows pop up anywhere.

If something broke, bring the packages back (on the PC):

```
./debloat.sh restore moto-ai.txt
sleep 20
adb shell svc power reboot
```

**What I deliberately did not include**

The `com.motorola.ccc.*` packages (“Motorola Services”) are not in the list. They are related to updates and device management, and I found no open data that they can be removed without consequences. Do not touch them. The only exception, `com.motorola.ccc.notification`, is already in the Moto list from Step 5.

The packages with the models for Moto AI (`com.motorola.aiservices` and `com.motorola.aicore`) are not in `moto.txt` either: they are removed separately, in item 6.

The `com.motorola.gesture` package is not included either. It is not described in open sources, and by the file name (GestureNavigationTutorial.apk) it is a navigation-gesture tutorial. I did not check any link to Moto gestures, the gain from removing it is small, so keep it.

## Step 5a. The remaining packages from the first list

In Step 2 you saved the list of all packages (`baseline_before.txt`): it has 533 lines. The lists of Steps 4–7 already contain 55 of these packages. I went through the rest one by one. The result:

- 171 overlays (the file path contains `/overlay/`) and 30 APEX modules. We do not touch them: see the section “What you must not remove”.
- 277 individual packages: we remove 75 with the lists of this step, 9 more lines are written into the lists as disabled (they start with `#`), and we keep 193. What exactly and why is written in the table at the end of the step.

**How to read the new lists.** A line looks like this: `package  # name: what it does`. If there is no mark at the end of the comment, the purpose of the package is described in open sources. An asterisk `*` means that the package is not described in open sources, and its purpose is clear from the lists, the APK name or a neighboring package. A line with `#` at the start is disabled, and the script skips it; a `?` sign in such a line means that I do not know what the package does. If you want to remove a disabled package, delete the ` #  ` at the start of the line and repeat the removal.

As in Steps 4–7, the commands remove packages only in the main profile (`--user 0`). In the “Google” profile they stay. The Shelter work profile is cleaned in Step 11: the new lists are added there too.

**1. Google leftovers: 13 packages (on the PC)**

What is inside: the Google Fi app (`apps.tycho`) and auto-connect to carriers' Wi-Fi (`carrierwifi`); first-setup service packages (`partnersetup`, `onetimeinitializer`, `searchselector`); sending of error reports (`feedback`); Android Device Policy (`clouddpc`); Android AICore, the Gemini Nano AI model service (`aicore`; this is not the Moto AICore from item 6 of Step 5); ARCore, augmented reality (`ar.core`); Cross-device services (`ambient.streaming`); the Quick Share extension (`mosey`); two packages of voice training for “OK Google” (`hotwordenrollment`).

What may stop working in the main profile: apps with augmented reality, casting apps to a Chromebook, the AI features of apps on Android AICore, Quick Share, voice training for “OK Google”. Without the Google services from Step 7, much of this did not work in the main profile anyway. Android Device Policy is needed only by someone whose phone is managed by an organization (a work account with a device policy): if that is you, delete its line.

Two lines at the end of the list are disabled: `com.google.android.as` (Android System Intelligence: Live Caption, Live Translate, smart auto-rotate) and `com.google.android.as.oss` (Private Compute Services, private analytics for it). If you remove them, these functions will disappear, and what else in Moto depends on them I do not know. Remove them only together.

Create the list (on the PC):

```
cat > google-more.txt <<'EOF'
# Google leftovers from the first list. Line: package  # name.
# No asterisk: the purpose is described in open sources. Asterisk: not described in open sources, the purpose is clear from the lists or from the name.
# Lines starting with # are disabled. If you want to remove such a package, delete the # and run the removal.
com.google.android.apps.tycho  # Google Fi: the app of Google's US carrier
com.google.android.partnersetup  # Google Partner Setup: partner settings of Google apps *
com.google.android.onetimeinitializer  # Google One Time Init: one-time setup after the first start *
com.google.android.feedback  # Google Feedback: sending error reports to Google *
com.google.android.apps.work.clouddpc  # Android Device Policy: management of the phone by an organization *
com.google.android.aicore  # Android AICore: Google's AI model service (Gemini Nano) for apps
com.google.ar.core  # Google Play Services for AR: augmented reality in apps *
com.google.ambient.streaming  # Cross-device services: casting apps to a Chromebook and other devices
com.google.android.mosey  # Quick Share (extension): sending files between devices
com.google.android.apps.carrier.carrierwifi  # Google Wi-Fi Provisioner: auto-connect to carriers' Wi-Fi (Google Fi)
com.google.android.apps.setupwizard.searchselector  # Search engine choice during the first setup *
com.android.hotwordenrollment.okgoogle  # Voice training for “OK Google” *
com.android.hotwordenrollment.xgoogle  # Voice training for “OK Google” *
# com.google.android.as  # Android System Intelligence: Live Caption, Live Translate, smart auto-rotate. Disabled: these functions will disappear, and I do not know what else in Moto depends on it. Remove together with as.oss
# com.google.android.as.oss  # Private Compute Services: private analytics and training for Android System Intelligence, according to Google's description. Disabled for the same reason. Remove together with as
EOF
```

**2. Moto and partners: 23 packages (on the PC)**

What is inside:

- Play Auto Install (`android.autoinstalls.config.motorola.layout`, `playautoinstallext`): the list of apps that Play installs by itself.
- Qira Core (`lenovo.qiracore`): the core of Motorola Qira. The Qira app itself was removed in Step 5.
- “Create with AI” (wallpapers made with AI) and Journal (the Moto AI diary).
- Error reports for Motorola: Bug2Go, Moto Feedback Assistant (`help.extlog`) and Native DropBox Agent (it has nothing to do with Dropbox).
- Two sets of interactive wallpapers (`blackened_blue`, `lily_white`). `com.motorola.livewallpaper3` itself was removed in Step 5.
- Smart Feed: the feed to the left of the home screen.
- Partner stubs: Amazon App Manager, Express Weather, apps for Indonesia (Etalase, Kompas, Vision+) and Italy (Rai News), Signature Club, the PayJoy client and four ironSource Aura packages (app recommendations during setup).

**Smart Feed.** In May 2026, when Amazon was launched, this feed substituted an affiliate code. Motorola called it [unintended](https://www.androidcentral.com/phones/motorola/motorola-affiliate-codes-your-amazon-orders-states-this-was-unintended) and [fixed the settings](https://www.notebookcheck.net/Motorola-comments-on-the-debacle-surrounding-hijacked-shopping-apps.1308600.0.html). How the home screen behaves without the feed, I did not check: see item 6. If something is wrong with the home screen, bring back `com.motorola.smartfeed`.

**PayJoy.** This is a lock client for a phone bought on installments. If the phone was not bought on installments, remove it. If it was, delete the line: what happens to the lock without the client, I did not check.

Six lines are disabled. Screenshot Editor (`screenshoteditor`: screenshot editing, long screenshots, GIFs) and “Secure folder” (`securevault`) were kept because of their functions: if you remove the second package, the files from the folder will become unavailable until you bring it back. Four packages with a question mark (`installer`, `appforecast`, `motosecurecore`, `securityhubext`) are disabled because I did not find out what they do. About `motosecurecore` it is only known that the Moto Secure package removed in Step 5 refers to it.

Create the list (on the PC):

```
cat > moto-more.txt <<'EOF'
# Moto and partner leftovers from the first list. Line: package  # name.
# No asterisk: the purpose is described in open sources. Asterisk: not described in open sources, the purpose is clear from the lists, the APK name or a related package.
# Lines starting with # are disabled: either the function may be needed, or the purpose is unclear (question mark). If you want to remove one, delete the # and run the removal.
android.autoinstalls.config.motorola.layout  # Play Auto Install: the list of apps that Play installs by itself *
com.motorola.playautoinstallext  # Moto: Play Auto Install extension *
com.lenovo.qiracore  # Qira Core: the core of Motorola Qira, the app itself is already gone *
com.motorola.createwithai  # Create with AI: creating wallpapers with AI
com.motorola.journal  # Journal: Moto AI diary (notes, photos, audio, AI summaries)
com.motorola.help.extlog  # Moto Feedback Assistant: extension for “Moto: feedback” *
com.motorola.livewallpaper3.prebuilt.blackened_blue  # Moto interactive wallpapers: the blackened_blue set *
com.motorola.livewallpaper3.prebuilt.lily_white  # Moto interactive wallpapers: the lily_white set *
com.motorola.smartfeed  # Smart Feed: the feed to the left of the home screen; in May 2026 it substituted an affiliate code when Amazon was launched
com.motorola.bug2go  # Bug2Go: collecting error reports for Motorola *
com.motorola.android.nativedropboxagent  # Native DropBox Agent: a Moto service agent, nothing to do with Dropbox *
com.amazon.appmanager  # Amazon App Manager: installer of Amazon apps *
com.handmark.expressweather  # Express Weather: partner weather stub *
id.mjs.etalaseapp  # Etalase Apps: a partner app for Indonesia *
com.ertanto.kompas.official  # Kompas.com: a news app for Indonesia *
it.rainet.android.rainews  # Rai News: news, a partner app for Italy *
com.signatureclub.prod  # Signature Club: a partner app *
com.zte.iptvclient.android.idmnc  # Vision+: television, a partner app for Indonesia *
com.payjoy.access  # PayJoy: lock client for a phone bought on installments (if the phone was bought on installments, delete this line) *
com.ironsource.appcloud.oobe.hutchison  # ironSource Aura (Hutchison): app recommendations during setup *
com.ironsrc.aura.tmo  # ironSource Aura (T-Mobile): app recommendations during setup *
com.orange.aura.oobe  # ironSource Aura (Orange): app recommendations during setup *
com.aura.oobe.vodafone  # ironSource Aura (Vodafone): app recommendations during setup *
# com.motorola.screenshoteditor  # Screenshot Editor: screenshot editing, long screenshots, GIFs. Disabled: the function may be needed
# com.motorola.securevault  # Moto “Secure folder”. Disabled: files from it will become unavailable until you bring the package back
# com.motorola.installer  # ? Moto Installer: Moto's app installation service; I do not rule out a link to updates
# com.motorola.appforecast  # ? Moto App Forecast: app suggestions on the home screen, by the name
# com.motorola.motosecurecore  # ? the Moto Secure core: the removed securityhub refers to it, what else depends on it I do not know
# com.motorola.securityhubext  # ? Moto Secure extension in the system partition, by the name
EOF
```

**3. Carrier stubs: 39 packages (optional, on the PC)**

These are apps and extensions of Verizon, T-Mobile and Metro, AT&T, Xfinity Mobile, Spectrum Mobile, Sprint, Telekom and Orange, the carrier's cloud and preloaded contacts, and two Google packages for US networks (monitoring of the CBRS private network and the address for emergency calls with Wi-Fi calling). Individually they are not described in open sources, and the purpose is visible from the package and APK file names. They are needed only with the SIM cards of these carriers. The list is optional: if you skip it, nothing changes. If you use Wi-Fi calling, delete the line `com.google.android.wfcactivation`. One line is disabled: `com.motorola.brapps` by its name looks like the apps of Brazilian carriers, but I am not sure.

Create the list (on the PC):

```
cat > carriers.txt <<'EOF'
# Stubs of US and European carriers. Line: package  # name. Needed only with the SIM cards of these carriers.
# The purpose is clear from the package and APK file names, these packages are not described individually in open sources.
com.vzw.apnlib  # Verizon: APN settings library
com.vzw.ecid  # Verizon: ECID caller identification
com.vzw.hss.myverizon  # Verizon: the My Verizon app
com.verizon.obdm  # Verizon: OBDM service (stub)
com.verizon.mips.services  # Verizon: MVS Lite service
com.vcast.mediamanager  # Verizon Cloud: backups and media
com.securityandprivacy.android.verizon.vms  # Verizon: Security & Privacy (VMS)
com.motorola.omadm.vzw  # Verizon: device management (OMA-DM)
com.motorola.vzw.cloudsetup  # Verizon: cloud setup
com.motorola.vzw.pco.extensions.pcoreceiver  # Verizon: PCO receiver
com.motorola.vzw.phone.extensions  # Verizon: dialer extensions
com.motorola.vzw.settings.extensions  # Verizon: settings extensions
com.motorola.vzw.provider  # Verizon: unified settings (provider)
com.motorola.vzw.voicemail.extensions  # Verizon: voicemail extensions
com.motorola.android.buacloudcontactadapter  # carrier: cloud contacts (BUA)
com.tmobile.pr.adapt  # T-Mobile: the Adapt client
com.tmobile.echolocate  # T-Mobile EchoLocate: network data
com.tmobile.echolocate.system  # T-Mobile: traffic statistics (EchoLocate)
com.tmobile.rsuadapter.qualcomm  # T-Mobile: remote SIM unlock (RSU), adapter
com.tmobile.rsusrv  # T-Mobile: RSU, server
com.tmobile.rsuapp  # T-Mobile: RSU, app
com.tmobile.m1  # T-Mobile: the screen to the left of the home screen
com.motorola.tmo.settingsext  # T-Mobile: settings extension
com.motorola.tmo.setupext  # T-Mobile: setup wizard extension
com.metro.minus1  # Metro by T-Mobile: the screen to the left of the home screen
com.metropcs.metrozone  # Metro by T-Mobile: MetroZone
com.motorola.att.phone.extensions  # AT&T: dialer extensions
com.motorola.attvowifi  # AT&T: Wi-Fi calling
com.xfinitymobile.cometcarrierservice  # Xfinity Mobile: carrier service
com.motorola.comcastext  # Xfinity Mobile: extensions
com.motorola.comcast.settings.extensions  # Xfinity Mobile: settings extensions
com.spectrum.cm.headless  # Spectrum Mobile: carrier service
com.motorola.spectrum.setup.extensions  # Spectrum Mobile: setup wizard extension
com.motorola.sprint.setupext  # Sprint: setup wizard extension
de.telekom.tsc  # Telekom: App Assistant, app recommendations
com.orange.update  # Orange: carrier updates
com.motorola.contacts.preloadcontacts  # the carrier's preloaded contacts
com.google.android.apps.cbrsnetworkmonitor  # CBRS: private network monitoring in the US
com.google.android.wfcactivation  # Operator setup: the address for emergency calls with Wi-Fi calling at US carriers
# com.motorola.brapps  # ? BRApps2: by the name it looks like the apps of Brazilian carriers, but I am not sure
EOF
```

**4. Run the removal and reboot the phone (on the PC)**

```
./debloat.sh remove google-more.txt
./debloat.sh remove moto-more.txt
./debloat.sh remove carriers.txt
sleep 20
adb shell svc power reboot
```

The answers are the same as in Step 4: `Success` means that the package is removed, “not on the phone or already removed, skipping” means that you do not have such a package. If you do not have the carrier list, the script will say that it did not find the file: this is normal. If a line says `Failure: package is non-disable` instead of `Success`, show the output in the chat from the header, as in item 4 of Step 5. The pause `sleep 20` and the normal reboot are needed for the same reason as in Step 4.

**5. Check that the packages are removed (on the PC)**

The command should print nothing:

```
grep -hEv '^[[:space:]]*(#|$)' google-more.txt moto-more.txt carriers.txt 2>/dev/null | tr -d '\r' | awk '{print "package:" $1}' > names-5a.txt
adb shell pm list packages --user 0 | tr -d '\r' | grep -Fxf names-5a.txt
```

The first line collects into the file `names-5a.txt` the names from the enabled lines of the three lists, and the second shows those of them that are still on the phone. If something was printed, repeat the removal from item 4 and reboot the phone again with the pause.

**6. Check on the phone**

- Calls and SMS go through, mobile internet works. If you use Wi-Fi calling, check it too.
- The home screen opens, and swiping sideways from the main screen does not freeze it (the Smart Feed was there).
- Settings open, and no “… has stopped” windows pop up anywhere.

If something broke, bring the packages back (on the PC):

```
./debloat.sh restore google-more.txt
./debloat.sh restore moto-more.txt
./debloat.sh restore carriers.txt
sleep 20
adb shell svc power reboot
```

Bring back a single package with the command from the “Rollback” section. These three lists are included in the check after an update (Step 10), so that removed packages do not come back unnoticed, and in the work profile (Step 11).

**What I kept and why**

Of the 277 packages I went through, 193 stay. The overlays and APEX modules are added to the table for completeness.

| Group | How many | What is inside and why I kept it |
| --- | --- | --- |
| Overlays | 171 | They adapt the system to the screens, the carrier and the theme. Must not be removed. |
| APEX modules | 30 | Parts of Android that are updated separately (Bluetooth, emergency alerts and others). Must not be removed. |
| Android core | 67 | Everything called `android` or `com.android.*`: contacts, the SIM menu, printing, data providers and so on. The rule from the section “What you must not remove”. |
| Qualcomm | 29 | `com.qualcomm.*`, `com.qti.*`, `vendor.qti.*`, `org.codeaurora.ims`: modem, GPS, Wi-Fi, sound. |
| Google modules for the system | 11 | `packageinstaller`, `documentsui`, `networkstack`, `captiveportallogin`, `ext.shared`, `modulemetadata`, `mainline.adservices`, `mainline.telemetry`, `configupdater`, `webview`, `carrier`. Installing apps, files, network, WebView, settings updates. |
| Moto core | 14 | `launcher3`, `launcherconfig`, `camera5`, `camera.ai.colortone`, `faceunlock`, `batterycare`, `coresettingsext`, `providers.chromehomepage`, `providers.settings`, `systemserver`, `hce`, `thermalservice`, `ccc.devicemanagement`, `ccc.ota`. The home screen, camera, face unlock, battery, settings, updates and device management. |
| Moto service packages | 17 | `motocare`, `paks`, `paks.notification`, `motocit`, `hiddenmenuapp`, `cqawizard`, `cloudconfig`, `lifetimedata`, `bach.modemstats`, `wifi.motowifimetrics`, `dciservice`, `nfwlocationattribution`, `slpc_sys`, `revoker.services`, `enterprise.adapter.service`, `enterprise.service`, `imagertuning_w`. Diagnostics, configuration, modem, Wi-Fi, geolocation. According to open data (Bayton, other Moto models), `motocit` and `bach.modemstats` have declared internet access, `motocare`, `hiddenmenuapp` and `nfwlocationattribution` do not, and for the rest there is no data about internet access. There is almost no benefit from removing them, so their network can be blocked without removing the packages (Step 12a). |
| Foldable screen and desktop mode | 5 | `freeform`, `systemui.desk`, `mobiledesktop.core`, `laptoppanel`, `motcameradesktop`. Needed by the Razr Fold. |
| Moto features | 23 | `actions`, `actioncore`, `touchshare`, `smartmode`, `deptheffect`, `easyprefix`, `dynamicvolume`, `dolbyvisionservice`, `dolby.dolbyui`, `daxservice`, `clockface`, `personalize`, `activestylus`, `mykey`, `smart5g`, `sstservice`, `aptxmode`, `motosignature`, `motosignature2`, `appdirectedsmsproxy`, `iotservice`, `account`, `gesture`. Moto gestures and actions, sound, the stylus, Moto Buds, 5G. Dolby was kept based on the author's check: it showed no internet access for it. |
| First setup and data transfer | 6 | `setup`, `setupwizard.controller`, `setupwizard.devicesetup`, `setupwizard.phoneservice`, `migrateassist`, `motomigrate`. They will be needed after a phone reset. |
| Telephony | 10 | `entitlement`, `omadm.service`, `carrierconfig`, `carriersettingsext`, `visualvoicemail`, `android.provisioning`, `euiccpartnerapp`, `callredirectionservice`, `msimsettings`, `rcsConfigService`. SIM, carrier settings, eSIM, voicemail, RCS. |
| Google packages that I kept | 11 | Described below. |

**Google packages that I did not remove**

- `verifier`: the Android Developer Verifier service, which checks the app developer at installation. In the first launch countries, the verification [applies from September 30, 2026](https://androidauthority.com/android-sideloading-developer-verification-first-wave-rollout-3717921); the details are [at Google](https://developer.android.com/developer-verification/guides). What happens if the package is hidden I do not know, so I kept it. If after a reboot or an update APKs stop installing, first check that the package is in place.
- `apps.turbo`: Device Health Services, battery life estimation. Without it, Settings → Battery [may show the message “Need to pack turbo apk”](https://github.com/ishacker003/turboproxy).
- `apps.safetyhub`: [Personal Safety](https://sysapps.bayton.org/packages/com.google.android.apps.safetyhub), emergency functions such as Emergency SOS and sending the location to contacts (according to the app description). I did not touch it.
- `apps.scone`: [Adaptive Connectivity Services](https://9to5google.com/2020/12/05/google-adaptive-connectivity-services/). Judging by its permissions in the [bayton database](https://sysapps.bayton.org/packages/com.google.android.apps.scone), it is related to the modem settings and metrics. I did not touch it.
- `euicc` (eSIM support), `apps.restore` (data transfer at first setup), `apps.wallpaper` (wallpaper picker), `marvin.talkback`, `accessibility.switchaccess` and `tts` (accessibility and speech synthesis), `apps.wellbeing` (Digital Wellbeing): needed or useful functions with a small gain from removal.

## Step 6. Replace the dialer, SMS, keyboard and the rest

First install the replacements and make them the defaults, check it with commands and only then remove the Google versions: otherwise you will be left without calls, SMS or a keyboard.

**1. Install F-Droid and the replacements (on the phone)**

1. On the phone, open a browser, go to f-droid.org and download the F-Droid store APK. When the system asks, allow the browser to install apps and install the file.
2. Open F-Droid and wait a couple of minutes while the lists update.
3. Find and install the apps from the table. Install Termux from F-Droid in the main profile: it is needed in Step 9 to turn on the auto-stop of the “Google” profile once, and also in Steps 11 and 12. You can do without it, but then you will have to stop the “Google” profile manually with a command from the PC.

| Role | What we remove (Google) | What to replace it with from F-Droid |
| --- | --- | --- |
| Dialer | `com.google.android.dialer` | Fossify Phone |
| SMS | `com.google.android.apps.messaging` | Fossify Messages |
| Contacts | `com.google.android.contacts` | Fossify Contacts |
| Keyboard | `com.google.android.inputmethod.latin` | Fossify Keyboard, or HeliBoard and FlorisBoard with swipe typing |
| Clock and alarm | `com.google.android.deskclock` | Fossify Clock |
| Calendar | `com.google.android.calendar` | Fossify Calendar |
| Calculator | `com.google.android.calculator` | Fossify Calculator |
| Photos and gallery | `com.google.android.apps.photos` | Fossify Gallery |
| Browser | `com.android.chrome` | any to your taste: Firefox, Brave, Vivaldi |

The app names in F-Droid are given from my memory. If something is not there, install any analog. If in Step 2 a role showed a different package than in the table, remove the one the command showed. Because Google Messages is replaced, RCS chats will disappear: ordinary SMS and MMS will remain.

**2. Move the contacts**

The `.vcf` file that you saved before starting is on the PC. Copy it to the phone with a command on the PC, substituting your own file name:

```
adb push contacts.vcf /sdcard/Download/
```

Then on the phone open Fossify Contacts, menu → Import and choose this file. Check that the contacts appeared.

**3. Make the apps the defaults (on the phone)**

- Phone, SMS and browser: Settings → Apps → Default apps. Choose Fossify Phone, Fossify Messages and your browser.
- Keyboard: Settings → System → Keyboard → On-screen keyboard. Turn on the new keyboard and make it the main one.
- Alarm: Settings → Apps → Fossify Clock → Battery → “Unrestricted”. Otherwise the system may silence the alarm.

The menu items on Moto may be named a little differently.

**4. Check that the roles have moved (on the PC)**

```
adb shell cmd role get-role-holders --user 0 android.app.role.DIALER
adb shell cmd role get-role-holders --user 0 android.app.role.SMS
adb shell cmd role get-role-holders --user 0 android.app.role.BROWSER
adb shell settings get secure default_input_method
```

Each command should name the new app. For Fossify the package name starts with `org.fossify.`. If you installed another keyboard or browser, the name will be different. The main thing is that no com.google.android.… or com.android.chrome packages remain. If they remain, go back to item 3.

**5. Remove the Google versions except the dialer (on the PC)**

```
cat > google-roles.txt <<'EOF'
# Google versions that have already been replaced. Line: package  # name
com.google.android.apps.messaging  # Google Messages
com.google.android.contacts  # Google Contacts
com.google.android.inputmethod.latin  # Gboard
com.google.android.deskclock  # Google Clock
com.google.android.calendar  # Google Calendar
com.google.android.calculator  # Google Calculator
com.google.android.apps.photos  # Google Photos
com.android.chrome  # Chrome
EOF
./debloat.sh remove google-roles.txt
sleep 20
adb shell svc power reboot
```

The pause and the normal reboot are here for the same reason as in Step 4.

**If Gboard says `Failure: package is non-disable`**

The package name is correct, the matter is Motorola's protection. [According to reports on XDA](https://xdaforums.com/t/uninstall-bloatware-with-adb.3948300/), on Moto phones every protected package is recorded as a separate file in the `nondisable` folder (usually `/product/etc/nondisable`). The partition is mounted read-only, so the file cannot be erased even with root. But it can be hidden from the system with a root module: the file stays in place, the system does not see it, and `pm` allows removing the package. [Modules that clean this folder](https://xdaforums.com/t/wireguard-kernel-module-for-xt2215-4-motorola-g-stylus-5g-2022.4535379/) have already been installed on other Moto phones. For APatch to attach module files to the system, it needs one more module, a metamodule: we install it in 5d. On the Razr Fold this worked: the file is in /product/etc/nondisable, the module together with the metamodule hid it, and Gboard was removed. Where the file is and what it is called on your phone now is shown by the command in 5b. If at any step the answer is not what is written, stop and show the output in the chat from the header.

**5a. Check the screen lock (on the phone)**

Open Settings → Security → Screen lock (the name of the item depends on the firmware). If a PIN or a pattern is set, go to 5b: they do not need a keyboard on the lock screen. If a password made of letters is set, first change it to a PIN. After a reboot the password is entered before the first unlock, and at that moment the keyboard must be able to work before unlock (in Android this is called Direct Boot). Gboard has this. If you remove Gboard and the new keyboard does not have it, you will not be able to type the password, and only a phone reset will help. [Such a case is described](https://discuss.grapheneos.org/d/16744-no-keyboard-on-lockscreen-on-pixel-9-pro-with-fossify-keyboard/13) on a Pixel with GrapheneOS: Fossify Keyboard was set as the main one, and the password was made of letters.

If you still want to keep a password made of letters, check the chosen keyboard with this command (on the PC):

```
adb shell cmd package query-services -a android.view.InputMethod | grep -E 'packageName=|directBootAware='
```

Next to the package name of your keyboard there must be the line `directBootAware=true`. If it says `false` or there are no such lines, you stay on a PIN. I did not check this command on the Razr Fold.

**5b. Find the Gboard file in the protection list (on the PC)**

The command goes through several partitions and runs for up to a minute:

```
adb shell "find /system /system_ext /product /vendor /odm -path '*nondisable*' -type f 2>/dev/null" | grep -i inputmethod
```

Expected answer: one line with a path, for example `/product/etc/nondisable/nondisable-com.google.android.inputmethod.latin.xml`. The file name on your phone may differ, but most likely it contains `inputmethod.latin`. If nothing is printed, stop: run the same command without `| grep -i inputmethod` and show the whole output in the chat from the header. That means the protection is arranged differently, and the steps below will not fit.

**5c. Build the module (on the PC)**

This block creates the `gboard-module` folder with two files and packs them into `unprotect-gboard.zip`. The module deletes nothing: it puts an empty “stub” over the protection file, and the system stops seeing it. Copy the block whole:

```
cd ~/razr-debloat
mkdir -p gboard-module
cd gboard-module
cat > module.prop <<'EOF'
id=unprotect_gboard
name=Unprotect Gboard (Motorola nondisable)
version=1.0
versionCode=1
author=local
description=Hides the Motorola nondisable marker of Gboard so that pm uninstall --user 0 works
EOF
cat > customize.sh <<'EOF'
# Hides the Motorola protection files (nondisable) from the system. Deletes nothing.
# PATTERN: a file name mask. '*inputmethod.latin*' means Gboard.
PATTERN='*inputmethod.latin*'

FOUND=0
for PART in product system_ext vendor odm system; do
  if [ "$PART" = system ]; then
    SRC=/system/etc/nondisable
    DST=$MODPATH/system/etc/nondisable
  else
    SRC=/$PART/etc/nondisable
    DST=$MODPATH/system/$PART/etc/nondisable
  fi
  [ -d "$SRC" ] || continue
  for F in "$SRC"/$PATTERN; do
    [ -e "$F" ] || continue
    mkdir -p "$DST"
    mknod "$DST/${F##*/}" c 0 0
    ui_print "- Hiding marker: $F"
    FOUND=1
  done
done

[ "$FOUND" = 1 ] || abort "! No marker file matches $PATTERN in /product /system_ext /vendor /odm /system"
set_perm_recursive "$MODPATH/system" 0 0 0755 0644
EOF
python3 -m zipfile -c ../unprotect-gboard.zip module.prop customize.sh
cd ..
adb push unprotect-gboard.zip /sdcard/Download/
```

If `python3` is not found, replace the line with it by `zip ../unprotect-gboard.zip module.prop customize.sh`.

**5d. Install the metamodule (on the PC and on the phone)**

APatch does not attach module files to the system by itself: for that you need one more module, a metamodule. Without it Unprotect Gboard installs without errors but does nothing. The sign is a red notice on the APM tab: “module not mounted, because metamodule not installed”. It disappears after the metamodule is installed and the phone is rebooted. If APM already has some metamodule, you do not need to install a second one: go to 5e. The order matters: first the metamodule and a reboot, then the module. A module installed before the metamodule is not attached: on the Razr Fold it had to be removed and installed again.

Take Magic Mount-rs. It is in the [metamodule list on the APatch website](https://apatch.dev/meta-module.html), APatch support is stated in its description, and it attaches the files by the magic mount method (bind-mount, as in Magisk), without overlayfs in the kernel. There can be only one metamodule on the phone, do not install a second one. It runs as root on every boot, so download it only from the links below. Checked on the Razr Fold: it installed, the red notice disappeared, and the Gboard protection file was hidden after that.

Download it and put it on the phone (on the PC):

```
cd ~/razr-debloat
curl -fL -o magic_mount_rs-arm64.zip https://github.com/Tools-cx-app/meta-magic_mount-rs/releases/download/v4.1.12/magic_mount_rs-4.1.12-959-arm64.zip
adb push magic_mount_rs-arm64.zip /sdcard/Download/
```

If `curl` prints an error (for example, 404), open the [releases page](https://github.com/Tools-cx-app/meta-magic_mount-rs/releases/latest) in a browser, download the file whose name ends with `-arm64.zip` (in the Assets block, not “Source code”) and put it into the Download folder on the phone.

Install it (on the phone): APatch → the APM tab → the install button → choose this zip in the Download folder. If a line starting with `!` appears in the log, stop and show the log in the chat from the header. After the installation, reboot the phone right away (with the button in the log or with the command `adb shell svc power reboot` on the PC): [APatch requires a reboot](https://apatch.dev/meta-module.html) after a metamodule, before installing other modules.

When the phone has booted, open APM: the red notice should be gone. Then 5e. If Unprotect Gboard was already installed before the metamodule, in 5e first remove it and reboot the phone.

**5e. Install the module (on the phone)**

If the module is already in the APM list and Gboard is not yet removed, first remove the module (APM → Remove) and reboot the phone. A module installed before the metamodule is not attached: on the Razr Fold only a reinstall helped. Then open the APatch app and go to the APM tab (APatch modules; the KPM tab does not fit for this). Tap the install button and choose `unprotect-gboard.zip` in the Download folder (Downloads). In the installation log a line `- Hiding marker:` with the path from 5b should appear. Then reboot the phone: with the button in the log or with the command `adb shell svc power reboot` on the PC.

If instead the log says `! No marker file matches`, the module did not install. This is by design: it does not install when there is nothing to hide. Show the output from 5b in the chat from the header.

**5f. Check that the file is hidden (on the PC)**

After the reboot make sure that `adb devices` shows `device`, and repeat the command from 5b:

```
adb shell "find /system /system_ext /product /vendor /odm -path '*nondisable*' -type f 2>/dev/null" | grep -i inputmethod
```

Now the output should be empty. If the Gboard line remains, the module did not take effect: do not go to 5g. First look in APM: if there is still the red notice about the metamodule, repeat 5d. If there is no notice, most likely the module was installed before the metamodule (on the Razr Fold the line remained exactly because of this). Remove the module (APM → Remove), reboot the phone, install the module again (5e), reboot once more and repeat this check. If the line still remains, show in the chat from the header a screenshot of APM and the output of the command.

**5g. Remove Gboard (on the PC)**

```
./debloat.sh remove google-roles.txt
sleep 20
adb shell svc power reboot
```

The `com.google.android.inputmethod.latin` line should now say `Success`, and the already removed packages answer “not on the phone or already removed”. After the reboot open any input field: the new keyboard should appear. If there is no keyboard at all, bring Gboard back with the command `adb shell cmd package install-existing --user 0 com.google.android.inputmethod.latin`, wait 20 seconds, reboot the phone with the command `adb shell svc power reboot` and show in the chat from the header the output of the command `adb shell settings get secure default_input_method`.

**About the module.** While Gboard is removed, do not delete or turn off either the module or the metamodule: without the metamodule the module does not work, and I did not check whether Motorola will bring the protected package back by itself when the file becomes visible again. If after a firmware update Gboard again says `non-disable`, open APatch → APM: the red notice about the metamodule should not be there (if it is, repeat 5d), and the module should be in the list and enabled. If everything is in place and the answer is the same, the file was named differently in the new firmware: repeat 5b–5f. If `non-disable` is reported by some other package, show the output in the chat from the header: the module is fixed with a one-line change.

**If the phone did not boot after installing the module or the metamodule.** This is unlikely: the module hides one file, and the metamodule only attaches module files. But if it did happen, turn on the APatch Safe mode: all modules are turned off in it, and in APM the “Remove” button works: remove what you installed last (the metamodule or the module). [APatch instructions](https://apatch.dev/rescue-bootloop.html): hold the power button until the screen lights up, then press and release it until the first screen appears. The heading of that page also mentions the volume “down” button, the number of presses is not specified, so the method may take several attempts. If it did not work, follow the section “If the phone does not boot” at the end.

After the reboot, check the keyboard, SMS, the alarm and the camera (a photo should open in Fossify Gallery). The full checklist is in Step 8.

**6. Remove the dialer last and with a check (on the PC)**

The “Emergency call” button on the lock screen may depend on the system dialer. So remove the dialer separately and check right away. This has been checked on the Razr Fold: after removing Google Phone (the default dialer is Fossify Phone), the “Emergency call” button opens the dial screen, and ordinary outgoing and incoming calls work. An actual emergency call was not checked. If in Step 2 the dialer was another package, substitute it.

Before removing, check that Fossify Phone is set as the default dialer: the command `adb shell cmd role get-role-holders --user 0 android.app.role.DIALER` should print `org.fossify.phone`, not `com.google.android.dialer`. If Google is printed, you must not remove the dialer: first do item 3.

```
adb shell pm uninstall -k --user 0 com.google.android.dialer
sleep 20
adb shell svc power reboot
```

After booting, lock the phone and tap “Emergency call” on the lock screen. The dial screen should open. You do not need to call. Then call an ordinary number and answer an incoming call. If the dial screen did not open, bring the dialer back and leave it in the system:

```
adb shell cmd package install-existing --user 0 com.google.android.dialer
```

A restored dialer should not interfere: while Fossify Phone is the default dialer, ordinary calls go through it, and the system one is needed only for the emergency call. This is from my knowledge and was not checked on your phone.

## Step 7. Remove Google services from the main profile

This is the last removal step: sign out of your Google account, reboot the phone to check, and hide Play Store, Google Services Framework and Google Play services.

After this step there is no Google in the main profile. Everything that does not work without Google lives in the “Google” profile from Step 3. For the list of what will stop working, see the section “What you get, and at what cost”.

Apps with their own background connection, for example Signal and Telegram, will keep receiving notifications, but will use more battery. This is from my knowledge and was not checked on your phone. Other apps that need push through Google will show notifications with a delay or not show them at all. Run such apps in the “Google” profile.

**1. Make sure the Google profile works**

The profile must be created and checked in Step 3: after this step it is the only place with Google. If you did not create it, go back there. If you did, briefly check it again (on the PC):

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

On the phone open Play Store: it should load. Then go back and turn the profile off (the block waits five seconds between the commands by itself):

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user 0
sleep 5
adb shell am stop-user -f $GU
```

**2. Sign out of the Google account in the main profile (on the phone)**

Settings → Passwords & accounts → Google → your account → “Remove account”. You already moved the contacts to Fossify Contacts in Step 6, and saved photos and files in the backups. Together with the account, everything that was stored only in it on this phone is erased.

**3. Reboot and make sure the previous steps did not break anything (on the PC)**

```
sleep 20
adb shell svc power reboot
```

If after Steps 4–6 the phone works without failures, go on. If not, deal with them first: there is no need to remove the Google services on a phone that does not work.

**4. Create the list and remove (on the PC)**

The order of lines matters: Google Play services are removed last.

If a line gets the answer `Failure: package is non-disable`, stop and show the whole output in the chat from the header. The module from Step 6 is set up only for Gboard, and it would have to be redone for these packages. On the Razr Fold they have no protection: all five answered Success.

```
cat > gms.txt <<'EOF'
# Google services. Line: package  # name
com.android.vending  # Google Play Store
com.google.android.gsf  # Google Services Framework
com.google.android.gms.location.history  # Google Location History (if present)
com.google.android.gms.supervision  # Google Parental Controls, Family Link (if present)
com.google.android.gms  # Google Play services
EOF
./debloat.sh remove gms.txt
sleep 20
adb shell svc power reboot
```

**5. Check the phone**

Wait two minutes after booting. Then check on the PC that what was removed has not come back:

```
adb shell pm list packages --user 0 | grep -E 'package:(com.android.vending|com.google.android.gsf|com.google.android.gms)'
```

The command should print nothing (the `overlay.gmsconfig` lines do not match here: they are not apps). On the Razr Fold, after the pause and the normal reboot, the output was empty. If something was printed, what was removed has come back: repeat item 4 without skipping the pause, and show the output in the chat from the header. If everything is clean, go through the checklist from Step 8. If “app has stopped” messages keep popping up, bring the services back with the command `./debloat.sh restore gms.txt`, wait 20 seconds, reboot the phone with the command `adb shell svc power reboot` and write down the name of the app that was in the message.

## Step 8. Check that the phone works

Go through this checklist after a reboot: if even one item does not work, go to the “Rollback” section while you remember what you changed last.

| Check | How to do it | What should happen |
| --- | --- | --- |
| Boot | `adb shell svc power reboot`, then unlock the phone | The home screen opens, there are no “app has stopped” messages |
| Settings | Open Settings, go to Apps, Battery and Display | All sections open |
| Call | Call an ordinary number and answer an incoming call | Audible both ways |
| Emergency call | Lock the screen and tap “Emergency call” | The dial screen opens. You do not need to call |
| SMS | Send and receive one SMS | Messages go out and arrive |
| Mobile data | Turn off Wi-Fi and open any website | The page loads |
| Wi-Fi and Bluetooth | Connect to a network and to headphones | Both work |
| NFC | Turn NFC on and off in the shade or in Settings | The switch works |
| Unlock | Lock and unlock the phone with a fingerprint or face | Unlocking works |
| Camera | Take a photo and a video, open a shot from the thumbnail | The shot opens in Fossify Gallery |
| Camera internet | If you did Step 12: read the log from item 7 of that step | The log has the lines “com.motorola.camera5: block sent” (and the same for the other packages from block-net.list) and “done”, and there is no error text (an error is allowed only for the line with get-package-networking-enabled) |
| Folding and the outer display | Fold and unfold the phone, open an app on the outer display | The screens switch without failures |
| Sound | Play music or a video, open Settings → Sound | Sound plays, the “Sound” section opens |
| Stylus | If you removed the stylus (item 5 in Step 5): check the stylus connection, draw a line with pressure, press the side button | The stylus is connected, pressure and the side button work |
| Brightness and charging | If you removed the Moto AI packages (item 6 in Step 5): open Settings → Display, check adaptive brightness, connect the charger | Brightness adjusts to the light, charging works |
| Keyboard | Type text in any field | The new keyboard appears |
| Alarm | Set it for two minutes ahead and turn off the screen | The alarm rings |
| GPS | Go outside and open maps without Google (for example, OsmAnd or Organic Maps from F-Droid) | The position is found. The first search may take a few minutes: there is no Wi-Fi positioning anymore |
| Google profile | Turn on the “Google” profile (Step 9) and open Play Store | Play Store works. Then turn the profile off again |

If an app crashes, look on the PC at what exactly crashed:

```
adb logcat -b crash -d | tail -n 60
```

In the output find the line with `Process:`: it has the name of the package that is crashing. If this package was in one of your lists, bring it back with the command from the “Rollback” section.

## Step 9. How to turn the “Google” profile on and off

Turn the profile on by switching the user, and make the turning off automatic with one command: then it is enough to return to the main profile, and “Google” will stop by itself.

While the profile runs in the background, it uses battery and data. By default Android does not stop background users by itself, so this setting has to be turned on explicitly.

**1. Once: turn on auto-stop (on the phone, in Termux)**

You need Termux with root. If you do not have Termux yet, install it from F-Droid in the main profile. To give it root, open the SuperUser tab in APatch Manager, find Termux and turn on root for it; if the tab is named differently, look for a list of apps with root switches. Then open Termux (in the main profile) and type `su -c id`: it should print `uid=0(root)`. Then type the command (the block can be pasted whole):

```
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
su -c "pm set-user-restriction --user $GU no_run_in_background 1"
```

This command forbids the “Google” profile to run in the background. According to the Android sources, such a profile is stopped when you switch to another user. This can be seen in the [AOSP commit, a copy in the /e/OS mirror](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/ffbe9543d74665595401be61f7a1ad71612c5ed0): the authors checked manually that the profile stopped. On the author's Razr Fold the command works and prints nothing on success, and you should check the stopping of the profile in item 3. If the command printed an error or turning on root for Termux failed, skip this item and stop the profile manually, as described in item 3.

Why not from the PC: on the Razr Fold the same command from adb shell answers `SecurityException: You need MANAGE_USERS permission to: setUserRestriction`. The adb shell on this firmware does not have the MANAGE\_USERS permission, and the profile restriction requires it. Root passes this check, so run the command above from Termux through su, not from the PC.

**2. Turn the profile on when you need the Google apps**

- Without root: Settings → System → Multiple users → “Google”. The same can be done with the user icon on the lock screen or in the shade, if you have one. If there is no “Multiple users” item in Settings, use the command from the PC below.
- From the PC: the block of commands below, under the list.
- With a button on the home screen: see “Buttons in Termux” below.

Turning on from the PC (on the PC):

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am switch-user $GU
```

**3. Turn the profile off and check that it stopped**

Return to the main profile: Settings → System → Multiple users → the first user (the owner). Wait ten seconds and check on the PC:

```
adb shell pm list users
```

The line with `Google` must not have the word `running`. If it does, auto-stop did not work. Then stop the profile manually after each use with a command on the PC:

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
adb shell am stop-user -f $GU
```

If you made the buttons from item 4 below, the profile can be stopped with the `google-off` button: it finds the profile number by itself.

**4. Buttons in Termux (optional)**

If you want to turn the profile on with one tap without a PC, create two scripts. You already turned on root for Termux in item 1.

1. Install the Termux:Widget app from F-Droid. It must be from the same source as Termux.
2. In Termux type the commands (the block can be pasted whole):

```
mkdir -p ~/.shortcuts
cat > ~/.shortcuts/google-on.sh <<'EOF'
#!/data/data/com.termux/files/usr/bin/sh
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
[ -n "$GU" ] && su -c "am switch-user $GU"
EOF
cat > ~/.shortcuts/google-off.sh <<'EOF'
#!/data/data/com.termux/files/usr/bin/sh
GU=$(su -c 'pm list users' | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
[ -n "$GU" ] && su -c "am stop-user -f $GU"
EOF
chmod +x ~/.shortcuts/google-on.sh ~/.shortcuts/google-off.sh
```

3. Long-press an empty place on the home screen → Widgets → Termux:Widget, and place the widget. Tap `google-on` to switch to the “Google” profile.

The `google-off` script works only from the main profile. If the auto-stop from item 1 works, it is almost not needed.

**Tip: set a PIN on the “Google” profile**

Without a lock, anyone who has unlocked the phone can open this profile, and your Google account is in it. Switch to it and set a PIN: Settings → Security → Screen lock.

## Step 10. After flashing a kernel or updating the system

Flashing a boot image with a patched kernel does not touch the data partition, where the list of hidden apps is stored, and after a system update you need to check whether packages have come back and repeat the removal.

**Why hidden apps should stay hidden**

The list of hidden packages is in `/data/system/users/0/package-restrictions.xml`. The `fastboot flash boot` command replaces only the boot image. An update on the A/B scheme writes the new system to the neighboring slot. The data partition stays the same. An update can only add new packages that did not exist before. This follows from how Android is built. I did not check it on your phone, so after the first flash do the check below.

**Check after flashing a kernel (on the PC)**

Run the removal for all the lists again. The script will skip the hidden packages and remove the ones that came back:

```
./debloat.sh remove google.txt
./debloat.sh remove moto.txt
./debloat.sh remove moto-extra.txt
./debloat.sh remove moto-ai.txt
./debloat.sh remove google-roles.txt
./debloat.sh remove gms.txt
./debloat.sh remove google-more.txt
./debloat.sh remove moto-more.txt
./debloat.sh remove carriers.txt
```

If every line says “not on the phone or already removed”, nothing came back. If somewhere it says `Success`, the package came back and is now removed again. If you do not have the files `moto-extra.txt` or `moto-ai.txt` (you skipped item 4 or 6 in Step 5), the script will say that it did not find the file: this is normal. The same applies to `google-more.txt`, `moto-more.txt` and `carriers.txt` from Step 5a. The dialer and the stylus are not in the lists, check them separately (on the PC):

```
adb shell pm list packages --user 0 | grep -E 'com.google.android.dialer|com.motorola.stylus'
```

If a line with the exact package name is printed, and you removed this package earlier (the dialer in Step 6, the others in Step 5), remove it again with the command `adb shell pm uninstall -k --user 0 PACKAGE_NAME`, substituting the name from the line. If you kept some package deliberately, its line will be printed: that is as it should be. Do not touch lines with other names (for example, ones ending in `overlay`). Then reboot the phone with the pause, as in Step 4:

```
sleep 20
adb shell svc power reboot
```

**Shelter work profile.** If you cleaned it (Step 11), after this reboot repeat items 4–6 from Step 11 in it: the script will remove the packages that came back there too. If the `profile-remove.sh` file is no longer on the phone, start with item 3.

**Searching for new packages after a system update (on the PC)**

After a system update, not just a kernel flash, also look for new packages. Compare today's list with the “before” snapshot from Step 2:

```
adb shell pm list packages -f -s -u --user 0 | sort > baseline_after.txt
sed 's/.*=//' baseline_before.txt | sort > names_before.txt
sed 's/.*=//' baseline_after.txt | sort > names_after.txt
comm -13 names_before.txt names_after.txt
```

The last command prints the packages that were not in the “before” snapshot. If among them there are Google or Moto apps, add them to the appropriate list and run the removal. Do not touch packages that are not in this guide's lists until you find out what they are. A line can be added to a list like this:

```
echo 'com.example.newpackage  # name' >> moto.txt
```

**Before a system update over the air (OTA)**

For now we skip updates. When you get to them, take the following into account.

- Do not hide `com.motorola.ccc.ota`: it is the Motorola update client. If you removed `com.motorola.ccc.notification` (it is in the Moto list from Step 5), bring it back before the update with the command `adb shell cmd package install-existing --user 0 com.motorola.ccc.notification`, and after the update it will be removed again when the lists are run again (the check above). This is a precaution, I did not check it.
- If you have a Shelter work profile where Moto and Google junk is removed (Step 11), after the update repeat items 4–6 from Step 11 in it: the script will remove the packages that came back again.
- An OTA on a phone with a patched boot may fail the check. A general technique for rooted devices: before the OTA bring back the stock boot with the command `fastboot flash boot boot_stock.img`, install the update, and after the first boot patch the new boot again in APatch and flash it. I did not check how this works with APatch on the Razr Fold.
- After the first boot with the new system, go through the two checks above: running the lists again and searching for new packages. If you did Step 12, repeat its item 7 (after a kernel change also item 3, the Termux check): after an update or a kernel change the block may not have turned on. If you blocked the network for other packages (Step 12a), repeat its items 1–3 and compare the list with the previous one: you will sort out the new packages with internet access by the same table. If you cleaned the Shelter work profile (Step 11), repeat its items 4–6. Then go through the checklist from Step 8.

**The Gboard protection module and the metamodule.** They are in `/data/adb/modules`, a system update does not touch `/data`, but the module works only while APatch is active, that is, after the boot is patched again. If when running the lists again Gboard says `non-disable`, open APatch → APM: the red notice about the metamodule should not be there (if it is, repeat 5d), and the module should be in the list and enabled. If everything is in place and the answer is the same, repeat 5b–5f from Step 6.

## Step 11. The Shelter work profile: remove Moto and Google junk, as in the main profile

The commands of Steps 4–7 remove apps only in the main profile: they all have `--user 0`. The work profile that Shelter creates is a separate user of the phone with its own list of hidden apps, and Moto and Google appear in it again. This step removes there the same things we removed in the main profile: the Google apps (`google.txt`), Moto junk (`moto.txt`, `moto-extra.txt`, `moto-ai.txt`), the Google versions from `google-roles.txt`, the Google dialer, Moto Notes and the Google services from `gms.txt` (Play Store, Services Framework, Google Play services) and the leftovers from Step 5a (`google-more.txt`, `moto-more.txt`, `carriers.txt`).

When you install Shelter and create a work profile, Google will most likely appear in it again. A work profile is one more user of the phone, and Shelter [only asks the system to add it](https://discuss.privacyguides.net/t/why-shelters-work-profile-comes-pre-loaded-with-some-google-services/14153): a user there writes that right after creating the profile they found 3–4 Google apps in it. Which system apps stay in such a profile is decided by the firmware: the manufacturer has a [file with the list of mandatory work profile apps](https://bayton.org/notes/103/), and in the [Bayton analysis](https://bayton.org/android/what-are-vital-apps/) Google Play services and Play Store are named among the mandatory ones for a work profile. What the Moto firmware will leave, I do not know in advance: it is visible only after the profile is created.

The main profile will not change from this: every user has their own list of hidden apps, and the commands with `--user 0` touched only the main one. This is how Android is built; I did not check it on your phone.

Do this step after Steps 1–10, when the main profile is already clean, and only if you have or will have a Shelter work profile. You need Termux with root. If you do not have it, install Termux from F-Droid in the main profile and turn on root for it following the actions from Step 9, item 1.

**1. Create the profile (on the phone)**

Install Shelter and go through its setup: it will create the work profile itself.

**2. Find out the profile number (on the PC)**

```
adb shell pm list users
```

Find the new line of the work profile: a name like “Work profile” (or its translation, if the phone is not in English). The number after `{` is its number, below it is 12 everywhere. **The number must be exactly that of the work profile.** If you substitute the number of the “Google” profile, the script will remove Google from there.

**3. Build the script from your lists (on the PC)**

The adb shell cannot enter the work profile: for commands with `--user 12` it answers `Shell does not have permission to access user 12` ([example](https://discuss.grapheneos.org/d/26196-installing-apps-via-adb-to-work-profile)). In the Android sources this check [fires only for the adb shell](https://gitlab.e.foundation/e/os/android_frameworks_base/-/commit/8cd28b57ed732656d002d97879e15c5695b54fff), and root is not affected by it. So root in Termux will run the commands, and the packages will be taken from your own lists: the script is built from `google.txt`, `moto.txt` and the other files. Protected packages (the same list as in `debloat.sh`) will not get into it.

```
N=12
{ echo com.google.android.dialer; echo com.motorola.stylus; grep -hEv '^[[:space:]]*(#|$)' google.txt moto.txt moto-extra.txt moto-ai.txt google-roles.txt gms.txt google-more.txt moto-more.txt carriers.txt 2>/dev/null; } | tr -d '\r' | awk -v n="$N" '$1 == "com.motorola.ccc.notification" || $1 !~ /^(android|com\.android\.(systemui|settings|phone|server\.telecom|providers\..*)|com\.qualcomm\..*|com\.motorola\.ccc\..*|.*overlay.*|com\.google\.android\.webview|com\.google\.android\.trichromelibrary)$/ { print "pm uninstall -k --user " n " " $1 " 2>&1 | grep -q Success && echo \"removed: " $1 "\"" }' > profile-remove.sh
adb push profile-remove.sh /data/local/tmp/profile-remove.sh
```

In the first line replace 12 with your own number. If you do not have some of the files, that is normal. The Google dialer and Moto Notes are added to the start of the list separately: in the main profile they were removed with their own commands (Steps 6 and 5). The `com.motorola.ccc.notification` package is removed in the profile as an ordinary one, as in Step 5; the script still skips the other `com.motorola.ccc.*`. The lists from Step 5a are connected the same way. The command silently skips packages that are not in the profile.

**4. Run the script (on the phone, in Termux)**

```
su -c 'sh /data/local/tmp/profile-remove.sh'
```

For each removed package “removed: …” will be printed. The script silently skips packages that are not in the profile and packages that the system did not let it remove: the ones that remain will be shown by the check in item 5. If root works in adb on your phone (`adb shell su -c id` prints `uid=0(root)`), the same command can be run from the PC: `adb shell "su -c 'sh /data/local/tmp/profile-remove.sh'"`.

**5. Check (on the phone, in Termux)**

```
comm -13 <(su -c 'pm list packages --user 0' | sort) <(su -c 'pm list packages --user 12' | sort) | grep -E 'google|motorola'
```

The command shows the Google and Moto packages that are in the work profile but are no longer in the main one. It should print nothing. If something was printed, show the output in the chat from the header. Do not touch other Google system packages in the profile, for example `com.google.android.permissioncontroller` or `com.google.android.webview`. Then reboot the phone with the pause (item 6).

**6. Reboot the phone with the pause (on the PC)**

```
sleep 20
adb shell svc power reboot
```

The pause is needed for the same reason as in Step 4. After the reboot repeat the check from item 5: it again should print nothing.

**If something broke in the profile.** The simplest way is to delete the profile (Step 1a) and create it again. A single package can be brought back with the command (on the phone, in Termux):

```
su -c 'cmd package install-existing --user 12 PACKAGE_NAME'
```

**After a system update.** If the profile number has not changed, run items 4–6 again (the script, the check, the reboot with the pause): the script will remove the packages that came back again.

I ran the script assembly and the check from item 5 on test lists. I did not check running through root in the profile on your phone, nor which exact Moto packages will stay in the profile.

## Step 12. Block internet access for the regular Moto camera

The regular camera `com.motorola.camera5` cannot be removed: without it the phone cannot shoot, and it is in the “What you must not remove” section. So we will cut off only its internet access. The INTERNET permission shows only that the camera can reach the network, not that it does. For ordinary shooting, to my knowledge, the network is not needed.

**How it works.** Android has a system-level network ban for an individual app: it does not use a VPN and does not need extra apps. It is controlled by the `cmd connectivity` commands. According to the description of the ShizuWall app, which uses the same mechanism, these commands work on Android 11 and newer, and the rules are reset at every reboot. So after the check we will put a small script on the phone: APatch runs it at every boot and turns the ban on again. The script lives in the `/data/adb/service.d` folder, and a system update or reflashing the boot image does not erase it.

Do this step after Steps 1–10. You need Termux with root, as in Step 11. I did not check these commands on your Razr Fold, so first check the ban on Termux (item 3) and only then block the network for the camera.

**1. Check that the phone knows these commands (on the PC)**

```
adb shell cmd connectivity help | grep -iE 'chain3|package-networking'
```

Lines with the words `set-chain3-enabled` and `set-package-networking-enabled` should be printed (there may also be lines with `get-` next to them). If nothing was printed, this firmware does not have such commands: stop and show the output of the `adb shell cmd connectivity help` command in the chat from the header, where they will suggest a fallback method.

**2. Find out whether the camera shares its number with other apps (on the PC)**

Android cuts the network by the app number (uid), not by the package name. If other packages share this number with the camera, they will lose internet access too.

```
U=$(adb shell pm list packages -U --user 0 | tr -d '\r' | sed -n 's/^package:com\.motorola\.camera5 uid:\([0-9]*\)$/\1/p'); echo "camera number: $U"; adb shell pm list packages -U --user 0 | tr -d '\r' | grep " uid:$U\$"
```

The first line will show a number of 10000 or more, and under it there will be exactly one line `package:com.motorola.camera5 uid:…`. If the number is empty, there is no such package on the phone: show the output of the command `adb shell pm list packages --user 0 | grep camera` in the chat from the header. If there are several lines under the number or it is less than 10000, stop and show the output in the chat from the header.

**3. Check the ban on Termux (on the phone and on the PC)**

First check the ban not on the camera but on Termux: that shows whether it works on your firmware. Make sure Termux has network access (on the phone, in Termux):

```
pkg install -y curl
curl -sS -m 5 -I https://example.com
```

Response headers should be printed, and the first line starts with `HTTP/`. Block Termux's network (on the PC):

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.termux
```

Repeat `curl -sS -m 5 -I https://example.com` in Termux (on the phone): now it should end with an error, for example `Could not resolve host` or `timed out`. If curl printed the headers again, the ban does not work on your firmware: stop and show the output in the chat from the header. If everything is as described, give Termux its network back (on the PC):

```
adb shell cmd connectivity set-package-networking-enabled true com.termux
```

**4. Block the camera's internet access (on the PC)**

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.motorola.camera5
```

The commands print nothing. Open the camera and shoot a photo and a video: shooting should work as before. If the camera does not open or the internet disappeared on the whole phone, reboot the phone with the command `adb shell svc power reboot`: the rules will be reset. Then describe in the chat from the header what happened. If the help output from item 1 had the line `get-package-networking-enabled`, you can check the ban: the answer for the camera should differ from the answer for an ordinary app, for example for `com.android.settings`:

```
adb shell cmd connectivity get-package-networking-enabled com.motorola.camera5
adb shell cmd connectivity get-package-networking-enabled com.android.settings
```

**5. Create the list and the script (on the PC)**

The list is built like the others: the package, then `#` and the name. To block the network for one more app, add its line to the list (the command is in item 6 of Step 12a) and repeat items 6 and 7. But first go through Step 12a: it shows which packages need the network and which do not, and checks that the package has its own number (uid). If the number is a system one or a shared one, the ban will cut the network for other packages too.

```
cat > block-net.list <<'EOF'
# Apps without internet. Line: package  # name
com.motorola.camera5  # regular Moto camera
EOF
```

The script:

```
cat > block-net.sh <<'EOF'
#!/system/bin/sh
# Block internet access for the apps in the list. APatch runs the script at every boot.
export PATH=/system/bin:/system/xbin:$PATH
LIST=/data/adb/block-net.list
LOG=/data/adb/block-net.log
exec > "$LOG" 2>&1
n=0
until [ "$(getprop sys.boot_completed)" = "1" ] || [ "$n" -ge 150 ]; do sleep 2; n=$((n+1)); done
sleep 15
echo "start: $(date)"
cmd connectivity set-chain3-enabled true </dev/null
grep -Ev '^[[:space:]]*(#|$)' "$LIST" | tr -d '\r' | while read -r pkg _; do
  cmd connectivity set-package-networking-enabled false "$pkg" </dev/null
  echo "$pkg: block sent"
  cmd connectivity get-package-networking-enabled "$pkg" </dev/null
done
echo "done"
EOF
```

The script waits for the end of the boot and 15 more seconds, turns on the general switch and blocks the network for every package in the list. It writes the result to `/data/adb/block-net.log`. The line with `get-package-networking-enabled` is needed only for the log: if your firmware does not have such a command, the log will show an error on that line, but the ban will still work.

**6. Put the files on the phone (on the PC and on the phone)**

Copy the files to the phone (on the PC):

```
adb push block-net.sh /data/local/tmp/block-net.sh
adb push block-net.list /data/local/tmp/block-net.list
```

Install them (on the phone, in Termux):

```
su -c 'mkdir -p /data/adb/service.d && cp /data/local/tmp/block-net.sh /data/adb/service.d/block-net.sh && chmod 755 /data/adb/service.d/block-net.sh && cp /data/local/tmp/block-net.list /data/adb/block-net.list && chmod 644 /data/adb/block-net.list && echo done'
```

“done” should be printed. If not, show the entire output in the chat from the header.

**7. Reboot the phone and check the log (on the PC and on the phone)**

Reboot the phone (on the PC):

```
adb shell svc power reboot
```

Unlock the phone, wait a minute and read the log (on the phone, in Termux):

```
su -c 'cat /data/adb/block-net.log'
```

The log should contain the line `start:` with a date, then `com.motorola.camera5: block sent`, the answer to `get-package-networking-enabled` and, at the end, `done`. If the file is missing or empty, APatch did not run the script: check the permissions with the command `su -c 'ls -l /data/adb/service.d/block-net.sh'` (the line should start with `-rwxr-xr-x`) and show the output in the chat from the header. If the log has an error, for example `Unknown command`, show the whole log in the chat from the header. Then go through the checklist from Step 8.

**8. How to remove the ban (on the PC and on the phone)**

Immediately, without a reboot (on the PC):

```
adb shell cmd connectivity set-package-networking-enabled true com.motorola.camera5
```

So that the ban does not turn on again at the next boot (on the phone, in Termux):

```
su -c 'rm /data/adb/service.d/block-net.sh'
```

**System updates and kernel changes.** The script and the list live in `/data/adb`, so a system update, reflashing the boot image and replacing the kernel do not affect them while APatch is working on the phone. While root has not come back (for example, between the update and the repeated flashing of boot from Step 10), the camera's internet is not closed. As far as I know, the rule is stored in the system BPF tables, not in kernel modules like netfilter, so a custom kernel for Droidspaces should not break it; this is my assumption, not a check. After an update or a kernel change repeat item 7, and after a kernel change also the Termux check from item 3: the log shows only that the command was sent, not that the ban is in effect.

**What I did not check on your phone.** The `cmd connectivity` commands themselves on this firmware, running the script from `service.d` under APatch, and whether the ban survives an update. That is why items 3 and 7 are mandatory. The ban applies only to the main profile: it does not affect the cameras in the “Google” profile and in the Shelter work profile, if they exist there.

## Step 12a. Which of the remaining packages can still reach the internet

This step answers the main question of the article: whether data leaves the phone through the packages we left in place. To send anything, a package needs the INTERNET permission. If it does not have it, the package will not reach the network on its own (it can only pass data to another app that does have network access). If it has the permission, that is not yet proof that the package sends anything: it shows only a possibility. The connections themselves cannot be seen without a network monitor, so we proceed like this: we find the packages with internet access, look at why they need the network, and close it for those that do not need it. The package stays in place, the method is the same as in Step 12, and the block is removed with a single command.

**What I found in open sources.** I searched for descriptions of the remaining service packages by name. The most data is in the [Bayton system apps database](https://sysapps.bayton.org/): it collects package names and permissions from various Moto models. This is a description of a sample, not a verdict, and the data is not from the Razr Fold (mostly the edge 50 fusion and the moto g series), so the permissions on your phone may differ. For the 17 Moto service packages from Step 5a the result is this: two have INTERNET declared, three do not, and for the other twelve there is no open data about internet access; the check on the phone itself comes below.

- [`com.motorola.motocit`](https://sysapps.bayton.org/packages/com.motorola.motocit) (CQATest): judging by the name and permissions (like `MOT_CAM_TEST` and `QCOM_DIAG`), this is the manufacturer's test app, and it is not shown to the user. It declares INTERNET, precise location, camera, microphone, body sensors and phone state. An ordinary owner does not need it, so close the network for it first.
- [`com.motorola.bach.modemstats`](https://sysapps.bayton.org/packages/com.motorola.bach.modemstats) (Modem Service): declares INTERNET, location, reading system logs and app usage statistics.
- [`com.motorola.ccc.devicemanagement`](https://sysapps.bayton.org/packages/com.motorola.ccc.devicemanagement) (Device Management): INTERNET, reading logs and the list of installed apps, no location. It belongs to the `ccc.*` family, which is tied to Motorola updates, so the script does not touch it, and you should not close the network for it either.
- [`com.motorola.omadm.service`](https://sysapps.bayton.org/packages/com.motorola.omadm.service) (Carrier Provisioning Service): INTERNET and the right to work with restricted carrier networks, no location or logs. Judging by the name and permissions, the carrier uses it to configure the phone under the OMA DM standard.
- [`com.motorola.paks`](https://sysapps.bayton.org/packages/com.motorola.paks) (PAKS): according to a moderator's reply on the [Lenovo forum](https://forums.lenovo.com/t5/moto-g-5G-2023/PAKS/m-p/5273709), it is a component for supply chain protection and pre-sale preparation, tied to the lower software layers. Among its permissions is INSTALL\_EXISTING\_PACKAGES, which technically allows bringing hidden packages back. I found no data that it does this; if the removed packages do come back after a reboot or an update, the check from Step 10 will catch them.
- [`com.motorola.motocare`](https://sysapps.bayton.org/packages/com.motorola.motocare) (Device Care), [`com.motorola.hiddenmenuapp`](https://sysapps.bayton.org/packages/com.motorola.hiddenmenuapp) (HiddenMenu), [`com.motorola.iotservice`](https://sysapps.bayton.org/packages/com.motorola.iotservice) (Moto Iot Service) and [`com.motorola.nfwlocationattribution`](https://sysapps.bayton.org/packages/com.motorola.nfwlocationattribution) (Carrier Location Access) do not declare internet access, so they will not send anything themselves. What they declare is something else: Device Care declares app usage statistics and the list of all apps, HiddenMenu declares location and the call log, Iot Service declares the connection with Moto Buds, and Carrier Location Access declares location, including background location.
- [`com.google.mainline.telemetry`](https://sysapps.bayton.org/packages/com.google.mainline.telemetry) (“Support components”) and [`com.google.mainline.adservices`](https://sysapps.bayton.org/packages/com.google.mainline.adservices) (“Main components”): service version tags of the Google Play System Updates modules; they have no interface, and they are not apps. The Privacy Sandbox ads themselves (topics, app-based ads, ad measurement) live in the AdServices module ([description in AOSP](https://source.android.com/docs/core/ota/modular-system/adservices)). Bayton advises managing it through the Android privacy settings rather than removing the packages: find the section about ads and Privacy Sandbox in Settings (the name on Moto may differ) and turn the switches off there.
- Only a name and the number of permissions (from 2 to 19), with no description, were found for `lifetimedata` (“LifetimeData”), `dciservice` (“Moto DCI Service”), `paks.notification` (“PAKS Notification”), `slpc_sys` (“Motorola Slpc System”), `revoker.services` (“Revoker Services”), `enterprise.service` (“Moto Thinkshield-MM”), `enterprise.adapter.service` (“Enterprise Adapter Service”) and `wifi.motowifimetrics` (no name). This is data [from the edge 50 fusion package list](https://sysapps.bayton.org/devices/117). The packages `cloudconfig`, `cqawizard` and `imagertuning_w` are not in it, and I did not find them by name either.

**1. Build the list of packages with internet access (on the PC)**

The command asks each system package of the main profile whether it declared INTERNET (overlays are skipped). It runs for about a minute and changes nothing:

```
adb shell 'pm list packages -s -U --user 0 | sed "s/^package://" | while read -r p u; do case "$p" in *overlay*) continue;; esac; if dumpsys package "$p" </dev/null | grep -q "android.permission.INTERNET"; then echo "INTERNET $u $p"; else echo "no-net $u $p"; fi; done' | tr -d '\r' | sort -k3 > audit_net.txt
wc -l audit_net.txt
```

The file will contain lines of the form `INTERNET uid:10123 com.package` (internet declared) and `no-net uid:10124 com.package` (not declared). If the file is empty, adb does not see the phone: check `adb devices`.

**2. Keep the Moto, Lenovo, Google and Qualcomm packages with internet access (on the PC)**

```
grep '^INTERNET' audit_net.txt | grep -E 'motorola|lenovo|google|qualcomm|qti|codeaurora' > audit_internet.txt
cat audit_internet.txt
```

This is the list of those that can reach the network. The packages `android` and `com.android.*` are not included: their code is open, and almost all of them are part of the system itself.

**3. Remove the packages whose network must not be cut (on the PC)**

As in item 2 of Step 12, Android cuts the network by the app number (uid), not by the package name. Do not touch packages with a number below 10000 (that is the system itself, and other services will lose the network) or packages that share a number with others (everyone sharing it will lose the network at once). The commands keep only the packages with their own number of 10000 or above:

```
awk '{print $2}' audit_net.txt | sort | uniq -d > audit_shared_uids.txt
grep -Ev 'uid:[0-9]{1,4} ' audit_internet.txt | grep -Fvwf audit_shared_uids.txt > audit_can_block.txt
cat audit_can_block.txt
```

`audit_can_block.txt` now holds the packages whose network can be closed technically. Whether you should is decided by the next item. If the package you need is not in the list, it has no internet access or its number is a system one or a shared one: there is nothing to close there, or it cannot be done.

**4. Decide whose network to close**

| Package | What to do | Why |
| --- | --- | --- |
| `com.motorola.ccc.*` (Motorola updates and device management) | Do not close | They are needed for updates, and the guide's condition is that we do not break updates. |
| Everything about SIM, eSIM, Wi-Fi calling, RCS and carrier settings: `entitlement`, `carrierconfig`, `carriersettingsext`, `euicc`, `omadm.service`, `com.google.android.carrier` | Do not close if you use mobile service, eSIM or Wi-Fi calling | They fetch settings from the carrier over the network. |
| Network checks, installing and updating system data: `networkstack`, `captiveportallogin`, `packageinstaller`, `configupdater`, `verifier`, `webview` | Do not close | Without the network you lose the connectivity check, the Wi-Fi sign-in window, app verification and installation, and certificate updates. |
| GPS and location: `com.qualcomm.*`, `apps.safetyhub` | Do not close | GPS downloads assistance data, and emergency functions send the location. |
| `paks`, `paks.notification` | Leave as is | The manufacturer calls PAKS part of the device's protection, and what happens without the network is unknown. |
| Moto diagnostics, statistics, factory tests, enterprise management and cloud configuration: `motocit`, `bach.modemstats`, `lifetimedata`, `dciservice`, `cloudconfig`, `enterprise.service`, `enterprise.adapter.service`, `wifi.motowifimetrics`; from Google `apps.turbo` | Close the network and test the phone | By their names and descriptions, they are not needed for calls, SMS, the camera or Wi-Fi, or for updates. |

If a package is neither in the table nor in `audit_can_block.txt`, do not touch it. A description of any package can be found by name in the [Bayton database](https://sysapps.bayton.org/) (the page `sysapps.bayton.org/packages/package_name`).

**5. Close the network as a trial (on the PC)**

Start with one or two packages. The rule takes effect immediately and is reset on reboot, as in Step 12, so nothing becomes permanent:

```
adb shell cmd connectivity set-chain3-enabled true
adb shell cmd connectivity set-package-networking-enabled false com.motorola.motocit
```

As long as the phone has not rebooted, use it as usual (for at least a day) and check calls, SMS, mobile internet, Wi-Fi and the camera. If the phone rebooted, the rule was reset: repeat the command. If something broke, give the network back (on the PC):

```
adb shell cmd connectivity set-package-networking-enabled true com.motorola.motocit
```

**6. Make the ban permanent (on the PC)**

If everything works, add the package to the list. Below is the command for `com.motorola.motocit`; for another package, replace the line:

```
cat >> block-net.list <<'EOF'
com.motorola.motocit  # CQATest: the manufacturer's test app
EOF
cat block-net.list
```

The output should contain the camera line and the new line. If the file `block-net.list` is no longer on the PC, create it again following item 5 of Step 12 and add all the lines to it. Then repeat items 6 and 7 of Step 12: after the reboot the log will show a block line for every package in the list. If something broke because of a package, remove its line from the list, repeat item 6 of Step 12 and give the network back with the command from item 8 of that step.

**After a system update or a kernel change.** New firmware versions can bring packages with internet access. After the first pass through items 1–3, save the list (on the PC):

```
cp audit_internet.txt audit_internet_before.txt
```

After the update, repeat items 1–3 and compare (on the PC, in Bash):

```
diff <(cut -d' ' -f3 audit_internet_before.txt) <(cut -d' ' -f3 audit_internet.txt)
```

Lines starting with `>` appeared after the update: sort them out using the table above. Closing the network works through a script in `/data/adb`, not through the removal lists, so it does not bring back anything that was removed. After a kernel change, check the ban again: items 3 and 7 of Step 12.

**What I did not check.** I ran this step's commands only against sample `pm` and `dumpsys` output; they were not run on the Razr Fold itself. The Bayton data comes from other models. The INTERNET permission shows a possibility, not the fact of sending: only a network monitor or a firewall with a connection log can prove that a package sends nothing or catch the sending, and that is not part of this guide.

## Rollback: if something broke

Any hiding is undone with a single command, as long as the phone boots and the PC can see it. All the commands in this guide that remove and bring back packages contain `--user 0` or `--user $GU`. In Step 11 the work profile number stands in place of zero, which is also fine. Do not remove this flag from any such command: without it, pm uninstall affects all profiles at once.

**Bring back a single package (on the PC)**

```
adb shell cmd package install-existing --user 0 com.example.package
```

Substitute the name of the package you need for `com.example.package`. The answer `Package ... installed for user: 0` means the package is back. If the app did not appear, wait 20 seconds and reboot the phone with the command `adb shell svc power reboot`.

**Bring back a whole list (on the PC)**

```
./debloat.sh restore google.txt
./debloat.sh restore moto.txt
./debloat.sh restore moto-extra.txt
./debloat.sh restore moto-ai.txt
./debloat.sh restore google-roles.txt
./debloat.sh restore gms.txt
./debloat.sh restore google-more.txt
./debloat.sh restore moto-more.txt
./debloat.sh restore carriers.txt
```

Run only the list you need. After bringing the Google services back into the main profile, wait 20 seconds, reboot the phone with the command `adb shell svc power reboot` and sign in to your account again: Settings → Passwords & accounts.

Packages that are not in the lists are brought back one at a time with the command from the “Bring back a single package” section: these are the dialer (Step 6) and `com.motorola.stylus` from Step 5. The package `com.motorola.ccc.notification` is now in `moto.txt`, so `./debloat.sh restore moto.txt` brings it back too; the script skips the other `ccc.*` packages on restore as well. The lists from Step 5a (`google-more.txt`, `moto-more.txt`, `carriers.txt`) are brought back with the same commands from the “Bring back a whole list” block. They do not affect the lines that start with `#`: those packages were not removed in the first place. The internet block (Steps 12 and 12a) is removed separately: the command is in the last item of Step 12; for another package, substitute its name for `com.motorola.camera5`.

**Bring back only Gboard (on the PC)**

```
adb shell cmd package install-existing --user 0 com.google.android.inputmethod.latin
sleep 20
adb shell svc power reboot
```

If you also want to remove the protection module from Step 6, first bring Gboard back with this command, then remove the module: APatch → APM → Remove, and reboot the phone. I did not check what the system does with an already removed package when its protection comes back, so this is the order. It is better to leave the metamodule in place: without it, the files of the other modules will stop being attached too.

**See what is hidden now (on the PC)**

```
adb shell pm list packages -s -u --user 0 | sort > all_system.txt
adb shell pm list packages -s --user 0 | sort > visible_system.txt
comm -23 all_system.txt visible_system.txt
```

The last command prints the system packages that are hidden in the main profile.

**If the phone does not boot**

This should be rare: the packages in the lists were taken from public “safe to remove” lists. The risk is not zero, so act in order.

1. Wait ten minutes. The first boot after removal can be long.
2. If the problem began right after flashing a kernel, not after removing apps, bring back the stock boot. The phone must be put into fastboot mode (turn it off and hold the power and volume “down” buttons), and the command is entered on the PC: `fastboot flash boot boot_stock.img`.
3. If the phone reboots in a loop after removing packages, run `adb devices` on the PC. If the phone is visible, bring back the package you removed last with the command from the “Bring back a single package” section. In a boot loop the system may not respond; then go to item 4.
4. If adb has a root shell through APatch, reset the whole list of hidden packages with one command on the PC and reboot the phone: `adb shell su -c 'rm /data/system/users/0/package-restrictions.xml'`. Without this file the system considers all system packages installed, and all hidden apps will come back. The “open by default” choices will be reset along with it. This is based on my knowledge of how Android reads this file. I did not check it on your phone.
5. The last resort: a data reset through recovery mode. All data on the phone will be lost, so the backups from the “Before you start” section are needed in advance. After the reset all built-in apps will come back, and you can start over.

**Remove the “Google” profile completely (on the PC)**

```
GU=$(adb shell pm list users | grep -o '{[0-9]*:Google:' | tail -n 1 | tr -dc '0-9')
echo Removing profile number: $GU
[ -n "$GU" ] && adb shell pm remove-user $GU
```

This is irreversible: all the profile's data, including the account sign-in, will be lost. Do not run the command if you still need the profile. Only the profile named “Google” is removed; the others are not affected.
