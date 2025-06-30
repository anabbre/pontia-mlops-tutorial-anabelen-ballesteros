
# 🛠️ Problemas Resueltos: Build y Deploy Pipelines

Durante la implementación del flujo CI/CD con GitHub Actions, surgieron algunos errores durante las ejecuciones automáticas de los scripts y workflows. A continuación, se documentan los problemas encontrados y sus respectivas soluciones.

---

## ❌ Problema 1: Error de importación de módulos en `build.yml`

### 🔍 Descripción

Al ejecutar el script principal con:

```bash
python src/main.py
```

se obtenía el siguiente error:

```text
ModuleNotFoundError: No module named 'src'
```

Esto ocurría porque Python no incluye automáticamente el directorio raíz del proyecto (`src/`) en el `PYTHONPATH`.

### ✅ Solución

1. Asegurarse de importar los módulos usando rutas absolutas con prefijo `src.` en todos los archivos Python:

```python
from src.data_loader import load_data, preprocess_data
from src.evaluate import evaluate
from src.model import train_model
```

2. Añadir al inicio del archivo `main.py` la siguiente línea para incluir el path base correctamente:

```python
import sys
from pathlib import Path
sys.path.append(str(Path(__file__).resolve().parent.parent))
```

3. Modificar el `build.yml` para exportar explícitamente el `PYTHONPATH` antes de ejecutar el script:

```yaml
- name: Train and save model
  run: |
    export PYTHONPATH=$PYTHONPATH:$(pwd)/src
    python src/main.py
```

### 🎉 Resultado

Tras aplicar estos cambios, la ejecución del script `main.py` desde GitHub Actions se realizó correctamente, reconociendo todos los módulos y permitiendo que la pipeline de build pasara sin errores.

---

## 🧩 Problema 2: Artefactos del modelo no aparecían en MLflow (`deploy.yml`)

### 🔍 Descripción

La pipeline `deploy.yml` finalizaba sin errores, pero los archivos `scaler.pkl` y `encoders.pkl` no se registraban en MLflow como artefactos.

### 📊 Análisis

El contenedor se desplegaba correctamente, pero estos ficheros no estaban siendo logueados explícitamente con `mlflow.log_artifact()`.

### ✅ Solución

Se añadieron las siguientes líneas en `main.py`:

```python
mlflow.log_artifact(str(MODEL_DIR / "scaler.pkl"), artifact_path="preprocessing")
mlflow.log_artifact(str(MODEL_DIR / "encoders.pkl"), artifact_path="preprocessing")
mlflow.log_artifact(str(model_path))
```

Además, se verificó que todos los cambios estuvieran integrados antes de lanzar la pipeline, ya que el problema también se debía a que el workflow se ejecutaba desde una rama incompleta.

Se ejecutó nuevamente `build.yml` desde la rama `main` y los artefactos fueron correctamente registrados en MLflow.

### 🎉 Resultado

¡Problema resuelto! Todos los artefactos aparecen correctamente en MLflow y el despliegue es funcional.

---

## 📎 Conclusión

Estos problemas se resolvieron mediante:
- Corrección del path base para `PYTHONPATH`.
- Uso explícito de `mlflow.log_artifact` para todos los artefactos necesarios.
- Verificación del estado de las ramas antes de ejecutar las pipelines.

Esto garantiza que el flujo CI/CD y el registro en MLflow se comporten de forma consistente tanto localmente como en GitHub Actions.
