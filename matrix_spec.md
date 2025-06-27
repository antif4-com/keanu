# keanu

keanu is a new, theoretical matrix homeserver implementation. It is only vaporware at the moment. 

Design goals:
- minimal effort zero to running server
    - This isn't just about new users. It's about resilience and durability, even complex installations of this server should be simple and easy to deploy and update.
    - config should be basic and minimal, keys, config file, and a dream. 
- minimal external dependencies
  - If possible, keep dependencies to small, code packages. 
- no cloud
- only as fancy as required, and not an ounce more
  - being boring is a good thing in our book

Until we learn it is not a good fit, the default technological approach will be to follow the best practices of the general ruby web development community. The goal is that while a standard ruby/rails dev might not understand the code and spec, they should instantly understand the project. 

## Details

### Versioning

keanu will use [semantic versioning](https://semver.org). 

### Project Tracking

[keanu project on github](https://github.com/orgs/antif4-com/projects/7)

### Roadmap

We will build keanu in a series of stages. These stages will become greater in number and detail as we progress in the project: 

- 0.0.1 : bare bones simple authenticated response
- 0.1.0 : Non-federated clear-text chat with username/password authN
- 0.2.0-alpha : Non-federated E2EE chat with username/password authN
- 0.3.0 : Non-federated E2EE chat with oauth authN
- 0.4.0-beta : Non-functionality requirements necessary to host as beta.
- 0.5.0 : Federated E2EE chat with oauth authN
- 1.0.0 : woot! :-D 

This gives us an initial sequence of functionality: 
- user cred auth
- core event handling
- E2EE
- oauth authN
- federation


### HTTP server

keanu will be a [Rack](https://github.com/rack/rack) application. We will begin using [Puma](https://github.com/puma/puma) as the HTTP server. For an HTTP framework keanu will start using [Sinatra](https://github.com/sinatra/sinatra). 

These choices are not made out of unique research or requirements driven by the keanu project but rather are considered "standard" starting points for building a ruby-based web app. We will revisit these selections as the need arrises. 

### Deployment

keanu will build into a docker image and be deployed via [Kamal](https://kamal-deploy.org). 

### Access Tokens

I wanted to use [Macaroons](https://research.google/pubs/macaroons-cookies-with-contextual-caveats-for-decentralized-authorization-in-the-cloud/) for access tokens. However, the only ruby gem I was able to find was [boulangerie](https://github.com/cryptosphere/boulangerie) which looks like what we want, but it hasn't been updated in a very long time. In addition, I don't see a lot of people discussing macaroons online in a few quick searches. This might be an incorrect view, but it doesn't look as though macaroons have taken off, despite their theoretical advantage. 

As a result, good ol' oauth is most likely our best starting point for access tokens: [ruby-jwt](https://github.com/jwt/ruby-jwt)

### content parsing

- Everything is JSON, but should be built in a way that can be another encoding
- base64 encoding is tied to JSON usage, since binary data can't be stored directly within JSON

keanu will use the standard ruby [JSON](https://github.com/ruby/json) and [base64](https://docs.ruby-lang.org/en/3.3/Base64.html) gem/methods.

In addition, we will need to build a canonical JSON wrapper, so that the data -> JSON -> base64 is deterministic for the same set of inputs. See [Canonical JSON](#Canonical-JSON)

### Settings

- settings aroudn how often/close to validate base64, basically slider from EVERYWHERE to NOWHERE.
- system to handle per API deprecation/status
- API system needs to manage multiple versions of same endpoint running side by side

## Future Things

These are things which need to be done, but I think we can ignore for the beginning. We shouldn't completely ignore them though, as we shouldn't paint ourselves in a corder implementing them after the fact:

- Rate Limiting - https://spec.matrix.org/v1.15/client-server-api/#rate-limiting
- Well-known URI - https://spec.matrix.org/v1.15/client-server-api/#well-known-uri

### Dependencies

- We are using github for git hosting and project management. While I think this is acceptable at the moment, and our git code is always "backed up", it is something that we should transition off of at some point. 

# Matrix Spec Notes

*These notes assume version [1.15](https://spec.matrix.org/v1.15/) of the Matrix Specification. If no version is specified, then the content comes from that version of the matrix spec.*

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

## Client-Server APIs
*Spec Link:* https://spec.matrix.org/v1.15/client-server-api/ 

- Baseline for all client-server communication is exchanging JSON objects over HTTP APIs. 
- HTTPS is RECOMMENDED. 
- Clients are authenticated using opaque `access_token` strings
- All server responses MUST include a `Content-Type` of `application/json` and include at least empty JSON body for 200 responses

### Error Responses
*Spec Link:* https://spec.matrix.org/v1.15/client-server-api/#standard-error-response

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

#### Standard Error Codes
*Spec Link:* https://spec.matrix.org/v1.15/client-server-api/#common-error-codes)

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

#### Other error codes

There are other error codes unique to different endpoints. For now, I am not listing them in my notes. They can be viewed here: [https://spec.matrix.org/v1.15/client-server-api/#other-error-codes](https://spec.matrix.org/v1.15/client-server-api/#other-error-codes). 

### Users
*Spec Link*: https://spec.matrix.org/v1.15/#users

`@localpart:domain`

- Matrix user IDs are sometimes refered to as MXIDs
- `localpart` MUST NOT be empty and MUST contain only `a-z`,`0-9`, and `._=-/+` characters.
- `domain` is the server name of the homeserver which created the account.
- The length of the user ID, including the `@` sigil and domain, MUST NOT exceed 255 bytes. 

TODO: [4.3.1.1 Historical User IDs](https://spec.matrix.org/v1.15/appendices/#historical-user-ids) - it looks like there is some work around supporting a broader character set for existing sender's user names but that it shouldn't be supported for new users/events. I'm ignoring it for now and leaving this note as I don't think it's important for the initial "happy path".

## Server-Server APIs

## Application Service APIs

## Identity Service APIs

## Push Gateway APIs
