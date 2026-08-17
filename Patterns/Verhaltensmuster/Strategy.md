Das **Strategy Pattern** ist ein Verhaltensmuster, bei dem verschiedene Algorithmen oder Verhaltensweisen in **separate Klassen ausgelagert** werden.

Diese Strategien verwenden einen gemeinsamen Vertrag und können dadurch **untereinander ausgetauscht werden**.

> [!info] Grundidee
> Ein Objekt besitzt ein bestimmtes Verhalten, implementiert dieses Verhalten aber **nicht selbst**.
>
> Stattdessen bekommt es eine Strategie, die dieses Verhalten übernimmt.

Vereinfacht:

```text
Context
   │
   │ verwendet
   ▼
IStrategy
   ▲
   │
┌──┴──────────────┐
│                 │
StrategyA      StrategyB
```

---

## Wann sollte man Strategy verwenden?

Das Strategy Pattern eignet sich besonders:

- wenn mehrere unterschiedliche Varianten eines Algorithmus existieren;
- wenn ein Verhalten während der Laufzeit ausgetauscht werden soll;
- wenn mehrere Klassen sich hauptsächlich durch ein bestimmtes Verhalten unterscheiden;
- wenn große `if`-/`switch`-Konstruktionen vermieden werden sollen;
- wenn der Client die konkrete Implementierung eines Algorithmus nicht kennen soll;
- wenn Verhalten und Geschäftslogik voneinander getrennt werden sollen.

---

## Grundstruktur

Das klassische Strategy Pattern besteht aus:

- **Strategy**
- **ConcreteStrategy**
- **Context**

```text
                    IStrategy
                        ▲
             ┌──────────┴──────────┐
             │                     │
   ConcreteStrategyA     ConcreteStrategyB
             ▲                     ▲
             └──────────┬──────────┘
                        │
                     Context
```

---

## Strategy

Die Strategy definiert den gemeinsamen Vertrag für alle Algorithmen.

```csharp
// Gemeinsamer Vertrag für alle Strategien.
public interface IStrategy
{
    void Execute();
}
```

Der `Context` interessiert sich nicht dafür, **wie** der Algorithmus funktioniert.

Er weiß nur:

```csharp
strategy.Execute();
```

---

## ConcreteStrategy

Jede konkrete Strategie implementiert den Algorithmus auf ihre eigene Weise.

```csharp
// Erste konkrete Strategie.
public class ConcreteStrategyA : IStrategy
{
    public void Execute()
    {
        Console.WriteLine("Strategie A wird ausgeführt.");
    }
}
```

```csharp
// Zweite konkrete Strategie.
public class ConcreteStrategyB : IStrategy
{
    public void Execute()
    {
        Console.WriteLine("Strategie B wird ausgeführt.");
    }
}
```

---

## Context

Der `Context` besitzt eine Strategie und delegiert die eigentliche Arbeit an sie.

```csharp
public class Context
{
    // Aktuell verwendete Strategie.
    private IStrategy _strategy;

    public Context(IStrategy strategy)
    {
        _strategy = strategy;
    }

    // Die Strategie kann während der Laufzeit
    // durch eine andere ersetzt werden.
    public void SetStrategy(IStrategy strategy)
    {
        _strategy = strategy;
    }

    // Der Context führt den Algorithmus nicht selbst aus,
    // sondern delegiert ihn an die Strategie.
    public void Execute()
    {
        _strategy.Execute();
    }
}
```

---

## Verwendung

```csharp
Context context =
    new Context(new ConcreteStrategyA());

context.Execute();

// Verhalten während der Laufzeit ändern.
context.SetStrategy(
    new ConcreteStrategyB());

context.Execute();
```

Ausgabe:

```text
Strategie A wird ausgeführt.
Strategie B wird ausgeführt.
```

---

# Praktisches Beispiel: Auto

Angenommen, ein Auto kann unterschiedliche Antriebsarten verwenden:

- Benzin
- Elektrizität
- Gas
- Wasserstoff

Das Auto selbst bleibt gleich.

Nur die Art, **wie es sich fortbewegt**, verändert sich.

---

## Strategy: `IMovementStrategy`

```csharp
// Gemeinsamer Vertrag für alle Antriebsstrategien.
public interface IMovementStrategy
{
    // Jede Strategie definiert selbst,
    // wie sich das Fahrzeug bewegt.
    void Move();
}
```

---

## Benzin-Strategie

```csharp
// Bewegung mit einem Benzinmotor.
public class PetrolMovement : IMovementStrategy
{
    public void Move()
    {
        Console.WriteLine(
            "Das Auto fährt mit Benzin.");
    }
}
```

---

## Elektro-Strategie

```csharp
// Bewegung mit einem Elektromotor.
public class ElectricMovement : IMovementStrategy
{
    public void Move()
    {
        Console.WriteLine(
            "Das Auto fährt elektrisch.");
    }
}
```

---

## Wasserstoff-Strategie

```csharp
// Bewegung mit einem Wasserstoffantrieb.
public class HydrogenMovement : IMovementStrategy
{
    public void Move()
    {
        Console.WriteLine(
            "Das Auto fährt mit Wasserstoff.");
    }
}
```

---

## Context: `Car`

```csharp
public class Car
{
    public int PassengerCount { get; }

    public string Model { get; }

    // Das konkrete Bewegungsverhalten
    // wird nicht direkt von Car implementiert.
    private IMovementStrategy _movementStrategy;

    public Car(
        int passengerCount,
        string model,
        IMovementStrategy movementStrategy)
    {
        PassengerCount = passengerCount;
        Model = model;
        _movementStrategy = movementStrategy;
    }

    // Das Auto delegiert seine Bewegung
    // an die aktuell verwendete Strategie.
    public void Move()
    {
        _movementStrategy.Move();
    }

    // Die Strategie kann zur Laufzeit
    // ausgetauscht werden.
    public void SetMovementStrategy(
        IMovementStrategy movementStrategy)
    {
        _movementStrategy = movementStrategy;
    }
}
```

---

## Verwendung

```csharp
public class Program
{
    public static void Main()
    {
        // Das Auto verwendet zunächst
        // einen Benzinantrieb.
        Car car = new(
            passengerCount: 4,
            model: "Volvo",
            movementStrategy: new PetrolMovement());

        car.Move();

        // Strategie zur Laufzeit austauschen.
        car.SetMovementStrategy(
            new ElectricMovement());

        car.Move();

        // Eine weitere Strategie verwenden.
        car.SetMovementStrategy(
            new HydrogenMovement());

        car.Move();
    }
}
```

Ausgabe:

```text
Das Auto fährt mit Benzin.
Das Auto fährt elektrisch.
Das Auto fährt mit Wasserstoff.
```

---

## Was ist hier was?

| Rolle | Klasse |
|---|---|
| Strategy | `IMovementStrategy` |
| ConcreteStrategy | `PetrolMovement` |
| ConcreteStrategy | `ElectricMovement` |
| ConcreteStrategy | `HydrogenMovement` |
| Context | `Car` |

---

## Ablauf

Zuerst:

```text
Car
 │
 ▼
PetrolMovement
 │
 ▼
Benzin
```

Dann:

```csharp
car.SetMovementStrategy(
    new ElectricMovement());
```

Jetzt:

```text
Car
 │
 ▼
ElectricMovement
 │
 ▼
Elektrizität
```

Der `Car` selbst muss nicht verändert werden.

---

# Warum Strategy statt `if` / `switch`?

Ohne Strategy könnte der Code so aussehen:

```csharp
public void Move(string engineType)
{
    if (engineType == "Petrol")
    {
        Console.WriteLine("Das Auto fährt mit Benzin.");
    }
    else if (engineType == "Electric")
    {
        Console.WriteLine("Das Auto fährt elektrisch.");
    }
    else if (engineType == "Hydrogen")
    {
        Console.WriteLine("Das Auto fährt mit Wasserstoff.");
    }
}
```

Bei neuen Varianten wächst diese Methode immer weiter:

```text
Benzin
Elektrisch
Wasserstoff
Gas
Diesel
Hybrid
...
```

Mit Strategy bekommt jede Variante eine eigene Klasse:

```text
IMovementStrategy
├── PetrolMovement
├── ElectricMovement
├── HydrogenMovement
├── GasMovement
└── DieselMovement
```

---

# Zusammenhang mit SOLID

## Open/Closed Principle

Neue Strategien können ergänzt werden, ohne den `Car` verändern zu müssen.

Zum Beispiel:

```csharp
public class GasMovement : IMovementStrategy
{
    public void Move()
    {
        Console.WriteLine(
            "Das Auto fährt mit Gas.");
    }
}
```

`Car` bleibt unverändert.

> [!tip]
> **Open for Extension, Closed for Modification**
>
> Neue Funktionalität hinzufügen, ohne bestehenden Code unnötig zu verändern.

---

## Dependency Inversion Principle

`Car` hängt nicht direkt von:

```text
PetrolMovement
ElectricMovement
HydrogenMovement
```

ab.

Sondern von der Abstraktion:

```csharp
IMovementStrategy
```

```text
Car
 │
 ▼
IMovementStrategy
 ▲
 │
 ├── PetrolMovement
 ├── ElectricMovement
 └── HydrogenMovement
```

---

# Composition over Inheritance

Strategy ist ein gutes Beispiel für:

> **Composition over Inheritance**

Statt:

```text
Car
├── PetrolCar
├── ElectricCar
├── GasCar
└── HydrogenCar
```

verwenden wir:

```text
Car
 │
 ▼
IMovementStrategy
```

Das Verhalten wird dem Objekt **gegeben**, statt über Vererbung fest eingebaut zu werden.

---

# Strategy mit Dependency Injection

In modernen .NET-Anwendungen werden Strategien häufig über **Dependency Injection** übergeben.

```csharp
public interface IPaymentStrategy
{
    void Pay(decimal amount);
}
```

```csharp
public class PaymentService
{
    private readonly IPaymentStrategy _paymentStrategy;

    public PaymentService(
        IPaymentStrategy paymentStrategy)
    {
        _paymentStrategy = paymentStrategy;
    }

    public void Pay(decimal amount)
    {
        _paymentStrategy.Pay(amount);
    }
}
```

`PaymentService` kennt nur:

```csharp
IPaymentStrategy
```

und nicht die konkrete Implementierung.

---

# Strategy mit Delegates und Lambdas

Für sehr kleine Strategien braucht man nicht immer eine eigene Klasse.

C# erlaubt auch Delegates und Lambdas:

```csharp
public class Calculator
{
    private Func<int, int, int> _strategy;

    public Calculator(
        Func<int, int, int> strategy)
    {
        _strategy = strategy;
    }

    public void SetStrategy(
        Func<int, int, int> strategy)
    {
        _strategy = strategy;
    }

    public int Calculate(int a, int b)
    {
        return _strategy(a, b);
    }
}
```

Verwendung:

```csharp
Calculator calculator =
    new((a, b) => a + b);

Console.WriteLine(
    calculator.Calculate(10, 5));

// Strategie wechseln.
calculator.SetStrategy(
    (a, b) => a * b);

Console.WriteLine(
    calculator.Calculate(10, 5));
```

Ausgabe:

```text
15
50
```

> [!note]
> Für kleine Strategien können Lambdas sehr praktisch sein.
>
> Bei komplexeren Algorithmen sind eigene Strategy-Klassen meistens übersichtlicher.

---

# Vorteile

- Algorithmen werden voneinander getrennt.
- Verhalten kann zur Laufzeit ausgetauscht werden.
- Große `if`-/`switch`-Blöcke können vermieden werden.
- Neue Strategien lassen sich einfach ergänzen.
- Strategien können separat getestet werden.
- Der Context kennt keine Implementierungsdetails.
- Unterstützt das Open/Closed Principle.
- Unterstützt Dependency Inversion.
- Fördert Komposition statt Vererbung.

---

# Nachteile

- Für jede größere Strategie entsteht häufig eine zusätzliche Klasse.
- Bei sehr einfachen Fällen kann das Pattern unnötig komplex sein.
- Der Client muss wissen, welche Strategie verwendet werden soll.
- Sehr viele Strategien können zu vielen kleinen Klassen führen.

---

# Strategy vs. State

Strategy und State sehen strukturell ähnlich aus.

## Strategy

Der Algorithmus wird ausgewählt:

```text
Sortierung
├── QuickSort
├── MergeSort
└── BubbleSort
```

> **Strategy → Welcher Algorithmus soll verwendet werden?**

---

## State

Das Verhalten hängt vom internen Zustand ab:

```text
Order
├── New
├── Paid
├── Shipped
└── Completed
```

> **State → Wie verhält sich das Objekt in seinem aktuellen Zustand?**

---

# Strategy vs. Template Method

## Template Method

Verwendet hauptsächlich **Vererbung**:

```text
BaseClass
   ▲
   │
Subclass
```

## Strategy

Verwendet **Komposition**:

```text
Context
   │
   ▼
Strategy
```

Merksatz:

```text
Template Method
→ Verhalten durch Vererbung variieren

Strategy
→ Verhalten durch Komposition variieren
```

---

# Merksatz

> **Strategy kapselt unterschiedliche Algorithmen in separate Klassen und macht sie austauschbar.**

Noch einfacher:

```text
Context:
"Ich weiß, WAS gemacht werden soll."

Strategy:
"Ich weiß, WIE es gemacht wird."
```

Oder:

```text
Strategy
=
gleiches Ziel
+
unterschiedliche Algorithmen
```

Beispiel:

```text
Ziel:
Auto bewegen

Strategien:
├── Benzin
├── Strom
└── Wasserstoff
```

> [!summary] Zusammenfassung
> Das **Strategy Pattern** lagert verschiedene Algorithmen oder Verhaltensweisen in separate Klassen aus.
>
> Alle Strategien implementieren einen gemeinsamen Vertrag, zum Beispiel `IStrategy`.
>
> Der **Context** arbeitet nur mit diesem Vertrag und delegiert die eigentliche Arbeit an die aktuell verwendete Strategie.
>
> Dadurch lässt sich das Verhalten eines Objekts auch **während der Laufzeit austauschen**.
>
> **Kurz gesagt:**
>
> `Strategy = Verhalten bzw. Algorithmus austauschbar machen.`