# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence  
**Fecha:** 23/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Clasifiqué las variables de la base de fidelización de RetailAndina S.A. según su naturaleza estadística y verifiqué su representación mediante código. Posteriormente estimé el ingreso mensual promedio de una población de 400 clientes utilizando muestras de 20 observaciones obtenidas mediante conveniencia, muestreo aleatorio simple y muestreo estratificado. El parámetro poblacional fue de **1,330.72 USD**. Los resultados mostraron que una muestra puede producir una estimación cercana al parámetro y aun así presentar problemas de representatividad, por lo que antes de tomar decisiones la gerencia debe considerar principalmente el diseño de muestreo utilizado y no solamente el tamaño de la muestra.

## Parte A — Clasificación de las variables

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Categórica nominal | Es un identificador; sus valores no tienen orden natural ni sentido aritmético. | `object`, 12 valores únicos (`C001`–`C012`). |
| `edad` | Numérica continua | Puede medirse con mayor precisión que años enteros; por ejemplo, 22.5 años es un valor posible. | `int64`, 12 valores únicos entre 22 y 48 años. |
| `numero_hijos` | Numérica discreta | Representa un conteo y solamente admite cantidades enteras; no tiene sentido tener 1.5 hijos. | `int64`, 5 valores únicos: 0, 1, 2, 3 y 4. |
| `genero` | Categórica nominal | Sus categorías no presentan un orden natural. | `object`, 3 valores únicos: Femenino, Masculino y Otro. |
| `nivel_educacion` | Categórica ordinal | Existe un orden natural: Primaria < Secundaria < Universidad < Posgrado. | `object` → `CategoricalDtype(ordered=True)`, 4 valores únicos; `.min()` = Primaria y `.max()` = Posgrado. |
| `ciudad` | Categórica nominal | Quito, Guayaquil y Cuenca son categorías sin un orden natural. | `object`, 3 valores únicos. |
| `ingresos_mensuales` | Numérica continua | Representa una cantidad monetaria sobre la que tienen sentido las operaciones aritméticas y puede expresarse con mayor precisión que números enteros. | `int64`, 12 valores únicos entre 760 y 2450 USD. |
| `satisfaccion` | Categórica ordinal | La escala 1–5 posee un orden, pero los números representan categorías; por ejemplo, 4 no significa el doble de satisfacción que 2. | `int64` → `CategoricalDtype(ordered=True)`, 4 valores observados: 2, 3, 4 y 5; `.min()` = 2 y `.max()` = 5. |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El `dtype` de `satisfaccion` fue `int64`, ya que sus categorías estaban representadas mediante números enteros. Sin embargo, comprobé que este tipo de dato no era suficiente para determinar su clasificación estadística. Los valores del 1 al 5 representan categorías con un orden natural, pero no cantidades con relaciones proporcionales; por ejemplo, un valor de 4 no representa el doble de satisfacción que un valor de 2. Por esta razón clasifiqué `satisfaccion` como una variable categórica ordinal.

También comprobé que una variable ordinal almacenada como texto puede perder su orden conceptual. En `nivel_educacion` declaré explícitamente el orden Primaria < Secundaria < Universidad < Posgrado mediante `CategoricalDtype(ordered=True)`. Después verifiqué el resultado con `.min()` y `.max()`, obteniendo `Primaria` y `Posgrado`.

Además, realicé una prueba escribiendo `"Postgrado"` en lugar de `"Posgrado"` dentro de las categorías. Como resultado, los dos registros que contenían `Posgrado` se convirtieron en valores faltantes (`NaN`). Esto mostró que un error de escritura en las categorías puede introducir datos faltantes de manera silenciosa si no se verifica la conversión.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes |
| Variable de interés | `ingresos_mensuales` |
| Parámetro poblacional (media) | 1,330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

En esta práctica pude calcular directamente el parámetro porque toda la población de 400 clientes estaba disponible. Sin embargo, utilicé muestras para observar cómo distintos diseños estimaban ese valor conocido.

En una situación real, analizar toda la población puede ser costoso, lento o incluso imposible. Por esta razón se utilizan muestras para estimar características de la población. En este ejercicio, conocer previamente el parámetro permitió utilizarlo como referencia para comparar la calidad de las estimaciones obtenidas con cada diseño.

### Los tres diseños

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---:|---:|---|---:|
| Conveniencia (`head`) | 20 | 1,348.50 USD | Quito: 20 | +17.78 USD |
| Aleatorio simple (`sample`) | 20 | 1,336.00 USD | Cuenca: 8, Quito: 8, Guayaquil: 4 | +5.28 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1,287.50 USD | Quito: 10, Guayaquil: 6, Cuenca: 4 | -43.22 USD |

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---:|---:|---:|---:|---:|
| Media | 1,336.00 | 1,405.00 | 1,235.00 | 1,491.50 | 1,288.00 |

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro poblacional fue de **1,330.72 USD**. La muestra por conveniencia estimó **1,348.50 USD**, con una diferencia de **+17.78 USD**. El muestreo aleatorio simple estimó **1,336.00 USD**, con una diferencia de **+5.28 USD**, mientras que el muestreo estratificado estimó **1,287.50 USD**, con una diferencia de **-43.22 USD**.

En esta ejecución, el muestreo aleatorio simple fue el que quedó más cerca del parámetro. Sin embargo, esto no fue suficiente para declararlo ganador, ya que al repetir el sorteo con diferentes semillas obtuve medias entre **1,235.00 USD y 1,491.50 USD**, con una amplitud de **256.50 USD**.

Además, aunque el muestreo estratificado quedó más lejos del parámetro en esta ejecución, garantizó que las tres ciudades estuvieran representadas en la misma proporción que en la población. Por esta razón, no evalué los diseños únicamente por la cercanía de una sola estimación, sino también por su procedimiento de selección y representatividad.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

La muestra por conveniencia presentó un sesgo de selección, ya que utilicé `head(20)` sobre un archivo cuya primera parte estaba formada por clientes de Quito. Como resultado, los 20 clientes seleccionados pertenecieron únicamente a Quito, mientras que Guayaquil y Cuenca no estuvieron representadas.

La composición por ciudad permitió identificar inmediatamente este problema. Aunque la media obtenida, **1,348.50 USD**, quedó relativamente cerca del parámetro de **1,330.72 USD**, esta cercanía no significó que la muestra fuera representativa.

El problema tampoco se corregiría simplemente tomando más filas si se mantuviera el mismo método de selección. Por ejemplo, `head(100)` seguiría seleccionando únicamente clientes de Quito. Por lo tanto, para corregir este defecto fue necesario cambiar el diseño de muestreo y no únicamente aumentar el tamaño de la muestra.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

Las cinco semillas produjeron estimaciones diferentes, desde **1,235.00 USD** hasta **1,491.50 USD**. Esto mostró la existencia de variabilidad muestral: diferentes muestras aleatorias de 20 clientes pueden producir resultados distintos aunque se utilice el mismo procedimiento de selección.

Para una empresa que tomara una decisión utilizando un solo estudio, esto implicaría que la estimación obtenida estaría sujeta a incertidumbre. Por ejemplo, con la semilla 1 obtuve una diferencia de solamente **+5.28 USD** respecto al parámetro, mientras que con la semilla 4 obtuve **1,491.50 USD**, es decir, **160.78 USD por encima** del valor poblacional.

Por esta razón concluí que una única estimación puntual debe interpretarse con precaución y que es importante considerar la variabilidad que puede existir entre diferentes muestras.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

Utilizaría el muestreo por conveniencia para realizar un piloto, comprobar rápidamente que el código funciona o hacer una exploración inicial de los datos. No lo utilizaría para realizar conclusiones sobre toda la población porque los registros seleccionados dependen de su posición o facilidad de acceso.

Utilizaría el **muestreo aleatorio simple** cuando todos los clientes pudieran ser seleccionados con la misma probabilidad y no fuera indispensable garantizar una cantidad determinada de cada subgrupo. Este diseño sería apropiado para realizar inferencias sobre la población siempre que se disponga de un marco de muestreo adecuado.

Utilizaría el **muestreo estratificado** cuando fuera importante garantizar la presencia de subgrupos específicos. En este caso permitió conservar la proporción de ciudades de la población: **50 % Quito, 30 % Guayaquil y 20 % Cuenca**, equivalentes a 10, 6 y 4 clientes en la muestra de 20.

## Conclusiones

1. Comprobé que el tipo de dato inferido por el lenguaje no determina por sí solo la clasificación estadística de una variable. `satisfaccion` fue interpretada como `int64`, pero por el significado y orden de sus categorías correspondió a una variable categórica ordinal. También comprobé que declarar correctamente el orden de una variable como `nivel_educacion` fue necesario para realizar comparaciones coherentes.

2. El diseño de muestreo fue más importante que el tamaño por sí solo. La muestra por conveniencia tuvo 20 observaciones y produjo una media relativamente cercana al parámetro, pero todos sus registros pertenecieron a Quito. En cambio, el muestreo estratificado garantizó la representación proporcional de Quito, Guayaquil y Cuenca. Por ello concluí que el diseño decide, no únicamente el tamaño de la muestra.

3. El muestreo aleatorio simple produjo la estimación más cercana al parámetro en la primera ejecución, con **1,336.00 USD frente a 1,330.72 USD**, pero las cinco semillas generaron medias entre **1,235.00 USD y 1,491.50 USD**. Por esta razón recomendaría a la gerencia revisar siempre el método con el que fue seleccionada una muestra y considerar la variabilidad de la estimación antes de utilizar sus resultados para tomar decisiones.
