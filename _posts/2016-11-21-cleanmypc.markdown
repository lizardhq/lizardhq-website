---
layout: post
title: "Security Advisory: File Read/Write as SYSTEM Vulnerability in MacPaw CleanMyPC"
date: 2016-11-21
categories:
---

*By slipstream/RoL.*

# Overview

CleanMyPC "is more than a PC cleaner — it’s an essential tool that cares for your computer".

An issue in CleanMyPC, versions 1.8.1.570 and below, can be exploited by a local attacker to read from and write to any file on the system with SYSTEM privileges. This can lead to local privilege escalation.

# Issues

CleanMyPC bundles a service, named `CleanMyPCService` which exposes various WCF services as named pipes.

One of the services exposed is `CmpDataFilesService` which exposes two methods: `byte[] GetCleanMyPCDataFileContent(string fileName)` which returns the content of `fileName`, read as SYSTEM, and `void SaveCleanMyPCDataFileContent(string fileName, byte[] content)` which writes `content` to `fileName`, again as SYSTEM.

Both methods try to work with a specific relative path; `GetCleanMyPCDataFileContent` if the file exists in that relative path (but this is always false if you pass an absolute path); and `SaveCleanMyPCDataFileContent` if the write failed with the path that was passed to it.

A PoC is [available](/files/climbmypc.zip): `climbmypc.cs` attempts to use this issue to copy between two passed file locations as SYSTEM.

# Affected Versions

Version 1.8.1.570 and below.

# Solution

Uninstallation of this software will prevent exploitation of the issue. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.

# Greetings

Thanks to Techdirt for [their accidental promotion of CleanMyPC](https://www.techdirt.com/articles/20161118/09225436083/daily-deal-cleanmypc-single-license-sorry-about-that.shtml). Without it, I probably wouldn't have found the software to audit it.
