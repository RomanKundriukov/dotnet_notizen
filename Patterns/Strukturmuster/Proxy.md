Das **Proxy Pattern** ist ein Strukturmuster, bei dem ein Stellvertreter-Objekt den Zugriff auf ein anderes, reales Objekt kontrolliert.

Der Proxy besitzt dabei normalerweise **dieselbe Schnittstelle** wie das reale Objekt. Dadurch kann der Client mit dem Proxy arbeiten, ohne unbedingt zu wissen, ob tatsächlich der Proxy oder das reale Objekt verwendet wird.

> [!info] Grundidee  
> Der Client greift nicht direkt auf das reale Objekt zu.
> 
> Stattdessen befindet sich ein **Proxy** dazwischen, der den Zugriff kontrollieren, verzögern, absichern, überwachen oder optimieren kann.

Vereinfacht:

```
Client
  │
  ▼
Proxy
  │
  ▼
RealSubject
```

Der Client arbeitet beispielsweise mit:

```
subject.Request();
```

Ob `subject` dabei ein echtes Objekt oder ein Proxy ist, muss ihn nicht interessieren.

---

# Wann sollte man Proxy verwenden?

Das Proxy Pattern eignet sich besonders:

- wenn der Zugriff auf ein Objekt kontrolliert werden soll;
- wenn ein Objekt teuer zu erstellen ist und erst bei Bedarf erzeugt werden soll;
- wenn ein entferntes Objekt über Netzwerk angesprochen werden soll;
- wenn Berechtigungen vor dem eigentlichen Zugriff geprüft werden sollen;
- wenn Logging oder Monitoring vor einem Methodenaufruf nötig ist;
- wenn Ergebnisse zwischengespeichert werden sollen;
- wenn zusätzliche Synchronisation oder Thread-Sicherheit erforderlich ist;
- wenn der Client vom eigentlichen Objekt entkoppelt bleiben soll.

---

# Grundstruktur

```
                 Subject
                    ▲
          ┌─────────┴─────────┐
          │                   │
        Proxy            RealSubject
          │
          │ verwendet
          ▼
      RealSubject
```

Sowohl `Proxy` als auch `RealSubject` implementieren dieselbe Abstraktion:

```
Subject
```

Dadurch können beide an derselben Stelle verwendet werden.

---

# Teilnehmer des Patterns

## Subject

`Subject` definiert die gemeinsame Schnittstelle für Proxy und reales Objekt.

```
public interface ISubject
{
    void Request();
}
```

---

## RealSubject

Der `RealSubject` enthält die eigentliche Funktionalität.

```
public class RealSubject : ISubject
{
    public void Request()
    {
        Console.WriteLine(
            "Die eigentliche Operation wird ausgeführt.");
    }
}
```

---

## Proxy

Der Proxy implementiert dieselbe Schnittstelle:

```
public class Proxy : ISubject
{
    private RealSubject? _realSubject;

    public void Request()
    {
        // Das reale Objekt wird erst erzeugt,
        // wenn es tatsächlich benötigt wird.
        _realSubject ??=
            new RealSubject();

        // Anfrage an das reale Objekt weiterleiten.
        _realSubject.Request();
    }
}
```

---

## Client

Der Client kennt nur:

```
ISubject
```

Zum Beispiel:

```
public class Client
{
    public void Execute(ISubject subject)
    {
        subject.Request();
    }
}
```

Der Client muss nicht wissen:

```
Benutze ich RealSubject?
oder
Benutze ich Proxy?
```

---

# Allgemeines Beispiel

```
ISubject subject =
    new Proxy();

subject.Request();
```

Der Ablauf:

```
Client
  │
  │ Request()
  ▼
Proxy
  │
  │ RealSubject vorhanden?
  │
  ├── Nein → erzeugen
  │
  ▼
RealSubject
  │
  │ Request()
  ▼
Operation
```

---

# Warum ist Proxy austauschbar?

Weil beide dieselbe Schnittstelle implementieren:

```
ISubject
├── RealSubject
└── Proxy
```

Deshalb kann der Client schreiben:

```
ISubject subject;
```

und später:

```
subject =
    new RealSubject();
```

oder:

```
subject =
    new Proxy();
```

Der Client-Code bleibt gleich.

---

# Arten von Proxy

Es gibt verschiedene typische Proxy-Varianten.

Die wichtigsten sind:

```
Virtual Proxy
Remote Proxy
Protection Proxy
Caching Proxy
Logging Proxy
Smart Reference
```

---

# Virtual Proxy

Ein **Virtual Proxy** verzögert die Erstellung eines teuren Objekts.

Das reale Objekt wird erst erzeugt, wenn es tatsächlich gebraucht wird.

Beispiel:

```
Client
  │
  ▼
ImageProxy
  │
  │ Bild wirklich anzeigen?
  ▼
LargeImage
```

---

# Beispiel: großes Bild

Angenommen, das Laden eines Bildes dauert lange:

```
public class LargeImage : IImage
{
    private readonly string _path;

    public LargeImage(string path)
    {
        _path = path;

        // Simuliert einen teuren Ladevorgang.
        LoadImageFromDisk();
    }

    private void LoadImageFromDisk()
    {
        Console.WriteLine(
            $"Bild '{_path}' wird geladen.");
    }

    public void Display()
    {
        Console.WriteLine(
            $"Bild '{_path}' wird angezeigt.");
    }
}
```

Interface:

```
public interface IImage
{
    void Display();
}
```

---

# ImageProxy

```
public class ImageProxy : IImage
{
    private readonly string _path;

    private LargeImage? _image;

    public ImageProxy(string path)
    {
        _path = path;
    }

    public void Display()
    {
        // Lazy Initialization:
        // Das teure Objekt wird erst jetzt erzeugt.
        _image ??=
            new LargeImage(_path);

        _image.Display();
    }
}
```

---

# Verwendung

```
IImage image =
    new ImageProxy("photo.jpg");

Console.WriteLine(
    "Proxy wurde erzeugt.");

image.Display();
```

Die Datei wird nicht beim:

```
new ImageProxy(...)
```

geladen.

Erst bei:

```
image.Display();
```

wird das reale Objekt erzeugt.

---

# Ablauf

```
new ImageProxy()
       │
       ▼
kein Bild geladen


Display()
   │
   ▼
ImageProxy
   │
   ▼
LargeImage erzeugen
   │
   ▼
Bild laden
   │
   ▼
anzeigen
```

Das ist ein klassischer **Virtual Proxy**.

---

# Remote Proxy

Ein **Remote Proxy** repräsentiert ein Objekt, das sich möglicherweise:

```
auf einem anderen Server
in einem anderen Prozess
in einem anderen Adressraum
```

befindet.

Der Client arbeitet scheinbar mit einem normalen Objekt:

```
service.GetCustomer(42);
```

Der Proxy könnte intern jedoch:

```
HTTP Request
gRPC
TCP
IPC
```

verwenden.

---

# Beispiel

```
Client
  │
  │ GetOrder(42)
  ▼
OrderServiceProxy
  │
  │ HTTP / gRPC
  ▼
Remote Server
  │
  ▼
OrderService
```

Für den Client bleibt die Verwendung möglichst einfach.

---

# Vereinfachtes Beispiel

```
public interface IWeatherService
{
    Task<string> GetWeatherAsync(
        string city);
}
```

Proxy:

```
public class WeatherServiceProxy
    : IWeatherService
{
    private readonly HttpClient _httpClient;

    public WeatherServiceProxy(
        HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> GetWeatherAsync(
        string city)
    {
        // Der Proxy übersetzt den lokalen Methodenaufruf
        // in einen Netzwerkaufruf.
        return await _httpClient.GetStringAsync(
            $"weather/{city}");
    }
}
```

Der Client kennt weiterhin nur:

```
IWeatherService
```

---

# Protection Proxy

Ein **Protection Proxy** überprüft, ob ein Client auf das reale Objekt zugreifen darf.

Beispiel:

```
Client
  │
  ▼
SecurityProxy
  │
  │ Berechtigung prüfen
  ▼
RealService
```

---

# Beispiel

```
public interface IDocumentService
{
    void DeleteDocument(int id);
}
```

RealSubject:

```
public class DocumentService
    : IDocumentService
{
    public void DeleteDocument(int id)
    {
        Console.WriteLine(
            $"Dokument {id} wird gelöscht.");
    }
}
```

Proxy:

```
public class DocumentServiceProxy
    : IDocumentService
{
    private readonly IDocumentService _inner;

    private readonly User _currentUser;

    public DocumentServiceProxy(
        IDocumentService inner,
        User currentUser)
    {
        _inner = inner;
        _currentUser = currentUser;
    }

    public void DeleteDocument(int id)
    {
        // Zugriff kontrollieren,
        // bevor das reale Objekt aufgerufen wird.
        if (!_currentUser.IsAdministrator)
        {
            throw new UnauthorizedAccessException(
                "Der Benutzer darf keine Dokumente löschen.");
        }

        _inner.DeleteDocument(id);
    }
}
```

---

# Ablauf

```
DeleteDocument()
      │
      ▼
ProtectionProxy
      │
      ├── Berechtigung vorhanden?
      │
      ├── Nein → Fehler
      │
      └── Ja
           │
           ▼
      RealService
```

---

# Caching Proxy

Ein Proxy kann auch Ergebnisse zwischenspeichern.

Genau dieses Prinzip verwendet das Beispiel mit den Buchseiten aus der ursprünglichen Quelle.

```
Client
  │
  ▼
BookStoreProxy
  │
  ├── Cache prüfen
  │
  ├── Treffer → direkt zurückgeben
  │
  └── kein Treffer
  │        │
  │        ▼
  │    BookStore
  │        │
  │        ▼
  │    Datenbank
  │
  └── Ergebnis cachen
```

---

# Praktisches Beispiel: Buchseiten

Wir möchten Seiten eines Buches laden.

Ein direkter Datenbankzugriff kann teuer sein.

Darum möchten wir bereits geladene Seiten zwischenspeichern.

---

# Gemeinsames Interface

```
public interface IBookStore
{
    Page? GetPage(int number);
}
```

---

# Page

```
public sealed class Page
{
    public int Id { get; init; }

    public int Number { get; init; }

    public string Text { get; init; } =
        string.Empty;
}
```

---

# RealSubject: BookStore

```
public class BookStore : IBookStore
{
    public Page? GetPage(int number)
    {
        Console.WriteLine(
            $"Seite {number} wird aus der Datenquelle geladen.");

        // Vereinfacht:
        // In einer echten Anwendung würde hier
        // beispielsweise EF Core verwendet.
        return new Page
        {
            Number = number,
            Text = $"Inhalt der Seite {number}"
        };
    }
}
```

---

# Proxy: BookStoreProxy

```
public class BookStoreProxy : IBookStore
{
    private readonly Dictionary<int, Page> _cache =
        new();

    private BookStore? _bookStore;

    public Page? GetPage(int number)
    {
        // Zuerst versuchen wir,
        // die Seite aus dem lokalen Cache zu lesen.
        if (_cache.TryGetValue(
                number,
                out Page? cachedPage))
        {
            Console.WriteLine(
                $"Seite {number} kommt aus dem Cache.");

            return cachedPage;
        }

        // Das reale Objekt wird erst erzeugt,
        // wenn tatsächlich ein Datenzugriff nötig ist.
        _bookStore ??=
            new BookStore();

        Page? page =
            _bookStore.GetPage(number);

        if (page is not null)
        {
            // Ergebnis für spätere Zugriffe speichern.
            _cache[number] = page;
        }

        return page;
    }
}
```

---

# Verwendung

```
IBookStore book =
    new BookStoreProxy();

Page? page1 =
    book.GetPage(1);

Page? page2 =
    book.GetPage(2);

Page? page1Again =
    book.GetPage(1);
```

Ausgabe ungefähr:

```
Seite 1 wird aus der Datenquelle geladen.
Seite 2 wird aus der Datenquelle geladen.
Seite 1 kommt aus dem Cache.
```

---

# Was passiert genau?

Beim ersten Aufruf:

```
book.GetPage(1);
```

prüft der Proxy:

```
Cache enthält Seite 1?
```

Antwort:

```
Nein
```

Also:

```
Proxy
  │
  ▼
BookStore
  │
  ▼
Datenquelle
```

Danach wird die Seite im Cache gespeichert.

---

# Zweiter Zugriff auf dieselbe Seite

```
book.GetPage(1);
```

Jetzt:

```
Cache enthält Seite 1?
```

Antwort:

```
Ja
```

Also:

```
Proxy
  │
  ▼
Cache
  │
  ▼
Page
```

Der `BookStore` muss gar nicht mehr aufgerufen werden.

---

# Kombination mehrerer Proxy-Aufgaben

Ein Proxy kann mehrere Aufgaben gleichzeitig übernehmen.

Zum Beispiel:

```
Client
  │
  ▼
ServiceProxy
  │
  ├── Berechtigung prüfen
  ├── Cache prüfen
  ├── Logging
  ├── Rate Limit prüfen
  └── RealService aufrufen
```

Das Pattern legt nicht fest, dass ein Proxy immer nur genau eine zusätzliche Aufgabe haben darf.

---

# Smart Reference

Eine sogenannte **Smart Reference** führt zusätzliche Aktionen aus, wenn auf ein Objekt zugegriffen wird.

Beispiele:

```
Referenzen zählen
Thread-Synchronisation
Locking
Ressourcenverwaltung
Logging
Objekt-Lebenszyklus verwalten
```

---

# Beispiel: Logging Proxy

```
public class LoggingProxy : IService
{
    private readonly IService _inner;

    public LoggingProxy(IService inner)
    {
        _inner = inner;
    }

    public void Execute()
    {
        Console.WriteLine(
            "Execute() wird gestartet.");

        _inner.Execute();

        Console.WriteLine(
            "Execute() wurde beendet.");
    }
}
```

Struktur:

```
Client
  │
  ▼
LoggingProxy
  │
  ▼
RealService
```

---

# Proxy und Lazy Loading

Eine sehr typische Proxy-Aufgabe ist **Lazy Loading**.

Statt:

```
Objekt erzeugen
   ↓
teure Daten sofort laden
```

wird:

```
Proxy erzeugen
   ↓
noch nichts laden
   ↓
Client benötigt Daten
   ↓
jetzt RealSubject erzeugen / laden
```

Das kann Ressourcen sparen.

---

# Aber Vorsicht bei Lazy Loading

Lazy Loading kann auch Nachteile haben.

Zum Beispiel:

```
customer.Orders
```

sieht wie ein einfacher Property-Zugriff aus.

Im Hintergrund könnte aber:

```
SQL Query
Netzwerkzugriff
große Datenmenge
```

ausgelöst werden.

Dadurch werden Kosten eines Aufrufs weniger offensichtlich.

> [!warning]  
> Ein Proxy kann technische Komplexität verbergen – manchmal so gut, dass Entwickler die tatsächlichen Kosten eines Aufrufs unterschätzen.

---

# Proxy vs. Decorator

Diese beiden Patterns sehen strukturell sehr ähnlich aus.

Beide:

- implementieren dieselbe Schnittstelle wie das innere Objekt;
- enthalten eine Referenz auf dieses Objekt;
- leiten Methodenaufrufe weiter.

Beispiel:

```
IService
├── RealService
└── Wrapper
      │
      ▼
   IService
```

Der Unterschied liegt hauptsächlich in der **Absicht**.

---

# Decorator

Decorator möchte:

> **Funktionalität eines Objekts erweitern.**

Beispiel:

```
Service
  ↓
LoggingDecorator
  ↓
CachingDecorator
```

Der Schwerpunkt liegt auf zusätzlichen Fähigkeiten.

---

# Proxy

Proxy möchte:

> **Zugriff auf ein Objekt kontrollieren.**

Zum Beispiel:

```
Lazy Loading
Security
Remote Access
Caching
Access Control
```

---

# Merksatz

```
Decorator
→ Verhalten erweitern


Proxy
→ Zugriff kontrollieren
```

---

# Proxy vs. Adapter

## Adapter

Adapter verändert die Schnittstelle:

```
Client erwartet A
      │
      ▼
Adapter
      │
      ▼
Objekt bietet B
```

---

## Proxy

Proxy behält normalerweise dieselbe Schnittstelle:

```
ISubject
├── Proxy
└── RealSubject
```

Ziel ist nicht die Interface-Übersetzung, sondern die Kontrolle des Zugriffs.

---

# Kurzvergleich

```
Adapter
→ andere Schnittstelle


Proxy
→ gleiche Schnittstelle,
  kontrollierter Zugriff
```

---

# Proxy vs. Facade

## Facade

Facade bietet:

```
eine vereinfachte Schnittstelle
zu mehreren Komponenten
```

```
Client
  │
  ▼
Facade
├── A
├── B
└── C
```

---

## Proxy

Proxy steht normalerweise vor einem bestimmten Objekt oder Service:

```
Client
  │
  ▼
Proxy
  │
  ▼
RealSubject
```

Kurz:

```
Facade
→ System vereinfachen

Proxy
→ Objektzugriff kontrollieren
```

---

# Proxy vs. Chain of Responsibility

## Chain of Responsibility

```
Request
   ↓
HandlerA
   ↓
HandlerB
   ↓
HandlerC
```

Eine Anfrage wird durch mehrere mögliche Handler weitergegeben.

---

## Proxy

```
Request
   ↓
Proxy
   ↓
RealSubject
```

Der Proxy besitzt normalerweise einen klaren realen Empfänger.

---

# Proxy vs. Mediator

## Mediator

koordiniert mehrere Objekte:

```
A ─┐
B ─┼──► Mediator
C ─┘
```

## Proxy

repräsentiert beziehungsweise kontrolliert ein einzelnes Objekt:

```
Client
  │
  ▼
Proxy
  │
  ▼
RealObject
```

---

# Proxy vs. Facade vs. Adapter vs. Decorator

Sehr wichtig:

```
Proxy
→ kontrolliert Zugriff


Decorator
→ erweitert Verhalten


Adapter
→ übersetzt Schnittstelle


Facade
→ vereinfacht Subsystem
```

Alle vier können äußerlich wie eine zusätzliche Klasse zwischen Client und eigentlichem Objekt aussehen.

Die **Absicht** unterscheidet die Patterns.

---

# Proxy und Dependency Injection

Ein Proxy kann sehr elegant zusammen mit Dependency Injection verwendet werden.

Angenommen:

```
public interface IOrderService
{
    Task<Order?> GetOrderAsync(int id);
}
```

Basisimplementierung:

```
public class OrderService : IOrderService
{
    public Task<Order?> GetOrderAsync(int id)
    {
        // Daten laden ...
        return Task.FromResult<Order?>(null);
    }
}
```

Caching Proxy:

```
public class CachedOrderService
    : IOrderService
{
    private readonly IOrderService _inner;

    private readonly Dictionary<int, Order> _cache =
        new();

    public CachedOrderService(
        IOrderService inner)
    {
        _inner = inner;
    }

    public async Task<Order?> GetOrderAsync(int id)
    {
        if (_cache.TryGetValue(
                id,
                out Order? cachedOrder))
        {
            return cachedOrder;
        }

        Order? order =
            await _inner.GetOrderAsync(id);

        if (order is not null)
        {
            _cache[id] = order;
        }

        return order;
    }
}
```

Der Client kennt trotzdem nur:

```
IOrderService
```

---

# Proxy in Clean Architecture

Proxy kann gut eingesetzt werden, wenn die Application-Schicht nur eine Abstraktion kennt:

```
Application
     │
     ▼
IProductRepository
     ▲
     │
CachingRepositoryProxy
     │
     ▼
EfProductRepository
```

Die Businesslogik muss nicht wissen, dass ein Cache existiert.

---

# Beispiel

```
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
}
```

RealSubject:

```
EfProductRepository
```

Proxy:

```
CachingProductRepository
```

Client:

```
Application Service
```

Die Struktur:

```
Application Service
        │
        ▼
IProductRepository
        ▲
        │
CachingProductRepository
        │
        ▼
EfProductRepository
        │
        ▼
Database
```

---

# Proxy und EF Core

Die ursprüngliche Quelle erwähnt Entity Framework im Zusammenhang mit Proxy.

Wichtig ist allerdings:

> [!note]  
> Das konkrete EF-Beispiel aus der alten Quelle implementiert selbst einen Caching-Proxy für ein Repository.
> 
> Das bedeutet nicht, dass jeder Datenbankzugriff in EF automatisch das klassische Proxy Pattern verwendet.

Proxy-artiges Verhalten findet man allerdings beispielsweise bei Mechanismen wie:

```
Lazy Loading Proxies
```

wo ein Stellvertreter beziehungsweise dynamisch erzeugter Typ das Nachladen von Beziehungen ermöglichen kann.

Für das Verständnis des Patterns ist aber das allgemeine Prinzip wichtiger:

```
Client
→ Proxy
→ RealSubject
```

---

# Caching: wichtiger Hinweis

Ein Cache klingt zunächst immer positiv:

```
weniger Datenbankzugriffe
=
schneller
```

aber er bringt neue Probleme mit sich.

Zum Beispiel:

```
Wie lange ist ein Eintrag gültig?

Wann wird Cache invalidiert?

Was passiert bei Änderungen?

Wie viel Speicher darf benutzt werden?

Ist der Cache thread-safe?
```

Deshalb sollte ein Caching Proxy nicht nur:

```
Dictionary<int, Page>
```

verwenden, wenn es sich um ein echtes produktives System handelt.

Das Beispiel dient hauptsächlich zum Verständnis des Patterns.

---

# Fehler im alten Beispiel

Im ursprünglichen Code passiert ungefähr:

```
page =
    bookStore.GetPage(number);

pages.Add(page);
```

Falls:

```
GetPage(number)
```

kein Ergebnis findet und `null` zurückliefert, würde anschließend versucht, `null` in die Sammlung aufzunehmen beziehungsweise später damit zu arbeiten.

Eine modernere Variante prüft daher:

```
if (page is not null)
{
    _cache[number] = page;
}
```

---

# `FirstOrDefault` und Nullable Reference Types

Bei modernem C# sollte berücksichtigt werden, dass:

```
FirstOrDefault(...)
```

kein Element finden kann.

Deshalb ist:

```
Page?
```

korrekter als blind:

```
Page
```

zu erwarten.

Zum Beispiel:

```
public Page? GetPage(int number)
```

---

# Vorteil: Transparent für den Client

Ein besonders wichtiger Aspekt von Proxy:

Der Client kann schreiben:

```
IBookStore store =
    new BookStoreProxy();
```

und danach einfach:

```
store.GetPage(1);
```

Für ihn sieht alles wie ein normales `IBookStore` aus.

Der Proxy kann intern entscheiden:

```
Cache?
Security?
Remote Server?
Lazy Loading?
RealSubject?
```

---

# Vorteile

- Zugriff auf das reale Objekt kann kontrolliert werden;
- teure Objekte können lazy erzeugt werden;
- Netzwerkzugriffe können abstrahiert werden;
- Berechtigungsprüfungen können zentralisiert werden;
- Caching kann transparent ergänzt werden;
- Client und RealSubject bleiben schwächer gekoppelt;
- zusätzliche technische Logik kann außerhalb des RealSubject bleiben;
- Proxy kann das gleiche Interface wie das RealSubject anbieten.

---

# Nachteile

- zusätzliche Indirektion erhöht die Komplexität;
- Aufrufe können langsamer werden;
- versteckte Netzwerk- oder Datenbankzugriffe können schwer erkennbar sein;
- Cache-Invalidierung kann komplex werden;
- Debugging wird schwieriger;
- sehr viele Proxy-Schichten können schwer nachvollziehbar sein;
- Fehler können erst zur Laufzeit auftreten, wenn das RealSubject tatsächlich aufgerufen wird.

---

# Warum kann Proxy langsamer sein?

Ohne Proxy:

```
Client
  │
  ▼
RealSubject
```

Mit Proxy:

```
Client
  │
  ▼
Proxy
  │
  ├── Cache prüfen
  ├── Berechtigung prüfen
  ├── Logging
  └── RealSubject
```

Wenn der Proxy keinen schnellen Weg findet, wird zunächst seine eigene Logik und danach trotzdem noch die eigentliche Operation ausgeführt.

Ein Proxy lohnt sich also nur, wenn seine zusätzliche Funktion einen echten Nutzen bietet.

---

# Wann Proxy nicht sinnvoll ist

Wenn:

```
RealSubject ist billig
+
kein Zugriffsschutz nötig
+
kein Remote-Zugriff
+
kein Lazy Loading
+
kein Caching nötig
```

dann bringt:

```
Client
→ Proxy
→ RealSubject
```

nur unnötige Komplexität.

Dann kann der Client direkt:

```
Client
→ RealSubject
```

verwenden.

---

# Typische reale Einsatzbereiche

Proxy eignet sich beispielsweise für:

```
Lazy Loading

Caching

Remote Services

Security

Authorization

Logging

Monitoring

Rate Limiting

Thread-Synchronisation

Ressourcenverwaltung

Virtualisierung teurer Objekte
```

---

# Beispiel: API Rate Limit Proxy

```
public interface IExternalApi
{
    Task<string> GetDataAsync();
}
```

RealSubject:

```
public class ExternalApi : IExternalApi
{
    public Task<string> GetDataAsync()
    {
        return Task.FromResult(
            "Daten vom externen Dienst");
    }
}
```

Proxy:

```
public class RateLimitedApiProxy
    : IExternalApi
{
    private readonly IExternalApi _inner;

    private DateTime _lastRequest =
        DateTime.MinValue;

    public RateLimitedApiProxy(
        IExternalApi inner)
    {
        _inner = inner;
    }

    public async Task<string> GetDataAsync()
    {
        TimeSpan elapsed =
            DateTime.UtcNow - _lastRequest;

        if (elapsed < TimeSpan.FromSeconds(1))
        {
            throw new InvalidOperationException(
                "Zu viele Anfragen.");
        }

        _lastRequest =
            DateTime.UtcNow;

        return await _inner.GetDataAsync();
    }
}
```

Der Proxy kontrolliert hier den Zugriff auf die externe Ressource.

---

# Beispiel: Kombination mit Logging

```
public class LoggingApiProxy
    : IExternalApi
{
    private readonly IExternalApi _inner;

    public LoggingApiProxy(
        IExternalApi inner)
    {
        _inner = inner;
    }

    public async Task<string> GetDataAsync()
    {
        Console.WriteLine(
            "API-Aufruf startet.");

        string result =
            await _inner.GetDataAsync();

        Console.WriteLine(
            "API-Aufruf beendet.");

        return result;
    }
}
```

Man könnte dann sogar kombinieren:

```
IExternalApi api =
    new ExternalApi();

api =
    new RateLimitedApiProxy(api);

api =
    new LoggingApiProxy(api);
```

Struktur:

```
LoggingProxy
    │
    ▼
RateLimitProxy
    │
    ▼
ExternalApi
```

Strukturell ähnelt das Decorator sehr stark.

Die Absicht lautet hier aber:

```
Zugriff kontrollieren
```

und nicht primär:

```
neue fachliche Fähigkeiten hinzufügen
```

---

# Das solltest du dir merken

Gemeinsame Abstraktion:

```
public interface ISubject
{
    void Request();
}
```

RealSubject:

```
public class RealSubject : ISubject
{
    public void Request()
    {
        // Eigentliche Funktionalität.
    }
}
```

Proxy:

```
public class Proxy : ISubject
{
    private RealSubject? _realSubject;

    public void Request()
    {
        // Zugriff kontrollieren oder vorbereiten.
        _realSubject ??=
            new RealSubject();

        // Anfrage weiterleiten.
        _realSubject.Request();
    }
}
```

Client:

```
ISubject subject =
    new Proxy();

subject.Request();
```

---

# Der entscheidende Code

```
public void Request()
{
    // Vor dem RealSubject kann zusätzliche
    // Zugriffskontrolle stattfinden.

    _realSubject ??=
        new RealSubject();

    _realSubject.Request();
}
```

Das ist die Essenz:

```
Client Request
      │
      ▼
    Proxy
      │
      ├── kontrollieren
      ├── vorbereiten
      ├── cachen
      ├── laden
      └── weiterleiten
             │
             ▼
        RealSubject
```

---

# Merksatz

> **Proxy stellt einen Stellvertreter für ein anderes Objekt bereit und kontrolliert den Zugriff auf dieses reale Objekt.**

Noch einfacher:

```
Client:
"Ich möchte das Objekt verwenden."

Proxy:
"Ich prüfe zuerst,
ob und wie du darauf zugreifen darfst."

RealSubject:
"Ich führe die eigentliche Arbeit aus."
```

Oder ganz kurz:

```
Proxy
=
gleiche Schnittstelle
+
Stellvertreter
+
Zugriffskontrolle
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Proxy
→ Zugriff kontrollieren


Decorator
→ Verhalten erweitern


Adapter
→ Schnittstelle anpassen


Facade
→ komplexes Subsystem vereinfachen


Composite
→ Baumstruktur erzeugen


Bridge
→ Abstraktion und Implementierung trennen
```

---

> [!summary] Zusammenfassung  
> Das **Proxy Pattern** ist ein **Strukturmuster**.
> 
> Es stellt einen Stellvertreter vor ein reales Objekt:
> 
> ```
> Client
>    │
>    ▼
> Proxy
>    │
>    ▼
> RealSubject
> ```
> 
> Proxy und RealSubject besitzen normalerweise dieselbe Schnittstelle:
> 
> ```
> ISubject
> ├── Proxy
> └── RealSubject
> ```
> 
> Dadurch kann der Client den Proxy wie das reale Objekt verwenden:
> 
> ```
> ISubject subject =
>     new Proxy();
> 
> subject.Request();
> ```
> 
> Der Proxy kann vor dem Weiterleiten zusätzliche Aufgaben übernehmen:
> 
> ```
> Lazy Loading
> Caching
> Berechtigungsprüfung
> Logging
> Remote Access
> Rate Limiting
> Synchronisation
> ```
> 
> Besonders wichtig ist der Unterschied zu Decorator:
> 
> ```
> Decorator
> → Verhalten erweitern
> 
> Proxy
> → Zugriff kontrollieren
> ```
> 
> **Kurz gesagt:**  
> `Proxy = Stellvertreter mit derselben Schnittstelle, der den Zugriff auf das reale Objekt kontrolliert.`