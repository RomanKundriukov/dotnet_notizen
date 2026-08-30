Das **Open-Closed Principle (OCP)** lässt sich folgendermaßen formulieren:

> [!IMPORTANT]  
> Software-Entitäten sollen **offen für Erweiterungen**, aber **geschlossen für Änderungen** sein.

Die Grundidee dieses Prinzips besteht darin, ein System so zu entwerfen, dass spätere Erweiterungen möglichst durch das **Hinzufügen neuen Codes** umgesetzt werden und nicht dadurch, bereits vorhandenen Code ständig zu verändern.

---

## Einfaches Beispiel: Koch-Klasse

Betrachten wir zunächst eine einfache Klasse `Cook`:

```csharp
class Cook
{
    public string Name { get; set; }

    public Cook(string name)
    {
        Name = name;
    }

    public void MakeDinner()
    {
        Console.WriteLine("Kartoffeln schälen");
        Console.WriteLine("Die geschälten Kartoffeln auf den Herd stellen");
        Console.WriteLine("Das restliche Wasser abgießen und die gekochten Kartoffeln zu Püree zerdrücken");
        Console.WriteLine("Das Püree mit Gewürzen und Kräutern bestreuen");
        Console.WriteLine("Das Kartoffelpüree ist fertig");
    }
}
```

Mit der Methode `MakeDinner()` kann jedes Objekt dieser Klasse Kartoffelpüree zubereiten:

```csharp
Cook bob = new Cook("Bob");
bob.MakeDinner();
```

Für einen Koch reicht es natürlich nicht aus, nur Kartoffelpüree zubereiten zu können.

Wenn wir möchten, dass der Koch weitere Gerichte zubereiten kann, müssten wir die Funktionalität der Klasse erweitern.

In der aktuellen Variante würde das bedeuten, die Methode `MakeDinner()` zu ändern.

Das widerspricht jedoch der Grundidee des Open-Closed-Prinzips:

> Die Klasse soll **erweiterbar** sein, ohne dass ihr bereits bestehender Code ständig geändert werden muss.

---

## Lösung mit dem Strategy Pattern

Eine mögliche Lösung ist das **Strategy Pattern**.

Dabei wird der Teil des Verhaltens, der sich ändern kann, aus der Klasse ausgelagert und gekapselt.

In unserem Beispiel betrifft das die Zubereitung des Essens.

> [!NOTE]  
> Es ist nicht immer sofort offensichtlich, welche Teile eines Systems sich später ändern werden.  
> Deshalb muss man mögliche Änderungsgründe analysieren und veränderliches Verhalten möglichst in eigene Komponenten auslagern.

Wir ändern die Klasse `Cook` folgendermaßen:

```csharp
class Cook
{
    public string Name { get; set; }

    public Cook(string name)
    {
        Name = name;
    }

    public void MakeDinner(IMeal meal)
    {
        meal.Make();
    }
}

interface IMeal
{
    void Make();
}

class PotatoMeal : IMeal
{
    public void Make()
    {
        Console.WriteLine("Kartoffeln schälen");
        Console.WriteLine("Die geschälten Kartoffeln auf den Herd stellen");
        Console.WriteLine("Das restliche Wasser abgießen und die gekochten Kartoffeln zu Püree zerdrücken");
        Console.WriteLine("Das Püree mit Gewürzen und Kräutern bestreuen");
        Console.WriteLine("Das Kartoffelpüree ist fertig");
    }
}

class SaladMeal : IMeal
{
    public void Make()
    {
        Console.WriteLine("Tomaten und Gurken schneiden");
        Console.WriteLine("Mit Kräutern, Salz und Gewürzen bestreuen");
        Console.WriteLine("Mit Sonnenblumenöl übergießen");
        Console.WriteLine("Der Salat ist fertig");
    }
}
```

Die Zubereitung des Essens ist jetzt über das Interface `IMeal` abstrahiert.

Die konkreten Arten der Zubereitung befinden sich in den einzelnen Implementierungen des Interfaces.

Die Klasse `Cook` selbst delegiert die Zubereitung lediglich an die Methode `Make()` des übergebenen `IMeal`-Objekts.

### Verwendung

```csharp
Cook bob = new Cook("Bob");

bob.MakeDinner(new PotatoMeal());

Console.WriteLine();

bob.MakeDinner(new SaladMeal());
```

### Konsolenausgabe

```
Kartoffeln schälen
Die geschälten Kartoffeln auf den Herd stellen
Das restliche Wasser abgießen und die gekochten Kartoffeln zu Püree zerdrücken
Das Püree mit Gewürzen und Kräutern bestreuen
Das Kartoffelpüree ist fertig

Tomaten und Gurken schneiden
Mit Kräutern, Salz und Gewürzen bestreuen
Mit Sonnenblumenöl übergießen
Der Salat ist fertig
```

Die Klasse `Cook` muss jetzt nicht mehr verändert werden, wenn ein neues Gericht hinzukommt.

Wir können die Funktionalität einfach erweitern, indem wir eine neue Implementierung von `IMeal` hinzufügen.

Zum Beispiel:

```csharp
class PastaMeal : IMeal
{
    public void Make()
    {
        Console.WriteLine("Pasta zubereiten");
    }
}
```

Die Klasse `Cook` bleibt dabei unverändert.

---

## Open-Closed-Prinzip mit dem Template Method Pattern

Eine weitere verbreitete Möglichkeit, das Open-Closed-Prinzip umzusetzen, ist das **Template Method Pattern**.

Dabei definiert eine Basisklasse den grundsätzlichen Ablauf eines Algorithmus, während einzelne Schritte von abgeleiteten Klassen implementiert werden.

Wir können das vorherige Beispiel so umsetzen:

```csharp
abstract class MealBase
{
    public void Make()
    {
        Prepare();
        Cook();
        FinalSteps();
    }

    protected abstract void Prepare();
    protected abstract void Cook();
    protected abstract void FinalSteps();
}

class PotatoMeal : MealBase
{
    protected override void Prepare()
    {
        Console.WriteLine("Kartoffeln schälen und waschen");
    }

    protected override void Cook()
    {
        Console.WriteLine("Die geschälten Kartoffeln auf den Herd stellen");
        Console.WriteLine("Etwa 30 Minuten kochen");
        Console.WriteLine("Das restliche Wasser abgießen und die gekochten Kartoffeln zu Püree zerdrücken");
    }

    protected override void FinalSteps()
    {
        Console.WriteLine("Das Püree mit Gewürzen und Kräutern bestreuen");
        Console.WriteLine("Das Kartoffelpüree ist fertig");
    }
}

class SaladMeal : MealBase
{
    protected override void Prepare()
    {
        Console.WriteLine("Tomaten und Gurken waschen");
    }

    protected override void Cook()
    {
        Console.WriteLine("Tomaten und Gurken schneiden");
        Console.WriteLine("Mit Kräutern, Salz und Gewürzen bestreuen");
    }

    protected override void FinalSteps()
    {
        Console.WriteLine("Mit Sonnenblumenöl übergießen");
        Console.WriteLine("Der Salat ist fertig");
    }
}
```

Die abstrakte Klasse `MealBase` definiert mit der Methode `Make()` den festen Ablauf:

```
Vorbereiten
    ↓
Kochen / Verarbeiten
    ↓
Abschließende Schritte
```

Die konkreten Details werden in den abgeleiteten Klassen umgesetzt.

---

## Cook mit einem Menü

Die Klasse `Cook` kann nun ein Array von `MealBase`-Objekten als Menü erhalten:

```csharp
class Cook
{
    public string Name { get; set; }

    public Cook(string name)
    {
        Name = name;
    }

    public void MakeDinner(MealBase[] menu)
    {
        foreach (MealBase meal in menu)
        {
            meal.Make();
        }
    }
}
```

Die Erweiterung erfolgt jetzt durch neue Klassen, die von `MealBase` erben und ihre eigene Implementierung bereitstellen.

### Verwendung

```csharp
MealBase[] menu =
{
    new PotatoMeal(),
    new SaladMeal()
};

Cook bob = new Cook("Bob");

bob.MakeDinner(menu);
```

### Konsolenausgabe

```
Kartoffeln schälen und waschen
Die geschälten Kartoffeln auf den Herd stellen
Etwa 30 Minuten kochen
Das restliche Wasser abgießen und die gekochten Kartoffeln zu Püree zerdrücken
Das Püree mit Gewürzen und Kräutern bestreuen
Das Kartoffelpüree ist fertig

Tomaten und Gurken waschen
Tomaten und Gurken schneiden
Mit Kräutern, Salz und Gewürzen bestreuen
Mit Sonnenblumenöl übergießen
Der Salat ist fertig
```

---

## Warum ist das Open-Closed-Prinzip wichtig?

Ohne OCP sieht eine Erweiterung häufig so aus:

```
Neue Anforderung
      ↓
Bestehende Klasse ändern
      ↓
Bestehenden Code erneut testen
      ↓
Risiko für neue Fehler
```

Mit OCP versucht man dagegen:

```
Neue Anforderung
      ↓
Neue Implementierung hinzufügen
      ↓
Bestehender Code bleibt möglichst unverändert
```

Dadurch können Änderungen besser isoliert werden.

---

## Typische Möglichkeiten zur Umsetzung

Das Open-Closed-Prinzip wird häufig mit folgenden Techniken kombiniert:

- Interfaces
    
- abstrakte Klassen
    
- Vererbung
    
- Polymorphie
    
- Dependency Injection
    
- Strategy Pattern
    
- Template Method Pattern
    
- Decorator Pattern
    

> [!WARNING]  
> OCP bedeutet nicht, dass bestehender Code **niemals** geändert werden darf.  
> Das Ziel ist vielmehr, stabile Komponenten so zu entwerfen, dass neue Varianten möglichst durch Erweiterung statt durch dauernde Anpassung des bestehenden Codes hinzugefügt werden können.

---

## Kurzfassung

> [!SUMMARY]  
> **OCP = Offen für Erweiterungen, geschlossen für Änderungen.**

Statt zum Beispiel eine vorhandene Klasse bei jeder neuen Variante zu ändern:

```
Cook
 ├── Kartoffeln
 ├── Salat
 ├── Pasta
 └── Suppe
```

wird das veränderliche Verhalten ausgelagert:

```
           Cook
             │
             ▼
           IMeal
        ┌────┼─────┐
        ▼    ▼     ▼
     Potato Salad Pasta
```

Dadurch können neue Varianten ergänzt werden, ohne die zentrale Klasse verändern zu müssen.

**Merksatz:**

> Wenn eine neue Funktion hinzukommt, sollte man möglichst **neuen Code hinzufügen**, statt stabilen bestehenden Code umzuschreiben.