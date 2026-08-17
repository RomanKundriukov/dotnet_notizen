Das **Visitor Pattern** ist ein Verhaltensmuster, mit dem neue Operationen für Objekte unterschiedlicher Klassen definiert werden können, **ohne diese Klassen selbst ständig verändern zu müssen**.

Dabei entstehen zwei getrennte Hierarchien:

```text
1. Elemente
   → Objekte, mit denen gearbeitet wird

2. Visitor
   → Operationen, die auf diesen Objekten ausgeführt werden
```

> [!info] Grundidee  
> Die Datenstruktur bleibt möglichst stabil.
> 
> Neue Operationen werden nicht direkt in die Element-Klassen eingebaut, sondern als eigene **Visitor-Klassen** ergänzt.

---

# Einfaches Beispiel

Angenommen, wir haben verschiedene Bankkunden:

```text
Person
Company
```

Später möchten wir diese Objekte exportieren als:

```text
HTML
XML
JSON
CSV
...
```

Ohne Visitor könnte man direkt Methoden hinzufügen:

```csharp
person.ToHtml();
person.ToXml();
person.ToJson();
```

und bei `Company` genauso.

Das Problem:

```text
Person
├── ToHtml()
├── ToXml()
├── ToJson()
├── ToCsv()
└── ...

Company
├── ToHtml()
├── ToXml()
├── ToJson()
├── ToCsv()
└── ...
```

Bei jedem neuen Exportformat müssten die bestehenden Klassen verändert werden.

Mit Visitor:

```text
Person ─────┐
            │
Company ────┼──► HtmlVisitor
            │
            ├──► XmlVisitor
            │
            └──► JsonVisitor
```

Die Klassen `Person` und `Company` bleiben weitgehend unverändert.

---

# Wann sollte man Visitor verwenden?

Visitor eignet sich besonders:

- wenn viele unterschiedliche Objektklassen existieren;
    
- wenn auf diesen Objekten ähnliche Operationen ausgeführt werden sollen;
    
- wenn häufig **neue Operationen** dazukommen;
    
- wenn die Struktur der Element-Klassen relativ stabil bleibt;
    
- wenn man zusätzliche Funktionalität nicht direkt in die bestehenden Klassen einbauen möchte.
    

Typischer Fall:

```text
Elementtypen ändern sich selten
+
neue Operationen kommen häufig hinzu
=
Visitor kann sinnvoll sein
```

---

# Wann ist Visitor besonders passend?

Stellen wir uns folgende stabile Klassen vor:

```text
Person
Company
BankAccount
CreditAccount
```

Die Klassen ändern sich kaum.

Dagegen kommen immer wieder neue Operationen hinzu:

```text
HTML exportieren
XML exportieren
JSON exportieren
Validieren
Statistik berechnen
Report erzeugen
```

Dann kann Visitor diese Operationen außerhalb der Element-Klassen kapseln.

---

# Grundstruktur

```text
                   IVisitor
                      ▲
           ┌──────────┴──────────┐
           │                     │
      HtmlVisitor            XmlVisitor


                   IElement
                      ▲
           ┌──────────┴──────────┐
           │                     │
        ElementA              ElementB
```

Die Verbindung erfolgt über:

```csharp
element.Accept(visitor);
```

---

# Teilnehmer des Patterns

## Visitor

Der Visitor definiert eine passende `Visit`-Methode für jeden konkreten Elementtyp.

Zum Beispiel:

```csharp
public interface IVisitor
{
    void Visit(ElementA element);

    void Visit(ElementB element);
}
```

---

## ConcreteVisitor

Ein konkreter Visitor implementiert eine bestimmte Operation.

Zum Beispiel:

```text
HtmlVisitor
XmlVisitor
JsonVisitor
```

Jeder Visitor kennt die konkrete Verarbeitung für:

```text
ElementA
ElementB
...
```

---

## Element

Das Element definiert:

```csharp
Accept(IVisitor visitor);
```

Über diese Methode akzeptiert das Objekt einen Visitor.

---

## ConcreteElement

Konkrete Elemente implementieren `Accept()`.

Beispiel:

```csharp
public void Accept(IVisitor visitor)
{
    visitor.Visit(this);
}
```

---

## ObjectStructure

Die `ObjectStructure` enthält mehrere Elemente.

Zum Beispiel:

```text
Bank
├── Person
├── Person
├── Company
└── Company
```

Sie kann den Visitor auf alle Elemente anwenden.

---

# Klassische Struktur in C#

```csharp
// Gemeinsamer Vertrag für alle Visitor.
public interface IVisitor
{
    void Visit(ElementA element);

    void Visit(ElementB element);
}
```

---

# Element

```csharp
// Gemeinsamer Vertrag für alle Elemente.
public interface IElement
{
    void Accept(IVisitor visitor);
}
```

---

# ElementA

```csharp
public class ElementA : IElement
{
    public void Accept(IVisitor visitor)
    {
        // Dieses Objekt teilt dem Visitor mit,
        // dass es sich um ElementA handelt.
        visitor.Visit(this);
    }

    public void OperationA()
    {
        Console.WriteLine(
            "Operation von ElementA");
    }
}
```

---

# ElementB

```csharp
public class ElementB : IElement
{
    public void Accept(IVisitor visitor)
    {
        // Der Visitor erhält das konkrete ElementB.
        visitor.Visit(this);
    }

    public void OperationB()
    {
        Console.WriteLine(
            "Operation von ElementB");
    }
}
```

---

# ConcreteVisitor

```csharp
public class ConcreteVisitor : IVisitor
{
    public void Visit(ElementA element)
    {
        Console.WriteLine(
            "Visitor verarbeitet ElementA.");

        element.OperationA();
    }

    public void Visit(ElementB element)
    {
        Console.WriteLine(
            "Visitor verarbeitet ElementB.");

        element.OperationB();
    }
}
```

---

# ObjectStructure

```csharp
public class ObjectStructure
{
    private readonly List<IElement> _elements =
        new();

    public void Add(IElement element)
    {
        _elements.Add(element);
    }

    public void Remove(IElement element)
    {
        _elements.Remove(element);
    }

    public void Accept(IVisitor visitor)
    {
        // Visitor auf alle Elemente anwenden.
        foreach (IElement element in _elements)
        {
            element.Accept(visitor);
        }
    }
}
```

---

# Verwendung

```csharp
ObjectStructure structure =
    new ObjectStructure();

structure.Add(
    new ElementA());

structure.Add(
    new ElementB());

IVisitor visitor =
    new ConcreteVisitor();

structure.Accept(visitor);
```

---

# Ablauf

```text
ObjectStructure
      │
      ▼
   ElementA
      │
      │ Accept(visitor)
      ▼
Visitor.Visit(ElementA)
```

Danach:

```text
ObjectStructure
      │
      ▼
   ElementB
      │
      │ Accept(visitor)
      ▼
Visitor.Visit(ElementB)
```

---

# Was passiert bei `Accept()`?

Das ist der wichtigste Teil des Patterns.

Bei `ElementA`:

```csharp
public void Accept(IVisitor visitor)
{
    visitor.Visit(this);
}
```

`this` ist hier konkret ein:

```text
ElementA
```

Dadurch wird:

```csharp
Visit(ElementA element)
```

aufgerufen.

Bei `ElementB`:

```csharp
public void Accept(IVisitor visitor)
{
    visitor.Visit(this);
}
```

ist `this` dagegen ein:

```text
ElementB
```

und deshalb wird:

```csharp
Visit(ElementB element)
```

aufgerufen.

---

# Double Dispatch

Diese Technik wird als:

**Double Dispatch**

bezeichnet.

Die tatsächlich ausgeführte Operation hängt von **zwei Typen** ab:

```text
1. vom konkreten Visitor
2. vom konkreten Element
```

Beispiel:

```text
HtmlVisitor + Person
→ HTML für Person erzeugen

HtmlVisitor + Company
→ HTML für Company erzeugen

XmlVisitor + Person
→ XML für Person erzeugen

XmlVisitor + Company
→ XML für Company erzeugen
```

---

# Warum heißt es Double Dispatch?

Bei einem normalen virtuellen Methodenaufruf entscheidet typischerweise der Laufzeittyp eines Objekts.

Bei Visitor kommen zwei Entscheidungen zusammen.

Erste Entscheidung:

```csharp
element.Accept(visitor);
```

Welche `Accept()`-Implementierung wird verwendet?

```text
Person?
Company?
```

Zweite Entscheidung:

```csharp
visitor.Visit(this);
```

Welcher Visitor arbeitet damit?

```text
HtmlVisitor?
XmlVisitor?
```

Also:

```text
konkretes Element
+
konkreter Visitor
=
konkrete Operation
```

---

# Praktisches Beispiel: Bankkunden

Wir haben zwei unterschiedliche Kontoinhaber:

```text
Person
Company
```

---

# Problem ohne Visitor

Man könnte zunächst schreiben:

```csharp
public interface IAccount
{
    void ToHtml();
}
```

Dann:

```csharp
public class Person : IAccount
{
    public string Name { get; set; } = string.Empty;

    public string AccountNumber { get; set; } =
        string.Empty;

    public void ToHtml()
    {
        // HTML erzeugen.
    }
}
```

und:

```csharp
public class Company : IAccount
{
    public string Name { get; set; } = string.Empty;

    public string RegistrationNumber { get; set; } =
        string.Empty;

    public string AccountNumber { get; set; } =
        string.Empty;

    public void ToHtml()
    {
        // HTML erzeugen.
    }
}
```

---

# Neues Format hinzufügen

Später möchten wir:

```text
XML
```

unterstützen.

Dann müssten wir:

```csharp
void ToXml();
```

zu `IAccount` hinzufügen.

Danach müssen:

```text
Person
Company
```

ebenfalls angepasst werden.

---

Später kommt:

```text
JSON
```

Dann:

```csharp
void ToJson();
```

und wieder müssen alle Klassen geändert werden.

---

# Problem

Die Klassen werden immer größer:

```text
Person
├── Geschäftsdaten
├── ToHtml()
├── ToXml()
├── ToJson()
├── ToCsv()
└── ...

Company
├── Geschäftsdaten
├── ToHtml()
├── ToXml()
├── ToJson()
├── ToCsv()
└── ...
```

Dabei gehört die gesamte Serialisierungslogik eigentlich nicht unbedingt zur Kernverantwortung dieser Domain-Klassen.

---

# Lösung mit Visitor

Wir lassen die Accounts nur noch Visitor akzeptieren.

```csharp
public interface IAccount
{
    void Accept(IAccountVisitor visitor);
}
```

Die Serialisierungslogik wird ausgelagert:

```text
IAccountVisitor
├── HtmlVisitor
├── XmlVisitor
└── JsonVisitor
```

---

# Visitor Interface

```csharp
public interface IAccountVisitor
{
    // Verarbeitung einer natürlichen Person.
    void Visit(Person person);

    // Verarbeitung eines Unternehmens.
    void Visit(Company company);
}
```

---

# Person

```csharp
public class Person : IAccount
{
    public string Name { get; set; } =
        string.Empty;

    public string AccountNumber { get; set; } =
        string.Empty;

    public void Accept(IAccountVisitor visitor)
    {
        // Der Visitor erhält die konkrete Person.
        visitor.Visit(this);
    }
}
```

Wichtig:

`Person` besitzt jetzt kein:

```csharp
ToHtml();
ToXml();
ToJson();
```

mehr.

---

# Company

```csharp
public class Company : IAccount
{
    public string Name { get; set; } =
        string.Empty;

    public string RegistrationNumber { get; set; } =
        string.Empty;

    public string AccountNumber { get; set; } =
        string.Empty;

    public void Accept(IAccountVisitor visitor)
    {
        // Der Visitor erhält das konkrete Unternehmen.
        visitor.Visit(this);
    }
}
```

---

# HtmlVisitor

```csharp
public class HtmlVisitor : IAccountVisitor
{
    public void Visit(Person person)
    {
        string result =
            $"""
            <table>
                <tr>
                    <td>Name</td>
                    <td>{person.Name}</td>
                </tr>
                <tr>
                    <td>Kontonummer</td>
                    <td>{person.AccountNumber}</td>
                </tr>
            </table>
            """;

        Console.WriteLine(result);
    }

    public void Visit(Company company)
    {
        string result =
            $"""
            <table>
                <tr>
                    <td>Name</td>
                    <td>{company.Name}</td>
                </tr>
                <tr>
                    <td>Registrierungsnummer</td>
                    <td>{company.RegistrationNumber}</td>
                </tr>
                <tr>
                    <td>Kontonummer</td>
                    <td>{company.AccountNumber}</td>
                </tr>
            </table>
            """;

        Console.WriteLine(result);
    }
}
```

---

# XmlVisitor

```csharp
public class XmlVisitor : IAccountVisitor
{
    public void Visit(Person person)
    {
        string result =
            $"""
            <Person>
                <Name>{person.Name}</Name>
                <AccountNumber>
                    {person.AccountNumber}
                </AccountNumber>
            </Person>
            """;

        Console.WriteLine(result);
    }

    public void Visit(Company company)
    {
        string result =
            $"""
            <Company>
                <Name>{company.Name}</Name>
                <RegistrationNumber>
                    {company.RegistrationNumber}
                </RegistrationNumber>
                <AccountNumber>
                    {company.AccountNumber}
                </AccountNumber>
            </Company>
            """;

        Console.WriteLine(result);
    }
}
```

---

# ObjectStructure: Bank

```csharp
public class Bank
{
    private readonly List<IAccount> _accounts =
        new();

    public void Add(IAccount account)
    {
        _accounts.Add(account);
    }

    public void Remove(IAccount account)
    {
        _accounts.Remove(account);
    }

    public void Accept(IAccountVisitor visitor)
    {
        // Visitor besucht jedes Konto.
        foreach (IAccount account in _accounts)
        {
            account.Accept(visitor);
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
        Bank bank =
            new Bank();

        bank.Add(
            new Person
            {
                Name = "Ivan Alekseev",
                AccountNumber = "82184931"
            });

        bank.Add(
            new Company
            {
                Name = "Microsoft",
                RegistrationNumber =
                    "EWUIR32141324",
                AccountNumber =
                    "3424131445"
            });

        // Alle Accounts als HTML verarbeiten.
        bank.Accept(
            new HtmlVisitor());

        // Dieselben Accounts als XML verarbeiten.
        bank.Accept(
            new XmlVisitor());
    }
}
```

---

# Ablauf beim HtmlVisitor

Wir starten:

```csharp
bank.Accept(
    new HtmlVisitor());
```

Die Bank läuft über alle Accounts:

```csharp
foreach (IAccount account in _accounts)
{
    account.Accept(visitor);
}
```

---

## Person

Wenn das aktuelle Element eine:

```text
Person
```

ist:

```csharp
person.Accept(htmlVisitor);
```

führt zu:

```csharp
htmlVisitor.Visit(person);
```

Also:

```text
Person
  │
  │ Accept(HtmlVisitor)
  ▼
HtmlVisitor
  │
  ▼
Visit(Person)
```

---

## Company

Beim Unternehmen:

```csharp
company.Accept(htmlVisitor);
```

führt zu:

```csharp
htmlVisitor.Visit(company);
```

Also:

```text
Company
  │
  │ Accept(HtmlVisitor)
  ▼
HtmlVisitor
  │
  ▼
Visit(Company)
```

---

# Der gleiche Objektbaum, andere Operation

Jetzt:

```csharp
bank.Accept(
    new XmlVisitor());
```

Die Elementstruktur ist unverändert:

```text
Bank
├── Person
└── Company
```

Nur die Operation wechselt:

```text
HtmlVisitor
     ↓
XmlVisitor
```

Das ist die zentrale Stärke von Visitor.

---

# Neues Verhalten hinzufügen

Angenommen, wir möchten jetzt zusätzlich JSON unterstützen.

Ohne Visitor müssten wir wahrscheinlich:

```text
Person ändern
Company ändern
IAccount ändern
```

Mit Visitor erstellen wir:

```csharp
public class JsonVisitor : IAccountVisitor
{
    public void Visit(Person person)
    {
        Console.WriteLine(
            $$"""
            {
                "name": "{{person.Name}}",
                "accountNumber": "{{person.AccountNumber}}"
            }
            """);
    }

    public void Visit(Company company)
    {
        Console.WriteLine(
            $$"""
            {
                "name": "{{company.Name}}",
                "registrationNumber": "{{company.RegistrationNumber}}",
                "accountNumber": "{{company.AccountNumber}}"
            }
            """);
    }
}
```

Danach:

```csharp
bank.Accept(
    new JsonVisitor());
```

Die Klassen:

```text
Person
Company
Bank
```

müssen dafür nicht verändert werden.

---

# Große Stärke von Visitor

Visitor ist besonders gut, wenn:

```text
Elementstruktur stabil
        │
        ▼
Person
Company
...
```

aber:

```text
Operationen ändern sich häufig
        │
        ├── HTML
        ├── XML
        ├── JSON
        ├── CSV
        └── Report
```

---

# Der wichtige Nachteil

Was passiert, wenn wir einen komplett neuen Elementtyp hinzufügen?

Zum Beispiel:

```text
GovernmentAccount
```

Dann muss das Visitor-Interface erweitert werden:

```csharp
public interface IAccountVisitor
{
    void Visit(Person person);

    void Visit(Company company);

    void Visit(GovernmentAccount account);
}
```

Und anschließend müssen **alle existierenden Visitor** ebenfalls angepasst werden:

```text
HtmlVisitor
→ Visit(GovernmentAccount)

XmlVisitor
→ Visit(GovernmentAccount)

JsonVisitor
→ Visit(GovernmentAccount)
```

---

# Deshalb ist die wichtigste Regel

> [!warning] Sehr wichtig  
> Visitor ist gut, wenn **neue Operationen häufig** hinzukommen, aber die **Elementtypen stabil** bleiben.
> 
> Wenn dagegen ständig neue Elementtypen hinzukommen, kann Visitor sehr aufwendig werden.

---

# Zwei Änderungsrichtungen

Das lässt sich sehr gut so merken:

## Neue Operation hinzufügen

Zum Beispiel:

```text
PdfVisitor
```

Einfach:

```text
neue Visitor-Klasse
```

Bestehende Element-Klassen müssen normalerweise nicht geändert werden.

✅ Sehr angenehm.

---

## Neues Element hinzufügen

Zum Beispiel:

```text
PrivateEntrepreneur
```

Dann:

```text
IVisitor ändern
HtmlVisitor ändern
XmlVisitor ändern
JsonVisitor ändern
...
```

❌ Aufwendig.

---

# Visitor dreht das Erweiterungsproblem um

Ohne Visitor ist häufig:

```text
Neue Operation
→ alle Elementklassen ändern
```

Mit Visitor:

```text
Neue Operation
→ neue Visitor-Klasse
```

Dafür gilt aber:

```text
Neuer Elementtyp
→ alle Visitor ändern
```

---

# Double Dispatch im Detail

Betrachten wir:

```csharp
IAccount account =
    new Person();

IAccountVisitor visitor =
    new HtmlVisitor();

account.Accept(visitor);
```

Obwohl die Variable den Typ:

```text
IAccount
```

hat, ist das konkrete Objekt:

```text
Person
```

Deshalb wird:

```csharp
Person.Accept(...)
```

aufgerufen.

Darin:

```csharp
visitor.Visit(this);
```

`this` besitzt jetzt konkret den Typ:

```text
Person
```

Deshalb wird:

```csharp
HtmlVisitor.Visit(Person)
```

aufgerufen.

---

# Ablauf

```text
IAccount
   │
   │ Laufzeittyp
   ▼
 Person
   │
   │ Accept(HtmlVisitor)
   ▼
HtmlVisitor
   │
   │ Visit(Person)
   ▼
HTML für Person
```

---

# Warum nicht einfach `switch` verwenden?

Man könnte auch schreiben:

```csharp
switch (account)
{
    case Person person:
        // Person behandeln
        break;

    case Company company:
        // Company behandeln
        break;
}
```

Für kleine Systeme kann das durchaus ausreichen.

Visitor wird interessanter, wenn:

```text
viele unterschiedliche Elementtypen
+
viele unterschiedliche Operationen
+
stabile Elementstruktur
```

vorliegen.

> [!tip]  
> Nicht jedes `switch` auf Objekttypen muss automatisch durch Visitor ersetzt werden.

---

# Visitor vs. Strategy

## Strategy

Strategy kapselt unterschiedliche Möglichkeiten, **wie eine bestimmte Aufgabe ausgeführt wird**.

```text
PaymentStrategy
├── PayPal
├── Bank
└── CreditCard
```

Der Context verwendet eine ausgewählte Strategie.

---

## Visitor

Visitor definiert eine Operation für **verschiedene unterschiedliche Elementtypen**.

```text
HtmlVisitor
├── Visit(Person)
├── Visit(Company)
└── Visit(...)
```

Merksatz:

```text
Strategy
→ Wie führe ich eine Aufgabe aus?

Visitor
→ Welche Operation führe ich
  auf verschiedenen Elementtypen aus?
```

---

# Visitor vs. Iterator

Diese Patterns können gemeinsam auftreten.

## Iterator

```text
Wie komme ich durch alle Elemente?
```

Beispiel:

```csharp
foreach (IAccount account in accounts)
```

---

## Visitor

```text
Was mache ich mit jedem konkreten Elementtyp?
```

Beispiel:

```csharp
account.Accept(visitor);
```

Also:

```text
Iterator
→ Elemente durchlaufen

Visitor
→ Operation auf Elementen ausführen
```

---

# Visitor vs. Composite

Visitor wird häufig zusammen mit Composite eingesetzt.

Angenommen:

```text
Folder
├── File
├── File
└── Folder
    ├── File
    └── File
```

Das ist eine Composite-Struktur.

Ein Visitor könnte dann verschiedene Operationen durchführen:

```text
SizeVisitor
→ Gesamtgröße berechnen

SearchVisitor
→ Dateien suchen

ExportVisitor
→ Struktur exportieren
```

---

# Visitor und Open/Closed Principle

Visitor unterstützt das Open/Closed Principle besonders bezüglich **neuer Operationen**.

Beispiel:

Bereits vorhanden:

```text
Person
Company
```

Neue Funktion:

```text
CsvVisitor
```

kann ergänzt werden, ohne `Person` und `Company` mit `ToCsv()` verändern zu müssen.

---

# Visitor und Single Responsibility Principle

Ohne Visitor:

```text
Person
├── Kundendaten
├── Geschäftslogik
├── HTML-Serialisierung
├── XML-Serialisierung
└── JSON-Serialisierung
```

Mit Visitor:

```text
Person
└── Kundendaten


HtmlVisitor
└── HTML


XmlVisitor
└── XML


JsonVisitor
└── JSON
```

Dadurch sind Verantwortlichkeiten stärker getrennt.

---

# Vorteile

- neue Operationen können relativ einfach ergänzt werden;
    
- Element-Klassen bleiben schlanker;
    
- unterschiedliche Operationen werden sauber voneinander getrennt;
    
- zusammengehörige Logik befindet sich in einer Visitor-Klasse;
    
- eignet sich gut für stabile Objektstrukturen;
    
- kann große Mengen typabhängiger Logik zentralisieren;
    
- unterstützt Double Dispatch.
    

---

# Nachteile

- neue Elementtypen sind teuer hinzuzufügen;
    
- alle Visitor müssen bei neuen Elementtypen angepasst werden;
    
- das Pattern erzeugt zusätzliche Klassen;
    
- Double Dispatch ist zunächst schwieriger zu verstehen;
    
- Visitor benötigt oft Zugriff auf Daten der Elemente;
    
- dadurch kann die Kapselung schwieriger werden;
    
- bei kleinen Objektstrukturen ist Visitor häufig unnötig komplex.
    

---

# Wann Visitor nicht sinnvoll ist

Visitor ist meistens keine gute Wahl, wenn die Elementstruktur häufig verändert wird.

Zum Beispiel:

```text
heute:
Person
Company

morgen:
Person
Company
Freelancer

übermorgen:
Person
Company
Freelancer
Government
Association
Foundation
...
```

Dann müssten ständig alle Visitor erweitert werden.

---

# Wann Visitor besonders sinnvoll ist

Sehr passend:

```text
Elemente ändern sich selten:

Person
Company


Operationen ändern sich häufig:

HtmlVisitor
XmlVisitor
JsonVisitor
CsvVisitor
StatisticsVisitor
ValidationVisitor
```

---

# Klassische Struktur

```text
                          IVisitor
                             ▲
                 ┌───────────┴───────────┐
                 │                       │
            VisitorA                VisitorB
                 │                       │
        ┌────────┴────────┐      ┌───────┴────────┐
        ▼                 ▼      ▼                ▼
   Visit(ElementA)   Visit(ElementB)          ...


                          IElement
                             ▲
                 ┌───────────┴───────────┐
                 │                       │
              ElementA                ElementB
                 │                       │
          Accept(visitor)         Accept(visitor)
```

---

# Der wichtigste Ablauf

```text
Client
  │
  ▼
element.Accept(visitor)
  │
  ▼
visitor.Visit(this)
  │
  ▼
konkrete Operation
für konkreten Elementtyp
```

---

# Das solltest du dir merken

Visitor:

```csharp
public interface IVisitor
{
    void Visit(ElementA element);

    void Visit(ElementB element);
}
```

Element:

```csharp
public interface IElement
{
    void Accept(IVisitor visitor);
}
```

ConcreteElement:

```csharp
public class ElementA : IElement
{
    public void Accept(IVisitor visitor)
    {
        visitor.Visit(this);
    }
}
```

ConcreteVisitor:

```csharp
public class ConcreteVisitor : IVisitor
{
    public void Visit(ElementA element)
    {
        // Operation für ElementA.
    }

    public void Visit(ElementB element)
    {
        // Operation für ElementB.
    }
}
```

---

# Merksatz

> **Visitor ermöglicht es, neue Operationen für bestehende Objektklassen hinzuzufügen, ohne diese Operationen direkt in den Klassen selbst zu implementieren.**

Noch einfacher:

```text
Element:
"Ich weiß, was ich bin."

Visitor:
"Ich weiß, was ich mit diesem Element tun möchte."
```

Oder:

```text
Element.Accept(visitor)
        ↓
Visitor.Visit(Element)
```

---

# Wichtigste Entscheidung

```text
Ändern sich häufig die Operationen?
        │
        └── Ja
            → Visitor kann sinnvoll sein


Ändern sich häufig die Elementtypen?
        │
        └── Ja
            → Visitor eher problematisch
```

---

> [!summary] Zusammenfassung  
> Das **Visitor Pattern** ist ein **Verhaltensmuster**.
> 
> Es trennt eine Objektstruktur von den Operationen, die auf dieser Struktur ausgeführt werden.
> 
> Die Elemente definieren nur:
> 
> ```csharp
> Accept(visitor);
> ```
> 
> und der Visitor definiert eine passende Operation für jeden konkreten Elementtyp:
> 
> ```csharp
> Visit(Person person);
> Visit(Company company);
> ```
> 
> Der zentrale Mechanismus ist:
> 
> ```csharp
> visitor.Visit(this);
> ```
> 
> Dadurch entsteht **Double Dispatch**: Die ausgeführte Operation hängt sowohl vom konkreten Visitor als auch vom konkreten Elementtyp ab.
> 
> Visitor ist besonders sinnvoll, wenn:
> 
> ```text
> Elementstruktur stabil
> +
> neue Operationen häufig
> ```
> 
> Neue Visitor wie:
> 
> ```text
> HtmlVisitor
> XmlVisitor
> JsonVisitor
> ```
> 
> können dann ergänzt werden, ohne die bestehenden Elementklassen mit immer mehr Operationen zu belasten.
> 
> Der wichtigste Nachteil:
> 
> ```text
> Neuer Elementtyp
> → alle Visitor müssen erweitert werden
> ```
> 
> **Kurz gesagt:**  
> `Visitor = neue Operationen auf einer stabilen Objektstruktur hinzufügen, ohne die Operationen direkt in die Elementklassen einzubauen.`