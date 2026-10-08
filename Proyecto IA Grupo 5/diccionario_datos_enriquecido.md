# Diccionario de datos — `stroke_enriquecido.csv`

Dataset real `stroke.csv` **limpiado y enriquecido** con 12 variables derivadas. **No contiene datos inventados ni sintéticos**: todas las variables nuevas se calculan fila a fila a partir de las originales, sin usar la variable objetivo (por tanto, sin riesgo de fuga). 5.110 filas, 24 columnas.

## Variables originales (limpiadas)

La columna `gender` tiene reasignado su único valor `Other` a `Female` (la moda). La columna `bmi` conserva sus valores faltantes como celda vacía **a propósito**: la imputación debe hacerse dentro del pipeline de validación, no aquí, para no introducir fuga de datos. El resto (`age`, `hypertension`, `heart_disease`, `ever_married`, `work_type`, `Residence_type`, `avg_glucose_level`, `smoking_status`) se mantienen tal cual, junto con `id` y el objetivo `stroke`.

## Variables derivadas nuevas

| Variable | Tipo | Cómo se calcula | Por qué puede ayudar |
|---|---|---|---|
| `bmi_missing` | binaria | 1 si el BMI original faltaba | La ausencia de BMI está asociada a mayor tasa de ictus: es señal, no ruido |
| `glucose_category` | categórica | normal (<100), prediabetes (100–125), diabetes (≥126) | Umbrales clínicos estándar; captura el efecto no lineal de la glucosa |
| `bmi_category` | categórica | OMS: bajo_peso / normal / sobrepeso / obesidad; `desconocido` si falta | Agrupa el IMC en rangos con sentido médico y maneja el faltante |
| `age_group` | categórica | menor (<18), adulto_joven (18–39), adulto (40–59), senior (≥60) | Discretiza la edad, la variable más predictiva |
| `comorbidity_count` | ordinal | hipertensión + cardiopatía (0, 1 o 2) | Resume la carga de comorbilidad en un solo número |
| `age_x_hypertension` | numérica | edad × hipertensión | Interacción: la hipertensión pesa más a edades altas |
| `age_x_heart` | numérica | edad × cardiopatía | Interacción edad–cardiopatía |
| `age_x_glucose` | numérica | edad × glucosa / 100 | Interacción edad–glucosa |
| `is_elderly` | binaria | 1 si edad ≥ 60 | Flag de grupo de alto riesgo |
| `is_child` | binaria | 1 si edad < 18 o trabajo = children | Marca a los menores (riesgo casi nulo) |
| `glucose_high` | binaria | 1 si glucosa ≥ 126 (rango diabético) | Flag binario del umbral diabético |
| `obese` | binaria | 1 si BMI ≥ 30 | Flag binario de obesidad |

## Qué consigue el enriquecimiento (medido, con honestidad)

Con validación cruzada sobre el train real, pasar de las variables originales "en bruto" a las enriquecidas sube el ROC-AUC de **0,839 a 0,851 (+0,013)**. Ahora bien, casi toda esa ganancia viene de una sola variable: `bmi_missing` (hacer explícito el dato ausente). Las categorías, grupos e interacciones apenas añaden nada por encima de eso, y el resultado se queda en **el mismo techo de ~0,85** que ya conocíamos.

Esto es coherente con todo lo demás: el enriquecimiento **hace más cómoda y explícita la señal que ya existe** (mejor organización, rangos clínicos, interacciones listas), lo cual es útil para modelar, interpretar y enseñar — pero **no crea información nueva**, así que no rompe el techo. Para eso solo sirven variables clínicas reales que hoy no están en el dataset.

*Nota: dataset de práctica; no apto para decisiones clínicas reales sin validación.*
