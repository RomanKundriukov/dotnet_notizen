Das **Adapter Pattern** ist ein Strukturmuster, das die Schnittstelle einer bestehenden Klasse in eine andere Schnittstelle übersetzt.

Dadurch können Klassen zusammenarbeiten, deren Interfaces ursprünglich **nicht kompatibel** sind.

> [!info] Grundidee  
> Der Client erwartet eine bestimmte Schnittstelle.
> 
> Eine vorhandene Klasse besitzt jedoch eine andere Schnittstelle.
> 
> Der Adapter sitzt dazwischen und **übersetzt die Aufrufe**.

Vereinfacht:

```
Client
  │
  │ erwartet ITarget
  ▼
Adapter
  │
  │ übersetzt
  ▼
Adaptee
```

---

# Das typische Problem

Angenommen, unser Client arbeitet ausschließlich mit:

```
ITransport
```

und erwartet:

```
void Drive();
```

Eine vorhandene Klasse `Camel` besitzt jedoch:

```
void Move();
```

Also:

```
Client erwartet:

Drive()


Camel bietet:

Move()
```

Die beiden Interfaces passen nicht zusammen.

Der Adapter macht daraus:

```
Drive()
  │
  ▼
Adapter
  │
  ▼
Move()
```

---

# Wann sollte man Adapter verwenden?

Das Adapter Pattern eignet sich besonders:

- wenn eine existierende Klasse verwendet werden soll, deren Interface nicht zum Client passt;
- wenn zwei vorhandene Komponenten mit inkompatiblen Schnittstellen zusammenarbeiten sollen;
- wenn eine externe Bibliothek in die eigene Architektur integriert werden soll;
- wenn Legacy-Code verwendet werden muss;
- wenn eine Drittanbieter-API nicht verändert werden kann;
- wenn der Client nicht von einer fremden Schnittstelle abhängig werden soll.

Typische Situation:

```
Eigene Anwendung
      │
      │ erwartet eigenes Interface
      ▼
    Adapter
      │
      │ übersetzt
      ▼
Third-Party Library
```

---

# Grundstruktur

```
                   Target
                     ▲
                     │
                  Adapter
                     │
                     │ enthält
                     ▼
                   Adaptee
```

Der Client arbeitet nur mit:

```
Target
```

Der `Adapter` implementiert beziehungsweise erbt `Target` und verwendet intern den `Adaptee`.

---

# Teilnehmer des Patterns

## Target

`Target` definiert die Schnittstelle, die der Client erwartet.

Beispiel:

```
public interface ITarget
{
    void Request();
}
```

---

## Client

Der Client arbeitet ausschließlich mit dem `Target`.

```
public class Client
{
    public void Execute(ITarget target)
    {
        target.Request();
    }
}
```

Der Client muss nicht wissen, ob er tatsächlich mit:

```
ConcreteTarget
```

oder einem:

```
Adapter
```

arbeitet.

---

## Adaptee

Der `Adaptee` ist die bereits existierende Klasse, deren Schnittstelle nicht zum Client passt.

```
public class Adaptee
{
    public void SpecificRequest()
    {
        Console.WriteLine(
            "Spezifische Operation des Adaptee.");
    }
}
```

---

## Adapter

Der Adapter implementiert das erwartete Interface:

```
ITarget
```

und verwendet intern:

```
Adaptee
```

Beispiel:

```
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;

    public Adapter(Adaptee adaptee)
    {
        _adaptee = adaptee;
    }

    public void Request()
    {
        // Übersetzt den erwarteten Request
        // in den Aufruf der bestehenden Klasse.
        _adaptee.SpecificRequest();
    }
}
```

---

# Allgemeines Beispiel

Adaptee:

```
public class LegacyService
{
    public void ExecuteLegacyOperation()
    {
        Console.WriteLine(
            "Legacy-Operation wird ausgeführt.");
    }
}
```

Der Client erwartet jedoch:

```
public interface IService
{
    void Execute();
}
```

Adapter:

```
public class LegacyServiceAdapter : IService
{
    private readonly LegacyService _legacyService;

    public LegacyServiceAdapter(
        LegacyService legacyService)
    {
        _legacyService = legacyService;
    }

    public void Execute()
    {
        // Die moderne Schnittstelle wird
        // auf die alte Schnittstelle abgebildet.
        _legacyService.ExecuteLegacyOperation();
    }
}
```

Verwendung:

```
LegacyService legacyService =
    new LegacyService();

IService service =
    new LegacyServiceAdapter(legacyService);

service.Execute();
```

Der Client sieht nur:

```
IService
```

und muss `LegacyService` nicht kennen.

---

# Ablauf

```
Client
  │
  │ Execute()
  ▼
LegacyServiceAdapter
  │
  │ ExecuteLegacyOperation()
  ▼
LegacyService
```

---

# Praktisches Beispiel: Auto und Kamel

Angenommen, ein Reisender kann mit verschiedenen Transportmitteln reisen.

Der Reisende erwartet:

```
ITransport
```

mit:

```
void Drive();
```

---

# Target: ITransport

```
public interface ITransport
{
    void Drive();
}
```

---

# Concrete Target: Auto

Ein Auto passt bereits direkt zu diesem Interface:

```
public class Car : ITransport
{
    public void Drive()
    {
        Console.WriteLine(
            "Das Auto fährt auf der Straße.");
    }
}
```

---

# Client: Driver

```
public class Driver
{
    public void Travel(ITransport transport)
    {
        // Driver kennt ausschließlich ITransport.
        transport.Drive();
    }
}
```

Der `Driver` arbeitet also mit allem, was:

```
ITransport
```

implementiert.

---

# Adaptee: Camel

Nun existiert aber ein Kamel.

```
public interface IAnimal
{
    void Move();
}
```

Implementierung:

```
public class Camel : IAnimal
{
    public void Move()
    {
        Console.WriteLine(
            "Das Kamel bewegt sich durch die Wüste.");
    }
}
```

Das Problem:

```
Driver erwartet:

ITransport.Drive()


Camel besitzt:

IAnimal.Move()
```

`Camel` ist also kein `ITransport`.

---

# Ohne Adapter

Das funktioniert nicht:

```
Driver driver =
    new Driver();

Camel camel =
    new Camel();

// ❌ Camel implementiert ITransport nicht.
// driver.Travel(camel);
```

Denn:

```
Travel(ITransport transport)
```

erwartet ein `ITransport`.

---

# Lösung: CamelToTransportAdapter

```
public class CamelToTransportAdapter
    : ITransport
{
    private readonly Camel _camel;

    public CamelToTransportAdapter(
        Camel camel)
    {
        _camel = camel;
    }

    public void Drive()
    {
        // Der Client erwartet Drive().
        //
        // Intern übersetzen wir diesen Aufruf
        // in Move() des Kamels.
        _camel.Move();
    }
}
```

---

# Verwendung

```
public class Program
{
    public static void Main()
    {
        Driver driver =
            new Driver();

        // Normales Transportmittel.
        ITransport car =
            new Car();

        driver.Travel(car);

        // Bestehendes inkompatibles Objekt.
        Camel camel =
            new Camel();

        // Camel wird an ITransport angepasst.
        ITransport camelTransport =
            new CamelToTransportAdapter(camel);

        driver.Travel(camelTransport);
    }
}
```

Ausgabe:

```
Das Auto fährt auf der Straße.
Das Kamel bewegt sich durch die Wüste.
```

---

# Was passiert genau?

Bei:

```
driver.Travel(car);
```

ist `car` direkt ein:

```
ITransport
```

Also:

```
Driver
  │
  │ Drive()
  ▼
Car
```

---

Beim Kamel:

```
driver.Travel(camelTransport);
```

ist:

```
camelTransport
=
CamelToTransportAdapter
```

Also:

```
Driver
  │
  │ Drive()
  ▼
CamelToTransportAdapter
  │
  │ Move()
  ▼
Camel
```

---

# Der Adapter übersetzt die Schnittstelle

Der entscheidende Code ist:

```
public void Drive()
{
    _camel.Move();
}
```

Von außen:

```
Drive()
```

Intern:

```
Move()
```

Der Adapter verbindet also zwei inkompatible Schnittstellen.

---

# Warum nicht einfach `Camel : ITransport`?

Man könnte theoretisch schreiben:

```
public class Camel :
    IAnimal,
    ITransport
{
    public void Move()
    {
        // ...
    }

    public void Drive()
    {
        Move();
    }
}
```

Manchmal wäre das tatsächlich ausreichend.

Aber häufig ist das nicht möglich oder nicht sinnvoll.

---

# Fall 1: Fremde Bibliothek

Vielleicht stammt `Camel` aus einer externen Bibliothek:

```
ThirdParty.Animals.Camel
```

Dann können wir die Klasse nicht verändern.

Der Adapter ist ideal:

```
ThirdParty Camel
       │
       ▼
     Adapter
       │
       ▼
eigene Schnittstelle
```

---

# Fall 2: Verantwortlichkeiten nicht vermischen

Ein Kamel ist fachlich:

```
IAnimal
```

und besitzt:

```
Move();
```

Die Methode:

```
Drive();
```

existiert nur deshalb, weil unser `Driver` dieses Interface erwartet.

Es wäre fragwürdig, `Camel` ausschließlich wegen eines bestimmten Clients zu verändern.

Der Adapter hält diese Übersetzungslogik außerhalb des Domain-Objekts.

---

# Fall 3: Mehrere Zielinterfaces

Vielleicht möchten verschiedene Systeme das Kamel unterschiedlich verwenden:

```
Travel-System
→ ITransport

Animal-System
→ IAnimal

Game-System
→ IMovableEntity
```

Dann könnten unterschiedliche Adapter existieren:

```
Camel
├── CamelToTransportAdapter
└── CamelToMovableEntityAdapter
```

Die ursprüngliche Klasse bleibt unverändert.

---

# Object Adapter

Das bisherige Beispiel ist ein **Object Adapter**.

Er verwendet Komposition:

```
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;
}
```

Struktur:

```
Adapter
  │
  └── HAS-A Adaptee
```

---

# Warum Object Adapter?

Der Adapter enthält das zu adaptierende Objekt:

```
private readonly Camel _camel;
```

Das ist:

```
HAS-A
```

also Komposition.

Diese Variante ist in C# sehr flexibel und sehr typisch.

---

# Class Adapter

In manchen Sprachen kann ein Adapter durch Mehrfachvererbung sowohl von `Target` als auch von `Adaptee` erben.

Vereinfacht:

```
       Target      Adaptee
          ▲          ▲
           \        /
            Adapter
```

C# unterstützt jedoch keine Mehrfachvererbung von Klassen.

Deshalb wird in C# meistens ein Object Adapter über:

```
Interface
+
Komposition
```

verwendet.

Beispiel:

```
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;
}
```

---

# Adapter kann mehr als Methoden umbenennen

Ein Adapter muss nicht nur:

```
Drive()
→ Move()
```

abbilden.

Er kann auch Daten transformieren.

---

# Beispiel: externe Benutzer-API

Angenommen, eine externe API liefert:

```
public class ExternalUser
{
    public string FullName { get; set; } =
        string.Empty;

    public string Mail { get; set; } =
        string.Empty;
}
```

Unsere Anwendung erwartet aber:

```
public interface IUser
{
    string FirstName { get; }

    string LastName { get; }

    string Email { get; }
}
```

Ein Adapter kann die Datenstruktur übersetzen.

---

# UserAdapter

```
public class UserAdapter : IUser
{
    private readonly ExternalUser _user;

    public UserAdapter(ExternalUser user)
    {
        _user = user;
    }

    public string FirstName
    {
        get
        {
            string[] parts =
                _user.FullName.Split(
                    ' ',
                    StringSplitOptions.RemoveEmptyEntries);

            return parts.Length > 0
                ? parts[0]
                : string.Empty;
        }
    }

    public string LastName
    {
        get
        {
            string[] parts =
                _user.FullName.Split(
                    ' ',
                    StringSplitOptions.RemoveEmptyEntries);

            return parts.Length > 1
                ? parts[^1]
                : string.Empty;
        }
    }

    public string Email =>
        _user.Mail;
}
```

Der Adapter übersetzt hier:

```
FullName
   ↓
FirstName + LastName

Mail
   ↓
Email
```

---

# Adapter kann auch Typen konvertieren

Zum Beispiel erwartet die eigene Anwendung:

```
decimal Price
```

eine fremde Bibliothek liefert aber:

```
string Price
```

Der Adapter könnte schreiben:

```
public decimal Price =>
    decimal.Parse(
        _externalProduct.Price);
```

Also:

```
fremdes Datenmodell
        │
        ▼
      Adapter
        │
        ▼
eigenes Datenmodell
```

---

# Adapter als Anti-Corruption Layer

In größeren Architekturen ist dieses Prinzip besonders wichtig.

Angenommen:

```
Eigene Domain
     │
     ▼
IEmailService
```

aber ein externer Anbieter besitzt:

```
SomeExternalMailSdk
```

Anstatt die gesamte Anwendung direkt von dessen Klassen abhängig zu machen:

```
Domain/Application
        │
        ▼
ThirdParty SDK
```

erstellt man einen Adapter:

```
Domain/Application
        │
        ▼
    IEmailService
        ▲
        │
ExternalMailAdapter
        │
        ▼
ThirdParty SDK
```

Dadurch bleibt die eigene Anwendung von externen Details entkoppelt.

---

# Beispiel aus Clean Architecture

In der Application-Schicht:

```
public interface IEmailSender
{
    Task SendAsync(
        string recipient,
        string subject,
        string body);
}
```

Die Application-Schicht kennt nur:

```
IEmailSender
```

---

# Externer Dienst

Angenommen, eine Bibliothek stellt bereit:

```
public class ExternalMailClient
{
    public Task SendMessageAsync(
        ExternalMailMessage message)
    {
        // Fremde Bibliothek ...
        return Task.CompletedTask;
    }
}
```

Die Interfaces passen nicht zusammen.

---

# Adapter in Infrastructure

```
public class ExternalMailAdapter
    : IEmailSender
{
    private readonly ExternalMailClient _client;

    public ExternalMailAdapter(
        ExternalMailClient client)
    {
        _client = client;
    }

    public Task SendAsync(
        string recipient,
        string subject,
        string body)
    {
        // Eigenes Datenmodell wird in das
        // Modell der externen Bibliothek übersetzt.
        ExternalMailMessage message =
            new()
            {
                Recipient = recipient,
                Subject = subject,
                Body = body
            };

        return _client.SendMessageAsync(message);
    }
}
```

Die Application-Schicht muss nichts über:

```
ExternalMailClient
ExternalMailMessage
```

wissen.

---

# Adapter und Dependency Inversion Principle

Dieses Beispiel passt sehr gut zum **Dependency Inversion Principle**.

Die Application definiert:

```
IEmailSender
```

Die Infrastructure implementiert:

```
ExternalMailAdapter : IEmailSender
```

Also:

```
Application
    │
    ▼
IEmailSender
    ▲
    │
Infrastructure Adapter
    │
    ▼
External API
```

Damit hängt die Businesslogik nicht direkt von einer externen Bibliothek ab.

---

# Adapter in deinem Clean-Architecture-Projekt

Eine typische Struktur könnte beispielsweise sein:

```
Helpdesk.Application
│
└── INotificationService


Helpdesk.Infrastructure
│
├── TeamsNotificationAdapter
├── EmailNotificationAdapter
└── SmsNotificationAdapter
```

Die Application arbeitet nur mit:

```
INotificationService
```

und die Adapter übersetzen zu konkreten externen APIs.

Das ist ein sehr typischer praktischer Einsatz des Adapter Patterns.

---

# Adapter und Legacy-Code

Ein weiteres klassisches Beispiel:

Alte Anwendung:

```
public class LegacyPaymentSystem
{
    public void MakePayment(
        int amountInCents)
    {
        // ...
    }
}
```

Neue Anwendung erwartet:

```
public interface IPaymentService
{
    void Pay(decimal amount);
}
```

---

# Adapter

```
public class LegacyPaymentAdapter
    : IPaymentService
{
    private readonly LegacyPaymentSystem _legacy;

    public LegacyPaymentAdapter(
        LegacyPaymentSystem legacy)
    {
        _legacy = legacy;
    }

    public void Pay(decimal amount)
    {
        // Euro-Betrag in Cent umrechnen.
        int amountInCents =
            checked((int)(amount * 100));

        _legacy.MakePayment(
            amountInCents);
    }
}
```

Der Adapter übersetzt hier gleichzeitig:

```
Methodenname:

Pay()
→ MakePayment()


Datentyp / Einheit:

decimal Euro
→ int Cent
```

---

# Vorteil: Client bleibt unverändert

Der Client:

```
public class CheckoutService
{
    private readonly IPaymentService _paymentService;

    public CheckoutService(
        IPaymentService paymentService)
    {
        _paymentService = paymentService;
    }

    public void Checkout(decimal total)
    {
        _paymentService.Pay(total);
    }
}
```

muss nicht wissen, ob im Hintergrund:

```
LegacyPaymentSystem
Stripe
PayPal
anderer Dienst
```

verwendet wird.

---

# Adapter vs. Decorator

Beide Patterns **umhüllen** ein Objekt.

Der Zweck ist jedoch unterschiedlich.

## Adapter

```
Schnittstelle A
      │
      ▼
   Adapter
      │
      ▼
Schnittstelle B
```

Ziel:

> **Schnittstelle verändern beziehungsweise übersetzen.**

---

## Decorator

```
IComponent
     │
     ▼
Decorator : IComponent
     │
     ▼
Component : IComponent
```

Ziel:

> **Verhalten erweitern, ohne die Schnittstelle grundlegend zu ändern.**

---

# Kurzvergleich

```
Adapter
→ anderes Interface
→ Kompatibilität herstellen


Decorator
→ gleiches Interface
→ Funktionalität hinzufügen
```

---

# Beispiel

Adapter:

```
Camel.Move()
     │
     ▼
Adapter
     │
     ▼
ITransport.Drive()
```

Decorator:

```
Pizza.GetCost()
      │
      ▼
CheeseDecorator.GetCost()
      │
      ▼
Pizza.GetCost() + 5
```

---

# Adapter vs. Facade

Auch Facade verändert die Art, wie ein Client mit einem System arbeitet.

Aber:

## Adapter

passt eine konkrete inkompatible Schnittstelle an:

```
Client
  │
Target
  │
Adapter
  │
Adaptee
```

---

## Facade

bietet eine vereinfachte Schnittstelle für ein komplexes Subsystem:

```
Client
  │
  ▼
Facade
├── Service A
├── Service B
└── Service C
```

Merksatz:

```
Adapter
→ macht inkompatibel kompatibel

Facade
→ macht kompliziert einfacher
```

---

# Adapter vs. Proxy

Beide können dieselbe Schnittstelle implementieren und ein anderes Objekt enthalten.

Aber:

## Adapter

```
Ziel:
Schnittstelle umwandeln
```

## Proxy

```
Ziel:
Zugriff kontrollieren
```

Beispiel Proxy:

```
Client
  │
  ▼
SecurityProxy
  │
  ▼
RealService
```

---

# Adapter vs. Bridge

Diese beiden Strukturmuster werden ebenfalls häufig verwechselt.

## Adapter

wird meistens eingesetzt, **nachdem bereits inkompatible Klassen existieren**.

```
Bestehende Klasse
+
bestehender Client
+
Interfaces passen nicht
=
Adapter
```

---

## Bridge

wird bewusst im Design eingesetzt, um:

```
Abstraktion
```

und:

```
Implementierung
```

unabhängig voneinander entwickeln zu können.

Kurz:

```
Adapter
→ bestehende Inkompatibilität lösen

Bridge
→ zukünftige Kopplung bewusst vermeiden
```

---

# Adapter vs. Strategy

## Strategy

Mehrere Implementierungen besitzen dieselbe gewünschte Schnittstelle:

```
IPaymentStrategy
├── PayPal
├── CreditCard
└── Bank
```

Der Client wählt eine Strategie.

---

## Adapter

Ein bestehendes Objekt besitzt zunächst gerade **nicht** die gewünschte Schnittstelle.

```
LegacyPayment
    │
    ▼
Adapter
    │
    ▼
IPaymentService
```

---

# Vorteile

- inkompatible Klassen können zusammenarbeiten;
- bestehende Klassen müssen nicht verändert werden;
- Fremdbibliotheken können sauber integriert werden;
- Legacy-Code kann weiterverwendet werden;
- Client-Code bleibt von fremden Interfaces entkoppelt;
- Übersetzungslogik befindet sich an einer zentralen Stelle;
- unterstützt häufig Dependency Inversion und Clean Architecture.

---

# Nachteile

- zusätzliche Klassen erhöhen die Strukturkomplexität;
- bei sehr vielen Adaptern kann die Architektur schwerer nachvollziehbar werden;
- komplexe Datenkonvertierungen können im Adapter umfangreich werden;
- Änderungen an einer externen API können Anpassungen am Adapter erforderlich machen;
- bei extrem ähnlichen Interfaces kann ein Adapter unnötige Abstraktion darstellen.

---

# Wann Adapter nicht sinnvoll ist

Wenn die Klasse problemlos direkt das gewünschte Interface implementieren kann und dies fachlich sinnvoll ist:

```
public class MyService : IService
{
}
```

braucht man keinen zusätzlichen:

```
MyServiceAdapter
```

Adapter lohnt sich besonders, wenn:

```
Klasse nicht veränderbar
oder
Interface fachlich nicht passend
oder
externe Abhängigkeit soll isoliert werden
```

---

# Ein wichtiger Vorteil für Tests

Angenommen, die Businesslogik hängt nur von:

```
IWeatherService
```

ab.

Produktion:

```
IWeatherService
      ▲
      │
WeatherApiAdapter
      │
      ▼
External Weather API
```

Test:

```
IWeatherService
      ▲
      │
FakeWeatherService
```

Dadurch ist die Businesslogik viel einfacher zu testen.

---

# Klassische Struktur

```
Client
  │
  │ verwendet
  ▼
Target
  ▲
  │
Adapter
  │
  │ verwendet
  ▼
Adaptee
```

---

# Object Adapter in C#

Die typische moderne Struktur ist:

```
public interface ITarget
{
    void Request();
}
```

Adaptee:

```
public class Adaptee
{
    public void SpecificRequest()
    {
        // Bestehende Funktion.
    }
}
```

Adapter:

```
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;

    public Adapter(Adaptee adaptee)
    {
        _adaptee = adaptee;
    }

    public void Request()
    {
        // Übersetzung der Schnittstelle.
        _adaptee.SpecificRequest();
    }
}
```

Client:

```
public class Client
{
    public void Execute(ITarget target)
    {
        target.Request();
    }
}
```

---

# Das solltest du dir merken

Das Pattern besteht typischerweise aus:

```
Target
→ das Interface, das der Client erwartet


Adaptee
→ bestehende inkompatible Klasse


Adapter
→ übersetzt Target-Aufrufe in Adaptee-Aufrufe


Client
→ kennt nur Target
```

---

# Der entscheidende Code

```
public class Adapter : ITarget
{
    private readonly Adaptee _adaptee;

    public Adapter(Adaptee adaptee)
    {
        _adaptee = adaptee;
    }

    public void Request()
    {
        _adaptee.SpecificRequest();
    }
}
```

Das ist die Essenz des Adapter Patterns:

```
Request()
    │
    ▼
Adapter
    │
    ▼
SpecificRequest()
```

---

# Merksatz

> **Adapter übersetzt die Schnittstelle einer bestehenden Klasse in eine Schnittstelle, die der Client versteht.**

Noch einfacher:

```
Client:
"Ich brauche Drive()."

Adaptee:
"Ich habe nur Move()."

Adapter:
"Kein Problem.
Drive() bedeutet bei dir Move()."
```

Oder ganz kurz:

```
Adapter
=
inkompatible Interfaces
+
Übersetzung
+
Kompatibilität
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Adapter
→ Schnittstelle anpassen

Decorator
→ Verhalten erweitern

Facade
→ komplexes System vereinfachen

Proxy
→ Zugriff kontrollieren

Composite
→ Baumstruktur erzeugen

Bridge
→ Abstraktion und Implementierung trennen
```

---

> [!summary] Zusammenfassung  
> Das **Adapter Pattern** ist ein **Strukturmuster**.
> 
> Es wird eingesetzt, wenn ein Client eine bestimmte Schnittstelle erwartet, eine vorhandene Klasse aber eine inkompatible Schnittstelle besitzt.
> 
> Typische Struktur:
> 
> ```
> Client
>    │
>    ▼
> Target
>    ▲
>    │
> Adapter
>    │
>    ▼
> Adaptee
> ```
> 
> Der Client arbeitet ausschließlich mit:
> 
> ```
> ITarget
> ```
> 
> Der Adapter verwendet intern den bestehenden `Adaptee` und übersetzt beispielsweise:
> 
> ```
> Drive()
>    ↓
> Move()
> ```
> 
> Besonders wichtig ist das Pattern bei:
> 
> ```
> Third-Party Libraries
> Legacy-Code
> externen APIs
> Datenbanktreibern
> Clean Architecture
> Infrastructure-Adaptern
> ```
> 
> In modernem C# wird meistens ein **Object Adapter** mit Interface und Komposition verwendet:
> 
> ```
> public class Adapter : ITarget
> {
>     private readonly Adaptee _adaptee;
> 
>     public Adapter(Adaptee adaptee)
>     {
>         _adaptee = adaptee;
>     }
> 
>     public void Request()
>     {
>         _adaptee.SpecificRequest();
>     }
> }
> ```
> 
> **Kurz gesagt:**  
> `Adapter = Eine vorhandene Schnittstelle so übersetzen, dass sie zum Client passt.`