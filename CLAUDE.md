# Spielesammlung — index.html

Eine einzelne, eigenständige HTML-Datei mit mehreren Lernspielen für Mathematik
und Physik am Beamer (Gymnasium Schleswig-Holstein). Läuft ohne Build, ohne
Server, ohne Internet — direkt per Doppelklick, auch vom USB-Stick.

Die ganze Sammlung steckt in `index.html`. Der Name kommt daher, dass GitHub
Pages genau diese Datei aus dem Repo-Root ausliefert (siehe *Veröffentlichung*),
und er ist zugleich der Grund, warum es nur eine einzige Quelle gibt. Historisch
hieß sie `Jeopardy_main.html` und enthielt nur das Jeopardy — beim Erweitern
also nicht vom Namen „Jeopardy" im Repo oder in alten Links irreleiten lassen.

## Grundregeln

- **Single-File bleibt.** Kein Aufteilen in Module, kein Bundler, kein npm.
  Das ist Absicht: fremder Rechner, kein Netz, kein Build.
- **Keine externen Libraries.** Einzige Ausnahme sind die Google Fonts im `<head>`.
  Ohne Netz greifen die Fallbacks, das Spiel funktioniert trotzdem.
- **ES5-Stil**, wie im Bestand: `var`, `function(){}`, kein `const`/Arrow/Template-String.
  Alles steckt in einer IIFE mit `"use strict"`.
- **Deutsche Bezeichner** für Fachliches (`bsatz`, `zeigeSeite`, `aufgaben`, `ziel`),
  englische nur für Technisches (`state`, `save`, `timer`).
- **JS- und CSS-Kommentare ohne Umlaute** (`Schluessel`, `Aufloesung`, `Laengenschaetzung`).
  Sichtbare Texte für die Klasse dagegen mit korrekten Umlauten und „…" -Anführungszeichen.
- **Farben nur über CSS-Variablen** aus `:root` (`--mathe`, `--fg-2`, `--line-1`, …),
  nie direkte Hexwerte im Markup.

## Landkarte

Die Datei ist ~6.800 Zeilen. **Nie komplett einlesen** — das kostet rund zwei
Drittel eines Kontextfensters. Stattdessen nach den Banner-Kommentaren greppen;
Zeilennummern driften bei jedem Edit, die Banner nicht.

```
ICONS (Lucide-Stil)      ic() und die Pfad-Sammlung ICONS
ABBILDUNGEN              plot(), Geometrie- und Diagramm-SVGs
AUFGABEN                 Jeopardy-Fragen, Katalog 1
FRAGENKATALOG 2 … 9      weitere Jeopardy-Kataloge
FRAGENKATALOGE           Registry KATALOGE (Fach/Stufe/Inhalt)
ZUSTAND                  state, STORAGE_KEY, el()
ORDNER LADEN             Klassenordner „classes" vom Namenstrainer
FRAGENKATALOG            Kaskade Fach → Stufe → Inhalt
GRUPPEN / ANWESENHEIT    Auslosen, Fehlende, Nachtragen
SPIEL / BETRIEBSART      Jeopardy-Board
TIMER                    fmt(), beep()
FRAGE / WERTUNG          Overlay und Punktvergabe
SIEGERPODEST             Endanimation
RESET UND SPEICHERN      save(), loadSaved(), askConfirm()
BINGO                    Kopfkommentar mit Satz-Aufbau, dann die Aufgabensaetze
BINGO: Laengenschaetzung B_PK, Binomialverteilung
BINGO: Zustand …         bstate, Timer, Anzeige, Ablauf, Auswertung, Vorbereitung
FEHLERJAGD               Kopfkommentar mit Satz-Aufbau, dann die Aufgabensaetze
FEHLERJAGD: Zustand …    fstate, Anzeige, Ablauf, Speichern, Vorbereitung
STARTSEITE               Registry SPIELE, Kacheln, Thumbnails
SEITENWECHSEL            SEITEN, zeigeSeite()
```

Im `<body>` gibt es sieben `<section>`: `startseite`, `setup`, `bingoSetup`,
`game`, `bingoGame`, `fehlerSetup`, `fehlerGame`. Dazu Overlays (`qOverlay`,
`podestOverlay`, `bCheckOverlay`, `confirmOverlay`) außerhalb von `.app`.

## Geteilte Infrastruktur

| Helfer | Zweck |
|---|---|
| `el(id)` | `document.getElementById`, überall statt des langen Aufrufs |
| `ic(name, size)` | Inline-SVG-Icon aus `ICONS`, erbt `currentColor` |
| `M(s)` | Formelsatz, Variablen kursiv in Cambria Math |
| `plot(...)` | Funktionsgraph als SVG |
| `fmt(ms)` | Millisekunden → `m:ss` |
| `beep()` | Ton bei Timer-Ende, per WebAudio |
| `askConfirm(titel, text, fn)` | Bestätigungsdialog statt `confirm()` |
| `zeigeSeite(name)` | Blendet genau eine Section ein, setzt Kopfzeile und Titel |

CSS-Bausteine, die jedes Spiel nutzt: `.card`, `.label`, `.sub`, `.note`, `.row`,
`.btn` (`.btn-primary`, `.btn-quiet`, `.btn-icon`), `.katalogwahl` + `.feld` für
Dropdown-Reihen, `.timerbox`, `.overlay` + `.dialog`, `.sep`.

Jedes Spiel hat seinen **eigenen** Zustand und localStorage-Key:

| Spiel | Zustand | Key |
|---|---|---|
| Jeopardy | `state` | `jeopardy_q1_wdh_e_v1` |
| Bingo | `bstate` | `bingo_klasse7_v2` |
| Fehlerjagd | `fstate` | `fehlerjagd_v1` |

Die Uhr ist die Ausnahme: `bTimerStart(sek, label, prefix)` steuert seine
Timerbox über das ID-Präfix und ist damit ohnehin allgemein. Die Fehlerjagd
hängt sich als dritte Box mit `"fTimer"` daran, statt eine eigene Uhr zu bauen.

## Rezept: neues Spiel ergänzen

1. `<section id="xySetup">` und `<section id="xyGame">` in `.app` anlegen,
   beide mit `style="display:none"`, aufgebaut aus `.card`-Blöcken mit
   `Schritt 1`/`Schritt 2`-Labels wie beim Bingo.
2. Banner-Kommentar `XY` im Skript, darunter Daten, Zustand, Ablauf, Bedienung —
   in derselben Reihenfolge wie beim Bingo.
3. Eigenes `xystate`-Objekt und eigener `XY_KEY` für localStorage.
4. Beide Section-IDs in das Array `SEITEN` eintragen, sonst blendet
   `zeigeSeite()` sie nie aus.
5. Kopfzeilen-Zweig in `zeigeSeite()` ergänzen (`el("kopfTitel").textContent`).
6. Thumbnail-Funktion `thumbXy()` schreiben, die ein `<svg>` zurückgibt —
   Vorbilder sind `thumbJeopardy` und `thumbBingo` direkt über `SPIELE`.
7. Eintrag in `SPIELE` anhängen: `id`, `name`, `icon`, `bild`, `text`,
   `badge` (zeigt „Angefangene Runde", wenn gespeicherter Stand existiert),
   `start`.

Fehlt Schritt 4 oder 7, ist das Spiel unerreichbar, ohne dass ein Fehler auftritt.

## Rezept: Inhalte ergänzen

**Bingo-Aufgabensatz.** Objekt mit `stufe`, `titel`, `unter`, `pool`, `aufgaben`
und **hinten** an `B_SAETZE` anhängen. `pool` braucht **genau 16** Ergebnisse,
`aufgaben` **genau zwei** je Poolzahl (`{e:Poolschlüssel, q:Term als HTML, s:Rechenweg}`).
Brüche im Pool als `{k:Schlüssel, h:HTML}` über die Helfer `f(z,n)` und `nf(z,n)`.
Die Einstellungsseite baut die Stufen-Dropdowns automatisch aus dem Feld `stufe`.

**Fehlerjagd-Aufgabensatz.** Objekt mit `fach`, `stufe`, `titel`, `unter`,
`aufgaben` und an `F_SAETZE` anhängen. `fach` ist „Mathematik" oder „Physik"
(fehlt es, gilt Mathematik), `stufe` ist `"5"` bis `"10"`, `"E"`, `"Q1"` oder
`"Q2"`; die Einstellungsseite baut daraus die Kaskade Fach → Stufe → Satz und
sortiert die Oberstufe hinter Klasse 10. Jede Aufgabe braucht `auftrag`, `zeilen`, `fehler`
(Zeilennummer, 1-basiert), `art`, `richtig` und `s`; `start` ist optional und
bleibt leer, wenn schon die erste nummerierte Zeile die gegebene Gleichung ist.
Zeilen sind Strings oder `{t:Zeile, op:Umformung am Rand}`. **Alles unterhalb
der Fehlerzeile muss aus der falschen Zeile sauber weitergerechnet sein**, sonst
ist „die erste falsche Zeile“ nicht mehr eindeutig — das ist die eigentliche
Sorgfaltsstelle beim Schreiben neuer Sätze.

**Jeopardy-Katalog.** Kategorien-Array anlegen, dann Eintrag in `KATALOGE` mit
`id`, `titel`, `fach`, `stufe`, `inhalt`, `unter`, `hinweis`, `bezug`,
optional `farben`. Die Kaskade Fach → Stufe → Inhalt entsteht daraus von selbst.

## Fallen

- **Gespeicherte Indizes.** `state.katalog`, `bstate.satz` und `fstate.satz`
  landen als *Zahl* im localStorage. Neue Einträge deshalb immer **ans Ende**
  von `KATALOGE`, `B_SAETZE` bzw. `F_SAETZE`, sonst zeigt eine unterbrochene
  Runde nach dem Update auf den falschen Satz. Die Anzeigereihenfolge entsteht ohnehin erst beim Aufbau der
  Dropdowns.
- **`B_PK` gilt nur für 16er-Pools.** Die Tabelle zur Längenschätzung ist für
  Pool 16 und 3×3-Karte exakt vorberechnet. Ein Satz mit anderer Poolgröße
  macht die Zeitschätzung still falsch.
- **Storage-Keys nicht umbenennen** ohne Grund — das verwirft laufende Runden
  auf dem Lehrerrechner. `bingo_klasse7_v2` heißt trotz Klasse-5-Sätzen weiter so.
- **Umlaute in Poolschlüsseln vermeiden.** Der Schlüssel ist zugleich
  Vergleichswert; Minuszeichen sind typografisch (`−`, U+2212), nicht ASCII.

## Testen

Kein Testframework. Stattdessen `index.html` aus diesem Repo-Ordner im
Browser-Pane öffnen — als `file:///`-URL mit dem Pfad des jeweiligen Rechners,
der Ordner liegt auf Arbeits- und Heimrechner unter verschiedenen Benutzernamen.
Dann per `javascript_tool` durchklicken: Spielkachel, Einstellungen, Runde
starten, durchspielen bis zur Auflösung. Danach `read_console_messages` auf
Fehler prüfen.

Die Bingo-Einstellungsseite hat eine **eingebaute Selbstkontrolle**: `bSatzInfo`
meldet „16 Ergebnisse, 32 Aufgaben" und warnt bei Aufgaben ohne Poolzahl oder
Poolzahlen ohne Aufgabe. Nach jedem neuen Satz einmal alle Stufen durchschalten
und diese Zeile lesen. Die Fehlerjagd hat dieselbe Zeile als `fSatzInfo`; sie
warnt, wenn eine Fehlerzeile außerhalb des Lösungswegs zeigt oder Fehlerart,
Korrektur oder Erläuterung fehlen.

Bei neuen Rechenaufgaben zusätzlich die Terme maschinell nachrechnen, statt sie
nur zu überfliegen — die Terme sind gültiges JS, wenn man `·`→`*`, `:`→`/`
und `−`→`-` ersetzt.

## Veröffentlichung

Die Sammlung läuft doppelt: lokal aus OneDrive am Beamer und online für
Kolleg:innen unter <https://timkucz-cmyk.github.io/jeopardy/>. GitHub Pages
liefert dabei `index.html` aus dem Repo-Root aus.

Dieser Ordner **ist** das Repository, es gibt also bewusst keine zweite Kopie.
Eine Änderung hier ist nach dem Push zugleich die Version, die die Kolleg:innen
sehen. Beides soll denselben Stand haben.

Deshalb: **nach einer abgeschlossenen Änderung committen und pushen**, nicht nach
jedem einzelnen Edit — ein halbfertiger Zwischenstand wäre sonst sofort online.
Vorher immer die Testrunde aus dem Abschnitt oben durchspielen.

Zugangsdaten gibt Claude nicht ein. Ist der Push nicht authentifiziert, meldet
git das; dann übernimmt die Anmeldung der Nutzer selbst.
