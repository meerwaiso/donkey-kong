# Donkey Kong - Testplan

## 1. Einführung

### 1.1 Zweck
Dieser Testplan beschreibt die Strategie, den Umfang, die Ressourcen und den Zeitplan für das Testing des Donkey Kong Arcade-Spiels.

### 1.2 Produktübersicht
Donkey Kong ist ein klassisches Arcade-Plattformspiel mit HTML5 Canvas, bei dem der Spieler (DK) Bananen einsammelt, Fässern ausweicht und Level abschließt.

### 1.3 Referenzen
- Architektur-Dokument: `/ARCHITEKTUR.md`
- Quellcode: `/index.html`
- Jira Board: SCRUM (SCRUM-101 bis SCRUM-138)

---

## 2. Testumfang

### 2.1 Im Testumfang enthalten
- Spiel-Initialisierung und Startbildschirm
- Spieler-Steuerung (Bewegung, Sprung, Richtung)
- Physik-Engine (Kollision, Schwerkraft, One-Way-Plattformen)
- Kamera-System (Follow, Weltgrenzen)
- Plattform-Rendering
- Bananen & Punktestand
- Gegner (Goombas, Donkey Kong)
- Fässer-System
- Leben-System & Invulnerabilität
- Game Over & Neustart
- Responsive Design (Desktop & Mobile)

### 2.2 Nicht im Testumfang enthalten
- Netzwerk-Performance
- Cross-Browser-Kompatibilität (außer Chromium)
- Barrierefreiheit (Accessibility)
- Last- und Performance-Testing

---

## 3. Teststrategie

### 3.1 Test-Level
| Test-Level | Beschreibung | Automatisierung |
|------------|-------------|-----------------|
| Unit-Tests | Einzelne Funktionen (Physics, Input) | ✅ Playwright |
| Integrations-Tests | Zusammenspiel der Systeme | ✅ Playwright |
| End-to-End-Tests | Vollständige Spielabläufe | ✅ Playwright |

### 3.2 Test-Typen
| Test-Typ | Beschreibung |
|----------|-------------|
| Funktionale Tests | Verifikation aller Spielmechaniken |
| UI-Tests | Rendering, HUD, Overlays |
| Responsive Tests | Desktop (800x600) und Mobile (iPhone 13) |

### 3.3 Test-Werkzeuge
- **Framework:** Playwright Test
- **Browser:** Chromium (headless)
- **Sprache:** TypeScript
- **CI/CD:** Manuell / npm-Skripte

---

## 4. Testfälle

### Suite 1: Spiel-Initialisierung (TC-01 bis TC-03)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-01 | Canvas existiert und Spiel initialisiert | Canvas sichtbar | SCRUM-101 |
| TC-02 | Start-Screen sichtbar bei Laden | #start-screen sichtbar | SCRUM-102 |
| TC-03 | HUD mit korrekten Werten | Score: 0, Lives: 3, Level: 1 | SCRUM-103 |

### Suite 2: Spiel-Start (TC-04 bis TC-05)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-04 | Spiel startet mit Enter | gameStarted === true | SCRUM-104 |
| TC-05 | Spiel startet nicht ohne Enter | gameStarted === false | SCRUM-105 |

### Suite 3: Physik-Engine (TC-06 bis TC-10)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-06 | Schwerkraft wirkt auf DK | y-Position nimmt zu | SCRUM-106 |
| TC-07 | Top-Collision mit Plattform | DK steht auf Plattform | SCRUM-107 |
| TC-08 | DK fällt durch Plattform von unten | y-Position < Plattform.y | SCRUM-108 |
| TC-09 | DK kann von unten durch Plattformen springen | y nach Space < y vor Space | SCRUM-109 |
| TC-10 | Side-Collision stoppt DK | vx === 0 nach Kollision | SCRUM-110 |

### Suite 4: Kamera (TC-11 bis TC-12)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-11 | Kamera folgt DK horizontal | Camera.x ändert sich | SCRUM-111 |
| TC-12 | Kamera bleibt in Weltgrenzen | 0 <= Camera.x <= 800 | SCRUM-112 |

### Suite 5: Spieler-Charakter (TC-13 bis TC-15)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-13 | DK wird als Pixel-Art Affe gerendert | visible: true, 32x32 | SCRUM-113 |
| TC-14 | Walk-Animation bei Bewegung | animFrame ändert sich | SCRUM-114 |
| TC-15 | DK schaut in Bewegungsrichtung | facing: left/right | SCRUM-115 |

### Suite 6: Plattformen (TC-16 bis TC-17)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-16 | 5 Plattformen werden gerendert | platforms.length === 5 | SCRUM-116 |
| TC-17 | Plattformen haben korrektes Design | height: 16, width > 0 | SCRUM-117 |

### Suite 7: Bananen & Punktestand (TC-18 bis TC-21)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-18 | 10 Bananen auf Plattformen verteilt | bananas.length === 10 | SCRUM-118 |
| TC-19 | Banane einsammeln - +1 Punkt | scoreAfter > scoreBefore | SCRUM-119 |
| TC-20 | Punktestand oben links korrekt | Text enthält "Score:" | SCRUM-120 |
| TC-21 | Einsammelte Bananen erscheinen nicht wieder | collected > 0 | SCRUM-121 |

### Suite 8: Goomba-Gegner (TC-22 bis TC-25)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-22 | 2 Goombas werden gerendert | goombas.length === 2 | SCRUM-122 |
| TC-23 | Goombas bewegen sich hin und her | x ändert sich | SCRUM-123 |
| TC-24 | Richtungsumkehr bei Plattformende | vx wechselt Vorzeichen | SCRUM-124 |
| TC-25 | Goombas haben Walk-Animation | animFrame ändert sich | SCRUM-125 |

### Suite 9: Leben-System (TC-26 bis TC-29)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-26 | 3 Leben bei Spielstart | lives === 3 | SCRUM-126 |
| TC-27 | -1 Leben bei Goomba-Berührung | lives <= 3 | SCRUM-127 |
| TC-28 | DK blinkt während Invulnerabilität | invulnerable > 0 | SCRUM-128 |
| TC-29 | Respawn am Start nach Treffer | x === 50, y === 300 | SCRUM-129 |

### Suite 10: Game Over (TC-30 bis TC-32)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-30 | Game Over Overlay bei 0 Leben | #game-over sichtbar | SCRUM-130 |
| TC-31 | Neustart-Button klickbar | lives === 3 nach Neustart | SCRUM-131 |
| TC-32 | Kein Input nach Game Over | player.x === 50 | SCRUM-132 |

### Suite 11: Pause-Funktion (TC-33 bis TC-35)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-33 | P-Taste aktiviert Pause | #pause-overlay sichtbar | SCRUM-133 |
| TC-34 | P-Taste deaktiviert Pause | #pause-overlay nicht sichtbar | SCRUM-134 |
| TC-35 | Keine Entity-Updates in Pause | player.x === 50 | SCRUM-135 |

### Suite 12: Responsive Design (TC-36 bis TC-38)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-36 | Canvas 800x600 auf Desktop | width: 800, height: 600 | SCRUM-136 |
| TC-37 | Canvas skaliert auf Mobile | width < 800 | SCRUM-137 |
| TC-38 | Touch-Controls sichtbar auf Mobile | touch-controls sichtbar | SCRUM-138 |

---

## 4.1 Sprint 2: Charaktere & 3 Welten

### Suite 13: World Selection Menu (TC-39 bis TC-43)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-39 | Menu wird bei Spielstart angezeigt | Menu-Overlay sichtbar, 3 Welt-Karten | SCRUM-139 |
| TC-40 | Pfeiltasten navigieren zwischen Welten | Highlight wechselt links/rechts | SCRUM-140 |
| TC-41 | Enter startet ausgewählte Welt | Spiel startet mit gewählter Welt | SCRUM-141 |
| TC-42 | 3 Welt-Karten mit korrekten Labels | "Dschungel", "Schnee", "Wüste" | SCRUM-142 |
| TC-43 | Default-Auswahl ist Dschungel (Welt 1) | selectedWorld === 0 | SCRUM-143 |

### Suite 14: 3 Welten (TC-44 bis TC-52)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-44 | Dschungel-Welt mit grüner Farbpalette | sky: #1a472a, ground: #3d2b1f | SCRUM-144 |
| TC-45 | Schnee-Welt mit blauer Farbpalette | sky: #4a6fa5, ground: #c8d6e5 | SCRUM-145 |
| TC-46 | Wüste-Welt mit goldener Farbpalette | sky: #e17055, ground: #d4a574 | SCRUM-146 |
| TC-47 | Dschungel-Hintergrund (Bäume, Lianen) | trees: true, vines: true | SCRUM-147 |
| TC-48 | Schnee-Hintergrund (Berge, Schneeflocken) | mountains: true, particles: "snow" | SCRUM-148 |
| TC-49 | Wüste-Hintergrund (Dünen, Pyramiden) | dunes: true, pyramids: true | SCRUM-149 |
| TC-50 | Partikel-System pro Welt aktiv | Partikel werden gerendert | SCRUM-150 |
| TC-51 | Welt-Layout korrekt (Plattformen, Bananen, Gegner) | Plattformen nach WorldConfig | SCRUM-151 |
| TC-52 | Parallax-Hintergrund (mehrere Schichten) | 2-3 Parallax-Layers | SCRUM-152 |

### Suite 15: Thematic Enemies (TC-53 bis TC-55)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-53 | Barrel-Gegner in Dschungel-Welt | enemyTheme: "barrel" | SCRUM-153 |
| TC-54 | Penguin-Gegner in Schnee-Welt | enemyTheme: "penguin" | SCRUM-154 |
| TC-55 | Scorpion-Gegner in Wüste-Welt | enemyTheme: "scorpion" | SCRUM-155 |

### Suite 16: Thematic Platforms (TC-56 bis TC-58)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-56 | Holz-Plattformen in Dschungel | platformTheme: "wood" | SCRUM-156 |
| TC-57 | Eis-Plattformen in Schnee | platformTheme: "ice" | SCRUM-157 |
| TC-58 | Stein-Plattformen in Wüste | platformTheme: "stone" | SCRUM-158 |

### Suite 17: Enhanced DK Character (TC-59 bis TC-61)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-59 | DK mit Halsband und "DK" gerendert | Halsband sichtbar, "DK" Text | SCRUM-159 |
| TC-60 | DK Walk-Animation (4-Frame) | animFrame ändert sich bei Bewegung | SCRUM-160 |
| TC-61 | DK Jump-Animation (Arme hoch) | Jump-Frame bei vy < 0 | SCRUM-161 |

### Suite 18: HUD Welt-Anzeige (TC-62 bis TC-63)
| ID | Beschreibung | Erwartetes Ergebnis | Jira |
|----|-------------|-------------------|------|
| TC-62 | Welt-Name im HUD angezeigt | Text enthält Welt-Name | SCRUM-162 |
| TC-63 | Game Over zeigt Welt-Name im Score | Score-Text enthält Welt | SCRUM-163 |

---

## 5. Testergebnis

### 5.1 Zusammenfassung (Sprint 1)
| Metrik | Wert |
|--------|------|
| Gesamtzahl Testfälle | 38 |
| Bestanden | 38 ✅ |
| Fehlgeschlagen | 0 ❌ |
| Übersprungen | 0 ⏭️ |
| Erfolgsrate | 100% |

### 5.2 Zusammenfassung (Sprint 2)
| Metrik | Wert |
|--------|------|
| Gesamtzahl Testfälle | 63 |
| Bestanden | 32 ✅ |
| Fehlgeschlagen | 31 ❌ |
| Übersprungen | 0 ⏭️ |
| Erfolgsrate | 50,8% |

### 5.3 Testlauf-Details
- **Datum:** 2026-06-13
- **Browser:** Chromium (headless)
- **Dauer:** ~20 Sekunden
- **Framework:** Playwright Test

### 5.4 Jira-Issues
- **Sprint 1:** 38 Testfälle als Jira-Issues erstellt (SCRUM-101 bis SCRUM-138) und mit Test-Ergebnissen kommentiert
- **Sprint 2:** 25 Testfälle als Jira-Issues erstellt (SCRUM-147 bis SCRUM-171) und mit Test-Automatisierungskommentaren versehen

---

## 6. Risiken & offene Punkte

### 6.1 Risiken
| Risiko | Auswirkung | Wahrscheinlichkeit | Maßnahme |
|--------|-----------|-------------------|----------|
| Canvas-basierte Tests | Rendering-Tests unzuverlässig | Mittel | evaluate() für State-Checks |
| Timing-sensitive Tests | Race Conditions | Mittel | Wartezeiten eingebaut |
| Mobile-Emulation | Touch-Events anders als echt | Hoch | iPhone 13 Preset verwendet |

### 6.2 Offene Punkte
- Cross-Browser-Testing (Firefox, Safari)
- Performance-Metriken (FPS, Ladezeit)
- Accessibility-Tests

---

## 7. Anhang

### 7.1 Ausführen der Tests
```bash
cd /home/fino/Desktop/SoftwareProjects/Donkey\ Kong/tests/donkeykong
npx playwright test
```

### 7.2 Projektstruktur
```
tests/donkeykong/
├── playwright.config.ts    # Playwright-Konfiguration
├── dk-tests.spec.ts        # 63 Testfälle in 20 Suiten
├── TESTPLAN.md             # Dieser Testplan
├── TESTBERICHT.html        # Automatischer Testbericht
├── package.json            # npm-Konfiguration
└── node_modules/           # Abhängigkeiten
