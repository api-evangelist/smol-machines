---
name: Deploy and scale an app on smolfleet
description: Deploy an app to the smolfleet cloud, tail its logs, scale it, and promote or redeploy a release.
api: openapi/smol-machines-smolfleet-openapi.json
operations: [app_deploy, app_get, app_logs, app_scale, app_promote, app_redeploy, app_destroy]
auth: Authorization Bearer smk_<api_key>
---

# Deploy and scale an app on smolfleet

Run a long-lived workload as a smolfleet **app** (as opposed to a one-off
machine). All calls need `Authorization: Bearer smk_<api_key>`.

## Steps

1. **Deploy** — `app_deploy` (`POST /v1/apps`) with the app name and source.
2. **Inspect** — `app_get` (`GET /v1/apps/{name}`) to read state; `app_logs`
   (`GET /v1/apps/{name}/logs`) to tail output.
3. **Scale** — `app_scale` (`POST /v1/apps/{name}/scale`) to change the number of
   running instances.
4. **Ship a new release** — `app_redeploy` (`POST /v1/apps/{name}/redeploy`) to
   roll out, or `app_promote` (`POST /v1/apps/{name}/promote`) to promote a
   candidate.
5. **Remove** — `app_destroy` (`DELETE /v1/apps/{name}`) when done.

## Rules

- App names are the natural key; deploying an existing name updates in place.
- Watch `app_logs` after a redeploy/scale to confirm health.
- Errors: `{"code","error"}` JSON or HTTP 402/422/429/503. See
  `errors/smol-machines-problem-types.yml`.
