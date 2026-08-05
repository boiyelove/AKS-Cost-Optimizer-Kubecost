# AKS-Cost-Optimizer-Kubecost

Turn AKS allocation telemetry into safe, measurable optimization recommendations.

## Example synopsis

A cost-policy request maps namespace allocation data to a guarded recommendation for requests, autoscaling, and Spot/on-demand capacity without automatically disrupting protected workloads.

## Real-world scenario

A startup shares one AKS cluster across products but cannot explain which team drives the bill. This framework combines allocation visibility with bounded optimization actions so savings decisions have an owner and reliability guardrails.

## Architecture

Kubecost ingests cluster and Azure billing context; dashboards allocate cost by namespace, workload, label, and tenant. Policies identify idle resources and recommend requests/limits, Spot placement, KEDA scaling, and node-pool changes with guardrails.

Primary services: `AKS`, `Azure Cost Management`, `Kubecost`, `Spot VMs`, `KEDA`.

This repository implements the first production-oriented vertical slice: a
fail-closed, adapter-neutral control plane that validates tenant scope,
freshness, approvals, secretless identity, private access, and the exact
project action before producing a deterministic execution plan. Azure adapters
consume that plan; they are deliberately outside the local simulator so local
tests cannot claim a live cloud change occurred.

```mermaid
flowchart LR
  Request[Desired-state request] --> Validate[Fail-closed validation]
  Validate -->|denied| Evidence[Sanitized denial evidence]
  Validate -->|approved| Plan[Idempotent project plan]
  Plan --> Adapter[Azure adapter integration gate]
  Adapter --> Monitor[Private evidence and monitoring plane]
```

## Quickstart

Requirements: Python 3.11+ and Git. No Azure credentials are required.

```bash
./scripts/validate.sh
python3 src/control_plane.py --request examples/approved-request.json
```

The command emits canonical JSON with a stable idempotency key. The denied
fixture exits with status 2 and explains the failed invariants.

## Security boundaries

- Managed identity or workload identity only; embedded credentials are denied.
- Public network access and stale evidence are denied.
- Production and break-glass targets require explicit approval.
- The IaC entry point is opt-in and defaults to deploying nothing.
- Evidence output contains identifiers and decisions, never credential values.

## Verification and limitations

Local validation covers 13 tests, deterministic replay, JSON parsing, Python
compilation, ignore hygiene, and Bicep compilation when a compiler is present.
It does **not** prove Azure deployment, service licensing, quota, data-plane
permissions, provider/API availability, cloud failover, load, cost, or teardown.
See [[`docs/test-matrix.md`](docs/test-matrix.md)](docs/test-matrix.md) and [[`docs/runbook.md`](docs/runbook.md)](docs/runbook.md) before any integration trial.

## Community

See [[`CONTRIBUTING.md`](CONTRIBUTING.md)](CONTRIBUTING.md), [[`SECURITY.md`](SECURITY.md)](SECURITY.md), [[`SUPPORT.md`](SUPPORT.md)](SUPPORT.md), and [[`LICENSE`](LICENSE)](LICENSE). The reference
is intentionally conservative and uses synthetic identifiers only.

## Repository guide

- [Architecture](docs/architecture.md)
- [Threat model](docs/threat-model.md)
- [Operations runbook](docs/runbook.md)
- [Test matrix](docs/test-matrix.md)
- [Cost model](docs/cost-model.md)
- [Security policy](SECURITY.md)
- [Contributing guide](CONTRIBUTING.md)
- [Support policy](SUPPORT.md)
- [Changelog](CHANGELOG.md)
- [License](LICENSE)
