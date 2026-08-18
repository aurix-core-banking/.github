# Aurix Core Banking

[![Backend](https://img.shields.io/badge/backend-java%2025%20%7C%20spring%20boot%204.1.0-blue)](https://github.com/aurix-core-banking/aurix-backend)
[![Frontend](https://img.shields.io/badge/frontend-react%2018%20%7C%20MUI%20v5-61dafb)](https://github.com/aurix-core-banking/aurix-frontend)
[![Open Finance](https://img.shields.io/badge/open%20finance-BACEN%20Fase%201-00b945)](https://github.com/aurix-core-banking/aurix-openfinance)
[![Infrastructure](https://img.shields.io/badge/infra-terraform%20%7C%20kubernetes%20%7C%20docker-purple)](https://github.com/aurix-core-banking/aurix-infrastructure)
[![Tests](https://img.shields.io/badge/tests-playwright%20%7C%20REST%20Assured%20%7C%20k6-orange)](https://github.com/aurix-core-banking/aurix-tests)
[![ML](https://img.shields.io/badge/ml-xgboost%20%7C%20MLflow%20%7C%20FastAPI-red)](https://github.com/aurix-core-banking/aurix-ml)

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
| **Fase 1** | Contas, transações, saldos, consentimento, dados pessoais (identificação, endereços, contatos) | ✅ Implementado |
| **Fase 2** | Cartões de crédito | 🚧 Placeholder |
| **Fase 3** | Pix, empréstimos, seguros | 📋 Planejado |

### Fase 1 — Endpoints

| Domínio | Endpoints | Kafka Consumers |
|---|---|---|
| Consentimento | `POST /consents`, `GET /consents/{id}`, `POST .../authorise`, `POST .../reject`, `POST .../revoke` | — |
| Contas | `GET /accounts`, `GET /accounts/{id}`, `GET /accounts/{id}/balances` | `core.conta.criada.v1`, `core.conta.atualizada.v1` |
| Transações | `GET /accounts/{id}/transactions` | `core.transacao.realizada.v1`, `core.transacao.liquidada.v1` |
| Dados pessoais | `GET /customers/personal/identifications`, `.../addresses`, `.../phone-numbers`, `.../email`, `GET /customers/business/identifications`, `.../addresses` | `customer.cliente.criado.v1`, `customer.cliente.atualizado.v1` |
| Cartões | `GET /credit-cards` (Fase 2) | `cartoes.transacao.autorizada.v1` |

### Fluxo de dados

```
svc-banking / svc-cards / svc-customer
         │ Kafka events
         ▼
  svc-openfinance (consumidor)
         │ filtra por consentimentos ativos
         │ escreve em réplicas isoladas
         ▼
  PostgreSQL (aurix_openfinance)
         │
         ▼ REST (banco receptor consulta)
  Externo (BACEN / receptor)
```

### Infraestrutura

| Componente | Configuração |
|---|---|
| Porta | `8096` |
| Docker Compose | `svc-openfinance` com health check |
| Traefik | Rota `/open-finance/*` → `svc-openfinance:8096` |
| Prometheus | Scrape config em `monitoring/prometheus.yml` |
| Helm | `kubernetes/charts/svc-openfinance` (deployment, service, hpa, pdb) |
| OpenAPI | `/swagger-ui.html` · Spec: `openapi/aurix-openfinance.yaml` |
| Flyway | V1 (consentimento, contas, transações) + V2 (pessoas) |
| Limpeza | `@Scheduled` expira consentimentos + apaga réplicas (1h) |

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
