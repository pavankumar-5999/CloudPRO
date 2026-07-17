# CIDR & Subnet Calculations — A Plain-English Guide

This guide walks through subnetting step by step, assuming no prior networking background.

---

## 0. The Basics You Need First

An IPv4 address (like `192.168.10.21`) has **4 numbers separated by dots**, called **octets**.
Each octet is 8 bits, so the whole address is 32 bits.

A **CIDR** like `/29` means: *"the first 29 bits of this address are the network part; the rest are for hosts (devices)."*

That's it — everything below is just working out the consequences of that one fact.

---

## 1. Subnet Mask — What Does /29 Actually Look Like?

The subnet mask is what marks which bits belong to the network vs. the host.

**Shortcut table** — memorize this, it covers almost everything you'll ever need:

| CIDR | Host bits left | Mask value (in the relevant octet) | Full Subnet Mask |
|------|-----------------|--------------------------------------|-------------------|
| /24  | 8               | 0                                    | 255.255.255.0     |
| /25  | 7               | 128                                  | 255.255.255.128   |
| /26  | 6               | 192                                  | 255.255.255.192   |
| /27  | 5               | 224                                  | 255.255.255.224   |
| /28  | 4               | 240                                  | 255.255.255.240   |
| /29  | 3               | 248                                  | 255.255.255.248   |
| /30  | 2               | 252                                  | 255.255.255.252   |


**Where does this number come from?**
`Mask value = 256 − 2^(host bits)`
Example: /29 has 3 host bits → 256 − 2³ = 256 − 8 = **248**

---

## 2. Block Size — The Most Important Number

**Block size = 256 − Mask value**

This tells you how far apart each subnet is.

| CIDR | Mask value | Block Size |
|------|------------|------------|
| /25  | 128        | 128        |
| /26  | 192        | 64         |
| /27  | 224        | 32         |
| /28  | 240        | 16         |
| /29  | 248        | 8          |
| /30  | 252        | 4          |

**Which octet do you apply the block size to?**
- CIDR is **24 or less** → work in the **3rd octet**
- CIDR is **more than 24** (25–30) → work in the **4th octet**

---

## 3. The 4-Step Method (memorize this order)

Given an IP + CIDR, you calculate things in this order:

1. **Network ID** — round the relevant octet *down* to the nearest multiple of the block size
2. **Broadcast Address** — Network ID + Block Size − 1
3. **First Usable Host** — Network ID + 1
4. **Last Usable Host** — Broadcast − 1

---

## 4. Full Worked Example

**IP: `192.168.10.21` — CIDR: `/29`**

| Step | Calculation | Result |
|------|-------------|--------|
| Mask octet | 256 − 2³ | 248 |
| Block size | 256 − 248 | 8 |
| Relevant octet | /29 > 24 → | 4th octet |
| List blocks in 4th octet | 0, 8, 16, 24, 32... | — |
| 21 falls between 16 and 24 → Network ID | | **192.168.10.16** |
| Broadcast | 16 + 8 − 1 | **192.168.10.23** |
| First usable host | 16 + 1 | **192.168.10.17** |
| Last usable host | 23 − 1 | **192.168.10.22** |
| Usable hosts | 8 − 2 | **6** |

---

## 5. Quick Reference Table (Usable Hosts Included)

| CIDR | Subnet Mask       | Block Size | Usable Hosts |
|------|-------------------|------------|--------------|
| /24  | 255.255.255.0     | 256        | 254          |
| /25  | 255.255.255.128   | 128        | 126          |
| /26  | 255.255.255.192   | 64         | 62           |
| /27  | 255.255.255.224   | 32         | 30           |
| /28  | 255.255.255.240   | 16         | 14           |
| /29  | 255.255.255.248   | 8          | 6            |
| /30  | 255.255.255.252   | 4          | 2            |

> Usable hosts = Block Size − 2 (you always lose 1 to the Network ID and 1 to the Broadcast address)

---

## 6. Practice Examples (with answers)

| IP                | CIDR | Network ID       | Broadcast         | First Host        | Last Host          |
|-------------------|------|------------------|--------------------|--------------------|----------------------|
| 192.168.10.21     | /29  | 192.168.10.16    | 192.168.10.23     | 192.168.10.17      | 192.168.10.22        |
| 10.1.5.138        | /29  | 10.1.5.136       | 10.1.5.143         | 10.1.5.137         | 10.1.5.142           |
| 192.168.85.140    | /26  | 192.168.85.128   | 192.168.85.191     | 192.168.85.129     | 192.168.85.190        |
| 192.168.200.75    | /28  | 192.168.200.64   | 192.168.200.79     | 192.168.200.65     | 192.168.200.78        |
| 192.168.10.21     | /30  | 192.168.10.20    | 192.168.10.23     | 192.168.10.21      | 192.168.10.22         |

---

## 7. The One Trick That Ties It All Together

Mask value and Block size always pair up like this (they add to 256):

```
128 ↔ 128
192 ↔ 64
224 ↔ 32
240 ↔ 16
248 ↔ 8
252 ↔ 4
```

Memorize **one side**, and you can always get the other by subtracting from 256.

---

## 8. Cheat Sheet Summary

1. Find host bits → get mask value → 256 − 2^(host bits)
2. Block size = 256 − mask value
3. CIDR ≤ 24 → 3rd octet | CIDR > 24 → 4th octet
4. Network ID = round IP's relevant octet **down** to nearest block size multiple
5. Broadcast = Network ID + Block Size − 1
6. First host = Network ID + 1
7. Last host = Broadcast − 1
8. Usable hosts = Block Size − 2
