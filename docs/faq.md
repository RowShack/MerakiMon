# MerakiMon FAQ

Back to the [README](../README.md).

## Does MerakiMon change anything in my Meraki dashboard?

No. It uses a read-only Dashboard API key and only reads.

## Where is my API key kept?

Encrypted in `config.json`, with a key stored beside it (`merakimon.key`; protected by Windows DPAPI on Windows, readable only
by the service user on Linux). It is never shown again and is sent only to Meraki.

## Will MerakiMon hit Meraki's API limit?

Meraki allows 10 requests a second per organisation. MerakiMon stays well under that: a poll costs about six calls per
organisation, however many devices it has, and the heavier reads (per-switch ports, stacks, topology, Wi-Fi) are spread
over minutes, a few at a time.

## Which tabs are verified on real gear?

The first release was checked against a real organisation, and every tab is tested against a simulated Meraki. **VPN & HA,
Network Map and Find Device are not verified on real gear yet.** The organisation used for checking has no site-to-site VPN, no
warm spare pair and an empty link-layer topology, so those were built from Cisco's documentation. Please report what you see.

## The VPN tab is empty

An organisation that does not use site-to-site VPN shows nothing there: a "VPN not enabled" answer from Meraki means "not used",
never an issue.

## Switch columns (traffic, PoE, clients, neighbour) are empty

Meraki's organisation-wide port call carries no traffic, PoE watts, clients or LLDP/CDP neighbour. MerakiMon asks each switch
separately, a few per poll, so the columns fill in over the first minutes. Only switches that Meraki reports online are asked.

## Why is there no "lost link" for a port that was already down?

Meraki keeps no port history, so MerakiMon records each port's up/down changes itself. A port that is already down the first
time MerakiMon sees it is not a lost link, and a port switched off by an admin or on an offline switch is neither a lost link
nor a flap.

## Are Wi-Fi client MACs stored?

No. The wireless client list and the failed-connection list are kept in memory only and shown only to people who may see that
site. The history keeps just a count per site.

## I forgot the admin password

On the server itself, run `--reset-password admin` (see [configuration](configuration.md#users-and-roles)).

## How do I update?

**Settings > Updates** checks once a day, and a service install can use **Update now**. Otherwise run the newer installer over
the old one: data and settings are kept.

## Can it feed SiteMon?

Yes. Make a **card** key under **Settings > API keys** and add MerakiMon in [SiteMon](https://github.com/RowShack/SiteMon), or
use **Send to SiteMon** on the same page if SiteMon cannot reach this server.
