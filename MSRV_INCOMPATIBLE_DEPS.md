# MSRV-Incompatible Dependencies

This file documents dependency versions that are incompatible with Solana's BPF toolchain (rustc 1.79.0-dev) used by `cargo build-sbf`.

## Problematic Versions (NEVER USE)

### base64ct
- **base64ct v1.8.1** - requires Cargo edition2024 (not rustc version, but Cargo 1.79 doesn't support it)

**Compatible alternatives:**
- base64ct v1.6.0 ✓

### indexmap
- **indexmap v2.12.1** - requires rustc 1.82+

**Source:** Transitive dependency from serde_json v1.0.148 → jupnet-signature

**Compatible alternatives:**
- indexmap v2.11.4 ✓ (but may conflict with some other requirements)
- indexmap v2.6.0 (conflicts with borsh requirements >= 2.11.4)

### five8 ecosystem
- **five8 v1.0.0** - requires rustc 1.81+
- **five8_core v1.0.0** - requires rustc 1.81+
- **five8_const v1.0.0** - requires rustc 1.81+

**Source:** Used by jupnet-signature v1.0.0 in jupnet-sdk fix/sdk-deps-test branch

**Compatible alternatives:**
- five8 v0.2.1 ✓
- five8_core v0.1.2 ✓
- five8_const v0.1.2 ✓

### indexmap
- **indexmap v2.12.1** - requires rustc 1.82+

**Source:** Transitive dependency from serde_json v1.0.148 → jupnet-signature

**Compatible alternatives:**
- indexmap v2.6.0 or earlier (but conflicts with borsh requirements >= 2.11.4)

### solana-address
- **solana-address v2.0.0** (from jupnet-sdk fix/sdk-deps-test) - requires rustc 1.81+
  - Uses five8 v1.0.0 internally

**Compatible alternatives:**
- solana-address v1.1.0 ✓ (from crates.io)

## Root Cause

The jupnet-sdk repository's `fix/sdk-deps-test` branch has jupnet-signature depending on five8 v1.0.0, which cascades into multiple MSRV violations.

## Dependency Chain

```
jupnet-signature v1.0.0 (jupnet-sdk fix/sdk-deps-test)
└── five8 v1.0.0 (requires rustc 1.81+)
    ├── five8_core v1.0.0 (requires rustc 1.81+)
    └── solana-address v2.0.0 (requires rustc 1.81+)
└── serde_json v1.0.148
    └── indexmap v2.12.1 (requires rustc 1.82+)
```

## Status

**RESOLVED:** jupnet-sdk fix/sdk-deps-test branch has been fixed:
- ✅ five8 downgraded from v1.0.0 to v0.2.1
- ✅ base64ct downgraded to v1.6.0
- ✅ indexmap downgraded to v2.11.4
- ✅ All Error trait implementations use conditional compilation for std::error::Error
- ✅ cargo build-sbf works (BPF compilation with rustc 1.79.0)
- ✅ cargo test --lib works (non-BPF builds)

## Other Known MSRV Issues

### base64ct
- **base64ct v1.8.1** - requires Cargo edition2024 (not rustc version, but Cargo 1.79 doesn't support it)

**Fix applied:** Downgraded to base64ct v1.6.0 using `cargo update base64ct --precise 1.6.0`

## Verification Commands

Check for MSRV violations:
```bash
cargo build-sbf
```

Check dependency tree:
```bash
cargo tree -i five8_core
cargo tree -i indexmap
cargo tree -i solana-address
```

## Date

Created: 2025-12-30
Last Updated: 2025-12-30
