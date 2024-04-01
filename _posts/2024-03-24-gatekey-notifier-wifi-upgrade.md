---
layout: post
title: "GateKey - A notifier upgrade"
date: 2024-03-24
categories: automation
tags: automation gatekey
---

In [The GateKey notifier](https://brydonleonard.github.io/automation/2023/12/31/gatekey-notifier.html), I wrote about the little box I built to play notification sounds when visitors enter our complex using keycodes. It worked well enough, but the WiFi and MQTT configuration had to be hardcoded and flashed to the ESP32, rather than being configured at runtime. This post is about how I fixed that.

## The high-level idea

Going into this project, I knew that ESP32s could use their 2.4GHz support to start up in access point (AP) mode, which allows them to create their own WiFi network. As such, my plan for WiFi configuration was:
1. Have the ESP32 start up in AP mode the first time it's powered on
2. Serve a web page from a REST endpoint to clients to allow them to specify the configuration.
3. Have a form on the page submit the config to the ESP32's endpoint [^1]
4. The ESP32 then stashes the config in its non-volatile flash memory so it doesn't need to be configured every time it's power cycled.
5. Finally, the ESP32 disables the AP and connects to the configured WiFi network and MQTT broker.

This new behaviour builds on the WiFi connection flow from the previous post, which now looks something like (the new state is green):

![Notifier WiFi connection diagram](/assets/images/2024-03-24-gatekey-notifier-upgrade/NotifierWifiFlow.png)

- On startup, the ESP32 checks for stashed config to determine whether to transition to the `Disconnected` or `APConfiguring` states.
- Once the creds are configured, the notifier transitions through `Disconnected` to the `Pending` state, where it attempts to establish a connection. This state can now also transition back to `APConfiguring` under two circumstances:
	- If the board was only just configured, it's possible that the user entered the WiFi credentials incorrectly, so fall back and let them try again after failing for 10 seconds.
	- If the board hasn't connected to WiFi since being powered on, has connected in previous power cycles, and fails to connect for 120 seconds, fall back and let the user reconfigure it [^2].

## RTFM

Given that I had a pretty good idea of what I wanted and could explain it in natural language, I decided to ask ChatGPT how to do it, rather than digging through ESP32 docs to find the specific libraries I needed to use. It worked really well; given

> Write some ESP32 arduino code to place the arduino in access point mode. When clients connect to the access point and open their browsers, they should see a form that prompts them to input the wifi username and password as well as a third unique identifier field. The arduino should then use those credentials to connect to the specified wifi network.

it dumped out code that showed me how to achieve what I wanted (some bits snipped for brevity):

```cpp
#include <WiFi.h>
#include <WiFiClient.h>
#include <WebServer.h>

const char *ssid = "ESP32-AP"; // Name of the Access Point
const char *password = "password"; // Password for the Access Point
WebServer server(80);

String webpage = R"(
<!DOCTYPE html>
<html>
<head>
    <title>WiFi Configuration</title>
</head>
<body>
    <h1>WiFi Configuration</h1>
    <form method='post' action='/configure'>
        WiFi SSID: <input type='text' name='ssid'><br>
        WiFi Password: <input type='password' name='password'><br>
        Unique Identifier: <input type='text' name='uniqueid'><br>
        <input type='submit' value='Submit'>
    </form>
</body>
</html>
)";

void handleRoot() {
  server.send(200, "text/html", webpage);
}

void handleConfigure() {
  String ssid = server.arg("ssid");
  String password = server.arg("password");
  String uniqueid = server.arg("uniqueid");

  //... Snip connection establishment
}

void setup() {
  Serial.begin(115200);
  WiFi.softAP(ssid, password); // Start the Access Point
  
  server.on("/", handleRoot);
  server.on("/configure", handleConfigure);
  
  server.begin();
}

void loop() {
  server.handleClient();
}
```

The code still had to be integrated with mine, but it achieved exactly what I'd asked for and saved me faffing around with HTML (which I haven't written much of in years). 

What really felt like the most useful part was that ChatGPT had operated like an index of the information I needed and had saved me several minutes of crafting google searches to discover the `WebServer` library. Given ChatGPT's output, I looked through the the `WebServer` docs and also wound my way to discovering the `ESPmDNS` library, which adds DNS support to the ESP32's AP network. Doing so lets clients navigate to http://GateKeyNotifier.local rather than `192.168.4.1`, making it a little more user-friendly. There's really nothing more to say about it because it's dead simple to use:

```cpp
#include <ESPmDNS.h>
...
MDNS.begin("GateKeyNotifier");
```
## Conclusions

From there, it was fairly straightforward to integrate the new configuration logic with my existing code. I won't dive into it in great detail here; for that, you can just take a look at [the commit](https://github.com/BrydonLeonard/GateKeyNotifier/commit/aabd957ce962233527df9afadd68cef14ab8208e). The bigger takeaway for me here was how much faster ChatGPT made the process of exploration and discovery[^3].

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fautomation%2F2024%2F03%2F24%2Fgatekey-notifier-wifi-upgrade.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)

## Footnotes

[^1]: Look ma, a [real](https://htmx.org/essays/how-did-rest-come-to-mean-the-opposite-of-rest/) REST API!
[^2]: If I revisit this again, I'll likely move this transition over to a little reset button on the box instead. The "Fail after 120 seconds" approach works fine, but could get frustrating if the power trips and the notifier resets itself.
[^3]: I recently bought a Windows 11 laptop, so I also tried Copilot while working on this change. It's convenient having it built into the OS, but it's much slower than ChatGPT, it's answers are needlessly verbose (even relative to ChatCPT), and it hallucinates _significantly_ more (at least anecdotally).

