# Simple ML Training Project

## Descripción

Repositorio de práctica para el módulo de DevOps (Máster IA, Cloud Computing y DevOps), siguiendo el tutorial de PontIA. El objetivo es implementar un flujo MLOps realista usando **GitHub Actions**, **MLflow** y buenas prácticas de control de versiones: integración continua, ramas, PRs, variables/secretos, ejecución automática de pipelines y registro de modelos.

---

## Estructura del proyecto

```
├── .github/
│ └── workflows/
│ ├── build.yml # Pipeline de build y training del modelo
│ └── integration.yml # Pipeline de integración continua (primer paso)
├── src/
│ ├── init.py
│ ├── main.py
│ ├── data_loader.py
│ ├── evaluate.py
│ └── model.py
├── model_tests/
│ └── test_model.py
├── scripts/
│ └── register_model.py
├── requirements.txt
└── README.md
```

- **.github/workflows/**: Definición de los pipelines de CI/CD.
- **src/**: Código principal del pipeline de ML.
- **model_tests/**: Tests automatizados para validación del modelo.
- **scripts/**: Scripts auxiliares, como el registro del modelo en MLflow.
- **requirements.txt**: Dependencias del proyecto.

---

## 🔧 Preparación del Repositorio

1. Crear nuevo repositorio con el nombre `pontia-mlops-tutorial-nombre-apellido`.
2. Clonar el repo original de referencia: `pontia-mlops-tutorial`.
3. Copiar el contenido al nuevo repositorio.
4. Añadir `.gitignore`, `requirements.txt`, y actualizar estructura de carpetas.
5. Configurar **secrets** en GitHub:
   - `AZURE_STORAGE_CONNECTION_STRING`
   - `MLFLOW_URL`
   - `EXPERIMENT_NAME`
   - `MODEL_NAME`
6. Probar ejecución local de `main.py` para entrenar y guardar modelo.
7. Subir cambios a GitHub (`push` a `main`).

## 🔁 Flujo de trabajo seguido (paso a paso)

### 1. **Pipeline de Integración Continua**: `integration.yml`

Esta pipeline verifica que el repositorio esté correctamente estructurado antes de integrar cambios a `main`.

📄 El archivo `.github/workflows/integration.yml` incluye:

- `workflow_dispatch`: permite ejecutar la pipeline manualmente desde GitHub Actions.
- Python 3.10.
- Instalación de dependencias desde `requirements.txt`.
- Se ejecuta automáticamente en **push** o **pull request** a la rama `integration`.
- Realiza validaciones básicas:
  - Que exista el archivo `main.py`.
  - Que se puedan instalar las dependencias.
- Añade `continue-on-error: true` e `if: always` para seguir la pipeline aunque fallen los tests.
- Al crear una PR desde `integration` a `main`, se solicita revisión y se valida con comentarios tipo: `Looks good to me`.

💡 **Consejo**: Esta pipeline es clave porque **bloquea los merges directos** a `main` si algo falla. Obliga a trabajar con ramas y revisiones.

---

### 2. **Pipeline de Build y Registro de Modelo**: `build.yml`

⚙️ Esta pipeline entrena y registra el modelo en MLflow automáticamente desde GitHub Actions.

📄 El archivo `.github/workflows/build.yml`:

- Instala dependencias.
- Descarga datasets.
- Llama al script `main.py` para entrenar el modelo.
- Usa `register_model.py` para registrar el modelo en MLflow.
- Usa `run_id` como variable.
- Sube artefactos (`scaler.pkl`, `encoders.pkl`, etc.).
- Corre tests (`test_model.py`).
- Solo se ejecuta con push a la rama `main`.
- Verificación final en MLflow:
  - Registro correcto del modelo.
  - Aparición de artefactos (modelo, scaler, encoder).

---
### 3. **Despliegue del modelo**: `deploy.yml` (pendiente de implementar)

- Crea rama `deploy` y añade un archivo `deploy.yml` dummy.
- Añadir secretos adicionales:
  - `AZURE_CREDENTIALS`, `ACR_USERNAME`, `ACR_PASSWORD`, `ACR_NAME`, `AZURE_RESOURCE_GROUP`
- Variables:
  - `MODEL_NAME`, `MODEL_ALIAS`, `AZURE_CONTAINER_NAME`, `IMAGE_NAME`, `AZURE_REGION`
- Copiar carpeta `deployment/` al root.
- Completar el `Dockerfile`.
- Corregir el error en `register_model.py` reemplazando `model artifact path` por `"model"`.
- Crear PR desde `deploy` a `main`.

---

## ▶️ Ejecución

- Ejecutar `Build Model` desde la rama `deploy`.
- Verificar que se suben correctamente los artefactos en MLflow.
- Ejecutar `Deploy Model` y validar el despliegue exitoso (puede tardar unos 15 min).

📋 Comando para inspeccionar logs del contenedor en Azure:
```
az container logs --resource-group mlflow-rg --name model-api-UCI
```
---

## Problemas habituales y cómo los he resuelto

* ❌ Error ModuleNotFoundError: No module named 'src'
Sucedía al ejecutar la pipeline en GitHub Actions (pero no siempre en local).

* Solución: Ejecutar SIEMPRE los scripts desde la raíz y llamar explícitamente a los scripts con el path completo:
```
python src/main.py
```