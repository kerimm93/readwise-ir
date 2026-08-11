# Readwise IR Inbox

Eine installierbare, offline-first Progressive Web App für gemischte Incremental-Reading-Sitzungen mit Readwise Reader. Eine Queue wird online zusammengestellt, lokal gespeichert und kann danach ohne API-Verbindung bearbeitet werden.

## Was die App auswählt

Eine Runde enthält je nach Einstellung:

- fällige Dokumente mit `ir-active`
- neue Dokumente ohne `ir-active`, `ir-paused`, `ir-completed` oder `ir-dropped`

Standardmäßig werden 60 % Wiedervorlagen und 40 % Neuaufnahmen angestrebt. Fehlen in einem Topf genügend Treffer, füllt der andere die freien Plätze auf. Die Rundengröße bleibt auf 5 bis 50 Dokumente begrenzt.

## Offline-first-Ablauf

1. **Neue Offline-Queue laden** ruft Readwise auf und speichert bis zu 50 vollständige Dokument-Snapshots lokal.
2. Jede Statusentscheidung wird zuerst dauerhaft auf dem Gerät gespeichert. Die Karte verschwindet erst nach erfolgreicher lokaler Speicherung aus der Queue.
3. Die App zeigt jederzeit, wie viele Änderungen noch nicht übertragen wurden.
4. **Mit Readwise abgleichen** prüft jedes betroffene Dokument frisch und überträgt die vorgemerkten Änderungen einzeln.
5. Fehlgeschlagene oder konfliktbehaftete Änderungen bleiben mit Fehlermeldung im lokalen Ausgang. Ein neuer Queue-Snapshot wird erst zugelassen, wenn dieser Ausgang leer ist.

Ein Konflikt oder dauerhaft fehlgeschlagener Sync-Eintrag kann erneut geprüft oder nach ausdrücklicher Bestätigung lokal verworfen werden. Beim Verwerfen bleibt der aktuelle Readwise-Stand unangetastet.

Das absolute Fälligkeitsdatum wird beim Entscheiden festgelegt. Eine offline gewählte Wiedervorlage „in 7 Tagen“ verschiebt sich daher nicht, wenn der Abgleich erst später stattfindet.

## Getrennte Warteschleifen auf mehreren Geräten

Name, Filter, Mischverhältnis, Queue und lokaler Änderungsausgang gelten nur im jeweiligen Browser beziehungsweise in der jeweiligen PWA-Installation. Dadurch kann beispielsweise gelten:

- Smartphone: `category:video AND in:later`
- E‑Paper-Tablet: `category__not:video AND in:later`

Die Filter sollten möglichst disjunkt sein. Überschneiden sich Queues dennoch, verhindert die Konfliktprüfung ein stilles Überschreiben desselben IR-Zustands. Es gibt keinen automatischen geräteübergreifenden Queue-Sync; Readwise bleibt die gemeinsame Datenbasis nach einem bewussten Abgleich.

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
- Queue, Dokument-Snapshots und ausstehende Änderungen werden in IndexedDB gespeichert und zusätzlich im Browser-Speicher gespiegelt.
- Schreibvorgänge mehrerer gleichzeitig geöffneter Tabs oder PWA-Fenster werden geräteweit serialisiert; andere offene Ansichten übernehmen den neuesten lokalen Stand automatisch.
- Das Entfernen des Tokens oder das Verwerfen einer offenen Queue löscht keine noch nicht synchronisierten Entscheidungen.
- Unmittelbar vor jedem API-Schreibvorgang lädt die App das konkrete Dokument frisch. Änderungen am selben IR-Block oder an IR-Tags werden als Konflikt markiert und nicht still überschrieben.
- Änderungen an fremdem Notiztext und thematischen Tags werden beim Abgleich erhalten.

## Offline-Verhalten

Nach dem ersten erfolgreichen Laden kann die gesamte Oberfläche offline geöffnet werden. Eine bereits geladene Queue lässt sich offline sichten und entscheiden; auch Intervall, Priorität und Phase werden lokal gespeichert. Nur diese Vorgänge benötigen Internet:

- eine neue Queue aus Readwise laden
- lokale Entscheidungen mit Readwise abgleichen
- ein Dokument öffnen, sofern es nicht bereits in Reader verfügbar ist

Der Online-/Offline-Status und die Anzahl ausstehender Änderungen bleiben dauerhaft sichtbar.
