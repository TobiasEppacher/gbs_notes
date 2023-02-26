# Kapitel 9 - Virtualisierung

## Einführung

- Abstrahierung von Hardware-Details erlaubt simulierte Hardwarefunktionalität
- Instruction Set Architecture: Interface zwischen BS und Hardware
- Application Binary Interface: Interface zwischen Anwendungen und BS/Hardware
- Virtuelle Laufzeitumgebung

## Anforderungen an virtualisierung

- Equivalence / Fidelity
  - BS und Anwendungen haben gleiches Verhalten wie auf einem nativen System
- Resource Control
  - Hypervisor hat Kontrolle über Ressourcen
  - Hypervisor isoliert Gast-BS gegeneinander
- Efficiency / P3erformance
  - Der Overhead durch die zusätzliche Virtualisierungsschicht muss gering gehalten werden
  - Möglichst wenige Instruktionen über das Virtualisierungssystem

## System Virtual Machines

- Hypervisor (Virtual Machine Monitor) virtualisiert ISA für Betriebssysteme
- Typ 1 Hypervisor (z.B. Hyper-V, Xen, ...)
  - Direkt auf Hardware ausgeführt
  - Benötigt Treiber für Hardware
- Typ 2 Hypervisor (z.B. VirtualBox)
  - Verwendet Dienste eines Host-BS für HW-Zugriff
  - u.a. dessen Treiber
- Aus Sicht des VMMs ist ein Gast-BS eine Anwendung
- Beliebige BS sind als Gast-BS möglich
- Mehrere Instanzen desselben BS können gleichzeitig laufen

## Gründe für System-VMs

- Emulation
  - BS unabhängig von HW-ISA ausführbar
  - Debuggen von BS bequemer
- Isolation
  - BS laufen isoliert voneinander auf derselben HW
  - Beschränkt Angriffe auf ein Gast-BS
- Ressourcennutzung
  - HW-Ressourcen können flexibler ausgenutzt werden

## CPU-Virtualisierung

## Speicher-Virtualisierung

- Durch Virtualisierung konkurrieren mehrere BS um Speicher
- Übergeordneter Hypervisor teilt Speicher zwischen den VMs auf
- Hypervisor überwacht Zugriffe der VMs auf die Seitentabelle
  - Shadow Page Tables (SPT)
  - Second Level Address Translation (SLAT, hardware-unterstützt)

### Shadow Page Tables