# Kubernetes MCP server

Argo CD deploys the upstream `containers/kubernetes-mcp-server` OCI Helm chart
in the `kubernetes-mcp-server` namespace. The root application discovers the
new application automatically. Chart version `0.1.0` and server image
`v0.0.67` are pinned explicitly.

The server uses its in-cluster service account to inspect homelab. Read-only
mode exposes the core inspection tools, including pod logs. The chart creates
a ClusterRole and binding with only get/list/watch permissions for common
cluster resources, metrics, and Argo CD resources. Secrets and pod exec are
not permitted. Other API groups require additional explicit RBAC rules.

Both Teleport agents register the ClusterIP service's Streamable HTTP endpoint
as `kubernetes-mcp-server`. No ingress or NetworkPolicy is configured.
All clients use the same Kubernetes service-account permissions; Teleport user
permissions are not forwarded as Kubernetes identities.

On both Teleport clusters, assign the preset `mcp-user` role, or merge these
permissions into an existing role's `spec.allow`:

```yaml
app_labels:
  env: homelab
  category: kubernetes
mcp:
  tools:
  - '*'
```

After the changes reach `main` and Argo CD syncs:

```sh
kubectl -n kubernetes-mcp-server rollout status deployment/kubernetes-mcp-server
tsh login --proxy=carson.teleport.sh:443
tsh mcp ls
tsh mcp config kubernetes-mcp-server

tsh login --proxy=carson-beams.beams.sh:443
tsh mcp ls
tsh mcp config kubernetes-mcp-server
```

Stdio clients launch `tsh mcp connect kubernetes-mcp-server`. HTTP clients can
use `tsh proxy mcp kubernetes-mcp-server -p 8888` and connect to
`http://127.0.0.1:8888`.

Upstream: https://github.com/containers/kubernetes-mcp-server
