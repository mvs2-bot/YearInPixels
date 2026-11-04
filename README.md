# YearInPixels

Single-File-PWA aus der mvs2-App-Familie: dein Jahr als Pixel-Tracker im Bullet-Journal-Stil.
YearInPixels erfasst selbst nichts. Es liest nur deine eigenen Einträge aus DailyMind, dem Migräne Tracker,
dem PegelTracker und TravelBase und malt daraus ein 12 × 31-Raster pro Ebene.

URL: https://mvs2-bot.github.io/YearInPixels/

## Dateien

| Datei | Inhalt |
| --- | --- |
| `index.html` | die komplette App: HTML, eigenes CSS, Alpine.js und supabase-js über CDN |
| `manifest.json`, `sw.js` | installierbar, App-Hülle offline verfügbar |
| `icon.svg`, `icon.png`, `apple-touch-icon.png`, `icons/` | App-Icon: bunte Pixel im Schachbrett auf Beige mit Langschatten, maskable und Favicon |
| `shared/ios-nav.js` | Kopie aus `mvs2-bot.github.io` (Zurück/Neu laden in der iOS-Web-App) |
| Datenbank | Migration `20261104090000_yearinpixels_core` im Repo `supabase-core` |

## Ebenen und Quellen

| Ebene | Quelle | Wert pro Tag | Farbstufen (gut → schlecht) |
| --- | --- | --- | --- |
| Stimmung | `dailymind_entries.mood` | 1–5 | strahlend, gut, okay, trüb, schwer |
| Energie | `dailymind_entries.energy` | 1–5 | voller Energie … sehr niedrig |
| Migräne | `migralog_entries.pain` | 0–5 (DB 0–10, über 5 zählt als 5) | frei, 1 … 5 stark |
| Alkohol | `pegel_drinks` | Standardgetränke (10 g) = ml × Vol-% / 100 × 0,8 / 10 | 0, 1–2, 3–4, 5–6, 7–8, 9–10, > 10 |
| Tschick | `dailymind_entries.tschick` + `tschick_tags` | 0–5 (5 = „5 oder mehr“) | keine, 1 … 5 oder mehr |
| Koffein | `pegel_drinks.caffeine_mg` | mg | 0, < 100, 100–199, 200–299, 300–399, ≥ 400 |
| Reisen | RPC `travelbase_my_trips()` | Reise an diesem Tag | eine Farbe je Reise |

- Eine Ebene erscheint nur, wenn es dazu Daten gibt.
- Alkohol und Koffein zählen ab dem ersten Getränk mit Alkohol bzw. Koffein; Tage ohne Getränk sind dann 0.
- Tagesgrenze (Standard 06:00, einstellbar 00–12 Uhr): ein Getränk davor zählt zum Vortag.
- Fehlen die Tschick-Spalten in der Datenbank, liest die App DailyMind ohne sie weiter.
- Ohne Rolle in einer Quell-App wird diese Quelle nicht abgefragt (Prüfung über `*_is_member`).

## Ansichten

- **Jahr**: Ebenen als Klebeband-Kacheln (Wischen über dem Raster wechselt die Ebene), Pixel-Raster,
  Farbzählung als 20 × 5-Waffel mit Liste, Tagesblatt mit allen Ebenen eines Tages, nach App gruppiert.
- **Übersicht**: Kennzahlen mit Vorjahr, Vergleiche (Stimmung nach dem Vorabend, Wann getschickt?,
  Migräne nach Koffein, Unterwegs oder zu Hause?, Energie je Wochentag), Stimmung je Monat, Jahrestabelle, alle Ebenen klein.
- **Ebenen**: ein/aus, Reihenfolge, Tagesgrenze. Gespeichert in `yearinpixels_settings`.

## Rollen und Datenschutz

- Zugang mit mindestens Rolle **Mitglied** für die App `yearinpixels`.
- Strikt privat (RLS): jede Abfrage filtert `user_id` = eigenes Konto, auch Admins sehen nichts Fremdes.
- Die App speichert keine Tageswerte in der Datenbank. Für die Offline-Anzeige liegt der letzte Stand
  im Browser unter `yearinpixels-cache-<user>`, wartende Einstellungen unter `yearinpixels-queue-<user>`.

## Design

Bullet Journal: Punktraster-Papier, Handschrift (Gochi Hand), Collage-Überschrift (Lilita One), Farben nur aus
Stefans 20er-Farbtafel. Kopfleiste wie in den anderen Apps (gleiche Höhe), Hell/Dunkel über `overall-theme`.
Handy einspaltig mit Tagesblatt als Karte von unten, ab 1024 px Doppelseite.
Bei Änderungen die Cache-Version in `sw.js` hochzählen.
