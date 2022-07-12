---
title: "Save Battery and Processing by Disabling Autoplay ads/videos"
date: "2021-08-03"
categories: 
  - "tech"
tags: 
  - "adblock"
  - "chrome"
  - "firefox"
  - "safari"
  - "save-battery"
image: "/assets/blog/59f21-image.png"
---

While using the M1 MacBook Air for a few weeks, and recently using my 2015 15" mbp, I've had a few incidents of LFN, Loud Fan Noise. Something that never happened on the Air, since the M1 version didn't even include a fan. So, aside from downloading [Mac Fan Control](https://crystalidea.com/macs-fan-control) and looking at [Activity Monitor,](https://support.apple.com/guide/activity-monitor/welcome/mac) I've looked for other ways to "lessen the load."

First, Safari is much more efficient than Chrome, so use that more often. Second, as most of the web is "free" we are constantly bombarded by side bar or random ads on webpages. Not only do they take screen real estate, but they also take memory and cpu processing, which can trigger LFN.

## Safari

So, lets take care of that. Apple [outlines](https://support.apple.com/guide/safari/stop-autoplay-videos-ibrw29c6ecf8/mac) a procedure to block video for all websites. Basically:  
1) Open Safari->Preferences->Websites->Auto-Play, in the bottom right there is a default setting, change it to "Never Auto-Play".

![](images/b2965-screen-shot-2021-08-03-at-2.21.37-pm-1024x750-1.png)

## Chrome

Chrome on the other hand has no easy way to disable autoplay videos, only to mute the sound which breaks functionality on all sites, including YouTube and Twitch. So for Chrome, you'll need to use an extension, [Adblock](https://getadblock.com/en/). There are a few, but they are the largest and have been around the longest.

![](images/9c455-screen-shot-2021-08-03-at-2.46.59-pm-1024x732-1.png)

## Firefox

Firefox has a setting to disable autoplay.  
Firefox->Preferences->Privacy & Security->Autoplay->Settings

![](images/aaef9-screen-shot-2021-08-03-at-2.48.48-pm-1024x600-1.png)

In the popup, change the "Default for all websites:" to Block Audio and Video, then save.
