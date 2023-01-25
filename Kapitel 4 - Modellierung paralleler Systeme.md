# Kapitel 4 - Modellierung Paralleler Systeme

## Konzepte zur Verhaltensbeschreibung

- Sicht auf Tätigkeiten: Aktionen werden modelliert (z.B. Maschinenbefehl)
- Sicht auf Veränderungen: Modellierte Einheiten die Veränderungen bewirken

## Eigenschaften paralleler Systeme

### Determiniertheit

- Bei gleichen Bedingungen entstehen die gleichen Ereignisse

### Störungsfreiheit

- Mit einer Festgelegten Ausführungsreihenfolge  paralleler Ereignisse und deren Aktionen wird das Ergebnis nicht beeinflusst

### Wechselseitiger Ausschluss exklusiver Ressourcen

- Maximal ein Prozess darf gleichzeitig auf gemeinsame Ressourcen zugreifen

### Veklemmungsfreiheit (Deadlock)

- Keine zyklische Wartesituation zwischen mehreren Prozessen

### Kein Verhungern (Starvation)

- Es gibt keine rechenwilligen Prozesse die unendlich lange aufgeschoben werden

## Petri-Netze

- Graphische Modellierung eines parallelen Systems und dessen Abläufen
- Besteht aus:
  - Stellen (passive Elemente, z.B. Speicherzellen)
  - Transitionen (aktive Elemente, z.B. Prozesse)
  - Flussrelation (beschreibt gerichtete Kanten zwischen Stellen und Transitionen)
- Stellen werden als Kreise dargestellt
- Transitionen werden als Rechtecke dargestellt
- Übergänge zwischen diesen Knoten (Stellen und Transitionen) werden als Pfeile dargestellt

### Verfeinerung

- Transitionen können ihrerseits wieder als eigenständiges Petrinetz modelliert werden
- Ein- und Ausgehende Kanten müssen dabei erhalten bleiben
- Somit ist eine schrittweise Verfeinerung möglich

## Dynamisches Verhalten eines Petrinetzes

- Stellen haben Kapazitäten (unendlich wenn nicht angegeben)
- Kanten haben Gewichtungen (1 wenn nicht angegeben)
- Die aktuell auf Stellen verteilte Tokens bilden eine _**Belegung**_
  - Ist ein Netz natürlichzahlig belegt heißt es _**Stellen-Transitionsnetz**_
  - Besitzt ein Netz nur Stellenbelegungen von 0 oder 1 heißt es _**Bedingungs-/Ereignis-**_ oder _**Boolsches Netz**_
- Eine Transition

## Schaltregel

- Eine Belegung beschreibt den aktuellen Zustand eines Netzes
- Ein Zustandsübergang erfolgt durch das Schalten von Transitionen
- Dazu muss für alle Kanten der Transition gelten:
  - Belegung vorige Stelle >= Gewicht der eingehenden Kante (genug Knoten in der eingehenden Stelle)
  - Kapazität - Belegung nachfolgende Stelle >= Gewicht der ausgehenden Kante
    (genug Platz in der ausgehenden Stelle)
- Durch das Schalten werden Tokens entsprechend der Kantengewichtung von Stellen entfernt/hinzugefügt

## Modellierung von Systemeigenschaften

### Nebenläufigkeit

- Zwei Transitionen sind nebenläufig wenn sie unabhängig voneinander schalten können

### Nichtdeterminismus

- Haben zwei Transitionen gemeinsame Eingangs- oder Ausgangsstellen und kann mit der aktuellen Belegung nur eine schalten, so stehen diese Transitionen in Konflikt
- Es entsteht eine nichtdeterministische Auswahl der zu schaltenden Transition

## Erreichbarkeit

- Eine Belegung M' ist von M erreichbar, wenn man M' der Endzustand einer endlichen Sequenz schaltender Transitionen ist
- Darstellbar im Erreichbarkeitsgraphen:
  - Knoten sind erreichbare Belegungen
  - Kanten sind schaltende Transitionen für den jeweiligen Zustandsübergang

## Lebendigkeit

- Ein Netz ist lebendig wenn von jeder erreichbaren Belegung ein Zustand erreicht werden kann, sodass eine Transition t schalten kann
- Dies muss für alle Transitionen gelten
- kurz: jede Transition muss von jeder Belegung ausgehend irgendwie wieder schalten können

## Verklemmung

- Ein Netz ist verklemmt wenn es zu einer erreichbaren Belegung keine Transition gibt die schalten kann
- **Lebendigkeit impliziert Verklemmungsfreiheit!!! UMGEKEHRT NICHT!!!**

## Verhungern

- Gibt es **mindestens eine** unendliche Sequenz schaltender Transitionen in der eine schaltbereite Transition nicht/endlich oft vorkommt, kann diese verhungern

## Fairness

- Gibt es **keine** unendliche Sequenz schaltender Transitionen in der eine schaltbereite Transition nicht/endlich oft vorkommt ist das Netz fair
- Kein Verhungern = Fair | Verhungern = Nicht Fair

