# Raumstruktur

![Symcon](https://img.shields.io/badge/Symcon-PHPModul-blue)
![Modul Version](https://img.shields.io/badge/Modul_Version-0.5.0-blue)
![Symcon Version](https://img.shields.io/badge/Symcon_Version-9.0%2B-blue)
![License](https://img.shields.io/badge/License-PolyForm_Noncommercial_1.0.0-lightgrey)
![Check Style](https://github.com/DG65/Raumstruktur/actions/workflows/check-style.yml/badge.svg)
[![PayPal](https://img.shields.io/badge/PayPal-Me-blue?logo=paypal)](https://paypal.me/DietmarGureth)

**DG65-Toolkit Raumstruktur** — Etagen und Räume des Objektbaums maschinenlesbar machen und
bei Bedarf als Kategorie-Gerüst anlegen. Das Modul hieß bis 13.10.2026 „Katasteramt“
(davor „StrukturHub“) und wurde auf seinen Funktionsnamen umbenannt: PHP-Klasse und
Modulordner `Raumstruktur`, Funktions-Präfix **`RAUM_`** (vormals `STRUKT_`), Repo
`DG65/Raumstruktur`. Die GUIDs von Bibliothek und Modul blieben gleich, bestehende Instanzen
samt Eigenschaften bleiben beim Umstieg erhalten. Der Vertrag `RAUM_GetStructure()` hat
dasselbe Rückgabeformat wie früher `STRUKT_GetStructure()`.

IP-Symcon bietet keinen Konfigurationsrahmen für den Objektbaum — jede Installation
strukturiert Räume/Etagen/Geräte anders, und andere Module können Geräte deshalb nicht
zuverlässig eingruppieren. **Raumstruktur** macht eine bestehende Objektbaum-Struktur
maschinenlesbar, damit Partnermodule (Raumzähler-Zuordnung, Dashboard-Gruppierung,
Lastmanagement-Eingruppierung, …) sie automatisch abfragen können, statt jeweils eigene
Heuristiken zu bauen.

## Drei Funktionen

- **v0.1 — Auskunft über Bestehendes, read-only.** Der Nutzer zeigt einmal im Formular, wo
  seine Struktur liegt (Wurzelkategorie der Räume, welche Unterkategorien Etagen sind).
  Raumstruktur liefert das als Vertrag. Legt nichts an, baut nichts um — gefahrlos
  installierbar.
- **v0.2 — Struktur anlegen.** Legt Etagen-/Raum-Kategorien nach der
  Verbund-Konvention an — Panel „🏗️ Struktur anlegen" im Formular. Alle Begriffe frei
  wählbar (Etage/Stockwerk/Geschoss, Raum/Büro/Zimmer), per Präfix+Nummernbereich auch in
  Masse (privat wie gewerblich), Nummer wahlweise vor oder nach dem Namen ("101 Büro" /
  "Büro 101"). Legt ausschließlich Kategorien an — **keine** Geräte-Instanzen/Links (die
  verknüpft man wie gewohnt selbst). Idempotent: mehrfaches Anlegen mit denselben Zeilen
  erzeugt keine Duplikate. Baut auf dem v0.1-Vertrag auf (das Erzeugte liest v0.1 automatisch
  korrekt ein, inklusive der daraus ableitbaren Raumnummer, siehe `number`-Feld unten).
- **v0.4 — die Namensprüfung.** Prüft Etagen-/Raumnamen auf Konsistenz — Panel
  „🏛️ Namensprüfung" im Formular. Rein **beschreibend**: die Mehrheit deiner eigenen Namen
  bestimmt, was „die Konvention" gerade ist, nichts wird fest vorgegeben (keine eigene Anlage
  als Norm). Prüft Zahlenposition, Groß-/Kleinschreibung, doppelte und sehr kurze/kryptische
  Namen. Zeigt Korrekturvorschläge, benennt aber **nur** die vom Nutzer angehakten Zeilen
  tatsächlich um (`IPS_SetName()`) — nichts geschieht automatisch.

## Vertrag `RAUM_GetStructure($id): string`

Rückgabe ist ein **JSON-STRING** (kein PHP-Array) — beim Aufrufer
`json_decode(RAUM_GetStructure($id), true)` verwenden.

```json
{
  "contractVersion": "1.1",
  "instanceID": 12345,
  "structureChangedAt": 1787900000,
  "levels": [ { "key": "eg", "label": "Erdgeschoss", "categoryID": 23050, "order": 0, "number": null } ],
  "rooms": [
    {
      "key": "kueche",
      "label": "101 Küche",
      "level": "eg",
      "categoryID": 51304,
      "order": 0,
      "roomType": "kueche",
      "number": "101",
      "deviceInstanceIDs": [30131, 27898, 48294]
    }
  ]
}
```

- **`key`** (levels UND rooms) ist garantiert: nur Zeichen aus `[a-z0-9_]` (Umlaute
  transliteriert), **eindeutig über levels UND rooms hinweg** (ein gemeinsamer Namensraum) und
  **stabil über eine Umbenennung hinweg** — der Key wird beim ersten Erfassen einer Kategorie
  aus ihrem damaligen Namen abgeleitet und danach dauerhaft an der `categoryID` festgemacht,
  nie bei jedem Aufruf neu aus dem aktuellen Label berechnet. Konsumenten dürfen daraus
  abgeleitete Idents/Variablen dauerhaft anlegen (z. B. `kueche_sum_power`), ohne dass eine
  spätere Umbenennung im Objektbaum sie verwaist.
- `deviceInstanceIDs` ist bereits **dedupliziert und um tote/namenlose Links bereinigt** —
  Konsumenten müssen das nicht selbst nochmal lösen. Ein Link, dessen Ziel eine **Variable**
  (statt einer Instanz) ist, wird auf deren **Elterninstanz** aufgelöst (z. B. ein "Licht"-Link
  direkt auf eine Schalter-Variable eines Aktors) — nur ein Link/eine Variable ganz ohne
  Instanz-Elternteil wird ignoriert. Wird bei jedem Aufruf live aus dem aktuellen Objektbaum
  berechnet (keine Zwischenspeicherung) — bei unverändertem Objektbaum und unveränderter
  Etagen-Auswahl liefert die Methode deterministisch dasselbe Ergebnis.
- `levels` ist leer, wenn keine Etagen-Ebene bestätigt wurde; `rooms[].level` ist dann `""`
  (Räume liegen direkt unter der Wurzelkategorie).
- `order` ist die vom Nutzer im Symcon-Objektbaum gesetzte Reihenfolge (Konsolen-Drag&Drop) —
  robuster als Array-Reihenfolge oder alphabetisches Sortieren nach `key`/`label`.
- `roomType` ist eine **heuristische Best-Effort-Ableitung** aus dem Raumnamen (z. B. für eine
  Icon-Auswahl), `null` wenn nicht erkannt — kein Fachwert, nur eine Anzeige-Hilfe.
- `number` (levels UND rooms, seit `contractVersion` 1.1) ist eine **heuristische
  Best-Effort-Ableitung** einer Geschoss-/Raumnummer aus dem Namen — die Zahl darf vor oder
  nach dem Namen stehen ("101 Küche"/"Küche 101"), mit oder ohne Trenner. **String, nicht
  Zahl** (führende Nullen bleiben erhalten), `null` wenn keine Nummer erkennbar. Funktioniert
  für jede Kategorie, nicht nur über den Struktur anlegen erzeugte — gedacht z. B. für
  Konsumenten, die Geräte-Idents aus der Raumnummer ableiten wollen.
- `structureChangedAt` ändert sich **nur**, wenn sich `levels`/`rooms` inhaltlich seit dem
  letzten Aufruf tatsächlich geändert haben (interner Hash-Vergleich) — Konsumenten können
  diesen einen Wert pollen/vergleichen, statt das komplette JSON zu diffen. Es gibt **keinen**
  Push-Mechanismus (kein Event/keine Nachricht bei Änderung).
- `instanceID` ist die ID dieser Raumstruktur-Instanz (Debugging/Logging).
- Immer hinter `function_exists('RAUM_GetStructure')` aufrufen — jedes Partnermodul ist
  optional, ohne Raumstruktur installiert bleibt jedes andere Modul unverändert funktionsfähig.

## Mehrere Instanzen

Mehrere Raumstruktur-Instanzen sind ausdrücklich zulässig (z. B. Haupthaus + Nebengebäude mit
getrennter Wurzelkategorie) — **keine Singleton-Annahme**. Konsumenten iterieren über ALLE
Instanzen von `IPS_GetInstanceListByModuleID('{CA700334-0982-F356-0617-6952868137E9}')`, nicht
nur die erste gefundene.

## Was Raumstruktur NICHT tut (v0.1)

Keine automatische Etagen-Erkennung — auf derselben Ebene wie Etagen können auch
Gewerke-Kategorien liegen (Energie, Heizung, Test, …), das lässt sich nicht zuverlässig
raten. Der Nutzer bestätigt einmal im Formular, welche Kategorien Etagen sind.

## Lizenz

PolyForm Noncommercial 1.0.0, siehe [LICENSE](LICENSE). Spenden willkommen:
[paypal.me/DietmarGureth](https://paypal.me/DietmarGureth).

---

Teil des **DG65 Toolkit** — eigenständige, domänenfreie Symcon-Bausteine von DG65, ohne
Energiebezug. Geschwister-Module (Stand 08.09.2026): **Destille**
([GleitenderMittelwert](https://github.com/DG65/NRGGleitenderMittelwert)), **Pförtner**
([Tueroeffner](https://github.com/DG65/Pfoertner)), **Vormund** (MigrationsHub). Kein
NRG-Stack-Modul, auch wenn erste Konsumenten des `RAUM_GetStructure()`-Vertrags (MeterHub,
Dashboard, EMS) aus NRG-Stack kommen — das ist Reihenfolge, keine fachliche Zugehörigkeit.
