Das **Builder Pattern** ist ein Erzeugungsmuster, das die Erstellung eines komplexen Objekts in **mehrere einzelne Schritte** aufteilt.

Dadurch kann derselbe Erstellungsprozess verwendet werden, um **unterschiedliche Varianten eines Objekts** zu erzeugen.

> [!info] Grundidee  
> Ein komplexes Objekt wird nicht auf einmal erzeugt, sondern **Schritt für Schritt aufgebaut**.

Zum Beispiel:

```
Computer
├── CPU
├── RAM
├── SSD
├── Grafikkarte
└── Betriebssystem
```

Statt eines riesigen Konstruktors:

```
var computer = new Computer(
    cpu,
    ram,
    ssd,
    gpu,
    operatingSystem,
    wifi,
    bluetooth,
    ...
);
```

kann ein Builder die Erstellung in einzelne Schritte zerlegen.

---

# Wann sollte man Builder verwenden?

Das Builder Pattern eignet sich besonders:

- wenn ein Objekt aus vielen einzelnen Bestandteilen besteht;
- wenn die Erstellung eines Objekts mehrere Schritte benötigt;
- wenn unterschiedliche Varianten desselben Objekts erzeugt werden sollen;
- wenn nicht alle Bestandteile zwingend benötigt werden;
- wenn ein Konstruktor zu viele Parameter bekommen würde;
- wenn die Erzeugungslogik vom eigentlichen Objekt getrennt werden soll.

---

# Grundidee

```
                    Director
                       │
                       │ verwendet
                       ▼
                     Builder
                       ▲
                       │
                ConcreteBuilder
                       │
                       │ erstellt
                       ▼
                     Product
```

Dabei besitzt jeder Teilnehmer eine bestimmte Aufgabe.

---

# Teilnehmer des Patterns

## Product

`Product` ist das Objekt, das am Ende erzeugt werden soll.

```
public class Product
{
    private readonly List<string> _parts = new();

    public void Add(string part)
    {
        _parts.Add(part);
    }
}
```

---

## Builder

Der `Builder` definiert die einzelnen Schritte zur Erstellung des Produkts.

```
public abstract class Builder
{
    public abstract void BuildPartA();

    public abstract void BuildPartB();

    public abstract void BuildPartC();

    public abstract Product GetResult();
}
```

Der Builder sagt also:

> „Diese Schritte werden benötigt.“

Er definiert aber noch nicht unbedingt, **wie** die einzelnen Bestandteile erstellt werden.

---

## ConcreteBuilder

Der konkrete Builder implementiert die einzelnen Schritte.

```
public class ConcreteBuilder : Builder
{
    private readonly Product _product = new();

    public override void BuildPartA()
    {
        _product.Add("Teil A");
    }

    public override void BuildPartB()
    {
        _product.Add("Teil B");
    }

    public override void BuildPartC()
    {
        _product.Add("Teil C");
    }

    public override Product GetResult()
    {
        return _product;
    }
}
```

---

## Director

Der `Director` kennt die **Reihenfolge der Erstellungsschritte**.

```
public class Director
{
    private readonly Builder _builder;

    public Director(Builder builder)
    {
        _builder = builder;
    }

    // Definiert die Reihenfolge,
    // in der das Produkt aufgebaut wird.
    public void Construct()
    {
        _builder.BuildPartA();
        _builder.BuildPartB();
        _builder.BuildPartC();
    }
}
```

Der Director weiß also:

```
1. Teil A bauen
2. Teil B bauen
3. Teil C bauen
```

Er weiß aber nicht unbedingt, **wie diese Teile konkret erzeugt werden**.

---

# Verwendung

```
Builder builder =
    new ConcreteBuilder();

Director director =
    new Director(builder);

// Das Produkt wird Schritt für Schritt aufgebaut.
director.Construct();

// Fertiges Produkt erhalten.
Product product =
    builder.GetResult();
```

---

# Ablauf

```
Client
  │
  ▼
Director
  │
  ├── BuildPartA()
  ├── BuildPartB()
  └── BuildPartC()
          │
          ▼
   ConcreteBuilder
          │
          ▼
       Product
```

---

# Praktisches Beispiel: Brot backen

Angenommen, wir möchten verschiedene Brotsorten erzeugen.

Ein Brot kann aus mehreren Bestandteilen bestehen:

```
Bread
├── Flour
├── Salt
└── Additives
```

Dabei unterscheiden sich die Zutaten je nach Brotsorte.

Zum Beispiel:

```
Roggenbrot
├── Roggenmehl
├── Salz
└── keine Zusatzstoffe
```

und:

```
Weizenbrot
├── Weizenmehl
├── Salz
└── Backverbesserer
```

Der **Herstellungsprozess ist ähnlich**, die konkrete Zusammensetzung unterscheidet sich.

Das ist ein typischer Anwendungsfall für Builder.

---

# Bestandteile des Produkts

## Flour

```
// Repräsentiert das verwendete Mehl.
public class Flour
{
    public string Type { get; set; } = string.Empty;
}
```

---

## Salt

```
// Repräsentiert Salz als Bestandteil des Brotes.
public class Salt
{
}
```

---

## Additives

```
// Repräsentiert optionale Zusatzstoffe.
public class Additives
{
    public string Name { get; set; } = string.Empty;
}
```

---

# Product: Bread

```
public class Bread
{
    // Das verwendete Mehl.
    public Flour? Flour { get; set; }

    // Optionales Salz.
    public Salt? Salt { get; set; }

    // Optionale Zusatzstoffe.
    public Additives? Additives { get; set; }

    public override string ToString()
    {
        StringBuilder result = new();

        if (Flour is not null)
        {
            result.AppendLine(Flour.Type);
        }

        if (Salt is not null)
        {
            result.AppendLine("Salz");
        }

        if (Additives is not null)
        {
            result.AppendLine(
                $"Zusatzstoffe: {Additives.Name}");
        }

        return result.ToString();
    }
}
```

> [!note]  
> `StringBuilder` aus diesem Beispiel hat **nichts direkt mit dem Builder Design Pattern zu tun**.
> 
> `System.Text.StringBuilder` ist einfach eine .NET-Klasse zum effizienten Zusammensetzen von Zeichenketten.

---

# Abstrakter Builder

Jetzt definieren wir die einzelnen Schritte für die Herstellung eines Brotes.

```
public abstract class BreadBuilder
{
    // Das aktuell aufgebaute Brot.
    public Bread Bread { get; private set; } = null!;

    // Erstellt zunächst das leere Produkt.
    public void CreateBread()
    {
        Bread = new Bread();
    }

    // Die konkreten Builder entscheiden,
    // welches Mehl verwendet wird.
    public abstract void SetFlour();

    // Die konkreten Builder entscheiden,
    // ob und welches Salz verwendet wird.
    public abstract void SetSalt();

    // Die konkreten Builder entscheiden,
    // welche Zusatzstoffe verwendet werden.
    public abstract void SetAdditives();
}
```

---

# ConcreteBuilder: Roggenbrot

```
// Builder für Roggenbrot.
public class RyeBreadBuilder : BreadBuilder
{
    public override void SetFlour()
    {
        Bread.Flour = new Flour
        {
            Type = "Roggenmehl, Type 1150"
        };
    }

    public override void SetSalt()
    {
        Bread.Salt = new Salt();
    }

    public override void SetAdditives()
    {
        // Roggenbrot benötigt in diesem Beispiel
        // keine zusätzlichen Zusatzstoffe.
    }
}
```

Dieser Builder erzeugt:

```
RyeBreadBuilder
     │
     ├── Roggenmehl
     ├── Salz
     └── keine Zusatzstoffe
```

---

# ConcreteBuilder: Weizenbrot

```
// Builder für Weizenbrot.
public class WheatBreadBuilder : BreadBuilder
{
    public override void SetFlour()
    {
        Bread.Flour = new Flour
        {
            Type = "Weizenmehl, Type 550"
        };
    }

    public override void SetSalt()
    {
        Bread.Salt = new Salt();
    }

    public override void SetAdditives()
    {
        Bread.Additives = new Additives
        {
            Name = "Backverbesserer"
        };
    }
}
```

Dieser Builder erzeugt:

```
WheatBreadBuilder
     │
     ├── Weizenmehl
     ├── Salz
     └── Backverbesserer
```

---

# Director: Baker

Der Bäcker übernimmt die Rolle des **Directors**.

Er kennt die Reihenfolge, in der das Brot hergestellt werden muss.

```
public class Baker
{
    public Bread Bake(BreadBuilder builder)
    {
        // Zunächst wird ein neues Brot erzeugt.
        builder.CreateBread();

        // Danach werden die einzelnen
        // Bestandteile Schritt für Schritt hinzugefügt.
        builder.SetFlour();
        builder.SetSalt();
        builder.SetAdditives();

        // Am Ende wird das fertige Brot zurückgegeben.
        return builder.Bread;
    }
}
```

Der `Baker` weiß:

```
1. Brot erstellen
2. Mehl hinzufügen
3. Salz hinzufügen
4. Zusatzstoffe hinzufügen
```

Er weiß aber nicht:

```
Welches Mehl?
Welche Zusatzstoffe?
```

Das entscheidet der konkrete Builder.

---

# Verwendung

```
public class Program
{
    public static void Main()
    {
        // Director erstellen.
        Baker baker = new();

        // Builder für Roggenbrot erstellen.
        BreadBuilder builder =
            new RyeBreadBuilder();

        // Roggenbrot Schritt für Schritt herstellen.
        Bread ryeBread =
            baker.Bake(builder);

        Console.WriteLine("Roggenbrot:");
        Console.WriteLine(ryeBread);


        // Anderen Builder verwenden.
        builder =
            new WheatBreadBuilder();

        // Der gleiche Director erzeugt jetzt
        // eine andere Variante des Produkts.
        Bread wheatBread =
            baker.Bake(builder);

        Console.WriteLine("Weizenbrot:");
        Console.WriteLine(wheatBread);
    }
}
```

Ausgabe:

```
Roggenbrot:
Roggenmehl, Type 1150
Salz

Weizenbrot:
Weizenmehl, Type 550
Salz
Zusatzstoffe: Backverbesserer
```

---

# Was passiert genau?

Bei:

```
BreadBuilder builder =
    new RyeBreadBuilder();
```

verwenden wir den Builder für Roggenbrot.

Dann:

```
Bread bread =
    baker.Bake(builder);
```

Der `Baker` ruft auf:

```
builder.CreateBread();
builder.SetFlour();
builder.SetSalt();
builder.SetAdditives();
```

Da das tatsächliche Objekt ein:

```
RyeBreadBuilder
```

ist, werden dessen Methoden ausgeführt.

Ergebnis:

```
Bread
├── Flour → Roggenmehl
├── Salt  → vorhanden
└── Additives → null
```

---

Wenn wir dagegen:

```
builder =
    new WheatBreadBuilder();
```

verwenden, bleibt der Ablauf im `Baker` exakt gleich:

```
builder.CreateBread();
builder.SetFlour();
builder.SetSalt();
builder.SetAdditives();
```

Aber jetzt entsteht:

```
Bread
├── Flour     → Weizenmehl
├── Salt      → vorhanden
└── Additives → Backverbesserer
```

---

# Der wichtigste Punkt

Der **Erstellungsprozess** bleibt gleich:

```
CreateBread
     ↓
SetFlour
     ↓
SetSalt
     ↓
SetAdditives
```

aber die konkrete Implementierung der einzelnen Schritte kann unterschiedlich sein.

```
             Baker
               │
        gleicher Ablauf
               │
       ┌───────┴───────┐
       ▼               ▼
RyeBreadBuilder   WheatBreadBuilder
       │               │
       ▼               ▼
  Roggenbrot       Weizenbrot
```

---

# Warum nicht einfach einen Konstruktor verwenden?

Ohne Builder könnte ein komplexes Objekt beispielsweise so erzeugt werden:

```
var computer = new Computer(
    "AMD Ryzen 9",
    64,
    2000,
    "RTX 5090",
    true,
    true,
    false,
    "Windows 11",
    "Deutsch");
```

Das Problem:

Was bedeutet:

```
64?
2000?
true?
true?
false?
```

Je mehr Parameter vorhanden sind, desto schwerer wird der Code zu verstehen.

Dieses Problem wird oft als:

**Telescoping Constructor Problem**

bezeichnet.

---

# Builder als Lösung

Mit einem Builder könnte derselbe Code beispielsweise so aussehen:

```
Computer computer = new ComputerBuilder()
    .SetCpu("AMD Ryzen 9")
    .SetRam(64)
    .SetStorage(2000)
    .SetGpu("RTX 5090")
    .EnableWifi()
    .InstallOperatingSystem("Windows 11")
    .Build();
```

Jetzt ist sofort verständlich, was jeder Wert bedeutet.

---

# Moderne Variante: Fluent Builder

In modernem C# begegnet man sehr häufig einem **Fluent Builder**.

Dabei geben die einzelnen Methoden den Builder selbst zurück:

```
return this;
```

Dadurch können die Methoden verkettet werden.

---

# Beispiel

```
public class Computer
{
    public string Cpu { get; set; } = string.Empty;

    public int RamGb { get; set; }

    public int StorageGb { get; set; }

    public string? GraphicsCard { get; set; }

    public string? OperatingSystem { get; set; }
}
```

Builder:

```
public class ComputerBuilder
{
    // Das Objekt wird während des Build-Prozesses
    // Schritt für Schritt konfiguriert.
    private readonly Computer _computer = new();

    public ComputerBuilder SetCpu(string cpu)
    {
        _computer.Cpu = cpu;

        // Der Builder selbst wird zurückgegeben,
        // damit weitere Methoden verkettet werden können.
        return this;
    }

    public ComputerBuilder SetRam(int ramGb)
    {
        _computer.RamGb = ramGb;

        return this;
    }

    public ComputerBuilder SetStorage(int storageGb)
    {
        _computer.StorageGb = storageGb;

        return this;
    }

    public ComputerBuilder SetGraphicsCard(string graphicsCard)
    {
        _computer.GraphicsCard = graphicsCard;

        return this;
    }

    public ComputerBuilder InstallOperatingSystem(
        string operatingSystem)
    {
        _computer.OperatingSystem =
            operatingSystem;

        return this;
    }

    // Gibt das fertig aufgebaute Objekt zurück.
    public Computer Build()
    {
        return _computer;
    }
}
```

---

# Verwendung des Fluent Builders

```
Computer gamingComputer =
    new ComputerBuilder()
        .SetCpu("AMD Ryzen 9")
        .SetRam(64)
        .SetStorage(2000)
        .SetGraphicsCard("RTX 5090")
        .InstallOperatingSystem("Windows 11")
        .Build();
```

Das nennt man:

**Method Chaining**

also das Verketten mehrerer Methodenaufrufe.

---

# Warum funktioniert Method Chaining?

Betrachten wir:

```
public ComputerBuilder SetRam(int ramGb)
{
    _computer.RamGb = ramGb;

    return this;
}
```

`this` ist das aktuelle `ComputerBuilder`-Objekt.

Deshalb liefert:

```
builder.SetRam(64)
```

wieder einen:

```
ComputerBuilder
```

zurück.

Darauf kann direkt die nächste Methode aufgerufen werden:

```
builder
    .SetRam(64)
    .SetStorage(2000)
    .SetGraphicsCard("RTX 5090");
```

---

# Builder mit optionalen Eigenschaften

Ein großer Vorteil des Builders besteht darin, dass nicht jede Eigenschaft gesetzt werden muss.

Zum Beispiel:

```
Computer officeComputer =
    new ComputerBuilder()
        .SetCpu("Intel Core i5")
        .SetRam(16)
        .SetStorage(512)
        .Build();
```

Hier wurde keine Grafikkarte angegeben.

Dagegen:

```
Computer gamingComputer =
    new ComputerBuilder()
        .SetCpu("AMD Ryzen 9")
        .SetRam(64)
        .SetStorage(2000)
        .SetGraphicsCard("RTX 5090")
        .Build();
```

Beide Objekte werden vom selben Builder erzeugt.

---

# Ist ein Director immer notwendig?

**Nein.**

Das ist ein wichtiger Punkt.

In der klassischen GoF-Struktur gibt es:

```
Director
Builder
ConcreteBuilder
Product
```

In modernen Anwendungen wird der `Director` jedoch häufig weggelassen.

Der Client verwendet dann den Builder direkt:

```
var computer =
    new ComputerBuilder()
        .SetCpu("AMD Ryzen 9")
        .SetRam(64)
        .Build();
```

Der Director ist besonders sinnvoll, wenn häufig **dieselben festen Build-Abläufe** verwendet werden.

Zum Beispiel:

```
BuildGamingComputer()
BuildOfficeComputer()
BuildDeveloperComputer()
```

---

# Director mit vordefinierten Konfigurationen

```
public class ComputerDirector
{
    public Computer BuildGamingComputer()
    {
        return new ComputerBuilder()
            .SetCpu("AMD Ryzen 9")
            .SetRam(64)
            .SetStorage(2000)
            .SetGraphicsCard("RTX 5090")
            .InstallOperatingSystem("Windows 11")
            .Build();
    }

    public Computer BuildOfficeComputer()
    {
        return new ComputerBuilder()
            .SetCpu("Intel Core i5")
            .SetRam(16)
            .SetStorage(512)
            .InstallOperatingSystem("Windows 11")
            .Build();
    }
}
```

Dann:

```
ComputerDirector director = new();

Computer gamingPc =
    director.BuildGamingComputer();

Computer officePc =
    director.BuildOfficeComputer();
```

---

# Builder vs. Factory Method

## Factory Method

Factory Method entscheidet hauptsächlich:

> **Welche konkrete Klasse soll erzeugt werden?**

```
Creator
   │
   ▼
Create()
   │
   ▼
ConcreteProduct
```

Beispiel:

```
House house =
    developer.Create();
```

---

## Builder

Builder beschäftigt sich hauptsächlich damit:

> **Wie wird ein komplexes Objekt Schritt für Schritt aufgebaut?**

```
Builder
   │
   ├── Schritt 1
   ├── Schritt 2
   ├── Schritt 3
   └── Build()
          │
          ▼
       Product
```

---

# Factory Method vs. Builder

```
Factory Method
→ Welches Objekt wird erzeugt?


Builder
→ Wie wird ein komplexes Objekt aufgebaut?
```

---

# Builder vs. Abstract Factory

## Abstract Factory

Erzeugt mehrere zusammengehörige Objekte:

```
HeroFactory
├── Weapon
└── Movement
```

---

## Builder

Baut normalerweise **ein komplexes Objekt aus mehreren Teilen**:

```
Computer
├── CPU
├── RAM
├── SSD
└── GPU
```

Merksatz:

```
Abstract Factory
→ Familie mehrerer Objekte

Builder
→ ein komplexes Objekt aus mehreren Teilen
```

---

# Builder vs. Prototype

## Prototype

```
vorhandenes Objekt
       │
       ▼
     Clone()
       │
       ▼
     Kopie
```

---

## Builder

```
leeres / neues Objekt
       │
       ▼
Schritt für Schritt
       │
       ▼
fertiges Objekt
```

Kurz:

```
Prototype
→ kopiert ein vorhandenes Objekt

Builder
→ baut ein neues Objekt schrittweise auf
```

---

# Vorteile

- Komplexe Objekterstellung wird übersichtlicher.
- Große Konstruktoren mit vielen Parametern können vermieden werden.
- Optionale Eigenschaften lassen sich einfach behandeln.
- Verschiedene Varianten desselben Produkts können erzeugt werden.
- Erzeugungslogik wird vom Produkt getrennt.
- Schrittweise Validierung ist möglich.
- Fluent Builder können sehr gut lesbaren Code erzeugen.

---

# Nachteile

- Zusätzliche Builder-Klassen erhöhen die Anzahl der Klassen.
- Für einfache Objekte ist das Pattern unnötig kompliziert.
- Änderungen am Product können Änderungen am Builder erfordern.
- Bei vielen Eigenschaften kann auch der Builder sehr groß werden.

---

# Wann Builder **nicht** sinnvoll ist

Für eine einfache Klasse:

```
public class User
{
    public string Name { get; set; } = string.Empty;

    public int Age { get; set; }
}
```

braucht man normalerweise keinen Builder.

Einfach:

```
User user = new()
{
    Name = "Roman",
    Age = 30
};
```

ist hier wesentlich verständlicher.

Builder lohnt sich eher bei:

```
vielen Eigenschaften
+
optionalen Eigenschaften
+
komplexer Validierung
+
mehreren Erstellungsschritten
```

---

# Klassischer Builder vs. Fluent Builder

|Klassischer Builder|Fluent Builder|
|---|---|
|häufig mit `Director`|häufig ohne Director|
|`BuildPartA()`|`SetCpu(...)`|
|`BuildPartB()`|`SetRam(...)`|
|Ablauf häufig vom Director gesteuert|Ablauf häufig vom Client gesteuert|
|klassische GoF-Struktur|sehr verbreitet in modernem C#|

---

# Vereinfachte klassische Struktur

```
                 Director
                    │
                    ▼
                  Builder
                    ▲
                    │
             ConcreteBuilder
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Part A     Part B     Part C
          │         │         │
          └─────────┼─────────┘
                    ▼
                  Product
```

---

# Vereinfachte moderne Struktur

```
ComputerBuilder
      │
      ├── SetCpu()
      │
      ├── SetRam()
      │
      ├── SetStorage()
      │
      ├── SetGpu()
      │
      └── Build()
             │
             ▼
          Computer
```

---

# Das solltest du dir merken

Die typische moderne Schreibweise sieht ungefähr so aus:

```
Product product =
    new ProductBuilder()
        .SetPropertyA(...)
        .SetPropertyB(...)
        .SetPropertyC(...)
        .Build();
```

Dabei:

```
Set...
→ konfiguriert einen Teil

return this
→ ermöglicht Method Chaining

Build()
→ liefert das fertige Objekt
```

---

# Merksatz

> **Builder erstellt ein komplexes Objekt Schritt für Schritt.**

Noch einfacher:

```
Factory:
"Welches Objekt soll ich erzeugen?"

Prototype:
"Welches Objekt soll ich kopieren?"

Builder:
"Wie soll ich das Objekt Schritt für Schritt zusammenbauen?"
```

> [!summary] Zusammenfassung  
> Das **Builder Pattern** trennt die Konstruktion eines komplexen Objekts vom eigentlichen Produkt.
> 
> Der **Builder** definiert beziehungsweise implementiert die einzelnen Erstellungsschritte.
> 
> Der optionale **Director** bestimmt die Reihenfolge dieser Schritte.
> 
> Dadurch kann derselbe Erstellungsprozess unterschiedliche Varianten eines Produkts erzeugen.
> 
> In modernem C# sieht man besonders häufig **Fluent Builder**:
> 
> ```
> var computer = new ComputerBuilder()
>     .SetCpu("AMD Ryzen 9")
>     .SetRam(64)
>     .SetStorage(2000)
>     .Build();
> ```
> 
> **Kurz gesagt:**  
> `Builder = komplexes Objekt Schritt für Schritt aufbauen.`