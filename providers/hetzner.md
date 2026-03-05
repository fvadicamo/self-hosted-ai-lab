# Hetzner Cloud

Provider-specific instructions for creating and configuring a server on [Hetzner Cloud](https://www.hetzner.com/cloud).

> New accounts get EUR 20 credit with [this referral link](https://hetzner.cloud/?ref=7UcZyMnU7io5) (disclosure: referral gives credit to both parties). Enough for 2-3 months on an entry-level ARM64 instance.

## Placeholders

| Placeholder | Description | Example |
|---|---|---|
| `<HOSTNAME>` | Server hostname | `myserver` |
| `<SSH_KEY_LABEL>` | Label of your SSH key in Hetzner | `my-macbook` |

## Prerequisites

- [ ] Hetzner Cloud account active with a project created
- [ ] SSH public key uploaded (Security > SSH Keys)
- [ ] Firewall created **before** server creation (see below)

## Step 1 - Create firewall

Create the firewall first. If you create it after the server, the server is exposed without protection until you attach it.

1. Go to **Firewalls** > **Create Firewall**
2. Add inbound rules:

| Port | Protocol | Source | Service |
|---|---|---|---|
| 22 | TCP | Your IP (`<YOUR_IP>/32`) | SSH |
| 80 | TCP | `0.0.0.0/0` | HTTP |
| 443 | TCP | `0.0.0.0/0` | HTTPS |

3. Name it (e.g., `default-web`)
4. Save

> Note: SSH restricted to your IP is the first layer of defense. HTTP/HTTPS open for Caddy ACME challenges and web traffic. Update your IP when it changes.

## Step 2 - Create server

1. Log in to [Hetzner Cloud Console](https://console.hetzner.cloud)
2. Select your project
3. Click **Add Server**
4. **Location:** choose datacenter (Falkenstein FSN1 or Nuremberg NBG1 for EU)
5. **Image:** Ubuntu 24.04
6. **Type:** choose from CAX series (ARM64) for best price/performance
7. **Networking:** leave IPv4 and IPv6 enabled
8. **SSH Keys:** select your uploaded key
9. **Firewalls:** select the firewall created in Step 1
10. **Cloud config:** paste your `cloud-init.yaml` in the **User data** field (see [templates/cloud-init.yaml](../templates/cloud-init.yaml))
11. **Name:** enter `<HOSTNAME>`
12. Click **Create & Buy Now**

### Recommended instance types

| Type | Arch | vCPU | RAM | Disk | Use case |
|---|---|---|---|---|---|
| CAX11 | ARM64 | 2 | 4 GB | 40 GB | Minimal: n8n only |
| CAX21 | ARM64 | 4 | 8 GB | 80 GB | Recommended: full stack |
| CAX31 | ARM64 | 8 | 16 GB | 160 GB | Heavy workloads, multiple services |

> Note: ARM64 (Ampere Altra) is deliberately chosen - lower cost than x86 at equivalent performance. Docker supports ARM64 natively. Verify that all Docker images you plan to use are available for `linux/arm64`.

## Step 3 - Wait for provisioning

After creation:

1. Note the **IPv4** and **IPv6** addresses from the console
2. Wait 3-5 minutes for cloud-init to complete
3. The server reboots automatically when done
4. Wait 1-2 minutes after reboot before connecting

## Step 4 - Verify connection

```bash
ssh <USER>@<IPV4_ADDRESS>
```

If connection fails, wait another 2-3 minutes (cloud-init may still be running). If it keeps failing, use the **VNC Console** in Hetzner for diagnosis.

## Step 5 - Update your records

After successful connection, note down:

```bash
# IPv4
curl -4 ifconfig.me

# IPv6
curl -6 ifconfig.me

# SSH host fingerprint
ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub
```

## Recovery

If you lose SSH access:

1. Open Hetzner Console > select server > **VNC Console**
2. Log in with root credentials (if available) or use Hetzner rescue mode
3. Check `/etc/ssh/sshd_config.d/99-hardening.conf` for misconfigurations
4. Verify your SSH key is in `/home/<USER>/.ssh/authorized_keys`
