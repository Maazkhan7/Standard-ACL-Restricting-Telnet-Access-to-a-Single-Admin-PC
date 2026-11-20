<div align="center">

# 🔑 Standard ACL: Restricting Telnet Access to a Single Admin PC

### ACL Series · Lab 3 · Cisco Packet Tracer

![Cisco](https://img.shields.io/badge/Cisco-IOS-1BA0D7?style=for-the-badge&logo=cisco&logoColor=white)
![Packet Tracer](https://img.shields.io/badge/Packet%20Tracer-Lab-049fd9?style=for-the-badge)
![ACL](https://img.shields.io/badge/Standard-ACL-red?style=for-the-badge)
![VTY](https://img.shields.io/badge/VTY-access--class-purple?style=for-the-badge)
![Telnet](https://img.shields.io/badge/Telnet-Restricted-orange?style=for-the-badge)
![Level](https://img.shields.io/badge/Level-CCNA-green?style=for-the-badge)
![Status](https://img.shields.io/badge/Status-Verified-success?style=for-the-badge)

*Only the admin PC may manage the router. Everyone else is refused.*

</div>

---

## 📑 Table of Contents
1. [Overview](#-overview)
2. [Topology](#-topology)
3. [Addressing Table](#-addressing-table)
4. [Series Comparison](#-series-comparison)
5. [Configuration](#-configuration)
6. [Verification: Stage by Stage](#-verification-stage-by-stage)
7. [Test Matrix](#-test-matrix)
8. [Why access-class and Not access-group](#-why-access-class-and-not-access-group)
9. [Key Takeaways](#-key-takeaways)
10. [Production Hardening Notes](#-production-hardening-notes)
11. [Troubleshooting Notes](#-troubleshooting-notes)
12. [Next Labs](#-next-labs-in-the-acl-series)
13. [Author](#-author)

---

## 🎯 Overview

In the earlier labs the ACL filtered traffic **passing through** a router. In this lab the ACL protects the **router itself**.

1. Enable **Telnet** on R1 through the VTY lines.
2. **Prove the risk:** before any ACL, every PC (LAN and WAN) can log in to R1.
3. Create a **Standard ACL** that permits only the admin PC `192.168.1.3`.
4. Apply it to the VTY lines with `access-class`.
5. **Prove the fix:** only the admin PC gets in, and ping still works for everyone.

| Item | Detail |
|---|---|
| ACL type | Standard, numbered (ACL 5) |
| Match method | `host` keyword (exact single source) |
| Applied on | R1 **VTY lines 0–4** |
| Command | `access-class 5 in` |
| Protected service | Telnet to R1 (1.1.1.1) |
| Only permitted host | **192.168.1.3** (admin PC) |

---

## 🗺 Topology

```
                         1.0.0.0 (Serial WAN)
        1.1.1.1  ●━━━━━━━━━━━━━━⚡━━━━━━━━━━━━━━●  1.1.1.2
              ┌──────┐                          ┌──────┐
              │  R1  │ ◄── Telnet target        │  R2  │
              │ 1941 │     VTY 0-4 + ACL 5      │ 1941 │
              └──┬───┘                          └──┬───┘
        192.168.1.1/24                       200.100.100.1/24
                 │                                 │
            ┌────┴────┐                       ┌────┴────┐
            │ Switch0 │                       │ Switch1 │
            └─┬──┬──┬─┘                       └─┬──┬──┬─┘
              │  │  │                           │  │  │
           PC-PT PC-PT PC-PT                  PC-PT PC-PT Server-PT
        192.168.1.2 .1.3  .1.4             200.100.100.2 .100.3  .100.4
        (🚫)     (✅ ADMIN) (🚫)            (🚫 WAN)
```

---

## 📋 Addressing Table

| Device | Interface | IP Address | Role |
|---|---|---|---|
| R1 (1941) | LAN interface | 192.168.1.1/24 | Gateway, LAN 1 |
| R1 (1941) | Serial | **1.1.1.1** | WAN link, Telnet target |
| R2 (1941) | Serial | 1.1.1.2 | WAN link |
| R2 (1941) | LAN interface | 200.100.100.1/24 | Gateway, LAN 2 |
| PC0 | NIC | 192.168.1.2/24 | LAN user, 🚫 denied |
| PC1 | NIC | **192.168.1.3/24** | ✅ **Admin PC** |
| PC2 | NIC | 192.168.1.4/24 | LAN user, 🚫 denied |
| PC3 | NIC | 200.100.100.2/24 | WAN user, 🚫 denied |
| PC | NIC | 200.100.100.3/24 | LAN 2 |
| Server-PT | NIC | 200.100.100.4/24 | LAN 2 |

---

## 🔄 Series Comparison

| | Lab 1 | Lab 2 | Lab 3 (this lab) |
|---|---|---|---|
| Goal | Allow one host through R1 | Block one host through R2 | Allow one host to **manage R1** |
| Filter style | Whitelist | Blacklist | Whitelist |
| Applied with | `ip access-group 1 in` | `ip access-group 1 out` | **`access-class 5 in`** |
| Applied to | Interface Se0/0/0 | Interface Gi0/0 | **VTY lines** |
| Traffic affected | Traffic through the router | Traffic through the router | **Traffic to the router only** |

---

## ⚙️ Configuration

### Step 1: Enable Telnet on R1

```
R1(config)# line vty 0 4
R1(config-line)# password cisco
R1(config-line)# login
R1(config-line)# transport input telnet
R1(config-line)# exit
R1# write memory
```

> The enable password was configured earlier (`R1>en` asks for it in the outputs). Add yours here if you publish the full config.

### Step 2: Create the Standard ACL

```
R1(config)# access-list 5 permit host 192.168.1.3
R1(config)# access-list 5 deny any
```

### Step 3: Apply it to the VTY lines

```
R1(config)# line vty 0 4
R1(config-line)# access-class 5 in
R1(config-line)# end
R1# write memory
```

### Config breakdown

| Command | Purpose |
|---|---|
| `line vty 0 4` | Select the five virtual terminal lines used by Telnet/SSH |
| `password cisco` + `login` | Require a password at the VTY prompt |
| `transport input telnet` | Allow Telnet on the VTY lines |
| `access-list 5 permit host 192.168.1.3` | Allow only the admin PC |
| `access-list 5 deny any` | Explicit deny for everyone else |
| `access-class 5 in` | Check the **source** of every incoming VTY session against ACL 5 |

---

## ✅ Verification: Stage by Stage

### 🟡 Phase A: Before the ACL (everyone can log in)

#### Stage 1: PC0 (192.168.1.2) → Telnet R1 ✅

```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...Open

User Access Verification

Password:
R1>en
Password:
R1#
```
**Annotation:** Telnet works and a LAN user reaches privileged mode, **a security risk**.

#### Stage 2: PC1 (192.168.1.3) → Telnet R1 ✅
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...Open
Password:
R1>en
Password:
R1#
```

#### Stage 3: PC2 (192.168.1.4) → Telnet R1 ✅
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...Open
Password:
R1>en
Password:
R1#
```

#### Stage 4: PC3 (200.100.100.2, **WAN network**) → Telnet R1 ✅
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...Open
Password:
R1>en
Password:
R1#
```
**Annotation:** Even a host from the **remote network** can manage R1. This is the problem we fix.

---

### 🔧 Phase B: Apply the ACL

#### Stage 5: Create and apply ACL 5 on R1

```
R1(config)# access-list 5 permit host 192.168.1.3
R1(config)# access-list 5 deny any
R1(config)# do wr
Building configuration...
[OK]

R1(config)# line vty 0 4
R1(config-line)# access-class 5 in
R1(config-line)# do wr
Building configuration...
[OK]
```

**Annotation:** The first `access-class` attempt returned `% Incomplete command` because the direction keyword was missing. `access-class 5 in` is the correct form.

---

### 🟢 Phase C: After the ACL

#### Stage 6: PC1 (192.168.1.3, admin) → Telnet R1 → ALLOWED ✅
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...Open

User Access Verification

Password:
R1>en
Password:
R1#
R1#show user
```
**Annotation:** The source `192.168.1.3` matches line 10 (`permit host`), so the session opens normally.

#### Stage 7: PC0 (192.168.1.2) → Telnet R1 → REFUSED ❌
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...
% Connection refused by remote host
```
**Annotation (before vs after):** The same PC that logged in at Stage 1 is now refused. The source `192.168.1.2` falls to `deny any`.

#### Stage 8: PC3 (200.100.100.2, WAN) → Telnet R1 → REFUSED ❌
```
C:\> telnet 1.1.1.1
Trying 1.1.1.1 ...
% Connection refused by remote host
```
**Annotation:** The WAN PC could log in at Stage 4 and is now blocked. The ACL works for remote sources too, not just LAN hosts.

#### Stage 9: ACL hit counters (run on R1)
```
R1# show access-lists
Standard IP access list 5
    10 permit host 192.168.1.3 (2 match(es))
    20 deny any (4 match(es))
```
**Annotation:** The counters prove the ACL is being hit. The permit line is matched by the admin PC's sessions, and `deny any` by the refused attempts.

#### Stage 10: Ping still works from every PC ✅
```
C:\> ping 1.1.1.1
Reply from 1.1.1.1: bytes=32 time<1ms TTL=255   (x4)
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)

C:\> ping 1.1.1.2
Reply from 1.1.1.2: bytes=32 time=15ms TTL=254
Reply from 1.1.1.2: bytes=32 time=7ms  TTL=254
Reply from 1.1.1.2: bytes=32 time=8ms  TTL=254
Reply from 1.1.1.2: bytes=32 time=1ms  TTL=254
Packets: Sent = 4, Received = 4, Lost = 0 (0% loss)
Minimum = 1ms, Maximum = 15ms, Average = 7ms
```
**Annotations**
- Only **Telnet to the router** is restricted. ICMP and all other traffic are untouched.
- **TTL 255** is a Cisco router replying directly. **TTL 254** is R2 (one router hop further, so one decrement).

---

## 🧪 Test Matrix

| # | Source | Test | Before ACL | After ACL |
|---|---|---|---|---|
| 1 | 192.168.1.2 (LAN) | Telnet 1.1.1.1 | ✅ R1# | ❌ Connection refused |
| 2 | **192.168.1.3 (admin)** | Telnet 1.1.1.1 | ✅ R1# | ✅ **R1#** |
| 3 | 192.168.1.4 (LAN) | Telnet 1.1.1.1 | ✅ R1# | ❌ Expected (matches `deny any`) |
| 4 | 200.100.100.2 (WAN) | Telnet 1.1.1.1 | ✅ R1# | ❌ Connection refused |
| 5 | 192.168.1.2 | Ping 1.1.1.1 / 1.1.1.2 | ✅ | ✅ |

> Row 3 (after) follows from the ACL logic. Add a screenshot of 192.168.1.4 being refused to confirm it.

---

## 🧠 Why access-class and Not access-group

| | `ip access-group` | `access-class` |
|---|---|---|
| Applied to | An **interface** | A **VTY line** |
| Filters | Traffic **passing through** the router | Sessions **to the router itself** |
| Affects ping / routed traffic | Yes | **No** |
| Direction `in` means | Packets entering the interface | Incoming Telnet/SSH sessions |

```
 Any PC ──Telnet──► R1 VTY line ──► ACL 5 ──► permit host 192.168.1.3 ✅
                                          └─► deny any                🚫
 Any PC ──Ping / data ─► R1 interface ──► (no ACL here) ──► normal forwarding
```

This also follows the placement rule from Lab 2. A Standard ACL goes **close to the destination**, and here the destination is the router itself.

---

## 💡 Key Takeaways

| # | Takeaway |
|---|---|
| 1 | Standard ACLs are ideal for "**who may manage this device**" because the source is all that matters |
| 2 | VTY lines use `access-class`, and interfaces use `access-group` |
| 3 | A VTY ACL only affects sessions to the router, so ping and transit traffic are not touched |
| 4 | Without an ACL, **any** reachable host can try to log in, including WAN hosts |
| 5 | `% Incomplete command` on `access-class 5` means the direction (`in`) is missing |
| 6 | `show access-lists` hit counters are proof that the ACL is working |
| 7 | A before/after test from several sources proves the change |
| 8 | The admin PC should have a **fixed IP**, or the ACL will lock the admin out |

---

## 🛡 Production Hardening Notes

This is a lab, so passwords are simple. In a real network:

| Weakness in lab | Real-world improvement |
|---|---|
| Telnet sends everything, **including passwords, in clear text** | Use **SSH** (`transport input ssh`) |
| Shared line password | Use `login local` with named users, or AAA |
| Weak password | Strong, unique passwords and `enable secret` |
| ACL only | Add `exec-timeout` and logging (`access-list 5 deny any log`) |

The same `access-class 5 in` protects SSH too, so the ACL from this lab carries over unchanged.

---

## 🛠 Troubleshooting Notes

| Symptom | Check |
|---|---|
| `% Incomplete command` | Missing direction: use `access-class 5 in` |
| Admin PC is refused too | Is its IP exactly 192.168.1.3? Is the permit line first? |
| Everyone can still log in | Is `access-class` applied to the correct VTY range (0 4)? Did you save? |
| Nobody can log in | Is the permit line missing or mistyped? Check `show access-lists` |
| Telnet fails before the ACL | Check `transport input telnet`, `login`, and a password on the line |

---

## 🚀 Next Labs in the ACL Series

- [x] Lab 1: Standard ACL, whitelist a host ✅
- [x] Lab 2: Standard ACL, blacklist a host with placement logic ✅
- [x] Lab 3: Standard ACL, restrict Telnet (VTY) to one admin PC ✅
- [ ] Standard ACL with network / wildcard mask ranges
- [ ] Named Standard ACL (edit with sequence numbers)
- [ ] Extended ACL
- [ ] SSH instead of Telnet with the same ACL

---

## 👨‍💻 Author

**Maaz Khan**
CCNA-certified Network & NOC Engineer · Lower Dir, KPK, Pakistan

[![LinkedIn](https://img.shields.io/badge/LinkedIn-maazkhanms-0A66C2?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/maazkhanms)
[![GitHub](https://img.shields.io/badge/GitHub-maazkhanms-181717?style=for-the-badge&logo=github)](https://github.com/maazkhanms)

⭐ If this lab helped you, give the repo a star!
