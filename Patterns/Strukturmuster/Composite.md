Das **Composite Pattern** ist ein Strukturmuster, mit dem einzelne Objekte und Gruppen von Objekten zu einer **hierarchischen Baumstruktur** zusammengesetzt werden können.

Der entscheidende Punkt ist:

> [!info] Grundidee  
> Der Client kann ein einzelnes Objekt und eine Gruppe von Objekten **über dieselbe gemeinsame Schnittstelle** behandeln.

Typische Beispiele:

```
Dateisystem
├── Ordner
│   ├── Datei
│   ├── Datei
│   └── Unterordner
│       └── Datei
└── Datei
```

oder:

```
Menü
├── Menüpunkt
├── Menüpunkt
└── Untermenü
    ├── Menüpunkt
    └── Untermenü
```

---

# Wann sollte man Composite verwenden?

Das Composite Pattern eignet sich besonders:

- wenn Objekte eine hierarchische Baumstruktur bilden;
- wenn eine **Teil-Ganzes-Beziehung** modelliert werden soll;
- wenn einzelne Elemente und Gruppen gleich behandelt werden sollen;
- wenn Composite-Objekte wiederum weitere Composite-Objekte enthalten können;
- wenn Operationen rekursiv auf einer gesamten Objektstruktur ausgeführt werden sollen.

Typische Anwendungsfälle:

```
Dateisysteme
Menüs
UI-Komponenten
Organisationsstrukturen
Produktgruppen
Dokumentstrukturen
Grafische Szenen
ASTs
Kategorien
Kommentare mit Antworten
```

---

# Teil-Ganzes-Beziehung

Composite modelliert eine sogenannte:

**Part-Whole-Hierarchy**

also:

```
Teil
↕
Ganzes
```

Ein Ordner ist beispielsweise:

```
Teil eines anderen Ordners
```

kann aber gleichzeitig:

```
selbst mehrere Dateien und Ordner enthalten
```

sein.

Beispiel:

```
FileSystem
└── C:
    ├── Program.cs
    ├── README.md
    └── Documents
        ├── CV.pdf
        └── Notes.txt
```

`Documents` ist:

```
ein Teil von C:
```

und gleichzeitig:

```
ein Ganzes für CV.pdf und Notes.txt
```

---

# Grundstruktur

```
                    Component
                       ▲
             ┌─────────┴─────────┐
             │                   │
            Leaf              Composite
                                 │
                                 │ enthält
                                 ▼
                          List<Component>
```

Ein `Composite` kann wiederum enthalten:

```
Leaf
Composite
```

Dadurch entsteht die rekursive Baumstruktur.

---

# Teilnehmer des Patterns

## Component

`Component` definiert die gemeinsame Abstraktion für alle Elemente.

Sowohl:

```
Leaf
```

als auch:

```
Composite
```

verwenden dieselbe Schnittstelle.

Zum Beispiel:

```
public abstract class Component
{
    public string Name { get; }

    protected Component(string name)
    {
        Name = name;
    }

    public abstract void Display();
}
```

---

# Leaf

Ein **Leaf** ist ein einzelnes Element.

Es besitzt keine untergeordneten Elemente.

Beispiele:

```
Datei
Menüpunkt
Mitarbeiter ohne Untergebene
einzelnes UI-Control
```

Zum Beispiel:

```
public class Leaf : Component
{
    public Leaf(string name)
        : base(name)
    {
    }

    public override void Display()
    {
        Console.WriteLine(Name);
    }
}
```

---

# Composite

Ein **Composite** ist ein Element, das andere `Component`-Objekte enthalten kann.

Beispielsweise:

```
public class Composite : Component
{
    private readonly List<Component> _children = new();

    public Composite(string name)
        : base(name)
    {
    }

    public void Add(Component component)
    {
        _children.Add(component);
    }

    public void Remove(Component component)
    {
        _children.Remove(component);
    }

    public override void Display()
    {
        Console.WriteLine(Name);

        // Die Operation wird rekursiv
        // auf alle untergeordneten Komponenten angewendet.
        foreach (Component child in _children)
        {
            child.Display();
        }
    }
}
```

---

# Client

Der Client arbeitet hauptsächlich mit:

```
Component
```

und muss nicht immer wissen, ob das konkrete Objekt:

```
Leaf
```

oder:

```
Composite
```

ist.

Das ist die zentrale Idee.

---

# Allgemeines Beispiel

```
Component root =
    new Composite("Root");

Component leaf =
    new Leaf("Leaf");

Composite subtree =
    new Composite("Subtree");
```

Wir können dann:

```
subtree.Add(
    new Leaf("Leaf A"));

subtree.Add(
    new Leaf("Leaf B"));
```

und:

```
((Composite)root).Add(leaf);
((Composite)root).Add(subtree);
```

Das funktioniert, ist aber nicht besonders elegant, weil wir casten müssen.

Deshalb sollten wir die Modellierung noch etwas verbessern.

---

# Zwei Möglichkeiten für `Add()` und `Remove()`

Beim Composite Pattern gibt es zwei typische Designvarianten.

---

## Variante 1: `Add()` befindet sich in `Component`

So wie in der ursprünglichen Quelle:

```
public abstract class Component
{
    public abstract void Display();

    public abstract void Add(Component component);

    public abstract void Remove(Component component);
}
```

Dann muss auch ein `Leaf` diese Methoden implementieren:

```
public class Leaf : Component
{
    public override void Add(Component component)
    {
        throw new NotSupportedException();
    }

    public override void Remove(Component component)
    {
        throw new NotSupportedException();
    }

    public override void Display()
    {
        // ...
    }
}
```

---

# Problem dieser Variante

Eine Datei kann logisch keine Kinder besitzen.

Trotzdem besitzt sie:

```
Add(...)
Remove(...)
```

Das ist aus Sicht der API etwas unsauber.

Der Client könnte schreiben:

```
file.Add(otherFile);
```

und bekommt erst zur Laufzeit einen Fehler.

---

# Variante 2: Nur Composite besitzt `Add()` und `Remove()`

Modern und häufig verständlicher:

```
public abstract class FileSystemEntry
{
    public string Name { get; }

    protected FileSystemEntry(string name)
    {
        Name = name;
    }

    public abstract void Print();
}
```

`File`:

```
public sealed class FileEntry : FileSystemEntry
{
    public FileEntry(string name)
        : base(name)
    {
    }

    public override void Print()
    {
        Console.WriteLine(Name);
    }
}
```

`Directory`:

```
public sealed class DirectoryEntry : FileSystemEntry
{
    private readonly List<FileSystemEntry> _children =
        new();

    public DirectoryEntry(string name)
        : base(name)
    {
    }

    public void Add(FileSystemEntry entry)
    {
        _children.Add(entry);
    }

    public void Remove(FileSystemEntry entry)
    {
        _children.Remove(entry);
    }

    public override void Print()
    {
        Console.WriteLine(Name);

        foreach (FileSystemEntry child in _children)
        {
            child.Print();
        }
    }
}
```

Hier besitzt nur ein Ordner:

```
Add
Remove
```

weil nur ein Ordner Kinder enthalten kann.

> [!tip]  
> Diese Variante ist oft typsicherer und drückt das Domänenmodell klarer aus.

---

# Transparenz vs. Sicherheit

Das ist ein klassischer Trade-off beim Composite Pattern.

## Gemeinsames `Add()` in Component

Vorteil:

```
Leaf und Composite besitzen exakt dieselbe API.
```

Das ist sehr transparent für den Client.

Nachteil:

```
Leaf besitzt Methoden,
die fachlich keinen Sinn ergeben.
```

---

## `Add()` nur im Composite

Vorteil:

```
Die API ist fachlich sauberer.
```

Nachteil:

Der Client muss wissen, wann ein Objekt tatsächlich ein Composite ist.

---

# Merksatz

```
Variante 1:
mehr Einheitlichkeit

Variante 2:
mehr Typsicherheit
```

---

# Praktisches Beispiel: Dateisystem

Das Dateisystem ist ein klassischer Anwendungsfall für Composite.

Wir haben:

```
FileSystemEntry
│
├── FileEntry
└── DirectoryEntry
```

Ein `DirectoryEntry` kann wiederum enthalten:

```
FileEntry
DirectoryEntry
```

---

# Gemeinsame Abstraktion

```
public abstract class FileSystemEntry
{
    public string Name { get; }

    protected FileSystemEntry(string name)
    {
        Name = name;
    }

    // Jede Komponente kann dargestellt werden.
    public abstract void Print(string indent = "");
}
```

---

# Leaf: Datei

```
public sealed class FileEntry : FileSystemEntry
{
    public FileEntry(string name)
        : base(name)
    {
    }

    public override void Print(string indent = "")
    {
        // Eine Datei besitzt keine weiteren Kinder.
        Console.WriteLine(
            $"{indent}📄 {Name}");
    }
}
```

---

# Composite: Ordner

```
public sealed class DirectoryEntry : FileSystemEntry
{
    // Ein Ordner kann Dateien
    // und weitere Ordner enthalten.
    private readonly List<FileSystemEntry> _children =
        new();

    public DirectoryEntry(string name)
        : base(name)
    {
    }

    public void Add(FileSystemEntry entry)
    {
        _children.Add(entry);
    }

    public void Remove(FileSystemEntry entry)
    {
        _children.Remove(entry);
    }

    public override void Print(string indent = "")
    {
        Console.WriteLine(
            $"{indent}📁 {Name}");

        foreach (FileSystemEntry child in _children)
        {
            // Rekursiver Aufruf:
            // Ein Kind kann eine Datei oder
            // wieder ein kompletter Ordner sein.
            child.Print(indent + "  ");
        }
    }
}
```

---

# Baum aufbauen

```
DirectoryEntry fileSystem =
    new DirectoryEntry("Dateisystem");

DirectoryEntry diskC =
    new DirectoryEntry("C:");

FileEntry image =
    new FileEntry("12345.png");

FileEntry document =
    new FileEntry("Document.docx");
```

Dateien hinzufügen:

```
diskC.Add(image);
diskC.Add(document);
```

Laufwerk hinzufügen:

```
fileSystem.Add(diskC);
```

---

# Aktuelle Struktur

```
Dateisystem
└── C:
    ├── 12345.png
    └── Document.docx
```

---

# Ausgabe

```
fileSystem.Print();
```

Ergebnis:

```
📁 Dateisystem
  📁 C:
    📄 12345.png
    📄 Document.docx
```

---

# Weitere Unterstruktur hinzufügen

Jetzt erstellen wir einen neuen Ordner:

```
DirectoryEntry documents =
    new DirectoryEntry("Meine Dokumente");
```

Dateien:

```
FileEntry readme =
    new FileEntry("readme.txt");

FileEntry program =
    new FileEntry("Program.cs");
```

Hinzufügen:

```
documents.Add(readme);
documents.Add(program);

diskC.Add(documents);
```

---

# Baum

```
Dateisystem
└── C:
    ├── 12345.png
    ├── Document.docx
    └── Meine Dokumente
        ├── readme.txt
        └── Program.cs
```

---

# Komplettes Beispiel

```
public class Program
{
    public static void Main()
    {
        DirectoryEntry fileSystem =
            new DirectoryEntry("Dateisystem");

        DirectoryEntry diskC =
            new DirectoryEntry("C:");

        FileEntry image =
            new FileEntry("12345.png");

        FileEntry document =
            new FileEntry("Document.docx");

        // Dateien dem Laufwerk hinzufügen.
        diskC.Add(image);
        diskC.Add(document);

        // Laufwerk der Dateisystemstruktur hinzufügen.
        fileSystem.Add(diskC);

        fileSystem.Print();

        Console.WriteLine();

        // Datei entfernen.
        diskC.Remove(image);

        // Neuen Unterordner erzeugen.
        DirectoryEntry documents =
            new DirectoryEntry("Meine Dokumente");

        documents.Add(
            new FileEntry("readme.txt"));

        documents.Add(
            new FileEntry("Program.cs"));

        diskC.Add(documents);

        fileSystem.Print();
    }
}
```

---

# Was passiert bei `Print()`?

Das Interessante ist die Rekursion.

Wir starten:

```
fileSystem.Print();
```

`fileSystem` ist ein Ordner.

Dieser ruft:

```
child.Print(...)
```

für jedes Kind auf.

---

# Schritt 1

```
FileSystem
```

enthält:

```
C:
```

Also:

```
diskC.Print();
```

---

# Schritt 2

`C:` enthält:

```
Document.docx
Meine Dokumente
```

Für `Document.docx`:

```
document.Print();
```

Da es ein Leaf ist, endet dieser Ast dort.

---

# Schritt 3

Für:

```
Meine Dokumente
```

wird wieder:

```
documents.Print();
```

aufgerufen.

Da `documents` wieder ein Composite ist, werden dessen Kinder durchlaufen.

---

# Rekursive Struktur

```
Print(FileSystem)
    │
    ▼
Print(C:)
    │
    ├── Print(Document.docx)
    │
    └── Print(Meine Dokumente)
            │
            ├── Print(readme.txt)
            └── Print(Program.cs)
```

Diese Rekursion ist typisch für Composite.

---

# Warum kann der Client alles gleich behandeln?

Alle Objekte sind:

```
FileSystemEntry
```

Also kann beispielsweise eine Methode schreiben:

```
public void PrintEntry(
    FileSystemEntry entry)
{
    entry.Print();
}
```

Die Methode funktioniert mit:

```
new FileEntry("test.txt");
```

genauso wie mit:

```
new DirectoryEntry("Documents");
```

Der Client muss nicht unterscheiden:

```
if (entry is FileEntry)
{
}
else if (entry is DirectoryEntry)
{
}
```

Die Polymorphie erledigt das.

---

# Genau das ist die Kernidee

```
Leaf
     \
      \
       Component
      /
     /
Composite
```

Client:

```
"Mir ist egal,
ob du ein einzelnes Element
oder eine ganze Gruppe bist.

Du bist für mich ein Component."
```

---

# Operationen über den gesamten Baum

Composite wird besonders interessant, wenn Operationen rekursiv aggregiert werden können.

Zum Beispiel:

```
Gesamtgröße berechnen
Dateien zählen
Namen suchen
Berechtigungen prüfen
Kosten summieren
Unterelemente darstellen
```

---

# Beispiel: Dateigröße

Wir erweitern die Abstraktion:

```
public abstract class FileSystemEntry
{
    public string Name { get; }

    protected FileSystemEntry(string name)
    {
        Name = name;
    }

    public abstract long GetSize();
}
```

---

# Datei

```
public sealed class FileEntry : FileSystemEntry
{
    private readonly long _size;

    public FileEntry(
        string name,
        long size)
        : base(name)
    {
        _size = size;
    }

    public override long GetSize()
    {
        // Eine Datei liefert einfach
        // ihre eigene Größe zurück.
        return _size;
    }
}
```

---

# Ordner

```
public sealed class DirectoryEntry : FileSystemEntry
{
    private readonly List<FileSystemEntry> _children =
        new();

    public DirectoryEntry(string name)
        : base(name)
    {
    }

    public void Add(FileSystemEntry entry)
    {
        _children.Add(entry);
    }

    public override long GetSize()
    {
        long totalSize = 0;

        // Die Größe eines Ordners ergibt sich
        // rekursiv aus allen enthaltenen Komponenten.
        foreach (FileSystemEntry child in _children)
        {
            totalSize += child.GetSize();
        }

        return totalSize;
    }
}
```

---

# Beispiel

```
Documents
├── CV.pdf       100 KB
├── Notes.txt     20 KB
└── Source
    ├── A.cs      10 KB
    └── B.cs      15 KB
```

Dann:

```
Source
= 10 + 15
= 25 KB
```

und:

```
Documents
= 100 + 20 + 25
= 145 KB
```

Der Client schreibt trotzdem nur:

```
long size =
    documents.GetSize();
```

---

# Das ist die Stärke von Composite

Der Client muss nicht selbst rekursiv unterscheiden:

```
if (entry is FileEntry file)
{
    // Dateigröße
}
else if (entry is DirectoryEntry directory)
{
    // alle Kinder durchlaufen
}
```

Die einzelnen Objekte wissen selbst, wie die Operation ausgeführt wird.

---

# Beispiel: Organisationsstruktur

Composite eignet sich nicht nur für Dateien.

Ein Unternehmen könnte so aussehen:

```
CEO
├── Development Department
│   ├── Developer A
│   ├── Developer B
│   └── Backend Team
│       ├── Developer C
│       └── Developer D
│
└── Sales Department
    ├── Salesperson A
    └── Salesperson B
```

Dabei könnten:

```
Employee
```

Leaves sein und:

```
Department
```

Composites.

---

# Gemeinsames Interface

```
public interface IOrganizationUnit
{
    decimal GetMonthlyCost();
}
```

Mitarbeiter:

```
public class Employee : IOrganizationUnit
{
    public decimal Salary { get; }

    public Employee(decimal salary)
    {
        Salary = salary;
    }

    public decimal GetMonthlyCost()
    {
        return Salary;
    }
}
```

Abteilung:

```
public class Department : IOrganizationUnit
{
    private readonly List<IOrganizationUnit> _units =
        new();

    public void Add(IOrganizationUnit unit)
    {
        _units.Add(unit);
    }

    public decimal GetMonthlyCost()
    {
        decimal total = 0;

        foreach (IOrganizationUnit unit in _units)
        {
            total += unit.GetMonthlyCost();
        }

        return total;
    }
}
```

Jetzt kann der Client schreiben:

```
decimal cost =
    company.GetMonthlyCost();
```

und bekommt die Summe des gesamten Baums.

---

# Composite und Rekursion

Composite basiert sehr häufig auf Rekursion.

Ein Composite enthält:

```
Component
```

aber dieses `Component` kann wiederum ein:

```
Composite
```

sein.

Also:

```
Composite
└── Composite
    └── Composite
        └── Leaf
```

Deshalb funktionieren Methoden oft rekursiv:

```
foreach (Component child in _children)
{
    child.Operation();
}
```

---

# Neue Elementtypen hinzufügen

Angenommen, unser Dateisystem besitzt bisher:

```
FileEntry
DirectoryEntry
```

Später möchten wir:

```
ShortcutEntry
```

hinzufügen.

Dann können wir einfach:

```
public sealed class ShortcutEntry
    : FileSystemEntry
{
    public ShortcutEntry(string name)
        : base(name)
    {
    }

    public override void Print(string indent = "")
    {
        Console.WriteLine(
            $"{indent}🔗 {Name}");
    }
}
```

implementieren.

Andere Komponenten müssen dafür nicht zwangsläufig verändert werden.

---

# Composite und Open/Closed Principle

Das Pattern kann damit das **Open/Closed Principle** unterstützen.

Die Abstraktion bleibt:

```
FileSystemEntry
```

und neue konkrete Elementtypen können ergänzt werden:

```
FileEntry
DirectoryEntry
ShortcutEntry
ArchiveEntry
...
```

ohne Client-Code stark verändern zu müssen.

---

# Composite und Polymorphie

Composite funktioniert stark über Polymorphie.

```
FileSystemEntry entry;
```

kann sein:

```
entry =
    new FileEntry("test.txt");
```

oder:

```
entry =
    new DirectoryEntry("Documents");
```

Der Client ruft immer:

```
entry.Print();
```

auf.

Welcher Code tatsächlich läuft, entscheidet der konkrete Laufzeittyp.

---

# Composite vs. Decorator

Diese beiden Patterns besitzen eine ähnliche rekursive Struktur.

## Decorator

Ein Decorator enthält typischerweise **genau eine** weitere Component:

```
Decorator
   │
   ▼
Component
```

Beispiel:

```
CachingDecorator
      ↓
LoggingDecorator
      ↓
Service
```

---

## Composite

Ein Composite enthält typischerweise **mehrere** Components:

```
Composite
├── Component
├── Component
└── Component
```

Beispiel:

```
Directory
├── File
├── File
└── Directory
```

---

# Kurzvergleich

```
Decorator
→ Wrapper-Kette
→ meistens genau ein inneres Objekt


Composite
→ Baum
→ mehrere Kinder
```

---

# Composite vs. Iterator

## Composite

beschreibt:

```
Wie ist eine Baumstruktur aufgebaut?
```

---

## Iterator

beschreibt:

```
Wie durchlaufe ich eine Struktur?
```

Beide Patterns können hervorragend zusammenarbeiten.

Zum Beispiel:

```
Composite
→ repräsentiert Dateisystem

Iterator
→ läuft durch alle Dateien
```

---

# Composite vs. Visitor

Auch diese Patterns werden häufig gemeinsam eingesetzt.

## Composite

erstellt die Baumstruktur:

```
Directory
├── File
└── Directory
```

## Visitor

kann anschließend Operationen über diese Struktur ausführen:

```
SizeVisitor
SearchVisitor
ExportVisitor
```

Also:

```
Composite
→ Struktur

Visitor
→ Operation auf der Struktur
```

---

# Composite vs. Facade

## Composite

modelliert:

```
hierarchische Teil-Ganzes-Struktur
```

---

## Facade

stellt bereit:

```
vereinfachte Schnittstelle
für ein komplexes Subsystem
```

Diese Patterns verfolgen vollkommen unterschiedliche Ziele.

---

# Composite vs. Tree

Ein Baum ist zunächst nur eine **Datenstruktur**.

Composite ist dagegen ein **Design Pattern**.

Nicht jeder Baum ist automatisch das Composite Pattern.

Composite bedeutet zusätzlich:

> Einzelne Knoten und zusammengesetzte Knoten besitzen dieselbe gemeinsame Abstraktion und können vom Client einheitlich behandelt werden.

---

# Typische Struktur

```
                        Component
                            ▲
                 ┌──────────┴──────────┐
                 │                     │
                Leaf                Composite
                                      │
                                      ▼
                               List<Component>
                                      │
                         ┌────────────┼────────────┐
                         ▼            ▼            ▼
                       Leaf       Composite       Leaf
                                     │
                                     ▼
                                weitere Kinder
```

---

# Safe Composite

Eine moderne Variante ist häufig:

```
public abstract class Component
{
    public abstract void Operation();
}
```

Nur `Composite` besitzt:

```
Add(...)
Remove(...)
```

Das verhindert sinnlose Aufrufe auf Leaves.

---

# Transparent Composite

Die klassische GoF-Variante kann dagegen definieren:

```
public abstract class Component
{
    public abstract void Operation();

    public abstract void Add(Component component);

    public abstract void Remove(Component component);
}
```

Dann besitzen Client und alle Komponenten dieselbe API.

Aber Leaves müssen beispielsweise:

```
throw new NotSupportedException();
```

verwenden.

---

# Welche Variante ist besser?

Es gibt keine universell richtige Antwort.

Für moderne C#-APIs würde ich häufig die **Safe-Variante** bevorzugen:

```
Component
→ gemeinsame Operationen

Composite
→ zusätzlich Add/Remove
```

weil die Typen dadurch klarer ausdrücken, welche Operationen tatsächlich erlaubt sind.

---

# Elternreferenz

In manchen Composite-Strukturen kann es sinnvoll sein, dass ein Element seinen Parent kennt.

Zum Beispiel:

```
public abstract class FileSystemEntry
{
    public DirectoryEntry? Parent { get; internal set; }
}
```

Beim Hinzufügen:

```
public void Add(FileSystemEntry child)
{
    child.Parent = this;

    _children.Add(child);
}
```

Dann kann man beispielsweise:

```
Parent
Pfad
Breadcrumb
Navigation nach oben
```

realisieren.

---

# Beispiel: vollständiger Pfad

```
Root
└── Documents
    └── Projects
        └── App.cs
```

Durch Parent-Referenzen könnte:

```
file.GetFullPath();
```

liefern:

```
Root/Documents/Projects/App.cs
```

Das gehört nicht zwingend zum Composite Pattern, ist aber bei Baumstrukturen häufig praktisch.

---

# Zyklen vermeiden

Bei echten Composite-Strukturen muss man aufpassen, keine Zyklen zu erzeugen.

Problematisch wäre:

```
Folder A
└── Folder B
    └── Folder A
```

Dann könnte eine rekursive Operation wie:

```
Print();
```

endlos laufen.

Also:

```
A → B → A → B → A → ...
```

> [!warning]  
> Eine Composite-Struktur sollte normalerweise ein echter Baum beziehungsweise azyklischer Graph bleiben.

---

# Weitere typische Operationen

Eine Composite-Struktur kann beispielsweise folgende Operationen anbieten:

```
GetSize();
Print();
Search();
CalculateCost();
GetItemCount();
Validate();
Export();
```

Das Entscheidende ist:

Dieselbe Operation kann auf:

```
Leaf
```

und:

```
Composite
```

aufgerufen werden.

---

# Beispiel: Anzahl Elemente

Leaf:

```
public override int Count()
{
    return 1;
}
```

Composite:

```
public override int Count()
{
    int count = 1;

    foreach (Component child in _children)
    {
        count += child.Count();
    }

    return count;
}
```

Dann:

```
int count =
    root.Count();
```

liefert die Anzahl der Elemente des gesamten Baums.

---

# Vorteil für Client-Code

Ohne Composite:

```
if (item is File)
{
    HandleFile(...);
}
else if (item is Directory)
{
    foreach (...)
    {
        // wieder unterscheiden
    }
}
```

Mit Composite:

```
item.Operation();
```

Der Client muss die Baumlogik nicht selbst kennen.

---

# Vorteile

- einzelne und zusammengesetzte Objekte können einheitlich behandelt werden;
- hierarchische Baumstrukturen lassen sich natürlich modellieren;
- rekursive Operationen werden einfach;
- Client-Code muss weniger konkrete Typen unterscheiden;
- neue Elementtypen können leicht ergänzt werden;
- komplexe Strukturen lassen sich aus einfachen Elementen zusammensetzen;
- unterstützt Polymorphie.

---

# Nachteile

- sehr allgemeine `Component`-Abstraktionen können zu breit werden;
- bei der transparenten Variante besitzen Leaves eventuell sinnlose Methoden wie `Add()`;
- rekursive Strukturen können beim Debugging schwieriger sein;
- Zyklen müssen gegebenenfalls verhindert werden;
- sehr große Bäume können Performance- oder Speicherprobleme verursachen;
- Regeln wie „dieser Composite darf nur bestimmte Kindtypen enthalten“ können zusätzliche Validierung benötigen.

---

# Wann Composite nicht sinnvoll ist

Wenn keine echte Hierarchie existiert:

```
Customer
Order
Payment
```

und diese Objekte keine rekursive Teil-Ganzes-Struktur bilden, bringt Composite normalerweise nichts.

Auch eine einfache Liste:

```
List<Product>
```

benötigt nicht automatisch Composite.

---

# Composite lohnt sich besonders bei

```
Baumstruktur
+
Teil-Ganzes-Beziehung
+
gleiche Operationen auf Leaf und Composite
```

---

# Ein praktischer Entscheidungsbaum

```
Habe ich eine hierarchische Struktur?
           │
      ┌────┴────┐
     Nein       Ja
      │          │
      ▼          ▼
Kein Composite   Müssen einzelne Elemente
                 und Gruppen ähnlich
                 behandelt werden?
                        │
                   ┌────┴────┐
                  Nein       Ja
                   │          │
                   ▼          ▼
               evtl. kein   Composite
               Composite    sinnvoll
```

---

# Das solltest du dir merken

Gemeinsame Abstraktion:

```
public abstract class Component
{
    public string Name { get; }

    protected Component(string name)
    {
        Name = name;
    }

    public abstract void Operation();
}
```

Leaf:

```
public sealed class Leaf : Component
{
    public Leaf(string name)
        : base(name)
    {
    }

    public override void Operation()
    {
        // Operation für ein einzelnes Element.
    }
}
```

Composite:

```
public sealed class Composite : Component
{
    private readonly List<Component> _children =
        new();

    public Composite(string name)
        : base(name)
    {
    }

    public void Add(Component child)
    {
        _children.Add(child);
    }

    public void Remove(Component child)
    {
        _children.Remove(child);
    }

    public override void Operation()
    {
        // Eigene Verarbeitung ...

        // Danach Operation rekursiv
        // auf alle Kinder anwenden.
        foreach (Component child in _children)
        {
            child.Operation();
        }
    }
}
```

---

# Der entscheidende Code

```
foreach (Component child in _children)
{
    child.Operation();
}
```

Warum ist das so wichtig?

Weil `child` sowohl:

```
Leaf
```

als auch:

```
Composite
```

sein kann.

Wenn es ein Composite ist, läuft die Methode automatisch weiter in die nächste Ebene.

---

# Merksatz

> **Composite fasst einzelne Objekte und Gruppen von Objekten zu einer Baumstruktur zusammen und ermöglicht, beide über dieselbe Abstraktion zu behandeln.**

Noch einfacher:

```
Leaf:
"Ich bin ein einzelnes Element."

Composite:
"Ich bin ein Element,
das weitere Elemente enthält."

Client:
"Für mich seid ihr beide Components."
```

Oder ganz kurz:

```
Composite
=
Baumstruktur
+
Leaf
+
Composite
+
gemeinsame Schnittstelle
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Composite
→ Teil-Ganzes-Baumstruktur


Decorator
→ Verhalten eines Objekts erweitern


Adapter
→ inkompatible Schnittstellen verbinden


Facade
→ komplexes Subsystem vereinfachen


Proxy
→ Zugriff auf Objekt kontrollieren


Bridge
→ Abstraktion und Implementierung trennen
```

---

> [!summary] Zusammenfassung  
> Das **Composite Pattern** ist ein **Strukturmuster**.
> 
> Es modelliert hierarchische **Teil-Ganzes-Beziehungen** als Baum:
> 
> ```
> Component
> ├── Leaf
> └── Composite
>     ├── Leaf
>     ├── Leaf
>     └── Composite
>         └── Leaf
> ```
> 
> Ein `Leaf` ist ein einzelnes Objekt.
> 
> Ein `Composite` kann weitere `Component`-Objekte enthalten.
> 
> Beide besitzen dieselbe gemeinsame Abstraktion:
> 
> ```
> Component
> ```
> 
> Dadurch kann der Client beispielsweise einfach:
> 
> ```
> component.Print();
> ```
> 
> aufrufen, unabhängig davon, ob `component` eine einzelne Datei oder ein kompletter Ordnerbaum ist.
> 
> Die Verarbeitung erfolgt bei Composite-Objekten typischerweise rekursiv:
> 
> ```
> foreach (Component child in _children)
> {
>     child.Operation();
> }
> ```
> 
> Das klassische Beispiel ist ein Dateisystem:
> 
> ```
> Directory
> ├── File
> ├── File
> └── Directory
>     ├── File
>     └── File
> ```
> 
> **Kurz gesagt:**  
> `Composite = Einzelobjekte und Objektgruppen als einheitliche Baumstruktur behandeln.`