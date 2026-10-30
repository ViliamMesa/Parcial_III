# Plan de Ejecución del Proyecto

## Sistema Integral de Gestión y Distribución de Donaciones Post-Desastre (Colombia)

**Asignatura:** Estructuras de Datos y Algoritmos — Universidad EAFIT
**Lenguaje:** Python 3.11+
**Evaluación Práctica 3**

---

## 1. Resumen Ejecutivo

El objetivo es construir un software de alto rendimiento que coordine la recepción, trazabilidad, priorización y despacho de ayuda humanitaria tras un sismo en Colombia. El sistema conecta **puntos de entrada** (aeropuertos, puertos, centros de acopio) con **municipios afectados** usando estructuras de datos óptimas (Hash Maps, Heaps, Árboles balanceados, Grafos) y algoritmos de caminos mínimos (Dijkstra/A*).

El plan se organiza en **4 fases metodológicas** (según el enunciado), cada una con su componente obligatorio de **pruebas automatizadas**, más un **anexo de transparencia de IA** y un **informe técnico** exigidos por la rúbrica.

---

## 2. Mapa de la Rúbrica → Entregables

| Criterio (peso) | Qué lo satisface | Fase |
|---|---|---|
| Modelado y Fuentes de Datos (15%) | Entidades fuertemente tipadas, inmutabilidad donde aplica, serialización robusta, manejo de I/O con excepciones | Fase 1 |
| Estructuras de Datos y Complejidad (30%) | Hash Map + Heap + AVL/Red-Black + Grafo con análisis Big-O formal y benchmarking vs. alternativas ingenuas | Fase 3 |
| Interfaz Gráfica y UX (15%) | GUI modular, paneles diferenciados, manejo asíncrono anti-congelamiento, validación de entradas | Fase 4 |
| Plan Integral de Pruebas Unitarias (20%) | Suite `pytest` con casos de borde, cobertura amplia y pruebas de rendimiento | Todas |
| Uso de IA y Sustentación Técnica (20%) | Anexo de prompts, justificación arquitectural y dominio demostrable del código | Transversal |

---

## 3. Arquitectura General (Python)

Separación en capas para desacoplar dominio, lógica, persistencia y UI (clave para "modelado desacoplado" en la rúbrica).

```
donaciones_postdesastre/
├── README.md
├── pyproject.toml                 # Configuración del proyecto y dependencias
├── requirements.txt
├── src/
│   └── donaciones/
│       ├── __init__.py
│       ├── domain/                # FASE 1 — Entidades del dominio
│       │   ├── __init__.py
│       │   ├── enums.py           # TipoAyuda, NivelUrgencia, CondicionFrio, EstadoLote
│       │   ├── donante.py
│       │   ├── lote_ayuda.py      # caducidad, tipo, peso, volumen, cadena de frío
│       │   ├── centro_acopio.py
│       │   └── municipio.py       # nivel de urgencia + necesidades cuantitativas
│       ├── data/                  # FASE 1 — Persistencia e intercambio
│       │   ├── __init__.py
│       │   ├── parsers.py         # Lectura/validación CSV y JSON
│       │   ├── serializers.py     # Serialización robusta
│       │   ├── validators.py      # Tipos, rangos, coherencia temporal
│       │   └── seed_data.py       # Datos sintéticos realistas (geografía CO)
│       ├── structures/            # FASE 3 — Estructuras de datos propias
│       │   ├── __init__.py
│       │   ├── hash_index.py      # Índices O(1) por llaves compuestas
│       │   ├── priority_queue.py  # Min/Max Heap de triaje
│       │   ├── avl_tree.py        # Árbol balanceado para consultas de rango
│       │   └── graph.py           # Grafo de transporte + Dijkstra / A*
│       ├── services/              # FASE 2 — Lógica de negocio
│       │   ├── __init__.py
│       │   ├── inventario.py      # Registro y categorización de donaciones
│       │   ├── busqueda.py        # Búsqueda multinivel
│       │   ├── asignacion.py      # Asignación automatizada por triaje
│       │   ├── ruteo.py           # Rutas óptimas de despacho
│       │   └── reportes.py        # Balance de stock, alertas tempranas
│       ├── gui/                   # FASE 4 — Interfaz gráfica
│       │   ├── __init__.py
│       │   ├── app.py             # Punto de entrada de la GUI
│       │   ├── panel_inventario.py
│       │   ├── panel_alertas.py
│       │   ├── panel_rutas.py
│       │   ├── formularios.py     # Formularios validados
│       │   └── async_workers.py   # Manejo asíncrono anti-congelamiento
│       └── app_main.py            # Orquestador / bootstrap
├── data/
│   ├── donantes.csv
│   ├── lotes.json
│   ├── centros.csv
│   └── municipios.json
├── tests/                         # Pruebas por fase
│   ├── test_domain.py
│   ├── test_parsers.py
│   ├── test_services.py
│   ├── test_structures.py
│   ├── test_benchmark.py          # Pruebas de rendimiento
│   └── test_gui.py
├── docs/
│   ├── INFORME_TECNICO.md         # Análisis Big-O + decisiones
│   ├── ANEXO_IA.md                # Transparencia IA (rúbrica 20%)
│   └── BENCHMARKS.md
└── scripts/
    └── generar_datos.py           # Generación de datos sintéticos
```

**Patrón:** Dominio puro (sin dependencias externas) → Servicios → GUI. Esto permite probar la lógica sin la interfaz y defender el "desacoplamiento" ante los evaluadores.

---

## 4. Decisiones Tecnológicas

| Componente | Elección | Justificación |
|---|---|---|
| Lenguaje | Python 3.11+ | Tipado con `typing`, `dataclasses`, rendimiento suficiente |
| Entidades | `@dataclass(frozen=True)` + `Enum` | Tipado fuerte e inmutabilidad donde aplica (rúbrica) |
| Serialización | `json` + `csv` estándar | Formatos interoperables exigidos |
| GUI | **PySide6 (Qt)** o `tkinter` | Qt para paneles profesionales y workers asíncronos (`QThread`) |
| Pruebas | `pytest` + `pytest-cov` | Suites automatizadas + cobertura |
| Benchmark | `timeit` / `pytest-benchmark` | Comparación vs. alternativas ingenuas |
| Estructuras | **Implementación propia** (AVL, Heap, Grafo) | La rúbrica evalúa la implementación y el análisis de complejidad |

> **Recomendación:** PySide6 para la GUI por su manejo nativo de concurrencia (`QThread`/`Signal`), que es lo que la rúbrica premia ("manejo asíncrono para evitar congelamiento de UI"). Si se prefiere cero dependencias de GUI, `tkinter` con `threading` + colas es alternativa viable.

---

## 5. Cronograma por Fases

### FASE 1 — Modelado y Fuentes de Datos *(días 1–3)*

**Entregables:**
- Entidades: `Donante`, `LoteAyuda` (caducidad, tipo, peso, volumen, cadena de frío), `CentroAcopio`, `Municipio` (urgencia + necesidades).
- Enums fuertemente tipados para categorías y estados.
- Parsers CSV/JSON con validación de registros mal estructurados o nulos.
- Validadores: tipos, rangos (cantidades ≥ 0), coherencia temporal (caducidad > fecha del evento).
- Datos sintéticos realistas (municipios y rutas reales de Colombia).

**Pruebas de la fase:**
- Parsing ante registros corruptos/nulos (no debe crashear; debe reportar).
- Validación de tipos, rangos y fechas.

**Criterio de aceptación:** un CSV/JSON con filas inválidas se carga reportando errores sin perder los registros válidos.

---

### FASE 2 — Especificación Funcional *(días 3–6)*

**Operaciones del sistema:**
1. Registro y categorización instantánea de donaciones en centros de acopio.
2. Búsqueda multinivel (por ID único, categoría, lotes de caducidad próxima).
3. Asignación automatizada por nivel de triaje/urgencia municipal.
4. Reportes en tiempo real: balance de stock, despachados vs. remanentes, alertas de desabastecimiento.

**Pruebas de la fase:**
- Una donación asignada **no** puede quedar en estado disponible.
- Consistencia de inventario tras transacciones masivas (secuenciales/concurrentes).

**Criterio de aceptación:** tras N asignaciones, `stock_total == disponible + asignado + despachado` siempre se cumple.

---

### FASE 3 — Estructuras de Datos y Complejidad *(días 5–10)* — **Núcleo (30%)**

| Estructura | Uso en el dominio | Complejidad objetivo |
|---|---|---|
| **Hash Map** | Índice de donantes, inventarios y municipios por llaves compuestas | O(1) amortizado |
| **Heap (Min/Max)** | Cola de triaje: urgencia del municipio + vida útil de perecederos | O(log n) push/pop |
| **Árbol AVL / Red-Black** | Consultas de rango por fecha de expiración o volumen de aporte | O(log n) búsqueda/rango |
| **Grafo + Dijkstra/A\*** | Red de transporte (acopios/municipios; aristas con peso = distancia/estado/tiempo) | Dijkstra O((V+E) log V) |

**Pruebas de la fase:**
- Análisis formal Big-O (tiempo y espacio) de cada estructura, documentado en `docs/INFORME_TECNICO.md`.
- Benchmarking: Heap vs. lista ordenada, Hash Map vs. búsqueda lineal, AVL vs. lista, Dijkstra sobre grafos crecientes.

**Criterio de aceptación:** las curvas de rendimiento confirman empíricamente la complejidad teórica (gráficas/tablas en `docs/BENCHMARKS.md`).

---

### FASE 4 — Interfaz Gráfica (GUI) *(días 9–13)*

**Componentes:**
- Panel de control global (estado del sistema sin saturación cognitiva).
- Tablas dinámicas de inventario.
- Formularios validados para ingreso de ayuda.
- Consola de alertas de prioridades de despacho.
- Visualización del flujo/red de distribución.
- Workers asíncronos para operaciones pesadas (ruteo, carga masiva) sin congelar la UI.

**Pruebas de la fase:**
- Respuesta ante eventos concurrentes de la UI.
- Retroalimentación visual clara ante errores o datos fuera de rango.

**Criterio de aceptación:** la UI no se congela durante un cálculo de rutas; las entradas inválidas muestran mensajes claros.

---

### CIERRE — Documentación y Sustentación *(días 13–15)*

- `docs/INFORME_TECNICO.md`: arquitectura, decisiones, análisis Big-O.
- `docs/ANEXO_IA.md`: prompts relevantes, justificación de decisiones asistidas por IA, demostración de dominio del código (rúbrica 20%).
- `README.md`: instalación, ejecución, estructura del repo.
- Suite de pruebas verde + reporte de cobertura.
- **Entrega individual** por cada integrante (código comprimido + informe + enlace al repo) en EAFIT Interactiva.

---

## 6. Estrategia de Pruebas (transversal, 20%)

- Framework: `pytest` con `pytest-cov`.
- Casos de **éxito**, de **borde** (listas vacías, fechas límite, cantidades cero, grafos desconectados) y de **error** (datos nulos/corruptos).
- Pruebas de **rendimiento** con `pytest-benchmark` o `timeit`.
- Meta de cobertura: **≥ 85%** en `domain`, `data`, `structures` y `services`.
- CI opcional (GitHub Actions) para ejecutar la suite en cada push.

---

## 7. Flujo de Trabajo en Equipo (máx. 4 integrantes)

| Rol sugerido | Responsabilidad principal |
|---|---|
| Integrante A | Fase 1 (dominio + persistencia) |
| Integrante B | Fase 3 (estructuras + Big-O + benchmarks) |
| Integrante C | Fase 2 (servicios/lógica de negocio) |
| Integrante D | Fase 4 (GUI + UX) |

- Git con ramas por feature y Pull Requests revisados.
- Todos participan en el `ANEXO_IA.md` y la sustentación (la rúbrica penaliza no dominar el código).
- Cada integrante sube la entrega completa individualmente (regla mandatoria EAFIT).

---

## 8. Riesgos y Mitigaciones

| Riesgo | Mitigación |
|---|---|
| GUI que congela la UI en cálculos pesados | Workers asíncronos (`QThread`/`threading` + colas) desde el inicio |
| Implementación de AVL/Grafo con bugs sutiles | Pruebas unitarias con invariantes (balanceo, propiedades de heap) |
| Datos sintéticos poco realistas | Usar municipios/carreteras reales de Colombia con distancias aproximadas |
| No poder sustentar código generado por IA | Documentar cada decisión en `ANEXO_IA.md` y revisar en equipo |

---

## 9. Primeros Pasos Inmediatos

1. Inicializar repositorio y estructura de carpetas.
2. Configurar `pyproject.toml` / `requirements.txt` (`pytest`, `pytest-cov`, `PySide6`).
3. Implementar los Enums y la entidad `LoteAyuda` (la más rica) + sus pruebas.
4. Construir los parsers/validadores con sus pruebas de borde.
5. Avanzar a estructuras de datos y servicios.

> **¿Siguiente acción?** Puedo generar el esqueleto completo del proyecto (carpetas, `pyproject.toml`, entidades de Fase 1 con sus pruebas) para arrancar de inmediato.
