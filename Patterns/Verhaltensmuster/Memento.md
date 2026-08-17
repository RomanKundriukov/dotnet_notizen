Das **Memento Pattern** ist ein Verhaltensmuster, das den internen Zustand eines Objekts speichert, damit dieser Zustand später wiederhergestellt werden kann, **ohne die Kapselung des Objekts zu verletzen**.

> [!info] Grundidee  
> Ein Objekt kann einen **Snapshot seines aktuellen Zustands** erzeugen.
> 
> Dieser Snapshot wird außerhalb des Objekts gespeichert und kann später verwendet werden, um den alten Zustand wiederherzustellen.

Vereinfacht:

```text
Objekt
  │
  │ Save()
  ▼
Memento
  │
  │ später
  ▼
Restore()
  │
  ▼
Objekt befindet sich wieder
im alten Zustand
```

---

# Wann sollte man Memento verwenden?

Das Pattern eignet sich besonders:

- wenn der Zustand eines Objekts gespeichert werden muss;
    
- wenn ein früherer Zustand später wiederhergestellt werden soll;
    
- wenn die internen Daten des Objekts dabei weiterhin gekapselt bleiben sollen;
    
- wenn Undo-/Redo-Funktionalität benötigt wird;
    
- wenn Snapshots oder Checkpoints gespeichert werden sollen;
    
- wenn Spielstände oder Bearbeitungszustände gespeichert werden.
    

Typische Beispiele:

```text
Spielstand speichern

Undo / Redo

Texteditor

Grafikprogramm

Konfigurationsänderungen zurücksetzen

Workflow-Checkpoint

Transaktionsähnliches Rollback
```

---

# Das zentrale Problem

Angenommen, wir besitzen eine Klasse:

```csharp
public class Hero
{
    private int _ammo;
    private int _lives;
    private int _health;
}
```

Diese Felder sind absichtlich:

```csharp
private
```

Damit wird die Kapselung eingehalten.

Ein anderes Objekt sollte also nicht einfach schreiben:

```csharp
hero._ammo = 10;
hero._lives = 5;
```

Das wäre ohnehin nicht möglich.

Gleichzeitig möchten wir aber vielleicht den gesamten Zustand speichern und später wiederherstellen.

Genau dafür ist Memento gedacht.

---

# Teilnehmer des Patterns

Das Pattern besteht klassisch aus drei Hauptkomponenten:

```text
Originator
Memento
Caretaker
```

---

# Originator

Der **Originator** ist das Objekt, dessen Zustand gespeichert werden soll.

Er:

- besitzt den eigentlichen Zustand;
    
- erzeugt ein Memento;
    
- kann seinen Zustand aus einem Memento wiederherstellen.
    

Beispiel:

```text
Hero
```

---

# Memento

Das **Memento** enthält einen Snapshot des Zustands.

Zum Beispiel:

```text
Ammo = 9
Lives = 5
Health = 80
```

Das Memento soll den gespeicherten Zustand möglichst nicht beliebig verändern lassen.

---

# Caretaker

Der **Caretaker** speichert Mementos.

Er soll normalerweise nicht selbst die interne Bedeutung des gespeicherten Zustands verändern.

Beispiel:

```text
GameHistory
```

Der Caretaker weiß:

> „Ich speichere diesen Snapshot.“

Er muss aber nicht wissen:

> „Wie genau wird daraus der Zustand des Helden rekonstruiert?“

---

# Grundstruktur

```text
                 Originator
                    │
          ┌─────────┴─────────┐
          │                   │
     CreateMemento()     Restore(...)
          │                   ▲
          ▼                   │
        Memento ───────────────┘
          │
          │ speichern
          ▼
       Caretaker
```

---

# Allgemeines Beispiel

```csharp
// Enthält den gespeicherten Zustand.
public class Memento
{
    public string State { get; }

    public Memento(string state)
    {
        State = state;
    }
}
```

Der Originator:

```csharp
public class Originator
{
    public string State { get; set; } = string.Empty;

    // Erzeugt einen Snapshot
    // des aktuellen Zustands.
    public Memento CreateMemento()
    {
        return new Memento(State);
    }

    // Stellt einen früheren Zustand wieder her.
    public void Restore(Memento memento)
    {
        State = memento.State;
    }
}
```

Der Caretaker:

```csharp
public class Caretaker
{
    public Memento? Memento { get; set; }
}
```

---

# Verwendung

```csharp
Originator originator =
    new Originator();

originator.State = "Zustand A";

Caretaker caretaker =
    new Caretaker();

// Aktuellen Zustand speichern.
caretaker.Memento =
    originator.CreateMemento();

// Zustand verändern.
originator.State = "Zustand B";

// Alten Zustand wiederherstellen.
originator.Restore(
    caretaker.Memento);
```

Danach besitzt `originator` wieder:

```text
Zustand A
```

---

# Praktisches Beispiel: Spielstand

Angenommen, wir haben einen Helden.

Sein Zustand besteht aus:

```text
Munition
Leben
```

Wir möchten den Spielstand speichern und später wiederherstellen.

---

# Originator: Hero

```csharp
public class Hero
{
    // Interner Zustand bleibt gekapselt.
    private int _ammo = 10;

    private int _lives = 5;

    public void Shoot()
    {
        if (_ammo > 0)
        {
            _ammo--;

            Console.WriteLine(
                $"Schuss abgegeben. Verbleibende Munition: {_ammo}");
        }
        else
        {
            Console.WriteLine(
                "Keine Munition mehr vorhanden.");
        }
    }

    // Erstellt einen Snapshot
    // des aktuellen Zustands.
    public HeroMemento SaveState()
    {
        Console.WriteLine(
            $"Spiel wird gespeichert: {_ammo} Munition, {_lives} Leben.");

        return new HeroMemento(
            _ammo,
            _lives);
    }

    // Stellt einen früher gespeicherten
    // Zustand wieder her.
    public void RestoreState(
        HeroMemento memento)
    {
        _ammo = memento.Ammo;
        _lives = memento.Lives;

        Console.WriteLine(
            $"Spielstand wiederhergestellt: {_ammo} Munition, {_lives} Leben.");
    }
}
```

---

# Memento: HeroMemento

```csharp
// Repräsentiert einen gespeicherten
// Zustand des Hero-Objekts.
public sealed class HeroMemento
{
    public int Ammo { get; }

    public int Lives { get; }

    public HeroMemento(
        int ammo,
        int lives)
    {
        Ammo = ammo;
        Lives = lives;
    }
}
```

Die Eigenschaften besitzen nur:

```csharp
get;
```

Dadurch kann der gespeicherte Zustand nach der Erstellung nicht einfach verändert werden.

---

# Caretaker: GameHistory

```csharp
public class GameHistory
{
    // Speichert mehrere Spielstände.
    private readonly Stack<HeroMemento> _history =
        new();

    public void Save(HeroMemento memento)
    {
        _history.Push(memento);
    }

    public HeroMemento Restore()
    {
        if (_history.Count == 0)
        {
            throw new InvalidOperationException(
                "Es existiert kein gespeicherter Spielstand.");
        }

        return _history.Pop();
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
        Hero hero =
            new Hero();

        GameHistory history =
            new GameHistory();

        // 10 → 9
        hero.Shoot();

        // Zustand speichern.
        history.Save(
            hero.SaveState());

        // 9 → 8
        hero.Shoot();

        // Zustand mit 9 Patronen wiederherstellen.
        hero.RestoreState(
            history.Restore());

        // 9 → 8
        hero.Shoot();
    }
}
```

---

# Ausgabe

```text
Schuss abgegeben. Verbleibende Munition: 9

Spiel wird gespeichert: 9 Munition, 5 Leben.

Schuss abgegeben. Verbleibende Munition: 8

Spielstand wiederhergestellt: 9 Munition, 5 Leben.

Schuss abgegeben. Verbleibende Munition: 8
```

---

# Ablauf Schritt für Schritt

## 1. Ausgangszustand

```text
Hero

Ammo  = 10
Lives = 5
```

---

## 2. Held schießt

```csharp
hero.Shoot();
```

Danach:

```text
Ammo  = 9
Lives = 5
```

---

## 3. Zustand speichern

```csharp
HeroMemento memento =
    hero.SaveState();
```

Jetzt enthält das Memento:

```text
HeroMemento

Ammo  = 9
Lives = 5
```

---

# Darstellung

```text
Hero
│
├── Ammo  = 9
└── Lives = 5

        │
        │ SaveState()
        ▼

HeroMemento
├── Ammo  = 9
└── Lives = 5
```

---

# 4. Zustand verändert sich

Der Held schießt erneut:

```csharp
hero.Shoot();
```

Jetzt:

```text
Hero

Ammo  = 8
Lives = 5
```

Das Memento bleibt aber unverändert:

```text
HeroMemento

Ammo  = 9
Lives = 5
```

---

# 5. Zustand wiederherstellen

```csharp
hero.RestoreState(memento);
```

Danach:

```text
Hero

Ammo  = 9
Lives = 5
```

Der alte Zustand wurde wiederhergestellt.

---

# Warum verwenden wir `Stack<T>`?

Im Beispiel speichern wir mehrere Mementos in:

```csharp
Stack<HeroMemento>
```

Ein Stack arbeitet nach:

```text
LIFO
```

also:

**Last In – First Out**

Der zuletzt gespeicherte Zustand wird zuerst wiederhergestellt.

Das eignet sich hervorragend für:

```text
Undo
```

---

# Beispiel

Wir speichern:

```text
State 1
State 2
State 3
```

Im Stack:

```text
┌─────────┐
│ State 3 │ ← zuerst zurück
├─────────┤
│ State 2 │
├─────────┤
│ State 1 │
└─────────┘
```

Bei:

```csharp
history.Pop();
```

erhalten wir zuerst:

```text
State 3
```

---

# Memento und Undo

Ein typischer Einsatz ist eine Undo-Funktion.

Angenommen, ein Texteditor besitzt:

```text
Text = "Hallo"
```

Dann wird gespeichert:

```text
Snapshot 1:
"Hallo"
```

Der Benutzer schreibt weiter:

```text
Hallo Welt
```

Snapshot:

```text
Snapshot 2:
"Hallo Welt"
```

Danach:

```text
Hallo Welt!
```

Bei `Undo()`:

```text
Hallo Welt
```

Noch einmal `Undo()`:

```text
Hallo
```

---

# Beispiel: Texteditor

```csharp
public sealed class EditorMemento
{
    public string Text { get; }

    public EditorMemento(string text)
    {
        Text = text;
    }
}
```

Originator:

```csharp
public class TextEditor
{
    private string _text = string.Empty;

    public void SetText(string text)
    {
        _text = text;
    }

    public void Print()
    {
        Console.WriteLine(_text);
    }

    // Aktuellen Text speichern.
    public EditorMemento Save()
    {
        return new EditorMemento(_text);
    }

    // Alten Text wiederherstellen.
    public void Restore(
        EditorMemento memento)
    {
        _text = memento.Text;
    }
}
```

---

# History

```csharp
public class EditorHistory
{
    private readonly Stack<EditorMemento> _history =
        new();

    public void Push(
        EditorMemento memento)
    {
        _history.Push(memento);
    }

    public EditorMemento Pop()
    {
        return _history.Pop();
    }
}
```

---

# Verwendung

```csharp
TextEditor editor =
    new TextEditor();

EditorHistory history =
    new EditorHistory();

editor.SetText("Hallo");

history.Push(editor.Save());

editor.SetText("Hallo Welt");

history.Push(editor.Save());

editor.SetText("Hallo Welt!");

editor.Print();
```

Ausgabe:

```text
Hallo Welt!
```

Undo:

```csharp
editor.Restore(
    history.Pop());

editor.Print();
```

Ausgabe:

```text
Hallo Welt
```

Noch einmal:

```csharp
editor.Restore(
    history.Pop());

editor.Print();
```

Ausgabe:

```text
Hallo
```

---

# Memento und Kapselung

Das zentrale Ziel des Patterns ist nicht nur:

```text
Zustand speichern
```

sondern:

```text
Zustand speichern
+
Kapselung erhalten
```

Der Caretaker sollte nicht direkt den internen Zustand des Originators manipulieren.

Schlecht wäre beispielsweise:

```csharp
history.State.Ammo = 99999;
```

Dadurch würde der gespeicherte Snapshot von außen verändert.

Deshalb sollte ein Memento möglichst **unveränderlich** sein.

---

# Immutable Memento

Eine gute moderne Variante ist beispielsweise ein `record`.

```csharp
public sealed record HeroMemento(
    int Ammo,
    int Lives);
```

Dann kann `Hero` schreiben:

```csharp
public HeroMemento SaveState()
{
    return new HeroMemento(
        _ammo,
        _lives);
}
```

Das ist wesentlich kürzer und passt sehr gut zu Snapshot-Objekten.

---

# Moderne Variante mit `record`

```csharp
// Immutable Snapshot.
public sealed record HeroMemento(
    int Ammo,
    int Lives);
```

Hero:

```csharp
public class Hero
{
    private int _ammo = 10;

    private int _lives = 5;

    public void Shoot()
    {
        if (_ammo == 0)
        {
            Console.WriteLine(
                "Keine Munition mehr.");

            return;
        }

        _ammo--;

        Console.WriteLine(
            $"Munition: {_ammo}");
    }

    public HeroMemento Save()
    {
        return new HeroMemento(
            _ammo,
            _lives);
    }

    public void Restore(
        HeroMemento memento)
    {
        _ammo = memento.Ammo;
        _lives = memento.Lives;
    }
}
```

---

# Warum eignet sich `record` gut?

Ein Memento ist im Grunde:

```text
eine Momentaufnahme von Daten
```

Es besitzt normalerweise:

- keine komplexe Geschäftslogik;
    
- unveränderliche Werte;
    
- einen klar definierten Zustand.
    

Deshalb passt ein:

```csharp
record
```

oft sehr gut.

---

# Ein Memento ist kein Backup-System

Memento bedeutet nicht automatisch:

```text
Datenbank-Backup
Datei-Backup
Cloud-Backup
```

Das Pattern beschreibt zunächst die **Objektstruktur und Verantwortung** für Snapshots.

Wo diese Mementos gespeichert werden:

```text
RAM
Stack
Datei
Datenbank
Redis
Cloud
```

ist eine separate technische Entscheidung.

---

# Mehrere Zustände speichern

Mit einem Stack:

```csharp
private readonly Stack<HeroMemento> _history = new();
```

können wir mehrere Snapshots speichern.

Zum Beispiel:

```text
Spielstand 1
Ammo = 10
Lives = 5

Spielstand 2
Ammo = 7
Lives = 4

Spielstand 3
Ammo = 3
Lives = 2
```

Dann können wir schrittweise zurückgehen.

---

# Undo und Redo

Für ein vollständiges Undo/Redo-System verwendet man häufig zwei Stacks:

```text
Undo Stack
Redo Stack
```

Beispiel:

```text
               Undo
Current ─────────────► alter Zustand

               Redo
alter Zustand ───────► neuer Zustand
```

---

# Beispielstruktur

```csharp
public class History<T>
{
    private readonly Stack<T> _undo =
        new();

    private readonly Stack<T> _redo =
        new();
}
```

Das Memento Pattern lässt sich also gut als Grundlage für Undo/Redo verwenden.

---

# Originator sollte seinen Zustand selbst kennen

Ein wichtiger Punkt:

Der Caretaker sollte normalerweise nicht wissen müssen:

```text
Hero besitzt Ammo
Hero besitzt Lives
Hero besitzt Health
Hero besitzt Weapon
```

Diese Details gehören zum `Hero`.

Der Hero selbst weiß, was für einen vollständigen Snapshot benötigt wird.

Deshalb:

```csharp
hero.Save();
```

statt:

```csharp
new HeroMemento(
    hero.Ammo,
    hero.Lives,
    hero.Health,
    hero.Weapon,
    ...);
```

außerhalb der Klasse.

---

# Falsch

```csharp
GameHistory history =
    new GameHistory();

history.Save(
    new HeroMemento(
        hero.Ammo,
        hero.Lives));
```

Dann müsste der Caretaker die interne Struktur des Hero kennen.

---

# Besser

```csharp
history.Save(
    hero.Save());
```

Der `Hero` entscheidet selbst, welche Informationen sein Memento benötigt.

---

# Memento vs. Prototype

Beide Patterns können Kopien von Zuständen erzeugen, aber ihre Absicht ist unterschiedlich.

## Prototype

```text
bestehendes Objekt
      │
      ▼
    Clone()
      │
      ▼
neues gleichartiges Objekt
```

Ziel:

> Ein neues Objekt auf Grundlage eines bestehenden Objekts erzeugen.

---

## Memento

```text
bestehendes Objekt
      │
      ▼
     Save()
      │
      ▼
   Snapshot
      │
      ▼
   Restore()
```

Ziel:

> Den Zustand eines bestehenden Objekts später wiederherstellen.

---

# Prototype vs. Memento

```text
Prototype
→ Objekt kopieren

Memento
→ Zustand speichern
  und später wiederherstellen
```

---

# Memento vs. Command

Diese beiden Patterns werden häufig gemeinsam für Undo verwendet.

## Memento

Speichert:

```text
Wie sah das Objekt vorher aus?
```

Beispiel:

```text
Text = "Hallo"
```

---

## Command

Speichert eher:

```text
Welche Aktion wurde ausgeführt?
```

Beispiel:

```text
InsertTextCommand(" Welt")
```

Ein Command kann dann möglicherweise eine Gegenoperation ausführen.

---

# Unterschied

```text
Memento
→ Zustand speichern

Command
→ Aktion kapseln
```

Für Undo-Systeme können beide kombiniert werden.

---

# Memento vs. State

Auch diese Namen können verwirrend sein.

## State Pattern

State bedeutet:

> Aktuelles Verhalten eines Objekts hängt von seinem Zustand ab.

```text
Order
→ NewState
→ PaidState
→ ShippedState
```

---

## Memento Pattern

Memento bedeutet:

> Einen früheren Zustand speichern und wiederherstellen.

```text
State A
  │
  │ speichern
  ▼
Memento
  │
  │ später
  ▼
State A wiederherstellen
```

---

# Vorteile

- früherer Zustand kann wiederhergestellt werden;
    
- Kapselung des Originators bleibt erhalten;
    
- Undo-Funktionalität lässt sich gut implementieren;
    
- Caretaker muss interne Details des Originators nicht kennen;
    
- Snapshot-Logik ist klar abgegrenzt;
    
- mehrere Zustände können gespeichert werden.
    

---

# Nachteile

Der größte Nachteil ist der Speicherverbrauch.

Angenommen, ein Objekt enthält:

```text
10 MB Daten
```

und wir speichern:

```text
100 Snapshots
```

Dann könnten theoretisch:

```text
≈ 1 GB
```

Speicher benötigt werden.

---

# Weitere Nachteile

- große Mementos können viel Speicher verbrauchen;
    
- häufige Snapshots können Performance kosten;
    
- Deep Copies können aufwendig sein;
    
- viele gespeicherte Zustände müssen eventuell begrenzt werden;
    
- bei komplexen Objektgraphen kann die Snapshot-Erstellung schwierig werden.
    

---

# Strategien gegen hohen Speicherverbrauch

Man kann beispielsweise:

```text
nur die letzten 20 Zustände speichern

alte Snapshots löschen

nur veränderte Daten speichern

Snapshots komprimieren

Snapshots persistent speichern
```

Welche Lösung sinnvoll ist, hängt von der Anwendung ab.

---

# Typische Anwendungsfälle

Memento eignet sich besonders für:

- Texteditoren;
    
- Grafikprogramme;
    
- Spiele;
    
- Undo/Redo;
    
- Dokumentbearbeitung;
    
- Konfigurationseditoren;
    
- Workflow-Systeme;
    
- Formulare mit Zurücksetzen-Funktion;
    
- komplexe Benutzerinteraktionen.
    

---

# Typisches Beispiel aus einer Desktop-App

Angenommen, ein Benutzer bearbeitet Einstellungen:

```text
Theme = Dark
Language = Deutsch
FontSize = 14
```

Vor Änderungen:

```csharp
SettingsMemento oldState =
    settings.Save();
```

Benutzer verändert:

```text
Theme = Light
Language = English
FontSize = 18
```

Dann klickt er:

```text
Abbrechen
```

Wir können:

```csharp
settings.Restore(oldState);
```

aufrufen.

Alle Änderungen werden verworfen.

---

# Vereinfachte Struktur

```text
                    Originator
                        │
                 ┌──────┴──────┐
                 │             │
              Save()        Restore()
                 │             ▲
                 ▼             │
               Memento ────────┘
                 │
                 ▼
              Caretaker
```

---

# Verantwortlichkeiten

```text
Originator
→ besitzt Zustand
→ erstellt Snapshot
→ stellt Snapshot wieder her


Memento
→ enthält Snapshot


Caretaker
→ speichert Snapshots
→ kennt interne Logik nicht
```

---

# Das solltest du dir merken

Memento:

```csharp
public sealed record Memento(
    int ValueA,
    int ValueB);
```

Originator:

```csharp
public class Originator
{
    private int _valueA;
    private int _valueB;

    public Memento Save()
    {
        return new Memento(
            _valueA,
            _valueB);
    }

    public void Restore(
        Memento memento)
    {
        _valueA = memento.ValueA;
        _valueB = memento.ValueB;
    }
}
```

Caretaker:

```csharp
public class History
{
    private readonly Stack<Memento> _history =
        new();

    public void Push(Memento memento)
    {
        _history.Push(memento);
    }

    public Memento Pop()
    {
        return _history.Pop();
    }
}
```

---

# Merksatz

> **Memento speichert den internen Zustand eines Objekts, damit dieser später wiederhergestellt werden kann, ohne die Kapselung zu verletzen.**

Noch einfacher:

```text
Originator:
"Das ist mein aktueller Zustand."

Memento:
"Ich speichere diesen Zustand."

Caretaker:
"Ich bewahre den Snapshot auf."
```

Oder ganz kurz:

```text
Memento
=
Save
+
Snapshot
+
Restore
```

---

> [!summary] Zusammenfassung  
> Das **Memento Pattern** ist ein **Verhaltensmuster**.
> 
> Es ermöglicht, den Zustand eines Objekts zu speichern und später wiederherzustellen.
> 
> Die drei wichtigsten Teilnehmer sind:
> 
> ```text
> Originator
> → besitzt den Zustand
> 
> Memento
> → speichert den Zustand
> 
> Caretaker
> → verwaltet die gespeicherten Zustände
> ```
> 
> Typischer Ablauf:
> 
> ```text
> Originator
>    │
>    │ Save()
>    ▼
> Memento
>    │
>    │ wird gespeichert
>    ▼
> Caretaker
> 
> später:
> 
> Memento
>    │
>    ▼
> Restore()
>    │
>    ▼
> Originator
> ```
> 
> Besonders typisch ist die Verwendung für **Undo**, **Spielstände**, **Snapshots** und **Rollback-Funktionen**.
> 
> In modernem C# eignet sich für ein Memento häufig ein unveränderliches `record`:
> 
> ```csharp
> public sealed record HeroMemento(
>     int Ammo,
>     int Lives);
> ```
> 
> **Kurz gesagt:**  
> `Memento = Zustand speichern, damit man später zu diesem Zustand zurückkehren kann.`