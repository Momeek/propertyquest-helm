# PropertyQuest Helm Chart!S

Helm chart for deploying the PropertyQuest client and server application on Kubernetes.

## Prerequisites

- Kubernetes cluster
- Helm 3+
- AWS Load Balancer Controller (for ingress)
- External Secrets Operator
- AWS Secrets Manager secret at path `propertyquest/application`
- AWS RDS MySQL instance

## Chart Structure

```
helm/propertyquest/
├── templates/
│   ├── Serverdeploy.yaml       # Backend deployment
│   ├── ServerSvc.yaml          # Backend service
│   ├── Clientdeploy.yaml       # Frontend deployment
│   ├── Clientsvc.yaml          # Frontend service
│   ├── Clientingress.yaml      # ALB ingress
│   ├── migration-job.yaml      # DB migration pre-install/upgrade job
│   ├── secret-store.yaml       # AWS Secrets Manager store
│   └── external-secret.yaml    # External secret synced from AWS
├── Chart.yaml
└── values.yaml
```

## Secrets

Secrets are managed via the External Secrets Operator, pulling from AWS Secrets Manager at `propertyquest/application` (region: `us-east-1`). The following keys are expected:

| Key | Description |
|-----|-------------|
| `db-password` | Database password |
| `google_client_secret` | Google OAuth client secret |
| `fb_client_secret` | Facebook OAuth client secret |
| `apple_private_key` | Apple Sign-In private key |
| `admin_secret_key` | Admin secret key |
| `admin_password` | Admin account password |

## Configuration

Key values in `values.yaml`:

| Parameter | Description | Default |
|-----------|-------------|---------|
| `server.image` | Backend image | ECR image URI |
| `server.replicas` | Backend replica count | `1` |
| `server.containerPort` | Backend container port | `8080` |
| `client.image` | Frontend image | ECR image URI |
| `client.replicas` | Frontend replica count | `2` |
| `client.containerPort` | Frontend container port | `3000` |
| `database.host` | RDS host | RDS endpoint |
| `database.port` | Database port | `3306` |
| `database.name` | Database name | `propertyquest` |
| `database.user` | Database user | `propertyquest_app` |
| `database.ssl` | Enable SSL connection to DB | `true` |
| `database.sslRejectUnauthorized` | Reject unauthorized SSL certs | `false` |
| `ingress.enabled` | Enable ALB ingress | `true` |
| `ingress.host` | Ingress hostname | `propertyquest.meeklab.site` |
| `migration.enabled` | Run DB migrations on deploy | `true` |

## Install

```bash
helm install propertyquest ./helm/propertyquest -n propertyquest --create-namespace
```

## Upgrade

```bash
helm upgrade propertyquest ./helm/propertyquest -n propertyquest
```

## Database Migrations

Migrations run automatically as a Helm pre-install/pre-upgrade Job using `sequelize-cli db:migrate`. Set `migration.enabled: false` to skip.

## Ingress

The ingress uses AWS ALB (internet-facing) with HTTPS redirect. HTTP traffic on port 80 is redirected to HTTPS on port 443 using an ACM certificate.
