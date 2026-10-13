# BMP Containerlab Lab

A three-AS routing laboratory built with Containerlab and FRRouting. The internal routing protocol is IS-IS, while iBGP and eBGP provide inter-domain route propagation. The middle AS will export BGP information to a BMP collector.

## Topology

```text
AS65001                    AS65002                         AS65003

as1-r1 -- as1-r2 -- as2-r3 ----- as2-r4              as3-r6 -- as3-r7
                       \           /                       /
                        \-- as2-r5 -----------------------/
```

AS65002 forms an internal triangle between `as2-r3`, `as2-r4`, and `as2-r5`.

## Nodes

| Autonomous System | Routers |
|---|---|
| AS65001 | as1-r1, as1-r2 |
| AS65002 | as2-r3, as2-r4, as2-r5 |
| AS65003 | as3-r6, as3-r7 |

## Addressing

### Loopbacks

| Router | Address |
|---|---|
| as1-r1 | 10.255.1.1/32 |
| as1-r2 | 10.255.1.2/32 |
| as2-r3 | 10.255.2.3/32 |
| as2-r4 | 10.255.2.4/32 |
| as2-r5 | 10.255.2.5/32 |
| as3-r6 | 10.255.3.6/32 |
| as3-r7 | 10.255.3.7/32 |

### Point-to-point links

| Link | Addresses |
|---|---|
| as1-r1 — as1-r2 | 10.1.0.0/31 — 10.1.0.1/31 |
| as2-r3 — as2-r4 | 10.2.0.0/31 — 10.2.0.1/31 |
| as2-r4 — as2-r5 | 10.2.0.2/31 — 10.2.0.3/31 |
| as2-r3 — as2-r5 | 10.2.0.4/31 — 10.2.0.5/31 |
| as3-r6 — as3-r7 | 10.3.0.0/31 — 10.3.0.1/31 |
| as1-r2 — as2-r3 | 10.254.0.0/31 — 10.254.0.1/31 |
| as2-r5 — as3-r6 | 10.254.0.2/31 — 10.254.0.3/31 |

## Routing design

- IS-IS Level 2 is used inside each AS.
- Router loopbacks are passive IS-IS interfaces.
- Inter-AS link prefixes are passively advertised into the local IS-IS domain, without forming cross-AS IS-IS adjacencies.
- iBGP sessions use loopback addresses.
- AS65002 uses a full-mesh iBGP topology.
- eBGP sessions use directly connected `/31` interface addresses.
- Explicit prefix lists and route maps control eBGP import and export.

## Announced prefixes

| Origin AS | Prefix |
|---|---|
| AS65001 | 192.168.1.0/24 |
| AS65003 | 192.168.3.0/24 |

The prefixes are backed by static Null0 routes and are used to generate controlled BGP announcements and withdrawals.

## BMP

The FRR BMP module is enabled on:

- as2-r3
- as2-r4
- as2-r5

Each corresponding `bgpd` process is started with:

```text
-M bmp
```

Collector connection parameters are currently pending.

## Requirements

- Containerlab
- Docker
- FRRouting image `quay.io/frrouting/frr:10.6.1`
- On macOS: OrbStack or another supported Linux VM environment

## Deployment

```bash
containerlab deploy -t bmp-lab.clab.yml
```

## Verification

On a router:

```text
show isis neighbor
show isis database
show bgp ipv4 unicast summary
show bgp ipv4 unicast
```

## Project structure

```text
.
├── bmp-lab.clab.yml
└── configs
    ├── as1-r1
    ├── as1-r2
    ├── as2-r3
    ├── as2-r4
    ├── as2-r5
    ├── as3-r6
    └── as3-r7
```

Each router directory contains:

```text
daemons
frr.conf
vtysh.conf
```

## Current status

- [x] Containerlab topology
- [x] IPv4 addressing
- [x] IS-IS underlay
- [x] iBGP sessions
- [x] eBGP sessions
- [x] Controlled BGP prefix propagation
- [x] BMP modules loaded on AS65002 routers
- [ ] BMP collector connection
- [ ] BMP route monitoring experiments
- [ ] BMP message aggregation analysis
