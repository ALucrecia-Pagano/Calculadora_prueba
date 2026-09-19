# Calculadora — FastAPI + HTML estático

Calculadora web simple, hecha con dos partes independientes:

- **`backend/`** — una API en [FastAPI](https://fastapi.tiangolo.com/) que recibe dos números y una operación, y devuelve el resultado. No sabe nada de HTML ni de botones.
- **`frontend/`** — una página HTML/CSS/JavaScript estática (sin frameworks ni librerías externas) que le pide los cálculos a esa API.

Ambas partes se comunican por HTTP: el front nunca calcula nada, solo manda los datos y muestra la respuesta.

## Operaciones disponibles

Suma, resta, multiplicación, división, potencia, raíz cuadrada y porcentaje (`a`% de `b`).

## Funcionalidades

- Validación de datos de entrada (números finitos) y de casos matemáticos imposibles (dividir por cero, raíz de un número negativo, potencia con base negativa y exponente no entero, resultados demasiado grandes para representar), devolviendo siempre un error entendible en vez de que el servidor se rompa.
- Historial de las últimas operaciones y un gráfico de barras con la distribución por tipo de operación, guardados en el navegador (`localStorage`) — no requieren base de datos ni configuración extra.
- Tests automáticos del backend con `pytest` (58 casos, incluyendo los casos borde de cada operación).

## Cómo probarlo en tu computadora (sin Docker ni EasyPanel)

Necesitás tener **Python 3** instalado. No hace falta Docker para esto — es la forma más simple de probar el proyecto en cualquier máquina.

### 1. Cloná el repositorio

```bash
git clone https://github.com/ALucrecia-Pagano/Calculadora_prueba.git
cd Calculadora_prueba
```

### 2. Levantá el backend

En una terminal, parado en la carpeta del proyecto:

```bash
cd backend
python -m venv .venv
source .venv/Scripts/activate      # Windows (Git Bash). En Mac/Linux: source .venv/bin/activate
pip install --break-system-packages -r requirements.txt
pytest -q                          # corre los tests — debería decir "58 passed"
uvicorn main:app --reload --port 8000
```

Dejá esa terminal abierta: ahí queda corriendo la API en `http://127.0.0.1:8000`.
La documentación interactiva de la API queda disponible en `http://127.0.0.1:8000/docs`.

### 3. Levantá el frontend

En **otra** terminal, parado en la carpeta del proyecto:

```bash
cd frontend
echo 'window.CONFIG = { API_URL: "http://127.0.0.1:8000" };' > config.js
python -m http.server 8080
```

### 4. Abrí la calculadora

Entrá a `http://127.0.0.1:8080` en tu navegador. Ya deberías poder usar las 7 operaciones, ver el historial y el gráfico.

> `config.js` no se sube al repositorio (está en `.gitignore`): en producción (por ejemplo en EasyPanel) se genera solo, al arrancar el contenedor del frontend, con la URL real de la API.

## Variables de entorno (para un despliegue real)

| Variable | Dónde | Para qué |
|---|---|---|
| `ORIGENES_PERMITIDOS` | backend | Orígenes (dominios) autorizados a llamar a la API, separados por coma. Sin esta variable, solo funciona desde `localhost`. |
| `API_URL` | frontend | URL pública de la API que va a usar el navegador de quien visite la página. |
| `DATABASE_URL` | backend | Opcional. Si no se configura, la API funciona igual pero sin historial guardado en el servidor (el endpoint `/api/historial` devuelve 503). |

## Estructura

```
backend/
  main.py          # la API
  db.py            # persistencia opcional en Postgres
  test_main.py     # tests
  Dockerfile
frontend/
  index.html       # toda la interfaz (HTML + CSS + JS, sin dependencias externas)
  Dockerfile
  docker-entrypoint.sh   # genera config.js al arrancar el contenedor
```