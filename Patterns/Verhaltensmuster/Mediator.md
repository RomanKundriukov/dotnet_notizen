Das **Mediator Pattern** ist ein Verhaltensmuster, das die Kommunikation zwischen mehreren Objekten über ein zentrales Vermittler-Objekt organisiert.

Dadurch müssen die beteiligten Objekte nicht direkt voneinander abhängig sein.

> [!info] Grundidee  
> Objekte kommunizieren nicht direkt miteinander, sondern über einen **Mediator**.

Ohne Mediator:

```text
ObjectA ↔ ObjectB
ObjectA ↔ ObjectC
ObjectA ↔ ObjectD
ObjectB ↔ ObjectC
ObjectB ↔ ObjectD
ObjectC ↔ ObjectD
```

Mit Mediator:

```text
             ObjectA
                │
                │
ObjectB ───► Mediator ◄─── ObjectC
                │
                │
             ObjectD
```

Dadurch wird aus einer komplizierten **Viele-zu-Viele-Kommunikation** eine wesentlich übersichtlichere Struktur.

---

# Wann sollte man Mediator verwenden?

Das Mediator Pattern eignet sich besonders:

- wenn viele Objekte miteinander kommunizieren;
    
- wenn die Beziehungen zwischen diesen Objekten kompliziert werden;
    
- wenn Klassen stark voneinander abhängig sind;
    
- wenn Objekte wiederverwendbar bleiben sollen;
    
- wenn die Kommunikationslogik zentral gesteuert werden soll;
    
- wenn man direkte Verbindungen zwischen vielen Komponenten vermeiden möchte.
    

---

# Typisches Problem

Angenommen, wir besitzen:

```text
Customer
Programmer
Tester
ProjectManager
Designer
DevOps
```

Ohne Mediator könnte jeder jeden kennen:

```text
Customer
├── kennt Programmer
├── kennt Tester
├── kennt Designer
└── kennt ProjectManager

Programmer
├── kennt Customer
├── kennt Tester
├── kennt DevOps
└── kennt Designer

Tester
├── kennt Programmer
├── kennt Customer
└── kennt DevOps
```

Dadurch entsteht eine starke Kopplung.

---

# Lösung mit Mediator

Stattdessen:

```text
Customer ─────┐
Programmer ───┤
Tester ───────┼──► ProjectMediator
Designer ─────┤
DevOps ───────┘
```

Alle Teilnehmer kennen hauptsächlich den Mediator.

Der Mediator kennt die Kommunikationsregeln.

---

# Grundstruktur

```text
                   Mediator
                      ▲
                      │
               ConcreteMediator
                /      |       \
               /       |        \
              ▼        ▼         ▼
        ColleagueA ColleagueB ColleagueC
```

---

# Teilnehmer des Patterns

## Mediator

Der `Mediator` definiert den Vertrag für die Kommunikation.

Zum Beispiel:

```csharp
public interface IMediator
{
    void Send(
        string message,
        Colleague sender);
}
```

---

## Colleague

`Colleague` ist die gemeinsame Basisklasse der beteiligten Objekte.

Jeder Colleague kennt den Mediator.

```csharp
public abstract class Colleague
{
    protected IMediator Mediator { get; }

    protected Colleague(IMediator mediator)
    {
        Mediator = mediator;
    }
}
```

---

## ConcreteColleague

Konkrete Kollegen sind beispielsweise:

```text
Customer
Programmer
Tester
```

Sie kommunizieren nicht direkt miteinander.

Stattdessen:

```csharp
Mediator.Send(message, this);
```

---

## ConcreteMediator

Der konkrete Mediator enthält die eigentliche Kommunikationslogik.

Er entscheidet:

```text
Wer hat die Nachricht gesendet?
        │
        ▼
Wer soll sie bekommen?
```

---

# Allgemeines Beispiel

## Mediator

```csharp
// Definiert den Vertrag für die Kommunikation
// zwischen mehreren Colleague-Objekten.
public interface IMediator
{
    void Send(
        string message,
        Colleague sender);
}
```

---

# Basisklasse Colleague

```csharp
public abstract class Colleague
{
    // Alle Kollegen kommunizieren
    // über denselben Mediator.
    protected IMediator Mediator { get; }

    protected Colleague(IMediator mediator)
    {
        Mediator = mediator;
    }

    // Nachricht über den Mediator senden.
    public void Send(string message)
    {
        Mediator.Send(message, this);
    }

    // Nachricht vom Mediator empfangen.
    public abstract void Notify(string message);
}
```

---

# ConcreteColleague1

```csharp
public class ConcreteColleague1 : Colleague
{
    public ConcreteColleague1(IMediator mediator)
        : base(mediator)
    {
    }

    public override void Notify(string message)
    {
        Console.WriteLine(
            $"Colleague 1 erhält: {message}");
    }
}
```

---

# ConcreteColleague2

```csharp
public class ConcreteColleague2 : Colleague
{
    public ConcreteColleague2(IMediator mediator)
        : base(mediator)
    {
    }

    public override void Notify(string message)
    {
        Console.WriteLine(
            $"Colleague 2 erhält: {message}");
    }
}
```

---

# ConcreteMediator

```csharp
public class ConcreteMediator : IMediator
{
    public ConcreteColleague1? Colleague1 { get; set; }

    public ConcreteColleague2? Colleague2 { get; set; }

    public void Send(
        string message,
        Colleague sender)
    {
        // Wenn Colleague1 sendet,
        // wird die Nachricht an Colleague2 weitergeleitet.
        if (sender == Colleague1)
        {
            Colleague2?.Notify(message);
        }
        // Wenn Colleague2 sendet,
        // wird die Nachricht an Colleague1 weitergeleitet.
        else if (sender == Colleague2)
        {
            Colleague1?.Notify(message);
        }
    }
}
```

---

# Verwendung

```csharp
ConcreteMediator mediator =
    new ConcreteMediator();

ConcreteColleague1 colleague1 =
    new ConcreteColleague1(mediator);

ConcreteColleague2 colleague2 =
    new ConcreteColleague2(mediator);

mediator.Colleague1 = colleague1;
mediator.Colleague2 = colleague2;

colleague1.Send("Hallo von Colleague 1");

colleague2.Send("Hallo von Colleague 2");
```

---

# Ablauf

```text
Colleague1
    │
    │ Send()
    ▼
Mediator
    │
    │ entscheidet Empfänger
    ▼
Colleague2
```

Andersherum:

```text
Colleague2
    │
    ▼
Mediator
    │
    ▼
Colleague1
```

---

# Der wichtigste Punkt

Ohne Mediator würde `ConcreteColleague1` möglicherweise direkt schreiben:

```csharp
_colleague2.Notify(message);
```

Damit müsste `ConcreteColleague1` den konkreten `ConcreteColleague2` kennen.

Mit Mediator:

```csharp
Mediator.Send(message, this);
```

kennt Colleague1 nur den Mediator.

---

# Praktisches Beispiel: Softwareprojekt

Ein Softwareprojekt besitzt verschiedene Beteiligte:

```text
Customer
Programmer
Tester
```

Normalerweise gibt es einen zentralen Ansprechpartner beziehungsweise Projektmanager.

Dieser übernimmt die Rolle des Mediators.

---

# Kommunikation

Ein möglicher Ablauf:

```text
Customer
   │
   │ "Wir brauchen eine neue Anwendung."
   ▼
Project Manager
   │
   ▼
Programmer
```

Danach:

```text
Programmer
   │
   │ "Die Anwendung ist fertig."
   ▼
Project Manager
   │
   ▼
Tester
```

Danach:

```text
Tester
   │
   │ "Die Anwendung wurde getestet."
   ▼
Project Manager
   │
   ▼
Customer
```

---

# Mediator Interface

```csharp
public interface IProjectMediator
{
    // Vermittelt eine Nachricht zwischen
    // den beteiligten Projektmitgliedern.
    void Send(
        string message,
        ProjectMember sender);
}
```

---

# Gemeinsame Basisklasse

```csharp
public abstract class ProjectMember
{
    protected IProjectMediator Mediator { get; }

    protected ProjectMember(
        IProjectMediator mediator)
    {
        Mediator = mediator;
    }

    // Nachricht über den Mediator senden.
    public virtual void Send(string message)
    {
        Mediator.Send(message, this);
    }

    // Nachricht vom Mediator empfangen.
    public abstract void Notify(string message);
}
```

---

# Customer

```csharp
public class Customer : ProjectMember
{
    public Customer(IProjectMediator mediator)
        : base(mediator)
    {
    }

    public override void Notify(string message)
    {
        Console.WriteLine(
            $"Nachricht an den Kunden: {message}");
    }
}
```

---

# Programmer

```csharp
public class Programmer : ProjectMember
{
    public Programmer(IProjectMediator mediator)
        : base(mediator)
    {
    }

    public override void Notify(string message)
    {
        Console.WriteLine(
            $"Nachricht an den Entwickler: {message}");
    }
}
```

---

# Tester

```csharp
public class Tester : ProjectMember
{
    public Tester(IProjectMediator mediator)
        : base(mediator)
    {
    }

    public override void Notify(string message)
    {
        Console.WriteLine(
            $"Nachricht an den Tester: {message}");
    }
}
```

---

# ProjectManagerMediator

```csharp
public class ProjectManagerMediator
    : IProjectMediator
{
    public ProjectMember? Customer { get; set; }

    public ProjectMember? Programmer { get; set; }

    public ProjectMember? Tester { get; set; }

    public void Send(
        string message,
        ProjectMember sender)
    {
        // Nachricht vom Kunden:
        // Der Entwickler soll die Aufgabe erhalten.
        if (sender == Customer)
        {
            Programmer?.Notify(message);
        }

        // Nachricht vom Entwickler:
        // Der Tester soll mit dem Testen beginnen.
        else if (sender == Programmer)
        {
            Tester?.Notify(message);
        }

        // Nachricht vom Tester:
        // Der Kunde soll das Ergebnis erhalten.
        else if (sender == Tester)
        {
            Customer?.Notify(message);
        }
    }
}
```

---

# Verwendung

```csharp
public class Program
{
    public static void Main()
    {
        ProjectManagerMediator mediator =
            new ProjectManagerMediator();

        Customer customer =
            new Customer(mediator);

        Programmer programmer =
            new Programmer(mediator);

        Tester tester =
            new Tester(mediator);

        // Teilnehmer beim Mediator registrieren.
        mediator.Customer = customer;
        mediator.Programmer = programmer;
        mediator.Tester = tester;

        customer.Send(
            "Es gibt einen neuen Auftrag. Eine Anwendung muss entwickelt werden.");

        programmer.Send(
            "Die Anwendung ist fertig und muss getestet werden.");

        tester.Send(
            "Die Anwendung wurde getestet und kann ausgeliefert werden.");
    }
}
```

---

# Ausgabe

```text
Nachricht an den Entwickler: Es gibt einen neuen Auftrag. Eine Anwendung muss entwickelt werden.

Nachricht an den Tester: Die Anwendung ist fertig und muss getestet werden.

Nachricht an den Kunden: Die Anwendung wurde getestet und kann ausgeliefert werden.
```

---

# Ablauf Schritt für Schritt

## 1. Kunde sendet eine Nachricht

```csharp
customer.Send(
    "Es gibt einen neuen Auftrag.");
```

Intern:

```text
Customer
   │
   ▼
Mediator.Send(...)
```

Der Mediator erkennt:

```csharp
sender == Customer
```

und führt aus:

```csharp
Programmer?.Notify(message);
```

Also:

```text
Customer
   │
   ▼
Mediator
   │
   ▼
Programmer
```

---

# 2. Entwickler ist fertig

```csharp
programmer.Send(
    "Die Anwendung ist fertig.");
```

Der Mediator erkennt:

```csharp
sender == Programmer
```

und sendet an:

```text
Tester
```

Also:

```text
Programmer
   │
   ▼
Mediator
   │
   ▼
Tester
```

---

# 3. Tester ist fertig

```csharp
tester.Send(
    "Die Anwendung wurde getestet.");
```

Der Mediator erkennt:

```csharp
sender == Tester
```

und informiert den Kunden:

```text
Tester
   │
   ▼
Mediator
   │
   ▼
Customer
```

---

# Ohne Mediator

Ohne Mediator könnte der Code ungefähr so aussehen:

```csharp
public class Customer
{
    private Programmer _programmer;

    public void CreateOrder()
    {
        _programmer.StartDevelopment();
    }
}
```

Der Programmer müsste wiederum Tester kennen:

```csharp
public class Programmer
{
    private Tester _tester;

    public void FinishDevelopment()
    {
        _tester.StartTesting();
    }
}
```

Der Tester müsste wiederum den Kunden kennen:

```csharp
public class Tester
{
    private Customer _customer;

    public void FinishTesting()
    {
        _customer.Notify();
    }
}
```

Dadurch entstehen direkte Abhängigkeiten:

```text
Customer ─────► Programmer
                   │
                   ▼
                 Tester
                   │
                   ▼
                Customer
```

---

# Mit Mediator

```text
Customer ─────┐
              │
Programmer ───┼──► Mediator
              │
Tester ───────┘
```

Jedes Objekt kennt hauptsächlich:

```text
Mediator
```

statt alle anderen Beteiligten.

---

# Viele-zu-Viele vs. Einer-zu-Viele

Ohne Mediator:

```text
A ↔ B
A ↔ C
A ↔ D
B ↔ C
B ↔ D
C ↔ D
```

Mit Mediator:

```text
       A
       │
       ▼
B ─► Mediator ◄─ C
       ▲
       │
       D
```

Das reduziert die Anzahl direkter Beziehungen erheblich.

---

# Vorteil: geringere Kopplung

Nehmen wir an, `Programmer` kennt direkt:

```text
Customer
Tester
DevOps
Designer
```

Dann ist `Programmer` stark an diese Klassen gekoppelt.

Mit Mediator kennt er nur:

```csharp
IProjectMediator
```

Dadurch kann der `Programmer` leichter:

- getestet;
    
- wiederverwendet;
    
- verändert;
    
- erweitert
    

werden.

---

# Vorteil: zentrale Kommunikationslogik

Die Regeln:

```text
Customer → Programmer
Programmer → Tester
Tester → Customer
```

befinden sich an einem zentralen Ort:

```csharp
ProjectManagerMediator
```

Dadurch kann man den Workflow leichter nachvollziehen.

---

# Nachteil: Mediator kann zu groß werden

Das ist der wichtigste Nachteil dieses Patterns.

Angenommen, wir haben:

```text
Customer
Programmer
Tester
Designer
DevOps
Support
ProductOwner
Architect
SecurityEngineer
```

und der Mediator enthält hunderte Bedingungen:

```csharp
if (sender == Customer)
{
    // ...
}
else if (sender == Programmer)
{
    // ...
}
else if (sender == Tester)
{
    // ...
}
else if (...)
{
    // ...
}
```

Dann kann der Mediator selbst zu einer riesigen Klasse werden.

Das wird häufig als:

**God Object**

bezeichnet.

> [!warning]  
> Mediator reduziert die Komplexität zwischen den Teilnehmern, kann diese Komplexität aber in den Mediator verschieben.

---

# Moderner Mediator mit Events

Ein Mediator muss nicht zwingend über:

```csharp
Send(string message, sender)
```

implementiert werden.

In C# können beispielsweise auch:

```text
Events
Commands
Notifications
Message Bus
```

verwendet werden.

---

# Beispiel mit Nachrichtentypen

Statt:

```csharp
Send(string message, ProjectMember sender);
```

könnten wir stärker typisierte Nachrichten verwenden.

```csharp
public interface IMessage
{
}
```

Zum Beispiel:

```csharp
public record OrderCreated(
    string Description) : IMessage;

public record DevelopmentCompleted(
    string Description) : IMessage;

public record TestingCompleted(
    string Description) : IMessage;
```

Dadurch vermeiden wir, Geschäftslogik ausschließlich über beliebige Strings abzubilden.

---

# Moderner Mediator-Vertrag

```csharp
public interface IMediator
{
    void Send<TMessage>(TMessage message)
        where TMessage : IMessage;
}
```

Das ist insbesondere bei größeren Anwendungen oft sauberer.

---

# Mediator und Dependency Injection

In modernen .NET-Anwendungen wird ein Mediator häufig über Dependency Injection eingebunden.

Zum Beispiel:

```csharp
public class OrderService
{
    private readonly IMediator _mediator;

    public OrderService(IMediator mediator)
    {
        _mediator = mediator;
    }
}
```

Dann:

```csharp
_mediator.Send(...);
```

Dadurch hängt die Klasse nur vom Interface:

```csharp
IMediator
```

ab.

---

# Mediator in CQRS

Das Mediator-Prinzip begegnet einem häufig bei **CQRS**.

Zum Beispiel:

```text
Controller
   │
   ▼
Mediator
   │
   ▼
CreateOrderCommandHandler
```

Der Controller muss den konkreten Handler nicht kennen.

Er sendet lediglich:

```text
CreateOrderCommand
```

an den Mediator.

---

# Beispiel

```csharp
public record CreateOrderCommand(
    string ProductName);
```

Der Client:

```csharp
await mediator.Send(
    new CreateOrderCommand("Laptop"));
```

Der Mediator sucht den passenden Handler:

```text
CreateOrderCommand
        │
        ▼
     Mediator
        │
        ▼
CreateOrderCommandHandler
```

Dieses Prinzip ist besonders in Clean Architecture sehr verbreitet.

---

# Mediator vs. Observer

Mediator und Observer werden häufig verwechselt.

## Mediator

Kommunikation läuft über eine zentrale Komponente.

```text
A
 \
  \
Mediator
  /
 /
B
```

Der Mediator kennt die Kommunikationsregeln.

---

## Observer

Ein Objekt veröffentlicht Ereignisse an mehrere Abonnenten.

```text
Subject
  │
  ├──► ObserverA
  ├──► ObserverB
  └──► ObserverC
```

Der Fokus liegt auf:

> „Informiere alle Interessenten über eine Änderung.“

---

# Unterschied Mediator vs. Observer

|Mediator|Observer|
|---|---|
|zentrale Vermittlungslogik|Publish/Subscribe|
|koordiniert Kommunikation|informiert Beobachter|
|Teilnehmer kommunizieren über Mediator|Subject benachrichtigt Observer|
|Mediator kennt häufig konkrete Regeln|Publisher kennt Beobachter oft nur über Interface|

---

# Mediator vs. Facade

Auch diese Patterns können ähnlich wirken.

## Facade

Bietet eine vereinfachte Schnittstelle zu einem komplexen Subsystem.

```text
Client
  │
  ▼
Facade
  │
  ├── ServiceA
  ├── ServiceB
  └── ServiceC
```

Die Services müssen nicht unbedingt über die Facade miteinander kommunizieren.

---

## Mediator

Koordiniert dagegen die **Kommunikation zwischen mehreren Objekten**.

```text
A ──┐
B ──┼──► Mediator
C ──┘
```

Merksatz:

```text
Facade
→ vereinfacht Zugriff auf ein System

Mediator
→ organisiert Kommunikation innerhalb eines Systems
```

---

# Mediator vs. Chain of Responsibility

## Chain of Responsibility

Eine Anfrage wandert von Handler zu Handler:

```text
HandlerA
   ↓
HandlerB
   ↓
HandlerC
```

Die Frage ist:

> „Welcher Handler kann die Anfrage bearbeiten?“

---

## Mediator

Mehrere Objekte kommunizieren über einen zentralen Vermittler:

```text
A ──┐
B ──┼──► Mediator
C ──┘
```

Die Frage ist:

> „Wie sollen diese Objekte miteinander kommunizieren?“

---

# Mediator vs. Observer vs. Chain

```text
Mediator
→ zentrale Kommunikation


Observer
→ Ereignisse an mehrere Beobachter verteilen


Chain of Responsibility
→ Request durch Handler-Kette weiterreichen
```

---

# Typische Einsatzbereiche

Mediator eignet sich beispielsweise für:

- Dialogfenster und UI-Komponenten;
    
- komplexe Formulare;
    
- Chat-Systeme;
    
- Workflow-Steuerung;
    
- Projektkoordination;
    
- Command/Query-Verarbeitung;
    
- modulare Anwendungen;
    
- CQRS;
    
- Nachrichtenvermittlung zwischen Komponenten.
    

---

# Beispiel: UI ohne Mediator

Angenommen, ein Formular besitzt:

```text
Checkbox
TextBox
Button
ComboBox
```

Direkte Kommunikation könnte schnell so aussehen:

```text
Checkbox → Button
Checkbox → TextBox
ComboBox → Button
ComboBox → TextBox
TextBox → Button
```

---

# UI mit Mediator

```text
Checkbox ──┐
TextBox ───┼──► DialogMediator
Button ────┤
ComboBox ──┘
```

Der Dialog-Mediator entscheidet:

```text
Checkbox aktiviert?
→ TextBox aktivieren

TextBox leer?
→ Button deaktivieren

ComboBox geändert?
→ andere UI-Komponenten aktualisieren
```

Das ist ein klassischer Anwendungsfall des Mediator Patterns.

---

# Vorteile

- reduziert direkte Abhängigkeiten zwischen Objekten;
    
- verhindert komplizierte Viele-zu-Viele-Beziehungen;
    
- zentralisiert Kommunikationsregeln;
    
- Teilnehmer lassen sich leichter wiederverwenden;
    
- Teilnehmer hängen häufig nur von einem Mediator-Interface ab;
    
- Interaktionen lassen sich an einer zentralen Stelle verändern;
    
- kann Testbarkeit verbessern.
    

---

# Nachteile

- der Mediator kann sehr groß werden;
    
- Geschäftslogik kann sich zu stark im Mediator sammeln;
    
- ein komplexer Mediator kann schwer testbar werden;
    
- zu viel zentrale Steuerung kann zu einem God Object führen;
    
- für wenige einfache Objekte kann das Pattern unnötig sein.
    

---

# Wann Mediator nicht sinnvoll ist

Wenn nur zwei Klassen miteinander kommunizieren:

```text
ClassA
  │
  ▼
ClassB
```

und diese Beziehung klar und stabil ist, braucht man meist keinen zusätzlichen Mediator.

Ein:

```text
A → Mediator → B
```

würde dann möglicherweise nur unnötige Komplexität erzeugen.

> [!tip]  
> Mediator lohnt sich besonders dann, wenn die Kommunikationsbeziehungen tatsächlich unübersichtlich werden.

---

# Klassische Struktur

```text
                    IMediator
                        ▲
                        │
                 ConcreteMediator
                   /     |     \
                  /      |      \
                 ▼       ▼       ▼
          ColleagueA ColleagueB ColleagueC
```

Kommunikation:

```text
ColleagueA
    │
    ▼
Mediator
    │
    ▼
ColleagueB
```

Nicht:

```text
ColleagueA ─────► ColleagueB
```

---

# Das solltest du dir merken

Mediator:

```csharp
public interface IMediator
{
    void Send(
        string message,
        Colleague sender);
}
```

Colleague:

```csharp
public abstract class Colleague
{
    protected IMediator Mediator { get; }

    protected Colleague(IMediator mediator)
    {
        Mediator = mediator;
    }

    public void Send(string message)
    {
        Mediator.Send(message, this);
    }

    public abstract void Notify(string message);
}
```

Konkreter Mediator:

```csharp
public class ConcreteMediator : IMediator
{
    public void Send(
        string message,
        Colleague sender)
    {
        // Entscheiden:
        // Wer soll die Nachricht erhalten?
    }
}
```

---

# Merksatz

> **Mediator zentralisiert die Kommunikation zwischen mehreren Objekten, damit diese nicht direkt voneinander abhängig sein müssen.**

Noch einfacher:

```text
Ohne Mediator:

A ↔ B ↔ C ↔ D


Mit Mediator:

A ─┐
B ─┼──► Mediator
C ─┤
D ─┘
```

Oder:

```text
Colleague:
"Ich möchte etwas mitteilen."

Mediator:
"Ich entscheide, wer diese Information bekommen soll."
```

---

> [!summary] Zusammenfassung  
> Das **Mediator Pattern** ist ein **Verhaltensmuster**.
> 
> Es reduziert direkte Abhängigkeiten zwischen mehreren miteinander kommunizierenden Objekten.
> 
> Statt:
> 
> ```text
> A ↔ B
> A ↔ C
> B ↔ C
> ```
> 
> kommunizieren die Objekte über einen zentralen Mediator:
> 
> ```text
> A ─┐
> B ─┼──► Mediator
> C ─┘
> ```
> 
> Die beteiligten Objekte – die **Colleagues** – kennen hauptsächlich den Mediator:
> 
> ```csharp
> Mediator.Send(message, this);
> ```
> 
> Der Mediator entscheidet anschließend, welcher andere Teilnehmer reagieren soll.
> 
> Ein wichtiger Nachteil besteht darin, dass der Mediator selbst zu komplex werden und sich zu einem **God Object** entwickeln kann.
> 
> In modernen .NET-Anwendungen findet man das Mediator-Prinzip häufig auch bei **CQRS**, Commands, Queries und modularer Kommunikation.
> 
> **Kurz gesagt:**  
> `Mediator = Kommunikation zwischen vielen Objekten über eine zentrale Vermittlungsstelle organisieren.`