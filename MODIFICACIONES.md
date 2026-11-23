# Modificaciones - Pokémon Cristal (Español)

Este documento detalla los cambios realizados en el código fuente de Pokémon Cristal para generar una versión en español para Game Boy Color con **multiplicador de experiencia x2** y **reparto de experiencia en toda la party** (estilo Generación 6+).

---

## 1. Base del Proyecto y Entorno

- **Base de código en español:** Rama basada en la decompilación en español de Pokémon Cristal (`MicaelAlighieri/pokecristal` / `erosunica/pokecrystal-es`).
- **Herramientas de compilación requeridas:**
  - `rgbds` (v1.0.4 o superior): `rgbasm`, `rgblink`, `rgbgfx`, `rgbfix`.
  - `make` y compilador de C (`gcc` o `clang`) para las herramientas auxiliares de gráficos y compresión.

---

## 2. Modificaciones en el Código Fuente

Archivo modificado: [`engine/battle/core.asm`](engine/battle/core.asm)

### A. Reparto de Experiencia a Todo el Equipo (Party EXP Share)

En el juego original, sólo los Pokémon que participaban activamente en combate o aquellos que sostenían el objeto *Repartir Experiencia* recibían puntos tras debilitar a un rival.

- **Ubicación:** `GiveExperiencePoints` (alrededor de la línea 6960).
- **Cambio:** Se eliminó la verificación del registro de bits `wBattleParticipantsNotFainted`.
- **Efecto:** Cada miembro del equipo con salud disponible (`MON_HP > 0`) recibe experiencia y puntos de esfuerzo (Stat EXP). Todos los eventos asociados a la subida de nivel (aumento de estadísticas, aprendizaje de movimientos y bandera para evolución al finalizar el combate) operan correctamente para cualquier miembro de la party.

### B. Sin División de Experiencia entre Participantes

- **Ubicación:** `GiveExperiencePoints` y subrutina `.EvenlyDivideExpAmongParticipants`.
- **Cambio:** Se eliminó la invocación de `call .EvenlyDivideExpAmongParticipants`.
- **Efecto:** La experiencia base del rival no se fragmenta si varios Pokémon lucharon o por tener varios miembros vivos en el equipo. Cada Pokémon vivo recibe el 100% íntegro de la experiencia.

### C. Multiplicador de Experiencia x2

- **Ubicación:** `GiveExperiencePoints` (justo tras el cálculo de modificadores como Lucky Egg / Pokémon intercambiado).
- **Cambio:** Se duplica el valor del cociente de experiencia (`hQuotient + 2` y `hQuotient + 3`) mediante rotación y suma con acarreo (`add a` / `adc a`), con saturación al valor máximo de 16 bits (`$ffff` / 65535) en caso de sobreflujo.
- **Efecto:** Todos los Pokémon del equipo reciben el doble de experiencia (200%). Tanto la cantidad numérica mostrada en el mensaje en pantalla (*"¡<Pokémon> ganó XXX Puntos EXP.!"*) como la animación de la barra de nivel reflejan con precisión la experiencia duplicada.

### D. Optimización del Flujo Post-Combate

- **Ubicación:** `UpdateBattleStateAndExperienceAfterEnemyFaint` (alrededor de la línea 2150).
- **Cambio:** Se retiró el bucle de doble comprobación de `IsAnyMonHoldingExpShare` y la división previa por 2 de los stats del enemigo.
- **Efecto:** La rutina llama directamente a `GiveExperiencePoints` en una sola pasada limpia y eficiente.

---

## 3. Instrucciones de Compilación

Para compilar la ROM desde la terminal:

```bash
# Compilación estándar utilizando todos los núcleos del procesador
make -j$(sysctl -n hw.ncpu 2>/dev/null || nproc) pokecristal.gbc
```

El archivo binario resultante será:
- **`pokecristal.gbc`** (2.0 MB / 2,097,152 bytes)
