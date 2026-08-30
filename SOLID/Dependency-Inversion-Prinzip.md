Das **Dependency Inversion Principle (DIP)** hilft dabei, **locker gekoppelte Komponenten** zu entwickeln, die einfacher zu testen, zu verändern und zu erweitern sind.

Das Prinzip lässt sich folgendermaßen formulieren:

> [!IMPORTANT]  
> **Module auf hoher Ebene sollen nicht von Modulen auf niedriger Ebene abhängen.**  
> Beide sollen von **Abstraktionen** abhängen.

Außerdem gilt:

> [!IMPORTANT]  
> **Abstraktionen sollen nicht von Details abhängen.**  
> **Details sollen von Abstraktionen abhängen.**

---

# Einfaches Beispiel

Betrachten wir zunächst folgende Implementierung:

```csharp
class Book
{
    public string Text { get; set; }

    public ConsolePrinter Printer { get; set; }

    public void Print()
    {
        Printer.Print(Text);
    }
}

class ConsolePrinter
{
    public void Print(string text)
    {
        Console.WriteLine(text);
    }
}
```

Die Klasse `Book` repräsentiert ein Buch.

Für die Ausgabe verwendet sie direkt die konkrete Klasse:

```csharp
ConsolePrinter
```

Dadurch hängt `Book` fest von `ConsolePrinter` ab.

Die Abhängigkeit sieht so aus:

```
Book
 │
 ▼
ConsolePrinter
```

Das bedeutet:

> `Book` kann nur mit genau dieser konkreten Drucker-Implementierung arbeiten.

---

# Was ist daran problematisch?

Mit dieser Struktur ist festgelegt, dass ein Buch nur über die Konsole ausgegeben werden kann.

Andere Varianten wären nur schwer integrierbar, zum Beispiel:

- Ausgabe in eine Datei
    
- Ausgabe als HTML
    
- Ausgabe auf einem echten Drucker
    
- Ausgabe in einer grafischen Benutzeroberfläche
    
- Ausgabe als PDF
    

Wenn wir eine andere Ausgabeart verwenden möchten, müssten wir die Klasse `Book` verändern.

Damit hängt eine höherwertige fachliche Komponente direkt von einem technischen Detail ab.

```
Fachlogik
Book
 │
 │ direkte Abhängigkeit
 ▼
Technisches Detail
ConsolePrinter
```

> [!WARNING]  
> Die Abstraktion „Buch ausgeben“ ist hier nicht von der konkreten technischen Implementierung getrennt.

Das verletzt das Dependency Inversion Principle.

---

# Lösung: Abstraktion einführen

Wir führen ein Interface ein:

```csharp
interface IPrinter
{
    void Print(string text);
}
```

Die Klasse `Book` hängt jetzt nicht mehr von `ConsolePrinter`, sondern von `IPrinter` ab.

```csharp
class Book
{
    public string Text { get; set; }

    public IPrinter Printer { get; set; }

    public Book(IPrinter printer)
    {
        Printer = printer;
    }

    public void Print()
    {
        Printer.Print(Text);
    }
}
```

Nun können verschiedene konkrete Implementierungen von `IPrinter` existieren.

---

## ConsolePrinter

```csharp
class ConsolePrinter : IPrinter
{
    public void Print(string text)
    {
        Console.WriteLine("Ausgabe in der Konsole");
        Console.WriteLine(text);
    }
}
```

---

## HtmlPrinter

```csharp
class HtmlPrinter : IPrinter
{
    public void Print(string text)
    {
        Console.WriteLine("Ausgabe als HTML");
        Console.WriteLine(text);
    }
}
```

---

# Neue Abhängigkeitsstruktur

Vorher:

```
Book
 │
 ▼
ConsolePrinter
```

Nachher:

```
             IPrinter
             ▲      ▲
             │      │
             │      │
           Book   ConsolePrinter
                    ▲
                    │
                HtmlPrinter
```

Genauer:

```
Book ───────────► IPrinter
ConsolePrinter ─► IPrinter
HtmlPrinter ─────► IPrinter
```

Damit hängen sowohl die höherwertige Klasse `Book` als auch die technischen Implementierungen von derselben Abstraktion ab.

> [!IMPORTANT]  
> `Book` kennt keine konkrete Druckerklasse mehr.

---

# Verwendung

```csharp
Book book = new Book(
    new ConsolePrinter());

book.Text = "SOLID-Prinzipien";

book.Print();
```

Später kann die Implementierung ausgetauscht werden:

```csharp
book.Printer = new HtmlPrinter();

book.Print();
```

Die Klasse `Book` muss dafür nicht verändert werden.

---

# Warum heißt es „Dependency Inversion“?

Ohne DIP hängt die Fachlogik direkt von technischen Details ab:

```
High-Level-Modul
Book
 │
 ▼
Low-Level-Modul
ConsolePrinter
```

Die Abhängigkeit zeigt also nach unten.

Mit DIP wird eine Abstraktion dazwischen eingeführt:

```
          IPrinter
          ▲      ▲
          │      │
        Book   ConsolePrinter
```

Jetzt hängen beide Seiten von der Abstraktion ab.

Die ursprüngliche direkte Abhängigkeit wurde damit **umgekehrt beziehungsweise entkoppelt**.

---

# High-Level- und Low-Level-Module

## High-Level-Modul

Ein High-Level-Modul enthält typischerweise fachliche Logik.

In unserem Beispiel:

```
Book
```

Die Klasse beschreibt:

> Ein Buch kann ausgegeben werden.

Sie sollte aber nicht festlegen, **wie technisch** diese Ausgabe erfolgt.

---

## Low-Level-Modul

Ein Low-Level-Modul enthält konkrete technische Details.

Zum Beispiel:

```
ConsolePrinter
HtmlPrinter
FilePrinter
PdfPrinter
```

Diese Klassen entscheiden, **wie genau** die Ausgabe technisch umgesetzt wird.

---

# Weitere Implementierungen

Da `Book` nur `IPrinter` kennt, können beliebig viele Implementierungen ergänzt werden.

Zum Beispiel:

```csharp
class FilePrinter : IPrinter
{
    public void Print(string text)
    {
        File.WriteAllText(
            "book.txt",
            text);
    }
}
```

Oder:

```csharp
class PdfPrinter : IPrinter
{
    public void Print(string text)
    {
        Console.WriteLine(
            "Das Buch wird als PDF ausgegeben.");
    }
}
```

`Book` bleibt unverändert.

---

# Dependency Injection

Das Beispiel verwendet gleichzeitig **Dependency Injection**.

Die Abhängigkeit wird von außen über den Konstruktor übergeben:

```csharp
public Book(IPrinter printer)
{
    Printer = printer;
}
```

Das nennt man:

```
Constructor Injection
```

Die Klasse erzeugt ihre Abhängigkeit also nicht selbst.

Schlecht wäre zum Beispiel:

```csharp
class Book
{
    private readonly ConsolePrinter printer =
        new ConsolePrinter();
}
```

Denn dadurch entscheidet `Book` selbst, welche konkrete Implementierung verwendet wird.

Besser:

```csharp
class Book
{
    private readonly IPrinter printer;

    public Book(IPrinter printer)
    {
        this.printer = printer;
    }
}
```

---

# DIP und Dependency Injection sind nicht dasselbe

Diese Begriffe werden häufig verwechselt.

## Dependency Inversion Principle

DIP ist ein **Entwurfsprinzip**:

> Abhängigkeiten sollen auf Abstraktionen zeigen.

## Dependency Injection

Dependency Injection ist eine **Technik**, mit der Abhängigkeiten von außen bereitgestellt werden.

Beispiel:

```csharp
new Book(
    new ConsolePrinter());
```

> [!NOTE]  
> Dependency Injection ist eine sehr häufige Möglichkeit, das Dependency Inversion Principle praktisch umzusetzen.

---

# Vorteil beim Testen

Durch die Abhängigkeit von einem Interface kann beim Testen eine eigene Testimplementierung verwendet werden.

Zum Beispiel:

```csharp
class FakePrinter : IPrinter
{
    public string? PrintedText { get; private set; }

    public void Print(string text)
    {
        PrintedText = text;
    }
}
```

Test:

```csharp
FakePrinter printer = new FakePrinter();

Book book = new Book(printer)
{
    Text = "Test"
};

book.Print();

Console.WriteLine(
    printer.PrintedText);
```

Jetzt muss der Test keine echte Konsole, Datei oder andere technische Ressource verwenden.

Das macht die Klasse leichter testbar.

---

# Typische Verletzung

Ein typisches Problem sieht so aus:

```csharp
class OrderService
{
    private readonly SqlServerRepository repository =
        new SqlServerRepository();

    public void SaveOrder()
    {
        repository.Save();
    }
}
```

Hier hängt die Businesslogik direkt von:

```
SqlServerRepository
```

ab.

Besser:

```csharp
interface IOrderRepository
{
    void Save();
}
```

Dann:

```csharp
class OrderService
{
    private readonly IOrderRepository repository;

    public OrderService(
        IOrderRepository repository)
    {
        this.repository = repository;
    }

    public void SaveOrder()
    {
        repository.Save();
    }
}
```

Konkrete Implementierung:

```csharp
class SqlServerRepository : IOrderRepository
{
    public void Save()
    {
        Console.WriteLine(
            "Daten werden in SQL Server gespeichert.");
    }
}
```

Abhängigkeiten:

```
OrderService ───────► IOrderRepository
SqlServerRepository ─► IOrderRepository
```

---

# Typische Anwendungsfälle

DIP wird häufig verwendet bei:

- Datenbankzugriff
    
- Logging
    
- E-Mail-Versand
    
- Dateisystem
    
- HTTP-Clients
    
- Druckfunktionen
    
- Benachrichtigungen
    
- externen APIs
    
- Caching
    
- Authentifizierung
    
- Repository-Klassen
    

Beispiel:

```
Service
 │
 ▼
Interface
 ▲
 │
Concrete Implementation
```

---

# Vorteile

Das Dependency Inversion Principle bringt mehrere Vorteile:

- geringere Kopplung
    
- bessere Testbarkeit
    
- leichter austauschbare Implementierungen
    
- bessere Erweiterbarkeit
    
- einfachere Wartung
    
- weniger Abhängigkeit von technischen Details
    

---

# Zusammenhang mit anderen SOLID-Prinzipien

DIP arbeitet häufig mit anderen SOLID-Prinzipien zusammen.

## OCP

Durch Abstraktionen können neue Implementierungen ergänzt werden, ohne bestehenden Code zu verändern.

```
IPrinter
 ├── ConsolePrinter
 ├── HtmlPrinter
 └── PdfPrinter
```

Das unterstützt das **Open-Closed Principle**.

---

## ISP

Interfaces sollten klein und zielgerichtet sein.

Ein Interface wie:

```csharp
interface IPrinter
{
    void Print(string text);
}
```

ist besser als ein riesiges Interface mit vielen nicht zusammengehörenden Funktionen.

Das unterstützt das **Interface Segregation Principle**.

---

## LSP

Jede Implementierung von `IPrinter` sollte sich so verhalten, dass sie dort eingesetzt werden kann, wo `IPrinter` erwartet wird.

Das unterstützt das **Liskov Substitution Principle**.

---

# Kurzfassung

> [!SUMMARY]  
> **DIP = Nicht von konkreten Klassen abhängen, sondern von Abstraktionen.**

Schlecht:

```
Book
 │
 ▼
ConsolePrinter
```

Besser:

```
          IPrinter
          ▲      ▲
          │      │
        Book   ConsolePrinter
```

Die zentrale Idee lautet:

```
High-Level-Modul
       │
       ▼
   Abstraktion
       ▲
       │
Low-Level-Modul
```

### Merksatz

> **Businesslogik soll nicht von technischen Details abhängen.**

Oder noch einfacher:

> **Programmiere gegen ein Interface, nicht gegen eine konkrete Implementierung.**