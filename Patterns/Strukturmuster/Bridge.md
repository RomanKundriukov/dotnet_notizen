Das **Bridge Pattern** ist ein Strukturmuster, das eine **Abstraktion von ihrer Implementierung trennt**, sodass beide Seiten **unabhängig voneinander weiterentwickelt und ausgetauscht** werden können.

> [!info] Grundidee  
> Statt eine große Vererbungshierarchie für jede mögliche Kombination aufzubauen, werden **zwei getrennte Hierarchien** erstellt:
> 
> 1. die **Abstraktion**
> 2. die **Implementierung**
> 
> Beide werden durch Komposition miteinander verbunden.

Vereinfacht:

```
Abstraction
    │
    │ verwendet
    ▼
Implementor
```

Daher der Name:

```
Abstraction ───── Bridge ───── Implementor
```

---

# Wann sollte man Bridge verwenden?

Das Bridge Pattern eignet sich besonders:

- wenn Abstraktion und Implementierung unabhängig verändert werden sollen;
- wenn eine feste Verbindung zwischen einer Abstraktion und einer konkreten Implementierung vermieden werden soll;
- wenn zwei voneinander unabhängige Variantenachsen existieren;
- wenn sonst sehr viele Klassen für alle Kombinationen entstehen würden;
- wenn die konkrete Implementierung zur Laufzeit ausgetauscht werden soll.

Typische Situation:

```
mehrere Arten von Abstraktionen
+
mehrere Arten von Implementierungen
```

Beispiel:

```
Programmierer:
├── Freelancer
└── Firmenentwickler

Programmiersprachen:
├── C#
├── C++
└── Java
```

Ohne Bridge könnten daraus Klassen entstehen wie:

```
CSharpFreelancer
CppFreelancer
JavaFreelancer

CSharpCorporateProgrammer
CppCorporateProgrammer
JavaCorporateProgrammer
```

Mit Bridge bleiben beide Dimensionen getrennt.

---

# Das eigentliche Problem

Angenommen, wir haben zwei unabhängige Eigenschaften:

## Art des Programmierers

```
Freelancer
Corporate Programmer
```

## Programmiersprache

```
C#
C++
```

Wenn wir alles über Vererbung kombinieren, könnten wir bekommen:

```
Programmer
├── CppFreelanceProgrammer
├── CSharpFreelanceProgrammer
├── CppCorporateProgrammer
└── CSharpCorporateProgrammer
```

Wenn später Java hinzukommt:

```
JavaFreelanceProgrammer
JavaCorporateProgrammer
```

Wenn noch weitere Programmierertypen hinzukommen, wächst die Zahl der Klassen weiter.

---

# Kombinatorische Explosion

Angenommen:

```
3 Arten von Programmierern
×
4 Programmiersprachen
```

Dann wären möglicherweise:

```
3 × 4 = 12 Kombinationen
```

notwendig.

Bei:

```
5 Abstraktionen
×
6 Implementierungen
```

bereits:

```
30 Kombinationen
```

Bridge verhindert diese Explosion, indem beide Dimensionen getrennt werden.

---

# Lösung mit Bridge

Statt:

```
FreelanceCSharpProgrammer
FreelanceCppProgrammer
CorporateCSharpProgrammer
CorporateCppProgrammer
```

erstellen wir:

```
Programmer
├── FreelanceProgrammer
└── CorporateProgrammer
```

und unabhängig davon:

```
ILanguage
├── CSharpLanguage
└── CppLanguage
```

Die Verbindung:

```
Programmer
    │
    ▼
ILanguage
```

Ein Programmierer **besitzt beziehungsweise verwendet** eine Sprache.

---

# Grundstruktur

```
                  Abstraction
                      ▲
                      │
             RefinedAbstraction
                      │
                      │ verwendet
                      ▼
                  Implementor
                      ▲
             ┌────────┴────────┐
             │                 │
   ConcreteImplementorA ConcreteImplementorB
```

Wir haben also **zwei parallele Hierarchien**:

```
Abstraction-Hierarchie
```

und:

```
Implementor-Hierarchie
```

---

# Teilnehmer des Patterns

## Abstraction

`Abstraction` definiert die öffentliche, höherwertige Schnittstelle für den Client.

Sie besitzt eine Referenz auf:

```
Implementor
```

und delegiert Teile ihrer Arbeit an diese Implementierung.

---

## RefinedAbstraction

Eine `RefinedAbstraction` erweitert oder spezialisiert die Abstraktion.

Beispiele:

```
FreelanceProgrammer
CorporateProgrammer
```

---

## Implementor

`Implementor` definiert die grundlegende Schnittstelle für die konkrete Implementierung.

Zum Beispiel:

```
public interface IImplementor
{
    void OperationImplementation();
}
```

---

## ConcreteImplementor

Konkrete Implementierungen:

```
ConcreteImplementorA
ConcreteImplementorB
```

implementieren das technische beziehungsweise konkrete Verhalten.

---

## Client

Der Client arbeitet hauptsächlich mit der `Abstraction`.

Er kann eine konkrete Implementierung injizieren oder austauschen.

---

# Allgemeine Struktur in C#

```
// Definiert die Schnittstelle für konkrete Implementierungen.
public interface IImplementor
{
    void OperationImplementation();
}
```

Konkrete Implementierung A:

```
public class ConcreteImplementorA : IImplementor
{
    public void OperationImplementation()
    {
        Console.WriteLine(
            "Implementierung A wird ausgeführt.");
    }
}
```

Konkrete Implementierung B:

```
public class ConcreteImplementorB : IImplementor
{
    public void OperationImplementation()
    {
        Console.WriteLine(
            "Implementierung B wird ausgeführt.");
    }
}
```

Abstraktion:

```
public abstract class Abstraction
{
    protected IImplementor Implementor { get; set; }

    protected Abstraction(IImplementor implementor)
    {
        Implementor = implementor;
    }

    public virtual void Operation()
    {
        // Die konkrete Arbeit wird
        // an die Implementierung delegiert.
        Implementor.OperationImplementation();
    }
}
```

RefinedAbstraction:

```
public class RefinedAbstraction : Abstraction
{
    public RefinedAbstraction(
        IImplementor implementor)
        : base(implementor)
    {
    }

    public override void Operation()
    {
        Console.WriteLine(
            "Zusätzliche Logik der Abstraktion.");

        base.Operation();
    }
}
```

---

# Verwendung

```
Abstraction abstraction =
    new RefinedAbstraction(
        new ConcreteImplementorA());

abstraction.Operation();
```

Dann könnte später die Implementierung gewechselt werden.

Eine sauberere moderne Variante wäre beispielsweise über eine Methode:

```
public void ChangeImplementor(
    IImplementor implementor)
{
    Implementor = implementor;
}
```

Dann:

```
abstraction.ChangeImplementor(
    new ConcreteImplementorB());

abstraction.Operation();
```

---

# Ablauf

```
Client
  │
  ▼
RefinedAbstraction
  │
  │ Operation()
  ▼
Implementor
  │
  ▼
ConcreteImplementorA
```

Nach dem Austausch:

```
Client
  │
  ▼
RefinedAbstraction
  │
  ▼
ConcreteImplementorB
```

Die Abstraktion bleibt dieselbe.

Nur die Implementierung verändert sich.

---

# Warum heißt das Pattern Bridge?

Die Abstraktionshierarchie und die Implementierungshierarchie existieren getrennt:

```
Abstraction-Hierarchie

Programmer
├── Freelancer
└── Corporate
```

und:

```
Implementierungs-Hierarchie

ILanguage
├── C#
├── C++
└── Java
```

Die Referenz:

```
protected ILanguage Language;
```

bildet die **Brücke** zwischen beiden Hierarchien.

```
Programmer ─────────► ILanguage
          Bridge
```

---

# Praktisches Beispiel: Programmierer und Programmiersprachen

Wir haben verschiedene Arten von Programmierern:

```
Programmer
├── FreelanceProgrammer
└── CorporateProgrammer
```

und verschiedene Programmiersprachen:

```
ILanguage
├── CppLanguage
└── CSharpLanguage
```

Beide Hierarchien sollen unabhängig voneinander erweiterbar sein.

---

# Implementor: ILanguage

```
public interface ILanguage
{
    // Quellcode bauen beziehungsweise kompilieren.
    void Build();

    // Erzeugtes Programm ausführen.
    void Execute();
}
```

Die Sprache bildet hier die **Implementor-Seite** des Bridge Patterns.

---

# ConcreteImplementor: C++

```
public sealed class CppLanguage : ILanguage
{
    public void Build()
    {
        Console.WriteLine(
            "Der C++-Compiler kompiliert den Quellcode.");
    }

    public void Execute()
    {
        Console.WriteLine(
            "Das kompilierte native Programm wird ausgeführt.");
    }
}
```

---

# ConcreteImplementor: C#

```
public sealed class CSharpLanguage : ILanguage
{
    public void Build()
    {
        Console.WriteLine(
            "Der C#-Compiler kompiliert den Quellcode.");
    }

    public void Execute()
    {
        Console.WriteLine(
            "Die .NET-Anwendung wird von der Runtime ausgeführt.");
    }
}
```

---

# Hinweis zum alten Beispiel

Die ursprüngliche Quelle beschreibt die Ausführung von C# sehr vereinfacht.

Für das Verständnis des Bridge Patterns ist diese technische Detailfrage nicht wichtig.

Entscheidend ist:

```
CppLanguage
→ implementiert Build() und Execute()

CSharpLanguage
→ implementiert Build() und Execute()
```

Beide stellen unterschiedliche konkrete Implementierungen derselben Abstraktion dar.

---

# Abstraction: Programmer

```
public abstract class Programmer
{
    protected ILanguage Language { get; private set; }

    protected Programmer(ILanguage language)
    {
        Language = language;
    }

    public void ChangeLanguage(ILanguage language)
    {
        // Die konkrete Implementierung
        // kann zur Laufzeit ausgetauscht werden.
        Language = language;
    }

    public virtual void DoWork()
    {
        // Die Abstraktion delegiert
        // technische Details an ILanguage.
        Language.Build();
        Language.Execute();
    }

    public abstract void EarnMoney();
}
```

`Programmer` kennt nicht:

```
C#
C++
Java
```

direkt.

Er kennt nur:

```
ILanguage
```

---

# RefinedAbstraction: Freelancer

```
public sealed class FreelanceProgrammer : Programmer
{
    public FreelanceProgrammer(
        ILanguage language)
        : base(language)
    {
    }

    public override void EarnMoney()
    {
        Console.WriteLine(
            "Bezahlung für den abgeschlossenen Auftrag erhalten.");
    }
}
```

---

# RefinedAbstraction: Firmenentwickler

```
public sealed class CorporateProgrammer : Programmer
{
    public CorporateProgrammer(
        ILanguage language)
        : base(language)
    {
    }

    public override void EarnMoney()
    {
        Console.WriteLine(
            "Am Monatsende wird das Gehalt ausgezahlt.");
    }
}
```

---

# Verwendung

```
public class Program
{
    public static void Main()
    {
        // Freelancer arbeitet zunächst mit C++.
        Programmer programmer =
            new FreelanceProgrammer(
                new CppLanguage());

        programmer.DoWork();
        programmer.EarnMoney();

        // Neuer Auftrag benötigt C#.
        programmer.ChangeLanguage(
            new CSharpLanguage());

        programmer.DoWork();
        programmer.EarnMoney();
    }
}
```

---

# Was passiert hier genau?

Zunächst:

```
Programmer programmer =
    new FreelanceProgrammer(
        new CppLanguage());
```

Wir haben:

```
Abstraction:
FreelanceProgrammer

Implementor:
CppLanguage
```

Die Verbindung:

```
FreelanceProgrammer
        │
        ▼
   CppLanguage
```

---

# `DoWork()`

Bei:

```
programmer.DoWork();
```

wird ausgeführt:

```
Language.Build();
Language.Execute();
```

Da `Language` aktuell:

```
CppLanguage
```

ist:

```
Programmer.DoWork()
        │
        ├──► CppLanguage.Build()
        │
        └──► CppLanguage.Execute()
```

---

# `EarnMoney()`

Bei:

```
programmer.EarnMoney();
```

ist entscheidend, welcher konkrete Programmierertyp verwendet wird.

Hier:

```
FreelanceProgrammer
```

also:

```
Bezahlung pro Auftrag
```

---

# Sprache zur Laufzeit wechseln

Danach:

```
programmer.ChangeLanguage(
    new CSharpLanguage());
```

Jetzt bleibt:

```
FreelanceProgrammer
```

unverändert.

Aber die Implementierung wechselt:

```
vorher:

FreelanceProgrammer
        │
        ▼
   CppLanguage


nachher:

FreelanceProgrammer
        │
        ▼
 CSharpLanguage
```

Genau das ist eine zentrale Idee von Bridge.

---

# Zwei unabhängige Dimensionen

Wir haben:

## Dimension 1

```
Art des Programmierers
```

mit:

```
Freelance
Corporate
```

## Dimension 2

```
Programmiersprache
```

mit:

```
C++
C#
```

Mit Bridge können wir frei kombinieren:

```
Freelancer + C++
Freelancer + C#

Corporate + C++
Corporate + C#
```

ohne vier spezialisierte Kombinationsklassen zu benötigen.

---

# Neue Programmiersprache hinzufügen

Angenommen, wir möchten:

```
Java
```

unterstützen.

Wir brauchen lediglich:

```
public sealed class JavaLanguage : ILanguage
{
    public void Build()
    {
        Console.WriteLine(
            "Java-Quellcode wird kompiliert.");
    }

    public void Execute()
    {
        Console.WriteLine(
            "Die Java-Anwendung wird ausgeführt.");
    }
}
```

Danach funktioniert:

```
Programmer freelancer =
    new FreelanceProgrammer(
        new JavaLanguage());

Programmer corporate =
    new CorporateProgrammer(
        new JavaLanguage());
```

Bestehende Programmiererklassen müssen nicht verändert werden.

---

# Neue Abstraktion hinzufügen

Angenommen, wir möchten:

```
PartTimeProgrammer
```

hinzufügen.

```
public sealed class PartTimeProgrammer
    : Programmer
{
    public PartTimeProgrammer(
        ILanguage language)
        : base(language)
    {
    }

    public override void EarnMoney()
    {
        Console.WriteLine(
            "Bezahlung nach geleisteten Stunden.");
    }
}
```

Dieser neue Typ kann automatisch mit allen bestehenden Sprachen kombiniert werden:

```
new PartTimeProgrammer(
    new CSharpLanguage());

new PartTimeProgrammer(
    new CppLanguage());

new PartTimeProgrammer(
    new JavaLanguage());
```

---

# Genau darin liegt die Stärke

Neue Abstraktion:

```
PartTimeProgrammer
```

benötigt keine:

```
PartTimeCSharpProgrammer
PartTimeCppProgrammer
PartTimeJavaProgrammer
```

Klassen.

Und eine neue Implementierung:

```
JavaLanguage
```

benötigt keine:

```
JavaFreelanceProgrammer
JavaCorporateProgrammer
JavaPartTimeProgrammer
```

Klassen.

---

# Ohne Bridge

```
Programmer
├── FreelanceCppProgrammer
├── FreelanceCSharpProgrammer
├── FreelanceJavaProgrammer
├── CorporateCppProgrammer
├── CorporateCSharpProgrammer
├── CorporateJavaProgrammer
├── PartTimeCppProgrammer
├── PartTimeCSharpProgrammer
└── PartTimeJavaProgrammer
```

---

# Mit Bridge

```
Programmer
├── FreelanceProgrammer
├── CorporateProgrammer
└── PartTimeProgrammer


ILanguage
├── CppLanguage
├── CSharpLanguage
└── JavaLanguage
```

Verbindung:

```
Programmer ─────► ILanguage
```

Viel übersichtlicher.

---

# Bridge verwendet Komposition

Das zentrale Prinzip ist:

```
Programmer
HAS-A
ILanguage
```

also:

```
private ILanguage _language;
```

statt:

```
FreelanceProgrammer
IS-A
CppProgrammer
```

Bridge nutzt damit stark:

> **Composition over Inheritance**

---

# Warum reicht einfache Vererbung nicht?

Eine Vererbungshierarchie eignet sich gut für **eine Dimension**.

Zum Beispiel:

```
Programmer
├── Freelancer
└── Corporate
```

Kein Problem.

Aber wenn eine zweite unabhängige Dimension hinzukommt:

```
Programmiersprache
```

wird Vererbung problematisch.

Dann müsste man beide Dimensionen miteinander kreuzen.

Bridge trennt sie.

---

# Typische Erkennungsregel

Wenn du denkst:

> „Dieses Objekt hat zwei voneinander unabhängige Eigenschaften, und für beide gibt es mehrere Varianten.“

dann könnte Bridge sinnvoll sein.

Beispiel:

```
Form:
├── Kreis
├── Rechteck
└── Dreieck

Rendering:
├── Windows
├── Linux
└── Web
```

Ohne Bridge könnten entstehen:

```
WindowsCircle
LinuxCircle
WebCircle

WindowsRectangle
LinuxRectangle
WebRectangle

...
```

Mit Bridge:

```
Shape
├── Circle
├── Rectangle
└── Triangle

Renderer
├── WindowsRenderer
├── LinuxRenderer
└── WebRenderer
```

---

# Klassisches Beispiel: Shape + Renderer

```
public interface IRenderer
{
    void DrawCircle(double radius);
}
```

Renderer:

```
public sealed class VectorRenderer : IRenderer
{
    public void DrawCircle(double radius)
    {
        Console.WriteLine(
            $"Vektorkreis mit Radius {radius} wird gezeichnet.");
    }
}
```

```
public sealed class RasterRenderer : IRenderer
{
    public void DrawCircle(double radius)
    {
        Console.WriteLine(
            $"Rasterkreis mit Radius {radius} wird gezeichnet.");
    }
}
```

---

# Abstraction

```
public abstract class Shape
{
    protected IRenderer Renderer { get; }

    protected Shape(IRenderer renderer)
    {
        Renderer = renderer;
    }

    public abstract void Draw();
}
```

---

# RefinedAbstraction

```
public sealed class Circle : Shape
{
    private readonly double _radius;

    public Circle(
        double radius,
        IRenderer renderer)
        : base(renderer)
    {
        _radius = radius;
    }

    public override void Draw()
    {
        Renderer.DrawCircle(_radius);
    }
}
```

---

# Frei kombinierbar

```
Shape vectorCircle =
    new Circle(
        10,
        new VectorRenderer());

Shape rasterCircle =
    new Circle(
        10,
        new RasterRenderer());
```

Also:

```
Circle
  +
VectorRenderer
```

oder:

```
Circle
  +
RasterRenderer
```

ohne:

```
VectorCircle
RasterCircle
```

erstellen zu müssen.

---

# Zweites praktisches Beispiel: Benachrichtigungen

Angenommen, wir haben unterschiedliche Arten von Nachrichten:

```
Notification
├── NormalNotification
└── UrgentNotification
```

und unterschiedliche Sendekanäle:

```
MessageSender
├── EmailSender
├── SmsSender
└── PushSender
```

Ohne Bridge könnten Klassen entstehen wie:

```
NormalEmailNotification
NormalSmsNotification
NormalPushNotification

UrgentEmailNotification
UrgentSmsNotification
UrgentPushNotification
```

Mit Bridge bleiben beide Dimensionen getrennt.

---

# Implementor

```
public interface IMessageSender
{
    void Send(string message);
}
```

---

# ConcreteImplementor

```
public sealed class EmailSender : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"E-Mail: {message}");
    }
}
```

```
public sealed class SmsSender : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"SMS: {message}");
    }
}
```

---

# Abstraction

```
public abstract class Notification
{
    protected IMessageSender Sender { get; }

    protected Notification(
        IMessageSender sender)
    {
        Sender = sender;
    }

    public abstract void Send(
        string message);
}
```

---

# RefinedAbstraction

```
public sealed class NormalNotification
    : Notification
{
    public NormalNotification(
        IMessageSender sender)
        : base(sender)
    {
    }

    public override void Send(
        string message)
    {
        Sender.Send(message);
    }
}
```

```
public sealed class UrgentNotification
    : Notification
{
    public UrgentNotification(
        IMessageSender sender)
        : base(sender)
    {
    }

    public override void Send(
        string message)
    {
        Sender.Send(
            $"DRINGEND: {message}");
    }
}
```

---

# Kombinationen

```
Notification notification =
    new UrgentNotification(
        new EmailSender());

notification.Send(
    "Server ist nicht erreichbar.");
```

Oder:

```
Notification notification =
    new UrgentNotification(
        new SmsSender());
```

Die Abstraktion:

```
UrgentNotification
```

bleibt dieselbe.

Die Implementierung:

```
EmailSender
↓
SmsSender
```

kann gewechselt werden.

---

# Bridge vs. Strategy

Bridge und Strategy sehen strukturell sehr ähnlich aus.

Beide verwenden:

```
Context / Abstraction
        │
        ▼
     Interface
        ▲
   Implementierungen
```

Der Unterschied liegt in der **Absicht**.

---

## Strategy

Strategy beantwortet:

> **Welchen Algorithmus möchte ich verwenden?**

Beispiel:

```
SortingStrategy
├── QuickSort
├── MergeSort
└── BubbleSort
```

Es geht um austauschbare Algorithmen.

---

## Bridge

Bridge beantwortet:

> **Wie kann ich zwei unabhängige Hierarchien miteinander kombinieren?**

Zum Beispiel:

```
Programmer
×
Language
```

---

# Merksatz

```
Strategy
→ Algorithmus austauschen


Bridge
→ zwei Dimensionen unabhängig entwickeln
```

---

# Bridge vs. Adapter

Diese beiden Patterns werden besonders häufig verwechselt.

Beide verbinden unterschiedliche Klassen.

Der Unterschied liegt aber im Zeitpunkt und Ziel.

---

## Adapter

Adapter wird meistens eingesetzt, wenn bereits:

```
bestehende Klasse
+
bestehender Client
+
inkompatible Interfaces
```

vorhanden sind.

Er repariert eine bestehende Inkompatibilität.

```
Adaptee
   │
Adapter
   │
Target
```

---

## Bridge

Bridge wird bewusst in der Architektur eingeplant, um zwei Dimensionen von Anfang an zu trennen.

```
Abstraction
     │
     ▼
Implementor
```

---

# Kurzvergleich

```
Adapter
→ bestehende Dinge kompatibel machen


Bridge
→ unabhängige Entwicklung ermöglichen
```

Oder:

> **Adapter repariert eine Verbindung.  
> Bridge entwirft die Verbindung bewusst flexibel.**

---

# Bridge vs. Decorator

## Decorator

Decorator erweitert das Verhalten eines Objekts:

```
LoggingDecorator
       ↓
CachingDecorator
       ↓
Service
```

---

## Bridge

Bridge verbindet zwei voneinander unabhängige Hierarchien:

```
Abstraction
     │
     ▼
Implementor
```

Kurz:

```
Decorator
→ Verhalten schichtweise ergänzen


Bridge
→ zwei Dimensionen entkoppeln
```

---

# Bridge vs. Facade

## Facade

Facade bietet:

```
einfachere Schnittstelle
```

zu einem komplexen Subsystem.

## Bridge

Bridge trennt:

```
Abstraktion
```

von:

```
Implementierung
```

damit beide unabhängig verändert werden können.

---

# Bridge vs. Abstract Factory

Diese Patterns können sogar gemeinsam verwendet werden.

Eine Abstract Factory könnte beispielsweise entscheiden, welche konkrete `Implementor`-Variante für eine Bridge erzeugt wird.

Zum Beispiel:

```
Bridge
→ Struktur zwischen zwei Hierarchien

Abstract Factory
→ passende Implementierungen erzeugen
```

Sie lösen also unterschiedliche Probleme.

---

# Bridge vs. Dependency Injection

Bridge ist nicht dasselbe wie Dependency Injection.

Dependency Injection ist eine Technik, mit der eine Abhängigkeit von außen bereitgestellt wird:

```
public Programmer(
    ILanguage language)
```

Bridge kann diese Technik nutzen.

Aber Bridge beschreibt zusätzlich eine Architektur mit **zwei unabhängig variierbaren Hierarchien**.

> [!note]  
> Constructor Injection allein bedeutet noch nicht automatisch, dass das Bridge Pattern vorliegt.

---

# Woran erkenne ich echtes Bridge?

Bridge wird interessant, wenn auf **beiden Seiten mehrere Varianten** existieren.

Zum Beispiel:

```
Abstraction:

Notification
├── NormalNotification
└── UrgentNotification
```

und:

```
Implementor:

MessageSender
├── EmailSender
├── SmsSender
└── PushSender
```

Dann:

```
2 × 3
```

mögliche Kombinationen.

Bridge verhindert dafür spezialisierte Kombinationsklassen.

---

# Wenn nur eine Seite variiert

Angenommen:

```
PaymentService
```

verwendet:

```
IPaymentGateway
├── Stripe
├── PayPal
└── Adyen
```

aber `PaymentService` selbst besitzt keine relevante Abstraktionshierarchie.

Dann kann es sich schlicht um:

```
Dependency Inversion
Strategy
Composition
```

handeln.

Es muss nicht unbedingt Bridge sein.

---

# Zwei parallele Hierarchien sind das Schlüsselmerkmal

```
Abstraction-Hierarchie
        │
        │ Bridge
        ▼
Implementor-Hierarchie
```

Beispiel:

```
       Programmer                   ILanguage
           ▲                            ▲
      ┌────┴────┐                 ┌─────┴─────┐
      │         │                 │           │
 Freelancer Corporate           CSharp       Cpp
```

Verbindungen entstehen erst über Komposition.

---

# Bridge und Open/Closed Principle

Neue Abstraktion:

```
PartTimeProgrammer
```

kann ergänzt werden, ohne bestehende Sprachklassen zu verändern.

Neue Implementierung:

```
JavaLanguage
```

kann ergänzt werden, ohne bestehende Programmiererklassen zu verändern.

Damit können beide Hierarchien unabhängig erweitert werden.

---

# Vorteile

- Abstraktion und Implementierung sind voneinander getrennt;
- beide Hierarchien können unabhängig erweitert werden;
- Klassenexplosion durch Kombinationen wird vermieden;
- Implementierungen können austauschbar sein;
- unterstützt Composition over Inheritance;
- reduziert harte Kopplung;
- kann Open/Closed Principle unterstützen;
- konkrete Implementierungen können zur Laufzeit gewechselt werden.

---

# Nachteile

- zusätzliche Interfaces und Klassen erhöhen zunächst die Komplexität;
- für einfache Systeme kann Bridge unnötig sein;
- zwei getrennte Hierarchien sind schwieriger zu verstehen als eine kleine Vererbungshierarchie;
- falsche Abstraktionsgrenzen können das Design unnötig kompliziert machen.

---

# Wann Bridge nicht sinnvoll ist

Wenn es nur:

```
eine Abstraktion
+
eine Implementierung
```

gibt und keine unabhängigen Varianten erwartet werden, ist Bridge meistens Overengineering.

Zum Beispiel:

```
ReportService
→ PdfWriter
```

wenn weder weitere Reporttypen noch weitere Writer erwartet werden.

Dann reicht häufig einfache Komposition.

---

# Bridge lohnt sich besonders bei

```
mehrere Varianten der Abstraktion
+
mehrere Varianten der Implementierung
+
beide sollen unabhängig wachsen
```

---

# Typische Entscheidungsfrage

```
Habe ich zwei voneinander unabhängige
Variationsdimensionen?
             │
        ┌────┴────┐
       Nein       Ja
        │          │
        ▼          ▼
   eher kein    Würde Vererbung
    Bridge      viele Kombinationen
                erzeugen?
                     │
                ┌────┴────┐
               Nein       Ja
                │          │
                ▼          ▼
           evtl. kein    Bridge
            Bridge       sinnvoll
```

---

# Weitere typische Beispiele

Bridge eignet sich beispielsweise für:

```
Shape × Renderer

Window × OperatingSystem

Notification × Transport

RemoteControl × Device

Report × ExportFormat

Message × DeliveryChannel

Programmer × ProgrammingLanguage
```

---

# Beispiel: RemoteControl und Device

Abstraktion:

```
RemoteControl
├── BasicRemote
└── AdvancedRemote
```

Implementierung:

```
Device
├── Television
└── Radio
```

Ohne Bridge:

```
BasicTvRemote
AdvancedTvRemote
BasicRadioRemote
AdvancedRadioRemote
```

Mit Bridge:

```
RemoteControl
      │
      ▼
    IDevice
```

frei kombinierbar.

---

# Klassische Struktur

```
                      Abstraction
                          ▲
                          │
                  RefinedAbstraction
                          │
                          │ Bridge
                          ▼
                      Implementor
                          ▲
                ┌─────────┴─────────┐
                │                   │
      ConcreteImplementorA ConcreteImplementorB
```

---

# Das solltest du dir merken

Implementor:

```
public interface IImplementor
{
    void Execute();
}
```

Konkrete Implementierung:

```
public sealed class ConcreteImplementor
    : IImplementor
{
    public void Execute()
    {
        // Konkrete technische Implementierung.
    }
}
```

Abstraktion:

```
public abstract class Abstraction
{
    protected IImplementor Implementor { get; }

    protected Abstraction(
        IImplementor implementor)
    {
        Implementor = implementor;
    }

    public virtual void Operation()
    {
        // Die Abstraktion delegiert
        // einen Teil ihrer Arbeit.
        Implementor.Execute();
    }
}
```

RefinedAbstraction:

```
public sealed class RefinedAbstraction
    : Abstraction
{
    public RefinedAbstraction(
        IImplementor implementor)
        : base(implementor)
    {
    }

    public override void Operation()
    {
        // Logik der spezialisierten Abstraktion.

        base.Operation();
    }
}
```

---

# Der entscheidende Code

```
protected IImplementor Implementor { get; }
```

und:

```
Implementor.Execute();
```

Das ist die eigentliche **Brücke**:

```
Abstraction
    │
    │ delegation
    ▼
Implementor
```

---

# Merksatz

> **Bridge trennt eine Abstraktion von ihrer Implementierung, damit beide unabhängig voneinander verändert und erweitert werden können.**

Noch einfacher:

```
Abstraction:
"Was möchte ich tun?"

Implementor:
"Wie wird es technisch umgesetzt?"
```

Oder:

```
Programmer:
"Ich arbeite und verdiene Geld."

Language:
"Ich weiß, wie Code gebaut
und ausgeführt wird."
```

Noch kürzer:

```
Bridge
=
zwei unabhängige Hierarchien
+
Komposition
+
Delegation
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Bridge
→ Abstraktion und Implementierung trennen


Adapter
→ inkompatible Schnittstellen verbinden


Decorator
→ Verhalten erweitern


Facade
→ komplexes Subsystem vereinfachen


Composite
→ Teil-Ganzes-Baumstruktur


Proxy
→ Zugriff kontrollieren
```

---

> [!summary] Zusammenfassung  
> Das **Bridge Pattern** ist ein **Strukturmuster**.
> 
> Es wird verwendet, wenn zwei voneinander unabhängige Dimensionen existieren:
> 
> ```
> Abstraction
> +
> Implementor
> ```
> 
> Beispiel:
> 
> ```
> Programmer
> ├── Freelancer
> └── Corporate
> 
> ILanguage
> ├── CSharp
> └── Cpp
> ```
> 
> Statt für jede Kombination eine eigene Klasse zu erstellen:
> 
> ```
> CSharpFreelancer
> CppFreelancer
> CSharpCorporate
> CppCorporate
> ```
> 
> verbindet Bridge beide Hierarchien durch Komposition:
> 
> ```
> Programmer
>     │
>     ▼
> ILanguage
> ```
> 
> Der entscheidende Mechanismus ist:
> 
> ```
> protected ILanguage Language { get; }
> 
> Language.Build();
> Language.Execute();
> ```
> 
> Dadurch können:
> 
> ```
> neue Programmierertypen
> ```
> 
> und:
> 
> ```
> neue Programmiersprachen
> ```
> 
> unabhängig voneinander ergänzt werden.
> 
> Der wichtigste Unterschied zum Adapter:
> 
> ```
> Adapter
> → bestehende Inkompatibilität lösen
> 
> Bridge
> → zwei Variationsdimensionen bewusst getrennt entwerfen
> ```
> 
> **Kurz gesagt:**  
> `Bridge = Zwei unabhängige Hierarchien durch Komposition verbinden, statt alle Kombinationen durch Vererbung abzubilden.`