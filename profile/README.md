# Aurix Core Banking

[![Backend CI](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/backend-ci.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/backend-ci.yml)
[![Frontend CI](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/frontend-ci.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/frontend-ci.yml)
[![Infrastructure](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/terraform-plan.yml/badge.svg)](https://github.com/aurix-core-banking/aurix-core-banking/actions/workflows/terraform-plan.yml)

---

Plataforma completa de core banking para o mercado brasileiro. Moderna, modular, e construída para escala.

## Ecossistema

A Aurix é dividida em componentes independentes, cada um em seu próprio repositório:

### Core

- **[aurix-backend](https://github.com/aurix-core-banking/aurix-backend)** — Microserviços em Java 21 com Spring Boot. API REST, eventos assíncronos via Kafka, PostgreSQL por domínio.
- **[aurix-frontend](https://github.com/aurix-core-banking/aurix-frontend)** — Interface web em React + TypeScript. Painel administrativo, portal do cliente, e aplicativo mobile (React Native).

### Dados e IA

- **[aurix-data-pipelines](https://github.com/aurix-core-banking/aurix-data-pipelines)** — Pipelines ETL e streaming com PySpark, Kafka e Airflow. Processamento de transações financeiras em tempo real.
- **[aurix-data-platform](https://github.com/aurix-core-banking/aurix-data-platform)** — Data lake e analytics com ClickHouse e dbt. Base para relatórios, dashboards e consultas OLAP.
- **[aurix-ml](https://github.com/aurix-core-banking/aurix-ml)** — Modelos de machine learning para scoring de crédito, detecção de fraude e análise de risco. Treinamento com Python/XGBoost, tracking com MLflow.

### Operações

- **[aurix-infrastructure](https://github.com/aurix-core-banking/aurix-infrastructure)** — Infraestrutura como código (Terraform + Kubernetes). Provisionamento de cloud, orquestração de containers, monitoramento com Prometheus/Grafana.
- **[aurix-api-specs](https://github.com/aurix-core-banking/aurix-api-specs)** — Especificações OpenAPI 3.x que definem os contratos entre frontend, backend e integradores terceiros.
- **[aurix-tests](https://github.com/aurix-core-banking/aurix-tests)** — Testes end-to-end (Playwright), integração (REST Assured) e performance (k6) que cruzam toda a plataforma.

### Conhecimento

- **[aurix-docs](https://github.com/aurix-core-banking/aurix-docs)** — Documentação técnica: arquitetura, ADRs, guias de desenvolvimento, runbooks operacionais e muito mais.

## Stack

| Categoria | Tecnologias |
|---|---|
| Backend | Java 21, Spring Boot, Maven, PostgreSQL, Kafka |
| Frontend | React, TypeScript, Vite, Tailwind CSS |
| Data | PySpark, Apache Kafka, Airflow, ClickHouse, dbt |
| ML | Python, XGBoost, LightGBM, MLflow |
| Infra | Terraform, Kubernetes, Docker, Helm, Prometheus |
| API | OpenAPI 3.x, REST, AsyncAPI |

## Início rápido

```bash
# Clone este meta-repositório
git clone git@github.com:aurix-core-banking/aurix-core-banking.git
cd aurix-core-banking

# Clone todos os componentes
make clone-all

# Sobe infraestrutura local
make infra-up

# Compila e roda
make build-backend
make build-frontend
```

## Documentação

Toda a documentação consolidada está em [`docs/`](./docs/), incluindo visão geral da arquitetura, guias de desenvolvimento, runbooks de operação e Architecture Decision Records (ADRs).

---

**Organização**: [aurix-core-banking](https://github.com/aurix-core-banking) &middot; **Versão**: 2.0.0
