---
layout: default
---

# Blog

## Research

### [Unlocking the Computation Inside Pretrained Models](/research/shaping-gap-sustainable-ai/)

**November 13, 2026** · Shaping Gap Benchmark

What becomes possible when pretrained neural computation gets a different
execution system? Explore acquired execution over frozen weights, compatibility
for existing programs, and the commercial research path toward more sustainable AI.
Includes measured results, the paper and reproducible code.

[Read the article](/research/shaping-gap-sustainable-ai/) · [Paper and code](https://github.com/calysteon/shaping-gap-benchmark/tree/main/checkpoints/04-experience-acquired-reuse)

## Security research

| Description | Post | Did I Report This? |
|---|---|---|
| Windows NTFS integer underflow enabling kernel privilege escalation (reverse-engineering writeup) | [CVE-2021-31956](cve/CVE-2021-31956.md) | No |
| Stack exhaustion in Libc `posix_spawnp()` via an oversized `PATH` | [CVE-2025-43295](cve/CVE-2025-43295.md) | Yes |
| Stack buffer overflow in Libc `setsourcefilter()` via an unvalidated `ss_len` | [CVE-2025-43299](cve/CVE-2025-43299.md) | Yes |
| Heap buffer overflow in Libinfo `clnt_sperror()` | [CVE-2025-43353](cve/CVE-2025-43353.md) | Yes |
| Static path buffer overflow in rpcgen's `-Y` option | [CVE-2025-43370](cve/CVE-2025-43370.md) | Yes |
| Stack buffer overflow in rpcgen's `get_prog_declaration()` | [CVE-2025-43375](cve/CVE-2025-43375.md) | Yes |
| Remote pre-authentication global buffer overflow in LLDB's `debugserver` | [CVE-2025-43504](cve/CVE-2025-43504.md) | Yes |
| Signed integer overflow to heap corruption in gm4's frozen state loader | [CVE-2025-43505](cve/CVE-2025-43505.md) | Yes |
| Out-of-bounds heap read in Safari's Web Audio resampler | [CVE-2026-20652](cve/CVE-2026-20652.md) | Yes |
| macOS TCC bypass via custom file-extension ownership | [CVE-2026-28845](cve/CVE-2026-28845.md) | Yes |
| TOCTOU race in JavaScriptCore's TypedArray `toReversed()` and `toSorted()` | [CVE-2026-28857](cve/CVE-2026-28857.md) | Yes |
| Out-of-bounds read in otool via a crafted universal binary | [CVE-2026-28890](cve/CVE-2026-28890.md) | Yes |
| TOCTOU heap buffer overflow in JavaScriptCore's `TypedArray.prototype.with()` | [CVE-2026-28902](cve/CVE-2026-28902.md) | Yes |
| XNU kernel heap information disclosure via uninitialized CMSG padding | [CVE-2026-43654](cve/CVE-2026-43654.md) | Yes |

---
