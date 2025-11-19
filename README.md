---
# 🛠️ Documentación: Automatización de Estadísticas de Perfil (`github-stats`)

Este repositorio contiene la lógica Python, las plantillas SVG y el flujo de trabajo de GitHub Actions que genera y actualiza automáticamente las insignias de estadísticas visibles en mi perfil principal (`CCDani`).

---
## 1. 💡 Funcionamiento del Sistema y Estructura

Para que las insignias aparezcan en tu página principal de GitHub, se requieren dos repositorios:

* **Repositorio de Lógica (github-stats) (Este Repo):**  Contiene el *script* de Python que, ejecutado diariamente por GitHub Actions, consulta la API de GitHub, genera los archivos SVG con los datos actualizados y los sube a la carpeta `/generated`.
* **Repositorio de Perfil (`CCDani/CCDani`):** El `README.md` de mi perfil principal **enlaza directamente** a las imágenes generadas en la carpeta `/generated` de este repositorio.

---

## 2. ⚙️ Configuración y Seguridad (GitHub Actions)

Para que el sistema funcione correctamente y de forma segura, se requiere la siguiente configuración:

### A. Permisos Críticos del Token PAT

El script necesita un **Token de Acceso Personal (PAT)**, guardado como un Secreto (ej., `PAT_STATS`), con permisos explícitos de lectura:

* **`repo`**: Para leer estrellas, *forks* y las estadísticas de repositorio.
* **`user`**: Para leer el historial de contribuciones del perfil.

* Copia: Copia el token generado inmediatamente.


## 3. ⚙️ Configuración de Permisos de GitHub Actions
Para que el flujo de trabajo (schedule.yml) pueda subir los archivos generados y usar tu token de forma segura, los permisos deben configurarse en dos niveles:

### A. Permisos Generales del Repositorio (GUI)Estos permisos son obligatorios para permitir la subida de archivos generados: 

Ve a Settings (Configuración) del repositorio, Actions, General. Baja hasta la sección Workflow permissions (Permisos del Flujo de Trabajo).Selecciona la opción: "Read and write permissions" (Permisos de lectura y escritura).Esto otorga al GITHUB_TOKEN permisos suficientes para que la acción de commit pueda escribir y hacer push en este repositorio.

### B. Configuración del Flujo de Trabajo (`schedule.yml`)

El siguiente código es el flujo de trabajo final que garantiza la funcionalidad y la integridad de la subida.

```yaml
name: Generar Imágenes de Estadísticas

on:
  push:
    branches: [ master ]
  schedule:
    - cron: "5 0 * * *"
  workflow_dispatch:

permissions:
  contents: write # Permiso necesario para el commit y push.
  models: read    # Configuración que asegura la sintaxis válida.

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Obtener código del repositorio
        uses: actions/checkout@v4

      - name: Configurar Python 3.8
        uses: actions/setup-python@v5
        with:
          python-version: '3.8'

      - name: Instalar dependencias
        run: |
          python3 -m pip install -r requirements.txt

      - name: Generar imágenes
        run: |
          python3 generate_images.py
        env:
          # Usa el Token Personal con permisos de lectura
          ACCESS_TOKEN: ${{ secrets.PAT_STATS }} # PAT_STATS es la variable asociada a tu token (privada y segura)
          EXCLUDED: ${{ secrets.EXCLUDED }}
          EXCLUDED_LANGS: ${{ secrets.EXCLUDED_LANGS }}
          EXCLUDE_FORKED_REPOS: true

      - name: Commit y Subir archivos generados (Generar el 'push')
        uses: stefanzweifel/git-auto-commit-action@v5
        with:
          commit_message: '🤖 Automated: Actualización de insignias de estadísticas'
          file_pattern: 'generated/*.svg'
          # Solución al error de sincronización de Git
          push_options: '--force'
