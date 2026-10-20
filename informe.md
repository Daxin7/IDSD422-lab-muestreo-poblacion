# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** 19/10/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

Se clasificaron las 7 variables de la base de fidelización (12 clientes de muestra inicial, 400 en la población completa) según su naturaleza: numéricas continuas (edad, ingresos_mensuales), numérica discreta (numero_hijos), categóricas nominales (cliente_id, genero, ciudad), y categóricas ordinales (nivel_educacion, satisfaccion). Se estimó el ingreso mensual promedio poblacional (parámetro = 1,330.72 USD) usando tres diseños de muestreo de n=20: conveniencia (head), aleatorio simple (sample) y estratificado (groupby + frac=0.05). La muestra por conveniencia tiene sesgo severo (solo Quito), el aleatorio simple varía entre sorteos (1,235–1,491.50 USD), y el estratificado garantiza representación de todas las ciudades. La gerencia debe saber que el **diseño**, no el tamaño, define la representatividad.

## Parte A — Clasificación de las variables

*Completa la tabla usando el **criterio** (¿tiene sentido la mitad del valor? ¿hay orden entre las categorías?), no el ejemplo. En «verificación» anota qué mostró el código: dtype, número de valores únicos y, si declaraste el orden, cómo lo comprobaste.*

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | Categórica nominal | Identificador único; no tiene orden ni sentido aritmético | dtype: str, 12 valores únicos (C001–C012) |
| `edad` | Numérica continua | Tiene sentido la mitad del valor (17.5 años existe) | dtype: int64, 12 valores únicos (22–48) |
| `numero_hijos` | Numérica discreta | Valores enteros; no tiene sentido 1.5 hijos | dtype: int64, 5 valores únicos (0–4) |
| `genero` | Categórica nominal | Categorías sin orden natural: Femenino, Masculino, Otro | dtype: str, 3 valores únicos |
| `nivel_educacion` | Categórica ordinal | Hay orden: Primaria < Secundaria < Universidad < Posgrado | dtype: str → CategoricalDtype(ordered=True); 4 valores únicos; .min() = Primaria, .max() = Posgrado |
| `ciudad` | Categórica nominal | Ciudades sin orden natural: Quito, Guayaquil, Cuenca | dtype: str, 3 valores únicos |
| `ingresos_mensuales` | Numérica continua | Tiene sentido la mitad del valor (725 USD existe) | dtype: int64, 12 valores únicos (760–2450) |
| `satisfaccion` | Categórica ordinal | Escala 1–5 con orden; 4 no es el doble de 2 | dtype: int64 → CategoricalDtype(ordered=True); 4 valores únicos (2–5); .min() = 2, .max() = 5 |

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype de `satisfaccion` es `int64` (número entero), lo que podría inducir a clasificarla erróneamente como numérica. Sin embargo, el criterio decide: aunque sean números, la escala 1–5 tiene orden pero **no sentido aritmético** (4 no es el doble de 2, no se puede promediar significativamente). Si una ordinal queda como texto sin declarar el orden (p. ej. `nivel_educacion` como string), pandas la ordena alfabéticamente: "Posgrado" quedaría antes que "Primaria", rompiendo la lógica del análisis. Declarar el orden con `CategoricalDtype(ordered=True)` fuerza el orden correcto y permite operaciones como `.min()` y `.max()` semánticamente válidas. Además, un typo en la lista de categorías (ej. "Postgrado" en vez de "Posgrado") convierte silenciosamente los valores no coincidentes en `NaN` (en la prueba: 2 NaN), un error que pasa desapercibido si no se verifica.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | 400 clientes |
| Variable de interés | ingresos_mensuales |
| Parámetro poblacional (media) | 1,330.72 USD |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

Se muestrea para **demostrar** cómo se comportan distintos diseños de muestreo y cuál produce estimaciones representativas. En la práctica real, la población no suele ser accesible completa (costo, tiempo, privacidad), por lo que se debe estimar el parámetro a partir de una muestra. Este ejercicio sirve como "espejo" para juzgar la calidad de cada diseño cuando sí se conoce la verdad.

### Los tres diseños

*Corre tu script y registra las salidas. La composición por ciudad es la que delata el sesgo: anótala tal como salió.*

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) | 20 | 1,348.50 USD | Quito: 20 | +17.78 USD |
| Aleatorio simple (`sample`) | 20 | 1,336.00 USD | Cuenca: 8, Quito: 8, Guayaquil: 4 | +5.28 USD |
| Estratificado (`groupby` + 5 %) | 20 | 1,287.50 USD | Quito: 10, Guayaquil: 6, Cuenca: 4 | -43.22 USD |

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media | 1,336.00 | 1,405.00 | 1,235.00 | 1,491.50 | 1,288.00 |

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro poblacional es **1,330.72 USD**. Estimaciones: Conveniencia = 1,348.50 USD (+17.78), Aleatorio simple (semilla 1) = 1,336.00 USD (+5.28), Estratificado = 1,287.50 USD (-43.22). El **aleatorio simple (semilla 1)** quedó más cerca del parámetro en esta corrida. Sin embargo, **un solo número no basta para declarar un ganador** porque: (1) el aleatorio simple tiene alta variabilidad entre sorteos (rango 1,235–1,491.50 USD, amplitud 256.50 USD), por lo que otra semilla podría estar lejos; (2) el estratificado, aunque en esta corrida está más lejos, **garantiza** representación de todas las ciudades (proporcionalidad exacta), eliminando el sesgo de selección sistemático; (3) la evaluación de un diseño debe basarse en sus propiedades estadísticas (sesgo, varianza, cobertura), no en la suerte de un sorteo puntual.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

El defecto es **sesgo de selección sistemático**: al tomar `head(20)` de un archivo ordenado por ciudad (primero 200 Quito), la muestra contiene **solo Quito** (20/20 Quito, 0 Guayaquil, 0 Cuenca). La composición por ciudad lo delata inmediatamente: faltan dos ciudades enteras. **No se corrige tomando más filas** (p. ej. `head(100)` seguiría siendo 100% Quito hasta la fila 200). El sesgo es estructural por el orden del archivo; solo cambiando el **diseño** (aleatorizar o estratificar) se elimina.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

La variabilidad entre semillas (1,235–1,491.50 USD) muestra **error de muestreo**: cada muestra aleatoria simple es un sorteo distinto y da una estimación distinta. Para una empresa que decide con **un solo estudio**, esto implica incertidumbre real: el resultado observado podría estar lejos del parámetro solo por azar (en este caso, hasta ±160 USD del parámetro). La solución no es "no muestrear", sino: (a) usar un diseño que reduzca la variabilidad (estratificado), (b) reportar intervalos de confianza, no solo puntuales, (c) aumentar n si es posible, y (d) repetir el estudio o usar métodos de remuestreo (bootstrap) para cuantificar la incertidumbre.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

- **Conveniencia (`head`):** Piloto rápido / prueba de concepto / exploración inicial del código o calidad de datos. **No** para inferencia sobre la población.
- **Aleatorio simple (`sample`):** Cuando la población es homogénea, no hay subgrupos críticos, y se acepta la variabilidad del sorteo. Útil para auditoría o cuando no hay marco de muestreo con variables de estratificación.
- **Estratificado (`groupby` + `frac`):** Cuando **subgrupos clave deben aparecer** en la muestra (ciudades, segmentos, sucursales) y se quiere concluir sobre **toda la población** con precisión controlada por estrato. Garantiza representación proporcional y reduce varianza entre estratos.

## Conclusiones

1. **El diseño de muestreo define la representatividad, no el tamaño de la muestra.** La muestra por conveniencia de 20 clientes (solo Quito) es sistemáticamente sesgada aunque se aumente a 100 o 200; en cambio, el diseño estratificado de 20 clientes garantiza que todas las ciudades estén representadas en su proporción poblacional.

2. **La variabilidad entre muestras aleatorias es real y debe comunicarse.** Un solo sorteo aleatorio simple puede dar una estimación cercana al parámetro (semilla 1: +5.28 USD) o muy lejana (semilla 4: +160.78 USD). Reportar solo la estimación puntual oculta esta incertidumbre; se deben usar intervalos de confianza o repetir el muestreo.

3. **Recomendación a gerencia:** Antes de aceptar un análisis basado en muestras, exijan conocer: (a) el **diseño de muestreo** usado (¿conveniencia, aleatorio simple, estratificado?), (b) si hay **subgrupos relevantes** que deben estar representados (ciudades, segmentos), y (c) la **incertidumbre** de la estimación (intervalo de confianza, no solo la media). Un análisis con muestra por conveniencia, por muy grande que sea, no permite generalizar a la población completa de RetailAndina.
