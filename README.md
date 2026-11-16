# ☢️ NERV DATA ANALYSIS DIVISION

## **PROJECT: MÓDULO 03 · PRÁCTICA 03**
### **POKÉMON · CLUSTERING & DREAM TEAM**

```text
╔══════════════════════════════════════════════╗
║ STATUS     :: COMPLETE                       ║
║ OPERATOR   :: JUAN                           ║
║ FRAMEWORK  :: PYTHON                         ║
║ OBJECTIVE  :: GROUP 801 POKÉMON (K-MEANS)    ║
╚══════════════════════════════════════════════╝
```

---

> ⚠️ **MISSION BRIEFING**
>
> Este repositorio contiene el desarrollo de la **Práctica #3 del Módulo 3** del Diplomado de Ciencia de Datos.
>
> El objetivo es **formar grupos de Pokémon a partir de sus características** con aprendizaje no supervisado, perfilarlos, ponerles nombre y usarlos para contestar tres preguntas: **¿cuál es el mejor grupo?, ¿cuál es el peor?** y **¿se puede armar un equipo de ensueño de 6 Pokémon?**
>
> La idea central es la misma de la práctica 1: **representar cada elemento como un vector y medir qué tan cerca están dos vectores**. Ahora, en lugar de recomendar, K-Means usa esas distancias para agrupar.

---

## 🧬 DATASET :: POKÉMON (SEREBII.NET)

```text
SOURCE  :: SEREBII.NET (SIETE GENERACIONES)
TASK    :: CLUSTERING (UNSUPERVISED)
ROWS    :: 801 POKÉMON
COLUMNS :: 41
STATUS  :: ANALYZED
```

| Bloque de columnas | Columnas | Uso |
|---|---|---|
| Estadísticas de combate | `hp`, `attack`, `defense`, `sp_attack`, `sp_defense`, `speed` | **Agrupar** |
| Tamaño | `height_m`, `weight_kg` (en logaritmo) | **Agrupar** |
| "Rareza" | `capture_rate`, `base_egg_steps` (en logaritmo), `experience_growth` | **Agrupar** |
| Defensa por tipo | 18 columnas `against_*` | Perfilar y duelos |
| Tipos y otros | `type1`, `type2`, `is_legendary`, `generation`, `base_happiness`, `percentage_male` | Perfilar |

### DATA OPERATIONS

- ✔️ Revisión de nulos: `type2` (384) y `percentage_male` (98) no son errores; `height_m` y `weight_kg` (20 c/u) se llenan con la mediana.
- ✔️ `capture_rate` venía como texto por **Minior** (`30 (Meteorite)255 (Core)`); se conserva el primer valor.
- ✔️ Logaritmo a las variables muy sesgadas (altura y peso pasan de una asimetría de ~5 a ~0).
- ✔️ Escalado con z-score, explicado primero con un ejemplo de juguete a mano (Pikachu, Jolteon y Snorlax).
- ✔️ **Ojo:** el enunciado habla de 802 Pokémon, pero el archivo trae **801**.

---

## 🟣 CLUSTERING CORE

```text
MODULE  :: FEATURE SPACE + K-MEANS
METRIC  :: EUCLIDEAN DISTANCE (Z-SCORE)
K       :: 6
STATUS  :: CLUSTERED
```

### DESIGN DECISIONS

- ✔️ **11 variables** para agrupar. Se dejan fuera `base_total` (es la suma de las estadísticas), las 18 `against_*` (agruparían por tipo y no por poder), y `is_legendary` a propósito, para ver si los grupos lo descubren solos.
- ✔️ Elección de *k* con **inercia, silhouette y Davies-Bouldin** (k = 2 a 10). Los índices prefieren k = 2 o 3, pero esos solo separan por nivel de poder. Se escoge **k = 6** para poder perfilar estilos de combate; es una decisión, no un óptimo.
- ✔️ Estabilidad: ARI de 0.95 entre semillas y 0.82 con el 80 % de los datos. Ward y la mezcla gaussiana coinciden solo a medias (0.52 y 0.25).
- ✔️ Grupos reordenados por `base_total` y **nombrados con una regla sobre su perfil** (no por su número, que es arbitrario).

---

## 🟢 SQUAD ROSTER :: LOS 6 GRUPOS

```text
GROUPS   :: 6
METHOD   :: K-MEANS + PROFILING
STATUS   :: NAMED
```

| Grupo | Pokémon | `base_total` | Duelos ganados | Perfil |
|---|:---:|:---:|:---:|---|
| **Titanes** | 71 | 627 | 82.3 % | Todo alto; 83 % legendarios; dragon, psychic y steel |
| **Relámpagos** | 133 | 503 | 65.2 % | Los más veloces (103) con buen ataque especial; frágiles a roca |
| **Gladiadores** | 129 | 492 | 63.4 % | Ataque físico (108), mucho HP, grandes y pesados; ground, fighting y rock |
| **Murallas** | 109 | 481 | 60.3 % | Defensas altas (198 sumadas), los más lentos (49); fairy, steel y water |
| **Reclutas** | 161 | 363 | 38.4 % | Nivel medio-bajo, sin especialidad clara |
| **Novatos** | 198 | 289 | 23.2 % | Los más débiles y fáciles de capturar |

Los **Titanes no son solo legendarios**: 12 de los 71 no lo son (Garchomp, Salamence, Metagross, Dragonite...), y 11 legendarios cayeron en otros grupos.

---

## 🟡 VISUALIZATION :: PCA & UMAP

```text
PCA   :: 2D + 3D
UMAP  :: 2D
EXTRA :: HEATMAP / RADAR / TYPE LIFT
```

- ✔️ **PCA:** PC1 (43 % de la varianza) es el "tamaño y poder general"; PC2 (11 %) separa rápidos y especiales de pesados y defensivos.
- ✔️ **UMAP:** el 83 % de los vecinos más cercanos de un Pokémon son de su mismo grupo (en PCA 2D, 66 %; al azar, 18 %).
- ✔️ Perfilamiento con mapa de calor (desviaciones estándar), radar de estadísticas, tipos sobrerrepresentados (*lift*) y daño recibido por tipo de ataque.

---

## 🔴 BATTLE MODEL :: MEJOR Y PEOR GRUPO

```text
MODULE  :: 1v1 DUEL MATRIX (801 x 801)
LEVEL   :: 50
STATUS  :: SIMULATED
```

Para comparar grupos con algo más completo que el `base_total` se arma un **modelo de duelos**: cada Pokémon golpea con el mejor de sus tipos (STAB x1.5, poder 80) y gana quien tumba al otro en menos golpes (si empatan, el más rápido).

| Pregunta | Respuesta |
|---|---|
| **¿Mejor grupo?** | **Titanes** (82.3 % de sus duelos; 80.6 % sin legendarios) |
| **¿Peor grupo?** | **Novatos** (23.2 % de sus duelos) |

Entre Relámpagos, Gladiadores y Murallas la diferencia es de 2 a 5 puntos y se traslapan, así que **no se declara un ganador claro** en la zona media.

---

## 🟠 DREAM TEAM

```text
SEARCH   :: LOCAL SEARCH (40 RESTARTS)
SCORE    :: COVERAGE + WINRATE - WEAKNESS PENALTY
STATUS   :: OPTIMIZED
```

| Equipo | Integrantes |
|---|---|
| **Ensueño (sin legendarios)** | Gyarados · Salamence · Metagross · Slaking · Garchomp · Lucario |
| **Ensueño (con legendarios)** | Gyarados · Dialga · Rayquaza · Metagross · Arceus · Xerneas |

- ✔️ **Cobertura del 100 %:** para cada uno de los 801 Pokémon hay al menos un integrante del equipo que le gana.
- ✔️ Nunca hay más de **2** integrantes débiles al mismo tipo de ataque.
- ✔️ Contra 5,000 equipos al azar gana el 91 % de los duelos; contra equipos "de élite" (armados con los 100 mejores), el 64 % y vence al 93.5 %.
- ✔️ Tres semillas distintas encuentran el mismo equipo sin legendarios.

**Respuesta:** sí es posible un equipo muy dañino y relativamente impermeable, pero **no invencible**.

---

## 🔵 LOADED MODULES

```text
PANDAS
NUMPY
SCIKIT-LEARN
UMAP-LEARN
PLOTLY
```

---

## 🔴 MISSION OBJECTIVES

```text
PRIMARY OBJECTIVES

[01] Preparar las variables (limpieza, logaritmos, escalado) y justificar cuáles se usan.
[02] Elegir el número de grupos con inercia, silhouette y Davies-Bouldin.
[03] Ajustar K-Means y revisar su estabilidad.
[04] Visualizar los grupos con PCA y UMAP.
[05] Perfilar y nombrar cada grupo.
[06] Determinar el mejor y el peor grupo con un modelo de duelos.
[07] Construir y poner a prueba un equipo de ensueño de 6 Pokémon.
```

---

## ⚙️ ACTIVATION PROTOCOL

```bash
# 1. Dependencias
pip install pandas numpy scikit-learn umap-learn plotly nbformat

# 2. Colocar el archivo pokemon.csv junto al notebook

# 3. Abrir y ejecutar el notebook
M3T3_PokemonClustering.ipynb
```

```text
Practica3_M3/
├── M3T3_PokemonClustering.ipynb   ← notebook de la práctica
├── pokemon.csv                    ← datos (colócalos aquí)
└── pokemon_grupos.csv             ← se genera al final del notebook
```

---

## ⚠️ FINAL PROTOCOL

```text
KNOWN LIMITATIONS

CLUSTER QUALITY :: GLOBAL SILHOUETTE 0.145 (NO NATURAL ISLANDS)
K SELECTION     :: CHOSEN FOR INTERPRETABILITY, NOT OPTIMUM
DUEL MODEL      :: NO MOVES, ABILITIES, ITEMS OR SWITCHING
```

- **Grupos no "naturales":** los Pokémon forman una nube continua de débil a fuerte y K-Means la corta en 6. Murallas y Gladiadores se traslapan con sus vecinos (36 % y 23 % de sus Pokémon quedan más cerca de otro grupo).
- **Modelo de duelos simplificado:** sin movimientos, habilidades ni objetos. Por eso **Slaking** sale en el equipo aunque su habilidad (*Truant*) lo haría mala opción en un combate real.
- **Equipo de ensueño dependiente del puntaje:** se evaluó contra Pokémon individuales y equipos armados al azar, no contra estrategias reales. Si se priorizara solo el daño saldría otro equipo, con más debilidades.
- **Datos:** 20 alturas y pesos imputados con la mediana, y el archivo trae 801 Pokémon en lugar de 802.

---

<div align="center">

```text
      ── CLUSTERING SCAN COMPLETE ──

         Grupos generados.
```

</div>
