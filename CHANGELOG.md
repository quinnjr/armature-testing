# Changelog — `armature-testing`

All notable changes to this crate will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this crate adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

Earlier changes are recorded in the workspace [`CHANGELOG.md`](../CHANGELOG.md).

## [Unreleased]

### Fixed

- **Breaking:** `Spy` is exported and requires `T: Clone`. It was advertised as a feature but unreachable outside the crate, and its manual-recording contract is now documented rather than implied to intercept.
- Load-test workers accumulate timings locally instead of contending on one mutex per completed request — the measurement path inflated the latencies it recorded — and the percentile ranks use one convention.

### Added

- `Spy` is exported. It was listed as a crate feature but `mod mock` was private, so it was unreachable. Its docs now state plainly that it is a manual recorder: `inner()` returns the wrapped value untouched and calls made through it are not intercepted.
- `Spy::calls()` and `Spy::clear()`.

### Changed

- Load-test workers accumulate latencies in per-worker buffers merged after join, instead of taking a shared `Mutex` per completed request — the measurement path no longer inflates the latencies it records.
- `median`, `p95` and `p99` all use one nearest-rank formula. `median` previously took the upper median while p95/p99 truncated, so the reported quantiles came from two different definitions.

### Changed — `0.2.1` → `0.2.2`

- Migrated onto `armature-core` `0.8`'s `Bytes`-backed request and response types. No behavior change beyond what that migration implies; see [`armature-core/CHANGELOG.md`](../armature-core/CHANGELOG.md).
- `TestRequestBuilder` builds its body as `Bytes`.

## [0.4.0] - 2026-08-05

### Changed

- **Requires `armature-core` 0.9 (breaking).** The requirement moved `0.8` →
  `0.9`. `armature-core 0.9.0` itself moves `armature-h1` across a breaking
  0.x boundary; because `armature-core` types appear in this crate's own
  public API, the requirement change is breaking here too and the minor moves
  with it. Under Cargo's 0.x caret rules the 0.8 and 0.9 types are distinct
  and do not unify, so a consumer holding an `armature-core 0.8` type cannot
  pass it to this crate. Part of the `armature-core 0.9.0` release train; see
  `armature-core`'s CHANGELOG for the publish order.

## [0.3.1] - 2026-08-04

### Fixed

- Requirements on sibling armature crates name a minor instead of `0`. Under
  Cargo's 0.x rules `version = "0"` matches any release ever made, and edition
  2024 selects the MSRV-aware resolver, so a consumer declaring an older
  `rust-version` was handed the oldest version satisfying it — resolving
  `armature-core = "0"` on Rust 1.89 produced `armature-core 0.2.3` while an
  explicit `armature-core = "0.8"` elsewhere in the same graph pulled 0.8.2.
  Two copies of core, and a build failing on symbols the older one lacks. Each
  0.x minor in this family is a breaking change, so the requirement now names
  one. No API change.
