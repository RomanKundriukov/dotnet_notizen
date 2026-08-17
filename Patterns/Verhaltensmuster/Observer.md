Das **Observer Pattern** ist ein Verhaltensmuster, das eine **1:n-Beziehung** zwischen Objekten beschreibt.

Dabei gibt es:

- ein **beobachtetes Objekt** (*Subject / Observable / Publisher*)
- mehrere **Beobachter** (*Observer / Subscriber*)

Wenn sich der Zustand des beobachteten Objekts verändert, werden automatisch alle registrierten Beobachter informiert.

> [!info] Grundidee
> Ein Objekt veröffentlicht Änderungen.
>
> Mehrere andere Objekte können sich dafür registrieren und werden automatisch benachrichtigt.

Vereinfacht:

```text
              Observable
                  │
        Änderung tritt ein
                  │
                  ▼
          NotifyObservers()
                  │
       ┌──────────┼──────────┐
       ▼          ▼          ▼
   Observer 1 Observer 2 Observer 3
```

Das Pattern wird deshalb häufig auch als:

**Publisher / Subscriber**

bezeichnet.

---

## Beispiel aus dem Alltag

Ein Newsletter funktioniert nach einem ähnlichen Prinzip:

```text
Website / Newsletter
        │
        │ veröffentlicht Nachricht
        ▼
┌───────────────┐
│ Abonnenten    │
├───────────────┤
│ Benutzer A    │
│ Benutzer B    │
│ Benutzer C    │
└───────────────┘
```

Die Benutzer müssen nicht ständig prüfen:

> „Gibt es etwas Neues?“

Stattdessen werden sie automatisch informiert.

---

# Wann sollte man Observer verwenden?

Das Observer Pattern eignet sich besonders:

- wenn ein Objekt mehrere andere Objekte über Änderungen informieren muss;
- wenn die Anzahl der Beobachter zur Entwicklungszeit nicht bekannt ist;
- wenn Beobachter zur Laufzeit hinzugefügt oder entfernt werden sollen;
- wenn Publisher und Subscriber möglichst wenig voneinander wissen sollen;
- wenn mehrere Objekte auf dieselbe Zustandsänderung unterschiedlich reagieren sollen;
- wenn UI-Komponenten auf Änderungen reagieren müssen;
- wenn ein Ereignissystem aufgebaut werden soll.

---

# Grundstruktur

```text
                  IObservable
                       ▲
                       │
               ConcreteObservable
                       │
                       │ Notify
                       ▼
                   IObserver
                       ▲
              ┌────────┴────────┐
              │                 │
       ObserverA            ObserverB
```

---

# Teilnehmer des Patterns

## Observable / Subject

Das Observable ist das Objekt, das beobachtet wird.

Es ermöglicht:

```text
Beobachter registrieren
Beobachter entfernen
Beobachter informieren
```

Zum Beispiel:

```csharp
public interface IObservable
{
    void AddObserver(IObserver observer);

    void RemoveObserver(IObserver observer);

    void NotifyObservers();
}
```

---

## Observer

Der Observer definiert, wie ein Beobachter benachrichtigt wird.

```csharp
public interface IObserver
{
    void Update();
}
```

---

## ConcreteObservable

Das konkrete Observable verwaltet eine Liste aller Beobachter.

```csharp
public class ConcreteObservable : IObservable
{
    // Liste aller aktuell registrierten Beobachter.
    private readonly List<IObserver> _observers = new();

    // Fügt einen neuen Beobachter hinzu.
    public void AddObserver(IObserver observer)
    {
        _observers.Add(observer);
    }

    // Entfernt einen Beobachter.
    public void RemoveObserver(IObserver observer)
    {
        _observers.Remove(observer);
    }

    // Informiert alle registrierten Beobachter.
    public void NotifyObservers()
    {
        foreach (IObserver observer in _observers)
        {
            observer.Update();
        }
    }
}
```

---

## ConcreteObserver

Ein konkreter Observer reagiert auf die Benachrichtigung.

```csharp
public class ConcreteObserver : IObserver
{
    public void Update()
    {
        Console.WriteLine(
            "Der Observer wurde über eine Änderung informiert.");
    }
}
```

---

# Verwendung

```csharp
ConcreteObservable observable = new();

IObserver observer1 =
    new ConcreteObserver();

IObserver observer2 =
    new ConcreteObserver();

observable.AddObserver(observer1);
observable.AddObserver(observer2);

// Beide Observer werden informiert.
observable.NotifyObservers();
```

---

# Wichtig

Das Observable kennt die konkreten Observer nicht.

Es weiß nur:

```csharp
IObserver
```

und dass dieser eine Methode besitzt:

```csharp
Update();
```

Dadurch entsteht eine relativ lose Kopplung:

```text
Observable
    │
    ▼
IObserver
    ▲
    │
┌───┴─────────────┐
│                 │
Bank             Broker
```

Das Observable muss nicht wissen:

```text
Was macht der Broker?
Was macht die Bank?
Wie reagieren sie?
```

Es informiert sie lediglich über die Änderung.

---

# Praktisches Beispiel: Börse

Angenommen, wir haben eine Börse.

An der Börse ändern sich regelmäßig Wechselkurse:

```text
USD
EUR
```

Mehrere Teilnehmer beobachten diese Kurse:

```text
Stock
├── Broker
└── Bank
```

Wenn sich die Kurse ändern:

```text
Stock
  │
  ├── Broker informieren
  └── Bank informieren
```

Broker und Bank reagieren unterschiedlich auf dieselben Informationen.

---

# Datenmodell: StockInfo

Zunächst benötigen wir ein Objekt, das den aktuellen Zustand der Börse enthält.

```csharp
// Enthält aktuelle Informationen über die Wechselkurse.
public class StockInfo
{
    public int Usd { get; set; }

    public int Euro { get; set; }
}
```

---

# Observer

Die Observer erhalten beim Update die aktuellen Börsendaten.

```csharp
public interface IObserver
{
    // Wird vom Observable aufgerufen,
    // wenn sich dessen Zustand geändert hat.
    void Update(StockInfo stockInfo);
}
```

---

# Observable

```csharp
public interface IObservable
{
    // Registriert einen neuen Beobachter.
    void RegisterObserver(IObserver observer);

    // Entfernt einen Beobachter.
    void RemoveObserver(IObserver observer);

    // Informiert alle registrierten Beobachter.
    void NotifyObservers();
}
```

---

# ConcreteObservable: Stock

Die Börse ist das Objekt, das beobachtet wird.

```csharp
public class Stock : IObservable
{
    // Aktuelle Informationen über die Börse.
    private readonly StockInfo _stockInfo = new();

    // Alle registrierten Beobachter.
    private readonly List<IObserver> _observers = new();

    // Registriert einen neuen Beobachter.
    public void RegisterObserver(IObserver observer)
    {
        _observers.Add(observer);
    }

    // Entfernt einen Beobachter.
    public void RemoveObserver(IObserver observer)
    {
        _observers.Remove(observer);
    }

    // Informiert alle Beobachter
    // und übergibt ihnen die aktuellen Börsendaten.
    public void NotifyObservers()
    {
        foreach (IObserver observer in _observers)
        {
            observer.Update(_stockInfo);
        }
    }

    // Simuliert Veränderungen an der Börse.
    public void Market()
    {
        _stockInfo.Usd =
            Random.Shared.Next(20, 40);

        _stockInfo.Euro =
            Random.Shared.Next(30, 50);

        // Nach der Änderung werden
        // alle Beobachter informiert.
        NotifyObservers();
    }
}
```

---

# Observer: Broker

Der Broker beobachtet den Dollar-Kurs.

```csharp
public class Broker : IObserver
{
    public string Name { get; }

    // Referenz auf die Börse,
    // damit sich der Broker später abmelden kann.
    private IObservable? _stock;

    public Broker(
        string name,
        IObservable stock)
    {
        Name = name;
        _stock = stock;

        // Broker registriert sich selbst
        // als Beobachter.
        _stock.RegisterObserver(this);
    }

    public void Update(StockInfo stockInfo)
    {
        if (stockInfo.Usd > 30)
        {
            Console.WriteLine(
                $"Broker {Name} verkauft Dollar. " +
                $"USD-Kurs: {stockInfo.Usd}");
        }
        else
        {
            Console.WriteLine(
                $"Broker {Name} kauft Dollar. " +
                $"USD-Kurs: {stockInfo.Usd}");
        }
    }

    // Broker beendet die Beobachtung.
    public void StopTrade()
    {
        if (_stock is null)
        {
            return;
        }

        _stock.RemoveObserver(this);

        _stock = null;
    }
}
```

---

# Observer: Bank

Die Bank beobachtet in unserem Beispiel den Euro-Kurs.

```csharp
public class Bank : IObserver
{
    public string Name { get; }

    public Bank(
        string name,
        IObservable stock)
    {
        Name = name;

        // Bank registriert sich
        // als Beobachter.
        stock.RegisterObserver(this);
    }

    public void Update(StockInfo stockInfo)
    {
        if (stockInfo.Euro > 40)
        {
            Console.WriteLine(
                $"Bank {Name} verkauft Euro. " +
                $"EUR-Kurs: {stockInfo.Euro}");
        }
        else
        {
            Console.WriteLine(
                $"Bank {Name} kauft Euro. " +
                $"EUR-Kurs: {stockInfo.Euro}");
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
        // Observable erstellen.
        Stock stock = new();

        // Observer registrieren sich
        // automatisch im Konstruktor.
        Bank bank =
            new("UnitBank", stock);

        Broker broker =
            new("Max Mustermann", stock);

        // Neue Börsendaten erzeugen.
        // Alle Observer werden informiert.
        stock.Market();

        // Broker meldet sich ab.
        broker.StopTrade();

        // Neue Börsendaten.
        // Jetzt wird nur noch die Bank informiert.
        stock.Market();
    }
}
```

---

# Ablauf

Am Anfang:

```text
                    Stock
                      │
            ┌─────────┴─────────┐
            ▼                   ▼
          Bank                Broker
```

Bei:

```csharp
stock.Market();
```

werden neue Daten erzeugt:

```text
USD = ...
EUR = ...
```

Danach:

```csharp
NotifyObservers();
```

und anschließend:

```text
Stock
 │
 ├── Bank.Update(stockInfo)
 │
 └── Broker.Update(stockInfo)
```

---

# Observer abmelden

Der Broker kann seine Registrierung entfernen:

```csharp
broker.StopTrade();
```

Intern:

```csharp
_stock.RemoveObserver(this);
```

Danach:

```text
Vorher:

           Stock
          /     \
       Bank     Broker


Nachher:

           Stock
             |
            Bank
```

Broker und Stock können trotzdem unabhängig voneinander weiterexistieren.

---

# Push-Modell

Im vorherigen Beispiel verwenden wir ein **Push-Modell**.

Das Observable sendet die Daten direkt an die Observer:

```csharp
observer.Update(_stockInfo);
```

Also:

```text
Stock
  │
  │ StockInfo
  ▼
Observer
```

Das Observable **pusht** die Daten zum Observer.

---

## Beispiel

```csharp
public interface IObserver
{
    void Update(StockInfo stockInfo);
}
```

Beim Update erhält der Observer direkt:

```text
USD
EUR
```

---

# Pull-Modell

Beim **Pull-Modell** informiert das Observable den Observer lediglich:

> „Mein Zustand hat sich geändert.“

Der Observer fragt die benötigten Daten anschließend selbst ab.

Zum Beispiel:

```csharp
public interface IObserver
{
    void Update();
}
```

Das Observable:

```csharp
public class Stock
{
    public StockInfo Info { get; private set; }

    // ...
}
```

Der Observer besitzt eine Referenz auf `Stock`:

```csharp
public void Update()
{
    StockInfo info = _stock.Info;

    // Daten auswerten ...
}
```

Ablauf:

```text
Observable
    │
    │ "Ich habe mich geändert"
    ▼
Observer
    │
    │ Daten selbst abfragen
    ▼
Observable
```

---

# Push vs. Pull

| Push | Pull |
|---|---|
| Observable sendet Daten | Observer holt Daten selbst |
| `Update(data)` | `Update()` |
| Observer bekommt Informationen direkt | Observer entscheidet, welche Daten benötigt werden |
| einfach bei kleinen Datenmengen | flexibel bei vielen Daten |

---

# Observer in modernem C#

In C# wird das Observer-Prinzip sehr häufig über **Events und Delegates** umgesetzt.

Man benötigt dann oft keine eigenen Interfaces wie:

```csharp
IObservable
IObserver
```

---

# Observer mit C# Events

Ein einfaches Beispiel:

```csharp
public class Stock
{
    // Event, auf das sich andere Objekte registrieren können.
    public event Action<StockInfo>? MarketChanged;

    public void Market()
    {
        StockInfo info = new()
        {
            Usd = Random.Shared.Next(20, 40),
            Euro = Random.Shared.Next(30, 50)
        };

        // Alle Abonnenten informieren.
        MarketChanged?.Invoke(info);
    }
}
```

Observer:

```csharp
public class Broker
{
    public string Name { get; }

    public Broker(
        string name,
        Stock stock)
    {
        Name = name;

        // Für das Event registrieren.
        stock.MarketChanged += OnMarketChanged;
    }

    private void OnMarketChanged(
        StockInfo stockInfo)
    {
        Console.WriteLine(
            $"{Name}: USD = {stockInfo.Usd}");
    }
}
```

---

# Anmeldung am Event

```csharp
stock.MarketChanged += OnMarketChanged;
```

bedeutet:

> „Wenn `MarketChanged` ausgelöst wird, rufe meine Methode `OnMarketChanged()` auf.“

---

# Abmeldung vom Event

```csharp
stock.MarketChanged -= OnMarketChanged;
```

bedeutet:

> „Ich möchte keine weiteren Benachrichtigungen erhalten.“

---

# Vollständiges modernes Beispiel

```csharp
public class StockInfo
{
    public int Usd { get; init; }

    public int Euro { get; init; }
}


public class Stock
{
    // Alle Subscriber können sich
    // für dieses Event registrieren.
    public event Action<StockInfo>? MarketChanged;

    public void Market()
    {
        StockInfo info = new()
        {
            Usd = Random.Shared.Next(20, 40),
            Euro = Random.Shared.Next(30, 50)
        };

        // Alle Subscriber benachrichtigen.
        MarketChanged?.Invoke(info);
    }
}


public class Broker
{
    public string Name { get; }

    private readonly Stock _stock;

    public Broker(
        string name,
        Stock stock)
    {
        Name = name;
        _stock = stock;

        // Beim Publisher anmelden.
        _stock.MarketChanged += OnMarketChanged;
    }

    private void OnMarketChanged(
        StockInfo info)
    {
        if (info.Usd > 30)
        {
            Console.WriteLine(
                $"{Name} verkauft Dollar.");
        }
        else
        {
            Console.WriteLine(
                $"{Name} kauft Dollar.");
        }
    }

    public void StopTrade()
    {
        // Vom Publisher abmelden.
        _stock.MarketChanged -= OnMarketChanged;
    }
}


public class Bank
{
    public string Name { get; }

    public Bank(
        string name,
        Stock stock)
    {
        Name = name;

        // Bank registriert sich für Änderungen.
        stock.MarketChanged += OnMarketChanged;
    }

    private void OnMarketChanged(
        StockInfo info)
    {
        if (info.Euro > 40)
        {
            Console.WriteLine(
                $"{Name} verkauft Euro.");
        }
        else
        {
            Console.WriteLine(
                $"{Name} kauft Euro.");
        }
    }
}
```

Verwendung:

```csharp
Stock stock = new();

Bank bank =
    new("UnitBank", stock);

Broker broker =
    new("Max Mustermann", stock);

stock.Market();

// Broker meldet sich ab.
broker.StopTrade();

stock.Market();
```

---

# Observer Pattern und C# Events

Die Zuordnung sieht ungefähr so aus:

| Observer Pattern | C# |
|---|---|
| Observable / Publisher | Klasse mit `event` |
| Observer / Subscriber | Klasse mit Event-Handler |
| `RegisterObserver()` | `+=` |
| `RemoveObserver()` | `-=` |
| `NotifyObservers()` | `Invoke()` |
| `Update()` | Event-Handler |

Zum Beispiel:

```text
RegisterObserver()
        ↓
       +=

RemoveObserver()
        ↓
       -=

NotifyObservers()
        ↓
    Invoke()
```

---

# Wichtig: Event-Abmeldung

Bei Observer-Systemen sollte man darauf achten, sich wieder abzumelden:

```csharp
publisher.Event -= Handler;
```

Besonders wenn:

- Publisher deutlich länger lebt als Subscriber;
- sehr viele Subscriber erzeugt werden;
- Subscriber eigentlich nicht mehr benötigt werden.

Andernfalls kann der Publisher weiterhin eine Referenz auf den Subscriber halten.

---

# Vorteile

- Publisher und Subscriber sind relativ lose gekoppelt.
- Ein Publisher kann beliebig viele Observer informieren.
- Observer können zur Laufzeit hinzugefügt werden.
- Observer können sich wieder abmelden.
- Neue Observer können hinzugefügt werden, ohne den Publisher zu verändern.
- Sehr gut für Event-basierte Architekturen geeignet.
- Sehr häufig in Benutzeroberflächen.

---

# Nachteile

- Bei vielen Observern kann schwer nachvollziehbar werden, wer auf welches Ereignis reagiert.
- Die Reihenfolge der Benachrichtigungen kann relevant werden.
- Ein langsamer Observer kann bei synchroner Ausführung andere Observer blockieren.
- Vergessene Event-Abmeldungen können zu unerwünschten Referenzen führen.
- Sehr viele Events können den Kontrollfluss schwer nachvollziehbar machen.

---

# Typische Einsatzgebiete

Observer findet man sehr häufig bei:

```text
UI Events
ViewModels
Benachrichtigungen
Domain Events
Messaging
Datenänderungen
Live Updates
Callbacks
Event-driven Systems
```

---

# Beispiel aus WinUI / WPF / MVVM

Ein sehr bekanntes Observer-Prinzip in .NET ist:

```csharp
INotifyPropertyChanged
```

Zum Beispiel:

```csharp
public class UserViewModel : INotifyPropertyChanged
{
    private string _name = string.Empty;

    public string Name
    {
        get => _name;

        set
        {
            _name = value;

            PropertyChanged?.Invoke(
                this,
                new PropertyChangedEventArgs(
                    nameof(Name)));
        }
    }

    public event PropertyChangedEventHandler?
        PropertyChanged;
}
```

Wenn:

```csharp
Name = "Roman";
```

geändert wird, werden interessierte Komponenten informiert.

Das ist im Prinzip ebenfalls das **Observer Pattern**.

---

# Observer vs. Strategy

## Strategy

```text
Context
   │
   ▼
Strategy
```

Der Context verwendet **eine Strategie**, um ein bestimmtes Verhalten auszuführen.

> **Strategy → Welchen Algorithmus verwende ich?**

---

## Observer

```text
Publisher
   │
   ├── Observer 1
   ├── Observer 2
   └── Observer 3
```

Ein Objekt informiert mehrere andere Objekte.

> **Observer → Wer soll informiert werden, wenn sich etwas ändert?**

---

# Observer vs. Mediator

## Observer

Ein Publisher informiert Subscriber direkt über deren gemeinsame Abstraktion.

```text
Publisher
 ├── Observer A
 ├── Observer B
 └── Observer C
```

## Mediator

Objekte kommunizieren über eine zentrale Vermittlungsstelle.

```text
Object A
    │
    ▼
 Mediator
    ▲
    │
Object B
```

---

# Vereinfachte Struktur

```text
                    Publisher
                        │
                        │ Notify
                        ▼
                    IObserver
                        ▲
              ┌─────────┼─────────┐
              │         │         │
           ObserverA ObserverB ObserverC
```

---

# Moderner C#-Ansatz

Sehr häufig:

```csharp
public event Action<Data>? Changed;
```

Anmelden:

```csharp
publisher.Changed += Handler;
```

Benachrichtigen:

```csharp
Changed?.Invoke(data);
```

Abmelden:

```csharp
publisher.Changed -= Handler;
```

---

# Merksatz

> **Observer informiert automatisch mehrere interessierte Objekte, wenn sich der Zustand eines beobachteten Objekts ändert.**

Noch einfacher:

```text
Publisher:
"Bei mir hat sich etwas geändert."

Observer:
"Dann reagiere ich darauf."
```

Oder:

```text
Observer
=
1 Publisher
+
n Subscriber
+
automatische Benachrichtigung
```

---

> [!summary] Zusammenfassung
> Das **Observer Pattern** beschreibt eine **1:n-Beziehung** zwischen einem beobachteten Objekt und mehreren Beobachtern.
>
> Wenn sich der Publisher verändert, werden alle registrierten Subscriber automatisch informiert.
>
> Klassisch:
>
> ```text
> IObservable
> ├── RegisterObserver()
> ├── RemoveObserver()
> └── NotifyObservers()
>
> IObserver
> └── Update()
> ```
>
> In modernem C# wird dieses Prinzip sehr häufig über **Events und Delegates** umgesetzt:
>
> ```csharp
> publisher.Changed += Handler;
> publisher.Changed -= Handler;
> ```
>
> Ein sehr bekanntes Beispiel aus .NET ist `INotifyPropertyChanged`.
>
> **Kurz gesagt:**
>
> `Observer = auf Änderungen reagieren und mehrere Abonnenten automatisch informieren.`