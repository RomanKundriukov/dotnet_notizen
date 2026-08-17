Die **Abstract Factory** ist ein Erzeugungsmuster, das eine Schnittstelle zur Erstellung von **Familien zusammengehöriger Objekte** bereitstellt, ohne dass der aufrufende Code die konkreten Klassen dieser Objekte kennen muss.

> [!info] Grundidee  
> Eine Abstract Factory erzeugt nicht nur **ein Produkt**, sondern mehrere **zusammengehörige Produkte**, die gemeinsam verwendet werden sollen.

Beispiel:

```
ElfFactory
├── erzeugt Crossbow
└── erzeugt FlyMovement

WarriorFactory
├── erzeugt Sword
└── erzeugt RunMovement
```

Der Client muss dabei nicht wissen, welche konkreten Klassen erzeugt werden.

---

# Wann sollte man Abstract Factory verwenden?

Das Pattern eignet sich besonders:

- wenn die Anwendung nicht von konkreten Klassen abhängig sein soll;
- wenn mehrere zusammengehörige Objekte gemeinsam erzeugt werden müssen;
- wenn unterschiedliche **Produktfamilien** existieren;
- wenn sichergestellt werden soll, dass nur miteinander kompatible Objekte kombiniert werden;
- wenn man zwischen kompletten Objekt-Konfigurationen wechseln möchte.

---

# Grundidee

Stellen wir uns zwei Produkttypen vor:

```
ProductA
ProductB
```

und davon jeweils mehrere Varianten:

```
Familie 1:
ProductA1
ProductB1

Familie 2:
ProductA2
ProductB2
```

Eine Factory erzeugt immer Produkte derselben Familie:

```
ConcreteFactory1
├── ProductA1
└── ProductB1

ConcreteFactory2
├── ProductA2
└── ProductB2
```

---

# Allgemeine Struktur

```
                  AbstractFactory
                         ▲
                ┌────────┴────────┐
                │                 │
        ConcreteFactory1   ConcreteFactory2
                │                 │
        ┌───────┴───────┐ ┌───────┴───────┐
        ▼               ▼ ▼               ▼
    ProductA1       ProductB1 ProductA2   ProductB2
        ▲               ▲       ▲           ▲
        │               │       │           │
 AbstractProductA       │ AbstractProductA  │
                        │                   │
                AbstractProductB    AbstractProductB
```

---

# Allgemeines C#-Beispiel

```
// Abstrakte Produktklasse A.
// Alle Produkte der Kategorie A leiten sich davon ab.
public abstract class AbstractProductA
{
}

// Abstrakte Produktklasse B.
// Alle Produkte der Kategorie B leiten sich davon ab.
public abstract class AbstractProductB
{
}

// Konkretes Produkt A aus Familie 1.
public class ProductA1 : AbstractProductA
{
}

// Konkretes Produkt B aus Familie 1.
public class ProductB1 : AbstractProductB
{
}

// Konkretes Produkt A aus Familie 2.
public class ProductA2 : AbstractProductA
{
}

// Konkretes Produkt B aus Familie 2.
public class ProductB2 : AbstractProductB
{
}
```

---

# Abstract Factory

Die abstrakte Factory definiert, **welche Arten von Produkten erzeugt werden können**.

```
// Abstrakte Fabrik.
// Definiert Methoden zur Erstellung verschiedener Produkttypen.
public abstract class AbstractFactory
{
    public abstract AbstractProductA CreateProductA();

    public abstract AbstractProductB CreateProductB();
}
```

Wichtig:

Die Factory gibt keine konkreten Typen zurück wie:

```
ProductA1
```

sondern Abstraktionen:

```
AbstractProductA
```

Dadurch muss der Client die konkrete Implementierung nicht kennen.

---

# Konkrete Factory 1

```
// Erzeugt Produkte aus Produktfamilie 1.
public class ConcreteFactory1 : AbstractFactory
{
    public override AbstractProductA CreateProductA()
    {
        return new ProductA1();
    }

    public override AbstractProductB CreateProductB()
    {
        return new ProductB1();
    }
}
```

Diese Factory produziert also:

```
ProductA1
ProductB1
```

---

# Konkrete Factory 2

```
// Erzeugt Produkte aus Produktfamilie 2.
public class ConcreteFactory2 : AbstractFactory
{
    public override AbstractProductA CreateProductA()
    {
        return new ProductA2();
    }

    public override AbstractProductB CreateProductB()
    {
        return new ProductB2();
    }
}
```

Diese Factory produziert:

```
ProductA2
ProductB2
```

---

# Client

Der Client kennt nur:

```
AbstractFactory
AbstractProductA
AbstractProductB
```

Die konkreten Klassen interessieren ihn nicht.

```
public class Client
{
    private readonly AbstractProductA _productA;
    private readonly AbstractProductB _productB;

    public Client(AbstractFactory factory)
    {
        // Die konkrete Factory entscheidet,
        // welche Produktfamilie erzeugt wird.
        _productA = factory.CreateProductA();
        _productB = factory.CreateProductB();
    }

    public void Run()
    {
        // Hier könnte mit beiden Produkten gearbeitet werden.
    }
}
```

---

# Teilnehmer des Patterns

## AbstractProduct

Die abstrakten Produkte definieren gemeinsame Verträge für verschiedene Produkttypen.

```
AbstractProductA
AbstractProductB
```

Beispiel:

```
public abstract class AbstractProductA
{
}

public abstract class AbstractProductB
{
}
```

---

## ConcreteProduct

Das sind konkrete Implementierungen der Produkte.

```
ProductA1
ProductA2

ProductB1
ProductB2
```

Dabei gehören:

```
ProductA1 + ProductB1
```

zur ersten Produktfamilie.

Und:

```
ProductA2 + ProductB2
```

zur zweiten Produktfamilie.

---

## AbstractFactory

Die Abstract Factory beschreibt, welche Produkte erzeugt werden können.

```
public abstract class AbstractFactory
{
    public abstract AbstractProductA CreateProductA();

    public abstract AbstractProductB CreateProductB();
}
```

---

## ConcreteFactory

Eine konkrete Factory entscheidet, **welche konkrete Produktfamilie** erzeugt wird.

```
ConcreteFactory1
→ ProductA1
→ ProductB1

ConcreteFactory2
→ ProductA2
→ ProductB2
```

---

## Client

Der Client verwendet nur Abstraktionen.

Er weiß zum Beispiel:

```
AbstractProductA productA;
```

aber nicht unbedingt, ob tatsächlich:

```
ProductA1
```

oder:

```
ProductA2
```

dahintersteckt.

---

# Praktisches Beispiel: Computerspiel

Angenommen, wir entwickeln ein Spiel mit verschiedenen Helden.

Jeder Held benötigt:

1. eine **Waffe**
2. eine **Art der Bewegung**

Wir haben beispielsweise zwei Helden:

### Elf

```
Waffe      → Armbrust
Bewegung   → Fliegen
```

### Krieger

```
Waffe      → Schwert
Bewegung   → Laufen
```

Damit entstehen zwei zusammengehörige Produktfamilien:

```
Elf-Familie
├── Crossbow
└── FlyMovement

Krieger-Familie
├── Sword
└── RunMovement
```

Genau dafür eignet sich die **Abstract Factory**.

---

# Abstraktes Produkt: Weapon

```
// Abstrakte Basisklasse für alle Waffen.
public abstract class Weapon
{
    // Jede konkrete Waffe definiert selbst,
    // wie sie angreift.
    public abstract void Hit();
}
```

---

# Abstraktes Produkt: Movement

```
// Abstrakte Basisklasse für Bewegungsarten.
public abstract class Movement
{
    // Jede konkrete Bewegungsart definiert,
    // wie sich der Held bewegt.
    public abstract void Move();
}
```

Wir besitzen damit zwei unterschiedliche Produkttypen:

```
Weapon
Movement
```

---

# Konkrete Waffen

## Armbrust

```
// Konkrete Waffe für den Elfen.
public class Crossbow : Weapon
{
    public override void Hit()
    {
        Console.WriteLine("Der Held schießt mit der Armbrust.");
    }
}
```

## Schwert

```
// Konkrete Waffe für den Krieger.
public class Sword : Weapon
{
    public override void Hit()
    {
        Console.WriteLine("Der Held schlägt mit dem Schwert.");
    }
}
```

---

# Konkrete Bewegungsarten

## Fliegen

```
// Bewegung für einen fliegenden Helden.
public class FlyMovement : Movement
{
    public override void Move()
    {
        Console.WriteLine("Der Held fliegt.");
    }
}
```

## Laufen

```
// Bewegung für einen laufenden Helden.
public class RunMovement : Movement
{
    public override void Move()
    {
        Console.WriteLine("Der Held läuft.");
    }
}
```

---

# Abstract Factory

Jetzt definieren wir die gemeinsame Factory für Helden.

```
// Abstrakte Fabrik für eine komplette Helden-Konfiguration.
//
// Jede konkrete Factory muss sowohl eine Waffe
// als auch eine Bewegungsart erzeugen.
public abstract class HeroFactory
{
    public abstract Weapon CreateWeapon();

    public abstract Movement CreateMovement();
}
```

Das ist der wichtigste Teil des Patterns.

Unsere Factory erzeugt nicht nur **ein Objekt**, sondern eine ganze Familie:

```
Weapon
+
Movement
```

---

# ElfFactory

Der Elf soll:

```
Armbrust + Fliegen
```

erhalten.

```
// Factory für einen Elfen.
//
// Sie erzeugt eine zusammengehörige Produktfamilie:
// Armbrust + Flugbewegung.
public class ElfFactory : HeroFactory
{
    public override Weapon CreateWeapon()
    {
        return new Crossbow();
    }

    public override Movement CreateMovement()
    {
        return new FlyMovement();
    }
}
```

---

# WarriorFactory

Der Krieger soll:

```
Schwert + Laufen
```

erhalten.

```
// Factory für einen Krieger.
//
// Sie erzeugt eine zusammengehörige Produktfamilie:
// Schwert + Laufbewegung.
public class WarriorFactory : HeroFactory
{
    public override Weapon CreateWeapon()
    {
        return new Sword();
    }

    public override Movement CreateMovement()
    {
        return new RunMovement();
    }
}
```

---

# Client: Hero

Der `Hero` kennt keine konkreten Waffen oder Bewegungsarten.

Er kennt nur:

```
Weapon
Movement
HeroFactory
```

```
// Client des Abstract-Factory-Patterns.
public class Hero
{
    private readonly Weapon _weapon;
    private readonly Movement _movement;

    public Hero(HeroFactory factory)
    {
        // Die Factory liefert die passende Waffe.
        _weapon = factory.CreateWeapon();

        // Die Factory liefert die dazu passende Bewegungsart.
        _movement = factory.CreateMovement();
    }

    // Der Held verwendet seine Bewegungsart.
    public void Run()
    {
        _movement.Move();
    }

    // Der Held verwendet seine Waffe.
    public void Hit()
    {
        _weapon.Hit();
    }
}
```

Der entscheidende Punkt ist:

`Hero` enthält nirgendwo:

```
new Sword();
new Crossbow();
new FlyMovement();
new RunMovement();
```

Der `Hero` kennt diese konkreten Klassen überhaupt nicht.

---

# Verwendung

```
public class Program
{
    public static void Main()
    {
        // Die ElfFactory erzeugt:
        // Crossbow + FlyMovement.
        Hero elf = new Hero(new ElfFactory());

        elf.Hit();
        elf.Run();

        // Die WarriorFactory erzeugt:
        // Sword + RunMovement.
        Hero warrior = new Hero(new WarriorFactory());

        warrior.Hit();
        warrior.Run();
    }
}
```

Ausgabe:

```
Der Held schießt mit der Armbrust.
Der Held fliegt.

Der Held schlägt mit dem Schwert.
Der Held läuft.
```

---

# Was passiert genau?

Bei:

```
Hero elf = new Hero(new ElfFactory());
```

wird eine:

```
ElfFactory
```

an `Hero` übergeben.

Der Konstruktor führt aus:

```
_weapon = factory.CreateWeapon();
_movement = factory.CreateMovement();
```

Da `factory` tatsächlich eine `ElfFactory` ist, entstehen:

```
Crossbow
FlyMovement
```

Also:

```
ElfFactory
    │
    ├── CreateWeapon()
    │       └── Crossbow
    │
    └── CreateMovement()
            └── FlyMovement
```

---

Bei:

```
Hero warrior = new Hero(new WarriorFactory());
```

entstehen dagegen:

```
Sword
RunMovement
```

Also:

```
WarriorFactory
    │
    ├── CreateWeapon()
    │       └── Sword
    │
    └── CreateMovement()
            └── RunMovement
```

---

# Warum braucht man überhaupt eine Abstract Factory?

Ohne Abstract Factory könnte unser `Hero` beispielsweise so aussehen:

```
public class Hero
{
    private Weapon _weapon = new Sword();
    private Movement _movement = new RunMovement();
}
```

Jetzt ist `Hero` direkt abhängig von:

```
Sword
RunMovement
```

Wenn wir einen Elf erzeugen möchten, müssten wir den Code verändern.

Mit Abstract Factory:

```
public Hero(HeroFactory factory)
{
    _weapon = factory.CreateWeapon();
    _movement = factory.CreateMovement();
}
```

ist `Hero` unabhängig von den konkreten Implementierungen.

Wir können einfach unterschiedliche Factories übergeben.

---

# Der wichtigste Vorteil: Produktfamilien

Abstract Factory ist besonders dann nützlich, wenn bestimmte Produkte **zusammengehören**.

Zum Beispiel:

```
ElfFactory
├── Crossbow
└── FlyMovement
```

und:

```
WarriorFactory
├── Sword
└── RunMovement
```

Die Factory stellt sicher, dass die richtige Kombination erzeugt wird.

Der Client muss nicht selbst schreiben:

```
var weapon = new Crossbow();
var movement = new FlyMovement();
```

Die Kombination ist in der Factory definiert.

---

# Erweiterung um einen neuen Helden

Angenommen, wir möchten einen Zauberer hinzufügen.

Er soll:

```
Waffe      → Zauberstab
Bewegung   → Teleportation
```

verwenden.

Zuerst erstellen wir die neuen Produkte.

```
// Neue Waffe für den Zauberer.
public class MagicStaff : Weapon
{
    public override void Hit()
    {
        Console.WriteLine("Der Held greift mit einem Zauberstab an.");
    }
}

// Neue Bewegungsart für den Zauberer.
public class TeleportMovement : Movement
{
    public override void Move()
    {
        Console.WriteLine("Der Held teleportiert sich.");
    }
}
```

Danach erstellen wir eine neue Factory:

```
// Factory für einen Zauberer.
public class WizardFactory : HeroFactory
{
    public override Weapon CreateWeapon()
    {
        return new MagicStaff();
    }

    public override Movement CreateMovement()
    {
        return new TeleportMovement();
    }
}
```

Jetzt können wir einfach schreiben:

```
Hero wizard = new Hero(new WizardFactory());

wizard.Hit();
wizard.Run();
```

Der bestehende `Hero` muss dafür **nicht geändert werden**.

---

# Vorteile

- Der Client kennt keine konkreten Produktklassen.
- Die Erzeugungslogik wird gekapselt.
- Zusammengehörige Produkte werden gemeinsam erzeugt.
- Ganze Produktfamilien können einfach ausgetauscht werden.
- Neue Produktfamilien lassen sich relativ einfach hinzufügen.
- Die Abhängigkeit von konkreten Implementierungen wird reduziert.
- Das Pattern unterstützt Dependency Inversion und häufig auch das Open/Closed Principle.

---

# Nachteile

Ein wichtiger Nachteil entsteht, wenn wir einen **neuen Produkttyp** hinzufügen möchten.

Angenommen, jeder Held soll zusätzlich Kleidung besitzen:

```
Weapon
Movement
Clothing
```

Dann muss die Factory erweitert werden:

```
public abstract class HeroFactory
{
    public abstract Weapon CreateWeapon();

    public abstract Movement CreateMovement();

    public abstract Clothing CreateClothing();
}
```

Jetzt müssen **alle bestehenden Factories** angepasst werden:

```
ElfFactory
WarriorFactory
WizardFactory
```

Denn jede Factory muss nun zusätzlich:

```
CreateClothing()
```

implementieren.

> [!warning] Wichtig  
> Eine **neue Produktfamilie** hinzuzufügen ist mit Abstract Factory relativ einfach.
> 
> Einen komplett **neuen Produkttyp innerhalb aller Familien** hinzuzufügen ist dagegen aufwendig.

---

# Sehr wichtiger Unterschied

Man muss zwischen diesen beiden Erweiterungen unterscheiden:

## Neue Familie hinzufügen

Zum Beispiel:

```
WizardFactory
├── MagicStaff
└── TeleportMovement
```

Das ist einfach.

Bestehende Factories müssen nicht geändert werden.

---

## Neue Produktart hinzufügen

Zum Beispiel:

```
Weapon
Movement
Clothing   ← NEU
```

Das ist schwieriger.

Jetzt müssen alle Factories angepasst werden:

```
ElfFactory
├── CreateWeapon()
├── CreateMovement()
└── CreateClothing()

WarriorFactory
├── CreateWeapon()
├── CreateMovement()
└── CreateClothing()
```

---

# Factory Method vs. Abstract Factory

Das ist ein sehr wichtiger Unterschied.

## Factory Method

Erzeugt typischerweise **ein Produkt**.

```
Developer
    │
    └── Create()
            │
            ▼
          House
```

Beispiel:

```
PanelDeveloper → PanelHouse
WoodDeveloper  → WoodHouse
```

Die konkrete Objekterzeugung wird an eine Unterklasse delegiert.

---

## Abstract Factory

Erzeugt eine **Familie zusammengehöriger Produkte**.

```
HeroFactory
    │
    ├── CreateWeapon()
    │
    └── CreateMovement()
```

Beispiel:

```
ElfFactory
├── Crossbow
└── FlyMovement
```

---

# Merksatz

> **Factory Method = Welche konkrete Variante eines Produkts soll erzeugt werden?**

> **Abstract Factory = Welche zusammengehörige Familie von Produkten soll erzeugt werden?**

Noch kürzer:

```
Factory Method
→ ein Produkt

Abstract Factory
→ eine Produktfamilie
```

---

# Vereinfachte Darstellung

```
                    HeroFactory
                         ▲
              ┌──────────┴──────────┐
              │                     │
         ElfFactory          WarriorFactory
              │                     │
       ┌──────┴──────┐       ┌──────┴──────┐
       │             │       │             │
       ▼             ▼       ▼             ▼
   Crossbow     FlyMovement Sword      RunMovement
       ▲             ▲       ▲             ▲
       │             │       │             │
    Weapon        Movement Weapon       Movement
```

---

# Kurzfassung

```
Abstract Factory
       │
       ▼
definiert mehrere Erzeugungsmethoden
       │
       ├── CreateWeapon()
       └── CreateMovement()
                │
                ▼
Konkrete Factory bestimmt eine Produktfamilie
```

Beispiel:

```
ElfFactory
├── Crossbow
└── FlyMovement

WarriorFactory
├── Sword
└── RunMovement
```

> [!summary] Zusammenfassung  
> Die **Abstract Factory** wird verwendet, um **Familien zusammengehöriger Objekte** zu erzeugen, ohne dass der Client ihre konkreten Klassen kennen muss.
> 
> Der Client arbeitet nur mit Abstraktionen. Welche konkreten Produkte erzeugt werden, entscheidet die konkrete Factory.
> 
> **Neue Produktfamilien** lassen sich gut ergänzen. Das Hinzufügen einer völlig **neuen Produktart** ist dagegen aufwendiger, da dafür alle Factories angepasst werden müssen.