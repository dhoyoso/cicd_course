# Taller Entregable 1: Introducción a CI/CD con GitHub y GitHub Actions

Este es el primer entregable de tu curso de CI/CD. Aprenderás los fundamentos de GitHub, el control de versiones con Git, la configuración con YAML y cómo automatizar tareas con GitHub Actions.  ¡No te preocupes si no tienes experiencia previa, te guiaremos paso a paso!

## 1. Introducción a GitHub y GitHub Actions

### ¿Qué es GitHub?

GitHub es una plataforma en la nube para alojar y gestionar código usando un sistema de control de versiones llamado Git.  Piensa en GitHub como un "Google Drive para código", pero con superpoderes:

*   **Control de Versiones:**  Git registra cada cambio que haces en tu código.  Puedes volver a versiones anteriores, ver quién hizo qué cambios y cuándo, y trabajar en diferentes "ramas" de tu proyecto sin afectar la versión principal.
*   **Colaboración:**  GitHub facilita el trabajo en equipo.  Varias personas pueden trabajar en el mismo proyecto simultáneamente, combinar sus cambios y resolver conflictos.
*   **Comunidad:**  GitHub es una red social para desarrolladores. Puedes compartir tu código, contribuir a proyectos de otros, y encontrar soluciones a problemas.
* **Almacenamiento en la Nube**: Tu código se guarda en los servidores de GitHub, lo que te permite acceder a él desde cualquier dispositivo con conexión a internet, y proporciona un respaldo seguro.

### ¿Qué es Git?

Git es el *sistema* de control de versiones. Es el motor que está por debajo. GitHub es la *plataforma* que usa Git y le añade una interfaz web, herramientas de colaboración, etc.  Git se usa desde la línea de comandos, mientras que GitHub se usa principalmente a través del navegador web (aunque también tiene una aplicación de escritorio).

### Creación de un Repositorio en GitHub

1.  **Crea una cuenta en GitHub:**  Ve a [https://github.com/](https://github.com/) y regístrate. Es gratis.
2.  **Crea un nuevo repositorio:**
    *   Una vez que hayas iniciado sesión, haz clic en el botón "+" en la esquina superior derecha y selecciona "New repository".  También puedes ir directamente a [https://github.com/new](https://github.com/new).
    *   **Repository name:**  Dale un nombre a tu repositorio (por ejemplo, `entregable-1-cicd`).  *No uses espacios, usa guiones (`-`) en su lugar*.
    *   **Description (opcional):**  Añade una breve descripción (por ejemplo, "Primer entregable del curso de CI/CD").
    *   **Public/Private:**
        *   **Public:**  Cualquiera puede ver tu código (pero solo tú puedes modificarlo, a menos que des permiso a otros).  Ideal para proyectos de código abierto.
        *   **Private:**  Solo tú (y las personas que invites) pueden ver y modificar el código.
        *   *Para este curso, puedes usar un repositorio público o privado.*
    *   **Initialize this repository with:**
        *   **Add a README file:**  Marca esta casilla.  Un archivo README es una descripción de tu proyecto.  Es una buena práctica tener uno.
    *   Haz clic en "Create repository".

### Comandos Básicos de Git (Versionamiento)

Para interactuar con tu repositorio de GitHub desde tu computadora, necesitarás usar Git desde la línea de comandos (Terminal en macOS/Linux, Git Bash o Command Prompt en Windows).  Aquí están los comandos esenciales:

1.  **`git clone <url_del_repositorio>`:**  Descarga una copia de tu repositorio remoto (en GitHub) a tu computadora local.  La URL la encuentras en la página principal de tu repositorio en GitHub, en el botón verde que dice "Code".  Copia la URL que aparece (la que termina en `.git`).

    ```bash
    git clone https://github.com/TU_USUARIO/entregable-1-cicd.git.git
    ```

    (Reemplaza `TU_USUARIO` con tu nombre de usuario de GitHub).

2.  **`cd <nombre_del_repositorio>`:**  Navega al directorio de tu repositorio.  "cd" significa "change directory".

    ```bash
    cd entregable-1-cicd
    ```

3.  **`git status`:**  Muestra el estado actual de tu repositorio local.  Te dice qué archivos han sido modificados, cuáles están listos para ser "commited" (guardados), etc.

    ```bash
    git status
    ```

4.  **`git add <archivo>` (o `git add .`)** :  Prepara los archivos modificados para ser "commited".
    *   `git add <archivo>`:  Añade un archivo específico.
    *   `git add .`:  Añade *todos* los archivos modificados y nuevos del directorio actual y subdirectorios.  ¡Úsalo con cuidado!

    ```bash
    git add README.md  # Añade solo el README
    git add .       # Añade todos los archivos modificados
    ```

5.  **`git commit -m "Mensaje descriptivo del cambio"`:**  "Guarda" los cambios que has añadido con `git add`.  El mensaje descriptivo explica *qué* cambiaste y *por qué*.  Sé claro y conciso.

    ```bash
    git commit -m "Actualizar el README con la introducción"
    ```

6.  **`git push origin <rama>`:**  Sube tus cambios "commited" a GitHub (tu repositorio remoto).  `origin` es el nombre por defecto que Git le da a tu repositorio remoto. `main` (o `master`) es el nombre de la rama principal (la rama por defecto).

    ```bash
    git push origin main
    ```
    Te pedirá tu nombre de usuario y contraseña (o token de acceso personal, ver más abajo) de GitHub.

7. **`git pull origin <rama>`:**: *Descarga* los cambios desde el repositorio remoto (GitHub) a tu copia local.  Esto es importante si estás trabajando en equipo, o si has hecho cambios directamente en GitHub (a través de la interfaz web).

    ```bash
    git pull origin main
    ```

**Importante: Tokens de Acceso Personal (en lugar de contraseñas)**
GitHub ya no recomienda usar tu contraseña directamente con `git push`. En su lugar, debes usar un *Token de Acceso Personal* (PAT).

1. **Crear un PAT:**
    * Ve a tu perfil en GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic) > Generate new token > Generate new token (classic)
    * Dale un nombre descriptivo al token (ej. "Acceso Git desde mi PC").
    * En "Select scopes", marca la casilla `repo` y `workflow`. Esto es suficiente para este curso.  *No marques más casillas de las necesarias por seguridad*.
    * Haz clic en "Generate token".
    * *Copia el token generado*.  GitHub no te lo volverá a mostrar.  Guárdalo en un lugar seguro.
2.  **Usar el PAT:** Cuando Git te pida tu contraseña, *pega el token* en su lugar.

### ¿Qué es YAML y por qué es importante para GitHub Actions?

YAML (YAML Ain't Markup Language) es un formato de serialización de datos legible por humanos.  Se usa comúnmente para archivos de configuración.  En GitHub Actions, los archivos YAML definen los *workflows*.

**Características clave de YAML:**

*   **Indentación:**  YAML usa la indentación (espacios, *no tabulaciones*) para definir la estructura de los datos.  Es *muy* importante que la indentación sea correcta.  Un error de indentación puede hacer que tu workflow falle.
*   **Clave-Valor:**  Los datos se organizan en pares clave-valor, similar a un diccionario de Python.
    ```yaml
    nombre: Mi Workflow
    version: 1.0
    ```
*   **Listas:**  Se representan con guiones (`-`).
    ```yaml
    frutas:
      - manzana
      - banana
      - naranja
    ```
*   **Comentarios:**  Comienzan con `#`.
    ```yaml
    # Esto es un comentario
    ```

**¿Por qué YAML para GitHub Actions?**

*   **Legibilidad:**  Es fácil de leer y entender, incluso para personas que no son programadores.
*   **Simplicidad:**  Tiene una sintaxis sencilla y concisa.
*   **Integración:**  GitHub Actions está diseñado para funcionar con archivos YAML.

### ¿Qué es GitHub Actions?

GitHub Actions es una plataforma de *Integración Continua y Entrega Continua (CI/CD)* integrada en GitHub.  Te permite automatizar tareas en tu flujo de desarrollo de software.

*   **Integración Continua (CI):**  La práctica de integrar los cambios de código de todos los desarrolladores en un repositorio compartido (como `main` o `master`) *frecuentemente*.  Cada integración se verifica mediante pruebas automatizadas.  El objetivo es detectar problemas de integración lo antes posible.
*   **Entrega Continua (CD):**  La práctica de automatizar el proceso de lanzamiento de software.  Después de que los cambios de código pasan las pruebas de CI, se pueden desplegar automáticamente a un entorno de pruebas o incluso a producción.

**GitHub Actions te permite definir "workflows" que automatizan estas tareas y muchas más.**

### ¿Por qué GitHub Actions? (Ventajas para el curso)

*   **Simplicidad:**  Es relativamente fácil de aprender y usar, especialmente para tareas básicas.
*   **Capa Gratuita:**  GitHub ofrece una capa gratuita generosa de GitHub Actions, que es más que suficiente para este curso:
    *   **Repositorios Públicos:**  500 MB de almacenamiento y 2000 minutos de ejecución de tareas automatizadas por mes.  Suficiente para proyectos pequeños y medianos.
    *   Puedes ver tu uso en Settings > Billing and plans.
*   **Integración:**  Está integrado directamente en GitHub, por lo que no necesitas configurar servicios externos.

### Conceptos Clave de GitHub Actions (con ejemplos YAML)

*   **Workflow (Flujo de trabajo):**  Un workflow es un proceso automatizado que defines en un archivo YAML.  Es el "contenedor" principal de todo lo que quieres que haga GitHub Actions.  Un workflow puede contener uno o más *jobs* (trabajos).

    ```yaml
    name: Mi Primer Workflow  # Nombre del workflow (opcional, pero recomendado)

    on: push  # Se ejecuta en cada push realizado al repositorio de GitHub

    jobs:
      mi_job:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Hola Mundo"
    ```

    **Explicación:**

    *   **`name`:** Un nombre descriptivo para tu workflow.  Aparece en la pestaña "Actions" de tu repositorio.
    *   **`on`:**  Define *cuándo* se ejecutará el workflow. En este caso, `push` significa que se ejecutará cada vez que se haga un `push` al repositorio (cuando subas cambios).
    *   **`jobs`:**  Aquí se definen los trabajos (jobs) que componen el workflow.
    *  El resto ( `mi_job`, `runs-on`, `steps`, `run`) se explican en los siguientes puntos.

*   **Job (Trabajo):** Un job es un conjunto de *steps* (pasos) que se ejecutan en el mismo *runner* (entorno de ejecución).  Los jobs se pueden ejecutar en paralelo (por defecto) o en secuencia.  Cada job se ejecuta en un entorno limpio (una nueva instancia del runner).

    ```yaml
    jobs:
      construir:  # Nombre del job
        runs-on: windows-latest  # Se ejecuta en Windows
        steps:
          - ...  # Pasos para construir el proyecto

      probar:  # Otro job
        runs-on: macos-latest  # Se ejecuta en macOS
        steps:
          - ...  # Pasos para probar el proyecto

      validar: # Otro job
        runs-on: ubuntu-latest  # Se ejecuta en Ubuntu (Linux)
        steps:
          - ... # Tus pasos aquí
    ```

    **Explicación:**
    *  `jobs`: Es la sección principal donde defines todos tus trabajos.
    * `construir`, `probar`, `validar`: Son *nombres* que tú eliges para identificar cada job. Es importante que sean descriptivos.
    * `runs-on`: Especifica el *tipo* de runner que se usará para ejecutar este job (Windows, macOS, Ubuntu).
    * `steps`: Aquí se definen los pasos individuales que se ejecutarán dentro de este job (se explica más adelante).

    **Ejecución en secuencia (usando `needs`):**

    Por defecto, los jobs se ejecutan en *paralelo*.  Si quieres que se ejecuten en un orden específico (por ejemplo, primero "construir" y *luego* "probar"), usa la palabra clave `needs`:

    ```yaml
    jobs:
      job1:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Job 1"
      job2:
        runs-on: ubuntu-latest
        needs: job1  # job2 se ejecuta *después* de que job1 termine exitosamente
        steps:
          - run: echo "Job 2"
    ```

    **Explicación de `needs`:**

    *   `needs: job1`:  Indica que `job2` *depende* de `job1`.  `job2` no comenzará hasta que `job1` haya terminado correctamente.  Si `job1` falla, `job2` no se ejecutará.  Puedes encadenar dependencias: `job3` podría depender de `job2`, y así sucesivamente.

*   **Step (Paso):** Un step es una tarea individual dentro de un job.  Puede ser:

    *   Una *acción* pre-construida (del Marketplace de GitHub o de tu propio repositorio).
    *   Un comando de shell (un comando que ejecutarías en tu terminal).

    ```yaml
    steps:
      - name: Clonar repositorio  # Nombre descriptivo (opcional)
        uses: actions/checkout@v3  # Acción pre-construida para clonar el código

      - name: Ejecutar comando
        run: echo "Hola desde el step"  # Comando de shell

      - name: Configurar Python
        uses: actions/setup-python@v3 # Acción para configurar Python
        with:
          python-version: '3.9'
    ```

    **Explicación:**

    *   `steps`:  La sección donde defines la secuencia de pasos.
    *   `- name: ...`:  Un nombre descriptivo (opcional, pero muy recomendable) para el paso.  Hace que los logs sean más fáciles de leer.
    *   `uses: ...`:  Indica que este paso usa una *acción*.  `actions/checkout@v3` es una acción muy común que clona tu repositorio en el runner. `@v3` especifica la *versión* de la acción (es importante usar versiones específicas).
    *   `run: ...`:  Indica que este paso ejecuta un comando de shell.
    *   `with:`: Se usa para pasar parámetros a una acción. En el ejemplo, `python-version: '3.9'` le dice a la acción `actions/setup-python` que configure Python 3.9.

*   **Action (Acción):** Una acción es una unidad de código *reutilizable* que realiza una tarea específica.  Puedes usar acciones creadas por la comunidad (del Marketplace de GitHub), crear tus propias acciones, o usar acciones pre-construidas por GitHub (como `actions/checkout` y `actions/setup-python`).

    ```yaml
    steps:
      - uses: actions/checkout@v3  # Acción para clonar el repo (de GitHub)
      - uses: actions/setup-python@v3  # Acción para configurar Python (de GitHub)
      - uses: aws-actions/configure-aws-credentials@v1 # Acción para configurar credenciales de AWS (de la comunidad)
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
    ```

    **Explicación:**

    *   `uses`:  La palabra clave para usar una acción.
    *   `nombre/de/la/accion@version`:  El formato para especificar una acción.
        *   `nombre/de/la/accion`:  El nombre de la acción.  Puede ser una acción oficial de GitHub (`actions/checkout`), una acción de la comunidad (`aws-actions/configure-aws-credentials`), o una acción de tu propio repositorio.
        *   `@version`:  La versión de la acción.  Es *muy importante* usar versiones específicas (como `@v3`) para evitar que cambios inesperados en la acción rompan tu workflow.  Puedes usar una etiqueta de versión (como `@v3`), un hash de commit específico, o una rama (aunque usar una rama es menos recomendado, ya que puede cambiar).
    *   `with:`:  (Opcional)  Permite pasar parámetros de entrada a la acción.  Cada acción define qué parámetros acepta.  En el ejemplo, estamos pasando credenciales de AWS a la acción `aws-actions/configure-aws-credentials`.

*   **Runner (Ejecutor):**  (Ya explicado en la sección anterior).  Es la máquina virtual donde se ejecutan los jobs.  GitHub proporciona runners con Ubuntu Linux (`ubuntu-latest`), Windows (`windows-latest`) y macOS (`macos-latest`).  También puedes usar runners auto-hospedados (máquinas que tú controlas).  Se especifica con `runs-on`.

*   **Event (Evento):** Un evento es un suceso que *desencadena* la ejecución de un workflow.  Hay muchos tipos de eventos, pero los más comunes son:

    *   `push`:  Cuando se suben cambios a una rama (o a `main`/`master`).
    *   `pull_request`:  Cuando se crea o actualiza una Pull Request.
    *   `schedule`:  Ejecución programada (por ejemplo, cada día a una hora específica).
    *   `workflow_dispatch`:  Permite ejecutar el workflow manualmente desde la interfaz de GitHub.

    ```yaml
    on:
      push:  # En cada push
        branches:
          - main
          - desarrollo  # Solo en las ramas 'main' y 'desarrollo'
      pull_request:  # En cada Pull Request
        branches:
          - main   # Solo en Pull Requests a la rama 'main'
      schedule:
        - cron: '0 0 * * *'  # Cada día a medianoche (UTC) - Sintaxis de cron
      workflow_dispatch:  # Permite ejecución manual
    ```

    **Explicación:**

    *   `on`:  La sección principal donde defines los eventos que desencadenan el workflow.
    *   `push`, `pull_request`, `schedule`, `workflow_dispatch`:  Son tipos de eventos.
    *   `branches`:  (Opcional)  Dentro de `push` y `pull_request`, puedes especificar las ramas a las que se aplica el evento.  Si no se especifica, se aplica a *todas* las ramas.
    *   `cron`:  Dentro de `schedule`, se usa la sintaxis de cron para definir la programación.  `'0 0 * * *'` significa "a las 00:00 (medianoche) todos los días".

*   **Artifacts (Artefactos):** Los artefactos son archivos generados por tu workflow que quieres conservar o compartir.  Por ejemplo, un archivo de log, un archivo de configuración, un archivo de salida de una prueba, etc.  Puedes subir artefactos a GitHub para su almacenamiento y descarga.

    ```yaml
    jobs:
      construir:
        runs-on: ubuntu-latest
        steps:
          - run: echo "Construyendo..."
          - run: echo "Generando archivo de log..."
          - name: Subir artefacto
            uses: actions/upload-artifact@v3
            if: always()
            with:
              name: nombre-artefacto
              path: log.json
    ```
    **Explicación:**

    *   `jobs`:  La sección principal donde defines tus trabajos.
    *   `construir`:  El nombre de tu job.
    *   `steps`:  La sección donde defines los pasos de tu job.
    *   `name: Subir artefacto`:  Un paso para subir un artefacto.
    *   `uses: actions/upload-artifact@v3`:  La acción oficial de GitHub para subir artefactos.
    *   `if: always()`:  Un condicional que indica que este paso se ejecutará *siempre*, independientemente del resultado de los pasos anteriores.
    *   `with: name: ...`:  El nombre del artefacto.
    *   `with: path: ...`:  La ruta del archivo que quieres subir.     

## 2. Ejercicio Práctico / Entregable

**Objetivo:** Crear un workflow de GitHub Actions que imprima un mensaje personalizado, la fecha/hora actual y ejecute un script simple de Python que imprima información del sistema operativo.

**Instrucciones:**

1.  **Crea un repositorio en GitHub** (si no lo has hecho ya) siguiendo las instrucciones de la sección "Creación de un Repositorio en GitHub".  Llámalo como quieras (por ejemplo, `entregable-1-cicd`). Asegúrate de inicializarlo con un archivo README. Para este ejercicio, la recomendación es crear un repositorio público para facilitar la visualización y calificación.

2.  **Clona el repositorio a tu computadora local:**

    ```bash
    git clone https://github.com/TU_USUARIO/entregable-1-cicd.git
    cd entregable-1-cicd
    ```

    (Reemplaza `TU_USUARIO` y el nombre del repositorio si es diferente).

3.  **Crea el archivo YAML del workflow:**
    *   Dentro de tu repositorio (en tu computadora), crea la carpeta `.github/workflows` (si no existe).
    *   Dentro de `.github/workflows`, crea un archivo llamado `mi_workflow.yml`.
    *   Copia la siguiente *plantilla* en el archivo `mi_workflow.yml`.  *Tendrás que completar los huecos (marcados con `???`) y responder a las preguntas.*

    ```yaml
    name: ???  # 1. Dale un nombre descriptivo a tu workflow

    on:
      ??? : # 2. ¿Qué evento(s) quieres usar para disparar este workflow? (añade mínimo 2)
            #    Piensa en cuándo quieres que se ejecute (push, pull_request, manualmente...).
        ??? : # 2b. (Opcional) ¿Quieres restringirlo a alguna rama específica?
          - main

    jobs:
      imprimir_info:  # No cambies este nombre, lo usaremos para evaluar
        runs-on: ???  # 3. ¿En qué sistema operativo quieres que se ejecute este job? (ubuntu-latest, windows-latest, macos-latest)

        steps:
          - uses: actions/checkout@v3  # Este paso clona tu repositorio. No lo modifiques.

          - name: Imprimir Mensaje Personalizado
            # 4. Reemplaza [TU NOMBRE COMPLETO] con tu nombre.
            # 5. ¿Qué comando de shell usarías para obtener la fecha y hora?
            run: |
              echo "Hola, soy [TU NOMBRE COMPLETO] y este es mi primer workflow!"  
              echo "La fecha y hora actual es: $(???)"  

          - name: Configurar Python
            uses: actions/setup-python@v3
            with:
              python-version: ??? # 6. Elige una versión de Python (ej: '3.8', '3.9', '3.10', '3.11', '3.12').

          - name: Ejecutar script Python
            run: ??? #7. Completa para ejecutar el script de python. Pista: python <nombre_archivo>
    ```

4.  **Crear el archivo mi_script.py**
    * Dentro de tu repositorio (en tu computadora), crea un archivo llamado `mi_script.py`.
    * Copia el script de python dentro del archivo que acabas de crear.

    ```python
    # mi_script.py
    import platform
    import datetime

    def main():
        print(f"Sistema Operativo: {platform.system()} {platform.release()}")
        print(f"Fecha y hora desde Python: {datetime.datetime.now()}")

    if __name__ == "__main__":
        main()
    ```

5.  **Añade, "commitea" y sube los archivos YAML y py:**

    ```bash
    git add .github/workflows/mi_workflow.yml
    git add mi_script.py
    git commit -m "Completar el workflow del entregable 1"
    git push origin main
    ```

6.  **Ve a tu repositorio en GitHub y verifica que el workflow se haya ejecutado:**
    *   Haz clic en la pestaña "Actions".
    *   Deberías ver tu workflow.
    *   Haz clic en la ejecución más reciente.
    *   Verás el resultado del job "imprimir_info".
    *   **Inspecciona cada paso y mira los logs para entender lo que sucedió y validar el correcto funcionamiento antes de realizar la entrega.**

7.  **Entrega:**
    *   En la página de la ejecución del workflow en GitHub, copia la URL completa de la página.
    *   Envía un correo electrónico a `dhoyoso@eafit.edu.co` con:
        *   **Asunto:** Entregable 1 Intro CI/CD con GitHub Actions
        *   **Cuerpo:**
            *   La URL de la ejecución de tu workflow (valida en una pestaña de incógnito que la URL sea pública y accesible).
            *   *Responde a las siguientes preguntas*:
                1.  ¿Qué eventos configuraste para disparar tu workflow y por qué?
                2.  ¿Qué sistema operativo elegiste para el runner y por qué?
                3.  Menciona al menos 3 acciones pre-construidas que encuentras en el Marketplace de GitHub Actions diferentes a las utilizadas en el workflow de este taller. Describe qué hacen y para qué sirven.
                4.  Investiga, ¿qué es un secreto en GitHub Actions y para qué se usa? ¿Cómo lo usarías en tu workflow?
                5.  Reflexiona sobre el uso de GitHub Actions en tu flujo de trabajo diario. ¿Qué tareas automatizarías? ¿Por qué?

**Criterios de evaluación:**

*   Creación correcta del repositorio.
*   Completar correctamente la plantilla YAML según las instrucciones descritas.
*   Uso correcto de los comandos Git.
*   Ejecución exitosa del workflow.
*   Mensaje personalizado correcto.
*   Respuestas correctas y *completas* a las preguntas del correo.  (Esto es importante para evaluar la comprensión).

**RECUERDA QUE TU REPOSITORIO GITHUB DEBE SER PÚBLICO O DE LO CONTRARIO NO SE PODRÁ CALIFICAR Y PERDERAS LA NOTA**

8. **Información adicional:**
   * Documentación oficial de GitHub Actions: https://docs.github.com/en/actions
   Especialmente la sección 'Quickstart' y 'Learn GitHub Actions'.
   * Ejemplos de workflows: https://github.com/actions/starter-workflows Repositorio de GitHub con templates.