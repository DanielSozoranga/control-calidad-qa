# Taller Autónomo: Modelado Combinatorio y All-Pairs

**Clase 09 - Bloque 2 | Configurador web SmartDrive Motors**

| Rol | Integrante |
|---|---|
| Analista de Modelado | Daniel Sozoranga |
| Tester Combinatorio | Ricardo Alvarez |
| Documentador / Git Lead | Daniel Sozoranga |

---

## Paso 1: Matriz de Parámetros y Valores (P&V)

| ID | Parámetro (P) | Valores (V) | n (Cantidad) |
|----|---------------|-------------|--------------|
| P1 | Motor | Gasolina, Híbrido, Eléctrico | 3 |
| P2 | Transmisión | Manual, Automática, Monomarcha | 3 |
| P3 | Frenos | Estándar, ABS, Regenerativo | 3 |
| P4 | Mercado | América, Europa, Asia | 3 |
| P5 | Modo de conducción | Eco, Sport, Autónomo | 3 |

**Supuestos del modelado** (las diapositivas del enunciado tenían inconsistencias):

- En el diagrama, el "PARAM 2" aparece rotulado como *Modo Conducción* con el valor *Monomarcha*, y el "PARAM 5" aparece duplicado. Como la tabla objetivo y las reglas de la Parte 4 usan **Transmisión**, se interpretó el P2 como Transmisión.
- La tabla objetivo incluye la columna **Mercado** (América, Europa), cuyos valores completos no se muestran. Se asumió un tercer valor, **Asia**, para mantener n = 3.
- Los valores de Transmisión (Manual, Automática, Monomarcha) se tomaron de la tabla objetivo y de la Regla 1.

---

## Paso 2: El peligro de la explosión combinatoria

### Cálculo

Cada parámetro tiene 3 valores y hay 5 parámetros, por lo tanto:

```
v1 × v2 × v3 × v4 × v5 = 3 × 3 × 3 × 3 × 3 = 3^5
```

### Demostración paso a paso

```
3^2 = 3 × 3     = 9
3^3 = 9 × 3     = 27
3^4 = 27 × 3    = 81
3^5 = 81 × 3    = 243
```

**Total de combinaciones puras = 243.**

### Costo de la prueba exhaustiva

Si ejecutar y verificar cada prueba toma 15 minutos:

```
243 × 15 min = 3.645 min = 60,75 horas ≈ 7,6 jornadas de 8 horas
```

La suite All-Pairs de este documento tiene 11 casos:

```
11 × 15 min = 165 min = 2,75 horas
```

### Conclusión

La prueba exhaustiva exige más de una semana laboral completa de una persona (≈ 61 horas) para un configurador de solo 5 parámetros, y el costo crece exponencialmente con cada parámetro nuevo; por eso es financieramente inviable frente a las ≈ 2,75 horas de la suite All-Pairs, que mantiene la cobertura de las interacciones que concentran la mayoría de los defectos.

---

## Paso 3: Derivación de la suite All-Pairs (t = 2)

### Método

Se generó la suite con un **algoritmo voraz (greedy)**: del conjunto de las 243 combinaciones se elige repetidamente la fila que cubre más pares aún no cubiertos, hasta cubrirlos todos. Se repitió con 3.000 semillas aleatorias y se conservó la suite más corta. Después se **verificó por programa** que cada par esté cubierto al menos una vez.

### Tamaño del problema

- Pares de columnas: C(5,2) = **10**
- Pares de valores por cada par de columnas: 3 × 3 = **9**
- Pares a cubrir en total: 10 × 9 = **90**

Cada fila cubre 10 pares, así que el mínimo teórico sería 90 / 10 = 9 filas. Para 5 parámetros de 3 valores no existe una cobertura de 9 filas; el mínimo conocido es **11**, que es lo que se obtuvo.

### Suite reducida (sin restricciones, matemáticamente pura)

| Test Case | Motor | Transmisión | Frenos | Mercado | Modo |
|-----------|-------|-------------|--------|---------|------|
| TC-01 | Eléctrico | Manual | Regenerativo | Europa | Sport |
| TC-02 | Gasolina | Automática | ABS | América | Sport |
| TC-03 | Gasolina | Monomarcha | Estándar | Europa | Autónomo |
| TC-04 | Híbrido | Automática | Regenerativo | Asia | Autónomo |
| TC-05 | Eléctrico | Monomarcha | ABS | Asia | Eco |
| TC-06 | Híbrido | Manual | Estándar | América | Eco |
| TC-07 | Gasolina | Automática | Regenerativo | Europa | Eco |
| TC-08 | Eléctrico | Automática | Estándar | América | Autónomo |
| TC-09 | Híbrido | Monomarcha | Regenerativo | América | Sport |
| TC-10 | Híbrido | Manual | ABS | Europa | Autónomo |
| TC-11 | Gasolina | Manual | Estándar | Asia | Sport |

**Reducción: de 243 a 11 casos (-95,5 %).**

### Verificación de ortogonalidad (t = 2)

Se comprobó cada una de las 10 parejas de columnas: las **9 combinaciones posibles están presentes** al menos una vez en las filas.

| Pareja de columnas | Combinaciones cubiertas |
|--------------------|-------------------------|
| Motor - Transmisión | 9 / 9 |
| Motor - Frenos | 9 / 9 |
| Motor - Mercado | 9 / 9 |
| Motor - Modo | 9 / 9 |
| Transmisión - Frenos | 9 / 9 |
| Transmisión - Mercado | 9 / 9 |
| Transmisión - Modo | 9 / 9 |
| Frenos - Mercado | 9 / 9 |
| Frenos - Modo | 9 / 9 |
| Mercado - Modo | 9 / 9 |
| **Total** | **90 / 90 (100 %)** |

---

## Paso 4: Identificación de Constraints (restricciones lógicas)

Al revisar la suite anterior se detecta que **varias filas son imposibles de fabricar** (por ejemplo, TC-07 pide Motor de Gasolina con Frenos Regenerativos). Se definen las siguientes reglas de negocio:

```
REGLA 1:
IF Motor = Eléctrico
THEN Transmisión = Monomarcha
 AND Frenos = Regenerativo

REGLA 2:
IF Motor = Gasolina
THEN Frenos != Regenerativo

REGLA 3:
IF Transmisión = Monomarcha
THEN Motor = Eléctrico
```

> La Regla 3 es una propuesta del equipo (un motor eléctrico no necesita caja de cambios y las demás motorizaciones sí). Debería confirmarse con el cliente.

### Impacto de las restricciones sobre la suite pura

| Test Case | ¿Válido? | Regla violada |
|-----------|----------|---------------|
| TC-01 | No | R1 (Eléctrico exige Monomarcha) |
| TC-02 | Sí | Ninguna |
| TC-03 | No | R3 (Monomarcha sin Motor Eléctrico) |
| TC-04 | Sí | Ninguna |
| TC-05 | No | R1 (Eléctrico exige Frenos Regenerativos) |
| TC-06 | Sí | Ninguna |
| TC-07 | No | R2 (Gasolina con Regenerativo) |
| TC-08 | No | R1 (Eléctrico exige Monomarcha y Regenerativo) |
| TC-09 | No | R3 (Monomarcha sin Motor Eléctrico) |
| TC-10 | Sí | Ninguna |
| TC-11 | Sí | Ninguna |

**6 de 11 casos (55 %) son inejecutables.** Si se enviara la suite pura a automatización, más de la mitad fallaría por una configuración inexistente.

### Suite All-Pairs con constraints aplicados

Se regeneró la suite partiendo únicamente de las **99 configuraciones válidas** (de las 243). Con las restricciones, algunos pares dejan de ser alcanzables (por ejemplo, Gasolina-Monomarcha), por lo que el objetivo pasa a ser cubrir los **81 pares válidos**.

| Test Case | Motor | Transmisión | Frenos | Mercado | Modo |
|-----------|-------|-------------|--------|---------|------|
| TC-01 | Gasolina | Automática | ABS | América | Autónomo |
| TC-02 | Eléctrico | Monomarcha | Regenerativo | América | Sport |
| TC-03 | Híbrido | Manual | Estándar | América | Eco |
| TC-04 | Híbrido | Manual | ABS | Europa | Sport |
| TC-05 | Híbrido | Manual | Regenerativo | Asia | Autónomo |
| TC-06 | Gasolina | Automática | Estándar | Asia | Sport |
| TC-07 | Híbrido | Automática | Regenerativo | Europa | Eco |
| TC-08 | Eléctrico | Monomarcha | Regenerativo | Asia | Eco |
| TC-09 | Gasolina | Manual | Estándar | Europa | Autónomo |
| TC-10 | Eléctrico | Monomarcha | Regenerativo | Europa | Autónomo |
| TC-11 | Gasolina | Automática | ABS | Asia | Eco |

**Verificación:** las 11 filas cumplen las 3 reglas y cubren el **100 % (81/81)** de los pares válidos.

---

## Protocolo Pre-Commit (autoevaluación)

- [x] ¿La Matriz P&V define los 5 parámetros y sus variables discretas?
- [x] ¿Se calculó y justificó el límite de la explosión combinatoria (3^5)?
- [x] ¿La suite All-Pairs fue generada y listada correctamente?
- [x] ¿Las reglas formales de exclusión (constraints) reflejan el caso de negocio?

---

## Cierre cognitivo (preguntas de transferencia)

### Pregunta 1 (Transferencia)

**¿Qué riesgo financiero y operativo corremos si ignoramos los constraints lógicos y enviamos la matriz matemática pura directamente al equipo de automatización (QA)?**

En este mismo taller, 6 de los 11 casos de la suite pura (55 %) describen vehículos que no se pueden fabricar, como un motor de gasolina con frenos regenerativos. Si se envían tal cual a automatización, esos scripts fallarán por una configuración inexistente y no por un defecto real: el equipo gasta horas escribiendo, ejecutando y depurando pruebas inválidas, los reportes se llenan de falsos positivos que ocultan los defectos genuinos y se pierde la confianza en la suite. En el peor caso, el configurador aceptaría combinaciones imposibles y enviaría a la línea de ensamblaje pedidos que no se pueden construir, lo que genera retrabajo, devoluciones y retrasos de entrega en un lanzamiento global donde cada fallo cuesta millones. Modelar las restricciones antes de generar la suite evita ese costo.

### Pregunta 2 (Elaboración)

**¿Por qué la técnica All-Pairs es matemáticamente y empíricamente superior a que un tester diseñe 20 casos de prueba basándose únicamente en su intuición?**

Matemáticamente, All-Pairs ofrece una garantía verificable: con solo 11 casos cubre el 100 % de los 90 pares de valores posibles, y esa cobertura se puede demostrar y medir, mientras que 20 casos elegidos por intuición normalmente repiten combinaciones "típicas" y dejan pares sin cubrir sin que nadie lo note; además, con 11 casos ya supera en eficiencia a los 20 casos manuales. Empíricamente, los estudios del NIST muestran que la mayoría de los fallos de software son causados por la interacción de uno o dos parámetros, por lo que cubrir todos los pares atrapa el grueso de los defectos críticos con un costo mínimo. La intuición, en cambio, es sesgada, no es reproducible entre testers y no se puede auditar, mientras que All-Pairs es sistemático, repetible y deja evidencia de cobertura.
