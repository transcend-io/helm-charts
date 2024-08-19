# Sombra Helm Charts

Official Helm charts to deploy Sombra and related services into a Kubernetes cluster.

## Deploying with Helm

1. Make sure you have a working Kubernetes cluster.

2. Add the Helm repo

   ```bash
   helm repo add sombra https://transcend-io.github.io/sombra-helm-chart/
   ```

3. To configure the chart and provide environment variables, create a `values.yaml` file:

   ```yaml
   imageCredentials:
     registry: docker.transcend.io
     username: Transcend
     password: "<TRANSCEND_API_TOKEN>"

   transcend_service:
     type: NodePort

   transcend_ingress:
     enabled: true
     className: alb
     annotations:
       alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
       alb.ingress.kubernetes.io/healthcheck-path: /health
       alb.ingress.kubernetes.io/healthcheck-port: "5042"
       alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
       alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5042}]'
       alb.ingress.kubernetes.io/scheme: internet-facing
       alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
       alb.ingress.kubernetes.io/tags: env=dev
       alb.ingress.kubernetes.io/target-type: ip
     hosts:
       - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
         paths:
           - path: /
             pathType: Prefix

   customer_service:
     type: NodePort

   customer_ingress:
     enabled: true
     className: alb
     annotations:
       alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
       alb.ingress.kubernetes.io/healthcheck-path: /health
       alb.ingress.kubernetes.io/healthcheck-port: "5039"
       alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
       alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5039}]'
       alb.ingress.kubernetes.io/scheme: internal
       alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
       alb.ingress.kubernetes.io/tags: env=dev
       alb.ingress.kubernetes.io/target-type: ip
     hosts:
       - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
         paths:
           - path: /
             pathType: Prefix

   envs:
     - name: ORGANIZATION_URI
       value: "<ORGANIZATION_URI>"
     - name: EMPLOYEE_AUTHENTICATION_METHODS
       value: "transcend,session"
     - name: DATA_SUBJECT_AUTHENTICATION_METHODS
       value: "transcend,session"

   envs_as_secret:
     - name: INTERNAL_KEY_HASH
       value: "<INTERNAL_KEY_HASH>"
     - name: JWT_ECDSA_KEY
       value: "<JWT_ECDSA_KEY>"
     - name: INTERNAL_KEY
       value: "<INTERNAL_KEY>"
   ```

   **Note:** This example of `values.yaml` assumes you have deployed (A) a working Kubernetes cluster, and (B) an `alb` AWS application load balancer ingress controller.

4. Install the package:

   ```bash
   helm install some-release sombra-chart/sombra -f ~/values.yaml
   ```

   _To upgrade the package after customization:_

   ```bash
   helm upgrade some-release sombra-chart/sombra -f ~/values.yaml
   ```

   _To uninstall a release:_

   ```bash
   helm uninstall some-release
   ```

## Deployment examples

The following are sample `values.yaml` files for deploying Sombra in an AWS EKS cluster with different configurations. AWS is not a requirement—Kubernetes can run on any cloud—but AWS is used in these examples for illustrative purposes.

- Each example assumes you have deployed (A) a working Kubernetes cluster, and (B) an `alb` AWS application load balancer ingress controller.
- In each example, we are exposing Sombra's `sombra-transcend-ingress` server (for communication with Transcend through an internet-facing load balancer) and Sombra's `sombra-customer-ingress` server (for communication with internal services through an internal load balancer).

### Deploying Sombra without TLS

TLS can be terminated at the load balancer, or at Sombra's server. In this example, TLS is terminated at the load balancer.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: NodePort

transcend_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5042"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5042}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
    alb.ingress.kubernetes.io/tags: env=dev
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

customer_service:
  type: NodePort

customer_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5039"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5039}]'
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
    alb.ingress.kubernetes.io/tags: env=dev
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: "<INTERNAL_KEY_HASH>"
  - name: JWT_ECDSA_KEY
    value: "<JWT_ECDSA_KEY>"
  - name: INTERNAL_KEY
    value: "<INTERNAL_KEY>"
```

### Deploying Sombra with TLS

TLS can be terminated at the load balancer, or at Sombra's server. In this example, TLS is terminated at Sombra's server.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: NodePort

transcend_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5041"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTPS
    alb.ingress.kubernetes.io/backend-protocol: HTTPS
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5041}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
    alb.ingress.kubernetes.io/tags: env=dev
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

customer_service:
  type: NodePort

customer_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5040"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTPS
    alb.ingress.kubernetes.io/backend-protocol: HTTPS
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5040}]'
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
    alb.ingress.kubernetes.io/tags: env=dev
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: "<INTERNAL_KEY_HASH>"
  - name: JWT_ECDSA_KEY
    value: "<JWT_ECDSA_KEY>"
  - name: INTERNAL_KEY
    value: "<INTERNAL_KEY>"
  - name: SOMBRA_TLS_KEY
    value: <SOMBRA_TLS_KEY>
  - name: SOMBRA_TLS_KEY_PASSPHRASE
    value: <SOMBRA_TLS_KEY_PASSPHRASE>
  - name: SOMBRA_TLS_CERT
    value: <SOMBRA_TLS_CERT>
```

### Deploying Sombra and the LLM Classifier

This example deploys Sombra with an accompanying LLM Classifier. The LLM Classifer requires an Nvidia GPU to run, so please make sure your cluster supports `nvidia.com/gpu` as a resource.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: NodePort

transcend_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5042"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5042}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

customer_service:
  type: NodePort

customer_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5039}]'
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: LLM_CLASSIFIER_URL
    value: http://<release-name>-llm-classifier.transcend.svc:6081

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: "<INTERNAL_KEY_HASH>"
  - name: JWT_ECDSA_KEY
    value: "<JWT_ECDSA_KEY>"
  - name: INTERNAL_KEY
    value: "<INTERNAL_KEY>"

llm-classifier:
  enabled: true
```

### Deploying the LLM Classifier with tls enabled

This example deploys LLM Classifier with TLS enabled to keep internal communication with sombra encrypted. The LLM Classifer requires an Nvidia GPU to run, so please make sure your cluster supports `nvidia.com/gpu` as a resource.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: NodePort

transcend_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5042"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5042}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

customer_service:
  type: NodePort

customer_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5039}]'
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: LLM_CLASSIFIER_URL
    value: https://<release-name>-llm-classifier.transcend.svc:6081

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: "<INTERNAL_KEY_HASH>"
  - name: JWT_ECDSA_KEY
    value: "<JWT_ECDSA_KEY>"
  - name: INTERNAL_KEY
    value: "<INTERNAL_KEY>"

llm-classifier:
  enabled: true
  tls:
    enabled: true,
    # saved as secret
    cert: |-
      -----BEGIN CERTIFICATE-----
      <base64>
      -----END CERTIFICATE-----
    # saved as secret
    key: |-
      -----BEGIN PRIVATE KEY-----
      <base64>
      -----END PRIVATE KEY-----

  # volume containg cert and key
  volumes:
    - name: llm-classifier-ssl
      secret:
        secretName: llm-classifier-secrets

  # mount the directory containing the cert and key to pod
  volumeMounts:
    - mountPath: "/etc/llm-classifier/ssl"
      name: llm-classifier-ssl
      readOnly: true

  # Set the location of cert and key in evironment
  envs:
    - name: LLM_CERT_PATH
      value: "/etc/llm-classifier/ssl/llm-classifier.cert"
    - name: LLM_KEY_PATH
      value: "/etc/llm-classifier/ssl/llm-classifier.key"
```

### Deploying Sombra and Pathfinder

The following example adds Pathfinder to the Kubernetes deployment.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: NodePort

transcend_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-port: "5042"
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5042}]'
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/subnets: <VPC_PUBLIC_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

customer_service:
  type: NodePort

customer_ingress:
  enabled: true
  className: alb
  annotations:
    alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
    alb.ingress.kubernetes.io/healthcheck-path: /health
    alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 5039}]'
    alb.ingress.kubernetes.io/scheme: internal
    alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
    alb.ingress.kubernetes.io/tags: env=prod
    alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: "<INTERNAL_KEY_HASH>"
  - name: JWT_ECDSA_KEY
    value: "<JWT_ECDSA_KEY>"
  - name: INTERNAL_KEY
    value: "<INTERNAL_KEY>"

pathfinder:
  enabled: true
  envs_as_secret:
    - name: OPEN_AI_API_KEY
      value: "<OPEN_AI_API_KEY>"
    - name: AUTHENTICATION_KEY_HASH
      value: "<AUTHENTICATION_KEY_HASH>"
    - name: TRANSCEND_API_KEY
      value: "<TRANSCEND_API_KEY>"

  service:
    type: NodePort

  ingress:
    enabled: true
    annotations:
      alb.ingress.kubernetes.io/certificate-arn: <CERT_ARN>
      alb.ingress.kubernetes.io/healthcheck-path: /health
      alb.ingress.kubernetes.io/healthcheck-protocol: HTTP
      alb.ingress.kubernetes.io/listen-ports: '[{"HTTPS": 3030}]'
      alb.ingress.kubernetes.io/scheme: internal
      alb.ingress.kubernetes.io/subnets: <VPC_PRIVATE_SUBNET>
      alb.ingress.kubernetes.io/tags: env=prod
      alb.ingress.kubernetes.io/target-type: ip
  hosts:
    - host: <PATHFINDER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix
```

### Deploying Sombra on Azure

The following example deploys Sombra on Azure. This would require setting up an Application Gateway, which will be used as the ingress controller, on Azure and adding the TLS secret into the cluster.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: "<TRANSCEND_API_TOKEN>"

transcend_service:
  type: ClusterIP

transcend_ingress:
  enabled: true
  className: "azure-application-gateway"
  annotations:
    appgw.ingress.kubernetes.io/health-probe-hostname: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
    appgw.ingress.kubernetes.io/health-probe-path: /health
    appgw.ingress.kubernetes.io/health-probe-port: "5042"
  hosts:
    - host: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: <SOMBRA_TRANSCEND_TLS_SECRET_NAME>
      hosts:
        - <SOMBRA_TRANSCEND_INGRESS_DOMAIN>

customer_service:
  type: ClusterIP

customer_ingress:
  enabled: true
  className: "azure-application-gateway"
  annotations:
    appgw.ingress.kubernetes.io/health-probe-hostname: <SOMBRA_TRANSCEND_INGRESS_DOMAIN>
    appgw.ingress.kubernetes.io/health-probe-path: /health
    appgw.ingress.kubernetes.io/health-probe-port: "5039"
  hosts:
    - host: <SOMBRA_CUSTOMER_INGRESS_DOMAIN>
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: <SOMBRA_CUSTOMER_TLSSECRET_NAME>
      hosts:
        - <SOMBRA_CUSTOMER_INGRESS_DOMAIN>

envs:
  - name: ORGANIZATION_URI
    value: "<ORGANIZATION_URI>"
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: "transcend,session"
  - name: SOMBRA_ID
    value: "<SOMBRA_ID>"

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: <INTERNAL_KEY_HASH>
  - name: JWT_ECDSA_KEY
    value: <JWT_ECDSA_KEY>
  - name: INTERNAL_KEY
    value: <INTERNAL_KEY>

livenessProbe:
  httpGet:
    path: /health
    port: 5042
    scheme: HTTP
  timeoutSeconds: 100
  periodSeconds: 30
  successThreshold: 1
  failureThreshold: 10

readinessProbe:
  httpGet:
    path: /health
    port: 5042
    scheme: HTTP
  timeoutSeconds: 100
  periodSeconds: 30
  successThreshold: 1
  failureThreshold: 10
```

## Configuring Sombra

The following is a list of enviroment variables supported by Sombra for its configuration. Please check out our detailed [guide](https://docs.transcend.io/docs/security/end-to-end-encryption/deploying-sombra) on self-hosting Sombra.

| Variables                              | Required                                                                       | default                                                                                                                    | secret | Description                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| -------------------------------------- | ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --- |
| ORGANIZATION_URI                       | yes                                                                            | N/A                                                                                                                        | no     | This value can be found under "Sombra Audience" here: [https://app.transcend.io/infrastructure/sombra/sombras](https://app.transcend.io/infrastructure/sombra/sombras)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| SOMBRA_ID                              | no                                                                             | N/A                                                                                                                        | no     | The SOMBRA_ID parameter is only required when deploying multiple Sombra gateways. This value can be found under "ID" here: [https://app.transcend.io/infrastructure/sombra/sombras](https://app.transcend.io/infrastructure/sombra/sombras).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| EMPLOYEE_AUTHENTICATION_METHODS        | yes                                                                            | N/A                                                                                                                        | no     | We recommend starting with the 'transcend' authentication method. After Single Sign On is setup, 'transcend' can be switched to 'saml'.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| DATA_SUBJECT_AUTHENTICATION_METHODS    | yes                                                                            | N/A                                                                                                                        | no     | We recommend starting with the 'transcend' authentication method. After Account Login is setup, 'transcend' can be switched to 'oauth' or 'jwt'.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| JWT_ECDSA_KEY                          | yes                                                                            | N/A                                                                                                                        | yes    | The root secrets that you should generate yourself and keep secret. If you are migrating from a Transcend-hosted multi-tenant Sombra to an on-premise and are mid-implementation, it is critical that you re-use the same JWT_ECDSA_KEY from the existing instance. If you already have connected integrations and DSRs, you should deploy the on premise Sombra gateway with the same JWT_ECDSA_KEY and then run a key rotation after the gateway is deployed. To obtain the JWT_ECDSA_KEY reach out to your account manager over Slack or email [support@transcend.io](mailto:support@transcend.io) to grant access for you to download the key. You will only be able to reveal the key once for security reasons. The key can be obtained by clicking the Reveal Multi Tenant Root Secret button on the Sombra Gateways panel in the Admin UI. |
| JWT_AUTHENTICATION_PUBLIC_KEY          | no                                                                             | N/A                                                                                                                        | no     | Company's data subject authentication via JWT public key(s). First entry is latest.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ALLOW_EMPLOYEE_CEK_ACCESS              | no                                                                             | true                                                                                                                       | no     | Allow employees to submit and download DSRs on behalf of the data subjects.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| CONSENT_IDENTIFIER_ENCRYPTION_KEY      | no                                                                             | N/A                                                                                                                        | yes    | The Elliptic Curve Diffie Hellman private key(s), for securely communicating with the data subject. Derived from JWT_ECDSA_KEY by default. First entry is latest.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| INTERNAL_KEY_HASH                      | yes                                                                            | N/A                                                                                                                        | yes    | The hash of a randomly-generated API key that will be used to verify incoming requests from your internal systems. You can use [this](https://docs.transcend.io/docs/security/end-to-end-encryption/deploying-sombra#6.-cycle-your-keys) guide for the generation of key.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| SAML_ENTRYPOINT                        | no                                                                             | N/A                                                                                                                        | no     | Identity provider entrypoint (is required to be spec-compliant when the request is signed).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| SAML_ISSUER                            | no                                                                             | `transcend`                                                                                                                | no     | Issuer string to supply to identity provider.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| SAML_CERT                              | no                                                                             | N/A                                                                                                                        | no     | The public key to validate the SAML assertion against. The "BEGIN CERTIFICATE" and "END CERTIFICATE" lines should be stripped out and the certificate should be provided on a single line.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| SAML_AUDIENCE                          | no                                                                             | `transcend`                                                                                                                | no     | Expected SAML response Audience (if not provided, Audience won't be verified)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| OAUTH_CLIENT_ID                        | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS. | no                                                                                                                         | no     | The Client ID of your privacy center's OAuth 2 application.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| OAUTH_CLIENT_SECRET                    | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS. | N/A                                                                                                                        | yes    | The Client Secret of your privacy center's OAuth 2 application.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| OAUTH_GET_TOKEN_URL                    | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS. | N/A                                                                                                                        | no     | The token URL for your OAuth API (authorization_code => access_token) e.g. [https://api.acme.com/oauth/token](https://api.acme.com/oauth/token).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| OAUTH_GET_TOKEN_HEADERS                | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS. | N/A                                                                                                                        | no     | The headers to include when fetching the OAuth token. Expected format is a stringified JSON object, e.g. `{"Content-Type": "application/x-www-form-urlencoded; charset=UTF-8"}`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| OAUTH_GET_TOKEN_METHOD                 | no                                                                             | `POST`                                                                                                                     | no     | The HTTP method to use when retrieving the OAuth token                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| OAUTH_GET_TOKEN_BODY_GRANT_TYPE        | no                                                                             | N/A                                                                                                                        | no     | The grant type for this OAuth token                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| OAUTH_GET_TOKEN_BODY_REDIRECT_URI      | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS  | N/A                                                                                                                        | no     | The redirect URI for the Oauth token body.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| OAUTH_GET_CORE_ID_URL                  | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS  | N/A                                                                                                                        | no     | The API endpoint where we can find a user ID or similar core identifier (e.g. [https://api.acme.com/user-profile](https://api.acme.com/user-profile)).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| OAUTH_GET_CORE_ID_PATH                 | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS  | N/A                                                                                                                        | no     | A path to the coreIdentifier (User ID) to traverse a JSON body (path.to.user.id).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| OAUTH_GET_CORE_ID_DATA_SUBJECT_TYPE    | yes if you configured oauth as one of the DATA_SUBJECT_AUTHENTICATION_METHODS  | N/A                                                                                                                        | no     | The type of Data Subject that the OAuth configuration refers to. This should use the Data Subject slug, which is the value found in parentheses under DSR Automation -> Request Settings -> Data Subjects.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| OAUTH_GET_PROFILE_PICTURE_URL          | no                                                                             | N/A                                                                                                                        | no     | Configuration to retrieve additional profile data for the data subject (e.g. nickname, picture).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| OAUTH_GET_PROFILE_PICTURE_PATH         | no                                                                             | N/A                                                                                                                        | no     | A path to the profile picture URL image, from the OAuth response body.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| OAUTH_GET_EMAIL_URL                    | no                                                                             | N/A                                                                                                                        | no     | The URL that should be hit to fetch the user email address.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| OAUTH_GET_EMAIL_PATH                   | no                                                                             | N/A                                                                                                                        | no     | In the JSON response body for the data subject oauth authentication, what is the path to the email? e.g. profile.email.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| OAUTH_EMAIL_IS_VERIFIED_PATH           | no                                                                             | no                                                                                                                         | no     | JSON path to the is_verified field on the Oauth profile.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| OAUTH_EMAIL_IS_VERIFIED                | no                                                                             | N/A                                                                                                                        | no     | Whether all OAuth emails are already verified by default, attested by the organization.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| DATA_SUBJECT_SESSION_EXPIRY_TIME       | no                                                                             | '5 days'                                                                                                                   | no     | The length in time that the session JWT will last                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| EMPLOYEE_SESSION_EXPIRY_TIME           | no                                                                             | '3 hour'                                                                                                                   | no     | The length in time that the session JWT will last                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| SOMBRA_TLS_KEY                         | yes, if you want encrypted traffic b/w Sombra and the load balancer.           | N/A                                                                                                                        | no     | The TLS private key for this server, base64-encoded in PEM format.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| SOMBRA_TLS_KEY_PASSPHRASE              | yes, if you want encrypted traffic b/w Sombra and the load balancer.           | N/A                                                                                                                        | yes    | when generating your TLS cert, you may have been prompted to add a passphrase to your private key. If you did, you can add the passphrase here to let Sombra access it.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| SOMBRA_TLS_CERT                        | yes, if you want encrypted traffic b/w Sombra and the load balancer.           | N/A                                                                                                                        | yes    | The TLS certificate for this server, base64-encoded in PEM format.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| TRUSTED_CLIENT_CA_CERT_ENCODED         | no                                                                             | N/A                                                                                                                        | yes    | The public CA cert from a client who is connecting to the internal Sombra over mutual TLS. If set, Sombra will enforce that all incoming requests to non-health routes have a client cert.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| EXTERNAL_PORT_HTTPS                    | no                                                                             | 5041                                                                                                                       | no     | Port for the external HTTPS server (faces inside firewall).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| EXTERNAL_PORT_HTTP                     | no                                                                             | 5042                                                                                                                       | no     | Port for the external HTTP server (faces inside firewall)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| INTERNAL_PORT_HTTPS                    | no                                                                             | 5040                                                                                                                       | no     | Port for the internal HTTPS server (faces inside firewall)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| INTERNAL_PORT_HTTP                     | no                                                                             | 5039                                                                                                                       | no     | Port for the internal HTTP server (faces inside firewall)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| LLM_CLASSIFIER_URL                     | no                                                                             | N/A                                                                                                                        | no     | The LLM classifier service endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| SOMBRA_IS_BEHIND_PROXY                 | no                                                                             | false                                                                                                                      | no     | Whether Sombra is behind a proxy.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ODBC_POOL_CACHE_SIZE                   | no                                                                             |                                                                                                                            | no     | Max size of LRU cache to store the connection pool per database.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ODBC_CONNECTION_MAX_POOL_SIZE          | no                                                                             | 100                                                                                                                        | no     | The maximum number of open Connections the Pool will create                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ODBC_CONNECTION_INITIAL_POOL_SIZE      | no                                                                             | Set a relevant value, to enable connection pooling, based on the number of odbc based Integrations you planning to connect | 10     | The initial number of Connections created in the Pool                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ODBC_CONNECTION_POOL_SIZE_INCREMENT    | no                                                                             | 10                                                                                                                         | no     | How many additional Connections to create when all of the Pool's connections are taken                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| ODBC_CONNECTION_POOL_SIZE_SHRINK       | no                                                                             | true                                                                                                                       | no     | Whether or not the number of Connections should shrink to initialSize as they free up                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| REUSE_ODBC_CONNECTIONS                 | no                                                                             | false                                                                                                                      | no     | Whether or not to reuse an existing Connection instead of creating a new one                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| ODBC_LOGIN_TIMEOUT_IN_SECONDS          | no                                                                             | 10                                                                                                                         | no     | The number of seconds to wait for a login request to complete before returning to the application                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ODBC_CONNECTION_TIMEOUT_IN_SECONDS     | no                                                                             | 0                                                                                                                          | no     | The number of seconds to wait for a request on the connection to complete before returning to the application                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| ODBC_QUERY_TIMEOUT_IN_SECONDS          | no                                                                             | 60                                                                                                                         | no     | How long to wait for each ODBC query to execute before returning to the application                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| ODBC_POOL_CACHE_TTL_MS                 | no                                                                             | 3600000                                                                                                                    | no     | How long the connection pool per database should be cached in milliseconds                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| MONGODB_SERVER_SELECTION_TIMEOUT_IN_MS | no                                                                             | 30000                                                                                                                      | no     | How long to wait to connect to MongoDB                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| MONGODB_CONNECT_TIMEOUT_IN_MS          | no                                                                             | 30000                                                                                                                      | no     | How long to wait for each ongoing connection                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| KMS_PROVIDER                           | no                                                                             | `local`                                                                                                                    | no     | The key management provider. Follow this [guide](https://docs.transcend.io/docs/security/end-to-end-encryption/deploying-sombra#kms) for setup.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| AWS_REGION                             | yes, when you are using `AWS` as `KMS_PROVIDER`.                               | N/A                                                                                                                        | no     | The AWS Region where the KMS is hosted                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| AWS_KMS_KEY_ARN                        | yes, when you are using `AWS` as `KMS_PROVIDER`.                               | N/A                                                                                                                        | yes    | The Amazon Resource Name for the Amazon KMS.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| AWS_ACCESS_KEY_ID                      | yes, when you are using `AWS` as `KMS_PROVIDER`.                               | N/A                                                                                                                        | yes    | The AWS access key ID, used to access the Amazon KMS.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| AWS_SECRET_ACCESS_KEY                  | yes, when you are using `AWS` as `KMS_PROVIDER`.                               | N/A                                                                                                                        | yes    | The AWS secret access key, used to access the Amazon KMS.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |     |
| LOG_HTTP_TRANSPORT_URL                 | yes if your want to forward Sombra logs to transcend                           | N/A                                                                                                                        | no     | The Transcend Collector's HTTPS ingress endpoint.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| LOG_HTTP_TRANSPORT_BATCH_INTERVAL_MS   | no                                                                             | 5000                                                                                                                       | no     | The maximum time to wait between batches of logs sent to the Collector.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| LOG_HTTP_TRANSPORT_BATCH_COUNT         | no                                                                             | 10                                                                                                                         | no     | The maximum number of log lines to send in a single batched request.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| DD_SERVICE_NAME                        | yes if your want to forward Sombra logs to transcend                           | `customer_hosted_sombra`                                                                                                   | no     | The name for your Sombra.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| AWS_ACCESS_KEY_ID                      | yes if you want to utilize any of the AWS integrations                         | N/A                                                                                                                        | no     | The AWS access key ID of the IAM user with the STS:AssumeRole                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| AWS_SECRET_ACCESS_KEY                  | yes if you want to utilize any of the AWS integrations                         | N/A                                                                                                                        | no     | The AWS secret access key of the IAM user with the STS:AssumeRole                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |

## Configuring the LLM Classifier

| Variables                     | Required | default          | secret | Description                                                              |
| ----------------------------- | -------- | ---------------- | ------ | ------------------------------------------------------------------------ |
| LLM_SERVER_PORT               | no       | 6081             | no     | Port on which server listen to.                                          |
| LLM_SERVER_CONCURRENCY        | no       | (cpu count) \* 2 | no     | The number of worker processes for handling requests.                    |
| LLM_SERVER_TIMEOUT            | no       | 120              | no     | Workers silent for more than this many seconds are killed and restarted. |
| LLM_SERVER_BACKLOG            | no       | 500              | no     | The maximum number of pending connections.                               |
| LLM_SERVER_WORKER_CONNECTIONS | no       | 1000             | no     | The maximum number of simultaneous clients.                              |

## Configuring Pathfinder

| Variables               | Required                                   | default | secret | Description                                                                                                                               |
| ----------------------- | ------------------------------------------ | ------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| AUTHENTICATION_KEY_HASH | Required if REQUIRE_AUTHENTICATION is true | N/A     | yes    | Hash to check Bearer tokens against when services are authenticating to the server. See section on Generating Keys below.                 |
| REQUIRE_AUTHENTICATION  | no                                         | N/A     | yes    | Whether to require services to authenticate to Pathfinder. May not be necessary if all services using Pathfinder are on the same network. |
| OPEN_AI_API_KEY         | yes                                        | N/A     | yes    | API key for OpenAI, obtained from [https://platform.openai.com/account/api-keys](https://platform.openai.com/account/api-keys).           |
| PORT                    | no                                         | 3030    | no     | The internal port to run Pathfinder on.                                                                                                   |

### Generating AUTHENTICATION_KEY_HASH & AUTHENTICATION_KEY for Pathfinder

Run the following to generate keys for Pathfinder

```bash
INTERNAL_KEY_BIN=$(openssl rand 32)
AUTHENTICATION_KEY=$(echo -n "$INTERNAL_KEY_BIN" | base64)
AUTHENTICATION_KEY_HASH=$(echo -n "$INTERNAL_KEY_BIN" | openssl dgst -binary -sha256 | openssl base64)
Cyan='\033[0;36m' # Cyan color
NC='\033[0m' # No Color
echo "\n- Set in Pathfinder environment:\n  AUTHENTICATION_KEY_HASH: $Cyan$AUTHENTICATION_KEY_HASH$NC"
echo "\n- Clients should pass this Bearer token in the HTTP authorization headers:\n  PATHFINDER_BEARER_TOKEN: $Cyan$AUTHENTICATION_KEY$NC\n\n  - For example:\n    { authorization: Bearer $AUTHENTICATION_KEY }"
```

### Configure a service to use Pathfinder

For any call to OpenAI, configure your service to use Pathfinder instead by using `<your.pathfinder.domain>/api/open-ai` instead of the OpenAI base host. Individual endpoints can be appended to this base. For example, `https://api.openai.com/v1/chat/completions` becomes `<your.pathfinder.domain>/api/open-ai/v1/chat/completions`.

If you set `REQUIRE_AUTHENTICATION` to `true` in your env file, you will also need to add an Authentication header to any API calls to Pathfinder. For example,

```javascript
{
  headers: {
    // AUTHENTICATION_KEY is the key output in the Generating Keys section above
    Authentication: "Bearer <AUTHENTICATION_KEY>";
  }
}
```
