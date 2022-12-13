# Kapitel 3: Parallele Systeme und Synchronisation

## Sequentielle und Parallele Programme

- Sequentiell
  - Determiniertes Programmverhalten
  - Gleiche Bedingungen und Eingabe -> gleiches Ergebnis
- Parallel
  - Programm(teile) unabhängig und in beliebieger Reihenfolge ausführen
  - Ausführung kann gleichzeitig oder zeitlich versetzt erfolgen
  - Kann nicht-determiniert sein

## Interaktion von parallelen Programmen

- Erwünscht: direkter Datenaustausch
- Unerwünscht: Konkurrenz um Ressourcen

### Interaktionsarten

- Kausale Beziehungen
- Kommunikation (Nachrichtenaustausch)
- Korrdinierung (Auftraggeber/-nehmer)
- Konkurrenz (um Ressourcen)

## Eigenschaften paralleler Systeme

- Determiniertheit
- Störungsfreiheit (bei festgelegter Ausführungsreihenfolge)
- Wechselseitiger Ausschluss (von exklusiven Ressourcen)
- Verklemmungsfreiheit (keine Deadlocks)
- Kein Verhungern (rechenwilliger Prozess darf nicht rechnen)


## Kritischer Abschnitt

- Programmabschnitt mit Zugriff auf gemeinsame Ressourcen
- Koordinierung zur Vermeidung von Race Conditions nötig
- Bei exklusiven Ressourcen erfolgt der Zugriff sequentiell

### Ablauf

1. Betreten eines kritischen Abschnitts
2. Ausführen der Aktionen im k.A.
3. Verlassen des k.A.
- Erfordert wechelseitigen Ausschluss

## Wechelseitiger Ausschluss

- Grundlage: Atomare, nicht teilbare Operationen

Prinzipien:
1. Kritische Abschnitte sind wechselseitig Ausgeschlossen
2. Muss unabhängig von der Ausführungsreihenfolge sein
3. Muss unabhängig von der Ausführungszeit sein
4. Darf kein unendliches Warten auslösen

## Lösungskonzepte

### Hardware

- Unterbrechungssperre
  - Verhindern der CPU-Abgabe im kritischen Abschnitt
  - Nur in Ein-Prozessor-Systemen möglich
  - I/O-Ereignisse köntnten verloren gehen
- Atomare Maschinenbefehle
  - Befehle sperren Speicherbus für alle CPUs
  - TSL: Test and Set Lock (mov rax, [lock]; mov [lock], 1; atomar)
    - Wenn rax=0, betrete kritischen Abschnitt sonst Schleife (Spin-Lock, aktives Warten)
  - CMPXCHG [lock], rbx: wenn rax=[lock], setze [lock]=rbx (rbx!=0)
    - Danach selbes Prinzip wie TSL
- BS-Dienste (low-level, fehlende Synchronisation)
  - sleep (Prozess wartend), wakeup (Prozess rechenbereit)
  - passives Warten, wecken wenn k.A. frei
- Semaphore (mit BS-Diensten)
  - ganzzahlige Kontrollvariable s
  - up() und down() zum verändern (atomar)
  - Wenn down() und s<0 -> Thread in Wait-Queue von Semaphore verschieben
  - Wenn up() und nachher s>=0 -> Thread in Wait-Queue wecken

## Deadlocks

Alle Prozesse einer Menge warten auf ein Ereignis das nur ein anderer dieser Prozesse aulösen kann.

## Bedingungen für Deadlocks

1. Exklusiv nutzbare Ressourcen
  - Gemeinsame Ressourcen nur exklusiv nutzbar
2. Belegen und Anfordern (Hold-and-Wait)
  - Belegt ein Prozess eine Ressource bleibt diese belegt auch wenn weitere angefordert werden 
3. Nicht entziehbar (no-preemption)
4. Zyklische Wartebedingungen
  - Es gibt eine Kette von min. zwei Prozessen die auf eine vom jeweils anderen Prozess belegte Ressource warten

Für das Auftreten von Deadlocks müssen alle diese Bedingungen erfüllt sein. Um Deadlocks zu verhindern muss min. eine Bedingung außer Kraft gesetzt werden.

## Strategien zum Umgang mit Deadlocks

1. Ignorieren (z.B. nicht wichtige Systemteile)
2. Deadlock-Detection (Erkennen und beseitigen)
3. Deadlock-Prevention (Deadlockbedingung außer Kraft setzen)
4. Deadlock-Avoidance (Vermeidung, BS erlaubt keine Ressourcenzuteilung mit potientiellem Deadlock)

### Deadlock-Detection

Suche Reihenfolge für mögliche Terminierung aller Prozesse
- Simuliere in aktuellem Zustand die volle Belegung eines Prozesses (wenn möglich)
- Gib **alle** Ressourcen dieses Prozesses frei
- Entferne diesen Prozess aus der Liste
- Wiederhole bis die Liste leer ist
Eine freie Liste garantiert eine deadlockfreie Durchführung. Ansonsten müssen Prozesse abgebronchen, Ressourcen entzogen oder verklemmte Prozesse zu Kontrollpunkten zurückgesetzt werden.

### Deadlock-Avoidance

Prüfe bei jeder Ressourcenbelegung ob durch zukünftige Anforderungen ein Deadlock entstehen kann, wenn ja wird die Ressourcenbelegung nicht zugelassen.
Beispiel: Bankier-Algorithmus (nicht praxistauglich)

### Deadlock-Prevention

Außerkraftsetzen einer der vier Deadlock-Bedingungen

- Exklusive Ressourcen
  - Spooling: Nur der Spooler-Prozess hat Zugriff auf Ressource, Zugriffe indirekt über den Spooler (Warteschlange)
  - Generell: Ressourcenzuweisung nur dann wenn absolut notwendig
- Belegen und Anfordern (Hold-and-Wait)
  - Anfordern von allen benötigten Ressourcen vor der Ausführung
  - Wenn eine Ressource nicht frei ist warten (belegte Ressourcen freigeben)
- Nicht unterbrechbare Ressourcen (no-preemption)
  - evtl. Virtualisierung von Ressourcen (nicht immer möglich)
- Zyklisches Warten (Circular Wait)
  - Ressourcen-Anforderung immer in festgelegter Ordnung (Reihenfolge)

## Livelock

Beispiel: Zwei parallele Prozesse benötigen belegte Ressourcen des jeweils anderen. Aus Höflichkeit geben beide ihre Ressourcen frei. Sie fordern die Ressourcen wieder an. Endlosschleife...

Trotz fortschreitender Ausführung kein Fortschritt.

## Starvation

Ein Prozess kann verhungern wenn seine Ressourcenanfrage nie akzeptiert wird weil z.B. höher priorisierte Prozesse Vorrang haben (unfaires Scheduling)

