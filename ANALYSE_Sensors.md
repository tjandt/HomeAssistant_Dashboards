# Ansicht „Sensors“ – Analyse & Neuentwurf

Grundlage: hochgeladene `sensors.yaml` (identisch mit der Ansicht `sensors` aus `Rochlitz.yaml`; als `Sensors_view.yaml` mit korrigierter Einrückung abgelegt) → Ergebnis: **`Sensors_neu.yaml`** (ersetzt die Ansicht; Titel „Raumklima“, Pfad `sensors` bleibt).
Die Analyse beruht auf der **Konfiguration**; Messwerte lagen nicht vor.

---

## 1. Kurzfazit

* Die Ansicht ist eine **Liste von 10 Roh-Wertkarten** plus zwei Wetter-Kacheln. Sie zeigt Zahlen, aber keine Aussage („Ist das gut?“), keinen Verlauf und keinen Vergleich.
* **Formal fehlerhaft**: Die hochgeladene Datei enthält 95 **Tabulatoren** in der Einrückung – das ist kein gültiges YAML, HA lehnt es ab (vermutlich Kopier-Artefakt).
* **Inhaltlich irreführend**: Geschätzte „CO₂-Äquivalente“ stehen unmarkiert neben der einzigen echten CO₂-Messung (Küche).
* Neu: **Ampel + Meldungen → Raumkacheln mit 24-h-Verlauf → Komfort-Check → Lüftungs-Assistent → Luftqualität → Vergleich → Draußen/Wetter → Details**.

---

## 2. Kritische Prüfung

### 2.1 Befunde (Zeilen der hochgeladenen Datei)

| # | Zeile | Befund | Schwere |
|---|---|---|---|
| 1 | gesamt | **Tabulatoren** statt Leerzeichen (95 Zeilen). YAML erlaubt keine Tabs zur Einrückung → Datei nicht ladbar. | hoch |
| 2 | 36–40, 59–62, 70–87, 91–100 | **Geschätzt vs. gemessen vermischt**: `eco2`, `co2_equivalent`, `tvoc_equivalent`, `iaq`, `voc` (Gassensor-Schätzungen) und `wetterstation_co2` (echtes CO₂) erscheinen gleichwertig. Schwellen wie „1 000 ppm“ gelten nur für echte CO₂-Werte. | hoch |
| 3 | 91–96 | **Matilda hat keinen CO₂-Wert**, nur „Air Quality Raw“ (Rohwiderstand) und eine Klassifizierung – im Kinderzimmer fehlt damit die wichtigste Größe, und die Ansicht weist nicht darauf hin. | mittel |
| 4 | 29–33 | Kartentitel „Wetterstation **Wetterstation Wetterstation** Balkon“ (automatisch erzeugter Name). | niedrig |
| 5 | 34–41 | Karte „Küche“ listet `wetterstation_*` ohne eigene Namen → Anzeige „Wetterstation CO2/Humidity/Noise/…“ (englisch, Raum nicht erkennbar). | niedrig |
| 6 | 14–18 | „Arbeitszimmer“ besteht aus `abstellkammer_*` + `office_co2_*`. Derselbe Raum heißt in anderen Ansichten Büro/Office/Abstellkammer. | mittel |
| 7 | 52–101 | **Doppelte Raumnamen und englische Reste** in den Zeilen: „Schlafzimmer Schlafzimmer…“, „Kids Matilda Kids Matilda…“, „BME280 Humidity“, „Frida CO2 Equivalent“. Der Kartentitel nennt den Raum bereits. | niedrig |
| 8 | 7–101 | **Nicht vergleichbar**: Wohnzimmer nur Temperatur, Bad Temperatur/Feuchte/Licht, Keller T/rF/Druck, Schlafzimmer 9 Werte; Reihenfolge je Karte unterschiedlich (Arbeitszimmer: Feuchte vor Temperatur, Küche alphabetisch, Frida gemischt). | mittel |
| 9 | 7–101 | **Keine Reihenfolge**: Außensensoren stehen mitten zwischen den Räumen (Balkon, Wetterstation Balkon), danach Küche/Bad/Keller, Kinderzimmer am Ende. | mittel |
| 10 | alle | **Keine Bewertung und kein Verlauf**: nur Momentanwerte, keine Farben/Schwellen, keine Sparklines, kein Raumvergleich; ein Auffälligkeits-Hinweis (zu feucht, CO₂ hoch, Sensor offline) fehlt. | hoch |
| 11 | 102–113 | Zwei Kacheln mit derselben `weather.forecast_home` (stündlich und täglich) ganz unten, ohne Bezug zu den Sensoren; Außenwerte der Balkon-Sensoren stehen weit davon entfernt. | niedrig |
| 12 | 22–33 | **Zwei Balkon-Sensoren** (`balkon_melsungen`, `wetterstation_…_balkon`) ohne Abgleich – Abweichungen (Sonne, Abschirmung, Drift) bleiben unbemerkt. | niedrig |
| 13 | Layout | Klassisches Masonry mit Karten unterschiedlicher Höhe → unruhiges, schwer scanbares Bild; `badges: []`, `theme: Backend-selected` ohne Funktion. | niedrig |

### 2.2 Was gut ist

Alle Räume und beide Außensensoren sind erfasst; Titel pro Raum; `visible`-Beschränkung vorhanden; Entities-Karten erlauben Tippen → Verlauf.

---

## 3. Das neue Layout (`Sensors_neu.yaml`)

| Bereich | Inhalt |
|---|---|
| **Auf einen Blick** | Innen-Ø Temperatur, Ø-Feuchte, CO₂ max, Außen (alle farbig nach Zustand) + **Meldungen** (CO₂ > 1 400 ppm, Feuchte > 65 %, Raum < 16 °C, Sensor ohne Messwert) |
| **Räume – letzte 24 h** | 9 Kacheln (8 Räume + Balkon) in 3×3: Kopfzeile mit Temperatur · Feuchte · CO₂, Farbe nach Komfort, darunter Verlauf; Tippen → Details/Historie |
| **Komfort-Check** | Tabelle je Raum: Temperatur, rF, CO₂, Bewertung (🟢 gut / 🔵 kühl / 🟠 warm / 💧 feucht / 🏜️ trocken / 💨 lüften), mit raumspezifischen Richtwerten (Schlafzimmer 16–21 °C, Keller 8–20 °C, Bad bis 70 % rF) |
| **Lüftungs-Assistent** | Taupunkt außen vs. innen je Raum → „Lüften entfeuchtet“ / „geschlossen halten“; Keller zusätzlich mit Kondensat-Regel; Hinweis „CO₂ hoch“ |
| **Luftqualität** | 4 Gauges (Beschriftung „eCO₂“ = geschätzt, „CO₂“ = gemessen), Bewertungen, Gas-Sensoren, Hinweis zur Messart |
| **Vergleich der Räume** | je ein Diagramm für Temperatur, Feuchte, CO₂, Luftdruck (48 h) mit allen Räumen |
| **Draußen & Wetter** | Stündlich/Täglich, Balkon-Sensoren im Vergleich + **Abweichungsanzeige** |
| **Alle Werte je Raum** | die ursprünglichen Wertelisten, jetzt einheitlich benannt und gleich sortiert (Temperatur, Feuchte, …) |

Eingeführt wurden einheitliche Raumnamen (Arbeitszimmer, Küche, …), deutsche Beschriftungen, einheitliche Reihenfolge der Werte und eine konsistente Farblogik (Temperatur orange/Skala, Feuchte blau, CO₂ grün→rot). Es werden **nur Entities der Originalansicht** verwendet (41 von 41, keine hinzugefügt, keine weggelassen).

---

## 4. Ableitungen

### 4.1 Was die Ansicht über das Haus verrät (Hypothesen)

1. **Vier Sensor-Familien**: BME280 (Keller: T/rF/Druck), BME680-Klasse (Schlafzimmer: IAQ, VOC, Helligkeit, „IAQ-Genauigkeit“ – typisch für die Einlaufphase des Sensors; Matilda: Rohwiderstand), CCS811/eCO₂-Klasse (Arbeitszimmer, Frida: eCO₂/TVOC) und eine **Netatmo-artige Wetterstation** (Küche: echtes CO₂, Lärm, Druck). Daraus folgt: nur in der Küche wird CO₂ wirklich gemessen.
2. **Kinderzimmer haben Priorität**: Frida und Matilda sind die einzigen Räume mit Luftgüte-Bewertung neben Schlafzimmer und Arbeitszimmer – aber ausgerechnet Matilda ohne CO₂-Schätzwert.
3. **Wohnzimmer ist unterversorgt**: nur Temperatur, weder Feuchte noch CO₂ – gerade der meistgenutzte Raum.
4. **Vier Luftdrucksensoren** (Küche, Schlafzimmer, Matilda, Keller) – redundant, aber nützlich für Plausibilität und Wettertrend.
5. **Zwei Balkon-Sensoren** deuten auf bekannte Abweichung (Sonneneinstrahlung) oder einen Austausch hin.

### 4.2 Empfehlungen

| Thema | Beobachtung | Empfehlung |
|---|---|---|
| **Echtes CO₂ im Schlaf-/Kinderbereich** | Nur geschätzte Werte (bzw. keiner bei Matilda). | NDIR-Sensor (SCD40/SCD41) für Schlafzimmer, Frida, Matilda; dann Schwellen 1 000/1 400 ppm belastbar. |
| **Wohnzimmer** | Nur Temperatur. | Temperatur-/Feuchtesensor ergänzen (Schimmel-/Lüftungslogik greift dann auch dort). |
| **IAQ-Genauigkeit** | Wird angezeigt, aber nicht ausgewertet. | IAQ-Werte nur beachten, wenn Genauigkeit ≥ 2 (Kalibrierung); sonst Hinweis. |
| **Schimmelschutz** | Bad und Keller sind die kritischen Räume. | Push, wenn Keller-rF > 65 % > 6 h oder Bad-rF > 70 % > 90 min; Lüften-Empfehlung als Benachrichtigung (Logik steckt jetzt im Dashboard). |
| **Nachtprofil** | Schlafzimmer hat Temperatur, rF, CO₂, Druck, Licht. | Nächtlicher CO₂-Höchstwert und Dauer > 1 000 ppm als Tageskennzahl; zeigt, ob Fenster gekippt/Tür offen sinnvoll ist. |
| **Heizung** | 8 Raumtemperaturen. | Nachtabsenkung/Heizkurve gegen Raumtemperaturen prüfen; Räume dauerhaft < 18 °C oder > 24 °C sichtbar machen (Komfort-Check). |
| **Sensor-Abgleich** | Zwei Balkonsensoren. | Dauerhafte Differenz als Diagnose (> 3 K = Abschirmung/Sonne); der kühlere/schattigere als Referenz. |
| **Sensor-Ausfälle** | Es gibt keine Überwachung. | Meldung „Keine Messwerte“ (bereits im Dashboard) zusätzlich als Push; Batteriestand der Funk-Sensoren, falls vorhanden, mit anzeigen. |
| **Wettertrend** | Mehrere Drucksensoren. | Druckänderung über 3 h als Wettertrend (fallend/steigend) neben die Vorhersage stellen. |
| **Entities aufräumen** | Duplikate wie `bad_*_2`, `humidity_2/3`, `workshop_*` existieren (siehe `ANALYSE.md`). | Umbenennen/Löschen, danach Namen in der Ansicht vereinfachen. |

---

## 5. Einrichtung, Annahmen, Grenzen

**Einrichtung:** Dashboard bearbeiten → Tab „Sensors“ → Stift → ⋮ → *In YAML bearbeiten* → Inhalt von `Sensors_neu.yaml` einfügen. Es werden nur **mushroom** und **mini-graph-card** benötigt (beides schon im Einsatz); keine URL-Anpassung.

**Geprüft:** YAML gültig; alle 41 Entity-IDs stammen aus der Originalansicht; alle 31 Templates in 4 Szenarien (leer / Problemfall / Normal / alles ok) mit strengem `float()`-Verhalten fehlerfrei gerendert.

**Nicht geprüft** (kein Zugriff auf Ihre HA-Instanz): Darstellung und echte Werte. Annahmen:
* `eco2`/`co2_equivalent`/`tvoc` sind Schätzwerte, `sensor.wetterstation_co2` ist echtes CO₂ (aus der Benennung abgeleitet).
* Komfort- und Lüftungsgrenzen sind Richtwerte und im Template anpassbar; die Taupunkt-Näherung `T − (100 − rF)/5` ist für rF > 50 % brauchbar, bei trockener Luft ungenauer.
* Die Aussage „Außenluft“ nutzt `sensor.balkon_melsungen_*` als Referenz.
