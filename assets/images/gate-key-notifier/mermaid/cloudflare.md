```mermaid 
sequenceDiagram
    actor Caller
    autonumber
    Caller ->> CloudFlare: DNS lookup
    CloudFlare ->> Caller: CloudFlare server address
    Caller ->> CloudFlare: HTTP request
    CloudFlare ->> GateKey: HTTP request
    GateKey ->> CloudFlare: Response
    CloudFlare ->> Caller Response
```