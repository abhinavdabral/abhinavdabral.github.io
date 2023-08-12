---
layout: post
title: How I use GrapheneOS on my Pixel 7
---

This is not a general guide for anyone in specific, really. I'm just using this as a public notebook of sharing my experience with it. What works, what doesn't and how to get around certain things. I'll probably just update this doc as I have new things to add, instead of creating new posts every time.

_Last updated: {{page.last_updated}}_

To set the context, let me give you a summary of my threat model, so it becomes relevant on why I'm doing things, the way I'm doing them.

## My threat model (in-short)

- Prevent as much unnecessary data-sharing as possible
- Prefer FOSS apps over closed-source apps with bunch of trackers in them.
- When using non-FOSS apps, restrict their data-sharing, fingerprinting and whatever else "extra" things they're doing, without breaking their functionality
- Self host as many services as manageable (excluding email services)

# Basic setup

## 1. Profiles

GrapheneOS offers an excellent layer to sandbox Google Play services. Which means that it behaves just like any other app and you gain the ability to control the permissions that it can have at any point of time.

Usually, this means that we keep this app without granting it any permission, specifically NOT the "Network" permission. So it is unable to connect to the mothership, unless we explicitly allow it to do so.

There are two basic setups that I've tried so far:

### 1.1. Personal profile (without GSF) & Work Profile (with GSF)

_(GSF = Google Services Framework / Google Play Services)_

The idea is to use FOSS apps and non-FOSS apps (that do no rely on GSF to work) in the personal profile. Rest goes into work profile.

#### 1.1.1 Benefits of this approach

- Two separate work-spaces mean the apps are much better isolated than having all of them in the same scope.
- Gives us the ability to force certain apps to run without GSF by installing them into the profile that doesn't have GSF in a sandbox (i.e. Personal profile, in this case).

#### 1.1.2 Problems with this approach

- Uses more system resources keeping double the apps.
- Each profile has to have their own specific local VPN proxy filtering setup. More on this later.
- Inability to access files in work-profile through USB connection.
- Setting up sync in both profiles or anything that needs to actively run in background just takes up more resources.
- Certain apps need to be in the main profile to function properly and some of the functionality is limited by installing them in the work-profile -- Like in some banking apps you lose the ability to do any NFC based payments
- Slight management overhead of keeping things updated in both profiles.

### 1.2. Just personal profile (with GSF)

The idea is to use app apps in the single workspace.

#### 1.2.1 Benefits of this approach

- Easy on system resources (storage & actively running background apps)
- Easier to manage -- do not need to update apps in both profile, separately.

#### 1.2.2 Disadvantages of this approach

- No profile-level segregation between apps.
- Cannot control which apps gets to use GSF and which don't. Since all of them are in the same profile, they all can access GSF, if they want to.

## 2. Apps

I've a separate post which I try to keep updated with everything important that I use.

# Troubleshooting

## 1. "Axis Mobile" app crashing after entering MPIN

### 1.1 When does Axis Mobile app crashes

- In the first launch, the app asks for your permission to send an SMS to get your account details.
- Once you're in the dashboard / home area of the app, the app crashes.
- This is the point where it has to go through the 2nd phase of verification and asks you for your Account number or Card details to verify your account. But it doesn't happen if your app crashes before that.

### 1.2 Confirm the reason behind the crash

The likely reason for crash could be that the app was not able to connect to Firebase Cloud Messaging service via GSF. But just to ensure that, we can let the app crash and open the crash logs to see the details.

If your app is crashing without a dialog, then try to interact with the app by random touches, just before it crashes. it seems to invoke that crash modal with "Show details" button.

Clicking on that will show the log and we can just see that the error states

```
... java.IO.Exception: SERVICE_NOT_AVAILABLE
OR
,,, java.IO.Exception: FIS_AUTH_ERROR
..
..
   at com.google.android.gms ...
```

All of this translates into the fact that Axis Mobile app is trying to utilize Google Services Framework (likely for FCM - Firebase Cloud Messaging) and the "failure" is not gracefully handled in the app, resulting in a crash.

### 1.3 Fixing the crash

- Just set "Network" permission of "Google Play services" to be "Allowed" for the initial login to succeed.
- I was using TrackerControl and I had to turn it off until I completed the second level verification. You might have to either allow or disable your VPN filter (if any) only until verification is done.

Once we do that and relaunch the Axis mobile app, it proceeds smoothly. After the initial login and verification is complete, it's safe to revoke the "Network" permission from the "Google Play services" app so that it can't continue to connect to internet again. And you can also proceed to enable your VPN filter back on again.
