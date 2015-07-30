---
layout: post
title: "Cards Against Security - Persistent XSS and Other Issues"
date: 2015-07-30
categories:
---

*By slipstream/RoL, IceMans- and David Davidson.*

# Overview
Earlier in this week, [Fortinet][fortinet] launched the [Cards Against Security][cardsagainstsecurity] online game, a [Cards Against Humanity][cardsagainsthumanity] clone with an Infosec twist. This kind of thing usually tends to happen around the "conference season". As it happened, this application suffered a number of rather severe security vulnerabilities which we were able to exploit for great justice. The application was taken offline following exploitation.

# Issues

## 1. Reflected XSS
The /?g= endpoint was vulnerable to reflective XSS:
{% highlight bash %}
http://www.cardsagainstsecurity.com/?g=<script>alert(1);</script>
{% endhighlight %}

## 2. Persistent XSS
The http://casbeta.herokuapp.com/process.php endpoint had several AJAX requests. The "spectategame" action could be passed a script tag in the "nickname" parameter, and this would execute the XSS on everybody playing or spectating in a random public game, if they viewed the chat or player list (which in most cases was automatic).

PoC of the request (as would be done using cURL as opposed to as a dump of a POST request...): 
{% highlight bash %}
curl --data "action=spectategame&user=1337&nickname=<script>alert('fix me');</script>" http://casbeta.herokuapp.com/process.php 
{% endhighlight %}

This XSS was used to redirect anyone playing or watching a public game to a YouTube video of Rick Astley - Never Gonna Give You Up.

## 3. SQL Injection
Another of the AJAX requests was the "getblackcard" action, which was vulnerable to SQL injection in the "game" parameter.

PoC  (as would be done using cURL as opposed to as a dump of a POST request...): 
{% highlight bash %}
curl -i --data "action=getblackcard&game=1 union select 1,2,3,@@version-- -" "http://casbeta.herokuapp.com/process.php"
{% endhighlight %}

# Video of Persistent XSS as seen ITW
<iframe width="560" height="315" src="https://www.youtube.com/embed/yhBqfaHTLPE" frameborder="0" allowfullscreen></iframe>

# Conclusion
Had one had malicious intent, one could have easily injected an iFrame that did things like, say, redirect users silently to browser exploits, and conducted an unauthorized penetration test of the userbase of this application (which of you have REALLY patched/removed Flash then?).  
Given the users were all infosec people, this was potentially an excellent watering-hole attack vector specifically for targetting the security industry. While Fortinet have implied that the vulnerabilities were deliberate, this is a wholly irresponsible approach for a vendor to take as it implies that they don't care that they put a bunch of their industry peers at risk.  
Our advice to people is quite simple: Don't blindly go clicking on, and trusting, random webapps that crop up for novelty purposes. Furthermore, we intend on releasing a not-vulnerable version of the game with an expanded card-deck shortly for you all to enjoy, based on information gathered during this.

As a final note, [Fortinet have some SSL issues to sort out][fortissl] <3

[fortinet]: http://www.fortinet.com/
[cardsagainstsecurity]: http://www.cardsagainstsecurity.com
[cardsagainsthumanity]: https://cardsagainsthumanity.com/
[fortissl]: https://www.ssllabs.com/ssltest/analyze.html?d=fortinet.com
