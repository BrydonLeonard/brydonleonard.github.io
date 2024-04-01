---
layout: post
title:  "GateKey - The notifier"
date:   2023-12-31
categories: automation
tags: automation gatekey
---

# The GateKey notifier

In a [previous blog post](https://brydonleonard.github.io/automation/2023/09/22/gatekey.html), I discussed the system I developed to manage keycodes for the gate to my complex. The system works great, but I'm not always with my phone, so I sometimes miss the notifications that it sends when the gate opens. I built a little box called the GateKey _notifier_ that solves the problem by playing sounds. Here's a video of it in action:

<iframe width="560" height="315" src="https://www.youtube.com/embed/vIo5TjSl2mE?si=YiEjn-ln5pkwTV34" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>

Because the rest of the GateKey system is designed to be used by multiple households in a complex, I wanted all of them to be able to use notifiers too. As such, it was important to give the same consideration to security and privacy as in the rest GateKey. It would be an invasion of privacy if, say, the notifier in house A played sounds when house B had a visitor, or the notifier API allowed someone to get information about everyone entering the complex.

## The notifier itself

Theoretically, the notifier could be anything that is able to use the notifier API (discussed below), but mine is an ESP32 dev board with an RGB LED and speaker inside an ABS enclosure. If you want to build one exactly like mine, here's the parts list:

If you want a list of parts to build something similar, I used
- [An ESP32 dev board](https://www.communica.co.za/products/hkd-esp-32-wifi-b-t-dev-board)
- [This plastic enclosure](https://www.communica.co.za/products/abse12-black?variant=17184475316297)
- [An individual RGB LED](https://www.communica.co.za/products/bmt-rgb-led-4p-5mm-c-cath-10-pk)
- [A 1W 8Ω speaker](https://www.communica.co.za/products/spkr-0-86in-1w-8ohm-22x3mm)
- [A micro USB cable](https://www.communica.co.za/products/usb-cable-1-2m-am-micro?variant=46670347632940)

And [here's the GitHub repo with all the code](https://github.com/BrydonLeonard/GateKeyNotifier) (MIT licenced, so go wild). I offloaded all of the complexity to software here, so the wiring is straightforward too (the diagram below ignores the _rest_ of the pins on the ESP32 that I didn't use):

![ESP32 wiring diagram](/assets/images/gate-key-notifier/circuit.svg)

Using the same ground pin for the RGB LED and speaker led to some interference (flickering lights and staticky sound), so I split them over two and that resolved the issue.

## Notification

With a box, the next step was to somehow tell it that the gate has opened. MQTT (which used to stand for MQ Telemetry Transport, but as of 2013 [doesn't stand for anything](https://www.hivemq.com/blog/mqtt-essentials-part-1-introducing-mqtt/) 🤷) is a pub-sub messaging protocol that's really popular in the IoT world and fit my use-case nicely. MQTT separates messages based on their _topics_. Publishers write messages to particular topics and subscribers subscribe to those that they're interested in. In the case of GateKey, each household can have its own topic that only gets notifications that apply to it. The service in between the publisher and subscriber (which is also responsible for all of the logic around message delivery guarantees) is called the MQTT _broker_.

![Notification flow](/assets/images/gate-key-notifier/mqtt.svg)

Working with MQTT in the Arduino ecosystem is made easy by the [PubSubClient library](https://www.arduino.cc/reference/en/libraries/pubsubclient/), which implenents an MQTT client. For now, I've hardcoded the box's WiFi creds; when it starts up, it immediately connects to the WiFi. Once that's done, the clint [subscribes to the MQTT topic](https://github.com/BrydonLeonard/GateKeyNotifier/blob/mainline/src/main.cpp#L209) and waits for something to happen. When something _does_ happen, [it makes exciting noises](https://github.com/BrydonLeonard/GateKeyNotifier/blob/mainline/lib/Notification/notification.cpp#L107).

![Notification flow](/assets/images/gate-key-notifier/notification-flow.svg)

On the GateKey side, I [extracted an interface from the notifier](https://github.com/BrydonLeonard/GateKey/commit/952aff5b8dccbf80993cf51191d4e741df1b7701) that already existed to send Telegram. The [NotificationSender](https://github.com/BrydonLeonard/GateKey/blob/mainline/src/main/kotlin/com/brydonleonard/gatekey/notification/NotificationSender.kt) receives a list of notifiers (today [Telegram](https://github.com/BrydonLeonard/GateKey/blob/mainline/src/main/kotlin/com/brydonleonard/gatekey/notification/TelegramNotifier.kt) and [MQTT](https://github.com/BrydonLeonard/GateKey/blob/mainline/src/main/kotlin/com/brydonleonard/gatekey/notification/MqttNotifier.kt)) and watches a queue for messages indicating that the gate has opened. When it sees one, it sends a notification via each notifier. Those "gate opened" messages are written by the same component that communicates with the gate - the [VoiceController](https://github.com/BrydonLeonard/GateKey/blob/mainline/src/main/kotlin/com/brydonleonard/gatekey/VoiceController.kt#L82).

![Notification flow](/assets/images/gate-key-notifier/notification-sender.svg)

If we didn't care about security or privacy, that would be it. The problem is that without authentication and authorization, anyone can get any notifications and without encryption, any attempts at auth are wasted because creds for auth are being sent as plaintext. To resolve the issue, I implemented a registration handshake (which allocates creds to the notifiers) and added SSL to the MQTT broker.

## Registration

Notifiers need credentials to subscribe to their hosueholds' MQTT topics. Flashing the notifier with the credentials would work, but create long-lived credentials. Long-lived creds are bad because they're hard to renew; if they're leaked, a malicious actor could gain access to the system for long periods of time because there's no easy way to invalidate those leaked creds. I wanted notifiers to be allocated easily-refreshable credentials on startup. Notifier registration is handled by the [`MqttDeviceRegisterer``](https://github.com/BrydonLeonard/GateKey/blob/mainline/src/main/kotlin/com/brydonleonard/gatekey/mqtt/MqttDeviceRegisterer.kt) and, at a high level, looks something like:

1. Each notifier is allocated (and flashed with) a unique ID. If GateKey is ever used widely enough that I can't speak to the notifiers' owners directly, those IDs can be encoded in a QR code on the notifiers' cases. 
2. When the notifier connects to the internet for the first time, it makes a `POST` request to the GateKey server at `/register_device` that includes the notifier ID (step 1):
3. The server stores the notifier ID and returns a `ACCEPTED` response (steps 2 - 4) # TODO check this 
4. The notifier continues polling with its ID and awaits a non-`ACCEPTED` response. (steps 1 - 4)
5. A user in the household that's adding the notifier clicks the `addDevice` button in Telegram and enters the ID of the notifier (step 5).
6. The notifier is registered (step 6 - 7)
7. The next poll from the notifier receives a `REGISTERED` response(steps 9 - 12), which includes
    - A set of credentials for the MQTT topic
    - The name of the MQTT topic to which the notifier should subscribe

![Notification flow](/assets/images/gate-key-notifier/registration.png)
8. The notifier uses the credentials to initialize its MQTT client (step 13).

> If the user requests the notifier before the first poll, the first response would be `REGISTERED` and the notifier would never need to poll at all.

### The notifier state machine

I didn't want the registration code on the notifier's side to be blocking or asynchronous. As such, the notifier runs a small state machine to handle the other side of the service above. If you're not familiar with Arduino development, the framework exposes two hooks into the device lifecycle:
- `setup()`, which is called once on startup
- `loop()`, which is called repeatedly as long as the device is running

Within `loop`, the notifier maintains two state machines; one for the WiFi connection and another for the connection to the MQTT broker. The WiFi state machine takes precedence; while the notifier has no internet connection, it has no reason to try connect to the broker, so the second state machine doesn't run at all. Once it is connected, the notifier registers (if necessary) and subscribes to the MQTT topic. Only notifiers in the MqttConnected state are actually watching for notifications.

> You can find a pretty ASCII art version of this diagram [here](https://github.com/BrydonLeonard/GateKeyNotifier/blob/mainline/src/main.cpp#L228-L265) if you're so inclined.

![Notification flow](/assets/images/gate-key-notifier/registration-state-machine.png)

## Secure connections

Before this project, I was unaware that Arduinos aren't flashed with a set of root CA certs. Practically, that means that (without some extra work) they're not able to use SSL. There are two types of connections from the notifier that needed encryption:
- The ones used to make HTTPS requests to the server for registration
- The ones used to connect to the MQTT broker

Initially, I'd planned to use the same cert for both, but that was complicated by the fact that I use [CloudFlare](https://www.cloudflare.com/) for DNS and proxy requests through their servers. When a caller does a DNS lookup for the GateKey server, instead of getting the server's IP address back, they get the IP address of a CloudFlare proxy server. Doing so creates a layer of separation between the caller and my server; if someone decides to launch a DDOS attack, I can block all requests at the CloudFlare proxy server

![CloudFlare proxy diagram](/assets/images/gate-key-notifier/cloudflare.png)

Unfortunately, CloudFlare only supports proxying for port 80 and 443, so secure MQTT traffic over port 8883 has to go straight to the server. I ended up creating another entry in my DNS record for MQTT that points straight to my server. That isn't _great_ because it exposes my server's IP address, but at least I've got protection for _some_ traffic.

My next issue was that while CloudFlare offers free certs for all proxied traffic, those certs [are _only_ valid for the requests from the CloudFlare proxy to my server](https://developers.cloudflare.com/ssl/origin-configuration/origin-ca/); they can't be used for requests directly to the server (like my MQTT connections). One option would be to disable CloudFlare proxying and create a single cert, but I opted to instead continue proxying HTTP traffic via CloudFlare and create a new cert (via [Let's Encrypt](https://letsencrypt.org/)) for MQTT traffic. 

### Generating Certs

> I _could_ have used a self-signed cert for MQTT traffic, given that I own all of the clients that connect to the MQTT broker, but I hadn't used Let's Encrypt before and wanted to try it out. 

[CertBot](https://certbot.eff.org/pages/about) is a tool that, when run, spins up a process that listens on port 80 and kicks off a cert provisioning request for whatever domain you specify. You run it on a server behind the DNS address for which you're requesting a cert and Let's Encrypt's makes a request against the domain to confirm that they're able to reach the CertBot process. Once that's done, the cert is provisioned and dumped out of the bot, ready for you to use.

## Conclusions

I've now got a little box that makes noises when people come to visit! There are two more changes that I'd like to implement before letting others use their own notifiers.

Firstly, the notifier plays sounds when it connects to the MQTT broker. That's useful, but sometimes it momentarily disconnects from the broker in the middle of the night and the sounds are loud enough to wake me up. I'd like to have a grace period where it doesn't give up on the connection to avoid the spurious wake-up calls.

Finally, all notifiers currently listen to their own MQTT topics, but do it with a single shared set of credentials. I'm running the [dynamic security plugin](https://mosquitto.org/documentation/dynamic-security/) on my MQTT broker, so the next step is to use that to generate new users for each household on the fly. Once that's done, the notifier system is fully secured and ready to be used by others in the complex.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fautomation%2F2023%2F09%2F22%2Fgatekey-notifier.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)
