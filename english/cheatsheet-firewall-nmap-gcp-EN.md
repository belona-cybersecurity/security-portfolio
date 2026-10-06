# Cheat Sheet — Firewall, nmap & GCP (Cloud Shell)

**Golden rule:** creating an ALLOW rule = **OPEN** a port · deleting the rule = **CLOSE** it.
nmap only **reads** (scans), it does NOT change anything; the **firewall** opens/closes.

---

## Environment
```bash
# Cloud Shell: >_ button (top-right) at console.cloud.google.com
gcloud config get-value project            # active project
gcloud auth list                           # accounts
gcloud config set account YOUR_EMAIL       # activate an account
```

## Firewall rules (VPC)
```bash
gcloud compute firewall-rules list                    # list all rules
gcloud compute firewall-rules describe RULE_NAME      # detail

# OPEN a port (create ALLOW rule)
gcloud compute firewall-rules create RULE_NAME \
  --network=default --direction=INGRESS --action=ALLOW \
  --rules=tcp:PORT --target-tags=TAG --source-ranges=0.0.0.0/0   # any IP; or YOUR_IP/32

# CLOSE a port (delete the rule)
gcloud compute firewall-rules delete RULE_NAME

# HARDEN (restrict source)
gcloud compute firewall-rules update RULE_NAME --source-ranges=YOUR_IP/32
```

## Port scanning (nmap = read-only)
```bash
sudo apt-get install -y nmap

# RECON (first look): all ports + versions + scripts + save
sudo nmap -sS -sV -sC -p- -T4 -oN recon.txt TARGET_IP

# VERIFY a change (targeted)
nmap -Pn -p PORT TARGET_IP

# SYN scan (professional standard; needs sudo)
sudo nmap -sS -Pn -p 22,80,443 TARGET_IP

# Show only open ports from a saved result
grep open recon.txt
```
**Port states:** `open` = service + allowed · `closed` = allowed but no service · `filtered` = firewall blocks.

## Lab VM (Compute Engine)
```bash
gcloud compute instances create NAME --zone=ZONE --machine-type=e2-micro \
  --image-family=debian-12 --image-project=debian-cloud --tags=TAG
gcloud compute instances list
gcloud compute ssh NAME --zone=ZONE
gcloud compute instances delete NAME --zone=ZONE     # DELETE (avoid cost)
```

## Syntax notes
- gcloud structure: `gcloud SERVICE RESOURCE ACTION --options`
- `-` = an **option/flag** (NOT "remove"). `-p 80` = "port 80"; `-Pn` = "no ping".
- `--` = long option (full word): `--zone`, `--action`.
- `tcp:80` = protocol:port · `0.0.0.0/0` = "any IP" · `/32` = "a single IP".

## nmap scan types
| Flag | What it does |
|---|---|
| `-sS` | SYN scan (half-open, standard; needs sudo) |
| `-sT` | TCP connect (no sudo; noisier) |
| `-sV` | Service/version detection |
| `-sC` | Run default scripts |
| `-sU` | UDP ports |
| `-p-` | ALL 65,535 ports |
| `-Pn` | Skip ping (assume host up) |
| `-oN file` | Save output to file |
