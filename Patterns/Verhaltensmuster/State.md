Das **State Pattern** ist ein Verhaltensmuster, bei dem ein Objekt sein Verhalten abhängig von seinem **aktuellen internen Zustand** verändert.

Anstatt viele `if`, `else if` oder `switch`-Abfragen zu verwenden, wird das zustandsabhängige Verhalten in **separate State-Klassen** ausgelagert.

> [!info] Grundidee  
> Dasselbe Objekt kann auf dieselbe Aktion unterschiedlich reagieren – abhängig davon, in welchem Zustand es sich gerade befindet.

Beispiel:

```text
Wasser
│
├── Fest
├── Flüssig
└── Gasförmig
```

Die Methode:

```csharp
water.Heat();
```

führt abhängig vom aktuellen Zustand zu einem anderen Ergebnis.

---

# Wann sollte man State verwenden?

Das State Pattern eignet sich besonders:

- wenn das Verhalten eines Objekts von seinem aktuellen Zustand abhängt;
    
- wenn sich dieser Zustand während der Laufzeit verändern kann;
    
- wenn viele `if / else if`- oder `switch`-Abfragen vom Zustand abhängen;
    
- wenn verschiedene Zustände viel eigene Logik besitzen;
    
- wenn Zustandsübergänge klar voneinander getrennt werden sollen.
    

Typischer problematischer Code:

```csharp
if (State == State.A)
{
    // Verhalten für Zustand A
}
else if (State == State.B)
{
    // Verhalten für Zustand B
}
else if (State == State.C)
{
    // Verhalten für Zustand C
}
```

Wenn solche Prüfungen in vielen Methoden vorkommen, wird der Code schnell unübersichtlich.

---

# Grundstruktur

```text
                Context
                   │
                   │ verwendet
                   ▼
                 IState
                   ▲
          ┌────────┼────────┐
          │        │        │
       StateA   StateB   StateC
```

Der `Context` besitzt immer einen aktuellen Zustand.

Die konkrete Aktion wird an diesen Zustand delegiert.

---

# Teilnehmer des Patterns

## State

`State` definiert den gemeinsamen Vertrag für alle Zustände.

Zum Beispiel:

```csharp
public interface IState
{
    void Handle(Context context);
}
```

---

## ConcreteState

Konkrete State-Klassen implementieren das Verhalten eines bestimmten Zustands.

Zum Beispiel:

```text
StateA
StateB
StateC
```

Jede State-Klasse weiß:

- wie sie auf Aktionen reagieren muss;
    
- ob danach ein Zustandswechsel stattfinden soll.
    

---

## Context

Der `Context` ist das eigentliche Objekt, dessen Verhalten sich abhängig vom Zustand verändert.

Er delegiert die Arbeit an das aktuelle State-Objekt.

---

# Allgemeines Beispiel

```csharp
// Gemeinsamer Vertrag für alle Zustände.
public interface IState
{
    // Jeder Zustand entscheidet selbst,
    // wie auf die Anfrage reagiert wird.
    void Handle(Context context);
}
```

---

# StateA

```csharp
public class StateA : IState
{
    public void Handle(Context context)
    {
        Console.WriteLine("Zustand A wird verarbeitet.");

        // Nach der Verarbeitung wechseln wir zu Zustand B.
        context.State = new StateB();
    }
}
```

---

# StateB

```csharp
public class StateB : IState
{
    public void Handle(Context context)
    {
        Console.WriteLine("Zustand B wird verarbeitet.");

        // Danach wechseln wir wieder zu Zustand A.
        context.State = new StateA();
    }
}
```

---

# Context

```csharp
public class Context
{
    // Aktueller Zustand des Objekts.
    public IState State { get; set; }

    public Context(IState initialState)
    {
        State = initialState;
    }

    public void Request()
    {
        // Der Context führt die Aktion
        // nicht selbst aus.
        //
        // Er delegiert sie an den aktuellen Zustand.
        State.Handle(this);
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
        Context context =
            new Context(new StateA());

        context.Request();
        context.Request();
        context.Request();
    }
}
```

Ablauf:

```text
StateA
  │
  ▼
Request()
  │
  ▼
StateB
  │
  ▼
Request()
  │
  ▼
StateA
```

---

# Der wichtigste Punkt

Der `Context` muss nicht wissen:

```text
Wenn Zustand A → mache X
Wenn Zustand B → mache Y
Wenn Zustand C → mache Z
```

Stattdessen schreibt er nur:

```csharp
State.Handle(this);
```

Der aktuelle Zustand entscheidet selbst, was passieren soll.

---

# Praktisches Beispiel: Wasser

Wasser kann vereinfacht drei Zustände besitzen:

```text
Fest
Flüssig
Gasförmig
```

Wir möchten zwei Aktionen unterstützen:

```csharp
Heat();
Frost();
```

also:

```text
Heat()  → erwärmen
Frost() → abkühlen / gefrieren
```

---

# Ohne State Pattern

Zunächst könnten wir einen Enum verwenden:

```csharp
public enum WaterState
{
    Solid,
    Liquid,
    Gas
}
```

Dann:

```csharp
public class Water
{
    public WaterState State { get; set; }

    public Water(WaterState state)
    {
        State = state;
    }

    public void Heat()
    {
        if (State == WaterState.Solid)
        {
            Console.WriteLine(
                "Das Eis wird zu flüssigem Wasser.");

            State = WaterState.Liquid;
        }
        else if (State == WaterState.Liquid)
        {
            Console.WriteLine(
                "Das flüssige Wasser wird zu Wasserdampf.");

            State = WaterState.Gas;
        }
        else if (State == WaterState.Gas)
        {
            Console.WriteLine(
                "Die Temperatur des Wasserdampfs steigt weiter.");
        }
    }

    public void Frost()
    {
        if (State == WaterState.Liquid)
        {
            Console.WriteLine(
                "Das flüssige Wasser wird zu Eis.");

            State = WaterState.Solid;
        }
        else if (State == WaterState.Gas)
        {
            Console.WriteLine(
                "Der Wasserdampf kondensiert zu flüssigem Wasser.");

            State = WaterState.Liquid;
        }
        else if (State == WaterState.Solid)
        {
            Console.WriteLine(
                "Das Eis wird weiter abgekühlt.");
        }
    }
}
```

---

# Das Problem

Bereits bei nur drei Zuständen entstehen viele Bedingungen:

```csharp
if (...)
{
}
else if (...)
{
}
else if (...)
{
}
```

Wenn später weitere Methoden dazukommen:

```text
Heat()
Frost()
Evaporate()
Compress()
Expand()
...
```

muss in jeder Methode erneut geprüft werden:

```text
Welchen Zustand hat das Wasser?
```

Das führt schnell zu:

```text
Water
├── viele Methoden
├── viele if-Abfragen
├── viele Zustandswechsel
└── schwer wartbarer Code
```

---

# Lösung mit State Pattern

Wir lagern das zustandsabhängige Verhalten in separate Klassen aus.

```text
IWaterState
│
├── SolidWaterState
├── LiquidWaterState
└── GasWaterState
```

---

# State-Interface

```csharp
public interface IWaterState
{
    // Reaktion auf Erwärmen.
    void Heat(Water water);

    // Reaktion auf Abkühlen.
    void Frost(Water water);
}
```

---

# Context: Water

```csharp
public class Water
{
    // Aktueller Zustand des Wassers.
    public IWaterState State { get; set; }

    public Water(IWaterState initialState)
    {
        State = initialState;
    }

    public void Heat()
    {
        // Verhalten wird an den
        // aktuellen Zustand delegiert.
        State.Heat(this);
    }

    public void Frost()
    {
        // Verhalten wird an den
        // aktuellen Zustand delegiert.
        State.Frost(this);
    }
}
```

Wichtig:

`Water` enthält jetzt keine:

```csharp
if
else if
switch
```

für seine Zustände.

---

# Zustand: Eis

```csharp
public class SolidWaterState : IWaterState
{
    public void Heat(Water water)
    {
        Console.WriteLine(
            "Das Eis schmilzt und wird zu flüssigem Wasser.");

        // Zustandswechsel:
        // Fest → Flüssig
        water.State = new LiquidWaterState();
    }

    public void Frost(Water water)
    {
        Console.WriteLine(
            "Das Eis wird weiter abgekühlt.");
    }
}
```

Zustandsübergang:

```text
Solid
  │
  │ Heat()
  ▼
Liquid
```

Bei:

```csharp
Frost();
```

bleibt der Zustand:

```text
Solid
```

---

# Zustand: Flüssiges Wasser

```csharp
public class LiquidWaterState : IWaterState
{
    public void Heat(Water water)
    {
        Console.WriteLine(
            "Das flüssige Wasser verdampft.");

        // Zustandswechsel:
        // Flüssig → Gasförmig
        water.State = new GasWaterState();
    }

    public void Frost(Water water)
    {
        Console.WriteLine(
            "Das flüssige Wasser gefriert zu Eis.");

        // Zustandswechsel:
        // Flüssig → Fest
        water.State = new SolidWaterState();
    }
}
```

Übergänge:

```text
           Heat()
Liquid ─────────────► Gas

           Frost()
Liquid ─────────────► Solid
```

---

# Zustand: Wasserdampf

```csharp
public class GasWaterState : IWaterState
{
    public void Heat(Water water)
    {
        Console.WriteLine(
            "Die Temperatur des Wasserdampfs steigt weiter.");
    }

    public void Frost(Water water)
    {
        Console.WriteLine(
            "Der Wasserdampf kondensiert zu flüssigem Wasser.");

        // Zustandswechsel:
        // Gasförmig → Flüssig
        water.State = new LiquidWaterState();
    }
}
```

Übergang:

```text
Gas
 │
 │ Frost()
 ▼
Liquid
```

---

# Verwendung

```csharp
public class Program
{
    public static void Main()
    {
        // Wasser startet im flüssigen Zustand.
        Water water =
            new Water(new LiquidWaterState());

        water.Heat();
        water.Frost();
        water.Frost();
    }
}
```

---

# Ablauf Schritt für Schritt

Start:

```text
Water
  │
  ▼
LiquidWaterState
```

Dann:

```csharp
water.Heat();
```

Da aktuell:

```text
LiquidWaterState
```

aktiv ist, wird ausgeführt:

```csharp
LiquidWaterState.Heat(...)
```

Ergebnis:

```text
Liquid
  │
  │ Heat()
  ▼
Gas
```

---

Danach:

```csharp
water.Frost();
```

Aktueller Zustand:

```text
GasWaterState
```

Deshalb:

```text
Gas
 │
 │ Frost()
 ▼
Liquid
```

---

Danach noch einmal:

```csharp
water.Frost();
```

Aktueller Zustand:

```text
LiquidWaterState
```

Deshalb:

```text
Liquid
  │
  │ Frost()
  ▼
Solid
```

---

# Gesamter Ablauf

```text
Start
  │
  ▼
Liquid
  │
  │ Heat()
  ▼
Gas
  │
  │ Frost()
  ▼
Liquid
  │
  │ Frost()
  ▼
Solid
```

---

# Ausgabe

```text
Das flüssige Wasser verdampft.
Der Wasserdampf kondensiert zu flüssigem Wasser.
Das flüssige Wasser gefriert zu Eis.
```

---

# Warum ist das besser?

Ohne State Pattern:

```text
Water
│
├── Heat()
│   ├── if Solid
│   ├── if Liquid
│   └── if Gas
│
├── Frost()
│   ├── if Solid
│   ├── if Liquid
│   └── if Gas
│
└── weitere Methoden ...
```

Mit State Pattern:

```text
Water
│
├── Heat()  → State.Heat()
└── Frost() → State.Frost()

IWaterState
│
├── SolidWaterState
├── LiquidWaterState
└── GasWaterState
```

Jeder Zustand enthält nur seine eigene Logik.

---

# Zustandsübergänge

Das State Pattern eignet sich besonders gut zur Darstellung einer **State Machine**.

Für unser Beispiel:

```text
                 Heat()
        ┌─────────────────────┐
        │                     ▼
      Solid ───────────────► Liquid
        ▲                     │
        │                     │ Heat()
        │ Frost()             ▼
        └─────────────────── Gas
                              │
                              │ Frost()
                              ▼
                            Liquid
```

Vereinfacht:

```text
Solid
  │ Heat
  ▼
Liquid
  │ Heat
  ▼
Gas
```

und rückwärts:

```text
Gas
  │ Frost
  ▼
Liquid
  │ Frost
  ▼
Solid
```

---

# Wer soll den Zustand wechseln?

Es gibt zwei typische Varianten.

## Variante 1: State wechselt den Zustand

Wie in unserem Beispiel:

```csharp
public void Heat(Water water)
{
    water.State =
        new GasWaterState();
}
```

Der State kennt also den nächsten Zustand.

Vorteil:

> Die Übergangslogik befindet sich direkt beim Zustand.

---

## Variante 2: Context wechselt den Zustand

Man kann den Zustandswechsel auch im `Context` kontrollieren.

Zum Beispiel:

```csharp
public void ChangeState(IWaterState state)
{
    State = state;
}
```

Dann:

```csharp
water.ChangeState(
    new GasWaterState());
```

Welche Variante besser ist, hängt von der Anwendung ab.

---

# Bessere Kapselung des Zustands

In unserem einfachen Beispiel haben wir:

```csharp
public IWaterState State { get; set; }
```

Dadurch könnte jeder beliebige Client schreiben:

```csharp
water.State =
    new GasWaterState();
```

In einer echten Anwendung möchte man das möglicherweise verhindern.

Dann kann man den Setter verstecken:

```csharp
public class Water
{
    public IWaterState State { get; private set; }

    public Water(IWaterState initialState)
    {
        State = initialState;
    }

    public void ChangeState(IWaterState state)
    {
        State = state;
    }

    public void Heat()
    {
        State.Heat(this);
    }

    public void Frost()
    {
        State.Frost(this);
    }
}
```

Jetzt können States schreiben:

```csharp
water.ChangeState(
    new GasWaterState());
```

aber der normale Client kann den Zustand nicht direkt setzen.

---

# Verbesserte Variante

```csharp
public class Water
{
    public IWaterState State { get; private set; }

    public Water(IWaterState initialState)
    {
        State = initialState;
    }

    // Kontrollierter Zustandswechsel.
    public void ChangeState(IWaterState newState)
    {
        State = newState;
    }

    public void Heat()
    {
        State.Heat(this);
    }

    public void Frost()
    {
        State.Frost(this);
    }
}
```

Dann beispielsweise:

```csharp
public class LiquidWaterState : IWaterState
{
    public void Heat(Water water)
    {
        Console.WriteLine(
            "Das Wasser verdampft.");

        water.ChangeState(
            new GasWaterState());
    }

    public void Frost(Water water)
    {
        Console.WriteLine(
            "Das Wasser gefriert.");

        water.ChangeState(
            new SolidWaterState());
    }
}
```

Diese Variante ist meist sauberer.

---

# State Pattern und Open/Closed Principle

Angenommen, wir möchten einen neuen Zustand ergänzen:

```text
PlasmaWaterState
```

Dann können wir eine neue Klasse erstellen:

```csharp
public class PlasmaWaterState : IWaterState
{
    public void Heat(Water water)
    {
        Console.WriteLine(
            "Die Temperatur des Plasmas steigt.");
    }

    public void Frost(Water water)
    {
        Console.WriteLine(
            "Das Plasma wird wieder gasförmig.");

        water.ChangeState(
            new GasWaterState());
    }
}
```

Die zentrale Klasse:

```csharp
Water
```

muss dafür nicht mit einem neuen:

```csharp
else if
```

erweitert werden.

---

# State vs. `enum`

Ein `enum` ist nicht grundsätzlich schlecht.

Für einfache Zustände reicht:

```csharp
public enum OrderStatus
{
    New,
    Paid,
    Shipped,
    Completed
}
```

vollkommen aus.

State Pattern lohnt sich eher, wenn:

```text
Zustand
+
viel eigenes Verhalten
+
viele Aktionen
+
komplexe Übergänge
```

zusammenkommen.

---

# Wann reicht ein `enum`?

Beispiel:

```csharp
public enum UserStatus
{
    Active,
    Inactive
}
```

Wenn man nur prüfen möchte:

```csharp
if (user.Status == UserStatus.Active)
{
    // ...
}
```

braucht man nicht sofort mehrere Klassen wie:

```text
ActiveUserState
InactiveUserState
```

Das wäre unnötig kompliziert.

> [!tip]  
> Nicht jede Statusvariable benötigt das State Pattern.

---

# State Pattern lohnt sich eher bei

```text
Viele Zustände
+
viele zustandsabhängige Aktionen
+
viele Zustandsübergänge
+
große if/switch-Blöcke
```

---

# Praktisches Beispiel aus echten Anwendungen

Ein Bestellprozess kann beispielsweise Zustände besitzen:

```text
New
  │
  ▼
Paid
  │
  ▼
Shipped
  │
  ▼
Delivered
```

Jeder Zustand erlaubt andere Aktionen.

---

## Neue Bestellung

```text
Erlaubt:
✓ bezahlen
✓ stornieren

Nicht erlaubt:
✗ versenden
```

---

## Bezahlte Bestellung

```text
Erlaubt:
✓ versenden
✓ erstatten

Nicht erlaubt:
✗ erneut bezahlen
```

---

## Versandte Bestellung

```text
Erlaubt:
✓ als geliefert markieren

Nicht erlaubt:
✗ stornieren
✗ bezahlen
```

Hier kann das State Pattern sehr sinnvoll sein.

---

# Beispielstruktur

```text
IOrderState
│
├── NewOrderState
├── PaidOrderState
├── ShippedOrderState
└── DeliveredOrderState
```

Der `Order`-Context:

```text
Order
│
├── Pay()
├── Ship()
├── Cancel()
└── Deliver()
        │
        ▼
    CurrentState
```

---

# State vs. Strategy

State und Strategy sehen strukturell sehr ähnlich aus.

Beide verwenden:

```text
Context
   │
   ▼
Interface
   ▲
 ┌─┴─────────────┐
 │               │
ImplementationA ImplementationB
```

Der Unterschied liegt hauptsächlich in der **Absicht**.

---

## Strategy

Strategy beantwortet:

> **Welchen Algorithmus möchte ich verwenden?**

Beispiel:

```text
PaymentStrategy
├── PayPal
├── CreditCard
└── BankTransfer
```

Der Client wählt typischerweise die Strategie.

---

## State

State beantwortet:

> **Wie soll sich das Objekt in seinem aktuellen Zustand verhalten?**

Beispiel:

```text
OrderState
├── New
├── Paid
├── Shipped
└── Delivered
```

Der Zustand verändert sich häufig automatisch während des Lebenszyklus.

---

# State vs. Strategy

|State|Strategy|
|---|---|
|repräsentiert einen Zustand|repräsentiert einen Algorithmus|
|Zustand ändert sich während des Lebenszyklus|Strategie wird meist bewusst ausgewählt|
|States kennen häufig andere States|Strategies kennen sich normalerweise nicht|
|Zustandsübergänge sind zentral|Algorithmusaustausch ist zentral|
|Verhalten hängt vom aktuellen Zustand ab|Verhalten hängt von gewählter Strategie ab|

Merksatz:

```text
Strategy
→ "Wie soll ich etwas tun?"

State
→ "Wie verhalte ich mich gerade?"
```

---

# State vs. Template Method

## Template Method

```text
fester Ablauf
+
überschreibbare Schritte
```

basiert auf Vererbung.

---

## State

```text
aktueller Zustand
+
austauschbares Verhalten
```

basiert auf Komposition.

---

# Vorteile

- reduziert große `if / else`- und `switch`-Blöcke;
    
- zustandsabhängige Logik wird sauber getrennt;
    
- jeder Zustand hat eine eigene Verantwortung;
    
- Zustandsübergänge werden übersichtlicher;
    
- neue Zustände können leichter ergänzt werden;
    
- der `Context` bleibt klein und übersichtlich;
    
- State-Klassen können separat getestet werden.
    

---

# Nachteile

- die Anzahl der Klassen steigt;
    
- bei sehr wenigen einfachen Zuständen ist das Pattern unnötig;
    
- Zustandsübergänge können über viele Klassen verteilt sein;
    
- bei sehr komplexen State Machines kann die Navigation zwischen Zuständen schwer nachvollziehbar werden.
    

---

# Wann State nicht sinnvoll ist

Für:

```csharp
enum Status
{
    Active,
    Inactive
}
```

mit nur einer einfachen Prüfung:

```csharp
if (Status == Status.Active)
{
    Console.WriteLine("Aktiv");
}
```

wäre State meistens Overengineering.

---

# Klassische problematische Struktur

```text
Context
│
├── MethodA()
│   ├── if State1
│   ├── if State2
│   └── if State3
│
├── MethodB()
│   ├── if State1
│   ├── if State2
│   └── if State3
│
└── MethodC()
    ├── if State1
    ├── if State2
    └── if State3
```

---

# Struktur mit State Pattern

```text
Context
│
├── MethodA() → State.MethodA()
├── MethodB() → State.MethodB()
└── MethodC() → State.MethodC()

State
│
├── State1
├── State2
└── State3
```

Dadurch ist die Logik nach **Zuständen** organisiert und nicht über viele Methoden verteilt.

---

# Das solltest du dir merken

Typische Struktur:

```csharp
public interface IState
{
    void Handle(Context context);
}
```

Context:

```csharp
public class Context
{
    public IState State { get; private set; }

    public Context(IState state)
    {
        State = state;
    }

    public void ChangeState(IState state)
    {
        State = state;
    }

    public void Request()
    {
        State.Handle(this);
    }
}
```

State:

```csharp
public class ConcreteState : IState
{
    public void Handle(Context context)
    {
        // Verhalten dieses Zustands

        // Optional:
        // Wechsel in einen anderen Zustand.
        context.ChangeState(
            new AnotherState());
    }
}
```

---

# Merksatz

> **State lagert zustandsabhängiges Verhalten in separate Klassen aus.**

Noch einfacher:

```text
Context:
"Ich weiß, in welchem Zustand ich bin."

State:
"Ich weiß, wie ich mich in diesem Zustand verhalten muss."
```

Oder ganz kurz:

```text
State
=
Zustand als Objekt
+
Verhalten abhängig vom Zustand
```

> [!summary] Zusammenfassung  
> Das **State Pattern** ist ein **Verhaltensmuster**.
> 
> Es wird verwendet, wenn sich das Verhalten eines Objekts abhängig von seinem aktuellen Zustand verändert.
> 
> Statt:
> 
> ```csharp
> if (state == ...)
> else if (state == ...)
> else if (state == ...)
> ```
> 
> wird jeder Zustand als eigene Klasse dargestellt:
> 
> ```text
> IState
> ├── StateA
> ├── StateB
> └── StateC
> ```
> 
> Der `Context` delegiert seine Aktionen an den aktuellen Zustand:
> 
> ```csharp
> State.Handle(this);
> ```
> 
> Zustände können anschließend selbst einen Zustandswechsel auslösen.
> 
> **Kurz gesagt:**  
> `State = Verhalten eines Objekts abhängig vom aktuellen Zustand in separate Klassen auslagern.`