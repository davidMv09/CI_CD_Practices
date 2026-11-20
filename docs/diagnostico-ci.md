# Laboratorio de integración continua

<!-- Al terminar, este archivo va al repositorio como docs/diagnostico-ci.md (guía de laboratorio del Tema 3, Parte 3). -->

> **Cómo completar esta plantilla**
>
> Cada texto que está entre los signos **menor que** y **mayor que** se reemplaza por lo que se pide,
> incluidos los dos signos. El resto queda como está.
>
> | Signo | Nombre |
> |:---:|---|
> | `<` | menor que |
> | `>` | mayor que |
>
> Ejemplo, con `mi-usuario` en lugar del usuario de GitHub:
>
> | En la plantilla | Ya completado |
> |---|---|
> | `https://github.com/<usuario>/api-tareas/tree/lab-ci` | `https://github.com/mi-usuario/api-tareas/tree/lab-ci` |
>
> En la vista previa, algunos de esos textos no se muestran, porque se leen como código HTML. En el editor
> siempre se ven.

## 1. Enlaces

| Qué | Enlace |
|---|---|
| Repositorio en el tag `lab-ci` | https://github.com/<usuario>/api-tareas/tree/lab-ci |
| Pull request que el check bloqueó (prueba rota a propósito) | <enlace al pull request> |

## 2. Diagnóstico de los tres pipelines rotos

### Caso 1

**Síntoma** (el paso que falló primero y su mensaje, copiado del log como texto):

```text
Run bash scripts/pruebas.sh
/opt/hostedtoolcache/Python/3.12.16/x64/bin/python: No module named pytest
Error: Process completed with exit code 1.
```

```text
Run actions/upload-artifact@v7
Error: No files were found with the provided path: reportes/. No artifacts will be uploaded.
```

**Causa:** <qué está mal y en qué archivo>
Se cambio el archivo de instalacion de requerimientos.
El archivo requirements-dev.txt

**Corrección:** Se agrego las dependencias necesarias al archivo de requerimientos · Commit: [enlace al commit de la corrección](https://github.com/davidMv09/CI_CD_Practices/commit/94ec5ccc63673ce72b96e05c64b5707c7ee557a2)

**Prevención:** Una validacion de dependecias previo a la corrida.
Chequeo de versiones de las dependencias instaladas.

### Caso 2

**Síntoma** (el paso que falló primero y su mensaje, copiado del log como texto):

```text
Run actions/upload-artifact@v7
Error: No files were found with the provided path: reporte/. No artifacts will be uploaded.
```

**Causa:** La ruta donde se almacena el archivo generado no coincide.
Pruebas.sh apunta a la carpeta `reportes` y el pipeline busca en la carpeta `reporte`

**Corrección:** Se modifico el `pruebas.sh` para que coincida con la carpeta que busca el pipeline · Commit: [enlace al commit de la corrección](https://github.com/davidMv09/CI_CD_Practices/commit/7fa2484d1792ee0c8975350785c7262a7c61ca2a)

**Prevención:** El pipeline debe fallar al publicar los resultados, y por ende el ruleset debe impedir el merge de ese PR.

### Caso 3

**Síntoma** (el paso que falló primero y su mensaje, copiado del log como texto):

```text
Run bash scripts/pruebas.sh
============================= test session starts ==============================
platform linux -- Python 3.12.16, pytest-9.1.1, pluggy-1.7.0
rootdir: /home/runner/work/CI_CD_Practices/CI_CD_Practices
configfile: pyproject.toml
testpaths: tests
plugins: cov-7.1.0
collected 23 items

tests/test_api.py ......................                                 [ 95%]
tests/test_datos_ejemplo.py E                                            [100%]

==================================== ERRORS ====================================
______________ ERROR at setup of test_carga_las_tareas_de_ejemplo ______________

    @pytest.fixture
    def tareas_ejemplo():
>       return json.loads(DATOS.read_text(encoding="utf-8"))
                          ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

tests/test_datos_ejemplo.py:14: 
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
/opt/hostedtoolcache/Python/3.12.16/x64/lib/python3.12/pathlib.py:1027: in read_text
    with self.open(mode='r', encoding=encoding, errors=errors) as f:
         ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = PosixPath('/home/runner/work/CI_CD_Practices/CI_CD_Practices/tests/datos/tareas_ejemplo.json')
mode = 'r', buffering = -1, encoding = 'utf-8', errors = None, newline = None

    def open(self, mode='r', buffering=-1, encoding=None,
             errors=None, newline=None):
        """
        Open the file pointed to by this path and return a file object, as
        the built-in open() function does.
        """
        if "b" not in mode:
            encoding = io.text_encoding(encoding)
>       return io.open(self, mode, buffering, encoding, errors, newline)
               ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
E       FileNotFoundError: [Errno 2] No such file or directory: '/home/runner/work/CI_CD_Practices/CI_CD_Practices/tests/datos/tareas_ejemplo.json'

/opt/hostedtoolcache/Python/3.12.16/x64/lib/python3.12/pathlib.py:1013: FileNotFoundError
- generated xml file: /home/runner/work/CI_CD_Practices/CI_CD_Practices/reportes/junit.xml -
=========================== short test summary info ============================
ERROR tests/test_datos_ejemplo.py::test_carga_las_tareas_de_ejemplo - FileNotFoundError: [Errno 2] No such file or directory: '/home/runner/work/CI_CD_Practices/CI_CD_Practices/tests/datos/tareas_ejemplo.json'
========================= 22 passed, 1 error in 0.26s ==========================
Error: Process completed with exit code 1.
```

**Causa:** El pipeline no encuentra el archivo necesario. El archivo no fue agregado al commit.
El archivo `.gitignore` tiene la linea `*.json`. Lo que hace que el `git add` ignore el archivo y no se agregue al commit.

**Corrección:** Se removio la linea de ignorar archivos .json en el `.gitignore` · Commit: [enlace al commit de la corrección](https://github.com/davidMv09/CI_CD_Practices/commit/4286ebdbb9b6a6ec60d76787b5496481df873b3a)

**Prevención:** La seccion de ejecutar prubeas debe fallar, el pipeline reportar el step como error, y el ruleset debe impedir el merge de ese PR.

## 3. Uso de IA

No se usó IA
