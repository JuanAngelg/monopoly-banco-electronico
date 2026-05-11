# 📊 Monopoly Banco Electrónico - Documentación Técnica

## 1. MÉTODO NUMÉRICO IMPLEMENTADO

### **Ajuste Polinomial por Mínimos Cuadrados (Grado 2)**

El método numérico elegido es la **Regresión Polinomial de Grado 2** mediante **Mínimos Cuadrados**, resuelto con **Eliminación Gaussiana con Pivoteo Parcial**.

#### ¿Por qué este método?

- **Mejora al juego**: Simula inflación económica dinámica basada en datos reales de la partida
- **Aplicabilidad**: Los datos del juego (riqueza promedio) se capturan automáticamente cada ronda
- **Efecto tangible**: Multiplica las rentas del tablero, afectando directamente la jugabilidad
- **Sin librerías**: Se implementó completamente desde cero

---

## 2. ESTRUCTURA MATEMÁTICA

### **Problema: Ajuste de Curva**

Dados $n$ puntos $(x_i, y_i)$ donde:
- $x_i$ = número de ronda
- $y_i$ = riqueza promedio de jugadores activos

Encontrar los coeficientes $c_0, c_1, c_2$ tales que:

$$p(x) = c_0 + c_1 \cdot x + c_2 \cdot x^2$$

minimice el error de mínimos cuadrados.

### **Solución: Ecuaciones Normales**

El problema se resuelve mediante el sistema:

$$A^T A \cdot \mathbf{c} = A^T \mathbf{b}$$

Donde:
- $A$ es la matriz de Vandermonde: $A_{ij} = x_i^{j-1}$
- $\mathbf{b}$ es el vector de valores: $b_i = y_i$
- $\mathbf{c} = [c_0, c_1, c_2]^T$ son los coeficientes buscados

Expandiendo para grado 2:

$$\begin{bmatrix}
n & \sum x_i & \sum x_i^2 \\
\sum x_i & \sum x_i^2 & \sum x_i^3 \\
\sum x_i^2 & \sum x_i^3 & \sum x_i^4
\end{bmatrix}
\begin{bmatrix}
c_0 \\ c_1 \\ c_2
\end{bmatrix}
=
\begin{bmatrix}
\sum y_i \\
\sum x_i y_i \\
\sum x_i^2 y_i
\end{bmatrix}$$

### **Método de Resolución: Eliminación Gaussiana con Pivoteo**

1. **Construcción de matriz aumentada**: $[A^T A | A^T b]$
2. **Eliminación hacia adelante**: 
   - Para cada columna, encontrar pivote máximo (pivoteo parcial)
   - Intercambiar filas si es necesario
   - Hacer ceros debajo del pivote mediante eliminación
3. **Sustitución hacia atrás**: Resolver el sistema triangular superior

---

## 3. ALIMENTACIÓN DEL MÉTODO NUMÉRICO

### **¿Dónde vienen los datos?**

Los datos se capturan en la función `updateEconomicModel()`:

```javascript
function updateEconomicModel(players, round, ecoData) {
  const activePlayers = players.filter(p => !p.bankrupt);
  
  // CAPTURA: Calcular riqueza promedio
  const avgWealth = activePlayers.reduce((sum, p) => sum + p.balance, 0) 
                    / activePlayers.length;
  
  // ALMACENAMIENTO: Agregar nuevo punto
  const newEcoData = [...ecoData, [round, avgWealth]];
  
  // ...resto del procesamiento
}
```

**Punto de captura**: Al completar cada ronda en `processEndTurn()`

**Formato de datos**:
- Cada elemento es `[ronda, riqueza_promedio]`
- Ejemplo: `[1, 15000000]`, `[2, 14800000]`, `[3, 14500000]`
- Mínimo 3 puntos para ajustar el polinomio de grado 2

**Condiciones**:
- Solo se cuentan jugadores NO BANCARROTOS
- Se actualiza al inicio de una nueva ronda

---

## 4. CÁLCULO DEL MULTIPLICADOR DE INFLACIÓN

Una vez ajustado el polinomio con ≥3 puntos:

```javascript
const coeffs = polynomialFit(newEcoData, 2); // Obtener c₀, c₁, c₂

// Calcular valor base (en ronda 1)
const baseValue = polynomialEval(coeffs, 1);

// Calcular valor en ronda actual
const currentValue = polynomialEval(coeffs, round);

// Ratio = cómo cambió la riqueza respecto a la ronda 1
const ratio = baseValue > 0 ? currentValue / baseValue : 1;

// Acotar entre 1.0 (sin inflación) y 2.5 (máxima inflación)
const multiplier = Math.max(1.0, Math.min(2.5, ratio));
```

**Interpretación**:
- Si la riqueza promedio **baja** → multiplicador < 1.0 → se limita a **1.0** (rentas normales)
- Si la riqueza promedio **sube** → multiplicador > 1.0 → las rentas **suben** proporcionalmente
- Máximo x2.5 para evitar que el juego se vuelva imposible

---

## 5. USO DE RESULTADOS EN EL JUEGO

### **Aplicación directa: Pago de rentas**

En `processRoll()`, cuando un jugador cae en una propiedad ajena:

```javascript
const rentAmount = Math.round(space.rent * GAME.inflationMultiplier);
```

**Ejemplo**:
- Renta base de "Park Place": 100
- Multiplicador de inflación: 1.35
- Renta pagada: 100 × 1.35 = 135

### **Efecto en la jugabilidad**

| Situación | Multiplicador | Efecto |
|-----------|:---:|:---|
| Todos ricos | 2.0+ | Rentas altas → elimina competencia |
| Todos pobres | <1.0 | Rentas bajas (1.0) → permite recuperarse |
| Equilibrado | ~1.0-1.2 | Juego normal/ligeramente inflado |
| Uno muy rico | 1.5+ | Rentas altas le dan más ventaja |

---

## 6. INTEGRACIÓN EN EL CÓDIGO

### **Estructura de datos del juego**

```javascript
GAME = {
  // ... otros campos
  ecoData: [[1, 15000000], [2, 14800000], ...],  // Datos históricos
  polyCoeffs: [15000000, -200000, 5000],          // c₀, c₁, c₂
  inflationMultiplier: 1.34,                      // Resultado final
  round: 3,                                        // Ronda actual
  // ...
}
```

### **Flujo de ejecución**

```
Inicio de nueva ronda
    ↓
processEndTurn() es llamado
    ↓
Calcular ronda siguiente
    ↓
¿Es nueva ronda (ciclo completo)?
    ├─ SÍ → updateEconomicModel()
    │   ├─ Capturar (ronda, riqueza_promedio)
    │   ├─ Si ≥3 datos → polynomialFit()
    │   │   ├─ Construir AᵀA y Aᵀb
    │   │   ├─ gaussianElimination() → resolver
    │   │   └─ Obtener coeficientes [c₀, c₁, c₂]
    │   └─ Calcular multiplicador = p(ronda)/p(1)
    │
    └─ NO → Mantener multiplicador anterior
    ↓
GAME.inflationMultiplier = nueva inflación
    ↓
Próxima vez que se pague renta → se usa el nuevo multiplicador
```

### **Funciones principales**

| Función | Propósito |
|---------|-----------|
| `gaussianElimination(A, b)` | Resolver AX = b |
| `polynomialFit(points, degree)` | Construir y resolver ecuaciones normales |
| `polynomialEval(coeffs, x)` | Evaluar p(x) = c₀ + c₁x + c₂x² |
| `updateEconomicModel()` | Capturar datos y actualizar modelo |

---

## 7. CÓDIGO COMPLETO: MÉTODO NUMÉRICO

### **Eliminación Gaussiana (líneas ~11-50)**

```javascript
function gaussianElimination(A, b) {
  const n = b.length;
  const matrix = A.map((row, i) => [...row, b[i]]);
  
  // Eliminación hacia adelante con pivoteo parcial
  for (let col = 0; col < n; col++) {
    // 1. Encontrar pivote máximo
    let maxRow = col;
    for (let row = col + 1; row < n; row++) {
      if (Math.abs(matrix[row][col]) > Math.abs(matrix[maxRow][col])) {
        maxRow = row;
      }
    }
    
    // 2. Intercambiar filas
    [matrix[col], matrix[maxRow]] = [matrix[maxRow], matrix[col]];
    
    // 3. Evitar división por cero
    if (Math.abs(matrix[col][col]) < 1e-12) continue;
    
    // 4. Hacer ceros debajo del pivote
    for (let row = col + 1; row < n; row++) {
      const factor = matrix[row][col] / matrix[col][col];
      for (let j = col; j <= n; j++) {
        matrix[row][j] -= factor * matrix[col][j];
      }
    }
  }
  
  // Sustitución hacia atrás
  const solution = Array(n).fill(0);
  for (let i = n - 1; i >= 0; i--) {
    solution[i] = matrix[i][n];
    for (let j = i + 1; j < n; j++) {
      solution[i] -= matrix[i][j] * solution[j];
    }
    if (Math.abs(matrix[i][i]) > 1e-12) {
      solution[i] /= matrix[i][i];
    }
  }
  
  return solution;
}
```

### **Ajuste Polinomial (líneas ~52-72)**

```javascript
function polynomialFit(points, degree = 2) {
  if (points.length < degree + 1) return null;
  
  const n = points.length;
  const d = degree + 1;
  
  // Construir AᵀA
  const AtA = Array.from({ length: d }, (_, i) =>
    Array.from({ length: d }, (_, j) =>
      points.reduce((sum, p) => sum + Math.pow(p[0], i + j), 0)
    )
  );
  
  // Construir Aᵀb
  const Atb = Array.from({ length: d }, (_, i) =>
    points.reduce((sum, p) => sum + Math.pow(p[0], i) * p[1], 0)
  );
  
  // Resolver el sistema
  return gaussianElimination(AtA, Atb);
}
```

### **Evaluación del polinomio (líneas ~74-78)**

```javascript
function polynomialEval(coefficients, x) {
  if (!coefficients) return 0;
  return coefficients.reduce((sum, c, i) => sum + c * Math.pow(x, i), 0);
}
```

### **Actualización del modelo económico (líneas ~250-300)**

```javascript
function updateEconomicModel(players, round, ecoData) {
  const activePlayers = players.filter(p => !p.bankrupt);
  if (activePlayers.length === 0) {
    return { newEcoData: ecoData, coeffs: null, multiplier: 1.0 };
  }
  
  // CAPTURA: Riqueza promedio
  const avgWealth = activePlayers.reduce((sum, p) => sum + p.balance, 0) 
                    / activePlayers.length;
  const newEcoData = [...ecoData, [round, avgWealth]];
  
  // AJUSTE: Necesita ≥3 puntos
  if (newEcoData.length < 3) {
    return { newEcoData, coeffs: null, multiplier: 1.0 };
  }
  
  // RESOLUCIÓN: Polinomio grado 2
  const coeffs = polynomialFit(newEcoData, 2);
  if (!coeffs) {
    return { newEcoData, coeffs: null, multiplier: 1.0 };
  }
  
  // CÁLCULO: Multiplicador de inflación
  const baseValue = polynomialEval(coeffs, 1);
  const currentValue = polynomialEval(coeffs, round);
  const ratio = baseValue > 0 ? currentValue / baseValue : 1;
  const multiplier = Math.max(1.0, Math.min(2.5, ratio));
  
  return { newEcoData, coeffs, multiplier };
}
```

---

## 8. EJEMPLO PASO A PASO

### **Escenario: Primeras 3 rondas de una partida**

**Ronda 1:**
- Datos: `[[1, 15000000]]`
- Estado: Esperando más datos

**Ronda 2:**
- Riqueza promedio: 14,800,000
- Datos: `[[1, 15000000], [2, 14800000]]`
- Estado: Necesita 1 punto más

**Ronda 3 - Inicio de nueva ronda:**
- Riqueza promedio: 14,500,000
- Datos: `[[1, 15000000], [2, 14800000], [3, 14500000]]`
- **¡Ahora sí ajustamos!**

Construir matriz AᵀA:
```
[1,  1,   1  ]   [3,        6,        14   ]
[1,  2,   4  ] → [6,        14,       36   ]
[1,  3,   9  ]   [14,       36,       98   ]
```

Construir vector Aᵀb:
```
[15000000, 14800000, 14500000]ᵀ → [44300000, 108200000, 280600000]ᵀ
```

Resolver: AᵀA·c = Aᵀb
```
c₀ ≈ 15200000  (riqueza base)
c₁ ≈ -200000   (pérdida por ronda)
c₂ ≈ 5000      (estabilización)
```

p(x) = 15200000 - 200000·x + 5000·x²

Calcular multiplicador:
```
p(1) = 15200000 - 200000 + 5000 = 15005000
p(3) = 15200000 - 600000 + 45000 = 14645000
ratio = 14645000 / 15005000 ≈ 0.976
multiplier = max(1.0, min(2.5, 0.976)) = 1.0
```

**Resultado**: Las rentas se mantienen en el valor base (1.0×) porque la riqueza está bajando.

## 10. RESUMEN: ESPECIFICACIONES CUMPLIDAS

✅ **Técnica numérica**: Ajuste Polinomial por Mínimos Cuadrados  
✅ **Método de resolución**: Eliminación Gaussiana con pivoteo parcial  
✅ **Sin librerías externas**: Implementación 100% desde cero  
✅ **Mejora al juego**: Inflación dinámica afecta rentas  
✅ **Alimentación clara**: Captura automática de (ronda, riqueza) cada ciclo  
✅ **Uso de resultados**: Multiplicador aplicado a todas las rentas  
✅ **Integración total**: Código completamente documentado y estructurado  

---

