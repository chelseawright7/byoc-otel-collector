# byoc-otel-collector

A bring-your-own OpenTelemetry Collector distribution for Bindplane, built with the [OpenTelemetry Collector Builder](https://opentelemetry.io/docs/collector/custom-collector/) through [otel-distro-builder](https://github.com/observIQ/otel-distro-builder).

- `manifest.yaml`: the components in the collector, including the ones Bindplane needs
- `.github/workflows/release.yml`: pushing a `v*` tag builds linux/amd64 and linux/arm64 and publishes a GitHub Release

Each release package (`byoc-otel-collector_<version>_linux_<arch>.tar.gz`) contains the collector, the upstream OpAMP Supervisor, an example `supervisor_config.yaml`, and service files.

## Components Bindplane needs

| Component | Kind | Why |
|---|---|---|
| `snapshotprocessor` | processor | Recent Telemetry (snapshots); injected into every modular config |
| `throughputmeasurement` | processor | Throughput measurements on the pipeline graph |
| `topology` | processor | Topology view (gateway hops) |
| `opamp` | extension | Used by the supervisor; the processors above report through it |
| `bindplane` | extension | Measurements fallback for servers or processors that can't self-report |
| `route` | receiver | Log-based metrics and routing |
| `prometheus` | receiver | Collector pipeline health: Bindplane adds a `prometheus/_agent_pipeline_health_metrics` receiver that scrapes the collector's own `:8888` metrics |
| `metricstransform` | processor | Collector health: Bindplane adds a `metricstransform/_agent_health_metrics` processor |
| `nop` | receiver and exporter | Supervisor bootstrap pipeline |

Bindplane adds the two health pipelines to every modular configuration. Without `prometheus` and `metricstransform` the collector rejects the whole configuration and restarts in a loop (v1.0.0 had this problem).

Bindplane components come from [bindplane-otel-contrib](https://github.com/observiq/bindplane-otel-contrib) v1.15.0, which is built on OpenTelemetry 0.161.0, so the upstream components are pinned to 0.161.0 to match.

## Bindplane

Bindplane identifies the agent type by `dist.name` (`byoc-otel-collector`). Point that agent type at this repo so Bindplane can list its releases:

```yaml
apiVersion: bindplane.observiq.com/v1
kind: AgentType
metadata:
  name: byoc-otel-collector
spec:
  repositoryLink: https://github.com/chelseawright7/byoc-otel-collector
```

## Release a new version

1. Change `manifest.yaml` and set `dist.version` to the new version.
2. Commit, then `git tag v<version> && git push --tags`.
