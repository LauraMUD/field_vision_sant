# ⚽ Field‑Vision‑Sant (PoC)

Este repositorio agrupa dos **notebooks de exploración** que sirven como **prueba de concepto** para
hacer tracking cinemático en vídeos de fútbol. No es un producto acabado, sino una base para
experimentar con detección, tracking y conversión píxel‑metro.

---

## 📁 Estructura del proyecto

```
field_vision_sant/
├── requirements.txt
├── notebooks/
│   ├── train_ball_tracking.ipynb
│   ├── train_player_kinematics.ipynb
│   ├── yolov8n.pt
│   ├── ball_tracking_data.csv   ← generado al ejecutar
│   ├── tracking_data.csv        ← generado al ejecutar
│   └── player_summary.csv       ← generado al ejecutar
└── videos/
    ├── input/                   ← coloca aquí los MP4 a analizar
    └── output/                  ← los vídeos anotados se guardan aquí
```

---

## 📓 Notebooks

### `train_ball_tracking.ipynb`
Flujo de trabajo para detectar y seguir el balón. Usa YOLOv8 (`yolov8n.pt`) sobre la clase
`sports ball`, aplica un **filtro de Kalman** para interpolar frames donde el balón no se detecta,
y filtra falsos positivos por tamaño de bounding box. Exporta la trayectoria a
`ball_tracking_data.csv` (posición, velocidad, aceleración y ángulo por frame) y genera un clip
anotado en `videos/output`.

### `train_player_kinematics.ipynb`
Detección y tracking de jugadores. Integra YOLOv8 (clase `person`) con **ByteTrack** para
mantener IDs consistentes entre frames. Ofrece conversión píxel → metro mediante **homografía**
(calibración interactiva por clic con `%matplotlib widget`). Exporta:

- `tracking_data.csv` — posición y velocidad por jugador por frame.
- `player_summary.csv` — resumen estadístico por jugador (distancia total, velocidad máxima/media, tiempo en campo).

Incluye visualizaciones: mapa de calor de posiciones, histograma de velocidades y ranking de jugadores.

---

## 🛠 Preparación en Windows

### 1. Crear el entorno virtual

```powershell
cd C:\ruta\a\tu\proyecto\field_vision_sant
python -m venv .venv
```

### 2. Activar el entorno

```powershell
.venv\Scripts\activate
```

### 3. Instalar dependencias

```powershell
pip install --upgrade pip
pip install -r requirements.txt
```

> El `requirements.txt` incluye: `ultralytics`, `supervision`, `opencv-python-headless`,
> `pandas`, `matplotlib`, `seaborn`, `filterpy` e `ipympl`.

---

## ⚠️ Notas importantes

- **Calibración interactiva (homografía):** la celda de selección de puntos del campo requiere
  un backend interactivo de matplotlib. Antes de ejecutarla instala `ipympl` y reinicia el kernel:
  ```powershell
  pip install ipympl
  ```
  Luego añade `%matplotlib widget` al inicio de la celda (o `%matplotlib tk` en Jupyter clásico).

- **Detección del balón:** `yolov8n.pt` es un modelo COCO de propósito general; la clase
  `sports ball` (ID 32) puede tener baja confianza en cámaras de broadcast. Si `ball_tracking_data.csv`
  sale vacío, prueba a bajar `CONF_THRESHOLD` a `0.15` o usa un modelo especializado en fútbol.

- **Conversión píxel → metro:** las distancias son aproximadas si no se usa la homografía.
  Ajusta `PIXELS_PER_METER` en la sección de configuración de cada notebook según tu cámara.
