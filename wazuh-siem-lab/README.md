# Wazuh SIEM Home Lab

> **Status:** Working — Windows agent reporting Active on dashboard
> **Built:** September 2026
> **My role:** Sole builder (guided, mentored learning)

---

## Overview

My first hands-on security project: a functional Wazuh SIEM running in a home lab. An Ubuntu Server VM acts as the Wazuh manager (receiving and analyzing logs), and a Windows VM acts as a monitored endpoint with a Wazuh agent installed. The goal was to understand how SOC teams actually collect endpoint logs and to see agent enrollment, service management, and the analyst dashboard firsthand.

Why Wazuh? It's open-source, widely used, and mirrors the exact workflow a junior SOC analyst runs: deploy manager → enroll agent → verify events in the dashboard.

---

## Architecture

```
                Host Laptop: Windows 11, 32 GB RAM
                VirtualBox 7.2.18
  +-------------------------------------------------------------+
  |                                                             |
  |   +----------------------+     +------------------------+   |
  |   | Ubuntu Server 26.04  |     | Windows 10 Pro VM      |   |
  |   | -- Wazuh Manager --  |<--->| -- Wazuh Agent ----    |   |
  |   | 8 GB RAM / 4 cores   |     | 8 GB RAM / 4 cores     |   |
  |   | Hostname: wazuh-srv  |     | IP: 192.168.56.103     |   |
  |   | IP: 192.168.56.102   |     |                        |   |
  |   +----------+-----------+     +------------------------+   |
  |              |                                              |
  |       Host-only network: 192.168.56.0/24                    |
  |              |                                              |
  +--------------+----------------------------------------------+
                 |
        Wazuh Dashboard: https://localhost (host browser)
        Port forwarding: 443 -> dashboard, 2222 -> SSH
```

Design choices and why:
- The manager starts on NAT (for downloads/updates), then moves to a host-only adapter so it is never internet-reachable.
- Static IPs on both VMs (manager .102, agent .103) — SOC tooling assumes stable addressing.
- Dashboard and SSH reached from the host only through VirtualBox port forwarding.

---

## Build Steps

### Phase 1 - Server VM
1. Downloaded Ubuntu Server 26.04.1 LTS; created VM (8 GB RAM, 4 cores, 50 GB dynamic disk).
2. Installed Ubuntu with OpenSSH server enabled; hostname `wazuh-server`.
3. Updated everything: `sudo apt update && sudo apt full-upgrade`.

### Phase 2 - Wazuh manager
1. Installed Wazuh 4.14 with the official installation assistant:
   ```bash
   curl -sO https://packages.wazuh.com/4.14/wazuh-install.sh && sudo bash ./wazuh-install.sh -a
   ```
2. Saved the installer-generated admin password shown at the end of the run.
3. Verified dashboard reachable from the host browser at `https://localhost`.

### Phase 3 - Network hardening
1. Added a VirtualBox Host-Only Ethernet Adapter.
2. Gave the manager a static IP (192.168.56.102/24) via netplan, applied with `sudo netplan apply`.
3. Locked down services: UFW deny incoming default; allowed only 22 (SSH), 443 (dashboard), 1514/1515 (agent comms/registration).
4. Set up SSH key-based access from the host laptop (ed25519 key pair; public key in `~/.ssh/authorized_keys`; PasswordAuthentication no in sshd_config).

### Phase 4 - Windows agent
1. Built a Windows 10 Pro VM on the same host-only network; static IPv4 192.168.56.103, subnet 255.255.255.0, gateway 192.168.56.102.
2. From elevated PowerShell, downloaded the Wazuh 4.14 agent:
   ```powershell
   Invoke-WebRequest -Uri "https://packages.wazuh.com/4.x/windows/wazuh-agent-4.14.8-1.msi" -OutFile "$env:TEMP\wazuh-agent.msi"
   ```
3. Silent install pointed at the manager:
   ```powershell
   msiexec.exe /i "$env:TEMP\wazuh-agent.msi" /q WAZUH_MANAGER="192.168.56.102" WAZUH_AGENT_NAME="windows-lab-vm"
   ```
4. Started the service (`Start-Service WazuhSvc`), verified Status = Running.
5. Confirmed the agent showed Active on the Wazuh dashboard as `windows-lab-vm`.

---

## Challenges & Fixes

| Challenge | Fix | What I learned |
| --- | --- | --- |
| Host-only interface showed `state DOWN`, reported as "cable unplugged" | The VM's `.vbox` config had `cable="false"` for Adapter 2. Powered off the VM and edited the XML directly to `"true"`. | VirtualBox stores network config in XML as well as the GUI — they can silently disagree. |
| Renaming interfaces caused netplan to apply the wrong adapter configs | Netplan binds to interface names; renaming NICs reordered which YAML block applied where. | Never rename interfaces without re-checking config bindings. |
| `wazuh-updates` service failed — localhost dependencies broken | Root cause: cloud-init had assigned `lo` a /32 netmask instead of /8. Corrected to 127.0.0.1/8. | Question "obvious" assumptions — verified with `ip addr show lo`. |
| Windows agent couldn't reach the manager | NAT gave VM-to-internet, not VM-to-VM. Both VMs needed the same host-only adapter and static IPs; verified with `ip addr` / `ipconfig`. | Most "agent won't enroll" problems are Layer-3 issues, not Wazuh issues. Map the network first. |
| Pasted a Markdown-formatted URL into PowerShell and it failed to parse | PowerShell needs plain strings: `-Uri "https://..."` not `[label](url)`. | Docs are written for browsers; shells need plain values. |

---

## Security Notes

- Lab traffic is confined to a VirtualBox host-only network; the manager is not internet-reachable.
- Dashboard and SSH are reachable only from the host via port forwarding.
- SSH is key-only; password authentication disabled.
- No real credentials or public IPs appear in this write-up.

---

## Next Steps

- [ ] Add a Kali Linux attacker VM to generate detectable events
- [ ] Write a custom Wazuh detection rule for SSH brute-force attempts and test it
- [ ] Add a Raspberry Pi (Pi-hole + honeypot) as a third log source
- [ ] Explore File Integrity Monitoring by modifying a watched file on the Windows endpoint

---

## Screenshots

See the [screenshots folder](./screenshots/) — I'll capture dashboard and service views in my next lab session.
