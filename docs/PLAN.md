# AEROFLOW · PLAN

Plan de desarrollo, decisiones tomadas y pendientes.

---

## 1. Comparativa matemática aeroflow ↔ aeroflow2

### 1.1 Dinámica del flujo de aire — veredicto: gana la ESTRUCTURA de aeroflow2, se conserva el integrador de aeroflow

| Aspecto | aeroflow (v1) | aeroflow2 (v2) | Decisión |
|---|---|---|---|
| Integración | Aceleración + arrastre `exp(-λ·dt)` (ecuación de momento con fricción aerodinámica) + slam de velocidad máx. | `vel.lerp(campo, 0.15)` (relajación cinemática de 1er orden) | **Conservar v1** (físicamente más riguroso) |
| Chorro (jet) | Pluma gaussiana: `exp(-x·0.3)·exp(-r²/2w²)`, `w` crece con `x` (ensanchamiento real) | Descomposición axial/radial con perfil parabólico `1-r²/R²` y `1/(1+0.6x)` | Ambos válidos; conservar gaussiana de v1 |
| Succión | Mismo modelo gaussiano limitado + zona de reciclaje al atravesar el plano | Ley inversa-cuadrática con corte a 2.4 u | Conservar v1 |
| Colisiones | AABB reales contra componentes + rebotes elásticos | Solo paredes del case | Conservar v1 |
| Reposo natural | ✗ Ruido ambiental incondicional (`sin(py·1.7+t)·0.09`), flotabilidad con piso no-nulo, respawn con velocidades aleatorias y reset de partículas "atascadas" → la corriente nunca muere | ✓ El campo es exactamente 0 sin fuentes activas; flotabilidad solo si `T > 24 °C` → el reposo es un **punto fijo de las ecuaciones** | **Portar de v2**: la quietud emerge de la física, no de una condición |

### 1.2 Temperaturas — veredicto: gana aeroflow2 (v1 no tenía modelo térmico) pero su modelado es mejorable

- v2 CPU: `T⁎ = 36 + (1−η)·32` (η = rpm/máx), apagado → drift a 92 °C (throttling). *Base fija 36 °C independiente del aire del case: mejora.*
- v2 GPU: `T⁎ = 42 + 16/max(0.4, intake/150)`; ignora RPM de propios fans y el exhaust.
- v2 Case: `24 + 0.08·cpu + 0.08·gpu` — suma lineal cruda, sin masa térmica ni balance de caudales.

**Modelo térmico adoptado en v1 (basado en las ideas de v2, corregido):**

1. **Nodo Case (con masa térmica `τ = 7 s`):** `T_case⁎ = T_amb + ΔQ·(load/190)·(1/(1+CFM/90))` — balance entre calor disipado al chasis y renovación de aire (intake+exhaust+PSU interior activos).
2. **CPU:** resistencia térmica del cooler decreciente con RPM: `T⁎ = T_case + 8 + 34·(1−η)^1.15` (`τ=2.6 s`). Sin ventilador: empapado térmico hacia `min(95, T_case+52)` con `τ=8 s` → throttling (idea de v2).
3. **GPU:** ventiladores propios con RPM: `T⁎ = min(92, T_case + Δfan(η) + 46·(1/(1+CFM/130))·carga)` (`τ=4 s`); retirada → ambiente.
4. **Ambiente:** 22 °C constante (oficina).
5. **Intercambio partícula↔componente advectivo:** el coeficiente de calentamiento es proporcional a la velocidad local (intercambio convectivo real). Con aire estancado el coeficiente tiende a 0 → **el aire no se calienta sin flujo y no nace convección**: quietud otra vez como punto fijo.

---

## 2. Implementado en esta iteración

- [x] Portar la semántica de campo de v2 al motor de partículas de v1 (mantener partículas, render y panel **sin cambios**):
  - Turbulencia ambiental ahora escala con la magnitud local del campo generado por los fans (cizalla, no ruido perpetuo).
  - Flotabilidad en °C: `ΔT·k` con piso natural en `T_case` (ya no `temp·1.25` con baseline fija).
  - Respawn ambiental con velocidad ~0; el "rescate" de partículas atascadas solo actúa si hay fuentes de flujo activas.
  - Temperatura de partículas en °C reales (acoplada al modelo térmico); paleta de color intacta vía mapeo `(T−20)/55`.
- [x] Modelo térmico CPU / GPU / CASE / AMBIENTE + panel de telemetría (4 tarjetas CPU·GPU·Case·Ambiente).
  - El "case" es nuevo (v2 lo tenía mal modelado; aquí es nodo de primer orden con masa).
- [x] Cooler CPU: posiciones de montaje **frente** (estilo Intel stock: ventilador sobre el disipador, en vista acostada de la placa queda encima del CPU; sopla a través del disipador hacia la placa), **arriba**, **abajo**, **izquierda** (atrás) y **derecha** (frente). El flujo atraviesa el disipador saliendo por la cara opuesta a la de montaje → 5 posiciones físicamente consistentes.
- [x] Fans de la GPU: ahora son fuentes de flujo reales (convección forzada a través del disipador, a lo v2 pero dependiente de RPM) y configurables por clic derecho: **encender/apagar**, **RPM** y **voltear dirección** (↑ a través del disipador / ↓ volteado). El giro y glow 3D siguen RPM/power/flip. El clic izquierdo sigue instalando/retirando la GPU.
- [x] Fan de GPU como fuente de flujo real — dinámica del disipador v4 (aspiración + colector de aletas + célula toroidal):
  - **Succión de column ancha** (banda `m ∈ (−3.0, 0.1)` medido desde la cara de los fans, envolvente parabólica hacia abajo, laterales gaussianas anchas) recluta el aire quieto de todo el sector inferior.
  - **Colector de aletas**: el aire pegado a la cara de aspiración se desliza **en paralelo a la tarjeta** (`az`) hacia las bocas (extremos en Z), como en una tarjeta real (el empuje vertical dentro del volumen sólido de la GPU no afecta a ninguna partícula: los canales están dentro del obstáculo).
  - **Impulso al cruzar las bocas** (a `|dz| > 0.95`) y ascenso final por flotabilidad del pluma caliente.
  - **Retorno toroidal** (`side || above ⇒ ay ∓ 0.13·strength`): el aire que ya subió desciende por los laterales del case y sobre la tarjeta, cerrando el circuito por conservación de masa (sin retorno, el case superior se drena y circula un vacío estacionario).
  - El **flip** invierte la cara de aspiración con las mismas bandas espejadas por `sgn`.
- [x] Chorro del cooler CPU: halo de *entrainment* (succión radial gaussiana σ = 2.6·ancho, decay 0.55 axial) que recluta aire quieto alrededor de la pluma sin condición artificial.
- [x] Verificado con partículas instrumentales (tracer de trayectoria + desplazamiento de población con vidas congeladas):
  - **Todo apagado 30 s: quietud exacta** — 0/700 partículas se mueven, `v = 8e-5`, p95 = 0, fuentes = 0 (la quietud es punto fijo de la física ✓).
  - Config completa: `v ≈ 2.1`, p95 ≈ 3.8, equilibrio térmico CPU 46.9 °C / GPU 49.2 °C / case 30.4 °C.
  - Solo GPU (2000 rpm): circuito completo verificado por tracer (aspirada → deslizamiento por aletas → boca → ascenso; pico ~2.3 u/s, ciclo ~6 s).
  - Solo GPU volteada: reversa funciona (desplazamiento p90 = 1.15 u / 6 s).
  - Solo CPU (1800 rpm): chorro + halo visibles (p99 = 0.8 u/s).
  - Trayectoria mínima d = 0.0000 exacta tras 5 s en campo vacío.
- [x] `resetSimulation` restaura **también el estado térmico** al arranque en frío (CPU 34 / GPU 44 / case 26 °C).

---

## 3. Pendientes (roadmap)

- [ ] Curvas de ventilador automáticas (modo "auto": RPM objetivo según temperatura del CPU/GPU, histéresis + pendiente configurable).
- [ ] Slider de temperatura ambiente (5–35 °C) y perfil de carga del CPU (idle/ juego / stress) para el componente térmico.
- [ ] Resistencia de las rejillas: pérdida de carga por filtro/polvo que escale el CFM efectivo (y contador visual de polvo asociado a la presión negativa).
- [ ] Presión por zonas (frontal/CPU/GPU/bajo) en vez de global en el medidor.
- [ ] Modelar el canal de aletas real del disipador (deflectores que redirigen el jet al chocar con la torre en vez de rebote elástico genérico).
- [x] ~~Extracción del GPU coherente con el flip~~ → hecho en v4: succión/deslizamiento/bocas son espejadas por `sgn`; queda pendiente solo el detalle visual de la "garganta" del backplate.
- [ ] Métrica de ruido estimada (dBA) y potencia consumida por ventilador en telemetría.
- [ ] Presets rápidos (silencioso / balanced / performance) en el panel derecho.
- [ ] Guardar/restaurar configuración (localStorage) y export/import de escena.
- [ ] Perf: refactorizar el campo de v1 a zero-allocation total (buffers prealocados al estilo de v2) y cache por frame de `totalAirflowCfm`.
- [ ] Móvil: recolocar tarjetas térmicas y verificar el panel en pantallas < 400 px.
- [ ] Suite de verificación: formalizar como script re-ejecutable el test determinista que se corrió esta iteración (apagar todo, medir `max|v| < ε` tras N s y encender fan a fan) usando `window.__AEROFLOW`.
- [ ] Tuning fino del flujo aislado del CPU (el chorro + halo existen; el reclutamiento del volumen restante es moderado — regla actual: fuente localista, es aceptable).

---

## 4. Notas de mantenimiento

- El HUD térmico(`updateThermal`) corre en el loop de animación; `updateTelemetry` sigue siendo solo por eventos de interacción.
- Constantes calibrables del modelo térmico: `38/190/90` (case), `8+34/1.15` (CPU), `46/130` (GPU soak), `τ`: 7·2.6·8·4 s.
- La posición del cooler CPU vive en `CPU_POSITIONS` (frente/arriba/abajo/izquierda/derecha) y todo el resto del código lee `cpu.dirName` (compatibilidad con tooltip/telemetría intacta).

Doc v1 · 2026-11-02
