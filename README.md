# SIMU_TPS — Simuladores de Distribuciones

Aplicación web (TP de Simulación) que genera muestras aleatorias de distintas distribuciones de probabilidad usando los algoritmos clásicos de generación (transformada inversa, Box-Muller), y muestra en tiempo real:

- Histograma interactivo de los datos generados.
- Tabla de frecuencias (absoluta, relativa, acumulada y porcentual).
- Descarga de los datos generados en formato CSV.

## Distribuciones disponibles

| Distribución | Parámetros | Método de generación |
|---|---|---|
| **Uniforme** | `A`, `B` | Transformada inversa |
| **Normal** | Media (`μ`), Desviación estándar (`σ`) | Box-Muller |
| **Exponencial** | `λ` | Transformada inversa |

Para cada distribución se puede configurar la cantidad de datos a generar y la cantidad de intervalos (bins) del histograma.

## Stack técnico

- [Dash](https://dash.plotly.com/) (Flask + Plotly) como framework web y de visualización.
- [Dash Bootstrap Components](https://dash-bootstrap-components.opensource.faculty.ai/) para el diseño de la UI.
- [Dash AG Grid](https://dash.plotly.com/dash-ag-grid) para la tabla de frecuencias.
- NumPy / Pandas para el procesamiento de datos.
- Gunicorn como servidor WSGI en producción.

## Estructura del proyecto

```
app.py                       # Punto de entrada, registra el layout base y el Navbar
component/Navbar.py          # Barra de navegación
pages/                       # Cada archivo es una página registrada con dash.register_page
├── Home.py                  # Página de inicio
├── Uniforme.py
├── Normal.py
└── Exponencial.py
GeneradorDeDistribuciones.py # Algoritmos de generación de números aleatorios
GenerarTabla.py              # Construcción de la tabla de frecuencias (AG Grid)
requirements.txt
Procfile                     # Comando de arranque para Render/Heroku
```

## Cómo correrlo en local

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements.txt
.venv/bin/python app.py
```

La app queda disponible en `http://127.0.0.1:8067/distribuciones/`.

> Nota: la app está montada bajo el prefijo `/distribuciones/` (configurado en `app.py` vía `routes_pathname_prefix`), no en la raíz `/`.

## Despliegue

Desplegado en [Render](https://render.com) como **Web Service** (no como Static Site, ya que Dash necesita un proceso Python corriendo para resolver los callbacks del lado del servidor).

- **Build Command**: `pip install -r requirements.txt`
- **Start Command**: `gunicorn app:server --bind 0.0.0.0:$PORT`

## Proyecto desplegado

🔗 **https://simu-tps.onrender.com/distribuciones/**
