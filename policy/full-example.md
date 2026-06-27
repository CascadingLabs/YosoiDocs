---
title: Full Policy Example
description: A complete YAML policy showing every major Yosoi policy namespace.
---

This is intentionally verbose. In real projects, split policy across `.yosoi/policy/` fragments and only set fields you mean to control.

```yaml
atom_reads: false
trust_tier: strict

model:
  provider: groq
  model_name: llama-3.3-70b-versatile
  temperature: 0.01
  max_tokens: null
  extra_params: null
  credential_ref:
    source: env
    name: GROQ_KEY

scrape:
  force: false
  skip_verification: false
  fetcher_type: auto
  selector_level: 5
  max_concurrency: null
  cross_origin_dom: false

discovery:
  max_concurrent: 5
  mode: auto
  lesson_cache: true
  replay_verify_threshold: 1.0
  static_mode_warning: true

telemetry:
  langfuse_public_key_ref:
    source: env
    name: LANGFUSE_PUBLIC_KEY
  langfuse_secret_key_ref:
    source: env
    name: LANGFUSE_SECRET_KEY
  langfuse_host: https://cloud.langfuse.com

output:
  formats: [json]
  quiet: true
  json_output: false
  plain_output: false
  debug_html: false
  debug_html_dir: .yosoi/debug
  flat_files: false
  logs: true

download:
  allow: false
  allowed_types: []
  directory: null
  max_bytes: null
  keep: true

page:
  fetcher_type: auto
  timeout_seconds: 30.0
  max_fetch_retries: 1
  allow_redirects: true
  clean_html: true
  cleaner_profile: discovery
  chrome_ws_urls: []

crawl:
  mode: contract_focus
  budget:
    max_pages: 25
    max_depth: 2
    max_attempts: null
    max_pages_per_host: null
    crawl_session_id: null
  scheduler:
    max_workers: 3
    per_host_concurrency: 1
    politeness_delay: 1.0
    fetch_timeout_seconds: 15.0
    max_fetch_retries: 2
  safety:
    respect_robots: true
    allow_redirects: false
    allow_cross_domain: false
    allowed_hosts: []
    denied_hosts: []
    blocked_path_prefixes: []
  escalation:
    allow_model_discovery: false
    allow_paid_scrapers: false
    max_llm_calls: 0
    max_paid_scraper_calls: 0
  target_contracts: []
  scrape_contracts: false
  scrape_url_limit_per_contract: 1
  fetcher_type: auto

fingerprint:
  signal_lane: true
  backpressure: defer
  max_queue: 256
```

Validate it:

```bash
uvx yosoi policy validate .yosoi/policy.yaml
uvx yosoi policy effective --format yaml
```
