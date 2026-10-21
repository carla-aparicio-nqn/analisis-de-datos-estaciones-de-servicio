# Triple Insight — Trabajo Final Integrador | Ciencia de Datos | Fundación YPF

**Comisión 2 · Grupo 9**

**Integrantes: Azul · Carla · Trinidad**

*Versión actualizada: 21/10/2026 (reemplaza al README del 29/09/2026). Notebook de referencia: TripleInsight_Pre-entrega3.ipynb.*

## Resumen del proyecto

Partimos de los registros oficiales de comercialización de combustibles de estaciones de servicio de Argentina para responder una pregunta concreta: ¿qué factores explican cuánto combustible vende una estación y es posible predecirlo? Trabajamos con siete extracciones anuales (enero de 2018 a enero de 2024, más febrero de 2022), las unificamos en una única base de 62.500 registros, la depuramos, la exploramos y entrenamos tres algoritmos supervisados (Regresión Lineal, Árbol de Decisión y Random Forest). La conclusión central: el volumen vendido es extremadamente asimétrico, por lo que predecirlo en su escala original no funciona; al transformarlo con logaritmo y simplificar las variables categóricas, los modelos logran explicar cerca del 78 % de la variabilidad (R² ≈ 0,78 en validación cruzada, sobre la escala log).

## Qué cambió respecto de la versión anterior

Decidimos modificar la forma en que se construye la base de datos. Estos son los cambios y su impacto en el análisis:

| **Aspecto**                       | **Versión anterior (29/09)**                                                                              | **Versión actual (21/10)**                                                                                                |
|-----------------------------------|-----------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Fuente de datos                   | Un único dataset consolidado de 101.500 registros y 27 columnas, que incluía diciembre 2024 y enero 2025. | Siete archivos CSV anuales (EESS_2018 a EESS_2024) cargados por separado en Colab y unificados con código en la notebook. |
| Dataset 2025                      | Incluido (17.399 registros de enero 2025 y 21.601 de diciembre 2024).                                     | Descartado: tenía otro formato de columnas y columnas adicionales, incompatibles con los años anteriores.                 |
| Volumen de datos                  | 101.500 filas, 27 columnas.                                                                               | 62.500 filas, 20 columnas (19 originales más un id incremental).                                                          |
| Identificador                     | Columna \_id de la fuente.                                                                                | Se eliminan los ids originales y se crea un id único incremental de 1 a 62.500.                                           |
| Cobertura temporal                | Enero de cada año más diciembre 2024 y enero 2025.                                                        | Enero de 2018 a enero de 2024, más febrero 2022 (incluido en el archivo 2022).                                            |
| Variables tributarias             | Presentes solo para diciembre 2024 y enero 2025.                                                          | Ya no forman parte de la base (existían solo en el dataset 2025).                                                         |
| Fechas en formato serial de Excel | Inconsistencia que afectaba al 38,4 % de los registros.                                                   | Resuelta de raíz: anio y mes tienen formato homogéneo en todos los archivos.                                              |
| Repositorio                       | Un archivo de datos.                                                                                      | Se subieron nuevamente las bases de cada año (ver sección de estructura del repositorio).                                 |

Con esta decisión ganamos consistencia (mismas columnas y mismos formatos en todos los períodos) y perdimos dos cosas que conviene tener presentes: el dato de diciembre 2024 – enero 2025 y el detalle impositivo. Ambas quedan fuera del alcance del análisis actual.

## Descripción del dataset

El dataset contiene información sobre estaciones de servicio y comercialización de combustibles, integrando datos de identificación y características de los establecimientos, ubicación geográfica, operadores y banderas comerciales, tipo de negocio, productos comercializados, canales de comercialización y variables asociadas al volumen y precio de los combustibles. La información se encuentra organizada a nivel de estación de servicio – producto – período, permitiendo analizar el comportamiento del volumen comercializado en función de diferentes características de la estación, el producto y las condiciones de comercialización.

## Entre las principales variables disponibles se encuentran

- **Identificación:** id (generado al unificar), CUIT y número de inscripción de la estación.
- **Ubicación:** dirección, localidad y provincia.
- **Características comerciales:** operador, bandera, tipo de negocio y canal de comercialización.
- **Producto:** tipo de combustible comercializado.
- **Actividad:** indicador de no movimientos (no_novimientos, nombre tal como figura en la fuente), condición de exento y fecha de baja.
- **Período:** año y mes correspondientes al registro.
- **Volumen:** cantidad de combustible comercializado (variable objetivo del modelado).
- **Precios:** precio sin impuestos, precio con impuestos y precio de surtidor.

Los componentes impositivos y tasas (impuesto a los combustibles líquidos, impuesto al dióxido de carbono, tasa vial, tasa municipal, ingresos brutos, IVA y fondo fiduciario de GNC) solo existían en el dataset de diciembre 2024 – enero 2025, que fue descartado; por eso no integran la base actual.

## Dimensión temporal

El dataset no constituye una serie temporal mensual continua. La información disponible corresponde a enero de cada año entre 2018 y 2024, complementada con febrero de 2022, que viene incluido en el archivo de ese año. Por este motivo, los períodos sin información no se consideran datos faltantes susceptibles de ser interpolados o reconstruidos. El análisis temporal se utiliza como una dimensión descriptiva y de comparación entre los períodos efectivamente observados, sin asumir continuidad mensual ni realizar pronósticos sobre meses no disponibles.

## Objetivo analítico

A partir de estas variables, el dataset permite explorar los factores asociados al volumen de combustible comercializado y desarrollar modelos de aprendizaje automático orientados a estimar dicho volumen a partir de las características disponibles de las estaciones, productos y condiciones de comercialización.

## Estructura del repositorio y reproducibilidad

Cada conjunto de datos se carga por separado y la unificación se realiza dentro de la notebook. Para esto se subieron nuevamente al repositorio las bases de cada año:

| **Archivo**     | **Registros** | **Observación**                                   |
|-----------------|---------------|---------------------------------------------------|
| EESS_2018.csv   | 2.250         | Enero 2018                                        |
| EESS_2019.csv   | 3.500         | Enero 2019                                        |
| EESS_2020.csv   | 7.750         | Enero 2020                                        |
| EESS_2021.csv   | 2.250         | Enero 2021 (año de pandemia)                      |
| EESS_2022.csv   | 35.000        | Enero y febrero 2022 (el archivo más grande)      |
| EESS_2023.csv   | 7.500         | Enero 2023                                        |
| EESS_2024.csv   | 4.250         | Enero 2024                                        |
| Total unificado | 62.500        | 19 columnas por archivo, 20 con el id incremental |

## Para reproducir el análisis

- Subir los siete CSV anuales a la carpeta /content de Google Colab (o clonar el repositorio con los archivos en esa ruta).
- Ejecutar la notebook TripleInsight_Pre-entrega3.ipynb en orden. La celda de carga detecta automáticamente los archivos EESS\_\*.csv, descarta las columnas de id previas, concatena todo en estaciones_de_servicio_unificado y genera el id incremental.
- No incluir el dataset 2025 en esa carpeta: tiene otra estructura de columnas y la carga unificada lo tomaría por error. El archivo de salida EESS_limpio.csv se excluye de la carga por su nombre.
- Librerías utilizadas: pandas, numpy, matplotlib, seaborn, scipy y scikit-learn.

El resultado de la limpieza se guarda como EESS_limpio.csv (codificación utf-8-sig), conservando df_original para mantener la trazabilidad.

## Principales hallazgos del análisis — actualizado 21/10/2026

### Calidad y estructura de los datos

- 62.500 registros y 20 columnas luego de unificar los siete archivos anuales. No hay filas duplicadas exactas. Las combinaciones estación – período – producto repetidas (10.798 filas) no se eliminan, porque pueden ser registros válidos (por ejemplo, distintos canales de comercialización).
- La base cubre ocho períodos: enero de cada año entre 2018 y 2024 y febrero de 2022. Los registros por archivo son muy dispares (de 2.250 a 35.000) y las cifras redondas sugieren que cada archivo es una extracción parcial; por eso la cantidad de estaciones por período varía entre 554 y 5.194 y los totales no son comparables entre años.
- Registros sin movimientos: 685 filas (1,1 %) tienen no_novimientos = SI, exentos nulo y volumen cero. Las interpretamos como estaciones inactivas y las eliminamos; con ello también desaparece el producto N/D. Crear la variable sin_movimientos (True/False) nos permitió comprobarlo con una tabulación cruzada: los 685 nulos de exentos coinciden exactamente con esas filas.
- Se eliminaron además 44 registros con precio sin impuestos mayor al precio con impuestos, una incoherencia lógica. El dataset limpio queda en 61.771 registros.
- fecha_de_baja tiene 92 % de nulos, y eso no es un error: son las estaciones que siguen activas. No se usa en el modelado.
- No se detectaron volúmenes negativos, años fuera de rango ni meses inválidos. Las dimensiones finales son 5.523 estaciones, 9 productos, 4.033 operadores y 24 provincias.
- precio_surtidor vale cero en el 15,1 % de los registros (9.332). Lo interpretamos como “no aplica” (por ejemplo, ventas que no son al público) y no como precio real, por lo que se trata como dato ausente cuando se analiza su relación con el volumen.

### Comportamiento del volumen

- El volumen es extremadamente asimétrico: la mediana es 55,8, el percentil 75 es 135,6 y el percentil 90 es 460,9, pero el percentil 95 salta a 72.930 y el máximo supera 1,1 millones. Conviven dos realidades: una enorme mayoría de registros pequeños y un grupo reducido de volúmenes gigantes. Estos valores altos no se eliminan, porque probablemente corresponden a ventas a terceros o a productos como el GNC, y no a errores.
- El GNC explota el promedio: AGIRA (115.571), ASPRO (106.304) e YPF (94.953) lideran el volumen promedio por registro, muy por encima de los combustibles líquidos. Es una hipótesis a validar que el GNC se mida en una unidad distinta a la de los líquidos, ya que la fuente no la documenta.
- Geografía: Buenos Aires concentra el mayor volumen registrado (205 millones), seguida de Córdoba (86 millones), Santa Fe (39 millones), Mendoza (30 millones) y Capital Federal (27 millones). Es volumen acumulado de los registros disponibles, no una medida de demanda anual.
- Banderas: Blanca lidera el volumen total (144 millones) pero con una mediana baja (19,3), mientras que YPF es la que más registros tiene (23.160) y una mediana de 80,0. Volumen total y mediana cuentan historias distintas y conviene mirar ambas.
- La mediana del volumen es estable entre períodos (entre 48,8 y 60,2): lo que cambia de un año a otro es cuántas estaciones tiene cada extracción, no el tamaño típico de una venta.

### Precios

- Los precios muestran el efecto de la inflación argentina: el precio de surtidor promedio pasa de 20 en 2018 a 715 en 2024 (más de 35 veces). Por eso comparar precios entre años sin controlar el período no tiene sentido.
- La correlación de Spearman entre volumen y precios es débil: −0,21 en el total de la base (que mezcla productos y años) y cercana a cero dentro de un mismo producto (−0,08 para nafta súper y −0,01 para gasoil grado 2). El precio de una estación no explica, por sí solo, cuánto vende. Los tres precios están casi perfectamente correlacionados entre sí (0,97), por lo que son variables redundantes.
- En 33.584 registros el precio con impuestos difiere del precio de surtidor. Lo dejamos documentado como alerta sin asumir un significado que la fuente no permite demostrar. Los precios se excluyeron del modelo.

### Modelado predictivo

|             | **Prueba I**                                                                                                            | **Prueba II**                                                                                                             |
|-------------|-------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| Target      | Volumen en escala original.                                                                                             | log1p(volumen); luego se revierte con expm1.                                                                              |
| Población   | 4 productos más frecuentes, con volumen mayor a cero (55.836 filas).                                                    | Solo venta minorista (bocas de expendio y estaciones de servicio): 58.711 filas, con las mismas 4 variedades de producto. |
| Predictores | Año, mes, operador, bandera, tipo de negocio, provincia, producto y canal.                                              | Año, mes, producto, localidad agrupada, bandera agrupada y variable de pandemia (2021).                                   |
| Partición   | Aleatoria por estación (GroupShuffleSplit 80/20): 3.990 estaciones en entrenamiento y 998 en prueba, sin superposición. | Cronológica: entrenamiento hasta febrero 2022 (47.730 filas) y prueba en enero 2023 y enero 2024 (10.981 filas).          |
| Validación  | GroupKFold de 5 pliegues por estación.                                                                                  | GroupKFold de 5 pliegues por estación.                                                                                    |

| **Resultado (validación cruzada)** | **MAE** | **RMSE** | **R²** | **Escala** |
|------------------------------------|---------|----------|--------|------------|
| Prueba I · Random Forest           | 474     | 7.721    | −0,01  | Original   |
| Prueba I · Árbol de Decisión       | 657     | 8.256    | −0,21  | Original   |
| Prueba I · Regresión Lineal        | 987     | 7.752    | −0,03  | Original   |
| Prueba II · Random Forest          | 0,83    | 1,19     | 0,78   | log1p      |
| Prueba II · Árbol de Decisión      | 0,83    | 1,19     | 0,78   | log1p      |
| Prueba II · Regresión Lineal       | 0,87    | 1,21     | 0,78   | log1p      |
- **Prueba I:** ningún modelo logró generalizar (R² cero o negativo) y un baseline que predice siempre la mediana los superó. Los pocos registros gigantes distorsionan las pérdidas cuadráticas y absolutas.
- **Prueba II:** la transformación logarítmica, el filtrado a venta minorista y la agrupación de categorías poco frecuentes (localidades y banderas con menos de 100 registros pasan a “Otras”) cambian el resultado: R² de 0,78 en los tres modelos. Los árboles tienen una leve ventaja en MAE sobre la regresión lineal, pero las diferencias entre modelos son pequeñas.
- Lo que mejoró el desempeño no fue la complejidad del algoritmo, sino cómo se prepararon los datos. Los resultados de la Prueba II están en escala logarítmica y no son comparables numéricamente con los de la Prueba I.

## La historia detrás de los datos

### 1. Siete fotos de enero

Todo empezó con una decisión de diseño. Teníamos una base grande que mezclaba diciembre de 2024 y enero de 2025 con formatos de fecha, columnas e impuestos que no existían en los años anteriores. En lugar de forzar una unión, optamos por la coherencia: cargamos cada año por separado, descartamos el dataset de 2025 y unificamos los siete archivos de 2018 a 2024 con código reproducible. Lo que tenemos no es una película del mercado de combustibles sino una colección de fotos de enero (y un febrero), y el análisis respeta esa naturaleza.

### 2. Poner orden

De los 62.500 registros iniciales, 685 eran estaciones sin movimientos, es decir, filas vacías de actividad, y 44 tenían un precio sin impuestos mayor al precio final. Tras depurarlos quedaron 61.771 registros confiables, repartidos entre 5.523 estaciones de las 24 provincias. Para modelar, el universo se achicó aún más: solo venta minorista y los cuatro combustibles que dominan el mercado (gasoil grado 2 y 3, nafta súper y premium).

### 3. Dos mundos de volumen

El primer hallazgo del análisis exploratorio fue que “volumen” no es una sola cosa. La mitad de los registros vende menos de 56 unidades, pero a partir del percentil 95 aparecen valores de decenas de miles, muchos de ellos asociados al GNC. Es un mercado de enormes desigualdades: Buenos Aires, Córdoba y Santa Fe concentran la mayor parte del volumen registrado, y banderas como Blanca suman mucho en total pero con ventas típicas muy chicas. Mientras tanto, los precios cuentan otra historia: se multiplicaron por más de 35 entre 2018 y 2024 por la inflación, y aun así no mostraron una relación clara con lo que vende cada estación.

### 4. El primer intento fracasa, y enseña

Cuando le pedimos a los algoritmos que predijeran el volumen tal cual está, no aprendieron nada útil: ganó la estrategia más simple posible, predecir siempre la mediana. No fue un error de los modelos sino de la pregunta: con una distribución tan extrema, los pocos valores gigantes dominan cualquier métrica de error.

### 5. El segundo intento: cambiar la escala

Replanteamos el problema. Pasamos el volumen a escala logarítmica, quedándonos con la venta minorista, agrupamos las localidades y banderas con pocos registros, incorporamos una marca para el año de pandemia y dividimos los datos por tiempo: entrenar con el pasado (hasta febrero 2022) y evaluar con el futuro (2023 y 2024), que es la prueba más exigente. El R² pasó de valores nulos o negativos a 0,78. Los tres algoritmos alcanzaron un nivel parecido, lo que sugiere que la información que explica el volumen está en dónde se ubica la estación, qué producto vende y bajo qué bandera opera, más que en la sofisticación del modelo.

### 6. Qué nos llevamos

Podemos afirmar que el volumen que vende una estación es en buena medida predecible a partir de su ubicación, su bandera y el producto, siempre que se trabaje en escala logarítmica y se interprete el resultado como un orden de magnitud y no como un valor exacto. También sabemos qué no podemos afirmar: que existan tendencias mensuales o estacionales, que el precio afecte la demanda o que el modelo sirva para pronosticar meses que no observamos.

## Limitaciones y próximos pasos

- La base son fotos de enero (y febrero 2022) con muestras de tamaño desigual entre años: no permite series de tiempo ni conclusiones de estacionalidad. La baja de 2021 puede reflejar tanto la pandemia como que ese archivo tiene solo 554 estaciones.
- La unidad de medida del volumen no está documentada en la fuente y puede diferir entre GNC y combustibles líquidos. Es necesario validarla antes de sumar o comparar volúmenes entre productos.
- El conjunto de prueba cronológico tiene solo dos períodos (enero 2023 y enero 2024), por lo que la evaluación final debe leerse como indicativa.
- Para agilizar la validación cruzada se usó un Random Forest reducido (30 árboles, profundidad 12). Queda pendiente ajustar hiperparámetros y consolidar las métricas finales sobre el conjunto de prueba en litros reales.
