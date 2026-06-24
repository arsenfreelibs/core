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

Scan after every merge:
```bash
grep -rn "Delta Chat\|DeltaChat\|delta\.chat\|deltachat\.org\|delta@merlinux\|support\.delta\.chat" \
  src/ --include="*.rs"
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
#    - imap.rs SETMETADATA: use upstream's hardcoded "INBOX" + keep our debug log lines

# 4. Scan for branding leaks

# 5. Fix any leaked strings, then:
git add -p  # or git add <files>
git merge --continue  # or git commit

# 6. Push
git push origin develop
```

### imap.rs SETMETADATA — known hybrid (preserve after merges):
```rust
let setmetadata_cmd = format_setmetadata("INBOX", &encrypted_device_token);
info!(context, "register_token: sending SETMETADATA to folder=INBOX");
info!(context, "register_token: encrypted_device_token={}", encrypted_device_token);
info!(context, "register_token: SETMETADATA command={}", setmetadata_cmd);
self.run_command_and_check_ok(&setmetadata_cmd)
```
The `"INBOX"` is intentionally hardcoded (upstream bug fix — dynamic folder lookup was broken for unconfigured IMAP). Our debug log lines must be preserved.

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
