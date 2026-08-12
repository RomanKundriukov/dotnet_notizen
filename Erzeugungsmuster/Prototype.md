Das **Prototype Pattern** ist ein Erzeugungsmuster, bei dem neue Objekte nicht vollständig neu konstruiert werden, sondern auf Basis eines bereits vorhandenen Objekts – des **Prototyps** – erzeugt werden.

Das vorhandene Objekt wird dabei **geklont**.

> [!info] Grundidee  
> Anstatt ein komplexes Objekt jedes Mal neu zu erzeugen und vollständig zu konfigurieren, erstellt man eine Kopie eines bereits vorbereiteten Objekts.

Vereinfacht:

```
Vorhandenes Objekt
        │
        │ Clone()
        ▼
   Neues Objekt
   mit denselben
   Ausgangsdaten
```

---

# Wann sollte man Prototype verwenden?

Das Prototype Pattern eignet sich besonders:

- wenn die konkrete Klasse des zu erzeugenden Objekts erst zur Laufzeit bestimmt wird;
- wenn die Erstellung und Initialisierung eines Objekts aufwendig ist;
- wenn bereits fertig konfigurierte Objekte als Vorlage verwendet werden sollen;
- wenn viele ähnliche Objekte benötigt werden;
- wenn man zusätzliche Factory-Hierarchien vermeiden möchte;
- wenn ein Objekt nur wenige typische Ausgangskonfigurationen besitzt.

Beispiel:

```
Vorlage:
Enemy
├── Health = 100
├── Damage = 20
├── Speed = 5
└── Weapon = Sword

        Clone()
           │
     ┌─────┴─────┐
     ▼           ▼
 Enemy 1      Enemy 2
```

Anstatt jeden Gegner von Grund auf zu konfigurieren, wird ein vorbereiteter Gegner kopiert.

---

# Grundstruktur

Das klassische Pattern besteht aus:

```
Prototype
    ▲
    │
 ┌──┴───────────────┐
 │                  │
ConcretePrototypeA  ConcretePrototypeB
 │                  │
 └──── Clone() ─────┘
```

Der Client arbeitet mit dem Prototype und ruft:

```
Clone();
```

auf.

---

# Allgemeine Implementierung

Für modernes C# ist ein eigenes generisches Interface oft verständlicher als `ICloneable`.

```
// Definiert einen Vertrag für Objekte,
// die sich selbst kopieren können.
public interface IPrototype<T>
{
    T Clone();
}
```

Dann beispielsweise:

```
// Konkreter Prototype.
public class ConcretePrototypeA
    : IPrototype<ConcretePrototypeA>
{
    public int Id { get; }

    public ConcretePrototypeA(int id)
    {
        Id = id;
    }

    // Erstellt eine neue Kopie
    // des aktuellen Objekts.
    public ConcretePrototypeA Clone()
    {
        return new ConcretePrototypeA(Id);
    }
}
```

Und ein zweiter Prototype:

```
// Zweiter konkreter Prototype.
public class ConcretePrototypeB
    : IPrototype<ConcretePrototypeB>
{
    public int Id { get; }

    public ConcretePrototypeB(int id)
    {
        Id = id;
    }

    public ConcretePrototypeB Clone()
    {
        return new ConcretePrototypeB(Id);
    }
}
```

---

# Verwendung

```
ConcretePrototypeA prototype =
    new ConcretePrototypeA(10);

// Neues Objekt auf Basis des Prototyps erzeugen.
ConcretePrototypeA clone =
    prototype.Clone();
```

Danach haben wir:

```
prototype
    │
    │ Id = 10
    ▼
┌───────────┐
│ Objekt A  │
└───────────┘


clone
    │
    │ Id = 10
    ▼
┌───────────┐
│ Objekt B  │
└───────────┘
```

Beide Objekte besitzen dieselben Ausgangsdaten, sind aber zwei unterschiedliche Instanzen.

---

# Teilnehmer des Patterns

## Prototype

Der Prototype definiert die Möglichkeit, ein Objekt zu kopieren.

Zum Beispiel:

```
public interface IPrototype<T>
{
    T Clone();
}
```

---

## ConcretePrototype

Eine konkrete Klasse implementiert die Kopierlogik.

```
public class ConcretePrototype
    : IPrototype<ConcretePrototype>
{
    public ConcretePrototype Clone()
    {
        return new ConcretePrototype();
    }
}
```

---

## Client

Der Client erstellt neue Objekte nicht unbedingt direkt über:

```
new ConcretePrototype();
```

sondern verwendet ein vorhandenes Objekt:

```
ConcretePrototype clone =
    prototype.Clone();
```

---

# Praktisches Beispiel: geometrische Figuren

Wir haben:

```
Figure
├── Rectangle
└── Circle
```

Alle Figuren sollen kopiert werden können.

---

# Interface

```
// Gemeinsamer Vertrag für alle Figuren.
public interface IFigure
{
    // Erstellt eine Kopie der aktuellen Figur.
    IFigure Clone();

    // Gibt Informationen über die Figur aus.
    void GetInfo();
}
```

---

# Rectangle

```
public class Rectangle : IFigure
{
    private readonly int _width;
    private readonly int _height;

    public Rectangle(int width, int height)
    {
        _width = width;
        _height = height;
    }

    // Erstellt ein neues Rectangle
    // mit denselben Eigenschaften.
    public IFigure Clone()
    {
        return new Rectangle(_width, _height);
    }

    public void GetInfo()
    {
        Console.WriteLine(
            $"Rechteck: Höhe {_height}, Breite {_width}");
    }
}
```

---

# Circle

```
public class Circle : IFigure
{
    private readonly int _radius;

    public Circle(int radius)
    {
        _radius = radius;
    }

    // Erstellt einen neuen Kreis
    // mit demselben Radius.
    public IFigure Clone()
    {
        return new Circle(_radius);
    }

    public void GetInfo()
    {
        Console.WriteLine(
            $"Kreis mit Radius {_radius}");
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
        // Ursprüngliches Rechteck.
        IFigure rectangle =
            new Rectangle(30, 40);

        // Kopie des Rechtecks.
        IFigure clonedRectangle =
            rectangle.Clone();

        rectangle.GetInfo();
        clonedRectangle.GetInfo();

        // Ursprünglicher Kreis.
        IFigure circle =
            new Circle(30);

        // Kopie des Kreises.
        IFigure clonedCircle =
            circle.Clone();

        circle.GetInfo();
        clonedCircle.GetInfo();
    }
}
```

Ausgabe:

```
Rechteck: Höhe 40, Breite 30
Rechteck: Höhe 40, Breite 30

Kreis mit Radius 30
Kreis mit Radius 30
```

---

# Wichtig: Kopie bedeutet nicht dieselbe Referenz

Nach:

```
IFigure clone = rectangle.Clone();
```

sollten `rectangle` und `clone` normalerweise zwei unterschiedliche Objekte sein.

```
rectangle ─────► Objekt 1

clone ─────────► Objekt 2
```

Nicht:

```
rectangle ──┐
            ▼
          Objekt
            ▲
clone ──────┘
```

---

# `MemberwiseClone()`

.NET besitzt bereits die Methode:

```
MemberwiseClone();
```

Sie ist in `System.Object` definiert und erzeugt eine **flache Kopie – Shallow Copy**. Werttypen werden kopiert; bei Referenztypen wird dagegen nur die Referenz kopiert, sodass Original und Klon weiterhin auf dasselbe untergeordnete Objekt zeigen.

Da `MemberwiseClone()` geschützt ist, wird sie typischerweise innerhalb der Klasse aufgerufen:

```
public class Circle : IFigure
{
    private int _radius;

    public Circle(int radius)
    {
        _radius = radius;
    }

    public IFigure Clone()
    {
        // Erstellt eine flache Kopie
        // des aktuellen Objekts.
        return (IFigure)MemberwiseClone();
    }

    public void GetInfo()
    {
        Console.WriteLine(
            $"Kreis mit Radius {_radius}");
    }
}
```

---

# Shallow Copy und Deep Copy

Beim Prototype Pattern ist dieser Unterschied **sehr wichtig**.

## Shallow Copy

**Shallow Copy = flache Kopie**

Nur das äußere Objekt wird kopiert.

Enthaltene Referenzobjekte werden **nicht erneut erzeugt**.

Beispiel:

```
Original Circle
     │
     └──────────► Point


Clone Circle
     │
     └──────────► Point
```

Beide Kreise verwenden dasselbe `Point`-Objekt.

---

# Beispiel für Shallow Copy

Wir erstellen zunächst eine Klasse `Point`:

```
public class Point
{
    public int X { get; set; }

    public int Y { get; set; }
}
```

Jetzt enthält unser Kreis ein `Point`-Objekt:

```
public class Circle
{
    public int Radius { get; set; }

    public Point Center { get; set; }

    public Circle(int radius, int x, int y)
    {
        Radius = radius;

        Center = new Point
        {
            X = x,
            Y = y
        };
    }

    // MemberwiseClone erzeugt nur
    // eine flache Kopie.
    public Circle ShallowClone()
    {
        return (Circle)MemberwiseClone();
    }
}
```

Verwendung:

```
Circle original =
    new Circle(30, 50, 60);

Circle clone =
    original.ShallowClone();

// Punkt im Original verändern.
original.Center.X = 100;

Console.WriteLine(original.Center.X);
Console.WriteLine(clone.Center.X);
```

Ausgabe:

```
100
100
```

Warum?

Weil beide `Circle`-Objekte auf **dasselbe `Point`-Objekt** zeigen. Genau dieses Verhalten beschreibt Microsoft für `MemberwiseClone()`.

---

# Darstellung der Shallow Copy

Vor dem Klonen:

```
original
   │
   ▼
Circle
   │
   │ Center
   ▼
Point
X = 50
Y = 60
```

Nach dem Klonen:

```
original
   │
   ▼
Circle
   │
   └─────────────┐
                 ▼
              Point
             X = 50
             Y = 60
                 ▲
   ┌─────────────┘
   │
Circle
   ▲
   │
 clone
```

Es gibt:

```
2 × Circle
1 × Point
```

---

# Was passiert bei einer Änderung?

```
original.Center.X = 100;
```

Wir verändern nicht die Referenz selbst, sondern das Objekt, auf das beide Referenzen zeigen.

Deshalb:

```
original.Center.X = 100
clone.Center.X    = 100
```

---

# Deep Copy

**Deep Copy = tiefe Kopie**

Bei einer tiefen Kopie werden auch die enthaltenen veränderbaren Referenzobjekte kopiert.

Danach:

```
Original Circle
     │
     ▼
 Original Point


Clone Circle
     │
     ▼
 Cloned Point
```

Jetzt existieren:

```
2 × Circle
2 × Point
```

---

# Deep Copy manuell implementieren

Für unser Beispiel ist eine explizite Kopie besonders verständlich:

```
public class Point
{
    public int X { get; set; }

    public int Y { get; set; }

    // Erstellt eine unabhängige Kopie
    // des aktuellen Punktes.
    public Point Clone()
    {
        return new Point
        {
            X = X,
            Y = Y
        };
    }
}
```

Dann:

```
public class Circle
{
    public int Radius { get; set; }

    public Point Center { get; set; }

    public Circle(int radius, int x, int y)
    {
        Radius = radius;

        Center = new Point
        {
            X = x,
            Y = y
        };
    }

    // Erstellt eine vollständige,
    // unabhängige Kopie des Kreises.
    public Circle DeepClone()
    {
        return new Circle(
            Radius,
            Center.X,
            Center.Y);
    }
}
```

Noch deutlicher kann man das verschachtelte Objekt explizit klonen:

```
public Circle DeepClone()
{
    Circle clone =
        (Circle)MemberwiseClone();

    // Das Point-Objekt muss zusätzlich
    // separat kopiert werden.
    clone.Center = Center.Clone();

    return clone;
}
```

---

# Deep Copy testen

```
Circle original =
    new Circle(30, 50, 60);

Circle clone =
    original.DeepClone();

// Nur das Original verändern.
original.Center.X = 100;

Console.WriteLine(original.Center.X);
Console.WriteLine(clone.Center.X);
```

Ausgabe:

```
100
50
```

Jetzt sind die Objekte voneinander unabhängig.

---

# Darstellung der Deep Copy

```
original
   │
   ▼
Circle
   │
   ▼
Point
X = 100


clone
   │
   ▼
Circle
   │
   ▼
Point
X = 50
```

Änderungen am Original verändern den Klon nicht.

---

# Shallow Copy vs. Deep Copy

|Shallow Copy|Deep Copy|
|---|---|
|flache Kopie|tiefe Kopie|
|äußeres Objekt wird kopiert|gesamte relevante Objektstruktur wird kopiert|
|Referenzobjekte können gemeinsam verwendet werden|veränderbare Referenzobjekte werden separat kopiert|
|schneller/einfacher|aufwendiger|
|`MemberwiseClone()`|eigene Kopierlogik erforderlich|

---

# Werttypen bei `MemberwiseClone()`

Angenommen:

```
public int Radius { get; set; }
```

`int` ist ein Werttyp.

Der Wert wird direkt kopiert:

```
Original:
Radius = 30

Clone:
Radius = 30
```

Eine spätere Änderung:

```
original.Radius = 100;
```

beeinflusst den Klon nicht.

---

# Referenztypen bei `MemberwiseClone()`

Bei:

```
public Point Center { get; set; }
```

ist `Point` eine Klasse und damit ein Referenztyp.

Bei einer flachen Kopie wird die Referenz kopiert:

```
Original.Center ──┐
                  ▼
                Point
                  ▲
Clone.Center ─────┘
```

Microsoft beschreibt `MemberwiseClone()` genau als eine solche flache Kopie.

---

# Modernes Interface für Prototype

Im ursprünglichen Beispiel wird:

```
IFigure Clone();
```

verwendet.

Das funktioniert.

Mit Generics können wir es aber typsicherer gestalten:

```
public interface IPrototype<T>
{
    T Clone();
}
```

Dann:

```
public class Circle : IPrototype<Circle>
{
    public int Radius { get; }

    public Circle(int radius)
    {
        Radius = radius;
    }

    public Circle Clone()
    {
        return new Circle(Radius);
    }
}
```

Verwendung:

```
Circle original = new Circle(30);

Circle clone = original.Clone();
```

Wir benötigen keinen Cast:

```
(Circle)original.Clone();
```

---

# Warum nicht einfach `ICloneable`?

.NET besitzt zwar:

```
ICloneable
```

mit:

```
object Clone();
```

aber Microsoft weist darauf hin, dass nicht festgelegt ist, ob `Clone()` eine **Shallow Copy oder Deep Copy** liefern muss. Deshalb wird `ICloneable` für öffentliche APIs nicht empfohlen.

Zum Lernen des Prototype Patterns ist deshalb ein eigenes Interface wie:

```
public interface IPrototype<T>
{
    T Clone();
}
```

oft verständlicher und expliziter.

---

# Ein moderneres vollständiges Beispiel

```
// Gemeinsamer Vertrag für Prototype-Objekte.
public interface IPrototype<T>
{
    T Clone();
}


// Repräsentiert einen Punkt im Koordinatensystem.
public class Point : IPrototype<Point>
{
    public int X { get; set; }

    public int Y { get; set; }

    // Erstellt eine unabhängige Kopie des Punktes.
    public Point Clone()
    {
        return new Point
        {
            X = X,
            Y = Y
        };
    }
}


// Repräsentiert einen Kreis.
public class Circle : IPrototype<Circle>
{
    public int Radius { get; set; }

    public Point Center { get; set; }

    public Circle(int radius, int x, int y)
    {
        Radius = radius;

        Center = new Point
        {
            X = x,
            Y = y
        };
    }

    // Tiefe Kopie:
    // Sowohl Circle als auch Point
    // werden als neue Objekte erzeugt.
    public Circle Clone()
    {
        Circle clone =
            (Circle)MemberwiseClone();

        clone.Center = Center.Clone();

        return clone;
    }

    public void GetInfo()
    {
        Console.WriteLine(
            $"Kreis mit Radius {Radius} " +
            $"und Mittelpunkt ({Center.X}, {Center.Y})");
    }
}
```

Verwendung:

```
public class Program
{
    public static void Main()
    {
        Circle original =
            new Circle(30, 50, 60);

        // Prototype wird geklont.
        Circle clone =
            original.Clone();

        // Nur das Original verändern.
        original.Center.X = 100;

        Console.WriteLine("Original:");
        original.GetInfo();

        Console.WriteLine("Klon:");
        clone.GetInfo();
    }
}
```

Ausgabe:

```
Original:
Kreis mit Radius 30 und Mittelpunkt (100, 60)

Klon:
Kreis mit Radius 30 und Mittelpunkt (50, 60)
```

---

# Warum ist das Prototype Pattern sinnvoll?

Stellen wir uns ein komplexeres Objekt vor:

```
Enemy enemy = new Enemy();
```

Danach müssen wir vielleicht setzen:

```
enemy.Health = 100;
enemy.Damage = 30;
enemy.Speed = 5;
enemy.Armor = 50;
enemy.Level = 10;
enemy.Weapon = ...;
enemy.Skills = ...;
enemy.Appearance = ...;
```

Wenn wir 100 ähnliche Gegner benötigen, wäre das wiederholte Konfigurieren unnötig.

Mit Prototype:

```
Enemy prototype =
    CreateConfiguredEnemy();

Enemy enemy1 = prototype.Clone();
Enemy enemy2 = prototype.Clone();
Enemy enemy3 = prototype.Clone();
```

Danach können einzelne Kopien angepasst werden:

```
enemy1.Health = 150;
enemy2.Damage = 50;
enemy3.Speed = 10;
```

---

# Praktisches Beispiel: Gegner im Spiel

```
public class Enemy : IPrototype<Enemy>
{
    public string Name { get; set; }

    public int Health { get; set; }

    public int Damage { get; set; }

    public Enemy(
        string name,
        int health,
        int damage)
    {
        Name = name;
        Health = health;
        Damage = damage;
    }

    // Erstellt einen Gegner
    // mit derselben Ausgangskonfiguration.
    public Enemy Clone()
    {
        return new Enemy(
            Name,
            Health,
            Damage);
    }
}
```

Wir erstellen einmal einen Prototype:

```
Enemy orcPrototype =
    new Enemy(
        "Ork",
        health: 100,
        damage: 25);
```

Danach:

```
Enemy orc1 = orcPrototype.Clone();
Enemy orc2 = orcPrototype.Clone();
Enemy orc3 = orcPrototype.Clone();
```

Jetzt besitzen wir:

```
             OrcPrototype
                  │
        ┌─────────┼─────────┐
        ▼         ▼         ▼
      Orc 1     Orc 2     Orc 3
```

---

# Prototype Registry

Eine typische Erweiterung des Patterns ist eine Sammlung vorbereiteter Prototypen.

Zum Beispiel:

```
EnemyPrototypes

"orc"    → OrcPrototype
"goblin" → GoblinPrototype
"boss"   → BossPrototype
```

Dann können wir zur Laufzeit bestimmen, welcher Typ geklont wird.

```
public class EnemyRegistry
{
    private readonly Dictionary<string, Enemy> _prototypes =
        new();

    public void Add(
        string key,
        Enemy prototype)
    {
        _prototypes[key] = prototype;
    }

    public Enemy Create(string key)
    {
        // Das gespeicherte Prototype-Objekt
        // wird geklont.
        return _prototypes[key].Clone();
    }
}
```

Verwendung:

```
EnemyRegistry registry =
    new EnemyRegistry();

registry.Add(
    "orc",
    new Enemy("Ork", 100, 25));

registry.Add(
    "boss",
    new Enemy("Boss", 1000, 100));


// Typ wird zur Laufzeit ausgewählt.
Enemy enemy =
    registry.Create("orc");
```

Das zeigt sehr gut einen der Vorteile von Prototype:

> Welcher konkrete vorkonfigurierte Objekttyp erzeugt wird, kann zur Laufzeit entschieden werden.

---

# Wichtig: Der alte `BinaryFormatter`-Ansatz

Im ursprünglichen Beispiel wird für Deep Copy:

```
BinaryFormatter
```

verwendet.

Diese Implementierung solltest du **nicht mehr in moderne .NET-Notizen übernehmen**.

`BinaryFormatter` gilt als unsicher; Microsoft hat seine eingebaute Implementierung ab **.NET 9** entfernt. Die entsprechenden APIs werfen dort zur Laufzeit `PlatformNotSupportedException`.

Also nicht mehr:

```
BinaryFormatter formatter =
    new BinaryFormatter();
```

und nicht:

```
formatter.Serialize(...);
formatter.Deserialize(...);
```

> [!danger] Veraltet  
> Deep Copy über `BinaryFormatter` ist ein **historisches Beispiel aus älteren .NET-Versionen** und sollte heute nicht als normale Lösung verwendet werden.

---

# Wie macht man Deep Copy heute?

Für ein klar definiertes Domain-Objekt ist eine **explizite Kopierlogik** meistens die verständlichste Lösung.

Zum Beispiel:

```
public Circle Clone()
{
    return new Circle(
        Radius,
        Center.X,
        Center.Y);
}
```

oder:

```
public Circle Clone()
{
    Circle clone =
        (Circle)MemberwiseClone();

    clone.Center = Center.Clone();

    return clone;
}
```

Dadurch ist direkt sichtbar:

```
Was wird kopiert?
Wie tief wird kopiert?
Welche Objekte bleiben gemeinsam?
```

---

# Copy Constructor als Alternative

Eine sehr übersichtliche Möglichkeit ist ein **Copy Constructor**.

```
public class Point
{
    public int X { get; set; }

    public int Y { get; set; }

    // Normaler Konstruktor.
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    // Copy Constructor.
    public Point(Point source)
    {
        X = source.X;
        Y = source.Y;
    }
}
```

Beim Kreis:

```
public class Circle
{
    public int Radius { get; set; }

    public Point Center { get; set; }

    public Circle(
        int radius,
        Point center)
    {
        Radius = radius;
        Center = center;
    }

    // Copy Constructor.
    public Circle(Circle source)
    {
        Radius = source.Radius;

        // Auch Point wird kopiert.
        Center = new Point(source.Center);
    }

    public Circle Clone()
    {
        return new Circle(this);
    }
}
```

Das ist sehr gut lesbar:

```
Circle clone =
    new Circle(original);
```

---

# Vorteile

- Komplexe Objekte müssen nicht vollständig neu konfiguriert werden.
- Bereits vorbereitete Objekte können als Vorlagen verwendet werden.
- Die konkrete Klasse kann zur Laufzeit ausgewählt werden.
- Factory-Hierarchien können in manchen Fällen vermieden werden.
- Gut geeignet für viele ähnliche Objekte.
- Vorkonfigurierte Zustände lassen sich einfach kopieren.

---

# Nachteile

- Deep Copy kann bei komplexen Objektstrukturen schwierig werden.
- Man muss genau entscheiden, welche Referenzobjekte kopiert werden.
- Zirkuläre Referenzen können die Kopierlogik komplizieren.
- Änderungen an der Objektstruktur können Änderungen an `Clone()` erforderlich machen.
- Bei unklarer Definition von `Clone()` kann schwer erkennbar sein, ob eine Shallow oder Deep Copy entsteht.

---

# Prototype vs. Factory Method

## Factory Method

Die Factory erzeugt ein neues Objekt:

```
Creator
   │
   ▼
Create()
   │
   ▼
new Product()
```

Beispiel:

```
return new PanelHouse();
```

---

## Prototype

Prototype verwendet dagegen ein vorhandenes Objekt:

```
Prototype
   │
   ▼
Clone()
   │
   ▼
Kopie des Prototyps
```

Beispiel:

```
Enemy enemy =
    prototype.Clone();
```

---

# Unterschied

```
Factory Method
→ neues Objekt anhand der Erzeugungslogik

Prototype
→ neues Objekt anhand eines vorhandenen Objekts
```

---

# Prototype vs. Abstract Factory

## Abstract Factory

```
Factory
├── CreateWeapon()
├── CreateMovement()
└── ...
```

Eine Factory erzeugt eine Produktfamilie.

---

## Prototype

```
bereits konfiguriertes Objekt
             │
             ▼
           Clone()
             │
             ▼
            Kopie
```

Es ist nicht zwingend eine separate Factory-Hierarchie erforderlich.

---

# Wichtigster Unterschied beim Kopieren

```
                 Clone
                   │
          ┌────────┴────────┐
          │                 │
     Shallow Copy       Deep Copy
          │                 │
          ▼                 ▼
 äußeres Objekt      gesamte relevante
 wird kopiert        Objektstruktur
          │           wird kopiert
          ▼                 ▼
Referenzen können     unabhängige
geteilt werden        Unterobjekte
```

---

# Merksatz

> **Prototype erzeugt neue Objekte, indem bereits vorhandene Objekte geklont werden.**

Noch einfacher:

```
Factory:
"Erstelle mir ein neues Objekt."

Prototype:
"Kopiere mir dieses Objekt."
```

---

# Das solltest du dir für C# merken

```
public interface IPrototype<T>
{
    T Clone();
}
```

Einfache Kopie:

```
public Circle Clone()
{
    return new Circle(Radius);
}
```

Shallow Copy:

```
public Circle Clone()
{
    return (Circle)MemberwiseClone();
}
```

Deep Copy:

```
public Circle Clone()
{
    Circle clone =
        (Circle)MemberwiseClone();

    clone.Center = Center.Clone();

    return clone;
}
```

> [!summary] Zusammenfassung  
> Das **Prototype Pattern** erzeugt neue Objekte durch das **Klonen bereits vorhandener Objekte**.
> 
> Es ist besonders sinnvoll, wenn ein Objekt bereits aufwendig konfiguriert wurde und weitere ähnliche Objekte benötigt werden.
> 
> Besonders wichtig ist der Unterschied zwischen **Shallow Copy** und **Deep Copy**:
> 
> - **Shallow Copy** → Referenzobjekte können gemeinsam verwendet werden.
> - **Deep Copy** → auch die enthaltenen veränderbaren Objekte werden kopiert.
> 
> `MemberwiseClone()` erzeugt in .NET eine **Shallow Copy**.
> 
> Für moderne C#-APIs ist ein eigenes typsicheres `Clone()`-Interface meist verständlicher als `ICloneable`, dessen Vertrag nicht festlegt, ob tief oder flach kopiert wird.
> 
> Die alte Deep-Copy-Lösung mit `BinaryFormatter` sollte nicht mehr verwendet werden; dessen eingebaute Implementierung wurde ab .NET 9 entfernt.