---
layout: default
---

# Blog

| Date | Post |
|---|---|
| **2026-03-06** | [Activation-Oriented Programming: Applying Binary Exploitation Intuition to AI Red Teaming](blog/aop) |
| **2026-03-17** | [Same Playbook, Different Context: Observing Claude Opus 4.6 Converge on Identical Browser Exploit Techniques Across Independent Research](blog/parallel) |

---

# CVE List
| CVE | Vendor | CWE | Reference | Writeup |
|---|---|---|---|---|
| **CVE-2026-28890** | Apple | TBD | TBD | - | 
| **CVE-2026-28857** | Apple | TBD | TBD | - | 
| **CVE-2026-27820** | Ruby | **CWE-122** (Heap-based Buffer Overflow) | [ruby-lang.org](https://www.ruby-lang.org/en/news/2026/03/05/buffer-overflow-zlib-cve-2026-27820/) | - |
| **CVE-2026-20652** | Apple | **CWE-191** (Integer Underflow) | [126354](https://support.apple.com/en-us/126354) | - |
| **CVE-2025-43505** | Apple | **CWE-787** (Out-of-Bounds Write / Heap Corruption) | [125641](https://support.apple.com/en-us/125641) | - |
| **CVE-2025-43504** | Apple | **CWE-121** (Stack-based Buffer Overflow) | [125641](https://support.apple.com/en-us/125641) | [True](https://objective-see.org/blog/blog_0x83.html) |
| **CVE-2025-43375** | Apple | **CWE-20** (Improper Input Validation) | [125117](https://support.apple.com/en-us/125117) | — |
| **CVE-2025-43370** | Apple | **CWE-20** (Improper Input Validation) | [125117](https://support.apple.com/en-us/125117) | — |
| **CVE-2025-43299** | Apple | **CWE-20** (Improper Input Validation) | [125109](https://support.apple.com/en-us/125109), [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | — |
| **CVE-2025-43295** | Apple | **CWE-20** (Improper Input Validation) | [125109](https://support.apple.com/en-us/125109), [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | — |
| **CVE-2025-43353** | Apple | **CWE-787** (Out-of-Bounds Write / Heap Corruption) | [125110](https://support.apple.com/en-us/125110), [125111](https://support.apple.com/en-us/125111), [125112](https://support.apple.com/en-us/125112) | [True](https://calysteon.github.io/cve/CVE-2025-43353.html) |
| **CVE-2025-53623** | Shopify | **CWE-78** (OS Command Injection) | — | — |
| **CVE-2025-43577** | Adobe | **CWE-416** (Use-After-Free) | — | — |
| **CVE-2024-13334** | WordPress | **CWE-79** (Reflected XSS) | — | — |
| **CVE-2024-10813** | WordPress | **CWE-200** (Information Exposure) | — | — |
| **CVE-2024-10792** | WordPress | **CWE-79** (Reflected XSS) | — | — |
| **CVE-2024-0848** | WordPress | **CWE-79** (Reflected XSS) | — | — |
| **CVE-2024-0847** | WordPress | **CWE-352** (CSRF) | — | — |
| **CVE-2024-1780** | WordPress | **CWE-79** (Reflected XSS) | — | — |
| **CVE-2024-1782** | WordPress | **CWE-79** (Reflected XSS) | — | — |
| **CVE-2024-0708** | WordPress | **CWE-200** (Information Exposure) | — | — |
| **CVE-2024-0859** | WordPress | **CWE-352** (CSRF) | — | — |

---

# Acknowledgements
| Vendor | Platform / Release | Component(s) | Reference |
|---|---|---|---|
| Apple | macOS Tahoe 26.2 | **FileVault** | [125886](https://support.apple.com/en-us/125886) | 
| Apple | iOS / iPadOS 26 | **darwinOS**, **libc**, **libpthread**, **libxml2** | [125108](https://support.apple.com/en-us/125108) |
| Apple | iOS / iPadOS 18.7 | **libpthread**, **libxml2** | [125109](https://support.apple.com/en-us/125109) |
| Apple | macOS Tahoe 26 | **AMD**, **Core Bluetooth**, **CoreMedia** , **darwinOS**, **libc**, **libedit**, **libpthread**, **libxml2** | [125110](https://support.apple.com/en-us/125110) |
| Apple | macOS Sequoia 15.7 | **libpthread**, **libxml2** | [125111](https://support.apple.com/en-us/125111) |
| Apple | macOS Sonoma 14.8 | **libpthread**, **libxml2** | [125112](https://support.apple.com/en-us/125112) |
| Apple | tvOS 26 | **darwinOS**, **libc**, **libpthread**, **libxml2** | [125114](https://support.apple.com/en-us/125114) |
| Apple | visionOS 26 | **darwinOS** | [125115](https://support.apple.com/en-us/125115) |
| Apple | watchOS 26 | **darwinOS**, **libc**, **libpthread**, **libxml2** | [125116](https://support.apple.com/en-us/125116) |
