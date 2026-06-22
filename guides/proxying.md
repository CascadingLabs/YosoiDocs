---
title: Proxying
description: Current proxy guidance and planned proxy configuration for Yosoi.
faqs:
  - q: "Does Yosoi have a proxy flag today?"
    a: "No. Use environment or browser-runtime proxy configuration today. A first-class Yosoi proxy policy is planned."
  - q: "Should I use public proxy lists?"
    a: "No. Public proxies are unreliable, often abusive to shared infrastructure, and can create legal or operational risk. Use dedicated providers that explicitly allow your use case."
---

Yosoi does not expose a first-class proxy policy yet. Today, configure proxies at the HTTP, browser, or deployment layer you already control, then pass the resulting HTML or fetcher behavior into Yosoi. First-class proxy settings are tracked on the [Roadmap](/roadmap/).

## Current Support

- Simple HTTP fetches can inherit environment-level proxy settings supported by the underlying HTTP client.
- Browser-backed fetches should be configured where the browser runtime is launched.
- Yosoi selector discovery, validation, and extraction work the same way after the page has been fetched.

## Planned Scope

The planned proxy feature should make proxy routing explicit in policy/config, preserve provenance in debug output, and avoid hiding network behavior behind selector logic.

## A Note on Responsible Use

Yosoi is designed for respectful scraping. When routing requests through proxies, please do not use TOR<sup>[△](#ref-1)</sup> exit nodes or public SOCKS5<sup>[○](#ref-2)</sup> servers. These are shared public IP systems; using them for scraping loads volunteer-run infrastructure with automated traffic it was not designed to carry, and it degrades the experience for everyone who relies on those networks for legitimate privacy use.

If you need to route requests through multiple IPs, use dedicated residential or datacenter proxies from providers that explicitly offer them for this purpose.

## FAQs

<details>
<summary>Does Yosoi have a proxy flag today?</summary>

No. Use environment or browser-runtime proxy configuration today. A first-class Yosoi proxy policy is planned.

</details>

<details>
<summary>Should I use public proxy lists?</summary>

No. Public proxies are unreliable, often abusive to shared infrastructure, and can create legal or operational risk. Use dedicated providers that explicitly allow your use case.

</details>

## References

<a id="ref-1"></a>△ **Tor Project**. The Tor Project. *Free, open-source software enabling anonymous communication online.* https://www.torproject.org/

<a id="ref-2"></a>○ **SOCKS5**. IETF RFC 1928. *Protocol for routing network packets between a client and server through a proxy.* https://datatracker.ietf.org/doc/html/rfc1928
