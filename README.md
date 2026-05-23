# OpenStack Installation Guide for Ubuntu

### Pre-Requisites & System Requirements

- VirtualBox or VMWare Player
- Ubuntu (Desktop)
- 8 GB RAM
- 6 CPU Cores
- 100 GB Disk Space
- Network Type : Bridged

### Credentials

```
username : openstack
password : openstack
```

### Commands

> For more information about commands look at the command brief section below. <br />
> **For notes refer to [Documentation.md](https://github.com/syncattacker/Project-Private-Cloud/blob/main/Documentation.md)**

```
apt update && apt upgrade -y
apt install net-tools -y
apt install git -y
sudo useradd -s /bin/bash -d /opt/stack -m stack
sudo chmod +x /opt/stack
echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack
sudo -u stack -i
git clone https://opendev.org/openstack/devstack
cd devstack
nano local.conf
./stack.sh
```

#### LOCAL CONFIGURATION

```
[[local|localrc]]
ADMIN_PASSWORD=admin
DATABASE_PASSWORD=$ADMIN_PASSWORD
RABBIT_PASSWORD=$ADMIN_PASSWORD
SERVICE_PASSWORD=$ADMIN_PASSWORD
```

## First Look of HORIZON Dashboard.

![OpenStack Login Page](/images/general/login.png)
<br />
<br />
<br />
![OpenStack Dashboard Overview](/images/general/openstack.png)

## Access The Ova File Here!

> **Note:** The OVA file has been updated. See [CHANGELOG.md](./CHANGELOG.md) for details on why this was done. <br />
> OpenStack Ubuntu Server OVA v1.0 — [Download OVA](#) _(link coming soon)_

## Command Briefing

`echo "stack ALL=(ALL) NOPASSWD: ALL" | sudo tee /etc/sudoers.d/stack`

> `echo` returns output as `stack ALL=(ALL) NOPASSWD: ALL` <br />

> `stack` refers the user created <br />

> `ALL:(ALL)` means stack is allowed to run commands as any user. <br />
> So, if the user stack wants to run a command as another user (like root, or any other user on the system), they can do so. Normally, a user can only run commands as themselves, but this allows stack to act as any user. <br />

> `| (PIPE)` takes the output of echo and passes it as input to the next command `sudo tee` in this case. <br />

> `sudo` getting administrative privileges. <br />

> `tee` reads from the standard input (in this case output by the echo command) and writes it to the file specified `/etc/sudoers.d/stack` <br /> <br />

`sudo -u stack -i`

> `sudo` run command as another user (default superuser).

> `-u stack` user you want to switch to, stack in this case.

> `-i` open interactive shell. <br />
