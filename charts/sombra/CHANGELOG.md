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
