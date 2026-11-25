# lib-http-openapi

Pure-handl **OpenAPI 3.0.x spec generator** over `lib-route` route
entries (P-008 / T-143). Produces a plain handl record shaped like an
OpenAPI document — `{ openapi: "3.0.3", info, paths, servers?,
components? }` — JSON-serializable via `lib-json` (no YAML).

## Layout

| path | role |
| --- | --- |
| `src/index.hnl` | `openapi/1` (nested route maps via `route.route_map`) and `openapi_entries/1` (flat `{path,method,handler,…}` entries) |
| `src/index.test.hnl`, `src/map.test.hnl`, `src/json.test.hnl` | tests |
| `pkg.hnl` | manifest — deps `json`, `route` (`branch:main`) |
| `scripts/hnl-gate.sh`, `ci.sh` | gate + bupha entry |
| `workflows/gate.yaml` | CI — shared `handl-gates` campaign |

## Contract

- Bridges `:param`/`*splat`/`(/optional)`/`?k=v` route patterns to
  OpenAPI paths, parameters and vendor extensions.
- Pure `.hnl`, no Rust crate.

## Verification

```bash
HNL=../host-cli/target/debug/hnl scripts/hnl-gate.sh
```

<!-- myco:agent-awareness -->

## Agent awareness

- Session start / any work discussion: run `myco status --prompt`
  before proposing next steps — know the board first.
- User describes new work → offer to capture it (`myco todo add`,
  `issue new`, `proposal new`). Don't wait to be asked.
- Picking up a task: `myco resume <ID>` is the briefing.
- Wrapping up: `myco handoff <ID>` anything resumable; keep statuses
  honest (claimed → in-progress → blocked/done).

<!-- /myco:agent-awareness -->
