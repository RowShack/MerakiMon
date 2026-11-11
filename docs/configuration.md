# MerakiMon configuration reference

Everything is set in the web interface under **Settings** (admins only), and applies immediately. Back to the
[README](../README.md).

## MerakiMon's own sections

| Section | What admins manage |
|---|---|
| **Organisations** | Add, edit, test, pause or delete organisations: the API key (stored encrypted, never shown again), which networks are monitored, and the Meraki host (global, or a regional host). |
| **Monitoring** | The poll interval (60 seconds by default; a poll costs about six API calls per organisation however many devices it has), how long history is kept, the saved weekly and monthly reports, the experience probes and whether security advisories are fetched. |

History is kept at three resolutions: every sample for a number of hours (default 6), 1-minute averages for a number of days
(3) and 15-minute averages (90), which reports and long charts use. A device's detail panel has history charts for the chosen
History range: online over time for every device; clients and channel use for an access point; uplink latency, loss and usage
for an MX; ports with a link and, where Meraki reports it, traffic for a switch.

## Experience probes

Off by default, because MerakiMon reads Meraki's cloud and its server can be anywhere. If the server sits inside one of the
monitored networks, turn the probes on under **Settings > Monitoring**, choose that network and enter its gateway address
(the appliance's LAN address in the server's VLAN; leave it empty to skip the gateway check). Every 30 seconds the server
then pings the gateway, asks two DNS servers, reaches the internet by address and opens two web sites. The results are on
that network's Overview, and the **Experience** area raises an issue that says where the problem is: near the server, between
the LAN and the internet, DNS, or one web site. They are checked even while Meraki's cloud can't be read.

## Security advisories

On each network's **Firmware & lifecycle** tab. Once a day MerakiMon asks the NVD (the US National Vulnerability Database)
for the CVEs of each MX and Z model in use, and matches their version ranges against the appliance firmware on your server.

- **What is sent:** the NVD's name of a model (for example the MX67's firmware) and the word "meraki". Never a firmware
  version, serial number, device, network or organisation name.
- **What you see:** CVE id, CVSS score, description, the devices, the fixed version. A network affected by one gets an issue
  (a warning from CVSS 7.0). An admin can **Dismiss** a CVE for one network or for all, with a note.
- **To check by hand:** the NVD has no version data for Meraki CVEs published from 2025 on, so those can't be matched. They
  are listed separately with a link (the NVD page links Cisco's advisory), and raise one note per network until dismissed.
- **Not covered:** access points, switches, cameras, sensors and cellular gateways. The NVD lists Meraki firmware for MX and
  Z appliances only.
- **Settings > Monitoring** turns the fetch off or sends it through a proxy. The demo uses made-up samples and never fetches.

This product uses data from the NVD API but is not endorsed or certified by the NVD.

## API tester

`merakimon test-api` (on Windows `MerakiMon.exe test-api`, as an administrator) makes every read call MerakiMon makes, once,
for each saved organisation, and prints for each whether it answered, whether the fields MerakiMon reads are there, and what
stays empty when they aren't. **Settings > Organisations > Full API test** shows the same list.

| Option | What it does |
|---|---|
| `--org NAME` | only that organisation (its name or Meraki id) |
| `--demo` | test against the built-in simulator, with no network |
| `--dump DIR` | also save every raw answer under `DIR`, with serial numbers, addresses and names replaced |
| `--no-redact` | with `--dump`: keep the real values (don't share that folder) |
| `--data-dir DIR` | use that data folder |

The API key is never printed and never saved in a dump.

## Issues & events

The tab has filters (severity, area, acknowledged or excluded, free text), tick boxes to **Acknowledge**, **Mute** or remove an
acknowledgement for several issues at once, **score exclusions** per network (a device by serial, a switch port, an SSID name
or the whole network: still listed, but it raises no alerts and does not lower the score) and **critical switch ports** (a lost
link on one raises a critical issue; marked with a star on the Switches tab). Both are stored per network.

## Thresholds

These health checks have settings in the `thresholds` section of the configuration (they have defaults that suit most
networks):

- **Wi-Fi channel busy:** a radio at or above 70% channel utilisation (warning) or 85% (critical) for several polls.
- **Failed Wi-Fi connections:** many different devices failing on one SSID or access point in the last hour.
- **Capacity:** switch ports forecast to reach 90% of a network's ports within 30 days (warning) or 7 (critical).
- **Port flapping:** at least 6 up/down changes within 60 minutes.
- **Port speed:** an uplink or a critical port linked below 1 Gbps, and any port at half duplex.
- **Wi-Fi interference:** non-Wi-Fi use of a radio's channel at or above 30% for several polls.
- **DHCP pool:** a pool of the appliance at or above 90% full (warning) or 98% (critical); pools under 8 addresses are skipped.
- **Cellular backup:** the appliance's cellular uplink is the active one.
- **Experience probes:** 5% or more of the server's own pings lost.
- **Warm spare (HA)** and **VPN peer unreachable** checks (see the note in the [README](../README.md) about these tabs).

## What is read, and how often

| Data | How often |
|---|---|
| Devices, uplinks, ports (organisation-wide calls) | every poll |
| Wi-Fi: SSIDs per access point and channel utilisation | 5 minutes |
| Wi-Fi clients and failed connections | 5 minutes, per monitored network, **kept in memory only** (client MACs are never stored in the history) |
| Site-to-site VPN | 2 minutes |
| Switch stacks, warm spare detail | 30 minutes |
| Link-layer topology (Network Map) | 15 minutes |
| Per-switch port statistics (traffic, PoE, neighbour) | a few switches per poll, each again after 10 minutes |
| DHCP pools of each appliance | up to three appliances per poll, each again after 10 minutes |
| CVE list (from the NVD, not from Meraki) | once a day, one request per MX or Z model in use and one more |

## Settings shared by every MonStack app

These sections are the same in FortiMon, UniMon, MerakiMon and SiteMon. Open **Settings** with the gear button at the top
right. Changes apply immediately; nothing here needs a restart.

### Users and roles

| Role | Can |
|---|---|
| **Admin** | Everything: the settings, users, API keys, backup and restore, the audit log, acknowledging issues and saving reports. |
| **Operator** | Everything a view-only user can, plus acknowledging and muting issues and saving reports. Cannot change settings or users. |
| **View only** | Every dashboard and report. Cannot change anything. |

Operators and view-only users can be limited to some sites; they then see only those everywhere, including the wallboard
and the API. No sites ticked means every site, including ones added later. Admins always see every site.

Admins add users under **Settings > Users** with a temporary password, and can change roles, reset a password (which
signs the person out everywhere), and disable or delete accounts. There is always at least one active admin. Everyone
can change their own password under **My account**.

- **Lockout:** after 5 wrong passwords in 15 minutes from one address, or for one username, sign-in is locked for a while.
- **Session expiry:** a session ends after 24 hours without use, and 14 days after signing in at the latest.
- **Two-step sign-in:** anyone can add authenticator-app codes (with 10 one-time backup codes) under **My account**. Admins
  can require it for **Nobody**, **Admins** or **Everyone** under **Settings > Users**.

Forgotten password with no other admin, or a lost phone with no backup codes? Run this on the server itself:

```
"C:\Program Files\MerakiMon\MerakiMon.exe" --reset-password admin     (Windows, as Administrator)
sudo merakimon --reset-password admin                                 (Linux)
```

`--reset-mfa USER` turns off two-step sign-in for a user, who sets it up again at the next sign-in.

### API keys

**Settings > API keys** makes keys for other programs. A **wallboard** key opens the full-screen wallboard on a screen nobody
signs in to. A **card** key lets SiteMon read this app's summary (`GET /api/card/v1`) and nothing else. A key is shown once,
stored only as a hash, can be limited to some sites, and can be revoked. The same page has **Send to SiteMon**, which
pushes the card to a SiteMon that cannot reach this server (see the [SiteMon guide](https://github.com/RowShack/SiteMon)).

### Appearance & branding

Dark, light or follow the device, the accent colour, and your own product name and logo.

### Web server & HTTPS

Who can connect (this computer or the network), the port, HTTPS on or off, the address used in links in email and alerts,
and the HTTPS certificate.

On first start the app makes a self-signed certificate for the server's names and addresses and renews it before it
expires, so browsers warn the first time. Either continue past the warning, upload your company's certificate (PEM, with
its key) here, or trust the app's own certificate on the computers that use it. Turn HTTPS off only behind a reverse proxy
that does HTTPS (`--http` on the command line does the same).

### Email reports

Enter your SMTP server (Microsoft 365, Google Workspace and internal relays all work) and use **Send test email**. Add
recipients, who do not need an account, and choose for each weekly and/or monthly reports and which sites. The weekly
report goes out on Monday at 06:00 by default (the day and hour are set under **Monitoring** in FortiMon, UniMon and
MerakiMon) and the monthly one on the 1st.
**Send now** sends this week's report straight away.

### Alerts

**Settings > Alerts** sends issues to people as they happen. Add destinations: **Microsoft Teams** (a Workflows webhook),
a **webhook** (JSON to any HTTPS address) or **email** (using the SMTP server above). Each destination has its own filters
for sites, areas, minimum severity and problem types, and can also send when an issue clears.

- Issues raised close together are grouped into one message a minute, and each destination is capped per hour.
- Muted and score-excluded issues never alert; an acknowledged issue alerts again only if it gets worse.
- A failed delivery is retried a few times. Each destination keeps a delivery log, and admins see a banner if one has been
  failing for over an hour. **Send test** sends a clearly marked sample.

To make a Teams URL: in the channel choose **... > Workflows**, pick **Post to a channel when a webhook request is
received**, name it, choose the team and channel, copy the URL it shows and paste it into the destination. Use a Workflows
URL: Microsoft is retiring the old Office 365 Incoming Webhook connectors.

### Updates

**Settings > Updates** checks this app's GitHub releases once a day (it is a public page, so no account or token is
needed) and shows what is new. For a **service** install, **Update now** downloads the release, checks its signature and
checksums, backs up your data, installs it, and checks that the new version answers. If it does not, the helper puts the
old version back by itself. A server without internet access can use the **offline update** file from the release.
The channel is **Stable**, or **Test releases too** for a test server.

### Audit log, backup and restore

The audit log records every sign-in (including failed ones), settings and user change, acknowledgement and report email,
with who, when and from which address, for 400 days.

**Backup & restore** downloads all settings and users as one file and restores it, for example on a new server. Secrets in
it (API keys, the SMTP password) are encrypted with a backup password you choose. The history database is not in the
backup: copy `merakimon.db` from the data folder if you need it. Two-step sign-in is not in a backup, so users set it up
again after a restore.

### The data folder

| File | What it is |
|---|---|
| `config.json` | Settings, written by MerakiMon when admins change them (the previous version is kept as `config.json.bak`) |
| `merakimon.key` | The key that encrypts stored secrets. On Windows it is itself protected by Windows (DPAPI); on Linux only the service user can read it. Without it the keys and passwords have to be entered again. |
| `admin.db` | Users (password hashes), sessions and the audit log |
| `merakimon.db` | History |
| `tls/` | The HTTPS certificate and key |
| `reports/`, `logs/` | Saved reports and the service log |

`C:\ProgramData\MerakiMon` on a Windows service, `/var/lib/merakimon` on a Linux service.

### Command line

| Option | Meaning |
|---|---|
| `--data-dir DIR` | Where settings, users and history are kept |
| `--demo` | Simulated data, no gear needed (it keeps its own settings and users) |
| `--port 6443` / `--bind 0.0.0.0` | Web interface port / address (saved to the settings) |
| `--http` | Turn HTTPS off, for example behind a reverse proxy (saved) |
| `--interval N` | Seconds between reads (saved) |
| `--reset-password USER` | Set a new password (creates the user as an admin if missing; `--make-admin` also makes an existing user admin) |
| `--reset-mfa USER` | Turn off two-step sign-in for a user |
| `--open` / `--no-browser` | Open the running app in the browser / do not open it on start |
| `--log FILE` | Also write the output to a file |
| `--version` | Print the version |
