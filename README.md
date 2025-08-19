# ETL & Análisis de Clientes Telco (Churn)

Pipeline completo (Jupyter) para **extraer**, **transformar**, **cargar y analizar** datos de clientes Telco a partir de un JSON con secciones anidadas, y generar un **reporte HTML interactivo** con KPIs y visualizaciones.

## 🧭 Objetivo

* Construir un **flujo ETL reproducible** (Extracción → Transformación → Carga y Análisis → Informe final).
* Estandarizar y tipar variables claves del negocio (churn, contratos, servicios, cargos).
* Calcular **KPIs de retención/churn** y **segmentaciones** útiles para decisiones.
* Entregar un **reporte legible** para negocio en `outputs/reporte_telco.html`.

---

## 📦 Datos de entrada

**Fuente:**
`https://raw.githubusercontent.com/alura-cursos/challenge2-data-science-LATAM/refs/heads/main/TelecomX_Data.json`

**Estructura:** JSON con campos anidados:

* `customer` (datos demográficos/ID)
* `phone` (servicio telefónico)
* `internet` (servicios de internet y add-ons)
* `account` (contrato, facturación y cargos)

**Diccionario clave** (resumen):

* `customerID`: identificador único
* `Churn`: cliente dejó o no la empresa
* `gender`, `SeniorCitizen`, `Partner`, `Dependents`
* `tenure`: meses de contrato
* Servicios: `PhoneService`, `MultipleLines`, `InternetService`, `OnlineSecurity`, `OnlineBackup`, `DeviceProtection`, `TechSupport`, `StreamingTV`, `StreamingMovies`
* Contratación y pagos: `Contract`, `PaperlessBilling`, `PaymentMethod`
* Cargos: `Charges.Monthly`, `Charges.Total`

> En la fase de transformación se renombran varias columnas a español para comunicación en reportes (ver sección **Transformación**).

---

## 🧪 Requisitos

* Python 3.10+
* Jupyter Notebook / JupyterLab
* Paquetes:

  * `pandas`, `numpy`, `plotly`, `pyarrow` (para parquet)

Instalación rápida:

```bash
pip install pandas numpy plotly pyarrow
```

---

## 🧰 Estructura del repositorio (sugerida)

```
.
├─ notebooks/
│  └─ 01_telco_etl.ipynb          # Notebook con las 4 celdas del flujo
├─ outputs/
│  ├─ telco_clean.parquet         # Dataset limpio (parquet)
│  └─ reporte_telco.html          # Reporte interactivo
├─ README.md
└─ requirements.txt               # (opcional)
```

---

## 🧱 Flujo ETL (celdas del notebook)

### 1) 📌 Extracción

* Descarga JSON remoto.
* **Normaliza** secciones anidadas (`customer`, `phone`, `internet`, `account`) a un único DataFrame tabular.
* Mantiene `customerID` y ordena columnas para legibilidad.

Salida: `df_raw`

### 2) 🔧 Transformación

* **Estandariza nombres** para alinear con el diccionario.
* **Limpieza de strings** (trim), tratamiento de vacíos.
* **Tipado robusto**:

  * `Churn` → `1/0` (tipo `Int64` con soporte de `NA`)
  * `Partner`, `Dependents`, `PhoneService`, `PaperlessBilling` → boolean
  * Servicios (`OnlineSecurity`, `TechSupport`, etc.) → **mapa 1/0/-1** (`Yes/No/No internet service`)
  * `tenure`, `Charges.Total`, `Charges.Monthly` → numéricos
* **Features**:

  * `Cargos_Diarios = Charges.Monthly / 30`
  * `Bucket_Tenure` (0–6, 7–12, 13–24, 25–36, 37–60, 60+)
* **Renombrado a español** para reporting:

  * `Churn` → `Activo` (1=activo, 0=no activo)\*
  * `gender` → `Género`, `tenure` → `Meses_Contrato`, etc.
  * `Charges.Monthly` → `Cargos_Mensuales`, `Charges.Total` → `Cargo_Total`

\* Nota: si prefieres mantener la semántica tradicional (Churn=1 es que se fue), ajusta el mapeo en una línea.

Salida: `df` (limpio y tipado)

### 3) 📊 Carga y análisis

* Guarda el dataset limpio en **Parquet**: `outputs/telco_clean.parquet`.
* **KPIs**:

  * `tasa_activos` y `tasa_churn` (1 - tasa\_activos)
  * `arpu_mensual` (media de `Cargos_Mensuales`)
  * `gasto_total_promedio` (media de `Cargo_Total`)
* **Segmentaciones** (rate y n):

  * Por `Bucket_Tenure`, `Contrato`, `Servicio_Internet`, `Método_Pago`
* **Serie temporal por tenure**:

  * `%_Activos` por `Meses_Contrato`

### 4) 📄 Informe final (interactivo)

* **Visualizaciones Plotly**:

  * Barras Activo vs No Activo
  * Histogramas/barras por `Género`, `Ciudadano_65+`, `Contrato`, `Servicio_Internet`, `Streaming_TV`, `Método_Pago` (coloreado por `Activo`)
  * Línea: `% de clientes activos` por `Meses_Contrato`
  * Tablas de segmentación (tasa y n)
* **Reporte HTML** autogenerado en `outputs/reporte_telco.html`:

  * Encabezado con fecha
  * KPIs formateados
  * Gráficas interactivas embebidas (Plotly CDN)
  * Notas de reproducibilidad

---

## ▶️ Cómo ejecutar

1. Abre `notebooks/01_telco_etl.ipynb`.
2. Ejecuta las celdas **en orden**:

   * **Extracción**
   * **Transformación**
   * **Carga y análisis**
   * **Informe final**
3. Revisa:

   * `outputs/telco_clean.parquet`
   * `outputs/reporte_telco.html` (abre en el navegador).

---

## ✅ Supuestos y decisiones

* Vacíos en `Charges.Total` → `0` (posibles clientes nuevos o sin facturación en el período).
* Campos tipo servicio con “No internet service” / “No phone service” → `-1` para distinguir “no aplica” de “No”.
* `Activo` se usa como **proxy inverso de churn** (1=activo, 0=no activo). Ajustable según negocio.
* Se evita eliminar filas salvo que falten **ambos** `Churn` y `Charges.Total`.

---

## 📈 KPIs entregados

* **Tasa de activos** y **tasa de churn**
* **ARPU mensual**
* **Gasto total promedio**
* Segmentaciones de tasa por:

  * **Tenure bucket**
  * **Tipo de contrato**
  * **Servicio de internet**
  * **Método de pago**

---

## 🧩 Extensiones sugeridas

* **Modelado de churn** (Logística/Árboles) + matriz de confusión y SHAP.
* Cohortes por **mes de alta**.
* **CLV** (valor de vida) con supuestos de margen y prob. de supervivencia.
* Dashboard con **Dash/Panel/Streamlit** para exploración.

---

## 🛠️ Troubleshooting

* `ValueError: cannot convert ...` → revisar columnas numéricas con `df.select_dtypes(object)`; hay conversión forzada en el pipeline, pero si cambió la fuente puede requerir un `replace` adicional.
* `FileNotFoundError` al guardar parquet → asegúrate de que exista la carpeta `outputs/` (el código ya la crea, pero en ambientes restringidos puede fallar permisos).
* Reporte pesado → usa `include_plotlyjs='cdn'` (ya aplicado) o reduce el número de figuras.

---

## 📜 Licencia

Uso educativo/demostrativo. Ajusta a la licencia de tu organización si lo integras en producción.
