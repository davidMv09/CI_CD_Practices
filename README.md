# API de tareas

Aplicación de laboratorio del curso SOFT-756 (Implementación de CI/CD y Prácticas DevOps). Es una
API pequeña en Python (Flask) que guarda tareas en memoria. El código de la aplicación casi no
cambia durante el curso: lo que crece es el **pipeline** que la construye, la prueba, la empaqueta
y la despliega.

## Endpoints

| Método | Ruta | Respuesta |
|---|---|---|
| GET | `/health` | `{"estado": "ok"}` si la API está viva |
| GET | `/version` | Versión desplegada y ambiente (`APP_VERSION`, `APP_ENV`) |
| GET | `/tareas` | Lista de tareas |
| GET | `/tareas/<id>` | Una tarea, o 404 |
| POST | `/tareas` | Crea una tarea con `{"titulo": "..."}`; 201, o 400 si el título no es válido |

Si la variable `API_TOKEN` tiene valor, `POST /tareas` exige el encabezado
`Authorization: Bearer <token>`. Las lecturas no lo exigen.

## Configuración

| Variable | Para qué sirve | Valor por defecto |
|---|---|---|
| `APP_ENV` | Nombre del ambiente (desarrollo, staging, produccion) | `desarrollo` |
| `APP_VERSION` | Versión que reporta `/version` | `dev` |
| `API_TOKEN` | Token que protege `POST /tareas` | sin valor (no se exige) |

## Correr en local

Requisitos: Python 3.10 o posterior, Git y una terminal Bash (en Windows, **Git Bash**).

```bash
python -m venv .venv
source .venv/bin/activate          # en Windows (Git Bash): source .venv/Scripts/activate
python -m pip install -r requirements-dev.txt
bash scripts/pruebas.sh            # corre las pruebas
python -m flask --app app run --port 8000
```

Con la API en marcha, en otra terminal:

```bash
bash scripts/smoke.sh http://localhost:8000
```

Resultado esperado:

```text
Smoke test contra http://localhost:8000
OK /health
OK /version -> {"ambiente":"desarrollo","version":"dev"}
OK /tareas
Smoke test superado
```

## Estructura

```text
app/            código de la API (config.py lee las variables de entorno)
tests/          pruebas con pytest
scripts/        pruebas.sh y smoke.sh: los mismos comandos en local y en el pipeline
docs/           documentación del flujo de entrega
requirements.txt       dependencias de la aplicación, con versión fija
requirements-dev.txt   dependencias para desarrollar y probar
```

## Despliegue y Rollback

### Dockignore files

|Archivo| Justificacion |
|--|--| 
| .git | La imagen no necesita el control de cambios del repositorio | 
| .github | La imagen no necesita los workflows que ejecuta github |  
| .venv | La imagen no ejecuta virtual enviroments |
| venv | La imagen no ejecuta virtual enviroments |
| **/__pycache__ | Los archivos cacheados no deben estar en una imagen nueva |
|**/*.pyc | Los archivos compilados del programa no deben estar en la imagen |
| .pytest_cache | El cache de los test solo sirven al ejecutar las pruebas |
| .ruff_cache | Las validaciones del codigo no son necesarias para contruir la imagen |
| .coverage | El coverage solo le sirve al pipeline no a la imagen |
| htmlcov |
| reportes | Los reportes generados durante el build no los necesita la imagen. |
| tests | Los test solo son utiles para el build y compilacion, no para generar la imagen |
| docs | El artefacto no necesita la documentacion del repositorio |
| .env |


Cómo saber de qué commit salió una
imagen, cuándo conviene el rollback y cómo se lanza?

```text
El tag que le asignamos a las imagenes son el SHA del commit que se utilizo para generar esa imagen.
El rollback conviene cuando algun pipeline de main esta en verde pero falla en staging o pipelines posteriores.
Puede lanzarse de forma manual en el apartado de Actions o puede configurarse un job que haga la validacion de los steps y se ejecute en caso de algun fallo.
```

### version 12
| IMAGE | ID | DISK USAGE |CONTENT SIZE |EXTRA |
|--|--|--|--|--|
|ghcr.io/davidmv09/ci_cd_practices:bc781dd00193e13ffc9c7b2ab0a82d7325924f31  | 8a7c29fa54a4|1.61GB| 414MB|

### version slim
| IMAGE | ID | DISK USAGE |CONTENT SIZE |EXTRA |
|--|--|--|--|--|
|ghcr.io/davidmv09/ci_cd_practices:61391e10671058b3f7f863d976af263425c752b6| 41dbd67a8385 |188MB |45.5MB|
