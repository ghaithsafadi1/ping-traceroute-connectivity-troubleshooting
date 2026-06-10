# Lab: Use Ping and Traceroute to Test Network Connectivity

**Tool:** Cisco Packet Tracer  
**Topics:** ICMP, IPv4/IPv6 troubleshooting, ping, traceroute  
**Status:** ✅ Completed

---

## Scenario

A multi-router network with connectivity issues across both IPv4 and IPv6. The objective is to identify the root cause of failed pings using `ping` and `traceroute`, implement a fix, and verify end-to-end connectivity is restored.

---

## Topology

```
PC1 ── R1 (G0/1) ── R1 (S0/0/1) ── R2 (S0/0/0)
                                         |
PC3 ── R3 (G0/1) ── R3 (S0/0/1) ── R2 (S0/0/1)

PC2 ── R1 (G0/0) [IPv6]
PC4 ── R3 (G0/0) [IPv6]
```

---

## Addressing Table

| Device | Interface | IP Address / Prefix | Subnet Mask | Default Gateway |
|--------|-----------|----------------------|-------------|-----------------|
| R1 | G0/0 | 2001:db8:1:1::1/64 | — | N/A |
| R1 | G0/1 | 10.10.1.97 | 255.255.255.224 | N/A |
| R1 | S0/0/1 | 10.10.1.6 | 255.255.255.252 | N/A |
| R1 | S0/0/1 | 2001:db8:1:2::2/64 | — | N/A |
| R1 | S0/0/1 | fe80::1 (link-local) | — | N/A |
| R2 | S0/0/0 | 10.10.1.5 | 255.255.255.252 | N/A |
| R2 | S0/0/0 | 2001:db8:1:2::1/64 | — | N/A |
| R2 | S0/0/1 | 10.10.1.9 | 255.255.255.252 | N/A |
| R2 | S0/0/1 | 2001:db8:1:3::1/64 | — | N/A |
| R2 | S0/0/1 | fe80::2 (link-local) | — | N/A |
| R3 | G0/0 | 2001:db8:1:4::1/64 | — | N/A |
| R3 | G0/1 | 10.10.1.17 | 255.255.255.240 | N/A |
| R3 | S0/0/1 | 10.10.1.10 | 255.255.255.252 | N/A |
| R3 | S0/0/1 | 2001:db8:1:3::2/64 | — | N/A |
| R3 | S0/0/1 | fe80::3 (link-local) | — | N/A |
| PC1 | NIC | *(collected via ipconfig)* | — | — |
| PC2 | NIC | *(collected via ipv6config)* | — | — |
| PC3 | NIC | *(collected via ipconfig)* | — | — |
| PC4 | NIC | *(collected via ipv6config)* | — | — |

---

## Part 1: IPv4 Troubleshooting

### Step 1 — Gather Information

Used `ipconfig /all` on PC1 and PC3 to collect IPv4 addresses, subnet masks, and default gateways.

### Step 2 — Locate the Failure

Ran `tracert` from PC1 to PC3 and from PC3 to PC1.

- Trace from PC1 stopped at R1's Serial interface — packets could not reach R2
- Ran `show ip interface brief` and `show ip route` on R1, R2, and R3 to compare configured addresses against the addressing table

**Root cause:** Misconfigured IP address on one of the Serial interfaces (R1 S0/0/1 or R2 S0/0/0) — the addresses on the point-to-point link did not match, breaking routing between R1 and R2.

### Step 3 — Solution

Correct the IP address on the misconfigured Serial interface to match the addressing table.

### Step 4 — Implementation

```ios
R1(config)# interface s0/0/1
R1(config-if)# ip address 10.10.1.6 255.255.255.252
R1(config-if)# no shutdown
```

*(Adjust based on which router had the misconfigured address in your instance of the lab.)*

### Step 5 — Verification

```
PC1> ping 10.10.1.17     ✅ Success
PC3> ping 10.10.1.97     ✅ Success
```

---

## Part 2: IPv6 Troubleshooting

### Step 1 — Gather Information

Used `ipv6config /all` on PC2 and PC4 to collect IPv6 addresses, prefixes, and default gateways.

### Step 2 — Locate the Failure

Ran `tracert` from PC2 to PC4 and from PC4 to PC2.

- Ran `show ipv6 interface brief` on R3
- The link-local address configured on R3 S0/0/1 did not match the gateway address recorded on PC4

**Root cause:** Wrong link-local address configured on R3's interface — PC4's default gateway pointed to `fe80::3` but R3 had a different link-local address configured.

### Step 3 — Solution

Correct the link-local address on R3 S0/0/1 to match `fe80::3`.

### Step 4 — Implementation

```ios
R3(config)# interface s0/0/1
R3(config-if)# ipv6 address fe80::3 link-local
R3(config-if)# no shutdown
```

### Step 5 — Verification

```
PC2> ping 2001:db8:1:4::    ✅ Success
PC4> ping 2001:db8:1:1::    ✅ Success
```

---

## Key Concepts Practiced

- Using `ping` and `traceroute` / `tracert` to isolate connectivity failures
- Reading `show ip interface brief` and `show ip route` output
- Point-to-point Serial link addressing (IPv4)
- IPv6 link-local address configuration and its role as a default gateway
- Methodical troubleshooting: document → test → isolate → fix → verify

---

## Part of My CCNA Study Portfolio

This lab is one in a series of Packet Tracer exercises I'm completing as I work toward the CCNA certification.

→ [View full portfolio](../README.md)
