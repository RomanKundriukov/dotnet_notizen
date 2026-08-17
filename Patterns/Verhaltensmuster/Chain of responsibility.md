Die **Chain of Responsibility** ist ein Verhaltensmuster, bei dem eine Anfrage nacheinander an mehrere mögliche Handler weitergegeben wird.

Jeder Handler entscheidet selbst:

- ob er die Anfrage bearbeiten kann;
    
- ob er die Bearbeitung beendet;
    
- oder ob er die Anfrage an den nächsten Handler in der Kette weitergibt.
    

> [!info] Grundidee  
> Der Sender einer Anfrage muss nicht wissen, **welcher konkrete Handler** die Anfrage am Ende bearbeitet.

Vereinfacht:

```text
Request
   │
   ▼
Handler A
   │
   │ kann nicht bearbeiten
   ▼
Handler B
   │
   │ kann nicht bearbeiten
   ▼
Handler C
   │
   │ kann bearbeiten
   ▼
Request erledigt
```

---

# Wann sollte man Chain of Responsibility verwenden?

Das Pattern eignet sich besonders:

- wenn mehrere Objekte eine Anfrage bearbeiten könnten;
    
- wenn zur Laufzeit nicht bekannt sein soll, welcher Handler tatsächlich zuständig ist;
    
- wenn der Sender nicht direkt von einem konkreten Empfänger abhängig sein soll;
    
- wenn Handler dynamisch zusammengestellt werden sollen;
    
- wenn die Reihenfolge der Handler eine Rolle spielt;
    
- wenn ein Request schrittweise durch verschiedene Prüfungen oder Verarbeitungsschritte laufen soll.
    

---

# Grundstruktur

```text
Client
  │
  ▼
Handler A
  │
  ▼
Handler B
  │
  ▼
Handler C
```

Jeder Handler besitzt normalerweise eine Referenz auf:

```text
nächster Handler
```

---

# Teilnehmer des Patterns

## Handler

Der `Handler` definiert den gemeinsamen Vertrag für alle Handler.

Er enthält häufig außerdem eine Referenz auf den nächsten Handler.

```csharp
public abstract class Handler
{
    protected Handler? NextHandler { get; private set; }

    public void SetNext(Handler nextHandler)
    {
        NextHandler = nextHandler;
    }

    public abstract void Handle(int request);
}
```

---

## ConcreteHandler

Konkrete Handler entscheiden, ob sie einen Request bearbeiten können.

Wenn nicht:

```csharp
NextHandler?.Handle(request);
```

---

## Client

Der Client startet die Verarbeitung:

```csharp
firstHandler.Handle(request);
```

Er muss nicht wissen, welcher Handler den Request tatsächlich bearbeitet.

---

# Allgemeines Beispiel

```csharp
public abstract class Handler
{
    // Referenz auf den nächsten Handler in der Kette.
    protected Handler? NextHandler { get; private set; }

    // Legt den nächsten Handler fest.
    public void SetNext(Handler nextHandler)
    {
        NextHandler = nextHandler;
    }

    // Jeder konkrete Handler implementiert
    // seine eigene Verarbeitung.
    public abstract void Handle(int request);
}
```

---

# ConcreteHandler1

```csharp
public class ConcreteHandler1 : Handler
{
    public override void Handle(int request)
    {
        if (request == 1)
        {
            Console.WriteLine(
                "ConcreteHandler1 verarbeitet die Anfrage.");

            return;
        }

        // Wenn dieser Handler nicht zuständig ist,
        // wird die Anfrage weitergegeben.
        NextHandler?.Handle(request);
    }
}
```

---

# ConcreteHandler2

```csharp
public class ConcreteHandler2 : Handler
{
    public override void Handle(int request)
    {
        if (request == 2)
        {
            Console.WriteLine(
                "ConcreteHandler2 verarbeitet die Anfrage.");

            return;
        }

        // Anfrage an den nächsten Handler weitergeben.
        NextHandler?.Handle(request);
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
        Handler handler1 =
            new ConcreteHandler1();

        Handler handler2 =
            new ConcreteHandler2();

        // Die Handler werden miteinander verbunden.
        handler1.SetNext(handler2);

        // Der Client kennt nur den ersten Handler.
        handler1.Handle(2);
    }
}
```

Ablauf:

```text
Request = 2
   │
   ▼
Handler1
   │
   │ request != 1
   ▼
Handler2
   │
   │ request == 2
   ▼
bearbeitet
```

---

# Was ist hier wichtig?

Der Client schreibt nur:

```csharp
handler1.Handle(2);
```

Er schreibt nicht:

```csharp
if (...)
{
    handler1.Handle(...);
}
else if (...)
{
    handler2.Handle(...);
}
```

Die Entscheidung liegt innerhalb der Handler-Kette.

---

# Entkopplung von Sender und Empfänger

Ohne Chain of Responsibility könnte der Client so aussehen:

```csharp
if (request.Type == RequestType.A)
{
    handlerA.Handle(request);
}
else if (request.Type == RequestType.B)
{
    handlerB.Handle(request);
}
else if (request.Type == RequestType.C)
{
    handlerC.Handle(request);
}
```

Der Client kennt dadurch:

```text
HandlerA
HandlerB
HandlerC
```

und ist direkt mit ihnen gekoppelt.

Mit Chain of Responsibility:

```csharp
firstHandler.Handle(request);
```

kennt der Client nur den Einstiegspunkt der Kette.

---

# Praktisches Beispiel: Zahlungsabwicklung

Angenommen, eine Anwendung soll Geld an einen Empfänger senden.

Der Empfänger unterstützt möglicherweise:

- Banküberweisung;
    
- PayPal;
    
- einen anderen Geldtransferdienst.
    

Wir möchten nicht bereits im Client festlegen, welche Zahlungsart verwendet werden soll.

Stattdessen bilden wir eine Handler-Kette.

```text
Bank
  │
  ▼
PayPal
  │
  ▼
Money Transfer
```

Jeder Handler prüft, ob seine Zahlungsart unterstützt wird.

---

# Receiver

Der `Receiver` beschreibt, welche Zahlungsarten unterstützt werden.

```csharp
public class Receiver
{
    public bool SupportsBankTransfer { get; }

    public bool SupportsMoneyTransfer { get; }

    public bool SupportsPayPal { get; }

    public Receiver(
        bool supportsBankTransfer,
        bool supportsMoneyTransfer,
        bool supportsPayPal)
    {
        SupportsBankTransfer =
            supportsBankTransfer;

        SupportsMoneyTransfer =
            supportsMoneyTransfer;

        SupportsPayPal =
            supportsPayPal;
    }
}
```

---

# Abstrakter PaymentHandler

```csharp
public abstract class PaymentHandler
{
    protected PaymentHandler? NextHandler { get; private set; }

    // Verbindet diesen Handler mit dem nächsten.
    public PaymentHandler SetNext(
        PaymentHandler nextHandler)
    {
        NextHandler = nextHandler;

        // Rückgabe des nächsten Handlers ermöglicht
        // ein bequemes Verketten.
        return nextHandler;
    }

    public abstract bool Handle(
        Receiver receiver,
        decimal amount);
}
```

---

# Warum `bool Handle(...)`?

Im älteren Beispiel war die Methode:

```csharp
void Handle(...)
```

Für echte Anwendungen ist es oft nützlicher zurückzugeben, ob die Anfrage tatsächlich verarbeitet wurde.

Zum Beispiel:

```text
true  → Zahlung wurde verarbeitet
false → kein Handler konnte sie verarbeiten
```

---

# BankPaymentHandler

```csharp
public class BankPaymentHandler : PaymentHandler
{
    public override bool Handle(
        Receiver receiver,
        decimal amount)
    {
        if (receiver.SupportsBankTransfer)
        {
            Console.WriteLine(
                $"Banküberweisung über {amount:C} wird ausgeführt.");

            return true;
        }

        // Wenn Banküberweisung nicht möglich ist,
        // wird der nächste Handler versucht.
        return NextHandler?.Handle(receiver, amount)
            ?? false;
    }
}
```

---

# PayPalPaymentHandler

```csharp
public class PayPalPaymentHandler : PaymentHandler
{
    public override bool Handle(
        Receiver receiver,
        decimal amount)
    {
        if (receiver.SupportsPayPal)
        {
            Console.WriteLine(
                $"PayPal-Zahlung über {amount:C} wird ausgeführt.");

            return true;
        }

        return NextHandler?.Handle(receiver, amount)
            ?? false;
    }
}
```

---

# MoneyTransferHandler

```csharp
public class MoneyTransferHandler : PaymentHandler
{
    public override bool Handle(
        Receiver receiver,
        decimal amount)
    {
        if (receiver.SupportsMoneyTransfer)
        {
            Console.WriteLine(
                $"Geldtransfer über {amount:C} wird ausgeführt.");

            return true;
        }

        return NextHandler?.Handle(receiver, amount)
            ?? false;
    }
}
```

---

# Kette aufbauen

```csharp
PaymentHandler bank =
    new BankPaymentHandler();

PaymentHandler paypal =
    new PayPalPaymentHandler();

PaymentHandler moneyTransfer =
    new MoneyTransferHandler();

bank.SetNext(paypal)
    .SetNext(moneyTransfer);
```

Dadurch entsteht:

```text
BankPaymentHandler
        │
        ▼
PayPalPaymentHandler
        │
        ▼
MoneyTransferHandler
```

---

# Verwendung

```csharp
public class Program
{
    public static void Main()
    {
        Receiver receiver =
            new Receiver(
                supportsBankTransfer: false,
                supportsMoneyTransfer: true,
                supportsPayPal: true);

        PaymentHandler bank =
            new BankPaymentHandler();

        PaymentHandler paypal =
            new PayPalPaymentHandler();

        PaymentHandler moneyTransfer =
            new MoneyTransferHandler();

        // Reihenfolge der Handler festlegen.
        bank.SetNext(paypal)
            .SetNext(moneyTransfer);

        bool success =
            bank.Handle(receiver, 250m);

        if (!success)
        {
            Console.WriteLine(
                "Keine geeignete Zahlungsart gefunden.");
        }
    }
}
```

Da der Empfänger:

```text
Bank       → nein
PayPal     → ja
Transfer   → ja
```

unterstützt, läuft die Verarbeitung so:

```text
BankPaymentHandler
        │
        │ nicht möglich
        ▼
PayPalPaymentHandler
        │
        │ möglich
        ▼
Zahlung wird durchgeführt
```

---

# Warum wird MoneyTransfer nicht mehr aufgerufen?

Der PayPal-Handler gibt:

```csharp
return true;
```

zurück.

Damit ist die Anfrage vollständig bearbeitet.

Die Kette endet dort.

---

# Reihenfolge der Handler

Die Reihenfolge kann entscheidend sein.

Zum Beispiel:

```text
Bank
 ↓
PayPal
 ↓
MoneyTransfer
```

bedeutet:

> Banküberweisung besitzt die höchste Priorität.

Wenn wir dagegen schreiben:

```text
PayPal
 ↓
Bank
 ↓
MoneyTransfer
```

wird zuerst PayPal geprüft.

Das ist einer der wichtigen Vorteile des Patterns:

> Die Priorität kann über die Reihenfolge der Handler gesteuert werden.

---

# Kette dynamisch aufbauen

Die Handler müssen nicht fest im Code definiert sein.

Man könnte beispielsweise zur Laufzeit entscheiden:

```csharp
PaymentHandler first =
    new BankPaymentHandler();

if (usePayPal)
{
    first.SetNext(
        new PayPalPaymentHandler());
}
```

Damit kann die Kette dynamisch aufgebaut werden.

---

# Was passiert, wenn niemand zuständig ist?

Das ist ein wichtiger Nachteil des Patterns.

Angenommen:

```text
Receiver
├── Bank = false
├── PayPal = false
└── MoneyTransfer = false
```

Dann:

```text
Bank
 │
 ▼
PayPal
 │
 ▼
MoneyTransfer
 │
 ▼
Ende
```

Kein Handler bearbeitet die Anfrage.

Deshalb sollte man in realen Anwendungen überlegen, wie damit umgegangen wird.

---

# Möglichkeit 1: `false` zurückgeben

Wie in unserem Beispiel:

```csharp
bool success =
    handler.Handle(...);
```

Danach:

```csharp
if (!success)
{
    Console.WriteLine(
        "Keine passende Verarbeitung gefunden.");
}
```

---

# Möglichkeit 2: Fallback-Handler

Man kann am Ende der Kette einen Handler hinzufügen, der immer reagiert.

```csharp
public class UnsupportedPaymentHandler
    : PaymentHandler
{
    public override bool Handle(
        Receiver receiver,
        decimal amount)
    {
        Console.WriteLine(
            "Keine unterstützte Zahlungsart vorhanden.");

        return false;
    }
}
```

Dann:

```text
Bank
 ↓
PayPal
 ↓
MoneyTransfer
 ↓
Fallback
```

Dadurch läuft die Anfrage niemals still ins Leere.

---

# Zwei Arten von Chains

Bei Chain of Responsibility gibt es zwei häufige Varianten.

## Variante 1: Ein Handler beendet die Verarbeitung

```text
Handler A
   │
   ▼
Handler B
   │
   └── bearbeitet
       → Ende
```

Das haben wir im Zahlungsbeispiel.

---

## Variante 2: Alle Handler verarbeiten die Anfrage

Manchmal soll jeder Handler etwas tun und die Anfrage trotzdem weitergeben.

Zum Beispiel:

```text
Logging
   ↓
Authentication
   ↓
Validation
   ↓
Business Logic
```

Jeder Schritt bearbeitet den Request und gibt ihn anschließend weiter.

---

# Beispiel einer Pipeline

```csharp
public abstract class RequestHandler
{
    protected RequestHandler? NextHandler { get; private set; }

    public RequestHandler SetNext(
        RequestHandler nextHandler)
    {
        NextHandler = nextHandler;

        return nextHandler;
    }

    public virtual void Handle(Request request)
    {
        // Standardmäßig einfach weitergeben.
        NextHandler?.Handle(request);
    }
}
```

Logging:

```csharp
public class LoggingHandler : RequestHandler
{
    public override void Handle(Request request)
    {
        Console.WriteLine(
            "Request wird protokolliert.");

        // Verarbeitung fortsetzen.
        base.Handle(request);
    }
}
```

Validierung:

```csharp
public class ValidationHandler : RequestHandler
{
    public override void Handle(Request request)
    {
        Console.WriteLine(
            "Request wird validiert.");

        base.Handle(request);
    }
}
```

Hier wird die Anfrage nicht nur von einem Handler bearbeitet.

Sie läuft durch mehrere Verarbeitungsschritte.

---

# Typischer Ablauf einer modernen Web-Anwendung

Das Prinzip findet man oft in Form von Middleware-Pipelines:

```text
HTTP Request
     │
     ▼
Logging
     │
     ▼
Authentication
     │
     ▼
Authorization
     │
     ▼
Exception Handling
     │
     ▼
Endpoint
```

Jede Komponente kann:

```text
Request bearbeiten
+
an nächste Komponente weitergeben
```

oder die Kette vorzeitig beenden.

---

# Beispiel: ASP.NET Core Middleware

Das Prinzip ähnelt sehr stark der Chain of Responsibility.

Vereinfacht:

```csharp
public async Task InvokeAsync(
    HttpContext context)
{
    Console.WriteLine(
        "Request wird verarbeitet.");

    // Request an den nächsten Schritt weitergeben.
    await _next(context);
}
```

Eine Middleware kann aber auch entscheiden:

```csharp
return;
```

und die Verarbeitung nicht weitergeben.

> [!note]  
> Eine Middleware-Pipeline ist nicht zwingend eine reine GoF-Implementierung der Chain of Responsibility, verwendet aber dasselbe zentrale Prinzip einer Verarbeitungskette.

---

# Chain of Responsibility vs. normale `if`-Kette

Ohne Pattern:

```csharp
if (canUseBank)
{
    PayByBank();
}
else if (canUsePayPal)
{
    PayByPayPal();
}
else if (canUseTransfer)
{
    PayByTransfer();
}
```

Problem:

Die gesamte Entscheidungslogik befindet sich an einer Stelle.

---

Mit Chain of Responsibility:

```text
BankHandler
      ↓
PayPalHandler
      ↓
TransferHandler
```

Jeder Handler kennt nur:

```text
seine eigene Verantwortung
+
den nächsten Handler
```

---

# Single Responsibility Principle

Chain of Responsibility passt gut zum **Single Responsibility Principle**.

Ohne Pattern:

```csharp
public class PaymentService
{
    // Banklogik
    // PayPal-Logik
    // Transferlogik
    // Auswahl der Zahlungsart
    // Fehlerbehandlung
}
```

Die Klasse besitzt viele Verantwortlichkeiten.

Mit Handlern:

```text
BankPaymentHandler
→ nur Bank

PayPalPaymentHandler
→ nur PayPal

MoneyTransferHandler
→ nur Geldtransfer
```

Jede Klasse besitzt eine klarere Verantwortung.

---

# Open/Closed Principle

Wenn eine neue Zahlungsart hinzukommt:

```text
CryptoPaymentHandler
```

können wir eine neue Klasse erstellen:

```csharp
public class CryptoPaymentHandler
    : PaymentHandler
{
    public override bool Handle(
        Receiver receiver,
        decimal amount)
    {
        // Eigene Logik ...

        return NextHandler?.Handle(
            receiver,
            amount) ?? false;
    }
}
```

und sie in die Kette einfügen.

Bestehende Handler müssen dafür nicht unbedingt verändert werden.

---

# Vorteile

- Sender und Empfänger sind schwächer gekoppelt.
    
- Der Client muss den konkreten Handler nicht kennen.
    
- Handler können leicht hinzugefügt oder entfernt werden.
    
- Reihenfolge und Priorität können flexibel geändert werden.
    
- Jeder Handler besitzt eine klar abgegrenzte Verantwortung.
    
- Große `if / else`- oder `switch`-Blöcke können reduziert werden.
    
- Ketten können zur Laufzeit aufgebaut werden.
    

---

# Nachteile

- Es gibt keine automatische Garantie, dass ein Request verarbeitet wird.
    
- Lange Ketten können schwer nachzuvollziehen sein.
    
- Debugging kann schwieriger werden, weil ein Request mehrere Objekte durchläuft.
    
- Falsche Reihenfolge der Handler kann zu unerwartetem Verhalten führen.
    
- Sehr viele kleine Handler erhöhen die Anzahl der Klassen.
    

---

# Wann Chain of Responsibility nicht sinnvoll ist

Wenn es nur eine einzige eindeutige Verarbeitung gibt:

```csharp
paymentService.Pay();
```

braucht man keine Handler-Kette.

Auch bei:

```text
2 einfache Bedingungen
+
keine dynamische Reihenfolge
+
keine Erweiterbarkeit nötig
```

kann ein einfaches `if` völlig ausreichend sein.

> [!tip]  
> Design Patterns sollen Komplexität lösen und nicht unnötig neue Komplexität erzeugen.

---

# Chain of Responsibility vs. State

## State

Die aktuelle Implementierung wird durch den Zustand eines Objekts bestimmt:

```text
Context
  │
  ▼
CurrentState
```

Frage:

> „Wie soll sich dieses Objekt in seinem aktuellen Zustand verhalten?“

---

## Chain of Responsibility

Eine Anfrage wandert durch mehrere mögliche Handler:

```text
Handler1
   ↓
Handler2
   ↓
Handler3
```

Frage:

> „Wer kann diese Anfrage bearbeiten?“

---

# Chain of Responsibility vs. Strategy

## Strategy

```text
Context
  │
  ▼
eine ausgewählte Strategy
```

Der Client beziehungsweise Context wählt meist gezielt eine Strategie.

---

## Chain of Responsibility

```text
Context
  │
  ▼
Handler1
  │
  ▼
Handler2
```

Der Client kennt den tatsächlichen Bearbeiter nicht unbedingt.

Merksatz:

```text
Strategy
→ "Welchen Algorithmus verwende ich?"

State
→ "Wie verhalte ich mich in meinem Zustand?"

Chain of Responsibility
→ "Wer in der Kette kann meine Anfrage bearbeiten?"
```

---

# Chain of Responsibility vs. Decorator

Strukturell können beide ähnlich aussehen:

```text
Object
  │
  ▼
Object
  │
  ▼
Object
```

Die Absicht ist aber verschieden.

## Decorator

Erweitert das Verhalten eines Objekts.

```text
Component
→ LoggingDecorator
→ CachingDecorator
```

---

## Chain of Responsibility

Leitet eine Anfrage durch mögliche Handler.

```text
Handler A
→ Handler B
→ Handler C
```

Der Fokus liegt auf der **Weitergabe einer Anfrage**.

---

# Typische reale Anwendungsfälle

Chain of Responsibility eignet sich beispielsweise für:

- Request-Validation
    
- Logging
    
- Authentifizierung
    
- Autorisierung
    
- Exception Handling
    
- Zahlungsabwicklung
    
- Support-Eskalation
    
- Genehmigungsprozesse
    
- Spam-Filter
    
- Datei-Verarbeitung
    
- Middleware
    
- Event-Processing
    

---

# Beispiel: Support-System

```text
Support Request
      │
      ▼
Level 1 Support
      │
      │ kann nicht lösen
      ▼
Level 2 Support
      │
      │ kann nicht lösen
      ▼
Administrator
```

Jede Ebene entscheidet:

```text
Kann ich das Problem lösen?
        │
    ┌───┴───┐
   Ja      Nein
   │         │
   ▼         ▼
lösen     weitergeben
```

Das ist ein sehr typisches Beispiel für Chain of Responsibility.

---

# Klassische Struktur

```text
                   Handler
                      ▲
          ┌───────────┼───────────┐
          │           │           │
      HandlerA     HandlerB    HandlerC
          │           │           │
          └───────►───┴──────►────┘
```

Genauer:

```text
Client
  │
  ▼
HandlerA
  │
  │ Next
  ▼
HandlerB
  │
  │ Next
  ▼
HandlerC
```

---

# Moderner Basishandler

Eine praktische Basisklasse kann so aussehen:

```csharp
public abstract class Handler<T>
{
    private Handler<T>? _next;

    // Verbindet den aktuellen Handler
    // mit dem nächsten Handler.
    public Handler<T> SetNext(
        Handler<T> next)
    {
        _next = next;

        return next;
    }

    public virtual bool Handle(T request)
    {
        // Wenn dieser Handler nichts macht,
        // wird standardmäßig weitergegeben.
        return _next?.Handle(request)
            ?? false;
    }

    // Hilfsmethode für abgeleitete Klassen.
    protected bool HandleNext(T request)
    {
        return _next?.Handle(request)
            ?? false;
    }
}
```

Dann kann ein konkreter Handler nur seine eigene Verantwortung implementieren:

```csharp
public class ConcreteHandler : Handler<Request>
{
    public override bool Handle(Request request)
    {
        if (CanHandle(request))
        {
            Process(request);

            return true;
        }

        return HandleNext(request);
    }

    private bool CanHandle(Request request)
    {
        // Prüfen, ob dieser Handler zuständig ist.
        return true;
    }

    private void Process(Request request)
    {
        // Request verarbeiten.
    }
}
```

---

# Das solltest du dir merken

Die typische Struktur ist:

```csharp
public abstract class Handler
{
    protected Handler? Next { get; set; }

    public abstract void Handle(Request request);
}
```

Konkreter Handler:

```csharp
public override void Handle(Request request)
{
    if (CanHandle(request))
    {
        Process(request);
        return;
    }

    Next?.Handle(request);
}
```

Aufbau:

```csharp
handler1.SetNext(handler2);
handler2.SetNext(handler3);
```

Start:

```csharp
handler1.Handle(request);
```

---

# Merksatz

> **Chain of Responsibility gibt eine Anfrage durch eine Kette von Handlern weiter, bis einer sie bearbeitet oder die Kette endet.**

Noch einfacher:

```text
Client:
"Hier ist meine Anfrage."

Handler:
"Kann ich sie bearbeiten?"

Ja
→ bearbeiten

Nein
→ an den nächsten weitergeben
```

Oder ganz kurz:

```text
Chain of Responsibility
=
Request
→ Handler
→ Handler
→ Handler
→ Ergebnis
```

> [!summary] Zusammenfassung  
> Die **Chain of Responsibility** ist ein **Verhaltensmuster**.
> 
> Mehrere Handler werden zu einer Kette verbunden:
> 
> ```text
> Handler A
>    ↓
> Handler B
>    ↓
> Handler C
> ```
> 
> Jeder Handler entscheidet:
> 
> - Anfrage selbst bearbeiten;
>     
> - Anfrage an den nächsten Handler weitergeben;
>     
> - oder die Verarbeitung beenden.
>     
> 
> Der Client kennt normalerweise nur den ersten Handler:
> 
> ```csharp
> firstHandler.Handle(request);
> ```
> 
> Dadurch werden Sender und konkreter Empfänger voneinander entkoppelt.
> 
> Wichtig ist jedoch: Wenn kein Handler zuständig ist, kann die Anfrage unbehandelt bleiben. Deshalb ist häufig ein Rückgabewert oder ein Fallback-Handler sinnvoll.
> 
> **Kurz gesagt:**  
> `Chain of Responsibility = Eine Anfrage durch mehrere mögliche Handler weiterreichen.`