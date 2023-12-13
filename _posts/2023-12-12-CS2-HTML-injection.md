---
layout: post
title: CS2 HTML Injection
date: 2023-12-12
---

---

## **Counter-Strike 2**

> **Counter-Strike 2** (CS2) is the most recent installment of the franchise Counter-Strike, which is a competative 5v5 based first person shooter.
> CS2 was released on September 27, 2023. But the franchise as been around for almost two decades with its most noteable games such as **Counter-Strike: Global Offensive** and **Counter-Strike: Source**.

## **Source Engine / Counter-Strike vuln history**

> Although Counter-Strike has been around for almost two decades they still have some bugs that come up in their games, as with any video game company. 
> Sometimes when these vulns turn up in CSGO it is an underlying issue with the **Source Game Engine** developed by **Valve** not to be confused with **Counter-Strike: Source**.
> Most if not all games from the Counter-Strike franchise run on the Source Game Engine, making it easy to spot vulns in games that run on the same game engine.
> some noteable ones can be found below.

+ [secretclubs's Source Engine RCE via server join](https://secret.club/2021/05/13/source-engine-rce-join.html) 
+ [bienpnn's Source Engine reboot RCE](https://twitter.com/bienpnn/status/1381616325391384577)
+ [teapotddd's CSGO RCE via server join](https://twitter.com/teapotddd/status/1383527700707479554)
+ [floesen's Source Engine RCE via game invite](https://secret.club/2021/04/20/source-engine-rce-invite.html)

## **The fish to fry**

**[vallu's big tweet](https://twitter.com/valluXD/status/1734196959504670937)**

> On December 11, 2023 it was made [public](https://www.unknowncheats.me/forum/counter-strike-2-a/614543-cross-site-scripting-xss-cs2.html) that a person could change their username on **Steam** a **video game digital distribution service and storefront developed by Valve Corporation**
> To any HTML under 32 characters long (Steam max username length).
> With the name change if a user in a CS2 match is on the same team as the person who edited their name to some HTML code usually similar to ```<img src="https://127.0.0.1/x">```
> When the user that edited their name name gets vote kicked everyone on the team will request that image from the attacker specified host in the img tag of their HTML name, which in turn
> Gives the attacker / the person who edited their name all of the IP addresses of their team.

## **My thoughts**

> Right now the most someone could do with this is log the IP’s of their teammates in CS2, things like getting IP’s of other players in video games is nothing new. People have done it through VoIP on Call of duty. They could mute everyone except their target and look at the traffic, muted players won’t generate VoIP traffic.

**Ways to protect yourself**

+ **Play with a VPN ProtonVPN, Mullvad, NordVPN and set a server near to you:** When your game requests the image from the attacker's server they will see your VPN IP.

+ **Always have a full team:** This will ensure that there are no random players trying to grab your IP unless one of your friends decides to have some fun.
