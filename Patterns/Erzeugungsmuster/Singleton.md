## Singleton

Der **Singleton** ist ein Erzeugungsmuster, das sicherstellt, dass von einer bestimmten Klasse **nur eine einzige Instanz** existiert und dass auf diese Instanz zentral zugegriffen werden kann.

> [!info] Grundidee  
> Von einer Klasse soll innerhalb der Anwendung nur **ein Objekt** existieren.

Typische Beispiele können sein:

- zentrale Konfiguration
- Cache
- Logging-Service
- zentrale Ressourcenverwaltung
- gemeinsame Statusverwaltung

---

# Wann sollte man Singleton verwenden?

Singleton kann sinnvoll sein, wenn:

- tatsächlich nur **eine Instanz** eines Services existieren soll;
- mehrere Teile der Anwendung dieselbe Instanz verwenden müssen;
- die Lebensdauer des Objekts der gesamten Anwendung entsprechen soll;
- die Erstellung des Objekts zentral kontrolliert werden muss.

> [!warning]  
> Singleton sollte nicht automatisch für jedes globale Objekt verwendet werden.
> 
> Besonders in modernen .NET-Anwendungen wird häufig **Dependency Injection** mit einer Singleton-Lebensdauer bevorzugt.

Beispiel in ASP.NET Core:

```
services.AddSingleton<MyService>();
```

Dadurch übernimmt der DI-Container die Lebensdauer des Objekts.

---

# Klassische Struktur

Ein Singleton benötigt normalerweise drei Dinge:

```
1. private Konstruktor
2. statisches Feld mit der einzigen Instanz
3. öffentliche Zugriffsmöglichkeit auf diese Instanz
```

---

# Klassische Implementierung

```
public class Singleton
{
    // Speichert die einzige Instanz der Klasse.
    private static Singleton? _instance;

    // Privater Konstruktor verhindert,
    // dass außerhalb der Klasse Objekte mit "new" erstellt werden.
    private Singleton()
    {
    }

    // Liefert immer dieselbe Singleton-Instanz zurück.
    public static Singleton GetInstance()
    {
        // Das Objekt wird erst beim ersten Zugriff erstellt.
        if (_instance == null)
        {
            _instance = new Singleton();
        }

        return _instance;
    }
}
```

Von außen ist Folgendes nicht möglich:

```
var singleton = new Singleton();
```

weil der Konstruktor:

```
private Singleton()
```

ist.

Stattdessen:

```
Singleton singleton = Singleton.GetInstance();
```

---

# Warum ist der Konstruktor `private`?

Normalerweise kann man eine Klasse folgendermaßen instanziieren:

```
var obj1 = new MyClass();
var obj2 = new MyClass();
var obj3 = new MyClass();
```

Damit entstehen drei unterschiedliche Objekte.

Beim Singleton soll genau das verhindert werden.

Deshalb:

```
private Singleton()
{
}
```

Nur die Klasse selbst kann ihren Konstruktor aufrufen.

---

# Lazy Initialization

Die klassische Implementierung:

```
if (_instance == null)
{
    _instance = new Singleton();
}
```

erzeugt das Objekt erst dann, wenn es tatsächlich benötigt wird.

Das nennt man:

**Lazy Initialization**  
= verzögerte Initialisierung.

```
Programm startet
     │
     │
     │ Singleton wird noch nicht benötigt
     │
     ▼
GetInstance()
     │
     ▼
Singleton wird jetzt erstellt
```

Wenn niemand auf das Singleton zugreift, wird das Objekt nicht erzeugt.

---

# Einfaches Beispiel

Angenommen, unsere Anwendung besitzt einen zentralen Konfigurationsmanager.

```
public class AppSettings
{
    // Die einzige Instanz.
    private static AppSettings? _instance;

    // Beispiel einer global verwendeten Einstellung.
    public string Language { get; set; } = "Deutsch";

    // Verhindert die direkte Erstellung von außen.
    private AppSettings()
    {
    }

    // Gibt immer dieselbe Instanz zurück.
    public static AppSettings GetInstance()
    {
        if (_instance == null)
        {
            _instance = new AppSettings();
        }

        return _instance;
    }
}
```

Verwendung:

```
AppSettings settings1 = AppSettings.GetInstance();
AppSettings settings2 = AppSettings.GetInstance();

settings1.Language = "English";

Console.WriteLine(settings2.Language);
```

Ausgabe:

```
English
```

Warum?

Weil:

```
settings1
```

und:

```
settings2
```

auf dasselbe Objekt zeigen.

---

# Überprüfung

```
AppSettings settings1 = AppSettings.GetInstance();
AppSettings settings2 = AppSettings.GetInstance();

Console.WriteLine(
    ReferenceEquals(settings1, settings2));
```

Ausgabe:

```
True
```

Also:

```
settings1 ─────┐
               │
               ▼
          AppSettings
               ▲
               │
settings2 ─────┘
```

Es existiert nur **eine Instanz**.

---

# Problem: Multithreading

Die einfache Implementierung ist **nicht threadsicher**.

Problematisch ist dieser Code:

```
if (_instance == null)
{
    _instance = new Singleton();
}
```

Stellen wir uns zwei Threads vor:

```
Thread 1                    Thread 2

_instance == null           _instance == null
       │                           │
       ▼                           ▼
new Singleton()             new Singleton()
```

Beide Threads können gleichzeitig feststellen:

```
_instance == null
```

und beide erzeugen anschließend ein Objekt.

Damit wäre die zentrale Garantie des Singleton-Musters verletzt.

---

# Beispiel für das Problem

```
public class Singleton
{
    private static Singleton? _instance;

    private Singleton()
    {
        Console.WriteLine("Singleton wurde erstellt.");
    }

    public static Singleton GetInstance()
    {
        if (_instance == null)
        {
            _instance = new Singleton();
        }

        return _instance;
    }
}
```

Bei parallelem Zugriff:

```
Thread thread1 = new(() =>
{
    Singleton.GetInstance();
});

Thread thread2 = new(() =>
{
    Singleton.GetInstance();
});

thread1.Start();
thread2.Start();

thread1.Join();
thread2.Join();
```

können theoretisch zwei Instanzen erzeugt werden.

---

# Threadsichere Variante mit `lock`

Eine Möglichkeit besteht darin, den kritischen Bereich mit `lock` zu schützen.

```
public class Singleton
{
    private static Singleton? _instance;

    // Dieses Objekt wird ausschließlich
    // für die Synchronisation verwendet.
    private static readonly object _lock = new();

    private Singleton()
    {
    }

    public static Singleton GetInstance()
    {
        // Nur ein Thread darf diesen Bereich
        // gleichzeitig ausführen.
        lock (_lock)
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }

            return _instance;
        }
    }
}
```

Jetzt kann immer nur ein Thread gleichzeitig den geschützten Bereich betreten.

---

# Nachteil dieser Variante

Bei jedem Aufruf wird:

```
lock (_lock)
```

ausgeführt.

Auch nachdem das Singleton bereits erzeugt wurde.

Das bedeutet zusätzlichen Synchronisationsaufwand.

Deshalb wurde früher häufig **Double-Checked Locking** verwendet.

---

# Double-Checked Locking

```
public class Singleton
{
    private static Singleton? _instance;

    private static readonly object _lock = new();

    private Singleton()
    {
    }

    public static Singleton GetInstance()
    {
        // Erste Prüfung ohne Lock.
        if (_instance == null)
        {
            lock (_lock)
            {
                // Zweite Prüfung innerhalb des Locks.
                if (_instance == null)
                {
                    _instance = new Singleton();
                }
            }
        }

        return _instance;
    }
}
```

Ablauf:

```
Ist Instanz vorhanden?
        │
     Ja │ Nein
        │
        ▼
      lock
        │
        ▼
Noch einmal prüfen
        │
        ▼
Instanz erzeugen
```

> [!note]  
> Diese Variante findet man häufig in älteren Beispielen.
> 
> In modernem C# gibt es einfachere und besser lesbare Möglichkeiten.

---

# Threadsichere Variante mit `static readonly`

.NET garantiert die threadsichere Initialisierung statischer Felder.

Dadurch kann man Singleton wesentlich einfacher implementieren:

```
public sealed class Singleton
{
    // Die CLR sorgt für die threadsichere Initialisierung.
    private static readonly Singleton _instance =
        new Singleton();

    private Singleton()
    {
    }

    // Zentraler Zugriff auf die einzige Instanz.
    public static Singleton Instance => _instance;
}
```

Verwendung:

```
Singleton singleton1 = Singleton.Instance;
Singleton singleton2 = Singleton.Instance;
```

Beide Variablen zeigen auf dasselbe Objekt.

---

# Warum `sealed`?

Bei Singleton-Klassen sieht man häufig:

```
public sealed class Singleton
```

`sealed` verhindert Vererbung.

Ohne `sealed` könnte eine abgeleitete Klasse theoretisch zusätzliche Instanzen ermöglichen oder die Singleton-Idee komplizierter machen.

Deshalb ist:

```
sealed
```

bei Singleton meistens sinnvoll.

---

# Eager Initialization

Bei:

```
private static readonly Singleton _instance =
    new Singleton();
```

wird die Instanz durch die statische Initialisierung erzeugt.

Das bezeichnet man häufig als:

**Eager Initialization**

also:

> Die Instanz wird automatisch initialisiert, sobald der entsprechende Typ initialisiert werden muss.

---

# Lazy Singleton mit verschachtelter Klasse

Eine klassische Alternative ist eine verschachtelte Klasse.

```
public sealed class Singleton
{
    private Singleton()
    {
        Console.WriteLine("Singleton wurde erstellt.");
    }

    // Zugriff auf die Singleton-Instanz.
    public static Singleton Instance =>
        Nested.Instance;

    // Diese Klasse wird erst initialisiert,
    // wenn tatsächlich auf Nested.Instance zugegriffen wird.
    private static class Nested
    {
        internal static readonly Singleton Instance =
            new Singleton();
    }
}
```

Dadurch wird das Singleton erst erzeugt, wenn:

```
Singleton.Instance
```

zum ersten Mal verwendet wird.

---

# Warum eine verschachtelte Klasse?

Angenommen, `Singleton` besitzt zusätzlich andere statische Mitglieder:

```
public static string Text = "Hallo";
```

Wir möchten vielleicht:

```
Console.WriteLine(Singleton.Text);
```

verwenden, ohne dass die Singleton-Instanz bereits erzeugt wird.

Mit einer separaten verschachtelten Klasse können wir die Initialisierung der Instanz von anderen statischen Mitgliedern trennen.

---

# Moderne Variante: `Lazy<T>`

In modernem C# ist `Lazy<T>` eine sehr saubere Lösung.

```
public sealed class Singleton
{
    // Lazy sorgt dafür, dass Singleton erst beim
    // ersten Zugriff auf Value erzeugt wird.
    private static readonly Lazy<Singleton> _lazy =
        new(() => new Singleton());

    private Singleton()
    {
    }

    // Öffentlicher Zugriff auf die einzige Instanz.
    public static Singleton Instance =>
        _lazy.Value;
}
```

Verwendung:

```
Singleton singleton1 = Singleton.Instance;
Singleton singleton2 = Singleton.Instance;

Console.WriteLine(
    ReferenceEquals(singleton1, singleton2));
```

Ausgabe:

```
True
```

---

# Was macht `Lazy<T>`?

Dieser Code:

```
private static readonly Lazy<Singleton> _lazy =
    new(() => new Singleton());
```

erzeugt zunächst **nicht** das eigentliche `Singleton`.

Es entsteht zunächst lediglich ein `Lazy<Singleton>`-Objekt.

Erst bei:

```
_lazy.Value
```

wird ausgeführt:

```
new Singleton()
```

Ablauf:

```
Programm startet
       │
       ▼
Lazy<Singleton> existiert
       │
       │ Singleton existiert noch nicht
       │
       ▼
Singleton.Instance
       │
       ▼
_lazy.Value
       │
       ▼
new Singleton()
```

Danach wird bei jedem weiteren Zugriff dieselbe Instanz zurückgegeben.

---

# `Lazy<T>` und Threadsicherheit

Ein großer Vorteil:

> `Lazy<T>` ist standardmäßig threadsicher.

Wenn mehrere Threads gleichzeitig:

```
Singleton.Instance
```

aufrufen, sorgt `Lazy<T>` dafür, dass die Instanz korrekt initialisiert wird.

Deshalb ist diese Variante meistens deutlich angenehmer als selbst geschriebenes:

```
lock
```

oder Double-Checked Locking.

---

# Empfohlene moderne Implementierung

Für Lernzwecke würde ich mir vor allem diese Variante merken:

```
public sealed class Singleton
{
    // Threadsafe und lazy.
    private static readonly Lazy<Singleton> _lazy =
        new(() => new Singleton());

    // Privater Konstruktor verhindert
    // die direkte Instanziierung von außen.
    private Singleton()
    {
    }

    // Globaler Zugriff auf die einzige Instanz.
    public static Singleton Instance =>
        _lazy.Value;
}
```

Das ist:

- threadsicher;
- lazy;
- kurz;
- leicht verständlich;
- idiomatisch für modernes C#.

---

# Warum Property statt `GetInstance()`?

Ältere Beispiele verwenden häufig:

```
Singleton.GetInstance();
```

In modernem C# ist auch diese Schreibweise sehr verbreitet:

```
Singleton.Instance
```

Zum Beispiel:

```
public static Singleton Instance =>
    _lazy.Value;
```

Die Verwendung sieht dann sauberer aus:

```
Singleton singleton = Singleton.Instance;
```

statt:

```
Singleton singleton = Singleton.GetInstance();
```

Beide Varianten funktionieren. `Instance` ist bei Singleton-Klassen jedoch sehr üblich.

---

# Singleton vs. globale Variable

Singleton und globale Variablen haben Gemeinsamkeiten, sind aber nicht identisch.

Eine globale beziehungsweise statische Variable könnte beispielsweise direkt erzeugt werden:

```
public static AppSettings Settings =
    new AppSettings();
```

Beim Singleton kontrolliert die Klasse dagegen selbst:

- wie die Instanz erzeugt wird;
- wann sie erzeugt wird;
- dass nur eine Instanz existiert;
- wie auf sie zugegriffen wird.

Singleton kapselt also die Lebensdauer und Erzeugung des Objekts.

---

# Beispiel: Logger

Ein verständlicheres Beispiel als eine Betriebssystem-Klasse ist ein Logger.

```
public sealed class Logger
{
    // Lazy sorgt für threadsichere
    // und verzögerte Initialisierung.
    private static readonly Lazy<Logger> _lazy =
        new(() => new Logger());

    private Logger()
    {
    }

    public static Logger Instance =>
        _lazy.Value;

    // Gemeinsame Funktionalität des Loggers.
    public void Log(string message)
    {
        Console.WriteLine(
            $"[{DateTime.Now:HH:mm:ss}] {message}");
    }
}
```

Verwendung:

```
Logger.Instance.Log("Anwendung gestartet.");

Logger.Instance.Log("Benutzer angemeldet.");
```

Beide Aufrufe verwenden denselben `Logger`.

---

# Nachteile des Singleton-Patterns

Singleton ist zwar einfach, besitzt aber einige wichtige Nachteile.

## 1. Globale Abhängigkeit

Eine Klasse kann einfach überall schreiben:

```
Logger.Instance.Log("Test");
```

Damit entsteht eine versteckte Abhängigkeit zum `Logger`.

Man erkennt am Konstruktor der Klasse nicht unbedingt, dass sie einen Logger benötigt.

---

## 2. Unit Tests werden schwieriger

Angenommen:

```
public class OrderService
{
    public void CreateOrder()
    {
        Logger.Instance.Log("Bestellung erstellt.");
    }
}
```

`OrderService` ist jetzt direkt mit dem konkreten Singleton verbunden.

Beim Unit Test kann man den Logger nicht einfach durch einen Mock ersetzen.

Mit Dependency Injection wäre das einfacher:

```
public class OrderService
{
    private readonly ILogger _logger;

    public OrderService(ILogger logger)
    {
        _logger = logger;
    }
}
```

Jetzt kann im Test ein Mock übergeben werden.

---

## 3. Gemeinsamer veränderbarer Zustand

Wenn das Singleton veränderbare Daten enthält:

```
public int CurrentUserId { get; set; }
```

können viele Stellen der Anwendung denselben Zustand verändern.

Das kann zu schwer nachvollziehbaren Fehlern führen.

Besonders problematisch wird dies bei Multithreading.

---

## 4. Starke Kopplung

Wenn Klassen direkt schreiben:

```
Database.Instance.Save();
```

sind sie stark an diese konkrete Singleton-Implementierung gekoppelt.

---

# Singleton und Dependency Injection

In modernen .NET-Projekten wird häufig nicht mehr manuell:

```
Singleton.Instance
```

verwendet.

Stattdessen registriert man einen Service beim DI-Container als Singleton.

Zum Beispiel in ASP.NET Core:

```
builder.Services.AddSingleton<ICacheService, CacheService>();
```

Danach:

```
public class ProductService
{
    private readonly ICacheService _cache;

    public ProductService(ICacheService cache)
    {
        _cache = cache;
    }
}
```

Der Dependency-Injection-Container garantiert:

```
ICacheService
      │
      ▼
eine Instanz für die entsprechende
Singleton-Lebensdauer
```

Der große Vorteil:

`ProductService` kennt nicht:

```
CacheService.Instance
```

sondern nur:

```
ICacheService
```

Das verbessert:

- Testbarkeit
- Austauschbarkeit
- Entkopplung
- Architektur

---

# Klassisches Singleton vs. DI Singleton

|Klassisches Singleton|Dependency Injection Singleton|
|---|---|
|`Singleton.Instance`|Konstruktor-Injection|
|Klasse kontrolliert ihre Instanz selbst|DI-Container kontrolliert die Lebensdauer|
|häufig globale Abhängigkeit|Abhängigkeit ist explizit|
|schwieriger zu mocken|gut testbar|
|starke Kopplung möglich|Arbeit mit Interfaces möglich|
|sinnvoll für kleine Fälle|meist besser für größere .NET-Anwendungen|

---

# Die wichtigsten Varianten

## 1. Einfache Variante

```
private static Singleton? _instance;
```

mit:

```
if (_instance == null)
{
    _instance = new Singleton();
}
```

❌ Nicht threadsicher.

---

## 2. Mit `lock`

```
lock (_lock)
{
    if (_instance == null)
    {
        _instance = new Singleton();
    }
}
```

✅ Threadsicher  
❌ Mehr eigener Synchronisationscode

---

## 3. `static readonly`

```
private static readonly Singleton _instance =
    new Singleton();
```

✅ Einfach  
✅ Threadsicher  
❌ Nicht explizit lazy im selben Sinn wie `Lazy<T>`

---

## 4. `Lazy<T>`

```
private static readonly Lazy<Singleton> _lazy =
    new(() => new Singleton());
```

✅ Threadsicher  
✅ Lazy Initialization  
✅ Wenig Code  
✅ Sehr gut verständlich

> [!tip]  
> Für ein klassisches manuell implementiertes Singleton in modernem C# ist `Lazy<T>` meistens die Variante, die man sich merken sollte.

---

# Struktur des Patterns

```
             Singleton
                 │
      ┌──────────┴──────────┐
      │                     │
private Konstruktor    static Instance
      │                     │
      │                     ▼
      └──────────────► einzige Instanz
```

---

# Ablauf

```
Singleton.Instance
        │
        ▼
Existiert bereits eine Instanz?
        │
     ┌──┴──┐
    Ja    Nein
     │      │
     │      ▼
     │   Instanz erzeugen
     │      │
     └──┬───┘
        ▼
gleiche Instanz zurückgeben
```

---

# Merksatz

> **Singleton stellt sicher, dass eine Klasse nur eine Instanz besitzt und bietet einen zentralen Zugriff auf diese Instanz.**

Noch kürzer:

```
Singleton
=
eine Klasse
+
eine Instanz
+
ein zentraler Zugriff
```

---

# Sehr wichtig für moderne .NET-Projekte

Das Pattern sollte man kennen, aber nicht überall einsetzen.

Für klassische Lernaufgaben:

```
public sealed class Singleton
{
    private static readonly Lazy<Singleton> _lazy =
        new(() => new Singleton());

    private Singleton()
    {
    }

    public static Singleton Instance =>
        _lazy.Value;
}
```

Für größere Anwendungen mit Dependency Injection ist häufig eher:

```
services.AddSingleton<IService, Service>();
```

die bessere Lösung.

> [!summary] Zusammenfassung  
> Der **Singleton** garantiert, dass von einer Klasse nur **eine Instanz** existiert.
> 
> Der Konstruktor ist normalerweise `private`, damit außerhalb der Klasse keine weiteren Objekte erzeugt werden können.
> 
> Für threadsichere und verzögerte Initialisierung bietet sich in modernem C# besonders `Lazy<T>` an.
> 
> In größeren .NET-Anwendungen sollte man jedoch häufig **Dependency Injection mit Singleton-Lifetime** einem klassischen globalen `Singleton.Instance` vorziehen.