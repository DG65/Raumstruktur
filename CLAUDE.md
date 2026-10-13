# Hinweise für die Arbeit an diesem Repository

## Name: Raumstruktur (DG65-Toolkit) — seit 13.10.2026 vollständig umbenannt

Dietmar (13.10.2026, siehe TOOLKIT.md „Umbenennung auf Funktionsnamen"): Toolkit-Module sollen
sagen, was sie tun. Aus „Katasteramt" (davor „StrukturHub") wurde **„DG65-Toolkit Raumstruktur"**:
PHP-Klasse + Modulordner `Raumstruktur`, `module.json→name` = `Raumstruktur`, Präfix **`RAUM`**
(Funktionen `RAUM_GetStructure()` usw., vormals `STRUKT_`), Repo `DG65/Raumstruktur`
(GitHub-Umbenennung, alter Name `Toolkit-Katasteramt` wird NIE wiederverwendet). **GUIDs von
Bibliothek und Modul blieben**, damit Dietmars Instanz samt Eigenschaften erhalten bleibt; genau ein
Alias = „DG65-Toolkit Raumstruktur". Sub-Funktionen heißen jetzt „Struktur anlegen" (vormals
„Baumeister") und „Namensprüfung" (vormals „Standesamt"). Die Einfrier-Faustregel aus TOOLKIT.md
gilt für den Namen NACH dieser Umbenennung unverändert.

Vertrag: `RAUM_GetStructure()` mit identischem Rückgabeformat, contractVersion unverändert (nur der
Präfix wanderte). Konsumenten stellen auf „RAUM_ zuerst, STRUKT_ als Rückfall" um.

**Verbindliches Manifest für DG65 Toolkit:** `/Users/dietmar/Nextcloud/Claude/TOOLKIT.md`
(analog zu SUITE.md bei NRG-Stack) — Branding-Faustregel, Branch-Strategie, Cross-Modul-
Verträge, Koordinationsrolle, Ideenfindungs-Prozess. Für alles Symcon-Allgemeine verweist
TOOLKIT.md selbst weiter auf SUITE.md.

## Verwandte Repositories

Teil desselben Modul-Verbunds (NRG-Stack, DG65), an mehreren wird teils gleichzeitig in
getrennten Sitzungen gearbeitet:

- **Raumstruktur** (dieses Repo, vormals Katasteramt/StrukturHub): Objektbaum-Struktur maschinenlesbar machen —
  https://github.com/DG65/Toolkit-Raumstruktur
- **EMS**: koordinierende Instanz, Verbund-Manifest (lokale SUITE.md, siehe unten) — https://github.com/DG65/NRGEMS
- **MeterHub**: erster geplanter Konsument (Raumzähler-Assistent: kaskadierte virtuelle
  Raumzähler gegen `RAUM_GetStructure()`)
- **NRGDashboard**: geplanter Konsument (Gruppierung nach Räumen/Etagen)

## Rolle und Grundregeln

1. **Reine Auskunft, kein Regler.** Raumstruktur liest die bestehende Struktur und publiziert
   sie über `RAUM_GetStructure`. v0.1 legt nichts an, ändert nichts — v0.2 (die Strukturanlage)
   ist ein separater, späterer Schritt.
2. **Eigenständigkeit.** Raumstruktur setzt kein anderes Modul voraus und wird selbst von
   keinem vorausgesetzt — jeder Fremdaufruf (sollte in v0.2+ nötig werden) hinter
   `function_exists()`. Prüfwerkzeug: `.tools/check-standalone.php`.
3. **Vertrag ist öffentliche API.** Einmal veröffentlichte Feldnamen in
   `RAUM_GetStructure()` werden nie umbenannt, nur additiv erweitert (Minor hoch), Bruch
   nur mit Major + Ankündigung.
4. **Rückgabe ist ein JSON-STRING, kein Array** — genau an dieser Stelle sind Dashboard und
   ChargerHub beim MeterHub-Vertrag schon je einmal gestolpert (`is_array()` auf einem String
   schlägt still fehl). Bei jeder Erweiterung diesen Hinweis in Doku-Kommentar/README
   mitführen.
8. **`key` ist ein garantiert stabiler Ident-Bestandteil** (MeterHub-Anforderung 28.08.2026,
   siehe `resolveKey()`/`KeyRegistry`-Attribut in `module.php`) — NIEMALS zurückbauen auf
   "live aus dem aktuellen Label berechnen". Ein Konsument leitet daraus dauerhafte
   Variablen-Idents ab; ein sich änderender Key würde bei jeder Umbenennung im Objektbaum
   Archiv-Historie verwaisen lassen. Neue Kategorien bekommen ihren Key beim ersten Erfassen,
   danach ist er an der `categoryID` fixiert, bis die Kategorie in Symcon gelöscht wird.
5. **Keine automatische Etagen-Erkennung.** Auf derselben Objektbaum-Ebene wie Etagen können
   Gewerke-Kategorien liegen (Energie, Heizung, Test, …) — das ist bei Dietmars eigener Anlage
   live beobachtet. Der Nutzer bestätigt einmal im Formular, welche Kategorien Etagen sind;
   raten wäre hier Grundregel-Verstoß ("keine eigene Anlage als Norm", siehe SUITE.md).
6. **Sprachregel:** alles Nutzersichtbare auf Deutsch, keine vermeidbaren Anglizismen; Idents,
   Property-/Methodennamen und feststehende Fachbegriffe ausgenommen.
7. **Store-/Stable-Regeln von Anfang an**: Formular-Buttons nur per `UpdateFormField`/`echo`
   (nie `IPS_SetProperty`+`ApplyChanges` selbst), `vendor=""` (reines Software-Modul),
   `library.json` nur mit den erlaubten Schlüsseln, keine Emojis in `Translate()`-Strings,
   Klassenname = Modulname.

## Eigenständigkeit prüfen: `.tools/check-standalone.php`

```
php .tools/check-standalone.php    # 0 = sauber, 1 = ungesicherter Fremdaufruf
```

Herkunft und Funktionsweise wie in den übrigen Hub-Repos — Prüflogik gleich halten.

## Verbund-Manifest SUITE.md — Bezugsquelle (geändert 31.08.2026)

SUITE.md liegt seit 31.08.2026 NICHT mehr in einem GitHub-Repo (die
Modul-Repos sind öffentlich, SUITE.md enthält das komplette Architektur-/
Debugging-Know-how des Verbunds — Dietmars Entscheidung). Primärquelle ist
ausschließlich die lokale Datei `/Users/dietmar/Nextcloud/Claude/SUITE.md`
auf Dietmars Maschine, versioniert in einem eigenen lokalen Git-Repo ohne
Remote. Frühere Kopien wurden zusätzlich aus der Historie aller Modul-Repos
entfernt (`git filter-repo` + Force-Push). Bei Konventionsfragen zuerst dort
nachlesen, nicht Code zwischen Modulen vergleichen.

## Roadmap

- **v0.1:** Read-only-Auskunft über bestehende Struktur. Live an Dietmars Anlage verifiziert.
- **v0.4 (aktuell):** die Namensprüfung — Namenskonventions-Berater (`RunNamingCheck`/
  `ApplyNamingFixes`/`analyzeNamingConventions` in `module.php`, Panel „🏛️ Namensprüfung"). Rein
  DESKRIPTIV (Mehrheit der eigenen Namen = Konvention, siehe "keine eigene Anlage als Norm"),
  vier Prüfungen (Zahlenposition/Case/Duplikate/Kurzlabel), Umbenennen nur für vom Nutzer
  angehakte Zeilen. Dabei nebenbei einen echten Bug gefunden+gefixt: `injectPreview()` (und
  jetzt auch der neue `injectNamensprüfungValues()`) durchsuchten/durchsuchen das Formular
  rekursiv (`findFormElementByName()`), weil `StructurePreview` in einem `ExpansionPanel`
  verschachtelt liegt — die alte, rein flache Suche fand es beim ERSTEN Formularöffnen nie,
  nur nachträglich per `UpdateFormField()` nach einem Button-Klick.
- **v0.2:** die Strukturanlage — legt Etagen-/Raum-Kategorien nach Konvention an
  (`AddLevelRows`/`AddRoomRows`/`PreviewSkeleton`/`BuildSkeleton` in `module.php`, Panel
  „🏗️ Strukturanlage" im Formular). Profitiert vom in v0.1 etablierten Vertrag (das
  Erzeugte ist per Definition vertragskonform — v0.1 liest die neuen Kategorien ohne Änderung
  korrekt ein).

  **Anforderung Dietmar (28.08.2026), vor Baubeginn festgehalten — Umsetzung s. u.:** Der
  Generator darf NICHT nur den privaten Wohnhaus-Fall abdecken (Grundregel "keine eigene
  Anlage als Norm", siehe SUITE.md) — er muss auch den **gewerblichen Bereich** bedienen
  können: ganze Bürogebäude/Etagen/Räume **in Masse** anlegen, inkl. Raumnummern, alle
  Begriffe/Strukturannahmen **konfigurierbar/abfragbar**, nicht hartkodiert.
  - **Terminologie:** frei — Etagen/Räume sind reine Freitext-Zeilen (`GenLevels`/`GenRooms`),
    keine festen Wörter wie "Etage"/"Raum" im Datenmodell, nur als Platzhalter-Beispiele im
    Formular.
  - **Masseneingabe:** `AddLevelRows`/`AddRoomRows` fügen per Präfix+Start+Ende viele Zeilen
    auf einmal ein (z. B. "Büro" 101–150) — Raumnummer ist dabei einfach Teil des generierten
    Labels, kein eigenes Feld (bewusste Scope-Entscheidung, siehe Plan
    `~/.claude/plans/gentle-inventing-puddle.md`: erst bei konkretem Konsumenten-Bedarf ein
    strukturiertes Feld einführen).
  - `inferRoomType()` (v0.1) deckt bereits privates UND gewerbliches Vokabular ab — vom
    Generator unverändert weiterverwendet (v0.1 liest die generierten Labels automatisch ein,
    keine Duplikation nötig).
  - **Nummer vor/nach dem Namen, Nummern-Vererbung Etage→Raum→Gerät (28.08.2026):**
    Dietmars Ergänzung — Geschoss-/Raumnummern können vor ODER nach dem Namen stehen, und
    Geräte-Nummern (Schalter/Steckdosen) leiten sich mitunter aus der Raumnummer ab, die
    wiederum aus der Geschossnummer abgeleitet sein kann. Umgesetzt: Generator-Massenhilfe hat
    ein Nummer-Position-Feld (vorne/hinten); `RAUM_GetStructure()` liefert seit
    `contractVersion` 1.1 ein zusätzliches, heuristisch aus dem Namen abgeleitetes `number`-Feld
    (levels UND rooms, siehe `extractNumber()`) — funktioniert für JEDE Kategorie, nicht nur
    generator-erzeugte. Geräte-Nummern selbst bleiben außerhalb des Scopes (Raumstruktur legt
    keine Geräte an); Konsumenten, die Geräte benennen, können `number` dafür nutzen.

  **Bewusst nicht in v0.2 enthalten** (siehe Plan-Datei für die Begründung): keine
  Geräte-Instanzen/Links, keine automatische Wurzelkategorie-Erzeugung, kein
  Zero-Padding/CSV-Import bei der Massen-Erzeugung.
