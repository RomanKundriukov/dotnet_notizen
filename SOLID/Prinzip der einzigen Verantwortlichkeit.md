Das **Single Responsibility Principle (SRP)** lässt sich folgendermaßen formulieren:

> [!IMPORTANT]  
> Jede Komponente sollte genau **einen einzigen Grund für eine Änderung** haben.

In C# kann eine Komponente zum Beispiel eine **Klasse**, eine **Struktur** oder eine **Methode** sein.

Unter einer Verantwortlichkeit versteht man dabei eine Gruppe von Aktionen, die gemeinsam eine bestimmte Aufgabe erfüllen.

Die Grundidee lautet also:

> Eine Klasse, Struktur oder Methode sollte nur **eine klar abgegrenzte Aufgabe** erfüllen.

Die gesamte Funktionalität einer Komponente sollte zusammengehören und eine hohe **Kohäsion (high cohesion)** besitzen.

Die konkrete Anwendung des Prinzips hängt immer vom Kontext ab. Entscheidend ist die Frage, **warum sich eine Komponente ändern müsste**.

Wenn eine Komponente mehrere unterschiedliche Aufgaben übernimmt und diese Aufgaben unabhängig voneinander geändert werden können, besitzt sie mehrere Gründe für Änderungen. Genau hier ist das Single Responsibility Principle relevant.

### Beispiel: Report-Klasse

Angenommen, wir möchten eine Klasse für einen Bericht erstellen. Man soll durch die Seiten navigieren und den Bericht außerdem ausgeben beziehungsweise drucken können.

Auf den ersten Blick könnte die Klasse so aussehen:

```csharp
class Report
{
    public string Text { get; set; } = "";

    public void GoToFirstPage() =>
        Console.WriteLine("Zur ersten Seite wechseln");

    public void GoToLastPage() =>
        Console.WriteLine("Zur letzten Seite wechseln");

    public void GoToPage(int pageNumber) =>
        Console.WriteLine($"Zu Seite {pageNumber} wechseln");

    public void Print()
    {
        Console.WriteLine("Bericht drucken");
        Console.WriteLine(Text);
    }
}
```

Ein zentraler Begriff beim SRP ist **Kohäsion**. Sie beschreibt, wie eng die Bestandteile einer Komponente funktional zusammengehören.

Je stärker die zusammengehörigen Funktionen innerhalb einer Komponente gebündelt sind, desto besser entspricht sie dem Prinzip der einzigen Verantwortlichkeit.

Die ersten drei Methoden gehören alle zur **Navigation innerhalb des Berichts** und bilden damit eine funktionale Einheit.

Die Methode `Print()` hat dagegen eine andere Aufgabe: Sie kümmert sich um die **Ausgabe des Berichts**.

Was passiert, wenn wir den Bericht später:

- in der Konsole ausgeben,
    
- an einen echten Drucker senden,
    
- in eine Datei schreiben,
    
- als HTML speichern,
    
- als TXT speichern,
    
- als RTF speichern
    

möchten?

Dann müsste sich die Logik von `Print()` ändern. Die Navigationsmethoden wären davon aber wahrscheinlich überhaupt nicht betroffen.

Umgekehrt gilt dasselbe: Änderungen an der Seitennavigation sollten keinen Einfluss auf die Ausgabe oder das Drucken des Berichts haben.

Damit besitzt die Klasse `Report` zwei unterschiedliche Gründe für Änderungen:

1. Änderungen an der Navigation
    
2. Änderungen an der Ausgabe
    

Die Klasse hat also **zwei Verantwortlichkeiten**.

Eine mögliche Lösung besteht darin, diese Verantwortlichkeiten auf getrennte Klassen aufzuteilen:

```csharp
class Report
{
    public string Text { get; set; } = "";

    public void GoToFirstPage() =>
        Console.WriteLine("Zur ersten Seite wechseln");

    public void GoToLastPage() =>
        Console.WriteLine("Zur letzten Seite wechseln");

    public void GoToPage(int pageNumber) =>
        Console.WriteLine($"Zu Seite {pageNumber} wechseln");
}

// Verantwortlichkeit: Ausgabe bzw. Drucken des Berichts
class Printer
{
    public void PrintReport(Report report)
    {
        Console.WriteLine("Bericht drucken");
        Console.WriteLine(report.Text);
    }
}
```

Die Ausgabe wurde jetzt in die Klasse `Printer` ausgelagert.

`Printer` erhält über `PrintReport()` ein `Report`-Objekt und gibt dessen Text aus.

---

## Zweites Beispiel

Verantwortlichkeiten innerhalb einer Klasse werden nicht immer einfach nach Methoden getrennt.

Eine **Verantwortlichkeit** bezieht sich auf eine Aufgabe einer Komponente. Diese Komponente kann eine Klasse, aber auch nur eine einzelne Methode oder Eigenschaft sein.

Es ist also durchaus möglich, dass **eine einzige Methode mehrere Verantwortlichkeiten gleichzeitig enthält**.

Beispiel:

```csharp
class Phone
{
    public string Model { get; }
    public int Price { get; }

    public Phone(string model, int price)
    {
        Model = model;
        Price = price;
    }
}

class MobileStore
{
    List<Phone> phones = new();

    public void Process()
    {
        // Dateneingabe
        Console.WriteLine("Modell eingeben:");
        string? model = Console.ReadLine();

        Console.WriteLine("Preis eingeben:");

        // Validierung
        bool result = int.TryParse(Console.ReadLine(), out var price);

        if (result == false || price <= 0 || string.IsNullOrEmpty(model))
        {
            throw new Exception("Die eingegebenen Daten sind ungültig");
        }
        else
        {
            phones.Add(new Phone(model, price));

            // Daten in einer Datei speichern
            using (StreamWriter writer = new StreamWriter("store.txt", true))
            {
                writer.WriteLine(model);
                writer.WriteLine(price);
            }

            Console.WriteLine("Die Daten wurden erfolgreich verarbeitet");
        }
    }
}
```

Die Klasse besitzt nur eine einzige Methode `Process()`. Trotzdem übernimmt diese Methode mindestens vier verschiedene Verantwortlichkeiten:

1. Eingabe der Daten
    
2. Validierung der Daten
    
3. Erzeugung eines `Phone`-Objekts
    
4. Speicherung der Daten
    

Die Klasse weiß dadurch praktisch alles:

- wie Daten eingegeben werden,
    
- wie Daten validiert werden,
    
- wie ein Objekt erzeugt wird,
    
- wie Daten gespeichert werden.
    

Solche Klassen werden oft als **God Class** beziehungsweise **Gott-Klasse** bezeichnet, weil sie einen sehr großen Teil der gesamten Funktionalität in sich vereinen.

Das ist ein verbreitetes **Anti-Pattern**, das möglichst vermieden werden sollte.

Auch wenn der Code im Beispiel noch relativ klein ist, kann die Methode `Process()` bei späteren Erweiterungen stark anwachsen und dadurch kompliziert und schwer wartbar werden.

---

## Aufteilung der Verantwortlichkeiten

Nun teilen wir die Verantwortlichkeiten auf verschiedene Komponenten auf:

```csharp
class Phone
{
    public string Model { get; }
    public int Price { get; }

    public Phone(string model, int price)
    {
        Model = model;
        Price = price;
    }
}

class MobileStore
{
    List<Phone> phones = new List<Phone>();

    public IPhoneReader Reader { get; set; }
    public IPhoneBinder Binder { get; set; }
    public IPhoneValidator Validator { get; set; }
    public IPhoneSaver Saver { get; set; }

    public MobileStore(
        IPhoneReader reader,
        IPhoneBinder binder,
        IPhoneValidator validator,
        IPhoneSaver saver)
    {
        Reader = reader;
        Binder = binder;
        Validator = validator;
        Saver = saver;
    }

    public void Process()
    {
        string?[] data = Reader.GetInputData();
        Phone phone = Binder.CreatePhone(data);

        if (Validator.IsValid(phone))
        {
            phones.Add(phone);
            Saver.Save(phone, "store.txt");
            Console.WriteLine("Die Daten wurden erfolgreich verarbeitet");
        }
        else
        {
            Console.WriteLine("Ungültige Daten");
        }
    }
}

interface IPhoneReader
{
    string?[] GetInputData();
}

class ConsolePhoneReader : IPhoneReader
{
    public string?[] GetInputData()
    {
        Console.WriteLine("Modell eingeben:");
        string? model = Console.ReadLine();

        Console.WriteLine("Preis eingeben:");
        string? price = Console.ReadLine();

        return new string?[] { model, price };
    }
}

interface IPhoneBinder
{
    Phone CreatePhone(string?[] data);
}

class GeneralPhoneBinder : IPhoneBinder
{
    public Phone CreatePhone(string?[] data)
    {
        if (data is { Length: 2 }
            && data[0] is string model
            && model.Length > 0
            && int.TryParse(data[1], out var price))
        {
            return new Phone(model, price);
        }

        throw new Exception(
            "Fehler beim Erstellen des Phone-Modells. Ungültige Daten.");
    }
}

interface IPhoneValidator
{
    bool IsValid(Phone phone);
}

class GeneralPhoneValidator : IPhoneValidator
{
    public bool IsValid(Phone phone) =>
        !string.IsNullOrEmpty(phone.Model) && phone.Price > 0;
}

interface IPhoneSaver
{
    void Save(Phone phone, string fileName);
}

class TextPhoneSaver : IPhoneSaver
{
    public void Save(Phone phone, string fileName)
    {
        using StreamWriter writer = new StreamWriter(fileName, true);

        writer.WriteLine(phone.Model);
        writer.WriteLine(phone.Price);
    }
}
```

Eine mögliche Verwendung sieht so aus:

```csharp
MobileStore store = new MobileStore(
    new ConsolePhoneReader(),
    new GeneralPhoneBinder(),
    new GeneralPhoneValidator(),
    new TextPhoneSaver());

store.Process();
```

Jetzt besitzt jede Verantwortlichkeit eine eigene Schnittstelle.

Die konkreten Implementierungen werden über diese Interfaces in die Zielklasse eingebunden.

Dadurch ist zwar mehr Code entstanden und die Struktur wirkt zunächst komplizierter.

Bei einer sehr kleinen Methode, die sich möglicherweise nie ändern wird, kann eine solche Aufteilung sogar übertrieben erscheinen.

Bei späteren Änderungen hat diese Struktur jedoch einen großen Vorteil:

> Neue Funktionalität kann leichter ergänzt werden, ohne bereits vorhandenen Code stark verändern zu müssen.

Die einzelnen Bestandteile der ursprünglichen `Process()`-Methode befinden sich nun in separaten Klassen und können unabhängig voneinander geändert werden.

---

## Häufige Verstöße gegen SRP

Ein häufiger Verstoß gegen das Single Responsibility Principle besteht darin, Funktionalität aus unterschiedlichen Ebenen in derselben Klasse zu vermischen.

Beispielsweise könnte eine Klasse gleichzeitig:

- Berechnungen durchführen,
    
- Ergebnisse im Benutzerinterface darstellen.
    

Damit würde sie **Businesslogik** und **UI-Logik** miteinander vermischen.

Ein anderes Beispiel wäre eine Klasse, die gleichzeitig:

- Daten speichert oder lädt,
    
- Berechnungen mit diesen Daten durchführt.
    

Auch diese Aufgaben sollten normalerweise getrennt werden.

Eine Klasse sollte möglichst eine klar definierte Aufgabe übernehmen, zum Beispiel:

- Businesslogik,
    
- Berechnungen,
    
- Datenzugriff,
    
- Benutzeroberfläche.
    

Ein weiterer typischer Verstoß ist das Vorhandensein von Funktionen innerhalb einer Klasse oder Methode, die funktional überhaupt nicht zusammengehören.

---

## Häufig ausgelagerte Verantwortlichkeiten

Bestimmte Aufgaben werden besonders häufig in eigene Komponenten ausgelagert:

- Datenzugriffs- und Speicherlogik
    
- Validierung
    
- Benachrichtigungen für Benutzer
    
- Fehlerbehandlung
    
- Logging
    
- Auswahl einer Klasse oder Erzeugung eines Objekts
    
- Formatierung
    
- Parsing
    
- Daten-Mapping
    

---

## Kurzfassung

> [!SUMMARY]  
> **SRP = Eine Komponente sollte nur einen Grund haben, geändert zu werden.**

Eine Klasse sollte nicht gleichzeitig mehrere voneinander unabhängige Aufgaben übernehmen.

Stattdessen werden Verantwortlichkeiten getrennt, zum Beispiel:

```
Eingabe        → Reader
Erzeugung      → Binder
Validierung    → Validator
Speicherung    → Saver
Koordination   → MobileStore
```

Dadurch werden die einzelnen Teile des Programms:

- leichter verständlich,
    
- einfacher testbar,
    
- besser austauschbar,
    
- leichter erweiterbar,
    
- unabhängiger voneinander.