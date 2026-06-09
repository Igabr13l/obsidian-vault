---
title: "Magnitudes, Cifras Significativas y Notación Científica"
type: note
status: active
tags:
  - quimica
  - cbc
  - magnitudes
  - calculos
aliases:
  - Cifras Significativas
  - Notación Científica
  - Magnitudes en Química
created: 2026-03-21
updated: 2026-03-21
source: Apuntes del CBC de Química
---

# Magnitudes, Cifras Significativas y Notación Científica

> [!INFO] Introducción
> En química y en todas las ciencias experimentales, medimos propiedades físicas y químicas. Para que estas medidas sean útiles y precisas, necesitamos entender cómo expresar **magnitudes**, cómo manejar números muy grandes o pequeños mediante **notación científica**, y cómo comunicar la precisión de la medida usando **cifras significativas**.

---

## Magnitudes Físicas

Una **magnitud física** es cualquier propiedad de un cuerpo o sistema que se puede medir y expresar mediante un número y una unidad. 

> [!DEFINITION] Magnitud
> **Magnitud** = Valor numérico + Unidad de medida. (Ejemplo: $5.0 \, \text{g}$, no solo "5").

### Sistema Internacional de Unidades (SI)

En química utilizamos convencionalmente el Sistema Internacional para estandarizar las mediciones:

| Magnitud | Unidad Base (SI) | Símbolo | Unidades Comunes en Química |
| :--- | :--- | :--- | :--- |
| **Masa** | Kilogramo | kg | gramo (g), miligramo (mg) |
| **Longitud** | Metro | m | centímetro (cm), nanómetro (nm) |
| **Volumen** | Metro cúbico | m³ | litro (L), mililitro (mL) o cm³ |
| **Temperatura** | Kelvin | K | grados Celsius (°C) |
| **Cantidad de sustancia** | Mol | mol | mol |

> [!WARNING] Cuidado con las unidades
> Nunca escribas un resultado numérico en química sin su unidad correspondiente. El número carece de sentido físico sin ella.

---

## Notación Científica

La **notación científica** es una forma compacta de escribir números que son extremadamente grandes o extremadamente pequeños, los cuales son muy comunes en química (por ejemplo, el Número de Avogadro o la masa de un electrón).

### Formato Estándar

Todo número en notación científica se escribe de la forma:
$$N \times 10^n$$

- **N (Coeficiente):** Es un número mayor o igual a 1 y estrictamente menor que 10 ($1 \le N < 10$).
- **n (Exponente):** Es un número entero (positivo, negativo o cero).

> [!TIP] Regla de desplazamiento de la coma
> - Si mueves la coma hacia la **izquierda**, el exponente $n$ es **positivo** (números grandes).
> - Si mueves la coma hacia la **derecha**, el exponente $n$ es **negativo** (números pequeños).

**Ejemplos:**
- $602.200.000.000.000.000.000.000 \rightarrow 6.022 \times 10^{23}$ (Movimos la coma 23 lugares a la izquierda).
- $0.00000000000000000016 \rightarrow 1.6 \times 10^{-19}$ (Movimos la coma 19 lugares a la derecha).

---

## Cifras Significativas

Las **cifras significativas** (CS) son todos los dígitos en una medida que se conocen con certeza, más un último dígito que es estimado (incierto). Indican la **precisión** del instrumento de medida.

### Reglas para contar Cifras Significativas

1. **Cualquier dígito distinto de cero es significativo.**
   - $123 \rightarrow$ 3 CS
   - $4.5 \rightarrow$ 2 CS
2. **Los ceros entre dígitos distintos de cero son significativos.**
   - $1005 \rightarrow$ 4 CS
   - $3.02 \rightarrow$ 3 CS
3. **Los ceros a la izquierda del primer dígito distinto de cero NO son significativos.** Solo indican la posición del decimal.
   - $0.002 \rightarrow$ 1 CS
   - $0.054 \rightarrow$ 2 CS
4. **Los ceros a la derecha de un número decimal SÍ son significativos.** Indican la precisión de la medida.
   - $2.0 \rightarrow$ 2 CS
   - $0.0400 \rightarrow$ 3 CS
5. **Para números enteros que terminan en cero (sin coma decimal), los ceros pueden o no ser significativos.** Para evitar ambigüedades, se **debe** usar la notación científica.
   - $1500$ puede tener 2, 3 o 4 CS.
   - $1.5 \times 10^3 \rightarrow$ 2 CS.
   - $1.50 \times 10^3 \rightarrow$ 3 CS.

### Operaciones Matemáticas con Cifras Significativas

> [!IMPORTANT] Reglas de Oro en Cálculos
> El resultado de un cálculo no puede ser más preciso que la medida menos precisa utilizada en el cálculo.

#### 1. Multiplicación y División
El resultado debe tener la **misma cantidad de cifras significativas** que el dato original que tenga **menor cantidad de cifras significativas**.

- Ejemplo: $4.56 \text{ (3 CS)} \times 1.4 \text{ (2 CS)} = 6.384 \xrightarrow{\text{Redondeo}} 6.4 \text{ (2 CS)}$

#### 2. Suma y Resta
El resultado debe tener la **misma cantidad de decimales** que el dato original que tenga **menor cantidad de cifras decimales**.

- Ejemplo: $12.11 \text{ (2 decimales)} + 18.0 \text{ (1 decimal)} + 1.013 \text{ (3 decimales)} = 31.123 \xrightarrow{\text{Redondeo}} 31.1 \text{ (1 decimal)}$

---

## Ver también

- [[Unidades de Medida]]
- [[Errores de Medición]]
- [[El Mol y Numero de Avogadro]]
- [[Quimica General]]