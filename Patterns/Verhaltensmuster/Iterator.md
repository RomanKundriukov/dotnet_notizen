Das **Iterator Pattern** ist ein Verhaltensmuster, das einen einheitlichen Zugriff auf die Elemente einer Sammlung ermöglicht, **ohne die interne Struktur dieser Sammlung offenzulegen**.

> [!info] Grundidee  
> Der Client soll eine Sammlung Element für Element durchlaufen können, ohne wissen zu müssen, ob die Daten intern beispielsweise in einem Array, einer Liste oder einer anderen Datenstruktur gespeichert sind.

Vereinfacht:

```text
Collection
    │
    ▼
 Iterator
    │
    ├── Element 1
    ├── Element 2
    ├── Element 3
    └── ...
```

---

# Iterator in C#

In C# begegnet man dem Iterator Pattern ständig.

Zum Beispiel:

```csharp
foreach (string name in names)
{
    Console.WriteLine(name);
}
```

Dabei muss `foreach` nicht wissen, wie `names` intern aufgebaut ist.

Es interessiert nur:

```text
Gibt es ein nächstes Element?
        │
        ▼
Welches Element ist aktuell?
```

In .NET wird dafür hauptsächlich mit:

```csharp
IEnumerable<T>
IEnumerator<T>
```

gearbeitet.

---

# Wann sollte man Iterator verwenden?

Das Iterator Pattern eignet sich besonders:

- wenn eine Sammlung durchlaufen werden soll, ohne ihre interne Struktur offenzulegen;
    
- wenn verschiedene Sammlungen über dieselbe Schnittstelle durchlaufen werden sollen;
    
- wenn mehrere unterschiedliche Traversierungsstrategien benötigt werden;
    
- wenn die Logik zur Navigation durch eine Sammlung von der Sammlung selbst getrennt werden soll;
    
- wenn ein Client nicht wissen soll, ob intern ein Array, eine Liste oder eine andere Datenstruktur verwendet wird.
    

---

# Grundstruktur

Die klassische Struktur sieht ungefähr so aus:

```text
                Aggregate
                    │
                    │ CreateIterator()
                    ▼
                 Iterator
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
       Element1  Element2  Element3
```

Dabei besitzt die Sammlung die Daten.

Der Iterator besitzt dagegen den Zustand des aktuellen Durchlaufs.

---

# Teilnehmer des Patterns

## Iterator

Der Iterator definiert, **wie durch eine Sammlung navigiert wird**.

Klassisch beispielsweise:

```csharp
public interface IIterator<T>
{
    bool HasNext();

    T Next();
}
```

---

## Aggregate

Das Aggregate repräsentiert die Sammlung.

Es kann einen Iterator erzeugen:

```csharp
public interface IAggregate<T>
{
    IIterator<T> CreateIterator();
}
```

---

## ConcreteIterator

Der konkrete Iterator implementiert die Navigation.

Er speichert beispielsweise:

```csharp
private int _index;
```

und weiß dadurch, welches Element aktuell verarbeitet wird.

---

## ConcreteAggregate

Das konkrete Aggregate enthält die eigentlichen Daten.

Zum Beispiel:

```csharp
private readonly Book[] _books;
```

---

## Client

Der Client verwendet nur den Iterator.

Er muss die interne Datenstruktur nicht kennen.

---

# Klassische Implementierung

Zunächst definieren wir einen Iterator:

```csharp
// Gemeinsamer Vertrag für Iteratoren.
public interface IIterator<T>
{
    // Prüft, ob noch ein weiteres Element vorhanden ist.
    bool HasNext();

    // Liefert das nächste Element.
    T Next();
}
```

Dann eine Sammlung:

```csharp
// Sammlung, die einen Iterator erzeugen kann.
public interface IAggregate<T>
{
    IIterator<T> CreateIterator();
}
```

---

# Beispiel: Bücher und Bibliothek

Angenommen, wir haben eine Bibliothek mit mehreren Büchern.

```csharp
public class Book
{
    public string Name { get; }

    public Book(string name)
    {
        Name = name;
    }
}
```

---

# Library

Die Bibliothek speichert intern Bücher.

Der Client soll aber nicht wissen müssen, **wie genau** diese gespeichert werden.

```csharp
public class Library : IAggregate<Book>
{
    // Die interne Datenstruktur bleibt privat.
    private readonly Book[] _books =
    {
        new Book("Krieg und Frieden"),
        new Book("Väter und Söhne"),
        new Book("Der Kirschgarten")
    };

    public int Count => _books.Length;

    // Ermöglicht dem Iterator den Zugriff
    // auf ein bestimmtes Buch.
    public Book this[int index] =>
        _books[index];

    // Erstellt einen Iterator für diese Bibliothek.
    public IIterator<Book> CreateIterator()
    {
        return new LibraryIterator(this);
    }
}
```

---

# LibraryIterator

```csharp
// Konkreter Iterator für die Bibliothek.
public class LibraryIterator : IIterator<Book>
{
    private readonly Library _library;

    // Speichert die aktuelle Position
    // innerhalb der Sammlung.
    private int _index;

    public LibraryIterator(Library library)
    {
        _library = library;
    }

    public bool HasNext()
    {
        return _index < _library.Count;
    }

    public Book Next()
    {
        return _library[_index++];
    }
}
```

---

# Client

Der Client verwendet nur den Iterator:

```csharp
public class Reader
{
    public void SeeBooks(Library library)
    {
        IIterator<Book> iterator =
            library.CreateIterator();

        while (iterator.HasNext())
        {
            Book book =
                iterator.Next();

            Console.WriteLine(book.Name);
        }
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
        Library library =
            new Library();

        Reader reader =
            new Reader();

        reader.SeeBooks(library);
    }
}
```

Ausgabe:

```text
Krieg und Frieden
Väter und Söhne
Der Kirschgarten
```

---

# Was passiert hier?

Der `Reader` bekommt:

```csharp
Library library
```

Er greift aber nicht direkt auf:

```csharp
_books
```

zu.

Stattdessen:

```csharp
IIterator<Book> iterator =
    library.CreateIterator();
```

Danach:

```csharp
while (iterator.HasNext())
{
    Book book =
        iterator.Next();
}
```

Der Iterator kümmert sich selbst darum:

```text
Index = 0
   │
   ▼
Book 1
   │
Index = 1
   │
   ▼
Book 2
   │
Index = 2
   │
   ▼
Book 3
```

---

# Warum ist das sinnvoll?

Ohne Iterator müsste der Client möglicherweise die interne Struktur kennen.

Zum Beispiel:

```csharp
for (int i = 0; i < library.Books.Length; i++)
{
    Console.WriteLine(
        library.Books[i].Name);
}
```

Damit kennt der Client:

```text
Library
→ besitzt Books
→ Books ist ein Array
```

Wenn wir später intern statt eines Arrays:

```csharp
List<Book>
```

verwenden würden, könnte Client-Code davon betroffen sein.

Mit Iterator:

```csharp
while (iterator.HasNext())
{
    Book book = iterator.Next();
}
```

bleibt die Traversierung unabhängig von der internen Datenstruktur.

---

# Iterator in .NET

In modernem C# muss man einen eigenen Iterator normalerweise **nicht selbst erfinden**.

.NET bietet bereits:

```csharp
IEnumerable<T>
IEnumerator<T>
```

---

# `IEnumerable<T>`

`IEnumerable<T>` beschreibt ein Objekt, dessen Elemente durchlaufen werden können.

Vereinfacht sieht das Prinzip so aus:

```csharp
public interface IEnumerable<T>
{
    IEnumerator<T> GetEnumerator();
}
```

Die entscheidende Methode ist:

```csharp
GetEnumerator();
```

Sie liefert einen Iterator.

---

# `IEnumerator<T>`

`IEnumerator<T>` repräsentiert den eigentlichen Iterator.

Die wichtigsten Member sind:

```csharp
bool MoveNext();

T Current { get; }
```

Vereinfacht:

```text
MoveNext()
→ zum nächsten Element wechseln

Current
→ aktuelles Element erhalten
```

---

# Wie funktioniert `foreach`?

Wenn wir schreiben:

```csharp
foreach (Book book in library)
{
    Console.WriteLine(book.Name);
}
```

läuft das konzeptionell ungefähr auf Folgendes hinaus:

```csharp
IEnumerator<Book> iterator =
    library.GetEnumerator();

while (iterator.MoveNext())
{
    Book book =
        iterator.Current;

    Console.WriteLine(book.Name);
}
```

> [!note]  
> Der Compiler übernimmt diese Arbeit normalerweise automatisch.

---

# Moderne Library mit `IEnumerable<Book>`

Statt eigener Interfaces wie:

```text
IBookIterator
IBookNumerable
```

würde man in modernem C# normalerweise direkt die .NET-Interfaces verwenden.

```csharp
public class Library : IEnumerable<Book>
{
    private readonly Book[] _books =
    {
        new Book("Krieg und Frieden"),
        new Book("Väter und Söhne"),
        new Book("Der Kirschgarten")
    };

    public IEnumerator<Book> GetEnumerator()
    {
        return ((IEnumerable<Book>)_books)
            .GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
```

Dafür benötigen wir:

```csharp
using System.Collections;
using System.Collections.Generic;
```

---

# Verwendung mit `foreach`

Jetzt kann direkt geschrieben werden:

```csharp
Library library =
    new Library();

foreach (Book book in library)
{
    Console.WriteLine(book.Name);
}
```

Der Client muss keinen Iterator explizit erzeugen.

Das übernimmt `foreach`.

---

# Noch einfacher mit `yield return`

C# besitzt eine sehr komfortable Möglichkeit zur Implementierung eines Iterators:

```csharp
yield return
```

Damit müssen wir `IEnumerator<T>` meistens nicht selbst implementieren.

---

# Beispiel mit `yield return`

```csharp
public class Library : IEnumerable<Book>
{
    private readonly Book[] _books =
    {
        new Book("Krieg und Frieden"),
        new Book("Väter und Söhne"),
        new Book("Der Kirschgarten")
    };

    public IEnumerator<Book> GetEnumerator()
    {
        foreach (Book book in _books)
        {
            // Der Compiler erzeugt automatisch
            // die notwendige Iterator-Logik.
            yield return book;
        }
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
```

Danach funktioniert wieder:

```csharp
foreach (Book book in library)
{
    Console.WriteLine(book.Name);
}
```

---

# Was macht `yield return`?

`yield return` liefert ein Element zurück und merkt sich gleichzeitig die aktuelle Position.

Beispiel:

```csharp
public IEnumerable<int> GetNumbers()
{
    yield return 10;
    yield return 20;
    yield return 30;
}
```

Verwendung:

```csharp
foreach (int number in GetNumbers())
{
    Console.WriteLine(number);
}
```

Ausgabe:

```text
10
20
30
```

---

# Ablauf von `yield return`

```text
GetNumbers()
     │
     ▼
yield return 10
     │
     │ foreach verarbeitet 10
     │
     ▼
yield return 20
     │
     │ foreach verarbeitet 20
     │
     ▼
yield return 30
```

Die Methode muss also nicht unbedingt zuerst alle Ergebnisse vollständig erzeugen.

---

# Lazy Evaluation

Iteratoren ermöglichen häufig eine **verzögerte Auswertung**.

Beispiel:

```csharp
public IEnumerable<int> GetNumbers()
{
    Console.WriteLine("Erzeuge 1");
    yield return 1;

    Console.WriteLine("Erzeuge 2");
    yield return 2;

    Console.WriteLine("Erzeuge 3");
    yield return 3;
}
```

Wenn wir schreiben:

```csharp
IEnumerable<int> numbers =
    GetNumbers();
```

werden die einzelnen Elemente noch nicht zwingend vollständig verarbeitet.

Erst beim Durchlaufen:

```csharp
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

wird Schritt für Schritt weitergearbeitet.

---

# Verschiedene Traversierungsstrategien

Ein Vorteil des Iterator Patterns besteht darin, dass dieselbe Sammlung auf unterschiedliche Weise durchlaufen werden kann.

Zum Beispiel:

```text
Bibliothek

Normaler Iterator:
Book 1
Book 2
Book 3

Rückwärts-Iterator:
Book 3
Book 2
Book 1
```

---

# Beispiel: Rückwärts durchlaufen

```csharp
public IEnumerable<Book> GetBooksReverse()
{
    for (int i = _books.Length - 1; i >= 0; i--)
    {
        yield return _books[i];
    }
}
```

Verwendung:

```csharp
foreach (Book book in library.GetBooksReverse())
{
    Console.WriteLine(book.Name);
}
```

Jetzt erhalten wir:

```text
Der Kirschgarten
Väter und Söhne
Krieg und Frieden
```

Die interne Sammlung bleibt dieselbe.

Nur die Traversierungsstrategie ändert sich.

---

# Mehrere Iteratoren gleichzeitig

Ein weiterer Vorteil:

Mehrere Iteratoren können unabhängig voneinander dieselbe Sammlung durchlaufen.

```csharp
IEnumerator<Book> iterator1 =
    library.GetEnumerator();

IEnumerator<Book> iterator2 =
    library.GetEnumerator();
```

Beide Iteratoren besitzen ihren eigenen Zustand.

```text
Library
  │
  ├── Iterator 1 → Position 2
  │
  └── Iterator 2 → Position 0
```

Die aktuelle Position gehört also nicht unbedingt zur Sammlung, sondern zum jeweiligen Iterator.

---

# Warum sollte `_index` im Iterator liegen?

Angenommen, die Position wäre direkt in `Library` gespeichert:

```csharp
private int _index;
```

Dann könnten nicht problemlos zwei Clients gleichzeitig unabhängig durch dieselbe Bibliothek navigieren.

Beim Iterator:

```csharp
public class LibraryIterator
{
    private int _index;
}
```

besitzt jeder Iterator seine eigene Position.

---

# Klassischer Iterator vs. .NET Iterator

## Klassische GoF-Struktur

```text
Aggregate
    │
    ▼
CreateIterator()
    │
    ▼
Iterator
    │
    ├── First()
    ├── Next()
    ├── Current()
    └── IsDone()
```

---

## Moderne .NET-Struktur

```text
IEnumerable<T>
      │
      ▼
GetEnumerator()
      │
      ▼
IEnumerator<T>
      │
      ├── MoveNext()
      └── Current
```

---

# Vergleich

|Klassisches Pattern|.NET|
|---|---|
|`Aggregate`|`IEnumerable<T>`|
|`Iterator`|`IEnumerator<T>`|
|`CreateIterator()`|`GetEnumerator()`|
|`Next()`|`MoveNext()` + `Current`|
|`CurrentItem()`|`Current`|
|eigener Client-Loop|meist `foreach`|

---

# `IEnumerator` vs. `IEnumerator<T>`

Es gibt eine ältere nicht-generische Variante:

```csharp
IEnumerator
```

Dabei ist:

```csharp
object Current
```

Der Rückgabewert ist also `object`.

Die moderne generische Variante:

```csharp
IEnumerator<Book>
```

liefert direkt:

```csharp
Book Current
```

Dadurch benötigt man keinen Cast.

> [!tip]  
> In modernem C# sollte man normalerweise die generischen Varianten `IEnumerable<T>` und `IEnumerator<T>` verwenden.

---

# `Reset()`

Beim alten nicht-generischen `IEnumerator` existiert auch:

```csharp
void Reset();
```

In der Praxis ist `Reset()` für eigene moderne Iteratoren meistens kaum relevant.

Normalerweise erstellt man einfach einen neuen Enumerator:

```csharp
IEnumerator<Book> iterator =
    library.GetEnumerator();
```

---

# `foreach` und Iterator Pattern

Ein typischer Ablauf:

```text
foreach
   │
   ▼
GetEnumerator()
   │
   ▼
IEnumerator<T>
   │
   ▼
MoveNext()
   │
   ├── false → Ende
   │
   └── true
        │
        ▼
      Current
        │
        ▼
    Schleifenrumpf
```

---

# Ein modernes vollständiges Beispiel

```csharp
using System.Collections;

public class Book
{
    public string Name { get; }

    public Book(string name)
    {
        Name = name;
    }
}


public class Library : IEnumerable<Book>
{
    // Die interne Struktur bleibt für den Client verborgen.
    private readonly List<Book> _books =
    [
        new Book("Krieg und Frieden"),
        new Book("Väter und Söhne"),
        new Book("Der Kirschgarten")
    ];

    // Liefert die Bücher nacheinander.
    public IEnumerator<Book> GetEnumerator()
    {
        foreach (Book book in _books)
        {
            yield return book;
        }
    }

    // Implementierung für das nicht-generische IEnumerable.
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}


public class Reader
{
    public void SeeBooks(IEnumerable<Book> books)
    {
        // Reader kennt die interne Struktur der Sammlung nicht.
        foreach (Book book in books)
        {
            Console.WriteLine(book.Name);
        }
    }
}


public class Program
{
    public static void Main()
    {
        Library library =
            new Library();

        Reader reader =
            new Reader();

        reader.SeeBooks(library);
    }
}
```

---

# Warum `Reader` besser `IEnumerable<Book>` bekommt

Statt:

```csharp
public void SeeBooks(Library library)
```

können wir schreiben:

```csharp
public void SeeBooks(
    IEnumerable<Book> books)
```

Dadurch ist der Reader nicht mehr direkt von `Library` abhängig.

Er kann jetzt auch Folgendes verarbeiten:

```csharp
List<Book>
Book[]
HashSet<Book>
Library
```

solange das Objekt:

```csharp
IEnumerable<Book>
```

implementiert.

Das reduziert die Kopplung.

---

# Beispiel

```csharp
List<Book> books =
[
    new Book("Clean Code"),
    new Book("Clean Architecture")
];

Reader reader =
    new Reader();

reader.SeeBooks(books);
```

Der `Reader` funktioniert unverändert.

---

# Iterator und LINQ

Auch LINQ arbeitet sehr stark mit:

```csharp
IEnumerable<T>
```

Beispiel:

```csharp
IEnumerable<Book> result =
    library.Where(
        book => book.Name.Contains("Krieg"));
```

Danach:

```csharp
foreach (Book book in result)
{
    Console.WriteLine(book.Name);
}
```

Viele LINQ-Operationen arbeiten ebenfalls mit verzögerter Auswertung.

---

# Vorteile

- Die interne Struktur einer Sammlung bleibt verborgen.
    
- Unterschiedliche Sammlungen können einheitlich durchlaufen werden.
    
- Mehrere Iteratoren können unabhängig voneinander arbeiten.
    
- Unterschiedliche Traversierungsstrategien sind möglich.
    
- Client-Code wird von der konkreten Datenstruktur entkoppelt.
    
- `foreach` integriert das Pattern sehr elegant in C#.
    
- `yield return` vereinfacht eigene Iteratoren erheblich.
    

---

# Nachteile

- Für sehr einfache Collections kann ein eigener Iterator unnötig sein.
    
- Eigene Iterator-Klassen erhöhen die Anzahl der Klassen.
    
- Änderungen an der Sammlung während einer Iteration können problematisch sein.
    
- Komplexe Traversierungen können schwieriger nachzuvollziehen sein.
    

---

# Änderung einer Sammlung während `foreach`

Ein typischer Fehler:

```csharp
foreach (Book book in books)
{
    books.Remove(book);
}
```

Viele Collections erlauben keine strukturelle Änderung während der Iteration.

Dann kann beispielsweise eine:

```text
InvalidOperationException
```

auftreten.

Das liegt daran, dass sich die Sammlung verändert, während der Iterator gerade ihren Zustand verfolgt.

---

# Iterator vs. Collection

Der Unterschied:

```text
Collection
→ speichert die Elemente

Iterator
→ weiß, wie man durch die Elemente navigiert
```

Beispiel:

```text
Library
├── Book A
├── Book B
└── Book C

LibraryIterator
└── aktuelle Position = 1
```

---

# Iterator vs. Strategy

Beide können unterschiedliches Verhalten kapseln.

## Iterator

```text
Wie durchlaufe ich eine Sammlung?
```

## Strategy

```text
Welchen Algorithmus verwende ich?
```

Iterator konzentriert sich speziell auf die **Traversal einer Datenstruktur**.

---

# Vereinfachte Struktur

```text
              Library
                 │
                 │ GetEnumerator()
                 ▼
          IEnumerator<Book>
                 │
          ┌──────┼──────┐
          ▼      ▼      ▼
        Book1  Book2  Book3
```

Mit `foreach`:

```text
Library
   │
   ▼
foreach
   │
   ▼
GetEnumerator()
   │
   ▼
MoveNext()
   │
   ▼
Current
```

---

# Der wichtigste Punkt

Der Client soll nicht schreiben müssen:

```csharp
library.InternalArray[index]
```

sondern einfach:

```csharp
foreach (Book book in library)
{
    Console.WriteLine(book.Name);
}
```

Der Client weiß nicht:

```text
Array?
List?
LinkedList?
Datenbank?
Generierte Daten?
```

Er weiß nur:

> „Dieses Objekt kann ich durchlaufen.“

---

# Das solltest du dir für C# merken

Die wichtigsten Interfaces:

```csharp
IEnumerable<T>
IEnumerator<T>
```

Typische Verwendung:

```csharp
foreach (T item in collection)
{
}
```

Eigener Iterator:

```csharp
public IEnumerator<T> GetEnumerator()
{
    foreach (T item in _items)
    {
        yield return item;
    }
}
```

Oder noch einfacher, wenn man nur eine Sequenz liefern möchte:

```csharp
public IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}
```

---

# Merksatz

> **Iterator ermöglicht das schrittweise Durchlaufen einer Sammlung, ohne deren interne Struktur offenzulegen.**

Noch einfacher:

```text
Collection:
"Hier sind meine Daten."

Iterator:
"Ich zeige dir nacheinander jedes Element."
```

Oder ganz kurz:

```text
Iterator
=
Sammlung durchlaufen
ohne interne Struktur zu kennen
```

> [!summary] Zusammenfassung  
> Das **Iterator Pattern** ist ein **Verhaltensmuster**.
> 
> Es trennt die Speicherung einer Sammlung von der Logik, mit der diese Sammlung durchlaufen wird.
> 
> In modernem C# wird dieses Pattern hauptsächlich durch:
> 
> ```csharp
> IEnumerable<T>
> IEnumerator<T>
> ```
> 
> umgesetzt.
> 
> `foreach` verwendet einen Enumerator automatisch:
> 
> ```csharp
> foreach (Book book in library)
> {
>     Console.WriteLine(book.Name);
> }
> ```
> 
> Mit `yield return` lassen sich eigene Iteratoren besonders einfach implementieren:
> 
> ```csharp
> public IEnumerable<Book> GetBooks()
> {
>     foreach (Book book in _books)
>     {
>         yield return book;
>     }
> }
> ```
> 
> **Kurz gesagt:**  
> `Iterator = Elemente nacheinander durchlaufen, ohne die interne Struktur der Sammlung zu kennen.`