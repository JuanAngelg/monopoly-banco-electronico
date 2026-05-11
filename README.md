# 🏦 Monopoly Banco Electrónico - Guía de Uso

## 📋 Contenido del Proyecto

```
App_Monopoly/
├── monopoly_banco_electronico.html      ← Aplicación principal
├── README.md                            ← Este archivo 
└── DOCUMENTACION_TECNICA.md             ← Detalles matemáticos del método numérico
```

---

## ⚡ Inicio Rápido

### Opción 1: Abrir directamente en el navegador
```bash
1. Navegar a la carpeta c:\Users\bueno\App_Monopoly\
2. Hacer doble clic en monopoly_banco_electronico.html
3. ¡El juego se abrirá en tu navegador predeterminado!
```

### Opción 2: Desde navegador (recomendado)
```bash
1. Abrir Chrome, Firefox, Edge o Safari
2. Presionar Ctrl+O (Cmd+O en Mac)
3. Navegar a: c:\Users\bueno\App_Monopoly\monopoly_banco_electronico.html
4. Seleccionar y abrir
```

---

## 🎮 Cómo Jugar

### **Pantalla de Setup**

1. **Seleccionar número de jugadores** (2, 3 o 4)
   - Cada jugador comienza con $15M
   - Los jugadores disponibles se muestran debajo

2. **Leer método numérico** (opcional)
   - Se explica brevemente en el panel azul
   - Ver documentación técnica para detalles

3. **Hacer clic en "¡Iniciar Juego!"**

### **Durante la Partida**

#### **Panel Izquierdo: Tablero**
- Grid 10×10 con las 40 casillas del Monopoly
- Cada casilla muestra:
  - **Nombre** de la propiedad
  - **Precio** (si es vendible)
  - **Renta** (si hay inflación activa)
  - **Propietario** (línea de color en el borde)
  - **Tokens** de jugadores presentes
   - **Hipoteca** si la propiedad está financiada

#### **Panel Derecho: Control del Juego**

**Turno Actual:**
- Muestra al jugador cuyo turno es
- Saldo actual
- Posición en el tablero
- Si está en cárcel

**Dados:**
- Se muestran después de lanzar

**Acciones disponibles:**
- 🎲 **Lanzar Dados** → Avanzar en el tablero
- **Comprar** / **Pasar** → Cuando caes en propiedad sin dueño
- **Terminar Turno →** → Pasar al siguiente jugador
- **Usar carta de salida** → Si estás en cárcel y tienes una carta libre
- **Hipotecar / Levantar** → Desde el panel de propiedades propias

**Registro:**
- Última línea con los eventos más recientes
- Compras, rentas pagadas, sorpresas, etc.
- Subastas automáticas cuando se declina una compra

### **Ver Método Numérico**

Hacer clic en **"Ver método"** para desplegar:
- **Puntos capturados**: Cuántos datos tiene el sistema
- **Multiplicador**: Factor de inflación actual (×1.00 a ×2.50)
- **Ronda actual**: Número de ronda
- **Fórmula**: p(x) = c₀ + c₁·x + c₂·x² con los coeficientes reales
- **Gráfico**: Visualización de la curva ajustada vs datos reales
- **Explicación**: Cómo funciona la alimentación y uso

---

## 🎯 Reglas del Juego

### **Movimiento**
- Lanzas dos dados, avanzas sus valores
- Si pasas por SALIDA, cobras $200K
- Si sacas dobles (d1 = d2), obtienes otro turno
- Si sacas tres dobles seguidos, vas directo a la cárcel

### **Visuales Mejorados**
- **Badges de propiedades**: 
  - ⭐ Monopolio completado (todas del color)
  - 🔒 Hipotecada (no paga renta)
  - 🏢 Ferrocarriles (muestra cantidad)
  - ⚡ Servicios públicos (muestra cantidad)
- **Modal de tarjetas**: Al caer en tarjeta, se abre un modal mostrando el evento
- **Indicadores de saldo**: Color verde si subes dinero, rojo si baja
- **Promedio de riqueza**: Se muestra en el panel del método numérico
- **Multiplicador visible**: Tag de inflación con color según intensidad
  - Verde: ≤1.2× (baja inflación)
  - Amarillo: 1.2-1.5× (inflación moderada)
  - Rojo: >1.5× (alta inflación)
- Caes en propiedad sin dueño → Puedes comprarla
- Si no la compras, el banco la subasta automáticamente
- Caes en propiedad ajena → Pagas renta (afectada por inflación)
- Si una propiedad está hipotecada, no cobra renta
- Si completas el color de una serie, la renta de esas propiedades se duplica
- La renta se calcula como: `renta_base × multiplicador_inflación`
- Ferrocarriles y servicios públicos usan reglas más cercanas al Monopoly real

### **Tarjetas de Suerte**
- Puedes ganar dinero, perder dinero, o cambiar de posición
- Algunas te envían al ferrocarril más cercano o a la utilidad más cercana
- Algunas te dan carta de salida de cárcel
- Se aplican automáticamente

### **Impuestos**
- Casilla 4: Pagas $150K (sin inflación)
- Casilla 38: Pagas $100K (sin inflación)

### **Cárcel**
- Entras en la casilla 30 → Ir directo a la cárcel
- Estás atrapado 3 turnos a menos que saques dobles
- Puedes pagar $500K para salir inmediatamente

### **Quiebra**
- Si tu saldo baja de $0, quedas eliminado
- Último jugador activo = ¡Ganador!

---

## 📊 Entendiendo la Inflación

### **¿Qué es el multiplicador?**
Es un factor que **multiplica TODAS las rentas del tablero** basado en:
- Cuánta riqueza tienen en promedio los jugadores
- Cómo cambia esa riqueza a lo largo de las rondas
- Un polinomio de grado 2 que se ajusta a los datos

### **¿Cómo afecta al juego?**

| Multiplicador | Interpretación | Efecto |
|:---:|:---|:---|
| **1.00** | Sin inflación | Rentas normales |
| **1.20** | Inflación baja | Rentas suben 20% |
| **1.50** | Inflación media | Rentas suben 50% |
| **2.00+** | Inflación alta | Rentas suben 100%+ |
| **2.50** (máx) | Inflación máxima | Tope para evitar juego imposible |

### **Ejemplo práctico:**
- Renta base de "Park Place" = 100
- Con multiplicador 1.50:
  - Renta pagada = 100 × 1.50 = **150**
  - Diferencia = +50 dinero en disputa cada aterrizaje

---

## 🔬 Cómo Funciona el Método Numérico (Resumen)

### **Los 3 pasos principales**

#### **1️⃣ Captura de Datos**
Cada vez que termina una ronda, el juego registra:
- Número de ronda
- Riqueza promedio de jugadores no bancarrotos
- Se almacena como `[ronda, riqueza]`

#### **2️⃣ Ajuste del Polinomio** (con ≥3 puntos)
Se resuelve el problema de mínimos cuadrados:
- Matriz normal: AᵀA
- Vector: Aᵀb
- Método: Eliminación Gaussiana con pivoteo parcial
- Resultado: Coeficientes `[c₀, c₁, c₂]` de p(x) = c₀ + c₁x + c₂x²

#### **3️⃣ Cálculo del Multiplicador**
```
inflación = p(ronda_actual) / p(1)
multiplicador = clamp(inflación, 1.0, 2.5)
```

### **Visualización en la app**
- Gráfico SVG con línea azul = polinomio ajustado
- Puntos rojos = datos reales capturados
- Línea naranja = ronda actual
- Las etiquetas muestran riqueza vs tiempo

---

## 💡 Caso de Uso: ¿Cuándo cambia la inflación?

**Escenario:**
- Ronda 1-2: Jugadores tienen $15M cada uno
- Ronda 3: Algunos pierden dinero, promedio baja a $13M
- El sistema detecta la tendencia

**Resultado:**
- Si la riqueza sigue bajando → inflación baja (×1.0-1.2)
- Si la riqueza se recupera → inflación sube (×1.3-1.8)
- Se calcula el ratio entre la riqueza actual y la inicial

**Efecto en jugabilidad:**
- Si todos se empobrecen → rentas bajas → sobreviven más fácil
- Si hay jugador rico dominando → rentas suben → lo limita

---

## ⚙️ Especificaciones Técnicas

### **Lenguaje**
- JavaScript puro (sin frameworks)
- HTML5 + CSS nativo
- Funciona en navegadores modernos

### **Constantes del Juego**
- **SALARY**: $200,000 por pasar por SALIDA (actualizado: antes era $2M)
- **INIT_BALANCE**: $15,000,000 saldo inicial
- **RENT_UNIT_SCALE**: $1,000 unidad de renta base
- **Multiplicador de inflación**: Rango [1.0, 2.5]

### **Compatibilidad**
- ✅ Chrome/Chromium
- ✅ Firefox
- ✅ Edge
- ✅ Safari
- ✅ Opera

### **Requisitos**
- Navegador con JavaScript habilitado
- No requiere conexión a internet
- ~600KB de datos (HTML + JS + CSS)

### **Rendimiento**
- Actualización de UI: <100ms
- Cálculo numérico: <10ms (incluso con 100 rondas)
- Sin lag ni retrasos

---

## 📚 Entendiendo el Código

### **Estructura de archivos dentro del HTML**

```javascript
// PARTE 1: MÉTODO NUMÉRICO (líneas ~11-78)
- gaussianElimination()       ← Resuelve Ax=b
- polynomialFit()             ← Ajusta polinomio
- polynomialEval()            ← Evalúa p(x)

// PARTE 2: DATOS DEL JUEGO (líneas ~80-190)
- BOARD[]                     ← 40 casillas del tablero
- PLAYERS_CONFIG[]            ← Configuración de jugadores
- CHANCE_CARDS[]              ← Tarjetas de suerte
- COMM_CARDS[]                ← Tarjetas de comunidad

// PARTE 3: LÓGICA NUMÉRICA (líneas ~192-220)
- updateEconomicModel()       ← Captura y ajusta

// PARTE 4: MOTOR DEL JUEGO (líneas ~222-450)
- processRoll()               ← Procesa lanzamiento de dados
- processBuy()                ← Procesa compra de propiedades
- processEndTurn()            ← Pasa al siguiente turno

// PARTE 5: RENDERIZADO (líneas ~452-650)
- render()                    ← Dibuja la UI actual
- renderSetup()               ← Pantalla inicial
- renderGame()                ← Tablero y controles
- renderGameOver()            ← Pantalla de victoria
```

### **Estructura del estado del juego**

```javascript
GAME = {
  players: [                          // Array de jugadores
    {id, name, color, token, balance, pos, inJail, jailTurns, bankrupt}
  ],
  properties: {spotId: playerId},     // Quién posee cada propiedad
  currentPlayerIdx: 0,                // Índice del jugador actual
  gamePhase: "roll" | "buy" | "end",  // Fase actual
  diceRoll: [d1, d2],                 // Último lanzamiento
  log: ["evento1", "evento2", ...],   // Historial de eventos
  round: 1,                           // Número de ronda
  ecoData: [[r, wealth], ...],        // Datos para el método numérico
  polyCoeffs: [c0, c1, c2],           // Coeficientes ajustados
  inflationMultiplier: 1.25,          // Factor actual
  pending: null | space,              // Propiedad a comprar
  winner: null | player               // Ganador (null = en progreso)
}
```

## 📄 Archivos Incluidos

| Archivo | Contenido |
|---------|-----------|
| `monopoly_banco_electronico.html` | Aplicación completa (HTML + CSS + JS) |
| `README.md` | Este archivo (guía de uso completa) |
| `DOCUMENTACION_TECNICA.md` | Explicación detallada del método numérico (matemática) |

