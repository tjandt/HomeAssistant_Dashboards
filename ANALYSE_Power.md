# Ansicht „Energy“ (Strom Melsungen) – Analyse & Neuentwurf

Grundlage: hochgeladene `power.yaml` (identisch mit der Ansicht `energy` aus `Rochlitz.yaml`; als `Power_view.yaml` mit korrigierter Einrückung abgelegt) → Ergebnis: **`Power_neu.yaml`** (ersetzt die Ansicht, Pfad `energy` bleibt, Titel „Strom“).
Die Analyse beruht auf der **Konfiguration**; Messwerte lagen nicht vor.

---

## 1. Kurzfazit

* Die Ansicht heißt „Energy“, enthält aber **kein einziges Diagramm** und keine Summe: 14 Geräte, 61 Entities, alles als Zahlenliste. Es gibt keine Antwort auf „Was verbrauche ich gerade / pro Monat / pro Jahr, was kostet es, wo lässt sich sparen?“.
* **Formal fehlerhaft**: 206 Zeilen mit Tabulatoren → kein gültiges YAML (vermutlich Kopier-Artefakt).
* **Riskant**: Die Kacheln „Waschmaschine“ und „Trockner“ schalten bei Antippen die **Steckdose** (`toggle`) – ohne Rückfrage kann so ein laufendes Programm unterbrochen werden.
* Neu: **Jetzt-Kennzahlen + Standby-Check → Ranking mit Balken → Geräte nach Räumen → Verlauf (Tag/Monat) → Kosten der Dauerläufer → Zähler → Detailtabelle**.

---

## 2. Kritische Prüfung

### 2.1 Befunde (Zeilen der hochgeladenen Datei)

| # | Zeile | Befund | Schwere |
|---|---|---|---|
| 1 | gesamt | **Tabulatoren** (206 Zeilen) statt Leerzeichen → Datei nicht ladbar. | hoch |
| 2 | 17–21, 30–34 | `tile` mit `switch.waschmaschine` / `switch.trockner`: Tippen = **Strom aus/ein**, keine Rückfrage. | hoch |
| 3 | 92–93, 201–204 | Plug `staubsauger_*` ist als „**Server**“ beschriftet (Glance und Karte „Keller & Server“). Entweder falscher Entity-Name oder falsche Beschriftung. | mittel |
| 4 | 78–79 vs. 112–124 | `wohnzimmer_power_consumption` heißt in der Glance-Leiste „**TV**“, in der Karte „Wohnzimmer & Deko“ dagegen „Power Consumption“ des Wohnzimmer-Schalters. Was hängt wirklich dran? | mittel |
| 5 | alle | **Keine Auswertung**: keine Summe, kein Ranking, kein Verlauf, keine Tages-/Monatswerte, keine Kosten. Die „Energy“-Werte (`…_energy`, „Licht (kWh)“, „Total Energy“) sind **Lebensdauer-Zähler** ohne Zeitbezug und für sich nicht aussagekräftig. | hoch |
| 6 | 72–97 | „Aktuelle Watt“ (11 Werte) steht unter „Küche & Technik“, enthält aber Waschmaschine, Schlafzimmer, Keller, Office; ungeordnet (nicht nach Leistung sortiert), gemischte Namen (TV, Kaffee, Office, Server, Keller). | mittel |
| 7 | verteilt | **11 Leistungswerte stehen doppelt** (z. B. `kuhlschrank_power`, `kaffeemaschine_power`, `technik_power`, `waschmaschine_power`, `trockner_power`, `staubsauger_power`, `arbeitsplatz_power_2`). | niedrig |
| 8 | 45–51, 102–108, … | Beschriftungen als Notbehelf: `'- power'`, `'- energy'`, `'- current'`, `'- voltage'` (führender Strich), englische Reste (Power/Energy/Current/Voltage/Consumption), `Config_overtemp_type`. | niedrig |
| 9 | 52, 67 | `config_overtemp_type` (Konfigurationswert des Steckers) in der Hauptkarte – ohne Informationswert für den Alltag. | niedrig |
| 10 | 65, 181 | Unklar, was **`consumption`** (Thermomix, Arbeitsplatz) von **`energy`** unterscheidet; beide stehen ohne Erklärung nebeneinander. | niedrig |
| 11 | 84–89, 173–182, 214, 157 | **Duplikat-Suffixe**: `arbeitsplatz_*_2` (5 Sensoren), `buro_energy_2`, `philips_lwb004_energy_2` → Hinweis auf doppelt angelegte Geräte/Integrationen. | niedrig |
| 12 | 150–151, 159–160 | Unbenannte Zeilen: `sensor.luftreiniger`, `…starkvind_…_power`, zwei IKEA-Lampen – Anzeige als Rohname. | niedrig |
| 13 | 205–219 | Abschnitt „Equipment & Sonos“ enthält auch WLED-Lightstrip und Nachttisch-Licht; „Schlafzimmer & Licht“ enthält den Luftreiniger – Überschriften passen nicht zum Inhalt. | niedrig |
| 14 | 125–135 | Saison-Deko (Schwippbogen, Schneemänner) ganzjährig gleichrangig mit Kühlschrank. | niedrig |
| 15 | 5, 222 | Eigenes Theme „Frosted Glass Dark“ nur für diese Ansicht; Rest-Eintrag `cards: []`. | niedrig |

### 2.2 Was gut ist

Sinnvolle thematische Gruppierung (Hauswirtschaft, Büro, Wohnzimmer, Schlafzimmer), vollständige Erfassung der Stecker mit Leistung, Energie, Strom, Spannung; `visible`-Beschränkung.

---

## 3. Das neue Layout (`Power_neu.yaml`)

| Bereich | Inhalt |
|---|---|
| **Jetzt** | Steckdosen gesamt (W), größter Verbraucher, aktive Geräte, Dauerläufer-Last und **Standby-Check** („mögliche Standby-Verbraucher … ≈ x € pro Jahr“) |
| **Ranking** | Tabelle nach Leistung sortiert mit Balken (█), Summe, dazu Donut-Verteilung |
| **Geräte nach Räumen** | 4 Gruppen mit Gruppensumme: *Hauswirtschaft & Küche*, *Wohnen & Deko*, *Schlafen & Luft*, *Büro, Technik & Keller*. Je Gerät eine Karte (Leistung · kWh), Farbe nach Last. Schaltbare Stecker (Waschmaschine, Trockner, Wohnzimmer) **mit Rückfrage**; Hold → Details |
| **Verlauf** | Leistung der großen Verbraucher (24 h), Verbrauch je Tag (14 Tage, gestapelt), je Monat (12 Monate) |
| **Kosten** | Hochrechnung der **Dauerläufer** (Kühlschrank, Technik-Schrank, Keller, Server): W → kWh/Jahr → €/Jahr bei 0,35 €/kWh |
| **Zähler** | Licht-, Audio- und Sonderzähler (kWh gesamt) mit lesbaren Namen |
| **Elektrische Details** | Eine Tabelle mit Leistung/Strom/Spannung/Zähler aller 14 Geräte; technische Zusatzwerte separat |

Alle 61 Entities der Originalansicht werden verwendet (keine hinzugefügt, keine weggelassen). Einheitliche deutsche Namen, einheitliche Farblogik (grau → gelb → orange → rot nach Leistung).

---

## 4. Ableitungen

### 4.1 Was die Ansicht verrät (Hypothesen)

1. **Messkonzept mit Zwischensteckern**: 14 Verbraucher in 4 Gruppen. Nicht gemessen: Heizung/Warmwasserbereitung, Herd/Backofen, Beleuchtung außerhalb der Smart-Lampen, Router (evtl. im „Technik-Schrank“).
2. **Dauerlast ist wahrscheinlich dominant**: Kühlschrank, Technik-Schrank, Keller-Stecker und der „Server“ laufen ständig; sie bestimmen vermutlich den größten Teil des gemessenen Jahresverbrauchs, während Waschmaschine/Trockner/Kaffee nur Spitzen erzeugen.
3. **Umgenutzte Hardware**: Ein Stecker mit Namen `staubsauger` versorgt einen „Server“; der Stecker `wohnzimmer` ist als „TV“ beschriftet – Namen und Realität sind auseinandergelaufen.
4. **Saisonale Geräte**: Schwippbogen und Schneemänner sind Weihnachtsdeko; ihre Leistung zeigt, ob sie außerhalb der Saison versehentlich dauerhaft laufen.
5. **Ein Hauptzähler existiert vermutlich außerhalb dieser Ansicht**: Die Startseite des Gesamt-Dashboards enthält eine `energy-distribution`-Karte, die ein konfiguriertes Energie-Dashboard voraussetzt. Falls dort ein Netzbezug hinterlegt ist, lässt sich der **nicht gemessene Rest** berechnen (Netzbezug − Σ Steckdosen).

### 4.2 Empfehlungen

| Thema | Beobachtung | Empfehlung |
|---|---|---|
| **Dauerläufer** | 4 Geräte laufen 24/7. 100 W Dauerlast ≙ 876 kWh/Jahr ≙ ca. 307 € bei 0,35 €/kWh. | Jedes Gerät prüfen: Technik-Schrank (NAS-Spindown, Netzteil-Effizienz, USV-Verluste), Server (Last, Energiesparmodus), Keller (was hängt dran?). Die Kostentabelle zeigt die Reihenfolge. |
| **Kühlschrank** | Dauermessung vorhanden. | Jahreswert aus der Monatsstatistik ablesen: moderne Geräte ~100–150 kWh/a, ältere oft 250–400 kWh/a → Austausch rechnet sich ggf. |
| **Standby** | TV, Arbeitsplatz, Schlafzimmer, Kaffeemaschine ziehen im Leerlauf. | Zeitschaltung/Automation „aus, wenn < Schwelle > 30 min“; Standby-Check im Dashboard als Kontrolle. |
| **Waschmaschine/Trockner** | Deutliche Lastspitzen, steuerbar. | „Fertig“-Benachrichtigung über den Leistungsverlauf (nach Lauf < 5 W); bei dynamischem Stromtarif Start in günstige Stunden verschieben; Rückfrage beim Abschalten (jetzt eingebaut). |
| **Saison-Deko** | Läuft evtl. ganzjährig. | Automation nach Datum (z. B. 1. Advent–6. Januar). |
| **Namen & Duplikate** | `staubsauger_*`, `arbeitsplatz_*_2`, `buro_energy_2`, `philips_lwb004_energy_2`, `wohnzimmer`=TV. | Entities umbenennen, Duplikate entfernen; danach die Namen in `Power_neu.yaml` anpassen. |
| **Nicht gemessener Rest** | Nur Steckdosen werden summiert. | Netzbezug aus dem Energie-Dashboard als Vergleichswert ergänzen; Grundlast = nächtliches Minimum. |
| **Strompreis** | In der Kostenrechnung fest 0,35 €/kWh. | Variable `price` in den Templates auf den echten Tarif setzen oder einen Preis-Helfer (`input_number`) verwenden. |
| **Licht & Audio** | Nur kWh-Zähler, keine Momentanwerte. | Verbrauch je Zeitraum über Statistik anzeigen (jetzt Monatsdiagramm für Geräte mit Energiesensor; für Lampen/Sonos bei Bedarf ergänzen). |

---

## 5. Einrichtung, Annahmen, Grenzen

**Einrichtung:** Dashboard bearbeiten → Tab „Energy“ → Stift → ⋮ → *In YAML bearbeiten* → Inhalt von `Power_neu.yaml` einfügen. Benötigt: **mushroom**, **mini-graph-card**, **apexcharts-card** (alle schon im Einsatz). Keine Navigation, keine URL-Anpassung.

**Geprüft:** YAML gültig; alle 61 Entity-IDs stammen aus der Originalansicht; alle 42 Templates in 4 Szenarien (leer / Problemfall / Normal / alles ok) mit strengem `float()`-Verhalten fehlerfrei gerendert.

**Nicht geprüft** (kein Zugriff auf die HA-Instanz): Darstellung und echte Werte. Annahmen:
* Die `…_energy`-Sensoren sind kumulative kWh-Zähler (`total_increasing`); nur dann liefern „Verbrauch je Tag/Monat“ sinnvolle Werte. Bei `sensor.wzschwippbogen` und `sensor.schwippschneemann` (laut Namen „Verbrauch“) bitte prüfen.
* **Dauerläufer** = Kühlschrank, Technik-Schrank, Keller (Shelly), Server – aus der Benennung abgeleitet.
* Standby-Bereich 0,5–20 W und Strompreis 0,35 €/kWh sind Annahmen und im Template anpassbar; ein laufender Fernseher fällt ebenfalls in diesen Bereich (daher „mögliche“ Standby-Verbraucher).
* Die Hochrechnung nimmt an, dass die aktuelle Leistung das ganze Jahr anliegt – beim Kühlschrank (Kompressorzyklen) nur Größenordnung.
* „Steckdosen gesamt“ ist die gemessene Summe, nicht der Hausverbrauch.
