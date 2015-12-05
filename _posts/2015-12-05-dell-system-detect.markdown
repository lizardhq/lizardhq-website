---
layout: post
title: "Security Advisory: Dell System Detect User Access Control (UAC) Bypass"
date: 2015-12-05
categories:
---

*By slipstream/RoL.*

# Overview

Dell System Detect (DSD) is "an application that runs on your Windows-based PC or tablet with your permission and interacts with the Dell Support website so that we can provide a better and more personalized support experience."

An issue in Dell System Detect, versions 6.12.0.1 and below, can be exploited to bypass UAC.

# Issues

Dell System Detect starts an HTTPd on localhost that listen on one of four ports: 8884, 8883, 8886 or 8885.

The APIs exposed by this HTTPd are protected by two methods:

1. Referer and origin header checks, referer or origin domain must be one of several Dell-owned domains.

2. Specific API methods can only be called with the correct signature, generated via an RSA-1024 key.

The first method can easily be bypassed by code running on the system (or a man-in-the-middle for that matter).

The second can also be easily bypassed, as specific pages on Dell's site can be scraped to gain valid signatures for some interesting APIs.

The Dell System Detect executable has a valid code signature; and on startup, if not started with specific arguments it tries to elevate itself.

Assuming the user denies the elevation. the `checkadminrights` API also tries to elevate. The API supposedly returns false if DSD is not elevated, but in practise, it returned true all the time.

This bug can be worked around by using the time that the request took to execute. With experimentation, it seems that if the API request had taken over 200ms, then a UAC request dialog had been spawned.

After the user has confirmed elevation, a wait must occur for DSD to restart its HTTPd. It seems a delay of 200ms every loop is adequate for this.

When DSD has been elevated, a second API, `installmanual`, can be used to run any file. The `\\` character is filtered out, but the `/` character can be used as a replacement.

A PoC is available as `dellsystempwned.d` in [this trio of OEM exploit PoCs.](http://rol.im/oemdrop/)

# Affected Versions

6.12.0.1 and below.

# Solution

Not even uninstallation of Dell System Detect will prevent exploitation of these issues; it runs from `%APPDATA%` so malware could easily drop it on your system to exploit this issue.

The checkadminrights method is hardcoded to execute `DellSystemDetect.exe` so, uninstallation of Dell System Detect and blacklisting of `DellSystemDetect.exe` from being executed will prevent exploitation of these issues.

The researchers cannot sanction any mitigations except to perform these actions.