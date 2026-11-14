# Pipeline de CI – Entregable 2

Diagrama del workflow `.github/workflows/ci.yml` (job `build-and-test`).

## Vista general

```mermaid
flowchart TD
    T1([push a main]) --> START
    T2([pull_request hacia main]) --> START
    T3([workflow_dispatch]) --> START
    START[Inicio del job<br/>build-and-test<br/>ubuntu-latest]

    START --> A1[Checkout<br/>fetch-depth: 0]
    A1 --> A2[Set up Python 3.13]
    A2 --> A3[Install dependencies<br/>requirements-dev.txt]

    A3 --> B1{Black --check<br/>¿formato OK?}
    B1 -- No --> FAIL
    B1 -- Sí --> B2[Pylint<br/>--exit-zero<br/>pylint-report.txt]
    B2 --> B3[Flake8<br/>--exit-zero<br/>flake8-report.txt]

    B3 --> C1{Pruebas unitarias y de componente<br/>pytest + cobertura ≥ 80%}
    C1 -- Falla --> FAIL
    C1 -- Pasa --> C2[Genera coverage.xml<br/>report.html y htmlcov]

    C2 --> D1[Build Docker image<br/>IMAGE_NAME:github.sha]
    D1 --> D2[Start container<br/>puerto 8000<br/>espera hasta 30 s]
    D2 --> D3{Pruebas de aceptación<br/>Selenium contra el contenedor}
    D3 -- Falla --> FAIL

    D3 -- Pasa --> E1[Upload Test Reports<br/>if: always]
    E1 --> E2{SonarCloud Scan<br/>¿Quality Gate Passed?}
    E2 -- No --> FAIL

    E2 -- Sí --> F1{¿push a main?}
    F1 -- No: es Pull Request --> OKPR([Pipeline en verde<br/>sin publicar imagen])
    F1 -- Sí --> G1[Login to Docker Hub]
    G1 --> G2[Tag latest + Push<br/>SHA y latest]
    G2 --> OKMAIN([Imagen publicada<br/>en Docker Hub])

    FAIL[Pipeline falla]
    FAIL -.-> DIAG[Show container logs<br/>if: failure]
    FAIL -.-> UP[Upload Test Reports<br/>if: always]
    FAIL --> BLOCK([Merge bloqueado<br/>Imagen NO publicada])

    classDef gate fill:#fff3cd,stroke:#b58900,color:#000;
    classDef fail fill:#f8d7da,stroke:#c0392b,color:#000;
    classDef ok fill:#d4edda,stroke:#2e7d32,color:#000;
    class B1,C1,D3,E2,F1 gate;
    class FAIL,BLOCK fail;
    class OKPR,OKMAIN ok;
```

## Secuencia de ejecución

```mermaid
sequenceDiagram
    actor Dev as Desarrollador
    participant GH as GitHub
    participant R as Runner (ubuntu-latest)
    participant D as Contenedor Docker
    participant S as SonarCloud
    participant H as Docker Hub

    Dev->>GH: push a main / abre o actualiza PR
    GH->>R: Dispara CI Pipeline (build-and-test)
    R->>R: Checkout + Python 3.13 + dependencias
    R->>R: Black --check
    R->>R: Pylint y Flake8 (reportes, --exit-zero)
    R->>R: pytest unit + componente (cobertura ≥ 80%)
    R->>R: docker build (tag = github.sha)
    R->>D: docker run -p 8000:8000
    R->>D: Selenium (APP_BASE_URL=localhost:8000)
    D-->>R: Resultados de aceptación
    R->>GH: Upload artifacts (reportes)
    R->>S: Scan + reportes (pylint, flake8, coverage.xml)
    S-->>R: Quality Gate (Passed / Failed)
    alt push a main y todo en verde
        R->>H: Login + push :sha y :latest
        H-->>R: Imagen publicada
    else Pull Request o fallo previo
        R-->>GH: Sin publicación
    end
    R-->>GH: Estado del check build-and-test
```

## Política de fallos

| Paso | ¿Detiene el pipeline? | Se ejecuta en |
|---|---|---|
| Black (`--check`) | Sí | push y PR |
| Pylint y Flake8 | No por hallazgos (`--exit-zero`) | push y PR |
| Pruebas unitarias y de componente (≥ 80%) | Sí | push y PR |
| Build y arranque del contenedor | Sí | push y PR |
| Pruebas de aceptación (Selenium) | Sí | push y PR |
| SonarCloud Quality Gate | Sí | push y PR |
| Login y push a Docker Hub | — | Solo push a `main` |
| Show container logs | — | Solo si hay fallo (`failure()`) |
| Upload Test Reports | — | Siempre (`always()`) |
