# Kapitel 7 - Dateisysteme

## Abstraktion der Festplatte

- Festplatte ist eine Sequenz von Speicherblöcken fester Größe
- Verwalten von Blöcken und finden von Information muss geregelt werden
- Virtualisierung der Festplatte durch das Dateikonzept
  - Abstrahiert von Speicherorten
  - Bietet Prozessen Zugriff auf den verwalteten Speicher
  - Linux: "Everything is a file"

## Dateisystem

- Zur Verwaltung von Dateien durch das Betriebssystem
- Beinhaltet Datenstrukturen zur Beschreibung von Dateien
  - auf der Festplatte (z.B. inode)
  - geöffnete Dateien (z.B. Filedeskriptor)
- Implementierung von Dateien
- Benennung
- Zugriff auf Dateien
- Schutz vor unberechtigtem Zugriff

## Dateien

- Können von unterschiedlichen Prozessen genutzt werden
- Erhalten einen Namen
  - Kein einheitliches Benennungsschema für alle BS
  - Häufig name.extension (Extensions können von BS interpretiert werden z.B. Windows)
  
### Strukturierungsmöglichkeiten für Dateien

- Unstrukturiert
  - BS hat keine Kenntniss über den Aufbau von Dateien
  - Benutzerprogramme müssen Strukturierungsinformationen selbst erstellen
  - Vorteil: Flexibilität für Benutzerprogramme
- Sequenzen von Einträgen fester Größe
  - Jeder Eintrag hat eine feste Größe und Struktur
  - Heute nicht mehr verwendet
- Baum mit Einträgen unterschiedlicher Größe
  - Jeder Eintrag mit Key versehen
  - Einträge sortiert gespeichert
  - Heute noch in Großrechner-BS verwendet für große Datenmengen

## Dateitypen

- Dateien (regular files)
  - Text-Dateien: bestehend aus getrennten Zeilen
  - Binärdateien: bestimmtes Format, z.B. executables oder archives
- Verzeichnisse (directories)
  - Systemdateien zur Verwaltung und Strukturierung des Dateisystems
- Character Special Files
  - Modellieren I/O-Geräte (z.B. Terminals, Drucker, ...)
- Block Special Files
  - Modellieren Massenspeicher (z.B. Festplatten, USB-Speicher, ...)

## Dateizugriff

- Sequential Access (veraltet)
  - Dateien müssen von vorne nach hinten durchlaufen werden (ohne überspringen)
- Random-access
  - Bytes können in beliebiger Reihenfolge gelesen werden
  - Auch Zugriff auf bestimmte Position in Datei möglich

## Dateiattribute

- Dateiname
- Dateiinhalt
- Metadaten
  - Zugriffsrechte
  - Flags (Bitfelder, z.B. verstecken, Binär/Archiv, ...)
  - Zeiteinträge (modified, accessed)
  - Dateigröße

## Dateioperationen

- BS definieren Operationen für Dateizugriff, angeboten über Syscalls
  - Erzeugen -> Open(..., O_CREAT|O_READ|O_WRITE)
  - Löschen -> Unlink

- Weitere Zugriffe über den Filedeskriptor
  - close, read, write, seek

## Verzeichnisse 

- Organisation von Dateien erfolgt häufig in Verzeichnissen
- Single-Level Verzeichnisstrukturen
  - Ein Verzeichnis für alle Dateien
  - Sehr schnelle Dateisuche, aber keine Struktur
  - Heute noch in Embedded Systemen
- Hierarchische Verzeichnisstrukturen
  - Gruppierung verwandter Dateien in einem Verzeichnis
  - Eigene Verzeichnisse für Benutzer

## Pfadnamen

- Benennung von Dateien in baumartigen Verzeichnisstrukturen
- Absolute Pfadnamen
  - Umfasst gesamten Pfad ausgehend vom root-Verzeichnis
- Relative Pfadnamen
  - Pfad ausgehend vom aktuellen Arbeitsverzeichnis
- Spezielle Einträge in jedem Verzeichnis
  - "." -> aktuelles Arbeitsverzeichnis
  - ".." -> übergeordnetes Verzeichnis

## Dateisystem-Layout

- Aufteilung von Festplatten in Partitionen (ein Dateisystem pro Partition)
- Sektor 0 der Platte ist der Master Boot Record (MBR)
  - gelesen vom BIOS (UEFI) beim Bootvorgang
  - bestimmt Suche und Ausführung des Bootblocks
  - das Programm im Boot-Block lädt das BS aus der Partition
- Partitions-Tabelle: Anfangs- und End-Adresse pro Partition
- Eine Partition beinhaltet:
  - Boot block
  - Superblock (Dateisystem-Informationen)
  - Informationen über freie Blöcke
  - Menge von Dateideskriptoren (in Unix i-nodes)
  - Daten der Verzeichnisse und Dateien

## Implementierungen von Dateien

### Continuous Allocation

- Datei ist eine zusammenhängende Folge von Blöcken auf der Festplatte
- Vorteile: Einfach, performantes Lesen
- Nachteile: (Externe) Fragmentierung mit zeitaufwendiger Defragmentierung
- Nutzung in Read-Only-Medien

### Linked List Allocation

- Die Verwaltung der belegten Blöcke einer Datei erfolgt in einer verketteten Liste
- Das erste Wort pro Block verweist auf den nächsten
- Vorteile: kein Speicherplatzverlust bis auf interne Fragmentierung
- Nachteile: Langsamer Random-Access, Speicher in einem Block ist keine 2er-Potenz
- Variante: File Allocation Table
  - Tabelle im Hauptspeicher für Verweise der Blöcke
  - Kein Verschnitt im Block
  - Random-Access ist schneller
  - ABER FAT muss im RAM bleiben

### I-Nodes

- Datenstruktur zur Dateirepräsentation
  - Datteiattribute
  - Adressen der Blöcke (direct, single-/double-/...-indirect für größere Dateien)
  - Vorteile: Nur geöffnete Datei-Nodes müssen im Speicher bleiben
  - Nachteil: Beschränkte Zahl an Blöcken (umgehbar mit Indirektion)

## Verzeichnisse

- Besitzen die Aufgabe zur Pfadauflösung bzw. Dateisuche
- Bestehen aus Verzeichniseinträgen
  - Dateinamen
  - Adress-Informationen (Adresse, Erste Blocknummer, I-Node)
  - Verweis auf Datei-Attribute (für schnellere Überprüfungen)
- Dateinamen und Attribute können im Verzeichnis oder in den i-nodes gespeichert werden

## Dateinamen-Speicherung

- Dateinamen können auch variabel sein (mit Limit)
- Verschiedene Möglichkeiten
  - Verzeichniseinträge speichern Länge
    - unterschiedlicher Platzverbrauch je nach Name
    - Fragmentierung
    - Schwierige Änderung von Dateinamen
  - Verzeichniseinträge referenzieren Namen auf dem Heap (am Ende des Verzeichnisses)
    - Vorteil: Alle Einträge gleich groß, keine Fragmentierung beim Löschen
    - Suchen von Dateien wird langsam
  - Verbesserung: Hashtabelle in jedem Verzeichnis
    - Index des Verzeichniseintrags bestimmt durch Hashwert des Dateinamens
    - Kollisionen durch verkettete Liste behandelt
    - Ermöglicht konstante Zeit für Löschen/Einfügen
    - Schnelle Suche

## Gemeinsame Nutzung von Dateien

- Links für mehrere Pfade zu einer Datei
- Hard Links
  - Verzeichnisverweis auf I-Node
  - I-Nodes haben Reference Counts
  - Freigeben der Datenstruktur wenn Reference Count = 0
  - zum Löschen in UNIX: unlink()
- Symbolic Links
  - Erzeugen einer Datei welche auf Dateinamen verweist
  - Keine Auswirkung auf den Reference Counter
  - Fehler bei Verweis auf gelöschte Dateien

## Journaling-Dateisysteme

- Löschen von Dateien (unter UNIX) ist nicht atomar
- Keine Konsistenz nach Crash
- Protokollieren von Änderungen vor dem Durchführen in Log-Eintrag
- Ausführen der Operationen (jeweils vermerkt nach Abschluss)
- Löschen des Log-Eintrags nach vollständiger Durchführung
- Bei Crash anhand des Log-Eintrags erneut ausführen (idempotente Operationen)

## Virtual File System (VFS)

- Nutzen mehrerer Dateisysteme auf einem Rechner
- Windows: jedes Dateisystem ein Laufwerk, kein einheitliches Dateisystem
- Unix-artig: Integration und einheitliche Struktur durch ein Virtuelles Dateisystem
- Einheitliche Schnittstelle
  - open, read, write, lseek, ...
  - im Kern auf einzelne Dateisystemoperationen abgebildet

## Optimierungen für Dateisysteme

- Buffercache
  - Puffer im Hauptspeicher für Datenblöcke
  - Beim Lesen in Cache-Lesen für schnelle erneute Verwendung
  - Beim Schreiben in Cache-Schreiben (schnell), evt. späteres ausschreiben (nicht bei Windows, write-through)
  - Ersetzungsstrategie im Cache nötig
- Read-Ahead
  - Benötigte Blöcke vorab in den Cache laden
  - Verschiedene Vorhersagemethoden möglich
- Reduktion der Schreib-/Lesekopf-Bewegungen
  - Benachbarte Daten nache beieinander speichern
  - Blöcke in Gruppen verwalten
  - i-Nodes nicht nur am Anfang der Partition
- Defragmentierung

## Blockgruppen (ext2)

- Block 0 reserviert für Bootvorgang
- Alle Blockgruppen umfassen
  - Superblock
  - Gruppen-Deskriptor (Position der Bitmaps, Anzahl Verzeichnisse/freie i-nodes,...)
  - Zwei Bitmaps (je 1 Block, Verwaltung von i-nodes und freie Blöcke)
  - i-Nodes
  - Datenblöcke

## Öffnen einer Datei (ext2)

- Suchen der Datei im Verzeichnis
- I-Node-Nummer verweist auf Eintrag in der i-node-Table auf der Platte
- Eintragen der i-Node in der i-node-Table im Speicher (nur geöffnete Dateien)
- Erstellen eines Open-File-Deskriptor-Verweises auf die i-Node mit Zugriffsrechten
- Jeweiliger Prozess speichert Verweis auf OFD-Verweis in File-Deskriptor-Table