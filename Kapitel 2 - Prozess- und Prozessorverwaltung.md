# Kapitel 2: Prozess- und Prozessorverwaltung

## Prozesse im Speicher

Der Prozesskontext im BS beinhaltet:
- Registerinhalte
- Program Counter
- Stack Pointer
- Statusregister
- Prozesszustand (ready, ...)
- Priorität
- Process ID
- Parent PID
- Process Group ID
- Dateideskriptoren
- User ID
- Group ID
  
Zusätzlich auch Pointer auf...
- Stack-Segment
- Code-Segment
- Data-Segment
- \+ Größer der Segmente

## Parallele Prozessverarbeitung

Wechsel zwischen Prozessen führt zu scheinbarer Parallelität.
Dies ist nützlich, da Prozesse Speicher und CPU unterschiedlich stark beanspruchen. Somit können die Ressourcen besser genutzt werden.

- CPU-Bound: Der Prozess beansprucht primär die CPU, benötigt wenig Wartezeit für I/O-Anfragen
- I/O Bound: Benötigt kurze Rechenzeit zwischen langen Warteperioden auf Daten von I/O-Geräten (z.B. Speicherplatten)

## Prozessor-Zustände

- rechenwillig (ready)
- rechnend (running)
- wartend (blocked)
- ausgelagert (swapped out)

## Prozess vs. Thread

Prozess ist ein Programm in Ausführung.
Trennt man Ressourcen und Kontrollfluss eines Prozess:
- Prozess gruppiert und verwaltet Ressourcen
- Thread definiert einen Kontrollfluss

### Thread

- Abstraktion eines Prozessors (eigener Befehlszähler)
- Ein Prozess besitzt min. einen Thread
- Threads teilen sich den Adressraum des Prozesses
- Besitzen eigenen Thread-Kontext
  - Program Counter
  - Registerwerte
  - Stack
- Können wie Prozesse Zustände annehmen (waiting, running, ...)

### Multithreading

Verwendung von mehreren Threads
- "ligthweight" (u.a. gemeinsamer Adressraum)
- einfache Kommunikation zwischen Threads eines Prozesses
- Erstellungsaufwand ist deutlich geringer
- Schneller Kontextwechsel 

## Implementierung von Prozessen

Der Kernel besitzt eine Datenstruktur, den sog. **Process Control Block** mit Inhalten zur Prozessverwaltung.
- Registerinhalte (+PC, SP, Statusregister, ...)
- Prozesszustand
- Priorität
- Process ID
- ...

Die Verwaltungerfolgt in Prozesstabellen (separate verkettete Listen von PCBs)
für die verschiedenen Prozesszustände
- Run Queue: (Ready Queue) rechenwillige Prozesse (Linux)
- Wait Queue: Wartende Prozesse (blocked, Linux)

## BS-Dienste zur Prozessorverwaltung

- Erzeugung eines Prozesses
- Terminierung von Prozessen
- Strategien zur Prozessorzuteilung (Scheduling)
- Prozessor-Anbindung (Dispatching)

## Prozesserzeugung

Auslöser:

- Während der Systeminitialisierung
- Durch andere Prozesse
- Durch Benutzer
- Durch das Betriebssystem (Batch-Systeme, zur Gruppierung von Aufgaben)

POSIX-Syscall: pid_t fork()

- Kopiert Speicher des Parents (Ausnahme - copy-on-write)
- Rückgabewert ist PID des Kindprozesses (nur für Parent, für Kind 0)
- Kind erbt auch Filedeskriptoren
- Für eigenen Code führt der Kindprozess _exec{...}_ aus um das Speicherabbild zu überschreiben

## Prozessterminierung
- Normale Beendigung
  - freiwillig
  - exit(), return() in main
- Vorzeitige Beendigung 
  - freiwillig, z.B. bei Fehlern,
  - exit() mit Fehlercode
- Vorzeitige Beendigung bei fatalem Fehler
  - erkannt durch BS
  - z.B. Segmentation Fault
- Terminierung durch einen anderen Prozess
  - z.B. Signale (SIGKILL, SIGSTOP, ...)
  - kill(pid, signal)

## Prozesshierarchien (UNIX)

Nach Initialisierung des Kernels folgt der erste Userspace-Prozess _init_ (erstellt Login-Prozesse).
Jeder Prozess mit Kindern ergibt eine Prozessgruppe (mit PGID)

### Zombies

Parent Prozess wartet mit _wait()_ oder _waitpid()_ auf Terminierung eines Kindes (Einfügen in Wait-Queue). Der Grund der Terminierung ist abfragbar.
Terminiert ein Kindprozess vor dem Elternprozess entsteht ein Zombie (speicher de-allokiert, Exit-Status in PCB erhalten). Durch lesen des Exit-Status mit _wait()_ wird der Zombie entfernt.

### Waisen

Terminiert der Eltern- vor dem Kindprozess wird das Kind zum Waisen und vom Init-Prozess adoptiert (PPID=1).
Durch lösen von der Prozessgruppe und BenutzerID kann ein Waise zum Daemon werden.

## Prozessorverwaltung

- Scheduler: Wählt einen Prozess aus der Run-Queue (rechenwillig)
- Dispatcher: Realisiert Zustandsübergänge von rechenwillig zu rechnend

### Dispatching

1.  Zustand des rechnenden Prozesses zu wartend/rechenbereit
2.  Sichern des (zuvor) rechnenden Prozesses im PCB
3.  Laden des neuen Prozesskontexts
4.  Zustand des neuen Prozesses zu rechnend

### Scheduling

Der Scheduler entscheidet durch verschiedene Strategien welcher Prozess die CPU erhält.
Er wird aktiv wenn:

- ein neuer Prozess erzeugt wird (parent oder child)
- ein Prozess terminiert
- ein Prozess blockiert
- ein Interrupt auftritt

## Implementierung von Threads

- User Space
- Kernel Space
- Hybride Varianten

### User-Level-Threads

Das Betriebssystem verwaltet nur Prozesse und kennt keine Threads. 
Diese werden mitsamt Thread-Scheduler durch ein Threadpaket implementiert.
Beispiele: _thread\_create_, _thread\_exit_, _thread\_join_, _thread\_yield_, ...
Das System arbeitet also mit zweistufigem Scheduling (Prozesse durch BS, Threads durch RE).
Ein Thread kann nicht vom Laufzeitsystem unterbrochen werden.

Vorteile:
- BS-unabgängig
- schnelleres Umschalten, keine Systemcalls
- anwendungsspezifisches Sceduling

Nachteile:
- Blockieren des ganzen Prozess, bei Blockieren eines Threads
- Ein Thread kann die CPU innerhalb des Prozesses monopolisieren

### Kernel-Level-Threads

Der BS-Kern verwaltet Threads selbst (Threadtabelle). Threadoperationen lösen Trap aus. Nicht mehr so "lightweight".
Das Scheduling wählt nun Threads aus, nicht nur Prozesse.
Der Scheduler muss Thread-Zugehörigkeit berücksichtigen. Bei Blockieren eines Threads kann gewechselt werden.

### Hybride Implementierung

- BS-Kern verwaltet Kernel-Threads
- Scheduling erfolgt für Kernel-Threads
- n User-Level-Threads werden auf m Kernel-Threads abgebildet

## Scheduling

Optimierungsziele sind systemabhängig
- Alle Systeme
  - Fairness (zwischen Prozessen)
  - Balance (der Ressourcenauslastung)
- Batch-Systeme (z.B. Backup-Systeme)
  - Durchsatz (Aufträge pro Zeit)
  - Ausführungszeit
  - CPU-Belegung
- Interaktive Systeme (z.B. mit GUI)
  - Antwortzeit
  - Propotionalität (Erwartungshaltung der Nutzer)
- Echtzeit-Systeme
  - Einhalten von Deadlines
  - Vorhersagbargkeit (z.B. Vermeiden von Qualitätsverlust)

Scheduling-Klassen
- Nicht-unterbrechend (non-preemptive)
- Unterbrechend (preemptive)

Scheduling-Strategien
- Batch-Systeme
  - First-Come-First-Served (non-preemptive)
  - Shortest-Job-First (non-preemptive, bekannte Ausführungszeit)
  - Shortest Remaining Time Next (preemptive bei Ankunft neuer Prozesse, bekannte Ausführungszeit)
- Interaktive Systeme
  - Round-Robin Scheduling (preemptive nach Ablauf der Zeitscheibe)
  - Priority Scheduling (preemptive)
- Echtzeitsysteme
  - Earliest Deadline First (non-/preemptive möglich)
  - Rate-Monotonic Scheduling (preemptive)

## Multilevel Scheduling

- Short-Term-Scheduler
  - Wählt nächsten Prozess aus der Ready-Queue aus
  - oft aktiv
- Medium-Term-Scheduler
  - Entscheidet über ein-/auslagern von Prozessen
- Long-Term-Scheduler
  - Wählt Mix von IO-Bound/CPU-Bound Prozessen für die Ready-Queue
  - Nicht immer vorhanden
  - seltener aufgerufen
  

## Multicore Scheduling

- Zuordnung von Prozessen zu Kernen/Prozessoren
- Beachtung der Prozessoreigenschaften

### Varianten

- Time Sharing
  - Abwechselnde Nutzung
  - Langsam für abhängige Prozesse
- Space Sharing
  - Abhängige Prozesse werden gemeinsam zugewiesen
  - Verbleiben bis zum Terminieren
  - Leichte Kommunikation abhängiger Prozesse
  - Keine Kontextwechsel
  - Teils hohe Wartezeiten
- Gang Scheduling
  - Abhängige Prozesse als eine Gang behandelt
  - Gangs bekommen Zeitscheiben

### Implementierung 

Globale Warteschlange für alle CPUs
- Vorteile: 
  - Gute Auslastung
  - Fair für alle Prozesse
- Nachteile:
  - Schleckt skalierbar (Locks)
  - Schlechte Cache Lokalität (Prozess kann Kern wechseln)

Eine Warteschlange pro CPU
- Vorteile:
  - Einfache Implementierung
  - Skalierbar
  - Bessere Cache Lokalität
- Nachteil:
  - CPUs ungleich ausgelastet, unfair für Prozesse

Hybrid, eine globale Queue, pro CPU eine lokale
- Falls nötig verschieben zwischen CPU-Queues (Work Stealing)
- Vorteil:
  - Gute Auslastung
  - Moderate Cache Lokalität
- Nachteil:
  - Komplex

