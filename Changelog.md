# Changelog

All notable changes to this project will be documented in this file.

## 1.0.0 - 2026-05-23

### Added

- `OpenStack Ubuntu Server OVA v1.0` - A pre-configured OVA with all fixes and configurations applied, replacing the previous OVA file. See the [README](./README.md) for the download link.
- Startup script `fix-br-ex.sh` to persist `br-ex` bridge IP address across reboots.
- Systemd service `fix-br-ex.service` to automatically run the bridge fix on every system restart.

### Fixed

#### Networking - `br-ex` Bridge IP Dissociation on Reboot

The core issue was that after every reboot, the `br-ex` bridge in OpenStack (the public bridge responsible for connecting the Ubuntu host to the internal OpenStack instances) would lose its assigned IP address. This caused all external connectivity to break - instances could not be pinged or reached via SSH from the Ubuntu host.

**Root Cause:** OpenStack's `br-ex` bridge IP (`172.24.4.1/24`) was not being persisted across reboots, so the bridge would come up without an IP, severing the route between the host and the internal instance network.

**Fix - Startup Script & Systemd Service:**

Create the fix script:

```bash
sudo nano /usr/local/bin/fix-br-ex.sh
```

Paste the following content:

```bash
#!/bin/bash
ip link set br-ex up
ip addr add 172.24.4.1/24 dev br-ex || true
```

Make it executable and create the systemd service:

```bash
sudo chmod +x /usr/local/bin/fix-br-ex.sh
sudo nano /etc/systemd/system/fix-br-ex.service
```

Paste the following content:

```ini
[Unit]
Description=Fix OpenStack br-ex bridge
After=network.target

[Service]
Type=oneshot
ExecStart=/usr/local/bin/fix-br-ex.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Enable and start the service:

```bash
sudo systemctl daemon-reload
sudo systemctl enable fix-br-ex.service
sudo systemctl start fix-br-ex.service
```

> **Note:** If `br-ex` is not connected to your Ubuntu bridge adapter, run the following command to attach it. Replace `<your_bridged_adapter>` with the name of your bridged network interface (e.g., `eth0`, `enp0s3`):
>
> ```bash
> sudo ovs-vsctl add-port br-ex <your_bridged_adapter>
> ```

---

#### Networking — Instances Unreachable via Ping and SSH

Even with `br-ex` correctly configured, instances were still unreachable because the OpenStack default security group blocks all ingress traffic by default.

**Fix — Security Group Rules:**

In the OpenStack Horizon Dashboard, navigate to:
`Network → Security Groups → Default → Manage Rules → Add Rule`

Add the following two ingress rules:

| Direction | Protocol | IP Version | CIDR      |
| --------- | -------- | ---------- | --------- |
| Ingress   | All ICMP | IPv4       | 0.0.0.0/0 |
| Ingress   | SSH (22) | IPv4       | 0.0.0.0/0 |

> **Note:** In a production environment, restrict the CIDR to trusted IP ranges rather than `0.0.0.0/0`.

---

### Important Configuration Notes

#### Static IP for OpenStack Host (Recommended)

The IP address of the Ubuntu host running OpenStack must be **static**. If the host IP is assigned dynamically via DHCP, it can change between sessions, breaking the `br-ex` bridge routing and any host-side routes pointing to the internal instance network.

Set a static IP on your Ubuntu host via your router's DHCP reservation settings or directly in the network configuration (`/etc/netplan/`).

---

#### Accessing Instances from Your Host Machine

To reach OpenStack instances from your physical host machine (e.g., Windows PC on the same network), add a persistent route that directs traffic for the internal OpenStack network (`172.24.4.0/24`) through the Ubuntu host's IP.

Run the following on your **host machine** (Windows):

```cmd
route add 172.24.4.0 mask 255.255.255.0 <Ubuntu Host IP> -p
```

> Replace `<Ubuntu Host IP>` with the actual static IP of your Ubuntu host on the local network. The `-p` flag makes the route persistent across reboots.

After adding the route, ICMP (ping) and SSH to instances should work from the host machine, provided the security group rules above are in place.

---

### Changed

- `README.md` - Updated OVA section with new versioned file name (`OpenStack Ubuntu Server OVA v1.0`) and download link.
- `README.md` - Added reference to this `CHANGELOG.md` in the OVA section.
- `README.md` - Added `Credentials` for initial access inside the pre-configured instance.
- `README.md` - Updated `Pre-Requisites & System Requirements` added network type.
