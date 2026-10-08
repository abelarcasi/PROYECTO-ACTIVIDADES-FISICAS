# Análisis de práctica de actividades recreativas en la población peruana, durante el año 2021
Proyecto de data analitics

## Resumen del proyecto

El estado peruano desea saber cómo viene la ejecución del presupuesto para tomar dos decisiones, la primera, reconocimiento a los lugares que vienen ejecutando por encima del 80%, la segunda, identificar/cuestionarse y saber los motivos de porque se le asigna presupuesto para que al final no lo ejecuten. Sin embargo, no cuentan con una visión clara de los datos pertinentes de los gobiernos locales. Mi objetivo es utilizar SQL dentro de SQL Server Management Studio, analizando sus datos para proporcionar recomendaciones al departamento de Instituto Peruano del Deporte por el Ministerio de Educación.

## Estructura del proyecto
    - Sobre los Datos
    - Limpieza de Datos

## Sobre los datos

El conjunto de datos incluye una tabla que capturan la ejecución del presupuesto a nivel departamental/regiones del Perú,girado de presupuesto, población, niveles de satisfacción, puntaje y avance del proyecto (%) distribuidos en más de 1,192 registros y 15 columnas.

![imagen de baner](./picture/TOP%203%20BASE%20UNIVERSAL.png)

## Tareas (Task)

En este análisis, ayudo al departamento de Instituto Peruano del Deporte por el Ministerio de Educación.
1.	Presupuesto promedio: ¿Cuál fue el presupuesto promedio para los gobiernos regionales para el año 2021?
2.	Retención: ¿Cuántos gobiernos locales en cada departamento superan el presupuesto promedio?
3.	Satisfacción de la población: ¿Cuáles son los top 10 de gobiernos locales según provincia con mejor puntuación y menor puntuación?
4.	Equidad de presupuesto: ¿A mayor población el presupuesto girado debería incrementar?
5.	Desempeño del proyecto medido por el avance (en porcentaje): ¿de un total de 1192 lugares cuantos son mayores al 85% con el supuesto que 85% sea considerado proyecto por terminarse?
6.	Población y avance del proyecto: ¿Acaso es cierto que donde existe mayor población el porcentaje de avance es cercano al 85%?
7.	Proyecto Top por los gobiernos locales: Identificar a los top 10 proyectos gobiernos locales según departamento, provincia y calificación de avance del proyecto.
8.	Criterio de ubicación: Toma consideración la ubicación geográfica que por ser región costa los proyectos se cumplen a comparación de región selva o región sierra.
9.	Dilema y criterio de ubicación regional: Toma consideración la ubicación geográfica que por ser región costa se le asigne mayor número de presupuesto considerándose que mayor a S/5M sea presupuesto alto a comparación de región selva o región sierra.
10.	Alerta de presupuesto o semáforo rojo: Identificar el porcentaje de proyectos que se le asignó al Compromiso anual S/0.00 de presupuesto 

## Limpieza de Datos

Antes de realizar el análisis, es fundamental asegurar que los datos estén limpios y que no haya valores faltantes. 
```SQL
SELECT COUNT(*) AS MissingValues
FROM ACTIVIDADES_FISICAS
WHERE PIA IS NULL
OR PIM IS NULL
--Efectivament, 0 como MissingValues
```
![imagen de baner](./picture/cero%20valores%20faltantes.jpg)

```SQL
SELECT GOBIERNOS_LOCALES, COUNT(*) AS REPETICION
FROM ACTIVIDADES_FISICAS
GROUP BY GOBIERNOS_LOCALES
HAVING COUNT(*) > 1
--Efectivamente, no se tiene valores duplicados.
```
![imagen de baner](./picture/cero%20valores%20duplicados.jpg)

## Análisis Exploratorio de Datos (EDA) e Insights

Pregunta #1: ¿Cuál fue el presupuesto promedio para los gobiernos regionales para el año 2021?

```SQL
SELECT AVG(GIRADO) AS AVERAGE_GIRADO
FROM ACTIVIDADES_FISICAS
WHERE GIRADO > 0

--El presupuesto promedio considerando a los montos girados mayores a 0 es de S/565,143 
```

![imagen de baner](./picture/PRESUPUESTO%20PROMEDIO.jpg)

Pregunta #2: ¿Cuál es el presupuesto promedio para los gobiernos regionales para el año 2021 según departamento excluyendo aquellos montos de presupuesto de S/0.00 ?

```SQL
SELECT DEPARTAMENTO,
AVG(GIRADO) AS AVERAGE_GIRADO
FROM ACTIVIDADES_FISICAS
WHERE GIRADO > 0
GROUP BY DEPARTAMENTO
ORDER BY AVERAGE_GIRADO DESC

--Primer puesto S/1,595,845 para Moquegua y último departamento Lambayeque S/115,620. Importante destacar que Lambayeque, es el único departamento que no tiene ningún gobierno local que supere el presupuesto promedio.
```

![imagen de baner](./picture/PRESUPUESTO%20PROMEDIO%20II.jpg)

Pregunta #3: ¿Cuántos gobiernos locales por departamento superan el presupuesto promedio?

```SQL
SELECT DEPARTAMENTO,
       COUNT(DISTINCT GOBIERNOS_LOCALES) AS CANT_GOBIERNOS_LOCALES
FROM ACTIVIDADES_FISICAS
WHERE GIRADO > (SELECT AVG(GIRADO) FROM ACTIVIDADES_FISICAS WHERE GIRADO > 0)
GROUP BY DEPARTAMENTO
ORDER BY CANT_GOBIERNOS_LOCALES DESC;

--Primer puesto para Cusco con 46 gobiernos locales, sin embargo, Pasco, Loreto y Ucayali solo superan una sola vez el presupuesto promedio. Otra inferencia a resaltar es que Lambayeque es el único departamento que no tiene ningún gobierno local que supere el presupuesto promedio.
```

![imagen de baner](./picture/PRESUPUESTO%20PROMEDIO%20III.jpg)

Pregunta #4: ¿Cuáles son los top 10 de gobiernos locales según provincia con mejor puntuación y menor puntuación?

-- TOP 10 con mayor puntuación
```SQL

SELECT TOP 10 DEPARTAMENTO,
			  PROVINCIA,
			  GOBIERNOS_LOCALES,
              PUNTAJE_DE_POBLACION
FROM ACTIVIDADES_FISICAS
ORDER BY PUNTAJE_DE_POBLACION DESC

--Mejor puntuación para Apurimac, Cusco, Lima y entre otros.
```

![imagen de baner](./picture/MEJOR%20PUNTUACION.jpg)

--TOP 10 con menor puntuación
```SQL
SELECT TOP 10 DEPARTAMENTO,
			  PROVINCIA,
			  GOBIERNOS_LOCALES,
			  PUNTAJE_DE_POBLACION
FROM ACTIVIDADES_FISICAS
ORDER BY PUNTAJE_DE_POBLACION ASC

--Menor puntuación Apurimac, Cusco, Lima y entre otros.No obstante, aunque Lima destaca por registrar algunos de los mejores resultados, también presenta bajas puntuaciones en las provincias de Huarochirí, Yauyos y Oyón.
```

![imagen de baner](./picture/MENOR%20PUNTUACION.jpg)

Pregunta #5: ¿A mayor población el presupuesto girado debería incrementar?

```SQL
SELECT TOP 10 DEPARTAMENTO,
			  PROVINCIA,
			  GOBIERNOS_LOCALES,
			  GIRADO,
			  POBLACION
FROM ACTIVIDADES_FISICAS
ORDER BY GIRADO DESC

--No existe una relación directa entre el tamaño de la población y el presupuesto aprobado. Por ejemplo, Arequipa (Arequipa), con 1,175,765 habitantes, tuvo un presupuesto aprobado de S/ 5,732,259; mientras que Espinar (Cusco), pese a contar con solo 38,886 habitantes, registró un presupuesto aprobado de S/ 23,132,819, superior al de Arequipa.
```

![imagen de baner](./picture/POBLACION%20Y%20PRESUPUESTO.jpg)

Pregunta #6: ¿de un total de 1,192 gobiernos locales cuantos son mayores al 85% con el supuesto que 85% sea considerado proyecto por terminarse?

```SQL
--Primer paso
SELECT	DEPARTAMENTO,
		PROVINCIA,
		GOBIERNOS_LOCALES,
		AVANCE_PORCENTAJE
FROM ACTIVIDADES_FISICAS
WHERE AVANCE_PORCENTAJE > 85.00
ORDER BY GIRADO DESC

--Menos del 50%, especificamente 42% con un total de 497 de gobiernos locales superan el 85% de proyecto por terminarse lo que intuye a que más de la media no tiene buen avance del proyecto haciendo que los proyectos se estiren para el próximo año.

--Segundo paso
SELECT COUNT(DISTINCT GOBIERNOS_LOCALES) AS CANT_SUPERAN_85
FROM ACTIVIDADES_FISICAS
WHERE AVANCE_PORCENTAJE > 85.00;
```
![imagen de baner](./picture/PROYECTO%20POR%20TERMINADO.jpg)
![imagen de baner](./picture/CONTEO%20PROYECTO%20CASI%20TERMINADO.jpg)

Pregunta #7: Identificar a los top 10 proyectos gobiernos locales según departamento, provincia y calificación de avance del proyecto.

```SQL
SELECT TOP 10 DEPARTAMENTO,
			  PROVINCIA,
			  GOBIERNOS_LOCALES,
			  PUNTAJE_DE_AVANCE,
			  GIRADO,
			  POBLACION
FROM ACTIVIDADES_FISICAS
ORDER BY PUNTAJE_DE_AVANCE DESC
```
![imagen de baner](./picture/TOP%2010.jpg)

Pregunta #8: Cuenta cuantas veces se repite puntaje de avance N°5 y puntaje de población N°5 según región para identificar que región predomina mejor.

```SQL
SELECT REGION,
       SUM(CASE WHEN PUNTAJE_DE_AVANCE = 5 THEN 1 ELSE 0 END) AS CANT_PUNTAJE_AVANCE,
       SUM(CASE WHEN PUNTAJE_DE_POBLACION = 5 THEN 1 ELSE 0 END) AS CANT_PUNTAJE_POBLACION
FROM ACTIVIDADES_FISICAS
GROUP BY REGION
ORDER BY CANT_PUNTAJE_AVANCE DESC, CANT_PUNTAJE_POBLACION DESC;

--El predominante es Sierra con mayor número de puntaje de avance y según puntaje de población Costa con nota de 5.
```
![imagen de baner](./picture/PUNTAJE%20DE%20LA%20POBLACION%20Y%20AVANCE.jpg)

Pregunta #9: ¿Acaso tiene consideración la ubicación geográfica que por ser región costa se le asigne mayor número de presupuesto considerándose que mayor a S/5M sea presupuesto alto a comparación de región selva o región sierra?

```SQL
SELECT REGION,
       COUNT(*) AS TOTAL_GOBIERNOS,
       SUM(CASE WHEN GIRADO > 5000000 THEN 1 ELSE 0 END) AS CANT_PRESUPUESTO_ALTO,
       CAST(SUM(CASE WHEN GIRADO > 5000000 THEN 1 ELSE 0 END) AS DECIMAL(10,2)) * 100.0 / COUNT(*) AS PORCENTAJE_PRESUPUESTO_ALTO
FROM ACTIVIDADES_FISICAS
GROUP BY REGION
ORDER BY PORCENTAJE_PRESUPUESTO_ALTO DESC;
--No necesariamente por ser catalogada un gobierno local como Región Costa se debe considerar como presupuesto alto.
```
![imagen de baner](./picture/Preg%209.jpg)

Pregunta #10. Alerta de presupuesto o semáforo rojo: ¿Cuáles son los proyectos de gobiernos locales que se le asignó S/0.00 de presupuesto?
```SQL
SELECT DEPARTAMENTO,
       PROVINCIA,
       GOBIERNOS_LOCALES,
	   REGION,
       GIRADO
FROM ACTIVIDADES_FISICAS
WHERE GIRADO = 0
ORDER BY DEPARTAMENTO, PROVINCIA;
--Se observa de manera detallada y organizada por departamento, provincia, gobierno local y región a cuantos se le asignó S/0.00 de presupuesto con la finalidad de considerarlos observados.
```
![imagen de baner](./picture/PRESUPUESTO%200.00%20I.jpg)

```SQL
SELECT DEPARTAMENTO,
       COUNT(*) AS CANT_PRESUPUESTO_CERO
FROM ACTIVIDADES_FISICAS
WHERE GIRADO = 0
GROUP BY DEPARTAMENTO
ORDER BY CANT_PRESUPUESTO_CERO DESC;
--Se observa el conteo de forma descendentemente por departamento de cuantos se le asignó S/0.00 de presupuesto con la finalidad de considerarlos observados.
```
![imagen de baner](./picture/PRESUPUESTO%200.00%20II.jpg)














