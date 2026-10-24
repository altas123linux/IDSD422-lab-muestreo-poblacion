# Informe — Clasificación de variables y población vs. muestra

**Para:** Gerencia de RetailAndina S.A. · **De:** Equipo de Business Intelligence
**Fecha:** ___/___/2026 · **Asunto:** Base de fidelización: tipos de variables y estimación por muestreo

---

## Resumen ejecutivo

En un inicio se clasificaron las 8 columnas de la base de fidelización en las cuales hay  4 numéricas y 4 categóricas, de las cuales 2 son ordinales y se verificó cada clasificación en código. Después se quiso estimar el ingreso mensual promedio de los 400 clientes usando muestras de 20, con tres diseños: conveniencia, aleatorio simple y estratificado por ciudad. El parámetro real es 1330.72 USD y las estimaciones fueron 1348.50, 1336.00 y 1287.50 USD, pero ese resultado no basta para elegir un diseño ganador. Lo que la gerencia debe saber es que la muestra por conveniencia dejó fuera a dos de las tres ciudades, y que cinco sorteos aleatorios dieron medias entre 1235 USD y 1491.5 USD se evidencia que el diseño del muestreo y la variabilidad del sorteo importan más que el número de filas.

## Parte A — Clasificación de las variables

*Completa la tabla usando el **criterio** (¿tiene sentido la mitad del valor? ¿hay orden entre las categorías?), no el ejemplo. En «verificación» anota qué mostró el código: dtype, número de valores únicos y, si declaraste el orden, cómo lo comprobaste.*

| Variable | Tipo | Justificación (criterio) | Verificación en código |
|---|---|---|---|
| `cliente_id` | 	Categórica nominal| 	Es una etiqueta que identifica al cliente, ademas no hay orden entre los códigos.| dtype object; 12 valores únicos|
| `edad` | 	Numérica continua| Se mide en una escala con sentido aritmético:| dtype int64; 12 valores únicos|
| `numero_hijos` | 	Numérica discreta| Se cuenta y solo toma valores entero|dtype int64 |
| `genero` | 	Categórica nominal| Tiene 3 categorías sin orden entre ellas como femenino masculino y otro| dtype object|
| `nivel_educacion` | Categórica ordinal| 	Hay un orden natural (Primaria < Secundaria < Universidad < Posgrado |dtype object
| `ciudad` | 	Categórica nominal| Quito, Guayaquil y Cuenca son grupos sin orden|dtype object |
| `ingresos_mensuales` | 	Numérica continua| Es un monto en USD con sentido aritmético:| dtype int64|
| `satisfaccion` | 	Categórica ordinal| Es una escala de 1 a 5 con orden| dtype int64|

**P1.** ¿Qué te dijo el dtype de `satisfaccion` y por qué no basta para clasificarla? ¿Qué pasa si una ordinal queda como texto sin declarar el orden?

El dtype de satisfaccion fue int64, es decir, pandas la leyó como un número entero. Pero el dtype solo describe cómo se almacenó el dato, no qué significa de modo que el criterio dice que es ordinal porque hay orden entre los niveles, pero no tiene sentido la mitad del valor ni afirmar que un 4 es el doble de un 2. Si se tratara como numérica, se podría calcular una media y darle un significado que no tiene. Si una ordinal queda como texto sin declarar el orden, y operaciones como .min() o una comparación darían resultados sin sentido. Además,  conviene declarar la escala completa 1 a 5 porque  en las 12 filas nadie puso 1, y sin declararla el nivel 1 no existiría.

## Parte B — Población vs. muestra

### La población y su parámetro

| Concepto | Valor |
|---|---|
| Población (N) | $ 400|
| Variable de interés | ingresos_variablles $|
| Parámetro poblacional (media) |$ 1330.72 |

*¿Por qué se muestrea si aquí el parámetro existe y se calculó?*

 Aqui el parámetro sirve para juzgar qué tan bien estima cada diseño. Se muestrea para estimarlo.


### Los tres diseños

*Corre tu script y registra las salidas. La composición por ciudad es la que delata el sesgo: anótala tal como salió.*

| Diseño | n | Media estimada | Composición por ciudad | Diferencia vs. parámetro |
|---|---|---|---|---|
| Conveniencia (`head`) |20 | 	1348.50 USD| 	Quito 20 | +17.78|
| Aleatorio simple (`sample`) | 20| 1336.00 USD| 	Cuenca 8, Quito 8, Guayaquil 4| +5.28|
| Estratificado (`groupby` + 5 %) | 20| 	1287.50 USD| 	Quito 10, Guayaquil 6, Cuenca 4| −43.22|

*Anota también las medias de 5 sorteos distintos del aleatorio simple (semillas 1 a 5):*

| Semilla | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|
| Media | 	1336.00| 	1405.00| 	1235.00| 	1491.50| 	1288.00|

**P2.** ¿Cuánto vale el parámetro poblacional y cuánto estimó cada diseño? ¿Cuál quedó más cerca y por qué eso **no** alcanza para declarar un ganador?

El parámetro vale 1330.72 USD. La conveniencia estimó 1348.50 (+17.78), el aleatorio simple 1336.00 (+5.28) y el estratificado 1287.50 (−43.22). El más cercano fue el aleatorio simple, pero eso no alcanza para declararlo ganador: cada diseño se probó con un solo sorteo, y esa cercanía puede estar sesgada. Con otras semillas el mismo diseño aleatorio simple se alejó mucho más (en las semillas 3 y 4 quedó a unos −96 y +161 USD del parámetro). Además, la conveniencia quedó razonablemente cerca solo por casualidad, porque su muestra es 100 % de Quito, y en la base Quito gana en promedio más que las otras ciudades, así que su cercanía no vuelve confiable al diseño. De modo que se evidencia que un diseño se juzga por cómo se comporta al repetirse muchas veces, no por un único resultado.

**P3.** La muestra por conveniencia también tiene 20 filas. ¿Qué defecto tiene y cómo lo delatas en la composición por ciudad? ¿Se corrige tomando más filas?

Tiene un sesgo de selección ya que el archivo está ordenado por ciudad, y al tomar las primeras 20 filas no se hace ningún sorteo, solo se toma el principio del archivo. Asi, La composición por ciudad lo delata donde las 20 filas son de Quito y faltan por completo Guayaquil (30 %) y Cuenca (20 %), es decir, la mitad de la base no está representada. No se corrige tomando más filas de modo que  lo que arregla el sesgo es el diseño.

**P4.** El sorteo aleatorio dio un número distinto en cada semilla. ¿Qué implica esa variabilidad para una empresa que va a decidir con un solo estudio?

Las cinco medias fueron 1336.00, 1405.00, 1235.00, 1491.50 y 1288.00 USD. Una empresa que decide con un solo estudio recibe una de esas cinco posibles respuestas sin saber cuál, y podría concluir que el ingreso medio es 1235 o 1491 USD solo por el azar del sorteo. Por eso la estimación de una muestra no es el parámetro si no que se debe presentarse con su margen de incertidumbre, y no debería tomarse una decisión importante con un solo número de una muestra pequeña.

**P5.** ¿Cuándo usarías cada diseño en la práctica? Relaciona cada uno con un uso real (piloto, subgrupos que deben aparecer, conclusión sobre toda la población).

Conveniencia: para un piloto o una exploración rápida por ejemplo, probar si una encuesta se entiende con los primeros 20 clientes que llegan, donde no se busca generalizar. 
Aleatorio simple: cuando se quiere una conclusión sobre toda la población y no se sabe nada especial de ella.
Estratificado: cuando hay subgrupos que deben aparecer en la muestra, por ejemplo para una campaña que se evalúa por ciudad. Se garantiza la proporción de cada grupo 10 de Quito, 6 de Guayaquil y 4 de Cuenca.

## Conclusiones

Se evidenció que el diseño decide la representatividad, no el tamaño. La muestra por conveniencia y las demás tuvieron las mismas 20 filas, pero la primera dejó fuera a Guayaquil y Cuenca por completo. Más filas del mismo diseño sesgado no habrían corregido el problema.

Se observó que un solo resultado no basta para juzgar un diseño ni para decidir.  Asi, el aleatorio simple quedó más cerca del parámetro con la semilla 1, pero sus cinco sorteos variaron entre 1235 y 1491.5 USD. Asi, Una media muestral es una estimación con incertidumbre, no el valor verdadero. Ademas el estratificado, aun con peor resultado en este sorteo, asegura la composición por ciudad.


