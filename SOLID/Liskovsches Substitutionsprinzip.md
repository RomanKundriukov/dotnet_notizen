Das **Liskov Substitution Principle (LSP)** beschreibt Regeln für den sinnvollen Aufbau von Vererbungshierarchien.

Die ursprüngliche Definition von **Barbara Liskov** aus dem Jahr 1988 lautet sinngemäß:

> [!QUOTE]  
> Wenn für jedes Objekt `o1` vom Typ `S` ein Objekt `o2` vom Typ `T` existiert, sodass sich das Verhalten eines Programms `P`, das mit `T` arbeitet, nicht verändert, wenn `o2` durch `o1` ersetzt wird, dann ist `S` ein Untertyp von `T`.

Einfacher formuliert:

> [!IMPORTANT]  
> Ein Objekt eines abgeleiteten Typs muss überall dort verwendet werden können, wo ein Objekt des Basistyps erwartet wird, **ohne dass sich das korrekte Verhalten des Programms ändert**.

Oder noch kürzer:

> Ein Untertyp muss seinen Basistyp vollständig ersetzen können.

Das Liskov Substitution Principle hilft dabei:

- sinnvolle Vererbungshierarchien zu entwerfen,
    
- Verantwortlichkeiten von Basis- und Unterklassen korrekt zu definieren,
    
- Probleme bei Vererbung und Polymorphie zu vermeiden.
    

---

## Klassisches Beispiel: Rectangle und Square

Ein typisches Problem lässt sich mit `Rectangle` und `Square` zeigen.

```csharp
class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }

    public int GetArea()
    {
        return Width * Height;
    }
}

class Square : Rectangle
{
    public override int Width
    {
        get
        {
            return base.Width;
        }

        set
        {
            base.Width = value;
            base.Height = value;
        }
    }

    public override int Height
    {
        get
        {
            return base.Height;
        }

        set
        {
            base.Height = value;
            base.Width = value;
        }
    }
}
```

Mathematisch ist ein Quadrat ein spezieller Fall eines Rechtecks:

- vier Seiten,
    
- vier rechte Winkel,
    
- Breite und Höhe sind beim Quadrat immer gleich.
    

Deshalb wird beim `Square` beim Setzen einer Eigenschaft gleichzeitig auch die andere geändert:

```csharp
set
{
    base.Height = value;
    base.Width = value;
}
```

Auf den ersten Blick scheint diese Vererbung korrekt zu sein.

Das Problem zeigt sich erst bei der Verwendung.

---

## Problem bei der Ersetzung

```csharp
class Program
{
    static void Main(string[] args)
    {
        Rectangle rect = new Square();

        TestRectangleArea(rect);

        Console.Read();
    }

    public static void TestRectangleArea(Rectangle rect)
    {
        rect.Height = 5;
        rect.Width = 10;

        if (rect.GetArea() != 50)
        {
            throw new Exception("Ungültige Fläche!");
        }
    }
}
```

Für ein normales Rechteck ist die Logik korrekt:

```csharp
Height = 5
Width  = 10

Fläche = 5 × 10 = 50
```

Bei `Square` passiert jedoch etwas anderes.

Nach:

```csharp
rect.Height = 5;
```

gilt:

```csharp
Width  = 5
Height = 5
```

Danach wird ausgeführt:

```csharp
rect.Width = 10;
```

Dadurch setzt `Square` gleichzeitig auch die Höhe auf `10`.

Das Ergebnis lautet:

```csharp
Width  = 10
Height = 10

Fläche = 100
```

Die Methode erwartet aber:

```csharp
Fläche = 50
```

Damit verhält sich `Square` **nicht so wie ein** `**Rectangle**`, obwohl es davon erbt.

> [!WARNING]  
> Genau hier wird das Liskov Substitution Principle verletzt.

---

## Schlechte Lösung: Typprüfung

Manchmal versucht man das Problem mit einer Typprüfung zu umgehen:

```csharp
public static void TestRectangleArea(Rectangle rect)
{
    if (rect is Square)
    {
        rect.Height = 5;

        if (rect.GetArea() != 25)
        {
            throw new Exception("Ungültige Fläche!");
        }
    }
    else if (rect is Rectangle)
    {
        rect.Height = 5;
        rect.Width = 10;

        if (rect.GetArea() != 50)
        {
            throw new Exception("Ungültige Fläche!");
        }
    }
}
```

Diese Lösung beseitigt das eigentliche Architekturproblem nicht.

Im Gegenteil: Die Typprüfung zeigt, dass die Vererbungshierarchie problematisch ist.

Die aufrufende Methode muss plötzlich wissen:

```
Ist das wirklich ein Rectangle?
Oder ist es ein Square?
```

Damit kann der Untertyp den Basistyp nicht mehr transparent ersetzen.

> [!NOTE]  
> Wenn aufrufender Code ständig mit `is`, `as` oder konkreten Typprüfungen unterscheiden muss, kann das ein Hinweis auf eine falsche Vererbungshierarchie sein.

Das eigentliche Problem lautet:

> `Square` verhält sich nicht vollständig wie `Rectangle`.

Deshalb sollte man überlegen, ob `Square` überhaupt von `Rectangle` erben sollte.

---

# Regeln des Liskov Substitution Principle

Für LSP spielen sogenannte **Verträge (Contracts)** eine wichtige Rolle.

Ein Vertrag beschreibt Regeln und Erwartungen, die ein Basistyp vorgibt und die von Untertypen eingehalten werden müssen.

Dazu gehören insbesondere:

1. Preconditions
    
2. Postconditions
    
3. Invariants
    

---

## 1. Preconditions dürfen nicht verschärft werden

**Preconditions** sind Bedingungen, die vor der Ausführung einer Methode erfüllt sein müssen.

Ein Untertyp darf keine strengeren Voraussetzungen verlangen als der Basistyp.

> [!IMPORTANT]  
> Ein Untertyp darf die Preconditions des Basistyps **nicht verschärfen**.

### Beispiel für eine Precondition

```csharp
public virtual void SetCapital(int money)
{
    if (money < 0)
    {
        throw new Exception(
            "Es kann kein Betrag kleiner als 0 eingezahlt werden.");
    }

    Capital = money;
}
```

Die Bedingung:

```csharp
money >= 0
```

ist eine Precondition.

Wenn sie nicht erfüllt ist, kann die Methode nicht erfolgreich ausgeführt werden.

---

## Was kann eine Precondition sein?

Preconditions beziehen sich auf Zustände, die der aufrufende Code beeinflussen kann, zum Beispiel:

- Methodenparameter
    
- öffentliche Properties
    
- öffentliche Felder
    

Ein privates Feld ist normalerweise keine Precondition für den Aufrufer, da dieser es nicht selbst setzen kann.

Beispiel:

```csharp
private bool isValid = false;

public virtual void SetCapital(int money)
{
    if (isValid == false)
    {
        throw new Exception("Die Validierung war nicht erfolgreich.");
    }

    Capital = money;
}
```

Hier kann der Aufrufer `isValid` nicht beeinflussen.

Deshalb handelt es sich aus Sicht des externen Aufrufers nicht um dieselbe Art von Vorbedingung.

---

## Verletzung durch verschärfte Preconditions

Angenommen, wir haben einen normalen Account:

```csharp
class Account
{
    public int Capital { get; protected set; }

    public virtual void SetCapital(int money)
    {
        if (money < 0)
        {
            throw new Exception(
                "Es kann kein Betrag kleiner als 0 eingezahlt werden.");
        }

        Capital = money;
    }
}
```

Der Vertrag lautet:

```csharp
money >= 0
```

Jetzt erstellen wir einen Untertyp:

```csharp
class MicroAccount : Account
{
    public override void SetCapital(int money)
    {
        if (money < 0)
        {
            throw new Exception(
                "Es kann kein Betrag kleiner als 0 eingezahlt werden.");
        }

        if (money > 100)
        {
            throw new Exception(
                "Es kann kein Betrag größer als 100 eingezahlt werden.");
        }

        Capital = money;
    }
}
```

`MicroAccount` akzeptiert nur:

```csharp
0 <= money <= 100
```

Der Basistyp akzeptiert dagegen:

```csharp
money >= 0
```

Damit hat der Untertyp eine zusätzliche Bedingung eingeführt.

Die Precondition wurde also **verschärft**.

---

## Praktisches Problem

```csharp
class Program
{
    static void Main(string[] args)
    {
        Account acc = new MicroAccount();

        InitializeAccount(acc);

        Console.Read();
    }

    public static void InitializeAccount(Account account)
    {
        account.SetCapital(200);

        Console.WriteLine(account.Capital);
    }
}
```

Aus Sicht von `Account` ist folgender Aufruf erlaubt:

```csharp
account.SetCapital(200);
```

`200` ist größer als `0`.

Bei einem `MicroAccount` entsteht jedoch eine Exception.

Damit kann `MicroAccount` den Basistyp `Account` nicht vollständig ersetzen.

> [!WARNING]  
> Das Liskov Substitution Principle wird verletzt.

---

# 2. Postconditions dürfen nicht abgeschwächt werden

**Postconditions** beschreiben Bedingungen, die nach erfolgreicher Ausführung einer Methode gelten müssen.

> [!IMPORTANT]  
> Ein Untertyp darf Postconditions des Basistyps **nicht abschwächen**.

### Einfaches Beispiel

```csharp
public static float GetMedium(float[] numbers)
{
    if (numbers.Length == 0)
    {
        throw new Exception("Die Länge des Arrays ist 0.");
    }

    float result = numbers.Sum() / numbers.Length;

    if (result < 0)
    {
        throw new Exception("Das Ergebnis ist kleiner als 0.");
    }

    return result;
}
```

Die erste Prüfung ist eine Precondition:

```csharp
numbers.Length > 0
```

Die zweite Prüfung beschreibt eine Bedingung für das Ergebnis:

```csharp
result >= 0
```

Sie fungiert damit als Postcondition.

---

## Verletzung durch abgeschwächte Postconditions

Betrachten wir einen Account:

```csharp
class Account
{
    public virtual decimal GetInterest(
        decimal sum,
        int month,
        int rate)
    {
        // Precondition
        if (sum < 0 ||
            month > 12 ||
            month < 1 ||
            rate < 0)
        {
            throw new Exception("Ungültige Daten.");
        }

        decimal result = sum;

        for (int i = 0; i < month; i++)
        {
            result += result * rate / 100;
        }

        // Postcondition:
        // Ab 1000 wird ein Bonus von 100 hinzugefügt.
        if (sum >= 1000)
        {
            result += 100;
        }

        return result;
    }
}
```

Nun überschreibt `MicroAccount` die Methode:

```csharp
class MicroAccount : Account
{
    public override decimal GetInterest(
        decimal sum,
        int month,
        int rate)
    {
        if (sum < 0 ||
            month > 12 ||
            month < 1 ||
            rate < 0)
        {
            throw new Exception("Ungültige Daten.");
        }

        decimal result = sum;

        for (int i = 0; i < month; i++)
        {
            result += result * rate / 100;
        }

        return result;
    }
}
```

Der Basistyp garantiert bei:

```csharp
sum >= 1000
```

einen Bonus von `100`.

Der Untertyp garantiert diesen Bonus nicht mehr.

Damit wurde die Postcondition abgeschwächt.

---

## Praktisches Problem

```csharp
class Program
{
    public static void CalculateInterest(Account account)
    {
        decimal sum = account.GetInterest(
            1000,
            1,
            10);

        // Erwartet:
        // 1000 + 100 + 100 Bonus = 1200
        if (sum != 1200)
        {
            throw new Exception(
                "Unerwartetes Ergebnis bei der Berechnung.");
        }
    }

    static void Main(string[] args)
    {
        Account acc = new MicroAccount();

        // Ergebnis: 1100 statt 1200
        CalculateInterest(acc);

        Console.Read();
    }
}
```

Aus Sicht des Basistyps erwarten wir:

```
1000
+ 10 % Zinsen = 100
+ Bonus        = 100
-------------------
1200
```

`MicroAccount` liefert jedoch:

```
1100
```

Damit erfüllt der Untertyp nicht mehr die Erwartungen des Basistyps.

> [!WARNING]  
> Auch hier wird LSP verletzt.

---

# 3. Invarianten müssen erhalten bleiben

**Invarianten** sind Bedingungen, die während der gesamten Lebensdauer eines Objekts gültig bleiben sollen.

> [!IMPORTANT]  
> Ein Untertyp muss die Invarianten seines Basistyps erhalten.

Beispiel:

```csharp
class User
{
    protected int age;

    public User(int age)
    {
        if (age < 0)
        {
            throw new Exception(
                "Das Alter darf nicht kleiner als 0 sein.");
        }

        this.age = age;
    }

    public int Age
    {
        get
        {
            return age;
        }

        set
        {
            if (value < 0)
            {
                throw new Exception(
                    "Das Alter darf nicht kleiner als 0 sein.");
            }

            age = value;
        }
    }
}
```

Die Invariante lautet:

```csharp
Age >= 0
```

Sowohl der Konstruktor als auch die Property sorgen dafür, dass diese Bedingung immer erfüllt bleibt.

Während der gesamten Lebensdauer eines gültigen `User`-Objekts gilt also:

```
Alter ist niemals negativ.
```

---

## Verletzung einer Invariante

Betrachten wir einen Account:

```csharp
class Account
{
    protected int capital;

    public Account(int sum)
    {
        if (sum < 100)
        {
            throw new Exception("Ungültiger Betrag.");
        }

        capital = sum;
    }

    public virtual int Capital
    {
        get
        {
            return capital;
        }

        set
        {
            if (value < 100)
            {
                throw new Exception("Ungültiger Betrag.");
            }

            capital = value;
        }
    }
}
```

Die Invariante lautet:

```csharp
Capital >= 100
```

Der Basistyp garantiert diese Regel sowohl im Konstruktor als auch beim Setzen der Property.

Jetzt erstellen wir einen Untertyp:

```csharp
class MicroAccount : Account
{
    public MicroAccount(int sum)
        : base(sum)
    {
    }

    public override int Capital
    {
        get
        {
            return capital;
        }

        set
        {
            capital = value;
        }
    }
}
```

Der Untertyp entfernt die Prüfung:

```csharp
if (value < 100)
```

Jetzt wäre möglich:

```csharp
Account account = new MicroAccount(100);

account.Capital = 10;
```

Der Basistyp verspricht eigentlich:

```csharp
Capital >= 100
```

Der Untertyp lässt aber zu:

```csharp
Capital = 10
```

Damit wurde die Invariante des Basistyps verletzt.

---

# Wie kann man solche Probleme lösen?

In vielen Fällen ist die bessere Lösung nicht:

```
MicroAccount
     │
     ▼
  Account
```

sondern eine gemeinsame Abstraktion:

```
        AccountBase
        /         \
       /           \
      ▼             ▼
 Account      MicroAccount
```

Beide Typen erben dann nur die Funktionalität, die tatsächlich für beide gültig ist.

Dadurch muss keiner der Typen einen Vertrag übernehmen, den er später nicht erfüllen kann.

Dasselbe Prinzip kann auch beim Beispiel `Rectangle` und `Square` angewendet werden.

Statt eine falsche direkte Vererbung zu erzwingen:

```
Rectangle
    │
    ▼
  Square
```

kann man eine gemeinsame Abstraktion verwenden:

```
       Shape
      /     \
     ▼       ▼
Rectangle  Square
```

---

# Woran erkennt man mögliche LSP-Verletzungen?

Typische Warnsignale sind:

- Unterklassen werfen Exceptions bei Eingaben, die der Basistyp akzeptiert.
    
- Eine überschriebene Methode liefert weniger Garantien als die Basismethode.
    
- Ein Untertyp verletzt Zustandsregeln des Basistyps.
    
- Aufrufender Code muss mit `is`, `as` oder konkreten Typprüfungen unterscheiden.
    
- Methoden funktionieren mit dem Basistyp, aber nicht mit bestimmten Untertypen.
    
- Unterklassen deaktivieren oder überschreiben Verhalten des Basistyps auf unerwartete Weise.
    
- Eine Unterklasse kann funktional weniger als die Basisklasse.
    

---

# Zusammenhang mit Polymorphie

Polymorphie funktioniert nur dann zuverlässig, wenn Untertypen die Erwartungen an den Basistyp erfüllen.

Beispiel:

```csharp
void Process(Account account)
{
    // Diese Methode sollte mit jedem gültigen
    // Untertyp von Account korrekt funktionieren.
}
```

Wenn die Methode für:

```csharp
new Account()
```

funktioniert, aber bei:

```csharp
new MicroAccount()
```

unerwartet fehlschlägt, obwohl `MicroAccount : Account` gilt, ist die Vererbung möglicherweise falsch modelliert.

> [!NOTE]  
> Vererbung bedeutet nicht nur:
> 
> **„Ist fachlich eine Art von …“**
> 
> sondern auch:
> 
> **„Kann sich überall so verhalten wie …“**

---

# Kurzfassung

> [!SUMMARY]  
> **LSP = Ein Untertyp muss seinen Basistyp ersetzen können, ohne das korrekte Verhalten des Programms zu verändern.**

Die wichtigsten Regeln:

```
Basistyp-Vertrag
      │
      ├── Preconditions
      │      dürfen nicht verschärft werden
      │
      ├── Postconditions
      │      dürfen nicht abgeschwächt werden
      │
      └── Invarianten
             müssen erhalten bleiben
```

### Merksatz

> Wenn eine Methode mit einem Basistyp arbeitet, sollte sie auch mit **jedem gültigen Untertyp** funktionieren, ohne Sonderbehandlungen zu benötigen.

Oder noch einfacher:

> **Ein Kindtyp darf die Erwartungen an seinen Elterntyp nicht brechen.**