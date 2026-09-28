# Local lifecycle contract

Graal Show desktop mirrors production lifecycle semantics while keeping the JVM/Graal implementation independent.

States:

- `active`: a tenant + deployment-pinned JVM/Graal cell is receiving invocations.
- `idle`: the cell is resident and reusable only for that same tenant + immutable deployment generation.
- `frozen`: execution is suspended and no new traffic is admitted.
- `reclaiming`: host pressure policy may reclaim/swap cold pages separately from suspension.
- `draining`: no new invocations enter an old generation while in-flight work finishes.
- `stopped`: cell memory is gone; immutable deployment artifacts remain.

Rules:

1. JVM cells may be reused only within the same tenant + deployment generation; tenant actors are fresh per invocation unless a workload explicitly opts into a stronger stateful actor contract.
2. Freeze and memory reclaim are distinct operations and must be observed separately.
3. Generation activation is prepare -> validate -> stage -> health-check -> activate -> drain old generation -> retire/rollback.
4. The Rust desktop daemon is the sole machine-lifecycle writer; CLI and desktop app are clients only.
5. Erlang may act as the long-lived granddaddy supervisor for hot deployment and host recovery.
6. Runtime stripping and capability admission are required before hostile multi-tenant use; JVM language safety is not a tenant sandbox by itself.
7. Invocation payloads and credentials never travel in argv.
