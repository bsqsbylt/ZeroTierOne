ZeroTier Release Notes
======

# 2017-03-14 -- Version 1.2.0

Version 1.2.0 is a major milestone release representing almost nine months of work. It includes our rules engine for distributed network packet filtering and security monitoring, federated roots, and many other architectural and UI improvements and bug fixes.

## New Features in 1.2.0

### The ZeroTier Rules Engine

The largest new feature in 1.2.0, and the product of many months of work, is our advanced network rules engine. With this release we achieve traffic control, security monitoring, and micro-segmentation capability on par with many enterprise SDN solutions designed for use in advanced data centers and corporate networks.

Rules allow you to filter packets on your network and vector traffic to security observers. Security observation can be performed in-band using REDIRECT or out of band using TEE.

Tags and capabilites provide advanced methods for implementing fine grained permission structures and micro-segmentation schemes without bloating the size and complexity of your rules table.

See the [rules engine announcement blog post](https://www.zerotier.com/blog/?p=927) for an in-depth discussion of theory and implementation. The [manual](https://www.zerotier.com/manual.shtml) contains detailed information on rule, tag, and capability use, and the `rule-compiler/` subfolder of the ZeroTier source tree contains a JavaScript function to compile rules in our human-readable rule definition language into rules suitable for import into a network controller. (ZeroTier Central uses this same script to compile rules on [my.zerotier.com](https://my.zerotier.com/).)

### Root Server Federation

It's now possible to create your own root servers and add them to the root server pool on your nodes. This is done by creating what's called a "moon," which is a signed enumeration of root servers and their stable points on the network. Refer to the [manual](https://www.zerotier.com/manual.shtml) for instructions.

Federated roots achieve a number of things:

 * You can deploy your own infrastructure to reduce dependency on ours.
 * You can deploy roots *inside your LAN* to ensure that network connectivity inside your facility still works if the Internet goes down. This is the first step toward making ZeroTier viable as an in-house SDN solution.
 * Roots can be deployed inside national boundaries for countries with data residency laws or "great firewalls." (As of 1.2.0 there is still no way to force all traffic to use these roots, but that will be easy to do in a later version.)
 * Last but not least this makes ZeroTier somewhat less centralized by eliminating any hard dependency on ZeroTier, Inc.'s infrastructure.

Our roots will of course remain and continue to provide zero-configuration instant-on deployment, a secure global authority for identities, and free traffic relaying for those who can't establish peer to peer connections.

### Local Configuration

An element of our design philosophy is "features are bugs." This isn't an absolute dogma but more of a guiding principle. We try as hard as we can to avoid adding features, especially "knobs" that must be tweaked by a user.

As of 1.2.0 we've decided that certain knobs are unavoidable, and so there is now a `local.conf` file that can be used to configure them. See the ZeroTier One documentation for these. They include:

 * Blacklisting interfaces you want to make sure ZeroTier doesn't use for network traffic, such as VPNs, slow links, or backplanes designated for only certain kinds of traffic.
 * Turning uPnP/NAT-PMP on or off.
 * Configuring software updates on Windows and Mac platforms.
 * Defining trusted paths (the old trusted paths file is now deprecated)
 * Setting the ZeroTier main port so it doesn't have to be changed on the command line, which is very inconvenient in many cases.

### Improved In-Band Software Updates

A good software update system for Windows and Mac clients has been a missing feature in previous versions. It does exist but we've been shy about using it so far due to its fragility in some environments.

We've greatly improved this mechanism in 1.2.0. Not only does it now do a better job of actually invoking the update, but it also transfers updates in-band using the ZeroTier protocol. This means it can work in environments that do not allows http/https traffic or that force it through proxies. There's also now an update channel setting: `beta` or `release` (the default).

Software updates are authenticated three ways:

 1. ZeroTier's own signing key is used to sign all updates and this signature is checked prior to installation. ZeroTier, Inc.'s signatures are performed on an air-gapped machine.

 2. Updates for Mac and Windows are signed using Apple and Microsoft (DigiCert EV) keys and will not install unless these signatures are also valid.

 3. The new in-band update mechanism also authenticates the source of the update via ZeroTier's built-in security features. This provides transport security, while 1 and 2 provide security of the update at rest.

Updates are now configurable via `local.conf`. There are three options: `disable`, `download`, and `apply`. The third (apply) is the default for official builds on Windows and Mac, making updates happen silently and automatically as they do for popular browsers like Chrome and Firefox. Updates are disabled by default on Linux and other Unix-type systems as these are typically updated through package managers.

### Path Link Quality Awareness

Version 1.2.0 is now aware of the link quality of direct paths with other 1.2.0 nodes. This information isn't used yet but is visible through the JSON API. (Quality always shows as 100% with pre-1.2.0 nodes.) Quality is measured passively with no additional overhead using a counter based packet loss detection algorithm.

This information is visible from the command line via `listpeers`:

    200 listpeers XXXXXXXXXX 199.XXX.XXX.XXX/9993;10574;15250;1.00 48 1.2.0 LEAF
    200 listpeers XXXXXXXXXX 195.XXX.XXX.XXX/45584;467;7608;0.44 290 1.2.0 LEAF

The first peer's path is at 100% (1.00), while the second peer's path is suffering quite a bit of packet loss (0.44).

Link quality awareness is a precursor to intelligent multi-path and QoS support, which will in future versions bring us to feature parity with SD-WAN products like Cisco iWAN.

### Security Improvements

Version 1.2.0 adds anti-DOS (denial of service) rate limits and other hardening for improved resiliency against a number of denial of service attack scenarios.

It also adds a mechanism for instantaneous credential revocation. This can be used to revoke certificates of membership instantly to kick a node off a network (for private networks) and also to revoke capabilities and tags. The new controller sends revocations by default when a peer is de-authorized.

Revocations propagate using a "rumor mill" peer to peer algorithm. This means that a controller need only successfully send a revocation to at least one member of a network with connections to other active members. At this point the revocation will flood through the network peer to peer very quickly. This helps make revocations more robust in the face of poor connectivity with the controller or attempts to incapacitate the controller with denial of service attacks, as well as making revocations faster on huge networks.

### Windows and Macintosh UI Improvements (ZeroTier One)

The Mac has a whole new UI built natively in Objective-C. It provides a pulldown similar in appearance and operation to the Mac WiFi task bar menu.

The Windows UI has also been improved and now provides a task bar icon that can be right-clicked to manage networks. Both now expose managed route and IP permissions, allowing nodes to easily opt in to full tunnel operation if you have a router configured on your network.

### Ad-Hoc Networks

A special kind of public network called an ad-hoc network may be accessed by joining a network ID with the format:

    ffSSSSEEEE000000
    | |   |   |
    | |   |   Reserved for future use, must be 0
    | |   End of port range (hex)
    | Start of port range (hex)
    Reserved ZeroTier address prefix indicating a controller-less network

Ad-hoc networks are public (no access control) networks that have no network controller. Instead their configuration and other credentials are generated locally. Ad-hoc networks permit only IPv6 UDP and TCP unicast traffic (no multicast or broadcast) using 6plane format NDP-emulated IPv6 addresses. In addition an ad-hoc network ID encodes an IP port range. UDP packets and TCP SYN (connection open) packets are only allowed to desintation ports within the encoded range.

For example `ff00160016000000` is an ad-hoc network allowing only SSH, while `ff0000ffff000000` is an ad-hoc network allowing any UDP or TCP port.

Keep in mind that these networks are public and anyone in the entire world can join them. Care must be taken to avoid exposing vulnerable services or sharing unwanted files or other resources.

### Network Controller (Partial) Rewrite

The network controller has been largely rewritten to use a simple in-filesystem JSON data store in place of SQLite, and it is now included by default in all Windows, Mac, Linux, and BSD builds. This means any desktop or server node running ZeroTier One can now be a controller with no recompilation needed.

If you have data in an old SQLite3 controller we've included a NodeJS script in `controller/migrate-sqlite` to migrate data to the new format. If you don't migrate, members will start getting `NOT_FOUND` when they attempt to query for updates.

## Major Bug Fixes in 1.2.0

 * **The Windows HyperV 100% CPU bug is FINALLY DEAD**: This long-running problem turns out to have been an issue with Windows itself, but one we were triggering by placing invalid data into the Windows registry. Microsoft is aware of the issue but we've also fixed the triggering problem on our side. ZeroTier should now co-exist quite well with HyperV and should now be able to be bridged with a HyperV virtual switch.
 * **Segmenation faults on musl-libc based Linux systems**: Alpine Linux and some embedded Linux systems that use musl libc (a minimal libc) experienced segmentation faults. These were due to a smaller default stack size. A work-around that sets the stack size for new threads has been added.
 * **Windows firewall blocks local JSON API**: On some Windows systems the firewall likes to block 127.0.0.1:9993 for mysterious reasons. This is now fixed in the installer via the addition of another firewall exemption rule.
 * **UI crash on embedded Windows due to missing fonts**: The MSI installer now ships fonts and will install them if they are not present, so this should be fixed.

## Other Improvements in 1.2.0

 * **Improved dead path detection**: ZeroTier is now more aggressive about expiring paths that do not seem to be active. If a path seems marginal it is re-confirmed before re-use.
 * **Minor performance improvements**: We've reduced unnecessary memcpy's and made a few other performance improvements in the core.
 * **Linux static binaries**: For our official packages (the ones in the download.zerotier.com apt and yum repositories) we now build Linux binaries with static linking. Hopefully this will stop all the bug reports relating to library inconsistencies, as well as allowing our deb packages to run on a wider variety of Debian-based distributions. (There are far too many of these to support officially!) The overhead for this is very small, especially since we built our static versions against musl-libc. Distribution maintainers are of course free to build dynamically linked versions for inclusion into distributions; this only affects our official binaries.
