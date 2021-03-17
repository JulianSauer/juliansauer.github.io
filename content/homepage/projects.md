---
title: "Projects"
weight: 2
header_menu: true
---

Here are a few projects I created for fun.

### [FritzPowerMonitor](https://github.com/JulianSauer/FritzPowerMonitor)
Background: My oven has a timer which turns the oven off after a certain amount of minutes. However it doesn't ring because it's broken. While setting a timer on my smartphone or checking the oven regularly would be an option I am currently using a [smart plug](https://avm.de/produkte/fritzdect/fritzdect-200/). That way I am able to poll the power consumption and send myself a message in Telegram when the oven turns itself off. I noticed that it takes about half a minute before the smart plug actually realizes it but that should be within reason for most use cases (dishwashers, ovens, coffee machines).

I placed a button on the home screen of my smartphone using an app called [HTTP Shortcuts](https://http-shortcuts.rmy.ch/). Now my current workflow is
1. Throw a frozen pizza in the oven and set it's timer to 15 minutes
2. Start the polling with a default of 25 minutes using the button on my home screen
3. Do something else
4. Telegram: "Your pizza is ready to be eaten"
5. Enjoy an unhealthy meal

### [RemoteController](https://github.com/JulianSauer/RemoteController)
There isn't really a backstory. I bought a few smart plugs with a remote as well as [this sensor](https://www.tinkerforge.com/en/shop/remote-switch-v2-bricklet.html) to be able to send and receive their 433MHz signals. Forwarding this to a Raspberry Pi I was able to send rest calls by pressing a button on the remote which could control other smart home devices like the FritzPowerMonitor from above or Philips Hue light bulbs. 

However the radio part isn't very secure since it's not encrypted. That's why I barely use it today. The only regular usage of it is the dehumidifier that I turn on after showering. It get's turned off again after a few hours by this service. However migrating this to the FritzPowerMonitor or having a bathroom with a window would probably be more sustainable.

### [envsubst](https://github.com/JulianSauer/envsubst)
Background: I have a server at home which runs Alpine with no additional software installed besides Docker and docker-compose (because why not). It basically allows me to deploy software quick and dirty while having everything tracked by Git. For example the server currently runs containers with Teamspeak, some IoT stuff as well as a tiny bit of data mining software and a private cloud. Since the related Docker files are [public](https://github.com/JulianSauer/Docker-Server) I moved passwords and secrets to a separate file.

This project is essentially a script that uses this file as well as the public placeholder files to generate the working configurations for the containers in directories not tracked by Git. Oh and of course it can be run using only Docker.
