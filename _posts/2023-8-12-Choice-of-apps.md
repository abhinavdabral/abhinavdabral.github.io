---
layout: post
title: Android Apps on my daily driver
---

The idea of "privacy" with Android apps comes with a fair share of compromises in most cases. So the chances that you are going to get the same (or better) experience than what you're current using is pretty slim (but it's still there). Personally, it's all worth it to me.

_Last updated: {{page.last_updated}}_

You can refer to my threat model on my other post. But specifically for apps, my definition of "privacy friendly" is:

- For FOSS Apps
  - Has a self-hosted server option available
  - At least gets security updates
  - Should be available on GitHub or F-Droid
- For Non-FOSS Apps
  - Works without internet
  - Doesn't require an online account or authentication to work (even if that's one-time)
  - Can be downloaded via Aurora Store anonymously
- Exceptions
  - Banking applications

# 1. Apps that I currently use

## 1.1 FOSS Apps

### 1.1.1 TrackerControl

This amazing project is my go-to solution for local DNS filtering. Personally, I have found this app as the most user-friendly solution to block apps from accessing stuff that you probably don't want it to access.

- Blocks ads/trackers at DNS level
- Blocks internet for apps (must-have for all stock ROMs)
- Gives you per-app control rather than just allowing or disallowing something globally.

## 1.2 Non-FOSS apps

Generally (that is, excluding the exceptions) for all Non-FOSS Apps, I turn off / deny them the `Network` permission so they're not able to use cellular data or wifi or anything. This allows us to use practically any application with bazillion trackers in them. They may be able to collect data from your usage/device but they won't be able to send them out until you allow them to use the network.

### 1.2.1 Google Camera

It's just the very best. There's really no comparison. With Google Pixel phones you don't even need GCam mod (unless there's a specific config that makes it even better). So, I personally just download this off-of Aurora Store.

### 1.2.2 Google Photos

Essential if you want to use official Google Camera app (or even most of the GCam Mod apps). Just disable the internet access and except "lens" everything else works. And it is a very nice Gallery app. Much better than current FOSS offerings.

### 1.2.3 Lawnchair 2

I recently switched to Lawnchair 2, from Lawnchair Alpha (which is available on GitHub). This version allows a bit more customization and allows categorization of apps within the app drawer. But other than that, it's still very lightweight and feels very close to stock launcher.

# 2. Apps that I have used in the past

## 2.1 FOSS Apps

### 2.1.1 Shelter

**Available on:** F-Droid

This is FOSS alternative of "Island" app on Google Play.

- It essentially helps you enable the "Work" profile on your phone. I
- With File Shuttle enabled, within settings, it allows you to share files between work profiles. That is, you're able to send someone a photo stored in work profile, using an app that's running on person profile (or vice-versa).

#### Why am I not using Shelter anymore?

In my other post [LINK] I mentioned that I was using a personal profile + work profile setup in the past to separate out the apps but now I've switched back to using just one profile. Therefore, I am not using this app in the current setup.

### 2.1.2 Lawnchair Alpha

**Available on**: IzzyOnDroid Repository / GitHub releases

It's actually called Lawnchair 12 Alpha 4 for some reason. But this is a very basic but capable launcher. It's very lightweight, supports icon packs and basic set of options. This is a very good option for people that do not require anything extra.

#### Why am I not using Lawnchair Alpha anymore?

Two reasons:

1. Lawnchair 2 has categories support, which allows us to arrange apps into categories within the App Drawer. Lawnchair Alpha didn't have that.
2. Lawnchair Alpha (on GitHub) has also not been updated since August, 2022.
