# Inter-Service Communication Standard: gRPC (Tier 3 — Not Supported)

```
====================================================================
[NOT YET SUPPORTED]
gRPC inter-service communication is currently classified under Tier 3.
The standards repository does not yet define certified Protocol Buffer
schemas, code generation pipelines, or gRPC transport contracts.
====================================================================
```

## 1. Purpose
Documents the status of gRPC communication.

---

## 2. Applies when
- `inter_service_comm: grpc` is selected in Step 7.

---

## 3. Mandatory Agent Action
- If the user selects this option during intake or in `project-spec.yaml`, the AI agent MUST NOT guess or invent gRPC contracts.
- The agent MUST halt execution, display the banner above, and ask the user to select an active Tier 1 (`rest`) or Tier 2 (`rabbitmq`) alternative.
