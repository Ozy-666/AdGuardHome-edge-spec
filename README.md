# AdGuardHome Edge - Public Specification

[![Production](https://img.shields.io/badge/Powered_By-dnsdoh.art-24292e?style=flat-square&logo=ghost)](https://dnsdoh.art)
[![Stack](https://img.shields.io/badge/Stack-Nginx%20%E2%86%92%20AGH%20%E2%86%92%20Unbound%20%E2%86%92%20dnscrypt--proxy-2b6cb0?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Protocols](https://img.shields.io/badge/Protocols-DoH3%20%7C%20DoH%20%7C%20DoQ%20%7C%20DoT%20%7C%20DNS-2b6cb0?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Binary Size](https://img.shields.io/badge/Binary%20Size-24.6%20MB%20(--10%20MB)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Bloat Removed](https://img.shields.io/badge/Codebase%20Bloat-13k%2B%20Lines%20Removed-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

---

**Repository status:** This document is the public architectural specification and progress tracker for the AdGuardHome Edge project. The main codebase and production configuration remain private. This repository exists to document the design philosophy, engineering decisions, and measurable outcomes of the project in a form that can be shared openly.

---

## 📊 Codebase Modifications & Performance Metrics

Below is a detailed breakdown of the exact subsystem prunings and system-level performance enhancements implemented across our custom forks to maintain near-zero latency and high uptime.

### 🗑️ Subsystem Stripping (Bloat Reduction)
*These badges represent non-essential features of standard AdGuardHome that have been completely stripped out of our build to reduce resource consumption and eliminate security attack surface:*

[![DHCP Subsystem](https://img.shields.io/badge/DHCP%20Subsystem-Stripped%20(--2400%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![SafeBrowsing & Parental](https://img.shields.io/badge/SafeBrowsing%20%26%20Parental-Removed%20(--2200%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Blocked Services](https://img.shields.io/badge/Blocked%20Services-Removed%20(--5200%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![SafeSearch Engine](https://img.shields.io/badge/SafeSearch%20Engine-Removed%20(--1419%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![DNSSEC Engine](https://img.shields.io/badge/DNSSEC%20Engine-Stripped%20%2F%20Offloaded-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![EDNS Subnet](https://img.shields.io/badge/EDNS%20Client%20Subnet-Stripped%20%2F%20Anti--Leak-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![UI Locales](https://img.shields.io/badge/UI%20Locales-English%20Only%20(--2%20MB)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

### ⚡ Engineering Patches & Optimizations
*These badges represent active code-level corrections, memory allocation pools, and logic improvements across our transport, proxy, filter, and resolver / OS-tuning layers:*

[![TCP DNS Framing](https://img.shields.io/badge/TCP%20DNS%20Framing-Hardened-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Goroutine Lifecycle](https://img.shields.io/badge/Goroutine%20Lifecycle-Leak--Free-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Config-Reload Stall](https://img.shields.io/badge/Config--Reload%20Stall-Removed-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![urlfilter Engine](https://img.shields.io/badge/urlfilter%20Fork-O(1)%20AST%20Regexp-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/urlfilter)
[![UDP Query Buffers](https://img.shields.io/badge/UDP%20Query%20Buffers-0%20allocs%2Fop-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/dnscrypt-proxy)
[![Encrypted Response](https://img.shields.io/badge/Encrypted%20Response-0%20allocs%2Fop-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/dnscrypt-proxy)
[![Session Map](https://img.shields.io/badge/Session%20Map-Lazy--Init%20(0%20allocs)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/dnscrypt-proxy)
[![Unbound Allocator](https://img.shields.io/badge/Unbound%20Allocator-jemalloc%20(LD__PRELOAD)-2b6cb0?style=flat-square&logo=linux)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Loopback RPS](https://img.shields.io/badge/Loopback%20RPS-Disabled%20(--14%25%20CPU)-2b6cb0?style=flat-square&logo=linux)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Client-Storage Lock](https://img.shields.io/badge/Per--Query%20Client%20Lock-Lock--Free%20(--79%25%20contention)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Real-Time WAF Feed](https://img.shields.io/badge/Real--Time%20WAF%20Feed-qfeed%20(0%20allocs%2C%20non--blocking)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

---
## Production Deployment

The architecture specified in this repository directly powers the backend
infrastructure of **[dnsdoh.art](https://dnsdoh.art)** — a next-generation,
ultra-fast, zero-telemetry public DNS resolver.

By running this highly optimised AdGuardHome Edge build tightly integrated with
a custom [dnsproxy](https://github.com/Ozy-666/dnsproxy) transport layer and a
custom [urlfilter](https://github.com/Ozy-666/urlfilter) filtering engine, the
production deployment delivers:

- **Strict Privacy:** Zero data retention, zero upstream client leakage
  (EDNS-CS stripped), and no cloud dependencies.
- **Maximum Throughput:** Lock-free processing paths designed to withstand
  high-RPS conditions on AMD EPYC edge hardware.
- **Hardened Filtering:** A custom `urlfilter` fork with AST-based shortcut
  extraction that keeps the rule-matching hot path O(1) even under
  unique-subdomain floods against heavy regexp blocklists.
- **Modern Protocols:** Native support for plain UDP/TCP DNS,
  DNS-over-TLS (DoT), DNS-over-QUIC (DoQ), and DNS-over-HTTPS (DoH).

*Frontend UI visualisations and infrastructure metric screenshots will be
published here as the public deployment scales.*

---

## Table of Contents

- [Production Deployment](#production-deployment)
1. [Why Edge? — Executive Summary](#1-why-edge--executive-summary)
2. [What Was Removed, and Why](#2-what-was-removed-and-why)
3. [Architecture Overview](#3-architecture-overview)
4. [The dnsproxy Fork](#4-the-dnsproxy-fork)
5. [The dnscrypt-proxy Fork](#5-the-dnscrypt-proxy-fork)
6. [The Unbound Resolver Layer & System Tuning](#6-the-unbound-resolver-layer--system-tuning)
7. [Performance Engineering — AGH Layer](#7-performance-engineering--agh-layer)
8. [Performance Engineering — Transport Layer (dnsproxy)](#8-performance-engineering--transport-layer-dnsproxy)
9. [Security Hardening](#9-security-hardening)
10. [Configuration Reference](#10-configuration-reference)
11. [Release History](#11-release-history)
12. [Completeness Status](#12-completeness-status)

---

## 1. Why Edge? — Executive Summary

Standard AdGuardHome is a general-purpose DNS filter designed for broad
consumer deployment. It ships with a large surface area: cloud-based threat
feeds (SafeBrowsing, Parental Controls), a full DHCP server, external whois
TCP queries, DNSSEC validation hooks, per-language UI bundles, an auto-updater,
and a QUIC transport layer that was never stress-hardened.

**AdGuardHome Edge starts from a different premise.** It is built for a single
class of deployment: a private, high-performance, privacy-maximising DNS
resolver running at the network edge — directly in front of user traffic,
with no tolerance for latency spikes, memory churn, or external dependencies
in the query hot path.

The engineering goals, in order of priority:

### Privacy by Default

Every feature that sends data outside the local machine was removed or
replaced with a local equivalent. This includes:

- **SafeBrowsing and Parental Controls** — both rely on AdGuard cloud hashing
  services. Each DNS query involving a suspicious domain triggers an outbound
  HTTP call to an external API. This is fundamentally incompatible with an
  edge resolver that must not leak query information to any third party.
- **EDNS Client Subnet** — forwards a truncated client IP address to upstream
  resolvers. Removed entirely: no user location data leaves the resolver.
- **External Whois** — replaced with local MaxMind GeoIP database lookups.
  The upstream implementation opened a new TCP connection per query-log entry
  enrichment; the edge build answers in microseconds from local mmdb files
  with zero outbound connections.
- **Auto-updater** — disabled. The upstream updater can replace the binary
  with a vanilla release, overwriting all edge patches. The `-edge` version
  suffix sets `CanAutoUpdate = false` at the code level.

### Zero-Allocation Hot Path

A DNS resolver's core loop — receive UDP packet → filter → pack response →
sendmsg — executes millions of times per day. Every heap allocation in this
path contributes to GC pause frequency and GC CPU overhead. The edge build
subjected the entire request pipeline to a systematic allocation audit and
eliminated every unnecessary allocation, from pipeline dispatch arrays to
per-response pack buffers.

The most impactful changes moved to a custom fork of the underlying
[dnsproxy](https://github.com/Ozy-666/dnsproxy) transport library, where
the write paths for UDP, TCP, and DoT now operate with **zero heap allocations
per response**.

### Lock Contention Elimination

The upstream codebase uses a single global `serverLock` reader-writer mutex
to protect DNS server state. Under high request concurrency, this lock
serializes goroutines that have no actual data dependency on each other. The
edge build systematically replaced every hot-path `serverLock` acquisition
with either:

- **`atomic.Pointer[T]`** for fields that are replaced atomically on
  reconfiguration (access manager, DNS filter), or
- **narrowed critical sections** that snapshot a pointer in two lines and
  release the lock before any I/O or channel operation.

Three additional global mutexes in the dnsproxy transport layer were
eliminated using the same technique.

### Uptime and Operational Correctness

Several latent bugs were fixed as part of the audit:

- A goroutine leak in the query log rotation path that accumulated one leaked
  goroutine per `Start/Shutdown` cycle.
- A `flushPending` flag that could get permanently stuck after a rare encoding
  error, silently blocking all future log flushes.
- A 100 ms `time.Sleep` held under the global server lock on every
  configuration reload — a pure unnecessary latency spike on admin changes.
- A short-read bug in the TCP DNS framing layer (`conn.Read` does not
  guarantee reading the full length prefix in one call; fixed with
  `io.ReadFull`).
- A latent data race on the access manager field in the middleware layer (read
  without lock or atomic, written under an exclusive lock — undefined behaviour
  under Go's memory model).

---

## 2. What Was Removed, and Why

| Feature | Lines removed | Reason |
|---|---|---|
| **DHCP subsystem** | ~2,400 | Not applicable to edge deployment; significant attack surface |
| **SafeBrowsing** | ~1,400 | Cloud-dependent; leaks query hashes to AdGuard servers |
| **Parental Controls** | ~800 | Cloud-dependent; same privacy concern as SafeBrowsing |
| **SafeSearch** | ~734 (engine) + ~685 (hash-prefix shared library) | Cloud-dependent; shared hash-prefix engine deleted alongside |
| **Blocked Services** | ~5,200 | 3,286-line service database + all handlers, config, and frontend; not applicable |
| **DNSSEC validation** | ~400 | Not needed in this deployment; eliminates AD-bit complexity from query path |
| **EDNS Client Subnet** | ~300 | Leaks truncated client IP to upstream resolvers; incompatible with privacy posture |
| **Non-English UI locales** | ~2 MB | 35 locale files; language selector; eliminates locale detection path at startup |
| **External Whois client** | Replaced | Local MaxMind mmdb substituted; zero outbound TCP connections |
| **OpenAPI dead spec** | 442 lines | Dead endpoint paths, schemas, and enum values for all removed features |

**What was NOT removed:** rule-list based DNS filtering (the core AGH value),
query logging, statistics, per-client settings, rate limiting, DoT/DoQ/DoH
support, the admin web UI (English), and all upstream DNS protocol handling.

---

## 3. Architecture Overview

### Network Topology

```
Internet clients
      │
      ├─── HTTPS/H2/H3 :443 ──▶ nginx (TLS termination)
      │                               │
      │                               └──▶ Unix domain socket ──▶ AGH (plain HTTP)
      │
      ├─── DoT :853 ────────────────────────────────────────────▶ AGH directly
      ├─── DoQ :853 (UDP) ──────────────────────────────────────▶ AGH directly
      └─── Plain DNS :53 (UDP+TCP) ─────────────────────────────▶ AGH directly

AGH upstream chain:
  AGH ──▶ local Unbound (recursive resolver)
        └──▶ dnscrypt-proxy ──▶ encrypted upstream resolvers
```

### Key Design Decisions

**Unix domain socket for admin and DoH.** AGH binds its plain HTTP server to
a Unix socket. nginx proxies all inbound HTTPS traffic to this socket. This
eliminates the redundant TLS termination AGH would otherwise perform for
its own HTTPS port — TLS is handled exactly once, at the nginx boundary.
A middleware layer on the UDS path reconstructs the client IP from the
`X-Real-IP` header nginx injects, so query logs record real client addresses.

**Timeout alignment.** AGH enforces a 1-second upstream timeout. dnscrypt-proxy
previously ran with a 2.5-second per-attempt timeout, continuing to issue
retries and hold sockets open for 1.5 seconds after AGH had already
returned an error to the client. This was pure wasted resource consumption.
The dnscrypt-proxy timeout was reduced to 800 ms — tight enough to resolve
before AGH's deadline, loose enough to not penalise normally-slow upstreams.

**go.mod replace for the forks.** Two dependencies are maintained as custom
forks, each loaded via a `go.mod replace` directive pointing to a local checkout
(no separate module path or versioned tag required):

- [dnsproxy](https://github.com/Ozy-666/dnsproxy) — the transport layer (§4).
- [urlfilter](https://github.com/Ozy-666/urlfilter) — the rule-matching engine,
  with AST-based shortcut extraction that keeps the regexp hot path O(1) under
  unique-subdomain floods (§7.7).

---

## 4. The dnsproxy Fork

> **Fork:** [https://github.com/Ozy-666/dnsproxy](https://github.com/Ozy-666/dnsproxy)
> Branch: `edge-udp-pool` | Base: upstream `v0.81.4`

The upstream [AdguardTeam/dnsproxy](https://github.com/AdguardTeam/dnsproxy)
library is the transport engine that AGH delegates all DNS protocol handling
to. It manages UDP sockets, TCP connections, DoT, DoQ, and DoH listeners. For
an edge resolver, the transport layer's allocation and lock behaviour directly
determines the GC and CPU profile of the server under sustained load.

A systematic audit of the dnsproxy codebase identified five structural
problems. All five are fixed in the edge fork.

### 4.1 Zero-Allocation UDP Write Path

**Problem:** Every UDP DNS response called `resp.Pack()`, which internally
allocates a `[]byte` of exactly the message size. The byte slice is used for
one `sendmsg` syscall and then discarded. At high query rates this is the
single most frequent heap allocation in the entire stack.

**Fix:** `respondUDP()` now draws a 2048-byte buffer from a `sync.Pool`,
calls `resp.PackBuffer()` to pack the message into the pooled buffer (no
allocation if the response fits), performs the synchronous write (the kernel
copies bytes before `sendmsg` returns), and immediately returns the buffer to
the pool. If a response exceeds 2048 bytes, the pool slot is updated to the
larger allocation, so subsequent oversized responses also benefit.

| | ns/op | B/op | allocs/op |
|---|---|---|---|
| Before | 255 | 160 | **1** |
| After | ~282 | **0** | **0** |

The ns/op variance is measurement noise. The allocation delta —
**−1 alloc, −160 B per UDP response** — compounds directly into lower GC
pressure at sustained query rates.

### 4.2 Zero-Allocation TCP/DoT Path

**Problem:** The TCP DNS framing path allocated four times per round-trip:
two small allocations for the length prefix, one for the message body, and
one inside `resp.Pack()`. Additionally, the original read call on the length
prefix was susceptible to a short-read: a streaming socket may return fewer
bytes than requested on a single read call, which was not handled.

**Fix:** A 65537-byte pool buffer (2 prefix bytes + maximum DNS message body)
serves both read and write for an entire TCP round-trip. Two `io.ReadFull`
calls replace the short-read-prone original. The unpacked DNS message is a
zero-copy sub-slice of the pool buffer. The response is packed into the same
pool slot and written as a single call.

| | ns/op | B/op | allocs/op |
|---|---|---|---|
| Before | — | ~512+ | **4** |
| After | ~251 | **0** | **0** |

### 4.3 Lock-Free Server State Checks

**Problem:** The server state check — called on every TCP keepalive iteration
to determine whether the proxy is still running — acquired the global embedded
`sync.RWMutex` read lock. Under a concurrent shutdown, the pending write-lock
caused all active read-lock callers to queue behind it, producing a
thundering-herd effect as all goroutines unblocked simultaneously on restart.

**Fix:** The started flag was converted from a plain boolean to `atomic.Bool`.
The state check is now a single atomic load with no lock acquisition.
`Start()` and `Shutdown()` retain their exclusive lock (correct: they mutate
listener slices), but use atomic store/load for the flag itself.

### 4.4 Lock-Free Upstream RTT Statistics (Copy-on-Write)

**Problem:** The upstream weight calculation — called on every load-balanced
DNS query to select an upstream server by RTT — acquired an **exclusive**
mutex to *read* the RTT statistics map. This serialized all concurrent DNS
goroutines at the upstream dispatch point.

**Fix:** The mutable map and its exclusive mutex are replaced with an
`atomic.Pointer` to an immutable map snapshot, plus a narrow write-only mutex.

- Weight calculation performs a single atomic pointer load — zero contention,
  no lock. The returned snapshot is consistent for the duration of the
  calculation.
- RTT update holds the write mutex only for a shallow map copy (typically
  2–3 entries), updates one entry, and stores the new pointer atomically.
  Readers see either the old or the new snapshot, never a partially-written map.

### 4.5 QUIC Per-Connection Stream Limit

**Problem:** The upstream dnsproxy set the maximum incoming streams to 65,535
per connection for both DoQ and DoH3. A single QUIC client could open 65,535
concurrent streams per connection, each spawning a goroutine and consuming a
slot from the global request semaphore. A misbehaving or malicious client
could starve every other client and protocol on the server.

**Fix:** A configurable `QUICMaxIncomingStreams` field replaces the hardcoded
constant. QUIC's transport-layer flow control (`MAX_STREAMS` frame) now
enforces the limit: no goroutine is spawned for a stream beyond the cap.
Validation logic handles the full input range:

| Input | Behaviour |
|---|---|
| `0` (absent / unset) | Silently use default **64** |
| `[1, 1024]` | Used as-is |
| Any other non-zero value | `WARN` log emitted; default **64** used |

The field is exposed in `AdGuardHome.yaml` under `dns.quic_max_incoming_streams`
(introduced in schema v35, deployed default: 64).

**Hardening (v0.107.108):** the cap above applies to *bidirectional* streams,
which is what DoQ queries use. The *unidirectional* stream limit was previously
tied to the same value — but HTTP/3 (DoH3) requires at least three uni streams
per connection (a control stream plus the QPACK encoder and decoder, RFC 9114).
Setting `quic_max_incoming_streams` to 1–2 would therefore have silently broken
DoH3 while DoQ kept working. The uni limit is now decoupled and fixed at 64,
independent of the bidirectional flood-control knob.

### 4.6 Bounded DoH POST Body

**Problem:** The DoH POST handler read the entire request body with no size
limit. Under a POST flood, each goroutine would allocate memory proportional
to the request body size until the HTTP read timeout fired, creating GC
pressure spikes proportional to attack intensity. An acknowledged `TODO`
comment in the upstream codebase noted this was never resolved.

**Fix:** The read is capped at the DNS wire-format maximum (65,535 bytes per
RFC 8484). Bodies exceeding this limit receive `413 Request Entity Too Large`
before any DNS unpacking occurs. All legitimate DoH requests fit within this
bound.

---

## 5. The dnscrypt-proxy Fork

> **Fork:** [https://github.com/Ozy-666/dnscrypt-proxy](https://github.com/Ozy-666/dnscrypt-proxy)
> Branch: `edge-stable` | Base: upstream `2.1.16`

[dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy) is the encrypted
upstream client at the end of the chain (AGH-Edge → Unbound → **dnscrypt-proxy**
→ Cloudflare / Quad9 / Google over DNSCrypt + DoH). The edge fork is stripped to
linux/amd64 only, compiled with `GOAMD64=v3` / `CGO_ENABLED=0`, and carries the
same zero-allocation / lock-free discipline applied to the rest of the stack.

**Slimming / attack-surface reduction**

- Removed the embedded monitoring UI — an HTTP + WebSocket + Prometheus server
  with bundled HTML/JS assets — and dropped the `gorilla/websocket` dependency:
  ~455 KB smaller binary and one fewer network-facing component. The config
  section is kept inert so existing TOML still decodes.
- Stripped Oblivious DoH (ODoH) entirely — ~800 LOC of config/crypto, the
  oblivious transport path, target-config fetch, all `StampProtoTypeODoH*`
  branches, and the `odoh_servers` option (the strict loader no longer
  recognises the key). Stripped binary ~13.9 MB.

**Privacy / no discretionary egress**

- Remote `[sources]` list downloads are disabled at the binary level —
  resolver/relay lists load only from local signed cache files (fail-closed) and
  upstreams are pinned by `[static]` stamps. Combined with the absence of any
  auto-update or version-check call (audited: `-version` is local-only, the sole
  `http.Client` is the DoH query transport, `netprobe` is just a UDP probe), the
  proxy makes **no** discretionary outbound HTTP — only the encrypted DNS queries
  themselves leave the host.

**Zero-allocation hot path**

- Inbound UDP query buffers and encrypted upstream-response buffers are pooled
  (`sync.Pool`): from 4096 B / 1 alloc per query to **0 B / 0 allocs**.
- The per-query plugin `sessionData` map is lazily allocated — **0 allocs** when
  the active plugins never write session state.
- Smaller trims: the UDP connection-pool address key is formatted once per query
  instead of twice; ASCII name reversal avoids the `[]rune` round-trip; a
  per-query debug string is elided unless debug logging is on.
- The EDNS0 padding hex (`strings.Repeat("58", …)`) is precomputed once and
  sliced per DoH query instead of rebuilt — one fewer allocation per padded query
  (the majority, since WP2 favours the DoH upstreams).

**Lock-free / concurrency**

- Removed three dead per-query `RWMutex` read-locks on the plugin pipeline (the
  protected slices are immutable after startup — there is no writer).
- Server selection (`getOne`) takes a shared read-lock on the WP2 path instead
  of the exclusive lock, and the O(n) dormant-server recovery scan was moved off
  the per-query path onto a 10 s maintenance goroutine. The per-query stats
  update no longer scans the server list under the lock.

**Runtime configuration**

- `lb_strategy = 'wp2'`, `lb_estimator = false` — WP2 is the only strategy that
  uses the lock-free read-lock selection path, giving parallel server selection
  across all four cores while spreading load over the anycast upstreams (no
  flip/herd jitter under load). A live A/B (each strategy restarted, warmed, then
  probed identically) confirmed WP2 is both the **lowest-latency** (seq p50
  4–5 ms vs 6 ms for `fastest`/`p2`) and **highest-throughput** option (2,285 QPS
  vs 1,760 / 1,579): `fastest` pins to whatever node was `inner[0]` at startup
  while WP2's power-of-two sampling keeps catching the momentarily-fastest
  upstream, and the ~25–30% throughput gap is the exclusive `Lock()` (other
  strategies) versus WP2's shared `RLock()`.

**Timeouts**

- The toml query budget (800 ms) is wired into the HTTP transport's
  connection-level timeouts (TCP-connect / `ResponseHeaderTimeout` /
  `ExpectContinueTimeout`), which had been left at a 30 s default config never
  overrode. The HTTP/2 idle health-check is decoupled (`ReadIdleTimeout`
  30 s → 10 s, explicit 5 s `PingTimeout`) so dead DoH keep-alive connections are
  reaped in ~15 s instead of riding the per-query timeout.

**Measured** (AMD EPYC 7542, 4 vCPU, median of 3):

| Change | Before | After |
|---|---|---|
| Plugin-pipeline lock (per query) | 64.6 ns | **6.8 ns** (~9.5×) |
| ASCII name reversal | 309.7 ns, 48 B | **59.2 ns, 32 B** (~5.2×) |
| Per-query stats update | 59.5 ns | **28.4 ns** (~2.1×) |
| UDP pool address key | 325.7 ns, 6 allocs | **164.2 ns, 3 allocs** |
| Pooled UDP / response buffers | 4096 B, 1 alloc | **0 B, 0 allocs** |

Validated under sustained load and malformed-packet fuzzing on the live
resolver: stable ~20 MB RSS with no leaks, all malformed input dropped at
validation with no crashes, and throughput bound by upstream RTT while the proxy
itself used a fraction of one core.

## 6. The Unbound Resolver Layer & System Tuning

> **Upstream:** [NLnet Labs Unbound](https://nlnetlabs.nl/projects/unbound/) 1.25.x — **tuned and re-linked to BoringSSL, not forked.**
> Profiled under load; the validated C core left unpatched (the crypto *substrate* was swapped — §6.6).

Unbound sits in the middle of the upstream chain (AGH → **Unbound :5353** →
dnscrypt-proxy :5053) as the validating cache. It forwards every query to
dnscrypt-proxy rather than recursing from root, so its job is caching, DNSSEC
validation, and response shaping — not iteration. Unlike the dnsproxy / dnscrypt
/ urlfilter layers, Unbound is mature, security-critical C maintained by NLnet
Labs for two decades. The engineering posture here is therefore deliberate:
**tune the deployment and the OS around it, profile before touching code, and
leave the validated core alone.**

### 6.1 Allocator — jemalloc actually linked

The build had long passed `--with-libjemalloc` to `configure` and the config
header advertised jemalloc — but **Unbound has no such configure option.** The
flag was silently ignored (`configure: WARNING: unrecognized options`), so every
build had in fact been running on glibc `malloc`. jemalloc is now supplied the
way Unbound actually supports it — a systemd `LD_PRELOAD` drop-in:

```ini
Environment=LD_PRELOAD=/usr/lib/x86_64-linux-gnu/libjemalloc.so.2
Environment=MALLOC_CONF=narenas:4,background_thread:true
```

`narenas:4` pins one arena per worker thread (the default `4×ncpu = 16` spreads
dirty pages across sixteen arenas); `background_thread:true` purges freed memory
asynchronously, off the query hot path. Verified live (mapped in
`/proc/<pid>/maps`; exactly four `jemalloc_bg_thd` threads). The build script now
carries a pre-flight check that the preload library and drop-in are present, so
the regression cannot silently recur.

### 6.2 Cache slab tuning

`msg-`, `rrset-`, `infra-` and `key-cache-slabs` raised from the default `4` to
**`8`** (2× `num-threads`), giving finer-grained per-cache locking headroom under
high-concurrency floods — slab-lock contention was the top in-process cost
measured (§6.4).

### 6.3 AppArmor confinement fixes

Two benign-but-noisy denials on the `unbound` profile were resolved without
granting any new privilege:

| Denial | Cause | Fix |
|---|---|---|
| `transparent_hugepage/enabled` read | jemalloc probes the THP setting at startup | grant read on that one sysctl |
| `CAP_NET_ADMIN` | `so-rcvbuf`/`so-sndbuf: 8m` make Unbound attempt `SO_RCVBUFFORCE` first | quiet `deny capability net_admin` — the plain `SO_RCVBUF` fallback already reaches 8 MB, since `net.core.rmem_max = 8 MB` |

Both eliminate a per-start audit-log denial while leaving the confinement intact.

### 6.4 Profile-driven verdict — kernel-bound, left unpatched

Unbound was profiled with `perf` under a synthetic **8,000 qps** load (≈600× the
production norm), cache-miss traffic routed to a local mock so no real upstreams
were touched, behind an automatic abort guard (system load + a real-user latency
canary):

| Measurement | Result |
|---|---|
| Throughput | 8k qps sustained, 0 lost, 100% NOERROR |
| Latency | cache-hit **145 µs**, cache-miss **1.1 ms** |
| On-CPU time in kernel | **70–79%** (network / syscall path) |
| Top in-process cost | `lruhash_lookup` + spin-lock ≈ **6%** — cache-hash contention on the single forward-target infra entry |
| DNSSEC crypto | not exercised by the insecure mock — measured + optimized separately in **§6.6** |

**Conclusion: Unbound is kernel/network-bound, not CPU-bound, with very large
headroom.** Its own code is a minority of even a 600×-load profile, and the one
in-process hot spot is cache-lock contention (addressed by slab tuning in §6.2,
not by patching). Source patching (a hypothetical "step 3") was assessed and
**declined** — the measured upside did not justify modifying a security-critical
resolver core. The dominant costs were all *outside* Unbound: kernel networking
(§6.5) and CPU security mitigations.

### 6.5 System tuning — RPS disabled on loopback

The single biggest kernel cost the profile surfaced was cross-CPU IPIs
(`net_rps_send_ipi` plus remote wakeups). Root cause: **Receive Packet Steering
was enabled on the loopback interface** (`/sys/class/net/lo/queues/rx-0/rps_cpus
= f`, all four cores). Because the whole stack is loopback (AGH → Unbound →
dnscrypt on `127.0.0.1`) and loopback receive normally runs in the *sender's* CPU
context, RPS re-steering every packet to a hash-chosen core generated an IPI
storm. RPS is a tool for physical NICs with few hardware queues; on loopback it
is counterproductive.

RPS was disabled on `lo` only — the NIC keeps RPS, which is correct for real
multi-flow external traffic. Measured under identical 10k-qps loopback load:

| Metric (whole box, 10 s) | RPS on `lo` | RPS off | Δ |
|---|---|---|---|
| CPU cycles | 75.8 B | 65.3 B | **−14%** |
| Instructions | 55.4 B | 44.8 B | −19% |
| Function-call IPIs | ~230k | ~130–190k | **−30…−40%** |
| Latency / throughput / loss | — | unchanged | — |

A ~14% box-wide CPU reduction under load at zero latency cost — a larger and
safer win than any in-process change, and one that benefits the entire loopback
stack (AGH and dnscrypt-proxy too), not just Unbound.

### 6.6 Cryptographic Substrate — Unbound on BoringSSL

§6.4 left one cost unmeasured: the insecure mock never exercised **DNSSEC
signature validation.** Closing that gap drove the most involved change in this
layer — and, crucially, one made *beneath* the validated core rather than inside
it. The posture of §6 holds: Unbound's C logic is still untouched; only the crypto
**library it links** was replaced.

**The measurement.** Re-run under a 100%-signed cache-miss flood (a locally-signed
ECDSA-P256 island zone, validated end-to-end — `ad` flag confirmed), `perf` showed
**ECDSA verify consuming ~46% of Unbound's on-CPU time** — the single dominant
in-process cost once crypto is in play. Validation is two scalar point-multiplies
per signature; under flood it dominates.

**The OpenSSL dead-end.** The reflex fix — a newer OpenSSL — was tested and
**rejected on evidence.** Private static builds of OpenSSL **4.0.0** and **3.6.2**
(Zen 2 `-march=znver2`, `enable-ec_nistp_64_gcc_128`) gained only **~3%** on the
EVP verify path. The reason is structural: ECDSA-P256 verify is ~80 µs/op and the
stock system OpenSSL 3.0 *already* uses the hand-tuned `ecp_nistz256` ADX assembly.
There was no version lever to pull.

**BoringSSL.** Google's BoringSSL pairs **fiat-crypto** (formally-verified field
arithmetic) with zero of OpenSSL 3.x's provider-dispatch/fetch overhead. Measured
end-to-end on the live production resolver, identical 8k-qps signed-miss flood:

| Metric (prod, 100% signed-miss) | System OpenSSL 3.0.16 | **Unbound on BoringSSL** | Δ |
|---|---|---|---|
| `libcrypto` CPU (self-time) | 46.1% | **39.8%** | **−14% relative** |
| Throughput | 6,607 qps | **7,241 qps** | **+9.6%** |
| Avg latency | 12.8 ms | **9.5 ms** | **−26%** |
| Loss / validation | 0% / 100% | 0% / 100% | — |

`perf` confirms the shift: BoringSSL spends its crypto time in lean
`ecp_nistz256_*` (fiat-crypto, ADX), where OpenSSL's 44–46% was fragmented across
`BN_*` plus `OPENSSL_LH_doall_arg` (provider fetch) and a per-verify key re-parse.

> **Measure on the target silicon.** A sister build on a *virtualised* host reported
> a ~2.4× BoringSSL win — not reproducible here. That VM had its CPUID
> ADX/BMI2 flags masked, forcing *OpenSSL* onto a slow generic-bignum fallback and
> inflating the gap. On real EPYC 7542 (Zen 2) the honest figure is ~1.1–1.3× — a
> worst-case-headroom win, invisible at the cache-warm production norm. Hypervisor
> benchmarks are not ground truth.

**Build engineering (reproducible, reversible).** BoringSSL is pinned to a fixed
commit (it has no release/ABI guarantees), staged once to `/opt/boring`, and found
at runtime via a baked-in **`RUNPATH`** — no `LD_LIBRARY_PATH`. Unbound configures
with `--with-ssl=/opt/boring --disable-gost` (BoringSSL has no GOST); the build
disables the wrongly-detected system `ENGINE` header, and builds the daemon + control
tools **but not `unbound-anchor`** (which needs PKCS7, absent in BoringSSL) — the
system `unbound-anchor` is preserved for root-key bootstrap, and in-daemon RFC 5011
still refreshes the trust anchor. The same pinned BoringSSL backs **Nginx** (static
link) for one unified crypto substrate across the edge. A one-command OpenSSL
fallback build is retained for instant reversion.

**The trap worth recording.** First deploy crashed at startup:
`undefined symbol: SSLv23_client_method`. The cause was *not* a missing BoringSSL
symbol (BoringSSL exports it) — it was **LSM path confinement**. Unbound runs under
an AppArmor profile that did not permit `/opt/boring/lib`; the loader silently fell
back to the *system* `libssl.so`, where `SSLv23_client_method` exists only as a
compatibility **macro, not an exported symbol**. The fix is one profile rule
(`/opt/boring/lib/*.so* mr,`) — and a lesson: validate the **full** production
config under the real systemd/AppArmor context, not a minimal one, before a swap.

**Correctness.** Verified on the live resolver across algorithms: real **ECDSA**
(cloudflare.com) and **RSA** (iana.org) chains validate (`ad`), and a deliberately
broken zone (`dnssec-failed.org`) is correctly rejected (`SERVFAIL`). Validation
behaviour is identical to the OpenSSL build — only faster.

---

## 7. Performance Engineering — AGH Layer

The following is a complete enumeration of all structural changes applied to
the AdGuardHome application layer. Each item was identified by static code
audit and verified by benchmark or race detector.

### 7.1 Memory Management & Allocation Elimination

| # | Location | Problem | Fix | Version |
|---|---|---|---|---|
| 1.1 | `ServeDNS` pipeline dispatch | Pipeline function slice heap-allocated per request | Package-level fixed-size array; zero per-request allocation | v0.107.76 |
| 1.2 | Filter result | Double allocation: result value forced to heap, second allocation immediately abandoned | Struct-copy into pre-allocated slot in DNS context; one allocation eliminated | v0.107.80 |
| 1.3 | Query log type/class strings | Map lookups on every log entry | Fixed `[256]string` lookup array initialised once; `ClassINET` fast-path returns interned constant | v0.107.81 |
| 1.4 | Upstream stats slice | Nil-slice + two `append` → two allocations when fallback upstream present | Stack-backed fixed array; reallocation eliminated | v0.107.82 |
| 1.5 | `NormalizeDomain` | Called three times on the same domain (middleware, filter, stats) | Computed once in `ServeDNS`; all downstream stages read the cached value | v0.107.76 |
| 1.6 | Client IP formatting | Intermediate allocation in IPv4-mapped address conversion | Direct `Unmap().AsSlice()` — correct IPv4-mapped handling, no intermediate slice | v0.107.76 |
| 1.7 | Client ID slice | Heap-allocated per request | Stack-backed fixed array | v0.107.76 |
| 1.8 | Per-request logger | Handler and logger allocated on every request when trace level disabled | Level-gate fast path: one `Enabled()` check, no allocation when level inactive | v0.107.83 |
| 4.3 | Filter settings | `*filtering.Settings` heap-allocated per DNS request | `sync.Pool`; reset-and-return on request completion; `ClientTags` backing array reused | v0.107.92 |
| 1.9 | `matchHost` — `urlfilter.DNSRequest` | `&urlfilter.DNSRequest{}` + two `NewSortedSliceSet` calls heap-allocated per query (tags + identifiers) | `ufReqPool` (`sync.Pool`): one pre-allocated struct reused; sets cleared via `Clear()`+`Add()` per call | v0.107.105 |
| 1.10 | `matchHost` — `urlfilter.DNSResult` | `MatchRequest()` allocates `*urlfilter.DNSResult` internally on each of the two engine calls per query | `dnsResPool` (`sync.Pool`) + `MatchRequestInto()`; fields nil-zeroed (not `[:0]`) to preserve nil-vs-empty semantics for downstream checks | v0.107.105 |
| 1.11 | `matchHost` — result cache | Repeated queries for the same domain/qtype re-enter the engine, acquire `engineLock.RLock`, and allocate on every call | Copy-on-Write `atomic.Pointer[matchCacheMap]`: read = 1 atomic load + 1 map lookup, 0 allocs, 0 locks. Cache miss writes under `matchCacheMu` while holding `engineLock.RLock` (prevents stale-write race vs. filter reload). Cap: 5000 entries, discard-and-restart on overflow | v0.107.106 |

### 7.2 Concurrency & Lock Contention

| # | Location | Problem | Fix | Version |
|---|---|---|---|---|
| 2.1 | Response filtering | `serverLock.RLock()` acquired once **per answer RR** (N locks per response) | Single lock over the full answer loop; lock-free inner function extracted | v0.107.77 |
| 2.2 | Stats update | Exclusive write lock used for read-only field access | Corrected to read lock; concurrent stat updates no longer serialised | v0.107.78 |
| 2.3 | Client count check | Redundant outer lock wrapping a call to an already-thread-safe method | Outer lock removed | v0.107.78 |
| 2.4 | Query log + stats | Global lock held across `ShouldLog`, `logQuery`, `ShouldCount`, `updateStats` | Narrowed to two-line pointer snapshot; lock released before any I/O | v0.107.79 |
| 2.5 | Client storage | Exclusive mutex serialised all concurrent reads behind writes | Promoted to `sync.RWMutex`; six read-only methods use read lock | v0.107.84 |
| 2.6 | Query log buffer | Lock held during goroutine creation | Explicit unlock before goroutine spawn | v0.107.85 |
| 2.7 | Query log flush | Flush-pending flag stuck after encoding error → permanent flush blockage | Buffer clear and flag reset now unconditional before error return | v0.107.85 |

### 7.3 serverLock Hot-Path Elimination (atomic.Pointer)

The global server RWMutex survived on three hot-path call sites after all
scope-narrowing work was complete. Each was converted to an atomic pointer:

| # | Call site | Before | After | Version |
|---|---|---|---|---|
| 5.1 | Access check (outermost middleware, every request) | RLock on every DNS request | Single atomic load; also fixed a latent data race with no synchronization at all | v0.107.94 |
| 5.2 | Client IP processing | RLock held across full address-processor call (including channel send) | Lock held only for two-word interface snapshot; released before call | v0.107.95 |
| 5.3 | DNS filter (entire filtering hot path) | RLock held across entire `filterDNSRequest` body | Field converted to `atomic.Pointer`; filtering path is completely lock-free | v0.107.96 |

### 7.4 State Lifecycle & Context Correctness

| # | Issue | Fix | Version |
|---|---|---|---|
| 3.1 | `context.TODO()` throughout query log and address processor | `Add/ShouldLog` gain `ctx` parameter; address processor accepts server context | v0.107.90 |
| 3.2 | Address processor silent IP drops | Drop counter; first drop logs at `Warn`; subsequent at `Debug` with running total | v0.107.91 |
| 3.3 | `Reconfigure` 100 ms sleep under global lock | Removed: `Shutdown()` is synchronous; `SO_REUSEADDR` makes the grace period unnecessary | v0.107.87 |
| 3.5 | Log rotation goroutine leak | Cancel/done lifecycle; `Shutdown()` cancels and waits; `Start()` re-entrant-safe | v0.107.88 |

### 7.5 Miscellaneous Correctness Fixes

| Fix | Version |
|---|---|
| Config disk-write full struct shallow-copy replaced with selective field assignment; runtime-only fields no longer shared with caller | v0.107.93 |
| Query log fast-paths: skip client storage lookup when logging globally disabled; skip when host matches ignore list | v0.107.89 |
| Stats render maps pre-sized across 6 accumulator sites; unbounded growth during large retention window render eliminated | v0.107.86 |
| GeoIP mmdb reader file-handle leak fixed: handles released on shutdown | v0.107.75 |
| IPv4-mapped address normalisation: `::ffff:x.x.x.x` looked up as 4-byte IPv4 in GeoIP, not 16-byte IPv6 | v0.107.75 |

### 7.6 Domain Match Cache (Copy-on-Write)

**Problem:** Every DNS query — including the 10th query for `google.com` —
re-entered the filtering engine. That means: acquiring `engineLock.RLock()`,
building a `urlfilter.DNSRequest`, calling the shortcut index, iterating the
noIndex linear scan (O(N_regex) for every query regardless of result), and
allocating the result objects. Real DNS traffic is highly skewed — the top 100
domains account for the majority of queries. This work was repeated identically
on every request.

**Fix:** A Copy-on-Write result cache behind `atomic.Pointer[matchCacheMap]`
in `DNSFilter` (`internal/filtering/matchcache.go`).

- **Read (hot path):** one `atomic.Pointer.Load()` + one map struct-key
  lookup. The key `{host string, rrtype uint16}` is a struct — the runtime
  accesses it in-place without heap allocation. **0 allocations, 0 locks.**
  The engine lock is never acquired on a cache hit.

- **Write (miss path):** `matchCacheMu.Lock()` → shallow map copy → insert →
  `atomic.Pointer.Store()`. Called while holding `engineLock.RLock()`, so the
  write is implicitly serialised against filter reload: `initFiltering` must
  acquire `engineLock.Lock()` (blocked until all RLock holders release),
  guaranteeing no stale result from the old engine lands in the cache after it
  has been cleared.

- **Invalidation:** `invalidateMatchCache()` (stores `nil`) called inside the
  `engineLock.Lock()` section of `initFiltering`. Any in-flight miss write
  completes before the lock is acquired, so the cache is clean before the new
  engine is installed.

- **Scope:** Only anonymous queries (no per-client tags or identifiers) are
  eligible. Per-client-rule queries always bypass the cache.

- **Cap:** 5,000 entries. On overflow the map is discarded and rebuilt from
  zero, bounding heap growth without requiring LRU bookkeeping.

Benchmark results (AMD EPYC 7542):

| Path | Before | After | Δ |
|---|---|---|---|
| Clean domain (warm cache) | 1553 ns · 4 allocs | **51 ns · 0 allocs** | −97% ns |
| Exact block (warm cache) | 3225 ns · 9 allocs | **51 ns · 0 allocs** | −98% ns |
| Regex N=1000, clean miss (warm) | 14,800 ns · 4 allocs | **53 ns · 0 allocs** | −99.6% ns |
| Cache hit, parallel (4 goroutines) | — | **13 ns · 0 allocs** | lock-free linear scaling |

The O(N_regex) noIndex scan cost — previously 14.8 μs with 1000 regex rules
and rising linearly with N — is completely eliminated for recurring queries.

A follow-up `noIndex`-gate optimization in the `urlfilter` fork (a blocking Bloom
filter, "Priority 4", and regex consolidation, "Priority 5") was investigated and
**shelved** — see §7.7.

> **Update (v0.107.107):** a system-wide audit found this copy-on-write cache
> copied the whole map on every *miss* (≈489 KB/op once full), which a
> unique-domain flood could weaponize into a DoS-amplification vector. It was
> replaced with a lock-free fixed-size table — see §7.8.

---

### 7.7 noIndex Regexp Scan — AST Shortcut Extraction (v0.107.109)

The `noIndex` slice in `urlfilter`'s network engine holds rules that fit neither
the shortcut index nor the domain index; it is scanned linearly on every request.

This optimization was first **shelved** (2026-05-24) on the rationale that
host-level regexp rules "never reach the DNS engine." A second review proved that
**false**, and the vector was closed proactively.

**The flaw.** A no-modifier regexp such as `/^ad-[a-z0-9]+\.com$/` passes
`IsHostLevelNetworkRule()` and *does* enter the DNS engine. The legacy shortcut
extractor returned an empty shortcut for any regexp containing `?` (it bailed on
lookahead/optional), so a large class of host-level regexps fell into the
empty-shortcut bucket and reached the regexp matcher on **every request**. The
match cache (§7.6/§7.8) shields only *warm hits*; a unique-subdomain flood is
all-miss, so against a heavy custom regexp list this is an O(N) CPU-exhaustion
vector — measured at ~110 ns per rule per query (N=1000 → ~110 µs per miss).

**Rejected: the alternation gate.** Merging the regexps into one RE2 alternation
as a fast-fail gate was prototyped and **rejected by benchmark** — it ran ~14×
*slower* than the linear scan. Anchored regexps reject a non-matching host in O(1)
individually; the union automaton loses that per-branch early-exit and its compiled
program grows with N. Empty-literal regexps cannot be prefiltered by any literal
structure. Killing this idea on data, not theory, was the right call.

**The fix: AST required-literal extraction.** `findRegexpShortcut` now parses each
regexp with `regexp/syntax` and extracts the longest **guaranteed-required
contiguous literal** (e.g. `/^ad[0-9]?-tracker\.com$/` → `-tracker.com`). Such
rules move into the shortcut index, so a random flood host — which doesn't contain
the literal — never evaluates them. The walk is conservative: optional, alternated,
repeated-zero and otherwise non-mandatory subexpressions contribute nothing, so it
can only *under*-extract and never claims a literal that isn't guaranteed (which
would drop real matches).

**Bluehat verification** (rules of engagement: any divergence or regression →
revert and drop):

| Gate | Result |
|---|---|
| Equivalence harness (AST vs legacy engine, real SDN+base lists + `requests.json` + procedural hosts) | **0 divergences / 39,983 hosts** |
| Fuzzing (`-fuzztime=30s`) | **130k executions, 0 divergences** |
| Pathological/malformed regexps | no panic, bounded work, ~29 allocs (load-time only) |
| Flood benchmark (N `?`-regexps, random host) | **111 µs → 449 ns at N=1000 (248×), flat in N, 0 query-path allocs** |

Residual: regexps with no ≥5 required literal anywhere (e.g. `/[0-9]{8}/`) remain in
the linear scan — rare, bounded per-rule, and further limited by per-IP firewall
rate limits.

---

### 7.8 Domain Match Cache v2 — Lock-Free Fixed-Size Table

**Problem (found in audit, v0.107.106 cache):** the copy-on-write cache (§7.6)
gave lock-free reads but copied the *entire* map on every cache **miss**
(≈489 KB/op once full, sawtooth-resetting at 5,000 entries), serialized under a
single mutex. Cache hits are not the adversarial case — *misses* are. A
distributed flood of unique random subdomains on port 53/853 (which bypasses the
nginx rate-limit tier; the firewall caps per-IP query rate but not the
*aggregate* miss rate) could force continuous O(N) map copies and GC pressure,
degrading latency for all traffic. Correctness was never affected; availability
was.

**Fix:** a lock-free, fixed-size, generation-stamped direct-mapped table.

- **Structure:** `[8192]atomic.Pointer[entry]` (a power-of-two slot count),
  indexed by `maphash.Comparable` over the `{host, rrtype}` key, masked with
  `& 8191`. 8,192 × 8 B = 64 KiB — L2-resident on the production EPYC 7542
  (512 KiB private L2 per core); the hot accessed subset stays in L1d.
- **Per-process random hash seed** so an attacker cannot craft hostnames that
  collide into one slot to evict a targeted domain.
- **Read (hot path):** one atomic pointer load + one atomic generation load +
  an in-place key compare. **0 allocations, no lock.**
- **Write (miss):** build an immutable entry, one atomic store. **O(1), no
  lock.** A slot collision overwrites the previous occupant (implicit
  eviction), so the memory footprint is hard-bounded by the slot count — no
  sawtooth, no unbounded growth.
- **Invalidation (filter reload):** a single atomic generation increment.
  Entries stamped with an older generation are ignored on read and lazily
  overwritten — no map reallocation, no GC spike. The same lock ordering as
  before guarantees no result computed against a pre-reload engine survives.

Benchmark results (AMD EPYC 7542, 4 vCPU):

| Path | v1 (CoW) | v2 (lock-free) | Δ |
|---|---|---|---|
| Warm hit | 0 allocs · ~13 ns (parallel) | 0 allocs · ~14 ns (parallel) | unchanged |
| Unique-domain miss | 350,364 ns · 488,648 B · 16 allocs | **2,098 ns · 272 B · 5 allocs** | **167× faster, 1797× less memory** |

The DoS-amplification vector is closed: a sustained unique-domain flood now costs
one small allocation and one atomic store per query instead of a ~489 KB
mutex-serialized map copy.

---

### 7.9 Per-Query Client-Storage Lock — Profile-Driven Fast Path (2026-06-03)

**Problem.** A mutex profile captured under a sustained DoT flood on the live
resolver (loopback generator, `SetMutexProfileFraction(1)`, pprof bound to
localhost) named the dominant lock on the DoT/DoQ hot path:
`client.(*Storage).CustomUpstreamConfig` accounted for **79% of all mutex-wait
delay** (1,059 s of 1,336 s over a 30 s window). It runs on *every* query
(`processUpstream → setCustomUpstream`) and takes the client storage's **writer**
lock to perform two **read-only** lookups that return nil for any address which is
not a configured persistent client — i.e. nearly every query. The CPU profile
confirmed the server was **not** CPU-bound (54 s of CPU samples against 1,059 s of
*off-CPU* mutex wait), matching the observed symptom exactly: all four cores busy,
none saturated, throughput capped at ≈2.7 cores. The originally-suspected per-query
querylog + statistics locks were only ~11% combined — real, but minor.

**Fix.** A lock-free fast-path gate. The upstream manager keeps an `atomic.Bool`
— true iff at least one client has an *effective* (non-comment, non-empty) custom
upstream — recomputed under the existing write lock whenever the client set
changes. `CustomUpstreamConfig` reads the flag locklessly and returns nil
**without acquiring the lock** when no per-client upstreams exist (the common
case, and the production configuration). When custom upstreams are present, the
original locked path is taken unchanged.

**Result.** Re-profiled under the identical load: `CustomUpstreamConfig` is
**gone** from the mutex profile, and total mutex-wait delay dropped **−45%**
(1,336 s → 733 s). With the dominant lock removed, the next contention point is
now visible (`DefaultAddrProc.process`, the client address/WHOIS path) — the
subject of follow-up profiling. The episode is a clean illustration of the
project's profile-before-patch discipline: the fix targeted the lock the *data*
identified, not the one first suspected.

---

### 7.10 Real-Time Query Feed to the Edge WAF (`qfeed`)

The edge deployment runs an L7 anti-abuse layer (an XDP + userspace WAF) in front
of the resolver. Historically it observed DoT/DoQ behaviour by *tailing AGH's JSON
query log* — a lossy, flush-lagged signal (tens of seconds behind real time) whose
per-query write was itself a lock-contention source. The edge build replaces that
with a purpose-built **real-time per-query feed**.

**Design.** A new `qfeed` package emits one tiny fire-and-forget binary event per
query to the WAF over an `AF_UNIX` / `SOCK_DGRAM` socket. Each record is a fixed
**36-byte little-endian** frame — client IP (16 B, IPv4-mapped-in-IPv6), source
port, an **XXH3-64 hash of the normalized query name**, response code, transport
(`Do53`/`DoT`/`DoQ`/`DoH`), and a nanosecond timestamp. **The query name never
crosses the wire** — only its hash — preserving the zero-retention privacy posture
while still letting the WAF measure distinct-name cardinality and repetition per
source.

**The resolver never blocks on it.** The hot path (beside querylog/stats) does a
single non-blocking send of a stack-built `[36]byte` into a buffered channel — no
heap allocation, no mutex, nanoseconds. A background goroutine drains the channel
and writes datagrams, dialing the WAF's socket lazily and re-dialing with backoff
if the WAF restarts. When the channel is full (WAF slow or down) the send takes the
`default` branch and the record is **dropped** — that drop *is* the intended shed;
a query is never delayed or failed by the feed. The emitter is a process-wide
singleton (so reconfiguration never leaks a writer goroutine), nil-safe, and can be
disabled by an environment variable without a rebuild.

**Why this matters (co-design).** The same per-query write that was a *liability*
inside the query log becomes, as a lock-free unix-socket tap, a real-time
*enforcement signal*. It removes the WAF's dependency on the flush-lagged log and
turns its volumetric scorer — query rate, NXDOMAIN ratio, and distinct-name spray
per source over a sliding window — from a slow sustained-abuse brake into a
real-time one. One mechanism, two problems addressed: the lossy log tail retired
and the WAF given a live `Do53`/`DoT`/`DoQ`/`DoH` window. The feed is detect-only
by design, and the query log itself is retained unchanged for the admin UI — this
runs alongside it as a parallel tap.

Validated end-to-end against the live WAF: records decode with **zero** errors and
the WAF's per-protocol counters track the real `Do53`/`DoT`/`DoQ`/`DoH` traffic
split the moment the resolver is deployed.

---

### 7.11 Goodput-Collapse Trilogy — Per-Query Serve-Path Locks (2026-06-04)

**Problem.** Under a sustained DoT flood the resolver exhibited *accept-then-collapse*:
goodput **fell** as offered load rose (≈10,900 answers/s at concurrency 100 down to
≈2,700 at concurrency 600) while AGH CPU **dropped** from ≈240% to ≈100% on a 4-core
box that never approached saturation. CPU falling under rising load is the signature of
goroutines going **off-CPU** — parking on a lock, not running out of compute. A mutex
profile at the collapse regime (c=500 loopback generator, `SetMutexProfileFraction(1)`,
pprof bound to localhost) measured **3,470 s of mutex-wait delay** over the window,
concentrated in three global locks taken on *every query* on the serve path:

| Lock | Share of mutex delay |
|---|---|
| query-log `bufferLock` (per-query buffer push) | **42%** |
| statistics `currMu` (per-query counter update) | **23%** |
| client `Storage` RWMutex via `UpdateAddress` (WHOIS enrichment write) | **20%** |

Notably the query-log lock — only ~9% at low concurrency, where it had earlier been
dismissed as minor — became the **dominant** offender at the collapse regime, because a
single global mutex serialises *harder* as concurrency rises. The three were removed in
sequence, re-profiling after each (profile-before-patch); each fix exposed the next lock,
exactly as predicted.

**Fix 1 — query log: async single-consumer (`querylog`).** `Add` no longer takes
`bufferLock`. It does a non-blocking send of the stack-built entry into a buffered channel;
a single background consumer is the *only* goroutine that pushes to the ring buffer, so the
lock is never contended across the many serve goroutines. The channel is FIFO (log order
preserved) and the consumer batches up to 1024 pushes per lock acquisition so search readers
are not starved. A full channel drops the entry — shedding *bookkeeping* under overload,
never the DNS answer. Graceful shutdown/reconfigure drains the queue first.

**Fix 2 — statistics: sharded current unit (`stats`).** The per-query `currMu.Lock()` that
serialised every counter update is replaced by 16 independently-locked shards. `Update` takes
a *read* lock on the set (so updates no longer mutually exclude) plus one round-robin shard
lock; reads and the hourly flush merge the shards. Counters are commutative, so the merge is
exact. The `unit` type and its serialization are unchanged — the sharding is contained to the
stats container.

**Fix 3 — WHOIS: bounded per-IP dedup cache (`whois`).** With the first two locks gone, the
entire residual contention (290 s, **90%**) was `Storage.UpdateAddress` taking the client
storage *writer* lock. Root cause: the edge fork's GeoIP-backed `whois` lookup (local MaxMind
ASN+City databases, no external WHOIS by design — see §3) reported `changed=true` on *every*
call when an IP had GeoIP data, having never carried a cache. So the address processor
re-enriched every query's source IP and `UpdateAddress` took the write lock per query, its
`host=="" && info==nil` fast path never engaging. This was confirmed **source-IP-cardinality
independent** — a 4-distinct-IP flood contended identically to a 16-million-IP one — and
therefore real for production traffic, where every real client IP resolves in GeoIP. The fix
adds the cache it always lacked: a two-generation bounded set (clock eviction, capped at
2¹⁶ per generation) of already-enriched IPs. A repeat IP returns `changed=false`, so
`UpdateAddress` hits its lock-free fast path — an IP's GeoIP data is static for the lifetime
of the databases — also eliminating the redundant per-query GeoIP lookups.

**Result.** Re-profiled under the identical c=500 load after each fix:

| Stage | Total mutex-wait delay | Top remaining lock | Goroutines parked on a lock |
|---|---|---|---|
| baseline | 3,470 s | query-log 42% | 207 |
| + async query log | 883 s (**−75%**) | stats 55% | (shifted) |
| + sharded stats | 324 s (**−91%**) | Storage 90% | 227 |
| + WHOIS cache | **15.75 s (−99.5%)** | none (scheduler noise) | **0** |

`Storage.UpdateAddress` fell **290 s → 3.6 s**. With every per-query serve-path lock removed,
**no goroutine parks on a lock** under the flood — the off-CPU collapse signature is gone and
the resolver is CPU-bound rather than lock-bound. Real-user query latency at the worst tested
concurrency (c=600) improved from **869 ms → 163 ms**, and goodput rose at every step
(c=600: 2,693 → 4,414 answers/s across the four binaries), with zero failed queries throughout.
Each fix is feature-flagged and behaviour-preserving when disabled (`AGH_QLOG_ASYNC`,
`AGH_WHOIS_CACHE`; the stats sharding is always on). The trilogy is a second clean instance of
the project's profile-driven discipline — three locks the *data* named, removed in the order it
named them.

### 7.12 Blocked-Response TTL Lock — Read Under the Writer Lock (2026-06-04)

A follow-on of the same bug class, surfaced by profiling the cache-enabled serve path (§7.13)
under a 26k-qps replay of real query names: once the front cache made the rest of the path fast,
the **top mutex contention (568 s, 86.8% of all mutex delay)** was
`filtering.(*DNSFilter).BlockedResponseTTL` — a read-only accessor that took the **exclusive**
`confMu.Lock()` to read a single `uint32`, serializing **every blocked query** on the filter-config
mutex. Its sibling getters (`BlockingMode`, protection-status) already used `RLock`; this one was
the lone outlier. Changed to `RLock` (writers are rare config changes only) — the serialization is
gone with no behaviour change. It matters under blocked-domain load (filtering-heavy traffic, or a
flood aimed at blocklisted/tracker domains).

### 7.13 AGH Front-Cache — Delegated Caching Reconsidered (2026-06-04)

The deployment historically ran AGH with **`cache_enabled: false`**, delegating all caching to the
on-box unbound (§6) to avoid a redundant second cache. Profiling the post-pooling resolver (§8.1)
showed that decision had a cost: with AGH caching nothing, **every** query — including ones unbound
would serve from *its* cache — still paid a full localhost round-trip (serialize → socket write →
unbound lookup → read → parse), measured at **~31% of CPU** under load.

**Measured the real hit rate first.** unbound's organic cache-hit rate (delta over a quiet window,
load-test traffic excluded) was **57%** — i.e. ~57% of real queries are repeats an AGH front-cache
would absorb without the hop. (The lifetime 98% figure was pollution from single-domain load tests.)

**Enabled** `cache_enabled: true` with **`cache_ttl_min: 0`** — the latter deliberately so AGH
respects real upstream TTLs and does **not** pin short-TTL CDN/GSLB answers (the one real regression
risk of a second cache). It is DNSSEC-safe (the cache keys on the DO bit), filtering still runs
*before* the cache, and EDNS-Client-Subnet is stripped so there is no per-subnet key explosion. The
cache is `glcache` — an in-memory LRU in the Go heap (no disk; `cache_size` is a RAM byte-budget).

**A/B (dnsperf replay of 25k real querylog names, 4,729 distinct, real popularity; UDP/53):**

| metric | cache off | cache on | Δ |
|---|---|---|---|
| throughput | 16,438 qps | 26,241 qps | **+60%** |
| avg latency | 18.1 ms | 11.2 ms | −38% |
| queries reaching unbound | 289,538 | 5,439 | **−98%** |

The looped replay compresses time, so its hit rate (~99%) overstates the organic 57%; the
scale-invariant benefit is that **~57% of real queries now skip the upstream hop**. Crucially, cache
hit rate *rises with traffic* (more users → more domain overlap), so this is the rare optimization
that pays off **more** as the resolver grows — and it also blunts the cacheable-flood class (a
repeated-domain flood is absorbed at the front). It does **not** help against cache-busting
(random-subdomain) floods — that remains the WAF/rate-limit layer's job. unbound stays the deep
cache (RRSET cache, prefetch, DNSSEC-validation cache, serve-stale); the AGH cache is just a fast
hot-set skimmer in front.

### 7.14 Lock-Free Per-Query Client-IP Enqueue (2026-06-05)

With the serve-path, UDP, and cache locks gone, a mutex profile put
`client.(*DefaultAddrProc).Process` at the top (~52% of mutex-delay). That method runs on **every**
query to hand the client IP to the background rDNS/WHOIS enrichment worker, and it took a global
`sync.Mutex` for one reason: to stop a concurrent `Close` from closing the queue channel mid-send (a
send on a closed channel panics). So every query goroutine serialized on a single lock to do a
nanosecond channel push.

**Fix.** Apply the standard multi-sender/one-closer pattern: never close the data channel from the
senders. Shutdown is signaled by closing a separate `closed` channel (once, via `sync.Once`); the
worker `select`s on it, drains the queue, and exits, while `Process` becomes a lock-free non-blocking
`select` over *send / closed / default-drop*. The queue-full drop counter and the
`ErrClosed`-on-double-close contract are preserved, and the change is race-clean under `go test -race`.

**Result (A/B, loopback dnsperf, before vs after, deployed):**

| load | qps before | qps after | Δ qps | avg before | avg after | tail (max) before | tail after |
|---|---|---|---|---|---|---|---|
| standard (`-c4 -q300`) | 27,788 | 31,302 | **+12.6%** | 10.62 ms | 9.36 ms | 1.69 s | **0.67 s** |
| heavy (`-c8 -q800`) | 28,144 | 28,969 | **+2.9%** | 28.29 ms | 27.48 ms | 2.04 s | **0.79 s** |

0 loss. The standout is **tail latency: −60% on both load levels** (max ~1.7–2.0 s → sub-second) — the
per-query head-of-line stalls on the global lock are gone. Re-profiled, `DefaultAddrProc.Process` is
absent from the mutex profile; the next contender is dnsproxy's recursion-detector cache
(`recursionDetector.check`). Unlike the cache shard (§8.4), this lock sat on the unconditional per-query
path, so removing it moved real throughput and latency, not just the profile. (AGH `58fc05a8`)

---

## 8. Performance Engineering — Transport Layer (dnsproxy)

Summary of all transport-layer changes in the
[edge fork](https://github.com/Ozy-666/dnsproxy) (`edge-udp-pool` branch,
based on upstream `v0.81.4`):

| Commit | Change | Impact |
|---|---|---|
| `bbd79ad` | `atomic.Bool` for server started state; TCP/DoT pool (65537-byte slots) | State check: lock-free. TCP: **4 allocs/RTT → 0** |
| `0b14b22` | Upstream RTT mutex → `atomic.Pointer` copy-on-write map | Weight calculation lock-free; upstream dispatch no longer serialised |
| `00fc061` | DoH POST body: unbounded read → capped at DNS wire maximum | Memory-exhaustion POST flood mitigated; 413 on oversized body |
| `716e780` | QUIC stream limit 65535 → configurable, default 64 | Stream-flood attack surface eliminated at transport layer |
| `7363632` | Plain (UDP/TCP) upstream connection **pool** — reuse instead of dial-and-close per query | Per-query `connect()`/`close()` syscalls eliminated; **goodput ≈ doubled** at high concurrency (§8.1) |

### 8.1 Plain Upstream Connection Pooling

When the front-end resolver's own cache is disabled — as in this deployment, where caching is
delegated to the on-box unbound at `127.0.0.1:5353` (§6) — AGH forwards *every* query upstream.
A CPU profile of the deployed resolver under a DoT flood (loopback generator, after the §7.11 lock
removal left it network-IO bound) showed `net.Dialer.DialContext` at **~19% of all CPU**: dnsproxy's
`plainDNS` dialed a fresh upstream connection and closed it on *every* exchange — pure
socket/connect/close syscall overhead, the single largest avoidable cost remaining.

**Fix.** A per-upstream, per-network LIFO connection pool. A connection is checked out for the
*exclusive* duration of one exchange (write + read) and returned only after a clean, validated
response — never shared concurrently, which would let DNS responses cross-talk between in-flight
queries. A pooled connection that errors (stale: upstream restart or idle keep-alive close) is
discarded and the exchange retried once with a fresh dial. The pool is bounded (1024 idle
connections per network; overflow closed on return) and drained on `Close`. Feature-flagged:
`DNSPROXY_PLAIN_POOL=0` restores dial-per-query.

**Result (A/B, identical DoT loopback ramp, before vs after, deployed):**

| concurrency | answers/s before | answers/s after | Δ | real-user canary |
|---|---|---|---|---|
| c=100 | 10,672 | 14,975 | **+40%** | 58 → 44 ms |
| c=200 | 7,319 | 11,110 | **+52%** | 65 → 59 ms |
| c=400 | 4,775 | 9,605 | **+101%** | 99 → 100 ms |
| c=600 | 3,196 | 6,126 | **+92%** | 306 → 151 ms |

Re-profiled, `DialContext` is **gone** from the CPU profile and total CPU *fell* (245% → 216%) while
throughput roughly doubled — the per-query dial was a blocking `connect()` serialization, not merely
CPU. Connection-count sampling confirmed the upstream sockets plateau at a bounded count per
concurrency level and are reused, instead of churning thousands of dials per second. The remaining CPU
is genuine work: the downstream TLS response write (~36%) and the upstream exchange (~31%) — the
irreducible floor for a forwarding resolver without front-end caching.

### 8.2 Debug-Log Serialization Off the Hot Path (2026-06-05)

A CPU profile of the cache-on resolver under a plain-UDP flood (loopback `dnsperf`, 25.8k qps, 0 loss)
showed `(*Proxy).logDNSMessage` at **~11% of all CPU** — `~14%` of the per-query pipeline — dominated by
`miekg/dns.(*Msg).String()` (the full text serialization of the message) plus the string-concat
allocations feeding it. The function ran on the hot path of *every* query:

```go
// before — proxy/server.go
slogutil.PrintLines(ctx, p.logger, slog.LevelDebug, msg, m.String())
```

`m.String()` was evaluated **unconditionally and twice per query** (once for the request, once for the
response), then handed to a `LevelDebug` log call that discards it whenever debug logging is off — the
production default (`verbose: false`). The handler dropped the string, but the work to build it was
already spent.

**Fix.** Guard the serialization behind the standard slog level check, so the message is never stringified
unless a debug sink is actually listening. Behavior is byte-for-byte identical when `verbose: true`.

```go
// after
if !p.logger.Enabled(ctx, slog.LevelDebug) {
	return
}
```

**Result (A/B, identical UDP loopback load, before vs after, deployed):**

| metric | before | after | Δ |
|---|---|---|---|
| throughput | 25,774 qps | 27,161 qps | **+5.4%** |
| avg latency | 11.49 ms | 10.84 ms | **−5.7%** |
| loss | 0 | 0 | — |
| `logDNSMessage` in CPU profile | ~11% of total | **absent** | path eliminated |

Re-profiled, `logDNSMessage` and `Msg.String()` are **gone** from the CPU graph. The throughput/latency
gain is modest because this generator is concurrency-capped (`-q 300`), not CPU-saturated — the box ran
at ~236–257% of 400% in both runs, so the reclaimed CPU surfaces as lower per-query latency and headroom
rather than a proportional qps jump. Under a CPU-bound flood the same ~11% would surface directly as
capacity. The durable result is structural: a provably-dead serialization is no longer executed on the
serve path. Strictly an improvement, upstreamable to dnsproxy. (fork `1e17078`)

### 8.3 UDP Listener Sharding — SO_REUSEPORT per Core (2026-06-05)

dnsproxy listened on a **single** UDP socket per address, with one reader goroutine and — more
importantly — one writer path. Every response `sendmsg` took that socket's `internal/poll.fdMutex`
write lock, so all concurrent in-flight responses serialized through a single mutex. A block profile
under load put `(*fdMutex).rwlock` at **~43% of all block-delay** — the dominant contention point once
the §7.11 serve-path locks and §8.2 logging were gone. The throughput signature was unmistakable: a
**heavier** concurrency level produced **lower** throughput than a lighter one (28,143 vs 30,339 qps),
the classic shape of a saturating lock.

**Fix.** Open `GOMAXPROCS` UDP sockets per listen address instead of one. `SO_REUSEPORT` is already set
on every socket by dnsproxy's `ListenConfig`, so the sockets co-bind the same `[::]:53` and the kernel
fans inbound datagrams across the independent file descriptors by flow hash. `server.go` already starts
one `udpPacketLoop` per socket, and each response is written back on the socket its query arrived on
(`DNSContext.Conn`), so the per-fd write lock splits N ways for both read and write. Source-address
correctness on the unspecified bind is preserved per-socket via the existing OOB `localIP` path.
Override via `DNSPROXY_UDP_SHARDS` (`=1` restores the single-socket upstream behavior); default is
`GOMAXPROCS` (4 here).

**Result (A/B, loopback dnsperf, single socket vs 4 sockets, deployed):**

| load | qps before | qps after | Δ qps | avg lat before | avg lat after | tail (max) before | tail after |
|---|---|---|---|---|---|---|---|
| standard (`-c4 -q300`) | 30,339 | 31,463 | **+3.7%** | 9.73 ms | 9.32 ms | 1.70 s | 1.80 s |
| heavy (`-c8 -q800`) | 28,143 | 31,351 | **+11.4%** | 28.28 ms | 25.39 ms | 1.75 s | **0.82 s** |

0 loss throughout. The structural win is clearest in the heavy column: before, heavy throughput
*collapsed below* standard (the contended single lock); after, heavy ≈ standard (31,351 ≈ 31,463) — the
collapse is gone — and tail latency under load fell **−53%** (1.75 s → 0.82 s) as the head-of-line
blocking on the shared write lock disappeared. Naturally bounded at core count; more sockets than cores
adds reader goroutines without splitting more work. Upstreamable to dnsproxy as an opt-in. (fork `3927e02`)

### 8.4 Request-Cache Sharding (2026-06-05)

With the serve-path locks (§7.11), the debug-log serialization (§8.2), and the UDP write `fdMutex`
(§8.3) all removed, a fresh mutex profile put the **single response cache** at the top: `proxy.(*cache).
set` took a global `itemsLock` *exclusively* on every store (blocking all readers), and even reads
serialized on the underlying glcache's own mutex (an LRU reorders on every `Get`). Together ~62% of
mutex-delay in the instrumented profile.

**Fix.** Stripe the main requests cache into a power-of-two number of shards (default 16), each with its
own `RWMutex` and LRU, selected by a `maphash` of the cache key. The shard count adapts *down* so every
shard's `MaxSize` still admits a full-size DNS message (glcache caps element size at the shard size), and
an unsized or small cache stays single — byte-for-byte the previous behavior. The cold EDNS-CS cache is
left unsharded (its longest-prefix lookup probes several keys per get, which doesn't fit single-key
striping). Override via `DNSPROXY_CACHE_SHARDS` (`=1` disables). Race-clean under `go test -race`.

**Result — honest:** re-profiled, `proxy.(*cache).set`/`get` are **gone** from the mutex profile; the
lock is provably eliminated. But end-to-end throughput is **flat** at the tested loopback load
(standard +4% with max latency halved 2.01 s → 1.02 s; heavy −2.8% — all inside run-to-run variance).
The ~62% figure was inflated by pprof's own mutex instrumentation; **without** it the box is not
cache-lock-bound at this load, so splitting the lock removes the contention and adds headroom without
raising the ceiling *here*. Kept because it is correct, low-risk, transport-independent (DoT/DoH hit the
same cache), and provides headroom for higher-concurrency regimes — not because it moved throughput
today. This is the honest shape of a contention fix on a path that isn't currently the bottleneck.
(fork `6645070`)

> **Method note.** This is why each lever is profiled *after* the previous one ships: removing the §8.3
> `fdMutex` reshuffled the contention ranking and surfaced the cache lock; removing the cache lock in turn
> surfaced per-query client-IP processing (`DefaultAddrProc`, ~52%) and the recursion-detector cache as
> the next contenders. Mutex-profile percentages are contention *rank*, not a throughput promise.

### Consolidated Benchmark Results

All benchmarks run on AMD EPYC 7542, 4 parallel goroutines,
typical NXDOMAIN/A-record response.

| Path | Before | After | Delta |
|---|---|---|---|
| UDP write (per response) | 255 ns · 160 B · **1 alloc** | ~282 ns · **0 B** · **0 allocs** | −1 alloc · −160 B |
| TCP round-trip (read+write) | — · ~512 B · **4 allocs** | ~251 ns · **0 B** · **0 allocs** | −4 allocs · −512 B |
| Server state check | RLock/RUnlock on every keepalive | Atomic load (no lock) | Thundering-herd on shutdown eliminated |
| Upstream weight calculation | Exclusive mutex on every query | Atomic pointer load (no lock) | Contention at dispatch point eliminated |

---

## 9. Security Hardening

| Area | Change | Severity |
|---|---|---|
| **DoH POST flood** | Body capped at DNS wire-format maximum (65,535 bytes); `413` returned before any DNS unpacking | Medium — memory exhaustion under targeted POST flood |
| **QUIC stream flood** | Per-connection stream limit configurable, default 64 (was 65,535); enforced by QUIC transport `MAX_STREAMS` frame | High — single client could drain global request semaphore, starving all others |
| **Cloud telemetry** | SafeBrowsing, Parental Controls, EDNS-CS all removed; no DNS query data leaves the local machine | Privacy — eliminates data leakage to third-party cloud services |
| **Auto-update block** | `-edge` suffix detected at runtime; auto-update disabled; upstream release cannot overwrite patched binary | Integrity |
| **ODoH removed (dnscrypt-proxy)** | Oblivious DoH stripped entirely (~800 LOC: crypto, transport, target-config fetch, stamps, `odoh_servers` option) | Surface reduction — removes an HPKE-style crypto/wire path and a content-type handler |
| **No discretionary egress (dnscrypt-proxy)** | Remote source-list downloads disabled (local signed cache only); upstreams pinned by `[static]` stamps; no auto-update / version-check call exists (audited) | Privacy — only encrypted DNS queries leave the host |
| **Transport timeout tightening (dnscrypt-proxy)** | 30 s default transport timeouts wired to the 800 ms query budget; h2 idle health-check decoupled (10 s read-idle / 5 s ping) | Resource — bounds connect/read and reaps dead keep-alive connections promptly |
| **Whois TCP connections** | Replaced with local mmdb; no per-query outbound TCP | Privacy — eliminates external connection from query log enrichment |
| **Data race fix** | Access manager field in middleware read without synchronization; fixed to atomic load | Correctness — undefined behaviour under Go memory model |
| **GLiNet path traversal (CVE-2026-41448)** | Backported the upstream v0.107.77 fix: GLiNet auth-token file reads confined to an `os.Root` (rejects `..`/absolute escapes), token filename reduced to a basename. Not exploitable in this deployment (runs without `--glinet`, so the GLiNet middleware is never installed); applied as defense-in-depth and to keep the fork mergeable | Medium — admin-auth bypass via a crafted `Admin-Token` cookie in GLiNet mode |

---

## 10. Configuration Reference

Key fields specific to the Edge build. All fields live under their respective
top-level sections in `AdGuardHome.yaml`.

### `dns` section

| Field | Default | Valid range | Description |
|---|---|---|---|
| `quic_max_incoming_streams` | `64` | `[1, 1024]` or `0` | Max concurrent QUIC streams per connection for DoQ/DoH3. `0` silently maps to 64. Values outside `[1, 1024]` log a warning and fall back to 64. Introduced in schema v35. |
| `upstream_timeout` | `1s` | any duration | Per-request upstream deadline. Should be set below dnscrypt-proxy's own timeout to avoid wasted post-deadline work. |
| `max_goroutines` | `300` | positive int | Maximum parallel DNS request goroutines. |
| `cache_enabled` | `true` | bool | AGH front-cache (§7.13). Enabled to skip the localhost hop to unbound for the hot set (~57% of organic queries). `glcache` in-memory LRU. |
| `cache_ttl_min` | `0` | seconds | Minimum TTL AGH pins on cached answers. Kept at **0** so real upstream TTLs are respected — never pin short-TTL CDN/GSLB answers. |
| `cache_size` | `4194304` | bytes | RAM byte-budget for the front-cache (heap, counts toward `GOMEMLIMIT`). Raise (e.g. 32–64 MB) as traffic grows so the hot set keeps fitting. |

### `http` section

| Field | Default | Description |
|---|---|---|
| `socket_path` | (unset) | When set, the plain HTTP server binds to a Unix domain socket instead of a TCP address. Required for nginx UDS proxying. |

### Environment flags

| Variable | Default | Description |
|---|---|---|
| `AGH_QFEED` | (enabled) | The real-time WAF query feed (§7.10). Set to `0`/`off`/`false`/`no` to disable the emitter without a rebuild. Harmless when the WAF socket is absent — records simply drop. |
| `AGH_QLOG_ASYNC` | (enabled) | The asynchronous single-consumer query-log add path (§7.11). Set to `0` to restore the synchronous per-query `bufferLock` push. |
| `AGH_WHOIS_CACHE` | (enabled) | The bounded per-IP WHOIS dedup cache (§7.11). Set to `0` to restore the original always-`changed` GeoIP lookup. |
| `DNSPROXY_PLAIN_POOL` | (enabled) | Plain UDP/TCP upstream connection pooling (§8.1). Set to `0` to restore dial-per-query. |

> **Runtime memory ordering.** `GOMEMLIMIT` is pinned to **380 MiB**, deliberately *below* the
> cgroup `MemoryHigh=400 MiB` throttle, so Go's GC reclaims under memory pressure before the kernel
> begins synchronous reclaim on the process. The reverse ordering (limit above the throttle) lets the
> kernel stall the resolver before the runtime ever acts — a latent footgun caught while profiling the
> post-lock IO ceiling (live heap there was only ~63 MB, so it had not yet bitten).

### Schema versions

| Version | Change |
|---|---|
| v35 | Added `dns.quic_max_incoming_streams` (no-op data migration; absent field silently maps to default 64) |
| v34 | DoH route configuration moved into `http.doh` sub-section |

---

## 11. Release History

> **Note (2026-06-03):** the binary version string was rebased from the internal
> edge-increment scheme onto the upstream base tag — the deployed build now reports
> `v0.107.77-edge` — so AGH's own update checker stops flagging the upstream
> v0.107.77 release (the `-edge` suffix already disables auto-update; this aligns
> the *base* version too). The three rows below the rebase therefore carry the real
> binary version rather than the older monotonic counter.

| Version | Date | Summary |
|---|---|---|
| `v0.107.77-edge` | 2026-06-04 | **perf:** AGH front-cache **enabled** (`cache_enabled:true`, `cache_ttl_min:0`) — measured 57% organic hit rate; A/B replay of real querylog names **+60% throughput / −98% backend load**; DNSSEC-safe, respects real TTLs; hit-rate rises with traffic so it scales with growth (§7.13). Plus `filtering.BlockedResponseTTL` exclusive-lock→`RLock` fix (was 86% of mutex delay under cache-on load once the cache made the rest fast) (§7.12) |
| `v0.107.77-edge` | 2026-06-04 | **perf:** plain-upstream **connection pooling** (dnsproxy fork `7363632`) — reuse UDP/TCP upstream connections instead of dial-and-close per query; eliminated the ~19% per-query `connect()` CPU the post-lock profile exposed. A/B: goodput **+40…+101%** across the c=100…600 ramp (≈doubled at c=400/600), c=600 canary 306→151 ms (§8.1). Also pinned `GOMEMLIMIT=380 MiB` below the cgroup `MemoryHigh=400 MiB` so Go GC precedes kernel reclaim (§10) |
| `v0.107.77-edge` | 2026-06-04 | **perf:** goodput-collapse trilogy — three per-query serve-path locks removed in profile-named order (query-log `bufferLock` → async single-consumer; statistics `currMu` → 16-way sharded unit; client `Storage.UpdateAddress` write-storm → bounded per-IP WHOIS dedup cache). Total mutex-wait delay under a c=500 DoT flood **3,470 s → 15.75 s (−99.5%)**; zero goroutines park on a lock (lock-bound collapse → CPU-bound); real-user tail latency at c=600 **869 ms → 163 ms** (§7.11) |
| `v0.107.77-edge` | 2026-06-03 | **feat:** shield real-time per-query feed (`qfeed`) — a 36-byte fire-and-forget unix-datagram event per query (XXH3 name *hash* only; the name never leaves the host), non-blocking lock-free emit with drop-on-backpressure shedding, replacing the WAF's flush-lagged querylog tail with a live `Do53`/`DoT`/`DoQ`/`DoH` window (§7.10) |
| `v0.107.77-edge` | 2026-06-03 | **perf:** per-query `client.CustomUpstreamConfig` writer-lock — measured at **79%** of all mutex-wait delay under a DoT flood — replaced with a lock-free `atomic.Bool` fast path; total mutex-wait **−45%**, the dominant DoT/DoQ contention point removed (§7.9) |
| `v0.107.77-edge` | 2026-06-03 | **security:** backported the upstream **v0.107.77** GLiNet path-traversal fix (CVE-2026-41448) via `os.Root` confinement; base rebased to v0.107.77. Not exploitable here (no `--glinet`); applied as defense-in-depth (§9) |
| Unbound / infra | 2026-05-29 | RPS disabled on the loopback interface (it was steering the all-loopback stack across all 4 cores → cross-CPU IPI storm); measured −14% box CPU and −30…−40% function-call IPIs under load at zero latency cost. NIC RPS retained (§6.5) |
| Unbound / infra | 2026-05-29 | Unbound: jemalloc actually linked via `LD_PRELOAD` (the `--with-libjemalloc` build flag was a silent no-op → glibc malloc), arenas tuned (`narenas:4`, `background_thread`); cache slabs 4→8; AppArmor THP/`net_admin` denials fixed. Profiled under 600× load — confirmed kernel-bound with large headroom, source patching assessed and declined (§6.1–6.4) |
| dnscrypt-proxy fork | 2026-05-28 | Transport timeouts wired to the 800 ms query budget + h2 idle health-check decoupled (10 s read-idle / 5 s ping); precomputed EDNS0 padding (per-DoH-query alloc removed); no-version-check / no-auto-update audit confirmed; live WP2 vs `fastest`/`p2` A/B (WP2 wins latency **and** throughput) |
| dnscrypt-proxy fork | 2026-05-27 | ODoH stripped (~800 LOC); remote source-list downloads disabled (local signed cache only) + `[static]` stamp pinning; WP2 lock-free `getOne` (`RLock`) + dormant-recovery moved to maintenance goroutine; hot-path UDP buffer pooling + lazy session map |
| `v0.107.109-edge` | 2026-05-25 | urlfilter fork: AST required-literal shortcut extraction indexes host-level regexp rules, closing the empty-shortcut `noIndex` O(N) cache-miss vector (flood N=1000 111µs→449ns, 248×); alternation gate prototyped and rejected by benchmark; verified by equivalence harness (0/39,983) + fuzz |
| `v0.107.108-edge` | 2026-05-24 | dnsproxy fork: QUIC unidirectional stream limit decoupled from the bidirectional flood cap (fixed 64) so a low cap can't break DoH3 control/QPACK streams (audit W1); inert hardening |
| `v0.107.107-edge` | 2026-05-24 | filtering: match cache v2 — lock-free fixed-size table replaces CoW map; unique-domain miss 350µs·489KB → 2.1µs·272B (167× / 1797×); closes DoS-amplification vector (audit C1) |
| `v0.107.106-edge` | 2026-05-24 | filtering: CoW match cache; warm hit 51 ns · 0 allocs (down from 3225 ns · 9 allocs); O(N_regex) scan eliminated for repeated queries |
| `v0.107.105-edge` | 2026-05-23 | filtering: `ufReqPool` + `dnsResPool` pools in `matchHost`; −3 allocs/op, −65 B/op on DNS hot path |
| `v0.107.104-edge` | 2026-05-23 | `dns.quic_max_incoming_streams` exposed in `AdGuardHome.yaml`; schema v34→v35 |
| `v0.107.103-edge` | 2026-05-23 | dnsproxy fork: configurable QUIC stream limit, default 64, range [1,1024] |
| `v0.107.102-edge` | 2026-05-23 | dnsproxy fork: DoH POST body bounded to DNS wire maximum |
| `v0.107.101-edge` | 2026-05-23 | dnsproxy fork: RTT mutex → atomic copy-on-write map |
| `v0.107.100-edge` | 2026-05-23 | dnsproxy fork: atomic server state; TCP/DoT zero-alloc pool (4 allocs → 0) |
| `v0.107.99-edge` | 2026-05-23 | dnsproxy fork: zero-alloc UDP pool; rebased to upstream v0.81.4 |
| `v0.107.98-edge` | 2026-05-22 | Querylog pack buffer pool; filter benchmark baseline established |
| `v0.107.97-edge` | 2026-05-22 | Infrastructure: dnscrypt-proxy timeout 2500 ms → 800 ms |
| `v0.107.96-edge` | 2026-05-22 | Filtering hot path lock-free via `atomic.Pointer` |
| `v0.107.95-edge` | 2026-05-22 | Client IP processing lock narrowed to two-line interface snapshot |
| `v0.107.94-edge` | 2026-05-21 | Access check lock-free via `atomic.Pointer`; data race in middleware fixed |
| `v0.107.93-edge` | 2026-05-21 | Config disk-write shallow copy → selective field assignment |
| `v0.107.92-edge` | 2026-05-21 | Filter settings per-request allocation pooled; tag slice backing array reused |
| `v0.107.91-edge` | 2026-05-21 | Address processor drop counter; warn-on-first-drop observability |
| `v0.107.90-edge` | 2026-05-21 | Context propagation: query log and address processor accept caller context |
| `v0.107.89-edge` | 2026-05-21 | Query log fast-paths: skip storage lookup when disabled or host ignored |
| `v0.107.88-edge` | 2026-05-21 | Log rotation goroutine lifecycle bounded; leak on Start/Shutdown fixed |
| `v0.107.87-edge` | 2026-05-21 | 100 ms sleep under global lock on config reload removed |
| `v0.107.86-edge` | 2026-05-21 | Stats render accumulator maps pre-sized |
| `v0.107.85-edge` | 2026-05-21 | Query log buffer lock scope narrowed; flush-stuck-flag bug fixed |
| `v0.107.84-edge` | 2026-05-21 | Client storage: exclusive mutex → reader-writer mutex |
| `v0.107.83-edge` | 2026-05-21 | Per-request logger allocation eliminated via level-gate fast path |
| `v0.107.82-edge` | 2026-05-21 | Upstream stats slice: nil+append → stack-backed fixed array |
| `v0.107.81-edge` | 2026-05-21 | Query log type/class string lookups: map → fixed array |
| `v0.107.80-edge` | 2026-05-21 | Filter result double-allocation eliminated |
| `v0.107.79-edge` | 2026-05-21 | Global lock scope in stats/log path narrowed to pointer snapshot |
| `v0.107.78-edge` | 2026-05-21 | Stats update corrected to read lock; redundant client lock removed |
| `v0.107.77-edge` | 2026-05-21 | Response filter N per-RR locks → single lock over answer loop |
| `v0.107.76-edge` | 2026-05-21 | Pipeline dispatch array; domain name cache; IPv4-mapped fix; ID slice stack |
| `v0.107.75-edge` | 2026-05-19–21 | Initial edge build: full feature strip; Unix socket support; local MaxMind whois; auto-update block |

---

## 12. Completeness Status

The initial architectural audit produced 20 tracked items across 9 categories.
Three additional items were added from profiler-driven analysis post-audit.
**All 23 items are complete as of v0.107.106-edge.**

| Category | Items | Status |
|---|---|---|
| Memory Management (§1) | 11 | ✅ All complete |
| Concurrency & Locks (§2) | 7 | ✅ All complete |
| Timeouts & Lifecycles (§3) | 4 tracked, 1 closed as N/A | ✅ Complete |
| Architectural Inefficiencies (§4) | 7 | ✅ All complete |
| serverLock Elimination (§5) | 3 | ✅ All complete |
| Infrastructure Tuning (§6) | 1 | ✅ Complete |
| Pack Buffer Pools (§7) | 2 | ✅ Complete |
| dnsproxy Structural (§8) | 2 | ✅ Complete |
| dnsproxy Remaining Audit (§9) | 3 | ✅ All complete |
| urlfilter noIndex regexp scan (§7.7) | 1 | ✅ Resolved via AST shortcut extraction (v0.107.109) |
| Post-audit hardening — match cache v2 (§7.8) | 1 | ✅ Complete (C1, v0.107.107) |
| Post-audit hardening — QUIC uni-stream decouple (§4.5) | 1 | ✅ Complete (W1, v0.107.108) |
| Production profiling — per-query client-storage lock (§7.9) | 1 | ✅ Complete (2026-06-03; −79% of mutex delay) |
| Co-design — real-time WAF query feed (§7.10) | 1 | ✅ Complete (2026-06-03) |

No open items remain. The `urlfilter` `noIndex` regex-gate optimization (§7.7) was
measured against the real filter lists and shelved as unwarranted for the DNS
workload. Future work is driven by profiling data from sustained production traffic
or new upstream dnsproxy releases requiring a rebase.

The two post-audit additions (items 1.9 and 1.10) were identified by running
`go test -bench . -benchmem -memprofile` on the filtering package after the
initial 20-item audit closed — demonstrating the value of profiler-driven
follow-up rather than treating the audit as a one-time exercise.

---

*This specification is maintained alongside the private AdGuardHome Edge
codebase. The public forks are available at
[github.com/Ozy-666/dnsproxy](https://github.com/Ozy-666/dnsproxy) (transport)
and [github.com/Ozy-666/urlfilter](https://github.com/Ozy-666/urlfilter)
(filtering engine).*
