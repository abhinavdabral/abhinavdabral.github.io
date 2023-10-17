---
layout: post
title: Android Apps on my daily driver
---

{% capture threat_model %}{% link _posts/2023-7-17-degoogled-android-setup-with-graphene-os.md %}#my-threat-model-in-short{% endcapture %}
The idea of "privacy" with Android apps comes with a fair share of compromises in most cases. So the chances that you are going to get the same (or better) experience than what you're current using is pretty slim (but it's still there). Personally, it's all worth it to me.

_Last updated: 2023-10-17_

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

### 1.1.2 Bitwarden

Bitwarden is a password manager (you can also stores notes, and other stuff as well).

#### Why I like it?

- Open source, mobile application as well as server
- There's a rust based server fork developed and maintained by the community
- Easy to self-host
- Comes with auto-fill extensions for browsers and phones (at least Android)
- Supports TOTP

And there's a lot more, like credential sharings, great password generator, etc. Never felt like it was lacking anything for me.

### 1.1.3 DuckDuckGo browser

As a search engine there's still mixed opinion in the community. As per my threat model I don't mind using DDG. DDG Browser, however, is my choice of browser for a very different reason.

#### Why I like it?

- Allows whitelisting certain websites, where you do not want the cookies to be deleted.
  - I haven't found any other browser that is able to do that.
- One click clear operation, which clears all caches/cookies and everything from all websites, except the ones that are whitelisted.

#### What I wish to be better?

- Ability to change search engine.
- In general just have more settings and configurations available that other browsers offer.

But overall, it's pretty good and I use it as my primary browser on my phone.


### 1.1.4 DAVx⁵

I use DAVx⁵ to sync my contacts and calendar from my self-hosted Nextcloud server. It works well and the integration works just as seamlessly as it did when I was using Google to do the same thing.

#### What I like about it?

- It's barebones. It's only made to sync WebDAV procols like CalDAV (for Calendar) and CardDAV (for contacts)
- Allows seamless login into Nextcloud account. No need to fiddle around with WebDAV URLs manually.
- Doesn't cause any unnecessary battery drain.

It's just a very straightforward app that does what it's supposed to do.

### 1.1.5 Etar

Etar is a calendar app. There was no highlighting reason for me to use this app, other than the fact that it's built well. It feels complete for what it's supposed to do and doesn't come with any unncessary features.

It also works well calendars added via DAVx⁵.

### FairEmail

I was using K-9 Mail until May, 2023 but then switched to FairEmail. FairEmail works well and comes with a lot of options to tweak your experience. The defaults are very privacy friendly and you can tweak that to adjust your experience to be more user-friendly, if you like.

#### What I like?

- It just works. The initial setup process does present you with a lot of options but they come prepopulated with defaults that work.
- Lots of customizations.

#### What I wish to be better?

- Some features are behind a paywall. Some of these features are too basic to be behind the paywall. However nothing critical is paywalled, so it's not too bad.
- Activation is not that straightforward, specially if you've installed it from F-Droid/Droify.
- UI is usable, but feels ancient.

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

In my other [post]({{threat_model}}) I mentioned that I was using a personal profile + work profile setup in the past to separate out the apps but now I've switched back to using just one profile. Therefore, I am not using this app in the current setup.

### 2.1.2 Lawnchair Alpha

**Available on**: IzzyOnDroid Repository / GitHub releases

It's actually called Lawnchair 12 Alpha 4 for some reason. But this is a very basic but capable launcher. It's very lightweight, supports icon packs and basic set of options. This is a very good option for people that do not require anything extra.

#### Why am I not using Lawnchair Alpha anymore?

Two reasons:

1. Lawnchair 2 has categories support, which allows us to arrange apps into categories within the App Drawer. Lawnchair Alpha didn't have that.
2. Lawnchair Alpha (on GitHub) has also not been updated since August, 2022.

### 2.1.3 K-9 Mail

K-9 Mail is actually pretty good and I prefer the UI of K-9 Mail over FairEmail. While FairEmail is like loaded with a whole bunch of useful features and customizations to fine tune your experience, K-9 Mail's philosogy is that it just works (kinda).

#### Why I'm not using K-9 Mail anymore?

Now this could be subjective and your mileage may vary. For me the main reasonw as that K-9 Mail would not load some of my mails, specially one with attachments. Also, sending mails with attachments was a hit-or-miss.

Now it could've been something related to Google's side of things, but then FairEmails works perfectly in that regard. So, I'm not sure what the reason was but it's absolutely unacceptable for me to use an app that I cannot rely on when expecting any sort of urgent/high priority emails, especially when I'm on the go. 