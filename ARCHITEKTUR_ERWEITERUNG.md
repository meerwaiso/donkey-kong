# Donkey Kong - Architektur-Erweiterung: Charaktere & 3 Welten

> **WICHTIG FÜR DEN DEVELOPER:** Dieses Dokument beschreibt eine **Erweiterung** der bestehenden Architektur.
> **KEIN NEUSTART, KEIN REWRITE.** Bestehender Code wird **NICHT** berührt — nur erweitert.
> Alle neuen Module werden als Add-ons integriert.

---

## 1. Zielsetzung

### 1.1 Anforderung 1: Donkey Kong Ähnlichkeit
Die Spielfiguren (Hauptfigur + Gegner) sollen optisch und mechanisch deutlich mehr an den klassischen Donkey Kong (Arcade, 1981) erinnern.

### 1.2 Anforderung 2: 3 Welten
Das Spiel soll 3 thematisch unterschiedliche Welten bieten:
| Welt | Theme | Farbpalette | Hintergrund |
|------|-------|-------------|-------------|
| **1 — Dschungel** | Ursprüngliches DK-Feeling | Grün, Braun, Gold | Lianen, Bäume, Wolken |
| **2 — Schnee** | Winterliche Bergwelt | Weiß, Eisblau, Silber | Schneeflocken, Eisblöcke, Berggipfel |
| **3 — Wüste** | Heiße Wüstenlandschaft | Gold, Ocker, Terrakotta | Dünen, Palmen, Pyramiden-Silhouette |

### 1.3 Sprint-Informationen
- **Sprint:** DK Sprint 2 — Charaktere & Welten
- **Zeitraum:** 13.06.2026 - 27.06.2026
- **Voraussetzung:** Sprint 1 MVP (bestehendes Spiel) muss stabil laufen
- **Goal:** DK-ähnliche Charaktere + 3 voll spielbare Welten mit Welt-Auswahl-Screen

---

## 2. Bestandsanalyse (Was existiert bereits)

### 2.1 Vorhandene Module (NICHT anfassen!)
```
Game (Main)          ── gameLoop, init, reset        [BERÜHREN NUR FÜR ERGÄNZUNG]
Input Manager        ── Pfeiltasten, Space           [ERWEITERN UM WELT-AUSWAHL]
Physics Engine       ── Gravitation, Kollision       [BERÜHRT NICHT]
Camera Manager       ── Follow, lerp                 [BERÜHRT NICHT]
Player (DK)          ── x,y,vx,vy,draw,anim          [DRAW-FUNKTION ERSETZEN]
Platform Entity      ── x,y,width,height,draw        [DRAW-FUNKTION ERWEITERN]
Banana Entity        ── x,y,collected,draw           [BERÜHRT NICHT]
Goomba Entity        ── x,y,vx,patrol,draw           [DRAW-FUNKTION ERSETZEN + NEUE GEGNER]
HUD Manager          ── Score, Leben, Restart        [ERWEITERN UM WELT-ANZEIGE]
Game State Manager   ── state, score, lives          [ERWEITERN UM WELT-ZUSTAND]
```

### 2.2 Was sich ändert (Übersicht)
| Modul | Aktion | Risiko |
|-------|--------|--------|
| `drawPlayer()` | **ERSETZEN** durch DK-Pixel-Art | Niedrig — Isolierte Funktion |
| `drawGoomba()` | **ERSETZEN** durch Barrel/Pauline | Niedrig — Isolierte Funktion |
| `drawPlatform()` | **ERWEITERN** um Welt-Theme | Niedrig — Switch-Case |
| `render()` | **ERWEITERN** um Hintergrund | Niedrig — Vor Entities zeichnen |
| `GameState` | **ERWEITERN** um `currentWorld` | Niedrig — Feld hinzufügen |
| `Input` | **ERWEITERN** um Welt-Auswahl | Niedrig — Nur im Menu-State |
| NEU: `WorldConfig` | **NEU** — Welt-Konfiguration | Kein Risiko |
| NEU: `WorldRenderer` | **NEU** — Hintergrund pro Welt | Kein Risiko |
| NEU: `MenuManager` | **NEU** — Welt-Auswahl-Screen | Kein Risiko |

---

## 3. Technische Architektur (Erweiterung)

### 3.1 Neue Module

```
┌──────────────────────────────────────────────────────────────┐
│                     Game (Main)                              │
│  + worldSelectState()   ── Menu vor Spielstart              │
│  + worldSelected(n)     ── Welt 1-3 auswählen               │
└──────────────────┬───────────────────────────────────────────┘
                   │
    ┌──────────────┼──────────────┬───────────────┬────────────┐
    ▼              ▼              ▼               ▼            ▼
┌─────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐ ┌──────────┐
│ World   │ │ World    │ │ Enhanced │ │ Enhanced   │ │ Menu     │
│ Config  │ │ Renderer │ │ DK Draw  │ │ Enemy Draw │ │ Manager  │
│         │ │          │ │          │ │            │ │          │
│ - themes│ │ - bg     │ │ - dkIdle │ │ - barrel   │ │ - show   │
│ - colors│ │ - parallax│ │ - dkWalk │ │ - barrel   │ │ - hide   │
│ - music │ │ - particles│ │ - dkJump │ │ - gorilla  │ │ - select │
│  arrays │ │ - theme  │ │ - detail │ │ - gorilla  │ │ - hover  │
│         │ │  switch  │ │  pixels  │ │  climb     │ │  effect  │
└─────────┘ └──────────┘ └──────────┘ └────────────┘ └──────────┘
```

### 3.2 Welt-Auswahl-Flow (State Machine)

```
  ┌─────────────┐
  │  MENU STATE │  ──► Welt-Auswahl-Screen (3 Karten)
  └──────┬──────┘
         │ Pfeiltasten + Enter / Klick
         │
    ┌────┴────┐
    │         │
   1/        2/        3/
 Dschungel  Schnee    Wüste
    │         │         │
    └────┬────┴────┬────┘
         │         │
         ▼         ▼
    ┌──────────────────┐
    │   PLAY STATE     │  ──► Bestehendes Spiel mit gewählter Welt
    │  (bestehend)     │
    └──────────────────┘
```

### 3.3 Datenmodell: WorldConfig

```javascript
// =====================================================
// NEU: WORLD CONFIGURATION
// =====================================================
const WorldConfig = {
  jungle: {
    id: 1,
    name: "Dschungel",
    // Farben
    colors: {
      sky: "#1a472a",           // Dunkelgrün
      skyGradient: "#2d5a3f",   // Helles Grün
      ground: "#3d2b1f",        // Dunkles Braun
      platform: "#8B6914",      // Holz (bestehend)
      platformHighlight: "#A0822A",
      platformShadow: "#6B5310",
      background: "#0d2818",    // Sehr dunkelgrün
      particle: "#4CAF50",      // Blatt-Grün
      accent: "#FFD700"         // Gold (Bananen)
    },
    // Hintergrund-Elemente
    background: {
      trees: true,           // Baum-Silhouetten
      vines: true,           // Hängende Lianen
      clouds: true,          // Wolken
      particles: "leaves",   // Fallende Blätter
      parallaxLayers: 2      // 2 Parallax-Schichten
    },
    // Gegner-Theme
    enemyTheme: "barrel",    // Fässer statt Goombas
    // Plattform-Theme
    platformTheme: "wood",   // Holzbalken (bestehend)
    // Welt-Layout (Plattformen, Bananen, Gegner)
    layout: {
      platforms: [
        { x: 0,     y: 360, width: 500 },
        { x: 400,   y: 280, width: 200 },
        { x: 100,   y: 200, width: 250 },
        { x: 450,   y: 160, width: 300 },
        { x: 850,   y: 240, width: 400 },
        { x: 1300,  y: 180, width: 200 },
      ],
      bananas: [
        { x: 50,  y: 340 }, { x: 150, y: 340 }, { x: 300, y: 340 },
        { x: 450, y: 260 }, { x: 550, y: 260 },
        { x: 150, y: 180 }, { x: 250, y: 180 },
        { x: 500, y: 140 }, { x: 650, y: 140 },
        { x: 950, y: 220 }, { x: 1050, y: 220 },
        { x: 1350, y: 160 },
      ],
      goombas: [
        { platformIndex: 2, x: 150, y: 176, speed: 1.5, patrolLeft: 100, patrolRight: 330 },
        { platformIndex: 4, x: 900, y: 216, speed: 2.0, patrolLeft: 850, patrolRight: 1230 },
        { platformIndex: 5, x: 1350, y: 156, speed: 1.0, patrolLeft: 1300, patrolRight: 1480 },
      ]
    }
  },

  snow: {
    id: 2,
    name: "Schnee",
    colors: {
      sky: "#4a6fa5",
      skyGradient: "#8ba4c7",
      ground: "#c8d6e5",
      platform: "#7f8fa6",
      platformHighlight: "#a4b0be",
      platformShadow: "#576574",
      background: "#2c3e50",
      particle: "#dfe6e9",      // Schneeflocken
      accent: "#74b9ff"         // Eis-Blau
    },
    background: {
      trees: false,
      vines: false,
      clouds: true,
      mountains: true,          // Berg-Silhouetten
      particles: "snow",        // Schneeflocken
      parallaxLayers: 3         // 3 Parallax-Schichten (Berge)
    },
    enemyTheme: "penguin",      // Pinguine statt Goombas
    platformTheme: "ice",       // Eis-Plattformen
    layout: {
      platforms: [
        { x: 0,     y: 360, width: 350 },
        { x: 300,   y: 300, width: 150 },
        { x: 500,   y: 240, width: 200 },
        { x: 200,   y: 180, width: 250 },
        { x: 550,   y: 140, width: 300 },
        { x: 950,   y: 220, width: 250 },
        { x: 1250,  y: 160, width: 200 },
      ],
      bananas: [
        { x: 50,  y: 340 }, { x: 150, y: 340 },
        { x: 350, y: 280 },
        { x: 550, y: 220 }, { x: 650, y: 220 },
        { x: 250, y: 160 }, { x: 350, y: 160 },
        { x: 600, y: 120 }, { x: 750, y: 120 },
        { x: 1000, y: 200 }, { x: 1100, y: 200 },
        { x: 1300, y: 140 },
      ],
      goombas: [
        { platformIndex: 2, x: 550, y: 216, speed: 1.8, patrolLeft: 500, patrolRight: 680 },
        { platformIndex: 4, x: 600, y: 116, speed: 2.2, patrolLeft: 550, patrolRight: 830 },
        { platformIndex: 6, x: 1300, y: 136, speed: 1.2, patrolLeft: 1250, patrolRight: 1430 },
      ]
    }
  },

  desert: {
    id: 3,
    name: "Wüste",
    colors: {
      sky: "#e17055",
      skyGradient: "#fdcb6e",
      ground: "#d4a574",
      platform: "#b8860b",
      platformHighlight: "#daa520",
      platformShadow: "#8b6914",
      background: "#6b3a2a",
      particle: "#fdcb6e",      // Sand-Partikel
      accent: "#e17055"         // Terrakotta
    },
    background: {
      trees: false,
      vines: false,
      clouds: false,
      dunes: true,               // Sanddünen
      pyramids: true,            // Pyramiden-Silhouette
      particles: "sand",         // Sand-Partikel (Wind)
      parallaxLayers: 2
    },
    enemyTheme: "scorpion",      // Skorpione statt Goombas
    platformTheme: "stone",      // Stein-Plattformen
    layout: {
      platforms: [
        { x: 0,     y: 360, width: 450 },
        { x: 400,   y: 300, width: 180 },
        { x: 650,   y: 240, width: 220 },
        { x: 150,   y: 180, width: 200 },
        { x: 500,   y: 140, width: 280 },
        { x: 900,   y: 200, width: 300 },
        { x: 1250,  y: 150, width: 200 },
      ],
      bananas: [
        { x: 50,  y: 340 }, { x: 200, y: 340 }, { x: 350, y: 340 },
        { x: 450, y: 280 },
        { x: 700, y: 220 }, { x: 800, y: 220 },
        { x: 200, y: 160 }, { x: 300, y: 160 },
        { x: 550, y: 120 }, { x: 700, y: 120 },
        { x: 950, y: 180 }, { x: 1100, y: 180 },
        { x: 1300, y: 130 },
      ],
      goombas: [
        { platformIndex: 2, x: 700, y: 216, speed: 2.0, patrolLeft: 650, patrolRight: 850 },
        { platformIndex: 4, x: 550, y: 116, speed: 1.5, patrolLeft: 500, patrolRight: 760 },
        { platformIndex: 5, x: 1000, y: 176, speed: 2.5, patrolLeft: 900, patrolRight: 1170 },
      ]
    }
  }
};
```

---

## 4. Detaillierte Modul-Spezifikationen

### 4.1 NEU: MenuManager (Welt-Auswahl-Screen)

**Verantwortlichkeit:** Zeigt den Welt-Auswahl-Screen vor Spielstart.

**Schnittstelle:**
```javascript
const MenuManager = {
  visible: false,
  selectedWorld: 0,           // 0=jungle, 1=snow, 2=desert

  show() { ... },              // Menu einblenden
  hide() { ... },              // Menu ausblenden
  selectLeft() { ... },        // Eine Welt nach links
  selectRight() { ... },       // Eine Welt nach rechts
  confirm() { ... },           // Enter: Gewählte Welt laden
  draw(ctx) { ... },           // Menu rendern
};
```

**Darstellung:**
```
┌─────────────────────────────────────────────────────┐
│                                                     │
│              DONKEY KONG BROWSER GAME               │
│                                                     │
│   ┌──────────┐  ┌──────────┐  ┌──────────┐        │
│   │  🌴      │  │  ❄️      │  │  🏜️      │        │
│   │ Dschungel │  │  Schnee  │  │  Wüste   │        │
│   │          │  │          │  │          │        │
│   │ [====]   │  │         │  │         │        │
│   └──────────┘  └──────────┘  └──────────┘        │
│                                                     │
│         ← → Auswählen    ENTER Starten             │
│                                                     │
└─────────────────────────────────────────────────────┘
```

**Welt-Karten:** Jede Karte ist 160x140px mit Mini-Vorschau der Welt-Farben.

---

### 4.2 NEU: WorldRenderer (Hintergrund pro Welt)

**Verantwortlichkeit:** Zeichnet den thematischen Hintergrund der aktuellen Welt.

**Schnittstelle:**
```javascript
const WorldRenderer = {
  particles: [],               // Aktuelle Partikel

  init(worldId) { ... },       // Partikel für Welt initialisieren
  update() { ... },            // Partikel-Positionen aktualisieren
  draw(ctx, worldId, camX) { ... },  // Hintergrund rendern
};
```

**Render-Reihenfolge (von hinten nach vorne):**
1. Himmel (Gradient)
2. Parallax-Schicht 1 (ferne Berge/Dünen/Bäume)
3. Parallax-Schicht 2 (nähere Elemente)
4. Partikel (Schnee/Blätter/Sand)
5. Vordergrund-Elemente (Lianen, Eisblöcke)

**Parallax-Formel:**
```javascript
// Schicht 1 (fern): 0.2x Kamera-Bewegung
const offset1 = camX * 0.2;
// Schicht 2 (nah): 0.5x Kamera-Bewegung
const offset2 = camX * 0.5;
```

---

### 4.3 ERSETZEN: drawPlayer() → Enhanced DK Pixel-Art

**Ziel:** Donkey Kong soll wie der klassische Arcade-DK aussehen:
- Großes braunes Affen-Gesicht
- Rotes Halsband mit "DK" Buchstaben
- Dicker Körper mit breiten Schultern
- Detaillierte Pixel-Art (32x32 bleibt, aber mehr Detail)

**Pixel-Art Map (32x32):**
```
Row  0: ..BBBBBBBBBBBBBBBB..........  (Kopfoberseite - Braun)
Row  1: .BBBBBBBBBBBBBBBBBBBBB......  (Kopf breit)
Row  2: .BBBFFFFFFBBBBBFFFFFFFF.....  (Ohren links/rechts)
Row  3: .BBBFFFFFFFFFFBFFFFFFFF.....  (Gesicht - Hautfarbe)
Row  4: .BBBFFFFFBBFFFFFBBFFFFF.....  (Augen)
Row  5: .BBBFFFFFFFFFFBFFFFFFFF.....  (Gesicht)
Row  6: .BBBBBBBFFFFBBBBFFFFFFF.....  (Mund/Nase)
Row  7: ..BBBBBBFFFFFFBBBBBB........  (Kinn)
Row  8: ...RRRRRRRRRRRRRRRRR.........  (Halsband - Rot)
Row  9: ...RWWWWWWWWWWWWWWWWW.......  (DK auf Halsband - Weiß)
Row 10: ..BBBBBBBBBBBBBBBBBBBBB......  (Schultern)
Row 11: .BBBBBBBBBBBBBBBBBBBBBBBB....  (Körper breit)
Row 12: .BBBBBBBBBBBBBBBBBBBBBBBB....  (Körper)
Row 13: .BBBBBBBBBBBBBBBBBBBBBBBB....  (Körper)
Row 14: .BBBBBBBBBBBBBBBBBBBBBBBB....  (Körper)
Row 15: .BBBBBBBBBBBBBBBBBBBBBBBB....  (Körper)
Row 16: ..BBBBBBBBBBBBBBBBBBBBB......  (Hüfte)
Row 17: ...DDDDDD..........DDDDDD....  (Beine - Dunkelbraun)
Row 18: ...DDDDDD..........DDDDDD....  (Beine)
Row 19: ...DDDDD...........DDDDD....  (Beine)
Row 20: ...SSSS............SSSS.....  (Schuhe - Rot)
Row 21: ...SSSSS...........SSSSS....  (Schuhsohlen)
```

**Legende:**
- `B` = Braun (#8B4513) — Körper, Kopf
- `F` = Hautfarbe (#DEB887) — Gesicht
- `R` = Rot (#cc0000) — Halsband
- `W` = Weiß (#ffffff) — "DK" auf Halsband
- `D` = Dunkelbraun (#5C3317) — Beine
- `S` = Rot (#cc0000) — Schuhe
- `.` = Transparent (Hintergrund)

**Animationen:**
| Zustand | Beschreibung |
|---------|-------------|
| `idle` | Arme leicht hin und her (2-Frame Toggle) |
| `walk` | Beine abwechselnd, Arke schwingen (4-Frame) |
| `jump` | Arme nach oben, Beine angewinkelt (1 Frame) |
| `climb` | Für Zukunft: Leiter klettern |

---

### 4.4 ERSETZEN: drawGoomba() → Thematic Enemies

**3 Gegner-Typen pro Welt:**

#### 4.4.1 Dschungel: Barrel (Fass)
```
Row 0: ..OOOOOOOOOOOOOOOOOOOOOO..    (Fass-Oben - Braun)
Row 1: .OOOOOOOOOOOOOOOOOOOOOOOO.    (Fass breit)
Row 2: .OOXXXOOOOOOOOXXXOOOOOOOO.    (Metall-Band)
Row 3: .OOOOOOOOOOOOOOOOOOOOOOOO.    (Fass-Körper)
Row 4: .OOOOOOOOOOOOOOOOOOOOOOOO.
Row 5: .OOOOOOOOOOOOOOOOOOOOOOOO.
Row 6: .OOXXXOOOOOOOOXXXOOOOOOOO.    (Metall-Band)
Row 7: .OOOOOOOOOOOOOOOOOOOOOOOO.
Row 8: ..OOOOOOOOOOOOOOOOOOOOOO..    (Fass-Unten)
```
- `O` = Braun (#8B4513) — Fass
- `X` = Grau (#888888) — Metall-Bänder
- **Bewegung:** Rollt (Rotation durch Animation)
- **Mechanik:** Wie bestehender Goomba (Patrouille)

#### 4.4.2 Schnee: Penguin (Pinguin)
```
Row 0: ..WWWWWWWWWWWWWWWWWWOO..    (Kopf - Weiß/Schwarz)
Row 1: .WWWWWWWWWWWWWWWWWWWWWW.    (Kopf breit)
Row 2: .WWWWBBWWWWWWWWWWBBWWWW.    (Augen - Schwarz)
Row 3: .WWWWWWWWWWWWWWWWWWWWWW.
Row 4: .WWWOOOOOOOOOOOOOOOOOWWW.    (Schnabel - Orange)
Row 5: ..WWWWWWWWWWWWWWWWWWWW..    (Hals)
Row 6: ..OOOOOOOOOOOOOOOOOOOO..    (Körper - Schwarz)
Row 7: .OOOOOOOOOOOOOOOOOOOOOO.
Row 8: .OOOOOOOOOOOOOOOOOOOOOO.
Row 9: .OOOOOOOOOOOOOOOOOOOOOO.
Row 10:.OOOOOOOOOOOOOOOOOOOOOO.
Row 11:..OOOOOOOOOOOOOOOOOOOO..
Row 12: ...OOOO..........OOOO.    (Füße - Orange)
Row 13: ...OOOO..........OOOO.
```
- **Bewegung:** Watschelt (Walk-Animation)
- **Mechanik:** Wie bestehender Goomba

#### 4.4.3 Wüste: Scorpion (Skorpion)
```
Row 0: ..YYYYYY..........YYYYYY..    (Schwänze hoch)
Row 1: .YYYYYY............YYYYYY.    (Schwänze)
Row 2: .YYYY................YYYY.    (Stachel)
Row 3: ..RRRRRRRRRRRRRRRRRRRR..    (Körper - Rotbraun)
Row 4: .RRRRBBRRRRRRRRRRBBRRRR.    (Augen)
Row 5: .RRRRRRRRRRRRRRRRRRRRRR.    (Körper)
Row 6: .RRRRRRRRRRRRRRRRRRRRRR.
Row 7: ..RRRRRRRRRRRRRRRRRRRR..
Row 8: ...RRRRR..........RRRRR.    (Beine)
Row 9: ....RRRR..........RRRR..    (Beine)
```
- `Y` = Gelb (#daa520) — Schwanz/Stachel
- `R` = Rotbraun (#8B4513) — Körper
- `B` = Schwarz (#000) — Augen
- **Bewegung:** Kriecht (Beine-Animation)
- **Mechanik:** Wie bestehender Goomba

---

### 4.5 ERWEITERN: drawPlatform() → Thematic Platforms

**3 Plattform-Themes:**

#### 4.5.1 Dschungel: Wood (Holz — bestehend)
Bestehender Code bleibt unverändert.

#### 4.5.2 Schnee: Ice (Eis)
```javascript
// Eis-Plattform
ctx.fillStyle = '#a4d8f0';      // Eis-Blau
ctx.fillRect(drawX, p.y, p.width, p.height);

// Glitzer-Effekt (weiße Reflexionen)
ctx.fillStyle = '#ffffff';
ctx.fillRect(drawX + 5, p.y + 2, 15, 3);
ctx.fillRect(drawX + 40, p.y + 1, 10, 2);

// Untere Schattierung (dunkleres Eis)
ctx.fillStyle = '#74b9ff';
ctx.fillRect(drawX, p.y + p.height - 3, p.width, 3);

// Eis-Kristalle an den Rändern
ctx.fillStyle = '#dfe6e9';
ctx.fillRect(drawX, p.y - 4, 4, 4);
ctx.fillRect(drawX + p.width - 4, p.y - 3, 4, 3);
```

#### 4.5.3 Wüste: Stone (Stein)
```javascript
// Stein-Plattform
ctx.fillStyle = '#b8860b';      // Dunkelgold
ctx.fillRect(drawX, p.y, p.width, p.height);

// Stein-Textur (zufällige dunkle Flecken)
ctx.fillStyle = '#8b6914';
for (let sx = drawX + 8; sx < drawX + p.width - 8; sx += 20) {
  ctx.fillRect(sx, p.y + 3, 6, 6);
}

// Obere Kante (hellere Sand-Textur)
ctx.fillStyle = '#daa520';
ctx.fillRect(drawX, p.y, p.width, 3);

// Untere Schattierung
ctx.fillStyle = '#6b4423';
ctx.fillRect(drawX, p.y + p.height - 3, p.width, 3);
```

---

### 4.6 ERWEITERN: GameState um currentWorld

```javascript
const GameState = {
  state: 'menu',              // NEU: 'menu' | 'play' | 'gameover'
  score: 0,
  lives: 3,
  bananasTotal: 10,
  bananasCollected: 0,
  currentWorld: 0,            // NEU: 0=jungle, 1=snow, 2=desert
};
```

---

### 4.7 ERWEITERN: Input Manager

```javascript
// NEU: Menu-Navigation
const Input = {
  keys: {
    left: false,
    right: false,
    jump: false,
    confirm: false,            // NEU: Enter-Taste
  },

  // ... bestehender Code ...

  onKeyDown(e) {
    switch (e.code) {
      // ... bestehende Fälle ...
      case 'Enter':
        this.keys.confirm = true;
        e.preventDefault();
        break;
    }
  },

  onKeyUp(e) {
    switch (e.code) {
      // ... bestehende Fälle ...
      case 'Enter':
        this.keys.confirm = false;
        break;
    }
  }
};
```

---

## 5. Implementierungsplan (Phasen)

> **PRINZIP:** Jede Phase baut auf der vorherigen auf. Nach jeder Phase ist das Spiel
> weiterhin spielbar und testbar. **Kein Big-Bang-Release.**

### Phase 1: WorldConfig & GameState-Erweiterung
**Ziel:** Datenbasis für 3 Welten + GameState-Erweiterung

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 1.1 | `WorldConfig`-Objekt einfügen (nach §1 CONSTANTS) | NEU nach Zeile 218 | 30 Min |
| 1.2 | `GameState.currentWorld` hinzufügen | Zeile 223-229 | 5 Min |
| 1.3 | `GameState.state` um `'menu'` erweitern | Zeile 224 | 5 Min |
| 1.4 | `getWorldConfig()` Helper-Funktion | NEU nach Zeile 218 | 10 Min |

**Ergebnis:** Welt-Konfiguration existiert, GameState ist vorbereitet. Spiel läuft weiterhin unverändert.

---

### Phase 2: Enhanced DK Pixel-Art
**Ziel:** Donkey Kong Figur mit Halsband und mehr Detail

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 2.1 | `drawPlayer()` komplett ersetzen | Zeile 351-424 | 45 Min |
| 2.2 | Pixel-Art Map implementieren (Idle) | NEU | 30 Min |
| 2.3 | Walk-Animation (4-Frame Beine+Arme) | NEU | 20 Min |
| 2.4 | Jump-Animation (Arme hoch) | NEU | 15 Min |
| 2.5 | Halsband mit "DK" zeichnen | NEU | 10 Min |

**Ergebnis:** DK sieht wie klassischer Arcade-DK aus mit Halsband und Animationen.

---

### Phase 3: Thematic Enemies
**Ziel:** 3 Gegner-Typen (Barrel, Penguin, Scorpion)

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 3.1 | `drawEnemy()` als neue Funktion (ersetzt `drawGoomba`) | NEU nach Zeile 515 | 40 Min |
| 3.2 | Barrel-Zeichenroutine (Dschungel) | NEU | 20 Min |
| 3.3 | Penguin-Zeichenroutine (Schnee) | NEU | 20 Min |
| 3.4 | Scorpion-Zeichenroutine (Wüste) | NEU | 20 Min |
| 3.5 | `render()` anpassen: `drawGoomba` → `drawEnemy` mit `enemyTheme` | Zeile 787-789 | 10 Min |

**Ergebnis:** Gegner passen zum Welt-Theme.

---

### Phase 4: Thematic Platforms & Backgrounds
**Ziel:** Plattformen und Hintergrund pro Welt

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 4.1 | `drawPlatform()` um `platformTheme` erweitern (Switch) | Zeile 426-446 | 25 Min |
| 4.2 | Eis-Plattform (Schnee) | NEU | 15 Min |
| 4.3 | Stein-Plattform (Wüste) | NEU | 15 Min |
| 4.4 | `WorldRenderer.draw()` Hintergrund-Funktion | NEU | 45 Min |
| 4.5 | Dschungel-Hintergrund (Bäume, Lianen) | NEU | 20 Min |
| 4.6 | Schnee-Hintergrund (Berge, Eis) | NEU | 20 Min |
| 4.7 | Wüste-Hintergrund (Dünen, Pyramiden) | NEU | 20 Min |
| 4.8 | Partikel-System (Schnee/Blätter/Sand) | NEU | 30 Min |
| 4.9 | `render()` anpassen: Hintergrund vor Entities zeichnen | Zeile 767-795 | 10 Min |

**Ergebnis:** Jede Welt hat einzigartige Plattformen, Hintergründe und Partikel.

---

### Phase 5: World Selection Menu
**Ziel:** Welt-Auswahl-Screen vor Spielstart

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 5.1 | `MenuManager` einfügen | NEU nach Zeile 639 | 30 Min |
| 5.2 | Menu-Darstellung (3 Welt-Karten) | NEU | 30 Min |
| 5.3 | Input: Pfeiltasten + Enter für Auswahl | Zeile 260-308 | 15 Min |
| 5.4 | `Game.init()`: Mit `state='menu'` starten | Zeile 806-818 | 10 Min |
| 5.5 | `Game.update()`: Menu-Logik vor Spiel-Logik | Zeile 674-765 | 20 Min |
| 5.6 | `Game.render()`: Menu oder Spiel rendern | Zeile 767-795 | 10 Min |
| 5.7 | `Game.reset()`: Welt-Konfiguration laden | Zeile 662-672 | 10 Min |

**Ergebnis:** Spieler wählt Welt vor Spielstart.

---

### Phase 6: Welt-Layout & Polish
**Ziel:** Welt-spezifische Level-Layouts + Feinschliff

| Schritt | Beschreibung | Datei-Bereich | Aufwand |
|---------|-------------|---------------|---------|
| 6.1 | `initEntities()`: Welt-Konfiguration laden | Zeile 644-648 | 20 Min |
| 6.2 | Dschungel-Layout testen & anpassen | WorldConfig | 15 Min |
| 6.3 | Schnee-Layout testen & anpassen | WorldConfig | 15 Min |
| 6.4 | Wüste-Layout testen & anpassen | WorldConfig | 15 Min |
| 6.5 | HUD: Welt-Name anzeigen | Zeile 621-624 | 10 Min |
| 6.6 | Game Over: "Welt X" im Score | Zeile 630-634 | 5 Min |
| 6.7 | Farben/Tuning aller Welten | — | 30 Min |

**Ergebnis:** 3 voll spielbare, balancierte Welten.

---

## 6. Code-Struktur (Nach der Implementierung)

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <style>
    /* === CSS: Retro Pixel-Stil === */
    /* - BESTEHEND: Canvas, HUD, Game Over */
    /* + NEU: Menu-Overlay Styles */
  </style>
</head>
<body>
  <div id="game-container">
    <canvas id="game" width="800" height="400"></canvas>
    <div id="hud"> ... </div>
    <div id="game-over"> ... </div>
    <!-- NEU: Menu Overlay -->
    <div id="world-menu"> ... </div>
  </div>

  <script>
    // =====================================================
    // 1. CONSTANTS & CONFIG
    // =====================================================
    // - BESTEHEND: Canvas, Physics, Entity sizes
    // + NEU: WorldConfig (3 Welten)

    // =====================================================
    // 2. GAME STATE
    // =====================================================
    // - BESTEHEND: GameState, Player, Camera
    // + NEU: currentWorld in GameState
    // + NEU: state='menu'

    // =====================================================
    // 3. INPUT MANAGER
    // =====================================================
    // - BESTEHEND: Pfeiltasten, Space
    // + NEU: Enter-Taste (confirm)

    // =====================================================
    // 4. ENTITY FACTORY + DRAWING
    // =====================================================
    // - BESTEHEND: createPlatforms, createBananas, createGoombas
    // + NEU: drawPlayer() — Enhanced DK (ERSETZT bestehend)
    // + NEU: drawEnemy() — Thematic (ERSETZT drawGoomba)
    // + NEU: drawPlatform() — Thematic (ERWEITERT bestehend)
    // + NEU: WorldRenderer

    // =====================================================
    // 5. PHYSICS ENGINE
    // =====================================================
    // - BESTEHEND: Unverändert!

    // =====================================================
    // 6. CAMERA MANAGER
    // =====================================================
    // - BESTEHEND: Unverändert!

    // =====================================================
    // 7. HUD MANAGER
    // =====================================================
    // - BESTEHEND: Score, Leben
    // + NEU: Welt-Name anzeigen

    // =====================================================
    // 8. GAME OVER MANAGER
    // =====================================================
    // - BESTEHEND: Unverändert!

    // =====================================================
    // 8.5 MENU MANAGER (NEU)
    // =====================================================
    // + NEU: Welt-Auswahl-Screen

    // =====================================================
    // 9. GAME (MAIN)
    // =====================================================
    // - BESTEHEND: init, reset, update, render, gameLoop
    // + NEU: Menu-Logik in update/render
    // + NEU: Welt-Konfiguration in reset/initEntities

    // =====================================================
    // 10. BOOTSTRAP
    // =====================================================
    // - BESTEHEND: init() on DOMContentLoaded
  </script>
</body>
</html>
```

---

## 7. Abhängigkeitsgraph (Phasen)

```
Phase 1 (Datenbasis)
  └── WorldConfig + GameState-Erweiterung
        │
        ▼
Phase 2 (DK Charakter)     Phase 3 (Gegner)
  │                           │
  └── drawPlayer()            └── drawEnemy()
        │                           │
        └───────────┬───────────────┘
                    │
                    ▼
Phase 4 (Welten-Visual)
  ├── Plattformen
  ├── Hintergründe
  └── Partikel
        │
        ▼
Phase 5 (Menu)
  └── Welt-Auswahl-Screen
        │
        ▼
Phase 6 (Layout & Polish)
  ├── Welt-Layouts
  └── Balancing
```

---

## 8. Risiken & Massnahmen

| Risiko | Einschätzung | Massnahme |
|--------|-------------|-----------|
| **Single-File wird sehr groß** | Mittel | WorldConfig ist kompakt; Pixel-Art als fillRect bleibt effizient |
| **Pixel-Art ist zeitaufwändig** | Mittel | 32x32 Grid bleibt; Map-basierter Ansatz beschleunigt Implementierung |
| **Parallax-Performance** | Niedrig | Nur 2-3 Schichten mit einfacher Multiplikation; kein Canvas-Offscreen |
| **Partikel-System friert ein** | Niedrig | Max 30 Partikel pro Welt; einfaches Array mit update/draw |
| **Menu-Input kollidiert mit Spiel-Input** | Niedrig | State-Maschine: Im Menu-State werden Spiel-Inputs ignoriert |
| **Bestehende Tests brechen** | Mittel | drawPlayer/drawGoomba ändern sich; Tests auf Visual prüfen |

---

## 9. Definition of Done (pro Phase)

Jede Phase gilt als erledigt, wenn:
- [ ] Alle Schritte der Phase sind implementiert
- [ ] Das Spiel läuft im Browser ohne Fehler (Console frei von Errors)
- [ ] Keine Regressionen in zuvor implementierten Phasen
- [ ] Alle 3 Welten sind spielbar (ab Phase 4)
- [ ] Welt-Auswahl funktioniert (ab Phase 5)

---

## 10. Glossar (Erweiterung)

| Begriff | Bedeutung |
|---------|-----------|
| WorldConfig | Konfigurationsobjekt für alle 3 Welten |
| WorldRenderer | Zeichnet thematische Hintergründe |
| MenuManager | Welt-Auswahl-Screen vor Spielstart |
| Parallax | Mehrlagiger Hintergrund mit unterschiedlicher Scroll-Geschwindigkeit |
| Particle System | Kleine animierte Elemente (Schnee, Blätter, Sand) |
| enemyTheme | Gegner-Typ pro Welt (barrel, penguin, scorpion) |
| platformTheme | Plattform-Stil pro Welt (wood, ice, stone) |

---

*Architektur-Erweiterung erstellt am 13.06.2026 als Nachfolger von ARCHITEKTUR.md (Sprint 1).*