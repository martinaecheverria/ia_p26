---
title: "Algoritmos y Máquinas de Turing"
---

# Algoritmos y Máquinas de Turing

El modelo fundamental de computación que define qué es computable.

## ¿Qué es un Algoritmo?

Intuitivamente, un **algoritmo** es:

> Un procedimiento paso a paso, preciso y finito, que toma una entrada y produce una salida.

**Características informales:**
1. **Finito** — Descripción de longitud finita (no infinitas instrucciones)
2. **Determinista** — Cada paso está completamente especificado
3. **Efectivo** — Cada paso es ejecutable mecánicamente
4. **Entrada/Salida** — Toma datos y produce resultados
5. **Eventualmente termina** — ...o al menos, eso esperamos

**Ejemplos cotidianos:**
- Receta de cocina
- Instrucciones de IKEA
- Algoritmo de ordenamiento
- Tu rutina matutina

**Problema:** Esta definición es vaga. ¿Cómo la hacemos **precisa**?

**Solución:** Alan Turing (1936) propuso un modelo matemático formal.

---

## La Máquina de Turing

Una **Máquina de Turing** (MT) es el modelo matemático más simple de computación que captura todo lo que podemos hacer con una computadora.

### Intuición: Una Computadora Minimalista

Imagina:
- Una **cinta infinita** dividida en celdas (la memoria)
- Un **cabezal lector/escritor** que puede moverse izquierda/derecha
- Un **conjunto finito de estados** (como un programa con un contador de programa)
- Una **función de transición** (las instrucciones del programa)

```
                 ┌─────┐
                 │  q₁ │  ← Estado actual
                 └──┬──┘
                    ↓
    ... □ │ a │ b │ b │ a │ □ │ □ ...
                    ↑
                 cabezal
```

En cada paso:
1. Lee el símbolo bajo el cabezal
2. Basándose en el estado actual y el símbolo leído:
   - Escribe un nuevo símbolo (o el mismo)
   - Mueve el cabezal izquierda (L) o derecha (R)
   - Cambia a un nuevo estado

**¡Eso es todo!** Y con esto podemos simular cualquier computadora.

---

### Definición Formal

Una **Máquina de Turing** es una 7-tupla:

$$M = (Q, \Sigma, \Gamma, \delta, q_0, q_{accept}, q_{reject})$$

Donde:

| Componente | Descripción |
|------------|-------------|
| $Q$ | Conjunto **finito** de estados |
| $\Sigma$ | Alfabeto de **entrada** (no incluye el símbolo blanco) |
| $\Gamma$ | Alfabeto de la **cinta** ($\Sigma \subseteq \Gamma$, incluye el símbolo blanco $\sqcup$) |
| $\delta$ | **Función de transición**: $Q \times \Gamma \to Q \times \Gamma \times \{L, R\}$ |
| $q_0$ | Estado **inicial** ($q_0 \in Q$) |
| $q_{accept}$ | Estado de **aceptación** |
| $q_{reject}$ | Estado de **rechazo** ($q_{reject} \neq q_{accept}$) |

**Función de transición:**

$$\delta(q, a) = (q', b, D)$$

Se lee: "Estando en estado $q$ y leyendo símbolo $a$, escribo $b$, me muevo en dirección $D$ (L o R), y cambio al estado $q'$."

**Estados especiales:**
- $q_{accept}$ y $q_{reject}$ son estados de **parada** (halting states)
- Una vez que la MT entra a uno, se detiene inmediatamente

---

## Ejemplo 1: Reconocer el Lenguaje $\{0^n1^n \mid n \geq 0\}$

**Problema:** Aceptar cadenas como "", "01", "0011", "000111" y rechazar "0", "1", "00", "011", "0110".

**Estrategia:**
1. Marcar un 0 con X
2. Ir a la derecha y marcar el primer 1 sin marcar con Y
3. Regresar al inicio
4. Repetir hasta que no queden 0s
5. Verificar que no queden 1s sin marcar

**Descripción de estados:**

```
Estado q₀ (buscar siguiente 0 sin marcar):
  - Si veo 0 → escribo X, muevo R, voy a q₁
  - Si veo X → muevo R, quedo en q₀ (saltar Xs ya marcadas)
  - Si veo Y → voy a q₃ (ya no hay 0s, verificar que no sobren 1s)
  - Si veo □ → acepto (cadena vacía, o todos emparejados)
  
Estado q₁ (buscar el 1 correspondiente):
  - Si veo 0 → muevo R, quedo en q₁ (saltar 0s)
  - Si veo Y → muevo R, quedo en q₁ (saltar 1s ya marcados)
  - Si veo 1 → escribo Y, muevo L, voy a q₂ (¡encontré el 1!)
  - Si veo □ → rechazo (más 0s que 1s)
  
Estado q₂ (regresar al inicio para buscar otro 0):
  - Si veo 0, X, Y → muevo L, quedo en q₂
  - Si veo □ → muevo R, voy a q₀ (llegué al inicio)
  
Estado q₃ (verificar que solo quedan Ys y blancos):
  - Si veo Y → muevo R, quedo en q₃ (bien, sigo verificando)
  - Si veo □ → acepto (no sobran 1s sin marcar)
  - Si veo 1 → rechazo (más 1s que 0s)
```

**Ejecución detallada en "0011":**

Usamos la notación: `Estado | Cinta con [cabezal]`

```
Paso  0: q₀ | [0] 0  1  1  □     Lee 0 → Escribe X, mueve R, va a q₁
Paso  1: q₁ |  X [0] 1  1  □     Lee 0 → Mueve R (saltando 0)
Paso  2: q₁ |  X  0 [1] 1  □     Lee 1 → Escribe Y, mueve L, va a q₂
Paso  3: q₂ |  X [0] Y  1  □     Lee 0 → Mueve L (regresando)
Paso  4: q₂ | [X] 0  Y  1  □     Lee X → Mueve L (regresando)
Paso  5: q₂ |[□] X  0  Y  1      Lee □ → Mueve R, va a q₀ (inicio!)
Paso  6: q₀ | □ [X] 0  Y  1      Lee X → Mueve R (saltar X)
Paso  7: q₀ | □  X [0] Y  1      Lee 0 → Escribe X, mueve R, va a q₁
Paso  8: q₁ | □  X  X [Y] 1      Lee Y → Mueve R (saltar Y marcada)
Paso  9: q₁ | □  X  X  Y [1]     Lee 1 → Escribe Y, mueve L, va a q₂
Paso 10: q₂ | □  X  X [Y] Y      Lee Y → Mueve L (regresando)
Paso 11: q₂ | □  X [X] Y  Y      Lee X → Mueve L (regresando)
Paso 12: q₂ | □ [X] X  Y  Y      Lee X → Mueve L (regresando)
Paso 13: q₂ |[□] X  X  Y  Y      Lee □ → Mueve R, va a q₀ (inicio!)
Paso 14: q₀ | □ [X] X  Y  Y      Lee X → Mueve R (saltar)
Paso 15: q₀ | □  X [X] Y  Y      Lee X → Mueve R (saltar)
Paso 16: q₀ | □  X  X [Y] Y      Lee Y → Va a q₃ (verificar!)
Paso 17: q₃ | □  X  X  Y [Y]     Lee Y → Mueve R (bien)
Paso 18: q₃ | □  X  X  Y  Y [□]  Lee □ → ¡ACEPTA! ✓
```

**Conclusión:** "0011" tiene 2 ceros y 2 unos, correctamente emparejados.

---

**Ejecución en cadena inválida "011":**

```
Paso 0: q₀ | [0] 1  1  □     Lee 0 → Escribe X, mueve R, va a q₁
Paso 1: q₁ |  X [1] 1  □     Lee 1 → Escribe Y, mueve L, va a q₂
Paso 2: q₂ | [X] Y  1  □     Lee X → Mueve L
Paso 3: q₂ |[□] X  Y  1      Lee □ → Mueve R, va a q₀
Paso 4: q₀ | □ [X] Y  1      Lee X → Mueve R
Paso 5: q₀ | □  X [Y] 1      Lee Y → Va a q₃ (verificar)
Paso 6: q₃ | □  X  Y [1]     Lee 1 → ¡RECHAZA! ✗
```

**Conclusión:** "011" tiene más 1s que 0s, es rechazada.

---

**Ejecución en cadena inválida "001":**

```
Paso 0: q₀ | [0] 0  1  □     Lee 0 → Escribe X, mueve R, va a q₁
Paso 1: q₁ |  X [0] 1  □     Lee 0 → Mueve R
Paso 2: q₁ |  X  0 [1] □     Lee 1 → Escribe Y, mueve L, va a q₂
Paso 3: q₂ |  X [0] Y  □     Lee 0 → Mueve L
Paso 4: q₂ | [X] 0  Y  □     Lee X → Mueve L
Paso 5: q₂ |[□] X  0  Y      Lee □ → Mueve R, va a q₀
Paso 6: q₀ | □ [X] 0  Y      Lee X → Mueve R
Paso 7: q₀ | □  X [0] Y      Lee 0 → Escribe X, mueve R, va a q₁
Paso 8: q₁ | □  X  X [Y]     Lee Y → Mueve R
Paso 9: q₁ | □  X  X  Y [□]  Lee □ → ¡RECHAZA! ✗ (no hay más 1s)
```

**Conclusión:** "001" tiene más 0s que 1s, es rechazada.

---

## Ejemplo 2: Sumar 1 a un Número Binario

**Problema:** Dado un número binario, sumarle 1.
- Entrada "1011" (11 en decimal) → Salida "1100" (12 en decimal)
- Entrada "111" (7 en decimal) → Salida "1000" (8 en decimal)

**Estrategia:** Ir al final del número, luego sumar 1 propagando el acarreo hacia la izquierda.

**Descripción de estados:**

```
Estado q₀ (ir al final del número):
  - Si veo 0 o 1 → muevo R, quedo en q₀
  - Si veo □ → muevo L, voy a q₁ (encontré el final)

Estado q₁ (sumar con acarreo):
  - Si veo 1 → escribo 0, muevo L, quedo en q₁ (1+1=10, acarreo continúa)
  - Si veo 0 → escribo 1, voy a q_accept (0+1=1, sin acarreo, terminamos)
  - Si veo □ → escribo 1, voy a q_accept (acarreo al inicio, añadir dígito)
```

**Ejecución detallada en "1011" (11₁₀):**

```
Paso 0: q₀ | [1] 0  1  1  □     Lee 1 → Mueve R
Paso 1: q₀ |  1 [0] 1  1  □     Lee 0 → Mueve R
Paso 2: q₀ |  1  0 [1] 1  □     Lee 1 → Mueve R
Paso 3: q₀ |  1  0  1 [1] □     Lee 1 → Mueve R
Paso 4: q₀ |  1  0  1  1 [□]    Lee □ → Mueve L, va a q₁ (¡encontró el final!)
Paso 5: q₁ |  1  0  1 [1] □     Lee 1 → Escribe 0, mueve L (acarreo: 1+1=10)
Paso 6: q₁ |  1  0 [1] 0  □     Lee 1 → Escribe 0, mueve L (acarreo: 1+1=10)
Paso 7: q₁ |  1 [0] 0  0  □     Lee 0 → Escribe 1, va a q_accept (sin acarreo: 0+1=1)

Resultado final: 1100 ✓ (que es 12 en decimal)
```

**Verificación:** 1011₂ = 8+2+1 = 11₁₀, y 1100₂ = 8+4 = 12₁₀ ✓

---

**Ejecución en "111" (7₁₀) — caso con overflow:**

```
Paso 0: q₀ | [1] 1  1  □     Lee 1 → Mueve R
Paso 1: q₀ |  1 [1] 1  □     Lee 1 → Mueve R
Paso 2: q₀ |  1  1 [1] □     Lee 1 → Mueve R
Paso 3: q₀ |  1  1  1 [□]    Lee □ → Mueve L, va a q₁
Paso 4: q₁ |  1  1 [1] □     Lee 1 → Escribe 0, mueve L (acarreo)
Paso 5: q₁ |  1 [1] 0  □     Lee 1 → Escribe 0, mueve L (acarreo)
Paso 6: q₁ | [1] 0  0  □     Lee 1 → Escribe 0, mueve L (acarreo)
Paso 7: q₁ |[□] 0  0  0      Lee □ → Escribe 1, va a q_accept (añadir dígito!)

Resultado final: 1000 ✓ (que es 8 en decimal)
```

**Verificación:** 111₂ = 4+2+1 = 7₁₀, y 1000₂ = 8₁₀ ✓

---

**Ejecución en "1010" (10₁₀) — caso sin propagación:**

```
Paso 0: q₀ | [1] 0  1  0  □     Lee 1 → Mueve R
Paso 1: q₀ |  1 [0] 1  0  □     Lee 0 → Mueve R
Paso 2: q₀ |  1  0 [1] 0  □     Lee 1 → Mueve R
Paso 3: q₀ |  1  0  1 [0] □     Lee 0 → Mueve R
Paso 4: q₀ |  1  0  1  0 [□]    Lee □ → Mueve L, va a q₁
Paso 5: q₁ |  1  0  1 [0] □     Lee 0 → Escribe 1, va a q_accept (¡sin acarreo!)

Resultado final: 1011 ✓ (que es 11 en decimal)
```

**Observación:** Si el último dígito es 0, la suma termina inmediatamente sin propagación.

---

## Ejemplo 3: Copiar una Cadena

**Problema:** Entrada "ab" → Salida "ab#ab" (copiar la cadena después de un separador)

**Estrategia:** 
1. Primero, ir al final y escribir el separador #
2. Regresar al inicio
3. Para cada símbolo: marcarlo, ir al final y escribir una copia, regresar
4. Repetir hasta que todos estén marcados

**Descripción de estados (simplificada):**

```
Estado q₀ (ir al final para poner #):
  - Si veo a, b → muevo R, quedo en q₀
  - Si veo □ → escribo #, muevo L, voy a q₁

Estado q₁ (regresar al inicio):
  - Si veo a, b, # → muevo L, quedo en q₁
  - Si veo □ → muevo R, voy a q₂

Estado q₂ (buscar siguiente símbolo sin marcar):
  - Si veo A, B → muevo R, quedo en q₂ (saltar marcados)
  - Si veo a → escribo A (marcar), voy a q₃ₐ (recordamos 'a')
  - Si veo b → escribo B (marcar), voy a q₃ᵦ (recordamos 'b')
  - Si veo # → voy a q_accept (¡terminamos!)

Estado q₃ₐ (ir al final y escribir 'a'):
  - Si veo cualquier símbolo excepto □ → muevo R, quedo en q₃ₐ
  - Si veo □ → escribo a, muevo L, voy a q₄

Estado q₃ᵦ (ir al final y escribir 'b'):
  - Si veo cualquier símbolo excepto □ → muevo R, quedo en q₃ᵦ
  - Si veo □ → escribo b, muevo L, voy a q₄

Estado q₄ (regresar al inicio para buscar siguiente):
  - Si veo cualquier símbolo excepto □ → muevo L, quedo en q₄
  - Si veo □ → muevo R, voy a q₂
```

**Ejecución simplificada en "ab":**

```
Fase 1: Poner separador
  q₀ | [a] b  □   →  q₀ |  a [b] □   →  q₀ |  a  b [□]  →  q₁ |  a [b] #
  (ahora regresamos al inicio)
  q₁ | [a] b  #   →  q₂ | [a] b  #

Fase 2: Copiar 'a'
  q₂ | [a] b  #        Lee a → Escribe A (marcar), va a q₃ₐ
  q₃ₐ| [A] b  #  □     Ir al final...
  q₃ₐ|  A  b  # [□]    Lee □ → Escribe a, va a q₄
  (cinta: A b # a)
  q₄ regresar al inicio...
  q₂ | [A] b  #  a     Lee A → Mueve R (saltar marcado)
  q₂ |  A [b] #  a     Lee b → Escribe B (marcar), va a q₃ᵦ

Fase 3: Copiar 'b'
  q₃ᵦ|  A [B] #  a  □   Ir al final...
  q₃ᵦ|  A  B  #  a [□]  Lee □ → Escribe b, va a q₄
  (cinta: A B # a b)
  q₄ regresar al inicio...
  q₂ | [A] B  #  a  b   Lee A → Mueve R
  q₂ |  A [B] #  a  b   Lee B → Mueve R
  q₂ |  A  B [#] a  b   Lee # → ¡ACEPTA!

Resultado: AB#ab (las A,B son versiones "marcadas" de a,b)
```

**Nota:** En la salida final, podríamos añadir una fase de "limpieza" que convierta A→a y B→b para obtener exactamente "ab#ab". La MT necesita estados adicionales para recordar qué símbolo está copiando — esto ilustra cómo los estados funcionan como "memoria" temporal.

---

## ¿Por Qué las Máquinas de Turing?

### Ventajas del Modelo

1. **Simplicidad matemática** — Fácil de analizar formalmente
2. **Poder expresivo** — Puede simular cualquier computadora
3. **Modelo de referencia** — Para comparar otros modelos

### Tesis de Church-Turing

> **Informal:** Todo lo que intuitivamente llamamos "computable" puede ser computado por una Máquina de Turing.

Esta NO es una afirmación matemática demostrable — es una **tesis** sobre la naturaleza de la computación.

**Evidencia a favor:**
- Todos los modelos propuestos (λ-calculus, funciones recursivas, Python, Java, ...) son equivalentes a MTs
- Nadie ha encontrado un modelo más poderoso que sea "razonable"
- 90 años sin contraejemplos

**Implicación:** Si no puede hacerse en una MT, no es computable por ninguna computadora.

---

## Variantes de Máquinas de Turing

Existen muchas variantes de MTs, pero todas tienen el **mismo poder expresivo**:

### MT con Múltiples Cintas

En lugar de una cinta, tiene $k$ cintas independientes.

**¿Más poderosa?** NO — se puede simular con una cinta (multiplexando)

**¿Más rápida?** SÍ — puede ser cuadráticamente más rápida

---

### MT No Determinista

En cada paso, puede tener **múltiples** transiciones posibles (como "adivinar").

$$\delta: Q \times \Gamma \to \mathcal{P}(Q \times \Gamma \times \{L, R\})$$

Acepta si **existe** alguna secuencia de elecciones que lleva a $q_{accept}$.

**¿Más poderosa?** NO — una MT determinista puede simularla (explorando todas las ramas)

**¿Más rápida?** ¡Quizás! Este es el problema **P vs NP**

---

### MT Enumeradora

En lugar de aceptar/rechazar, **imprime** strings en una cinta de salida.

**Uso:** Enumerar todos los strings de un lenguaje.

**Equivalencia:** Un lenguaje es reconocible ↔ tiene un enumerador

---

### Todas son Equivalentes

**Teorema:** MT estándar ≡ MT multi-cinta ≡ MT no determinista ≡ Enumeradores ≡ ...

Este teorema justifica usar la definición más simple (MT estándar) para estudiar computabilidad.

---

## Lenguajes y MTs

Una MT define un **lenguaje**:

$$L(M) = \{w \in \Sigma^* \mid M \text{ acepta } w\}$$

### Tres Posibles Resultados

Dada una entrada $w$, una MT puede:

1. **Aceptar** — Llega a $q_{accept}$
2. **Rechazar** — Llega a $q_{reject}$
3. **Loop** — Nunca para (se ejecuta infinitamente)

Esta tercera opción es crucial para entender los límites de la computación.

---

## ¿Tu Laptop es una Máquina de Turing?

**Casi.**

Diferencias:
- Tu laptop tiene **memoria finita** (RAM limitada)
- Una MT tiene **cinta infinita**

Formalmente:
- Tu laptop es un **autómata finito gigante** (con $2^{10^{10}}$ estados o más)
- Una MT es más poderosa en teoría

**En la práctica:** Para la mayoría de los propósitos, podemos pensar en nuestras computadoras como MTs (ignorando límites de memoria).

---

## Computación Universal

**Pregunta:** ¿Puede una MT simular otra MT?

**Respuesta:** ¡SÍ! Existe una **Máquina de Turing Universal** $U$.

$$U(\langle M \rangle, w) = M(w)$$

Donde $\langle M \rangle$ es una codificación (string) de la MT $M$.

**Implicación:** Esto es como un **intérprete** — $U$ lee la "descripción" de otra MT y la ejecuta.

**Analogía moderna:** Tu computadora puede ejecutar cualquier programa — es una "máquina universal".

**Conexión con Halting:** Esta universalidad es clave para demostrar que el Halting Problem es indecidible.

---

## Resumen

| Concepto | Descripción |
|----------|-------------|
| **Algoritmo** | Procedimiento finito, determinista, efectivo |
| **Máquina de Turing** | Modelo formal de computación (cinta + estados + transiciones) |
| **Aceptar/Rechazar/Loop** | Tres resultados posibles de una MT |
| **Church-Turing Thesis** | MTs capturan todo lo "computable" |
| **Variantes** | Multi-cinta, no determinista, etc. — todas equivalentes |
| **MT Universal** | Una MT puede simular cualquier otra MT |

**Punto clave:** Las Máquinas de Turing son el modelo fundamental que usaremos para estudiar qué es computable y qué no.

---

**Siguiente:** [Computabilidad vs Decidibilidad →](03_computabilidad_decidibilidad.md)
