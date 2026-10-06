# Langfuse Codex Plugin

This repository is [wwwyo's fork](https://github.com/wwwyo/codex-observability-plugin)
of [langfuse/codex-observability-plugin](https://github.com/langfuse/codex-observability-plugin).
It contains custom compact turn tracing and tool-error handling changes for personal
use; these changes are not part of the upstream release. This fork is distributed
as `@wwwyo/codex-observability-plugin`, with the built hook included in the npm
package. Generated bundles are not committed to Git.

Codex plugin that sends OpenAI Codex session telemetry to Langfuse. It traces agent turns, model generations, reasoning summaries, system prompts, tool calls, images, subagent threads, skills, and token usage.

Langfuse also documents this integration on the [Codex integration page](https://langfuse.com/integrations/developer-tools/codex).

## Quick Start

Add the plugin marketplace:

```bash
codex plugin marketplace add wwwyo/codex-observability-plugin --ref main
codex plugin add tracing@codex-observability-plugin
```

The marketplace pins this fork's npm package version. After a new release, refresh
the registered marketplace and install the updated plugin:

```bash
codex plugin marketplace upgrade codex-observability-plugin
codex plugin add tracing@codex-observability-plugin
```

Then enable hooks and the plugin in `~/.codex/config.toml`, or only for one project in `<project>/.codex/config.toml`:

```toml
[features]
hooks = true

[plugins."tracing@codex-observability-plugin"]
enabled = true
```

Restart Codex after changing the config. When **Hooks need review** appears, review the Langfuse `Stop` hook in `/hooks` and trust it. An installed and enabled plugin is not yet a trusted hook, and no traces are uploaded before you trust it. Codex records trust against the current hook hash, so a plugin update can require another review.

## Supported Versions

- Codex `0.143` and newer
- Node.js `22` and newer
- The `npm` CLI on your `PATH`, which Codex uses to fetch the plugin from the npm registry
- Langfuse Cloud, or self-hosted Langfuse `3.95.0` and newer

## Langfuse Credentials

Create `~/.codex/langfuse.json` (global) or `<project>/.codex/langfuse.json` (per-project) with your Langfuse credentials.

```json
{
  "enabled": true,
  "public_key": "pk-lf-...",
  "secret_key": "sk-lf-...",
  "base_url": "https://cloud.langfuse.com"
}
```

Only `enabled`, `public_key` and `secret_key` are required. If `base_url` is not set, the plugin uses `https://cloud.langfuse.com` for the 🇪🇺 EU region. The other regions are `https://us.cloud.langfuse.com` (🇺🇸 US), `https://jp.cloud.langfuse.com` (🇯🇵 Japan) and `https://hipaa.cloud.langfuse.com` (⚕️ HIPAA).

You can also set credentials with environment variables:

```bash
export TRACE_TO_LANGFUSE="true"
export LANGFUSE_PUBLIC_KEY="pk-lf-..."
export LANGFUSE_SECRET_KEY="sk-lf-..."
export LANGFUSE_BASE_URL="https://cloud.langfuse.com"
```

Tracing stays off until `enabled` or `TRACE_TO_LANGFUSE` is true, so you opt in explicitly. Config is resolved as defaults, then `~/.codex/langfuse.json`, then `<project>/.codex/langfuse.json`, then environment variables, and the environment wins. `LANGFUSE_CODEX_*` variables take precedence over the matching `LANGFUSE_*` ones, so you can scope credentials to Codex without disturbing other Langfuse tooling on the same machine.

The remaining options are settable both ways, as a `langfuse.json` key or as the matching `LANGFUSE_CODEX_*` variable:

- `environment` labels the traces with an environment, for example `production`.
- `user_id` attaches a user to every trace. It defaults to the Codex auth email, if one is found.
- `tags` adds your own tags to every trace, either as a JSON array or as a comma-separated list.
- `metadata` attaches a JSON object to every trace.
- `skill_tags` tags traces with `skill:<name>` for every skill invoked in the turn, and defaults to `true`.
- `trace_seed` derives deterministic trace ids, so a headless caller knows a run's trace id up front. Use a unique seed per session.
- `detail` selects `full` (default) or `turn`, with `LANGFUSE_CODEX_DETAIL` as the environment override.
- `debug` logs verbosely to stderr, and defaults to `false`.
- `fail_on_error` fails the hook on upload errors instead of failing open, and defaults to `false`.

Everything the plugin traces is uploaded to Langfuse, including prompts and tool inputs and outputs, so do not enable it for sessions containing data you do not want stored there.

### Turn detail

Set `"detail": "turn"` in `~/.codex/langfuse.json` (without `enabled`) to
use one observation per turn while preserving per-project opt-in. Override with
`"detail": "full"` per project, or `LANGFUSE_CODEX_DETAIL=full`, for individual
model and tool observations.

Turn mode preserves the user input and final response, turn timing, context,
subagent turns, and a versioned `telemetry_summary` JSON value in turn metadata.
The summary contains generation/tool counts, tool names, bounded tool error
messages and times, and token usage grouped by model. It excludes full tool
input/output bodies and intermediate model responses. Failure diagnostics can
include up to 300 characters of tool output; turn mode reduces volume rather than
redacting sensitive content. If no model step has valid usage, the summary falls
back to the turn's delta of cumulative session counters. Aggregate-only usage
from a mixed-model turn is recorded under `unknown` rather than attributed to a
single model. Standard token/cost dashboards, tool-error filtering, and individual
generation latency views require full mode. Previously uploaded turns are not replayed.

## Contributing

See the [contributing guide](./CONTRIBUTING.md).

## License

[MIT](./LICENSE)
