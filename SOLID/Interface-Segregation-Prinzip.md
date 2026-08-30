Das **Interface Segregation Principle (ISP)** beschäftigt sich mit sogenannten **„fetten Interfaces“** – also Interfaces, die zu viele Methoden und Eigenschaften enthalten.

Dadurch müssen Klassen häufig Mitglieder implementieren, die sie eigentlich gar nicht benötigen.

Das Prinzip lässt sich so formulieren:

> [!IMPORTANT]  
> Clients sollten nicht gezwungen sein, von Methoden abhängig zu sein, die sie nicht verwenden.

Wenn dieses Prinzip verletzt wird, hängt ein Client von einem Interface mit Mitgliedern ab, die für ihn irrelevant sind.

Dadurch entstehen unnötige Abhängigkeiten zwischen verschiedenen Teilen eines Interfaces, obwohl diese fachlich möglicherweise gar nicht zusammengehören.

---

## Typische Hinweise auf eine ISP-Verletzung

Mögliche Warnsignale sind:

- sehr große Interfaces,
    
- schwach zusammenhängende Mitglieder innerhalb eines Interfaces,
    
- Methoden ohne sinnvolle Implementierung,
    
- viele `NotImplementedException`,
    
- leere Methoden,
    
- Klassen implementieren nur einen kleinen Teil des Interfaces sinnvoll.
    

> [!NOTE]  
> Das Interface Segregation Principle steht in engem Zusammenhang mit:
> 
> - **SRP** — Single Responsibility Principle
>     
> - **LSP** — Liskov Substitution Principle
>     

Die Lösung besteht darin, ein großes Interface in mehrere kleinere, fachlich zusammengehörige Interfaces aufzuteilen.

Diese kleineren Interfaces können anschließend unabhängig voneinander verwendet und verändert werden.

Dadurch wird das System **lockerer gekoppelt**, verständlicher und leichter wartbar.

---

# Beispiel: Nachrichten versenden

Angenommen, wir haben ein Interface zum Versenden von Nachrichten:

```csharp
interface IMessage
{
    void Send();

    string Text { get; set; }
    string Subject { get; set; }
    string ToAddress { get; set; }
    string FromAddress { get; set; }
}
```

Das Interface enthält alles, was für eine E-Mail sinnvoll erscheint:

- Nachrichtentext,
    
- Betreff,
    
- Absender,
    
- Empfänger,
    
- Versandmethode.
    

Eine mögliche Implementierung:

```csharp
class EmailMessage : IMessage
{
    public string Subject { get; set; } = "";
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";

    public void Send() =>
        Console.WriteLine($"E-Mail wird gesendet: {Text}");
}
```

Die Klasse `EmailMessage` wirkt in sich stimmig.

Alle Mitglieder des Interfaces werden tatsächlich benötigt.

---

# Problem mit SMS

Jetzt möchten wir auch SMS-Nachrichten unterstützen:

```csharp
class SmsMessage : IMessage
{
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";

    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }

        set
        {
            throw new NotImplementedException();
        }
    }

    public void Send() =>
        Console.WriteLine($"SMS wird gesendet: {Text}");
}
```

Hier entsteht ein Problem:

Eine SMS besitzt normalerweise keinen Betreff.

Die Property `Subject` ist für `SmsMessage` also unnötig.

Trotzdem ist die Klasse gezwungen, sie zu implementieren.

> [!WARNING]  
> `SmsMessage` hängt damit von Funktionalität ab, die es nicht benötigt.

---

# Noch deutlicher: VoiceMessage

Angenommen, wir möchten zusätzlich Sprachnachrichten unterstützen.

Eine Sprachnachricht enthält:

- Absender,
    
- Empfänger,
    
- Audiodaten.
    

Die Audiodaten könnten zum Beispiel als `byte[]` dargestellt werden.

Man könnte deshalb das bestehende Interface erweitern:

```csharp
interface IMessage
{
    void Send();

    string Text { get; set; }
    string ToAddress { get; set; }
    string Subject { get; set; }
    string FromAddress { get; set; }

    byte[] Voice { get; set; }
}
```

Jetzt könnte eine Voice-Nachricht so aussehen:

```csharp
class VoiceMessage : IMessage
{
    public string ToAddress { get; set; } = "";
    public string FromAddress { get; set; } = "";

    public byte[] Voice { get; set; } = Array.Empty<byte>();

    public string Text
    {
        get
        {
            throw new NotImplementedException();
        }

        set
        {
            throw new NotImplementedException();
        }
    }

    public string Subject
    {
        get
        {
            throw new NotImplementedException();
        }

        set
        {
            throw new NotImplementedException();
        }
    }

    public void Send() =>
        Console.WriteLine("Sprachnachricht wird übertragen");
}
```

Jetzt besitzt `VoiceMessage` gleich mehrere unnötige Properties:

```
Text
Subject
```

Zusätzlich müssten auch `EmailMessage` und `SmsMessage` plötzlich die neue Property `Voice` implementieren, obwohl sie sie überhaupt nicht benötigen.

Damit ist die Verletzung des Interface Segregation Principle deutlich sichtbar.

---

# Lösung: Interface aufteilen

Wir teilen das große Interface in mehrere kleinere Interfaces auf.

```csharp
interface IMessage
{
    void Send();

    string ToAddress { get; set; }
    string FromAddress { get; set; }
}

interface IVoiceMessage : IMessage
{
    byte[] Voice { get; set; }
}

interface ITextMessage : IMessage
{
    string Text { get; set; }
}

interface IEmailMessage : ITextMessage
{
    string Subject { get; set; }
}
```

Jetzt können die Klassen genau die Interfaces implementieren, die sie wirklich benötigen.

---

## VoiceMessage

```csharp
class VoiceMessage : IVoiceMessage
{
    public string ToAddress { get; set; } = "";
    public string FromAddress { get; set; } = "";

    public byte[] Voice { get; set; } = Array.Empty<byte>();

    public void Send() =>
        Console.WriteLine("Sprachnachricht wird übertragen");
}
```

---

## EmailMessage

```csharp
class EmailMessage : IEmailMessage
{
    public string Text { get; set; } = "";
    public string Subject { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";

    public void Send() =>
        Console.WriteLine($"E-Mail wird gesendet: {Text}");
}
```

---

## SmsMessage

```csharp
class SmsMessage : ITextMessage
{
    public string Text { get; set; } = "";
    public string FromAddress { get; set; } = "";
    public string ToAddress { get; set; } = "";

    public void Send() =>
        Console.WriteLine($"SMS wird gesendet: {Text}");
}
```

Jetzt enthält keine Klasse mehr unnötige Mitglieder.

Durch die Vererbung der Interfaces wird außerdem unnötige Wiederholung vermieden.

Die Struktur ist:

```
           IMessage
          /        \
         /          \
        ▼            ▼
ITextMessage    IVoiceMessage
      │
      ▼
IEmailMessage
```

Die Klassen implementieren nur das, was sie wirklich benötigen:

```
EmailMessage  → IEmailMessage
SmsMessage    → ITextMessage
VoiceMessage  → IVoiceMessage
```

---

# Leere Methoden als Warnsignal

Ein typischer Hinweis auf eine Verletzung des Interface Segregation Principle sind **nicht implementierte oder leere Methoden**.

Solche Methoden können außerdem auf eine mögliche Verletzung des **Liskov Substitution Principle** hinweisen.

Beispiel:

```csharp
interface IPhone
{
    void Call();
    void TakePhoto();
    void MakeVideo();
    void BrowseInternet();
}
```

Eine Smartphone-Klasse könnte alle Funktionen sinnvoll implementieren:

```csharp
class Phone : IPhone
{
    public void Call() =>
        Console.WriteLine("Anruf wird gestartet");

    public void TakePhoto() =>
        Console.WriteLine("Foto wird aufgenommen");

    public void MakeVideo() =>
        Console.WriteLine("Video wird aufgenommen");

    public void BrowseInternet() =>
        Console.WriteLine("Im Internet surfen");
}
```

Das Interface passt gut zu einem Smartphone.

---

# Client: Photograph

Angenommen, wir haben eine Klasse, die ein Gerät nur zum Fotografieren verwendet:

```csharp
class Photograph
{
    public void TakePhoto(Phone phone)
    {
        phone.TakePhoto();
    }
}
```

Verwendung:

```csharp
Photograph photograph = new Photograph();

Phone myPhone = new Phone();

photograph.TakePhoto(myPhone);
```

Das funktioniert.

Aber ein Fotograf könnte natürlich auch eine normale Kamera verwenden.

---

# Problem mit Camera

Wenn wir `Camera` ebenfalls `IPhone` implementieren lassen:

```csharp
class Camera : IPhone
{
    public void Call()
    {
    }

    public void TakePhoto()
    {
        Console.WriteLine("Foto wird aufgenommen");
    }

    public void MakeVideo()
    {
    }

    public void BrowseInternet()
    {
    }
}
```

müssen wir mehrere Methoden leer lassen:

```csharp
Call()
MakeVideo()
BrowseInternet()
```

Diese Methoden gehören fachlich nicht zur Kamera.

> [!WARNING]  
> `Camera` wird dadurch gezwungen, von Funktionen abhängig zu sein, die sie gar nicht benötigt.

Das Interface `IPhone` ist für diesen Anwendungsfall zu groß.

---

# Lösung durch kleinere Interfaces

Wir teilen die Fähigkeiten in einzelne Interfaces auf:

```csharp
interface ICall
{
    void Call();
}

interface IPhoto
{
    void TakePhoto();
}

interface IVideo
{
    void MakeVideo();
}

interface IWeb
{
    void BrowseInternet();
}
```

Jetzt kann jede Klasse nur die Fähigkeiten implementieren, die sie wirklich besitzt.

---

## Camera

```csharp
class Camera : IPhoto
{
    public void TakePhoto()
    {
        Console.WriteLine(
            "Foto wird mit der Kamera aufgenommen");
    }
}
```

---

## Phone

```csharp
class Phone : ICall, IPhoto, IVideo, IWeb
{
    public void Call()
    {
        Console.WriteLine("Anruf wird gestartet");
    }

    public void TakePhoto()
    {
        Console.WriteLine(
            "Foto wird mit dem Smartphone aufgenommen");
    }

    public void MakeVideo()
    {
        Console.WriteLine("Video wird aufgenommen");
    }

    public void BrowseInternet()
    {
        Console.WriteLine("Im Internet surfen");
    }
}
```

Die Klassenstruktur sieht jetzt so aus:

```
      ICall      IPhoto      IVideo      IWeb
        ▲          ▲           ▲          ▲
        │          │           │          │
        └──────────┴──── Phone ┴──────────┘
                   ▲
                   │
                 Camera
              (nur IPhoto)
```

`Phone` implementiert alle benötigten Fähigkeiten.

`Camera` implementiert ausschließlich `IPhoto`.

---

# Client verbessern

Jetzt ändern wir die Klasse `Photograph`.

Statt einen konkreten `Phone` zu verlangen:

```csharp
class Photograph
{
    public void TakePhoto(Phone phone)
    {
        phone.TakePhoto();
    }
}
```

verwenden wir nur die tatsächlich benötigte Abstraktion:

```csharp
class Photograph
{
    public void TakePhoto(IPhoto photoMaker)
    {
        photoMaker.TakePhoto();
    }
}
```

Jetzt kann die Methode sowohl mit einem Smartphone als auch mit einer Kamera arbeiten:

```csharp
Photograph photograph = new Photograph();

IPhoto phone = new Phone();
IPhoto camera = new Camera();

photograph.TakePhoto(phone);
photograph.TakePhoto(camera);
```

Der Client benötigt nur:

```csharp
TakePhoto()
```

und ist deshalb ausschließlich von `IPhoto` abhängig.

Er muss nichts wissen über:

```csharp
Call()
MakeVideo()
BrowseInternet()
```

---

# Warum ist ISP wichtig?

Ohne Interface Segregation:

```
              IPhone
        ┌──────┼──────┐
        │      │      │
      Call   Photo   Video   Web
        ▲
        │
      Camera
```

`Camera` müsste Funktionen implementieren, die sie gar nicht besitzt.

Mit ISP:

```
ICall   IPhoto   IVideo   IWeb
          ▲
          │
        Camera
```

Die Klasse implementiert nur das relevante Interface.

---

# Zusammenhang mit SRP

Das Interface Segregation Principle ähnelt dem Single Responsibility Principle.

**SRP fragt:**

> Hat diese Klasse zu viele Verantwortlichkeiten?

**ISP fragt:**

> Enthält dieses Interface zu viele unabhängige Fähigkeiten?

Beide Prinzipien versuchen, Komponenten klein, fokussiert und verständlich zu halten.

---

# Zusammenhang mit LSP

Eine Klasse, die gezwungen wird, Methoden so zu implementieren:

```csharp
public void Call()
{
    throw new NotImplementedException();
}
```

ist häufig ein Hinweis darauf, dass der Typ das Interface nicht vollständig erfüllen kann.

Damit kann zusätzlich das Liskov Substitution Principle verletzt werden.

---

# Typische schlechte Anzeichen

Achte auf solche Stellen:

```
throw new NotImplementedException();
```

oder:

```csharp
public void SomeMethod()
{
}
```

oder Kommentare wie:

```
// Wird für diese Klasse nicht benötigt
```

Solche Stellen können darauf hindeuten, dass das Interface zu groß ist.

---

# Kurzfassung

> [!SUMMARY]  
> **ISP = Eine Klasse soll nur von den Methoden abhängig sein, die sie wirklich benötigt.**

Statt eines großen Interfaces:

```
IPhone
 ├── Call()
 ├── TakePhoto()
 ├── MakeVideo()
 └── BrowseInternet()
```

besser mehrere kleine Interfaces:

```
ICall
 └── Call()

IPhoto
 └── TakePhoto()

IVideo
 └── MakeVideo()

IWeb
 └── BrowseInternet()
```

Dann kann jede Klasse nur die passenden Fähigkeiten auswählen:

```
Phone
 ├── ICall
 ├── IPhoto
 ├── IVideo
 └── IWeb

Camera
 └── IPhoto
```

### Merksatz

> **Viele kleine, gezielte Interfaces sind meistens besser als ein einziges großes Interface.**

Oder noch einfacher:

> **Eine Klasse soll nichts implementieren müssen, was sie nicht braucht.**