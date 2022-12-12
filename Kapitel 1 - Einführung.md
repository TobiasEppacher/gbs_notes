# Kapitel 1: Einführung

## Aufgabe der Informatik

- Entwicklung von Rechensystemen
- Diese für Anwender nutzbar machen

## Definition Rechensystem

_**Offene, dynamische und technische Systeme**_ zur _**Speicherung**_ und _**Verarbeitung**_ von Informationen und _**Kommunikation**_

### Offenes System

- Besteht aus Komponenten
- Zwischen Komponenten bestehen Abhängigkeiten
- Es gibt Schnittstellen zur externen Einwirkung
- _(Durch Dokumentation und Gestaltung sind Schnittstellen für Dritte nutzbar)_

## Schnittstellen mit verschiedenen Sichten

Ermöglichen die äußere Einwirkung auf das System, z.B. Benutzerinteraktion, Netzschnittstellen usw.

- Von außen: **Black-Box**
  - Abgegrenzte Einheiten
  - Zusammengefasste Komponenten

- Von innen: **White-Box**
  - Einzelne Komponenten sichtbar
  - Diese wiederum in kleinere White-Box-Sichten aufteilbar

## Dynamisches System

- Der **Zustand des Systems** wird zu jedem Zeitpunkt durch eine Reihe von Eigenschaften beschrieben.
- Verändert sich der Zustand über die Zeit spricht man von einem **dynamischen System**, wobei eine Folge von Zustandsänderungen beschreiben das **Systemverhalten** beschreiben.
- **Aktive** Komponenten (z.B. CPU) bewirken durch Aktionen Zustandsänderungen.
- **Passive** Komponenten (z.B. Arbeitsspeicher) sind dabei Hilfsmittel. 

## Technisches System

Realisiert mit hardware- und softwaretechnischen Mitteln

## Definitionen Betriebssystem

- **Nach DIN 44300 (1972):**
  - Siehe Foliensatz Kapitel 1, S.14

- **Alternativ:**
  - **Ein Programm**, das die Ausführung von System- oder Anwendungsprogrammen **steuert** und **überwacht**, sowie eine **Schnittstelle zu den Hardware-Komponenten** (z.B. read, write zur Speichermanipulation) bereitstellt.

## Aufgaben des Betriebssystems

### 1. Abstraktion

- Vereinfachung, Bereitstellung von abstrakten Konzepten und gemeinsamen Schnittstellen
- z.B. Dateien, Zugriff auf I/O-Geräte
- read, write: Umsetzung auf gerätespezifische Befehle durch Betriebssystem (mithilfe von Treibern)

### 2. Ressourcen-Management

- Steuerung und Kontrolle von Programmausführung
- Priorisierung, adaptive Änderung und operatives Eingreifen zur Zuteilung der CPU Nutzung

## Faktoren für die Entwicklung von Betriebssystemen

- Fortschritte der Hardwaretechnologie
- Übergang zu allgemeiner Informationsverarbeitung
- Neue Anwendungsbereiche und Digitalisierung

## Betriebsarten

- Stapelverarbeitung (batch processing)
  - Programme komplett vordefiniert
  - Keine Nutzer-Interaktion
- Transaktionsbetrieb (transaction system)
  - z.B. Datenbanken
  - ACID-Kriterien (Atomarität, Konsistenz, Isolation, Dauerhaftigkeit)
- Dialogbetrieb (time sharing)
  - Nutzerinteratkion, benötigt (G)UIs
- Echtzeitbetrieb (real time system)
  - hard deadlines: Keine Überschreitung (Robotik)
  - soft deadlines: Überschreitung innerhalb Toleranz (Multimedia)

## Klassifikation von Ressourcen

Kriterien:
- #Nutzungen
- Parallelität
- Dauerhaftigkeit (unterbrechbar?)
- Zentrale/Periphere Ressourcen (CPU vs. Kommunikationseinheiten)
- Aktiv/Passiv
- Hardware/Software
- ...

## Prozesse

### Prozess vs. Programm

- Prozess ist ein Programm in Ausführung
- Prozess ist aktiv, Programm ist passiv
- Mehrere Ausführungen eines Programms möglich

### Prozesse

- Ein Prozess benötigt Ressourcen zur Erfüllung seiner Aufgabe (z.B. CPU, Speicher, I/O-Geräte, ...)
- Jeder Prozess hat einen eigenen (virtuellen) Adressraum (wird vom BS verwaltet)

### Sequenziell vs. Parallel

- Sequenziell: Ein Kontrollfluss (mit Befehlszähler)
- Multi-Threaded: Eigene Befehlszähler, Stack und Registerinhalte pro Thread

### Dateien und Dateisysteme

- Ein root-Verzeichnis als Ausgangspunkt
- Linux: Everything is a file
  - Dateien, Ordner, I/O-Geräte, ... werden im Dateisystem als Dateien repräsentiert, über diese erfolgt die Kommunikation.
- Als gemeinsame Abstraktion: File Descriptor (Referenz auf Datei für I/O)

### Betriebssystem-Modi

Schutz des Betriebssystems vor Programmierfehlern/Angriffen

- Benutzermodus
  - Kein direkter HW-Zugriff
  - Keine privilegierte Befehle (z.B. I/O)
  - Eingeschränkter Zugriff auf Systemcode/-daten
- Systemmodus
  - Alle Befehle
  - Direkter HW-Zugriff
  - Betriebssystem in diesem Modus

### System-Aufrufe

- Schnittstelle zwischen BS und User-Level-Anwendungen
- Erlauben Zugriff auf privilegierte Befehle
- Angeboten durch Bibliotheken
- Durchführung:
  - Aufruf Bibliotheksfunktion
  - Aufruf System-Call (durch Trap, Unterbrechung des aktuellen Ablaufs)
  - Ausführung Syscall in BS-Kern
  - Rücksprung in Anwendung

### Betriebssystem-Architekturen

- Monolithisch
  - Viele Funktionen
  - Treiber in Kernel
  - Ein großes Programm das immer läuft
  - Sehr flexibel, hohe Modularität
  - unübersichtlich, schwierige Wartung
  - Wenig resilient (ein Crash genug)
- Mikrokernel
  - BS aufgeteilt, nur ein Modul (Kern) im Kernel-Mode
  - kleiner Kern
  - Nur Basismechanismen (Prozesskommunikation, Scheduling, ...) im Kern
  - leichter wartbar
  - geringe Modularität des Kerns
  - Systemfunktionen sind Serverprozesse, Dienste kommunizieren über den Kern
- andere (z.B. geschichtet, Client-Server, Virtual Machines, ...)
