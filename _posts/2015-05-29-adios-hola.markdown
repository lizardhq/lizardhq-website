---
layout: post
title: "Security Advisory: Multiple Critical Vulnerabilities in Hola Overlay Network Client"
date: 2015-05-29
categories:
---

*By slipstream/RoL, Donncha O'Cearbhaill, joepie91 (Sven Slootweg), IceMans/RoL, infodox, pathfinder/braenaru, APT1337 and spoonzy, LeShadow.*

*A public information website is available at [adios-hola.org](http://adios-hola.org).*

# Overview

Hola 'Unblocker'/Hola 'Better Internet'/Hola 'VPN' is a commercial service offered on a 'freemium' basis that advertises itself variously as a censorship-evasion and privacy/anonymity-enhancement tool. The number of users is difficult to quantify objectively, but estimates range from 8 million users at the low end, to 42 million users as claimed on the Hola website.

The Hola Unblocker Windows client, Firefox addon, Chrome extension and Android application contain multiple vulnerabilities which allow a remote or local attacker to gain code execution and potentially escalate privileges on a user's system. Additional design flaws allow a Hola user to be tracked across the internet via a persistent ID. Furthermore, as Hola users - wittingly, or otherwise - act as exit-nodes for the overlay network, each is capable of acting as a Man-in-the-Middle for other users of the free or premium Hola network, or its commercial 'bandwidth' service, Luminati, and thereby compromising the privacy and anonymity of their browsing and exposing them to further attacks.

# Issues

The Hola software creates an API server listening on port 6853 of the loopback interface (127.0.0.1). The server exposes JSON and other endpoints over HTTP, setting an `Access-Control-Allow-Origin: *` CORS header, which allows a remote website to read the responses for requests to these endpoints, and thereby enumerate sensitive information, as well as affecting the security and integrity of the system.

## 1. Local File Read

The `/file_read.json` endpoint allows a remote website to read arbitrary files on the local system via path traversal. The files can be read up to the first null byte.

## 2. Information Disclosure

* The `/callback.json` endpoint provides uniquely identifying information about the users local Hola install. An attacker could exploit this issue to persistently track a Hola user across the Internet. When nodes are acting as exit nodes, they also obtain implicit unique identifiers for the traffic they relay, compounding the privacy concerns.

* The `/procinfo/ps` endpoint provides internal debugging information of the Hola software. With certain query arguments, the information provided contains pointers to functions that can be used to defeat Address Space Layout Randomization (ASLR). This would make it easier for an attacker to exploit memory corruption, such as a buffer overflow and other vulnerabilities, potentially allowing the attacker to achieve remote code execution in the context of the service in conjunction with other bugs.

As it happens, remote code execution is achieved without the use of the information leak.

## 3. Remote Code Execution

* The `/vlc_mv.json` endpoint allows files on the local system to be moved. The `/vlc_start.json` endpoint allows for the bundled VLC executable to be run with full controlled arguments. Together these vulnerabilities can be utilized to gain arbitrary remote code execution. Details will be fully disclosed at the discretion of the researchers when some time has elapsed for users to remove the software.

* If an attacker can perform a Man-in-the-Middle attack against a target running the Hola client on Windows - either as a network adversary, ISP, intelligence agency or another Hola client acting as an exit node -- they can create a connection seeming to originate from the hola.org or client.hola.org hosts to the local websocket port.

This web socket connection can be used to download a remote file and execute it with either the privileges of the logged in user, or as SYSTEM, depending on what version of the client is installed (browser extension or Windows client). The Android app is also affected by this vulnerability, which can be used to remotely execute arbitrary code as the Hola app user. An XSS vulnerability on the hola.org or client.hola.org domains could also allow an attacker to exploit this issue to remotely execute arbitrary code.

## 4. Privilege Escalation

The Hola software creates a websocket server listening on port 6863 of the loopback interface. The software checks the Origin header against some whitelisted values, but this can be spoofed if RCE has been achieved. The Windows client runs its service (`hola_svc.exe`) as SYSTEM. The websocket server allows access to commands implemented within the service such as `wget`, which downloads a file from a remote server, and `exec` which executes a file with the same privileges as the service. Therefore, the websocket can be abused to escalate privileges if the Windows client is installed.

# Affected Versions

Hola Engine for Windows and the Hola Firefox addon on Windows are vulnerable to the first described remote code execution vulnerability. Hola Engine for Windows is also vulnerable to the privilege escalation issue. These versions along with the "Chrome app on Windows" .crx extension (`hola_chrome_ext_dll_1.7.333.crx`) distributed on the Hola website and the Hola Android app are vulnerable to the second described remote code execution issue, the information disclosure and local file read issues.

# Solution

No solution yet exists. Uninstallation of the Hola service, Firefox addon and Android application will prevent exploitation of these issues. To fully uninstall the Firefox addon, ``%localappdata%\Hola` must also be deleted. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices. To fully uninstall Hola Engine for Windows, in some cases, deletion of `C:\Program Files\Hola` is also needed.
