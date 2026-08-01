---
name: Run a PxL script against a Pixie cluster
description: >-
  Authenticate to the Pixie gRPC API, find a healthy Kubernetes cluster, and
  execute a PxL observability script, streaming back the result tables.
api: grpc/pixie-labs-vizierapi.proto
operations:
- GetClusterInfo
- HealthCheck
- ExecuteScript
---

# Run a PxL script against a Pixie cluster

Use this skill to query live Kubernetes observability data (metrics, traces,
events, logs) from a Pixie-instrumented cluster via the gRPC API.

## Prerequisites
- A Pixie API key (opaque, `px-api-` prefix). Create one with `px api-key create`
  or in the Live UI under Admin > API Keys. See
  `authentication/pixie-labs-authentication.yml`.
- The target Cluster ID (from `px get viziers` or `GetClusterInfo`).

## Steps
1. **Create a client with your API key.** Pass the key to the client library —
   Go: `pxapi.NewClient(ctx, pxapi.WithAPIKey(key))`; Python:
   `pxapi.Client(token=key)`. The key is sent on the gRPC connection to Pixie
   Cloud.
2. **Find a healthy cluster.** Call the Cloud `VizierClusterInfo.GetClusterInfo`
   RPC (Python `client.list_healthy_clusters()`) and pick a cluster that is
   healthy/connected. Confirm with the Vizier `HealthCheck` RPC (server stream).
3. **Execute a PxL script.** Call the Vizier `ExecuteScript` RPC with the PxL
   source (Go `vz.ExecuteScript`, Python `conn.prepare_script(pxl).run()`). This
   is a server-streaming RPC.
4. **Consume the streamed results.** Read the stream: table metadata arrives
   first, then row batches per table. Access results by table name
   (Python `script_executor.results(table_name)`).
5. **Handle errors.** Check the embedded `Status` (code/message/error_details) in
   `ExecuteScriptResponse` and gRPC status codes; see
   `conventions/pixie-labs-conventions.yml`.

## Notes
- The API is gRPC-only (no REST/OpenAPI). Client libraries are v0.x — expect
  possible breaking changes before v1.0.
- ExecuteScript is a read/query over live telemetry; there is no idempotency key
  and no pagination (results stream as row batches).
