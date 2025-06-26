**These notes are from version [1.14](https://spec.matrix.org/v1.14/) of the Matrix Specification. If no version is specified, then the content comes from that version of the matrix spec. **

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

# [Client-Server APIs](https://spec.matrix.org/v1.14/client-server-api/)

- Baseline for all client-server communication is exchanging JSON objects over HTTP APIs. 
- HTTPS is RECOMMENDED. 
- Clients are authenticated using opaque `access_token` strings
- All server responses MUST include a `Content-Type` of `application/json` and include at least empty JSON body for 200 responses

## [Error Responses](https://spec.matrix.org/v1.14/client-server-api/#standard-error-response)

- All API errors MUST return an error JSON object: 
```
{
  "errcode": "<error code>",
  "error": "<error message>"
}
```

- `error` is a human readable string, a sentence
- `errcode` is a unique string representing the error code. 
- `errcode` is namespaced with the _ seperator. `M_` is the matrix standard errors.
- Some errors have additional keys which SHOULD be present. But `error` and `errcode` are the required.
- In general, ignore the HTTP status code unless the error is `M_UNKNOWN` in which case, look towards the HTTP status code for guidance. 

### [Standard Error Codes](https://spec.matrix.org/v1.14/client-server-api/#common-error-codes)

Any API endpoint can return these codes: 

- `M_FORBIDDEN`: Forbidden access. 
- `M_UNKNOWN_TOKEN`: Either the access or refresh token was not recognized.
- `M_MISSING_TOKEN`: No access toekn was specified in the request. 
- `M_USER_LOCKED`: The account has been locked and cannot be used at this time. 
- `M_USER_SUSPENDED`: The account has been suspended and can only be used by limited actions at this time. 
- `M_BAD_JSON`: Request contained valid JSON, but it was malformed in some way. 
- `M_NOT_JSON`: Request did not contain valid JSON. 
- `M_NOT_FOUND`: No resource was found for this request. 
- `M_LIMIT_EXCEEDED`: Too many requests. 
- `M_UNRECOGNIZED`: Expected to be coupled with a `404` HTTP status code if the endpoint is not implemented or a `405` HTTP status code if the endpoint is implemented but the incorrect HTTP method was used. 
- `M_UNKNOWN`: Unknown. 

### Other error codes

There are other error codes unique to different endpoints. For now, I am not listing them in my notes. They can be viewed here: [https://spec.matrix.org/v1.14/client-server-api/#other-error-codes](https://spec.matrix.org/v1.14/client-server-api/#other-error-codes). 

# Server-Server APIs

# Application Service APIs

# Identity Service APIs

# Push Gateway APIs

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
