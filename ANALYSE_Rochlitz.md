# Rochlitz-Dashboard – Analyse & Neuentwurf

Grundlage: `Rochlitz_view.yaml` (main, 833 Zeilen – inhaltlich identisch mit der Ansicht `rochlitz` aus `Rochlitz.yaml`) → Ergebnis: **`Rochlitz_neu.yaml`** (eigenständiges Dashboard mit 6 Ansichten).
Die Analyse beruht nur auf der **Konfiguration**; Live-Werte waren nicht verfügbar. Die „Ableitungen“ (Abschnitt 4) sind deshalb Schlüsse aus Aufbau und Benennung.

---

## 1. Kurzfazit

* Eine einzige, **endlos lange Ansicht mit 47 Karten** (62 inkl. Unterkarten). Es gibt keine Reihenfolge nach Themen: die Heizung (UVR16x2) ist über **9 Karten** verstreut, der Serverraum über 3, das Büro über 5, der Kühlschrank über 4, der Kaffeevollautomat und das NAS jeweils doppelt bis dreifach.
* Die **wichtigste Information fehlt**: Es gibt keine Aussage „ist alles in Ordnung?“. Wer eine Störung sucht (Oma-Zimmer zu kalt, Warmwasser kalt, Backofen an, Platte defekt), muss sie selbst in 47 Karten finden.
* **Fehler**: vertauschte Beschriftung (Außenfühler), zwei Entity-Namen für denselben Warmwasser-Fühler, uneinheitliche Kühlschrank-Sensoren, eine `bar-card` ohne Skala.
* **Sicherheit**: Kameras und Backofen-START/STOP-Tasten sind für jeden Dashboard-Nutzer ohne Rückfrage erreichbar.
* Neu: **Übersicht mit Ampel** → Themenansichten (Klima, Heizung, Strom, Haus, Technik) → Details (aufklappbare Listen, `more-info`).

---

## 2. Kritische Prüfung von `Rochlitz_view.yaml`

### 2.1 Kennzahlen

| Merkmal | Alt | Neu |
|---|---|---|
| Ansichten | 1 (flach) | 6 |
| Karten (inkl. Unterkarten) | 62 | 104 (zählweise leicht abweichend), dafür thematisch auf 6 Ansichten verteilt |
| Entity-Referenzen / verschiedene | 222 / 160 | 144 verschiedene (143 aus der Originalansicht + Pegel Wechselburg) |
| Entities in mehreren Karten | 41 | gezielt (Übersicht ↔ Detailansicht) |
| `mini-graph-card` | 21 (davon 14 fast gleiche Sparklines mit `line_width: 8`) | 21, aber gezielt: Raumkacheln, Warmwasser, NAS, Pegel |
| Layout | `custom:horizontal-layout` (layout-card) + Theme „Liquid Glass“ | natives `sections`-Layout, kein Theme erzwungen |
| Warmwasser-Verlaufskarten | 3 (24 h, 1 Woche, 48 h) an 3 verschiedenen Stellen | 2 + 1 Statistik, gemeinsam in „Heizung“ |

### 2.2 Fehler und Inkonsistenzen (Zeilen aus `Rochlitz_view.yaml`)

| # | Zeile | Befund | Schwere |
|---|---|---|---|
| 1 | 453–456 | `sensor.outdoor_temperature` heißt „Outdoor **Humidity**“, `sensor.outdoor_humidity` heißt „Outdoor **Temperature**“ – vertauscht. | hoch |
| 2 | 266, 283 vs. 529 | **Zwei Entity-IDs für den Warmwasser-Fühler**: `sensor.node_2_analog_logging_2` (24 h, 1 Woche) und `sensor.uvr16x2_node_2_analog_logging_2` (48 h). Mindestens einer ist ein Überbleibsel. | mittel |
| 3 | 826–829 | `custom:bar-card` „Solar Leistung“ ohne `max` → Standardskala 0–100. Ist der Wert größer (Watt?), steht der Balken dauerhaft voll. | mittel |
| 4 | 716–719 | Backofen **START** und **STOPRESET** sind normale Tasten in einer Liste, ohne Rückfrage, für alle Nutzer. | mittel |
| 5 | 318, 436, 441 | Kühlschrank: `kuhlschrank_rochlitz` (7 Tage) vs. `kuhlschrank_rochlitz_2` (24 h); der **Vorratsraum-Kühlschrank** taucht nur im 24-h-Diagramm auf, nicht im 7-Tage-Diagramm und nicht in der Live-Leiste. | mittel |
| 6 | 153 | Der Außensensor `…7cd9` heißt in der Karte „Aussentemperatur“ **„Tilo“**, an anderer Stelle „Outdoor“/„Rochlitz Outdoor“. | niedrig |
| 7 | 814–818 | `mushroom-chips-card` in einem 2-spaltigen `grid` mit nur einer Karte → eine Spalte bleibt leer. | niedrig |
| 8 | 742, 795 | Zwei Karten mit gleichem Titel „Backofen“; zusammen **47 Entities** inkl. Fahrenheit-Doubletten (`…targettemperaturef`, `…displaytemperaturef`), „Message queue sync“-IDs, englische Namen. | niedrig |
| 9 | 4, 1, 833 | `icon: ''` (kein Tab-Symbol), Abhängigkeit von Theme „Liquid Glass“ und `layout-card`. | niedrig |
| 10 | 32–39 vs. 657–671 | Kaffeevollautomat zweimal; der Status „Online“ steht nur in der oberen, Kindersicherung/Stopp nur in der unteren Karte. | niedrig |
| 11 | 7–26 | „Aktueller Verbrauch“ zeigt 8 Einzelwerte, aber keine Summe; die Kaffeemaschine hat als einzige Steckdose keine Detailkarte. | niedrig |
| 12 | diverse | Beschriftungen uneinheitlich/englisch: „OmaPower“, „Rochlitz Music“, „Rochlitz_Serverraum“, „RL Office Temp“, „Config overtemp type“, „Current/Power/Voltage“ ohne Einheit im Titel. | niedrig |

### 2.3 Informationsdesign

* **Reihenfolge ist Zufall**: Verbrauch → Kaffee → NAS → Klima → Wochenwerte → Warmwasser → Strom-Diagramm → UVR → Kühlschrank → Heizungsaktivität → Kameras → Stecker-Details → NAS-Details → Backofen → Heizungskacheln (erst ganz am Ende, Z. 796!) → Mähroboter.
* **Rohdaten statt Aussage**: Roh-Entities („UVR16x2“: 5 unbenannte Eingänge, 6 unbenannte Ausgänge), keine Einordnung (zu kalt? zu warm? lädt Solar?).
* **Optik**: 14 Sparkline-Karten mit `line_width: 8` und festen Farben (`blue`, `#e74c3c` für den Luftdruck) – bei drei Räumen mal Temperatur/Feuchte/Luftdruck entstehen 14 fast gleiche Kacheln; der Luftdruck ist rot, obwohl rot sonst „warm“ bedeutet.
* **Performance**: 21 `mini-graph-card` + 6 `statistics-graph` + `apexcharts` + `history-graph` fragen beim Öffnen gleichzeitig Verlaufsdaten ab (ca. 30 Abfragen auf einer Seite).

### 2.4 Sicherheit / Privatsphäre

* Kameras „Eingang“ und „Hintereingang“ stehen ohne Hinweis in der Ansicht; die Ansicht hat keine Nutzerbeschränkung.
* Backofen kann über das Dashboard gestartet werden (Fernsteuerung des Geräts ist zusätzlich geräteseitig freizuschalten).

---

## 3. Das neue Dashboard (`Rochlitz_neu.yaml`)

### 3.1 Aufbau

| Ansicht | Inhalt |
|---|---|
| **Übersicht** | Datum, **Auffälligkeiten-Liste**, 6 Raum-Kacheln (farbig nach Zustand), Heizung/Warmwasser/Solar-Ladung/Kessel, Steckdosen-Summe + Verteilung (Donut), Kaffee/Backofen/Mähroboter/NAS, Kamera Eingang, Pegel Mulde |
| **Klima** | **Komfort-Check** (Tabelle: Temperatur, Feuchte, Bewertung je Raum), 48-h-Kacheln, Tageswerte Max/Ø/Min (Woche), Langzeit 60/90 Tage, alle Außenfühler |
| **Heizung** | Anlage auf einen Blick (Kollektor, Kessel, Speicher oben/unten), Solar-Ladung (Δ), Kessel-Strategie, Warmwasser 24 h/7 Tage/14 Tage, „Wer heizt wann?“ (Kessel vs. Speicher, Pumpen als Zeitleiste), Rohwerte mit Klarnamen |
| **Strom** | Leistung je Steckdose (schaltbar), 7-Tage-gestapelt, 24-h-stündlich, Kühlschränke 30 Tage, Tabelle „Alle Steckdosen“ (W/A/V/kWh) |
| **Haus** | Kaffeevollautomat (Steuerung, Zähler), Backofen (Status, Steuerung **mit Rückfrage**, Einstellungen), Kameras, Mähroboter, Flightradar |
| **Technik** | RochlitzNAS (Gauges, Durchsatz, CPU/RAM), Festplatte, Serverraum |

### 3.2 Was neu ist

* **Auffälligkeiten-Liste**: Frost, Oma-Zimmer < 18 °C, Feuchte > 65 %, Warmwasser < 35 °C, Backofen in Betrieb, Reinigung/Entkalken fällig, Kaffeevollautomat/Kameras/Kühlschrank-Messung offline, Serverraum > 30 °C, NAS-Sicherheit/defekte Sektoren/Restlebensdauer/Temperatur/CPU/RAM sowie **Datenqualität** (Schlüsselsensoren ohne Messwert). Schwellen stehen im Template und sind dort anpassbar.
* **Komfort-Check** je Raum (Oma ab 20 °C, andere 18–24/25 °C, Feuchte 35–60 %).
* **Solar-Ladung** (Δ Kollektor − Speicher unten > 8 K) und **Kessel-Strategie** (Vorlauf vs. Speicher oben) als Klartext statt Rohwerten.
* **Pumpen als Zeitleiste** (`history-graph` der Ausgänge) – zeigt Laufzeiten auf einen Blick.
* **Zustandsfarben** (Raum, Warmwasser, Last, Gesundheit) statt fester Farben; gleiche Farbsemantik überall.
* **Rückfragen** bei Backofen START/STOP und „Programm stoppen“.
* Raumkacheln mit Kopfzeile + Verlauf statt 14 gleicher Sparklines; **eine** Warmwasser-Entity (`uvr16x2_…`), die alte steht nur in der Rohwerteliste.

### 3.3 Bewusst weggelassen

Fahrenheit-/Message-Queue-/CPV-Werte des Backofens, `config_overtemp_type` der Steckdosen (3 Stück ohne Aussagewert), doppelte Karten (Serverraum, Büro, NAS, Kaffee).

---

## 4. Ableitungen

### 4.1 Was die Ansicht über Rochlitz verrät (Hypothesen)

1. **Rochlitz wird betreut, nicht nur bewohnt.** Ein eigener Zimmer-Sensor und eine eigene Messsteckdose für „Oma“, Kameras, Rasenmäher-Roboter und Kühlschrank-Überwachung sprechen für *Fernbetreuung* → die Auffälligkeiten-Liste ist wichtiger als die Diagramme.
2. **Heizung mit Solar und Kessel ist ein aktives Optimierungsthema**: Die Karte „Kessel vs. Speicher Strategie“, 14-Tage-Kollektor-Statistik und Warmwasser-Farbstufen (30/35/40/45/**55 °C**) zeigen, dass man beobachtet, wann der Kessel nachheizt und wann Solar reicht. 55 °C ist die übliche Legionellen-Marke.
3. **Messung ist steckdosen-basiert**: 8 Zwischenstecker + 2 Kühlschränke (einer ohne Live-Wert). Es gibt keinen Hauptzähler → Grundlast, Heizungs-/Lichtanteil und Gesamtverbrauch sind nicht ableitbar.
4. **Das NAS hat nur eine gemeldete Platte** („Drive 1“) → keine Redundanz, Backup entscheidend.
5. **Zwei Außensensoren plus Dach-Fühler** (`…7cd9`, `outdoor_*`, UVR „Dach“) – der Vergleich zeigt Sonneneinfluss/Messfehler.
6. **Das Dashboard ist gewachsen** (Ein-Zeilen-Titel, englische Namen, Reste wie Fahrenheit-Werte), nicht entworfen.

### 4.2 Konkrete Empfehlungen

| Thema | Beobachtung | Empfehlung |
|---|---|---|
| **Oma-Zimmer** | Temperatur, Feuchte, Druck + eigene Steckdose | Alarm < 18 °C (Push) und < 35 % rF; optional *neutrale* Aktivitätsanzeige (Steckdosenleistung ändert sich) nur **mit Einverständnis** der Betroffenen. |
| **Heizperiode** | Es ist Oktober: Solar liefert selten Δ > 8 K, der Kessel übernimmt. | Vor der Saison prüfen: Kessel-Vorlauf ↔ Speicher oben im 24-h-Verlauf; Warmwasser-Alarm auf < 35 °C aktivieren; Frostschutz für leere Räume (< 5 °C). |
| **Solar-Auswertung** | Pumpen-Ausgänge und Δ sind vorhanden, aber nicht verdichtet. | `history_stats` für Pumpenlaufzeit/Tag; Tagesertrag gegen Wetter; Alarm, wenn Warmwasser 7 Tage nie ≥ 55 °C erreicht. |
| **Entities bereinigen** | `node_2_analog_logging_2` vs. `uvr16x2_node_2_analog_logging_2`, `kuhlschrank_rochlitz` vs. `…_2`, `outdoor_*` | Je einen führen, den anderen löschen/umbenennen; Statistik-Historie ggf. migrieren. |
| **Kühlschränke** | Dauermessung, nirgends ausgewertet; Vorratsraum ohne Live-Wert. | Jahreshochrechnung `Ø-Watt × 8,76 = kWh/Jahr` (Richtwert modern ~100–150, alt 250–400 kWh/a); Live-Messung auch für Vorratsraum; Alarm bei 0 W länger als 2 h. |
| **Standby** | „Computer“ und „Büro/Musik“ laufen vermutlich dauerhaft. | Automation: aus, wenn Leistung < Standby-Schwelle > 30 min; Stromkosten/Jahr abschätzen. |
| **Hauptzähler** | Nur Zwischenstecker. | Zähler am Hausanschluss (Shelly 3EM o. ä.) → Grundlast, €/Tag, Heizungs-Strom. |
| **Kaffeevollautomat** | Zähler für Kaffees/Milch/Heißwasser. | Wartungsintervall (Entkalken/Reinigen) nach Tassenanzahl; Kosten je Tasse. |
| **Backofen** | Status inkl. Reinigungs-/Entkalkungserinnerung. | Push „Backofen läuft > 2 h“ (Sicherheit); Erinnerungen im Dashboard sind jetzt sichtbar. |
| **NAS** | Einzelne Platte, 3 Warnflags (nur im Dashboard). | Push-Benachrichtigung bei `on`; externes Backup (z. B. zum JandtNAS); SMART-Trend (Temperatur, Sektoren). |
| **Serverraum** | Temperatur + Uptime des Sensors | Alarm > 30 °C; Uptime-Sprung (Neustart des Sensors) als Hinweis auf Stromausfall. |
| **Hochwasser** | Mulde-Pegel Wechselburg liegt flussaufwärts von Rochlitz. | Schwelle = Meldestufe des Pegels (Landesamt) + Anstiegsrate. |
| **Zugriff** | Kameras/Backofen für alle sichtbar. | Entscheiden, wer sie sehen darf (`visible:` je Ansicht); Rückfrage ist bereits eingebaut. |

---

## 5. Einrichtung, Annahmen, Grenzen

**Einrichtung:** *Einstellungen → Dashboards → Dashboard hinzufügen* → Titel „Rochlitz“, URL **`dashboard-rochlitz`** → Roh-Konfigurationseditor → Inhalt von `Rochlitz_neu.yaml` einfügen. Anderer Pfad: `sed -i 's#/dashboard-rochlitz/#/MEIN-PFAD/#g' Rochlitz_neu.yaml`.
Benötigt: HACS-Karten **mushroom**, **mini-graph-card**, **apexcharts-card**; für Einzelkarten **landroid-card**, **flightradar-flight-card** (wie bisher). Entfallen: `layout-card`, `bar-card`, Theme „Liquid Glass“.

**Geprüft:**
* YAML gültig, 6 Ansichten, alle Navigationsziele vorhanden.
* Jede Entity-ID stammt aus dem Original (`Rochlitz_view.yaml`; zusätzlich `sensor.wechselburg_mulde_pegelstand` aus `Rochlitz.yaml`).
* Alle 80 Jinja-Templates in 4 Szenarien (leer / Problemfall / Normal / alles ok) mit strengem `float()`-Verhalten fehlerfrei gerendert.

**Nicht geprüft** (kein Zugriff auf die HA-Instanz): tatsächliche Darstellung und Messwerte. Bitte gegenprüfen:
* UVR16x2-Zuordnung: Eingang 1 Kessel-Vorlauf, 2 Speicher oben, 4 Speicher unten, 7 Kollektor; Ausgang 3 = Heizung. (Namen stammen aus dem alten Dashboard; Schwelle Δ > 8 K ist eine übliche Solar-Einschaltdifferenz.)
* `binary_sensor.rochlitznas_*` (Sicherheit, Sektoren, Restlebensdauer): **on = Problem**.
* `sensor.uvr16x2_node_2_analog_logging_2` ist der aktuelle Warmwasser-Sensor in °C.
* `sensor.aeg_backofen_upperoven_appliancestate` meldet im Betrieb `RUNNING`.
* Komfortgrenzen (Oma ≥ 20 °C, Feuchte 35–60 %) sind Richtwerte und im Template anpassbar.
* „Steckdosen gesamt“ ist die gemessene Summe, nicht der Hausverbrauch.
