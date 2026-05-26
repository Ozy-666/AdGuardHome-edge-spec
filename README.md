# AdGuardHome Edge - Public Specification

[![Production](https://img.shields.io/badge/Powered_By-dnsdoh.art-000000?style=flat-square&logo=ghost)](https://dnsdoh.art)
[![Stack](https://img.shields.io/badge/Stack-Nginx%20%E2%86%92%20AGH%20%E2%86%92%20Unbound%20%E2%86%92%20dnscrypt--proxy-007acc?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Protocols](https://img.shields.io/badge/Protocols-DoH3%20%7C%20DoH%20%7C%20DoQ%20%7C%20DoT%20%7C%20DNS-8a2be2?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Binary Size](https://img.shields.io/badge/Binary%20Size-24.6%20MB%20(--10%20MB)-brightgreen?style=flat-square&logo=go)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![UDP Write Allocations](https://img.shields.io/badge/UDP%20Write%20Allocations-0%20B%2Fop-brightgreen?style=flat-square&logo=go)](https://github.com/Ozy-666/dnsproxy)
[![Query Buffers](https://img.shields.io/badge/Query%20Buffers-Zero--Allocation-brightgreen?style=flat-square&logo=go)](https://github.com/Ozy-666/dnscrypt-proxy)
[![Regexp Engine](https://img.shields.io/badge/urlfilter-O(1)%20Regexp%20Shortcut-brightgreen?style=flat-square&logo=go)](https://github.com/Ozy-666/urlfilter)
[![EDNS Subnet](https://img.shields.io/badge/EDNS--Client%20Subnet-Stripped%20%2F%20No%20Leakage-success?style=flat-square&logo=securityscorecard)](https://github.com/Ozy-666/AdGuardHome-edge-spec)
[![Bloat Removed](https://img.shields.io/badge/Codebase%20Bloat-13k%2B%20Lines%20Removed-red?style=flat-square)](https://github.com/Ozy-666/AdGuardHome-edge-spec)

---

**Repository status:** This document is the public architectural specification and progress tracker for the AdGuardHome Edge project. The main codebase and production configuration remain private. This repository exists to document the design philosophy, engineering decisions, and measurable outcomes of the project in a form that can be shared openly.

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
5. [Performance Engineering — AGH Layer](#5-performance-engineering--agh-layer)
6. [Performance Engineering — Transport Layer (dnsproxy)](#6-performance-engineering--transport-layer-dnsproxy)
7. [Security Hardening](#7-security-hardening)
8. [Configuration Reference](#8-configuration-reference)
9. [Release History](#9-release-history)
10. [Completeness Status](#10-completeness-status)

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
  unique-subdomain floods (§5.7).

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

## 5. Performance Engineering — AGH Layer

The following is a complete enumeration of all structural changes applied to
the AdGuardHome application layer. Each item was identified by static code
audit and verified by benchmark or race detector.

### 5.1 Memory Management & Allocation Elimination

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

### 5.2 Concurrency & Lock Contention

| # | Location | Problem | Fix | Version |
|---|---|---|---|---|
| 2.1 | Response filtering | `serverLock.RLock()` acquired once **per answer RR** (N locks per response) | Single lock over the full answer loop; lock-free inner function extracted | v0.107.77 |
| 2.2 | Stats update | Exclusive write lock used for read-only field access | Corrected to read lock; concurrent stat updates no longer serialised | v0.107.78 |
| 2.3 | Client count check | Redundant outer lock wrapping a call to an already-thread-safe method | Outer lock removed | v0.107.78 |
| 2.4 | Query log + stats | Global lock held across `ShouldLog`, `logQuery`, `ShouldCount`, `updateStats` | Narrowed to two-line pointer snapshot; lock released before any I/O | v0.107.79 |
| 2.5 | Client storage | Exclusive mutex serialised all concurrent reads behind writes | Promoted to `sync.RWMutex`; six read-only methods use read lock | v0.107.84 |
| 2.6 | Query log buffer | Lock held during goroutine creation | Explicit unlock before goroutine spawn | v0.107.85 |
| 2.7 | Query log flush | Flush-pending flag stuck after encoding error → permanent flush blockage | Buffer clear and flag reset now unconditional before error return | v0.107.85 |

### 5.3 serverLock Hot-Path Elimination (atomic.Pointer)

The global server RWMutex survived on three hot-path call sites after all
scope-narrowing work was complete. Each was converted to an atomic pointer:

| # | Call site | Before | After | Version |
|---|---|---|---|---|
| 5.1 | Access check (outermost middleware, every request) | RLock on every DNS request | Single atomic load; also fixed a latent data race with no synchronization at all | v0.107.94 |
| 5.2 | Client IP processing | RLock held across full address-processor call (including channel send) | Lock held only for two-word interface snapshot; released before call | v0.107.95 |
| 5.3 | DNS filter (entire filtering hot path) | RLock held across entire `filterDNSRequest` body | Field converted to `atomic.Pointer`; filtering path is completely lock-free | v0.107.96 |

### 5.4 State Lifecycle & Context Correctness

| # | Issue | Fix | Version |
|---|---|---|---|
| 3.1 | `context.TODO()` throughout query log and address processor | `Add/ShouldLog` gain `ctx` parameter; address processor accepts server context | v0.107.90 |
| 3.2 | Address processor silent IP drops | Drop counter; first drop logs at `Warn`; subsequent at `Debug` with running total | v0.107.91 |
| 3.3 | `Reconfigure` 100 ms sleep under global lock | Removed: `Shutdown()` is synchronous; `SO_REUSEADDR` makes the grace period unnecessary | v0.107.87 |
| 3.5 | Log rotation goroutine leak | Cancel/done lifecycle; `Shutdown()` cancels and waits; `Start()` re-entrant-safe | v0.107.88 |

### 5.5 Miscellaneous Correctness Fixes

| Fix | Version |
|---|---|
| Config disk-write full struct shallow-copy replaced with selective field assignment; runtime-only fields no longer shared with caller | v0.107.93 |
| Query log fast-paths: skip client storage lookup when logging globally disabled; skip when host matches ignore list | v0.107.89 |
| Stats render maps pre-sized across 6 accumulator sites; unbounded growth during large retention window render eliminated | v0.107.86 |
| GeoIP mmdb reader file-handle leak fixed: handles released on shutdown | v0.107.75 |
| IPv4-mapped address normalisation: `::ffff:x.x.x.x` looked up as 4-byte IPv4 in GeoIP, not 16-byte IPv6 | v0.107.75 |

### 5.6 Domain Match Cache (Copy-on-Write)

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
**shelved** — see §5.7.

> **Update (v0.107.107):** a system-wide audit found this copy-on-write cache
> copied the whole map on every *miss* (≈489 KB/op once full), which a
> unique-domain flood could weaponize into a DoS-amplification vector. It was
> replaced with a lock-free fixed-size table — see §5.8.

---

### 5.7 noIndex Regexp Scan — AST Shortcut Extraction (v0.107.109)

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
match cache (§5.6/§5.8) shields only *warm hits*; a unique-subdomain flood is
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

### 5.8 Domain Match Cache v2 — Lock-Free Fixed-Size Table

**Problem (found in audit, v0.107.106 cache):** the copy-on-write cache (§5.6)
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

## 6. Performance Engineering — Transport Layer (dnsproxy)

Summary of all transport-layer changes in the
[edge fork](https://github.com/Ozy-666/dnsproxy) (`edge-udp-pool` branch,
based on upstream `v0.81.4`):

| Commit | Change | Impact |
|---|---|---|
| `bbd79ad` | `atomic.Bool` for server started state; TCP/DoT pool (65537-byte slots) | State check: lock-free. TCP: **4 allocs/RTT → 0** |
| `0b14b22` | Upstream RTT mutex → `atomic.Pointer` copy-on-write map | Weight calculation lock-free; upstream dispatch no longer serialised |
| `00fc061` | DoH POST body: unbounded read → capped at DNS wire maximum | Memory-exhaustion POST flood mitigated; 413 on oversized body |
| `716e780` | QUIC stream limit 65535 → configurable, default 64 | Stream-flood attack surface eliminated at transport layer |

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

## 7. Security Hardening

| Area | Change | Severity |
|---|---|---|
| **DoH POST flood** | Body capped at DNS wire-format maximum (65,535 bytes); `413` returned before any DNS unpacking | Medium — memory exhaustion under targeted POST flood |
| **QUIC stream flood** | Per-connection stream limit configurable, default 64 (was 65,535); enforced by QUIC transport `MAX_STREAMS` frame | High — single client could drain global request semaphore, starving all others |
| **Cloud telemetry** | SafeBrowsing, Parental Controls, EDNS-CS all removed; no DNS query data leaves the local machine | Privacy — eliminates data leakage to third-party cloud services |
| **Auto-update block** | `-edge` suffix detected at runtime; auto-update disabled; upstream release cannot overwrite patched binary | Integrity |
| **Whois TCP connections** | Replaced with local mmdb; no per-query outbound TCP | Privacy — eliminates external connection from query log enrichment |
| **Data race fix** | Access manager field in middleware read without synchronization; fixed to atomic load | Correctness — undefined behaviour under Go memory model |

---

## 8. Configuration Reference

Key fields specific to the Edge build. All fields live under their respective
top-level sections in `AdGuardHome.yaml`.

### `dns` section

| Field | Default | Valid range | Description |
|---|---|---|---|
| `quic_max_incoming_streams` | `64` | `[1, 1024]` or `0` | Max concurrent QUIC streams per connection for DoQ/DoH3. `0` silently maps to 64. Values outside `[1, 1024]` log a warning and fall back to 64. Introduced in schema v35. |
| `upstream_timeout` | `1s` | any duration | Per-request upstream deadline. Should be set below dnscrypt-proxy's own timeout to avoid wasted post-deadline work. |
| `max_goroutines` | `300` | positive int | Maximum parallel DNS request goroutines. |

### `http` section

| Field | Default | Description |
|---|---|---|
| `socket_path` | (unset) | When set, the plain HTTP server binds to a Unix domain socket instead of a TCP address. Required for nginx UDS proxying. |

### Schema versions

| Version | Change |
|---|---|
| v35 | Added `dns.quic_max_incoming_streams` (no-op data migration; absent field silently maps to default 64) |
| v34 | DoH route configuration moved into `http.doh` sub-section |

---

## 9. Release History

| Version | Date | Summary |
|---|---|---|
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

## 10. Completeness Status

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
| urlfilter noIndex regexp scan (§5.7) | 1 | ✅ Resolved via AST shortcut extraction (v0.107.109) |
| Post-audit hardening — match cache v2 (§5.8) | 1 | ✅ Complete (C1, v0.107.107) |
| Post-audit hardening — QUIC uni-stream decouple (§4.5) | 1 | ✅ Complete (W1, v0.107.108) |

No open items remain. The `urlfilter` `noIndex` regex-gate optimization (§5.7) was
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
