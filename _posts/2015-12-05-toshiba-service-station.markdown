---
layout: post
title: "Security Advisory: Toshiba Service Station Unauthorized Registry Read"
date: 2015-12-05
categories:
---

*By slipstream/RoL.*

# Overview

Toshiba Service Station "allows your computer to automatically search for TOSHIBA software updates or other alerts from Toshiba that are specific to your computer system and its programs".

An issue in Toshiba Service Station, versions 2.6.14 and below, can be exploited to read parts of the registry as SYSTEM by local users of lower privilege.

# Issues

Toshiba Service Station installs a service named `TMachInfo` that runs as SYSTEM and starts an XML-based API on localhost via UDP port 1233.

One of the methods exposed by this API is `Reg.Read` which reads the provided registry value as SYSTEM, casts the result to a string, and returns the result.

Unfortunately, due to .NET casting, `REG_BINARY` values (like values in SAM, or the bootkey) cannot be read via this method, as casting a `byte[]` to `string` in .NET returns the string `"System.Byte[]"`.

However, this method could be used to bypass any read-deny permissions on the registry for lower-privileged users.

A PoC is available as `loadofoldtosh.d` in [this trio of OEM exploit PoCs.](http://rol.im/oemdrop/)

# Affected Versions

Version 2.6.14 and below.

# Solution

Uninstallation of this software will prevent exploitation of the issue. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.