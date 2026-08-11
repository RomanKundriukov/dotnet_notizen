
Die **Factory Method** ist ein Erzeugungsmuster, bei dem die Erstellung eines Objekts nicht direkt im Basistyp erfolgt.

Stattdessen definiert eine Basisklasse nur eine Methode zur Objekterzeugung. Die konkrete Entscheidung, **welches Objekt tatsächlich erzeugt wird**, treffen die abgeleiteten Klassen.

> [!info] Grundidee  
> Die Basisklasse beschreibt **wie ein Objekt erzeugt werden kann**, aber die Unterklassen entscheiden, **welcher konkrete Typ erzeugt wird**.

---

# Wann sollte man Factory Method verwenden?

Die Factory Method eignet sich besonders:

- wenn zur Entwicklungszeit noch nicht genau bekannt ist, welche konkreten Objekttypen erzeugt werden müssen;
    
- wenn die Anwendung unabhängig von konkreten Klassen sein soll;
    
- wenn neue Produkttypen später relativ einfach ergänzt werden sollen;
    
- wenn die Erstellung eines Objekts an abgeleitete Klassen delegiert werden soll;
    
- wenn man vermeiden möchte, überall im Programm direkt `new ConcreteClass()` aufzurufen.
    

---

# Grundstruktur

Die klassische Factory Method besteht aus vier wichtigen Bestandteilen:

```text
Product
├── ConcreteProductA
└── ConcreteProductB

Creator
├── ConcreteCreatorA
└── ConcreteCreatorB
```

Dabei gilt:

```text
Creator
   ↓
FactoryMethod()
   ↓
Product
```

Die konkreten Creator entscheiden anschließend, welcher konkrete Product-Typ erzeugt wird.

---

# Allgemeines Beispiel

```csharp
// Abstraktes Produkt.
// Definiert die gemeinsame Basis für alle Produkte.
public abstract class Product
{
}

// Konkretes Produkt A.
public class ConcreteProductA : Product
{
}

// Konkretes Produkt B.
public class ConcreteProductB : Product
{
}

// Abstrakter Erzeuger.
// Definiert die Factory Method.
public abstract class Creator
{
    // Die konkrete Implementierung dieser Methode
    // wird den abgeleiteten Klassen überlassen.
    public abstract Product FactoryMethod();
}

// Konkreter Erzeuger für ProductA.
public class ConcreteCreatorA : Creator
{
    public override Product FactoryMethod()
    {
        return new ConcreteProductA();
    }
}

// Konkreter Erzeuger für ProductB.
public class ConcreteCreatorB : Creator
{
    public override Product FactoryMethod()
    {
        return new ConcreteProductB();
    }
}
```

---

# Teilnehmer des Patterns

## Product

`Product` ist die gemeinsame Basisklasse oder ein Interface für die Objekte, die erzeugt werden sollen.

```csharp
public abstract class Product
{
}
```

---

## ConcreteProduct

`ConcreteProductA` und `ConcreteProductB` sind konkrete Implementierungen des Produkts.

```csharp
public class ConcreteProductA : Product
{
}

public class ConcreteProductB : Product
{
}
```

Es können beliebig viele konkrete Produkte existieren.

---

## Creator

`Creator` definiert die Factory Method.

```csharp
public abstract class Creator
{
    public abstract Product FactoryMethod();
}
```

Der Creator weiß:

> Es wird ein `Product` erzeugt.

Er weiß aber nicht unbedingt:

> Welche konkrete `Product`-Klasse erzeugt wird.

---

## ConcreteCreator

Die konkreten Creator implementieren die Factory Method.

```csharp
public class ConcreteCreatorA : Creator
{
    public override Product FactoryMethod()
    {
        return new ConcreteProductA();
    }
}
```

und:

```csharp
public class ConcreteCreatorB : Creator
{
    public override Product FactoryMethod()
    {
        return new ConcreteProductB();
    }
}
```

Damit wird die Entscheidung über den konkreten Produkttyp an die Unterklassen delegiert.

---

# Praktisches Beispiel: Hausbau

Angenommen, wir entwickeln eine Anwendung für eine Baufirma.

Es gibt unterschiedliche Arten von Häusern:

- Plattenhaus
    
- Holzhaus
    
- später vielleicht ein Ziegelhaus
    

Für jeden Haustyp gibt es einen entsprechenden Bauträger.

---

## Produkt

Die gemeinsame Basisklasse aller Häuser ist `House`.

```csharp
// Abstrakte Basisklasse für alle Häuser.
public abstract class House
{
}
```

---

## Konkrete Produkte

```csharp
// Konkretes Produkt:
// ein Plattenhaus.
public class PanelHouse : House
{
    public PanelHouse()
    {
        Console.WriteLine("Das Plattenhaus wurde gebaut.");
    }
}

// Konkretes Produkt:
// ein Holzhaus.
public class WoodHouse : House
{
    public WoodHouse()
    {
        Console.WriteLine("Das Holzhaus wurde gebaut.");
    }
}
```

---

# Creator

Der abstrakte `Developer` stellt den Erzeuger dar.

```csharp
// Abstrakte Basisklasse für Bauträger.
public abstract class Developer
{
    public string Name { get; }

    protected Developer(string name)
    {
        Name = name;
    }

    // Factory Method.
    // Jede konkrete Baufirma entscheidet selbst,
    // welchen Haustyp sie erstellt.
    public abstract House Create();
}
```

Die Methode:

```csharp
public abstract House Create();
```

ist unsere **Factory Method**.

---

# Concrete Creator

## Bauträger für Plattenhäuser

```csharp
// Bauträger, der Plattenhäuser erstellt.
public class PanelDeveloper : Developer
{
    public PanelDeveloper(string name) : base(name)
    {
    }

    public override House Create()
    {
        return new PanelHouse();
    }
}
```

---

## Bauträger für Holzhäuser

```csharp
// Bauträger, der Holzhäuser erstellt.
public class WoodDeveloper : Developer
{
    public WoodDeveloper(string name) : base(name)
    {
    }

    public override House Create()
    {
        return new WoodHouse();
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
        // Der konkrete Bauträger bestimmt,
        // welches House erzeugt wird.
        Developer developer = new PanelDeveloper("Musterbau GmbH");

        House panelHouse = developer.Create();

        // Wechsel des Bauträgers.
        developer = new WoodDeveloper("Holzbau GmbH");

        House woodHouse = developer.Create();
    }
}
```

Ausgabe:

```text
Das Plattenhaus wurde gebaut.
Das Holzhaus wurde gebaut.
```

---

# Was passiert hier genau?

Zunächst erstellen wir:

```csharp
Developer developer = new PanelDeveloper("Musterbau GmbH");
```

Die Variable besitzt den Typ:

```csharp
Developer
```

Das tatsächliche Objekt ist aber:

```csharp
PanelDeveloper
```

Anschließend wird aufgerufen:

```csharp
House house = developer.Create();
```

Da das tatsächliche Objekt ein `PanelDeveloper` ist, wird dessen Implementierung ausgeführt:

```csharp
public override House Create()
{
    return new PanelHouse();
}
```

Das Ergebnis ist:

```text
PanelHouse
```

---

Danach ändern wir den konkreten Creator:

```csharp
developer = new WoodDeveloper("Holzbau GmbH");
```

Beim nächsten:

```csharp
developer.Create();
```

wird deshalb aufgerufen:

```csharp
public override House Create()
{
    return new WoodHouse();
}
```

Jetzt entsteht:

```text
WoodHouse
```

---

# Warum ist das besser als direkt `new` zu verwenden?

Ohne Factory Method könnte der Client beispielsweise überall schreiben:

```csharp
var house = new PanelHouse();
```

Dadurch kennt der aufrufende Code die konkrete Klasse `PanelHouse`.

Mit Factory Method:

```csharp
House house = developer.Create();
```

arbeitet der Client hauptsächlich mit den Abstraktionen:

```text
Developer
House
```

statt direkt mit:

```text
PanelDeveloper
PanelHouse
```

Dadurch entsteht eine geringere Kopplung zwischen den einzelnen Klassen.

---

# Erweiterung um einen neuen Haustyp

Angenommen, wir möchten später ein Ziegelhaus hinzufügen.

Dafür erstellen wir zunächst ein neues Produkt:

```csharp
// Neuer Produkttyp.
public class BrickHouse : House
{
    public BrickHouse()
    {
        Console.WriteLine("Das Ziegelhaus wurde gebaut.");
    }
}
```

Danach erstellen wir den passenden Creator:

```csharp
// Bauträger für Ziegelhäuser.
public class BrickDeveloper : Developer
{
    public BrickDeveloper(string name)
        : base(name)
    {
    }

    public override House Create()
    {
        return new BrickHouse();
    }
}
```

Danach kann der neue Typ direkt verwendet werden:

```csharp
Developer developer =
    new BrickDeveloper("Ziegelbau GmbH");

House house = developer.Create();
```

Der bestehende Code für:

```text
PanelHouse
WoodHouse
```

muss dafür nicht verändert werden.

---

# Zusammenhang mit dem Open/Closed Principle

Factory Method unterstützt häufig das **Open/Closed Principle** aus SOLID.

> [!tip] Open/Closed Principle  
> Software sollte **offen für Erweiterungen**, aber möglichst **geschlossen für Änderungen** sein.

Wir können zum Beispiel einen neuen Haustyp hinzufügen:

```text
BrickHouse
```

und einen neuen Creator:

```text
BrickDeveloper
```

ohne bestehende Klassen wie:

```text
PanelDeveloper
WoodDeveloper
PanelHouse
WoodHouse
```

ändern zu müssen.

---

# Vorteile

- Der Client ist weniger stark an konkrete Klassen gekoppelt.
    
- Neue Produkttypen können relativ einfach ergänzt werden.
    
- Die Objekterzeugung wird an einer klar definierten Stelle gekapselt.
    
- Der Code arbeitet stärker mit Abstraktionen.
    
- Das Pattern unterstützt Erweiterbarkeit.
    
- Es passt gut zum Open/Closed Principle.
    

---

# Nachteile

Ein Nachteil besteht darin, dass die Anzahl der Klassen wachsen kann.

Für ein neues Produkt:

```text
BrickHouse
```

benötigt man beim klassischen Factory-Method-Ansatz häufig auch einen neuen Creator:

```text
BrickDeveloper
```

Dadurch können bei vielen Produkttypen sehr viele Klassen entstehen.

---

# Factory Method vereinfacht dargestellt

```text
                House
                 ▲
          ┌──────┴──────┐
          │             │
     PanelHouse      WoodHouse


              Developer
                 ▲
          ┌──────┴──────┐
          │             │
 PanelDeveloper    WoodDeveloper
          │             │
          ▼             ▼
     PanelHouse      WoodHouse
```

---

# Wichtig zu verstehen

Der zentrale Punkt des Patterns ist nicht einfach:

```csharp
new PanelHouse();
```

sondern:

```csharp
House Create();
```

Die Basisklasse sagt:

> „Ich kann ein `House` erzeugen.“

Die konkrete Unterklasse entscheidet:

```text
PanelDeveloper → PanelHouse
WoodDeveloper  → WoodHouse
BrickDeveloper → BrickHouse
```

---

# Merksatz

> **Factory Method delegiert die Entscheidung, welcher konkrete Objekttyp erzeugt wird, an eine Unterklasse.**

Oder noch einfacher:

```text
Basisklasse:
"Ich weiß, dass ein Produkt erzeugt werden muss."

Unterklasse:
"Ich entscheide, welches konkrete Produkt erzeugt wird."
```

---

# Kurzfassung

```text
Factory Method
      │
      ▼
Creator definiert Create()
      │
      ▼
Unterklasse überschreibt Create()
      │
      ▼
Unterklasse erzeugt konkretes Produkt
```

Beispiel:

```text
Developer
    │
    ├── PanelDeveloper
    │       └── erzeugt PanelHouse
    │
    └── WoodDeveloper
            └── erzeugt WoodHouse
```

> [!summary] Zusammenfassung  
> **Factory Method** wird verwendet, wenn die konkrete Objekterzeugung nicht fest im Basistyp implementiert werden soll. Eine Basisklasse definiert eine Factory Method, während die abgeleiteten Klassen entscheiden, welcher konkrete Produkttyp erzeugt wird.