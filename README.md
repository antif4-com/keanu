

# Commonalities

## Versioning

- Versioning is vX.Y. 
- X changes are big, breaking. 
- Y is smoother, with transitions managed. 
- Endpoints are individually versioned (e.g. /v3/sync and /v4/profile)

## Deprecation

- Functionality begins as `stable`
- Functionality will remain in a `deprecated` state for usually 1 version before being removed.
- `deprecated` functionality MUST be implimented to support that version

## Base Assumptions

- JSON over HTTP/S 

### Pagination

- `next_batch` is used to indicate when additional results are available.
- `next_batch` should be a string token which can be passed into subsequent calls. 
- Each subsequent call will return the next batch and follow the same logic. 
- *Omitting* the `next_batch` property is an indication that all results have been returned.
- When requesting the next batch, `from` MUST be used as the input parameter. 
- If two directions are required, a `prev_batch` field can be used in addition to `next_batch`

### Binary Data

- **All binary values should be encoded and represented in JSON as unpadded base64**
- "Unpadded base64" means [RFC 4648](https://tools.ietf.org/html/rfc4648) without "=" padding.
- When decoding base64, accept input with AND without padding. 
- **A homeserver MAY check for correctness of a base64 encoded value at any point**
- In the future, if an alternative to JSON is used, where binary encoding is not necessary, then base64 encoding MAY be skipped altogether. 

### Canonical JSON 

- Shortest UTF-8 JSON encoding with dictionary keys lexicographically sorted by Unicode codepoint. 
- Numbers MUST be integers in the range `[-(2**53)+1, (2**53)-1]` without exponents or decimal places. 
- `-0` MUST NOT appear. 

```
import json

def canonical_json(value):
    return json.dumps(
        value,
        # Encode code-points outside of ASCII as UTF-8 rather than \u escapes
        ensure_ascii=False,
        # Remove unnecessary white space.
        separators=(',',':'),
        # Sort the keys of dictionaries.
        sort_keys=True,
        # Encode the resulting Unicode as UTF-8 bytes.
    ).encode("UTF-8")
```

# Links

- [Conventions for Matrix APIs](https://spec.matrix.org/v1.14/appendices/#conventions-for-matrix-apis)


# keanu

- components for JSON, base64 (options to verify)
- JSON is pluggable, looking towards being able to use something else in the future
- base64 encoding is tied to JSON usage (can be avoided if JSON isn't used)

## Settings

- settings aroudn how often/close to validate base64, basically slider from EVERYWHERE to NOWHERE. 
- system to handle per API deprecation/status
- API system needs to manage multiple versions of same endpoint running side by side
