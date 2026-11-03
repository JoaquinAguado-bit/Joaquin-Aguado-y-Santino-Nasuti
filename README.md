# <M19> · <¿Esta seleccion pasa fase de grupos?>

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

Pesos: Ranking -1, jugadores_top +1
Sesgo: +29

**Variables que usa y por qué.** <Usa ranking FIFA y jugadores en ligas top: se conocen antes del Mundial y separaron bien los casos de entrenamiento. Descarta:
- **Amistosos ganados:** no es lo que queremos predecir (es el indicador con que evalúan al técnico) y además no ayuda: las selecciones que pasan ganaron en promedio 2,8 amistosos y las que no, 5,7.
- **Goles en eliminatorias:** no separó los casos (el caso 13, que no pasó, tiene 2,0, y el caso 8, que sí pasó, tiene 1,2) y depende de qué tan fácil fue la clasificatoria.>

**Cómo se corre en vivo.** <Se toman el ranking y los jugadores en ligas top de la selección nueva y se aplican las reglas en orden. En Colab: `predecir(ranking, jugadores_top)` devuelve 1 (SÍ) o 0 (NO). También se puede hacer a mano con las tres reglas.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <ranking 10, 15 jugadores en ligas top> | <SÍ (regla 1)> |
| B | <ranking 40, 10 jugadores en ligas top > | < No (regla 3)> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <3> | FN = <2> |
| **Real NO** | FP = <1> | VN = <8> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <2> | FN = <1> |
| **Real NO** | FP = <1> | VN = <2> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <78.6%)> | <66.7%> |
| Precisión | <75.0%> | <66.7%> |
| Sensibilidad | <60.0%> | <66.7%> |
| Especificidad | <88.9%> | <66.7%> |

**Entrenamiento contra prueba.** <Comparación contra la línea base ("responder siempre NO").** En entrenamiento, la línea base acierta 9 de 14 (64,3 %) pero con sensibilidad 0: no detecta ninguna selección que pasa. Su costo de errores (falso positivo = 2, falso negativo = 1) es 5, contra 3 del sistema. En prueba, la línea base acierta 3 de 6 (50 %), también con sensibilidad 0, y su costo es 3, contra 0 del sistema. El sistema le gana a la línea base en ambos conjuntos.

**Entrenamiento contra prueba.** El sistema rinde mejor en prueba que en entrenamiento, lo cual es lo contrario de lo esperable. Con solo 6 casos de prueba lo atribuimos al azar, no a que el modelo generalice mejor: un solo error habría bajado la exactitud a 83 %. Los pesos no se ajustaron mirando la prueba. Además, el caso 19 quedó con un puntaje apenas positivo (+0,06), por lo que acertó con margen muy chico.

**Limitaciones.** Datos inventados, 14 casos de entrenamiento y 6 de prueba: las métricas son ruidosas y no alcanzan para afirmar que el sistema funcionaría con selecciones reales.>

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
| 1 | < Encuadrar el caso (pregunta SÍ/NO, PEAS, objetivo y métrica)> | <Dejar afuera "amistosos ganados": es el indicador con que evalúan al técnico, no el objetivo, y además no ayuda a predecir. Usar la precisión como métrica principal.> | <Usar la exactitud como única métrica, porque hay que compararla contra una línea base trivial> |
| 2 | Armar un sistema de reglas si-entonces con los casos 1 a 14 | Reglas con ranking ≤ 20 o 24 o más jugadores en ligas top → SÍ. | Umbral de 20 jugadores (daba 4 falsos positivos) y usar goles en eliminatorias (no separaba los casos). |
| 3 | Armar una neurona (perceptrón) con los casos 1 a 14 | Pesos y sesgo elegidos probando combinaciones sobre el entrenamiento. | La regla clásica de aprendizaje del perceptrón: no converge porque los datos no se separan con una línea, y daba muy poca sensibilidad. |
| 4 | Una segunda neurona, porque la primera daba precisión y especificidad en 100 % | Neurona v2: z = −1 × ranking + 7 × jugadores top − 100. Acepta 1 falso positivo en entrenamiento a cambio de subir la sensibilidad de 60 % a 80 %. | Ajustar los pesos mirando los casos 15 a 20 para forzar errores en la prueba: se usaría información de la prueba para decidir, y la prueba no sería una medición honesta. |
| 5 | Una tercer neurona, porque en la segunda neurona la precisión y especificidad seguian en 100 % | Neurona v3: z = −1 × ranking + 1 × jugadores top +29. Acepta 1 falso negativo en entrenamiento a cambio de tener la misma sensibilidad de 60 %| Ajustando los pesos y el sesgo para que no la sesibilidad y la especifidad no den 100% |
| 6 | Calcular matrices, métricas y línea base, y escribir el código en Python | Código en Colab con la función neurona y las matrices de entrenamiento y prueba. | Pegar los números sin comprobarlos: los corrimos y verificamos que den igual que en el README. |
| 7 | Redactar las secciones del README | Textos de PEAS, error más caro, línea base y riesgo. | Las versiones largas: pedimos textos más cortos, con una idea por celda. |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 7,5 | Aplicá las tres reglas en orden a los casos 1 al 14 y contá VP, FN, FP y VN. ¿Te da la matriz que pusieron? Fijate especialmente en los casos 1 y 8. | Agregamos toda la bitacora, de los casos 1 a 6. |
| 2 | 7 | ¿El sistema que describen en el README es el mismo que midieron? Corré neurona_mundial_v3.py y compará su salida con las matrices del README. Otros: ¿Con qué pesos y sesgo corre tu sistema, y dónde están escritos en el README? ¿De dónde sale el margen +0,06 del caso 19? | modificamos la primer corrida, porque nos faltaba agregar que cambiamos y agregamos los pesos y el sesgo de la neurona en el sistema |
| 3 | 7,5 | Agujero principal: ¿Cuál de los dos sistemas es el que midieron? Aplicá las reglas escritas al caso 20 (ranking 23, 8 jugadores) y compará con lo que da la neurona del script. Otros: ¿El caso B del README (ranking 40, 20 jugadores) da NO con los pesos que pusiste? ¿Qué sensibilidad tiene tu tabla de prueba, y cuál escribiste en el texto? | modificamos la bitacora, agregamos la seccion 5, la tercer neurona, que aceptamos y porque la aceptamos |
| 4 | 7 | Agujero principal: ¿Cuánto da el costo de tu sistema en prueba y cuánto el de la línea base? ¿Le gana en las dos partes, como dice el README? Otros: ¿Qué responde tu sistema con el caso B, con los pesos que escribiste? ¿Qué sensibilidad y exactitud tiene v3 frente a v2, según tus propias tablas? | Modificamos la primer corrida porque decia que no explicamos que cambiamos, tambien modificamos la sección 5 de la bitacora, cambiando el que pedimos y el porque, tambien modificamos el caso B de caso inventado, para que de resultado NO |
