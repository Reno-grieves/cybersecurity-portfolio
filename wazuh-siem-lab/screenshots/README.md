# Screenshots Guide

Drop screenshots of your working lab in this folder. Once added, they'll show up on the project page.

## Must-haves
- `dashboard-agents.png` — Wazuh Dashboard > Agents, showing `windows-lab-vm` with Active status
- `agent-service-running.png` — Windows PowerShell: `Get-Service -Name WazuhSvc` showing Status: Running
- `ssh-connection.png` — your laptop successfully SSH'd into the manager

## Nice-to-haves
- Dashboard login page (no credentials visible!)
- Overview page after login

## How to take a screenshot while a VM is running
1. Click outside the VM window so the host laptop has focus.
2. Press `Win + Shift + S` and drag a box around what you want.
3. Paste into Paint, save as a PNG with the exact name from the list above.
4. Later, copy the PNG into this folder (drag and drop it on the GitHub web page works too).

## Sanitize before adding anything
- Screenshots may include your host's public IP or username — crop or blur those.
- 192.168.56.x lab IPs are fine; they only exist inside your lab.
- Never screenshot passwords, agent keys, or Wazuh API credentials.
