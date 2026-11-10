# MerakiMon setup guide

This takes you from nothing to a dashboard showing your first Meraki organisation. Back to the [README](../README.md).

MerakiMon reads Meraki with a **read-only Dashboard API key**, so it can look but not change anything.

## Requirements

- A **Windows** server (64-bit) or a **Linux** server with systemd and Python 3.9 or newer (Ubuntu, Debian, RHEL, Rocky
  and similar). The Windows installer includes everything it needs.
- A web browser. MerakiMon serves its dashboard over HTTPS, on port 6443 by default.

## Install

Download the newest files from the [latest release](https://github.com/RowShack/MerakiMon/releases/latest).

### Windows

1. Download `MerakiMon-<version>.msi` and `SHA256SUMS.txt`.
2. [Check the download](#check-the-download).
3. Double-click the MSI and answer the questions:

   | Question | Choices |
   |---|---|
   | **Run MerakiMon** | **As a Windows service** (recommended): starts with Windows, keeps monitoring when nobody is signed in and restarts itself if it stops. Or **as a program** that runs only while it is open. |
   | **Who can open the web interface** | **Other computers on the network** (adds a Windows Firewall rule) or **only this computer**. |
   | **Port** | 6443 by default (HTTPS). |

   At the end it opens MerakiMon in your browser, where you create the first admin account.

Silent install (Intune, GPO, an RMM tool):

```
msiexec /i MerakiMon-<version>.msi /qn MODE=service ACCESS=all PORT=6443
```

`MODE` is `service` or `app`, `ACCESS` is `all` or `local`. An upgrade keeps the earlier choices and all data: install the
newer MSI over the old one.

The service is called **MerakiMon** in `services.msc`. Its data (settings, users, history, certificates, logs) is in
`C:\ProgramData\MerakiMon`. Uninstalling keeps that folder; delete it yourself to remove everything.

**Windows may warn you.** The installer is not signed with a paid code-signing certificate yet, so SmartScreen can show
"Windows protected your PC". Check the SHA-256 below first. If it matches, choose **More info**, then **Run anyway**.

### Linux

```
sudo sh merakimon-<version>-linux.run
```

It asks whether to install a systemd service (recommended), the port, whether other computers may connect and, if ufw or
firewalld is active, whether to open the port. Run a newer `.run` over the top to upgrade; data is kept.

| | |
|---|---|
| Program | `/opt/merakimon` |
| Data | `/var/lib/merakimon` |
| Service | `systemctl status merakimon`; logs with `journalctl -u merakimon -f` |
| Command | `sudo merakimon --help` runs MerakiMon's commands against the service's data |
| Uninstall | `sudo /opt/merakimon/uninstall.sh` (add `--purge` to delete the data too) |

For no questions: `sudo sh merakimon-<version>-linux.run --unattended --service --port 6443`. `--local-only`,
`--no-firewall` and `--no-service` also work; `--help` lists everything.

### Check the download

Each release lists `SHA256SUMS.txt`. The checksum of your file must match its line there.

```
# Windows (PowerShell)
Get-FileHash .\MerakiMon-<version>.msi -Algorithm SHA256

# Linux
sha256sum -c SHA256SUMS.txt --ignore-missing
```

The in-app updater goes further: it only accepts releases signed with MonStack's own key, which is built into the app.

## First sign-in

Open `https://<server>:6443/`. The first time, MerakiMon asks you to **create the first admin account**. If you create it
from another computer rather than on the server itself, it asks for a setup code, which is in `setup-code.txt` in the data
folder and in the log. MerakiMon makes its own self-signed HTTPS certificate, so browsers warn the first time.

## Make an API key in the Meraki dashboard

In the Meraki dashboard open your profile (**My profile > API access**) and generate an **API key**. Use an account with
read-only (monitor-only) access to the organisation.

## Add an organisation

1. In MerakiMon open **Settings > Organisations > + Add organisations**.
2. Paste the key and choose **Find my organisations**, then tick the ones to monitor.
3. Choose which networks of an organisation are monitored. By default every network is, including new ones; once some are
   unticked, networks that appear later are marked new and wait to be ticked. You can change this later under
   **Settings > Organisations > Edit**.

Each Meraki network becomes a site. The key is sent to Meraki and nowhere else. Meraki's API allows 10 requests a second per
organisation and MerakiMon stays well under that. An organisation that uses a regional Meraki host has a base-URL setting for it.

## Try it without a Meraki account

`merakimon --demo` (or `python merakimon.py --demo` from source) runs simulated organisations. It keeps its own settings and
users.

## Some tabs fill in over time

The Switches tab shows each port's traffic, PoE watts, clients and neighbour only after the switch has been asked, which is a
few switches per poll, so give it a few minutes. Wi-Fi, Wi-Fi Clients, Network Map and VPN & HA have their own read intervals
(5 minutes for Wi-Fi, 15 for the map, 2 for VPN); see the [FAQ](faq.md).

## Next steps

Create users, set up alerts and reports, and put MerakiMon's address under **Settings > Web server & HTTPS** so links in alerts
open the right page: see the [configuration reference](configuration.md).
