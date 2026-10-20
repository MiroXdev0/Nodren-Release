# Nodren

![Nodren logo](Logo/Nodren_Logo.png)

Nodren is distributed computing software that connects multiple computers so
they can work together on supported workloads. A Controller coordinates the
cluster, while Workers provide the machines that perform computation.

## How Nodren works

```text
CLI / Desktop
      │ submit and monitor jobs
      ▼
  Controller
      │ assigns bounded work over authenticated TCP
  ┌───┴────┐
Worker   Worker
  │        │
  └───┬────┘
      ▼
  Native Core
      │ results
      └──────────────► Controller ► CLI / Desktop
```

1. A user submits a supported job through Nodren's CLI or desktop interface.
2. The Go Controller validates the request, tracks cluster and job state, and
   divides eligible work into bounded partitions.
3. Rust Workers connect to the Controller, authenticate, and send heartbeats
   so the Controller can track their availability.
4. The Controller assigns partitions to available Workers. Each Worker
   performs its assigned supported computation using Nodren's native Core.
5. Workers return partition results. The Controller tracks progress, combines
   completed work into the job result, and makes the result available to the
   client. If a Worker disconnects, unfinished work can be requeued.

The Controller exposes the management API used by Nodren's clients and
coordinates scheduling; Workers perform distributed computation. The native
Core supplies the built-in `sum`, `xor`, and `dot_product` workloads.

Nodren does not run arbitrary programs. Process, command, script, AI, and
extension-code execution are disabled; submitted numeric data is used only by
the supported built-in workloads.
