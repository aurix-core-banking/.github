# Aurix Core Banking

[![Backend CI](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/backend-ci.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/backend-ci.yml)
[![Frontend CI](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/frontend-ci.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/frontend-ci.yml)
[![Infrastructure](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/terraform-plan.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/terraform-plan.yml)

---

Plataforma completa de core banking para o mercado brasileiro. Modular, event-driven, e construída para escala. Toda a base de código e documentação é em **português**.

## Ecossistema

A Aurix é dividida em repositórios independentes, cada um responsável por uma camada da plataforma:

### Core

| Repositório | Descrição | Stack |
|---|---|---|
| **[aurix-backend](https://github.com/aurix-core-banking/aurix-backend)** | 14 microserviços Spring Boot (contas, transações, PIX, crédito, câmbio, cartões, compliance, fraude, IA, etc.). API REST, eventos via Kafka, banco único compartilhado (PostgreSQL). | Java 25, Spring Boot 4.1.0, Spring Cloud 2025.1.2, Maven |
| **[aurix-openfinance](https://github.com/aurix-core-banking/aurix-openfinance)** | Microserviço dedicado ao Open Finance Brasil (BACEN). Consome eventos do core banking via Kafka, mantém réplicas de leitura isoladas para exposição segura a bancos receptores. APIs de consentimento, contas, transações e cartões (Fase 1 + Fase 2). | Java 25, Spring Boot 4.1.0, PostgreSQL, Flyway, Kafka, Springdoc |
| **[aurix-frontend](https://github.com/aurix-core-banking/aurix-frontend)** | Interface web (React + MUI), painel administrativo (React Admin) e app mobile (React Native). | React 18, MUI v5, React Native 0.73, CRA |

### Dados e IA

| Repositório | Descrição | Stack |
|---|---|---|
| **[aurix-data-pipelines](https://github.com/aurix-core-banking/aurix-data-pipelines)** | Pipelines ETL/streaming: ingestão PostgreSQL → Bronze, processamento Spark/Flink, transformação dbt (Bronze → Silver → Gold), orquestração Airflow. | PySpark, Apache Flink, dbt, Airflow, Kafka |
| **[aurix-data-platform](https://github.com/aurix-core-banking/aurix-data-platform)** | Data lake e analytics: ClickHouse para OLAP, TimescaleDB para séries temporais, Elasticsearch para buscas. Replicação real-time do core banking. | ClickHouse, TimescaleDB, Elasticsearch |
| **[aurix-ml](https://github.com/aurix-core-banking/aurix-ml)** | Modelos de ML: scoring de crédito (XGBoost), detecção de fraude, previsão de default, segmentação de clientes. Serving via FastAPI, governança por regime (R1-R3). | Python, scikit-learn, XGBoost, MLflow, FastAPI |

### Operações

| Repositório | Descrição | Stack |
|---|---|---|
| **[aurix-infrastructure](https://github.com/aurix-core-banking/aurix-infrastructure)** | Infraestrutura como código: Docker Compose (dev local), Terraform (multi-cloud AWS/Azure/GCP), Kubernetes (Helm charts + ArgoCD), monitoramento (Prometheus + Grafana). | Terraform, Kubernetes, Docker, Helm, Istio |
| **[aurix-tests](https://github.com/aurix-core-banking/aurix-tests)** | Testes E2E (Playwright), integração (REST Assured) e performance (k6). Cobertura cross-service que valida a plataforma inteira. | Python, Playwright, REST Assured, k6 |

### Conhecimento

| Repositório | Descrição |
|---|---|
| **[aurix-docs](https://github.com/aurix-core-banking/aurix-docs)** | Documentação técnica: visão geral da arquitetura, ADRs (Architecture Decision Records), guias de desenvolvimento, runbooks operacionais, checklists de completude, specs de design e planos de implementação. |

## Arquitetura

```
┌─────────────────────────────────────────────────────────────────┐
│                        API Gateway (Traefik)                     │
│                     /api/* → roteamento por serviço               │
└─────────┬───────────────────────────────────────────┬───────────┘
          │ REST (norte-sul)                          │
┌─────────▼───────────────────────────────────────────▼───────────┐
│                                                                   │
│  svc-banking ──┐   svc-payments ─┐   svc-credit ─┐               │
│  svc-customer ─┤   svc-cards ────┤   svc-cambio ─┤               │
│  svc-fraud ────┤   svc-platform ─┤   svc-ai ─────┤  14 serviços │
│  svc-compliance┤   svc-products ─┤   svc-contracts┤               │
│  svc-intelligence  svc-finance-mgmt              │               │
│                                                                   │
└──────────────────────────┬──────────────────────────────────────┘
                           │ Kafka (leste-oeste, eventos de domínio)
                           │ Topics: core.conta.criada.v1,
                           │         core.transacao.realizada.v1,
                           │         cartoes.transacao.autorizada.v1,
                           │         contracts.contrato.assinado.v1, ...
                           ▼
┌─────────────────────────────────────────────────────────────────┐
│                     svc-openfinance (porta 8096)                 │
│                                                                   │
│  Kafka Consumer → filtra por consentimentos ativos                │
│                 → escreve em réplicas isoladas                    │
│                 → expira e limpa dados automaticamente            │
│                                                                   │
│  APIs REST: /open-finance/v1/consents                             │
│             /open-finance/v1/accounts                             │
│             /open-finance/v1/accounts/{id}/transactions           │
│             /open-finance/v1/credit-cards (Fase 2)                │
│                                                                   │
│  PostgreSQL: aurix_openfinance (réplica isolada do core)          │
└──────────────────────────┬──────────────────────────────────────┘
                           │ REST (banco receptor consulta)
                           ▼
                    ┌──────────────┐
                    │  Externo     │
                    │  (BACEN /    │
                    │   receptor)  │
                    └──────────────┘
```

## Stack

| Camada | Tecnologias |
|---|---|
| Backend | Java 25, Spring Boot 4.1.0, Spring Cloud 2025.1.2, Maven multi-module |
| Frontend | React 18, MUI v5, React Native 0.73, react-router-dom v6 |
| Open Finance | Spring Boot 4.1.0, Spring Security (FAPI-Brazil), Flyway, Springdoc OpenAPI |
| Data | PySpark, Apache Flink, dbt, Airflow, Kafka Streams |
| ML | Python, scikit-learn, XGBoost, MLflow, FastAPI |
| Infra | Terraform (AWS/Azure/GCP), Kubernetes, Helm, Docker Compose, Istio |
| Observabilidade | Prometheus, Grafana, Elasticsearch, Kibana |
| Banco de dados | PostgreSQL 15, ClickHouse, TimescaleDB, Redis 7 |
| Mensageria | Apache Kafka + Zookeeper (61+ tópicos domain-driven) |
| Segurança | Keycloak 23 (OIDC/FAPI), mTLS, API keys, OAuth2 |
| Testes | Playwright (E2E), REST Assured (integração), k6 (performance) |

## Repositórios

| Repo | Porta(s) | Descrição curta |
|---|---|---|
| `aurix-backend` | 8080-8207 | 14 microserviços Spring Boot |
| `aurix-openfinance` | 8096 | APIs Open Finance Brasil (BACEN) |
| `aurix-frontend` | 3000/3001 | Web, admin, mobile |
| `aurix-data-pipelines` | — | ETL e streaming |
| `aurix-data-platform` | 8123/9002 | ClickHouse, Elasticsearch |
| `aurix-ml` | 8300 | Modelos de ML + serving |
| `aurix-infrastructure` | — | Terraform, K8s, Docker |
| `aurix-tests` | — | E2E, integração, performance |
| `aurix-docs` | — | Documentação técnica |

## Início rápido

```bash
# Clone o monorepo
git clone git@github.com:aurix-core-banking/aurix-core-banking.git
cd aurix-core-banking

# Clone todos os componentes
make clone-all

# Sobe infraestrutura local (PostgreSQL, Kafka, Redis, Keycloak, etc.)
cd aurix-infrastructure && docker-compose up -d

# Compila o backend
cd aurix-backend && ./mvnw clean install -DskipTests

# Roda um serviço específico
./mvnw spring-boot:run -pl svc-banking

# Roda o Open Finance
cd ../aurix-openfinance && ./mvnw spring-boot:run -pl svc-openfinance
```

## Open Finance

O `aurix-openfinance` implementa as APIs do **Open Finance Brasil** (BACEN) seguindo o padrão FAPI-Brazil:

| Fase | Escopo | Status |
|---|---|---|
| **Fase 1** | Contas, transações, saldos, consentimento | ✅ Implementado |
| **Fase 2** | Cartões de crédito | 🚧 Placeholder |
| **Fase 3** | Pix, empréstimos, seguros | 📋 Planejado |

**Fluxo de dados:**
1. Usuário autoriza consentimento via `/open-finance/v1/consents`
2. `svc-openfinance` consome eventos Kafka do core banking (contas, transações, cartões)
3. Dados são filtrados por consentimentos ativos e armazenados em réplicas isoladas
4. Banco receptor consulta via REST — nunca acessa o core banking diretamente
5. Consentimentos expiram automaticamente e dados são limpos (job a cada 1h)

**Porta:** `8096` · **OpenAPI:** `/swagger-ui.html` · **Spec:** `openapi/aurix-openfinance.yaml`

## Documentação

Toda a documentação consolidada está em [`aurix-docs`](https://github.com/aurix-core-banking/aurix-docs), incluindo:

- Visão geral da arquitetura
- Architecture Decision Records (ADRs)
- Guias de desenvolvimento
- Runbooks operacionais
- Specs de design e planos de implementação
- Checklists de completude

---

**Organização**: [aurix-core-banking](https://github.com/aurix-core-banking) · **Versão**: 2.0.0
