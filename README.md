# iptables-firewall-simulation-lab
## Obective:
In this lab, I build and document a three zone perimeter firewall simulation using Containerlab, Alpine Linux, and iptables. It should meet the following specification:
```mermaid
flowchart TB
    subgraph WAN ["WAN (Untrusted Zone - 17.16.1.0/24)"]
        wan_client["WAN Client<br/>172.16.1.100/24<br/>GW: 172.16.1.1"]
        wan_attacker["WAN Attacker / Blacklisted Host<br/>172.16.1.200/24<br/>GW: 172.16.1.2"]
    end

    subgraph FIREWALL ["Perimeter Firewall Router"]
        router["Alpine Linux Router<br/>(iptables)<br/><br/>eth1: 10.1.1.1/24<br/>eth2: 192.168.10.1/24<br/>eth3: 172.16.1.1/24<br/>eth4: 172.16.1.2/24"]
    end

    subgraph LAN ["LAN (Trusted Zone - 10.1.1.0/24)"]
        lan_client["LAN Client<br/>10.1.1.10/24<br/>GW: 10.1.1.1"]
    end

    subgraph DMZ ["DMZ (Public Services - 192.168.10.0/24)"]
        dmz_server["DMZ Web Server<br/>192.168.10.10/24<br/>GW: 192.168.10.1"]
    end

    router ---|eth1 <--> eth1| lan_client
    router ---|eth1 <--> eth2| dmz_server
    wan_client ---|eth1 <--> eth3| router
    wan_attacker ---|eth1 <--> eth4| router

    classDef firewall fill:#5b5b5b,stroke:#333,stroke-width:2px;
    classDef wan fill:#b91c1c,stroke:#d9534f,stroke-width:1px;
    classDef lan fill:#44742e,stroke:#5cb85c,stroke-width:1px;
    classDef dmz fill:#4e4787,stroke:#f0ad4e,stroke-width:1px;

    class FIREWALL firewall;
    class WAN wan;
    class LAN lan;
    class DMZ dmz;
```
---
