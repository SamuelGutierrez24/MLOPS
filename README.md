# Servicio de diagnóstico (reglas simples)

Este repositorio contiene una pequeña aplicación de ejemplo (Flask) que expone una función determinística
de "diagnóstico" médico para propósitos educativos. No hay entrenamiento de modelos ML reales —
la función `simple_diagnosis` es una regla demostrativa que devuelve uno de los siguientes estados:

- `NO ENFERMO`
- `ENFERMEDAD LEVE`
- `ENFERMEDAD AGUDA`
- `ENFERMEDAD CRÓNICA`

El objetivo es proporcionar un servicio que un médico pueda ejecutar localmente (por ejemplo en Docker) para
probar la interfaz y la integración con un front-end simple o con otras herramientas.

---

## Documentación de la función

Inputs (requeridos):

- `temperature` (float) — temperatura corporal en °C.
- `cough` (0|1) — indicador de tos (0 = no, 1 = sí).
- `duration_days` (int) — número de días con síntomas.

Output: cadena con uno de los cuatro estados enumerados arriba.

Errores: si faltan campos o son inválidos, el endpoint devuelve HTTP 400 con `{"error": "..."}`.

---

## Endpoints

- UI web: `GET /` — formulario para ingresar los 3 parámetros y ver el resultado.
- API JSON: `POST /api/predict` — espera JSON con los campos `temperature`, `cough`, `duration_days`.

Ejemplo request JSON:

```json
{"temperature": 38.5, "cough": 1, "duration_days": 3}
```

Ejemplo response:

```json
{"prediction": "ENFERMEDAD AGUDA", "inputs": {"temperature":38.5,"cough":1,"duration_days":3}}
```

---

## Ejecutación con Docker

Estos pasos asumen que tienes Docker instalado y corriendo en tu máquina.

1) Construir la imagen (desde la raíz del repo donde están `Dockerfile` y `requirements.txt`):

```powershell
docker build -t mlops-diagnosis:latest .
```

2) Ejecutar el contenedor (mapear puerto 5000):

Ejecución en segundo plano:
```powershell
docker run -d --name mlops-diagnosis -p 5000:5000 mlops-diagnosis:latest
```
Ejecutar el contenedor en primer plano (verás logs en la consola)
 ```powershell
docker run --rm -p 5000:5000 mlops-diagnosis:latest
```

3) Abrir la UI en el navegador: `http://localhost:5000/`.

4) Probar la API (ejemplo PowerShell / curl):

PowerShell:
```powershell
$body = @{ temperature = 38.5; cough = 1; duration_days = 3 } | ConvertTo-Json
Invoke-RestMethod -Method Post -Uri http://localhost:5000/api/predict -Body $body -ContentType 'application/json'
```

curl (bash):
```bash
curl -X POST http://localhost:5000/api/predict -H "Content-Type: application/json" -d '{"temperature":38.5,"cough":1,"duration_days":3}'
```

---

## Notas
- Si modificas el código y quieres probar los cambios en la imagen, reconstruye:

```powershell
docker build -t mlops-diagnosis:latest .
```
- Si quieres detener el contenedor ejecutando el proyecto (si esta en segundo plano):
```powershell
docker stop mlops-diagnosis.
```

---

## Desarrollo rápido sin Docker

1. Crear y activar virtualenv (ubicado desde la carpeta fuente del proyecto):

```bash
python -m venv .venv
source .venv/bin/activate   # bash/WSL/macOS
# o en PowerShell: .\.venv\Scripts\Activate.ps1
```

2. Instalar dependencias:

```bash
pip install -r requirements.txt
```

3. Ejecutar en modo dev:

```bash
cd src
python main.py
```

---

## Pruebas sugeridas

Para que pruebes diferentes resultados aqui hay 4 diferentes consultas que puedes hacer:

* Inputs: {"Temperatura": 38.5, "Tos": 1, "Duración de sintomas": 3}

    Respuesta esperada: ENFERMEDAD AGUDA
    
* Inputs: {"Temperatura": 36.6, "Tos": 0, "Duración de sintomas": 0}

    Respuesta esperada: NO ENFERMO

* Inputs: {"Temperatura": 39, "Tos": 1, "Duración de sintomas": 30}

    Respuesta esperada: ENFERMEDAD CRÓNICA
* Inputs: {"Temperatura": 36.6, "Tos": 1, "Duración de sintomas": 15}

    Respuesta esperada: ENFERMEDAD LEVE


---
