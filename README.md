# dsh-image-gen-multi

Multi-provider image generation setup for DeepSeek Harness.

This extends [@goodandready/dsh-image-gen](https://github.com/GooDAnDReaDY/dsh-image-gen) so you can:

- Configure **multiple providers** (fal, Replicate, custom, etc.) with their own API keys
- Define **multiple models per provider**
- Set an explicit **fallback order**
- Let the **orchestrator** choose the right model via a skill (no manual reconfiguration)

## Install the base plugin

```bash
# Inside the DSH container
dsh plugin --profile web add @goodandready/dsh-image-gen
```

Or from the host:

```bash
CONTAINER=$(podman ps --format '{{.Names}}' | grep deepseek-harness | head -1)
podman exec "$CONTAINER" dsh plugin --profile web add @goodandready/dsh-image-gen
```

Restart DSH after installing.

## Configuration

See `config/providers.example.yml` for a full example.

Merge it into your DSH profile patch (`~/.dsh/profiles/web/cordis.patch.yml`) or settings. Key points:

- `provider` / `model` — primary used when the orchestrator does not specify
- `apiKeyEnv` / `replicateKeyEnv` / `customKeyEnv` — credential references
- `fallbackProviders` — ordered list tried on 429/5xx/timeout
- `providers` — catalog of providers and models with use-case tags (`drafts`, `normal`, `final`, `text`)

## Orchestrator skill

Copy `skills/image-gen/SKILL.md` into your project:

```
<project>/.dsh/skills/image-gen/SKILL.md
```

The skill tells the orchestrator which provider/model to request for drafts, normal assets, final quality, and text-in-image.

## Environment variables

```bash
export FAL_API_KEY="your-fal-key"
export REPLICATE_API_TOKEN="r8_your-token"
export CUSTOM_IMAGE_API_KEY="optional"
```

## Notes

- The base plugin already supports fal and Replicate natively.
- `fallbackProviders` controls automatic failover on rate limits and server errors.
- The orchestrator skill controls **intentional** model choice per task.
- If the tool does not yet accept a `provider`/`model` argument, the skill falls back to the configured primary + fallback chain.
