# analisis-de-datos-estaciones-de-servicio
Principales hallazgos del análisis - 29/09/2026
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
Total general	101500
• El dataset tiene información de solo 1 mes por cada año y contamos con 8 años, salvo 2024 que tenemos 2 meses.
•	El dataset tiene una inconsistencia temporal importante:
    o	históricamente anio = año y mes = mes;
    o	diciembre 2024 y enero 2025 están almacenados como seriales de fecha de Excel.
•	Esos dos períodos representan 39.000 registros, aproximadamente 38,4% del dataset.
•	Hay un cambio muy marcado en la cobertura de los campos tributarios:
    o	hasta enero 2024 están prácticamente 100% nulos;
    o	en diciembre 2024/enero 2025 pasan a estar informados en aproximadamente 99% de los registros.
•	Detectamos además un cambio de comportamiento/escala en las variables de precio, especialmente precio_con_impuestos, precio_sin_impuestos y su relación con precio_surtidor. Lo documentamos como alerta y no asumimos una unidad ni significado que el Excel no permita demostrar.
•	no_novimientos = SI constituye un caso especial: aparece asociado a producto = N/D y valores cero en métricas. No debemos tratarlo simplemente como missing.
•	fecha_de_baja tiene un formato particular como 27/2021/10, que corresponde a 27-oct-2021, por lo que requiere transformación explícita.
•	Hay valores negativos puntuales en algunas variables tributarias. Los marcamos para validación y no vamos a eliminarlos automáticamente, porque podrían ser ajustes/reversiones.
