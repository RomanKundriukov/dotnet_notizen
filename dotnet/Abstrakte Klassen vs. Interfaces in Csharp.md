
### Wann sollten abstrakte Klassen verwendet werden:

    Wenn eine gemeinsame Funktionalität für ver# Abstrakte Klassen vs. Interfaces in C#

## Abstrakte Klassen

### Wann sollte man abstrakte Klassen verwenden?

Abstrakte Klassen eignen sich besonders dann, wenn mehrere **verwandte Klassen** gemeinsame Eigenschaften und gemeinsames Verhalten besitzen.

Typische Fälle:

- Wenn mehrere verwandte Klassen eine **gemeinsame Basisfunktionalität** benötigen.
    
- Wenn die Basisklasse bereits einen Teil der Logik implementieren soll.
    
- Wenn mehrere Klassen einen **gemeinsamen Zustand** besitzen, z. B.:
    
    - Eigenschaften
        
    - Felder
        
    - Konstruktoren
        
- Wenn Änderungen an der gemeinsamen Implementierung zentral in der Basisklasse vorgenommen werden sollen.
    

Eine abstrakte Klasse kann sowohl:

- **abstrakte Methoden** ohne Implementierung
    
- als auch **normale Methoden** mit fertiger Implementierung
    

enthalten.

### Beispiel

```csharp
// Abstrakte Basisklasse für alle Fahrzeuge
public abstract class Vehicle
{
    // Abstrakte Methode:
    // Jede abgeleitete Klasse muss selbst definieren,
    // wie sie sich bewegt.
    public abstract void Move();
}

// Car erbt von Vehicle
public class Car : Vehicle
{
    // Implementierung der abstrakten Move()-Methode
    public override void Move()
    {
        Console.WriteLine("Das Auto fährt");
    }
}

// Bus erbt ebenfalls von Vehicle
public class Bus : Vehicle
{
    public override void Move()
    {
        Console.WriteLine("Der Bus fährt");
    }
}

// Tram ist ebenfalls ein Fahrzeug
public class Tram : Vehicle
{
    public override void Move()
    {
        Console.WriteLine("Die Straßenbahn fährt");
    }
}
```

Hier gehören `Car`, `Bus` und `Tram` zur gleichen fachlichen Gruppe:

> Alle sind **Fahrzeuge**.

Deshalb ist eine gemeinsame Basisklasse `Vehicle` sinnvoll.

---

## Interfaces

### Wann sollte man Interfaces verwenden?

Interfaces eignen sich besonders dann, wenn verschiedene, möglicherweise völlig **unabhängige Klassen**, dieselbe Fähigkeit besitzen sollen.

Typische Fälle:

- Wenn unterschiedliche Klassen dieselbe Funktion anbieten sollen.
    
- Wenn die Klassen keine gemeinsame Basisklasse benötigen.
    
- Wenn man eine bestimmte **Fähigkeit oder ein Verhalten** beschreiben möchte.
    
- Wenn eine Klasse mehrere unterschiedliche Fähigkeiten besitzen soll.
    

Ein Interface beschreibt hauptsächlich einen **Vertrag**:

> „Eine Klasse, die dieses Interface implementiert, muss diese Funktionalität anbieten.“

### Beispiel

```csharp
// Interface beschreibt eine Fähigkeit:
// Ein Objekt kann sich bewegen.
public interface IMovable
{
    // Jede Klasse, die IMovable implementiert,
// muss eine Move()-Methode besitzen.
    void Move();
}

// Vehicle ist eine abstrakte Basisklasse
// und implementiert gleichzeitig IMovable.
public abstract class Vehicle : IMovable
{
    // Die konkrete Implementierung wird
    // den abgeleiteten Klassen überlassen.
    public abstract void Move();
}

// Car ist ein Vehicle und damit indirekt auch IMovable.
public class Car : Vehicle
{
    public override void Move() =>
        Console.WriteLine("Das Auto fährt");
}

// Bus ist ebenfalls ein Vehicle.
public class Bus : Vehicle
{
    public override void Move() =>
        Console.WriteLine("Der Bus fährt");
}

// Horse ist kein Vehicle,
// kann sich aber ebenfalls bewegen.
public class Horse : IMovable
{
    public void Move() =>
        Console.WriteLine("Das Pferd läuft");
}

// Aircraft ist ebenfalls kein Vehicle
// in unserer Klassenhierarchie,
// besitzt aber die Fähigkeit, sich zu bewegen.
public class Aircraft : IMovable
{
    public void Move() =>
        Console.WriteLine("Das Flugzeug fliegt");
}
```

Hier sind:

- `Car` → Fahrzeug
    
- `Bus` → Fahrzeug
    
- `Horse` → Tier
    
- `Aircraft` → Flugzeug
    

Diese Klassen sind fachlich unterschiedlich.

Trotzdem können sie alle:

```csharp
Move();
```

Deshalb wird die gemeinsame Fähigkeit in das Interface `IMovable` ausgelagert.

---

# Der wichtigste Unterschied

> **Abstrakte Klasse = „Was ist das Objekt?“**

> **Interface = „Was kann das Objekt?“**

Beispiel:

```text
Car IS A Vehicle
Car CAN Move

Horse IS AN Animal
Horse CAN Move

Aircraft IS AN Aircraft
Aircraft CAN Move
```

Daraus ergibt sich:

```csharp
Car : Vehicle, IMovable
Horse : Animal, IMovable
Aircraft : IMovable
```

---

## Abstrakte Klasse oder Interface?

|Abstrakte Klasse|Interface|
|---|---|
|Für verwandte Klassen|Für möglicherweise unabhängige Klassen|
|Beschreibt häufig eine gemeinsame Basis|Beschreibt eine Fähigkeit / einen Vertrag|
|Kann Felder besitzen|Sollte keinen gemeinsamen Objektzustand verwalten|
|Kann Konstruktoren besitzen|Keine normalen Instanz-Konstruktoren|
|Kann fertige Methoden enthalten|Kann Verträge für Methoden definieren|
|Eine Klasse kann nur von **einer Klasse** erben|Eine Klasse kann **mehrere Interfaces** implementieren|

---

## Merksatz

> Wenn mehrere **homogene bzw. verwandte Klassen** einen gemeinsamen Zustand und gemeinsame Basislogik besitzen, ist eine **abstrakte Klasse** meistens sinnvoll.

> Wenn mehrere **heterogene bzw. unabhängige Klassen** lediglich dieselbe Fähigkeit besitzen sollen, ist ein **Interface** meistens die bessere Wahl.

Kurz gesagt:

```text
Abstract Class → gemeinsame Herkunft / Basis

Interface → gemeinsame Fähigkeit
```

### Beispiel

```text
Vehicle
├── Car
├── Bus
└── Tram
```

Hier passt eine abstrakte Klasse.

Dagegen:

```text
IMovable
├── Car
├── Bus
├── Horse
└── Aircraft
```

Hier passt ein Interface, weil völlig unterschiedliche Objekte dieselbe Fähigkeit `Move()` besitzen.wandte Objekte definiert werden soll.

    Wenn wir eine relativ große Funktionseinheit entwerfen, die viele grundlegende Funktionen enthält

    Wenn alle abgeleiteten Klassen auf allen Vererbungsstufen eine bestimmte gemeinsame Implementierung aufweisen sollen. Bei der Verwendung abstrakter Klassen reicht es aus, die grundlegende Funktionalität in der abstrakten Basisklasse zu ändern, wenn wir sie in allen Nachkommen ändern möchten.

    Sollten wir hingegen den Namen oder die Parameter einer Schnittstellenmethode ändern müssen, müssen wir die Änderungen ebenfalls in allen Klassen vornehmen, die diese Schnittstelle implementieren.

### Wann sollten Schnittstellen verwendet werden:

    Wenn wir Funktionen für eine Gruppe von voneinander unabhängigen Objekten definieren müssen, die möglicherweise in keiner Weise miteinander verbunden sind.

    Wenn wir einen kleinen Funktionstyp entwerfen


## Beispiel:

### Abstract

```csharp
public abstract class Vehicle
{
    public abstract void Move();
}

public class Car : Vehicle
{
    public override void Move()
    {
        Console.WriteLine("Машина едет");
    }
}

public class Bus : Vehicle
{
    public override void Move()
    {
        Console.WriteLine("Автобус едет");
    }
}

public class Tram : Vehicle
{
    public override void Move()
    {
        Console.WriteLine("Трамвай едет");
    }
}
```

### interface

```csharp
public interface IMovable
{
    void Move();
}

public abstract class Vehicle : IMovable
{
    public abstract void Move();
}

public class Car : Vehicle
{
    public override void Move() =>
        Console.WriteLine("Машина едет");
}

public class Bus : Vehicle
{
    public override void Move() =>
        Console.WriteLine("Автобус едет");
}

public class Horse : IMovable
{
    public void Move() =>
        Console.WriteLine("Лошадь скачет");
}

public class Aircraft : IMovable
{
    public void Move() =>
        Console.WriteLine("Самолет летит");
}
```

> Wenn also heterogene Klassen eine gemeinsame Funktion aufweisen, sollte diese Funktion besser in eine Schnittstelle ausgelagert werden. Bei homogenen Klassen, die einen gemeinsamen Zustand haben, ist es hingegen besser, eine abstrakte Klasse zu definieren.