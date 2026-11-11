# Triple Insight — Trabajo Final Integrador | Ciencia de Datos | Fundación YPF

**Comisión 2 · Grupo 9**  
**Integrantes:** Azul · Carla · Trinidad

## Resumen del proyecto (Parte I - Aprendizaje Supervisado)

Analizamos información oficial sobre la comercialización de combustibles en estaciones de servicio de Argentina. El trabajo integra dos enfoques de aprendizaje automático:

- **Aprendizaje supervisado:** estimar el volumen comercializado a partir de características temporales, geográficas y comerciales.
- **Aprendizaje no supervisado:** explorar perfiles de estaciones de servicio mediante técnicas de clustering.

El proyecto comprende la carga y unificación de siete archivos anuales, la limpieza de los datos, el análisis exploratorio, la comparación de modelos predictivos y la segmentación exploratoria de estaciones.

## Fuente y alcance de los datos

**Fuente:** [Datos Argentina – Precios de estaciones de servicio (Resolución 1104/04)](https://www.datos.gob.ar/dataset/precios-eess-resolucion-1104-04).

Se utilizaron siete archivos CSV, cargados individualmente y unificados mediante código en las notebooks:

| Archivo | Registros | Período informado |
|---|---:|---|
| `EESS_2018.csv` | 2.250 | Enero 2018 |
| `EESS_2019.csv` | 3.500 | Enero 2019 |
| `EESS_2020.csv` | 7.750 | Enero 2020 |
| `EESS_2021.csv` | 2.250 | Enero 2021 |
| `EESS_2022.csv` | 35.000 | Enero y febrero 2022 |
| `EESS_2023.csv` | 7.500 | Enero 2023 |
| `EESS_2024.csv` | 4.250 | Enero 2024 |
| **Total** | **62.500** | **Ocho períodos observados** |

La base unificada contiene 62.500 registros y 20 columnas: 19 columnas de origen y un identificador incremental generado durante la integración. Luego de aplicar los criterios de limpieza, quedan **61.771 registros**, correspondientes a **5.523 estaciones**, **9 productos**, **4.033 operadores** y **24 provincias**.

La información incluye identificación y ubicación de las estaciones, operador, bandera, tipo de negocio, producto, canal de comercialización, período, volumen y precios. Los componentes impositivos que aparecían en una extracción posterior de 2024–2025 no forman parte de esta base, porque esa extracción tenía una estructura diferente y fue descartada.

### Alcance temporal

La base no constituye una serie mensual continua. Contiene enero de cada año entre 2018 y 2024 y febrero de 2022. Los períodos sin información no se consideran datos faltantes susceptibles de interpolación o reconstrucción. Las comparaciones temporales se limitan a los períodos efectivamente observados.

## Preparación y calidad de los datos

La carga y unificación de los siete archivos se realiza dentro de cada notebook. Se eliminan los identificadores originales de los archivos cuando corresponde y se genera un identificador incremental para la base integrada.

Entre las principales decisiones de limpieza:

- No se eliminaron las combinaciones de negocio repetidas, ya que pueden corresponder a registros válidos de una misma estación, período y producto en distintos canales de comercialización.
- Se identificaron **685 registros sin movimientos**, con volumen cero y `exentos` nulo. Se excluyeron del dataset limpio.
- Se excluyeron **44 registros** en los que el precio sin impuestos superaba al precio con impuestos.
- Los valores nulos de `fecha_de_baja` se conservaron: son compatibles con estaciones que no registran fecha de baja.
- Los valores cero de `precio_surtidor` se consideran no disponibles para los análisis que utilizan esa variable como precio.

El resultado de esta etapa es una base limpia de **61.771 registros y 21 columnas** (incluye la variable de control generada durante la limpieza).

## Análisis exploratorio

El análisis exploratorio permitió describir la distribución de los volúmenes, los productos comercializados, la distribución geográfica de los registros y el comportamiento de los precios.

El volumen presenta una distribución fuertemente asimétrica: la mediana es aproximadamente 55,8 y el percentil 75 es 135,6, mientras que el percentil 95 supera los 72.900 y el máximo supera 1,1 millones. Por ese motivo, los valores extremos se conservaron en el análisis descriptivo y se evaluaron alternativas de transformación para el modelado.

Los volúmenes acumulados por provincia o bandera describen los registros disponibles; no representan demanda anual comparable, porque la cobertura y la cantidad de registros varían entre períodos.

Los precios presentan un fuerte efecto temporal: el precio de surtidor promedio pasa de aproximadamente 20 en 2018 a 715 en 2024. Por lo tanto, las comparaciones entre años deben interpretarse considerando el contexto inflacionario. La relación entre precio y volumen también depende del producto y del período analizado.

## Aprendizaje supervisado: predicción de volumen

El objetivo del aprendizaje supervisado es estimar `volumen` a partir de características temporales, geográficas y comerciales. Se desarrollaron dos pruebas para evaluar el efecto de la distribución asimétrica del volumen.

### Prueba I: volumen en escala original

Se compararon tres algoritmos:

- Regresión Lineal.
- Árbol de Decisión.
- Random Forest.

En esta primera prueba se utilizó el volumen en su escala original. Los modelos no lograron una generalización satisfactoria: los valores extremos influyen fuertemente en las métricas y el baseline basado en la mediana supera a los modelos. Esta experiencia motivó una segunda estrategia de modelado.

### Prueba II: transformación logarítmica

Para reducir el efecto de la asimetría se aplicó `log1p` a la variable objetivo durante el entrenamiento. Las predicciones se transformaron nuevamente con `expm1` para calcular las métricas finales en la escala original.

La población se restringió a registros de venta minorista de los cuatro combustibles líquidos seleccionados:

- Gas Oil Grado 2.
- Gas Oil Grado 3.
- Nafta súper.
- Nafta premium.

Se utilizaron variables temporales y comerciales, incluyendo año, producto, localidad agrupada, bandera agrupada e indicador de pandemia. La separación de datos es cronológica: el entrenamiento comprende los períodos hasta febrero de 2022 (**47.730 registros**) y la prueba corresponde a enero de 2023 y enero de 2024 (**10.981 registros**).

La validación cruzada se realizó sobre el conjunto de entrenamiento mediante `GroupKFold` de cinco pliegues, agrupando por `nro_inscripcion`. De esta manera, los registros de una misma estación no quedan distribuidos entre entrenamiento y validación dentro de un mismo pliegue.

La Prueba II no incorpora variables de historia individual de ventas por estación. El modelo aprende patrones a partir de las características incluidas, pero no modela la evolución histórica particular de cada establecimiento.

### Resultados de la Prueba II

Métricas finales sobre el conjunto de prueba, luego de revertir la transformación logarítmica:

| Modelo | MAE | RMSE | R² |
|---|---:|---:|---:|
| Baseline (mediana) | 7.493,24 | 29.535,04 | -0,07 |
| Regresión Lineal | 5.356,38 | 23.117,82 | 0,35 |
| Árbol de Decisión | 3.890,91 | 18.433,81 | 0,58 |
| **Random Forest base** | **3.760,65** | **18.072,34** | **0,60** |

En esta prueba, Random Forest base obtuvo el menor MAE y RMSE y el mayor R² entre los modelos comparados. La transformación logarítmica permitió mejorar el desempeño frente a la Prueba I y superar al baseline.

### Optimización de Random Forest con GridSearchCV

Se incorporó una búsqueda acotada de hiperparámetros mediante `GridSearchCV`, utilizando `GroupKFold` de cinco pliegues y agrupación por estación. Se evaluaron ocho combinaciones:

- `n_estimators`: 30 y 50.
- `max_depth`: 8 y 12.
- `min_samples_leaf`: 5 y 10.

La combinación seleccionada fue `n_estimators=50`, `max_depth=12` y `min_samples_leaf=10`. El MAE medio de validación cruzada fue **0,8254 en escala logarítmica**. Este valor corresponde a la validación cruzada sobre el entrenamiento y no debe confundirse con las métricas del conjunto de prueba.

| Modelo | MAE | RMSE | R² |
|---|---:|---:|---:|
| Random Forest base | 3.760,65 | 18.072,34 | 0,60 |
| Random Forest optimizado | 3.804,04 | 18.223,30 | 0,59 |

En el conjunto de prueba, el Random Forest optimizado no mejoró los resultados del modelo base: presentó un MAE y un RMSE levemente mayores y un R² de 0,59 frente a 0,60. Se reportan los resultados obtenidos sin afirmar que el ajuste de hiperparámetros haya mejorado el desempeño final.

El conjunto de prueba contiene dos períodos puntuales —enero de 2023 y enero de 2024—. Por eso, los resultados son indicativos y no garantizan el mismo desempeño para otros meses o escenarios futuros.

## Resumen del proyecto (Parte II - Aprendizaje No Supervisado)

Analizamos información oficial sobre la comercialización de combustibles en estaciones de servicio de Argentina. Este README documenta **cuatro pruebas** (notebooks) que recorren la evolución del trabajo, desde una primera exploración hasta una segmentación de estaciones con caracterización comercial:

- **Prueba 1:** carga y unificación de los datos, limpieza, análisis exploratorio y primera segmentación con K-Means a nivel de registro.
- **Prueba 2:** estimación del volumen futuro con un enfoque amplio: SVD, K-Means, DBSCAN, embeddings con base de datos vectorial (ChromaDB).
- **Prueba 3:** segmentación exploratoria a nivel de **estación de servicio**, con K=4 y comparación contra K=5.
- **Prueba 4 (ampliada):** la Prueba 3 más una caracterización comercial de los clusters (banderas, operadores, tipos de negocio, provincias, canales y productos).

> **Nota sobre la nomenclatura.** Las "Pruebas 1 a 4" de este documento corresponden a las notebooks entregadas. No deben confundirse con las "Prueba I" y "Prueba II" de modelado supervisado (volumen en escala original y con `log1p`, Random Forest y GridSearchCV) del README anterior, que pertenecen a la Pre-entrega 3 y **no forman parte de este documento**.

## Las cuatro pruebas de un vistazo

| Prueba | Notebook | Unidad de análisis | Técnicas | Resultado principal |
|---|---|---|---|---|
| 1 | `Prueba 1_ TripleInsight_Pre-entrega4_NOSUPERVISADO.ipynb` | Registro de venta (55.836) | K-Means (K=6 y K=4), PCA, inercia y Silhouette | K=4 con Silhouette 0,41. Incluye `nro_inscripcion` como variable: es la versión inicial, superada por las Pruebas 3 y 4 |
| 2 | `Prueba 2_DS_EESS_volumen2.ipynb` | Establecimiento × producto × canal × período (panel de 61.710 filas) | SVD, K-Means, DBSCAN, embeddings, ChromaDB| K-Means con K=5 (Silhouette 0,29); DBSCAN útil para detectar atípicos; el MLP no mejora a la persistencia |
| 3 | `Prueba 3_ TripleInsight_Pre-entrega4_NOSUPERVISADO_VF.ipynb` | Estación de servicio (4.988) | K-Means (K=4 y K=5), PCA, gráficos de silueta | Silhouette de 0,36 (K=4) y 0,43 (K=5) |
| 4 | `prueba 4_TripleInsight_Pre_entrega4_NOSUPERVISADO_AMPLIADA.ipynb` | Estación de servicio (4.988) | Prueba 3 + caracterización comercial por cluster | K=4 como solución principal, K=5 como alternativa y perfiles comerciales de cada grupo |

Las Pruebas 1, 3 y 4 cargan y unifican de forma autónoma siete archivos CSV anuales. La Prueba 2 parte del archivo `EESS_limpio.csv`, que genera la Prueba 1.

| Archivo | Registros | Período informado |
|---|---:|---|
| `EESS_2018.csv` | 2.250 | Enero 2018 |
| `EESS_2019.csv` | 3.500 | Enero 2019 |
| `EESS_2020.csv` | 7.750 | Enero 2020 |
| `EESS_2021.csv` | 2.250 | Enero 2021 |
| `EESS_2022.csv` | 35.000 | Enero y febrero 2022 |
| `EESS_2023.csv` | 7.500 | Enero 2023 |
| `EESS_2024.csv` | 4.250 | Enero 2024 |
| **Total** | **62.500** | **Ocho períodos observados** |

La base unificada tiene 62.500 registros y 20 columnas: 19 de origen y un identificador incremental generado durante la integración. Tras la limpieza quedan **61.771 registros**, correspondientes a **5.523 estaciones**, **9 productos**, **4.033 operadores** y **24 provincias**.

### Alcance temporal

La base **no es una serie mensual continua**: contiene enero de cada año entre 2018 y 2024 y febrero de 2022. La cobertura por período es muy despareja, por lo que los volúmenes acumulados de distintos períodos no son comparables entre sí.

| Período | Registros | Establecimientos |
|---|---:|---:|
| Enero 2018 | 2.230 | 592 |
| Enero 2019 | 3.452 | 865 |
| Enero 2020 | 7.658 | 1.907 |
| Enero 2021 | 2.226 | 554 |
| Enero 2022 | 21.625 | 5.194 |
| Febrero 2022 | 12.960 | 3.148 |
| Enero 2023 | 7.421 | 1.831 |
| Enero 2024 | 4.199 | 1.045 |

## Preparación y calidad de los datos

La limpieza es la misma en las cuatro pruebas y se documenta en la Prueba 1:

- Se normalizan los nombres de columnas (minúsculas, sin acentos ni caracteres especiales) y se eliminan los identificadores originales de cada archivo.
- `exentos` (`t`/`f`) se convierte a verdadero/falso. `no_novimientos` (`SI`/`NO`, con la errata del nombre original) se convierte en la variable de control `sin_movimientos`.
- Se identificaron **685 registros sin movimientos**: coinciden exactamente con los registros de `exentos` nulo y se excluyen.
- Se excluyeron **44 registros** en los que el precio sin impuestos superaba al precio con impuestos.
- No hay duplicados exactos. **10.798 filas** pertenecen a combinaciones repetidas de estación, período y producto; **no se eliminan**, porque pueden ser registros válidos de distintos canales de comercialización.
- `fecha_de_baja` tiene 92,15 % de nulos (56.920 registros); se conserva, ya que es compatible con estaciones sin fecha de baja.
- `precio_surtidor` vale 0 en **9.332 registros (15,1 %)**. Se considera "no disponible" y se reemplaza por nulo en los análisis que lo usan como precio.
- No hay volúmenes negativos, años fuera de rango ni meses inválidos.

El resultado es una base limpia de **61.771 registros y 21 columnas** (22 en el archivo `EESS_limpio.csv` que exporta la Prueba 1, que además incluye `periodo`).

### Productos y población de análisis

| Producto | Registros |
|---|---:|
| Gas Oil Grado 2 | 16.539 |
| Gas Oil Grado 3 | 14.038 |
| Nafta (súper) entre 92 y 95 Ron | 13.194 |
| Nafta (premium) de más de 95 Ron | 12.065 |
| GNC | 5.270 |
| Kerosene | 592 |
| Nafta (común) hasta 92 Ron | 39 |
| Gas Oil Grado 2B | 24 |
| GLPA | 10 |

Las Pruebas 1, 3 y 4 trabajan con los **cuatro combustibles líquidos más frecuentes** (Gas Oil Grado 2 y 3, Nafta súper y Nafta premium): **55.836 registros** de **4.988 estaciones**, equivalentes al 90,4 % de la base limpia. La Prueba 2 utiliza todos los productos.

## Hallazgos del análisis exploratorio

- **Volumen muy asimétrico.** Mediana 55,84; percentil 75 de 135,62; percentil 90 de 460,90; percentil 95 de 72.930,65 y máximo de 1.110.980. Los valores extremos se conservaron.
- **El GNC explica gran parte de esa asimetría.** Sus 5.270 registros (8,5 % de la base) tienen una mediana de 80.296,5, frente a decenas en los combustibles líquidos. La **unidad de `volumen` no está documentada** en los archivos; por eso no se asumió ninguna y se trabajó con transformaciones logarítmicas.
- **Fuerte efecto temporal en los precios.** El precio de surtidor promedio pasa de aproximadamente 20 en 2018 a 715 en 2024 (32,7 en 2019; 48,1 en 2020; 62,5 en 2021; 84,7 en 2022; 172,9 en 2023). Son precios nominales, afectados por la inflación.
- **Los ceros de `precio_surtidor` son estructurales.** Corresponden al 100 % de los registros de los canales Agro, Otros sectores, Reventa a otras estaciones, Transporte de cargas y Transporte público de pasajeros, y a ninguno del canal "Al público".
- **Relación débil entre precio y volumen** dentro de cada producto (correlaciones entre -0,46 y 0,06 sobre `log1p(volumen)` y `log(precio)`, con valores cercanos a cero en los productos más frecuentes).
- Los volúmenes acumulados por provincia o bandera describen los registros disponibles; **no** representan demanda anual comparable.

## Prueba 1: acondicionamiento, EDA y primera segmentación

Es la primera versión del flujo. Carga y unifica los siete CSV, limpia, explora y aplica K-Means.

**Población.** Los cuatro combustibles líquidos, con volumen positivo: 55.836 registros.

**Variables de K-Means.** `anio`, `nro_inscripcion`, `precio_surtidor` y `volumen`, estandarizadas con `StandardScaler`, sin transformación logarítmica. La unidad de análisis es el **registro de venta**.

**Evaluación de K (matriz completa de 55.836 registros):**

| K | Inercia | Silhouette |
|---:|---:|---:|
| 2 | 163.848,07 | 0,63 |
| 3 | 122.072,02 | 0,41 |
| 4 | 85.396,27 | 0,41 |
| 5 | 56.276,22 | 0,49 |
| 6 | 45.961,28 | 0,49 |

Se ajustaron K=6 y K=4. El equipo avanzó con **K=4**, priorizando clusters menos desbalanceados que los de K=5 y una segmentación más interpretable.

| Cluster (K=4) | Registros | Rasgo distintivo (promedios estandarizados) |
|---:|---:|---|
| 0 | 25.703 | `nro_inscripcion` alto (+0,89) |
| 1 | 26.917 | `nro_inscripcion` bajo (-0,85) |
| 2 | 3.172 | Precio de surtidor muy alto (+3,84) y años recientes (+1,70) |
| 3 | 44 | Volumen extremadamente alto (+28,91) |

Con K=6 los tamaños son 20.913, 11.557, 3.163, 20.111, 8 y 84 registros.

**Lectura crítica.** Esta versión es útil como antecedente, pero tiene limitaciones que las Pruebas 3 y 4 corrigen: usa `nro_inscripcion` (un identificador) como magnitud en las distancias, por lo que dos de los cuatro clusters se separan en buena medida por ese número; segmenta registros y no estaciones; y no corrige la asimetría de volumen y precio. Además, el gráfico de cargas del PCA muestra que `anio` y `precio_surtidor` aportan juntos al primer componente, lo que refleja la inflación nominal.

## Prueba 2: estimación del volumen futuro y análisis multimetodológico

**Objetivo.** Estimar el volumen que se venderá en un período posterior para una combinación establecimiento–producto–canal, y complementar el análisis con reducción de dimensionalidad, segmentación, representaciones vectoriales y redes neuronales.

**Datos.** Parte de `EESS_limpio.csv` (61.771 registros, 22 columnas). Construye un panel con una fila por establecimiento × producto × canal × período: se agregan las 61 filas con clave repetida (se suma el volumen y se promedian los precios), lo que da **61.710 filas y 23.468 claves únicas**. La variable objetivo es `y = log1p(volumen)`. Todo el cuaderno usa `SEMILLA = 42`.

**Decisiones de diseño relevantes:**

- `precio_surtidor == 0` pasa a nulo y no se usa como predictor; se usa `precio_con_impuestos`.
- No se eliminan atípicos: sin documentación de unidades no puede afirmarse que sean errores. Se marcan con IQR dentro de cada producto (escala log). Los diez volúmenes mayores corresponden a GNC y a gasoil en los canales Agro, Transporte de cargas y Otros sectores.
- Los modelos predictivos usan volumen y precio **rezagados** y excluyen las variables del mismo período, para evitar fuga de información.
- `mes` no se usa: casi todos los períodos son enero y no puede estimarse estacionalidad.

### Reducción de dimensionalidad (SVD)

Sobre una matriz de 31 variables (numéricas estandarizadas y dummies centradas de producto, canal y tipo de negocio), **7 componentes retienen 91,2 % de la varianza** (error relativo de reconstrucción de 0,088). El primer componente está dominado por `y` y `log_precio` junto con el indicador de GNC; el cuarto, por el tipo de negocio.

### Clustering: K-Means

Se evaluó K de 2 a 10 sobre el espacio reducido (Silhouette calculado sobre una muestra de 10.000 puntos):

| K | Inercia | Silhouette | Davies-Bouldin | Calinski-Harabasz |
|---:|---:|---:|---:|---:|
| 2 | 197.828,88 | 0,656 | 0,513 | 19.263,8 |
| 3 | 142.533,55 | 0,477 | 0,612 | 25.337,8 |
| 4 | 112.077,25 | 0,349 | 0,950 | 27.071,2 |
| 5 | 93.614,58 | 0,286 | 1,147 | 27.349,7 |
| 6 | 83.134,63 | 0,276 | 1,201 | 26.193,1 |
| 7 | 74.626,74 | 0,313 | 1,115 | 25.488,1 |
| 8 | 66.717,15 | 0,348 | 0,978 | 25.481,6 |
| 9 | 60.670,61 | 0,351 | 0,975 | 25.286,8 |
| 10 | 56.765,05 | 0,371 | 0,994 | 24.495,0 |

K=2 tiene el Silhouette más alto, pero solo separa GNC del resto. La regla de la notebook exige K ≥ 3 y toma el codo: **K=5**.

| Cluster | Filas (%) | Volumen mediano | Precio mediano (con impuestos) | Rasgo principal |
|---:|---:|---:|---:|---|
| 0 | 18.566 (30,1 %) | 50,26 | 99,32 | Combustibles líquidos |
| 1 | 31.127 (50,4 %) | 48,09 | 99,50 | Combustibles líquidos |
| 2 | 1.612 (2,6 %) | 48,04 | 96,00 | 100 % de registros `exentos` |
| 3 | 5.205 (8,4 %) | 81.105,00 | 49,25 | GNC (95,9 %) |
| 4 | 5.200 (8,4 %) | 44,01 | 822,00 | Precios muy superiores al resto |

Dos de los grupos (GNC y precios altos) provienen de escalas de volumen y de precio por producto o por registro, no necesariamente de comportamientos comerciales distintos. No está documentado a qué corresponden los precios altos (otra unidad, otro concepto de reporte u otro período).

### Clustering: DBSCAN

Con `min_samples` base de 14 y `eps` sugerido por el codo de la curva de k-distancias de 0,23, se evaluó una grilla de nueve configuraciones sobre una muestra de 15.000 puntos. Entre las que tenían al menos 2 clusters y hasta 15 % de ruido, se eligió la de mayor Silhouette: `eps = 0,345` y `min_samples = 7`.

| Algoritmo | Clusters | Observaciones agrupadas | Ruido | Silhouette | Davies-Bouldin | Calinski-Harabasz |
|---|---:|---:|---:|---:|---:|---:|
| K-Means | 5 | 61.710 | 0 | 0,286 | 1,147 | 27.349,7 |
| DBSCAN | 110 | 60.548 | 1.162 (1,9 %) | 0,237 | 1,015 | 3.941,6 |

El índice de Rand ajustado entre ambas particiones (sin ruido) es 0,216. DBSCAN genera **110 clusters casi puros en un producto** (el 97 % tiene más de 95 % de un único producto): una fragmentación demasiado fina para segmentar el mercado. Su aporte está en el **ruido**, que reúne los volúmenes extremos (GNC de más de 800.000 y gasoil de hasta aproximadamente 680.000) y funciona como candidato de auditoría de posibles errores de carga o de unidad.

**Conclusión de la prueba:** K-Means para segmentar el mercado; DBSCAN para auditar atípicos.

### Embeddings y base de datos vectorial (ChromaDB)

- Se construye un documento de texto por combinación única de operador, bandera, tipo de negocio, localidad, provincia, producto y canal: **24.011 documentos**.
- Se generan embeddings de 384 dimensiones con `paraphrase-multilingual-MiniLM-L12-v2` (`sentence-transformers`), con TF-IDF + SVD como alternativa si el modelo no puede descargarse. En la ejecución de referencia, el cálculo tardó alrededor de 26 minutos.
- La similitud coseno recupera establecimientos comparables (por ejemplo, otras bocas YPF de GNC en Buenos Aires con distancias de 0,03). K-Means sobre los embeddings arroja Silhouette coseno de 0,12 a 0,17 (K de 4 a 12; el mejor, K=8 con 0,166), es decir, una estructura débil.
- Los 24.011 documentos se indexan en ChromaDB (índice HNSW, métrica coseno) con metadatos de provincia, producto, bandera, canal, `n_obs` e `y_medio`. Se prueban consultas por texto, con filtros de metadatos y por documento de referencia; la coincidencia entre HNSW y búsqueda exacta en el top-5 fue de 5/5.
- Los embeddings capturan similitud de nombres y texto, **no** comportamiento de ventas: las variables numéricas se guardan como metadatos.

## Prueba 3: segmentación de estaciones con K-Means (K=4 y K=5)

La notebook es autónoma: carga y unifica los siete CSV, aplica los criterios de limpieza y construye una tabla con una fila por estación.

### Unidad de análisis y variables

La unidad de análisis es la **estación**, identificada por `nro_inscripcion`, que se usa **solo como clave de agrupación** y no entra en las distancias de K-Means. La población son los cuatro combustibles líquidos con volumen positivo: **55.836 registros de 4.988 estaciones**.

Variables resumen por estación:

- `anio_ultimo_observado`: año más reciente disponible.
- `volumen_mediano`: mediana del volumen de sus registros.
- `precio_surtidor_mediano`: mediana del precio de surtidor disponible (los ceros se tratan como nulos).
- `cantidad_registros` y `cantidad_productos`: se conservan para describir los grupos, pero no entran en el clustering.

**422 estaciones (8,5 %) no tienen precio de surtidor válido** y se imputan con la mediana observada entre estaciones (104,21). Se aplica `log1p` a volumen y precio, y las tres variables se estandarizan. La matriz final contiene **4.988 estaciones y 3 variables**.

### Evaluación de la cantidad de clusters

| K | Inercia | Silhouette |
|---:|---:|---:|
| 2 | 11.307,26 | 0,66 |
| 3 | 8.465,79 | 0,35 |
| 4 | 6.294,30 | 0,36 |
| 5 | 4.360,26 | 0,43 |
| 6 | 3.718,53 | 0,37 |

K=2 tiene el Silhouette más alto, pero es una segmentación demasiado general. K=5 es el mejor valor entre K=3 y K=6.

### Resultados de K=4

Silhouette de **0,3598**; PCA con 2 componentes que explican **83,42 %** de la varianza.

| Cluster | Estaciones | Año último promedio | Volumen mediano promedio | Precio mediano promedio | Registros promedio | Productos promedio |
|---:|---:|---:|---:|---:|---:|---:|
| 0 | 3.139 | 2022,55 | 346,66 | 103,50 | 11,30 | 3,75 |
| 1 | 172 | 2019,17 | 45,79 | 40,09 | 4,94 | 2,87 |
| 2 | 1.619 | 2022,50 | 15,20 | 107,74 | 11,82 | 3,48 |
| 3 | 58 | 2024,00 | 1.924,71 | 711,54 | 6,83 | 3,67 |

### Comparación con K=5

Silhouette de **0,43**.

| Cluster K=5 | Estaciones | Año último promedio | Volumen mediano promedio | Precio mediano promedio | Registros promedio | Productos promedio |
|---:|---:|---:|---:|---:|---:|---:|
| 0 | 2.211 | 2022,05 | 449,87 | 103,05 | 6,19 | 3,75 |
| 1 | 1.281 | 2023,67 | 78,89 | 105,84 | 22,86 | 3,70 |
| 2 | 171 | 2019,18 | 45,95 | 40,09 | 4,95 | 2,87 |
| 3 | 57 | 2024,00 | 1.957,79 | 719,51 | 6,84 | 3,67 |
| 4 | 1.268 | 2022,22 | 13,49 | 107,46 | 9,16 | 3,47 |

K=5 divide el grupo mayoritario de K=4 en tres niveles de volumen y deja casi intactos los grupos pequeños. Los gráficos de silueta muestran que K=4 tiene un solapamiento mayor en el cluster de bajo volumen (valores de silueta negativos) y que K=5 los reduce. La notebook concluye recomendando K=5 por cohesión; el equipo mantiene K=4 como solución principal por interpretabilidad (ver Prueba 4).

La notebook exporta `estaciones_clusterizadas_k4.csv`.

## Prueba 4 (ampliada): caracterización comercial de los clusters

Reproduce la Prueba 3 (mismas variables, mismas soluciones y mismos resultados: 4.988 estaciones, Silhouette 0,3598 para K=4 y 0,43 para K=5) y agrega una **caracterización comercial** de los grupos de K=4, sin volver a entrenar el modelo.

**Método.** Se vincula cada estación con sus registros originales. Para cada estación se toma la **categoría predominante** (moda) de operador, bandera, tipo de negocio, provincia, localidad y canal, y se marca si registró cada uno de los cuatro combustibles en algún período. Estas variables categóricas **no entran a K-Means**: solo describen los grupos. Los porcentajes usan como denominador la cantidad de estaciones del cluster y los de productos indican presencia en los registros, no participación en el volumen.

### Composición comercial por cluster (K=4)

| Cluster | Estaciones (% del total) | Bandera predominante (% del cluster) | Tipo de negocio predominante | Provincias principales | Canal "Al público" |
|---:|---:|---|---|---|---:|
| 0 | 3.139 (62,93 %) | YPF (48,93 %), Shell (20,17 %), Axion (7,74 %) | Combustibles líquidos únicamente (60,66 %); duales líquidos + GNC (29,88 %) | Buenos Aires (35,49 %), Córdoba (10,83 %), Santa Fe (9,40 %) | 91,27 % |
| 1 | 172 (3,45 %) | Axion (34,30 %), Blanca (21,51 %), YPF (18,02 %) | Estación de servicio (84,88 %) | Buenos Aires (43,02 %), Capital Federal (12,79 %) | 97,09 % |
| 2 | 1.619 (32,46 %) | Blanca (38,23 %), YPF (10,62 %), Puma (8,46 %) | Combustibles líquidos únicamente (59,73 %); duales (29,34 %) | Buenos Aires (35,15 %), Santa Fe (17,42 %), Córdoba (15,94 %) | 85,92 % (Agro: 12,11 %) |
| 3 | 58 (1,16 %) | Blanca (29,31 %), YPF (24,14 %), Shell (22,41 %) | Combustibles líquidos únicamente | Buenos Aires (39,66 %), Santa Fe (31,03 %) | 91,38 % |

**Porcentaje de estaciones que registraron cada producto:**

| Cluster | Gas Oil Grado 2 | Gas Oil Grado 3 | Nafta súper | Nafta premium |
|---:|---:|---:|---:|---:|
| 0 | 98,06 % | 96,08 % | 91,53 % | 89,55 % |
| 1 | 82,56 % | 75,00 % | 69,19 % | 59,88 % |
| 2 | 97,96 % | 85,30 % | 89,13 % | 75,97 % |
| 3 | 94,83 % | 93,10 % | 94,83 % | 84,48 % |

### Lectura de los perfiles

- **Cluster 0 (mayoritario).** Reproduce la composición general de la red: predominio de YPF y Shell, mayoría de bocas de líquidos únicamente y buena cobertura de los cuatro combustibles. Su operador más frecuente es Automóvil Club Argentino.
- **Cluster 1.** Concentra estaciones cuyo último año observado es temprano (2019 en promedio), con precios nominales bajos. Se destaca por la presencia de Axion (el operador Axion Energy Argentina agrupa a 56 de sus 172 estaciones, 32,56 %), por el tipo de negocio "Estación de servicio" y por la menor cobertura de productos (Nafta premium en 59,88 %).
- **Cluster 2.** Tiene la mayor proporción de bandera Blanca, la mayor presencia del canal Agro (12,11 %) y operadores como cooperativas agropecuarias (Agricultores Federados Argentinos y Asociación de Cooperativas Argentinas), con una participación mayor de Santa Fe y Córdoba que los demás grupos.
- **Cluster 3.** Grupo muy pequeño, de estaciones observadas en 2024, con volúmenes medianos y precios nominales altos. Incluye operadores como ANJOR S.A. y Balsuar 3 S.R.L.

### Cautelas de interpretación

- **El precio alto del cluster 3 es principalmente un efecto de calendario.** Su precio mediano promedio (711,54) coincide con el precio de surtidor promedio de 2024 (aproximadamente 715) y su último año observado es 2024 para todas las estaciones. Las diferencias de precio entre clusters reflejan sobre todo **inflación nominal**, no un posicionamiento "premium".
- **El último año observado no demuestra que una estación esté activa o inactiva.** El cluster 1 agrupa estaciones cuyo último registro es temprano, pero la base solo cubre enero (y un febrero) de cada año.
- No deben asignarse etiquetas comerciales (premium, inactiva, mayorista, minorista) solo por el número de cluster: requieren evidencia adicional.
- Las categorías predominantes describen toda la ventana observada, no la situación comercial actual.

La notebook exporta `estaciones_clusterizadas_k4.csv`, `estaciones_clusterizadas_k4_perfil_comercial.csv`, `fichas_perfil_comercial_k4.csv` y `cobertura_productos_k4.csv`.

## Comparación entre pruebas

| | Prueba 1 | Prueba 2 | Prueba 3 | Prueba 4 |
|---|---|---|---|---|
| Unidad | Registro | Registro agregado (panel) | Estación | Estación |
| Variables de clustering | `anio`, `nro_inscripcion`, `precio_surtidor`, `volumen` | `log1p(volumen)`, `log(precio)`, `exentos` y dummies, reducidas con SVD (7 componentes) | Año último, volumen mediano y precio mediano (log + estandarización) | Igual que la Prueba 3 |
| Tratamiento de `nro_inscripcion` | Entra como variable | No se usa como variable | Solo clave de agrupación | Solo clave de agrupación |
| Transformación log | No | Sí | Sí | Sí |
| Algoritmos | K-Means | K-Means, DBSCAN | K-Means | K-Means |
| K elegido | 4 (Silhouette 0,41) | 5 (Silhouette 0,29) | 4 (0,36); K=5: 0,43 | 4 (0,36); K=5: 0,43 |
| Aporte principal | Primer flujo completo | Atípicos, similitud, redes neuronales | Segmentación válida a nivel de estación | Perfiles comerciales de los grupos |

Los valores de Silhouette **no son comparables entre pruebas**, porque se calculan sobre matrices, unidades de análisis y poblaciones distintas.

## Conclusiones y limitaciones

El conjunto de pruebas permite reconstruir cómo maduró el enfoque: de segmentar registros incluyendo un identificador (Prueba 1) a segmentar **estaciones** con variables resumen transformadas (Pruebas 3 y 4), y de ahí a describirlas comercialmente. La Prueba 2 aporta una mirada complementaria: la estructura del mercado está dominada por el producto (GNC), los atípicos son relevantes y un MLP no supera a la persistencia con tan pocos períodos.

En la segmentación de estaciones, K=4 se conserva como solución principal por interpretabilidad, mientras que K=5 tiene mejor Silhouette (0,43 frente a 0,36) y queda como alternativa de mayor granularidad. K=5 separa, además, un grupo de 1.281 estaciones con último año observado cercano a 2024 y más registros por estación, lo que refleja en parte la cobertura temporal de los datos.

Las principales limitaciones son:

- La cobertura temporal es discontinua (enero de cada año y febrero de 2022), sin estacionalidad estimable ni serie mensual completa.
- Los archivos tienen cantidades de registros muy distintas, por lo que los volúmenes acumulados por período no son comparables.
- La unidad de `volumen` no está documentada y su escala difiere mucho entre productos (GNC frente a combustibles líquidos).
- Los precios son nominales: `anio` y `precio_surtidor` están fuertemente asociados por la inflación, de modo que dos de las tres variables del clustering de estaciones comparten información temporal.
- El 8,5 % de las estaciones carece de precio de surtidor válido y se imputa con la mediana.
- Las segmentaciones son exploratorias, dependen de las variables resumen y del K elegido, y no son clasificaciones comerciales definitivas.
- En la Prueba 2, el período de prueba es único (enero de 2024), y las filas sin historial previo solo pueden predecirse con atributos estáticos.

## Conclusiones generales

En función de los resultados obtenidos en las cuatro pruebas, se concluye que K-Means constituye el método más adecuado para la segmentación de las estaciones de servicio dentro del conjunto de datos analizado. En particular, la configuración de cinco clústeres presentó el mejor desempeño estadístico entre las alternativas evaluadas, alcanzando un coeficiente de silueta de 0,43. Este resultado evidencia una estructura de agrupamiento más cohesiva y con mayor diferenciación entre los segmentos identificados que la solución de cuatro clústeres.

No obstante, la configuración de cuatro grupos también demostró ser una alternativa válida, especialmente desde una perspectiva aplicada, debido a su mayor simplicidad e interpretabilidad comercial. Por su parte, DBSCAN resultó un método complementario para la identificación de observaciones atípicas y estructuras particulares dentro del conjunto de datos, aunque no presentó las mismas ventajas para construir una segmentación general del mercado.

Es importante destacar que las cuatro pruebas no persiguieron exactamente el mismo objetivo analítico. Mientras que las pruebas 1, 3 y 4 estuvieron orientadas principalmente a identificar y caracterizar perfiles de estaciones de servicio según sus características operativas y comerciales, en la prueba 2 la aplicación de técnicas de clustering tuvo una finalidad diferente: optimizar la predicción del volumen comercializado, utilizando los agrupamientos como una herramienta para mejorar el desempeño de los modelos predictivos y no como un mecanismo para establecer perfiles comerciales. En este último caso, la evaluación de los resultados debe considerar la capacidad predictiva de los modelos y su comparación con una referencia de persistencia, y no exclusivamente las métricas de calidad del agrupamiento.

Asimismo, el análisis permitió demostrar que la calidad de los resultados depende tanto de la selección del algoritmo como de las decisiones adoptadas durante la preparación de los datos, la transformación de variables y la reducción de dimensionalidad. Las limitaciones relacionadas con la disponibilidad de información temporal, la cobertura desigual de los establecimientos y la influencia de factores externos, como la inflación sobre los precios nominales, también deben considerarse al interpretar los resultados y plantear posibles aplicaciones predictivas.

Por lo tanto, K-Means con K=5 representa la mejor alternativa estadística para la caracterización y segmentación comercial de las estaciones de servicio, mientras que K-Means con K=4 ofrece una solución más sencilla para su interpretación y utilización en contextos de gestión comercial. DBSCAN, en cambio, resulta de utilidad como herramienta complementaria para detectar observaciones atípicas. Finalmente, para el objetivo predictivo abordado en la prueba 2, la selección metodológica debe realizarse en función de la mejora efectiva en la predicción del volumen comercializado.

En conclusión, no existe un único método óptimo para todos los objetivos planteados: la elección del algoritmo debe responder al problema que se busca resolver, priorizando la calidad estadística de la segmentación cuando el objetivo es identificar perfiles, o la capacidad predictiva cuando se pretende estimar el comportamiento futuro de las variables comerciales.

## Estructura del repositorio

- `EESS_2018.csv` a `EESS_2024.csv`: archivos fuente anuales utilizados para la unificación.
- `Prueba 1_ TripleInsight_Pre-entrega4_NOSUPERVISADO.ipynb`: carga y unificación, limpieza, análisis exploratorio y primera segmentación con K-Means. Genera `EESS_limpio.csv`.
- `Prueba 2_DS_EESS_volumen2.ipynb`: estimación de volumen futuro; SVD, K-Means, DBSCAN, embeddings y ChromaDB, perceptrón y MLP. Lee `EESS_limpio.csv`.
- `Prueba 3_ TripleInsight_Pre-entrega4_NOSUPERVISADO_VF.ipynb`: segmentación de estaciones con K-Means (K=4 y K=5) y gráficos de silueta.
- `prueba 4_TripleInsight_Pre_entrega4_NOSUPERVISADO_AMPLIADA.ipynb`: Prueba 3 más caracterización comercial de los clusters y exportación de resultados.
- `requirements.txt`: dependencias de Python.

## Reproducibilidad y ejecución

Las notebooks están preparadas para ejecutarse en **Google Colab** y utilizan rutas bajo `/content/`.

1. Abrir la notebook correspondiente en Google Colab.
2. Subir a `/content/` los siete archivos `EESS_2018.csv` a `EESS_2024.csv` (Pruebas 1, 3 y 4) o `EESS_limpio.csv` (Prueba 2).
3. Instalar las dependencias:

   ```bash
   !pip install -r /content/requirements.txt
   ```

4. Reiniciar el entorno de ejecución y ejecutar todas las celdas en orden.

**Orden recomendado.** Ejecutar primero la **Prueba 1**, que genera `EESS_limpio.csv` en `/content/`. Las **Pruebas 3 y 4** cargan y unifican los siete CSV por su cuenta y no dependen de la Prueba 1 ni de ese archivo; la **Prueba 2** sí depende de él.

**Prueba 2.** La primera celda instala `chromadb` y `sentence-transformers`. Hay dos interruptores en la celda de configuración: `USAR_PRECIO_ACTUAL` (por defecto `False`, para evitar fuga de información) y `USAR_SENTENCE_TRANSFORMERS` (requiere internet para descargar el modelo; si falla, se usa TF-IDF + SVD). El cálculo de embeddings puede demorar varios minutos.

**Pruebas 3 y 4.** Exportan sus resultados a `/content/` (por ejemplo, `estaciones_clusterizadas_k4.csv`).

No incluir en la carpeta de entrada datasets de otros años ni archivos de salida que puedan incorporarse accidentalmente al proceso de unificación. Para comprobar la reproducibilidad, se recomienda reiniciar el entorno y ejecutar cada notebook de principio a fin.

## Dependencias

Las librerías utilizadas por las notebooks se encuentran listadas en [`requirements.txt`](requirements.txt). Además de las habituales (`pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`), la Prueba 2 requiere `chromadb` y `sentence-transformers` y, para la sección de redes neuronales, `tensorflow`/`keras`; conviene verificar que `requirements.txt` las incluya.
