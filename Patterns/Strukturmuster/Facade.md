Das **Facade Pattern** ist ein Strukturmuster, das eine **vereinfachte Schnittstelle zu einem komplexen Subsystem** bereitstellt.

Der Client muss dadurch nicht alle internen Klassen, Abhängigkeiten und Aufrufreihenfolgen des Subsystems kennen.

> [!info] Grundidee  
> Ein komplexes System besitzt viele einzelne Komponenten.
> 
> Die Facade stellt davor eine einfache, zentrale Schnittstelle bereit.

Vereinfacht:

```
Client
  │
  ▼
Facade
  │
  ├──► SubsystemA
  ├──► SubsystemB
  └──► SubsystemC
```

Der Client sagt beispielsweise nur:

```
facade.Start();
```

anstatt selbst:

```
editor.CreateCode();
editor.Save();
compiler.Compile();
runtime.Execute();
```

aufrufen zu müssen.

---

# Wann sollte man Facade verwenden?

Das Facade Pattern eignet sich besonders:

- wenn ein Subsystem aus vielen Klassen besteht;
- wenn die Benutzung des Subsystems kompliziert ist;
- wenn der Client nicht alle Details des Subsystems kennen soll;
- wenn mehrere Methoden immer in einer bestimmten Reihenfolge aufgerufen werden müssen;
- wenn Abhängigkeiten zwischen Client und Subsystem reduziert werden sollen;
- wenn eine zentrale Einstiegsschnittstelle für einen Teil der Anwendung benötigt wird;
- wenn Subsysteme besser voneinander getrennt werden sollen.

---

# Typisches Problem

Angenommen, das Starten einer Anwendung benötigt folgende Schritte:

```
1. Code erstellen
2. Code speichern
3. Code kompilieren
4. Anwendung starten
```

Ohne Facade müsste jeder Client diese Details kennen:

```
textEditor.CreateCode();
textEditor.Save();

compiler.Compile();

runtime.Execute();
```

Ein anderer Client müsste exakt dieselbe Reihenfolge wiederholen.

Das führt zu:

```
Client
├── kennt TextEditor
├── kennt Compiler
├── kennt Runtime
├── kennt Reihenfolge
└── kennt technische Details
```

---

# Lösung mit Facade

Wir erstellen:

```
DevelopmentFacade
```

Diese kennt die Komponenten und koordiniert deren Verwendung.

Der Client kennt nur:

```
DevelopmentFacade
```

und schreibt:

```
facade.Start();
```

---

# Grundstruktur

```
                  Facade
                 /  |   \
                /   |    \
               ▼    ▼     ▼
        SubsystemA SubsystemB SubsystemC
```

Der Client:

```
Client
  │
  ▼
Facade
```

muss nicht direkt mit allen Subsystemen kommunizieren.

---

# Teilnehmer des Patterns

## Facade

Die Facade bietet eine vereinfachte API für komplexere Abläufe.

Beispiel:

```
public class Facade
{
    private readonly SubsystemA _subsystemA;
    private readonly SubsystemB _subsystemB;
    private readonly SubsystemC _subsystemC;

    public Facade(
        SubsystemA subsystemA,
        SubsystemB subsystemB,
        SubsystemC subsystemC)
    {
        _subsystemA = subsystemA;
        _subsystemB = subsystemB;
        _subsystemC = subsystemC;
    }

    public void Operation1()
    {
        _subsystemA.Operation();
        _subsystemB.Operation();
        _subsystemC.Operation();
    }

    public void Operation2()
    {
        _subsystemB.Operation();
        _subsystemC.Operation();
    }
}
```

---

## Subsystem-Klassen

Die Subsystem-Klassen enthalten die eigentliche Funktionalität.

Zum Beispiel:

```
public class SubsystemA
{
    public void Operation()
    {
        Console.WriteLine(
            "Subsystem A wird ausgeführt.");
    }
}
```

```
public class SubsystemB
{
    public void Operation()
    {
        Console.WriteLine(
            "Subsystem B wird ausgeführt.");
    }
}
```

```
public class SubsystemC
{
    public void Operation()
    {
        Console.WriteLine(
            "Subsystem C wird ausgeführt.");
    }
}
```

---

## Client

Der Client verwendet hauptsächlich die Facade.

```
public class Client
{
    public void Execute(Facade facade)
    {
        facade.Operation1();
    }
}
```

Der Client muss dadurch nicht wissen, wie viele Subsysteme beteiligt sind.

---

# Was macht die Facade genau?

Die Facade enthält normalerweise **keine vollständig neue Kernfunktionalität**.

Sie koordiniert bereits existierende Funktionen.

Beispiel:

```
public void Start()
{
    _editor.CreateCode();
    _editor.Save();
    _compiler.Compile();
    _runtime.Execute();
}
```

Die einzelnen Operationen existieren bereits.

Die Facade organisiert sie zu einem einfacheren Gesamtvorgang.

---

# Praktisches Beispiel: Entwicklungsumgebung

Das ursprüngliche Beispiel verwendet eine Entwicklungsumgebung wie Visual Studio.

Vereinfacht besteht der Vorgang aus:

```
Code schreiben
     ↓
Code speichern
     ↓
kompilieren
     ↓
Anwendung ausführen
```

Für den Entwickler erscheint das wie eine einfache Aktion:

```
"Anwendung starten"
```

Intern sind aber mehrere Komponenten beteiligt.

---

# Subsystem: TextEditor

```
public class TextEditor
{
    public void CreateCode()
    {
        Console.WriteLine(
            "Code wird geschrieben.");
    }

    public void Save()
    {
        Console.WriteLine(
            "Code wird gespeichert.");
    }
}
```

---

# Subsystem: Compiler

Im ursprünglichen Beispiel heißt die Klasse:

```
Compiller
```

Korrekt ist:

```
Compiler
```

```
public class Compiler
{
    public void Compile()
    {
        Console.WriteLine(
            "Die Anwendung wird kompiliert.");
    }
}
```

---

# Subsystem: Runtime

Die ursprüngliche Quelle verwendet dafür `CLR`.

Für das Pattern ist entscheidend, dass diese Klasse die Ausführung repräsentiert.

```
public class Runtime
{
    public void Execute()
    {
        Console.WriteLine(
            "Die Anwendung wird ausgeführt.");
    }

    public void Stop()
    {
        Console.WriteLine(
            "Die Anwendung wird beendet.");
    }
}
```

---

# Facade

```
public class DevelopmentFacade
{
    private readonly TextEditor _textEditor;
    private readonly Compiler _compiler;
    private readonly Runtime _runtime;

    public DevelopmentFacade(
        TextEditor textEditor,
        Compiler compiler,
        Runtime runtime)
    {
        _textEditor = textEditor;
        _compiler = compiler;
        _runtime = runtime;
    }

    public void Start()
    {
        // Die Facade kennt die notwendige Reihenfolge
        // der einzelnen Subsystem-Aufrufe.
        _textEditor.CreateCode();
        _textEditor.Save();
        _compiler.Compile();
        _runtime.Execute();
    }

    public void Stop()
    {
        // Der Client muss nicht wissen,
        // welches Subsystem die Anwendung beendet.
        _runtime.Stop();
    }
}
```

---

# Client: Programmer

```
public class Programmer
{
    public void CreateApplication(
        DevelopmentFacade facade)
    {
        // Der Client verwendet nur
        // die einfache Facade-Schnittstelle.
        facade.Start();

        facade.Stop();
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
        TextEditor textEditor =
            new TextEditor();

        Compiler compiler =
            new Compiler();

        Runtime runtime =
            new Runtime();

        DevelopmentFacade facade =
            new DevelopmentFacade(
                textEditor,
                compiler,
                runtime);

        Programmer programmer =
            new Programmer();

        programmer.CreateApplication(facade);
    }
}
```

---

# Ausgabe

```
Code wird geschrieben.
Code wird gespeichert.
Die Anwendung wird kompiliert.
Die Anwendung wird ausgeführt.
Die Anwendung wird beendet.
```

---

# Ablauf Schritt für Schritt

Der Client ruft:

```
facade.Start();
```

auf.

Die Facade führt intern aus:

```
DevelopmentFacade.Start()
        │
        ├──► TextEditor.CreateCode()
        │
        ├──► TextEditor.Save()
        │
        ├──► Compiler.Compile()
        │
        └──► Runtime.Execute()
```

Der Client muss davon nichts wissen.

Für ihn gibt es nur:

```
Start()
```

---

# Ohne Facade

Der Client müsste schreiben:

```
public class Programmer
{
    public void CreateApplication(
        TextEditor textEditor,
        Compiler compiler,
        Runtime runtime)
    {
        textEditor.CreateCode();
        textEditor.Save();

        compiler.Compile();

        runtime.Execute();

        // ...

        runtime.Stop();
    }
}
```

Der Client kennt jetzt:

```
TextEditor
Compiler
Runtime
```

und zusätzlich die richtige Reihenfolge.

---

# Mit Facade

```
public void CreateApplication(
    DevelopmentFacade facade)
{
    facade.Start();

    // ...

    facade.Stop();
}
```

Der Client kennt nur:

```
DevelopmentFacade
```

---

# Abhängigkeiten vergleichen

## Ohne Facade

```
             Client
             / |  \
            /  |   \
           ▼   ▼    ▼
       Editor Compiler Runtime
```

Der Client ist direkt mit mehreren Klassen gekoppelt.

---

## Mit Facade

```
Client
  │
  ▼
Facade
 / | \
▼  ▼  ▼
A  B  C
```

Die Anzahl direkter Client-Abhängigkeiten wird reduziert.

---

# Facade versteckt nicht zwingend alles

Ein wichtiger Punkt:

Die Existenz einer Facade bedeutet nicht automatisch, dass die Subsystem-Klassen nicht mehr direkt verwendet werden dürfen.

Zum Beispiel kann der Client trotzdem:

```
TextEditor editor =
    new TextEditor();

editor.CreateCode();
```

verwenden, wenn er die detaillierte Funktionalität benötigt.

> [!important]  
> Eine Facade bietet eine **einfachere Alternative** zum direkten Zugriff.
> 
> Sie muss den direkten Zugriff auf das Subsystem nicht vollständig verbieten.

---

# Facade vereinfacht häufige Abläufe

Angenommen, ein komplexes System bietet 40 unterschiedliche Methoden.

Der typische Client benötigt aber meistens nur:

```
Start()
Stop()
Restart()
GetStatus()
```

Dann kann die Facade genau diese häufig benötigten Use Cases anbieten.

```
public interface IServerFacade
{
    void Start();

    void Stop();

    void Restart();

    ServerStatus GetStatus();
}
```

Intern können dafür viele weitere Komponenten verwendet werden.

---

# Beispiel aus einer echten Anwendung: Bestellung

Angenommen, eine Bestellung erfordert:

```
Lager prüfen
   ↓
Zahlung durchführen
   ↓
Bestellung speichern
   ↓
E-Mail senden
   ↓
Versand vorbereiten
```

Ohne Facade:

```
inventory.CheckStock(...);

payment.Process(...);

repository.Save(...);

email.Send(...);

shipping.CreateShipment(...);
```

Der Controller müsste alle Details kennen.

---

# OrderFacade

```
public class OrderFacade
{
    private readonly InventoryService _inventory;
    private readonly PaymentService _payment;
    private readonly OrderRepository _repository;
    private readonly EmailService _email;
    private readonly ShippingService _shipping;

    public OrderFacade(
        InventoryService inventory,
        PaymentService payment,
        OrderRepository repository,
        EmailService email,
        ShippingService shipping)
    {
        _inventory = inventory;
        _payment = payment;
        _repository = repository;
        _email = email;
        _shipping = shipping;
    }

    public void PlaceOrder(Order order)
    {
        // Die Facade koordiniert den gesamten Ablauf.
        _inventory.CheckStock(order);

        _payment.Process(order);

        _repository.Save(order);

        _shipping.CreateShipment(order);

        _email.SendConfirmation(order);
    }
}
```

Client:

```
orderFacade.PlaceOrder(order);
```

Statt:

```
inventory.CheckStock(order);
payment.Process(order);
repository.Save(order);
shipping.CreateShipment(order);
email.SendConfirmation(order);
```

---

# Facade und Clean Architecture

Das Prinzip begegnet einem auch in Schichtenarchitekturen.

Beispiel:

```
UI
 │
 ▼
Application Service / Facade
 │
 ├── Repository
 ├── Domain Service
 ├── Notification Service
 └── externe Services
```

Die UI muss nicht alle internen Komponenten direkt koordinieren.

---

# Beispiel für eine Application Facade

```
public interface IHelpdeskFacade
{
    Task<int> CreateTicketAsync(
        CreateTicketRequest request);

    Task CloseTicketAsync(
        int ticketId);
}
```

Implementierung:

```
public class HelpdeskFacade
    : IHelpdeskFacade
{
    private readonly ITicketRepository _repository;
    private readonly INotificationService _notifications;

    public HelpdeskFacade(
        ITicketRepository repository,
        INotificationService notifications)
    {
        _repository = repository;
        _notifications = notifications;
    }

    public async Task<int> CreateTicketAsync(
        CreateTicketRequest request)
    {
        // Mehrere interne Schritte werden
        // hinter einer einfachen API verborgen.
        Ticket ticket =
            Ticket.Create(
                request.Title,
                request.Description);

        await _repository.AddAsync(ticket);

        await _notifications.SendTicketCreatedAsync(
            ticket);

        return ticket.Id;
    }

    public async Task CloseTicketAsync(
        int ticketId)
    {
        Ticket ticket =
            await _repository.GetAsync(ticketId);

        ticket.Close();

        await _repository.SaveChangesAsync();

        await _notifications.SendTicketClosedAsync(
            ticket);
    }
}
```

Die UI kann einfach:

```
await helpdeskFacade.CreateTicketAsync(request);
```

aufrufen.

Sie muss nicht wissen:

```
Wie wird Ticket erzeugt?
Welches Repository wird benutzt?
Wann wird gespeichert?
Wann wird Benachrichtigung gesendet?
```

---

# Facade als API eines Moduls

Eine Facade kann auch eine gute Möglichkeit sein, ein Modul nach außen abzuschirmen.

Zum Beispiel:

```
HelpdeskModule
│
├── TicketRepository
├── TicketService
├── AssignmentService
├── NotificationService
├── EscalationService
│
└── HelpdeskFacade
```

Andere Module sollen möglichst nur:

```
HelpdeskFacade
```

kennen.

Dadurch entsteht:

```
Andere Module
      │
      ▼
HelpdeskFacade
      │
      ▼
interne Helpdesk-Komponenten
```

---

# Facade und Information Hiding

Facade unterstützt die Idee:

> Clients sollen möglichst wenig über interne Implementierungsdetails wissen.

Wenn sich intern beispielsweise:

```
SubsystemB
```

ändert, kann die Facade eventuell dieselbe öffentliche API behalten.

Client-Code:

```
facade.Start();
```

bleibt dadurch unverändert.

---

# Mehrere Facades sind möglich

Eine komplexe Anwendung muss nicht genau eine riesige Facade besitzen.

Zum Beispiel:

```
Application
│
├── OrderFacade
├── CustomerFacade
├── PaymentFacade
└── ReportingFacade
```

Das ist oft besser als:

```
SuperApplicationFacade
```

mit hunderten Methoden.

> [!warning]  
> Eine Facade sollte selbst nicht zu einem **God Object** werden.

---

# Facade kann verschiedene Abstraktionsstufen anbieten

Zum Beispiel:

```
public class BuildFacade
{
    public void Build()
    {
        // Standard-Build.
    }

    public void BuildAndRun()
    {
        // Build + Start.
    }

    public void CleanBuildAndRun()
    {
        // Clean + Build + Start.
    }
}
```

Damit können verschiedene häufige Workflows abstrahiert werden.

---

# Facade vs. Adapter

Diese beiden Strukturmuster werden häufig verwechselt.

## Facade

```
komplexes Subsystem
        │
        ▼
      Facade
        │
        ▼
einfache Schnittstelle
```

Ziel:

> **Komplexität vereinfachen.**

---

## Adapter

```
inkompatibles Interface
        │
        ▼
      Adapter
        │
        ▼
erwartetes Interface
```

Ziel:

> **Schnittstellen kompatibel machen.**

---

# Kurzvergleich

```
Facade
→ kompliziert → einfach


Adapter
→ inkompatibel → kompatibel
```

Oder:

> **Facade verändert die Sicht auf ein System.  
> Adapter übersetzt eine Schnittstelle.**

---

# Facade vs. Decorator

## Decorator

Decorator erweitert das Verhalten eines einzelnen Objekts.

```
Decorator
   │
   ▼
Component
```

Ziel:

```
mehr Funktionalität
```

---

## Facade

Facade koordiniert mehrere Objekte.

```
Facade
├── A
├── B
└── C
```

Ziel:

```
einfachere Verwendung
```

---

# Kurzvergleich

```
Decorator
→ Funktionalität erweitern


Facade
→ komplexe Verwendung vereinfachen
```

---

# Facade vs. Mediator

Die Struktur kann auf den ersten Blick ähnlich wirken.

Beide können mehrere Objekte koordinieren.

Der Zweck ist jedoch unterschiedlich.

---

## Mediator

```
A ──┐
B ──┼──► Mediator
C ──┘
```

Der Mediator organisiert die **Kommunikation zwischen beteiligten Objekten**.

Die Objekte kommunizieren über den Mediator.

---

## Facade

```
Client
  │
  ▼
Facade
├── A
├── B
└── C
```

Die Facade bietet dem **externen Client eine vereinfachte Schnittstelle** zum Subsystem.

---

# Merksatz

```
Mediator
→ Wie kommunizieren interne Objekte miteinander?


Facade
→ Wie kann ein Client das komplexe System einfach verwenden?
```

---

# Facade vs. Proxy

## Proxy

Ein Proxy kontrolliert den Zugriff auf ein bestimmtes Objekt.

```
Client
  │
  ▼
Proxy
  │
  ▼
RealService
```

Ziel:

```
Zugriff kontrollieren
```

---

## Facade

Eine Facade bündelt mehrere Komponenten.

```
Client
  │
  ▼
Facade
├── ServiceA
├── ServiceB
└── ServiceC
```

Ziel:

```
komplexes Subsystem vereinfachen
```

---

# Facade vs. Application Service

In realen .NET-Anwendungen kann eine Application-Service-Klasse manchmal **Facade-ähnlich** wirken.

Zum Beispiel:

```
public class CheckoutService
{
    public Task CheckoutAsync(...)
    {
        // mehrere Komponenten koordinieren
    }
}
```

Sie kapselt einen kompletten Use Case hinter einer einfachen Methode.

Allerdings sollte man nicht automatisch jede Application-Service-Klasse als GoF-Facade bezeichnen.

Der wichtige Gedanke bleibt:

```
viele interne Details
        ↓
einfaches öffentliches Interface
```

---

# Vorteile

- vereinfacht komplexe Subsysteme;
- reduziert direkte Abhängigkeiten des Clients;
- bietet einen zentralen Einstiegspunkt;
- kapselt häufige Abläufe;
- verbessert Lesbarkeit des Client-Codes;
- interne Komponenten können leichter verändert werden;
- Subsysteme können besser voneinander isoliert werden;
- Clients müssen weniger technische Details kennen.

---

# Nachteile

- die Facade kann mit der Zeit sehr groß werden;
- zu viel Logik kann zu einem God Object führen;
- eine übermäßig vereinfachte API kann spezielle Funktionen verbergen;
- manche Clients benötigen trotzdem direkten Zugriff auf Subsysteme;
- bei sehr kleinen Systemen erzeugt eine Facade nur zusätzliche Abstraktion.

---

# Wann Facade nicht sinnvoll ist

Angenommen, wir haben nur:

```
repository.Save(entity);
```

Eine zusätzliche Klasse:

```
RepositoryFacade
```

die nur:

```
public void Save(Entity entity)
{
    _repository.Save(entity);
}
```

weiterleitet, bringt kaum einen Vorteil.

Facade ist besonders sinnvoll, wenn:

```
mehrere Komponenten
+
mehrere koordinierte Schritte
+
komplizierte Benutzung
```

existieren.

---

# Gute Facade

Eine gute Facade bietet Methoden auf einer höheren Abstraktionsebene.

Nicht:

```
facade.CallSubsystemA();
facade.CallSubsystemB();
facade.CallSubsystemC();
```

sondern eher:

```
facade.BuildApplication();
```

oder:

```
facade.PlaceOrder(order);
```

oder:

```
facade.CreateTicket(request);
```

Die Methode beschreibt **was der Client erreichen möchte**, nicht jeden internen technischen Schritt.

---

# Schlechte Facade

```
public class Facade
{
    public void CallA()
    {
        _a.CallA();
    }

    public void CallB()
    {
        _b.CallB();
    }

    public void CallC()
    {
        _c.CallC();
    }
}
```

Wenn sie nur jede einzelne Methode 1:1 weiterleitet, entsteht kaum Vereinfachung.

---

# Bessere Facade

```
public void StartApplication()
{
    _editor.Save();

    _compiler.Compile();

    _runtime.Execute();
}
```

Hier wird ein vollständiger Workflow hinter einer sinnvollen Operation versteckt.

---

# Dependency Injection

In modernem C# werden die Subsysteme normalerweise über den Konstruktor übergeben.

```
public class ApplicationFacade
{
    private readonly ICompiler _compiler;
    private readonly IRuntime _runtime;

    public ApplicationFacade(
        ICompiler compiler,
        IRuntime runtime)
    {
        _compiler = compiler;
        _runtime = runtime;
    }
}
```

Das ist meist besser als:

```
public ApplicationFacade()
{
    _compiler = new Compiler();
    _runtime = new Runtime();
}
```

weil dadurch:

- Abhängigkeiten sichtbar werden;
- Tests einfacher werden;
- Implementierungen austauschbar bleiben.

---

# Facade testen

Da eine Facade mehrere Services koordiniert, kann man testen, ob die erwarteten Schritte ausgeführt werden.

Zum Beispiel:

```
PlaceOrder()
    │
    ├── CheckStock()
    ├── ProcessPayment()
    ├── SaveOrder()
    └── SendConfirmation()
```

Der Test kann prüfen:

```
Wurde alles aufgerufen?
War die Reihenfolge korrekt?
Wird bei einem Fehler abgebrochen?
```

---

# Typische reale Einsatzgebiete

Facade eignet sich unter anderem für:

```
komplexe SDKs

Dateiverarbeitung

Multimedia-Systeme

Build-Systeme

Datenbankzugriffe

Payment-Systeme

Bestellprozesse

komplexe externe APIs

Application Services

Subsystem-Grenzen

Module
```

---

# Beispiel: Dateikonvertierung

Angenommen, Videokonvertierung benötigt:

```
Video öffnen
      ↓
Codec analysieren
      ↓
Audio extrahieren
      ↓
Video konvertieren
      ↓
Datei speichern
```

Der Client möchte aber nur:

```
converter.Convert(
    "video.mp4",
    "video.webm");
```

Das ist ein klassischer Facade-Anwendungsfall.

---

# Struktur als Diagramm

```
                       Client
                          │
                          ▼
                       Facade
                ┌─────────┼─────────┐
                │         │         │
                ▼         ▼         ▼
           SubsystemA SubsystemB SubsystemC
```

---

# Ablauf

```
Client
  │
  │ PlaceOrder()
  ▼
OrderFacade
  │
  ├──► InventoryService
  │
  ├──► PaymentService
  │
  ├──► OrderRepository
  │
  ├──► ShippingService
  │
  └──► EmailService
```

Der Client sieht nur:

```
PlaceOrder()
```

---

# Das solltest du dir merken

Subsystem:

```
public class SubsystemA
{
    public void OperationA()
    {
    }
}
```

```
public class SubsystemB
{
    public void OperationB()
    {
    }
}
```

Facade:

```
public class Facade
{
    private readonly SubsystemA _a;
    private readonly SubsystemB _b;

    public Facade(
        SubsystemA a,
        SubsystemB b)
    {
        _a = a;
        _b = b;
    }

    public void Execute()
    {
        // Die Facade koordiniert
        // mehrere Subsystem-Komponenten.
        _a.OperationA();
        _b.OperationB();
    }
}
```

Client:

```
Facade facade =
    new Facade(
        new SubsystemA(),
        new SubsystemB());

facade.Execute();
```

---

# Merksatz

> **Facade bietet eine einfache Schnittstelle zu einem komplexen Subsystem.**

Noch einfacher:

```
Client:
"Ich will die Anwendung starten."

Facade:
"Okay. Ich kümmere mich um
Speichern, Kompilieren und Ausführen."
```

Oder ganz kurz:

```
Facade
=
komplexes Subsystem
+
einfache Schnittstelle
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Facade
→ komplexes System vereinfachen

Adapter
→ Schnittstelle anpassen

Decorator
→ Verhalten erweitern

Proxy
→ Zugriff kontrollieren

Composite
→ Baumstruktur erzeugen

Bridge
→ Abstraktion und Implementierung trennen
```

---

> [!summary] Zusammenfassung  
> Das **Facade Pattern** ist ein **Strukturmuster**.
> 
> Es stellt eine vereinfachte Schnittstelle vor ein komplexes Subsystem:
> 
> ```
> Client
>    │
>    ▼
> Facade
> ├── SubsystemA
> ├── SubsystemB
> └── SubsystemC
> ```
> 
> Statt viele Komponenten selbst aufzurufen:
> 
> ```
> editor.Save();
> compiler.Compile();
> runtime.Execute();
> ```
> 
> kann der Client nur:
> 
> ```
> facade.Start();
> ```
> 
> aufrufen.
> 
> Die Facade kennt die notwendigen Subsysteme und deren richtige Aufrufreihenfolge.
> 
> Wichtig: Eine Facade **verbietet direkten Zugriff auf Subsysteme nicht zwingend**. Sie bietet lediglich einen einfacheren, zentralen Zugang für häufige Abläufe.
> 
> Besonders sinnvoll ist sie bei:
> 
> ```
> vielen Komponenten
> +
> komplizierter Koordination
> +
> wiederkehrenden Workflows
> ```
> 
> **Kurz gesagt:**  
> `Facade = Komplexität hinter einer einfachen Schnittstelle verstecken.`