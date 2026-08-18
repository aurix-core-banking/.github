# Aurix Core Banking

[![Backend](https://img.shields.io/badge/backend-java%2025%20%7C%20spring%20boot%204.1.0-blue)](https://github.com/aurix-core-banking/aurix-backend)
[![Frontend](https://img.shields.io/badge/frontend-react%2018%20%7C%20MUI%20v5-61dafb)](https://github.com/aurix-core-banking/aurix-frontend)
[![Open Finance](https://img.shields.io/badge/open%20finance-BACEN%20%7C%20FAPI--Brazil-00b945)](https://github.com/aurix-core-banking/aurix-openfinance)
[![Infrastructure](https://img.shields.io/badge/infra-terraform%20%7C%20kubernetes%20%7C%20docker-purple)](https://github.com/aurix-core-banking/aurix-infrastructure)
[![Tests](https://img.shields.io/badge/tests-playwright%20%7C%20REST%20Assured%20%7C%20k6-orange)](https://github.com/aurix-core-banking/aurix-tests)
[![ML](https://img.shields.io/badge/ml-xgboost%20%7C%20MLflow%20%7C%20FastAPI-red)](https://github.com/aurix-core-banking/aurix-ml)

---

Plataforma de core banking para o mercado brasileiro. Modular, event-driven, e construída para escala. Toda a base de código e documentação é em **português**.

## Arquitetura

```mermaid
graph TB
    EXT["🌐 Externo<br/>(BACEN / Receptor)"]
    GW["<b>API Gateway</b><br/>Traefik · /api/*"]

    subgraph BACKEND ["14 Microserviços · Java 25 · Spring Boot 4.1.0"]
        direction LR
        B1["svc-banking<br/>:<i>8200</i>"]
        B2["svc-payments<br/>:<i>8201</i>"]
        B3["svc-credit<br/>:<i>8082</i>"]
        B4["svc-customer<br/>:<i>8083</i>"]
        B5["svc-cards<br/>:<i>8094</i>"]
        B6["svc-products<br/>:<i>8084</i>"]
        B7["svc-compliance<br/>:<i>8205</i>"]
        B8["svc-platform<br/>:<i>8092</i>"]
        B9["svc-intelligence<br/>:<i>8091</i>"]
        B10["svc-finance-mgmt<br/>:<i>8089</i>"]
        B11["svc-ai<br/>:<i>8206</i>"]
        B12["svc-fraud<br/>:<i>8207</i>"]
        B13["svc-contracts<br/>:<i>8085</i>"]
        B14["svc-cambio<br/>:<i>8093</i>"]
    end

    subgraph OPENFINANCE ["svc-openfinance · :<i>8096</i>"]
        direction TB
        KF["Kafka Consumers"]
        REPLICAS["Réplicas Isoladas"]
        OFAPI["REST API · FAPI-Brazil"]
        DB_OF[("PostgreSQL<br/><code>aurix_openfinance</code>")]
    end

    EXT <-->|"REST<br/>banco receptor"| GW
    GW -->|"REST"| BACKEND
    BACKEND -->|"Kafka<br/>eventos de domínio"| KF
    KF --> REPLICAS --> DB_OF
    OFAPI <--> DB_OF
    GW -->|"REST<br/><code>/open-finance/*</code>"| OFAPI
```

## Open Finance Brasil

`svc-openfinance` expõe as APIs do **Open Finance Brasil** (BACEN) no padrão FAPI-Bridge. Dados do core banking são replicados via Kafka para tabelas isoladas — o core nunca é acessado diretamente por bancos receptores.

### APIs disponíveis

| Domínio | Endpoints |
|---|---|
| **Consentimentos** | `POST /consents`, `GET /consents/{id}`, `POST .../authorise`, `POST .../reject`, `POST .../revoke` |
| **Contas** | `GET /accounts`, `GET /accounts/{id}`, `GET /accounts/{id}/balances` |
| **Transações** | `GET /accounts/{id}/transactions` |
| **Dados pessoais** | `GET /customers/personal/identifications`, `.../addresses`, `.../phone-numbers`, `.../email`, `GET /customers/business/identifications`, `.../addresses` |
| **Cartões de crédito** | `GET /credit-cards`, `GET /credit-cards/{id}`, `GET /credit-cards/{id}/bills`, `GET /credit-cards/{id}/bills/{billId}`, `GET /credit-cards/{id}/transactions` |
| **Empréstimos** | `GET /loans`, `GET /loans/{id}` |
| **Seguros** | `GET /insurance`, `GET /insurance/{id}` |
| **PIX** | `GET /pix/credit-transfers`, `GET /pix/credit-transfers/{id}` |

### Como funciona

1. **Ingestão**: Microserviços publicam eventos Kafka (`core.conta.criada.v1`, `cartoes.transacao.autorizada.v1`, etc.)
2. **Consumo**: `svc-openfinance` consome os eventos e filtra por consentimentos ativos
3. **Replicação**: Dados são escritos em tabelas isoladas (`aurix_openfinance`), com limpeza automática de consentimentos expirados
4. **Consulta**: Banco receptor consulta via REST com token de consentimento

## Ecossistema

| Repositório | Descrição | Stack |
|---|---|---|
| **[aurix-backend](https://github.com/aurix-core-banking/aurix-backend)** | 14 microserviços: contas, transações, PIX, crédito, câmbio, cartões, compliance, fraude, IA. API REST, eventos Kafka, PostgreSQL compartilhado. | Java 25, Spring Boot 4.1.0, Maven |
| **[aurix-openfinance](https://github.com/aurix-core-banking/aurix-openfinance)** | APIs Open Finance Brasil (BACEN) — FAPI-Bridge. Consome eventos Kafka, mantém réplicas isoladas para exposição segura. | Java 25, Spring Boot 4.1.0, Flyway, Kafka |
| **[aurix-frontend](https://github.com/aurix-core-banking/aurix-frontend)** | Internet banking (React), admin (React Admin), mobile (React Native). | React 18, MUI v5, React Native 0.73 |
| **[aurix-data-pipelines](https://github.com/aurix-core-banking/aurix-data-pipelines)** | ETL/streaming: PostgreSQL → Bronze → Silver → Gold. Orquestração Airflow. | PySpark, Flink, dbt, Airflow |
| **[aurix-data-platform](https://github.com/aurix-core-banking/aurix-data-platform)** | Data lake: ClickHouse (OLAP), TimescaleDB (séries temporais), Elasticsearch (buscas). | ClickHouse, TimescaleDB, Elasticsearch |
| **[aurix-ml](https://github.com/aurix-core-banking/aurix-ml)** | Scoring de crédito, detecção de fraude, previsão de default, segmentação. Serving FastAPI, governança por regime. | Python, XGBoost, MLflow, FastAPI |
| **[aurix-infrastructure](https://github.com/aurix-core-banking/aurix-infrastructure)** | Docker Compose (dev), Terraform (multi-cloud), Kubernetes (Helm + ArgoCD), monitoramento. | Terraform, K8s, Helm, Istio |
| **[aurix-tests](https://github.com/aurix-core-banking/aurix-tests)** | E2E (Playwright), integração (REST Assured), performance (k6). | Python, Playwright, k6 |
| **[aurix-docs](https://github.com/aurix-core-banking/aurix-docs)** | ADRs, guias, runbooks, specs de design, planos de implementação. | — |

## Stack

| Camada | Tecnologias |
|---|---|
| Backend | Java 25, Spring Boot 4.1.0, Spring Cloud 2025.1.2, Maven multi-module |
| Frontend | React 18, MUI v5, React Native 0.73, react-router-dom v6 |
| Open Finance | FAPI-Bridge, Flyway, Springdoc OpenAPI |
| Data | PySpark, Apache Flink, dbt, Airflow, Kafka Streams |
| ML | scikit-learn, XGBoost, MLflow, FastAPI |
| Infra | Terraform (AWS/Azure/GCP), Kubernetes, Helm, Docker Compose, Istio |
| Observabilidade | Prometheus, Grafana, Elasticsearch, Kibana |
| Banco de dados | PostgreSQL 15, ClickHouse, TimescaleDB, Redis 7 |
| Mensageria | Apache Kafka + Zookeeper (61+ tópicos domain-driven) |
| Segurança | Keycloak 23 (OIDC/FAPI), mTLS, API keys, OAuth2 |
| Testes | Playwright (E2E), REST Assured (integração), k6 (performance) |

## Início rápido

```bash
git clone git@github.com:aurix-core-banking/aurix-core-banking.git
cd aurix-core-banking
make clone-all

cd aurix-infrastructure && docker-compose up -d
cd ../aurix-backend && ./mvnw clean install -DskipTests
./mvnw spring-boot:run -pl svc-banking
```

---

**Organização**: [aurix-core-banking](https://github.com/aurix-core-banking) · **Versão**: 2.0.0
