# Wireshark — Study Notes

---

## What Is Wireshark?

Open-source, cross-platform network packet analyser. Used for:
- Detecting and troubleshooting network problems
- Detecting security anomalies (rogue hosts, abnormal port usage, suspicious traffic)
- Investigating protocol details, response codes, and payload data

**Not an IDS** — it reads packets, doesn't detect intrusions automatically. Analysis depends entirely on the analyst's knowledge.

---

## GUI Layout

| Section | Purpose |
|---------|---------|
| Toolbar | Menus and shortcuts for sniffing, filtering, sorting, exporting, merging |
| Display Filter Bar | Main query/filtering input |
| Recent Files | Previously opened captures — double-click to reopen |
| Capture Filter & Interfaces | Available network interfaces and capture filters |
| Status Bar | Tool status, profile, packet count |

---

## Packet Panes (when a capture is open)

| Pane | Shows |
|------|-------|
| Packet List | Summary of each packet — source/dest address, protocol, info |
| Packet Details | Full protocol breakdown of selected packet (expandable layers) |
| Packet Bytes | Hex + ASCII representation of selected packet |

Click anything in Packet Details → highlights the corresponding bytes in Packet Bytes pane.

---

## Packet Layers in Wireshark

Wireshark breaks packets into up to 7 layers matching the OSI model:

| Layer | Wireshark Label | What It Shows |
|-------|----------------|---------------|
| 1 | Frame | Physical layer info — frame number, capture details |
| 2 | Source [MAC] | Source and destination MAC addresses |
| 3 | Source [IP] | Source and destination IP addresses |
| 4 | Protocol | TCP/UDP details — ports, flags, sequence numbers |
| 4 (cont.) | Protocol Errors | TCP reassembly segments |
| 5 | Application Protocol | HTTP, FTP, SMB, etc. specifics |
| 5 (cont.) | Application Data | Actual application payload |

---

## Packet Colouring

Wireshark colours packets by protocol/condition to spot anomalies at a glance.

**Two types:**
- **Temporary** — lasts current session only (right-click → Conversation Filter)
- **Permanent** — saved to profile (View → Coloring Rules)

Toggle colouring: **View → Colourise Packet List**

---

## Expert Info

Wireshark flags protocol anomalies automatically. View via: **Analyse → Expert Information** or the lower-left status bar icon.

| Severity | Colour | Meaning |
|----------|--------|---------|
| Chat | Blue | Normal workflow info |
| Note | Cyan | Notable events (app error codes) |
| Warn | Yellow | Unusual errors or problem statements |
| Error | Red | Malformed packets |

Common groups: Checksum errors, Deprecated protocol usage, Packet comments, Malformed packets.

---

## Working with Captures

### Loading Files
File menu / drag and drop / double-click the file

### Traffic Sniffing (live capture)
- 🔵 Blue shark button → Start
- 🔴 Red button → Stop
- 🟢 Green button → Restart

### Merging PCAP Files
**File → Merge** — combines two captures into one. Must save the merged file before working with it.

### View File Details
**Statistics → Capture File Properties** — shows file hash, capture time, interface, statistics.

### Time Display Format
Default: Seconds Since Beginning of Capture.  
Recommended: **View → Time Display Format → UTC**

---

## Navigating Packets

| Action | How |
|--------|-----|
| Go to specific packet number | Go menu or toolbar |
| Find packet by content | Edit → Find Packet |
| Mark a packet | Edit or right-click → Mark/Unmark |
| Add a comment to a packet | Right-click → Packet Comment |
| Export specific packets | File → Export Specified Packets |
| Export transferred files | File → Export Objects (HTTP, SMB, FTP, TFTP, DICOM) |

**Find Packet input types:** Display filter / Hex / String / Regex  
**Find Packet search fields:** Packet list / Packet details / Packet bytes — choose the right one or it won't find anything.

**Marked packets** show in black regardless of colour rules. Lost when the file is closed.  
**Packet comments** persist in the capture file until removed.

---

## Filtering

Two types:
- **Capture filters** — applied before capture; only captures matching packets
- **Display filters** — applied to an existing capture; hides non-matching packets

### Display Filter Methods

| Method | What it does |
|--------|-------------|
| Apply as Filter | Right-click a field value → filters to show only that value |
| Conversation Filter | Right-click → shows all packets for that IP/port conversation |
| Colourise Conversation | Highlights related packets without hiding others |
| Prepare as Filter | Builds the filter query but doesn't apply it yet — lets you chain conditions |
| Apply as Column | Adds a field as a visible column in the packet list |
| Follow Stream | Reconstructs full application-layer stream (TCP/UDP/HTTP) |

**Golden rule:** "If you can click on it, you can filter and copy it."

---

## Common Display Filters

### By Protocol Name
```
http
arp
dhcp
ftp
smtp
pop
imap
dns
tcp
udp
```

### By Port Number
```
tcp.port == 80
tcp.port == 443
udp.port == 53
```

### By IP Address
```
ip.addr == 192.168.1.2
ip.src == 192.168.1.2
ip.dst == 192.168.1.2
```

### Combining Filters
```
ip.addr == 192.168.1.2 and tcp.port == 80
http or dns
!(arp)
```

---

## Follow Stream

Reconstructs the full conversation at the application layer — shows raw data as it appeared to the application, including plaintext credentials if the protocol is unencrypted.

**Right-click → Follow → TCP/UDP/HTTP Stream**

- Client traffic: **red**
- Server traffic: **blue**

Wireshark auto-applies a display filter when you follow a stream. Click the **X** on the filter bar to clear it and return to all packets.

---

## Notes / Things to Add

*Use this space as you pick up new stuff — Packet Operations room content goes here:*

- Link to github for detailed dissecting packets https://github.com/boundary/wireshark/blob/master/doc/README.dissector
-
-
