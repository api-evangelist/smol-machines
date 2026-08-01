---
name: Sandbox and run code on the local smolvm API
description: Against a running smolvm server, create a sandbox, run a command or an OCI container in it, stream logs, then delete it.
api: openapi/smol-machines-smolvm-openapi.json
operations: [create_sandbox, start_sandbox, run_command, exec_command, stream_logs, delete_sandbox]
auth: none (loopback 127.0.0.1:8080)
---

# Sandbox and run code on the local smolvm API

Talk to a locally running smolvm server (`smolvm serve start`, default
`http://127.0.0.1:8080`). No auth on the loopback interface.

## Steps

1. **Create a sandbox** — `create_sandbox` (`POST /api/v1/sandboxes`) with a
   unique `name`, optional `resources`, `mounts`, and `ports`. A duplicate name
   returns `409`.
2. **Start it** — `start_sandbox` (`POST /api/v1/sandboxes/{id}/start`).
3. **Run work** — either `run_command` (`POST /api/v1/sandboxes/{id}/run`) to run
   inside a pulled OCI image, or `exec_command` (`POST /api/v1/sandboxes/{id}/exec`)
   to exec directly in the sandbox. Both return stdout/stderr and an exit code.
4. **Stream logs** — `stream_logs` (`GET /api/v1/sandboxes/{id}/logs`) for live
   stdout/stderr (server-sent events).
5. **Clean up** — `delete_sandbox` (`DELETE /api/v1/sandboxes/{id}`); pass
   `?force=true` to force-delete if a stop fails.

## Rules

- Filesystem changes persist across `exec` sessions for a named sandbox; a bare
  `run` is ephemeral. See `conventions/smol-machines-conventions.yml`.
- Errors are `{"code","error"}` JSON, e.g. `{"code":"NOT_FOUND","error":"sandbox 'x' not found"}`.
- To run OCI images, pull first with `pull_image`
  (`POST /api/v1/sandboxes/{id}/images/pull`).
