# Changelog

## 0.1.0

* Initial version

## 0.2.0

* Chart fixes

## 0.2.1

* Chart fixes

## 0.3.0

* TLS support llm-classifier

## 0.3.1
* Update docs
* Remove old helm packages

## 0.4.0
* Repo name change

## 0.5.0
* Repo structure change

## 0.6.0

* Allow for more advanced Sombra configurations to be deployed.
* Allow for external metrics to be used with the Sombra HPA.
* Allow for Datadog `DD_SERVICE_NAME` environment variable to be customizable.
* Ignore generating a `.dockerconfigjson` if `imagePullSecrets`.`enabled` is set to false.
* Fix typos.

## 0.6.1

* Fix the issue with the `isMultiTenant` parameter by converting it to a string before using it.

## 0.6.2

* Fix issue with external metrics resolution for Sombras Horizontal Pod Autoscaler.

## 0.6.3

* Fixed a bug with the `secrets.yaml` template, which was causing this error during `helm install`: "Secret in version "v1" cannot be handled as a Secret: json: cannot unmarshal string into Go struct field Secret.data of type map[string][]uint8"

## 0.6.4

* Added support for specifying an optional service account name in the chart's values
  * Added optional serviceAccount configuration section (commented out by default)
  * Created serviceaccount.yaml template for conditional service account creation
  * Updated deployment to conditionally use the specified service account
  * **Non-breaking change**: Service account functionality is completely opt-in and disabled by default

## 0.6.5

* Added support for client-managed secrets via `envFrom` configuration
  * Allow loading environment variables from ConfigMaps and Secrets using the `envFrom` field
  * **Non-breaking change**: `envFrom` functionality is completely opt-in and disabled by default

## 0.6.6

* Added support for pod-level and container-level [securityContext](https://kubernetes.io/docs/tasks/configure-pod-container/security-context/) configuration
  * Allow users to configure security contexts for enhanced security and compliance
  * Added `podSecurityContext` and `containerSecurityContext` options in values.yaml for all charts (sombra, llm-classifier, pathfinder)
  * **Non-breaking change**: Security context functionality is completely opt-in and disabled by default

## 0.6.7

* See 0.6.6. This is a re-release of 0.6.6 to test an updated chart release process.

## 0.6.8

* Added support for host-aliases, and volume mounts to the Sombra customer-ingress pods.

## 0.7.0

* Added support for specifying a custom termination grace period for Sombra pods.

## 0.8.0

* Increase default memory and cpu allocation to prevent pod termination.
* Relax `liveness` and `readiness` probe to prevent pod termination. 

## 0.9.0

* Added support for Asynchronous Sombra Job Scheduler.

## 0.10.0

* Added support for Kubernetes Downward API `fieldRef` environment variables via `envs_field_ref`.
  * Supported in both the main Sombra chart and the sombra-job-scheduler subchart.

## 0.11.0

* Added support for a Kubernetes `startupProbe` on the Sombra container.
  * Defaults to the same `/health` endpoint on port `5042` used by the existing liveness and readiness probes, with a longer `failureThreshold` to accommodate slow start-ups.
  * Configurable via the `startupProbe` field in `values.yaml`. Set `startupProbe: null` to disable it entirely.
  * **Non-breaking change**: existing deployments will pick up the default startup probe; behaviour can be reverted by overriding or disabling the value.

## 0.12.0

* Added opt-in OPA (Open Policy Agent) sidecar support to the Sombra Deployment.
  * Gated by `opa.enabled` (default `false`); existing single-container deployments are unchanged.
  * Configurable `opa.image`, `opa.args`, `opa.env`/`opa.envFrom`, `opa.resources`, `opa.securityContext`, `opa.livenessProbe`/`opa.readinessProbe`/`opa.startupProbe`, `opa.lifecycle`, `opa.port`, and supplemental `opa.volumeMounts`.
  * When `opa.config` is non-empty, a `<fullname>-opa-config` ConfigMap is rendered and mounted read-only at `opa.configMountPath` (as `opa.configFileName`), and `--config-file` is appended to generated or custom `opa.args`. A `checksum/opa-config` pod annotation rolls pods on config changes. When `opa.config` is empty and `opa.args` is null, no `args` key is rendered so the image `CMD` is preserved (e.g. seneca-opa baked config).
  * OPA listens on `:8181` (all interfaces) by default so kubelet probes can reach `/health` via the pod IP, and is never exposed through the Sombra Service. Override `opa.args` and disable probes for strict loopback-only isolation.
  * **Non-breaking change**: OPA is completely opt-in and disabled by default.
