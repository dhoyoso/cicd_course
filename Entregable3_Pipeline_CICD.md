# Pipeline CI/CD – Entregable 3

Diagrama del workflow `.github/workflows/ci-cd.yml`: 5 jobs en cadena (CI + despliegue en Staging y Producción con Terraform sobre AWS ECS).

## Vista general

```mermaid
flowchart TD
    T1([push a main<br/>excepto solo .md o docs]) --> J1
    T2([pull_request hacia main]) --> J1
    T3([workflow_dispatch]) --> J1

    subgraph J1 [Job 1: build-and-test]
        direction TB
        A1[Checkout + Python 3.13<br/>+ dependencias] --> A2{Black --check}
        A2 -- OK --> A3[Pylint y Flake8<br/>--exit-zero]
        A3 --> A4{Unit + Componente<br/>cobertura ≥ 80%}
        A4 -- Pasa --> A5{Terraform<br/>fmt -check + validate}
        A5 -- OK --> A6[Docker build<br/>tag = github.sha]
        A6 --> A7[Start container<br/>espera /health]
        A7 --> A8{Pruebas de aceptación<br/>Selenium contra contenedor}
        A8 -- Pasa --> A9[Upload reports<br/>if: always]
        A9 --> A10{SonarCloud<br/>Quality Gate}
        A10 -- Passed --> A11{¿ref = main?}
        A11 -- Sí --> A12[Login Docker Hub<br/>Push :sha y :latest<br/>output: image_uri]
    end

    A2 -- Falla --> FAIL
    A4 -- Falla --> FAIL
    A5 -- Falla --> FAIL
    A8 -- Falla --> FAIL
    A10 -- Failed --> FAIL
    A11 -- No: Pull Request --> PRDONE([CI en verde<br/>sin publicar ni desplegar])

    A12 --> J2

    subgraph J2 [Job 2: deploy-staging]
        direction TB
        B1[Configure AWS credentials] --> B2[terraform init<br/>workspace staging]
        B2 --> B3[terraform apply<br/>imagen :sha + app_version]
        B3 --> B4[terraform output<br/>alb_url]
    end

    B3 -- Falla --> FAIL
    B4 --> J3

    subgraph J3 [Job 3: test-staging]
        direction TB
        C1[Espera que /health<br/>devuelva el SHA<br/>máx. 5 min] --> C2{Pruebas de aceptación<br/>Selenium contra Staging}
        C2 --> C3[Upload staging report]
    end

    C1 -- Timeout --> FAIL
    C2 -- Falla --> FAIL
    C3 --> GATE

    GATE{{Aprobación manual<br/>environment: production<br/>Required reviewers}}
    GATE -- Aprobado --> J4
    GATE -- Rechazado --> STOP([Producción sin cambios])

    subgraph J4 [Job 4: deploy-production]
        direction TB
        D1[Configure AWS credentials] --> D2[terraform init<br/>workspace production]
        D2 --> D3[terraform apply<br/>misma imagen :sha]
        D3 --> D4[terraform output<br/>alb_url]
    end

    D3 -- Falla --> FAIL
    D4 --> J5

    subgraph J5 [Job 5: smoke-test-production]
        direction TB
        E1[Espera que /health<br/>devuelva el SHA<br/>máx. 5 min] --> E2{Pruebas de humo<br/>EXPECTED_VERSION = SHA}
        E2 --> E3[Upload smoke report]
    end

    E1 -- Timeout --> FAIL
    E2 -- Falla --> FAIL
    E3 --> DONE([Versión en Producción<br/>verificada])

    FAIL([Pipeline falla<br/>jobs siguientes omitidos])

    classDef gate fill:#fff3cd,stroke:#b58900,color:#000;
    classDef fail fill:#f8d7da,stroke:#c0392b,color:#000;
    classDef ok fill:#d4edda,stroke:#2e7d32,color:#000;
    classDef manual fill:#d1ecf1,stroke:#0c5460,color:#000;
    class A2,A4,A5,A8,A10,A11,C2,E2 gate;
    class FAIL fail;
    class PRDONE,DONE ok;
    class GATE,STOP manual;
```

## Cadena de jobs

```mermaid
flowchart LR
    BT[build-and-test<br/>CI + publica imagen] --> DS[deploy-staging<br/>Terraform workspace staging]
    DS --> TS[test-staging<br/>Aceptación Selenium]
    TS --> DP[deploy-production<br/>Terraform workspace production<br/>aprobación manual]
    BT -. image_uri .-> DP
    DP --> SP[smoke-test-production<br/>Pruebas de humo]

    classDef ci fill:#e3f2fd,stroke:#1565c0,color:#000;
    classDef cd fill:#e8f5e9,stroke:#2e7d32,color:#000;
    class BT ci;
    class DS,TS,DP,SP cd;
```

## Secuencia de ejecución

```mermaid
sequenceDiagram
    actor Dev as Desarrollador
    participant GH as GitHub Actions
    participant S as SonarCloud
    participant H as Docker Hub
    participant TF as Terraform (estado en S3)
    participant AWS as AWS ECS + ALB
    actor Rev as Revisor (production)

    Dev->>GH: Merge de PR a main (push)
    Note over GH: Job 1: build-and-test
    GH->>GH: Black, Pylint, Flake8, pytest (≥ 80%)
    GH->>GH: terraform fmt y validate
    GH->>GH: docker build + Selenium contra contenedor local
    GH->>S: Scan + Quality Gate
    S-->>GH: Passed
    GH->>H: Push :sha y :latest
    H-->>GH: image_uri

    Note over GH: Job 2: deploy-staging
    GH->>TF: init + workspace staging + apply (imagen :sha)
    TF->>AWS: Nueva Task Definition + actualiza servicio
    AWS-->>GH: alb_url staging

    Note over GH: Job 3: test-staging
    loop hasta 30 intentos, cada 10 s
        GH->>AWS: GET /health (¿devuelve el SHA?)
    end
    GH->>AWS: Selenium contra Staging

    Note over GH: Job 4: deploy-production
    GH->>Rev: Solicita aprobación (environment production)
    Rev-->>GH: Aprueba
    GH->>TF: init + workspace production + apply (misma imagen :sha)
    TF->>AWS: Despliegue rolling con circuit breaker
    AWS-->>GH: alb_url production

    Note over GH: Job 5: smoke-test-production
    loop hasta 30 intentos, cada 10 s
        GH->>AWS: GET /health (¿devuelve el SHA?)
    end
    GH->>AWS: Pruebas de humo (/ y /health con EXPECTED_VERSION)
    GH-->>Dev: Pipeline en verde
```

## Resumen de jobs

| # | Job | Depende de | Se ejecuta en | Qué valida o hace |
|---|---|---|---|---|
| 1 | `build-and-test` | — | push, PR y manual | Calidad, pruebas, Terraform fmt/validate, aceptación en contenedor, Quality Gate y publicación de la imagen (solo `main`) |
| 2 | `deploy-staging` | 1 | Solo `main` | `terraform apply` en workspace `staging` con la imagen `:sha` |
| 3 | `test-staging` | 2 | Solo `main` | Espera la versión nueva y ejecuta pruebas de aceptación contra Staging |
| 4 | `deploy-production` | 1 y 3 | Solo `main` | Aprobación manual (7.11) y `terraform apply` en workspace `production` con la misma imagen |
| 5 | `smoke-test-production` | 4 | Solo `main` | Espera la versión nueva y ejecuta pruebas de humo con `EXPECTED_VERSION` |

## Política de fallos

| Paso | ¿Detiene el pipeline? | Se ejecuta en |
|---|---|---|
| Black (`--check`) | Sí | push y PR |
| Pylint y Flake8 | No por hallazgos (`--exit-zero`) | push y PR |
| Pruebas unitarias y de componente (≥ 80%) | Sí | push y PR |
| Terraform `fmt -check` y `validate` | Sí | push y PR |
| Build y arranque del contenedor | Sí | push y PR |
| Pruebas de aceptación en el contenedor (Selenium) | Sí | push y PR |
| SonarCloud Quality Gate | Sí | push y PR |
| Login y push a Docker Hub | — | Solo `main` (push y manual) |
| `terraform apply` en Staging | Sí | Solo `main` |
| Espera de `/health` y aceptación en Staging | Sí | Solo `main` |
| Aprobación manual de `production` | Si se rechaza, Producción queda sin cambios | Solo `main` |
| `terraform apply` en Producción | Sí | Solo `main` |
| Espera de `/health` y pruebas de humo en Producción | Sí | Solo `main` |
| Show container logs | — | Solo si hay fallo (`failure()`) |
| Upload de reportes | — | Siempre (`always()`) |

## Configuración requerida en el repositorio

| Tipo | Nombre | Para qué se usa |
|---|---|---|
| Secreto | `SONAR_TOKEN` | Análisis de SonarCloud |
| Secreto | `DOCKERHUB_TOKEN` | Push de la imagen |
| Secretos | `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_SESSION_TOKEN` | `terraform apply` en los dos ambientes |
| Variable | `DOCKERHUB_USERNAME` | Nombre de la imagen y login |
| Variable | `SONAR_HOST_URL` | `https://sonarcloud.io` |
| Ambiente | `production` con **Required reviewers** | Aprobación manual (sección 7.11) |
| Ruleset | `proteger-main` | 1 aprobación y check `build-and-test` |

## Detalles clave

- **Misma imagen en todo el flujo:** se construye una vez, se prueba en el runner, se publica con el tag `github.sha` y esa misma imagen se despliega en Staging y Producción.
- **`concurrency` con `cancel-in-progress: false`:** una ejecución nueva espera a la anterior, para no interrumpir un `terraform apply`.
- **Aprobación manual (sección 7.11):** convierte el Despliegue Continuo en Entrega Continua. Staging sigue automático y Producción espera a los revisores.
- **Rollback (sección 8):** ECS usa circuit breaker. Si el despliegue falla, revierte a la versión anterior, y `/health` sigue mostrando el SHA viejo, lo que hace fallar el paso de espera.
- **Política de fallos:** si un job falla, los siguientes se omiten. Los pasos `failure()` y `always()` (logs y reportes) sí se ejecutan.
