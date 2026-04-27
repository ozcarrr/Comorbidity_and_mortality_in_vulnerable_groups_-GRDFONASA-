
# Informe "ANÁLISIS DE DATOS E INFERENCIA ESTADÍSTICA"

PubMed


**Integrantes:**

- Nicolas Róman
- Miguel Figueroa
- Oscar Pichuman


**Pregunta de investigación:** ¿Qué factores clínicos predicen la mortalidad durante la hospitalización?

Con los datos, somos capaces de llegar a una posible respuesta mediante el analisis de diferentes variables que tenemos disponibles a mayor escala para poder encontrar algun descubrimiento relevante respecto a la mortalidad durante la hospitalización. 

Para esto se usaran los datos de GRD fonasa **(post pandemia)** para poder enfocar nuestra investigación en otros factores más que la propias enfermedades. 

#### Contexto y reconocimiento de variables

De acuerdo a los datasets y nuestra pregunta de investigación, se logran percibir columnas relevantes a dicha pregunta:

**'Diagnostico n' para la comorbilidad:** Se tienen hasta 35 diagnosticos en el dataset, por lo que el "Diagnostico1" (causa  inicial o básica) no suele ser el motivo de mortalidad, sino suelen ser la acumulación de enfermedades.

**Severidad** : La columna 'IR_29301_SEVERIDAD' complementa bastante la columna de los diagnosticos, permitiendo ser un puntaje de predicción fuerte.


**Factores demográficos**: Las columnas 'Fecha_Nacimiento', 'Genero' e 'Ingreso' son capaces de demostrar alguna correlación respecto a la mortalidad en ciertos grupos de edad, grupos vulnerables, comorbolidades que afecten a cierto grupo de genero, etc.


**Enfermedades de temporada** :  Revisando rangos de fechas especificos (muy probable en las estaciones del año), podemos ver si la mortalidad aumenta en cierto rango o estación. 


**Trayectoria del Paciente:** El `TIPO_INGRESO` (Urgencia o Programado) es crítico. Un ingreso por urgencia normalmente tiene una probabilidad de mortalidad mucho más alta.

**Complejidad del Manejo:** La columna `USOSPABELLON` y la cantidad de `PROCEDIMIENTO` realizados indican qué tan invasivo fue el tratamiento.

**Sobrecarga en establecimientos de salud**: Verificar si ciertos establecimientos especificos enfrentan una demanda más grande de la que pueden manejar. 

**Previsión**: Permite saber si ciertos planes de salud ofrecidos por FONASA tienden a tener indices más altos de mortalidad. 

**Peso**: Permite saber si el peso juega un rol importante durante la hospitalización para que resulte en algo mortal para el paciente.

# falta esto

#### Metodología:



### Análisis:


#### Observaciones:


#### Conclusiones:




### Fuentes: 

-  "The neutrophil-to-lymphocyte ratio levels over time correlate to all-cause hospital mortality in sepsis" (Comorbolidad en casos de mortalidad durante hospitalización)" (https://pubmed.ncbi.nlm.nih.gov/39253154/) 
- "Association Between Body Mass Index and Morbidity and Mortality During Hospitalization After Trauma" (Peso alto en casos de mortalidad durante hospitalización) (https://pubmed.ncbi.nlm.nih.gov/35275109/)

