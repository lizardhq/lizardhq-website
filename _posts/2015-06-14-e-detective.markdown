---
layout: post
title: "Security Advisory: Multiple Critical Vulnerabilities in E-Detective Lawful Interception System"
date: 2015-06-14
categories:
---

*By Mustafa Al-Bassam and slipstream/RoL.*

# Overview

From the [vendor's website](http://www.edecision4u.com/E-DETECTIVE.html):
>E-Detective is a real-time Internet interception, monitoring and forensics system that captures, decodes, and reconstructs various types of Internet traffic. It is commonly used for organization Internet behavioral monitoring, auditing, record keeping, forensics analysis, and investigation, as well as, legal and lawful interception for lawful enforcement agencies such as Police Intelligence, Military Intelligence, Cyber Security Departments, National Security Agencies, Criminal Investigation Agencies, Counter Terrorism Agencies etc. It also can provide a compliance solution for many standards or acts like Sarbanes Oxley Act (SOX), HIPAA, GLBA, SEC, NASD, E-Discovery and many others.

Decision Group also [claims](http://www.edecision4u.com/HTTPS-SSL.html) that its products are used by "more than 100 law enforcement agencies", including the National Security Bureau of the Republic of China.

E-Detective suffers from multiple critical security vulnerabilities, detailed in this advisory.

# Issues

E-Detective has a web management interface for viewing intercepted communications and managing the system, which can be exploited in multiple ways.

## 1. Unauthenticated Local File Disclosure

The `common/download.php` script in the web root allows for unauthenticated users to read arbitrary files on the system. This may include database credentials and captured data intercepts.

The `file` URL query parameter of the script is "protected" by the following encoding which is trivially reversible: base64 followed by rot40.

A [proof of concept for this vulnerability is available here](https://github.com/lizardhq/exploits/blob/master/e-detective/pwned-detective.py).

## 2. Authenticated Remote Code Execution

Combining this vulnerability with the Local File Disclosure vulnerability above can result in unauthenticated remote code execution.

The restore feature on the "config backup" page extracts a .tar file encrypted with Blowfish using OpenSSL into the system's root directory (`/`) as root.

The .tar file must be encrypted with the static key `/tmp/.charlie`. Yes, that's the actual key - the software passes the wrong argument to OpenSSL. `-K` is used to pass the keyfile instead of `-kfile`, meaning that the key is the path of the keyfile rather than the contents of the keyfile.

This allows an attacker to upload a shell into the web root, or overwrite any sensitive system files such as `/etc/shadow`.

# Solution

Uninstallation of E-Detective will prevent exploitation of these issues. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.
