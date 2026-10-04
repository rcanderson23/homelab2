# Grafana MCP

Argo CD deploys the Grafana `grafana-mcp` chart in the `grafana` namespace.
The root application discovers `application.yaml` automatically. The server
uses streamable HTTP at `/mcp` and connects to the existing Grafana service.
External Secrets supplies the Grafana service-account token from Bitwarden
secret `895b1950-3c76-4901-bffc-b4d9003664e2`. Token permissions determine which
Grafana operations are available.

Both Teleport agents register it as `mcp-grafana`. The service uses ClusterIP
on port 8000. Health probes use a separate listener on port 8080.

## Teleport permissions

Assign the preset `mcp-user` role on each cluster, or add the following to an
existing role's `spec.allow` (merge labels with existing app access rules):

```yaml
app_labels:
  env: homelab
  category: monitoring
mcp:
  tools:
  - '*'
```

The repository does not manage user roles for these two hosted clusters.
Tool access can be narrowed by replacing `'*'` with selected tool names.

After the changes reach `main` and Argo CD syncs, verify the ExternalSecret
and deployment:

```sh
kubectl -n grafana get externalsecret mcp-grafana
kubectl -n grafana rollout status deployment/mcp-grafana
```

Log in to either cluster and generate a client configuration:

```sh
tsh login --proxy=carson.teleport.sh:443
tsh mcp ls
tsh mcp config mcp-grafana

tsh login --proxy=carson-beams.beams.sh:443
tsh mcp ls
tsh mcp config mcp-grafana
```

Clients using stdio launch `tsh mcp connect mcp-grafana`. For an HTTP client,
run `tsh proxy mcp mcp-grafana -p 8888` and use `http://127.0.0.1:8888`.

References: [Grafana Helm deployment](https://grafana.com/docs/grafana/latest/developer-resources/mcp/set-up/deploy-with-helm/)
and [Teleport MCP enrollment](https://goteleport.com/docs/enroll-resources/mcp-access/enrolling-mcp-servers/streamable-http/).
