teleport-cloud-metrics-client
===

The following resources need to be created in Teleport for this work:
```yaml
---
kind: bot
version: v1
metadata:
  name: teleport-cloud-metrics-client
spec:
  roles:
  - teleport-cloud-metrics-client
---
kind: role
metadata:
  name: teleport-cloud-metrics-client
spec:
  allow:
    spiffe:
    - path: /_teleport-cloud/metrics-exporter/client
  deny: {}
version: v6
---
kind: token
metadata:
  name: teleport-cloud-metrics-client
spec:
  allow:
  - aws_account: "210920726206"
  join_method: iam
  roles:
  - Bot
  bot_name: teleport-cloud-metrics-client
version: v2
```
