# firewall
 
Firewall project: configure a host-based firewall (`ufw`) on a web server so that only the traffic it needs can reach it.
 
**Author:** amtoha-prog
 
## Concepts
 
- **Firewall**: a hardware or software security system that filters incoming and outgoing network traffic using rules.
- **Host-based firewall**: runs on a single machine and protects only that machine (for example `ufw` on Ubuntu).
- **Network firewall**: protects a whole network, sitting between it and the internet (for example an AWS security group).
- **Default deny**: block everything, then allow only the ports that are needed.
## Files
 
| File | Task | Description |
|------|------|-------------|
| `0-block_all_incoming_traffic_but` | 0 | Bash script with the `ufw` commands that block all incoming traffic except TCP ports 22, 80 and 443 |
 
## Task 0: Block all incoming traffic but
 
Applied on `web-01` (the same rules can be used on `lb-01` and `web-02`).
 
| Port | Protocol | Service |
|------|----------|---------|
| 22 | TCP | SSH |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS (SSL/TLS) |
 
All other incoming traffic is denied. Outgoing traffic is allowed.
 
### Commands
 
```bash
sudo apt-get update -y
sudo apt-get install -y ufw
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 22/tcp
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw --force enable
```
 
**Order matters:** port 22 is allowed *before* the firewall is enabled. Enabling `ufw` first would cut the SSH connection and lock you out of the server.
 
### Verify
 
```bash
sudo ufw status verbose
```
 
Expected result:
 
```
Status: active
Default: deny (incoming), allow (outgoing), disabled (routed)
22/tcp    ALLOW IN    Anywhere
80/tcp    ALLOW IN    Anywhere
443/tcp   ALLOW IN    Anywhere
```
 
(plus the same three rules for IPv6.)
 
## Notes
 
- Cloud servers can have two firewalls: the cloud provider's (such as an AWS security group) and `ufw`. A port must be open in both for traffic to arrive.
- Test a new SSH connection in a second terminal before closing the first one.
- To undo everything: `sudo ufw disable`.
