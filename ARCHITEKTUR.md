# Donkey Kong Browser Game - Architektur-Dokument

## 1. Projektueberblick

### 1.1 Ziel
Ein Donkey Kong Browser Game als MVP (Minimum Viable Product), das in einer einzigen `index.html` Datei lauffaehig ist - ohne externe Abhaengigkeiten oder Bilder.

### 1.2 Sprint-Informationen
- **Sprint:** DK Sprint 1 - MVP Browser Game
- **Zeitraum:** 12.06.2026 - 26.06.2026
- **Goal:** Donkey Kong Browser Game: MVP mit allen Core-Features lauffaehig in einer einzigen index.html Datei

### 1.3 Epics (4)
| Epic | Issue-Key | Beschreibung |
|------|-----------|-------------|
| Spiel-Engine & Core | SCRUM-81 | Canvas, Input, Physik & Kamera |
| Game Design & Assets | SCRUM-82 | Spieler, Plattformen & Bananen |
| Gegner & Gameplay | SCRUM-83 | Goombas, Kollision & Leben |
| UI & Polish | SCRUM-84 | HUD, Game Over & Technik |

---

## 2. Technische Architektur

### 2.1 Architektur-Entscheidungen

| Entscheidung | Begründung |
|--------------|------------|
| **Single-File Architektur** (SCRUM-95) | Einfachste Verteilung, sofort im Browser lauffaehig, keine Build-Pipeline notwendig |
| **Canvas-basiertes Rendering** (SCRUM-85) | Pixel-Art Darstellung ohne externe Bilder, volle Kontrolle ueber Rendering |
| **requestAnimationFrame Game Loop** (SCRUM-85) | Standard fuer Browser-Games, 60 FPS, energieeffizient |
| **Keine externen Abhaengigkeiten** (SCRUM-95) | Zero-Dependency, keine Netzwerklatenz, offline-faehig |
| **Inline CSS/JS** (SCRUM-95) | Single-File Constraint, keine HTTP-Requests fuer Resources |

### 2.2 System-Architektur (Ebene: Anwendung)

```
┌─────────────────────────────────────────────────────────┐
│                    index.html                           │
│  ┌─────────────┐  ┌─────────────┐  ┌────────────────┐  │
│  │   <style>    │  │   <canvas>  │  │   <script>     │  │
│  │  CSS Styles  │  │  800x400px  │  │  JavaScript    │  │
│  │  Retro/Pixel │  │  Rendering  │  │  Game Logic    │  │
│  └─────────────┘  └─────────────┘  └────────────────┘  │
└─────────────────────────────────────────────────────────┘
```

### 2.3 Software-Architektur (Module / Klassen)

```
┌──────────────────────────────────────────────────────────────┐
│                        Game (Main)                           │
│  - gameLoop()              ── requestAnimationFrame @ 60fps │
│  - init()                  ── Setup aller Module            │
│  - reset()                 ── Vollstaendiger Spiel-Reset    │
└──────────────────┬───────────────────────────────────────────┘
                   │
     ┌─────────────┼─────────────┬──────────────┬──────────────┐
     ▼             ▼             ▼              ▼              ▼
┌─────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────┐
│ Input   │ │  Physics  │ │  Camera  │ │   HUD      │ │  Game    │
│ Manager │ │  Engine   │ │  Manager │ │  Manager   │ │  State   │
│         │ │           │ │          │ │            │ │          │
│ - keys  │ │ - gravity │ │ - x     │ │ - score   │ │ - lives  │
│ - left  │ │ - jump    │ │ - y     │ │ - lives   │ │ - score  │
│ - right │ │ - collide │ │ - follow│ │ - restart │ │ - state  │
│ - jump  │ │  (top)    │ │  (smooth)│ │  button   │ │  (play, │
└─────────┘ │ - collide │ │          │ └────────────┘ │  gameover)│
            │  (side)   │ └────────────┘              └──────────┘
            └──────────┘
                 │
     ┌───────────┼───────────┬──────────────┬──────────────┐
     ▼           ▼           ▼              ▼              ▼
┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐
│  Player   │ │ Platform │ │  Banana  │ │  Goomba   │ │  World   │
│  (DK)    │ │  Entity  │ │  Entity  │ │  Entity   │ │  Config  │
│          │ │          │ │          │ │          │ │          │
│ - x, y   │ │ - x, y   │ │ - x, y   │ │ - x, y   │ │ - width  │
│ - vx, vy │ │ - width  │ │ - collected│ │ - vx    │ │ - height │
│ - onGround│ │ - height │ │ - draw() │ │ - patrol │ │ - platforms│
│ - draw() │ │ - draw() │ │          │ │  bounds  │ │ - bananas│
│ - update │ │          │ └──────────┘ │ - draw() │ │ - goombas│
│ - anim   │ └──────────┘              │ - collide│ └──────────┘
└──────────┘                           └──────────┘
```

### 2.4 Datenfluss (Game Loop)

```
┌────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌────────┐
│ Input  │ ──► │ Update   │ ──► │ Physics  │ ──► │ Render   │ ──► │ Display│
│ Poll   │     │ Entities │     │ Resolve  │     │ Camera   │     │ Canvas │
└────────┘     └──────────┘     └──────────┘     └──────────┘     └────────┘
     │              │               │               │
     │              ▼               │               │
     │         ┌──────────┐         │               │
     │         │  Collision│ ───────┘               │
     │         │  Check    │                         │
     │         └──────────┘                         │
     │              │                                │
     │              ▼                                │
     │         ┌──────────┐                         │
     └─────────│  Game    │─────────────────────────┘
               │  State   │
               │  Update  │
               └──────────┘
```

---

## 3. Detaillierte Modul-Architektur

### 3.1 Game (Hauptmodul)

**Verantwortlichkeit:** Einstiegspunkt, Game Loop, Initialisierung, Reset

**Schnittstelle:**
- `init()` - Initialisiert alle Module und startet den Game Loop
- `gameLoop(timestamp)` - Wird von requestAnimationFrame aufgerufen
- `reset()` - Setzt den gesamten Spielzustand zurueck (ohne Page Reload)

**Abhaengigkeiten:** Alle Module

### 3.2 Input Manager

**Verantwortlichkeit:** Tastatureingaben erfassen und verwalten (SCRUM-86)

**Schnittstelle:**
- `isPressed(key)` - Gibt true zurueck, wenn Taste gedrueckt ist
- `keys` - Objekt mit aktuellen Tasten-Zustaenden

**Zustaende:**
```
keys = {
  left: false,    // Pfeiltaste links
  right: false,   // Pfeiltaste rechts
  jump: false     // Space oder Pfeil oben
}
```

### 3.3 Physics Engine

**Verantwortlichkeit:** Gravitation, Springen, Kollision mit Plattformen (SCRUM-87)

**Schnittstelle:**
- `applyGravity(entity)` - Wendet Gravitation auf Entity an
- `applyJump(entity)` - Setzt nach oben gerichtete Geschwindigkeit
- `checkTopCollision(entity, platforms)` - Kollision von oben (Landen)
- `checkSideCollision(entity, platforms)` - Kollision von der Seite

**Konstanten:**
```
GRAVITY = 0.6        // Pixel pro Frame²
JUMP_FORCE = -12     // Pixel pro Frame (nach oben)
MAX_FALL = 15        // Maximale Fallgeschwindigkeit
```

### 3.4 Camera Manager

**Verantwortlichkeit:** Scrollende Kamera mit Follow-Logik (SCRUM-88)

**Schnittstelle:**
- `follow(target, smoothing)` - Folgt dem Ziel mit Glättung
- `apply(ctx)` - Wendet Kamera-Offset auf Canvas-Context an
- `x, y` - Aktueller Kamera-Offset

**Verhalten:**
- DK bleibt grob in der Bildschirmmitte
- Smooth-Offset (lerp mit Faktor ~0.1)
- Kamera geht nicht außerhalb der Weltgrenzen

### 3.5 Player (Donkey Kong)

**Verantwortlichkeit:** Spieler-Charakter, Bewegung, Animation (SCRUM-89)

**Eigenschaften:**
```
Player = {
  x, y,              // Position
  vx, vy,            // Geschwindigkeit
  width: 32,         // Breite in Pixel
  height: 32,        // Hoehe in Pixel
  onGround: false,   // Am Boden?
  facing: 'right',   // Richtung: 'left' | 'right'
  animFrame: 0,      // Animations-Frame
  animTimer: 0,      // Timer fuer Animation
  invulnerable: 0,   // Invulnerabilitaets-Timer (Frames)
  visible: true      // Sichtbar (fuer Blink-Effekt)
}
```

**Animationen:**
- **Idle:** Ruhestellung, keine Bewegung
- **Walk:** Bei horizontaler Bewegung (2-Frame Toggle)
- **Blink:** Waehrend Invulnerabilitaet (sichtbar/unsichtbar toggle)

### 3.6 Platform Entity

**Verantwortlichkeit:** Plattformen auf verschiedenen Hoehen (SCRUM-90)

**Eigenschaften:**
```
Platform = {
  x, y,              // Position
  width,             // Breite (variiert)
  height: 16         // Hoehe in Pixel
}
```

**Level-Design (5 Plattformen):**
- Plattformen mit unterschiedlichen Y-Positionen
- Verschiedene X-Positionen und Breiten
- Pixel-Art Balken Darstellung

### 3.7 Banana Entity

**Verantwortlichkeit:** Einsammelbare Bananen mit Punktestand (SCRUM-91)

**Eigenschaften:**
```
Banana = {
  x, y,              // Position
  collected: false,   // Einsammelte?
  width: 16,         // Breite
  height: 16         // Hoehe
}
```

**Verhalten:**
- 10 Bananen auf Plattformen verteilt
- Bei Berührung: `collected = true`, Score +1
- Einsammelte Bananen erscheinen nicht wieder

### 3.8 Goomba Entity

**Verantwortlichkeit:** Gegner mit Patrouillen-Logik (SCRUM-92)

**Eigenschaften:**
```
Goomba = {
  x, y,              // Position
  vx,                // Geschwindigkeit (+/- Richtung)
  width: 24,         // Breite
  height: 24,        // Hoehe
  patrolLeft,        // Linke Patrouillen-Grenze
  patrolRight,       // Rechte Patrouillen-Grenze
  animFrame: 0,      // Walk-Animation
  platformRef        // Referenz zur Plattform
}
```

**Verhalten:**
- 2 Goombas auf verschiedenen Plattformen
- Hin-und-her Bewegung auf ihrer Plattform
- Bei Plattformende: Richtung umkehren (`vx = -vx`)
- Pixel-Art brauner Pilz mit Walk-Animation

### 3.9 HUD Manager

**Verantwortlichkeit:** Punktestand, Leben, Restart-Button (SCRUM-96)

**Schnittstelle:**
- `draw(ctx, camera)` - Zeichnet das HUD (kamera-unabhaengig)
- `drawScore(score)` - "Score: X" oben links
- `drawLives(lives)` - "❤❤❤" oben rechts
- `drawRestartButton()` - Restart-Button mit Klick-Handler

**Design:**
- Retro-Pixel-Stil
- Kamera-unabhaengig (fixed overlay)

### 3.10 Game State Manager

**Verantwortlichkeit:** Spielzustand, Leben, Game Over (SCRUM-93, SCRUM-94)

**Eigenschaften:**
```
GameState = {
  state: 'play',     // 'play' | 'gameover'
  score: 0,          // Punktestand
  lives: 3,          // Leben
  bananasTotal: 10,  // Gesamte Bananen
  bananasCollected: 0 // Einsammelte Bananen
}
```

**Zustandsuebergänge:**
```
  ┌─────────┐
  │  PLAY   │ ──(Berührung Goomba, lives > 0)──► Leben verloren
  └────┬────┘                                         │
       │                                               ▼
       │                                          ┌─────────┐
       │                                          │ Invuln  │ (Blink, Respawn)
       │                                          └────┬────┘
       │                                               │
       │                                               ▼
       │                                          (lives == 0)
       │                                               │
       │                                               ▼
       │                                          ┌──────────┐
       ◄──────────────────────────────────────────│ GAME OVER│
       │              (Restart)                    └──────────┘
```

---

## 4. Implementierungsplan fuer Developer

### 4.1 Implementierungsreihenfolge (Phasen)

Die Reihenfolge folgt dem Prinzip: **Fundament zuerst, dann Aufbau, dann Polish**. Jede Phase baut auf der vorherigen auf und liefert ein testbares Zwischenergebnis.

---

### Phase 1: Fundament (Spiel-Engine Core)

**Ziel:** Lauffaehiger Game Loop mit Canvas, auf dem alles aufbaut.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 1.1 | SCRUM-95 | HTML-Geruest mit inline CSS/JS, Canvas 800x400px | - |
| 1.2 | SCRUM-85 | Game Loop mit requestAnimationFrame, dunkler Hintergrund, zentriert | 1.1 |

**Ergebnis nach Phase 1:** Leeres Canvas (800x400) mit dunklem Hintergrund, das bei 60 FPS rendern tut.

---

### Phase 2: Spieler-Steuerung

**Ziel:** Beweglicher Spieler mit Input-Steuerung.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 2.1 | SCRUM-86 | Input-System: Pfeiltasten links/rechts, Space fuer Springen | Phase 1 |
| 2.2 | SCRUM-89 | Pixel-Art DK zeichnen, Idle/Walk-Animation, Richtung | 2.1 |

**Ergebnis nach Phase 2:** DK auf Canvas, steuerbar mit Pfeiltasten, Animation bei Bewegung.

---

### Phase 3: Physik & Welt

**Ziel:** Physik-Engine mit Gravitation, Plattformen und Kamera.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 3.1 | SCRUM-87 | Physik: Gravitation, Springen, Kollision (top + side) | Phase 2 |
| 3.2 | SCRUM-90 | 5 Plattformen auf verschiedenen Hoehen platzieren | 3.1 |
| 3.3 | SCRUM-88 | Scrollende Kamera mit Follow-Logik (smooth lerp) | 3.1 |

**Ergebnis nach Phase 3:** DK kann auf Plattformen laufen und springen, Kamera folgt ihm.

---

### Phase 4: Game Design (Bananen & Punktestand)

**Ziel:** Sammelbare Objekte mit Punktestand.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 4.1 | SCRUM-91 | 10 Bananen auf Plattformen, Einsammeln, +1 Punkt | Phase 3 |

**Ergebnis nach Phase 4:** DK kann Bananen einsammeln, Punktestand wird angezeigt.

---

### Phase 5: Gegner & Leben-System

**Ziel:** Goomba-Gegner mit Patrouille und Leben-System.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 5.1 | SCRUM-92 | 2 Goombas mit Patrouillen-Logik, Walk-Animation | Phase 3 |
| 5.2 | SCRUM-93 | 3 Leben, bei Goomba-Beruehrung -1, Invulnerabilitaet, Respawn | 5.1 |

**Ergebnis nach Phase 5:** Goombas patrouillieren, DK verliert Leben bei Beruehrung, blinkt waehrend Invulnerabilitaet.

---

### Phase 6: UI & Polish

**Ziel:** Vollstaendiges HUD, Game Over Screen, Restart.

| Schritt | Issue | Beschreibung | Abhaengigkeiten |
|---------|-------|-------------|-----------------|
| 6.1 | SCRUM-96 | HUD: Score oben links, Herzen oben rechts, Restart-Button | Phase 4 |
| 6.2 | SCRUM-94 | Game Over Screen bei 0 Leben, Restart ohne Page Reload | 5.2, 6.1 |

**Ergebnis nach Phase 6:** Vollstaendiges Spiel mit HUD, Game Over, Restart - MVP fertig!

---

### 4.2 Abhaengigkeitsgraph

```
Phase 1 (Fundament)
  SCRUM-95 ──► SCRUM-85
       │
       ▼
Phase 2 (Steuerung)
  SCRUM-86 ──► SCRUM-89
       │
       ▼
Phase 3 (Physik & Welt)
  SCRUM-87 ──► SCRUM-90
       │          │
       │          └──────────────┐
       ▼                         ▼
  SCRUM-88                  (parallel)
       │
       ▼
Phase 4 (Game Design)
  SCRUM-91
       │
       ▼
Phase 5 (Gegner & Leben)
  SCRUM-92 ──► SCRUM-93
       │
       ▼
Phase 6 (UI & Polish)
  SCRUM-96 ──► SCRUM-94
```

---

### 4.3 Code-Struktur innerhalb von index.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Donkey Kong Browser Game</title>
  <style>
    /* === CSS: Retro Pixel-Stil === */
    /* - Canvas Zentrierung */
    /* - Dunkler Hintergrund */
    /* - HUD Styles (Pixel-Font) */
    /* - Game Over Overlay */
  </style>
</head>
<body>
  <canvas id="game" width="800" height="400"></canvas>

  <script>
    // =====================================================
    // 1. CONSTANTS & CONFIG
    // =====================================================
    // - World dimensions
    // - Physics constants (GRAVITY, JUMP_FORCE, etc.)
    // - Entity sizes
    // - Level layout (platforms, bananas, goombas)

    // =====================================================
    // 2. GAME STATE
    // =====================================================
    // - GameState object (score, lives, state)
    // - Player object
    // - Entity arrays (platforms, bananas, goombas)

    // =====================================================
    // 3. INPUT MANAGER
    // =====================================================
    // - Keydown/keyup handlers
    // - isPressed(key) function

    // =====================================================
    // 4. ENTITY CLASSES / FACTORY FUNCTIONS
    // =====================================================
    // - Player: update(), draw()
    // - Platform: draw()
    // - Banana: draw()
    // - Goomba: update(), draw()

    // =====================================================
    // 5. PHYSICS ENGINE
    // =====================================================
    // - applyGravity(entity)
    // - checkTopCollision(entity, platforms)
    // - checkSideCollision(entity, platforms)

    // =====================================================
    // 6. CAMERA MANAGER
    // =====================================================
    // - follow(target, smoothing)
    // - apply(ctx)

    // =====================================================
    // 7. HUD MANAGER
    // =====================================================
    // - draw(ctx, gameState)
    // - drawScore(), drawLives(), drawRestartButton()

    // =====================================================
    // 8. GAME OVER MANAGER
    // =====================================================
    // - drawGameOver(ctx)
    // - handleRestart()

    // =====================================================
    // 9. GAME (MAIN)
    // =====================================================
    // - init()
    // - reset()
    // - update(dt)
    // - render(ctx)
    // - gameLoop(timestamp)

    // =====================================================
    // 10. BOOTSTRAP
    // =====================================================
    // - init() on DOMContentLoaded
  </script>
</body>
</html>
```

---

## 5. Level-Design (Konfiguration)

### 5.1 Welt-Dimensionen
```
WORLD_WIDTH = 1600   // Breite der Spielwelt (Pixel)
WORLD_HEIGHT = 400   // Hoehe der Spielwelt (Pixel)
```

### 5.2 Plattform-Konfiguration (5 Plattformen)
```
PLATFORMS = [
  { x: 0,     y: 360, width: 400 },   // Ground (links)
  { x: 350,   y: 280, width: 200 },   // Mitte-unten
  { x: 100,   y: 200, width: 250 },   // Links-oben
  { x: 450,   y: 160, width: 300 },   // Rechts-oben
  { x: 850,   y: 240, width: 350 },   // Rechts-unten (weiter)
]
```

### 5.3 Bananen-Konfiguration (10 Bananen)
```
BANANAS = [
  // Auf Plattform 1 (Ground)
  { x: 50,  y: 340 }, { x: 150, y: 340 }, { x: 300, y: 340 },
  // Auf Plattform 2 (Mitte)
  { x: 400, y: 260 }, { x: 500, y: 260 },
  // Auf Plattform 3 (Links-oben)
  { x: 150, y: 180 }, { x: 250, y: 180 },
  // Auf Plattform 4 (Rechts-oben)
  { x: 500, y: 140 }, { x: 650, y: 140 },
  // Auf Plattform 5 (Rechts-unten)
  { x: 950, y: 220 },
]
```

### 5.4 Goomba-Konfiguration (2 Goombas)
```
GOOMBAS = [
  {
    platformIndex: 2,    // Auf Plattform 3
    x: 150, y: 176,
    speed: 1.5,
    patrolLeft: 100,
    patrolRight: 330
  },
  {
    platformIndex: 4,    // Auf Plattform 5
    x: 900, y: 216,
    speed: 2.0,
    patrolLeft: 850,
    patrolRight: 1180
  },
]
```

---

## 6. Risiken & Open Points

### 6.1 Identifizierte Risiken

| Risiko | Einschätzung | Massnahme |
|--------|-------------|-----------|
| **Pixel-Art ohne externe Bilder** | Mittel | Canvas-Drawing mit fillRect-Pixel-Mapping; kann zeitaufwaendig sein |
| **Single-File wird gross** | Niedrig | Bei ~12 Issues und MVP-Umfang ist die Dateigroesse vertretbar |
| **Kollision mit diagonalen Plattformen** | Mittel | Klare Trennung von top- und side-collision notwendig |
| **Kamera-Clamping an Weltgrenzen** | Niedrig | Einfache Math.max/Math.min Logik |

### 6.2 Open Points (klaeren vor Implementierung)

1. **Weltbreite:** Ist 1600px ausreichend? Sollte so gross sein, dass die Kamera-Bewegung sinnvoll ist.
2. **Pixel-Font:** Soll eine externe Pixel-Font geladen werden (verletzt Single-File) oder soll mit Standard-Font gearbeitet werden?
3. **Sound:** Ist Sound im MVP geplant? Falls ja, muss Base64-encoding fuer Audio-Dateien in Betracht gezogen werden.

---

## 7. Definition of Done (pro Issue)

Jedes Issue gilt als erledigt, wenn:
- [ ] Alle Akzeptanzkriterien aus der Issue-Beschreibung sind erfuellt
- [ ] Der Code ist in `index.html` integriert (Single-File)
- [ ] Das Spiel laeuft im Browser ohne Fehler (Console frei von Errors)
- [ ] Keine Regressionen in zuvor implementierten Features

---

## 8. Glossar

| Begriff | Bedeutung |
|---------|-----------|
| DK | Donkey Kong (Spieler-Charakter) |
| MVP | Minimum Viable Product |
| FPS | Frames per Second |
| HUD | Heads-Up Display |
| AABB | Axis-Aligned Bounding Box (Kollisionsdetection) |
| Lerp | Linear Interpolation (fuer Kamera-Glaettung) |
| Invulnerabilitaet | Kurzzeitige Unverwundbarkeit nach Treffer |

---

*Architektur-Dokument erstellt am 12.06.2026 basierend auf den Jira-Issues des SCRUM-Boards.*