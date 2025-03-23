# Taller Grupal Entregable 2: Pipeline de CI Completo con Python, GitHub Actions y Herramientas Open Source

En este taller construiremos un pipeline de Integración Continua (CI) completo para una aplicación Python simple, utilizando GitHub Actions y herramientas open source.  Aprenderás a:

*   Analizar la calidad de tu código con Pylint, Flake8, Black y medir la cobertura con Coverage.py.
*   Integrar SonarCloud para análisis estático continuo.
*   Escribir y ejecutar pruebas unitarias con pytest.
*   Automatizar pruebas de aceptación con Selenium y un headless browser.
*   Configurar un workflow de GitHub Actions que ejecute todas estas etapas en cada push.
*   Usar variables de entorno y secretos de forma segura.
*  (Opcional) Generar un reporte HTML con los resultados de las pruebas.

## 1. Introducción: ¿Qué es un Pipeline de CI?

Un pipeline de CI es un proceso automatizado que se ejecuta cada vez que se realizan cambios en el código de un repositorio.  Típicamente, un pipeline de CI incluye las siguientes etapas:

1.  **Code (Código):**  El desarrollador escribe código y lo sube al repositorio (usando Git, idealmente siguiendo un flujo de trabajo como Gitflow).

    **Gitflow** es un flujo de trabajo de Git que define una serie de ramas y reglas para gestionar el desarrollo de software de manera estructurada. Utiliza ramas principales como main y develop, y ramas de soporte como feature, release, y hotfix para organizar el trabajo en nuevas funcionalidades, preparaciones para lanzamientos y correcciones de errores críticos.

2.  **Build (Construcción):**  Se compila el código (si es necesario, en Python no es estrictamente una compilación, pero se le llama build a la fase de preparación), se instalan las dependencias y se prepara el entorno de ejecución.  En Python, esta etapa a menudo implica crear un entorno virtual e instalar paquetes con `pip` desde un archivo `requirements.txt`.

3.  **Test (Pruebas):**  Se ejecutan pruebas automatizadas para verificar que el código funciona correctamente y cumple con los requisitos de calidad.  Esto incluye:
    *   **Análisis de Calidad de Código:**  Herramientas que verifican el estilo del código, la complejidad, la presencia de errores potenciales, code smells y el cumplimiento de estándares (ej: PEP 8 en Python).
    *   **Pruebas Unitarias:**  Prueban unidades individuales de código (funciones, clases, métodos) de forma aislada.
    *   **Pruebas de Aceptación (o Pruebas Funcionales/E2E):**  Prueban la aplicación desde la perspectiva del usuario, simulando interacciones reales, usualmente a través de la interfaz de usuario (UI).

4.  **Release/Deploy (Liberación/Despliegue):**  Si todas las pruebas pasan, el código se considera apto para ser liberado.  En un entorno de *Integración Continua*, esta etapa suele ser solo una verificación (ej: generar un reporte, crear un tag en Git).  En un entorno de *Entrega/Despliegue Continuo (CD)*, esta etapa implicaría el despliegue automático a un entorno (por ejemplo, staging o producción).  *En este taller, nos centraremos en las etapas de CI (Code, Build, Test).*

## 2. Herramientas que Usaremos

*   **Python:**  Lenguaje de programación.
*   **GitHub Actions:**  Plataforma de CI/CD.
*   **Pylint:**  Analizador estático de código Python.  Detecta errores, problemas de estilo y código sospechoso.
*   **Flake8:**  Otra herramienta de análisis estático.  Combina PyFlakes (detección de errores lógicos), pycodestyle (verificación de estilo según PEP 8) y un detector de complejidad McCabe.
*   **Black:**  Formateador de código automático.  Aplica un estilo consistente a tu código.
*   **pytest:**  Framework para escribir y ejecutar pruebas unitarias.
*   **Coverage.py:** Mide la *cobertura de código* de tus pruebas unitarias (qué porcentaje de tu código está siendo ejecutado por las pruebas).
*   **Selenium:**  Framework para automatizar pruebas de aplicaciones web (pruebas de aceptación).  Permite simular la interacción de un usuario con un navegador.
    *   **Webdriver Manager:**  Simplifica la gestión de los drivers de navegador para Selenium.
*   **SonarCloud:**  Plataforma *online* gratuita (para proyectos open source) para inspección continua de la calidad del código.  Se integra con GitHub Actions.
*   **Flask:** Microframework web de Python (para crear una aplicación web sencilla de ejemplo).

## 3. Configuración del Proyecto

1.  **Crea un nuevo repositorio en GitHub:**  Llámalo, por ejemplo, `ci-pipeline-python`.

2.  **Clona el repositorio a tu máquina local**

3.  **Crea un entorno virtual y activalo:**

    ```bash
    python3 -m venv venv      # En algunos SO puede ser sólo python, si no te funciona instala virtualenv con el comando 'pip install virtualenv'
    source venv/bin/activate  # En Linux/macOS
    venv\Scripts\activate     # En Windows
    ```

4. **Crea un archivo 'requirements.txt' en la raíz de tu repositorio:**

    ```plaintext
    pylint
    flake8
    black
    pytest
    coverage
    selenium
    webdriver-manager
    flask
    ```

    Este archivo lista las dependencias de tu proyecto y sus versiones.  GitHub Actions usará este archivo para instalar las mismas versiones en el entorno de ejecución.

5.  **Instala las dependencias:**

    ```bash
    pip install -r requirements.txt
    ```

## 4. La Aplicación de Ejemplo (Calculadora Web)

Crearemos una aplicación web muy sencilla con Flask (una calculadora) para tener algo que probar tanto a nivel unitario como a nivel de aceptación.

1.  **Crea una carpeta `app`**: Dentro de la carpeta raíz de tu proyecto (es decir `ci-pipeline-python`)
2.  **Crea un archivo llamado `calculadora.py` dentro de la carpeta `app`**:

    ```python
    # app/calculadora.py
    def sumar(a, b):
        return a + b

    def restar(a, b):
        return a - b

    def multiplicar(a, b):
        return a * b

    def dividir(a, b):
        if b == 0:
            raise ZeroDivisionError("No se puede dividir por cero")
        return a / b
    ```

3.  **Crea un archivo `app.py` dentro de la carpeta `app`**:

    ```python
    # app/app.py
    from flask import Flask, render_template, request
    from .calculadora import sumar, restar, multiplicar, dividir  # Importa desde el mismo paquete

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

    if __name__ == "__main__":
        app.run(debug=True) #  Quita debug=True en producción
    ```

4.  **Crea una carpeta `templates` dentro de la carpeta `app`**

5.  **Crea un archivo `templates/index.html` dentro de la carpeta `templates`:**

    ```html
    <!DOCTYPE html>
    <html>
    <head>
        <title>Calculadora</title>
    </head>
    <body>
        <h1>Calculadora</h1>
        <form method="post">
            <input type="text" name="num1" placeholder="Número 1" required>
            <input type="text" name="num2" placeholder="Número 2" required>
            <select name="operacion">
                <option value="sumar">Sumar</option>
                <option value="restar">Restar</option>
                <option value="multiplicar">Multiplicar</option>
                <option value="dividir">Dividir</option>
            </select>
            <button type="submit">Calcular</button>
        </form>
        {% if resultado %}
            <h2>Resultado: {{ resultado }}</h2>
        {% endif %}
    </body>
    </html>
    ```

6. **Crea un archivo '__init __.py' dentro de la carpeta 'app'**:

    ```python
    # app/__init__.py
    ```

    Este archivo vacío le dice a Python y a los módulos de prueba que la carpeta `app` es un paquete y puede contener módulos.

7. **Crea un archivo '.gitignore' en la raíz de tu repositorio**:

    ```plaintext
    # virtual envs
    venv/
    env/

    # python 
    *.pyc
    *.pyo
    *.pyd
    __pycache__/

    # pytest
    .pytest_cache/
    pytestdebug.log

    # coverage
    .coverage
    htmlcov/

    # IDE
    .vscode/
    ```

    Este archivo le dice a Git qué archivos y carpetas *no* debe incluir en el repositorio (por ejemplo, el entorno virtual, archivos de caché, archivos de configuración locales, etc.).

8. **Ejecuta la aplicación Flask:**
    Parado en la raíz de tu repositorio, ejecuta:

    ```bash
    python -m app.app # en algunos SO, es python3 -m app.app
    ```

    Abre tu navegador y ve a [http://localhost:5000](http://localhost:5000).  Deberías ver la calculadora web.  Prueba a hacer algunas operaciones para asegurarte de que funciona correctamente.

    *Nota:*  Para detener la aplicación Flask, presiona `Ctrl+C` en la terminal.

## 5. Análisis de Calidad de Código

### 5.1. Pylint, Flake8 y Black (Local)

1.  **Ejecuta Pylint:**

    ```bash
    pylint app
    ```

    Verás un informe con los problemas encontrados (si los hay) y una puntuación.  Presta atención a los errores (E), warnings (W), convenciones (C) y refactorizaciones (R).

2.  **Ejecuta Flake8:**

    ```bash
    flake8 app
    ```

    Flake8 mostrará errores de estilo (según PEP 8) y otros problemas.

3.  **Ejecuta Black:**

    ```bash
    black app
    ```

    Black reformateará tu código automáticamente para que cumpla con un estilo consistente.  Después de ejecutar Black, vuelve a ejecutar Pylint y Flake8 para ver si los problemas de estilo han desaparecido.

    *Importante:*  Es una buena práctica ejecutar estas herramientas *antes* de hacer un commit. Así te aseguras de que tu código cumple con los estándares de calidad y estilo, evitando con antelación que el pipeline de CI falle.

### 5.2. Integración con SonarCloud (Análisis Estático Continuo)

SonarCloud es una plataforma en la nube que proporciona análisis estático continuo de código.  Es gratuita para proyectos de código abierto y se integra muy bien con GitHub Actions.

1.  **Crea una cuenta en SonarCloud:**  Ve a [https://sonarcloud.io/](https://sonarcloud.io/) y regístrate con tu cuenta de GitHub.
2.  **Crea una organización:** Si no tienes una, te pedirá que crees una organización (puede ser tu nombre de usuario de GitHub).
3.  **Importa tu repositorio:**
    *   Haz clic en el botón "+" y selecciona "Analyze a new project".
    *   Selecciona tu repositorio (`ci-pipeline-python`).
    *   SonarCloud analizará tu repositorio y te mostrará un panel de control con los resultados.  Inicialmente, puede que no muestre mucha información porque no hemos configurado el análisis en el workflow.
4.  **Obtén tu token de SonarCloud:**
    *   Ve a "My Account" (en la esquina superior derecha).
    *   Ve a la pestaña "Security".
    *   Genera un token (dale un nombre, por ejemplo, "GitHub Actions Token").  *Copia este token*.  Lo necesitarás más adelante.
5.  **Crea los secretos en GitHub:**
    *   Ve a tu repositorio en GitHub > Settings > Secrets and variables > Actions > New repository secret.
    *   Crea *dos* secretos:
        *   **`SONAR_TOKEN`:**  Pega el token que copiaste de SonarCloud.
        *   **`SONAR_HOST_URL`:**  Pon `https://sonarcloud.io`.

6.  **Crea un archivo `sonar-project.properties` en la raíz de tu repositorio:** Este archivo le dice a SonarCloud cómo analizar tu proyecto.

    ```properties
    sonar.projectKey=[TU_USUARIO]_ci-pipeline-python
    sonar.organization=[TU_ORGANIZACION]
    sonar.sources=app  # Analiza solo el código dentro de la carpeta 'app'
    sonar.python.version=3.x  # Reemplaza x con tu versión de Python (ej: 3.9)
    sonar.python.pylint.reportPaths=pylint-report.txt
    sonar.python.flake8.reportPaths=flake8-report.txt
    sonar.coverage.exclusions=**/tests/**,**/setup.py # Excluye los tests y setup.py
    sonar.python.coverage.reportPaths=coverage.xml

    ```
     **Importante:**
    *   Reemplaza `[TU_USUARIO]` con tu *nombre de usuario de GitHub*.
    *   Reemplaza `[TU_ORGANIZACION]` con el *nombre de tu organización en SonarCloud* (que puede ser diferente a tu nombre de usuario de GitHub).
    *   `sonar.sources=app`:  Esto le dice a SonarCloud que solo analice el código dentro de la carpeta `app`.  Si tuvieras código en otros directorios, los añadirías aquí (separados por comas).

## 6. Pruebas Unitarias con pytest y Cobertura

1.  Crea un directorio llamado `tests` en la raíz de tu proyecto.

2.  Dentro de `tests`, crea un archivo llamado `test_calculadora.py`:

    ```python
    # tests/test_calculadora.py
    import pytest
    from app.calculadora import sumar, restar, multiplicar, dividir  # Importa desde app

    def test_sumar():
        assert sumar(2, 3) == 5
        assert sumar(-1, 1) == 0
        assert sumar(0, 0) == 0

    def test_restar():
        assert restar(5, 2) == 3
        assert restar(1, -1) == 2
        assert restar(0, 0) == 0

    def test_multiplicar():
        assert multiplicar(2, 3) == 6
        assert multiplicar(-1, 5) == -5
        assert multiplicar(0, 10) == 0

    def test_dividir():
        assert dividir(10, 2) == 5.0
        assert dividir(5, -1) == -5.0
        with pytest.raises(ZeroDivisionError):
            dividir(1, 0)
    ```

3.  **Ejecuta las pruebas (localmente):**

    ```bash
    pytest
    ```

    pytest buscará automáticamente archivos que comiencen con `test_` o terminen con `_test` y ejecutará las funciones que comiencen con `test_`.

4. **Mide la cobertura (localmente):**

    ```bash
     coverage run -m pytest
     coverage report -m  # Muestra un informe en la consola
     coverage html       # Genera un informe HTML
    ```
    * `coverage run -m pytest`: Ejecuta las pruebas usando `coverage`.
    * `coverage report -m`: Muestra un resumen de la cobertura en la consola, incluyendo las líneas que no fueron cubiertas (`missing`).
    *  `coverage html`: Genera un informe HTML detallado en una carpeta llamada `htmlcov`. Puedes abrir `htmlcov/index.html` en tu navegador para ver la cobertura línea por línea.

## 7. Pruebas de Aceptación con Selenium

Las pruebas de aceptación (o pruebas E2E - End-to-End) simulan la interacción de un usuario real con la aplicación, normalmente a través de la interfaz de usuario (en nuestro caso, la página web de la calculadora).  Selenium es un framework que permite automatizar esta interacción.

1.  Crea un archivo llamado `tests/test_app.py`:

    ```python
    # tests/test_app.py
    import pytest
    from selenium import webdriver
    from selenium.webdriver.common.by import By
    from selenium.webdriver.support.ui import Select
    from webdriver_manager.chrome import ChromeDriverManager
    from webdriver_manager.firefox import GeckoDriverManager

    # Configuración del driver (elige uno: Chrome o Firefox)
    @pytest.fixture
    def browser():
        # Opción 1: Chrome (headless - sin interfaz gráfica)
        options = webdriver.ChromeOptions()
        options.add_argument("--headless")  # Ejecuta sin interfaz gráfica
        options.add_argument("--no-sandbox") # Necesario para algunos entornos
        options.add_argument("--disable-dev-shm-usage") # Necesario para algunos entornos
        driver = webdriver.Chrome(options=options)

        # Opción 2: Firefox (headless)
        # options = webdriver.FirefoxOptions()
        # options.add_argument("--headless")
        # driver = webdriver.Firefox(options=options)

        # Opción 3: Chrome (con interfaz gráfica - para depuración local)
        # driver = webdriver.Chrome()

        # Opción 4: Firefox (con interfaz gráfica)
        # driver = webdriver.Firefox()
        yield driver
        driver.quit()

    def test_calculadora(browser):
        browser.get("http://localhost:5000")  # Cambia si tu app se ejecuta en otro puerto

        # Encuentra los elementos de la página
        num1_input = browser.find_element(By.NAME, "num1")
        num2_input = browser.find_element(By.NAME, "num2")
        operacion_select = Select(browser.find_element(By.NAME, "operacion"))
        calcular_button = browser.find_element(By.CSS_SELECTOR, "button[type='submit']")
        resultado = browser.find_element(By.TAG_NAME, "h2")

        # Prueba 1: Sumar 2 + 3
        num1_input.send_keys("2")
        num2_input.send_keys("3")
        operacion_select.select_by_value("sumar")
        calcular_button.click()
        assert "Resultado: 5" in resultado.text

        # Prueba 2: Restar 5 - 2
        num1_input.clear()  # Limpia los campos
        num2_input.clear()
        num1_input.send_keys("5")
        num2_input.send_keys("2")
        operacion_select.select_by_value("restar")
        calcular_button.click()
        assert "Resultado: 3" in resultado.text

        # Prueba 3: Multiplicar 4 * 6
        num1_input.clear()
        num2_input.clear()
        num1_input.send_keys("4")
        num2_input.send_keys("6")
        operacion_select.select_by_value("multiplicar")
        calcular_button.click()
        assert "Resultado: 24" in resultado.text

        # Prueba 4: Dividir 10 / 2
        num1_input.clear()
        num2_input.clear()
        num1_input.send_keys("10")
        num2_input.send_keys("2")
        operacion_select.select_by_value("dividir")
        calcular_button.click()
        assert "Resultado: 5" in resultado.text  # O "5.0" dependiendo de tu formateo

        # Prueba 5: Dividir por cero (debería mostrar un mensaje de error)
        num1_input.clear()
        num2_input.clear()
        num1_input.send_keys("5")
        num2_input.send_keys("0")
        operacion_select.select_by_value("dividir")
        calcular_button.click()
        assert "Error: No se puede dividir por cero" in resultado.text

        # Prueba 6: Valores no numéricos
        num1_input.clear()
        num2_input.clear()
        num1_input.send_keys("abc")
        num2_input.send_keys("def")
        calcular_button.click()
        assert "Error: Introduce números válidos" in resultado.text
    ```

2.  **Ejecuta las pruebas de aceptación (localmente):**

    Para ejecutar las pruebas de aceptación *localmente*, primero necesitas ejecutar tu aplicación Flask:

    ```bash
    python app/app.py
    ```
    Luego, en *otra* terminal (manteniendo la aplicación Flask en ejecución):
    ```bash
    pytest tests/test_app.py
    ```

**Explicación del código de prueba de aceptación:**

*   **`import pytest`:** Importa el framework pytest.
*   **`from selenium import webdriver`:** Importa las clases necesarias de Selenium.
*   **`from selenium.webdriver.common.by import By`:** Importa `By` para seleccionar elementos por diferentes criterios (nombre, ID, CSS selector, etc.).
*  **`from selenium.webdriver.support.ui import Select`**: Importa `Select` para interactuar con elementos `<select>` (dropdowns).
*   **`from webdriver_manager.chrome import ChromeDriverManager`**:  Importa `webdriver-manager` para Chrome.
*   **`from webdriver_manager.firefox import GeckoDriverManager`**: Importa `webdriver-manager` para Firefox.
*   **`@pytest.fixture`:**  Define un *fixture* de pytest.  Un fixture es una función que prepara el entorno para las pruebas (en este caso, crea y cierra la instancia del navegador).
    *   **`def browser():`:**  La función del fixture.
        *   **Opciones de Chrome/Firefox:**  Configura las opciones del navegador.  `--headless` hace que se ejecute sin interfaz gráfica (ideal para CI). `--no-sandbox` y `--disable-dev-shm-usage` son opciones a veces necesarias en entornos de CI.
        * **`webdriver.Chrome(options=options)`/`webdriver.Firefox(options=options)`:** Crea una instancia del driver de Chrome/Firefox.
        * **`yield driver`:**  "Devuelve" el driver a las pruebas.  La ejecución se pausa aquí hasta que la prueba termine.
        * **`driver.quit()`:**  *Después* de que la prueba termina, cierra el navegador.
*   **`def test_calculadora(browser):`:**  La función de prueba.  Recibe el `browser` (el driver) del fixture.
    *   **`browser.get("http://localhost:5000")`:**  Abre la página de la calculadora.  Cambia la URL si tu aplicación se ejecuta en un puerto diferente.
    *   **`browser.find_element(...)`:**  Encuentra elementos en la página (inputs, botones, etc.).  Estamos usando `By.NAME` y `By.CSS_SELECTOR` para localizarlos, pero hay otras opciones (ver documentación de Selenium).
    *   **`num1_input.send_keys(...)`:**  Escribe texto en un campo de entrada.
    *   **`operacion_select.select_by_value(...)`:**  Selecciona una opción en un dropdown.
    *   **`calcular_button.click()`:**  Hace clic en un botón.
    *   **`assert "..." in resultado.text`:**  Verifica que el resultado de la operación sea el esperado.

## 8. GitHub Actions Workflow (CI Pipeline)

Ahora, vamos a crear el archivo YAML que define nuestro pipeline de CI en GitHub Actions.

1.  Crea el archivo `.github/workflows/ci.yml`:

    ```yaml
    name: CI Pipeline

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
              python-version: '3.12'

          - name: Install dependencies
            run: |
              python -m pip install --upgrade pip
              pip install -r requirements.txt
              pip install flask

          - name: Run Black (Formatter)
            run: black app tests --check  # --check solo verifica, no modifica

          - name: Run Pylint (Linter)
            run: pylint app --output-format=pylint-report.txt || true
            #El flag --exit-zero se sustituye por || true, para evitar errores si hay warnings.

          - name: Run Flake8 (Linter)
            run: flake8 app --output-file=flake8-report.txt || true

          - name: Run Unit Tests with pytest and Coverage
            run: |
              coverage run -m pytest tests/test_calculadora.py
              coverage xml -o coverage.xml  # Genera un informe XML para SonarCloud

          - name: Run Acceptance Tests with Selenium
            run: |
              python -m http.server 8000 & # Inicia un servidor web simple en segundo plano.
              pytest tests/test_app.py

          - name: SonarCloud Scan
            uses: SonarSource/sonarcloud-github-action@master
            env:
              GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}  # Automáticamente proporcionado
              SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}   # El secreto que creaste
            with:
              args: >
                -Dsonar.projectKey=[TU_USUARIO]_ci-pipeline-python
                -Dsonar.organization=[TU_ORGANIZACION]
                -Dsonar.sources=app
                -Dsonar.python.version=3.x #Pon tu version de python.
                -Dsonar.python.pylint.reportPaths=pylint-report.txt
                -Dsonar.python.flake8.reportPaths=flake8-report.txt
                -Dsonar.coverage.exclusions=**/tests/**,**/setup.py
                -Dsonar.python.coverage.reportPaths=coverage.xml

    ```

**Explicación del Workflow:**

*   **`name`:** Nombre del workflow.
*   **`on`:**  Se ejecuta en `push` a `main`, en `pull_request` a `main`, y manualmente (`workflow_dispatch`).
*   **`jobs`:**  Un solo job llamado `build-and-test`.
    *   **`runs-on: ubuntu-latest`:**  Se ejecuta en un runner con Ubuntu.
    *   **`steps`:**
        *   **`actions/checkout@v3`:**  Clona el repositorio.
        *   **`actions/setup-python@v3`:**  Configura Python.
        *   **`Install dependencies`:** Instala dependencias.  Se asume el archivo `requirements.txt`.
        *   **`Run Black`:**  Ejecuta Black en modo de verificación (`--check`).
        *   **`Run Pylint`:**  Ejecuta Pylint. El `|| true` evita que el workflow falle si Pylint encuentra *warnings* (solo fallará si hay errores, lo cual es deseable).  Guarda el informe para SonarCloud.
        *   **`Run Flake8`:**  Ejecuta Flake8.  `|| true` para evitar fallos por warnings. Guarda el informe.
        *   **`Run Unit Tests with pytest and Coverage`:**  Ejecuta las pruebas unitarias con pytest y genera un informe de cobertura en formato XML (`coverage.xml`).  Este informe será usado por SonarCloud.
        *   **`Run Acceptance Tests with Selenium`:**
            *   `python -m http.server 8000 &`:  Inicia un servidor web *simple* en segundo plano (usando el módulo `http.server` de Python) para servir la aplicación Flask.  Esto es necesario porque las pruebas de Selenium necesitan interactuar con la aplicación web en ejecución.  *No uses este servidor en producción*.  Es solo para pruebas locales y en el entorno de CI.  El `&` al final ejecuta el comando en segundo plano, permitiendo que el workflow continúe.
            *   `pytest tests/test_app.py`:  Ejecuta las pruebas de aceptación.
        *   **`SonarCloud Scan`:**
            *   **`SonarSource/sonarcloud-github-action@master`:**  La acción oficial de SonarCloud para GitHub Actions.
            *   **`env`:**
                *   **`GITHUB_TOKEN`:**  Token proporcionado automáticamente por GitHub Actions.
                *   **`SONAR_TOKEN`:**  El token que creaste en SonarCloud (como secreto).
            *   **`with/args`:**  Argumentos de configuración para SonarCloud.  Estos *deben coincidir* con la configuración en tu archivo `sonar-project.properties`.  Aquí se especifica dónde encontrar los informes de Pylint, Flake8 y Coverage.

2.  **Asegúrate de que tienes un archivo `requirements.txt`:** Si no lo has creado ya, ejecuta `pip freeze > requirements.txt` en tu entorno virtual *activado* y con todas las dependencias instaladas.

3.  **Sube los cambios a GitHub:**

    ```bash
    git add .
    git commit -m "Crear pipeline de CI completo"
    git push origin main
    ```

4.  **Verifica la ejecución del workflow:**  Ve a la pestaña "Actions" de tu repositorio en GitHub.  Deberías ver tu workflow ejecutándose (o ya completado).  Haz clic en él para ver los detalles y los logs de cada paso.

5.  **Ve a SonarCloud y verifica los resultados del análisis:**  Deberías ver un informe detallado con la calidad de tu código, la cobertura de las pruebas, etc.

## 9. Entregable en grupo

Para completar este taller, envía la siguiente información a `dhoyoso@eafit.edu.co` con el asunto "Entregable Taller 2 CI":

1.  **URL del repositorio PUBLICO de GitHub:**  Envía la URL de tu repositorio `ci-pipeline-python` en GitHub.
2.  **URL de la ejecución del workflow:**  Envía la URL de una ejecución *exitosa* de tu workflow en GitHub Actions (de la pestaña "Actions").
3.  **URL del proyecto en SonarCloud:** Envía la URL de tu proyecto en SonarCloud (donde se ven los resultados del análisis).
4.  **Responde a las siguientes preguntas:**
    *   ¿Qué ventajas le proporciona a un proyecto el uso de un pipeline de CI?  Menciona al menos tres ventajas *específicas* y explica por qué son importantes.
    *   ¿Cuál es la diferencia principal entre una prueba unitaria y una prueba de aceptación?  Da un ejemplo de algo que probarías con una prueba unitaria y algo que probarías con una prueba de aceptación (en el contexto de cualquier aplicación que conozcas (describela primero)).
    *   ¿Qué es la cobertura de código y por qué es útil medirla?  ¿Qué nivel de cobertura consideras aceptable para un proyecto real?
    *   Describe brevemente qué hace cada uno de los *steps* de tu workflow de GitHub Actions.  Explica el propósito de cada uno (no me digas que hace, sino para qué se hace).
    *   ¿Qué problemas o dificultades encontraste al implementar este taller?  ¿Cómo los solucionaste?  (Si no encontraste ningún problema, describe algo nuevo que hayas aprendido).
5. **Integrantes del grupo:**  Lista los nombres y apellidos de los integrantes de tu grupo.

## 10. (Opcional) Mejoras Adicionales

Si quieres profundizar más, aquí tienes algunas ideas para mejorar tu pipeline:

*   **Generar un reporte HTML de pytest:**  Usa el plugin `pytest-html` para generar un reporte HTML con los resultados de las pruebas.
    *   Instala `pytest-html`:  `pip install pytest-html`
    *   Modifica el paso de pytest en tu workflow:
        ```yaml
          - name: Run Unit Tests with pytest and Coverage
            run: |
              coverage run -m pytest tests/test_calculadora.py --html=report.html --self-contained-html
              coverage xml -o coverage.xml
        ```
    *   Sube el archivo `report.html` como un *artefacto* del workflow:
        ```yaml
          - name: Upload pytest report
            uses: actions/upload-artifact@v3
            if: always()  # Sube el artefacto incluso si las pruebas fallan
            with:
              name: pytest-report
              path: report.html
        ```
        Un artefacto es un archivo o directorio que se guarda y se puede descargar desde la interfaz de GitHub Actions después de que el workflow haya terminado. Es muy útil para guardar informes, logs, código compilado, etc.

*   **Usar un archivo `.pylintrc`:**  Configura Pylint con un archivo `.pylintrc` para personalizar las reglas.
*   **Usar un archivo de configuración para Flake8:**  Configura Flake8 con un archivo `.flake8` (o dentro de `setup.cfg` o `tox.ini`).
*   **Agregar Badges:**  Añade badges (insignias) al README de tu repositorio para mostrar el estado del workflow, la cobertura, etc. (GitHub Actions y SonarCloud proporcionan URLs para generar estos badges).
*   **Integración con otras herramientas:**  Podrías integrar tu pipeline con otras herramientas, como:
    *   Herramientas de gestión de vulnerabilidades (ej: Snyk, Dependabot).
    *   Herramientas de análisis de seguridad estático (SAST).
*   **Ejecutar las pruebas en una matriz (Avanzado):**  Ejecuta las pruebas en múltiples sistemas operativos y versiones de Python:

    ```yaml
    jobs:
      build-and-test:
        runs-on: ${{ matrix.os }}
        strategy:
          matrix:
            os: [ubuntu-latest, windows-latest, macos-latest]
            python-version: ['3.8', '3.9', '3.10', '3.11']
        steps:
          - uses: actions/checkout@v3
          - name: Set up Python ${{ matrix.python-version }}
            uses: actions/setup-python@v3
            with:
              python-version: ${{ matrix.python-version }}
          # ... el resto de los pasos ...
    ```
* **Agregar pruebas para app.py:** Se podrían agregar pruebas unitarias para el archivo `app.py`, aunque las pruebas de aceptación con Selenium ya cubren la funcionalidad principal de la aplicación web.