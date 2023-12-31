<!-- Mermaid doesn't work on GitHub pages AFAICT, so just pre-rendering this and dumping it here for now -->

```mermaid
sequenceDiagram
    autonumber
    actor Resident
    participant Notifier
    box rgb(245, 245, 245) GateKey
        participant Telegram Bot
        participant HTTP API
        participant Device Manager
    end
    par
        note over Notifier: In reality, this could happen after step 5
        loop Until registered
            Notifier ->> HTTP API: register_device
            HTTP API ->> Device Manager: Check device
            Device Manager ->> HTTP API: pending
            HTTP API ->> Notifier: ACCEPTED
        end
    and
        Resident ->> Telegram Bot: Add device 123
        Telegram Bot ->> Device Manager: Add device 123 to household
        Telegram Bot ->> Resident: Added
    end
    note over Resident,HTTP API: The device is now registered on the server
    Notifier ->> HTTP API: register_device
    HTTP API ->> Device Manager: Check device
    Device Manager ->> HTTP API: registered
    HTTP API ->> Notifier: REGISTERED
    Notifier ->> Notifier: Init MQTT client
```