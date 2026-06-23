# SYNC.md — Storage spec

## Overview
State lives in `state.json` in this repo, AES-256-GCM encrypted. The browser (index.html) and Claude both read/write it using identical crypto.

## Encryption

- **Key**: PBKDF2-HMAC-SHA256(password = PAT, salt = `hmar-v1-salt-2026`, iterations = 200,000, dklen = 32)
- **Cipher**: AES-256-GCM, random 12-byte IV per write, 128-bit tag appended
- **Envelope**: `{"enc":1,"v":1,"iv":"<base64>","data":"<base64 ciphertext+tag>"}`
- **Plaintext**: `{"a":[...active...],"c":[...completed...]}`

## Task schema

| Field | Type | Notes |
|---|---|---|
| `id` | string | `tactiq-YYYYMMDD-N` or `manual-<ms>` |
| `title` | string | Imperative |
| `description` | string | What "done" looks like |
| `priority` | critical\|high\|medium\|low | |
| `dueDate` | YYYY-MM-DD | |
| `estimatedTime` | string | e.g. "1h 30m" |
| `estimatedMinutes` | number | |
| `assignedBy` | string | Who asked; "Self" if own |
| `people` | string | Comma-separated |
| `meetingTitle` | string | |
| `meetingDate` | string | |
| `meetingContext` | string | Verbatim source quote |
| `status` | "wip" | Shows In-Progress badge |
| `addedAt` | ms | |
| `updatedAt` | ms | |
| `completedAt` | ms | When moved to c |
| `uncompletedAt` | ms | When moved back to a |

## GitHub Contents API

- GET `https://api.github.com/repos/gorilagav/actions/contents/state.json`
- PUT same URL with `{message, content (base64), sha}`
- 409 = stale sha → re-GET and retry
