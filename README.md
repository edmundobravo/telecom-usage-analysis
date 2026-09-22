# telecom-usage-analysis
# Análisis de uso de servicios móviles — ConnectaTel (2024)

Análisis exploratorio del comportamiento de consumo de 4,000 clientes de una empresa de telecomunicaciones con operaciones en México y Colombia, orientado a identificar segmentos accionables y detectar desalineaciones entre los planes contratados y el uso real.

**Proyecto del Sprint 7 — Bootcamp de Análisis de Datos, TripleTen.**

---

## 🎯 Objetivo

> ¿Cómo usan realmente los clientes los servicios de llamadas y mensajes, y qué segmentos presentan oportunidades comerciales desaprovechadas?

El análisis responde cuatro preguntas de negocio: qué segmentos muestran mayor o menor consumo, qué usuarios presentan comportamientos atípicos, cómo varía el uso según edad y plan, y qué patrones permiten rediseñar la oferta de planes.

---

## 📊 Datos

| Fuente | Registros | Descripción |
|---|---|---|
| `plans.csv` | 2 × 8 | Catálogo de planes: precio, minutos y mensajes incluidos, costo por excedente |
| `users_latam.csv` | 4,000 × 8 | Clientes: edad, ciudad, fecha de registro, plan contratado, fecha de baja |
| `usage.csv` | 40,000 × 6 | Bitácora de eventos: llamadas (duración) y mensajes (longitud) |

**Cobertura temporal:** clientes registrados entre 2022 y 2024; actividad registrada de enero a junio de 2024.

### Problemas de calidad encontrados y resueltos

| Problema | Alcance | Tratamiento |
|---|---|---|
| Sentinel `-999` en `age` | 55 filas (1.4%) | Imputado con la mediana, calculada excluyendo el sentinel |
| Doble faltante en `city` (`NaN` y `"?"`) | 565 filas (14.1%) | Unificados como nulos |
| `churn_date` en notación científica | 466 valores | Identificada como timestamps Unix corrompidos |
| Fechas de registro en 2026 | 40 filas (1.0%) | Marcadas como `NaT` |
| Eventos sin fecha | 50 filas (0.13%) | Documentados |
| Campos cruzados entre tipos de evento | 28 filas (0.07%) | Anulado el campo que no aplica |
| Nulos estructurales en `duration`/`length` | ~20,000 por columna | **Preservados**: verificados como MAR, no son datos perdidos |

El último caso merece atención: cada fila de `usage` registra un solo tipo de evento, por lo que una llamada nunca tiene longitud en caracteres ni un mensaje tiene duración. Esos nulos se verificaron mediante su dependencia respecto a `type` (0% de faltantes en un grupo contra 99.9% en el otro) y se conservaron. Imputarlos con cero habría sesgado a la baja todas las métricas de consumo.

---

## 📂 Contenido del repositorio

```
├── notebooks/
│   └── ConnectaTel_analysis.ipynb
├── data/
│   ├── plans.csv
│   ├── users_latam.csv
│   └── usage.csv
└── README.md
```

> **Nota sobre los datasets:** los tres archivos de entrada se encuentran en la carpeta data/ incluidos en este repositorio.

---

## ▶️ Cómo abrir el notebook en Google Colab

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](URL_DEL_NOTEBOOK_EN_GITHUB)

O manualmente: abre el archivo `.ipynb` desde GitHub y haz clic en **Open in Colab**.

---

## 🔁 Cómo reproducir el análisis

1. Abre `notebooks/connectatel_analysis.ipynb`.
2. Ejecuta las celdas **en orden**, de arriba hacia abajo. Cada paso depende del anterior.
3. Los tres datasets están incluidos en `data/`. Si ejecutas el notebook fuera 
   del entorno de TripleTen, ajusta las rutas de `pd.read_csv()` en la celda de 
   carga del Paso 1 a `../data/`.

**Dependencias:** `pandas`, `numpy`, `seaborn`, `matplotlib`

```bash
pip install pandas numpy seaborn matplotlib
```

---

## 🛠️ Etapas del análisis

1. **Carga y exploración** — estructura, tipos de datos y revisión preliminar de las tres fuentes.
2. **Identificación de problemas de calidad** — conteo y proporción de nulos, detección de sentinels, validación de rangos de fecha.
3. **Limpieza** — imputación de sentinels, conversión de tipos, verificación del patrón de missingness (MCAR / MAR / MNAR) antes de decidir el tratamiento.
4. **Agregación por usuario** — construcción de un perfil de consumo por cliente (`cant_mensajes`, `cant_llamadas`, `cant_minutos_llamada`) e integración con los atributos demográficos mediante `left join`.
5. **Distribuciones y outliers** — histogramas segmentados por plan, boxplots y detección formal mediante el método IQR.
6. **Segmentación** — clasificación por nivel de uso y por grupo de edad, con visualización de proporciones.
7. **Insight ejecutivo** — traducción de los hallazgos a recomendaciones comerciales.

---

## 🔍 Hallazgos principales

**1. El plan contratado no guarda relación con el consumo real.**

| Métrica (mediana) | Básico | Premium |
|---|---|---|
| Edad | 48.0 | 48.0 |
| Mensajes | 5.0 | 5.0 |
| Llamadas | 4.0 | 4.0 |
| Minutos de llamada | 19.5 | 20.3 |

Los clientes Premium pagan 25 USD por 600 minutos y 500 mensajes, mientras consumen lo mismo que los clientes Básico, que pagan 12 USD. La proporción Básico/Premium se mantiene en ~65/35 dentro de los tres niveles de uso, confirmando que la segmentación comercial no refleja el comportamiento observado.

**2. Toda la cartera subutiliza masivamente su plan.**

El consumo mediano representa menos del 20% de los minutos y el 5% de los mensajes incluidos en el plan más económico. La oferta está dimensionada muy por encima de la demanda real.

**3. La edad no predice el consumo.**

La distribución de edad es prácticamente uniforme entre los 18 y 79 años, y la proporción de clientes de alto uso se mantiene entre 6.1% y 7.6% en los tres grupos etarios.

**4. Dos segmentos mal asignados, accionables de inmediato.**

- **185 clientes de alto uso en plan Básico** — los que más consumen pagando la tarifa más baja. Audiencia natural para migración a Premium.
- **266 clientes de bajo uso en plan Premium** — pagan la tarifa alta por capacidad que no utilizan, escenario clásico de cancelación por percepción de sobrepago.

**5. Los outliers son clientes valiosos, no errores.**

El método IQR identifica 96 clientes (2.4%) con consumo de minutos por encima del límite, con un máximo de 155.69 contra una mediana de 19.7. Se conservaron: son usuarios reales de consumo intensivo y constituyen el segmento de mayor potencial de ingreso.

---

## 💡 Recomendaciones

- **Campaña de migración dirigida** a los 185 clientes de alto uso en Básico, con audiencia definida por comportamiento real y no por supuestos demográficos.
- **Retención preventiva** sobre los 266 clientes de bajo uso en Premium, ofreciendo un plan intermedio antes de que decidan cancelar.
- **Rediseñar la escalera de planes**: la brecha entre Básico y Premium es demasiado amplia frente al consumo observado. Se recomienda evaluar un plan de entrada y un plan intermedio.
- **Abandonar la segmentación por edad** y reorientar los recursos de marketing por comportamiento de uso.
- **Corregir el pipeline de captura en origen**: los problemas detectados son sistemáticos, no aleatorios.

---

## ⚠️ Limitaciones

- La ventana de actividad cubre seis meses (enero–junio 2024), mientras los clientes se registraron desde 2022. Todo análisis de consumo se refiere a ese semestre, no a la vida completa del cliente.
- Los datasets no incluyen consumo de datos móviles, aunque el catálogo de planes especifica GB incluidos. El análisis cubre únicamente llamadas y mensajes.
- La tasa de churn resulta mayor en el segmento de alto uso (14.0%) que en los de uso medio y bajo (11.5%). El resultado es contraintuitivo y la ventana temporal disponible es insuficiente para sustentarlo; requiere verificación con series más amplias.
- Los valores 120.0 minutos y 1490.0 caracteres se repiten exactamente 30 veces cada uno en los datos originales, lo que sugiere topes artificiales del sistema de captura y no observaciones reales.

---

## 🧰 Stack

`Python` · `pandas` · `NumPy` · `Matplotlib` · `seaborn` · `Jupyter`

---

## ✍️ Autor

**Edmundo Bravo Sánchez**
Bootcamp de Análisis de Datos — TripleTen · Sprint 7
