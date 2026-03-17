---
title: 'Found Vulnerabilities'
date: 2026-01-10T15:14:39+10:00
weight: 1
---

Security research and bug bounty findings.
<!--more-->

## Security Research
 
- [Stack Overflow in FreeType](https://gitlab.freedesktop.org/freetype/freetype/-/issues/1395) - Tab crash in Mozilla Firefox and Chromium
- [Signed int overflow in dr_libs MS-ADPCM decoder](https://github.com/mackron/dr_libs/issues/295) - Potential DoS in applications using Qt Multimedia for audio processing
- [CVE-2025-12097](https://nvd.nist.gov/vuln/detail/CVE-2025-12097) - LFI in NI System Web Server
- [Hacking via eSCL protocol](https://writeups.csirt.muni.cz/content/blog/escl/) - Article on the eSCL protocol and its security implications.

## Bug Bounty and Vulnerability Disclosure

The table below lists vulnerabilities that were found in my free time and reported as part of the bug bounty or vulnerability disclosure program. All have been addressed and resolved by the respective teams.

| Date       | Name            | Org         |  Info |
|------------|-----------------|-------------|-------|
| 2026-02    | Denial of Service via Cookie Bombing | seznam.cz | [Seznam HoF](https://www.seznam.cz/.well-known/security-hall-of-fame.html) |
| 2026-01    |  Open Redirect  |   seznam.cz   | Bounty reward |
| 2025-12    |  rXSS to partial account takeover  |   undisclosed   | Bounty reward |
| 2025-12    |  sXSS (Old browsers only) | seznam.cz  | [Seznam HoF](https://www.seznam.cz/.well-known/security-hall-of-fame.html) |
| 2025-12   |  rXSS           | hrad.cz    |  |
| 2025-11 |  CVE-2025-12097  | Many big universities | [Utwente HoF](https://www.utwente.nl/en/cyber-safety/responsible/hall-of-fame/) |
