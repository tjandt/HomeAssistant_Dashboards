# Dashboard-Analyse & Neuentwurf

Grundlage: `Rochlitz.yaml` (altes Dashboard, 4 293 Zeilen) → Ergebnis: `Zuhause.yaml` (neues Dashboard).
Die Analyse beruht ausschließlich auf der **Konfiguration**. Live-Werte aus Home Assistant standen nicht zur Verfügung – alle „Ableitungen“ in Abschnitt 4 sind daher Schlussfolgerungen aus Aufbau und Benennung, keine Messwert-Auswertungen.

---

## 1. Kurzfazit

* Das alte Dashboard ist **inhaltlich reich, aber nicht lesbar**: 13 Ansichten, 1 057 Entity-Referenzen für nur 551 verschiedene Entities. Ein Drittel davon (224 Entities) steht in mehr als einer Ansicht, 74 sogar in drei oder mehr.
* Der Standort **Rochlitz** existiert dreifach (`rochlitz`, `rochlitz-modern`, `rochlitznew`), **Klima Melsungen** ebenfalls dreifach (`sensors`, `statistik`, `sensors_melsungen`). `rochlitznew` enthält alle Entities von `rochlitz-modern` und 128 von 160 aus `rochlitz`.
* Es gibt **echte Fehler** (vertauschte Beschriftungen, falsch zugeordnete Sensoren, ungültige Optionen) und mehrere Stellen, an denen das Dashboard **falsche Sicherheit** vermittelt (statisch grüne Status-Symbole, „Gesamtverbrauch“ = nur Summe von 8 Steckdosen).
* Neu: **drei Ebenen** – Übersicht (Ampel + Kennzahlen) → Standort/Thema (Karten mit Verlauf) → Detail (Unteransichten, `more-info`). Dazu eine **Auffälligkeiten-Liste**, die Probleme aus beiden Standorten aktiv zusammenträgt.

---

## 2. Kritische Prüfung des bestehenden Dashboards

### 2.1 Kennzahlen

| Merkmal | Alt | Neu |
|---|---|---|
| Ansichten | 13 (davon 3× Rochlitz, 3× Klima) | 9 + 3 Detail-Unteransichten |
| Layout-Technik | 4 (klassisch, `masonry-layout`, `horizontal-layout`, `sections`) | 1 (`sections`, nativ) |
| Themes | 3 verschiedene (`Backend-selected`, `Frosted Glass Dark`, `Liquid Glass`) | keins erzwungen → folgt Ihrem Profil |
| `mini-graph-card` | 75× | 36× |
| Warmwasser-Verlaufskarte (`…analog_logging_2`) | 7× | 2× |
| Zusatz-Karten (HACS) | + `layout-card`, `bar-card` | nur noch mushroom, mini-graph, apexcharts (+ die vorhandenen Spezialkarten) |
| Entities | 551 | 487 (Rest: Duplikate, API-Dumps, Altlasten – siehe 3.4) |

### 2.2 Fehler und Inkonsistenzen

| # | Fundstelle (`Rochlitz.yaml`) | Befund | Schwere |
|---|---|---|---|
| 1 | Z. 1938–1941 | `sensor.outdoor_temperature` heißt „Outdoor **Humidity**“, `sensor.outdoor_humidity` heißt „Outdoor **Temperature**“ – vertauscht. | hoch |
| 2 | Z. 1207–1216 | Pi-hole-Karten haben **fest** `icon_color: green` + Schild-Haken. Sie sind auch grün, wenn Pi-hole ausgefallen ist. | hoch |
| 3 | Z. 2529–2538 | „Gesamtverbrauch“ ist nur die Summe von 8 Steckdosen (ohne Heizung, Licht, Herd …) und wertet `unavailable` als 0 W. Das unterschätzt still. | mittel |
| 4 | Z. 487, 596 | Plug `staubsauger_*` ist als „**Server**“ beschriftet. Entweder falscher Name oder falsche Beschriftung. | mittel |
| 5 | Z. 825, 891, 870 | `statistik`-Ansicht verwendet andere Sensoren als der Rest: „Schlafzimmer“ → `sensor.schlaf_temperature` (sonst `schlafzimmer_schlafzimmer_temperatur`), „Matilda“ → `sensor.temperature` (sonst `kids_matilda_…`), „Büro“ → `abstellkammer_*`. | mittel |
| 6 | mehrere | **Ein Raum, vier Namen**: `abstellkammer_*` heißt Abstellkammer, Arbeitszimmer, Office und Büro. | mittel |
| 7 | Z. 2345 | `apexcharts-card` mit `stacked: true` auf oberster Ebene – dafür gibt es keinen Schlüssel (gehört nach `apex_config.chart.stacked`). Die Balken werden nicht gestapelt. | mittel |
| 8 | Z. 3683 | `formatter: function(val) { … + '%' }` ohne `EVAL:`-Präfix und mit **%** für ppm-Werte; Radial-Bars für eCO₂/TVOC sind außerdem schwer ablesbar. | niedrig–mittel |
| 9 | Z. 2432 | Gauge `uvr16x2_node_2_output_16` mit `max: 5000` – Skala/Einheit nicht belegt (eher Drehzahlstufe/% als Watt?). | niedrig |
| 10 | Z. 2527 | `now().strftime('%A, %d. %B')` liefert englische Tag-/Monatsnamen. | niedrig |
| 11 | Z. 944–1153 | 3× identische ~70-Zeilen-Konfiguration der `soundtouchplus-card` inkl. **Beispieldaten aus der Dokumentation** (`YourSpotifyUserId`, „Zach Williams Radio“, „K-Love Radio“). | niedrig |
| 12 | Z. 4129, 4240, 1489 | Platzhalter „Neuer Abschnitt“ (2×), `icon: ''`, Titel „Wetterstation Wetterstation Wetterstation Balkon“, View-Titel `hauswirtschaft` mit Pfad `s`. | niedrig |
| 13 | Z. 3331 | Beschriftung „Mindestlebensdauer **OK**“ für `…unterhalb_der_mindestrestlebensdauer` – der Sensor ist `on`, wenn es **nicht** OK ist. Logik invertiert dargestellt. | mittel |

### 2.3 Informationsdesign

* **Rohdaten statt Information**: Backofen = 47 Entities (inkl. Fahrenheit-Doubletten, Message-Queue-IDs), Strava = 52 Entities in zwei Listen, Tanken = 24 Entities flach, Startseite „Energy“ = 46 Entities in einer Liste (inkl. Waage-Impedanz, `binary_sensor.configure*`).
* **Keine Priorisierung**: NAS-Temperatur, Toner und Wasserstand stehen gleichrangig neben Sofa-Temperatur; nirgends sagt das Dashboard „Es gibt ein Problem“.
* **Doppelte Wahrheit**: Waschmaschine existiert in drei Integrationen/Entity-Sätzen (`run_state`/`remaining_time` *und* `aktueller_status`/`verbleibende_zeit` *und* Steckdose) und wird in fünf Karten mit demselben Titel „Waschmaschine“ dargestellt.
* **Zwei ungeprüfte Duplikat-Paare**: FRITZ!Box-Durchsatz (`internetgatewaydevicev2_…` und `fritz_box_7520_ui_…`), Samsung C430W/C43x (zwei Einträge für vermutlich *ein* Gerät).
* **Visuell uneinheitlich**: hart codierte Farben (`#e74c3c`, `blue`, `yellow`), `line_width: 8` bei Sparklines, drei Themes, vier Layout-Engines.

### 2.4 Sicherheit / Privatsphäre

* Die Rochlitz-Ansichten haben **keine** `visible`-Einschränkung. Jeder Dashboard-Nutzer sieht dort Kameras (Eingang/Hintereingang) und kann den **Backofen starten/stoppen** (Buttons ohne Rückfrage). Ob das so gewollt ist (Familie vor Ort), sollte bewusst entschieden werden.
* Die Benutzer-ID `091148…` steht im Klartext im Repository (unkritisch, aber nicht nötig).

---

## 3. Das neue Dashboard (`Zuhause.yaml`)

### 3.1 Informationsarchitektur

```
Ebene 1  Übersicht           Ampel: „Was braucht Aufmerksamkeit?“ + 6–8 Kennzahlen je Standort
Ebene 2  Melsungen / Rochlitz / Energie / Verlauf / Technik / Mobilität / Sport / Medien
Ebene 3  Detail              Tippen → more-info (Verlauf), Halten/Überschrift → Unteransicht
                             (NAS-Details, Backofen, Strava-Details)
```

| Ansicht | Inhalt (Cluster) |
|---|---|
| **Übersicht** | Begrüßung/Datum (deutsch), Wetter, Kalender, **Auffälligkeiten**, Kennzahlen Melsungen & Rochlitz, Auto, günstigster E10, Pollen, Pegel, Gewicht |
| **Melsungen** | Raumklima (9 Räume als Kopfzeile + Sparkline), Luftqualität (Gauges), **Lüftungs-Assistent**, Hauswirtschaft, Licht, Steckdosen/Strom, Zähler & Umwelt |
| **Rochlitz** *(für alle sichtbar)* | Raumklima, Heizung & Solar (UVR16x2), Strom je Steckdose, Küche/Backofen/Rasenmäher, Kameras, NAS |
| **Energie** | Leistung jetzt (Donut je Standort), Energie-Dashboard, Tages-/Stundenverbrauch, Geräteliste, Licht-/Audio-Zähler |
| **Verlauf** | Langzeit-Charts Temperatur, Feuchte, CO₂, TVOC, Lärm, Licht (Melsungen 90 d) · Innen/Außen, Kollektor, Warmwasser (Rochlitz) |
| **Technik** | JandtNAS, TimeMachine (Gauges + Durchsatz), Pi-hole ×2, FRITZ!Box, USV, Drucker (Toner-Gauges) |
| **Mobilität** | Audi A5 (Tank, Reichweite, Service, Karte), Spritpreis-Tabelle (sortiert, „geschlossen“ markiert), Preisverlauf |
| **Sport** | Waage (Trend-Sparklines, alle Körperwerte), Strava-Jahresbilanz + letzte Aktivität |
| **Medien** | 8 Player als kompakte Karten, Büro-Klang, Sonos/SoundTouch (erweitert) |
| *Detail* | NAS-Details · Backofen (Status/Steuerung/Einstellungen) · Strava-Details |

### 3.2 Neu gegenüber dem Original

* **Auffälligkeiten-Liste** (Übersicht): sammelt Probleme aus beiden Standorten – Fehler Waschmaschine, CO₂ > 1 400 ppm, Feuchte > 65 %, Kühlschrank aus/offline, Frostgefahr, Oma-Zimmer < 16 °C, Serverraum > 30 °C, Warmwasser < 35 °C, Backofen in Betrieb, Kameras offline, NAS-Sicherheit/Platte/Füllstand/Temperatur, USV im Batteriebetrieb, Pi-hole down, Toner ≤ 15 %, DSM-Update – plus **Datenqualität** (Liste der Schlüssel-Sensoren ohne Messwert). Schwellwerte stehen im Template und lassen sich dort anpassen.
* **Lüftungs-Assistent**: Taupunkt-Vergleich Außen/Keller (Näherung `Td ≈ T − (100 − rF)/5`): Keller nur lüften, wenn die Außenluft *trockener* ist und nicht an den kühlen Wänden kondensiert. Dazu ein Feuchte-Check (> 60 % rF).
* **Solar-Karte** (Rochlitz): Δ Kollektor − Speicher unten, daraus „Solar lädt vermutlich“ (> 8 K).
* **Spritpreis-Tabelle**: alle 6 Stationen, nach E10 sortiert, geschlossene Stationen ausgeblendet/markiert; günstigster E10 auf der Übersicht.
* **Zustandsabhängige Farben** statt fester: jede Raum-Kachel färbt sich nach Temperatur/Feuchte/CO₂; Pi-hole, NAS, Toner, Tank nach Zustand.
* **Rückfrage** bei Backofen START/STOP und „Programm stoppen“.
* **Deutsche Beschriftung**, einheitliche Raumnamen, einheitliche Farbsemantik (Temperatur orange, Feuchte blau, Energie gelb, Wasser grün-blau).

### 3.3 Bewusst nicht übernommen

Gesamtliste der Backofen-API-Werte (Fahrenheit, Message-Queue), Waage-Rohwerte (`xiaomi_*`), alle `*_config_overtemp_type`, `workshop_*`/`ambient_light*`/`bme280_*`/`humidity_2/3`/`pressure_2/3` (Altlasten ohne Raumzuordnung), die dreifache Rochlitz-Darstellung. Voltage/Current der Steckdosen stehen weiterhin als Detail-Karte (Melsungen) bzw. in der Gerätezeile (Rochlitz).

### 3.4 Nicht mehr referenzierte Original-Entities (64)

Fast ausschließlich Duplikate/Altlasten (`*_2`, `*_3`, `workshop_*`, `bme280_*`, `lywsdcgq_*`, `temperature_humidity_sensor_903e_*`, `binary_sensor.configure*`), technische API-Werte des Backofens und der Samsung-Drucker-Seriennummern. Die vollständige Differenz lässt sich mit dem Entity-Abgleich (Abschnitt 5) reproduzieren.

---

## 4. Ableitungen

### 4.1 Was das Dashboard über Haushalt und Nutzung verrät (Hypothesen)

1. **Zwei Standorte mit unterschiedlicher Rolle.** Melsungen = Wohnort der Familie (Kinderzimmer *Frida* und *Matilda* mit Luftgütesensoren, Schlafzimmer-IAQ, Auto, Spritpreise, Sport). Rochlitz = **betreutes Zweithaus** (Zimmer „Oma“ mit eigener Messsteckdose, Kameras, Rasenmäher-Roboter, Heizung mit Solar, eigenes NAS/Serverraum, Backofen). Für Rochlitz ist *Fernüberwachung und Fürsorge* wichtiger als Komfort-Visualisierung → Alarme vor Diagrammen.
2. **Rochlitz hat mehrere Nutzer.** Nur dort fehlt die `visible`-Beschränkung.
3. **Kleines Homelab**: 3 NAS (JandtNAS, TimeMachine, RochlitzNAS), 2 Pi-hole, USV, FRITZ!Box, UVR-Datenlogger. Es gibt Auslastung/Temperatur, aber **keine** Überwachung der Standort-Verbindung und **keinen** Backup-Status.
4. **Das Dashboard ist durch wiederholtes Generieren gewachsen** (Emoji-Titel, Spiegelansichten, „Neuer Abschnitt“) – ein Zeichen, dass Struktur fehlte.
5. **Heizungsoptimierung ist ein aktives Thema**: Karten wie „Kessel vs. Speicher Strategie“ und die Warmwasser-Schwellen 30/35/40/45/**55 °C** (55 °C = übliche Legionellen-Marke) zeigen, dass jemand abwägt, wann der Kessel läuft und wann Solar reicht.

### 4.2 Technische Ableitungen und Empfehlungen

| Thema | Beobachtung | Empfehlung |
|---|---|---|
| **Messlücke Hauptzähler** | ~21 Zwischenstecker, aber kein Gesamtzähler. Hausverbrauch, Grundlast und Eigenanteil der Heizung sind nicht ableitbar. Die Karte `energy-distribution` des Originals setzt einen Hauptzähler voraus. | Zähler am Hausanschluss (Shelly 3EM, Tibber Pulse o. ä.) an beiden Standorten. Danach: *Grundlast = Nachtminimum*, *€/Tag*, Anteil messbarer Geräte. |
| **Kühlschränke** (3 gemessen: Melsungen, Rochlitz, Vorratsraum) | Dauermessung vorhanden, aber nirgends ausgewertet. | Jahreshochrechnung `Ø-Watt × 8,76 = kWh/Jahr`. Richtwert: moderne Geräte ~100–150 kWh/a, alte oft 250–400 kWh/a → Austausch rechnet sich ggf. in wenigen Jahren. |
| **Waschmaschine** | Drei parallele Datenquellen (zwei Integrationen + Steckdose). | Eine Integration entfernen. „Wäsche fertig“-Push zusätzlich über die Steckdose (Leistung > 20 W, danach < 5 W für 2 min) – funktioniert auch bei Cloud-Ausfall. |
| **Solarthermie/Heizung** | Δ Kollektor−Speicher und Pumpenausgänge sind vorhanden, aber nicht verdichtet. | `history_stats` für Pumpenlaufzeit/Tag; Tageswert „Solar-Stunden“ gegen Wetter; Alarm, wenn Warmwasser 7 Tage nie ≥ 55 °C erreicht (Legionellen) bzw. wenn Kessel heizt, obwohl Δ > 8 K. |
| **Lüften / Schimmel** | 9 Raumsensoren + 2 Außensensoren. | Lüftungsempfehlung als Push (Taupunkt-Logik aus dem Dashboard in eine Automation). Die beiden Balkon-Sensoren gegeneinander abgleichen → Sonnen-Einfluss/Drift erkennen. |
| **Luftqualität** | Die meisten Werte heißen `eco2`/`co2_equivalent`/`iaq` → vermutlich **geschätzte** CO₂ (BME680/CCS811), kein NDIR. Echte CO₂-Messung nur in der Küchen-Wetterstation. Matilda hat nur `air_quality_raw`. | Schwellwerte als Richtwerte lesen. Für Schlaf-/Kinderzimmer einen echten CO₂-Sensor (SCD40/SCD41) nachrüsten. |
| **Hochwasser-Frühwarnung** | Pegel Fulda/Eder (bei Melsungen) und Mulde/Wechselburg (liegt flussaufwärts von Rochlitz) werden nur angezeigt. | Alarmschwelle = Meldestufe des jeweiligen Pegels (beim Landesamt nachschlagen) + Anstiegsrate (cm/h). |
| **Tanken** | Preise von 6 Stationen, 90-Tage-Verlauf nur für eine. Tankfüllstand und Reichweite des Audis sind bekannt. | Automation „Tank < 30 % und günstigster E10 liegt ≥ 2 ct unter dem 30-Tage-Mittel“. Statistik nach Wochentag/Uhrzeit → beste Tankzeit. |
| **Sport/Gewicht** | Tageswerte der Waage schwanken stark. | 7-Tage-Gleitmittel statt Tagesmittel; Trainingsvolumen (Strava) gegen Körperfett/Gewicht korrelieren. |
| **Backup/Netz** | Kein Backup-Status, keine Standort-Verbindungsprüfung. | Ping-/Status-Sensor Rochlitz↔Melsungen (VPN), letzter erfolgreicher HA-Backup-Zeitpunkt, Hyper-Backup-Alter als Alarm. |
| **Sicherheit** | Backofen-Buttons und Kameras für alle Nutzer ohne Rückfrage. | Rückfrage ist jetzt eingebaut; zusätzlich Benachrichtigung „Backofen läuft > 2 h“. Entscheiden, ob Kameras für alle sichtbar sein sollen. |
| **Datenhygiene** | Mindestens ein Dutzend Mehrfach-Einträge (`configure_3/4`, `samsung_c430w_2`, `bad_*_2`, `workshop_*_2`, `temperature_3`, zwei Marantz-Player, zwei FRITZ!Box-Paare …). | Unter *Einstellungen → Geräte & Dienste → Entitäten* nach „nicht verfügbar/verwaist“ filtern und aufräumen; Entitäten sinnvoll umbenennen (z. B. `staubsauger_*` → `server_*`). Danach können die `Zuhause.yaml`-Namen vereinfacht werden. |

---

## 5. Einrichtung, Annahmen, Grenzen

**Einrichtung**

1. *Einstellungen → Dashboards → Dashboard hinzufügen* → Titel „Zuhause“, URL **`dashboard-zuhause`**.
2. Dashboard öffnen → ⋮ → *Dashboard bearbeiten* → ⋮ → *Roh-Konfigurationseditor* → Inhalt von `Zuhause.yaml` einfügen.
3. Anderer URL-Pfad? `sed -i 's#/dashboard-zuhause/#/MEIN-PFAD/#g' Zuhause.yaml` (die Navigations- und Zurück-Links verwenden ihn).
4. Benötigte HACS-Frontend-Karten: **mushroom**, **mini-graph-card**, **apexcharts-card**. Einzelne Spezialkarten (wie im Original): sonos-card, soundtouchplus-card, body-miscale-card, landroid-card, flightradar-flight-card, pi-hole. `layout-card` und `bar-card` werden nicht mehr benötigt.

**Was geprüft wurde**

* YAML ist syntaktisch gültig; alle 12 Ansichten, Navigationsziele und `back_path` stimmen überein.
* **Jede** Entity-ID im neuen Dashboard kommt im Original vor (nichts erfunden).
* Alle 185 Jinja-Templates wurden mit simulierten Zuständen (leer / Problemfall / Normalfall) und strengem `float()`-Verhalten gerendert – ohne Fehler.

**Was *nicht* geprüft werden konnte** (kein Zugriff auf Ihre Home-Assistant-Instanz)

* Die tatsächliche Darstellung in Ihrer HA-/Karten-Version und die realen Zustandswerte.
* Annahmen, die Sie bitte kurz gegenprüfen:
  * `binary_sensor.*security_status` und `…unterhalb_der_mindestrestlebensdauer` sind bei **on = Problem** (Synology-Standard).
  * `sensor.ups_status_data` enthält bei Netzbetrieb `OL`.
  * `sensor.warmwasser…analog_logging_2` ist in °C; `audi_a5_avant_tank_level` in %.
  * Pollen-Sensoren liefern Stufen wie `0`, `0-1`, `1` (DWD).
  * `smart_series_7000_cb00` ist eine Oral-B-Zahnbürste.
  * Die beiden `custom:pi-hole`-`device_id`s sind unverändert aus dem Original übernommen (Zuordnung Melsungen/Rochlitz nicht verifiziert).
  * `sensor.uvr16x2_node_2_analog_logging_2` ist der aktuelle Warmwasser-Sensor; der ältere `sensor.node_2_analog_logging_2` wird nicht mehr verwendet.
* Fester Hinweis: Das Dashboard zeigt „Steckdosen gesamt“ bewusst als *gemessene Summe*, nicht als Hausverbrauch.
