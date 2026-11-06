# Taller Entregable 1: Introducción a Git, GitHub y la automatización con GitHub Actions

Este es el primer entregable de tu curso de CI/CD. Aprenderás los fundamentos de GitHub, el control de versiones con Git, la configuración con YAML y cómo automatizar tareas con GitHub Actions.  ¡No te preocupes si no tienes experiencia previa, te guiaremos paso a paso!

**Alcance:** Este taller es introductorio: ejecutarás tu primera automatización, sin construir todavía un pipeline de pruebas ni desplegar una aplicación. El Taller 2 se enfoca en Integración Continua (CI) y el Taller 3 en CD.

## 1. Introducción a GitHub y GitHub Actions

### ¿Qué es GitHub?

GitHub es una plataforma en la nube para alojar y gestionar código usando un sistema de control de versiones llamado Git.  Piensa en GitHub como un "Google Drive para código", pero con superpoderes:

*   **Control de Versiones:**  Git registra cada cambio que haces en tu código.  Puedes volver a versiones anteriores, ver quién hizo qué cambios y cuándo, y trabajar en diferentes "ramas" de tu proyecto sin afectar la versión principal.
*   **Colaboración:**  GitHub facilita el trabajo en equipo.  Varias personas pueden trabajar en el mismo proyecto simultáneamente, combinar sus cambios y resolver conflictos.
*   **Comunidad:**  GitHub es una red social para desarrolladores. Puedes compartir tu código, contribuir a proyectos de otros, y encontrar soluciones a problemas.
* **Almacenamiento en la Nube**: Tu código se guarda en los servidores de GitHub, lo que te permite acceder a él desde cualquier dispositivo con conexión a internet, y proporciona un respaldo seguro.

### ¿Qué es Git?

Git es el *sistema* de control de versiones. Es el motor que está por debajo. GitHub es la *plataforma* que usa Git y le añade una interfaz web, herramientas de colaboración, etc. Git puede usarse desde la línea de comandos o desde interfaces gráficas, como VS Code y GitHub Desktop. En este taller usaremos comandos de Git y la interfaz web de GitHub.

### Creación de un Repositorio en GitHub

1.  **Crea una cuenta en GitHub:**  Ve a [https://github.com/](https://github.com/) y regístrate. Es gratis.
2.  **Crea un nuevo repositorio:**
    *   Una vez que hayas iniciado sesión, haz clic en el botón "+" en la esquina superior derecha y selecciona "New repository".  También puedes ir directamente a [https://github.com/new](https://github.com/new).
    *   **Repository name:**  Dale un nombre a tu repositorio (por ejemplo, `entregable-1-cicd`).  *No uses espacios, usa guiones (`-`) en su lugar*.
    *   **Description (opcional):**  Añade una breve descripción (por ejemplo, "Primer entregable del curso de CI/CD").
    *   **Public/Private:**
        *   **Public:**  Cualquiera puede ver tu código (pero solo tú puedes modificarlo, a menos que des permiso a otros).  Ideal para proyectos de código abierto.
        *   **Private:**  Solo tú (y las personas que invites) pueden ver y modificar el código.
        *   **Para este taller, el repositorio debe ser público para poder calificarlo. Un repositorio privado no cumple el requisito de entrega.**
    *   **Initialize this repository with:**
        *   **Add a README file:**  Marca esta casilla.  Un archivo README es una descripción de tu proyecto.  Es una buena práctica tener uno.
    *   Haz clic en "Create repository".

### Comandos Básicos de Git (Versionamiento)

Para interactuar con tu repositorio de GitHub desde tu computadora, necesitarás tener Git instalado y usar la línea de comandos (Terminal en macOS/Linux; PowerShell, Git Bash o Command Prompt en Windows). Los comandos de Git de esta guía funcionan en esos sistemas. Esto es independiente del runner Ubuntu que usaremos en GitHub Actions. Aquí están los comandos esenciales:

1.  **`git clone <url_del_repositorio>`:**  Descarga una copia de tu repositorio remoto (en GitHub) a tu computadora local.  La URL la encuentras en la página principal de tu repositorio en GitHub, en el botón verde que dice "Code".  Copia la URL que aparece (la que termina en `.git`).

    ```bash
    git clone https://github.com/TU_USUARIO/entregable-1-cicd.git
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
    Si no tienes la autenticación configurada, Git solicitará credenciales o abrirá un inicio de sesión mediante tu gestor de credenciales. Para HTTPS, si solicita una contraseña, debes usar un token de acceso personal, no tu contraseña de GitHub (ver más abajo).

7. **`git pull origin <rama>`:**: *Descarga* los cambios desde el repositorio remoto (GitHub) a tu copia local.  Esto es importante si estás trabajando en equipo, o si has hecho cambios directamente en GitHub (a través de la interfaz web).

    ```bash
    git pull origin main
    ```

**Importante: Tokens de Acceso Personal (en lugar de contraseñas)**
GitHub no admite la contraseña de tu cuenta para autenticar operaciones de Git por HTTPS. Puedes usar un *Token de Acceso Personal* (PAT). El token sirve para autorizar el `push` desde tu computadora; **no necesitas agregarlo al workflow de este taller**. `actions/checkout` utiliza automáticamente el `GITHUB_TOKEN` que GitHub proporciona para la ejecución.

1. **Crear un PAT clásico (si necesitas autenticarte manualmente):**
    * Ve a tu perfil en GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic) > Generate new token > Generate new token (classic)
    * Dale un nombre descriptivo al token (ej. "Acceso Git desde mi PC").
  * Configura un vencimiento corto que cubra el taller, por ejemplo, 30 días. Revócalo cuando ya no lo necesites.
  * En "Select scopes", selecciona `public_repo` (dentro de `repo`) para subir cambios a repositorios públicos, y `workflow` para crear o modificar archivos en `.github/workflows`. No necesitas seleccionar todo `repo`: ese alcance también permite acceso a repositorios privados. **Un PAT clásico no se limita a un único repositorio; no marques permisos adicionales.**
    * Haz clic en "Generate token".
  * *Copia el token generado*. GitHub no te lo volverá a mostrar. Guárdalo en un gestor de contraseñas o credenciales seguro, no en un archivo del proyecto.
2.  **Usar el PAT:** Cuando Git solicite credenciales para HTTPS, introduce tu usuario de GitHub y, en el campo de contraseña, *pega el token*. Es normal que el token no se vea mientras lo introduces. Si ya hay credenciales guardadas, puede que Git no vuelva a solicitarlas.

**Alternativas para profundizar:** GitHub recomienda [Git Credential Manager](https://github.com/git-ecosystem/git-credential-manager) o [GitHub CLI](https://cli.github.com/) para iniciar sesión desde la computadora sin copiar manualmente un PAT. También recomienda los PAT *fine-grained* cuando sea posible: para tu propio repositorio, selecciona únicamente el del taller y concede `Contents: Read and write` y `Workflows: Read and write`. Si usas un repositorio de una organización, sus políticas pueden exigir aprobación del token o autorización SSO, o restringir los PAT clásicos. Consulta la [documentación oficial de PAT](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens).

> **Seguridad y privacidad:** Nunca publiques PAT, contraseñas, claves de acceso ni otras credenciales en código, YAML, README, capturas, artefactos, logs o en la URL del repositorio remoto. Tu repositorio y los logs de Actions serán públicos, y tu nombre completo quedará visible como autor del ejercicio: no añadas documentos de identidad, teléfonos, direcciones ni otros datos personales. Si expones una credencial, revócala o rótala inmediatamente; borrarla del último archivo no la elimina del historial de Git.

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
*   **Entrega Continua (Continuous Delivery):** Después de pasar CI, el software se construye y prepara automáticamente para poder liberarlo. El despliegue a producción requiere una decisión o aprobación manual.
*   **Despliegue Continuo (Continuous Deployment):** Cada cambio que supera las validaciones se despliega automáticamente a producción, sin una aprobación manual por cambio.

La sigla **CD** puede referirse a entrega o despliegue continuo; la diferencia es si existe esa aprobación manual para producción. Un workflow que solo imprime mensajes, como el de este taller, es una automatización introductoria, no un pipeline completo de CI/CD.

**GitHub Actions te permite definir "workflows" que automatizan estas tareas y muchas más.**

### ¿Por qué GitHub Actions? (Ventajas para el curso)

*   **Simplicidad:**  Es relativamente fácil de aprender y usar, especialmente para tareas básicas.
*   **Capa Gratuita:**  GitHub ofrece una capa gratuita generosa de GitHub Actions, que es más que suficiente para este curso:
  *   **Repositorios públicos:** El uso de runners estándar alojados por GitHub, como `ubuntu-latest`, es gratuito y no consume una cuota mensual de minutos. Esto no significa que no existan límites técnicos de ejecución.
  *   **Almacenamiento:** GitHub Free incluye 500 MB para artefactos, compartidos con GitHub Packages; no es el almacenamiento del código del repositorio. Las cuotas de almacenamiento se gestionan por separado del tiempo de ejecución.
  *   Los repositorios privados tienen uso de cómputo incluido según el plan; los runners de mayor capacidad (*larger runners*) pueden generar cargos incluso en repositorios públicos. No son necesarios para este taller.
  *   Consulta el uso en la sección de facturación de tu cuenta y las condiciones vigentes en la [documentación de facturación de GitHub Actions](https://docs.github.com/en/billing/concepts/product-billing/github-actions).
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
        uses: actions/checkout@v7  # Acción pre-construida para clonar el código

      - name: Ejecutar comando
        run: echo "Hola desde el step"  # Comando de shell

      - name: Configurar Python
        uses: actions/setup-python@v7 # Acción para configurar Python
        with:
          python-version: '3.13'
    ```

    **Explicación:**

    *   `steps`:  La sección donde defines la secuencia de pasos.
    *   `- name: ...`:  Un nombre descriptivo (opcional, pero muy recomendable) para el paso.  Hace que los logs sean más fáciles de leer.
    *   `uses: ...`:  Indica que este paso usa una *acción*. `actions/checkout@v7` descarga el código de tu repositorio en el runner. `@v7` selecciona la versión mayor de la acción.
    *   `run: ...`:  Indica que este paso ejecuta un comando de shell.
    *   `with:`: Se usa para pasar parámetros a una acción. En el ejemplo, `python-version: '3.13'` le dice a la acción `actions/setup-python` que configure Python 3.13. `platform` y `datetime` forman parte de su biblioteca estándar, así que el script del taller no necesita instalar paquetes con `pip`.

*   **Action (Acción):** Una acción es una unidad de código *reutilizable* que realiza una tarea específica.  Puedes usar acciones creadas por la comunidad (del Marketplace de GitHub), crear tus propias acciones, o usar acciones pre-construidas por GitHub (como `actions/checkout` y `actions/setup-python`).

    ```yaml
    steps:
      - uses: actions/checkout@v7  # Acción para clonar el repo (de GitHub)
      - uses: actions/setup-python@v7  # Acción para configurar Python (de GitHub)
        with:
          python-version: '3.13'
      - uses: aws-actions/configure-aws-credentials@v6.3.0 # Acción mantenida por AWS
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
    ```

    **Explicación:**

    *   `uses`:  La palabra clave para usar una acción.
    *   `nombre/de/la/accion@version`:  El formato para especificar una acción.
        *   `nombre/de/la/accion`:  El nombre de la acción.  Puede ser una acción oficial de GitHub (`actions/checkout`), una acción de la comunidad (`aws-actions/configure-aws-credentials`), o una acción de tu propio repositorio.
        *   `@version`: La referencia de la acción. En este taller usamos etiquetas de versión mayor (como `@v7`) por simplicidad; estas pueden recibir actualizaciones compatibles, no fijan un código inmutable. Una etiqueta de versión completa (como `@v6.3.0`) identifica una publicación concreta; para fijar exactamente el código de una acción, se usa el SHA completo de un commit. Evita referenciar ramas como `@main`, que pueden cambiar sin una nueva versión.
    *   `with:`:  (Opcional)  Permite pasar parámetros de entrada a la acción.  Cada acción define qué parámetros acepta.  En el ejemplo, estamos pasando credenciales de AWS a la acción `aws-actions/configure-aws-credentials`.

      **Este ejemplo de AWS es solo conceptual:** No necesitas una cuenta de AWS ni debes copiar ese paso al entregable. Requiere credenciales válidas configuradas como secretos; para despliegues reales, se recomienda OIDC con credenciales temporales en lugar de claves de larga duración.

    **Secretos en GitHub Actions:** Son valores sensibles que configuras fuera del código, en el repositorio: Settings > Secrets and variables > Actions > New repository secret. Se referencian con expresiones como `${{ secrets.AWS_SECRET_ACCESS_KEY }}` y se proporcionan al workflow durante la ejecución. GitHub intenta ocultarlos en los logs, pero no debes imprimirlos: el enmascaramiento no garantiza protección si los transformas o los escribes en archivos o artefactos. El ejercicio de este taller no requiere crear secretos.

    **Compatibilidad de las acciones:** Los ejemplos usan `actions/checkout@v7`, `actions/setup-python@v7` y `actions/upload-artifact@v7`, versiones publicadas para GitHub.com. Utilizan Node.js 24 internamente, sin que debas instalar Node.js ni cambiar el script Python. Los runners estándar alojados por GitHub se mantienen actualizados. Si profundizas con runners auto-hospedados, estas acciones requieren un runner compatible con Node.js 24 (versión 2.327.1 o posterior); `upload-artifact@v4` y posteriores no están soportadas en GitHub Enterprise Server. Este taller utiliza GitHub.com y `ubuntu-latest`, no esos entornos.

*   **Runner (Ejecutor):**  (Ya explicado en la sección anterior).  Es la máquina virtual donde se ejecutan los jobs.  GitHub proporciona runners con Ubuntu Linux (`ubuntu-latest`), Windows (`windows-latest`) y macOS (`macos-latest`).  También puedes usar runners auto-hospedados (máquinas que tú controlas).  Se especifica con `runs-on`.

    **Para el entregable usaremos `ubuntu-latest`, aunque tu computadora sea Windows o macOS.** Los pasos `run` se ejecutan en el runner de GitHub, no en tu computadora. En Ubuntu/macOS el shell predeterminado normalmente es Bash; en Windows es PowerShell. Por eso los comandos de shell pueden cambiar al cambiar el runner. En este taller fijaremos también `shell: bash` para el paso del mensaje y la fecha; `$(date)` obtiene la fecha y hora del runner, no necesariamente la hora local de tu computadora.

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
          - run: printf '{"mensaje":"Ejemplo de artefacto"}\n' > log.json
          - name: Subir artefacto
            uses: actions/upload-artifact@v7
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
    *   El paso con `printf` crea `log.json` antes de subirlo. Anunciar la generación con `echo` sin escribir un archivo no crea un artefacto.
    *   `uses: actions/upload-artifact@v7`: La acción oficial de GitHub para subir artefactos.
    *   `if: always()`: Permite ejecutar este paso aunque hayan fallado pasos anteriores; no crea el archivo si falta ni garantiza que la subida tenga éxito.
    *   `with: name: ...`:  El nombre del artefacto.
    *   `with: path: ...`:  La ruta del archivo que quieres subir.     

## 2. Ejercicio Práctico / Entregable

**Objetivo:** Crear y verificar una automatización introductoria de GitHub Actions que imprima un mensaje personalizado, la fecha/hora actual y ejecute un script simple de Python que imprima información del sistema operativo del runner.

**Configuración del ejercicio:** Runner `ubuntu-latest` y shell Bash para el mensaje y la fecha. Se recomienda Python **3.13** (`python-version: '3.13'`): es una versión mantenida, admitida por `setup-python` y compatible con todo el script del taller. No necesitas instalar Ubuntu ni Python en tu computadora para ejecutarlo en GitHub Actions. Si quieres probar el script localmente, sí necesitas Python instalado. Consulta el [estado de las versiones de Python](https://devguide.python.org/versions/) antes de elegir otra versión.

**Modalidad e identificación:** Puedes realizar este taller individualmente o en pareja (máximo dos integrantes). Para una pareja basta un único repositorio público y un entregable conjunto. Identifica a todos los integrantes con sus nombres completos en el saludo del workflow, en `AUTOR` y en una sección **Integrantes** del README. Para parejas, separa los nombres con ` | ` (espacio, barra vertical, espacio) y conserva exactamente el mismo texto y orden en los tres lugares.

Ejemplo individual: `Ana María Pérez`. Ejemplo en pareja: `Ana María Pérez | Luis Andrés Gómez`. `AUTOR` sigue siendo una cadena de texto, no una lista. En el README, añade también el usuario de GitHub de cada integrante, indicando quién es el propietario del repositorio. No incluyas credenciales ni datos personales adicionales.

**Instrucciones:**

1.  **Crea un repositorio en GitHub** (si no lo has hecho ya) siguiendo las instrucciones de la sección "Creación de un Repositorio en GitHub". Llámalo como quieras (por ejemplo, `entregable-1-cicd`). Asegúrate de inicializarlo con un archivo README. **Debe ser público para poder calificarlo.**

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
    *   Configura dos eventos separados: **`push`** (al subir cambios a `main`) y **`workflow_dispatch`** (ejecución manual). Cada evento es una clave distinta dentro de `on`, al mismo nivel de indentación. `branches` se aplica a `push`; no lo añadas a `workflow_dispatch`. Puedes añadir `pull_request` como evento adicional.
    *   Mantén el runner `ubuntu-latest` y el shell `bash`. Completa `$(???)` con el comando de fecha para Bash en Ubuntu: `date`. Escribe la versión de Python entre comillas, por ejemplo `'3.13'`, para que YAML la trate como texto.

    ```yaml
    name: ???  # 1. Dale un nombre descriptivo a tu workflow

    on:
      ???: # 2. Evento al subir cambios: push
        branches:
          - main
      ???: # 2b. Evento de ejecución manual: workflow_dispatch

    permissions:
      contents: read

    jobs:
      imprimir_info:  # No cambies este nombre, lo usaremos para evaluar
        runs-on: ubuntu-latest

        steps:
          - uses: actions/checkout@v7  # Este paso clona tu repositorio. No lo modifiques.

          - name: Imprimir Mensaje Personalizado
            shell: bash
            # 4. Reemplaza [NOMBRES COMPLETOS] con los nombres de todos los integrantes.
            # 5. ¿Qué comando de shell usarías para obtener la fecha y hora?
            run: |
              echo "Hola, soy [NOMBRES COMPLETOS] y este es mi primer workflow!"  
              echo "La fecha y hora actual es: $(???)"  

          - name: Configurar Python
            uses: actions/setup-python@v7
            with:
              python-version: ??? # 6. Usa una versión mantenida; recomendada: '3.13'.

          - name: Ejecutar script Python
            run: ??? #7. Completa para ejecutar el script de python. Pista: python <nombre_archivo>
    ```

    `permissions: contents: read` concede al `GITHUB_TOKEN` solo lectura del contenido del repositorio, suficiente para descargar el código en este ejercicio. El PAT de tu computadora y el `GITHUB_TOKEN` del workflow son credenciales distintas.

    Para parejas, adapta el saludo a `echo "Hola, somos Ana María Pérez | Luis Andrés Gómez y este es nuestro primer workflow!"`, usando sus nombres reales. Para entrega individual conserva `Hola, soy ... y este es mi primer workflow!`. El texto de los nombres, sin el resto del saludo, debe coincidir exactamente con `AUTOR` y con los nombres declarados en el README.

4.  **Crear el archivo mi_script.py**
    * Dentro de tu repositorio (en tu computadora), crea un archivo llamado `mi_script.py`.
    * Copia el script de python dentro del archivo que acabas de crear.

    ```python
    # mi_script.py
    import platform
    import datetime

    AUTOR = "???"  # Nombres completos del saludo y del README, en el mismo orden; para parejas usa "Nombre 1 | Nombre 2".

    def main():
        print(f"Autor: {AUTOR}")
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
    *   Comprueba que aparecen tu saludo, la fecha/hora y las tres líneas del script Python. El sistema operativo mostrado debe ser Linux, incluso si tu computadora usa Windows o macOS.
    *   Comprueba que el saludo y la línea `Autor:` muestran a todos los integrantes, con los mismos nombres y orden del README. Si cambias los nombres, el workflow o el script, vuelve a ejecutarlo antes de entregar; una ejecución anterior con otros autores no demuestra el funcionamiento de tu entrega actual.
    *   Prueba también el evento manual: en Actions, selecciona tu workflow y haz clic en **Run workflow**, elige `main` y confirma. Para que aparezca ese botón, el archivo debe estar en la rama predeterminada del repositorio y contener `workflow_dispatch`. En este taller usamos `main`; verifica que sea tu rama predeterminada.

7.  **Responde las preguntas en tu README.md:**
    *   Incluye primero una sección **Integrantes** con el texto de nombres usado en `AUTOR` y, para cada persona, su nombre completo y usuario de GitHub. Si trabajas en pareja, identifica a ambos, no solo al propietario del repositorio.
    *   Abre el archivo `README.md` de tu repositorio y añade tus respuestas a las siguientes preguntas. Puedes estructurarlas como prefieras (con subtítulos, numeradas, en párrafos, etc.).
        1.  ¿Qué eventos configuraste para disparar tu workflow y por qué?
        2.  ¿Qué sistema operativo utiliza el runner `ubuntu-latest` y por qué puede ejecutar tu workflow aunque tu computadora sea Windows o macOS? ¿Qué relación tiene el runner con el shell de los pasos `run`?
        3.  Menciona al menos 3 acciones pre-construidas del Marketplace de GitHub Actions **no mencionadas en este material**. Excluye `actions/checkout`, `actions/setup-python`, `actions/upload-artifact` y `aws-actions/configure-aws-credentials`, aunque alguna solo aparezca como ejemplo conceptual. Describe qué hacen y para qué sirven.
        4.  ¿Qué es un secreto en GitHub Actions y para qué se usa? ¿Cómo lo usarías en tu workflow?
        5.  Reflexiona sobre el uso de GitHub Actions en tu flujo de trabajo diario. ¿Qué tareas automatizarías? ¿Por qué?
    *   En pareja, las respuestas 1 a 4 pueden ser conjuntas. En la pregunta 5, identifica la reflexión de cada integrante por su nombre; entre ambas deben incluir al menos dos ideas concretas con justificación. Para entrega individual, incluye esas dos ideas en tu reflexión.
    *   Haz commit y push del README actualizado:

          ```bash
          git add README.md
          git commit -m "Agregar respuestas al README"
          git push origin main
          ```

8.  **Entrega:**
    *   Ingresa al buzón del **Entregable 1** en **EAFIT Interactiva** y envía la URL de tu repositorio de GitHub (ejemplo: `https://github.com/TU_USUARIO/entregable-1-cicd`).
    *   Verifica en una pestaña de incógnito que el repositorio es público y accesible antes de hacer la entrega.
    *   En pareja, cada integrante envía la misma URL en su buzón de entrega. No es necesario duplicar el repositorio.
    *   Incluye en el README un enlace a una ejecución exitosa de `mi_workflow.yml` que corresponda al workflow y al script entregados y muestre los autores correctos. Puedes copiar la URL al abrir esa ejecución en Actions; GitHub permite ver allí el commit asociado y los logs. Actualizar solo el README después no invalida una ejecución si el workflow y el script no cambiaron.

    **Criterios de evaluación:**

      *   Repositorio público con `README.md`, `mi_workflow.yml` y `mi_script.py`.
      *   Plantilla YAML completada correctamente (nombre, eventos `push` y `workflow_dispatch`, runner `ubuntu-latest`, nombre personalizado, versión mantenida de Python, ejecución del script).
      *   Variable `AUTOR` en `mi_script.py` reemplazada con los nombres completos de todos los integrantes, coincidiendo exactamente con el saludo del workflow y con la identificación del README (no con el título `name` del workflow).
      *   Ejecución exitosa del workflow entregado, con los autores correctos en los logs y enlace a esa ejecución en el README.
      *   Respuestas a las 5 preguntas incluidas en el `README.md`.

      **RECUERDA QUE TU REPOSITORIO GITHUB DEBE SER PÚBLICO O DE LO CONTRARIO NO SE PODRÁ CALIFICAR Y PERDERÁS LA NOTA**

9. **Información adicional:**
   * Documentación oficial de GitHub Actions: https://docs.github.com/en/actions
   Especialmente la sección 'Quickstart' y 'Learn GitHub Actions'.
   * Ejemplos de workflows: https://github.com/actions/starter-workflows Repositorio de GitHub con templates.
   * Versiones y requisitos de las acciones: https://github.com/actions/checkout · https://github.com/actions/setup-python · https://github.com/actions/upload-artifact
   * Versiones mantenidas de Python: https://devguide.python.org/versions/
   * Facturación de GitHub Actions: https://docs.github.com/en/billing/concepts/product-billing/github-actions
   * Autenticación y PAT: https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/managing-your-personal-access-tokens