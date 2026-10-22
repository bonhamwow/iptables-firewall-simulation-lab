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
## Requirements to Demo this Lab
This lab was created inside Containerlabs and is meant to be run within Linux or WSL. The YAML file and config files are available from the repository.

# Running the Lab
## Deploying the Lab
Open Containerlabs and navigate to the downloaded YAML file, then run the following command:
```
sudo containerlab deploy -t topology.clab.yml
```
## Testing the Firewall
To test connections between networks, where activity from the Attacker should be automatically blocked and HTTP and HTTPS from external WAN clients is all that is allowed, run the following test commands in the WSL terminal to see that the firewall rules are properly set up:
### **Test 1: Web Access from WAN Client to DMZ**

* **Command:** `docker exec -it clab-iptables-firewall-lab-wan-client curl -I http://192.168.10.10`
* **Expected Result:** `HTTP/1.1 200 OK`

### **Test 2: SSH Access from WAN Client to DMZ (Blocked)**

* **Command:** `docker exec -it clab-iptables-firewall-lab-wan-client nc -zv -w 3 192.168.10.10 22`
* **Expected Result:** Connection timeout / failure

### **Test 3: Blacklisted Attacker Access to DMZ (Blocked)**

* **Command:** `docker exec -it clab-iptables-firewall-lab-wan-attacker curl --connect-timeout 3 http://192.168.10.10`
* **Expected Result:** Connection timeout

### **Test 4: Direct Unsolicited WAN Access to Internal LAN (Blocked)**

* **Command:** `docker exec -it clab-iptables-firewall-lab-wan-client ping -c 3 10.1.1.10`
* **Expected Result:** 100% packet loss

### **Test 5: LAN Workstation Access to DMZ**

* **Command:** `docker exec -it clab-iptables-firewall-lab-lan-client curl -I http://192.168.10.10`
* **Expected Result:** `HTTP/1.1 200 OK`

## Destroying the Lab
Once you've finished with the lab, run the following command to destroy and clean-up:
```
sudo containerlab destroy -t topology.clab.yml
```
