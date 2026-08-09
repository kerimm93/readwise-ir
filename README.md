# Readwise IR Inbox

Eine installierbare Progressive Web App für gemischte Incremental-Reading-Sitzungen mit Readwise Reader.

## Was die App auswählt

Eine Runde enthält je nach Einstellung:

- fällige Dokumente mit `ir-active`
- neue Dokumente ohne `ir-active`, `ir-paused`, `ir-completed` oder `ir-dropped`

Standardmäßig werden 60 % Wiedervorlagen und 40 % Neuaufnahmen angestrebt. Fehlen in einem Topf genügend Treffer, füllt der andere die freien Plätze auf. Die Rundengröße bleibt auf 5 bis 50 Dokumente begrenzt.

## In Readwise gespeicherter Zustand

Die App spiegelt die für Auswahl und Planung nötigen Werte in austauschbaren Dokument-Tags:

| Tagfamilie | Beispiel | Regel |
|---|---|---|
| Status | `ir-active` | genau einer der vier Statuswerte |
| Termin | `ir-due-2026-08-06` | nur bei `ir-active`, absolutes ISO-Datum |
| Priorität | `ir-priority-3` | genau ein Wert von 1 bis 5 |
| Phase | `ir-phase-reading` | `orientation`, `reading`, `extracting` oder `integrating` |

Beim Ändern wird der alte Tag derselben Familie entfernt und durch den neuen ersetzt. Thematische Tags bleiben unverändert. Der ausführliche Zustand mit Lesepunkt, Fragestellung und Lesetagebuch bleibt im abgegrenzten Incremental-Reading-Bereich der Document-Note.

Alte `ir-active`-Dokumente ohne `ir-due-*` werden über das Wiedervorlagedatum in ihrem vorhandenen IR-Block gefunden. Beim nächsten Speichern ergänzt die App den Termin-Tag und bringt Tag und Note wieder in Übereinstimmung.

## Auf GitHub Pages veröffentlichen

1. Ein neues GitHub-Repository anlegen.
2. Den **Inhalt dieses Ordners** in die oberste Ebene des Repositorys hochladen. `index.html` muss direkt im Repository-Stamm liegen.
3. In GitHub **Settings → Pages** öffnen.
4. Unter **Build and deployment** „Deploy from a branch“ wählen.
5. Den Branch `main` und den Ordner `/ (root)` auswählen und speichern.
6. Die von GitHub angezeigte HTTPS-Adresse öffnen.

## Als App installieren

- Android/Chrome: Seite öffnen → Browsermenü → **App installieren** oder **Zum Startbildschirm hinzufügen**.
- Desktop/Chrome oder Edge: Installationssymbol rechts in der Adressleiste verwenden.
- iPhone/iPad/Safari: Teilen → **Zum Home-Bildschirm**.

Nach einem Update lädt der Service Worker die App-Shell grundsätzlich netzwerkbasiert und verwendet den Cache nur als Offline-Rückfall. Falls ein Browser dennoch einen sehr alten Stand hält: installierte App schließen, Seite neu laden und notfalls die Websitedaten beziehungsweise den Service Worker der Pages-Adresse entfernen.

## Dokument in Reader öffnen

- Im Modus **Automatisch** verwendet eine schmale Android-Ansicht einen direkten Intent für die installierte App **Readwise Reader** (`com.readermobile`). Dadurch wird nicht zuerst die Web-App auf `read.readwise.io` geladen. Auf PCs und in einer breiten Desktop-Ansicht öffnet sich stattdessen die Webversion in einem neuen Tab.
- Unter **Einstellungen → Dokumente öffnen** kann jedes Gerät diese Erkennung lokal mit **Reader-App (Android)** oder **Webversion** überschreiben.
- Ist Reader auf Android nicht installiert, führt der Ersatzweg zur offiziellen Play-Store-Seite und nicht zum Web-Reader.
- Je nach Browser oder Android-Einstellung kann beim ersten Öffnen weiterhin eine Sicherheitsabfrage erscheinen. Unter **Einstellungen → Apps → Reader → Standardmäßig öffnen** sollte **Unterstützte Links öffnen** aktiviert sein.
- Auf anderen Betriebssystemen öffnet der Button weiterhin die von der Reader-API gelieferte Webadresse.

## Sicherheit

- Der Readwise Access Token wird ausschließlich im lokalen Browser-Speicher der jeweiligen GitHub-Pages-Adresse abgelegt.
- Der Token gehört niemals in `index.html`, GitHub Actions, das Repository oder den Service Worker.
- Offene lokale Runden werden nicht zwischen Geräten synchronisiert. Nach „Neu laden“ wird die Queue aus dem aktuellen Readwise-Zustand rekonstruiert.
- Vor jeder Änderung lädt die App das konkrete Dokument frisch. Ein auf einem anderen Gerät geänderter Status oder Termin wird nicht still überschrieben.

## Offline-Verhalten

Die Oberfläche selbst kann nach dem ersten erfolgreichen Laden offline geöffnet werden. Lesen und Ändern von Readwise-Dokumenten benötigt weiterhin eine Internetverbindung.
