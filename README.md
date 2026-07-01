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

## Cómo funciona / comportamiento según los parámetros

Las tres páginas (`Uniforme`, `Normal`, `Exponencial`) comparten el mismo patrón: un layout con inputs + un `callback` que se dispara cada vez que cambia alguno de ellos.

- **Slider de bins (5 a 25)**: define en cuántos intervalos se agrupa el histograma. No cambia los datos generados, solo cómo se agrupan visualmente — con pocos bins se ven barras más anchas (menos detalle), con más bins, barras más finas.
- **Cantidad de datos (`n`)**: cuántos números aleatorios generar. A mayor `n`, el histograma se aproxima más a la forma teórica de la distribución (ley de los grandes números). Es útil comparar `n=100` vs `n=100.000` para ver cómo el histograma se "prolija".
- **Parámetros propios de cada distribución**:
  - *Uniforme*: `A` y `B` (se valida que `A < B`; si no, se muestra un mensaje de error y no se calcula nada).
  - *Normal*: `μ` (media) y `σ` (se valida que `σ > 0`).
  - *Exponencial*: `λ` (se valida que `λ > 0`).

Ante cualquier cambio de input, el callback:

1. Valida los parámetros (si son inválidos, muestra el error y no recalcula).
2. Genera un nuevo vector de `n` valores llamando a la función correspondiente de `GeneradorDeDistribuciones.py` (con `numpy.random` como fallback si el algoritmo manual fallara).
3. Calcula los bordes de los bins con `numpy.linspace`, cerrando el último borde (`np.nextafter`) para que el valor máximo generado no se pierda por redondeo de floats.
4. Regenera el histograma (Plotly) y la tabla de frecuencias (AG Grid) usando esos mismos bordes.
5. Guarda el dataset generado en un `dcc.Store` (memoria del navegador) para que el botón **Descargar CSV** exporte exactamente lo que se ve en pantalla, sin volver a generar números aleatorios nuevos (que serían distintos a los mostrados).

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
