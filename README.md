# GCP Production-Grade Deployments

A comprehensive collection of production-ready GitHub Actions CI/CD workflows for deploying AI/ML applications to Google Cloud Platform (GCP), including Cloud Run services, API Gateway, nightly batch jobs, and automated security scanning.

## Description

This repository provides battle-tested GitHub Actions workflow templates for production-grade GCP deployments. It covers the full software delivery lifecycle — from continuous integration (linting, unit tests, OpenAPI validation) to continuous deployment (containerized Cloud Run services, Cloud Run Jobs, and API Gateway management), along with security scanning via OWASP ZAP.

## Features

- **CI Pipeline** — Automated linting (Ruff), unit testing (pytest), and OpenAPI spec validation (Spectral) on every pull request
- - **CD Pipeline** — Build and push Docker images to Artifact Registry, then deploy multiple Cloud Run services and Cloud Run Jobs on every merge to `main`
  - - **Workload Identity Federation** — Keyless authentication to GCP using GitHub's OIDC token (no service account key files)
    - - **API Gateway Deployment** — Automated deployment and versioning of GCP API Gateway with JWT/OIDC authentication and CORS configuration
      - - **Nightly Cloud Run Job** — Scheduled (cron-based) execution of batch orchestration jobs at 2 AM ET daily
        - - **OWASP ZAP Baseline Scan** — Weekly automated API security scan against the deployed Gateway URL
          - - **Secret Management** — Secrets injected at runtime from GCP Secret Manager (no secrets in environment variables or code)
            - - **Multi-Service Deployment** — Simultaneous deployment of orchestrator API, chatbot API, and batch job containers
             
              - ## Architecture
             
              - ```
                GitHub PR → CI (Lint + Tests + OpenAPI Validation)
                             ↓
                GitHub Push to main → CD Pipeline
                             ↓
                        [Artifact Registry]
                        Docker image built & pushed
                             ↓
                    ┌────────────────────────────┐
                    │  Cloud Run - orchestrator  │
                    │  Cloud Run - chatbot-api   │
                    │  Cloud Run Job - nightly   │
                    └────────────────────────────┘
                             ↓
                    [API Gateway] ← JWT/OIDC + CORS
                             ↓
                    [OWASP ZAP Scan] (weekly)
                ```

                ## Workflows Included

                | Workflow | Trigger | Description |
                |---|---|---|
                | `ci.yml` | Pull Request to main/dev | Lint, test, validate OpenAPI spec |
                | `cd.yml` | Push to main | Build image, deploy Cloud Run services + Job |
                | `gateway-deploy.yml` | Push to main (openapi.yaml changed) | Deploy/update API Gateway config |
                | `nightly-job.yml` | Cron: 2 AM ET daily | Execute Cloud Run Job |
                | `zap-baseline.yml` | Cron: weekly / manual | OWASP ZAP API baseline security scan |

                ## Tech Stack

                | Component | Technology |
                |---|---|
                | CI/CD | GitHub Actions |
                | Container Registry | GCP Artifact Registry |
                | Compute | Google Cloud Run (Services + Jobs) |
                | API Management | GCP API Gateway |
                | Authentication | Workload Identity Federation (OIDC) |
                | Secret Storage | GCP Secret Manager |
                | Linting | Ruff (Python) |
                | Testing | pytest |
                | API Validation | Stoplight Spectral |
                | Security Scanning | OWASP ZAP |
                | Language | Python 3.11 |

                ## Setup Instructions

                ### Prerequisites

                1. A GCP project with the following APIs enabled:
                2.    - Cloud Run API
                      -    - Artifact Registry API
                           -    - API Gateway API
                                -    - Secret Manager API
                                     -    - IAM Credentials API (for Workload Identity Federation)
                                      
                                          - 2. A GitHub repository with Actions enabled.
                                           
                                            3. ### Step 1: Configure Workload Identity Federation
                                           
                                            4. ```bash
                                               # Create a Workload Identity Pool
                                               gcloud iam workload-identity-pools create "github-pool" \
                                                 --location="global" \
                                                 --display-name="GitHub Actions Pool"

                                               # Create a provider for GitHub
                                               gcloud iam workload-identity-pools providers create-oidc "github-provider" \
                                                 --location="global" \
                                                 --workload-identity-pool="github-pool" \
                                                 --display-name="GitHub Provider" \
                                                 --attribute-mapping="google.subject=assertion.sub,attribute.repository=assertion.repository" \
                                                 --issuer-uri="https://token.actions.githubusercontent.com"
                                               ```

                                               ### Step 2: Configure GitHub Secrets

                                               Add the following secrets to your GitHub repository settings:

                                               | Secret | Description |
                                               |---|---|
                                               | `GCP_WORKLOAD_IDP` | Workload Identity Provider resource name |
                                               | `GCP_SERVICE_ACCOUNT_EMAIL` | Service account email for deployments |
                                               | `GCP_PROJECT_ID` | Your GCP project ID |
                                               | `RAW_BUCKET` | GCS bucket for raw data |
                                               | `UNIFIED_BUCKET` | GCS bucket for processed data |
                                               | `OIDC_ISSUER` | Your OIDC identity provider issuer URL |
                                               | `OIDC_AUDIENCE` | OIDC client ID / audience |
                                               | `OIDC_JWKS_URI` | JWKS endpoint for token verification |
                                               | `ALLOWED_ORIGINS` | CORS allowed origins (comma-separated) |
                                               | `GATEWAY_BACKEND_SA` | Service account for API Gateway backend |
                                               | `OPENAI_API_KEY` | OpenAI API key (stored in Secret Manager) |
                                               | `LANGCHAIN_API_KEY` | LangChain API key (stored in Secret Manager) |

                                               ### Step 3: Create Artifact Registry Repository

                                               ```bash
                                               gcloud artifacts repositories create ai-images \
                                                 --repository-format=docker \
                                                 --location=us-central1 \
                                                 --description="AI application Docker images"
                                               ```

                                               ### Step 4: Deploy the Workflows

                                               Copy the workflow YAML files from this notebook into your `.github/workflows/` directory and push to the `main` branch.

                                               ### Step 5: Run the CI/CD Pipeline

                                               - Open a pull request to trigger the CI workflow
                                               - - Merge to `main` to trigger the full CD deployment
                                                 - - Monitor Cloud Run services in the GCP Console
                                                  
                                                   - ## Usage
                                                  
                                                   - Clone this repository and use the notebook as a reference for setting up your own CI/CD pipelines:
                                                  
                                                   - ```bash
                                                     git clone https://github.com/sanikacentric/GCP-_ProductionGrade-Deployments.git
                                                     cd GCP-_ProductionGrade-Deployments
                                                     jupyter notebook GCP__ProductionGrade_Deployments.ipynb
                                                     ```

                                                     ## License

                                                     Open source — for educational and cloud architecture demonstration purposes.
