# analisis-de-datos-estaciones-de-servicio

## Descripción del dataset
El dataset contiene información sobre estaciones de servicio y comercialización de combustibles, integrando datos de identificación y características de los establecimientos, ubicación geográfica, operadores y banderas comerciales, tipo de negocio, productos comercializados, canales de comercialización y variables asociadas al volumen y precio de los combustibles.
La información se encuentra organizada a nivel de estación de servicio – producto – período, permitiendo analizar el comportamiento del volumen comercializado en función de diferentes características de la estación, el producto y las condiciones de comercialización.

## Entre las principales variables disponibles se encuentran:
Identificación: identificadores de la estación, CUIT y número de inscripción.
Ubicación: dirección, localidad y provincia.
Características comerciales: operador, bandera, tipo de negocio y canal de comercialización.
Producto: tipo de combustible comercializado.
Actividad: cantidad de movimientos y condición de exento.
Período: año y mes correspondientes al registro.
Volumen: cantidad de combustible comercializado.
Precios: precio sin impuestos, precio con impuestos y precio de surtidor.
Componentes impositivos y tasas: impuesto a los combustibles líquidos, impuesto al dióxido de carbono, tasa vial, tasa municipal, ingresos brutos, IVA y fondo fiduciario de GNC. solo para diciembre 2024 y enero 2025
## Dimensión temporal:
El dataset no constituye una serie temporal mensual continua. La información disponible corresponde principalmente al mes de enero de distintos años, complementada con información de diciembre de 2024.
Por este motivo, los períodos sin información no se consideran datos faltantes susceptibles de ser interpolados o reconstruidos. El análisis temporal se utiliza como una dimensión descriptiva y de comparación entre los períodos efectivamente observados, sin asumir continuidad mensual ni realizar pronósticos sobre meses no disponibles.

## Objetivo analítico:
A partir de estas variables, el dataset permite explorar los factores asociados al volumen de combustible comercializado y desarrollar modelos de aprendizaje automático orientados a estimar dicho volumen a partir de las características disponibles de las estaciones, productos y condiciones de comercialización.

-------------------------------------------------------------------------------------------------------------------------------------------------------------------------

## Principales hallazgos del análisis - 29/09/2026
•	101.500 registros y 27 columnas.
•	_id es único y no hay filas duplicadas.
• Años	CA Registros
2018	2250
2019	3500
2020	7750
2021	2250
2022	35000
2023	7500
2024	4250
12-2024	21601
01-2025	17399

• El dataset tiene información de solo 1 mes por cada año y contamos con 8 años, salvo 2024 que tenemos 2 meses (enero y diciembre).
•	El dataset tiene una inconsistencia temporal importante:
    o	históricamente anio = año y mes = mes;
    o	diciembre 2024 y enero 2025 están almacenados como seriales de fecha de Excel.
•	Esos dos períodos representan 39.000 registros, aproximadamente 38,4% del dataset.
•	Hay un cambio muy marcado en la cobertura de los campos tributarios:
    o	hasta enero 2024 están prácticamente 100% nulos;
    o	en diciembre 2024/enero 2025 pasan a estar informados en aproximadamente 99% de los registros.
•	Detectamos además un cambio de comportamiento/escala en las variables de precio, especialmente precio_con_impuestos, precio_sin_impuestos y su relación con precio_surtidor. Lo documentamos como alerta y no asumimos una unidad ni significado que el Excel no permita demostrar.
•	no_novimientos = SI constituye un caso especial: aparece asociado a producto = N/D y valores cero en métricas. No debemos tratarlo simplemente como missing.
•	fecha_de_baja tiene un formato particular como 27/2021/10, que corresponde a 27-oct-2021, por lo que va a requerir transformación explícita.
•	Hay valores negativos puntuales en algunas variables tributarias. Los marcamos para validación y no vamos a eliminarlos automáticamente, porque podrían ser ajustes/reversiones.
