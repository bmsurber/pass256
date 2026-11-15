# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Pass256 is a tiny, fully client-side, stateless password generator with no dependencies. There is no build step, package manager, linter, or test suite. To run it, open `index.html` in a browser (or serve the directory statically, e.g. `python3 -m http.server`). It uses the browser's Web Crypto API (`crypto.subtle`), which is only available in a secure context: HTTPS, `localhost`, or a local `file://` page — not plain `http://`.

## Architecture

- `index.html` is the whole app: markup, CSS, and the inline `hmacSha256Base64()` and `pass256()` functions.
- Older versions used a vendored SJCL build (`sjcl.js` plus an `sjcl/` submodule); it was replaced by Web Crypto with byte-identical output.

### Derivation algorithm (`hmacSha256Base64()` / `pass256()`)

The password is `base64( HMAC-SHA256(key = UTF-8(Master Key), message = UTF-8(Account)) )` (standard base64 alphabet), truncated to `Length` characters (default 32, capped at 42, since base64 of 32 bytes is 44 chars). Output is deterministic: the same Master Key + Account + Length always yields the same password, so nothing is stored. Changing the algorithm, encoding, or truncation silently changes every user's generated passwords — treat it as a compatibility-breaking change.

Web Crypto rejects zero-length HMAC keys, so an empty Master Key is replaced by 64 zero bytes. This is mathematically identical (HMAC zero-pads keys to the 64-byte block size) and keeps empty-key output matching the original SJCL implementation; don't remove it.

`pass256()` is `async` because Web Crypto is Promise-based.

### UI behavior to preserve

- Enter in any input triggers `pass256()`; the form has no real submit.
- Master Key and the output field are `type=password`; the output field switches to `text` on focus and back to `password` on blur (and is reset to `password` after each generation). Click selects all for copying.
- `autocomplete="new-password"` and the autocomplete/autocorrect/spellcheck-off attributes are intentional to keep browsers from saving or suggesting secrets.
