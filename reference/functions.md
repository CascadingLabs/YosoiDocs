---
title: Functions
description: Function reference for yosoi v0.0.2a19
---

> Generated from yosoi `v0.0.2a19`. Only symbols in `__all__` are listed.

## `File`

`File(trigger: str | None = ..., href: str | None = ..., url: str | None = ..., description: str | None = ..., allowed_types: Iterable[str] | None = ..., max_bytes: int | None = ..., kwargs: Any = {}) -> Any`

## `attr`

`attr(value: str, name: str) -> SelectorEntry`

## `check_policy`

`check_policy(policy: str | CrawlPolicy | Policy | None = ..., seeds: tuple[str, ...] = ...) -> PolicyCheck`

## `claude_sdk`

`claude_sdk(model_name: str = ..., kwargs: Any = {}) -> ModelPolicy`

## `crawl`

`crawl(seeds: str | Sequence[str], contracts: Sequence[type[Contract] | str] | type[Contract] | str | None = ..., limit: int | None = ..., policy: Policy | None = ..., fetcher_type: str | None = ..., persist: bool = ..., progress: bool | None = ..., console: Any | None = ...) -> CrawlRunSummary`

## `css`

`css(value: str) -> SelectorEntry`

## `discover`

`discover() -> SelectorEntry`

## `fingerprint`

`fingerprint(source: object, ax_snapshot: Any = ..., headers: dict[str, str] | None = ..., endpoints: Sequence[str] | None = ...) -> PageFingerprint`

## `global_id`

`global_id(value: str, name: str) -> SelectorEntry`

## `js` <a href="https://github.com/CascadingLabs/Yosoi/blob/a0ea34303d9c9841ea60a2b7a3f427003d55fac0/yosoi/types/field.py#L12" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

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

## `load_urls_from_file` <a href="https://github.com/CascadingLabs/Yosoi/blob/a0ea34303d9c9841ea60a2b7a3f427003d55fac0/yosoi/utils/urls.py#L139" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`load_urls_from_file(filepath: str) -> list[str]`

Load URLs from a file (JSON, plain text, CSV, Excel, Parquet, or Markdown).
**Args:**

- `filepath` `str` — Path to file containing URLs.

**Returns:** `list[str]` — List of URL strings.

**Raises:**

- `FileNotFoundError` — If file does not exist.
- `ValueError` — If file format requires unavailable dependencies.

## `opencode`

`opencode(model_name: str = ..., kwargs: Any = {}) -> ModelPolicy`

## `policy_arn`

`policy_arn(namespace: str, name: str) -> str`

## `regex`

`regex(value: str) -> SelectorEntry`

## `register_coercion` <a href="https://github.com/CascadingLabs/Yosoi/blob/a0ea34303d9c9841ea60a2b7a3f427003d55fac0/yosoi/types/registry.py#L78" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

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

## `resolve_contract` <a href="https://github.com/CascadingLabs/Yosoi/blob/a0ea34303d9c9841ea60a2b7a3f427003d55fac0/yosoi/utils/contracts.py#L134" target="_blank" rel="noopener noreferrer" title="View source on GitHub"><svg aria-hidden="true" height="14" viewBox="0 0 16 16" version="1.1" width="14" xmlns="http://www.w3.org/2000/svg" style="vertical-align:-2px;display:inline-block"><path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/></svg></a>

`resolve_contract(name: str | dict[str, Any] | ContractSpec) -> type[Contract]`

Resolve a contract to a Contract class.

This is the programmatic API. No fuzzy matching or file scanning is
performed — those are CLI-only DX features in ``SchemaParamType``.

Resolution order:
1. ContractSpec / dict → rehydrate via ``ContractSpec.to_contract()``
2. Exact match in BUILTIN_SCHEMAS
3. Case-insensitive match in BUILTIN_SCHEMAS
4. Exact / case-insensitive match in _CONTRACT_REGISTRY (custom schemas)
5. Dynamic import via ``path:ClassName``
**Args:**

- `name` `str | dict[str, Any] | ContractSpec` — Contract name, ``path:ClassName`` string, inline ContractSpec, or dict.

**Returns:** `type[Contract]` — The resolved Contract subclass.

**Raises:**

- `ValueError` — If no matching contract is found.

## `resolve_crawl_policy`

`resolve_crawl_policy(policy: str | CrawlPolicy | Policy | None = ...) -> CrawlPolicy`

## `role`

`role(value: str, name: str, nth: int = ...) -> SelectorEntry`

## `scrape`

`scrape(url: str | Sequence[str], contract: type[Contract] | str | Sequence[type[Contract] | str], model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> list[dict[str, Any]] | dict[str, list[dict[str, Any]]] | dict[str, dict[str, list[dict[str, Any]]]]`

## `scrape_many`

`scrape_many(urls: list[str] | tuple[str, ...], contract: type[Contract] | str, model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> dict[str, list[dict[str, Any]]]`

## `scrape_sync`

`scrape_sync(url: str, contract: type[Contract] | str, model: _YosoiConfig | _LLMConfig | ModelPolicy | str | None = ..., kwargs: Any = {}) -> list[dict[str, Any]]`

## `show`

`show(value: Any, format: Literal['auto', 'table', 'plain', 'json'] = ..., title: str | None = ..., console: Any = ..., fingerprint: object | bool | None = ...) -> None`

## `visual`

`visual(x: float, y: float, value: str = ...) -> SelectorEntry`

## `xpath`

`xpath(value: str) -> SelectorEntry`

