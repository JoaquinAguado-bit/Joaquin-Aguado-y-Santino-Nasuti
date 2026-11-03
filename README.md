# <M19> · <¿Esta Sseleccion pasa fase de grupos?>

> **Hackatón IIA · UADE · 3.4.257.** Este README es la plantilla del grupo. En su fork, reemplacen todo lo que está entre `< >` y borren esta nota. Las bases están en `BASES.md` y su ficha en `problematicas/`.

**Integrantes:** <Joaquin AGUADO> · <Santino Nasuti>
**IA declarada:** <Claude>
**Pista:** <Neurona>
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada seleccion, ¿supera la fase de grupos? SÍ significa: La seleccion pasa la fase grupos>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | <Costo total de errores sobre los casos de prueba (c_FP × FP + c_FN × FN, con c_FP = 2 y c_FN = 1), comparado contra la línea base "responder siempre NO". Se mide contra el resultado real de la fase de grupos. Se reportan también exactitud, precisión, sensibilidad y especificidad.> |
| Entorno | < Selecciones que disputan un Mundial. El analista de la federación decide antes del torneo con la información disponible en ese momento. Parcialmente observable, estocástico y episódico (cada selección se evalúa por separado).> |
| Actuadores | <Emite SÍ o NO para cada selección; el analista usa esa respuesta para decidir a qué selecciones apostar.> |
| Sensores | <Ranking FIFA (puesto), goles a favor en eliminatorias (por partido) y jugadores en ligas top (de 26). No usa amistosos ganados.> |

**Objetivo y métrica.** <No son lo mismo. El objetivo es que el analista apueste solo a las selecciones que realmente pasan la fase de grupos y no subestime a las que sí pasan. La métrica es la forma de medir cuánto se logra: el costo de los errores (un falso positivo cuesta el doble que un falso negativo) y la comparación contra decir siempre NO, que acierta bastante en exactitud pero tiene sensibilidad cero. Los amistosos ganados no se usan como entrada ni como indicador de éxito: es el indicador con el que la federación evalúa al técnico, no si la selección pasa.>

## 2. El sistema

**Cómo decide.** <Pista Reglas, con tres reglas si-entonces que se aplican en orden:
1. Si el ranking FIFA es 20 o mejor (puesto ≤ 20) → SÍ.
2. Si no, y tiene 24 o más jugadores en ligas top (de 26) → SÍ.
3. En cualquier otro caso → NO.>

**Variables que usa y por qué.** <Usa ranking FIFA y jugadores en ligas top: se conocen antes del Mundial y separaron bien los casos de entrenamiento. Descarta:
- **Amistosos ganados:** no es lo que queremos predecir (es el indicador con que evalúan al técnico) y además no ayuda: las selecciones que pasan ganaron en promedio 2,8 amistosos y las que no, 5,7.
- **Goles en eliminatorias:** no separó los casos (el caso 13, que no pasó, tiene 2,0, y el caso 8, que sí pasó, tiene 1,2) y depende de qué tan fácil fue la clasificatoria.>

**Cómo se corre en vivo.** <Se toman el ranking y los jugadores en ligas top de la selección nueva y se aplican las reglas en orden. En Colab: `predecir(ranking, jugadores_top)` devuelve 1 (SÍ) o 0 (NO). También se puede hacer a mano con las tres reglas.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <ranking 10, 15 jugadores en ligas top> | <SÍ (regla 1)> |
| B | <ranking 40, 20 jugadores en ligas top > | < NO (regla 3)> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <> | <> |
| Precisión | <> | <> |
| Sensibilidad | <> | <> |
| Especificidad | <> | <> |

**Entrenamiento contra prueba.** <¿Hay diferencia? ¿Por qué? ¿Sobreajuste o generalización?>

## 4. El error más caro

<**El falso positivo es peor.** Según la ficha, es apostar a que pasa una selección que queda afuera. El falso negativo, en cambio, es subestimar a una selección que sí pasa. Equivocarse apostando cuesta más que quedarse corto.

**Qué cambiamos para cometer menos falsos positivos.**
- El sistema responde NO por defecto y solo dice SÍ cuando hay evidencia fuerte.
- Subimos el umbral de jugadores en ligas top de 20 a 24. Con 20 aparecían 4 falsos positivos en entrenamiento (casos 2, 3, 11 y 13). Con 24 quedaron en 0.
- El costo fue perder el caso 8 (ranking 57, 23 jugadores, SÍ), que ahora es 1 falso negativo. Lo aceptamos a cambio de eliminar los falsos positivos.>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Un sistema que responde siempre NO (lo más frecuente en entrenamiento) saca 9/14 = 64,3 % de exactitud en entrenamiento y 3/6 = 50 % en prueba, pero no detecta ninguna selección que pasa (sensibilidad 0 %). Nuestro sistema saca 92,9 % en entrenamiento y 100 % en prueba, y además detecta los SÍ (sensibilidad 80 % y 100 %). Supera a la línea base en las dos partes.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <Ficha de 20 selecciones inventadas: 14 de entrenamiento (casos 1 a 14) y 6 de prueba (casos 15 a 20).> |
| ¿Qué métrica? | <Precisión como principal, junto con exactitud, sensibilidad y especificidad.> |
| ¿Comparado contra qué? | <Contra un sistema que responde siempre NO> |
| ¿Quién lo midió? | <El grupo, en Colab con los datos de la ficha, y el evaluador de la cátedra. |> |

## 6. Riesgo: el camino completo

<**Datos.** Son solo 20 selecciones inventadas, y el umbral de 24 jugadores en ligas top quedó justo por encima del máximo de los casos NO (23). Los datos de la ficha son pocos y el ajuste fue sobre ellos.

**Modelo.** Las reglas pueden fallar con selecciones que no se parezcan a esos 20 casos. Por ejemplo, una selección con ranking 25 y 23 jugadores en ligas top sería NO, aunque podría pasar (es el mismo error del caso 8). Además, el 100 % en prueba se apoya en solo 6 casos, por lo que no garantiza que funcione igual con casos nuevos.

**Decisión.** Si el sistema dice SÍ y la selección queda afuera (falso positivo), el analista apuesta por ella.

**Consecuencia.** Se pierde la apuesta o se toma una mala decisión en la federación. Por eso el sistema responde NO ante la duda y exige mucho para decir SÍ: acepta perder algunos SÍ reales (falsos negativos) para evitar el error más caro.

**Riesgo concreto: amistosos ganados.** Si alguien agregara esa variable al sistema, los equipos que ganan amistosos sin presión parecerían mejores de lo que son, y sería más fácil caer en falsos positivos. Además, estaría optimizando lo que mide al técnico y no lo que importa, que es pasar la fase de grupos. Por eso la dejamos afuera.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | <> | <> | <> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <> | <> | <> |
