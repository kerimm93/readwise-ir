# Readwise IR Inbox

Eine installierbare, offline-first Progressive Web App für gemischte Incremental-Reading-Sitzungen mit Readwise Reader. Ein großer Master-Vorrat wird einmal online zusammengestellt, lokal gespeichert, danach ohne API-Verbindung gefiltert und in kleinen Teilrunden bearbeitet.

## Was die App auswählt

Ein Offline-Vorrat enthält je nach Einstellung:

- fällige Dokumente mit `ir-active`
- neue Dokumente ohne `ir-active`, `ir-paused`, `ir-completed` oder `ir-dropped`

Standardmäßig werden 60 % Wiedervorlagen und 40 % Neuaufnahmen angestrebt. Fehlen in einem Topf genügend Treffer, füllt der andere die freien Plätze auf. Der Vorrat kann 10 bis 1.000 Dokumente enthalten; sichtbar sind davon jeweils nur 5 bis 50 Dokumente pro Teilrunde. Die Voreinstellung ist 100 Dokumente im Vorrat und 10 pro Teilrunde. Ein Download von 1.000 Dokumenten kann wegen der Rate-Limits und mehrerer API-Seiten deutlich dauern, wird danach aber nur bei einem bewusst angeforderten neuen Snapshot wiederholt.

Wenn eine Teilrunde endet, öffnet **Nächste … passende** ohne Netzwerkzugriff den nächsten lokalen Stapel. Das funktioniert auch dann, wenn frühere Entscheidungen noch nicht mit Readwise abgeglichen wurden. Erst wenn der gesamte Master-Vorrat verbraucht ist, muss eine neue Auswahl aus Readwise geladen werden.

## Download-Filter und lokale Ansichtsfilter

Die Eingabe folgt der [offiziellen Reader-Filtersyntax](https://docs.readwise.io/reader/guides/filtering/syntax-guide). Beispiele:

- `category:epub OR category:pdf`
- `category:note OR category:highlight`
- `tag:shortlist`
- `category__not:video AND in:later`
- `progress__gt:3 AND minutes__gt:20`

Die offizielle Schreibweise enthält kein Leerzeichen nach dem Doppelpunkt. Ein versehentlich eingegebenes `category: epub` wird beim Speichern komfortabel zu `category:epub` normalisiert. Leerzeichen innerhalb zitierter Werte bleiben unverändert, beispielsweise in `title:"foo category: bar"`. `category` beziehungsweise der Alias `type` sind gültige Reader-Felder; unterstützt werden auch die Reader-Dokumentkategorien `note` und `highlight`.

`shortlist` kann in Reader zwei verschiedene Dinge bezeichnen:

- `in:shortlist` beziehungsweise `location:shortlist` ist der offizielle Bibliotheksort der Shortlist-Konfiguration.
- `tag:shortlist` ist ein Dokument-Tag.

In der hier vorgesehenen persönlichen Ablage wird Shortlist als Dokument-Tag verwendet; deshalb setzt der Standardknopf **Shortlist-Tag** den Ausdruck `tag:shortlist`. Der Standortausdruck `in:shortlist` bleibt weiterhin gültig, wenn er manuell eingegeben wird.

Der **Download-Filter** bestimmt nur, welche Dokumente beim ausdrücklichen Nachladen aus Readwise in den Master-Vorrat gelangen. Für einen vielseitigen Vorrat bietet sich beispielsweise `in:inbox OR in:later` an. Reader-Filter mit mehreren API-seitig filterbaren `OR`-Zweigen werden in getrennte API-Abfragen zerlegt und anschließend vereinigt. Die vollständige Bedingung wird zusätzlich lokal auf jedes Ergebnis angewendet.

Der davon getrennte **lokale Ansichtsfilter** wird auf den bereits gespeicherten Master-Vorrat angewandt. Die Schnellleiste bietet unter anderem Alle, Videos, Ohne Videos, EPUB/PDF, Inbox, Later sowie Inbox + Later. Ein Wechsel erzeugt sofort eine neue sichtbare Teilrunde, ruft die API nicht auf und löscht die ausgeblendeten Dokumente nicht. Eigene lokale Filter lassen sich unter einem frei wählbaren Namen speichern und erscheinen ebenfalls in der Schnellleiste. Auch die leere Ansicht **Alle** kann als benanntes Profil gespeichert, exportiert und nach einem Neustart wieder angewandt werden.

Sind PWA und Browser-Tab gleichzeitig geöffnet, werden lokale Filtereinstellung und sichtbare Teilrunde unter derselben geräteweiten Sperre aktualisiert. Alle offenen Ansichten laden Einstellungs- und Queue-Stand gemeinsam nach, sodass Beschriftung, Trefferzahl und nächste Teilrunde nicht auseinanderlaufen.

Filter auf Standort, Kategorie und Tags können bereits von der Reader-API eingegrenzt werden und laden deshalb besonders effizient. Bedingungen, die erst auf den gelieferten Dokumentdaten geprüft werden können, dürfen dagegen mehr API-Seiten durchsuchen und entsprechend länger dauern; die App bricht diese Suche nicht nach einer kleinen, willkürlichen Seitenzahl ab.

## Offline-first-Ablauf

1. **Neuen Offline-Vorrat laden** ruft Readwise auf und speichert bis zu 1.000 vollständige Dokument-Snapshots lokal.
2. Jede Statusentscheidung wird zuerst dauerhaft auf dem Gerät gespeichert. Die Karte verschwindet erst nach erfolgreicher lokaler Speicherung aus der Queue.
3. Die App zeigt jederzeit, wie viele Änderungen noch nicht übertragen wurden.
4. **Mit Readwise abgleichen** prüft jedes betroffene Dokument frisch und überträgt die vorgemerkten Änderungen einzeln.
5. Fehlgeschlagene oder konfliktbehaftete Änderungen bleiben mit Fehlermeldung im lokalen Ausgang. Weitere Teilrunden aus dem vorhandenen Vorrat bleiben möglich. Ein neuer Readwise-Snapshot wird erst zugelassen, wenn dieser Ausgang leer ist.

Ein Konflikt oder dauerhaft fehlgeschlagener Eintrag kann erneut geprüft oder nach ausdrücklicher Bestätigung lokal verworfen werden. Beim Verwerfen bleibt der aktuelle Readwise-Stand unangetastet. Dadurch blockiert ein nicht mehr abrufbares Reader-Dokument keine späteren Queues dauerhaft.

Das absolute Fälligkeitsdatum wird beim Entscheiden festgelegt. Eine offline gewählte Wiedervorlage „in 7 Tagen“ verschiebt sich daher nicht, wenn der Abgleich erst später stattfindet.

Beim Öffnen der App findet niemals automatisch ein API-Aufruf statt. Sie stellt zuerst ausschließlich den vorhandenen lokalen Stand wieder her. Ist noch kein Vorrat auf diesem Gerät gespeichert, bleibt die App auf der lokalen Leeransicht, bis **Neuen Offline-Vorrat laden** ausdrücklich betätigt wird. Das gilt auch dann, wenn der Browser seinen Verbindungsstatus unzuverlässig meldet.

## Getrennte Warteschleifen auf mehreren Geräten

Name, Filterprofile, Mischverhältnis, E‑Ink-Modus, Geräte-Spur, Vorrat und lokaler Änderungsausgang gelten nur im jeweiligen Browser beziehungsweise in der jeweiligen PWA-Installation. Dadurch kann beispielsweise gelten:

- Smartphone: `category:video AND in:later`
- E‑Paper-Tablet: E‑Ink-Modus plus `in:later`

Der E‑Ink-Modus schaltet zusätzlich auf eine kontrastreiche, animationsfreie Darstellung und entfernt beim Zusammenstellen und lokalen Filtern alle Dokumente mit Kategorie `video` sowie Quellen von YouTube, `youtu.be` und `youtube-nocookie.com`. Diese Geräte-Regel wird getrennt von Download- und Ansichtsfilter angewandt und in der Einstellungsansicht ausgewiesen.

Für sich überschneidende Filter bietet die App **Geräte-Spuren**. Auf allen beteiligten Geräten wird dieselbe Anzahl Spuren gewählt, aber jedes Gerät erhält eine andere Spurnummer. Eine stabile Hashfunktion weist jede Dokument-ID genau einer Spur zu. Dadurch bleiben die Vorräte auch dann disjunkt, wenn sie zu unterschiedlichen Zeitpunkten geladen und danach lange offline benutzt werden. Eine bloße Reihenfolge „älteste auf Gerät A, neueste auf Gerät B“ wäre nicht konfliktfest, weil die API-Snapshots zu verschiedenen Zeitpunkten überlappen können.

Die Spurzahl sollte nur geändert werden, wenn alte Vorräte abgearbeitet oder bewusst verworfen wurden. Ist keine Aufteilung gewählt, sollten die Filter möglichst disjunkt sein. Überschneiden sich Vorräte dennoch, verhindert die bestehende Konfliktprüfung ein stilles Überschreiben desselben IR-Zustands. Es gibt keinen automatischen geräteübergreifenden Queue-Sync; Readwise bleibt die gemeinsame Datenbasis nach einem bewussten Abgleich.

## Gerätekonfiguration sichern

Unter **Einstellungen → Gerätekonfiguration** kann die Konfiguration als lesbare JSON-Datei exportiert und später wieder importiert werden. Enthalten sind:

- Name des Geräts beziehungsweise der Warteschleife
- Download-Filter, lokaler Ansichtsfilter und gespeicherte lokale Filterprofile
- Vorrats- und Teilrundengröße
- Fällig/Neu-Mischung und Standardintervall
- Öffnungsmodus, E‑Ink-Modus und Geräte-Spur

Nicht exportiert werden Access Token, Dokument-Snapshots, Offline-Vorrat oder ausstehende Änderungen. Die JSON-Datei kann deshalb beispielsweise in Google Drive gesichert werden. Wird dieselbe Datei zum Einrichten eines weiteren Geräts benutzt, muss dessen Spurnummer anschließend auf eine andere Spur gestellt werden.

Mehrere gleichzeitig geöffnete Ansichten derselben GitHub-Pages-Installation – etwa die installierte PWA und ein Browser-Tab – teilen dagegen denselben lokalen Stand. Zustandsänderungen werden geräteweit serialisiert, lesen unmittelbar vor der Änderung den neuesten IndexedDB-Snapshot und informieren die anderen offenen Ansichten anschließend über den neuen Stand. So bleiben Queue- und Ausgangsänderungen beider Fenster erhalten.

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

Nach dem ersten erfolgreichen Online-Aufruf startet die App-Shell direkt aus dem lokalen PWA-Cache. Falls ein Browser nach einem Update dennoch einen sehr alten Stand hält: installierte App schließen, die Seite einmal online neu laden und notfalls die Websitedaten beziehungsweise den Service Worker der Pages-Adresse entfernen.

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
- Gleichzeitige Schreibvorgänge aus PWA und Browser-Tab werden über eine originweite Sperre serialisiert. Browser ohne Web Locks verwenden primär eine atomare IndexedDB-Lease; nur ohne IndexedDB wird eine abgesicherte `localStorage`-Lease versucht. Wartende Ansichten brechen nicht nach einem festen Zeitlimit ab, solange die aktive Lease gültig erneuert wird.
- Das Entfernen des Tokens oder das Verwerfen einer offenen Queue löscht keine noch nicht synchronisierten Entscheidungen.
- Unmittelbar vor jedem API-Schreibvorgang lädt die App das konkrete Dokument frisch. Änderungen am selben IR-Block oder an IR-Tags werden als Konflikt markiert und nicht still überschrieben.
- Änderungen an fremdem Notiztext und thematischen Tags werden beim Abgleich erhalten.

## Offline-Verhalten

Nach dem ersten erfolgreichen Laden kann die gesamte Oberfläche offline geöffnet werden. Ein bereits geladener Vorrat lässt sich in beliebig vielen lokalen Teilrunden sichten und entscheiden; auch Intervall, Priorität und Phase werden lokal gespeichert. Nur diese Vorgänge benötigen Internet:

- einen neuen Offline-Vorrat aus Readwise laden
- lokale Entscheidungen mit Readwise abgleichen
- ein Dokument öffnen, sofern es nicht bereits in Reader verfügbar ist

Der Online-/Offline-Status und die Anzahl ausstehender Änderungen bleiben dauerhaft sichtbar.

Der angezeigte Netzwerkstatus ist nur ein Hinweis des Browsers. Auch wenn er „Offline“ meldet, bleiben ausdrücklich gestartete Aktionen wie „Queue laden“ und „Mit Readwise abgleichen“ bedienbar. Die App versucht dann die echte Readwise-Anfrage; erst deren Ergebnis entscheidet, ob die Verbindung verfügbar ist. Bei einem Fehlschlag bleiben Queue und lokale Änderungen unverändert erhalten.

Wichtig: Offline kann nur eine Queue bearbeitet werden, die auf demselben Gerät und unter derselben GitHub-Pages-Adresse zuvor erfolgreich geladen wurde. Eine Queue eines anderen Geräts wird bewusst nicht lokal synchronisiert.
