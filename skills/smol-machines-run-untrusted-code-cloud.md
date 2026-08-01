---
name: Run untrusted code in a cloud microVM
description: Create a hardware-isolated machine on the smolfleet cloud, execute a command in it, collect output/files, then tear it down.
api: openapi/smol-machines-smolfleet-openapi.json
operations: [machine_create, machine_start, machine_exec, machine_file_download, machine_stop, machine_delete]
auth: Authorization Bearer smk_<api_key>
---

# Run untrusted code in a cloud microVM

Use the hosted smolfleet Cloud API (`https://api.smolmachines.com`) to run
untrusted or agent-generated code inside a disposable, hardware-isolated
microVM. Every call needs `Authorization: Bearer smk_<api_key>`.

## Steps

1. **Create the machine** — `machine_create` (`POST /v1/machines`) with a source
   image, e.g. `{"source":{"type":"image","reference":"alpine"},"resources":{"cpus":1,"memoryMb":256},"network":{"mode":"open"},"ephemeral":true}`.
   The machine returns in `state: "stopped"` with an `id` (`mach-...`). Keep
   `network.mode` closed for untrusted code unless egress is required.
2. **Start it** — `machine_start` (`POST /v1/machines/{id}/start`). (A machine
   also auto-starts on first exec.)
3. **Execute** — `machine_exec` (`POST /v1/machines/{id}/exec`) with the command
   and args. Capture stdout/stderr and the exit code.
4. **Collect artifacts** — `machine_file_download` (`GET /v1/machines/{id}/files`)
   to pull any output files off the machine.
5. **Tear down** — `machine_stop` (`POST /v1/machines/{id}/stop`) then
   `machine_delete` (`DELETE /v1/machines/{id}`). Set `ephemeral: true` at create
   time to have it reclaimed automatically.

## Rules

- Isolation is the hypervisor boundary — still scope `network` tightly for
  untrusted workloads.
- Errors come back as `{"code","error"}` JSON (or HTTP 402/422/429/503 on the
  Cloud API). Branch on `code`; retry 429 with backoff. See
  `errors/smol-machines-problem-types.yml`.
- No Idempotency-Key header; `name` is the natural dedupe key (re-create → 409).
