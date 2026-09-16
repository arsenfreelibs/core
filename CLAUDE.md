# Alt Chat Core (Rust) — Claude Rules

## Project Overview

This is the rust core for **Alt Chat**, a fork of [deltachat/deltachat-core-rust](https://github.com/deltachat/deltachat-core-rust).

- **This repo:** `arsenfreelibs/core.git` — branch `develop`
- **Upstream:** `upstream` remote → `deltachat/deltachat-core-rust` (branch `main`)
- **Used by Android:** `arsenfreelibs/altchat-android`, submodule at `jni/deltachat-core-rust/`
- **Used by iOS:** `arsenfreelibs/altchat-ios`, submodule at `deltachat-ios/libraries/deltachat-core-rust/`
- **Support email:** `child.aplic@gmail.com`

---

## CRITICAL: Branding Rules

**NEVER** let any of the following remain in user-facing strings:

| Forbidden | Replace with |
|-----------|-------------|
| `Delta Chat` | `Alt Chat` |
| `DeltaChat` | `Alt Chat` |
| `delta.chat` | *(remove)* |
| `deltachat.org` | *(remove)* |
| `delta@merlinux.eu` | `child.aplic@gmail.com` |
| `support.delta.chat` | `child.aplic@gmail.com` |

**Do NOT change:**
- `github.com/deltachat/` links (source code references)
- Rust crate names (`deltachat`, `deltachat-rpc-server`)
- HTML anchor IDs like `#what-is-delta-chat`
- **IMAP folder name `"DeltaChat"`** (`imap.rs` `if folder == "DeltaChat"`, `sql/migrations.rs` default mvbox) — server-side folder shared with iOS/desktop/upstream clients; renaming orphans users' mail.
- **IMAP ID `("name", "Delta Chat")`** in `imap/client.rs` — client identifier sent to the server, not shown to users.
- **Default ICE servers** `nine.testrun.org` / `turn.delta.chat` (+ public credential) in `calls.rs` — working infrastructure for calls; replace only when we run our own STUN/TURN.
- **`NOTIFIERS_PUBLIC_KEY` in `push.rs`** — this is OUR key for `notifications.alt-to.online` (differs from upstream's). Device tokens are encrypted to it; taking upstream's `push.rs` wholesale breaks push. Always verify after a merge (see "Known hybrids").
- Test fixtures / test mails containing "Delta Chat" (`*_tests.rs`, `e2ee.rs`, `simplify.rs`, `dehtml.rs`, `reaction.rs`) — test data, not user-facing. Test **assertions** that check our branded strings must say "Alt Chat" though (see table).

**Invite links:** the fork uses `https://alt-chat.me/#` instead of upstream's `https://i.delta.chat/#` — generation in `securejoin.rs`, parsing via `IDELTACHAT_SCHEME` in `qr.rs`, tests in `qr/qr_tests.rs`. Must stay in sync with the Android `AndroidManifest.xml` deep-link host.

Scan after every merge:
```bash
grep -rn "Delta Chat\|DeltaChat\|delta\.chat\|deltachat\.org\|delta@merlinux\|support\.delta\.chat" \
  src/ --include="*.rs" --exclude-dir="target"
```

---

## Known Branding Fixes (verify after each upstream merge)

| File | Location | Fix |
|------|----------|-----|
| `src/accounts.rs` | "already running" error string | "Delta Chat is already running..." → "Alt Chat is already running..." |
| `src/sql.rs` | DB update failure message | Replace `delta@merlinux.eu` + `support.delta.chat` → `child.aplic@gmail.com` |
| `src/receive_imf.rs` | multi-device warning | "using Delta Chat on multiple devices" → "Alt Chat" |
| `src/imex.rs` | import error + 2 test assertions | "newer version of Delta Chat" → "Alt Chat" |
| `src/webxdc.rs` | webxdc HTML error | "requires a newer Delta Chat version" → "Alt Chat version" |
| `src/qr/dclogin_scheme.rs` | QR code error | "DeltaChat does not understand this QR Code" → "Alt Chat" |
| `src/tools.rs` | truncation-failed message | "bug in the Delta Chat core" → "Alt Chat core" (appeared upstream 2026-08) |
| `src/mimeparser.rs` | "cannot be decrypted" placeholder | "re-installed Delta Chat … re-setup Delta Chat" → "Alt Chat" |
| `src/imap.rs` | "Failed to receive a message" error | contact → `child.aplic@gmail.com`; keep upstream's no-trailing-dot |
| `src/securejoin.rs` | invite link host (3 `format!`) | `https://i.delta.chat/#` → `https://alt-chat.me/#`, keep upstream's `{r_param}` etc. |
| `src/stock_str.rs` | update reminder + donate | `get.delta.chat` → `get.alt-chat.me`, `delta.chat/donate` → `alt-chat.me/donate` |
| `src/webxdc/webxdc_tests.rs` | test assertion | must expect "requires a newer Alt Chat version" |
| `src/receive_imf/receive_imf_tests.rs` | test assertion | must expect "using Alt Chat on multiple devices" |
| `src/contact/contact_tests.rs` | `test_get_contacts` | fork matches `addr` as substring (`alice@` → 1 result, upstream expects 0); keep the fork expectation |

---

## Workflow: Merging Upstream

```bash
# 1. Fetch and review
git fetch upstream
git log HEAD..upstream/main --oneline

# 2. Merge
git merge upstream/main

# 3. Resolve conflicts:
#    - User-facing strings → keep our branding (manually edit)
#    - Logic/bug fixes → take upstream (they fix real bugs)
#    - push.rs: take upstream, then verify NOTIFIERS_PUBLIC_KEY is still ours

# 4. Scan for branding leaks

# 5. Fix any leaked strings, then commit
git add <files>
git merge --continue  # or git commit

# 6. Push
git push origin develop
```

### Known hybrids / things that changed in upstream merges

- **`push.rs`**: upstream 2.57 removed heartbeat push (`subscribe()`, `NotifyState`, `push_state()`); our only customization there was the notifier URL, now gone. What must survive every merge is `NOTIFIERS_PUBLIC_KEY` (ours, see "Do NOT change"). Resolve conflicts by taking upstream, then diff the key block against `origin/develop`.
- **`imap.rs` `register_token`**: upstream 2.58 rewrote it (no `XDELTAPUSH` requirement, `push_token_registered` flag, inline `SETMETADATA "INBOX"`). Our former debug-logging hybrid (`format_setmetadata` + `register_token:` log lines) was dropped in the 2026-09 merge — take upstream here.
- **`chat.rs` mark-as-unread**: upstream added `markfresh_chat()` (marks only the *latest* message fresh). Our older `mark_fresh_chat()` (marked *all* messages) was removed in the 2026-09 merge; `dc_markfresh_chat` FFI and the `markfresh_chat` JSON-RPC now point at upstream's implementation.
- **`src/net/tls.rs` + `src/net/tls/danger.rs`: aws-lc-rs is a deliberate fork divergence.** Upstream introduced it (207c2e6e4, 2026-06-05) then reverted it (61898b478, 2026-07-21); we re-applied it (dd789bfcd). A merge will silently restore `rustls::ClientConfig::builder()` and `crypto::ring::default_provider()` — after every sync verify `builder_with_provider(aws_lc_rs)` in `tls.rs`, the four provider sites in `danger.rs`, and the `aws-lc-rs` feature on `tokio-rustls` in `Cargo.toml`. Rationale: the ring JA3 fingerprint is the documented cause of the June 2026 blocking wave in Russia.
- **Removed upstream APIs to expect** (2.56–2.60): provider-db + `dc_provider_*`, all OAuth, `dc_chat_is_protected`, `dc_chatlist_get_context`, `getPushState`, `list_transports_ex`/`TransportListEntry`, `set_transport_unpublished`. The Android JNI wrapper and Java bindings are pure upstream files — merge upstream android *before* rebuilding the `.so`, otherwise `dc_wrapper.c` fails to compile against the new `deltachat.h`.

---

## After Pushing: Update Submodule Pointers

**Android:**
```bash
git -C /Users/romanvalchuk/Projects/alt-chat-android add jni/deltachat-core-rust
git -C /Users/romanvalchuk/Projects/alt-chat-android commit -m "chore: update rust submodule to latest develop"
git -C /Users/romanvalchuk/Projects/alt-chat-android push origin main
```

**iOS:**
```bash
git -C /Users/romanvalchuk/Projects/alt-chat-ios add deltachat-ios/libraries/deltachat-core-rust
git -C /Users/romanvalchuk/Projects/alt-chat-ios commit -m "chore: update rust submodule to latest develop"
git -C /Users/romanvalchuk/Projects/alt-chat-ios push origin main
```
