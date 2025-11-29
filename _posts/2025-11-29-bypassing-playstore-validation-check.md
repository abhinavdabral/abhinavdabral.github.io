---
layout: post
title: Bypassing check for Google Play Store as app's installation source (without root on GrapheneOS)
---

This is likely a short-term solution, which helps in bypassing the validation where apps check if they were installed from Google Play Store. I've been using this workaround, for nearly an year now and so far it has been working fine for me.

_Last updated: 2024-11-29_

# What’s happening?

Some financial apps now check whether they were installed from the Google Play Store.  
They usually do this by:

- Checking the package name (`com.android.vending`), or
- Verifying the installer’s signature.

Spoofing the package name is easy, but most apps also check the signature, which makes the problem harder.

## How to bypass this, without root

As of writing this post, I am using GrapheneOS (build 2025112100) on a Pixel. I don't have any Google accounts logged into the Sandbox Play Store Services. That also means that I can't use Google Play Store.  
Instead, I use Aurora Store to get the apps I need. It also allows me to export their bundles, which is useful for the process described below.

### Pre-requisites

- A computer
- ADB setup to access your device
- JRE or JDK is required for ApkRenamer

### 1. Spoofing the Google Play Store

1. Download SAI APK from [github.com/Aefyr/SAI](https://github.com/Aefyr/SAI).
2. Download ApkRenamer from [github.com/dvaoru/ApkRenamer](https://github.com/dvaoru/ApkRenamer)
3. Follow the instructions from ApkRenamer and rename the SAI APK. Change the following:
   - Package name: **`com.android.vending`**
   - (Optional) App name: **Google Play Store**
4. Once you have the APK, transfer it to your phone. I prefer to keep it in `Download` folder.

### 2. Install the spoofed Play Store

_GrapheneOS specific:_

1. Uninstall the real Play Store by opening the pre‑installed **App Store** app.
2. Enable Developer Options → USB Debugging.
3. Verify connection: `adb devices`.
4. Open a shell and run:

   ```sh
   # copy the APK to a temporary location
   cp /sdcard/Download/vending.apk /data/local/tmp/vending.apk

   # install it
   pm install -t -r /data/local/tmp/vending.apk
   ```

### 3. Exporting and re-installing apps

1. In Aurora Store, open **My apps & games** (from gear ⚙️ menu on top-right).
2. Long‑press any app that enforces Google Play Store validation and choose **Save app bundle**.
3. In the renamed SAI, queue up all the exported bundles and begin installing them.
4. Do **not** open any of these apps yet.

### 4. Wrapping up

1. Uninstall the spoofed Google Play Store (SAI).
2. Re‑install the real Play Store from GrapheneOS' **App Store**.
3. Disable Developer Options (It'll prompt you to restart).
4. After restart, you can launch the apps.

## Does it work?

For most banking/financial apps that I was previously struggling with, this workaround works. It doesn’t work for only 2 of my apps, so I currently rely on an older version of them, until they inevitably will stop working. That's a problem for future-me.

## What about updates?

For security updates, you'll likely have to follow the process above all over again to make them work.

Rarely some apps may get updated where warn users about validation failure but still allows users to skip and continue using the app.

I usually try to update my financial apps over weekends where I can perform this procedure. Normally it takes no more than 5mins, but there's always a risk where certain apps will stop working after updates. In such a case, the short term solution is to downgrade them back to older version, which can take some extra time.

## Alternatives / Other Approaches

### A. Install via ADB with installer parameter

The installer flag of package manager utility (within ADB Shell), allows passing the package name for the installer. We can use it, something like this,

```sh
adb shell pm install -i "com.android.vending" -r /data/local/tmp/app.apk
```

This tricks the installer name but didn't help me to run any of the apps that I was struggling with.

### B. Root‑based methods

These require a rooted device; I’ve not tested them.

- **AppManager** – [github.com/MuntashirAkon/AppManager](https://github.com/MuntashirAkon/AppManager)  
  (offers installer‑change utilities in root mode).
- **BetterKnownInstalled** – [github.com/Pixel-Props/BetterKnownInstalled](https://github.com/Pixel-Props/BetterKnownInstalled)  
  (Magisk/KernelSU module that patches `packages.xml`).

# Final thoughts

Without decentralization this situation is only going to get worse. Google's [recent announcement](https://arstechnica.com/gadgets/2025/08/google-will-block-sideloading-of-unverified-android-apps-starting-next-year/) to block sideloading apps that were not developed by "verified" developers was concerning. But in a follow-up announcement they seemed to have [eased it a little](https://www.androidauthority.com/android-power-users-install-unverified-apps-3615310/). It doesn't make the situation any better. It's still more restricting being added in the name of user's safety.
