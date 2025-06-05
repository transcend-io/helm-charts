# Deploying Sombra and Pathfinder

The following example adds Pathfinder to the Kubernetes deployment.

```yaml
imageCredentials:
  registry: docker.transcend.io
  username: Transcend
  password: '<TRANSCEND_API_TOKEN>'

transcend_service:
  type: NodePort

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
    value: '<ORGANIZATION_URI>'
  - name: EMPLOYEE_AUTHENTICATION_METHODS
    value: 'transcend,session'
  - name: DATA_SUBJECT_AUTHENTICATION_METHODS
    value: 'transcend,session'

envs_as_secret:
  - name: INTERNAL_KEY_HASH
    value: '<INTERNAL_KEY_HASH>'
  - name: JWT_ECDSA_KEY
    value: '<JWT_ECDSA_KEY>'
  - name: INTERNAL_KEY
    value: '<INTERNAL_KEY>'

pathfinder:
  enabled: true
  envs_as_secret:
    - name: OPEN_AI_API_KEY
      value: '<OPEN_AI_API_KEY>'
    - name: AUTHENTICATION_KEY_HASH
      value: '<AUTHENTICATION_KEY_HASH>'
    - name: TRANSCEND_API_KEY
      value: '<TRANSCEND_API_KEY>'

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
    Authentication: 'Bearer <AUTHENTICATION_KEY>';
  }
}
```
