# Taller Autónomo: Modelado Combinatorio y All-Pairs

**Clase 09 - Bloque 2 | Configurador web SmartDrive Motors**

| Rol | Integrante |
|---|---|
| Analista de Modelado | _(nombre)_ |
| Tester Combinatorio | _(nombre)_ |
| Documentador / Git Lead | _(nombre)_ |

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

<!--PASO3-->

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
