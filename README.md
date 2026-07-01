# Open WebUI Langfuse v3 Filter Pipeline

Working Langfuse v3 filter pipeline for Open WebUI Pipelines.

Based on the upstream Open WebUI Pipelines Langfuse v3 filter:

- Upstream repo: <https://github.com/open-webui/pipelines>
- Upstream file: `examples/filters/langfuse_v3_filter_pipeline.py`
- License: MIT

## Tested with

- `ghcr.io/open-webui/pipelines:main`
- Langfuse Python SDK `3.2.8`
- Self-hosted Langfuse v3
- Open WebUI connected to Pipelines as an OpenAI-compatible endpoint

## What changed from upstream

This version keeps the upstream pipeline structure, with two fixes for the working deployment:

1. Reads token usage from Open WebUI's nested format first:

   ```python
   assistant_message["info"]["usage"]
   ```

   and falls back to:

   ```python
   assistant_message["usage"]
   ```

2. Sends token counts to Langfuse v3 using `usage_details` in `start_generation(...)`, with:

   ```python
   input_tokens
   output_tokens
   total_tokens
   ```

See `patches/open-webui-upstream.diff` for the exact diff.

## Run a Pipelines container

If you do not already have an Open WebUI Pipelines container, start one first.

### Docker run

```bash
docker run -d \
  --name pipelines \
  --restart always \
  -p 9099:9099 \
  --add-host=host.docker.internal:host-gateway \
  -v pipelines:/app/pipelines \
  ghcr.io/open-webui/pipelines:main
```

### Docker Compose

```yaml
services:
  pipelines:
    image: ghcr.io/open-webui/pipelines:main
    container_name: pipelines
    restart: always
    ports:
      - "9099:9099"
    extra_hosts:
      - "host.docker.internal:host-gateway"
    volumes:
      - pipelines:/app/pipelines

volumes:
  pipelines:
```

Start it with:

```bash
docker compose up -d pipelines
```

## Install into a Pipelines container

Copy the filter into the persistent Pipelines volume/container:

```bash
docker cp langfuse_v3_filter_pipeline.py pipelines:/app/pipelines/langfuse_v3_filter_pipeline.py
docker restart pipelines
```

Or mount/copy it into your Pipelines volume at:

```text
/app/pipelines/langfuse_v3_filter_pipeline.py
```

## Pipeline valves

Configure these in the Pipelines UI, or create a local `valves.json` beside the pipeline in the container.

Example:

```json
{
  "pipelines": ["*"],
  "priority": 0,
  "secret_key": "sk-lf-...",
  "public_key": "pk-lf-...",
  "host": "http://langfuse-web:3000",
  "insert_tags": true,
  "use_model_name_instead_of_id_for_generation": false,
  "debug": false
}
```

Do **not** commit `valves.json` because it contains secrets.

Common `host` values:

- Same Docker Compose network as Langfuse: `http://langfuse-web:3000`
- Langfuse exposed on host: `http://host.docker.internal:3000` or `http://<server-ip>:3000`
- Langfuse Cloud: `https://cloud.langfuse.com`

## Connect Open WebUI to Pipelines

Add Pipelines as an OpenAI-compatible endpoint in Open WebUI.

Common endpoint URLs:

- Open WebUI using host networking: `http://localhost:9099`
- Same Docker Compose network: `http://pipelines:9099`
- Separate container via host port: `http://host.docker.internal:9099` or `http://<server-ip>:9099`

## Version history

### 0.1.0

- Fix token usage extraction from Open WebUI `info.usage`.
- Send Langfuse v3 token usage via `usage_details`.

## Notes

- Keep Langfuse keys in valves/UI/env only.
- This repo intentionally does not include `valves.json`.
- If upstream Open WebUI changes its Langfuse v3 filter, re-check the diff before updating.
