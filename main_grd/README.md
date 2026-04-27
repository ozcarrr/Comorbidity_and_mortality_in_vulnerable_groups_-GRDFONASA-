# Mortalidad Intrahospitalaria y sus Predictores — Análisis GRD Chile

Proyecto del curso **Análisis de Datos e Inferencia Estadística**  
Universidad del Desarrollo · Facultad de Ingeniería · 2026

**Integrantes:** Oscar Pichuman · Nicolás Román · Miguel Figueroa  
**Docente:** Karen Orostica

---

## Pregunta de investigación

> **¿Qué factores clínicos predicen la mortalidad durante la hospitalización en la red pública de salud chilena (2022–2025)?**

### Hipótesis

La mortalidad intrahospitalaria en la red pública chilena está determinada por una interacción multivariada donde la severidad clínica y la carga de comorbilidades actúan como predictores primarios, pero cuyo riesgo se ve significativamente exacerbado por factores de vulnerabilidad demográfica y la naturaleza del flujo hospitalario.

---

## Descripción del proyecto

Este proyecto analiza los datos de **Grupos Relacionados por el Diagnóstico (GRD)** del FONASA para los años 2022, 2023 y 2024, con el objetivo de identificar qué variables clínicas y demográficas predicen la mortalidad durante la hospitalización en la red pública chilena.

El análisis se estructura en cuatro etapas progresivas:

1. **Estadística descriptiva y EDA** — caracterización del dataset y distribución de la variable objetivo (`TIPOALTA = '2'`)
2. **Construcción de índices** — Índice de Comorbilidad de Charlson (CCI) e Índice de Mortalidad Hospitalaria por Vulnerabilidad (IMHV, índice propio)
3. **Pruebas de hipótesis** — análisis en cadena desde el panorama general hasta casos específicos
4. **Modelado predictivo** — regresión logística y Random Forest comparando CCI vs IMHV

### Comorbilidades focales

| Variable | ICD-10 | Rol en el análisis |
|---|---|---|
| Insuficiencia Cardíaca (IC) | I50.x | Comorbilidad crónica focal — predictor primario |
| Sepsis | A40.x, A41.x | Segunda comorbilidad focal — mecanismo agudo |
| IC + Sepsis | I50.x + A40/A41 | Efecto sinérgico — grupo de máximo riesgo |
| Abandono | TIPOALTA: Alta voluntaria / Fuga | Proxy de riesgo no resuelto |

---

## Estructura del repositorio

```
Datos-GRD/
│
├── main/
│   ├── Analisis.ipynb          # Notebook principal — limpieza, EDA y CCI
│   └── Analisis_v2.ipynb       # Versión con pruebas de hipótesis y análisis en cadena
│
├── test/
│   ├── Pruebas.ipynb                    # Exploración inicial de los tres datasets
│   ├── prueba_dias.ipynb                # Análisis de fechas y días de estadía
│   └── muestreo_estadistico_(3).ipynb   # Técnicas de muestreo sobre datos GRD 2023
│
├── data/                        # Carpeta de datos (archivos excluidos por .gitignore)
│   └── .gitkeep
│
├── Investigaciones.md           # Recopilación de papers y bibliografía relevante
├── Informe ANÁLISIS DE DATOS E INFERENCIA ESTADÍSTICA.md  # Borrador del informe
└── README.md
```

---

## Datos

Los datos **no están incluidos en el repositorio** por restricciones de tamaño y privacidad. Deben descargarse directamente desde la fuente oficial y ubicarse en la carpeta `data/`.

### Fuente

**FONASA — Ministerio de Salud de Chile**  
Bases de datos GRD Público Externo, disponibles en:  
[https://deis.minsal.cl/#datosabiertos](https://deis.minsal.cl/#datosabiertos)

### Archivos requeridos

| Archivo | Encoding | Separador | Año |
|---|---|---|---|
| `GRD_PUBLICO_EXTERNO_2022.txt` | UTF-16 | `\|` | 2022 |
| `GRD_PUBLICO_2023.txt` | UTF-16 | `\|` | 2023 |
| `GRD_PUBLICO_2024.txt` | Latin-1 | `\|` | 2024 |

> **Nota:** el archivo de 2024 usa una codificación distinta a los años anteriores y renombra la columna `ID_BENEFICIARIO` como `CIP_ENCRIPTADO`. El notebook principal maneja estas diferencias automáticamente.

### Variables seleccionadas

De las más de 100 columnas disponibles, se trabaja con ~34 variables relevantes:

| Categoría | Variables |
|---|---|
| Identificadores | `COD_HOSPITAL`, `CIP_ENCRIPTADO` |
| Demografía | `SEXO`, `FECHA_NACIMIENTO`, `PREVISION`, `COMUNA` |
| Gestión y flujo | `TIPO_INGRESO`, `ESPECIALIDAD_MEDICA`, `FECHA_INGRESO`, `FECHAALTA`, `USOSPABELLON` |
| Severidad clínica | `IR_29301_SEVERIDAD`, `IR_29301_MORTALIDAD` |
| Variable objetivo | `TIPOALTA` |
| Diagnósticos | `DIAGNOSTICO1` – `DIAGNOSTICO9` (ICD-10) |

### Variables derivadas

| Variable | Tipo | Descripción |
|---|---|---|
| `EDAD` | Numérica | Diferencia en años entre `FECHA_NACIMIENTO` y `FECHA_INGRESO` |
| `DIAS_ESTADIA` | Numérica | Diferencia en días entre `FECHAALTA` y `FECHA_INGRESO` |
| `FALLECIDO` | Binaria | `True` si `TIPOALTA == '2'` |
| `CCI` | Numérica | Índice de Comorbilidad de Charlson (mapeo ICD-10 → Quan et al. 2011) |
| `IC` | Binaria | `1` si algún diagnóstico empieza en `I50` |
| `SEPSIS` | Binaria | `1` si algún diagnóstico empieza en `A40` o `A41` |
| `IMHV` | Continua [0–1] | Índice propio: edad ≥70, FONASA A/B, urgencia, estadía >10 días, pabellón ≥2 |
| `ABANDONO` | Binaria | `1` si `TIPOALTA` es Alta voluntaria o Fuga del paciente |

---

## Metodología

### Índice de Comorbilidad de Charlson (CCI)

Implementación propia basada en el mapeo de prefijos ICD-10 a las 17 categorías del índice, siguiendo los pesos actualizados de **Quan et al. (2011)**. Cada paciente recibe un score entre 0 y 37 según sus diagnósticos registrados en `DIAGNOSTICO1`–`DIAGNOSTICO9`.

| Score | Interpretación |
|---|---|
| 0 | Sin comorbilidades relevantes |
| 1–2 | Comorbilidad leve |
| 3–4 | Comorbilidad moderada |
| ≥ 5 | Comorbilidad severa |

### Índice IMHV (propio)

Índice aditivo que captura vulnerabilidad clínica y socioeconómica no contemplada por el CCI:

```
IMHV = 0.25·(Edad ≥ 70) + 0.20·(FONASA A/B) + 0.20·(Ingreso urgencia)
      + 0.20·(Estadía > 10 días) + 0.15·(Pabellón ≥ 2 usos)
```

### Análisis en cadena de hipótesis

El análisis de pruebas de hipótesis sigue una estructura en cuatro niveles donde cada hallazgo motiva la siguiente pregunta:

```
Nivel 1: Panorama general
  N1-A: Mortalidad urgencia vs programado       → Z dos proporciones
  N1-B: Mortalidad FONASA A/B vs C/D            → Z dos proporciones
       |
       ↓ "La via de ingreso y la prevision predicen mortalidad.
          ¿Se explica por la carga de comorbilidades?"
       |
Nivel 2: Comorbilidades como mediadores
  N2-A: Mortalidad Sepsis+ vs Sepsis-           → Z dos proporciones
  N2-B: CCI en 4 grupos (IC/Sepsis/ambas/ninguna) → Kruskal-Wallis + Bonferroni
       |
       ↓ "IC+Sepsis tiene el mayor riesgo combinado.
          ¿Los que abandonan tienen un perfil similar?"
       |
Nivel 3: Abandono como riesgo no resuelto
  N3-A: CCI en abandonos vs altas normales      → Mann-Whitney U
  N3-B: Abandono en IC+ vs IC-, Sepsis+ vs Sepsis- → Chi-cuadrado + V de Cramér
       |
       ↓ "Los abandonos concentran un perfil especifico.
          ¿El IMHV los captura mejor que el CCI?"
       |
Nivel 4: Validación empírica del IMHV
  N4-A: Poder predictivo individual de cada componente → Z dos proporciones ×5
```

### Pruebas de hipótesis aplicadas

| ID | Prueba | Variables | Pregunta |
|---|---|---|---|
| H1 | Z dos proporciones | IC, FALLECIDO | Mortalidad IC+ > IC-? |
| H2 | Mann-Whitney U | CCI, FALLECIDO | CCI mayor en fallecidos? |
| H3 | Mann-Whitney U | IMHV, FALLECIDO | IMHV mayor en fallecidos? |
| H4 | Spearman | CCI, IMHV | Los índices se correlacionan? |
| H5 | Kruskal-Wallis + Bonferroni | DIAS_ESTADIA, IR_29301_SEVERIDAD | Estadía difiere por severidad? |
| H6 | Z una proporción | FALLECIDO | Mortalidad != 5% de referencia? |

---

## Requisitos

```bash
pip install pandas numpy scipy statsmodels scikit-learn matplotlib seaborn
```

| Librería | Versión mínima | Uso |
|---|---|---|
| pandas | 2.0 | Carga y manipulación de datos |
| numpy | 1.24 | Operaciones numéricas |
| scipy | 1.10 | Pruebas estadísticas |
| statsmodels | 0.14 | Pruebas Z de proporciones |
| scikit-learn | 1.3 | Modelos predictivos (avance final) |
| matplotlib | 3.7 | Visualizaciones base |
| seaborn | 0.12 | Visualizaciones estadísticas |

**Python recomendado:** 3.11+

---

## Cómo ejecutar

1. Clonar el repositorio:
```bash
git clone https://github.com/<usuario>/Datos-GRD.git
cd Datos-GRD
```

2. Descargar los tres archivos GRD desde [deis.minsal.cl](https://deis.minsal.cl/#datosabiertos) y ubicarlos en `data/`:
```
data/
  GRD_PUBLICO_EXTERNO_2022.txt
  GRD_PUBLICO_2023.txt
  GRD_PUBLICO_2024.txt
```

3. Instalar dependencias:
```bash
pip install pandas numpy scipy statsmodels scikit-learn matplotlib seaborn
```

4. Abrir el notebook principal:
```bash
jupyter notebook main/Analisis_v2.ipynb
```

Ejecutar las secciones en orden: carga de datos → limpieza → EDA → CCI → variables derivadas → pruebas de hipótesis.

---

## Referencias

- Charlson, M. E. et al. (1987). A new method of classifying prognostic comorbidity in longitudinal studies. *Journal of Chronic Diseases*, 40(5), 373–383.
- Quan, H. et al. (2011). Updating and validating the Charlson comorbidity index using data from 6 countries. *American Journal of Epidemiology*, 173(6), 676–682.
- Elixhauser, A. et al. (1998). Comorbidity measures for use with administrative data. *Medical Care*, 36(1), 8–27.
- Fonarow, G. C. et al. (2008). Factors identified as precipitating hospital admissions for heart failure. *Archives of Internal Medicine*, 168(8), 847–854.
- Ministerio de Salud de Chile. (2023). Bases GRD Público Externo 2022. DEIS. [https://deis.minsal.cl](https://deis.minsal.cl)
- Análisis de defunciones intrahospitalarias en hospital comunitario de Ñuble, Chile. *Rev. Chil. Aten. Primaria Salud Fam.* [https://rchapsf.uchile.cl/index.php/RCHAPSF/article/view/75767](https://rchapsf.uchile.cl/index.php/RCHAPSF/article/view/75767)
- Asociación entre ingreso hospitalario en fines de semana y mortalidad intrahospitalaria. Sistema público chileno. *Dialnet.* [https://dialnet.unirioja.es/servlet/articulo?codigo=10575797](https://dialnet.unirioja.es/servlet/articulo?codigo=10575797)

---

## Estado del proyecto

| Etapa | Estado |
|---|---|
| Carga y limpieza de datos | Completo |
| EDA y estadística descriptiva | Completo |
| Implementación del CCI | Completo |
| Variables derivadas (IC, IMHV, Sepsis, Abandono) | Completo |
| Verificación de normalidad | Completo |
| Pruebas de hipótesis H1–H6 | Completo |
| Análisis en cadena Niveles 1–4 | Completo |
| Modelado predictivo (Reg. Logística + Random Forest) | Pendiente |
| Comparación AUC CCI vs IMHV | Pendiente |
| Validación con datos DEIS externos (Sepsis) | Pendiente |
| Informe final | En redacción |
