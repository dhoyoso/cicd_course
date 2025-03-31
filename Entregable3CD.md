# Taller Grupal Entregable 3: Despliegue Continuo (CD) con Proveedores Cloud

Este taller es una continuación del Taller 2, donde construimos un pipeline de Integración Continua (CI). Ahora, nos enfocaremos en la **Entrega Continua (Continuous Delivery)**, automatizando el despliegue de nuestra aplicación Python primero a un entorno de **Staging** para pruebas de aceptación, y luego a un entorno de **Producción** en un proveedor de nube.

## 1. Conceptos de Entrega Continua y Despliegue Continuo (CD)

* **Entrega Continua (Continuous Delivery):** Práctica donde los cambios de código se construyen, prueban y preparan *automáticamente* para producción, desplegándose automáticamente a un entorno de **Staging**. La *liberación* a **Producción** requiere una **aprobación manual** (o un trigger deliberado). El objetivo es tener *siempre* una versión validada lista para ir a producción rápidamente.

* **Despliegue Continuo (Continuous Deployment):** Va un paso más allá. *Cada cambio* que pasa *todas* las etapas automatizadas (incluyendo pruebas en Staging) se despliega **automáticamente a Producción** sin intervención humana. Requiere una confianza extrema en la automatización.  **Este es el enfoque que implementaremos principalmente en este taller.**

**Flujo de trabajo General:**

1.  **CI (Integración Continua):** Código -> Build -> Pruebas (Unit, Integ) -> Análisis -> Artefacto. (Cubierto en Taller 2).
2.  **CD (Este Taller - Deployment con Staging):** Artefacto -> **Deploy Staging (Auto)** -> **Pruebas Aceptación en Staging (Auto)** -> **Deploy Producción (Auto)** -> **Pruebas Humo en Producción (Auto)** -> Monitoreo.

## 2. Diferencias entre Continuous Deployment y Continuous Delivery

| Característica         | Continuous Delivery                      | Continuous Deployment                          |
| :--------------------- | :--------------------------------------- | :--------------------------------------------- |
| **Despliegue a Prod.** | Manual (requiere aprobación)             | Automático (sin intervención manual)           |
| **Frecuencia Despl.** | Controlada (puede ser frecuente)         | Muy frecuente (cada cambio validado)           |
| **Confianza Automat.** | Alta                                     | **Extrema** (requiere pruebas/monitoreo robustos) |
| **Velocidad (Lead Time)**| Más lento                                | Más rápido                                     |
| **Intervención Humana**| Sí (para liberar a Prod)                 | No (idealmente)                                |
| **Feedback Producción**| Retrasado por aprobación                 | Muy rápido (directamente de producción)        |

La elección depende del contexto. En este taller implementaremos un flujo más cercano a Continuous Deployment al introducir un entorno de Staging y un despliegue automático en Producción.  Empresas como Netflix, Amazon y Facebook utilizan Despliegue Continuo, mientras que otras prefieren la Entrega Continua para tener un mayor control.

## 3. Introducción a un Proveedor de Nube con Capa Gratuita

Para este taller, utilizaremos **Render** como nuestro proveedor de nube por su simplicidad y capa gratuita, ideal para este ejercicio de aprendizaje. Recuerda que entornos productivos reales suelen usar **AWS, GCP o Azure**.

**¿Por qué Render para este taller?**

* Facilidad de uso.
* Soporte Python y Gunicorn.
* Despliegue automático desde GitHub.
* Capa gratuita adecuada.
* Manejo implícito de contenedores.

**Pasos para Render:**

1.  **Crea una cuenta en Render:** Ve a [https://render.com/](https://render.com/) y regístrate con tu cuenta de GitHub.
2.  **Conecta tu cuenta de GitHub:**  Render te pedirá permiso para acceder a tus repositorios de GitHub.  Esto es necesario para el despliegue automático.
3.  **IMPORTANTE:** Para este taller, necesitarás crear **DOS** servicios web en Render: uno para **Staging** y otro para **Producción**.

## 4. CI/CD en Arquitecturas Modernas

### 4.1 Contenedores (Docker)

Los contenedores, como Docker, son una pieza clave de las arquitecturas modernas de CI/CD.  Un contenedor empaqueta una aplicación y todas sus dependencias (bibliotecas, runtime, configuración) en una unidad estandarizada que se puede ejecutar de forma consistente en cualquier entorno (desarrollo, pruebas, producción).

**Ventajas de los contenedores:**

*   **Consistencia:**  El mismo contenedor se ejecuta igual en cualquier lugar.
*   **Aislamiento:**  Los contenedores están aislados entre sí y del sistema host.
*   **Portabilidad:**  Se pueden mover fácilmente entre diferentes entornos y proveedores de nube.
*   **Eficiencia:**  Son más ligeros que las máquinas virtuales.
*   **Escalabilidad:**  Se pueden crear y destruir rápidamente para escalar la aplicación.

**Docker en CI/CD:**

1.  **Dockerfile:**  Un archivo de texto que define cómo construir una imagen de Docker (la "receta" del contenedor).
2.  **Construcción de la imagen:**  En el pipeline de CI, se construye la imagen de Docker a partir del Dockerfile.
3.  **Registro de la imagen:**  La imagen se sube a un registro de contenedores (Docker Hub, Google Container Registry, Amazon ECR, etc.).
4.  **Despliegue:**  El proveedor de nube descarga la imagen del registro y ejecuta el contenedor.

### 4.2 Kubernetes

Kubernetes (K8s) es un sistema de orquestación de contenedores.  Administra, escala y despliega aplicaciones en contenedores.  Es ideal para aplicaciones complejas con múltiples servicios.

**Conceptos clave de Kubernetes:**

*   **Pods:**  La unidad más pequeña en Kubernetes.  Un pod contiene uno o más contenedores que comparten recursos (red, almacenamiento).
*   **Servicios (Services):**  Una forma de exponer una aplicación que se ejecuta en un conjunto de pods como un servicio de red.
*   **Despliegues (Deployments):**  Describen el estado deseado de una aplicación (qué imagen de Docker usar, cuántas réplicas, etc.).  Kubernetes se encarga de mantener ese estado.
*   **Nodos (Nodes):**  Las máquinas (físicas o virtuales) donde se ejecutan los pods.
*   **Clúster (Cluster):**  Un conjunto de nodos gestionados por Kubernetes.

**Kubernetes en CI/CD:**

1.  **Manifiestos YAML:**  Archivos que describen los recursos de Kubernetes (pods, servicios, despliegues, etc.).
2.  **Despliegue:**  El pipeline de CI aplica los manifiestos YAML a un clúster de Kubernetes.  Kubernetes se encarga de crear/actualizar los recursos.
3.  **Rollouts y Rollbacks:**  Kubernetes permite actualizaciones graduales (rollouts) y reversiones (rollbacks) fáciles.

### 4.3 Serverless

Serverless es un modelo de computación en la nube donde el proveedor de nube gestiona la infraestructura (servidores, escalado, etc.).  El desarrollador solo se preocupa por el código (funciones).

**Conceptos clave de Serverless:**

*   **Funciones (Functions):**  Pequeñas unidades de código que se ejecutan en respuesta a eventos (ej: peticiones HTTP, mensajes en una cola, cambios en una base de datos).
*   **Eventos (Events):**  Lo que desencadena la ejecución de una función.
*   **Pago por uso:**  Solo se paga por el tiempo de cómputo que consume la función.
*   **Escalado automático:**  El proveedor de nube escala automáticamente las funciones según la demanda.

**Serverless en CI/CD:**

1.  **Frameworks Serverless:**  Herramientas como Serverless Framework, AWS SAM, etc., simplifican el desarrollo y despliegue de aplicaciones serverless.
2.  **Despliegue:**  El pipeline de CI despliega las funciones y configura los eventos que las desencadenan.

**Nota:** **En este taller específico**, desplegaremos directamente código Python a Render (que usa contenedores internamente pero lo abstrae). No gestionaremos Dockerfiles, K8s o FaaS explícitamente.

## 5. Despliegue Continuo: Automatización Completa y Rollbacks

### 5.1 Automatización Completa

El objetivo del Despliegue Continuo es automatizar *todo* el proceso, desde el commit hasta la producción.  Esto implica:

*   **Desencadenar el despliegue automáticamente:**  Cada vez que se hace un push a la rama principal (o se aprueba un pull request), el pipeline de CI/CD se ejecuta y, si todo va bien, despliega la nueva versión.
*   **Despliegue sin intervención manual:**  No hay pasos manuales en el proceso de despliegue.
*   **Pruebas exhaustivas:**  Es fundamental tener un conjunto completo de pruebas automatizadas (unitarias, de integración, de aceptación) para garantizar que el código que se despliega es de alta calidad.
*   **Monitoreo continuo:**  Es esencial monitorear la aplicación en producción para detectar problemas rápidamente.

### 5.2 Rollbacks

Un rollback es la capacidad de volver a una versión anterior de la aplicación en caso de que algo salga mal después de un despliegue.  Es una parte crucial del Despliegue Continuo.

**Estrategias de rollback:**

*   **Redespliegue de la versión anterior:**  La forma más sencilla es volver a desplegar la versión anterior de la aplicación.  Esto requiere que se conserve la versión anterior (ej: etiqueta en Git, imagen de Docker anterior).
*   **Rollback de la base de datos (si es necesario):**  Si el despliegue incluye cambios en la base de datos, puede ser necesario revertir esos cambios también.  Esto puede ser complejo y requiere una planificación cuidadosa.
*  **Blue/Green Deployments**: Se mantienen dos entornos idénticos (azul y verde). Uno está activo (azul) y el otro inactivo (verde). Se despliega en el entorno inactivo, se prueba y luego se cambia el tráfico al entorno verde. Si hay problemas se puede volver al azul rápidamente.
*   **Canary Deployments:** Se despliega una nueva versión a un pequeño subconjunto de usuarios.  Si todo va bien, se despliega gradualmente a más usuarios.  Si hay problemas, se revierte.

**Nota sobre Render y Rollbacks:** Render facilita volver a desplegar un commit anterior desde su UI, lo que sirve como un rollback manual simple para este taller.

## 6. Importancia del Monitoreo en Despliegues Continuos

El monitoreo es *crítico* en un entorno de Despliegue Continuo.  Dado que los cambios se despliegan automáticamente, es fundamental detectar problemas lo antes posible.

**¿Qué monitorear?**

*   **Métricas de la aplicación:**
    *   Tiempo de respuesta (latencia).
    *   Tasa de errores.
    *   Número de peticiones por segundo.
    *   Uso de recursos (CPU, memoria, disco, red).
    *   Métricas de negocio (ej: número de usuarios registrados, ventas por hora).
*   **Logs:**  Registros de eventos de la aplicación.  Útiles para diagnosticar problemas.
*   **Disponibilidad (uptime):**  Asegurarse de que la aplicación esté disponible y respondiendo.
*   **Alertas:**  Configurar alertas para ser notificado cuando algo va mal (ej: alta tasa de errores, tiempo de respuesta lento, uso excesivo de recursos).

El monitoreo es **CRÍTICO**. Permite validar despliegues, detectar regresiones, tomar decisiones (rollback) y entender el impacto. Monitorear: Métricas Clave, Logs, Disponibilidad (Health Checks), Alertas.

## 7. Herramientas de Observabilidad, Monitoreo - Health - Pruebas de Humo (Smoke Tests)

### 7.1 Observabilidad

La observabilidad es la capacidad de entender el estado interno de un sistema a partir de sus salidas externas.  Se basa en tres pilares:

*   **Métricas:**  Valores numéricos que se miden a lo largo del tiempo (ej: tiempo de respuesta, tasa de errores).
*   **Logs:**  Registros de eventos que ocurren en el sistema.
*   **Traza (Tracing):**  Seguimiento del flujo de una petición a través de los diferentes componentes de un sistema distribuido.

### 7.2 Monitoreo

El monitoreo es el proceso de recopilar y analizar datos de observabilidad para detectar problemas y tomar decisiones.

### 7.3 Health Checks

Un health check es una verificación simple que indica si una aplicación está funcionando correctamente. Normalmente, es un endpoint HTTP (ej: `/health`) que devuelve un código de estado 200 OK si la aplicación está sana, y un código de error (ej: 500) si no lo está.

**Implementación de un health check en Flask:**

```python
# app/app.py (añadir a tu app existente)
@app.route("/health")
def health():
    """Health Check Endpoint."""
    return "OK", 200
```

**Añade este health check a tu aplicación Flask.**  Render y otros proveedores de nube pueden usar este endpoint para verificar si tu aplicación está funcionando. **Además lo utilizaremos más adelante.**

Este endpoint `/health` simplemente devuelve "OK" con un código de estado 200. Render, y otros proveedores de nube, pueden usar este endpoint para verificar si tu aplicación está funcionando. Si el endpoint no responde o devuelve un código de error, el proveedor puede tomar medidas (ej: reiniciar la aplicación, escalar horizontalmente, etc.).

### 7.4 Pruebas de Humo (Smoke Tests)
Las pruebas de humo son un conjunto pequeño de pruebas que verifican la funcionalidad *básica* de una aplicación después de un despliegue. Son una forma rápida de detectar problemas graves. No reemplazan a las pruebas unitarias, de integración o de aceptación, sino que las complementan.  Las pruebas de humo se ejecutan *después* del despliegue para asegurar que la aplicación está, al menos, "funcionando de manera básica".

**Ejemplo de prueba de humo:**
```python
# tests/test_smoke_app.py (crea este archivo nuevo en tests/)
import os
from selenium.webdriver.common.by import By

def test_smoke_test(browser):
    """SMOKE TEST: Verifica carga básica y título."""
    app_url = os.environ.get("APP_URL", "http://localhost:5000")
    browser.get(app_url + "/")
    assert "Calculadora" in browser.title
    h1_element = browser.find_element(By.TAG_NAME, "h1")
    assert h1_element.text == "Calculadora"
```

**Añade esta prueba de humo a tu aplicación Flask.**. **Lo utilizaremos más adelante.**

Esta prueba de humo simplemente verifica que la página principal de la calculadora se cargue correctamente y que el título sea "Calculadora". Si esta prueba de humo falla, *después de un despliegue*, significa que la aplicación no está funcionando en absoluto, o que algo muy básico está roto.

**Es importante ejecutar las pruebas de humo *después* de cada despliegue.** En un pipeline de CD, las pruebas de humo serían un paso *posterior* al despliegue.

**Clarificación Importante:**
* **Health Check:** ¿Está vivo/responde? (Nivel Infra/Proceso)
* **Smoke Test:** ¿Funciona lo más básico? (Nivel Funcional Crítico Post-Deploy)
* **Acceptance Test:** ¿Cumple los requisitos/historias de usuario? (Nivel Funcional Completo)

## 8. Diferencias entre Entornos On-Premise, Cloud e Híbridos
*   **On-Premise:**  La infraestructura (servidores, red, almacenamiento) se encuentra en las instalaciones de la empresa, y la empresa es responsable de *todo* el hardware y software.

    *   **Ventajas:**  Mayor control, seguridad (potencialmente, si se gestiona bien), cumplimiento normativo (en algunos casos, es más fácil cumplir con regulaciones on-premise).

    *   **Desventajas:**  Mayor costo inicial (hardware, licencias), mayor costo operativo (mantenimiento, personal de IT, electricidad, refrigeración), menor escalabilidad (es difícil escalar rápidamente), mayor tiempo de despliegue (aprovisionar hardware lleva tiempo).

*   **Cloud:**  La infraestructura se encuentra en los centros de datos del proveedor de nube (ej: AWS, Google Cloud, Azure, Render). El proveedor es responsable de la infraestructura física y, en muchos casos, de parte del software.

    *   **Ventajas:**  Menor costo (pago por uso, no hay inversión inicial en hardware), mayor escalabilidad (se puede escalar rápidamente hacia arriba o hacia abajo), menor tiempo de despliegue (aprovisionar recursos en la nube es rápido), acceso a servicios gestionados (bases de datos, colas de mensajes, etc.).

    *   **Desventajas:**  Menor control (sobre la infraestructura), dependencia del proveedor (vendor lock-in), seguridad (depende del proveedor, aunque los proveedores suelen tener altos estándares de seguridad).

*   **Híbrido:**  Una combinación de on-premise y cloud.  Algunos recursos se ejecutan en la infraestructura de la empresa, y otros en la nube.

    *   **Ventajas:**  Flexibilidad, permite aprovechar lo mejor de ambos mundos (ej: mantener datos sensibles on-premise y usar la nube para escalar), puede ser una estrategia de transición a la nube.

    *   **Desventajas:**  Mayor complejidad de gestión (hay que gestionar dos entornos diferentes), puede ser más difícil de asegurar (hay que asegurar la conexión entre los dos entornos).

## 9. Implementación del Pipeline con Staging y Producción en Render

Ahora, vamos a implementar el Despliegue Continuo de nuestra aplicación a Render. Usaremos la integración de Render con GitHub para que cada push a la rama `main` desencadene un nuevo despliegue.

1.  **Modifica `app/app.py` para usar un puerto configurable (necesario para Render):**

    Asegúrate que `app.py` usa puerto configurable (`os.environ.get("PORT", 5000)`) y tiene `debug=False` en `app.run`.

    Asegúrate de tener el endpoint `/health` en `app.py`.

    Asegúrate de tener las pruebas de aceptación y humo en `tests/test_acceptance_app.py` y `tests/test_smoke_app.py` respectivamente para verificar el funcionamiento de la aplicación después del despliegue en **staging** y **producción** respectivamente.

    ```python
    # app/app.py
    from flask import Flask, render_template, request
    from .calculadora import (
        sumar,
        restar,
        multiplicar,
        dividir,
    )  # Importa desde el mismo paquete
    import os

    app = Flask(__name__)


    @app.route("/", methods=["GET", "POST"])
    def index():
        resultado = None
        if request.method == "POST":
            try:
                num1 = float(request.form["num1"])
                num2 = float(request.form["num2"])
                operacion = request.form["operacion"]

                if operacion == "sumar":
                    resultado = sumar(num1, num2)
                elif operacion == "restar":
                    resultado = restar(num1, num2)
                elif operacion == "multiplicar":
                    resultado = multiplicar(num1, num2)
                elif operacion == "dividir":
                    resultado = dividir(num1, num2)
                else:
                    resultado = "Operación no válida"
            except ValueError:
                resultado = "Error: Introduce números válidos"
            except ZeroDivisionError:
                resultado = "Error: No se puede dividir por cero"

        return render_template("index.html", resultado=resultado)

    @app.route("/health")
    def health():
        return "OK", 200

    if __name__ == "__main__":
        port = int(os.environ.get("PORT", 5000))  # Obtiene el puerto de la variable de entorno PORT, o usa 5000 por defecto
        app.run(host='0.0.0.0', port=port, debug=False) # Se quita el debug para producción.
    ```

    **Explicación:**
    *   `import os`: Importamos el módulo `os` para acceder a las variables de entorno.
    *   `port = int(os.environ.get("PORT", 5000))`:  Intentamos obtener el valor de la variable de entorno `PORT`.  Si la variable `PORT` existe (Render la define), la usamos.  Si no existe, usamos el puerto 5000 por defecto (para desarrollo local).
    *   `app.run(host='0.0.0.0', port=port, debug=False)`:  Ejecutamos la aplicación en el host `0.0.0.0` (para que sea accesible desde fuera del contenedor) y en el puerto especificado.  `debug=False` es importante para producción.

2.  **Añade al archivo `requirements.txt`:**
    
    Asegúrate que `requirements.txt` incluye `flask` y `gunicorn`.
    ```
    pylint
    flake8
    black
    pytest
    coverage
    selenium
    webdriver-manager
    flask
    pytest-cov
    pytest-html
    gunicorn #Nuevo
    ```

    **Importante:** Agrega `gunicorn` a tu `requirements.txt`.  Gunicorn es un servidor WSGI HTTP para aplicaciones Python.  Es necesario para ejecutar aplicaciones Flask en producción (el servidor de desarrollo de Flask no es adecuado para producción).

3.  **Crea un archivo `Procfile` en la raíz de tu repositorio:**

    Asegúrate que `Procfile` contiene `web: gunicorn app.app:app`.
    ```
    web: gunicorn app.app:app
    ```
    Este archivo le dice a Render cómo ejecutar tu aplicación.  `web:` indica que es un servicio web.  `gunicorn app.app:app` le dice a Gunicorn que ejecute la aplicación `app` que se encuentra en el archivo `app.py`.

4.  **Crea DOS nuevos servicios web en Render:**
    *   Ve a tu panel de control de Render ([https://dashboard.render.com/](https://dashboard.render.com/)).
    *   Haz clic en "New Web Service".
    *   Conecta tu repositorio de GitHub (`cicd-pipeline-python` o el nombre de tu repo).
    *   En "Name", dale un nombre a tu servicio (ej: `mi-calculadora-staging`).
    *   En "Branch", asegúrate de que esté seleccionada la rama `main` (desplegaremos `main` a staging primero).
    *   En "Language", selecciona "Python 3".
    *   En "Region", deja el valor por defecto.
    *   En "Root Directory", deja el valor por defecto (vacío).
    *   En "Build Command", deja el valor por defecto (`pip install -r requirements.txt`).
    *   En "Start Command", pon `gunicorn app.app:app`.
    *   Selecciona el plan "Free".
    *   Deja la sección "Environment Variables" vacía.
    *   Haz clic en "Deploy Web Service".
    *   El deploy debería comenzar automáticamente pero fallará porque no hemos hecho el commit con los cambios necesarios.
    *   Repite el proceso para crear otro servicio, pero esta vez nómbralo `mi-calculadora` (o como prefieras) y asegúrate de que el "Branch" esté en `main` también. Este será tu servicio de producción.

5.  **Obtén el ID del servicio y la clave API de Render:**
    *   **Service ID:**  De la URL de la página de configuración de tu servicio en Render, obtén el ID del servicio que aparece después de web/ (ej: `srv-xxxxxxxxxxxxxxxxxxxx`). *Copia este ID*.
    *   **Nota:**  Repite el proceso para ambos servicios (staging y producción).  Necesitarás el ID del servicio de ambos.
    *   **API Key:**
        *   Ve a tu configuración de cuenta de Render (Account Settings arriba a la derecha).
        *   Ve a la sección "API Keys".
        *   Haz clic en "Create API Key".
        *   Dale un nombre a la clave (ej: "GitHub Actions").
        *   *Copia la clave API*.

6.  **Crea los secretos en GitHub:**
    *   Ve a tu repositorio en GitHub (`cicd-pipeline-python`) -> Settings -> Secrets and variables -> Actions -> New repository secret.
    *   Crea *cinco* secretos:
        *   **`RENDER_API_KEY`:**  Pega la clave API que copiaste de Render.
        *   **`RENDER_SERVICE_ID`:**  Pega el ID del servicio **productivo** que copiaste de Render.
        *   **`RENDER_APP_URL`:** Pega la URL completa (https://...) de tu app **productiva**.
        *   **`RENDER_SERVICE_ID_STAGING`:**  Pega el ID del servicio de **staging** que copiaste de Render.
        *   **`RENDER_APP_URL_STAGING`:** Pega la URL completa (https://...) de tu app **Staging**.

7.  **Modifica y renombra tu archivo `ci.yml` existente a `ci-cd.yml` y modificalo para incluir el despliegue a Render:**

    La mejor práctica, es mover las pruebas de aceptación *después* del despliegue en **staging**, y que se ejecuten *contra* la aplicación desplegada en Render. Esto asegura que estás probando un entorno idéntico de producción. Adicionalmente, las pruebas de humo se ejecutan después del despliegue en producción.

    ** NOTA: ** No olvides cambiar el puerto en la prueba de aceptación para que use la URL de Staging.  Puedes usar `os.environ.get("APP_URL", "http://localhost:5000")` para obtener la URL de Staging desde las variables de entorno.

    ```yaml
    name: CI/CD Pipeline 

    on:
      push:
        branches:
          - main
      pull_request:
        branches:
          - main
      workflow_dispatch:

    jobs:
      build-and-test:
        runs-on: ubuntu-latest  # Usamos Ubuntu

        steps:
          - uses: actions/checkout@v3

          - name: Set up Python
            uses: actions/setup-python@v3
            with:
              python-version: '3.12' # Reemplaza con tu versión de Python

          - name: Install dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt

          - name: Run Black (Formatter)
            run: black app --check  # --check solo verifica, no modifica

          - name: Run Pylint (Linter)
            run: pylint app --output-format=text --fail-under=9 > pylint-report.txt || true

          - name: Run Flake8 (Linter)
            run: flake8 app --output-file=flake8-report.txt || true

          - name: Run Unit Tests with pytest and Coverage
            run: |
              pytest --cov=app tests/ --ignore=tests/test_acceptance_app.py --ignore=tests/test_smoke_app.py # Genera un informe XML para SonarCloud
              # No olvides ignorar las pruebas de aceptación y humo aquí.

          # Se eliminan las pruebas de aceptación y humo de aquí.
            
          - name: SonarCloud Scan
            uses: SonarSource/sonarqube-scan-action@v5.0.0
            env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Automáticamente proporcionado
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}   # El secreto que creaste


      # --- Job de Despliegue a Staging ---
      deploy-staging:
        needs: build-and-test # Depende de CI
        runs-on: ubuntu-latest

        # Solo ejecuta en push a main (no en PRs)
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'

        steps:
          - uses: actions/checkout@v3
          - name: Deploy to Staging Environment
            uses: johnbeynon/render-deploy-action@v0.0.8 # Accion oficial de Render.
            with:
              service-id: ${{ secrets.RENDER_SERVICE_ID_STAGING }}  # Secreto de Render para Staging.
              api-key: ${{ secrets.RENDER_API_KEY }}    # Secreto de Render
              wait-for-success: true # Espera a que el despliegue termine.

      # --- Job de Pruebas de Aceptación en Staging ---
      test-staging:
        needs: deploy-staging # Depende del despliegue a Staging
        runs-on: ubuntu-latest

        # Solo ejecuta en push a main
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'

        steps:
          - uses: actions/checkout@v3

          - name: Set up Python
            uses: actions/setup-python@v3
            with:
              python-version: '3.12' # Reemplaza con tu versión de Python

          - name: Install test dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt # Instala dependencias de test (selenium, pytest)

          - name: Run Acceptance Tests against Staging
            env:
              APP_URL: ${{ secrets.RENDER_APP_URL_STAGING }} # URL de Staging
            run: |
              # Ejecuta TODAS las pruebas de aceptación/humo definidas en el archivo
              pytest tests/test_acceptance_app.py

      # --- Job de Despliegue a Producción ---
      deploy-production:
        needs: test-staging # Depende de las pruebas en Staging
        runs-on: ubuntu-latest

        # Solo ejecuta en push a main (no en PRs)
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'

        steps:
          - uses: actions/checkout@v3
          - name: Deploy to Render

            uses: johnbeynon/render-deploy-action@v0.0.8 # Accion oficial de Render.
            with:
              service-id: ${{ secrets.RENDER_SERVICE_ID }}  # Secreto de Render
              api-key: ${{ secrets.RENDER_API_KEY }}    # Secreto de Render
              wait-for-success: true # Espera a que el despliegue termine.

      # --- Job de Pruebas de Humo en Producción ---
      smoke-test-production:
        needs: deploy-production # Depende del despliegue a Producción
        runs-on: ubuntu-latest

        # Solo ejecuta en push a main
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'

        steps:
          - uses: actions/checkout@v3
          - name: Set up Python
            uses: actions/setup-python@v3
            with:
              python-version: '3.12'

          - name: Install dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt

          - name: Run Smoke Tests against Production
            env:
              APP_URL: ${{ secrets.RENDER_APP_URL }} # URL de Producción
            run: |
              pytest tests/test_smoke_app.py # Ejecuta las pruebas de humo definidas en el archivo
    ```

    **Explicación de la Nueva Estructura:**
    * `build-and-unit-test`: Job de CI (sin cambios mayores).
    * `deploy-staging`: Despliega a Render Staging (usa `RENDER_SERVICE_ID_STAGING`).
    * `test-staging`: Ejecuta **todas** las pruebas de `test_acceptance_app.py` contra la URL de Staging (`RENDER_APP_URL_STAGING`).
    * `deploy-production`: **¡NUEVO!**
        * Depende de `test-staging`.
        * Despliega a Render Producción (usa `RENDER_SERVICE_ID`).
    * `smoke-test-production`: **¡NUEVO!**
        * Depende de `deploy-production`.
        * Ejecuta **sólo** las pruebas de humo (usando `test_smoke_app.py`) contra la URL de Producción (`RENDER_APP_URL`).
    * Se movió las pruebas de aceptación y humo a los jobs correspondientes para que se ejecuten después del despliegue cómo se espera en un pipeline de CD.

8.  **Sube los cambios a GitHub:**
    ```bash
    git add .
    git commit -m "Implement CD pipeline with Staging and Production environments"
    git push origin main
    ```

9. **Verifica el despliegue:**
    *   Ve a la pestaña "Actions" de tu repositorio en GitHub. Deberías ver tu workflow ejecutándose.
    *   Si falla en alguna etapa de CI, revisa los logs y corrige el error hasta que pase todas las pruebas y logre el despliegue a Staging. Probablemente sea un tema de cobertura o de linting. Puedes ayudarte de la inteligencia artificial para corregir errores de linting o de cobertura. **Pista:** Si necesitas probar el módulo de `if __name__ == "__main__":` puedes usar `pyrun`, `pytest-mock` y mockear con `mock_run = mocker.patch('flask.Flask.run')` para implementar la prueba unitaria.
    *   Si todo va bien, el job `deploy` se ejecutará y desplegará tu aplicación a Render.
    *   Ve a tu panel de control de Render. Deberías ver tu servicio desplegado y en ejecución.
    *   Haz clic en el enlace de tu servicio para abrir tu aplicación en el navegador. (La URL será algo como `https://mi-calculadora.onrender.com`, cambia al inicio `mi-calculadora` por el nombre que le diste a tu servicio).
    *   Prueba tu calculadora.
    *   Prueba el health check:  Abre por ejemplo `https://mi-calculadora.onrender.com/health` en tu navegador. Deberías ver "OK".

      **Valida** en Github Actions y Render que el despliegue se haya realizado correctamente y que las pruebas de aceptación y humo se ejecuten correctamente **después** del despliegue.



## 10. Creación de un Monitor de Salud con GitHub Actions (monitor.yml)

Van a implementar un sistema de monitoreo para nuestra aplicación utilizando GitHub Actions. Este sistema verificará periódicamente el *health check* de nuestra aplicación y, si detecta un fallo, creará automáticamente un *issue* en nuestro repositorio de GitHub.

**Ventajas de este enfoque:**

*   **Simplicidad:** No necesitamos gestionar credenciales de correo electrónico ni dependencias externas complejas.
*   **Integración:** Se integra directamente con el ecosistema de GitHub. Las notificaciones llegan a través de los mecanismos habituales de GitHub (correo electrónico, notificaciones web, y la sección de "Issues" del repositorio).
*   **Seguimiento:** Los *issues* de GitHub son una excelente forma de rastrear problemas y su resolución, manteniendo un historial.
*   **No requiere servicios externos:** Todo se gestiona dentro de GitHub Actions.

**Características del monitoreo a implementar:**

*   **trigger**: el monitoreo debe desencadenarse todos los días a las 8 am o también de forma manual.
*   **permissions**: se le debe dar permisos de escritura a los *issues*:
    ```yaml
    permissions: # No lleva identación, va al mismo nivel de jobs u on.
      issues: write
    ```
*   **runner**: ubuntu linux.
*   **jobs**: 1 sólo job llamado `health-check`.
*   **steps y actions**: 
    - checkout.
    - validar la salud de la aplicación con un curl a la URL del health `{{ secrets.RENDER_APP_URL }}/health`:
        ```yaml
      - name: Check Application Health
        id: health_check
        continue-on-error: true
        run: |
          response=$(curl -s -o /dev/null -w "%{http_code}" "${{ secrets.RENDER_APP_URL }}/health")
          if [[ "$response" != "200" ]]; then
            echo "::set-output name=status::failed"
            echo "Health check failed with status code: $response"
            exit 1
          else
            echo "::set-output name=status::success"
            echo "Health check successful."
          fi
        ```
        *   **`- name: Check Application Health`:** Este paso verifica el health check de tu aplicación.
            *   **`id: health_check`:** Le da un ID al paso. Esto es importante para referenciarlo más tarde.
            *   **`continue-on-error: true`:** *Crucial*. Esto le dice a GitHub Actions que continúe con el siguiente paso *incluso si este paso falla*. Normalmente, si un paso falla, el workflow se detiene. Pero en este caso, queremos continuar para poder crear el issue.
            *   **`run:`:** Aquí ejecutamos un script de shell.
                *   `response=<span class="math-inline">\(curl \-s \-o /dev/null \-w "%\{http\_code\}" "</span>{{ secrets.RENDER_APP_URL }}/health")`: Hace una petición HTTP al endpoint `/health` de tu aplicación (usando la variable de entorno `RENDER_APP_URL`).
                    *   `curl`: Es una herramienta de línea de comandos para hacer peticiones HTTP.
                    *   `-s`: Modo silencioso (no muestra información de progreso).
                    *   `-o /dev/null`: Descarta la salida del cuerpo de la respuesta (solo nos interesa el código de estado).
                    *   `-w "%{http_code}"`: Imprime el código de estado HTTP (ej: 200, 500).
                    *   `${{ secrets.RENDER_APP_URL }}`: Usa el secreto de GitHub que contiene la URL de tu aplicación en producción.
                *   `if [[ "$response" != "200" ]]`: Verifica si el código de estado es diferente de 200 (OK).
                *   `echo "::set-output name=status::failed"`: Si el código no es 200, establece una *variable de salida* del paso llamada `status` con el valor `failed`. Esto nos permite saber en el siguiente paso si el health check falló.
                *   `echo "Health check failed..."`: Imprime un mensaje de error (opcional, para los logs).
                *    `exit 1`: Importante, termina la ejecución del *step* con un código de error.
                *   `else`: Si el código es 200.
                *    `echo "::set-output name=status::success"`: Establece la variable de salida del paso llamada `status` con el valor `success`.
                *   `echo "Health check successful."`: Imprime un mensaje (opcional, para los logs).

    - crear un issue en caso de que el health no funcione:
        ```yaml
      - name: Create Issue on Failure
        if: steps.health_check.outputs.status == 'failed'  # Solo se ejecuta si falló el health check.
        uses: actions/github-script@v6
        with:
          script: |
            github.rest.issues.create({
              owner: context.repo.owner,
              repo: context.repo.repo,
              title: 'Health Check Failed',
              body: 'The application health check failed. Please investigate.'
            })
        ```
        *   **`- name: Create Issue on Failure`:** Este paso crea un issue en GitHub *solo si el health check falló*.
            *   **`if: steps.health_check.outputs.status == 'failed'`:** Esta condición asegura que este paso solo se ejecute si el paso anterior (`health_check`) estableció la variable de salida `status` a `failed`.
            *   **`uses: actions/github-script@v6`:** Usa la acción `github-script`. Esta acción te permite ejecutar scripts de JavaScript que usan la API de GitHub (autenticándose automáticamente).
            *   **`with: script:`:** Aquí defines el script de JavaScript.
                *   `github.rest.issues.create(...)`: Llama a la API de GitHub para crear un issue.
                    *   `owner: context.repo.owner`: El propietario del repositorio (obtenido del contexto del workflow).
                    *   `repo: context.repo.repo`: El nombre del repositorio (obtenido del contexto del workflow).
                    *   `title: 'Health Check Failed'`:** El título del issue.
                    *   `body: 'The application health check failed. Please investigate.'`:** El cuerpo del issue.

1.  **Crea un nuevo archivo `.github/workflows/monitor.yml` en tu repositorio basado en las instrucciones y características indicadas**

2.  **Asegúrate de tener el secreto `RENDER_APP_URL` configurado en tu repositorio de GitHub:** Ve a Settings -> Secrets and variables -> Actions -> New repository secret.  El valor del secreto debe ser la URL completa de tu aplicación en producción en Render (ej: `https://mi-calculadora.onrender.com`).

3. **Sube el archivo `monitor.yml` a tu repositorio en la carpeta `.github/workflows/`**.
      ```bash
      git add .
      git commit -m "Health Check"
      git push origin main
      ```

4. **Da permisos de escritura a los issues:** En la página del repositorio, ve a Settings -> Code and automation -> Actions -> Permissions -> Workflow permissions, y selecciona `Read and write permissions`.

5. **Verifica que el workflow se ejecute correctamente:**
    *   Ve a la pestaña "Actions" de tu repositorio en GitHub.
    *   Ejecuta manualmente el workflow de health con el nombre definido en el `monitor.yml` (desde github actions).
    *   Deberías ver tu nuevo workflow ejecutándose.
    *   Si el health check de tu aplicación falla, deberías ver un nuevo issue creado en tu repositorio, cosa que hasta el momento no debería pasar pues tu aplicación está funcionando correctamente.

6. **Prueba el monitoreo ante un fallo:**
    *   Ve a tu aplicación en Render, selecciona `Settings` y baja hasta el final de la página. Encontrarás un botón para `Suspend Service`. Haz clic en él para detener tu aplicación.
    *   Ejecuta manualmente el workflow de health con el nombre definido en el `monitor.yml` (desde github actions) y valida que se cree un issue en tu repositorio.
    *   Valida que el workflow se ejecute correctamente y que se cree un issue en tu repositorio con nombre `Health Check Failed`.
    *   Vuelve a Render y selecciona `Resume Service` para reanudar tu aplicación.

**Cómo funciona:**

*   El workflow se ejecuta automáticamente todos los días a las horas que definas (según el `cron`). También se podrá ejecutar manualmente.
*   Hace una petición HTTP al endpoint `/health` de tu aplicación.
*   Si el código de respuesta es 200, el workflow termina con éxito.
*   Si el código de respuesta es diferente de 200, el paso `Check Application Health` falla (pero el workflow continúa gracias a `continue-on-error: true`).
*   Como el paso `Check Application Health` falló, el paso `Create Issue on Failure` se ejecuta.
*   El paso `Create Issue on Failure` usa la API de GitHub para crear un nuevo issue en tu repositorio.
*   Recibirás una notificación de GitHub (por correo electrónico y en la web) sobre el nuevo issue, indicando que el health check falló.
* Si existen issues abiertos con el título "Health Check Failed" y el health check es satisfactorio, se podría agregar un paso adicional para cerrarlos automáticamente. Esto requeriría usar `github.rest.issues.listForRepo` para buscar los issues y `github.rest.issues.update` para cambiar su estado a `closed`.

Este sistema de monitoreo, aunque básico, es una excelente forma de comenzar a supervisar la salud de tu aplicación y recibir notificaciones automáticas en caso de problemas, todo dentro del ecosistema de GitHub Actions. Sin embargo, **es importante aclarar que para una aplicación productiva es recomendable usar un sistema de monitoreo más robusto y completo, como New Relic, Datadog, Prometheus, Grafana, etc.**

## 13. Entregable en grupo

Para completar este taller, envía **un correo por grupo** con la siguiente información a `dhoyoso@eafit.edu.co` con el asunto "Entregable Final CI/CD":

1.  **URL del repositorio PÚBLICO de GitHub:** Envía la URL de tu repositorio en GitHub.

2.  **URL de la ejecución COMPLETA del workflow `ci-cd.yml`:** Envía la URL de una ejecución que haya completado **todos** los jobs exitosamente (build -> deploy-staging -> test-staging -> deploy-prod -> smoke-test-prod). Si configuraste aprobación manual, asegúrate de haberla aprobado para que se complete.

3.  **URL de la aplicación desplegada en RENDER (STAGING):** Envía la URL de tu app en el entorno de **Staging**.

4.  **URL de la aplicación desplegada en RENDER (PRODUCCIÓN):** Envía la URL de tu app en el entorno de **Producción**. Debe funcionar el health check (`/health`).

4.  **URL de la ejecución del workflow de monitoreo:** Envía la URL de una ejecución de tu workflow `monitor.yml` en GitHub Actions.  No es necesario que esta ejecución haya fallado (y creado un issue); basta con que se haya ejecutado.  *Incluye también un pantallazo del issue creado en GitHub cuando probaste el monitor deteniendo la aplicación.*  Si no detuviste la aplicación para probar, detén la aplicación, ejecuta el monitor *manualmente* y envía el pantallazo del issue creado.

5.  **Responde a las siguientes preguntas:**

    * Explica brevemente el flujo de trabajo **nuevo** completo que implementaste (commit -> CI -> Deploy Staging -> Test Staging -> Deploy Prod -> Smoke Test Prod). Sé *específico* sobre *qué se ejecuta en cada job, en qué entorno, y qué valida cada tipo de prueba*.
    *   ¿Qué ventajas encuentras en todo este pipeline de ci/cd implementado a la hora de trabajar con múltiples desarrolladores en un proyecto? ¿Qué mejorarías a este pipeline para hacerlo más eficiente en términos del time to market de la aplicación?
    * ¿Qué ventajas y desventajas tiene introducir un entorno de Staging en el pipeline? ¿Cómo impacta esto la velocidad vs. la seguridad?
    * ¿Qué diferencia hay entre las pruebas ejecutadas contra Staging (`test-staging`) y las ejecutadas contra Producción (`smoke-test-production`) en tu pipeline? ¿Por qué esta diferencia?
    *   ¿Qué ventajas tiene usar variables de entorno para la configuración de una aplicación?
    *   Explica *en detalle* cómo funciona el script de monitoreo (`monitor.yml`). Explica qué hace cada *job* y cada *step*, incluyendo las condiciones (`if`) y las acciones (`uses`).
    * ¿Qué mejoras harías al monitoreo para producción? (Menciona 3 mejoras concretas, justifica, investiga herramientas).
    * ¿Qué problemas/dificultades encontraste? ¿Soluciones? ¿Aprendizaje nuevo?
    * ¿Qué partes del ciclo CI/CD le faltan a este pipeline? (Menciona 2, por qué, cómo implementarlas).

6.  **Integrantes del grupo:** Lista los nombres y apellidos *completos* de los integrantes de tu grupo.

**Criterios de evaluación:**

* URLs del repo y workflows correctas y funcionales. La ejecución de `ci-cd.yml` debe mostrar todos los jobs completados.
* URLs de **ambas** aplicaciones (Staging y Producción) funcionando. `/health` debe funcionar en Producción.
* Ejecución del monitor y pantallazo del issue de fallo **contra producción**.
* Respuestas claras, correctas, completas y profundas, **reflejando la nueva estructura con Staging**.
* Completitud y funcionamiento de todos los pasos del taller.