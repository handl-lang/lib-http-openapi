# lib-http-openapi

Pure-handl **OpenAPI 3.0.x spec generator** over
[lib-route](../lib-route) route entries (P-008 / T-143). Produces a
plain handl record shaped like an OpenAPI document —
`{ openapi: "3.0.3", info: {...}, paths: {...}, servers?,
components? }` — JSON-serializable via the `json` library (no YAML).

## API

```hnl
from "http-openapi" import * as oa

doc := oa.openapi(routes)                 -- nested route map
doc := oa.openapi_entries(entries)        -- flat entry list
doc := oa.openapi_entries(entries, opts)  -- + options
```

- `openapi(routes, opts?)` — `routes` is the same nested
  `{ name: leaf | subtree }` shape http-router consumes; it is
  flattened with `route.route_map(routes, route-opts)` first.
- `openapi_entries(entries, opts?)` — `entries` is a flat list of
  `{ path, method | methods, handler?, name?, doc?, constraints? }`
  records (`handler` is accepted but unused — only routing metadata
  is read).

`opts` (all optional):

| key | effect |
| --- | ------- |
| `info` | record merged over `{ title: "API", version: "0.0.0" }` (right-wins per key) |
| `servers` | passed through to top-level `servers` |
| `components` | passed through to top-level `components` |
| `methods` | verb list: expands `"*"` methods into concrete verbs; for `openapi/1` also forwarded to `route_map` as recognized method-map keys |
| `sep`, `leaf_keys`, `spec_keys` | forwarded to `route_map` (`openapi/1` only); `spec_keys` defaults to `["name", "doc"]` so method-map leaves like `{ get: h, doc: {...} }` stay method maps |

## Bridging rules

lib-route path patterns → OpenAPI 3.0.x:

| handl pattern | OpenAPI output |
| ------------- | -------------- |
| `:param` | `{param}` path template + auto `in: path, required: true, schema: {type: "string"}` parameter |
| `:file.:ext` (mixed segment) | `{file}.{ext}` — suffix/prefix vars join with their literals |
| `*name` splat | `{name}` + `x-handl-pattern` vendor ext with the original pattern (a splat matches many segments; a path param matches one) |
| bare `*` splat | literal `*` segment + `x-handl-pattern` (binds nothing) |
| `(/optional)` groups | enumerated expansions — each expanded path gets the same operation (present-branch-before-absent), capped at **16 expansions per pattern**; beyond the cap the base pattern (groups omitted) is emitted with `x-handl-pattern` |
| `?k=v` in path | required `in: query` parameter with `schema.enum: [v]` |
| bare `?k` in path | required `in: query` parameter (presence check, `schema: {type: "string"}`) |
| `constraints` field | same query-parameter treatment (`{k,v}` record or pair list; `v == ""`/`null` = bare) |
| method `"*"` | `x-handl-methods: ["*"]` vendor ext on the operation under the `"*"` key, or expanded into concrete verbs when `opts.methods` is a non-empty list |

`doc` passthrough — `summary`, `description`, `tags`,
`operationId`, `deprecated`, `params`, `requestBody`, `responses`,
`security` are copied verbatim onto the operation (unknown keys
ignored). Generated path/query parameters merge with `doc.params`;
on a `(name, in)` clash the **pattern-derived** parameter wins.

`operationId` precedence: `doc.operationId` → `entry.name` →
generated `<method>_<sanitized-path>` (`*` → `any`); collisions get
`_2`, `_3`, … suffixes in encounter order.

## Example

```hnl
doc := oa.openapi({
  health: { get: h },
  users: {
    ":id": { get: h, delete: h,
             doc: { summary: "user by id", tags: ["users"] } },
    search: { get: h, path: "/users/search?q=rust" }
  },
  ".": h
})
```

produces `paths` keys `/health`, `/users/{id}`, `/users/search`, `/`
with `get`/`delete` operations, an auto `id` path parameter, and a
required `q` query parameter with `enum: ["rust"]`.

## Develop

```bash
HNL=~/.local/bin/hnl bash scripts/hnl-gate.sh
~/.local/bin/hnl test --jobs 2 src/index.test.hnl
```
