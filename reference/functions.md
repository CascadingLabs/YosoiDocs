---
title: Functions
description: Function reference for yosoi v0.0.3a26
---

> Generated from yosoi `v0.0.3a26`. Only symbols in `__all__` are listed.

## `Extractor` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/types/field.py#L84" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`Extractor(default: Any = PydanticUndefined, default_factory: Callable[[], Any] | None = None, using: Callable[[Any], Any] | str | None = None, key: str | None = None, version: str | None = None, config: Mapping[str, Any] | None = None, kwargs: Any = {}) -> Any`

Declare an async-capable, deterministic extractor field.

The annotation remains the field's real value type. This marker stores only
extraction configuration; it never becomes a model value and never performs
fetching, browser actions, or LLM discovery.

A fluent selector plan, ``using=`` callable, or ``@ys.extraction`` binding is
explicit. Without one, resolution falls back to legacy ``extract_<field>``
methods, output-type ``__yosoi_extract__`` hooks, then an exact registry entry.

## `File`

`File(trigger: str | None = ..., href: str | None = ..., url: str | None = ..., description: str | None = ..., allowed_types: Iterable[str] | None = ..., max_bytes: int | None = ..., kwargs: Any = {}) -> Any`

## `absent`

`absent(target: Any) -> Any`

## `attr`

`attr(value: str, name: str) -> SelectorEntry`

## `check_policy`

`check_policy(policy: str | CrawlPolicy | Policy | None = ..., seeds: tuple[str, ...] = ...) -> PolicyCheck`

## `claude_sdk`

`claude_sdk(model_name: str = ..., kwargs: Any = {}) -> ModelPolicy`

## `click`

`click(target: Any) -> Any`

## `click_all`

`click_all(target: Any, kwargs: Any = {}) -> Any`

## `compile_contract` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L184" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`compile_contract(source: ContractInput | str | Path) -> ContractSpec`

Compile a Python/authored contract source into canonical ``ContractSpec`` JSON.

## `count`

`count(target: Any, kwargs: Any = {}) -> Any`

## `crawl`

`crawl(seeds: str | Sequence[str], contracts: Sequence[type[Contract] | str] | type[Contract] | str | None = ..., limit: int | None = ..., policy: Policy | None = ..., fetcher_type: str | None = ..., persist: bool = ..., progress: bool | None = ..., console: Any | None = ...) -> CrawlRunSummary`

## `css`

`css(value: str) -> SelectorEntry`

## `discover`

`discover() -> SelectorEntry`

## `dom_stable`

`dom_stable(kwargs: Any = {}) -> Any`

## `execute_content`

`execute_content(request: ContentRequest) -> ContentResult`

## `execute_fetch`

`execute_fetch(request: FetchRequest) -> FetchResult`

## `export_a3nodes` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L654" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`export_a3nodes(domains: Sequence[str] = (), source_urls: Sequence[str] = ()) -> list[RecipeA3Node]`

Export local scoped A3Node cache entries as portable recipe nodes.

## `export_a3nodes_sync` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L675" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`export_a3nodes_sync(domains: Sequence[str] = (), source_urls: Sequence[str] = ()) -> list[RecipeA3Node]`

Synchronous wrapper for :func:`export_a3nodes`.

## `extract`

`extract(html: str, contract: type[Contract] | str, url: str = ..., selectors: Mapping[str, Mapping[str, Any]] | None = ..., root: SelectorEntry | Mapping[str, Any] | str | None = ..., runtime_evidence: Mapping[str, Sequence[str]] | None = ..., policy: Policy | None = ..., fingerprint_store: _FingerprintStore | None = ...) -> list[dict[str, Any]]`

## `fetch`

`fetch(url: str | Sequence[str], view: str = ..., fetcher_type: str | None = ..., page: int = ..., page_size: int = ..., chars: int | None = ..., include: Sequence[str] = ..., contracts: Any = ..., output_dir: str | None = ..., policy: Policy | None = ...) -> FetchResult`

## `fingerprint`

`fingerprint(source: object, ax_snapshot: Any = ..., headers: dict[str, str] | None = ..., endpoints: Sequence[str] | None = ...) -> PageFingerprint`

## `global_id`

`global_id(value: str, name: str) -> SelectorEntry`

## `input`

`input(name: str) -> Any`

## `install_recipe` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L384" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`install_recipe(source: str, recipe_id: str | None = None, cache_dir: str | Path | None = None, trust: RecipeTrust | None = None, policy: Policy | None = None) -> RecipeInstallResult`

Backward-compatible alias for :func:`install`.

## `js` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/types/field.py#L158" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`js(script: str | None = None, description: str | None = None, kwargs: Any = {}) -> Any`

Declare a contract field extracted by a JS program run in the live browser tab.

Two modes:

**Hand-authored** — provide ``script``. The expression is evaluated as-is
on every fetch. No LLM involved::

    signals: dict = ys.js("(() => ({ has_alita: !!window.__alita__ }))()")

**Discovery-driven** — omit ``script``, provide ``description``. Yosoi's
:class:`JsDiscoveryOrchestrator` writes and verifies the script once per
domain, then caches it (CAS-92)::

    signals: dict = ys.js(description="Detect Alita embed and competitor widgets")
**Args:**

- `script` `str | None` — JavaScript IIFE to evaluate. ``None`` triggers JS discovery.
- `description` `str | None` — Human-readable description used by the LLM during discovery.
Required when ``script`` is ``None``.
- `**kwargs` `Any` — Additional arguments forwarded to ``pydantic.Field``
(e.g. ``default``, ``description`` as a pydantic field description).

**Returns:** `Any` — A pydantic FieldInfo with ``yosoi_action`` metadata.

**Raises:**

- `ValueError` — When neither ``script`` nor ``description`` is provided.

## `jsonld`

`jsonld(value: str) -> SelectorEntry`

## `load_recipe` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L342" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`load_recipe(source: str, recipe_id: str | None = None, trust: RecipeTrust | None = None, policy: Policy | None = None) -> Recipe`

Backward-compatible alias for :func:`load`.

## `load_urls_from_file` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/utils/urls.py#L139" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`load_urls_from_file(filepath: str) -> list[str]`

Load URLs from a file (JSON, plain text, CSV, Excel, Parquet, or Markdown).
**Args:**

- `filepath` `str` — Path to file containing URLs.

**Returns:** `list[str]` — List of URL strings.

**Raises:**

- `FileNotFoundError` — If file does not exist.
- `ValueError` — If file format requires unavailable dependencies.

## `map`

`map(url: str, max_sitemaps: int = ..., max_urls: int = ..., max_subdomains: int = ..., subfinder_bin: str = ..., subfinder_timeout: int = ..., include_robots: bool = ..., include_default_sitemaps: bool = ..., include_subdomains: bool = ..., discover_subdomains: bool = ...) -> MapResult`

## `matches`

`matches(pattern: str) -> str`

## `mint_recipe` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L297" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`mint_recipe(contract: ContractInput, selectors: SelectorInput, out: str | Path | None = None, a3nodes: Sequence[Mapping[str, Any]] = (), validation: RecipeValidation | Mapping[str, Any] | None = None, name: str | None = None, domain_scope: Sequence[str] = (), source_urls: Sequence[str] = (), url_patterns: Sequence[str] = (), notes: str | None = None) -> Recipe`

Backward-compatible alias for :func:`mint`.

## `nearest_scroll_parent`

`nearest_scroll_parent(anchor: Any) -> Any`

## `opencode`

`opencode(model_name: str = ..., kwargs: Any = {}) -> ModelPolicy`

## `policy_arn`

`policy_arn(namespace: str, name: str) -> str`

## `regex`

`regex(value: str) -> SelectorEntry`

## `register_coercion` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/types/registry.py#L78" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`register_coercion(type_name: str, description: str = '', semantic: SemanticRule | None = None, config_defaults: Any = {}) -> Callable[[Callable[..., CoercedValue]], Callable[..., Any]]`

Decorator that registers a coercion function and returns a Field factory.

The decorated function becomes the Field factory — its name is what you use
in a Contract.  The coercion logic is stored internally in the registry.

Decorator kwargs define the config schema:
- ``description``: default field description
- all other kwargs: config keys that appear in ``json_schema_extra`` and are
  forwarded to the coerce function via ``config``
**Args:**

- `type_name` `str` — The ``yosoi_type`` identifier (e.g. ``'price'``).
- `description` `str` — Default field description shown in manifests and to the AI.
- `semantic` `SemanticRule | None` — Optional :class:`SemanticRule` describing the shape a correct
value should have. Used by the discovery semantic-retry loop.
- `**config_defaults` `Any` — Config keys with their default values. These become
keyword arguments on the generated factory function.

Example::

    @register_coercion('phone', description='A phone number', country_code='+1')
    def PhoneNumber(v, config, source_url=None):
        import re
        digits = re.sub(r'\D', '', str(v))
        return config.get('country_code', '+1') + digits

    # PhoneNumber is now a Field factory:
    # PhoneNumber(country_code='+44') -> Field(json_schema_extra={...})

## `register_extractor` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/models/extraction.py#L619" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`register_extractor(target: Any, using: Callable[[ExtractionRow], Any] | None = None, key: str | None = None, version: str | None = None, config: Mapping[str, Any] | None = None) -> Any`

Register an exact deterministic extractor for an annotation or semantic type.

This supports direct and decorator forms. String targets name a Yosoi semantic
type; all other targets are normalized as complete Python annotations.

## `render_contract_py` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L197" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`render_contract_py(source: Recipe | ContractSpec | ContractInput | str | Path) -> str`

Render a canonical contract spec as importable, reviewable Python.

## `resolve_contract` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/utils/contracts.py#L133" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`resolve_contract(name: str | dict[str, Any] | ContractSpec) -> type[Contract]`

Resolve a contract to a Contract class.

This is the programmatic API. No fuzzy matching or file scanning is
performed — those are CLI-only DX features in ``SchemaParamType``.

Resolution order:
1. ContractSpec / dict → rehydrate via ``ContractSpec.to_contract()``
2. Exact match in BUILTIN_SCHEMAS
3. Exact match in _CONTRACT_REGISTRY (custom schemas)
4. Dynamic import via ``path:ClassName``
**Args:**

- `name` `str | dict[str, Any] | ContractSpec` — Contract name, ``path:ClassName`` string, inline ContractSpec, or dict.

**Returns:** `type[Contract]` — The resolved Contract subclass.

**Raises:**

- `ValueError` — If no matching contract is found.

## `resolve_crawl_policy`

`resolve_crawl_policy(policy: str | CrawlPolicy | Policy | None = ...) -> CrawlPolicy`

## `role`

`role(value: str, name: str, nth: int = ...) -> SelectorEntry`

## `run_content`

`run_content(request: ContentRequest) -> ContentResult`

## `run_fetch`

`run_fetch(request: FetchRequest) -> FetchResult`

## `scrape`

`scrape(url: str | Sequence[str], contract: type[Contract] | str | Sequence[type[Contract] | str], model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> ScrapeResult`

## `scrape_many`

`scrape_many(urls: list[str] | tuple[str, ...], contract: type[Contract] | str, model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> ScrapeResult`

## `scrape_sync`

`scrape_sync(url: str, contract: type[Contract] | str, model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> ScrapeResult`

## `scroll_until`

`scroll_until(container: Any, kwargs: Any = {}) -> Any`

## `search`

`search(query: str | Sequence[str], kind: str | None = ..., provider: str | None = ..., backend: str | None = ..., region: str | None = ..., safesearch: str | None = ..., timelimit: str | None = ..., max_results: int | None = ..., limit: int | None = ..., page: int | None = ..., policy: Policy | None = ..., max_concurrency: int = ...) -> SearchResult | SearchBatchResult`

## `selector_map` <a href="https://github.com/CascadingLabs/Yosoi/blob/v0.0.3a26/yosoi/recipe.py#L222" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`selector_map(url: str, domain: str | None = None, discovered_at: datetime | None = None, source: str = 'discovered', fields: SelectorValue = {}) -> SnapshotMap`

Build a portable one-domain selector map from field CSS/XPath entries.
**Args:**

- `url` `str` — Source URL these selectors were minted from.
- `domain` `str | None` — Optional domain override. Defaults to the URL host.
- `discovered_at` `datetime | None` — Optional stable timestamp. Defaults to now.
- `source` `str` — Selector provenance label accepted by :class:`SelectorSnapshot`.
- `**fields` `SelectorValue` — Field name to selector payload. Strings become primary selectors;
dict values are kept as structured selector payloads.

## `show`

`show(value: Any, format: Literal['auto', 'table', 'plain', 'json'] = ..., title: str | None = ..., console: Any = ..., fingerprint: object | bool | None = ...) -> None`

## `visual`

`visual(x: float, y: float, value: str = ...) -> SelectorEntry`

## `wait_until`

`wait_until(kwargs: Any = {}) -> Any`

## `xpath`

`xpath(value: str) -> SelectorEntry`

