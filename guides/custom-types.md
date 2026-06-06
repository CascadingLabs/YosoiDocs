---
title: Custom Types
description: Extend Yosoi with your own semantic field types.
---

When built-in types don't cover your domain, register a custom type. Once registered, it works exactly like a built-in: the AI sees it in the manifest, and the coercion runs automatically during extraction.

## Register a Coercion

One function, zero boilerplate. The decorator registers the coerce logic and replaces the name with a `Field` factory.

```python
import re
from typing import Any
import yosoi as ys

@ys.register_coercion('phone', description='A phone number', country_code='+1')
def PhoneNumber(v: object, config: dict[str, Any], source_url: str | None = None) -> str:
    """Strip formatting and prepend a country code."""
    raw = str(v).strip()
    digits = re.sub(r'\D', '', raw)
    if not digits:
        raise ValueError(f'No digits found in phone value: {v!r}')
    cc = config.get('country_code', '+1')
    cc_digits = re.sub(r'\D', '', cc)
    if digits.startswith(cc_digits):
        return f'+{digits}'
    return f'{cc}{digits}'
```

Decorator kwargs become the factory's parameter schema:

```python
# Default country code
us_phone: str = PhoneNumber()

# Override per field
uk_phone: str = PhoneNumber(country_code='+44')
```

## Using Custom Types in a Contract

Registered coercions compose naturally with built-ins:

```python
import yosoi as ys

class ContactPage(ys.Contract):
    name: str = ys.Title()
    us_phone: str = PhoneNumber()
    uk_phone: str = PhoneNumber(country_code='+44')
    website: str = ys.Url()
```

Custom types appear in `Contract.generate_manifest()`, so the AI knows about them during selector discovery.

## FAQs

<details>
<summary>Which pattern should I use?</summary>

Use `@register_coercion`. It is the supported custom-type path.

</details>

<details>
<summary>Can custom types be used with list[T]?</summary>

Yes. Custom coercions run element-by-element on list fields, the same as built-in types.

</details>

<details>
<summary>How do I debug a coercion failure?</summary>

Raise a `ValueError` with a descriptive message inside your coerce function. Pydantic<sup>[△](#ref-1)</sup> will surface it as a `ValidationError` with field context. Run with `--debug` to inspect the raw extracted values before coercion.

</details>

## References

<a id="ref-1"></a>△ **Pydantic**. Pydantic Services Inc. *Data validation library for Python.* https://docs.pydantic.dev/
