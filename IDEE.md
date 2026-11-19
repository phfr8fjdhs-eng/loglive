# Idee: «Nahansicht» – die Karte wird zur Baustelle und zum Schlachtfeld

**Eine Überlegung, wie Grenzland (Repo `lllkkkopa`) realistischer wird:** Beim Hineinzoomen kippt die flache Karte in eine **3D-Schrägansicht**. Städte wachsen sichtbar zu Skylines, die Truppen stehen als **Panzer** an der Front, und jeder Angriff ist ein Panzerkeil, der Land aufreisst. Es ändert sich keine Regel: Die Ansicht liest nur den Spielstand und zeichnet ihn anders.

Spielbarer Prototyp: [`prototyp/index.html`](prototyp/index.html) (eine Datei, ohne Bibliothek, im Browser öffnen, am besten quer).

## Warum das hilft

Heute ist alles Farbe und Zahl: Eine Stadt auf Stufe 80 sieht aus wie eine auf Stufe 1, 150K Truppen sind eine Zahl im Menü. Die Nahansicht gibt jeder Zahl einen Körper:

| Gefühl | Was man sieht |
| --- | --- |
| **Aufbauen** | Jede Stadtstufe lässt die Stadt sichtbar wachsen: Hütte, Block, Turm, Hochhausgruppe. Beim Ausbau steht ein Baukran darauf, bis die Stufe fertig ist. Ein frisch erobertes Gebiet beginnt als Baustelle und gräbt sich ein (Sandsäcke, Gräben). |
| **Krieg** | Truppen sind Panzer. Dichte Zellen im Netz sind Kolonnen, Reserven rollen als Konvoi nach hinten durch, auf Schienen fahren sie auf Zügen. Ein Stoss ist ein Keil aus Panzern mit Mündungsfeuer, Kratern, Rauch und brennenden Gebäuden. |
| **Schneller wachsen als die anderen** | Die Skyline ist die Rangliste. Wer an Stufen und Einkommen vorne liegt, sieht man an der Höhe der Türme, nicht erst in einer Tabelle. Zusätzlich ein Balken «Wachstumstempo» (Einkommen du gegen Gegner), damit man spürt, ob man führt oder hinterherrennt. |

Das passt zu den Leitsätzen in `doku/vision.md`:
- **Wo zählt:** man sieht, wo die Panzer stehen und wo die Front dünn ist.
- **Wissen ist eine Ressource:** Die Nahansicht zeigt nur, was der Spieler sieht. Versteckte Gebäude (Tarnturm, `hiddenFrom`) und Netz des Gegners bleiben unsichtbar.
- **Wenige Gesten:** keine Einzeleinheiten zum Schieben. Die Panzer sind reine Darstellung der Zahlen, die es schon gibt.

## Vom Spielstand zum Bild

| Spielstand (heute) | Darstellung in der Nahansicht |
| --- | --- |
| Gebäudestufe (1–100, `G.MAX_LEVEL`) | Höhe und Anzahl der Türme, logarithmisch: Stufe 1 Hütte, 5 Block, 20 Turm, 50+ Hochhausgruppe |
| Bauzeit beim Ausbau (`G.BUILD_TICKS`) | Baukran und Gerüst, die Stufe wächst am Ende hoch |
| Truppen pro Zelle im Netz (`03b_posture`, Zellen 16 × 16 Felder, Dichte) | Anzahl Panzer je Zelle (ein Panzer pro fester Truppenmenge, pro Zelle begrenzt) |
| Marsch von Reserven, Alarm | Konvoi, der von hinten zur angegriffenen Zelle rollt |
| Eingraben (`c.e`) | Sandsäcke und Gräben an der Grenze, je tiefer, desto mehr |
| Stoss / Keil (`G.THRUST`, höchstens 4) | Panzerkeil entlang des Korridors, Pfeil bleibt als Orientierung |
| Eroberung eines Feldes | Mündungsfeuer, Krater, Rauch; Bäume und Gebäude auf dem Feld brennen |
| Kessel (`cut`) | Panzer ohne Nachschub stehen still und rauchen |
| Schienen, Züge (`04b_rail`) | Züge mit Wagen, ein paar davon mit Panzern geladen |
| Bomben, Flugabwehr | Pilz, Druckwelle, Abwehrraketen als Rauchspur |

## Grundsatz: nur Darstellung

- Es wird **kein Simulationscode** angefasst. Damit bleibt der Gleichlauf auf allen Geräten (`node test/det.js`, `node test/posture.js` bleiben grün).
- Zufall und Animation nutzen einen **eigenen RNG in der Ansicht**, nie den der Simulation und nie `Math.random` in der Simulation.
- Die Nahansicht liest nur, was `hiddenFrom` für den Betrachter freigibt.

## Technik: Canvas 2.5D statt WebGL

- Gleiche Isometrie wie im Prototyp (Raute 64 × 32, Körper aus drei Flächen). Das bleibt **eine einzige HTML-Datei ohne Bibliothek** wie der Rest von Grenzland; three.js würde das Bündel gross machen und auf älteren Handys schwer laufen.
- **Nur im Zoomfenster:** ab einer Zoomstufe wird in der Nahansicht gezeichnet, sonst bleibt die bekannte flache Karte. Schalter «3D» in der Bedienung, Standard flach, damit niemand etwas verliert.
- **Budget:** nur sichtbare Zellen, höchstens etwa 400 Objekte (Panzer, Türme, Bäume), Partikel gedeckelt, Detailstufen nach Zoom.
- Neue Datei `src/06b_close.js` (in `tools/build.py` eintragen), kleiner Haken in `06_view.js`, der bei hohem Zoom dorthin zeichnet.

## Was der Prototyp zeigt (und was nicht)

`prototyp/index.html` ist eine **Ersatz-Simulation** mit erfundenen Regeln (kleines Raster, Gold, Truppen, Stadtausbau, Angriff als Keil). Er zeigt nur Aussehen und Gefühl:
- Stadt antippen = ausbauen, Skyline und Baukran wachsen.
- Ins Feindesland tippen = Panzerkeil, Krater, Rauch, eroberte Städte brennen.
- Oben der Balken «Wachstumstempo»; «Zuschauen» lässt beide Seiten selbst spielen.
- Der Gegner baut und greift an, du musst mit dem Wachstum mithalten.

Nicht enthalten: Netz, Schienen, Gelände, Bomben, Nebel. Das sind die späteren Schritte unten.

## Schritte im echten Spiel

1. **Gerüst:** Schrägansicht bei hohem Zoom, Boden und Grenzen, ohne Objekte. Prüfen, ob Zoomen und Tippen auf dem Handy sauber bleiben.
2. **Städte:** Stufe zu Skyline, Baukran beim Ausbau.
3. **Panzer aus dem Netz:** Anzahl je Zelle, Aufstellung an der Grenze, Marsch von Reserven.
4. **Stösse:** Keil, Feuer, Krater, Rauch bei Eroberung.
5. **Eingraben, Kessel, Schienen und Züge.**
6. **Bomben und Flugabwehr.**

Nach jedem Schritt ein paar Runden spielen (wie in der Vision verlangt), dann korrigieren. Test dazu: `test/nahansicht.js` mit Bildern wie bei `test/feat.js`.

## Offene Fragen

- **Lesbarkeit im Gedränge:** Bei Dichte 6 stehen viele Panzer in einer Zelle. Vermutlich Obergrenze pro Zelle und ein Zahlenabzeichen darüber.
- **Karte der Erde:** Auf der Weltkarte sind Felder sehr klein. Die Nahansicht wäre dort nur ganz nah und sonst ein Zoomziel, kein Dauerzustand.
- **Spielgefühl vs. Tempo:** Rein darstellerische Panzer dürfen nie den Eindruck erwecken, man könne sie steuern. Vorschlag: sie reagieren nicht auf Klicks, nur das Gelände darunter.
