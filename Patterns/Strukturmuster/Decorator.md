Das **Decorator Pattern** ist ein Strukturmuster, mit dem einem Objekt **dynamisch zusätzliche Funktionalität hinzugefügt** werden kann, ohne die ursprüngliche Klasse zu verändern.

Dabei wird das ursprüngliche Objekt von einem anderen Objekt **umhüllt**.

> [!info] Grundidee  
> Ein Decorator besitzt dieselbe Schnittstelle wie das Objekt, das er dekoriert.
> 
> Er leitet Aufrufe an das ursprüngliche Objekt weiter und kann davor oder danach zusätzliche Logik ausführen.

Vereinfacht:

```text
Component
    │
    ▼
Decorator
    │
    ▼
weiterer Decorator
```

Oder:

```text
Pizza
  │
  ▼
TomatoDecorator
  │
  ▼
CheeseDecorator
```

---

# Wann sollte man Decorator verwenden?

Das Decorator Pattern eignet sich besonders:

- wenn einem Objekt zur Laufzeit zusätzliche Funktionalität hinzugefügt werden soll;
    
- wenn Funktionalität später wieder entfernt beziehungsweise anders kombiniert werden können soll;
    
- wenn Vererbung zu sehr vielen Klassen führen würde;
    
- wenn unterschiedliche Features frei miteinander kombiniert werden sollen;
    
- wenn eine bestehende Klasse nicht verändert werden soll;
    
- wenn ein Objekt schrittweise erweitert werden soll.
    

---

# Problem mit Vererbung

Angenommen, wir haben:

```text
ItalianPizza
BulgarianPizza
```

und folgende Extras:

```text
Tomaten
Käse
Pilze
Schinken
```

Mit Vererbung könnte man anfangen, Klassen wie diese zu erstellen:

```text
ItalianPizza
ItalianPizzaWithTomatoes
ItalianPizzaWithCheese
ItalianPizzaWithTomatoesAndCheese
ItalianPizzaWithMushrooms
ItalianPizzaWithCheeseAndMushrooms
...
```

Mit jeder zusätzlichen Option steigt die Anzahl möglicher Kombinationen enorm.

---

# Class Explosion

Bei:

```text
2 Pizza-Arten
+
4 mögliche Extras
```

gibt es bereits sehr viele mögliche Kombinationen.

Mit weiteren Optionen:

```text
Tomaten
Käse
Pilze
Oliven
Schinken
Zwiebeln
...
```

wird eine Vererbungshierarchie schnell unübersichtlich.

> [!warning]  
> Genau diese **Explosion von Klassen** ist ein typischer Anwendungsfall für Decorator.

---

# Lösung mit Decorator

Statt für jede Kombination eine eigene Klasse zu erzeugen:

```text
ItalianPizza
    │
    ▼
TomatoDecorator
    │
    ▼
CheeseDecorator
```

Die Funktionalitäten werden dynamisch zusammengesetzt.

---

# Grundstruktur

```text
                  Component
                     ▲
          ┌──────────┴──────────┐
          │                     │
ConcreteComponent           Decorator
                                ▲
                      ┌─────────┴─────────┐
                      │                   │
              ConcreteDecoratorA  ConcreteDecoratorB
```

Der Decorator:

1. implementiert dieselbe Abstraktion wie das dekorierte Objekt;
    
2. enthält gleichzeitig eine Referenz auf ein anderes `Component`-Objekt.
    

---

# Teilnehmer des Patterns

## Component

`Component` definiert die gemeinsame Schnittstelle.

Zum Beispiel:

```csharp
public interface IComponent
{
    void Operation();
}
```

Sowohl das eigentliche Objekt als auch alle Decorator implementieren diese Schnittstelle.

---

## ConcreteComponent

Der `ConcreteComponent` enthält die grundlegende Funktionalität.

```csharp
public class ConcreteComponent : IComponent
{
    public void Operation()
    {
        Console.WriteLine(
            "Grundlegende Operation.");
    }
}
```

---

## Decorator

Der Decorator implementiert ebenfalls `IComponent`, enthält aber zusätzlich eine Referenz auf ein anderes `IComponent`.

```csharp
public abstract class Decorator : IComponent
{
    protected readonly IComponent Component;

    protected Decorator(IComponent component)
    {
        Component = component;
    }

    public virtual void Operation()
    {
        // Standardmäßig wird die Operation
        // an das dekorierte Objekt weitergegeben.
        Component.Operation();
    }
}
```

---

## ConcreteDecorator

Ein konkreter Decorator ergänzt die ursprüngliche Funktionalität.

```csharp
public class ConcreteDecoratorA : Decorator
{
    public ConcreteDecoratorA(IComponent component)
        : base(component)
    {
    }

    public override void Operation()
    {
        // Zuerst Verhalten des dekorierten Objekts.
        base.Operation();

        // Danach zusätzliche Funktionalität.
        Console.WriteLine(
            "Zusätzliche Funktionalität A.");
    }
}
```

---

# Allgemeines Beispiel

```csharp
IComponent component =
    new ConcreteComponent();

component =
    new ConcreteDecoratorA(component);

component.Operation();
```

Die Struktur sieht jetzt so aus:

```text
ConcreteDecoratorA
        │
        ▼
ConcreteComponent
```

---

# Mehrere Decorator kombinieren

```csharp
IComponent component =
    new ConcreteComponent();

component =
    new ConcreteDecoratorA(component);

component =
    new ConcreteDecoratorB(component);
```

Jetzt:

```text
ConcreteDecoratorB
        │
        ▼
ConcreteDecoratorA
        │
        ▼
ConcreteComponent
```

Beim Methodenaufruf läuft die Verarbeitung durch diese gesamte Kette.

---

# Praktisches Beispiel: Pizza

Wir haben verschiedene Grundpizzen:

```text
ItalianPizza
BulgarianPizza
```

und möchten verschiedene Zutaten dynamisch ergänzen:

```text
Tomaten
Käse
```

Jede Zutat:

- verändert den Namen;
    
- erhöht den Preis.
    

---

# Component: Pizza

```csharp
public abstract class Pizza
{
    public string Name { get; protected set; }

    protected Pizza(string name)
    {
        Name = name;
    }

    // Jede Pizza definiert ihren aktuellen Preis.
    public abstract decimal GetCost();
}
```

---

# ConcreteComponent: ItalianPizza

```csharp
public class ItalianPizza : Pizza
{
    public ItalianPizza()
        : base("Italienische Pizza")
    {
    }

    public override decimal GetCost()
    {
        return 10m;
    }
}
```

---

# ConcreteComponent: BulgarianPizza

Im ursprünglichen Beispiel wurde der Name `BulgerianPizza` verwendet.

Sauberer wäre:

```csharp
BulgarianPizza
```

```csharp
public class BulgarianPizza : Pizza
{
    public BulgarianPizza()
        : base("Bulgarische Pizza")
    {
    }

    public override decimal GetCost()
    {
        return 8m;
    }
}
```

---

# Abstrakter PizzaDecorator

```csharp
public abstract class PizzaDecorator : Pizza
{
    // Referenz auf das dekorierte Pizza-Objekt.
    protected readonly Pizza Pizza;

    protected PizzaDecorator(
        Pizza pizza,
        string name)
        : base(name)
    {
        Pizza = pizza;
    }
}
```

Wichtig:

`PizzaDecorator` ist selbst eine:

```text
Pizza
```

und enthält gleichzeitig eine:

```text
Pizza
```

Also:

```text
Vererbung
+
Komposition
```

---

# TomatoDecorator

```csharp
public class TomatoPizza : PizzaDecorator
{
    public TomatoPizza(Pizza pizza)
        : base(
            pizza,
            $"{pizza.Name}, mit Tomaten")
    {
    }

    public override decimal GetCost()
    {
        // Preis der bestehenden Pizza
        // plus Preis der Tomaten.
        return Pizza.GetCost() + 3m;
    }
}
```

---

# CheeseDecorator

```csharp
public class CheesePizza : PizzaDecorator
{
    public CheesePizza(Pizza pizza)
        : base(
            pizza,
            $"{pizza.Name}, mit Käse")
    {
    }

    public override decimal GetCost()
    {
        // Preis der dekorierten Pizza
        // plus Preis des Käses.
        return Pizza.GetCost() + 5m;
    }
}
```

---

# Verwendung

## Italienische Pizza mit Tomaten

```csharp
Pizza pizza1 =
    new ItalianPizza();

pizza1 =
    new TomatoPizza(pizza1);

Console.WriteLine(
    pizza1.Name);

Console.WriteLine(
    pizza1.GetCost());
```

Ergebnis:

```text
Italienische Pizza, mit Tomaten
13
```

---

# Was passiert intern?

Zunächst:

```csharp
new ItalianPizza();
```

```text
ItalianPizza
Preis = 10
```

Dann:

```csharp
new TomatoPizza(pizza);
```

Struktur:

```text
TomatoPizza
    │
    ▼
ItalianPizza
```

Beim:

```csharp
pizza.GetCost();
```

ruft `TomatoPizza` auf:

```csharp
Pizza.GetCost() + 3m;
```

`Pizza` ist hier die `ItalianPizza`.

Also:

```text
10 + 3 = 13
```

---

# Italienische Pizza mit Käse

```csharp
Pizza pizza2 =
    new ItalianPizza();

pizza2 =
    new CheesePizza(pizza2);
```

Struktur:

```text
CheesePizza
    │
    ▼
ItalianPizza
```

Preis:

```text
10 + 5 = 15
```

---

# Bulgarische Pizza mit Tomaten und Käse

```csharp
Pizza pizza3 =
    new BulgarianPizza();

pizza3 =
    new TomatoPizza(pizza3);

pizza3 =
    new CheesePizza(pizza3);
```

Jetzt entsteht:

```text
CheesePizza
     │
     ▼
TomatoPizza
     │
     ▼
BulgarianPizza
```

---

# Preisberechnung Schritt für Schritt

Grundpreis:

```text
BulgarianPizza
= 8
```

Tomaten:

```text
TomatoPizza.GetCost()

= BulgarianPizza.GetCost()
+ 3

= 8 + 3
= 11
```

Dann Käse:

```text
CheesePizza.GetCost()

= TomatoPizza.GetCost()
+ 5

= 11 + 5
= 16
```

Ergebnis:

```text
Bulgarische Pizza, mit Tomaten, mit Käse

Preis = 16
```

---

# Warum funktioniert die Verschachtelung?

Jeder Decorator ist selbst eine `Pizza`.

Deshalb kann:

```csharp
new TomatoPizza(...)
```

nicht nur eine `ItalianPizza` erhalten, sondern auch einen anderen Decorator.

Zum Beispiel:

```csharp
new CheesePizza(
    new TomatoPizza(
        new BulgarianPizza()));
```

Typisch für Decorator:

```text
Decorator
   enthält
      │
      ▼
Component

und ist selbst wieder
      │
      ▼
Component
```

Dadurch kann beliebig verschachtelt werden.

---

# Eine kompakte Schreibweise

Man könnte auch direkt schreiben:

```csharp
Pizza pizza =
    new CheesePizza(
        new TomatoPizza(
            new BulgarianPizza()));
```

Das bedeutet:

```text
Cheese
  │
Tomato
  │
BulgarianPizza
```

---

# Reihenfolge kann wichtig sein

Angenommen, Decorator verändern nicht nur Kosten, sondern führen Logik vor oder nach einem Methodenaufruf aus.

Dann macht:

```text
A(B(Component))
```

nicht zwingend dasselbe wie:

```text
B(A(Component))
```

Beispiel:

```text
Logging
→ Caching
→ Service
```

kann ein anderes Verhalten besitzen als:

```text
Caching
→ Logging
→ Service
```

> [!important]  
> Bei Decorator-Ketten kann die **Reihenfolge der Decorator** relevant sein.

---

# Decorator statt Vererbung

Mit Vererbung:

```text
Pizza
├── ItalianPizza
├── ItalianPizzaWithCheese
├── ItalianPizzaWithTomatoes
├── ItalianPizzaWithTomatoesAndCheese
├── BulgarianPizza
├── BulgarianPizzaWithCheese
├── BulgarianPizzaWithTomatoes
└── BulgarianPizzaWithTomatoesAndCheese
```

Mit Decorator:

```text
Pizza
├── ItalianPizza
└── BulgarianPizza

Decorator
├── TomatoPizza
└── CheesePizza
```

Die Kombination erfolgt erst zur Laufzeit.

---

# Decorator und Komposition

Decorator ist ein gutes Beispiel für:

> **Composition over Inheritance**

statt immer neue Unterklassen zu erstellen, werden Objekte miteinander kombiniert.

```text
Decorator
    HAS-A Component

und gleichzeitig:

Decorator
    IS-A Component
```

---

# IS-A und HAS-A gleichzeitig

Ein Decorator:

```csharp
public abstract class PizzaDecorator : Pizza
```

ist eine Pizza:

```text
IS-A Pizza
```

und:

```csharp
protected readonly Pizza Pizza;
```

enthält eine andere Pizza:

```text
HAS-A Pizza
```

Genau diese Kombination ist charakteristisch für Decorator.

---

# Moderne Variante mit Interface

In C# muss `Component` nicht unbedingt eine abstrakte Klasse sein.

Oft eignet sich ein Interface sehr gut:

```csharp
public interface INotifier
{
    void Send(string message);
}
```

Concrete Component:

```csharp
public class EmailNotifier : INotifier
{
    public void Send(string message)
    {
        Console.WriteLine(
            $"E-Mail: {message}");
    }
}
```

---

# Decorator

```csharp
public abstract class NotifierDecorator
    : INotifier
{
    protected readonly INotifier Notifier;

    protected NotifierDecorator(
        INotifier notifier)
    {
        Notifier = notifier;
    }

    public virtual void Send(string message)
    {
        Notifier.Send(message);
    }
}
```

---

# SMS Decorator

```csharp
public class SmsNotifierDecorator
    : NotifierDecorator
{
    public SmsNotifierDecorator(
        INotifier notifier)
        : base(notifier)
    {
    }

    public override void Send(string message)
    {
        // Zuerst ursprüngliche Benachrichtigung.
        base.Send(message);

        // Danach zusätzliche SMS.
        Console.WriteLine(
            $"SMS: {message}");
    }
}
```

---

# Push Decorator

```csharp
public class PushNotifierDecorator
    : NotifierDecorator
{
    public PushNotifierDecorator(
        INotifier notifier)
        : base(notifier)
    {
    }

    public override void Send(string message)
    {
        base.Send(message);

        Console.WriteLine(
            $"Push: {message}");
    }
}
```

---

# Verwendung

```csharp
INotifier notifier =
    new EmailNotifier();

notifier =
    new SmsNotifierDecorator(notifier);

notifier =
    new PushNotifierDecorator(notifier);

notifier.Send(
    "Ihre Bestellung wurde versendet.");
```

Ausgabe:

```text
E-Mail: Ihre Bestellung wurde versendet.
SMS: Ihre Bestellung wurde versendet.
Push: Ihre Bestellung wurde versendet.
```

Struktur:

```text
PushDecorator
      │
      ▼
SmsDecorator
      │
      ▼
EmailNotifier
```

---

# Typische reale Anwendungsfälle

Decorator eignet sich beispielsweise für:

- Logging;
    
- Caching;
    
- Retry;
    
- Monitoring;
    
- Kompression;
    
- Verschlüsselung;
    
- Authentifizierung;
    
- Autorisierung;
    
- Benachrichtigungen;
    
- Streams;
    
- HTTP-Handler;
    
- Service-Erweiterungen.
    

---

# Beispiel aus .NET: Streams

Das Decorator-Prinzip findet man sehr häufig bei Streams.

Beispielhaft:

```text
FileStream
    │
    ▼
BufferedStream
    │
    ▼
CryptoStream
```

Jeder äußere Stream erweitert das Verhalten des inneren Streams.

Das entspricht sehr stark der Decorator-Idee:

```text
Basisfunktion
+
zusätzliche Funktion
+
weitere Funktion
```

---

# Beispiel aus ASP.NET / .NET Services

Angenommen:

```csharp
public interface IProductService
{
    Product GetProduct(int id);
}
```

Basisimplementierung:

```csharp
public class ProductService : IProductService
{
    public Product GetProduct(int id)
    {
        // Produkt laden.
        return new Product();
    }
}
```

---

# Logging Decorator

```csharp
public class LoggingProductService
    : IProductService
{
    private readonly IProductService _inner;

    public LoggingProductService(
        IProductService inner)
    {
        _inner = inner;
    }

    public Product GetProduct(int id)
    {
        Console.WriteLine(
            $"Produkt {id} wird geladen.");

        Product product =
            _inner.GetProduct(id);

        Console.WriteLine(
            $"Produkt {id} wurde geladen.");

        return product;
    }
}
```

---

# Caching Decorator

```csharp
public class CachingProductService
    : IProductService
{
    private readonly IProductService _inner;

    private readonly Dictionary<int, Product> _cache =
        new();

    public CachingProductService(
        IProductService inner)
    {
        _inner = inner;
    }

    public Product GetProduct(int id)
    {
        if (_cache.TryGetValue(
                id,
                out Product? product))
        {
            return product;
        }

        product =
            _inner.GetProduct(id);

        _cache[id] = product;

        return product;
    }
}
```

---

# Kombination

```csharp
IProductService service =
    new ProductService();

service =
    new LoggingProductService(service);

service =
    new CachingProductService(service);
```

Struktur:

```text
CachingProductService
        │
        ▼
LoggingProductService
        │
        ▼
ProductService
```

---

# Vorteil für Clean Architecture

Das ist insbesondere für Service-Schichten sehr interessant.

Die Kernklasse:

```text
ProductService
```

muss nichts über:

```text
Logging
Caching
Retry
Metrics
```

wissen.

Diese technischen Querschnittsfunktionen können außen herum ergänzt werden.

Das reduziert Verantwortlichkeiten.

---

# Decorator und Single Responsibility Principle

Ohne Decorator:

```csharp
public class ProductService
{
    // Geschäftslogik
    // Logging
    // Caching
    // Retry
    // Metrics
}
```

Die Klasse erledigt sehr viele Dinge.

Mit Decorator:

```text
ProductService
→ Geschäftslogik

LoggingDecorator
→ Logging

CachingDecorator
→ Caching

RetryDecorator
→ Retry
```

Jede Klasse hat eine klarere Verantwortung.

---

# Decorator und Open/Closed Principle

Bestehende Klasse:

```csharp
ProductService
```

muss nicht verändert werden, wenn später:

```text
Logging
Caching
Monitoring
Retry
```

hinzukommt.

Wir fügen neue Decorator hinzu.

Damit unterstützt Decorator das:

**Open/Closed Principle**

> Offen für Erweiterung, möglichst geschlossen für Veränderung.

---

# Decorator vs. Vererbung

## Vererbung

Funktionalität wird statisch über die Klassenhierarchie festgelegt:

```text
BaseClass
   ▲
   │
Subclass
```

Die Kombination steht normalerweise bereits beim Kompilieren fest.

---

## Decorator

Funktionalität wird durch Objektkomposition zusammengesetzt:

```text
Decorator
   │
   ▼
Decorator
   │
   ▼
Component
```

Die Kombination kann zur Laufzeit erfolgen.

---

# Vergleich

|Vererbung|Decorator|
|---|---|
|statisch|dynamisch|
|Klassenkombination|Objektkombination|
|kann viele Unterklassen erzeugen|wenige kombinierbare Decorator|
|schwerer flexibel zu kombinieren|Decorator frei kombinierbar|
|Verhalten durch Basisklasse/Unterklasse|Verhalten durch Wrapping|

---

# Decorator vs. Adapter

Beide Patterns umhüllen ein Objekt, aber mit unterschiedlichem Ziel.

## Decorator

Behält dieselbe Schnittstelle:

```text
IComponent
    │
    ▼
Decorator : IComponent
```

Ziel:

> Funktionalität erweitern.

---

## Adapter

Ändert die Schnittstelle:

```text
ExistingInterface
      │
      ▼
    Adapter
      │
      ▼
TargetInterface
```

Ziel:

> inkompatible Schnittstellen kompatibel machen.

---

# Merksatz

```text
Decorator
→ gleiche Schnittstelle
→ mehr Verhalten

Adapter
→ andere Schnittstelle
→ Kompatibilität
```

---

# Decorator vs. Proxy

Decorator und Proxy sehen im Code sehr ähnlich aus.

Beide besitzen typischerweise:

```text
Interface
   │
   ├── RealObject
   └── Wrapper
          │
          ▼
       RealObject
```

Der Unterschied liegt hauptsächlich in der Absicht.

---

## Decorator

> Fügt Verhalten hinzu.

Zum Beispiel:

```text
Service
→ Logging
→ Caching
```

---

## Proxy

> Kontrolliert den Zugriff auf das Objekt.

Zum Beispiel:

```text
SecurityProxy
LazyLoadingProxy
RemoteProxy
```

---

# Decorator vs. Proxy

```text
Decorator
→ Objekt erweitern

Proxy
→ Zugriff auf Objekt kontrollieren
```

---

# Decorator vs. Chain of Responsibility

Auch hier gibt es eine Kette von Objekten.

## Decorator

Jeder Decorator erweitert normalerweise denselben Vorgang:

```text
Logging
  ↓
Caching
  ↓
Service
```

Der gesamte Stack bildet gemeinsam ein Objektverhalten.

---

## Chain of Responsibility

Eine Anfrage wandert durch mögliche Handler:

```text
HandlerA
  ↓
HandlerB
  ↓
HandlerC
```

Ein Handler kann die Anfrage bearbeiten oder weitergeben.

---

# Unterschied

```text
Decorator
→ Verhalten schichtweise erweitern

Chain of Responsibility
→ Anfrage entlang mehrerer Handler weitergeben
```

---

# Decorator vs. Composite

Beide verwenden rekursive Komposition.

## Decorator

Typischerweise:

```text
Decorator
   │
   ▼
genau ein Component
```

---

## Composite

Typischerweise:

```text
Composite
├── Component
├── Component
└── Component
```

Also:

```text
Decorator
→ 1 Kind / Wrapper

Composite
→ mehrere Kinder / Baum
```

---

# Vorteile

- Funktionalität kann dynamisch hinzugefügt werden;
    
- Decorator können beliebig kombiniert werden;
    
- bestehende Klassen müssen nicht verändert werden;
    
- reduziert die Notwendigkeit großer Vererbungshierarchien;
    
- vermeidet viele Kombinationsklassen;
    
- unterstützt Composition over Inheritance;
    
- unterstützt häufig SRP und OCP;
    
- einzelne Features lassen sich separat testen.
    

---

# Nachteile

- viele kleine Decorator-Klassen können entstehen;
    
- tief verschachtelte Decorator-Ketten können schwer nachvollziehbar sein;
    
- Debugging kann schwieriger werden;
    
- Reihenfolge der Decorator kann das Verhalten verändern;
    
- der tatsächliche Laufzeittyp ist eventuell schwieriger zu erkennen;
    
- für sehr einfache Erweiterungen kann das Pattern unnötig sein.
    

---

# Tief verschachtelte Decorator

Zum Beispiel:

```csharp
IService service =
    new RetryDecorator(
        new LoggingDecorator(
            new CachingDecorator(
                new MetricsDecorator(
                    new Service()))));
```

Das funktioniert, kann aber irgendwann schwer lesbar werden.

Dann helfen oft:

```text
Dependency Injection
Factory
Extension Methods
Konfiguration
```

beim Zusammenbauen der Decorator-Kette.

---

# Wann Decorator nicht sinnvoll ist

Wenn eine Funktionalität:

```text
immer vorhanden
+
untrennbarer Bestandteil der Klasse
```

ist, muss sie nicht unbedingt als Decorator ausgelagert werden.

Beispiel:

```text
Car
→ Drive()
```

`Drive()` ist zentrale Funktionalität eines Autos.

Dafür wäre ein:

```text
DrivingDecorator
```

meist unsinnig.

Decorator lohnt sich besonders für **optionale und kombinierbare Erweiterungen**.

---

# Typische Entscheidung

```text
Muss Funktionalität optional sein?
        │
        ▼
       Ja
        │
        ▼
Soll sie mit anderen Funktionen
kombiniert werden können?
        │
        ▼
       Ja
        │
        ▼
Decorator kann sinnvoll sein
```

---

# Klassische Struktur

```text
                    Component
                        ▲
               ┌────────┴────────┐
               │                 │
      ConcreteComponent       Decorator
                                   ▲
                           ┌───────┴───────┐
                           │               │
                    DecoratorA        DecoratorB
```

Der Decorator enthält:

```text
Decorator
    │
    └── Component
```

und ist selbst:

```text
Decorator : Component
```

---

# Wichtigste Idee als Diagramm

```text
Client
  │
  ▼
Decorator C
  │
  ▼
Decorator B
  │
  ▼
Decorator A
  │
  ▼
ConcreteComponent
```

Beim Aufruf:

```text
Operation()
```

läuft die Verarbeitung durch die gesamte Kette.

---

# Das solltest du dir merken

Component:

```csharp
public interface IComponent
{
    void Operation();
}
```

ConcreteComponent:

```csharp
public class ConcreteComponent : IComponent
{
    public void Operation()
    {
        // Basisfunktionalität.
    }
}
```

Decorator:

```csharp
public abstract class Decorator : IComponent
{
    protected readonly IComponent Inner;

    protected Decorator(IComponent inner)
    {
        Inner = inner;
    }

    public virtual void Operation()
    {
        Inner.Operation();
    }
}
```

ConcreteDecorator:

```csharp
public class LoggingDecorator : Decorator
{
    public LoggingDecorator(IComponent inner)
        : base(inner)
    {
    }

    public override void Operation()
    {
        // Verhalten vor dem Aufruf.
        Console.WriteLine("Vorher");

        base.Operation();

        // Verhalten nach dem Aufruf.
        Console.WriteLine("Nachher");
    }
}
```

---

# Merksatz

> **Decorator erweitert ein Objekt dynamisch, indem es mit einem anderen Objekt derselben Schnittstelle umhüllt wird.**

Noch einfacher:

```text
Objekt
  +
Wrapper
  +
Wrapper
  +
Wrapper
```

Oder:

```text
ConcreteComponent
→ Grundfunktion

Decorator
→ zusätzliche Funktion
```

---

# Kurzvergleich wichtiger Strukturmuster

```text
Decorator
→ Verhalten erweitern

Adapter
→ Schnittstelle anpassen

Proxy
→ Zugriff kontrollieren

Composite
→ Baumstruktur aufbauen

Facade
→ komplexes Subsystem vereinfachen
```

---

> [!summary] Zusammenfassung  
> Das **Decorator Pattern** ist ein **Strukturmuster**.
> 
> Es erweitert ein bestehendes Objekt dynamisch mit zusätzlichen Funktionen.
> 
> Der Decorator:
> 
> ```text
> ist selbst ein Component
> +
> enthält ein Component
> ```
> 
> Dadurch können mehrere Decorator verschachtelt werden:
> 
> ```text
> CheeseDecorator
>       ↓
> TomatoDecorator
>       ↓
> BulgarianPizza
> ```
> 
> Für das Pizza-Beispiel bedeutet das:
> 
> ```csharp
> Pizza pizza =
>     new CheesePizza(
>         new TomatoPizza(
>             new BulgarianPizza()));
> ```
> 
> Der große Vorteil gegenüber Vererbung besteht darin, dass nicht für jede Kombination eine eigene Klasse benötigt wird.
> 
> Decorator eignet sich besonders gut für optionale Querschnittsfunktionen wie:
> 
> ```text
> Logging
> Caching
> Retry
> Monitoring
> Verschlüsselung
> Kompression
> ```
> 
> **Kurz gesagt:**  
> `Decorator = Objekt umhüllen und dabei sein Verhalten erweitern.`