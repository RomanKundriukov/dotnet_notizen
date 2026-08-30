Das **Fluent Builder Pattern** vereinfacht die Erstellung komplexer Objekte mithilfe von **verketteten Methodenaufrufen**.

Jede Methode setzt dabei eine bestimmte Eigenschaft des zu erzeugenden Objekts und gibt anschließend den Builder selbst zurück.

Dadurch entsteht eine gut lesbare, schrittweise Objektkonfiguration.

> [!IMPORTANT]  
> Ein Fluent Builder macht die Objekterstellung transparenter und den Code besser lesbar.

Historisch wurde der Fluent Builder vor allem verwendet, um das Problem sogenannter **überladener Konstruktoren** zu lösen.

---

# Problem: Zu viele Konstruktorparameter

Betrachten wir die Klasse `User`:

```csharp
public class User
{
    public string Name { get; set; }       // Name
    public string Company { get; set; }    // Unternehmen
    public int Age { get; set; }           // Alter
    public bool IsMarried { get; set; }    // Verheiratet

    public User(
        string name,
        string company,
        int age,
        bool isMarried)
    {
        Name = name;
        Company = company;
        Age = age > 0 ? age : 18;
        IsMarried = isMarried;
    }
}
```

In diesem Beispiel besitzt der Konstruktor nur vier Parameter.

In einer realen Anwendung könnten es jedoch deutlich mehr sein.

Zum Beispiel:

```csharp
new User(
    "Tom",
    "Microsoft",
    23,
    false);
```

Bei vielen Parametern entstehen mehrere Probleme:

- der Konstruktor wird sehr lang,
    
- die Reihenfolge der Parameter muss bekannt sein,
    
- der Code wird schlechter lesbar,
    
- Änderungen am Konstruktor beeinflussen viele Aufrufstellen,
    
- zusätzliche Validierungslogik bläht den Konstruktor weiter auf.
    

Beispiel:

```csharp
new User(
    "Tom",
    "Microsoft",
    23,
    false);
```

Ohne die Signatur des Konstruktors zu kennen, ist nicht sofort klar, wofür zum Beispiel `23` oder `false` stehen.

---

# Lösung mit Fluent Builder

Eine mögliche Lösung ist ein eigener Builder.

```csharp
public class UserBuilder
{
    private User user;

    public UserBuilder()
    {
        user = new User();
    }

    public UserBuilder SetName(string name)
    {
        user.Name = name;
        return this;
    }

    public UserBuilder SetCompany(string company)
    {
        user.Company = company;
        return this;
    }

    public UserBuilder SetAge(int age)
    {
        user.Age = age > 0 ? age : 0;
        return this;
    }

    public UserBuilder IsMarried
    {
        get
        {
            user.IsMarried = true;
            return this;
        }
    }

    public User Build()
    {
        return user;
    }
}
```

Der Builder enthält Methoden für die einzelnen Eigenschaften des `User`-Objekts.

Zum Beispiel:

```csharp
public UserBuilder SetName(string name)
{
    user.Name = name;
    return this;
}
```

Der entscheidende Teil ist:

```csharp
return this;
```

Dadurch wird wieder dasselbe `UserBuilder`-Objekt zurückgegeben.

Das ermöglicht die Verkettung mehrerer Methoden.

---

# Method Chaining

Dadurch können wir schreiben:

```csharp
new UserBuilder()
    .SetName("Tom")
    .SetCompany("Microsoft")
    .SetAge(23)
    .Build();
```

Der Ablauf ist:

```
UserBuilder
    │
    ▼
SetName("Tom")
    │
    ▼
SetCompany("Microsoft")
    │
    ▼
SetAge(23)
    │
    ▼
Build()
    │
    ▼
User
```

Jede Methode verändert den internen Zustand des Builders und gibt anschließend wieder den Builder zurück.

---

# Die Build-Methode

Typischerweise besitzt ein Builder eine Methode namens:

```csharp
Build()
```

Diese liefert das fertig erzeugte Objekt zurück.

```csharp
public User Build()
{
    return user;
}
```

Damit wird die Erstellung abgeschlossen.

---

# User-Klasse anpassen

Die Klasse `User` benötigt jetzt keinen großen Konstruktor mehr.

```csharp
public class User
{
    public string Name { get; set; }
    public string Company { get; set; }
    public int Age { get; set; }
    public bool IsMarried { get; set; }

    public static UserBuilder CreateBuilder()
    {
        return new UserBuilder();
    }
}
```

Zusätzlich besitzt die Klasse eine statische Methode:

```csharp
CreateBuilder()
```

Sie erzeugt einen neuen `UserBuilder`.

---

# Verwendung

Es gibt jetzt zwei mögliche Varianten.

## Builder direkt erzeugen

```csharp
User tom = new UserBuilder()
    .SetName("Tom")
    .SetCompany("Microsoft")
    .SetAge(23)
    .Build();
```

---

## Builder über User erzeugen

```csharp
User alice = User.CreateBuilder()
    .SetName("Alice")
    .IsMarried
    .SetAge(25)
    .Build();
```

Die zweite Variante liest sich fast wie eine Beschreibung:

```
Erstelle einen User
→ Name Alice
→ verheiratet
→ Alter 25
→ fertig
```

Dadurch ist direkt erkennbar, welche Werte für welche Eigenschaften gesetzt werden.

---

# Warum ist Fluent Builder besser lesbar?

Vergleichen wir beide Varianten.

## Konstruktor

```csharp
User user = new User(
    "Alice",
    "",
    25,
    true);
```

Hier muss man wissen:

```csharp
Parameter 1 = Name
Parameter 2 = Company
Parameter 3 = Age
Parameter 4 = IsMarried
```

---

## Fluent Builder

```csharp
User user = User.CreateBuilder()
    .SetName("Alice")
    .SetAge(25)
    .IsMarried
    .Build();
```

Hier ist die Bedeutung sofort sichtbar.

> [!NOTE]  
> Der große Vorteil liegt nicht darin, dass weniger Code geschrieben wird, sondern darin, dass der Code **verständlicher und flexibler** wird.

---

# Properties statt Methoden

Ein Fluent Builder muss nicht ausschließlich Methoden verwenden.

Im Beispiel wird `IsMarried` als Property umgesetzt:

```csharp
public UserBuilder IsMarried
{
    get
    {
        user.IsMarried = true;
        return this;
    }
}
```

Dadurch kann man schreiben:

```csharp
User.CreateBuilder()
    .SetName("Alice")
    .IsMarried
    .SetAge(25)
    .Build();
```

Statt:

```csharp
User.CreateBuilder()
    .SetName("Alice")
    .SetIsMarried(true)
    .SetAge(25)
    .Build();
```

Die erste Variante kann natürlicher lesbar wirken.

---

# Alternative ohne Build()

C# erlaubt das Überladen von Konvertierungsoperatoren.

Dadurch kann man den Builder implizit in ein `User`-Objekt umwandeln.

```csharp
public class UserBuilder
{
    private User user;

    public UserBuilder()
    {
        user = new User();
    }

    public UserBuilder SetName(string name)
    {
        user.Name = name;
        return this;
    }

    public UserBuilder SetCompany(string company)
    {
        user.Company = company;
        return this;
    }

    public UserBuilder SetAge(int age)
    {
        user.Age = age > 0 ? age : 0;
        return this;
    }

    public UserBuilder IsMarried
    {
        get
        {
            user.IsMarried = true;
            return this;
        }
    }

    public static implicit operator User(
        UserBuilder builder)
    {
        return builder.user;
    }
}
```

Der entscheidende Teil ist:

```csharp
public static implicit operator User(
    UserBuilder builder)
{
    return builder.user;
}
```

Damit kann C# einen `UserBuilder` automatisch in `User` umwandeln.

---

# Verwendung ohne Build()

Jetzt kann man schreiben:

```csharp
User tom = new UserBuilder()
    .SetName("Tom")
    .SetCompany("Microsoft")
    .SetAge(23);
```

Oder:

```csharp
User alice = User.CreateBuilder()
    .SetName("Alice")
    .IsMarried
    .SetAge(25);
```

Die Methode `Build()` ist dann nicht mehr notwendig.

---

# Build() oder implizite Konvertierung?

Beide Varianten sind möglich.

## Mit Build()

```csharp
User user = new UserBuilder()
    .SetName("Tom")
    .Build();
```

Vorteile:

- explizit,
    
- leicht verständlich,
    
- klar erkennbarer Abschluss,
    
- sehr verbreitet.
    

---

## Mit impliziter Konvertierung

```csharp
User user = new UserBuilder()
    .SetName("Tom");
```

Vorteile:

- kürzer,
    
- weniger Schreibaufwand.
    

Nachteile:

- weniger offensichtlich,
    
- automatische Konvertierung kann überraschend wirken,
    
- schwerer zu erkennen, wann das Objekt tatsächlich erzeugt beziehungsweise zurückgegeben wird.
    

> [!TIP]  
> In den meisten modernen C#-APIs ist eine explizite `Build()`-Methode häufig leichter verständlich als eine implizite Konvertierung.

---

# Typischer Fluent Builder

Ein moderner Builder sieht oft ungefähr so aus:

```csharp
public class UserBuilder
{
    private string name = "";
    private string company = "";
    private int age;
    private bool isMarried;

    public UserBuilder WithName(string name)
    {
        this.name = name;
        return this;
    }

    public UserBuilder WithCompany(string company)
    {
        this.company = company;
        return this;
    }

    public UserBuilder WithAge(int age)
    {
        this.age = age;
        return this;
    }

    public UserBuilder Married()
    {
        isMarried = true;
        return this;
    }

    public User Build()
    {
        return new User
        {
            Name = name,
            Company = company,
            Age = age,
            IsMarried = isMarried
        };
    }
}
```

Verwendung:

```csharp
User user = new UserBuilder()
    .WithName("Tom")
    .WithCompany("Microsoft")
    .WithAge(23)
    .Married()
    .Build();
```

---

# Vorteile

Ein Fluent Builder bietet insbesondere folgende Vorteile:

- bessere Lesbarkeit,
    
- verständliche Objektkonfiguration,
    
- weniger große Konstruktoren,
    
- optionale Werte lassen sich leichter behandeln,
    
- Validierung kann zentralisiert werden,
    
- komplexe Objekterstellung wird gekapselt,
    
- einzelne Schritte können klar benannt werden.
    

---

# Wann ist ein Fluent Builder sinnvoll?

Ein Builder ist besonders hilfreich, wenn:

- ein Objekt viele Eigenschaften besitzt,
    
- viele Parameter optional sind,
    
- die Objekterstellung mehrere Schritte benötigt,
    
- Validierungslogik vorhanden ist,
    
- Konstruktoren sehr lang werden,
    
- unterschiedliche Konfigurationen desselben Objekts benötigt werden.
    

Beispiel:

```
User
 ├── Name
 ├── Company
 ├── Age
 ├── Address
 ├── Phone
 ├── Email
 ├── Role
 ├── Permissions
 └── Settings
```

Hier kann ein Builder die Erstellung deutlich übersichtlicher machen.

---

# Wann ist ein Builder unnötig?

Bei sehr einfachen Objekten kann ein Builder unnötige Komplexität erzeugen.

Beispiel:

```csharp
class Point
{
    public int X { get; set; }
    public int Y { get; set; }
}
```

Hier reicht meistens:

```csharp
Point point = new Point
{
    X = 10,
    Y = 20
};
```

Ein eigener `PointBuilder` wäre wahrscheinlich übertrieben.

---

# Kurzfassung

> [!SUMMARY]  
> **Fluent Builder = Schrittweise Objekterstellung durch verkettete Methoden.**

Grundprinzip:

```
Builder
  │
  ▼
SetName()
  │
  ▼
SetCompany()
  │
  ▼
SetAge()
  │
  ▼
Build()
  │
  ▼
Fertiges Objekt
```

Entscheidend ist:

```csharp
return this;
```

Dadurch entsteht die Methodenkette:

```csharp
new UserBuilder()
    .SetName("Tom")
    .SetCompany("Microsoft")
    .SetAge(23)
    .Build();
```

### Merksatz

> **Ein Fluent Builder macht komplexe Objekterstellung lesbarer, indem die Konfiguration in klar benannte, verkettete Schritte aufgeteilt wird.**