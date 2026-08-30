Das **Tell-Don't-Ask-Prinzip** hilft dabei, **Daten und das zugehörige Verhalten in derselben Komponente zu kapseln**.

Statt Daten eines Objekts von außen abzufragen und anschließend auf Basis dieser Daten eine Aktion auszuführen, soll man dem Objekt möglichst direkt sagen, **was es tun soll**.

> [!IMPORTANT]  
> **Tell, don't ask** bedeutet:
> 
> Frage ein Objekt nicht nach seinem internen Zustand, um anschließend selbst zu entscheiden, was passieren soll.  
> Sage dem Objekt stattdessen direkt, welche Aufgabe es ausführen soll.

---

# Beispiel: AlarmClock

Betrachten wir zunächst eine Variante, bei der der Zustand des Objekts von außen abgefragt wird.

```csharp
AlarmClock clock = new AlarmClock(6);

clock.CurrentHour = 6;

if (clock.CurrentHour == clock.GetUpHour) // ask
{
    clock.Alarm();
}

class AlarmClock
{
    public int GetUpHour { get; set; }
    public int CurrentHour { get; set; }

    public AlarmClock(int hour)
    {
        GetUpHour = hour;
    }

    public void Alarm()
    {
        Console.WriteLine("Kompanie! Aufstehen!");
    }
}
```

Die Klasse `AlarmClock` enthält zwei wichtige Informationen:

```
GetUpHour    → Uhrzeit, zu der der Alarm ausgelöst werden soll
CurrentHour  → aktuelle Uhrzeit
```

Die Methode:

```csharp
Alarm()
```

führt den eigentlichen Alarm aus.

---

# Problem: Zustand abfragen und selbst entscheiden

Außerhalb des Objekts steht folgende Logik:

```csharp
if (clock.CurrentHour == clock.GetUpHour) // ask
{
    clock.Alarm();
}
```

Der aufrufende Code:

1. fragt `CurrentHour` ab,
    
2. fragt `GetUpHour` ab,
    
3. vergleicht beide Werte,
    
4. entscheidet selbst, ob `Alarm()` ausgeführt werden soll.
    

Das ist der **Ask-Teil**:

```
Objekt fragen
     │
     ▼
Daten erhalten
     │
     ▼
außerhalb entscheiden
     │
     ▼
Aktion ausführen
```

Die Entscheidung über das Verhalten befindet sich also außerhalb von `AlarmClock`.

---

# Lösung mit Tell-Don't-Ask

Die Prüfung kann direkt in die Klasse `AlarmClock` verschoben werden.

```csharp
AlarmClock clock = new AlarmClock(6);

clock.CurrentHour = 6;

class AlarmClock
{
    private int currentHour;

    public int GetUpHour { get; set; }

    public int CurrentHour
    {
        get => currentHour;

        set
        {
            if (value == GetUpHour)
            {
                Alarm(); // tell
            }

            currentHour = value;
        }
    }

    public AlarmClock(int hour)
    {
        GetUpHour = hour;
    }

    public void Alarm()
    {
        Console.WriteLine("Kompanie! Aufstehen!");
    }
}
```

Jetzt befindet sich die Entscheidung direkt im Objekt.

Der externe Code muss nicht mehr wissen:

```
Wann soll Alarm() ausgeführt werden?
```

Er setzt lediglich:

```csharp
clock.CurrentHour = 6;
```

Die Klasse `AlarmClock` entscheidet selbst, was anschließend passiert.

---

# Warum ist das besser?

Vorher:

```
Code außerhalb von AlarmClock
        │
        ├── CurrentHour lesen
        ├── GetUpHour lesen
        ├── vergleichen
        └── Alarm() aufrufen
```

Nachher:

```
AlarmClock
   │
   ├── kennt CurrentHour
   ├── kennt GetUpHour
   └── entscheidet selbst,
       wann Alarm() ausgeführt wird
```

Daten und Verhalten liegen jetzt näher beieinander.

> [!NOTE]  
> Wenn sich die Regeln für den Alarm ändern, muss nur die Klasse `AlarmClock` angepasst werden.  
> Der aufrufende Code bleibt davon weitgehend unberührt.

---

# Grundidee

Tell-Don't-Ask versucht diese Struktur zu vermeiden:

```
Objekt
  │
  ▼
Daten auslesen
  │
  ▼
außerhalb prüfen
  │
  ▼
außerhalb entscheiden
  │
  ▼
Objekt erneut aufrufen
```

Stattdessen:

```
Objekt
  │
  ▼
Aufgabe übergeben
  │
  ▼
Objekt entscheidet selbst
```

---

# Zweites Beispiel: Nachrichten

Ein weiterer typischer Anwendungsfall betrifft Klassenhierarchien.

Angenommen, wir besitzen verschiedene Nachrichtentypen.

```csharp
IMessage[] messages =
{
    // Beispiel für eine Sprachnachricht
    new VoiceMessage(
        Array.Empty<byte>(),
        "tom",
        "sam"),

    // Textnachricht
    new TextMessage(
        "Hello",
        "sam",
        "tom")
};

foreach (var message in messages)
{
    if (message is TextMessage textMessage) // ask
    {
        textMessage.Print();
    }
    else if (message is VoiceMessage voiceMessage) // ask
    {
        voiceMessage.Play();
    }
}

interface IMessage
{
    string Sender { get; }
    string Receiver { get; }
}

class TextMessage : IMessage
{
    public string Sender { get; }
    public string Receiver { get; }
    public string Text { get; }

    public TextMessage(
        string text,
        string sender,
        string receiver)
    {
        Text = text;
        Receiver = receiver;
        Sender = sender;
    }

    public void Print()
    {
        Console.WriteLine(
            $"Textnachricht: {Text}");
    }
}

class VoiceMessage : IMessage
{
    public string Sender { get; }
    public string Receiver { get; }
    public byte[] Voice { get; }

    public VoiceMessage(
        byte[] voice,
        string sender,
        string receiver)
    {
        Voice = voice;
        Receiver = receiver;
        Sender = sender;
    }

    public void Play()
    {
        Console.WriteLine(
            "Sprachnachricht wird abgespielt");
    }
}
```

---

# Was ist hier das Problem?

Wir besitzen zwei Implementierungen von `IMessage`:

```
TextMessage
VoiceMessage
```

Beide enthalten:

```
Sender
Receiver
```

Die konkrete Aktion unterscheidet sich jedoch:

```
TextMessage  → Print()
VoiceMessage → Play()
```

Wenn wir alle Nachrichten verarbeiten möchten, prüfen wir ihre konkreten Typen:

```csharp
foreach (var message in messages)
{
    if (message is TextMessage textMessage)
    {
        textMessage.Print();
    }
    else if (message is VoiceMessage voiceMessage)
    {
        voiceMessage.Play();
    }
}
```

Der externe Code fragt also:

```
Bist du eine TextMessage?
Bist du eine VoiceMessage?
```

und entscheidet anschließend selbst, welche Methode ausgeführt werden soll.

Das ist wieder **Ask**.

---

# Problematische Struktur

```
IMessage
   │
   ▼
Typ prüfen
   │
   ├── TextMessage?
   │      └── Print()
   │
   └── VoiceMessage?
          └── Play()
```

Der aufrufende Code kennt damit die konkreten Untertypen.

Wenn später ein neuer Typ hinzukommt:

```
VideoMessage
```

müsste auch diese `if`-Logik erweitert werden.

Zum Beispiel:

```csharp
else if (message is VideoMessage videoMessage)
{
    videoMessage.PlayVideo();
}
```

Dadurch wächst die Typprüfung immer weiter.

---

# Lösung: Gemeinsames Verhalten definieren

Statt nach dem konkreten Typ zu fragen, können wir dem Interface eine gemeinsame Aktion geben.

```csharp
IMessage[] messages =
{
    new VoiceMessage(
        Array.Empty<byte>(),
        "tom",
        "sam"),

    new TextMessage(
        "Hello",
        "sam",
        "tom")
};

foreach (var message in messages)
{
    message.Launch();
}

interface IMessage
{
    string Sender { get; }
    string Receiver { get; }

    void Launch();
}
```

Jetzt entscheidet jede konkrete Nachricht selbst, **wie sie gestartet beziehungsweise dargestellt wird**.

---

# TextMessage

```csharp
class TextMessage : IMessage
{
    public string Sender { get; }
    public string Receiver { get; }
    public string Text { get; }

    public TextMessage(
        string text,
        string sender,
        string receiver)
    {
        Text = text;
        Receiver = receiver;
        Sender = sender;
    }

    private void Print()
    {
        Console.WriteLine(
            $"Textnachricht: {Text}");
    }

    public void Launch()
    {
        Print(); // tell
    }
}
```

---

# VoiceMessage

```csharp
class VoiceMessage : IMessage
{
    public string Sender { get; }
    public string Receiver { get; }
    public byte[] Voice { get; }

    public VoiceMessage(
        byte[] voice,
        string sender,
        string receiver)
    {
        Voice = voice;
        Receiver = receiver;
        Sender = sender;
    }

    private void Play()
    {
        Console.WriteLine(
            "Sprachnachricht wird abgespielt");
    }

    public void Launch()
    {
        Play(); // tell
    }
}
```

Jetzt reicht im Client:

```csharp
foreach (var message in messages)
{
    message.Launch();
}
```

Der Client muss nicht mehr wissen, welchen konkreten Nachrichtentyp er verarbeitet.

---

# Vorher und nachher

## Vorher: Ask

```csharp
if (message is TextMessage textMessage)
{
    textMessage.Print();
}
else if (message is VoiceMessage voiceMessage)
{
    voiceMessage.Play();
}
```

Der Client kennt:

```csharp
TextMessage
VoiceMessage
Print()
Play()
```

---

## Nachher: Tell

```csharp
message.Launch();
```

Der Client kennt nur:

```csharp
IMessage
Launch()
```

Die Details bleiben in den konkreten Klassen.

---

# Zusammenhang mit Polymorphie

Dieses Beispiel nutzt Polymorphie.

```csharp
IMessage message = new TextMessage(
    "Hello",
    "sam",
    "tom");

message.Launch();
```

Oder:

```csharp
IMessage message = new VoiceMessage(
    Array.Empty<byte>(),
    "tom",
    "sam");

message.Launch();
```

In beiden Fällen ruft der Client dieselbe Methode auf:

```csharp
Launch()
```

Die konkrete Implementierung entscheidet, was tatsächlich passiert.

```
              IMessage
                 │
               Launch()
              /        \
             /          \
            ▼            ▼
     TextMessage    VoiceMessage
        Print()         Play()
```

---

# Zusammenhang mit Kapselung

Tell-Don't-Ask unterstützt die **Kapselung**.

Schlechter Ansatz:

```csharp
if (order.Status == OrderStatus.New &&
    order.Total > 100)
{
    order.Status = OrderStatus.Approved;
}
```

Hier kennt der externe Code die internen Regeln von `Order`.

Besser wäre zum Beispiel:

```csharp
order.Approve();
```

Die Klasse entscheidet selbst:

```csharp
class Order
{
    public OrderStatus Status { get; private set; }

    public decimal Total { get; private set; }

    public void Approve()
    {
        if (Status == OrderStatus.New &&
            Total > 100)
        {
            Status = OrderStatus.Approved;
        }
    }
}
```

Jetzt befindet sich die fachliche Regel direkt dort, wo auch die dazugehörigen Daten liegen.

---

# Typische Warnsignale

Tell-Don't-Ask kann relevant sein, wenn du häufig solchen Code siehst:

```csharp
if (customer.Status == ...)
{
    customer.DoSomething();
}
```

oder:

```csharp
if (account.Balance > ...)
{
    account.Withdraw(...);
}
```

oder:

```csharp
if (message is SomeConcreteType)
{
    ...
}
```

Besonders problematisch wird es, wenn derselbe Zustand an vielen verschiedenen Stellen abgefragt wird.

---

# Aber nicht jede Abfrage ist schlecht

> [!WARNING]  
> Tell-Don't-Ask bedeutet **nicht**, dass Properties oder Getter grundsätzlich verboten sind.

Abfragen sind völlig normal, wenn Daten tatsächlich benötigt werden.

Zum Beispiel:

```csharp
Console.WriteLine(user.Name);
```

ist kein Problem.

Problematisch wird es eher, wenn der aufrufende Code viele interne Daten ausliest, um anschließend Geschäftslogik auszuführen, die eigentlich zum Objekt selbst gehört.

---

# Einfaches Entscheidungsmodell

Frage dich:

```
Lese ich Daten nur aus,
weil ich anschließend entscheide,
welches Verhalten dieses Objekt ausführen soll?
```

Wenn ja, könnte die Logik möglicherweise in das Objekt verschoben werden.

---

# Vorteile

Tell-Don't-Ask kann zu folgenden Vorteilen führen:

- bessere Kapselung,
    
- weniger Wissen über interne Zustände,
    
- weniger Typprüfungen,
    
- weniger duplizierte Geschäftslogik,
    
- stärkere Kohäsion,
    
- bessere Polymorphie,
    
- leichter veränderbare Klassen.
    

---

# Kurzfassung

> [!SUMMARY]  
> **Tell-Don't-Ask = Sage einem Objekt, was es tun soll, statt seinen Zustand abzufragen und außerhalb zu entscheiden.**

Schlecht:

```
Objekt fragen
     │
     ▼
Zustand prüfen
     │
     ▼
Entscheidung außerhalb
     │
     ▼
Methode aufrufen
```

Besser:

```
Objekt
  │
  ▼
Aufgabe geben
  │
  ▼
Objekt entscheidet selbst
```

Beispiel:

```csharp
if (message is TextMessage textMessage)
{
    textMessage.Print();
}
```

besser:

```csharp
message.Launch();
```

### Merksatz

> **Nicht fragen: „Was bist du und welchen Zustand hast du?“**
> 
> **Sondern sagen: „Führe deine Aufgabe aus.“**