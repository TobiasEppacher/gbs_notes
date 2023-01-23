# Kapitel 6: Speicherverwaltung

## Themen

- Verwaltung des physischen Speichers
- Abbildung von virtuellen auf physische Speicheradressen
- Datenstrukturen zur Freispeicher-Verwaltung
- Strategien zur Speicherzuteilung
- Virtueller Speicher

## Adressierungsarten

### Direkte Adressierung

- Physische Adresse = Virtuelle Adresse (häufig in Microcontrollern)
- Jedes Programm hat Zugriff auf den gesamten Speicher
- Ausführung mehrer Programme:
  - Option 1: Nur ein Programm im Speicher (Swapping)
  - Option 2: Programme an unterschiedlichen Speicheradressen (benötigt Adressübersetzung)

### Basisadressierung

- BS legt Basisadresse für jeden Prozess an
- Laden der Basisadresse bei Prozess-Start
- Physische Adresse = Virtuelle Adresse + Basisadresse 
- Mehrere Prozesse gleichzeitig im Speicher möglich
- Verschiebung von Prozessen im Hauptspeicher möglich
- Benötigt Addition bei jeder Adresse

### Segmentadressierung

- Prozessadressraum wird in Segmente (Stack, Code, Daten) unterteilt
- Jedes Segment besitzt Anfangsadresse und Länge
- Hardwareunterstützung: Segmentregister
- Intel x86: Global Descriptor Table (GDT)
  - Basisadresse, Länge, Flags (Zugriffsbeschränkungen) für jedes Segment
  - Segmentregister -> Index in GDT

## Freispeicherverwaltung

### Datenstrukturen

Voraussetzung: Speicher aufgeteilt in gleich große Blöcke
  
- Bitmaps 
  - Bitmap zeigt Belegung (1 = belegt, 0 = frei)
  - Einfach und schneller Zugriff auf einzelne Blöcke
  - Suche nach längeren freien Speicherbereichen schwer
- Verkettete Liste
  - Liste mit zusammenhängenden Speicherbereichen (auch mehrere Blöcke)
  - Jeder Bereiche entweder einem Prozess zugeordnet oder frei
  - Listeneintrag: Anfangsadresse, Länge in Blöcken, Zeiger auf den nächsten Eintrag (oder NULL)
  - Lineare Suche, dynamische Verwaltungsstruktur
  - Getrennte Listen für belegt und frei möglich 
  - Sortierung möglich
  - Aufeinanderfolgende freie Bereiche können zusammengefasst werden

## Belegungsstrategien

- First Fit
  - Immer von Anfang suchen
  - Erster ausreichend großer Bereich wird belegt
  - Nicht benötigter Speicher wird als neuer, kleinerer Bereich in die Freibereichsliste eingefügt
  - Erzeugt viele kleine Bereiche am Anfang der Liste
- Next Fit
  - Verwendet nächsten ausreichend großen Bereich
  - Beginnt Suche beim letzten belegten Bereich
  - Nicht genutzter Speicher wieder in die Liste
- Best Fit
  - Vollständiges Durchsuchen der Liste
  - Belegt Freibereich mit geringstem Verschnitt
  - Langsam, viele sehr kleine Restbereiche
- Worst Fit 
  - Vollständiges Durchsuchen der Liste
  - Belegt Freibereich mit größtem Verschnitt
  - Langsam, erzeugt größere, nutzbare Restspeicherbereiche
  - Große Bereiche schnell verbraucht

- Buddy-Algorithmus
  - Listen von $2^k$ großen Blöcken ($l \leq k \leq u$)
  - $2^l$: kleinste Blockgröße
  - $2^u$: größte Blockgröße (gesamter Speicher)
  - Start mit einem Block in der obersten Liste
  - Bei Anforderung spalten bis passende Größe erreicht
  - Bei Freigabe muss der 'Buddy' (bei Spaltung entstandener Block) frei sein um zu verschmelzen
  - Vorteil: Sehr effizient, Verschnitt maximal ein halber Block
  - Nachteil: Benachbarte freie Blöcke können nicht immer verschmelzen

## Fragmentierung

- Interne Fragmentierung 
  - Verschnitt beim blockweisen Allokieren von Speicher
  - Nicht nutzbarer Speicher innerhalb eines Blockes
- Externe Fragmentierung
  - Freie Speicherbereiche so klein, dass sie nicht genutzt werden können obwohl sie in Summe ausreichend wären
  - Nicht nutzbare Blöcke da zu wenig zusammenhängender Speicher vorhanden ist

## Paging

- Nötig wenn der virtuelle Adressraum größer als der physische Hauptspeicher ist.
- Auslagern von Adressbereichen auf den Hintergrundspeicher
- Aufteilung von virtuellen Adressräumen in Seiten und physischem Speicher in Kacheln 
- I.d.R gleiche Größe oder Seiten ein Vielfaches der Kacheln
- Abbildung von Seiten auf Kacheln (durch MMU - Hardware)

### Seitenfehler

- Tritt auf bei Zugriff auf eine virtuelle Adresse einer nicht eingelagerte Seite
- Hardware löst Interrupt (page fault) aus
- BS muss Seite einlagern#

### Page Table

- Verwaltet Zuordnung von Seiten zu Kacheln
- Eine pro Prozess
- Bei Prozesswechsel, Anfangsadresse in Page-Table-Register geladen
- Zusätzliche Bits für:
  - P (present): ist die physische Adresse existent
  - U/S (User/Supervisor): darf nur Kernel lesen/schreiben
  - R (referenced): wird bei CPU-Zugriff gesetzt
  - M (modify): wird bei Schreib-Zugriff gesetzt
  - XD (execute-disable): darf die CPU hier gespeicherte Daten ausführen (als Instruktion)
- Mehrstufig möglich
  - Seitentabelleneinträge verweisen auf weitere Seitentabellen erst diese auf Kacheln
  - I.A. ein-, zwei- und dreistufige Ausführungen

### Memory Management Unit und Translation Lookaside Buffer

- MMU führt Adressübersetzung durch
- Da die Pagetable im Hauptspeicher liegt sind Zugriffe langsam
- TLB ist ein Cache (ca. 256 Einträge) für Virtuelle Adressen und die entsprechenden Pagetable-Einträge für schnelleren Zugriff
- Speicherzugriff Ablauf:
  1. MMU prüft TLB mit virtueller Adresse
  2. Gefunden? Ja: Prüfe Zugriffsrechte, Nein: Schritt 5
  3. Zugriff erlaubt? -> Lade Daten aus physischer Adresse
  4. Zugriff verweigert? -> Löse Protection Fault aus (SIGSEGV)
  5. Nicht gefunden? -> prüfe Page Table in Hauptspeicher
  6. Eingelagert? -> Prüfe Zugriffsrechte (zu Schritt 3/4)
  7. Nicht eingelagert -> BS sucht lagert Seite ein, dann Zugriffsrechte prüfen

## Seitenersetzungsstrategien

- Sind alle Kacheln belegt muss beim Einlagern einer Seite eine andere ausgelagert werden
- Wann?
  - On-Demand: Laden wenn eine Seite gebraucht wird
  - Prefetching: Seiten im voraus laden
- Wohin?
  - Alle Kacheln gleichwertig
  - Praxis: Spezialbehandlung von Kernel-Code/-Daten (Form von Identity-Mapping)
- Welche Seiten auslagern?
  - Im Optimalfall die am spätesten benötigte Seite (nicht implementierbar)
- Ist die zu ersetzende Seite modifiziert (M-bit gesetzt) muss sie zurückgeschrieben werden

### FIFO

- Die Seite die am längsten im Speicher ist wird ausgelagert
- Béládys Anomalie: mehr Kacheln für Prozess verursachen u.U. mehr Seitenfehler
- Implementiert als Liste (neue Seiten ans Ende, Kopf wird ausgelagert)
- Selten eingesetzt

### Second-Chance

- Wie FIFO, aber referenzierte Seiten erhalten zweite Chance
- R-bit wird dann auf 0 gesetzt und die Seite ans Ende der Liste verschoben
- Unnötig häufige Verschiebungen

### Clock-Algorithmus

- Second Chance, aber verwalten der Seiten in einer Zirkulären Liste
- Zeiger auf die älteste Seite
- Laufe durch die Liste bis eine Seite mit R=0 gefunden wird, diese wird ersetzt
- Bei allen anderen überlaufenen Seiten setze R=0

### Least-Recently-Used

- Ersetze die Seiten die am längsten nicht verwendet wurden
- Problem: exakte Implementierung nötig, zeitaufwendiges Listensortieren
- Ersatz: Not-Frequently-Used

### Not-Frequently-Used (Aging)

- Jede Seite besitzt Software-Counter
- Bei jedem Timerinterrupt wird der Zähler aktualisiert
  - Rechtsshift um 1 bit
  - Wenn R-Bit=1 das höchste Zählerbit auf 1 und R-Bit auf 0 setzten
- Die Seite mit dem kleinsten Zählerstand wird ausgelagert

### Working-Set

- Die aktuell benötigten Seiten eines Prozesses sind das _**Working Set**_
- Umso weniger des Working Sets eingelagert ist, desto mehr Seitenfehler werden erzeugt
- Idee: Nur Seiten ersetzen die sich nicht im Working Set befinden
- Problem: Aufwendige Berechnung des Working Sets, Approximation über die Rechenzeit des Prozesses (Zähler Z)

- Umsetzung:
  - Inspektion der R-Bits aller Seiten (R=1 -> R=0, Seiten-Zähler Zp auf Z setzen)
  - Ansonsten wenn Z - Zp > Konstante: nicht im Working Set und ersetzbar
  - Wenn Z - Zp <= Konstante: Im Working Set

## Shared Memory

- Beim Starten eines Prozesses nicht alle Seiten einlagern
- Bei Fork zunächst keine Kopie des Speichers erstellen (erst bei Änderung im Kind)
- Shared Libraries ermöglichen nutzen durch mehrere Prozesse (nur einmal geladen)

## Malloc

- Malloc-Aufruf verwendet nicht immer BS-Speicherallokation
- Allokiert mehr Speicher als benötigt und verwaltet diese innerhalb der library
- Eigene Datenstruktur als Header für den Datenbereich den der Nutzer erhält
  - Größe des vorigen Datenblockes (nur wenn voriger frei ist)
  - Größe dieses Datenblockes
  - P-Bit: belegt oder nicht
  - Pointer auf den vorigen und nächsten freien Datenbereich (wenn dieser frei ist)
- Schreiben über den Datenbereich (vorne oder hinten) überschreibt Verwaltungsdaten

