📁 banking-microplatform-demo/ – Project Root
banking-microplatform-demo/
├── services/
│   ├── user-service/
│   ├── account-service/
│   ├── transaction-service/
│   ├── audit-service/
│   └── shared-lib/
│
├── aws/
│   ├── lambda/
│   │   └── customer-upload-processor/
│   ├── stepfunctions/
│   ├── s3/
│   └── terraform/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf
│
├── rosa/
│   ├── kafka/                # Strimzi setup
│   ├── keycloak/             # Realm, clients
│   ├── infinispan/           # Role cache
│   ├── postgres/             # External RDS configuration files (if needed)
│   ├── secrets/              # OpenShift Secrets YAMLs
│   └── manifests/            # K8s YAMLs for service deployments
│
├── ui/
│   └── streamlit-ui/
│       ├── app.py
│       └── requirements.txt
│
├── gateway/
│   └── kong-config/          # Or AWS API Gateway Swagger spec
│
├── docs/
│   ├── requirements.md
│   ├── architecture.png
│   └── flow-diagrams/
│
├── .github/
│   └── workflows/            # GitHub Actions CI/CD per module
│
├── .gitignore
└── README.md

----

Per-Folder Scaffolding

✅ 1. services/ (Spring Boot microservices)

<service-name>/
├── src/
│   ├── main/
│   │   ├── java/com/mikcore/<servicename>/
│   │   └── resources/
│   └── test/
├── Dockerfile
├── pom.xml
└── README.md
All services share shared-lib/ for DTOs, constants, logging setup, and utility classes.

✅ 2. aws/
lambda/ – Java-based Lambda for parsing S3 uploads

stepfunctions/ – ASL definitions for Step Functions

s3/ – Bucket policy templates

terraform/ – All provisioning logic for:

S3 bucket

Lambda

Step Functions

IAM roles

Secret Manager

VPC peering (if required)

✅ 3. rosa/
kafka/ – Strimzi CRDs for Kafka cluster and topics

keycloak/ – Realm export and client config

infinispan/ – StatefulSet or Operator-based Infinispan deployment

postgres/ – External DB secret injectors if needed

secrets/ – .yaml files for Kafka creds, RDS creds, etc.

manifests/ – Service Deployment, Service, Ingress, ConfigMap

✅ 4. ui/streamlit-ui/
app.py – Upload UI + View audit logs

Connects to S3 via signed URL or API Gateway

Retrieves JWT token via Keycloak

Sends metadata/events to API Gateway

✅ 5. gateway/
Kong or AWS API Gateway config

Handles routing + JWT verification

Optional Swagger import for route config

✅ 6. .github/workflows/
Each microservice has its own CI/CD pipeline:

name: Build and Push account-service

on:
  push:
    paths:
      - 'services/account-service/**'

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: Build Docker Image
        run: |
          docker build -t <registry>/account-service:latest services/account-service
      - name: Push Image
        run: docker push <registry>/account-service:latest

----

🧰 Tooling + Common Tech Stack
Tool	Purpose
Spring Boot (Java 21)	API services
Terraform	AWS infra deployment
Strimzi	Kafka on OpenShift
Keycloak	Auth (OAuth2/JWT)
Infinispan	Role cache
PostgreSQL (RDS)	Core DB
Streamlit	Minimal UI
Swagger (springdoc-openapi)	API docs
logback + MDC	Logging with request context

🔲 Aligned with Your Goals
Goal	Met?	Notes
Modular Spring Boot microservices	✅	services/ with one folder per service and shared-lib
Kafka inside ROSA	✅	Strimzi manifests in rosa/kafka/
AWS S3 + Lambda + Step Functions	✅	Java Lambda + Step Functions defined in aws/ with Terraform
PostgreSQL (RDS) as external DB	✅	Integrated into Lambda and microservices, secrets managed via Terraform and OpenShift
Infinispan in ROSA for Keycloak role cache	✅	rosa/infinispan/ covers StatefulSet or Operator deployment
Terraform-based AWS Infra	✅	Defined in aws/terraform/ to manage S3, Lambda, IAM, Secrets
API Gateway setup (Kong/AWS)	✅	Configs in gateway/ (support for Swagger-based routing)
CI/CD via GitHub Actions	✅	.github/workflows/ scaffold per microservice
UI for interaction	✅	ui/streamlit-ui/ for S3 upload, Keycloak login, audit viewer
Secure design (mTLS, JWT, RBAC, Secrets)	✅	Enforced across all layers via Keycloak, OpenShift Secrets, and IAM roles
Logging and auditing	✅	audit_log RDS table + Kafka audit events + structured logback

----

🧩 Modular Coverage
Microservices
user-service: Auth entry point

account-service: Core domain logic (create/delete accounts)

transaction-service: Ingested events and data

audit-service: Central logging of CRUD operations

shared-lib: DTOs, common config

AWS Data Processing Pipeline
Upload → S3 → Step Function → Lambda → RDS Insert + Kafka publish

ROSA-native Components
Kafka, Infinispan, Keycloak managed with OpenShift YAMLs

Infra-as-Code
Terraform handles:

S3 buckets

Lambda + Step Functions

IAM roles/policies

AWS Secrets Manager



