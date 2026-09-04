# AdGuardHome Edge - Public Specification

[![Production](https://img.shields.io/badge/Powered_By-dnsdoh.art-24292e?style=flat-square&logo=ghost)](https://dnsdoh.art)
[![Stack](https://img.shields.io/badge/Stack-Nginx%20%E2%86%92%20AGH%20%E2%86%92%20Unbound%20%E2%86%92%20dnscrypt--proxy-2b6cb0?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Protocols](https://img.shields.io/badge/Protocols-DoH3%20%7C%20DoH%20%7C%20DoQ%20%7C%20DoT%20%7C%20DNS-2b6cb0?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Binary Size](https://img.shields.io/badge/Binary%20Size-24.6%20MB%20(--10%20MB)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

---

**Repository status:** This document is the public architectural specification and progress tracker for the AdGuardHome Edge project. The main codebase and production configuration remain private. This repository exists to document the design philosophy, engineering decisions, and measurable outcomes of the project in a form that can be shared openly.

---

## 📊 Codebase Modifications & Performance Metrics

Below is a detailed breakdown of the exact subsystem prunings and system-level performance enhancements implemented across our custom forks to maintain near-zero latency and high uptime.

### 🔐 Cryptographic Substrate (BoringSSL)
*The TLS termination and DNSSEC-validation hot paths run on a statically-linked BoringSSL substrate — formally-verified field arithmetic (fiat-crypto), zero provider-dispatch overhead:*

[![nginx BoringSSL](https://img.shields.io/badge/nginx-BoringSSL%20TLS%201.3-2f855a?style=flat-square&logo=nginx&logoColor=white)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Unbound BoringSSL](https://img.shields.io/badge/Unbound-BoringSSL%20(ECDSA%20--26%25%20latency)-2f855a?style=flat-square&logo=letsencrypt&logoColor=white)](https://github.com/Ozy-666/AdGuardHome-edge-spec#66-cryptographic-substrate--unbound-on-boringssl)

### 🗑️ Subsystem Stripping (Bloat Reduction)
*Non-essential features of standard AdGuardHome, stripped to cut resource use and attack surface — ~13k LOC removed in total (largest items shown):*

[![Blocked Services](https://img.shields.io/badge/Blocked%20Services-Removed%20(--5200%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![DHCP Subsystem](https://img.shields.io/badge/DHCP%20Subsystem-Stripped%20(--2400%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![SafeBrowsing & Parental](https://img.shields.io/badge/SafeBrowsing%20%26%20Parental-Removed%20(--2200%20LOC)-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![EDNS Subnet](https://img.shields.io/badge/EDNS%20Client%20Subnet-Stripped%20%2F%20Anti--Leak-4a5568?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

### ⚡ Engineering Patches & Optimizations
*Highlights across the transport, proxy, filter and resolver layers — full benchmarks in the sections below:*

[![urlfilter Engine](https://img.shields.io/badge/urlfilter%20Fork-O(1)%20AST%20Regexp-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/urlfilter)
[![Hot Paths](https://img.shields.io/badge/UDP%20%2F%20TCP%20Hot%20Paths-0%20allocs%2Fop-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/dnsproxy)
[![Transport Hardening](https://img.shields.io/badge/Transport-Framing%20%2F%20Goroutines%20%2F%20Reload-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/dnsproxy)
[![Client Lock](https://img.shields.io/badge/Per--Query%20Client%20Lock-Lock--Free%20(--79%25)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Real-Time WAF Feed](https://img.shields.io/badge/Real--Time%20WAF%20Feed-qfeed%20(0%20allocs)-2b6cb0?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

---
## Production Deployment

The architecture specified in this repository directly powers the backend
infrastructure of **[dnsdoh.art](https://dnsdoh.art)** — a zero-telemetry,
no-logs public DNS resolver.

By running this optimised AdGuardHome Edge build tightly integrated with
a custom [dnsproxy](https://github.com/Ozy-666/dnsproxy) transport layer and a
custom [urlfilter](https://github.com/Ozy-666/urlfilter) filtering engine, the
production deployment delivers:

- **Strict Privacy:** Zero data retention, zero upstream client leakage
  (EDNS-CS stripped), and no cloud dependencies.
- **High Throughput:** Lock-free processing paths designed to withstand
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

**Security backports & dependency audit (2026-06-10)**

- **Response-question validation** backported from upstream master (post-2.1.16,
  credited to Kun-Ta Chu): responses whose question name/type/class don't match
  the original query are rejected on the central response path, manual DNS
  exchanges, and the DNSCrypt/DoH server probes; `_dnsExchange` also verifies
  the response bit and ID. The channels are already authenticated (DNSCrypt
  signatures / DoH TLS), so this guards against a misbehaving upstream resolver
  — cheap defense-in-depth on the exact query path. The ODoH hunks of the
  upstream patch were dropped (ODoH is removed in this fork).
- **`golang.org/x/net` v0.54.0 → v0.55.0** — `govulncheck` flagged
  [GO-2026-5026](https://pkg.go.dev/vuln/GO-2026-5026) (idna Punycode
  acceptance) as *reachable* via the DoH transport
  (`xtransport.go` → `http.Transport` → `idna.ToASCII`); low practical
  exploitability (hostnames come from the static resolver config), fixed by the
  bump. After it: **0 reachable vulnerabilities**. The re-vendor also dropped
  the orphaned `go-hpke-compact` dependency left behind by the ODoH strip.
- Verified **not** applicable, not backported: upstream's cloaking-rule cycle
  fix (no cloaking rules) and the TCP-fallback fix for truncated *forwarded*
  queries (no forwarding rules).

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

### 6.7 Layer Recheck — Libtool-Wrapper Trap & the June 2026 OpenSSL Advisory (2026-06-10)

A scheduled recheck of the unbound + BoringSSL layer. The healthy parts first:
unbound 1.25.1 is the current upstream release; the daemon runs BoringSSL from
`/opt/boring` under an enforcing AppArmor profile (the §6.3 rule intact), with
the jemalloc preload active; DNSSEC validation re-verified live (`ad` on clean
chains, `SERVFAIL` on dnssec-failed.org); RFC5011 keeps `root.key` fresh
in-daemon. BoringSSL's pin was 81 commits behind master with nothing
security-relevant pending — notably, upstream explicitly documents BoringSSL as
**not affected** by the OpenSSL security advisory of June 9th, 2026
(**CVE-2026-45447**, High: heap use-after-free in `PKCS7_verify()`), which both
BoringSSL consumers here (unbound, nginx) therefore dodge entirely.

The trap the recheck caught: **`/usr/sbin/unbound-anchor` and
`/usr/sbin/unbound-host` had been libtool wrapper *scripts*, not binaries,
since early May.** In a libtool build the top-directory "binaries" of tools
that link `libunbound.la` are generated wrapper scripts (the real ELF lives in
`./.libs/` and is only relinked by `make install`); an earlier manual install
had copied the wrappers, which fail with `'/usr/sbin/.libs/unbound-anchor'
does not exist`. The breakage was invisible because `ExecStartPre=-` tolerates
the failure and in-daemon RFC5011 keeps the trust anchor fresh — but the
recovery tool for a missed KSK rollover was dead, and every backup since May 5
was a copy of the same broken wrapper. Both tools were restored from the
March 26 distro backups (real ELF, system OpenSSL — fine for a bootstrap and a
diagnostic tool), and both update scripts now refuse to copy the wrappers,
with the trap documented inline.

Residual note: the restored `unbound-anchor` links the *system* OpenSSL
(3.0.16+quic1, a manually-installed package with **no repository update
candidate**), which is version-affected by CVE-2026-45447. Practical exposure
is minimal — it runs only at unbound start, fetches from data.iana.org over
certificate-pinned HTTPS, and the PKCS#7 parse sits behind that — but the
+quic1 OpenSSL package should be rebased to 3.0.21 whenever it is next
touched.

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

### 7.15 Defense-in-Depth: DNS QName Sanity Guard & Stats Pollution Watchdog (2026-06-06)

A WebUI report turned into a forensic case study. The dashboard's *top queried / blocked domains*
briefly listed entries like `www.google.com\ a`, `gc.kis.v2.scr.kaspersky-labs.com\ https`,
`49.98.30.81.in-addr.arpa\ ptr` — a domain with its **own lowercased query type** glued on after a
`\ ` separator. It appeared under heavy load and cleared on its own.

**Forensics (the interesting part).** Every layer was cross-checked rather than guessed at:

- *Real, not a render artifact.* A hexdump of `stats.db` showed the literal bytes `…com` `5c`(`\`)
  `20`(space) `61`(`a`). The `\ ` is exactly how miekg's `sprintName` escapes a 0x20 space byte. The
  dirty keys had been flushed to bbolt and now survive only in **freed pages** — every *live* hourly
  bucket decoded 100% clean (verified with a bbolt + gob walker; unit id = hours-since-epoch). The
  query log showed the same `\ a` before its in-RAM file rotated the incident entries out.
- *A Go-level concatenation, not a buffer/wire bug.* The wire qtype is **binary** (`0x0001` for `A`),
  never the ASCII text `a`/`https`/`srv`. So the type text can only originate from a Go
  `name + " " + dns.Type(qt).String()` concatenation reaching `dnsContext.normalizedName`
  (`NormalizeDomain(q.Name)`), then lowercased. This **excluded** any packet-buffer corruption — and,
  notably, exonerated an in-flight `recvmmsg` read-path experiment (§8.9). With no `unsafe` string
  conversions anywhere in AGH and a single `stats.Update` caller, `normalizedName` is a safe immutable
  string and the *only* stats-domain source; so `q.Name` itself was `name<space>TYPE` for a small
  **race subset** (~199 of ~14 M queries), only during the load window.
- *Functional, not cosmetic.* A crafted query with an embedded-space qname returns **SERVFAIL** —
  malformed-name queries fail to resolve — a plausible contributor to the resolution slowness observed
  during the window.
- *Not reproducible* on the shipped build afterward (heavy DoT, DoH-over-socket, UDP, and a full
  AF_XDP-blaster + DoT + DoH storm all stayed clean). A genuine transient.

**The defense (shipped).** A malformed name could not be root-caused by reading alone, so rather than
guess, two cheap **defense-in-depth guards** convert a silent heisenbug into a catchable one. A
normalized DNS name can never legitimately contain a space, so the condition is unambiguous:

- *AdGuard Home* — `dnsforward.sanitizeNormalizedName` (`e89d326d`): on every request, if the
  normalized question name contains a space, **trim** the qtype suffix (and a preceding escape
  backslash) so stats and the query log keep correct keys, and **log** the raw name, qtype and
  transport (exponentially sampled). A single byte-scan on the hot path; it cannot affect valid
  traffic.
- *dnsproxy* — `Proxy.logMalformedQName` (`71dc552`): at the single post-`Unpack` chokepoint
  (`handleDNSRequest`, all transports), the same space check logs WARN with the raw name, qtype,
  `DNSContext.Proto` **and client address**. Behavior is unchanged; it exists purely so a real
  recurrence pins the exact offending **transport** and **upstream handler**.

Both were verified to fire and sanitize against a crafted embedded-space query (`proto=udp`). The
lesson is the method as much as the fix: **prove the layer with a hexdump before theorizing**, use an
*impossible-by-construction* invariant (no spaces in a normalized name) as the guard condition, and
when a race can't be reproduced, ship a sampled watchdog that captures the producing context instead
of a speculative rewrite.

---

### 7.16 DDR: Advertising Proxy-Terminated DoH (`ddr_external_doh`) (2026-07-02)

**The gap.** Discovery of Designated Resolvers (RFC 9462) lets a client that only knows a
resolver's bare IP upgrade to encryption by sending one SVCB query for `_dns.resolver.arpa`
to that resolver. Upstream's handler (`makeDDRResponse`, `internal/dnsforward/process.go`)
builds the answer purely from the listeners AGH terminates itself:

- **DoH** — iterates `TLSConf.HTTPSListenAddrs`. In this deployment DoH TLS is terminated by
  the fronting nginx on 443 and reaches AGH as plain HTTP over the Unix socket (`port_https: 0`),
  so the list is empty and no DoH designation is ever emitted.
- **DoT** — gated on the certificate containing an IP SAN (upstream #4927); the Let's Encrypt
  certificate is name-only, so DoT is skipped.
- **DoQ** — emitted unconditionally from `QUICListenAddr`.

Net effect: DDR advertised **DoQ only**, and discovery-capable clients (Windows 11, Apple
platforms) never learned that the standard DoH endpoint exists.

**The fix.** A new opt-in bool `dns.ddr_external_doh` (default `false`). When set,
`makeDDRResponse` appends one more designation targeting `tls.server_name`:

```
_dns.resolver.arpa. 300 IN SVCB 1 dnsdoh.art. alpn="h2,h3" port=443 dohpath="/dns-query{?dns}"
```

Standard RFC 8484 port and path template; ALPN matches what the fronting proxy actually serves
(HTTP/2 and HTTP/3). The DoQ record is unchanged and both are served at priority 1.

**Honest scope.** Strict *verified* DDR requires the designated server's certificate to cover the
original resolver IP the client started from; without an IP SAN the designations (DoQ included)
serve opportunistic upgrade only. DoT stays un-advertised for the same reason upstream gates it.

**Follow-up (same day): verified discovery via a dedicated designation target.** Let's Encrypt
issues IP-SAN certificates (GA 2026-01) only under the *shortlived* profile — 160 h validity,
renewal every ~4 days. Putting the whole production stack on that treadmill was rejected: a failed
renewal would take down web, DoH, DoT and DoQ within days. Instead the blast radius is isolated:

- A new optional string `dns.ddr_external_doh_target` overrides the DoH designation target
  (default `tls.server_name`).
- Deployment: a dedicated nginx vhost (`ddr.dnsdoh.art`) exposes **only** `/dns-query` (same
  upstream, same rate limits) and serves a shortlived certificate carrying
  `{DNS:ddr.dnsdoh.art, IP:194.180.189.33}` (certbot `--required-profile shortlived`,
  `--reuse-key`, webroot http-01 — the IP identifier cannot be validated via dns-01).
- The DDR answer becomes `1 ddr.dnsdoh.art. alpn="h2,h3" port=443 dohpath="/dns-query{?dns}"`
  plus the stock DoQ record. Strict clients (Windows 11, Apple) can now complete verified
  discovery against the IP SAN; if the shortlived lineage ever fails to renew, only discovery
  verification degrades — the main certificate and every existing endpoint are untouched.

---

### 7.17 DDR: `ipv4hint` in Every Designation (2026-07-10)

**The gap.** A DDR answer names the designated resolver (`dnsdoh.art`) but, without address
hints, a bootstrapping client must resolve that name's A record before it can open the
encrypted session - one extra round trip through the very plain-DNS path DDR is trying to
leave, at the one moment latency is most visible. Cloudflare's production DDR records ship
`ipv4hint`/`ipv6hint` on every designation for exactly this reason.

**The fix.** A new optional `dns.ddr_ipv4_hint` (`netip.Addr`). When set to a valid IPv4
address, `makeDDRResponse` appends `ipv4hint=<addr>` to **every** designation - the
external-DoH record, DoT and DoQ:

```
_dns.resolver.arpa. 300 IN SVCB 1 dnsdoh.art. alpn="h2,h3" port=443 ipv4hint=194.180.189.33 key7="/dns-query{?dns}"
_dns.resolver.arpa. 300 IN SVCB 1 dnsdoh.art. alpn="dot" port=853 ipv4hint=194.180.189.33
_dns.resolver.arpa. 300 IN SVCB 1 dnsdoh.art. alpn="doq" port=853 ipv4hint=194.180.189.33
```

SVCB parameters must be wire-encoded in ascending key order (RFC 9460 §2.2), so the hint
(key 4) is inserted between `port` (3) and `dohpath` (7) rather than appended. Hints are
non-authoritative (RFC 9460 §7.3): clients still resolve the target name and prefer those
answers, so a future re-addressing propagates normally. Unset keeps prior behavior exactly;
the deployment sets it to the resolver's own anycast-facing address (IPv4-only service, so
no `ipv6hint` counterpart).

---

### 7.18 Size-Gated DNS Response Rate Limiting with TC=1 Slip (2026-07-12)

An open resolver that answers a large record over UDP is a **reflection/amplification
weapon**: an attacker spoofs a victim's source address, sends a tiny query for a fat
cached record, and the resolver mails the amplified answer to the victim. The edge
deployment was observed doing exactly this during intermittent, hours-long floods that
recur periodically — a spoofed swarm querying `cloudflare.com`/`TXT` (a 2,257-byte
SPF + domain-verification record) at **~130–250 q/s** from thousands of carpet-bombed
source addresses. A **71-byte query yields a 2,257-byte response: a 31.8× amplification
factor**, fragmented on the wire.

The two source-address rate limiters already present at L4 (per-IP and per-/24) never
fired: the attack is deliberately sub-threshold per source (1–5 queries per spoofed
address across thousands of them). Amplification is a property of the **response**, not
the address — so the defense has to live where the response is built, and key on what the
attacker *cannot* cheaply vary: the query name.

**The engine (`internal/rrl`).** A new pipeline stage runs after the answer is resolved
(cache or upstream) and before it is written, so an over-budget response can be truncated
in place. Every outgoing **UDP** response passes three gates:

| Gate | Key | Role |
|---|---|---|
| **1 — Size pass** | `len(response) ≤ 512 B` | *FP-safety.* Below the gate the response can never be a slip candidate, at any rate. This exempts essentially all normal browsing (A/AAAA/HTTPS answers) — measured at **99.7%** of live traffic — making it structurally untouchable. |
| **2 — Subnet bucket** | `(src /24, qname, qtype)` | *Fairness.* Stops one noisy network from draining the global allowance for a hot record. |
| **3 — Global bucket** | `(qname, qtype)` | *Amplification killer.* A carpet-bombing flood spread over thousands of /24s converges on **one** global bucket. This is the gate that actually collapses the attack — spoofing more sources is free, finding another 32× record is not. |

Both buckets are token buckets (default **5 tokens/s, burst 20**) and are **charged on
every call** — the engine never short-circuits on the first exhausted bucket, so neither
gate can shadow the other. A slip fires if **either** is exhausted.

**The slip (TC=1).** An over-budget response is not dropped — it is replaced with a
**32-byte** header carrying the Truncated (`TC=1`) bit and an empty answer section
(RFC 1035 §4.1.1). The consequences split cleanly by whether the source is real:

- **A legitimate client** — even one NAT'd behind the same /24 as the attack — receives
  the `TC=1` signal and immediately retries over **TCP/53** (RFC 7766), where it gets the
  full answer. Zero service disruption.
- **A spoofed source** cannot complete a TCP three-way handshake, so the attack simply
  cannot follow the truncation to TCP. The 32-byte response is *smaller than the 71-byte
  query*: amplification collapses from 31.8× to **~0.45×** (32 / 71), i.e. the resolver
  stops being an amplifier entirely and emits fewer bytes than it receives.

Only plain UDP is gated. **DoT, DoQ, DoH, and TCP are connection-verified and inherently
immune to source spoofing**, so they bypass the engine untouched.

**Co-design and validation (shadow → parity → flip).** RRL is the enforcement half of a
split defense: the L7 WAF (§7.10, the `qfeed` co-design) runs the *same* size-gated
dual-bucket logic in **shadow** over its query feed — deciding, counting, changing
nothing — while this stage enforces inside the resolver. Because both sides share the
frozen qname-hash rule (the XXH3-64 normalization of §7.10) and an identical engine, their
decisions agree by construction. Before enforcement was enabled, the two were compared on
the *same* live flood over a 30-second window and read identically — **98.0% slip, 5.0
answered/second** on both — confirming the resolver would truncate exactly the set the WAF
had already validated as safe. Enforcement was then flipped on and verified on the wire.

**Observed result (live flood, 2026-07-12).**

| Metric | Before (no RRL) | After (enforce) | Effect |
|---|---|---|---|
| Response to a spoofed query | 2,257 B (2 fragments) | **32 B** (`TC=1`) | — |
| Per-query amplification | **31.8×** (2,257 / 71) | **~0.45×** (32 / 71) | net de-amplifier |
| Large outbound response fragments | ~132 /s | **~4.5 /s** (= the 5 q/s legit budget) | **96.6%** fewer |
| Amplified egress for the target record | ~2.4 Mbps | **~0.08 Mbps** | ~96% suppressed |
| Flood-name query under enforce (20 samples) | full 2.2 KB answer | **20/20 `TC=1`** | truncated |
| Non-flood large record (control, `DNSKEY`) | served | **served, 0 slipped** | FP-safe |
| Legitimate resolution | served (shared-bucket risk) | **untouched** | 100% FP-safe |
| False-positive ledger | — | **CLEAN over 67,905 shadow decisions** | validated |
| Per-query CPU cost | — | **0.061 ms** median (n=125k) | sub-100 µs |
| Cumulative responses neutralized | — | **2.8 M+** and climbing | — |

The residual ~4.5 outbound fragments per second is not leakage — it is exactly the
5 q/s the token buckets legitimately allow through, so a real client that happens to ask
for the same record during a flood is still served.

**Configuration.** Environment-gated, default off; enabled and flipped from shadow to
enforce without a rebuild. Defaults: rate 5 tokens/s, burst 20, size gate 512 B, UDP-only.
**Standards:** the mechanism is the classic Vixie/Schryver RRL-with-SLIP technique
expressed in the resolver; it honors EDNS buffer limits (RFC 6891) and relies on the
mandatory truncation-retry-over-TCP contract (RFC 1035 §4.1.1, RFC 7766). It is *not* a
blind L4 rate limiter: normal small queries and every connection-oriented transport are
exempt by construction, and only weaponized fat records over spoofable UDP are ever
touched.

---

### 7.19 DDR: Ranked Designations Instead of a Three-Way Tie (2026-07-30)

**The gap.** Upstream emits every DDR designation with `SvcPriority` 1, mirroring the
examples in the draft that became RFC 9462, and carries an in-tree
`TODO: Consider setting the priority values based on the protocol`. Equal priorities are a
tie, and RFC 9460 §2.4.1 has clients break ties **at random**. A client supporting all
three transports could therefore land on any of them, including the slowest one we offer,
on every fresh discovery.

**The fix.** Three named constants rank the designations, and the DoQ record is emitted
before DoT so a `dig`/`kdig` capture reads in rank order (answer order carries no meaning
on the wire):

```
_dns.resolver.arpa. 300 IN SVCB 1 dnsdoh.art. alpn="h3,h2" port=443 ipv4hint=... key7="/dns-query{?dns}"
_dns.resolver.arpa. 300 IN SVCB 2 dnsdoh.art. alpn="doq"   port=853 ipv4hint=...
_dns.resolver.arpa. 300 IN SVCB 3 dnsdoh.art. alpn="dot"   port=853 ipv4hint=...
```

**The ranking is by reachability, not by speed, and that is deliberate.** Our own
transport benchmarks make DoQ the fastest to a first answer - a full round trip ahead of
DoH3 on a cold connection, with a bounded tail under packet loss - so a speed ranking would
put it first. It lives on port 853, which restrictive networks block outright, and a
blocked port costs the client a timeout before it falls back to the next record. DDR exists
to upgrade clients on networks nobody controls, so *reachable* beats *fast*. Port 443
carries `h3` and `h2` in a single designation, so a client that cannot complete an HTTP/3
handshake falls back to TCP without leaving the record at all. Ranking DoH first costs a
DoQ-capable client nothing where 443 works, because a client only considers designations
whose ALPN it supports.

The `alpn` list is a **set**, not a preference order: RFC 9460 §7.1.1 has the client choose
from the intersection by its own preference. Writing `h3,h2` rather than `h2,h3` is
presentation only and steers no client.

Verified live over UDP/53, TCP/53 and all three encrypted transports; the designation is
identical on every path.

---

### 7.20 DoT ALPN Confirms What DDR Advertises (2026-07-30)

**The gap.** The DDR designation above advertises `alpn="dot"` for the port-853 TCP
endpoint, but the DoT listener negotiated **no ALPN at all**. Upstream `dnsproxy` hands the
shared `TLSConfig` to `tls.NewListener` unchanged in `initTLSListeners`, while the HTTPS
and QUIC listeners clone that config and set their own `NextProtos`. The resolver was
promising a token it never confirmed:

```
$ openssl s_client -connect <resolver>:853 -alpn dot </dev/null | grep ALPN
No ALPN negotiated
```

**The fix** lives in the dnsproxy fork (`14c4d5b`): clone the config and offer the RFC 7858
§6 token. After deployment the same command returns `ALPN protocol: dot`, confirmed against
three independent client stacks.

ALPN remains **optional** for DoT - RFC 7858 §3.1 forbids rejecting a connection that does
not use it, and a client sending no ALPN extension still connects, which was verified
explicitly rather than assumed. The one behavior change is RFC 7301 conformance: a client
that offers an ALPN list with no token in common is now refused instead of silently
accepted. No DoT client does this; it is recorded here because it is the only regression
surface.

---

### 7.21 Upstream v0.107.78: Reviewed, Not Merged (2026-07-31)

This fork tracks upstream by **review**, not by merge. It strips DHCP, ECS, large parts of
the dashboard and the CI pipeline, so a `git merge` of an upstream release spends most of
its effort re-adding code that was deliberately removed: a trial merge of v0.107.78
produced 67 conflicted files, 34 of them "deleted by us, modified by them". The version
string is a **review marker** — it records which upstream release was read, not which
commit the tree descends from.

**The structural finding: nothing in v0.107.78 needed a change in AdGuard Home itself.**
Every item under the release's *Security* heading resolves to a `dnsproxy` fix that reaches
AGH through the `go.mod` bump. On any future release, read `dnsproxy` first.

**Ported into the dnsproxy fork** (`edge-udp-pool`):

| upstream | fork | change |
|---|---|---|
| `a221504` | `35baa86` | The AA bit is cleared on upstream responses (AGH #7955). A forwarding resolver is not authoritative for what it relays; clean cherry-pick. |
| `0d3732b` | `c9d9863` | A UDP packet whose header parses but whose body does not is answered **FORMERR** instead of dropped, and a request whose question count is not 1 returns FORMERR rather than SERVFAIL (RFC 1035 §4.1.1). This is the GHSA-p5f5-3p5g-rfjw (JIGGLE) mitigation. Hand-merged against the fork's `respondUDP`, which carries the pooled pack buffer and a different signature. |
| `9f66818` | `df16938` | DoQ now refuses unidirectional QUIC streams outright (`MaxIncomingUniStreams: -1`); DoH3 moves to its own `newServerDoH3Config`, keeping the 64 it needs for the HTTP/3 control stream and the QPACK encoder/decoder streams (≥3 per RFC 9114). |

Two details worth recording because they are easy to get wrong. The upstream commit *named*
for JIGGLE, `f35ca3e`, contains only the regression test and a Go bump; the mitigation is
`0d3732b`. And `5235b4c` "set alpn set dot" touches `upstream/dot.go`, the **outgoing** DoT
client — it does not overlap §7.20, which offers the ALPN on the DoT **listener**.

On the QUIC advisory (GHSA-qr92-rwvw-mhgh, GHSA-cccx-2r6r-m9r4) this fork was already ahead:
`f9ab1de` had bounded `MaxIncomingUniStreams` at 64 against upstream's vulnerable
`math.MaxUint16`, so the memory-exhaustion vector was capped before the advisory. Upstream's
own fix drops the configurable *bidirectional* limit; the knob is kept here (§10).

**Already present, so not re-taken:** GHSA-4qjf-2hgm-92q6 DoH upstream validation
(`2c1f097` + `25d8f46`), AGH #8276 `ech=` parsing (`161b0817`), the h2c upgrade path removal
(`d9e8847d`), the rulelist size cap (`3aac5695`), CVE-2026-41448 (`3336007e`), and Go 1.26.5
as the build toolchain.

**Deliberately skipped.** The stricter DNSCrypt *upstream* validation and the surrounding
DNSCrypt refactors do not apply: `upstream_dns` is `127.0.0.1:5353`, plain DNS to a local
dnscrypt-proxy process (§5), so AdGuard Home never speaks the DNSCrypt protocol itself. The
new `max_http_size` option is skipped because the cap here is hardcoded, which is stricter.
Everything else in the release — DHCP, updater logging, translations, the dashboard, the
filter-update interval range — sits in code this fork either removed or does not reach.

**On answering malformed packets.** The JIGGLE mitigation converts a silent drop into a
reply, which deserved scrutiny after the reflection incident in §7.18. Two findings. First,
the reply bypasses the RRL: rate limiting lives in AGH's request handler, while FORMERR is
emitted earlier, inside dnsproxy. Second, it cannot amplify. Measured against the deployed
build, a 33-byte malformed query drew a **12-byte** response — **0.36×**, a deamplifier,
because a question that fails to parse is not echoed back:

```
reply 12 B  (request was 33 B)   rcode=1 (FORMERR)  qr=1 aa=0
qd=0 an=0 ns=0 ar=0
```

The packet filter covers the RRL gap independently: UDP/53 is metered at 30 packets per
second per source address and 150 per `/24`, both with 30-minute bans, and packets shorter
than 28 bytes or carrying the QR bit or a nonzero opcode never reach the daemon.

**Verified after deployment:** `v0.107.78-edge`; no `aa` flag in answers; DoT ALPN `dot`;
DoQ answering (the `-1` unidirectional setting does not affect DoQ clients, which open none);
DDR still ranked 1/2/3 with `ipv4hint`; DNS cookies validating; no errors in the journal.

Two test failures in the AGH tree are **pre-existing and unrelated**, confirmed by re-running
the untouched baseline: `internal/configmigrate` expects `session_ttl: 720h` where golibs now
renders `30d`, and `internal/next/dnssvc` expects one listen address where the SO_REUSEPORT
sharding (§8) creates one per core.

### 7.22 Upstream v0.107.79: Reviewed, One Fix Adapted (2026-08-21)

Upstream released v0.107.79 on 2026-08-18 under a *Security* heading. Read as a review
rather than a merge (§7.21), it resolves into three parts, and **only one of them was code
this fork could take unchanged — and even that one could not be taken unchanged.**

**The two security items needed nothing from upstream's tree.** GHSA-w6v6-f44j-3rj2, "DoQ
unidirectional stream state exhaustion", is a `dnsproxy` advisory, patched upstream in
dnsproxy v0.83.1: a client opens the highest-numbered unidirectional stream, QUIC stream-ID
semantics implicitly open all 65,535 below it, and dnsproxy never reads or cancels them, so
receive-stream state is pinned until the connection closes. This fork answered it three
weeks earlier in `df16938` (§7.21) by refusing unidirectional streams outright on DoQ. The
second item is the Go toolchain: the deployed binary was built with Go **1.26.5**, and Go
1.26.6 (2026-08-13) carries security fixes to the `go` command and to `crypto/tls`,
`encoding/asn1`, `encoding/xml`, `html/template`, `net`, `net/http` and `net/url`. That one
was real, and it is answered by rebuilding — not by merging.

**Taken:**

| upstream | change |
|---|---|
| `internal/dnsforward/msg.go` | `Server.reply` echoes the request's EDNS(0) state — DO bit and advertised UDP buffer — onto every response AGH generates itself (blocked, NXDOMAIN, REFUSED, SERVFAIL, NODATA, FORMERR), and adds no OPT at all when the request carried none. `NewMsgNOTIMPLEMENTED` keeps its constant 1452 and now sets EDNS(0) only if the request used it. AGH #8183. |
| `internal/dnsforward/upstreams.go` | `newBootstrap` filters comment and empty lines, which `newPrivateConfig` already did here. No-op for this deployment's single `127.0.0.1:5353` bootstrap. |
| `internal/filtering/rewrites.go` | A legacy rewrite answer that is neither an IP address nor a valid domain name is rejected by name instead of being stored as a CNAME target. |

**Where the copy-paste would have hurt.** Upstream echoes `opt.UDPSize()` raw. This fork
clamps: `maxAdvertisedUDPSize` in the dnsproxy fork (`proxy/dnscontext.go`) caps both the
honoured buffer and the truncation point at **1232**, per DNS Flag Day 2020, because
amplification floods advertise huge buffers deliberately — 10000 observed live (§7.18). The
upstream line would have advertised a size this stack never honours, on exactly the
responses that are cheapest for a reflector to trigger. `Server.reply` therefore echoes
`min(opt.UDPSize(), maxAdvertisedUDPSize)`, with the constant mirrored in `msg.go` and
pointed back at its source. Against upstream's unclamped version the fork's own test fails
on the number itself:

```
--- FAIL: TestServer_reply_edns/oversized_buffer_clamped
    expected: 0x4d0     (1232, honoured)
    actual  : 0x2710    (10000, the client's raw ask)
```

**Two local patches had to be checked before taking it.** `setCookie`
(`internal/dnscookie`) attaches the DNS Cookie to an existing OPT and only creates one when
the response has none; `scrub` in the dnsproxy fork likewise only adds an OPT when the
response has none. Both now find the OPT that `reply` created, so the response still carries
exactly **one** OPT record. What changed is its content: the cookie path used to create the
OPT itself, at `dns.MinMsgSize` with the DO bit dropped.

Measured against the live resolver, same query before and after deployment:

```
# v0.107.78-edge — dig @127.0.0.1 +dnssec doubleclick.net A
; EDNS: version: 0, flags:; udp: 512
; COOKIE: 29295a0a91e11e80... (good)

# v0.107.79-edge — same query
; EDNS: version: 0, flags: do; udp: 1232
; COOKIE: 4cd1e9685440e85c... (good)

# v0.107.79-edge — dig +bufsize=10000  (clamp holds on the wire)
; EDNS: version: 0, flags: do; udp: 1232

# v0.107.79-edge — dig +noedns        (no OPT in the reply, as RFC 6891 requires)
(no OPT PSEUDOSECTION)
```

**Deliberately skipped.** The **DDR / `TLSConfigProvider` refactor** moves `hasIPAddrs`
behind an interface and reorders `makeDDRResponse` so DoT is appended before DoQ — but it
also restores the three-way priority tie that §7.19 removed, so taking it would be a
regression; it is behaviourally inert here in any case, since this host's certificate
carries no IP addresses. The **DNS64 CNAME/DNAME chain fix** (dnsproxy #438) is unreachable:
`use_dns64` is `false`. **dnsproxy v0.84.0/v0.84.1** is not taken as a dependency bump: its
only functional content is that DNS64 fix, a `dnsproxytest` helper package, a Go bump, and
`AGDNS-4357`, which **unexports every field of `proxy.Proxy`** — a breaking API change
straight through the surface this fork patches (the pooled UDP write path, the QUIC server
config, the RRL and cookie hooks) for no security benefit. The `strict_sni_check`
deprecation is already `false` here. The rest — the `client_v2` dashboard and the `edge`
channel's new versioning scheme, the DHCPv6/`dhcpsvc` database rework, the install API's
`language` property, static-lease hostname removal, translations and blocked-services data —
sits in code this fork strips or does not reach.

**On the version string as a review marker.** §7.21 records that the version is a review
marker, not an ancestry claim. This release exposed a hole in how that marker was read:
`v0.107.78` existed in the build clone only because `git fetch upstream --tags` had pulled
**upstream's** tag, and the tooling that reports "which release was reviewed" reads the
newest local `v*.*.*` tag. Fetching upstream would therefore have declared v0.107.79
reviewed before anyone read it. Fixed by tagging the fork's own commits: `v0.107.79` (and,
retroactively, `v0.107.78`) are now annotated tags in `Ozy-666/AdGuardHome-Edge` on the
reviewed commit, and upstream's tags are kept locally under `upstream-v*`.

**Verified after deployment:** `v0.107.79-edge`, built with Go 1.26.6, `GOAMD64=v3`; the
three EDNS behaviours above; plain DNS, DoH via nginx, DoT and DoQ all answering; DNSSEC
still validating (`AD` on `cloudflare.com`, SERVFAIL on `dnssec-failed.org`); no errors in
the journal. Go 1.26.7 (2026-08-19) exists and is stable but carries `net/http` bug fixes
only — it is not a security release, and the move that mattered was 1.26.5 → 1.26.6.

The same two test failures remain pre-existing and unrelated, re-confirmed against an
untouched worktree of the pre-change commit: `internal/configmigrate` (`session_ttl: 720h`
vs golibs' `30d`) and `internal/next/dnssvc` (one listen address vs one per core under
SO_REUSEPORT sharding).

---

### 7.23 Truncated UDP Responses Carry No Answer Bytes (2026-09-01)

§7.22 clamped the advertised EDNS(0) buffer to 1232 B, which decides **when** the `TC`
bit is set. It said nothing about **how much** is reflected once it is set — and the
answer, until now, was "as much as fits".

`(*dns.Msg).Truncate` in `miekg/dns` walks the record sections and appends as many RRs
as the budget allows before setting `TC`; only the overflow is discarded. RFC 2181 §9
tells a client that receives `TC=1` to ignore the response and retry over a transport
that permits larger replies. So every byte of answer in a truncated datagram is a byte
no conforming client may use — but a byte a spoofed source address is still mailed.
That is the whole amplification primitive, surviving inside a defence built to remove
it. Google, Cloudflare and Quad9 all return a bare header here.

**The change** (`c9c8de7`, `Ozy-666/dnsproxy` `edge-udp-pool`). After `Truncate`, when
the transport is plain UDP and `TC` ended up set, `proxy.scrub` drops `Answer` and `Ns`
and reduces `Extra` to the OPT record:

```go
dctx.Res.Truncate(int(dnsSize(dctx.Proto == ProtoUDP, dctx.Req)))
if dctx.Proto == ProtoUDP && dctx.Res.Truncated {
	emptyTruncated(dctx.Res)
}
```

What survives is header + question + OPT. The **OPT is deliberately kept**: it carries
the DNS Cookie a real client needs in order to prove return-routability on its retry —
the same reasoning as the RRL slip's `cookieOnlyExtra` (§7.18) — and a response is
required to echo the request's EDNS(0) state. A request with no OPT gets no additional
section at all, per RFC 6891.

TCP, DoT, DoQ and DoH are structurally untouched: `dnsSize` returns `dns.MaxMsgSize`
for them, so `TC` is never set and `emptyTruncated` never runs. This composes with, and
does not replace, the 1232 clamp and the size-gated RRL slip.

**One accepted behaviour change.** `Truncate` sets `TC` when *any* section overflows,
including the additional section. A response whose answer fits but whose additional
records do not previously reached the client complete-with-`TC`; it is now a bare header
and the client must retry over TCP. This matches the three majors, and RFC 2181 §9 says
such a client should have retried regardless.

**Measured — TXT amplification census, run from the ad-astra vantage (2026-09-01
17:18–17:24 Riga), `+dnssec +bufsize=4096 +ignore +notcp`.** Measuring our own edge from
the box that serves it gives wrong answers, so the census runs from the second VPS.
`amp = (udp + 28) / query`, where `query` is the wire size a *cookie-less* attack tool
would send.

| domain | full | udp @ our edge | amp | udp @ 8.8.8.8 / 1.1.1.1 / 9.9.9.9 |
|---|---|---|---|---|
| `sony.com` | 9,047 | **65** | 1.4× | 37 |
| `cisco.com` | 6,842 | **66** | 1.4× | 38 |
| `adobe.com` | 6,032 | **66** | 1.4× | 38 |
| `intel.com` | 5,943 | **66** | 1.4× | 38 |
| `amazon.com` | 5,237 | **67** | 1.4× | 39 |
| `microsoft.com` | 4,918 | **70** | 1.4× | 42 |
| `hp.com` | 4,326 | **63** | 1.4× | 35 |
| `samsung.com` | 4,235 | **68** | 1.4× | 40 |
| `oracle.com` | 3,636 | **67** | 1.4× | 39 |
| `ibm.com` | 3,391 | **64** | 1.4× | 36 |
| `salesforce.com` | 3,384 | **71** | 1.4× | 43 |
| `ebay.com` | 2,740 | **65** | 1.4× | 37 |
| `cloudflare.com` | 2,619 | **71** | 1.4× | 43 |
| `apple.com` | 2,177 | **66** | 1.4× | 38 |
| `paypal.com` | 1,265 | **67** | 1.4× | 1,237 / 1,237 / 39 |
| `aol.com` | 1,256 | 1,256 | 20.1× | 1,228 / 1,228 / 1,228 |
| `google.com` | 1,131 | 1,131 | 17.3× | 1,103 / 1,103 / 1,103 |
| `yahoo.com` | 824 | 824 | 12.9× | 796 / 796 / 796 |

Fifteen of eighteen collapse from a packed datagram to a bare header. The residual
**1.4× against the majors' 1.0× is entirely the DNS Cookie** — verified directly rather
than inferred:

```
# aol.com TXT @194.180.189.33          # microsoft.com TXT @194.180.189.33
with cookie: rcvd=1256                 with cookie: rcvd=70
+nocookie:   rcvd=1228                 +nocookie:   rcvd=42
```

**Two findings this census forced, neither of them about the change itself.**

*The majors are not 1.0× across the board.* `aol.com`, `google.com` and `yahoo.com`
return 796–1,228 B over UDP from Google, Cloudflare **and** Quad9 — because those
answers fit under 1232 and are therefore never truncated by anyone. No resolver
truncates what fits, and a full answer that fits in one datagram is not a defect. Any
comparison table that reads "them 1.0×, us 17×" is comparing truncation-eligible names
against names that were never eligible. Our rows for those three names are the majors'
rows plus the 28-byte cookie — `aol.com` 1,228 + 28 = 1,256, `yahoo.com` 796 + 28 = 824.

*`google.com` is the exception, and it is not a size behaviour.* A re-run at 17:38 put our
`google.com` row at 1,215 against the majors' 1,103 — **+112, not +28**. The cause is a
record count, not a byte count: our chain returns **17** TXT records where all four
`ns[1-4].google.com` return **16**. The extra one is
`arcules-domain-verification=…`, which the authoritative servers no longer publish. It is
not a long-lived stale entry in AGH — the RRset was 42 s old (TTL 258 of 300) when
measured.

**The mechanism: `google.com`'s own authoritative fleet does not agree with itself, and
which half you reach depends on where you are standing.** Four diagnoses were proposed
during the afternoon and evening. All four were wrong, and they are kept here because the
sequence is more instructive than the answer:

1. *"It enters at the DNSCrypt cache."* Rested on a single comparison — `dnscrypt-proxy`
   served 17 while plain `9.9.9.9` served 16 — which never established a cache at all,
   since `dnscrypt-proxy` reaches Quad9 over a DNSCrypt stamp, not over plain UDP to that
   address.
2. *"Something above `dnscrypt-proxy` re-supplies it on every refresh."* Better evidenced
   (the TTL was watched across zero at two layers, still returning 17), but it never
   identified what or why.
3. *"A withdrawn record only we still serve."* False: the record is still published, by
   half the fleet.
4. *"A transient divergence that has converged."* False: it came back. The reading that
   produced it — all four `ns[1-4].google.com` returning 16 — was taken **from ad-astra
   only** and treated as the authoritative state rather than as one vantage's view of it.

The actual state, 24 samples at 5 s intervals from this host, 2026-09-01 21:27–21:29:

```
                    16-record view   17-record view
ns1 216.239.32.10        24/24             —
ns3 216.239.36.10          —             24/24     <- stable split, not a flap
dnscrypt-proxy           19/24            5/24     <- lottery, per cache refill
unbound                  24/24             —
edge                     24/24             —
```

`ns1`/`ns2` serve 16 records; `ns3`/`ns4` serve 17, including
`arcules-domain-verification=…`. Both readings are rock stable — 24 out of 24 each — so
this is not a propagation flap on a moving zone. The same four addresses queried from
ad-astra at the same moment return **16 from all four**. One anycast address, two
versions of the RRset, decided by which instance the query lands on.

So nothing in our chain was ever stale, and nothing was being withdrawn. Each cache layer
holds whichever view it happened to fetch, which is why the layers disagree with each
other and flip independently: at 21:26:10 the edge and unbound held 17 while
`dnscrypt-proxy` already held 16; ninety seconds later all three read 16 while
`dnscrypt-proxy` flipped back to 17 for five samples. A byte count for a name in this
state measures which server the resolver asked, not how the resolver behaves — our census
row read 1,215 on the 17-record view and 1,131 on the 16-record view, and 1,131 is
Google's 1,103 plus our 28-byte cookie exactly.

**The method lesson, which is the durable part — and it is not rule 4.** The obvious
reading is "small sample, take more". That reading is wrong, and the numbers say so: the
final samplings were 24/24 from this host and 14/14 from the other, both perfectly stable,
both entirely correct about their own vantage. Nobody was under-sampling. **The
disagreement was between the vantages, not within either one**, and more samples from one
place would never have found it — they would have hardened the wrong conclusion, because
each vantage's view is stable, reproducible, and self-consistent. Stability was the trap,
not the cure.

That is a distinct failure from the usual `n=1` anecdote and deserves its own name.
Rule 4 says a handful of runs is not a finding; this says **a thousand runs from one
observation point is still one observation point**. Where a result depends on where you
are standing — anycast, CDN edges, GeoDNS, per-instance caches, split-horizon zones —
the axis to vary is *location*, and no amount of repetition along the other axis
substitutes for it. Three sessions produced four mechanisms in one evening; what settled
it was a second host, not more data and not better argument.

Anyone re-running this census on `google.com` may see +28 or +112 depending on which
instance their resolver reached, and should not read either as our behaviour.

*Our responses can exceed the advertised 1232 clamp by the cookie length.* `aol.com`
leaves at **1,256 B** where the clamp advertises 1232: the pre-cookie message is 1,228 B,
fits, is not truncated — and the server cookie is appended afterwards, outside the
budget `Truncate` accounted for. This is **pre-existing and independent of this change**
(it needs an untruncated response, which this change never touches).

**This is a claim about us and only us — an earlier revision of this section said
Cloudflare "does exactly the same thing", and that was a misreading of the OPT.** The
correction is recorded here rather than edited away, because the misreading is an easy
one to repeat:

```
# TXT paypal.com, +dnssec +bufsize=4096 +ignore +notcp, from ad-astra 2026-09-01 17:35
194.180.189.33   advertises udp: 1232   sent 67      (truncated: 1237 + 28 > 1232)
1.1.1.1          advertises udp: 1232   sent 1237
8.8.8.8          advertises udp:  512   sent 1237    <- the row that settles it
9.9.9.9          advertises udp:  512   sent 39
```

The payload size in a **response** OPT is the responder's own maximum *receive* size, not
a cap on what it will send. RFC 6891 §6.2.4 describes it as what a requestor probes with
"an arbitrary QUERY used as a probe to discover a responder's maximum UDP payload size,
followed immediately by an UPDATE that takes advantage of this size" — a buffer for
traffic sent *to* the responder. What bounds the answer is §6.2.3, the **requestor's**
size, and our probe advertised 4096. The `8.8.8.8` row proves it empirically: if the
response OPT were a send limit, Google would be exceeding its own by 725 B on every large
answer. Cloudflare advertising 1232 and sending 1,237 B is therefore not an overshoot at
all — it is a receive buffer and a response size, two unrelated numbers.

So the 1,256 B `aol.com` answer violates no RFC: the query advertised 4096. What it
contradicts is **our own documented send behaviour** — §7.22 records 1232 as the size this
stack honours on the wire, and the cookie puts us 24 B past it. That inconsistency is ours
to own, and nothing comparable about Cloudflare is knowable from outside. **The cookie
cuts both ways:** because 1,237 + 28 = 1,265 exceeds our clamp, we *truncate*
`paypal.com` to 67 B — a name Cloudflare and Google send whole. Recorded, not fixed:
correcting it means accounting for the cookie before the clamp, which is a change to the
cookie path, not the truncation path.

**Verified after deployment:** `v0.107.79-edge` rebuilt against `dnsproxy` `94f9910`,
`GOAMD64=v3`, installed 2026-09-01 17:16 Riga. Truncated answers empty on the wire from
the external vantage; `TCP-full` sizes unchanged either side of the deploy (`microsoft.com`
4,918 B, `google.com` 1,131 B), confirming the RRsets did not move under the measurement;
normal small answers untouched (`A example.com` 100 B `ad`, `MX gmail.com` 189 B,
`NS microsoft.com` 204 B); DNSSEC still validating (`AD` on `cloudflare.com`, 213 B);
cookies still issued and marked `(good)`. Six unit tests in `proxy/dnscontext_internal_test.go`;
the two that assert the new behaviour were confirmed to **fail** against the pre-change
source before being accepted — the other two are guards (cookie preserved, untruncated
responses untouched) and pass either side by design.

---

### 7.24 DoQ 0-RTT Carries Only Replayable Opcodes (2026-09-04)

Upstream `dnsproxy` v0.84.2 is two commits. One (`9b6551d`) bumps the Go directive from
1.26.6 to 1.26.8 — one line of `go.mod`, plus five CI and Docker strings — which this fork
does not need, as builds run on Go 1.27.1. The other (`096ff91`, AGH-32) closes a check
that had been open since `serverquic.go` was written.

> **Correction (2026-09-04, same day).** This section and the release-history row first
> described `9b6551d` as a Go *and* quic-go v0.59.0 → v0.60.0 bump, and presented the
> quic-go half as a dependency move deliberately refused. There was no quic-go half. The
> entire `go.mod` diff between `v0.84.1` and `v0.84.2` is the single `go` line; upstream has
> pinned quic-go v0.60.0 since **`v0.82.0`** — this correction first said `v0.84.0`, which was
> wrong in turn, and the tag walk settles it: `v0.81.1`…`v0.81.4` carry v0.59.0, every tag
> from `v0.82.0` on carries v0.60.0. So the v0.59.0 this fork pins is a standing delta two
> minor releases old, predating this release and untouched by taking v0.84.2. The technical conclusion is unchanged — `ConnectionState().Used0RTT` is present in
> v0.59.0, so the port needed no bump — but "refused a bump" was a claim about something
> upstream never offered here. Raised by the `/var/www` session while fact-checking the news
> article against the release.

`validQUICMsg` walks the protocol-error list of
[RFC 9250 §4.3.3](https://www.rfc-editor.org/rfc/rfc9250#section-4.3.3) and, until now,
stopped at the seventh entry with a comment where a check belongs:

```go
// 7. a server receives a "replayable" transaction in 0-RTT data
//
// The information necessary to validate this is not exposed by quic-go.
```

§4.5 is unambiguous about what that entry means — *"only transactions that have an OPCODE
of QUERY or NOTIFY are considered replayable; therefore, other OPCODES MUST NOT be sent in
0-RTT data"* — and a server supporting 0-RTT "MUST NOT immediately process non-replayable
transactions received in 0-RTT data": it must queue them until the handshake completes,
answer REFUSED with the EDE "Too Early", or close the connection with `DOQ_PROTOCOL_ERROR`.

**Reachable here, not theoretical.** This fork's DoQ listener is created with
`tr.ListenEarly` and `Allow0RTT: true`, AGH serves DoQ on 853 (`port_dns_over_quic: 853`),
and no opcode filter existed anywhere in the proxy — `grep -n Opcode proxy/ upstream/` over
non-test sources returned nothing before this change. A replayed 0-RTT `UPDATE` was handled
like any other message.

**The change** (`e66fa7f`, `Ozy-666/dnsproxy` `edge-udp-pool`). Upstream's implementation
was taken verbatim — the 52-line region is byte-identical to `v0.84.2` — and adapted only
at the seams, this fork's `serverquic.go` having diverged from upstream's by 60/61 lines.
`validQUICMsg` gains `ctx` and the connection; the gate itself:

```go
func isNonReplayableEarlyData(conn *quic.Conn, req *dns.Msg) (ok bool) {
	if isReplayableOpcode(req.Opcode) {
		return false
	}

	if !conn.ConnectionState().Used0RTT {
		return false
	}

	// The data is early data only if it arrived before the handshake completed
	// on a resumed 0-RTT connection.
	select {
	case <-conn.HandshakeComplete():
		return false
	default:
		return true
	}
}
```

The caller closes the connection with `DoQCodeProtocolError` — the third of the three
behaviours §4.5 permits.

**What it does not do.** The handshake state is read when the message is validated, not
when it arrived, so early data whose handshake completes first is classified as 1-RTT and
processed; conversely a genuine 1-RTT non-QUERY message that overtakes the
handshake-complete signal aborts the connection. Both are properties of upstream's design,
kept rather than improved on, and both are confined to opcodes this resolver has no
legitimate use for either way. This is a conformance gate, not a guarantee.

**Verified.** `go build ./...`, `go vet ./proxy/` and `gofmt -l` clean; `go test ./proxy/`
green. `TestIsReplayableOpcode` — added here, upstream shipped no test — covers the half
the RFC pins exactly, and was confirmed able to fail: a mutant returning
`opcode != dns.OpcodeStatus` was caught on the `iquery` and `update` cases before the real
implementation was accepted. The new debug string is present in the installed binary and
absent from the pre-change backup (`AdGuardHome.bak.20260904-171836`), against a negative
control that matches nothing. DoQ serves normally after the deploy:

```
$ kdig +quic +timeout=5 @127.0.0.1 example.com A
;; QUIC session (QUICv1)-(TLS1.3)-(ECDHE-X25519)-(ECDSA-SECP256R1-SHA256)-(AES-128-GCM)
;; ->>HEADER<<- opcode: QUERY; status: NOERROR; id: 0
;; EDNS PSEUDOSECTION: UDP size: 1232 B
example.com.        	287	IN	A	104.20.23.154
;; From 127.0.0.1@853(QUIC) in 5.5 ms
```

**Not verified: the gate itself.** The above establishes that the code is in the running
binary and that a normal QUERY over DoQ is unaffected — the regression risk. It does not
establish that a non-QUERY transaction in early data is actually refused. `kdig` offers no
control over 0-RTT or the opcode, so that path is reasoned from the source, not observed.
Proving it needs a purpose-built quic-go client that resumes a session and writes an
`UPDATE` as early data. Recorded as an open item in §12 rather than claimed.

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

### 8.5 Rejected: Recursion-Detector Cache Shard — a Profiler Phantom (2026-06-05)

After §7.14, the mutex profile pointed hard at the next target: `proxy.(*recursionDetector).check` →
`golibs/cache.(*cache).Get` at **83.74% of mutex-delay**. The recursion detector keeps a small LRU
(`MaxCount: 1000`) of recently-forwarded request signatures to break forwarding loops, and it is checked
*and* updated on every query — an LRU `Get` is a write under the hood (it reorders), so it serializes on
the cache's internal mutex. The obvious move was to reuse the §8.4 shard pattern on it. **We did not — the
data said it would be wasted effort.**

**Clean-room discipline (no patch yet).** With pprof fully off, a heavy baseline (`-c8 -q800 -l30`) held at
**28,152 qps**, and a scaling probe (`-c16 -q1600`) gave **+0% qps for +2× latency** — the shape of a
plateau. But a per-process CPU census exposed the real story: the 4-core box ran at **~0% idle** while
**AdGuardHome used only ~2.4 cores** — the co-located `dnsperf` (~0.8–1.1 cores) and the loopback kernel
network stack (`sys` ~35% + `softirq` ~11%, ~1.8 cores) ate the rest. AGH wasn't lock-bound; it was
**CPU-starved by the test rig**. A lock ceiling looks like *parked goroutines with idle cores to spare* —
the opposite of what we measured.

**The decisive test — ablation.** Stubbing `check`→`false` and `add`→no-op (a full bypass of the cache,
the absolute upper bound of any optimization) and re-running the identical load:

| metric | clean baseline | recursion detector **fully bypassed** | Δ |
|---|---|---|---|
| heavy qps | 28,152 | 28,990 | **+3.0% (within ±3–4% run noise)** |
| avg latency | 28.28 ms | 27.45 ms | −2.9% |
| tail (max) latency | 2.07 s | 2.05 s | flat |
| AGH CPU | ~245% | ~245% | flat |
| box idle | ~0% | ~0% | saturated |

**Deleting the detector entirely buys nothing measurable.** Since that is the ceiling on any change to this
code, *sharding* it — which keeps the work and only splits the lock — would yield strictly less. The
83.74% was a **profiler phantom**: `runtime.SetMutexProfileFraction(1)` inflates an otherwise-cheap LRU
mutex into the top of the *instrumented* delay graph, but it costs ~zero physical throughput. The change
was **rejected** and the ablation reverted; nothing shipped.

> **Lesson.** A mutex profile ranks *where goroutines wait when you ask them to record waiting*, not where
> the wall-clock ceiling is. Two independent guards catch the phantom: (1) **read CPU idle** — if the box
> is saturated and the process is *not* at full cores, the limit is the rig/CPU, not the lock; (2)
> **ablate before you optimize** — if removing the code outright doesn't move throughput, neither will a
> cleverer lock. The honest ceiling on this loopback rig is the harness itself (AGH + generator + kernel
> on the same 4 cores); finding AGH's *true* internal ceiling needs an **off-box load generator** so AGH
> owns all cores. Until then, the four shipped wins (§8.2–8.4, §7.14) are the harvest.

### 8.6 The Test-Harness Ceiling — Measured AGH Capacity is ≈ 2× the Observed Numbers (2026-06-05)

Every throughput number in §8.2–§8.5 was measured with the load generator (`dnsperf`) **on the same
4-core box** as the resolver, driving traffic over loopback. The CPU census in §8.5 hinted at the problem;
two follow-up measurements proved it and **reframe the entire campaign upward.**

**Measurement 1 — sub-saturation CPU sweep.** Driving rate-capped loads (`dnsperf -Q`) and sampling AGH's
own CPU at each rate:

| target qps | achieved | AGH CPU | box idle | µs/query (avg) |
|---|---|---|---|---|
| 4,000 | 3,909 | 82% | 43% | 209.8 |
| 8,000 | 7,783 | 132% | 25% | 169.6 |
| 16,000 | 15,827 | 147% | 4% | 92.9 |
| 20,000 | 18,669 | 156% | 3% | 83.6 |

The falling µs/query is a fixed-cost artifact (≈0.6 core of rate-independent background — 4 UDP reader
loops, stats, qfeed, GC — amortizing over more queries); the **marginal** per-query cost from the slope is
**~50–54 µs**. The decisive observation: when the **box** saturates (idle → ~2-4% at ~18.7k qps), **AGH is
using only ~1.5 cores.** The other ~2.5 cores are the generator plus the loopback kernel netstack
(`sys`+`softirq`). And the ceiling moves with the *generator's* config — `-c8 -q800` reaches 28k, `-c4 -Q`
tops out at ~18.7k for the *same resolver* — the tell-tale sign that **the load tool, not AGH, is the limit.**

**Measurement 2 — CPU-pinned isolation (the clean read).** Pin AGH to 2 cores, `dnsperf` to the other 2:

| config | achieved qps | AGH CPU | box idle | per-core |
|---|---|---|---|---|
| AGH on 2 cores (saturated) | **23,988** | 192% / 200% | **22%** | **~12,500 qps/core** |

AGH **saturated its 2 cores (96%) and served ~24k qps with 22% of the box still idle** — i.e., AGH itself
was the bottleneck on that 2-core budget, cleanly isolated from the generator. Per core ≈ **12.5k qps**
(the sweep independently gave ~11.7k/core). Extrapolated to all 4 cores:

> **AGH true ceiling ≈ 50,000 qps** (flat per-core scaling) **to ≈ 62,000 qps** (fixed-cost-amortized model:
> 62% fixed + ~54 µs/query marginal). **≈ 2× the ~28k the co-located harness ever exposed.**

**What this means for §8.2–§8.5.** Every win was measured through a rig that handed AGH only ~half the
machine. The +5–12% deltas are therefore **understated** — on un-confounded hardware the same per-query
savings ride a ~50–60k ceiling, not a ~28k one. It also recolors the two "null" results honestly: the §8.4
cache-shard reading "throughput-flat" and the §8.5 recursion-detector "phantom" verdict are correct **for
this rig**, where AGH never runs hot enough to stress those locks — but at a real ~50k load some of them
(cache `get`, recursion detector) could re-emerge. They are not bottlenecks at 28k; they are unproven at 55k.

**The honest ceiling number is a load-generation problem, not a resolver problem.** To *drive* AGH to its
real ceiling and re-hunt locks there, the generator must stop stealing the resolver's cores: an off-box
client, or an on-box **AF_XDP generator over a veth pair** (reclaims the generator's ~1 core of netstack
cost; the box already runs a patched xdp-tools data-plane in shield), or at minimum a `sendmmsg`-batched
generator with CPU-pinning. Until then, **~12.5k qps/core is the durable, harness-independent capacity
figure** — and it says the resolver has roughly twice the headroom the raw benchmark rows imply.

### 8.7 Proving the Ceiling — the AF_XDP/veth Load Harness (2026-06-05)

§8.6 *extrapolated* AGH's ceiling at ~50–60k qps. This section *measured* it, by building a load
generator lean enough to stop stealing the resolver's cores.

**Topology.** A `veth` pair bridges a generator network namespace to the host: `veth_gen` (10.0.0.2,
inside `test_ns`) ↔ `veth_agh` (10.0.0.1, host). AGH already listens on `0.0.0.0:53`, so it serves on
`veth_agh` through the *normal* netstack — its real per-query cost is preserved. A custom **AF_XDP TX
blaster** (C, libxdp, SKB/copy mode since veth has no zero-copy) pre-crafts a complete Ethernet+IP+UDP+DNS
`A?` frame and injects it straight into the TX ring, bypassing the generator's userspace stack. An
**`XDP_DROP`** program on `veth_gen` discards AGH's replies at the driver, so the generator never pays a
receive cost. Static neighbor entries let AGH complete its real response `sendmsg` (the reply is then
dropped at `veth_gen`).

**Two traps worth recording.** (1) AGH's reply path needs a static `neigh` for 10.0.0.2, else the queries
are processed but `XDP_DROP` eats the ARP and no reply is emitted. (2) **shield's own nft anti-DDoS ate the
test traffic** — a single-source blaster trips the per-IP `30 qps` limiter in `raw_filter` `prerouting`
(priority `raw`, *before* the `protection input` chain), which autobans the source into `@dns_bad_ip` for
30 min. The fix is an `iifname "veth_agh" accept` at the top of *both* chains — the private veth subnet is
trusted exactly like `lo`/`tailscale0`. (A useful incidental proof that the WAF works.)

**Result — rate sweep, goodput = `veth_agh` `txpck/s` (replies AGH actually served), cloudflare.com cached:**

| offered qps | goodput (served) | loss | AGH CPU | blaster CPU | box idle |
|---|---|---|---|---|---|
| 20,000 | 19,972 | 0% | 0.97 core | 0.09 core | 61% |
| 40,000 | 39,999 | **0%** | 2.04 core | 0.27 core | 31% |
| 50,000 | **50,005** | **0.1%** | 2.97 core | 0.27 core | 14% |
| 60,000 | 58,605 | 2.3% | 3.23 core | 0.27 core | 3% |
| 75,000 | **61,923** (peak) | 17% | 3.23 core | 0.33 core | 2% |
| 110,000 | 52,868 (collapsing) | 52% | 3.07 core | 0.50 core | 3% |

> **AGH served 50,005 qps at 0.1% loss, and peaks at ~62k qps, on a 4-core box** — and the real-user canary
> stayed green throughout, even under the 110k-pps onslaught. Beyond ~75k offered, goodput **collapses**
> (more offered → less served), the signature of an overloaded UDP receive path.

**This confirms §8.6 and overturns the ~28k loopback number — AGH's true ceiling is ~2.2× higher.** The
mechanism is exactly as predicted: the **blaster costs ~0.27 core at 60k pps** vs dnsperf's ~0.8–1.1 core,
so AGH gets ~3.2 cores of the box instead of ~1.6. AGH is CPU-bound at ~3.2 cores (the remaining ~0.8 is
kernel softirq + the light generator) — it never starves on a lock, vindicating the §8.4/§8.5 "those locks
aren't the bottleneck *here*" calls while showing the headroom is real.

**Honest scope.** This is the **cache-hit serving ceiling** (a single hot name served from AGH's cache —
the fast path, no upstream). It is the right "max capacity" figure (and ~57% of real traffic is cache
hits), but a fully cache-missing mix that forwards every query to unbound would serve fewer qps per core.
The harness lives in `xdp-loadgen/` (`topology.sh`, `blaster.c`, `xdp_drop.bpf.c`, `sweep.sh`) and tears
down cleanly (netns + veth + nft whitelist all removed). Headline: **the edge resolver does enterprise-grade
~50k qps clean / ~62k peak on four cores.**

### 8.8 The Full Ceiling Map — Cache-Hit vs Forwarding, and What Actually Bounds It (2026-06-05)

With the harness in hand, two follow-ups: what does a *realistic* (cache-missing) load do, and at the real
ceiling, what is the actual bottleneck?

**(a) Cache-hit vs forwarding ceilings** (same AF_XDP harness, real querylog domains):

| traffic | peak goodput | bound by |
|---|---|---|
| cache-hit (one hot name) | ~50k clean / ~62k peak | AGH CPU (~3.2 cores) |
| real 25k-domain working set, **cache ON** | **50,024 @ 0.2% loss** | still cache (unbound stayed at **0% CPU** — the working set is cache-resident) |
| **cache OFF, every query → unbound** | **~26k** (collapses past ~40k offered) | the upstream round-trip + unbound, *not* AGH (AGH only ~2.5 cores) |

The middle row is the important real-world result: a production-representative working set **fits the 8 MB
front-cache**, so steady-state traffic is ~100% hits and serves the full ~50k. The forwarding path (every
query a miss) tops out near **~26k**, limited by the AGH→unbound round-trip, not AGH's own CPU. Real
production (≈57% hit) therefore lands **between**, ~35–45k qps — still well above the old 28k loopback figure.

**(b) What bounds AGH at its ceiling** (pprof CPU/mutex/block captured under a sustained ~55k AF_XDP load —
AGH served **54,921 qps even with the mutex profiler running**):

- **Block profile: 93% in `respondUDP → UDPWrite → sendmsg`.** The per-response **UDP write syscall** is where
  goroutines spend their blocked time — the dominant serialization at the ceiling, even across 4 SO_REUSEPORT
  sockets.
- **CPU: ~27% in syscalls** (sendmsg/recvmsg), then stats (11%), `mallocgc` (11%), filtering (9.5%),
  cache-get (9%). The cost is **I/O and allocation, not locks.**
- **The recursion-detector lock is a confirmed phantom at 2× load.** It is **40% of mutex-delay** yet
  **absent from the CPU profile (~0%)** — contended-but-cheap, exactly as §8.5's ablation showed at 28k.
  AGH sustaining 55k *with* mutex profiling on is the proof it isn't the limiter.

> **Conclusion: at ~55k the resolver is UDP-syscall/I-O-bound, not lock-bound.** The lock-removal campaign
> (§7.11–§7.14, §8.2–§8.4) took AGH as far as the per-query *compute* path usefully goes; the locks that
> remain in the mutex ranking cost no CPU and cap nothing. The next real frontier is the **per-packet
> syscall** — `sendmmsg`/`recvmmsg` batching or `io_uring` to amortize the write path — a transport-layer
> rearchitecture, not another mutex. That is the honest end of the lock-contention thread.

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

### 8.9 Rejected: recvmmsg Read-Path Batching — Measured, Not Worth It (2026-06-06)

§8.8 named per-packet syscall batching as the only remaining transport lever once the lock-contention
thread closed. The write side (`sendmmsg`) had already been micro-benchmarked at ~24% raw-send /
≈4% total CPU — modest, and it adds response latency. The **read** side was the cleaner candidate:
`udpPacketLoop` already runs one loop per socket, so coalescing receives needs no cross-goroutine queue
and adds no latency.

Implemented as `UDPBatchReader` over `ipv4.PacketConn.ReadBatch` (poller-integrated `recvmmsg`,
`flags=0` = drain only what is already queued, so the netpoller still blocks only until the first
datagram), gated by `DNSPROXY_UDP_RECV_BATCH` (default 16, `=1` restores single-read). A/B'd honestly
on the §8.7 AF_XDP harness:

| offered | 4 SO_REUSEPORT shards (prod) | 1-shard funnel |
|---|---|---|
| 35 k | batch off ≈196% → on ≈175% CPU (noise) | off ≈170% → on ≈195% (**worse**) |
| 45 k | off ≈236% → on ≈242% (flat) | off ≈208% → on ≈227% (**worse**) |

The decisive test is the **1-shard funnel** — the configuration that *should* favor batching (a deep
per-socket backlog) — where it was flat-to-**worse**: `x/net`'s `ReadBatch` carries per-message
overhead (`mmsghdr` pack/unpack, a per-datagram sockaddr allocation) that eats the saved syscalls. No
throughput or ceiling change anywhere. This confirms §8.8's verdict that the cost is the **write** path
(`sendmsg`, 93% of block-delay) plus per-packet kernel work, not read syscalls. **Reverted**, recorded
like §8.5. The `sendmmsg`/`recvmmsg` syscall-batching thread is now closed in both directions — measure
before rearchitecting, both times.

### 8.10 Client-reset write-errors logged at Debug, not Error

A shield siege test surfaced a self-inflicted CPU sink that fired **exactly when the box is under
attack**. Under a DoT/DoQ flood (~95 k req/s, ~400 pipelined clients that hang up mid-response),
the dnsproxy response-write path logs one line per client that drops before AGH finishes writing:

```
[error] dnsproxy: responding request proto=tls err="writing message: write tcp …: write: connection reset by peer"
```

At that load this is ~10 k lines/min, which spiked **systemd-journald to ~45% CPU** — a whole core
spent on *logging benign client hangups* during the flood (it showed up as a post-siege catch-up
flush because journald was CPU-starved at 0% idle during the attack itself).

These resets are not a server fault: a flooding, pipelined, or cancelled client simply closes
before the response is written. dnsproxy's `logWithNonCrit` already downgrades `io.EOF`,
`net.ErrClosed`, `EPIPE` (broken pipe), and timeouts to **Debug** — but **`ECONNRESET`
("connection reset by peer") was the missing case**, so it alone escaped to Error. Fixed in the
dnsproxy fork (`b74b7bb`) by adding an `isECONNRESET` helper (parallel to the existing `isEPIPE`,
with a Plan 9 string variant) to that same Debug branch. Typed `errors.Is(err, syscall.ECONNRESET)`
unwrapping is used rather than substring matching, and genuine non-client write errors stay at
Error so real problems still surface.

A journald rate-limit on the shield box (`LogRateLimitIntervalSec=10s` / `LogRateLimitBurst=500`,
verified to cap 10 k → 500 lines and bring journald CPU 45% → 0%) was the prod stopgap; with the
fork logging resets at Debug it becomes belt-and-suspenders rather than load-bearing.

### 8.11 Fork Recheck — Upstream Parity + GO-2026-5026 Sweep (2026-06-10)

A scheduled recheck of the dnsproxy fork against upstream and the Go vulnerability
database. Upstream had exactly **one** commit since our v0.81.4 base (AGDNS-4074
"limit reads", two hunks):

- **Inbound DoH POST bound** (`serverhttps.go`) — upstream caught up to the fork:
  our `00fc061` had already bounded it, with stricter semantics
  (`dns.MaxMsgSize+1` so an oversized body fails `Unpack` instead of being
  silently truncated to a parseable prefix). Nothing to take.
- **Outbound DoH response bound** (`upstream/doh.go`) — the response body of the
  DoH *upstream* transport was still an unbounded `io.ReadAll`; a misbehaving DoH
  server could feed an arbitrarily large body. Backported in fork commit
  `25d8f46` in the same stdlib style. Unreachable in the edge deployment (single
  plain-UDP upstream to Unbound) — inert hardening for the compiled-in transport.

**GO-2026-5026** (`x/net/idna` accepts ASCII-only Punycode labels it should
reject) was flagged by `govulncheck` as *reachable* in the AGH binary on two
paths: `dnsforward.NewServer → netutil.ValidateDomainName → idna.ToASCII`
(startup) and — the one that matters — `querylog.questionPayload →
idna.ToUnicode`, which runs on **attacker-controlled question names** when
rendering query-log entries (display/spoofing class, not memory safety).
`golang.org/x/net` bumped v0.53.0 → v0.55.0 in AGH (`10ffc8eb`) and the dnsproxy
fork (`ad5e739`); the same sweep had already fixed the dnscrypt-proxy fork (§5).
After the bumps, `govulncheck` reports **0 reachable vulnerabilities** across
all three Go forks. Rebuilt + redeployed; DNS/DoT verified live.

---

## 9. Security Hardening

| Area | Change | Severity |
|---|---|---|
| **DNS reflection/amplification** | Size-gated Response Rate Limiting with `TC=1` slip (§7.18): over-budget large UDP responses to a hot `(qname, qtype)` truncated to a 32-byte header, collapsing a 31.8× amplifier to ~0.45×. Live-validated at 96.6% egress reduction, FP-clean over 67,905 decisions. UDP-only; DoT/DoQ/DoH/TCP exempt | High — resolver used as a reflection weapon against third parties |
| **DoH POST flood** | Body capped at DNS wire-format maximum (65,535 bytes); `413` returned before any DNS unpacking | Medium — memory exhaustion under targeted POST flood |
| **QUIC stream flood** | Per-connection stream limit configurable, default 64 (was 65,535); enforced by QUIC transport `MAX_STREAMS` frame | High — single client could drain global request semaphore, starving all others |
| **Cloud telemetry** | SafeBrowsing, Parental Controls, EDNS-CS all removed; no DNS query data leaves the local machine | Privacy — eliminates data leakage to third-party cloud services |
| **Auto-update block** | `-edge` suffix detected at runtime; auto-update disabled; upstream release cannot overwrite patched binary | Integrity |
| **ODoH removed (dnscrypt-proxy)** | Oblivious DoH stripped entirely (~800 LOC: crypto, transport, target-config fetch, stamps, `odoh_servers` option) | Surface reduction — removes an HPKE-style crypto/wire path and a content-type handler |
| **No discretionary egress (dnscrypt-proxy)** | Remote source-list downloads disabled (local signed cache only); upstreams pinned by `[static]` stamps; no auto-update / version-check call exists (audited) | Privacy — only encrypted DNS queries leave the host |
| **Transport timeout tightening (dnscrypt-proxy)** | 30 s default transport timeouts wired to the 800 ms query budget; h2 idle health-check decoupled (10 s read-idle / 5 s ping) | Resource — bounds connect/read and reaps dead keep-alive connections promptly |
| **Whois TCP connections** | Replaced with local mmdb; no per-query outbound TCP | Privacy — eliminates external connection from query log enrichment |
| **Data race fix** | Access manager field in middleware read without synchronization; fixed to atomic load | Correctness — undefined behaviour under Go memory model |
| **x/net idna Punycode (GO-2026-5026)** | `golang.org/x/net` bumped to v0.55.0 across AGH, dnsproxy, and dnscrypt-proxy forks; reachable via query-log name rendering (`idna.ToUnicode` on attacker-controlled qnames) and the dnscrypt DoH transport. `govulncheck`: 0 reachable vulns after | Low/Medium — bad-Punycode acceptance (spoofing/display), not memory safety |
| **DoH response body bound (dnsproxy)** | Upstream AGDNS-4074 backport: DoH *upstream* response read bounded at `dns.MaxMsgSize+1` (inbound POST side was already bounded by the fork first) | Low — unbounded read from a misbehaving DoH server; transport unreachable in this deployment |
| **GLiNet path traversal (CVE-2026-41448)** | Backported the upstream v0.107.77 fix: GLiNet auth-token file reads confined to an `os.Root` (rejects `..`/absolute escapes), token filename reduced to a basename. Not exploitable in this deployment (runs without `--glinet`, so the GLiNet middleware is never installed); applied as defense-in-depth and to keep the fork mergeable | Medium — admin-auth bypass via a crafted `Admin-Token` cookie in GLiNet mode |
| **DoH upstream ID handling + response validation (dnsproxy)** | Upstream AGDNS-4080 backport: RFC 8484 ID-zeroing moved into the packed wire bytes (request message no longer mutated), responses must echo ID 0, and DoH upstream responses now get the same question-section (qtype/qname) validation as plain DNS — a malformed response was previously patched up (question section reconstructed from the request) instead of rejected. Fork adaptation: an invalid response also evicts the pooled plain-upstream connection (§8.1) instead of returning it to the pool. Transport unreachable in this deployment (single plain-UDP upstream); inert hardening for the compiled-in transport | Low — acceptance of malformed/mismatched upstream DoH responses |
| **h2c upgrade path removed** | Upstream AGDNS-4111/AGDNS-4038 backport: the plain-HTTP server (unix-socket DoH ingress + localhost UI) replaced the x/net h2c handler - whose HTTP/1.1→h2c upgrade path reads the request body without a bound - with stdlib `http.Protocols` (HTTP/1.1 + prior-knowledge unencrypted HTTP/2 only, RFC 9113). Unreachable from outside (fronting nginx clears the `Connection` header); defense-in-depth + removes the x/net h2c dependency from the server path | Low here — unbounded read on the upgrade path, admin/API surface is localhost-only |
| **Rulelist download size cap** | Upstream AGDNS-4081 backport: filter-list updates stream through `ioutil.LimitReader` capped by new `filtering.max_http_size` (default 256 MB, upstream's raised `DefaultMaxRuleListSize`); download path split into `readFromHTTP`/`readFromFile`. Relevant here: 4 external blocklists auto-fetched on schedule; a misbehaving source can no longer stream an unbounded body through the parser | Low/Medium — disk/CPU exhaustion from a malicious or broken blocklist source |
| **ECH base64 in DNS rewrites** | Upstream #8276 backport (1 line, `svcbmsg.go`): `ech=` values in SVCB/HTTPS rewrite rules are unpadded base64 but the parser required padding (`base64.StdEncoding` → `RawStdEncoding`), so valid `ech=` pairs were silently dropped from rewrite answers | Low — correctness of user-defined SVCB/HTTPS rewrites, not the server's own ECH |
| **DoQ 0-RTT replay (dnsproxy)** | RFC 9250 §4.5 enforced at last (upstream `096ff91`/AGH-32 in v0.84.2, fork `e66fa7f`): a transaction whose opcode is neither QUERY nor NOTIFY, arriving as QUIC early data before the handshake completes, closes the connection with `DOQ_PROTOCOL_ERROR` instead of being processed. The listener runs with `Allow0RTT` and no opcode filter existed anywhere in the proxy, so replayed early data was previously handled like any other message. Best-effort by construction — the handshake state is read at validation time, not at arrival (§7.24) | Low — replay of a captured non-QUERY 0-RTT transaction; this deployment forwards to a local upstream that refuses such opcodes anyway |

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
| `ddr_external_doh` | `false` | bool | Adds a DoH designation (`alpn=h2,h3 port=443 dohpath=/dns-query{?dns}` at `tls.server_name`) to DDR responses when DoH TLS is terminated by the fronting proxy instead of AGH (§7.16). |
| `ddr_external_doh_target` | (empty) | hostname | Overrides the `ddr_external_doh` designation target. Point at a vhost whose cert contains the target name AND the resolver IP for verified DDR, RFC 9462 §4.2 (§7.16). |
| `ddr_ipv4_hint` | (unset) | IPv4 address | Adds `ipv4hint=<addr>` to every DDR designation (DoH, DoT, DoQ) so clients skip the A lookup for the target name during bootstrap (§7.17). |

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
| `v0.107.79-edge` | 2026-09-04 | **security:** DoQ 0-RTT early data now carries **only replayable opcodes** (dnsproxy fork `e66fa7f`, adapting upstream `096ff91`/AGH-32, the sole functional commit of v0.84.2). RFC 9250 §4.5 permits only QUERY and NOTIFY in 0-RTT; `validQUICMsg` had carried the other six protocol-error checks and a comment in place of the seventh ("not exposed by quic-go") since the file was written. A non-replayable opcode on a resumed connection, seen before `HandshakeComplete()`, now closes the connection with `DOQ_PROTOCOL_ERROR`. Reachable here — the listener uses `ListenEarly` with `Allow0RTT` and the proxy filtered no opcodes at all. The release's other commit is a Go directive bump only (1.26.6 → 1.26.8); this fork builds on 1.27.1. No quic-go bump was needed or on offer — `ConnectionState().Used0RTT` is already in the pinned v0.59.0 (this row first said otherwise; see the correction in §7.24). The gate's 0-RTT half is not empirically verified — no client here can drive it (§7.24, §12) |
| `v0.107.79-edge` | 2026-09-01 | **security:** truncated UDP responses are now **emptied, not filled** (dnsproxy fork `c9c8de7`). `(*dns.Msg).Truncate` packs the datagram with as many RRs as fit before setting `TC`; RFC 2181 §9 forbids a conforming client from using them, so they were reflectable bytes serving no client. `proxy.scrub` now drops `Answer`/`Ns` and reduces `Extra` to the OPT once `TC` is set on plain UDP — header + question + OPT, as Google/Cloudflare/Quad9 return. The OPT is kept for the DNS Cookie (return-routability on the TCP retry) and the RFC 6891 EDNS echo. Complements the 1232 clamp rather than replacing it: the clamp decides *when* `TC` is set, this decides *how much* is reflected once it is. Census from the ad-astra vantage: 15 of 18 TXT-amplification names collapse from 1,023–1,256 B to **63–71 B**; the residual 1.4× vs the majors' 1.0× is the 28-byte cookie, verified with `+nocookie`. TCP/DoT/DoQ/DoH structurally unaffected (`dnsSize` returns `MaxMsgSize`, so `TC` never sets) (§7.23) |
| `v0.107.79-edge` | 2026-08-21 | **security:** upstream v0.107.79 reviewed; the release's own security content needed nothing from its tree — GHSA-w6v6-f44j-3rj2 (DoQ unidirectional stream state exhaustion) is a `dnsproxy` advisory this fork answered on 2026-07-31 in `df16938`, and the other item is the Go toolchain, addressed by rebuilding on **Go 1.26.6** (security fixes to the `go` command, `crypto/tls`, `encoding/asn1`, `encoding/xml`, `html/template`, `net`, `net/http`, `net/url`; the deployed binary had been built with 1.26.5). **fix:** generated responses now echo the request's EDNS(0) DO bit and buffer size (AGH #8183) — adapted, not copied: the advertised size is clamped to `maxAdvertisedUDPSize` (1232), because upstream's raw echo would advertise a buffer this stack never honours on exactly the responses cheapest to reflect. Blocked answers previously carried the cookie path's OPT at 512 with DO dropped. Also taken: bootstrap comment filtering and CNAME rewrite-target validation. Skipped: the DDR/`TLSConfigProvider` refactor (restores the priority tie §7.19 removed), the DNS64 fix (`use_dns64: false`), and dnsproxy v0.84.x (`AGDNS-4357` unexports every `proxy.Proxy` field, no security content) (§7.22) |
| `v0.107.78-edge` | 2026-07-31 | **security:** upstream v0.107.78 reviewed and three `dnsproxy` patches taken — AA bit cleared on relayed responses (AGH #7955), FORMERR instead of a silent drop for malformed UDP (GHSA-p5f5-3p5g-rfjw, the JIGGLE mitigation), and unidirectional QUIC streams refused on DoQ with DoH3 split into its own config (GHSA-qr92-rwvw-mhgh / GHSA-cccx-2r6r-m9r4, where this fork was already ahead at 64 vs upstream's `math.MaxUint16`). Nothing was needed in AdGuard Home itself; the DNSCrypt-upstream fixes and `max_http_size` were skipped as not applicable. The FORMERR reply was measured at **0.36×** the triggering packet, a deamplifier. Closes the "deferred by decision" dnsproxy v0.83.0 item below (§7.21) |
| `v0.107.77-edge` | 2026-07-30 | **feat:** DDR designations now **ranked** `1` DoH (443, `h3,h2`) / `2` DoQ / `3` DoT instead of sharing priority 1 (upstream's own `TODO`). Equal priorities are a tie that RFC 9460 §2.4.1 has clients break at random, so a client supporting all three could land on the slowest transport on every discovery. Ranked by **reachability, not speed**: DoQ measures fastest but sits on port 853, which restrictive networks block, and a blocked port costs a timeout before fallback; 443 carries `h3`+`h2` in one designation so an HTTP/3 failure falls back to TCP without leaving the record (§7.19) |
| `v0.107.77-edge` | 2026-07-30 | **fix:** DoT listeners now offer the RFC 7858 §6 `dot` ALPN (dnsproxy fork `14c4d5b`) — the DDR designation advertised `alpn="dot"` while the port-853 handshake negotiated no ALPN at all, because upstream passes the shared `TLSConfig` to `tls.NewListener` unchanged. ALPN stays optional per RFC 7858 §3.1 (a client sending none still connects, verified); the only behavior change is RFC 7301 conformance for a client offering no common token (§7.20) |
| `v0.107.77-edge` | 2026-07-10 | **feat:** `dns.ddr_ipv4_hint` — every DDR designation (DoH, DoT, DoQ) now carries `ipv4hint=<resolver IP>`, matching Cloudflare's record shape; bootstrapping clients open the encrypted handshake without a separate A lookup for the target name (one RTT saved). Inserted before `dohpath` to keep SVCB params in ascending wire order (RFC 9460 §2.2) (§7.17) |
| `v0.107.77-edge` | 2026-07-07 | **security:** upstream cherry-pick sweep — dnsproxy AGDNS-4080 (DoH upstream ID-zeroing in wire bytes, ID-0 echo required, question-section validation unified with plain DNS; fork's pooled connection evicted on invalid response) + AGH #8276 (`ech=` rewrite values parse as unpadded base64). Same-day follow-ups: AGH AGDNS-4081 (rulelist downloads bounded by new `filtering.max_http_size`, default 256 MB) + AGDNS-4111/4038 (h2c upgrade path removed, stdlib prior-knowledge HTTP/2; AG-54599 assessed N/A - blocked services stripped). Deferred by decision: dnsproxy v0.83.0 rebase (FORMERR responses land on the fork's UDP-pool code) (§9) |
| `v0.107.77-edge` | 2026-07-02 | **feat:** `dns.ddr_external_doh_target` — DDR DoH designation can target a dedicated vhost whose shortlived LE cert carries the resolver IP SAN, enabling strict verified discovery (RFC 9462 §4.2) with the blast radius confined to the discovery lineage (§7.16) |
| `v0.107.77-edge` | 2026-07-02 | **feat:** DDR now advertises the proxy-terminated DoH endpoint — new opt-in `dns.ddr_external_doh` appends `alpn="h2,h3" port=443 dohpath="/dns-query{?dns}"` to `_dns.resolver.arpa` SVCB answers (upstream only advertises listeners AGH terminates itself, so the nginx-fronted DoH was invisible to discovery and only DoQ was designated) (§7.16) |
| `v0.107.77-edge` | 2026-06-08 | **resilience:** client connection-reset write-errors logged at **Debug, not Error** (dnsproxy fork `b74b7bb`) — a DoT/DoQ flood of pipelined clients that hang up mid-response was emitting ~10 k ERROR lines/min, spiking systemd-journald to **~45% CPU during the attack**. `logWithNonCrit` already downgraded EOF/`ErrClosed`/EPIPE/timeout; `ECONNRESET` was the missing case. Added `isECONNRESET` to that Debug branch; genuine write errors stay at Error (§8.10) |
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
| Server cookie appended outside the 1232 clamp budget (§7.23) | 1 | ⚠️ Open — recorded 2026-09-01, not fixed |
| DoQ 0-RTT opcode gate not empirically verified (§7.24) | 1 | ⚠️ Open — verification gap, recorded 2026-09-04 |

**Open items outside the original audit.** The DNS Cookie is appended to the response
OPT after `Truncate` has accounted for the EDNS(0) buffer, so an untruncated response that
fits at 1,228 B leaves at 1,256 B — 24 B above the clamp this stack advertises
(`aol.com`/TXT, measured 2026-09-01, §7.23). It predates the §7.23 truncation change and is
not reachable through it: the overshoot needs a response that was never truncated. It
violates no RFC — the probe advertised 4096, and a response OPT is a receive buffer, not a
send cap (§6.2.4) — but it contradicts the 1232 send behaviour §7.22 documents for this
stack. Fixing it means charging the cookie against the budget in the cookie path, not the
truncation path.

**The second is a verification gap, not a defect.** The DoQ 0-RTT opcode gate (§7.24) is
deployed and its opcode half is unit-tested, but no client available here can send a
non-QUERY transaction as QUIC early data, so the branch that closes the connection has
been reasoned from the source rather than observed on the wire. Closing it means writing a
quic-go client that resumes a session and writes an `UPDATE` in 0-RTT. Until then the
claim in §9 rests on code reading, and is marked as such.

All 23 audit items remain closed. The `urlfilter` `noIndex` regex-gate optimization (§7.7) was
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
