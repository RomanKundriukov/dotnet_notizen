Das **Command Pattern** ist ein Verhaltensmuster, bei dem eine Anfrage bzw. Aktion als **eigenständiges Objekt** gekapselt wird.

Dadurch wird derjenige, der eine Aktion auslöst, von dem Objekt getrennt, das die Aktion tatsächlich ausführt.

> [!info] Grundidee
> Der Sender einer Aktion muss nicht wissen, **wie** die Aktion ausgeführt wird.
>
> Er kennt lediglich ein Command-Objekt und ruft beispielsweise `Execute()` auf.

Vereinfacht:

```text
Invoker
   │
   │ Execute()
   ▼
Command
   │
   │ ruft Aktion auf
   ▼
Receiver
```

Beispiel:

```text
Fernbedienung
      │
      ▼
TVOnCommand
      │
      ▼
Fernseher
```

Die Fernbedienung weiß nicht, **wie ein Fernseher eingeschaltet wird**.

Sie weiß nur:

```csharp
command.Execute();
```

---

# Wann sollte man Command verwenden?

Das Command Pattern eignet sich besonders:

- wenn Aktionen als Objekte behandelt werden sollen;
- wenn Sender und Empfänger einer Aktion voneinander getrennt werden sollen;
- wenn Aktionen als Parameter übergeben werden sollen;
- wenn Befehle in einer Warteschlange gespeichert werden sollen;
- wenn Aktionen rückgängig gemacht werden sollen;
- wenn eine **Undo-/Redo-Funktion** benötigt wird;
- wenn ausgeführte Aktionen protokolliert werden sollen;
- wenn mehrere Befehle zu einer größeren Aktion zusammengefasst werden sollen.

---

# Grundstruktur

```text
Client
  │
  │ erstellt
  ▼
ConcreteCommand
  │
  │ kennt
  ▼
Receiver


Invoker
  │
  │ verwendet
  ▼
Command
  ▲
  │
ConcreteCommand
```

---

# Teilnehmer des Patterns

## Command

`Command` definiert den gemeinsamen Vertrag für alle Befehle.

Typischerweise:

```csharp
public interface ICommand
{
    void Execute();

    void Undo();
}
```

Dabei bedeutet:

```text
Execute()
→ Aktion ausführen

Undo()
→ Aktion rückgängig machen
```

---

## ConcreteCommand

Eine konkrete Command-Klasse implementiert `ICommand`.

Sie enthält meistens eine Referenz auf den **Receiver**.

```csharp
public class ConcreteCommand : ICommand
{
    private readonly Receiver _receiver;

    public ConcreteCommand(Receiver receiver)
    {
        _receiver = receiver;
    }

    public void Execute()
    {
        // Die eigentliche Arbeit wird
        // an den Receiver delegiert.
        _receiver.Operation();
    }

    public void Undo()
    {
        // Aktion rückgängig machen.
    }
}
```

---

## Receiver

Der `Receiver` ist das Objekt, das die eigentliche Arbeit ausführt.

```csharp
public class Receiver
{
    public void Operation()
    {
        Console.WriteLine(
            "Die eigentliche Aktion wird ausgeführt.");
    }
}
```

---

## Invoker

Der `Invoker` löst eine Command aus.

Er kennt dabei nicht unbedingt den konkreten Receiver.

```csharp
public class Invoker
{
    private ICommand? _command;

    public void SetCommand(ICommand command)
    {
        _command = command;
    }

    public void Run()
    {
        _command?.Execute();
    }

    public void Cancel()
    {
        _command?.Undo();
    }
}
```

---

## Client

Der Client verbindet die einzelnen Bestandteile miteinander.

```csharp
Receiver receiver = new();

ICommand command =
    new ConcreteCommand(receiver);

Invoker invoker = new();

invoker.SetCommand(command);

invoker.Run();
```

---

# Rollen im Überblick

| Rolle | Aufgabe |
|---|---|
| `Command` | definiert den gemeinsamen Vertrag |
| `ConcreteCommand` | implementiert eine konkrete Aktion |
| `Receiver` | führt die eigentliche Arbeit aus |
| `Invoker` | löst die Command aus |
| `Client` | erstellt und verbindet die Objekte |

---

# Ablauf

```text
Client
  │
  ├── erstellt Receiver
  │
  ├── erstellt Command
  │
  └── übergibt Command an Invoker
                │
                ▼
             Invoker
                │
                │ Execute()
                ▼
             Command
                │
                ▼
             Receiver
                │
                ▼
         eigentliche Aktion
```

---

# Praktisches Beispiel: Fernbedienung

Angenommen, wir möchten einen Fernseher über eine Fernbedienung steuern.

Dabei haben wir:

```text
Fernseher       → Receiver
TVOnCommand     → Command
Fernbedienung   → Invoker
Program         → Client
```

---

# Command Interface

```csharp
// Gemeinsamer Vertrag für alle Befehle.
public interface ICommand
{
    // Führt die Aktion aus.
    void Execute();

    // Macht die Aktion rückgängig.
    void Undo();
}
```

---

# Receiver: Fernseher

```csharp
// Receiver:
// Führt die eigentlichen Aktionen aus.
public class TV
{
    public void On()
    {
        Console.WriteLine(
            "Der Fernseher wurde eingeschaltet.");
    }

    public void Off()
    {
        Console.WriteLine(
            "Der Fernseher wurde ausgeschaltet.");
    }
}
```

---

# ConcreteCommand: Fernseher einschalten

```csharp
// Konkrete Command zum Einschalten des Fernsehers.
public class TVOnCommand : ICommand
{
    // Receiver, der die eigentliche Aktion ausführt.
    private readonly TV _tv;

    public TVOnCommand(TV tv)
    {
        _tv = tv;
    }

    public void Execute()
    {
        // Command delegiert die Arbeit
        // an den Receiver.
        _tv.On();
    }

    public void Undo()
    {
        // Gegenteil der ursprünglichen Aktion.
        _tv.Off();
    }
}
```

---

# Invoker: Fernbedienung

```csharp
// Invoker:
// Löst Befehle aus, kennt aber deren
// konkrete Implementierung nicht.
public class RemoteControl
{
    private ICommand? _command;

    public void SetCommand(ICommand command)
    {
        _command = command;
    }

    public void PressButton()
    {
        _command?.Execute();
    }

    public void PressUndo()
    {
        _command?.Undo();
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
        // Receiver erstellen.
        TV tv = new();

        // Invoker erstellen.
        RemoteControl remote = new();

        // Command erstellen und mit
        // dem Receiver verbinden.
        ICommand command =
            new TVOnCommand(tv);

        // Command der Fernbedienung zuweisen.
        remote.SetCommand(command);

        // Fernseher einschalten.
        remote.PressButton();

        // Aktion rückgängig machen.
        remote.PressUndo();
    }
}
```

Ausgabe:

```text
Der Fernseher wurde eingeschaltet.
Der Fernseher wurde ausgeschaltet.
```

---

# Was passiert hier genau?

Zuerst:

```csharp
TV tv = new();
```

Der Fernseher ist unser:

```text
Receiver
```

Dann:

```csharp
ICommand command =
    new TVOnCommand(tv);
```

Die Command kennt den Fernseher.

Danach:

```csharp
remote.SetCommand(command);
```

Die Fernbedienung erhält die Command.

Wenn wir:

```csharp
remote.PressButton();
```

aufrufen, passiert:

```text
RemoteControl
      │
      │ Execute()
      ▼
TVOnCommand
      │
      │ On()
      ▼
     TV
```

---

# Der wichtigste Punkt

Die Fernbedienung enthält **keinen Code wie**:

```csharp
_tv.On();
```

Sie kennt den Fernseher nicht.

Sie kennt nur:

```csharp
ICommand
```

und kann:

```csharp
_command.Execute();
```

aufrufen.

Dadurch entsteht eine lose Kopplung:

```text
RemoteControl
      │
      ▼
   ICommand
      ▲
      │
┌─────┴──────────────┐
│                    │
TVOnCommand     MicrowaveCommand
```

---

# Neue Command hinzufügen

Angenommen, die Fernbedienung soll zusätzlich eine Mikrowelle steuern.

Dann muss die Fernbedienung selbst nicht verändert werden.

Wir erstellen lediglich:

```text
Microwave
+
MicrowaveCommand
```

---

# Receiver: Mikrowelle

```csharp
public class Microwave
{
    public void StartCooking(int milliseconds)
    {
        Console.WriteLine(
            "Das Essen wird erwärmt.");

        // Simuliert die Dauer des Vorgangs.
        Task.Delay(milliseconds)
            .GetAwaiter()
            .GetResult();
    }

    public void StopCooking()
    {
        Console.WriteLine(
            "Das Erwärmen wurde beendet.");
    }
}
```

---

# MicrowaveCommand

```csharp
public class MicrowaveCommand : ICommand
{
    private readonly Microwave _microwave;

    private readonly int _time;

    public MicrowaveCommand(
        Microwave microwave,
        int time)
    {
        _microwave = microwave;
        _time = time;
    }

    public void Execute()
    {
        // Receiver führt die eigentliche Arbeit aus.
        _microwave.StartCooking(_time);

        _microwave.StopCooking();
    }

    public void Undo()
    {
        _microwave.StopCooking();
    }
}
```

---

# Dieselbe Fernbedienung verwenden

```csharp
RemoteControl remote = new();

TV tv = new();

remote.SetCommand(
    new TVOnCommand(tv));

remote.PressButton();


Microwave microwave = new();

remote.SetCommand(
    new MicrowaveCommand(
        microwave,
        5000));

remote.PressButton();
```

Wir verwenden denselben:

```text
RemoteControl
```

aber unterschiedliche Commands:

```text
TVOnCommand
MicrowaveCommand
```

---

# Problem: Keine Command gesetzt

Wenn keine Command gesetzt wurde:

```csharp
RemoteControl remote = new();

remote.PressButton();
```

dürfte der Invoker keine `NullReferenceException` verursachen.

Eine Möglichkeit ist:

```csharp
public void PressButton()
{
    if (_command is not null)
    {
        _command.Execute();
    }
}
```

In modernem C# noch kürzer:

```csharp
public void PressButton()
{
    _command?.Execute();
}
```

---

# Null Object Pattern: NoCommand

Eine andere Möglichkeit besteht darin, eine leere Command zu verwenden.

```csharp
// Leere Command.
//
// Sie führt bewusst keine Aktion aus.
public class NoCommand : ICommand
{
    public void Execute()
    {
    }

    public void Undo()
    {
    }
}
```

Dann kann der Invoker immer eine gültige Command besitzen:

```csharp
public class RemoteControl
{
    // Standardmäßig wird eine leere Command verwendet.
    private ICommand _command =
        new NoCommand();

    public void SetCommand(ICommand command)
    {
        _command = command;
    }

    public void PressButton()
    {
        _command.Execute();
    }

    public void PressUndo()
    {
        _command.Undo();
    }
}
```

Dadurch benötigen wir keine:

```csharp
if (_command != null)
```

Prüfung.

> [!note]
> `NoCommand` ist gleichzeitig ein einfaches Beispiel für das **Null Object Pattern**.

---

# Mehrere Commands

Ein Invoker muss nicht nur eine einzelne Command besitzen.

Eine Fernbedienung kann beispielsweise mehrere Tasten haben:

```text
Taste 0 → Fernseher ein
Taste 1 → Lautstärke erhöhen
Taste 2 → Licht ein
Taste 3 → Mikrowelle starten
```

Dafür kann ein Array oder eine Collection von Commands verwendet werden.

---

# Receiver: Volume

```csharp
public class Volume
{
    public const int Off = 0;

    public const int High = 20;

    private int _level = Off;

    // Lautstärke erhöhen.
    public void RaiseLevel()
    {
        if (_level < High)
        {
            _level++;
        }

        Console.WriteLine(
            $"Lautstärke: {_level}");
    }

    // Lautstärke reduzieren.
    public void DropLevel()
    {
        if (_level > Off)
        {
            _level--;
        }

        Console.WriteLine(
            $"Lautstärke: {_level}");
    }
}
```

---

# VolumeCommand

```csharp
public class VolumeCommand : ICommand
{
    private readonly Volume _volume;

    public VolumeCommand(Volume volume)
    {
        _volume = volume;
    }

    public void Execute()
    {
        // Lautstärke erhöhen.
        _volume.RaiseLevel();
    }

    public void Undo()
    {
        // Letzte Erhöhung rückgängig machen.
        _volume.DropLevel();
    }
}
```

---

# Command-History

Ein großer Vorteil des Command Patterns ist, dass Commands als normale Objekte gespeichert werden können.

Zum Beispiel:

```csharp
Stack<ICommand> history = new();
```

Nach jeder ausgeführten Aktion:

```csharp
history.Push(command);
```

Beim Rückgängigmachen:

```csharp
ICommand command =
    history.Pop();

command.Undo();
```

Damit lässt sich eine einfache **Undo-History** implementieren.

---

# MultiRemoteControl

```csharp
public class MultiRemoteControl
{
    // Verschiedene Buttons besitzen
    // jeweils eine eigene Command.
    private readonly ICommand[] _buttons;

    // Speichert die Reihenfolge
    // der ausgeführten Commands.
    private readonly Stack<ICommand> _history = new();

    public MultiRemoteControl(int buttonCount)
    {
        _buttons =
            new ICommand[buttonCount];

        // Alle Buttons erhalten zunächst
        // eine leere Command.
        for (int i = 0; i < _buttons.Length; i++)
        {
            _buttons[i] =
                new NoCommand();
        }
    }

    public void SetCommand(
        int buttonNumber,
        ICommand command)
    {
        _buttons[buttonNumber] =
            command;
    }

    public void PressButton(int buttonNumber)
    {
        ICommand command =
            _buttons[buttonNumber];

        // Command ausführen.
        command.Execute();

        // Ausgeführte Command
        // in der History speichern.
        _history.Push(command);
    }

    public void PressUndoButton()
    {
        if (_history.Count == 0)
        {
            return;
        }

        // Zuletzt ausgeführte Command holen.
        ICommand command =
            _history.Pop();

        // Aktion rückgängig machen.
        command.Undo();
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
        TV tv = new();

        Volume volume = new();

        MultiRemoteControl remote =
            new(buttonCount: 2);

        // Taste 0:
        // Fernseher einschalten.
        remote.SetCommand(
            0,
            new TVOnCommand(tv));

        // Taste 1:
        // Lautstärke erhöhen.
        remote.SetCommand(
            1,
            new VolumeCommand(volume));


        // Fernseher einschalten.
        remote.PressButton(0);

        // Lautstärke dreimal erhöhen.
        remote.PressButton(1);
        remote.PressButton(1);
        remote.PressButton(1);


        // Letzte Aktionen rückgängig machen.
        remote.PressUndoButton();
        remote.PressUndoButton();
        remote.PressUndoButton();
        remote.PressUndoButton();
    }
}
```

Ausgabe:

```text
Der Fernseher wurde eingeschaltet.
Lautstärke: 1
Lautstärke: 2
Lautstärke: 3

Lautstärke: 2
Lautstärke: 1
Lautstärke: 0
Der Fernseher wurde ausgeschaltet.
```

---

# Warum `Stack<ICommand>`?

Ein Stack arbeitet nach dem Prinzip:

**LIFO – Last In, First Out**

Also:

> Das zuletzt hinzugefügte Element wird zuerst entfernt.

Beispiel:

```text
Ausgeführte Commands:

TVOnCommand
VolumeCommand
VolumeCommand
VolumeCommand
```

Stack:

```text
┌─────────────────┐
│ VolumeCommand   │ ← zuerst Undo
├─────────────────┤
│ VolumeCommand   │
├─────────────────┤
│ VolumeCommand   │
├─────────────────┤
│ TVOnCommand     │
└─────────────────┘
```

Das eignet sich sehr gut für **Undo**.

---

# Undo

Beim Ausführen:

```csharp
command.Execute();
```

wird die Command gespeichert:

```csharp
_history.Push(command);
```

Beim Undo:

```csharp
ICommand command =
    _history.Pop();

command.Undo();
```

Damit wird immer die **zuletzt ausgeführte Aktion zuerst rückgängig gemacht**.

---

# Commands als Log

Da Commands normale Objekte sind, können sie gespeichert werden.

Zum Beispiel:

```text
Command 1
Command 2
Command 3
Command 4
```

Damit kann man eine History erzeugen.

Das Prinzip kann unter anderem für folgende Funktionen verwendet werden:

```text
Undo
Redo
Logging
Warteschlangen
Wiederherstellung von Aktionen
```

---

# Makrocommands

Eine **MacroCommand** fasst mehrere einzelne Commands zu einer einzigen Command zusammen.

Beispiel:

```text
MacroCommand
├── Command A
├── Command B
└── Command C
```

Von außen wird die MacroCommand wie eine normale Command behandelt:

```csharp
macroCommand.Execute();
```

Intern führt sie mehrere Commands aus.

---

# MacroCommand

```csharp
public class MacroCommand : ICommand
{
    private readonly List<ICommand> _commands;

    public MacroCommand(
        IEnumerable<ICommand> commands)
    {
        _commands =
            commands.ToList();
    }

    public void Execute()
    {
        // Alle enthaltenen Commands ausführen.
        foreach (ICommand command in _commands)
        {
            command.Execute();
        }
    }

    public void Undo()
    {
        // Beim Undo ist die umgekehrte Reihenfolge
        // häufig sinnvoll.
        for (int i = _commands.Count - 1; i >= 0; i--)
        {
            _commands[i].Undo();
        }
    }
}
```

> [!important]
> Bei `Undo()` ist es meistens sinnvoll, die Commands **in umgekehrter Reihenfolge** rückgängig zu machen.
>
> Wurden die Aktionen als:
>
> ```text
> A → B → C
> ```
>
> ausgeführt, sollte das Undo normalerweise:
>
> ```text
> C → B → A
> ```
>
> sein.

---

# Praktisches Beispiel: Softwareprojekt

Angenommen, mehrere Personen arbeiten an einem Softwareprojekt:

```text
Programmierer
Tester
Marketing
```

Ein Manager soll das gesamte Projekt starten.

---

# Receiver: Programmer

```csharp
public class Programmer
{
    public void StartCoding()
    {
        Console.WriteLine(
            "Der Programmierer beginnt mit der Entwicklung.");
    }

    public void StopCoding()
    {
        Console.WriteLine(
            "Der Programmierer beendet die Entwicklung.");
    }
}
```

---

# Receiver: Tester

```csharp
public class Tester
{
    public void StartTesting()
    {
        Console.WriteLine(
            "Der Tester beginnt mit den Tests.");
    }

    public void StopTesting()
    {
        Console.WriteLine(
            "Der Tester beendet die Tests.");
    }
}
```

---

# Receiver: Marketing

```csharp
public class MarketingSpecialist
{
    public void StartAdvertising()
    {
        Console.WriteLine(
            "Die Marketingkampagne wird gestartet.");
    }

    public void StopAdvertising()
    {
        Console.WriteLine(
            "Die Marketingkampagne wird beendet.");
    }
}
```

---

# CodeCommand

```csharp
public class CodeCommand : ICommand
{
    private readonly Programmer _programmer;

    public CodeCommand(
        Programmer programmer)
    {
        _programmer = programmer;
    }

    public void Execute()
    {
        _programmer.StartCoding();
    }

    public void Undo()
    {
        _programmer.StopCoding();
    }
}
```

---

# TestCommand

```csharp
public class TestCommand : ICommand
{
    private readonly Tester _tester;

    public TestCommand(
        Tester tester)
    {
        _tester = tester;
    }

    public void Execute()
    {
        _tester.StartTesting();
    }

    public void Undo()
    {
        _tester.StopTesting();
    }
}
```

---

# AdvertiseCommand

```csharp
public class AdvertiseCommand : ICommand
{
    private readonly MarketingSpecialist _marketing;

    public AdvertiseCommand(
        MarketingSpecialist marketing)
    {
        _marketing = marketing;
    }

    public void Execute()
    {
        _marketing.StartAdvertising();
    }

    public void Undo()
    {
        _marketing.StopAdvertising();
    }
}
```

---

# Invoker: Manager

```csharp
public class Manager
{
    private ICommand _command =
        new NoCommand();

    public void SetCommand(
        ICommand command)
    {
        _command = command;
    }

    public void StartProject()
    {
        _command.Execute();
    }

    public void StopProject()
    {
        _command.Undo();
    }
}
```

---

# Verwendung der MacroCommand

```csharp
public class Program
{
    public static void Main()
    {
        // Receiver erstellen.
        Programmer programmer = new();

        Tester tester = new();

        MarketingSpecialist marketing = new();


        // Einzelne Commands erstellen.
        List<ICommand> commands = new()
        {
            new CodeCommand(programmer),

            new TestCommand(tester),

            new AdvertiseCommand(marketing)
        };


        // Alle Commands zu einer
        // MacroCommand zusammenfassen.
        ICommand projectCommand =
            new MacroCommand(commands);


        // Invoker erstellen.
        Manager manager = new();

        manager.SetCommand(
            projectCommand);


        // Führt alle Commands aus.
        manager.StartProject();


        // Macht alle Commands rückgängig.
        manager.StopProject();
    }
}
```

---

# Ablauf der MacroCommand

Bei:

```csharp
manager.StartProject();
```

passiert:

```text
Manager
   │
   ▼
MacroCommand
   │
   ├── CodeCommand
   │      │
   │      ▼
   │   Programmer
   │
   ├── TestCommand
   │      │
   │      ▼
   │    Tester
   │
   └── AdvertiseCommand
          │
          ▼
       Marketing
```

Der Manager muss nicht wissen:

- wie programmiert wird;
- wie getestet wird;
- wie Marketing funktioniert.

Er kennt lediglich:

```csharp
ICommand
```

---

# Warum Command statt direkter Methodenaufrufe?

Ohne Command:

```csharp
tv.On();

volume.RaiseLevel();

microwave.StartCooking(5000);
```

Der aufrufende Code muss alle konkreten Objekte kennen.

Mit Command:

```csharp
command.Execute();
```

Der Invoker kennt nur:

```csharp
ICommand
```

Dadurch können Commands einfach:

- ausgetauscht;
- gespeichert;
- verzögert ausgeführt;
- wiederholt;
- rückgängig gemacht;
- gruppiert werden.

---

# Command und lose Kopplung

Ohne Command:

```text
RemoteControl
      │
      ▼
      TV
```

Die Fernbedienung hängt direkt vom Fernseher ab.

Mit Command:

```text
RemoteControl
      │
      ▼
   ICommand
      ▲
      │
 TVOnCommand
      │
      ▼
      TV
```

Der Invoker ist dadurch vom Receiver entkoppelt.

---

# Command in WPF / MVVM

Das Command-Prinzip ist besonders wichtig bei XAML-basierten Anwendungen.

Zum Beispiel:

```xml
<Button
    Content="Speichern"
    Command="{Binding SaveCommand}" />
```

Das UI ruft nicht direkt:

```csharp
Save();
```

auf.

Stattdessen ist die Aktion in einer Command gekapselt:

```text
Button
   │
   ▼
SaveCommand
   │
   ▼
ViewModel
```

Das passt sehr gut zum MVVM-Prinzip.

---

# Command und CQRS

Auch in CQRS begegnet man dem Begriff **Command**.

Beispiele:

```text
CreateOrderCommand
DeleteUserCommand
UpdateCustomerCommand
SendEmailCommand
```

Eine solche Command beschreibt typischerweise die Absicht:

> „Führe diese Änderung aus.“

Beispiel:

```csharp
public record CreateOrderCommand(
    int CustomerId,
    decimal Amount);
```

> [!note]
> CQRS Commands und das klassische GoF Command Pattern sind nicht exakt dasselbe Konzept, besitzen aber eine ähnliche Grundidee:
>
> **Eine Aktion bzw. Absicht wird als eigenes Objekt dargestellt.**

---

# Command vs. Strategy

Die beiden Patterns können strukturell ähnlich aussehen.

## Strategy

```text
Context
   │
   ▼
Strategy
```

Strategy beantwortet:

> **Wie soll etwas ausgeführt werden?**

Beispiel:

```text
Sortieren
├── QuickSort
├── MergeSort
└── BubbleSort
```

---

## Command

```text
Invoker
   │
   ▼
Command
   │
   ▼
Receiver
```

Command beantwortet:

> **Welche Aktion soll ausgeführt werden?**

Beispiel:

```text
TVOnCommand
TVOffCommand
VolumeUpCommand
SaveCommand
DeleteCommand
```

Kurz:

```text
Strategy
→ Algorithmus kapseln

Command
→ Aktion / Anfrage kapseln
```

---

# Vorteile

- Sender und Empfänger werden voneinander getrennt.
- Aktionen werden als eigenständige Objekte behandelt.
- Commands können einfach ausgetauscht werden.
- Undo/Redo lässt sich gut implementieren.
- Commands können gespeichert werden.
- Commands können protokolliert werden.
- Commands können in Warteschlangen gelegt werden.
- Mehrere Commands können zu MacroCommands kombiniert werden.
- Neue Commands können hinzugefügt werden, ohne den Invoker zu verändern.

---

# Nachteile

- Für viele Aktionen entstehen viele zusätzliche Klassen.
- Die Architektur kann bei einfachen Aktionen unnötig komplex werden.
- `Undo()` kann bei komplexen Operationen schwierig zu implementieren sein.
- Commands mit viel internem Zustand können komplex werden.

---

# Klassische Struktur

```text
                     Client
                       │
                       │ erstellt
                       ▼
                ConcreteCommand
                   │       │
                   │       ▼
                   │    Receiver
                   │
                   ▼
                 Command
                   ▲
                   │
                Invoker
```

---

# Praktische Struktur

```text
                 RemoteControl
                     │
                     ▼
                  ICommand
                     ▲
          ┌──────────┼──────────┐
          │          │          │
   TVOnCommand  VolumeCommand  MicrowaveCommand
          │          │          │
          ▼          ▼          ▼
         TV       Volume     Microwave
```

---

# Das solltest du dir merken

Der gemeinsame Vertrag:

```csharp
public interface ICommand
{
    void Execute();

    void Undo();
}
```

Eine konkrete Command:

```csharp
public class TVOnCommand : ICommand
{
    private readonly TV _tv;

    public TVOnCommand(TV tv)
    {
        _tv = tv;
    }

    public void Execute()
    {
        _tv.On();
    }

    public void Undo()
    {
        _tv.Off();
    }
}
```

Invoker:

```csharp
public class RemoteControl
{
    private ICommand _command =
        new NoCommand();

    public void SetCommand(
        ICommand command)
    {
        _command = command;
    }

    public void PressButton()
    {
        _command.Execute();
    }

    public void PressUndo()
    {
        _command.Undo();
    }
}
```

---

# Merksatz

> **Command kapselt eine Aktion oder Anfrage in einem eigenen Objekt.**

Noch einfacher:

```text
Invoker:
"Führe diese Command aus."

Command:
"Ich weiß, welchen Receiver
und welche Aktion ich aufrufen muss."

Receiver:
"Ich erledige die eigentliche Arbeit."
```

Oder:

```text
Command
=
Aktion als Objekt
```

---

> [!summary] Zusammenfassung
> Das **Command Pattern** kapselt eine Aktion oder Anfrage in einem eigenen Objekt.
>
> Der **Invoker** löst die Command aus:
>
> ```csharp
> command.Execute();
> ```
>
> Die **ConcreteCommand** kennt den entsprechenden **Receiver** und delegiert die eigentliche Arbeit:
>
> ```text
> Invoker
>     ↓
> Command
>     ↓
> Receiver
> ```
>
> Da Commands normale Objekte sind, können sie:
>
> - gespeichert,
> - protokolliert,
> - in Warteschlangen gelegt,
> - rückgängig gemacht,
> - wiederholt,
> - zu MacroCommands kombiniert
>
> werden.
>
> Besonders wichtig ist das Pattern für **Undo/Redo**, **WPF/MVVM**, Benutzeraktionen und command-basierte Architekturen.
>
> **Kurz gesagt:**
>
> `Command = eine Aktion als Objekt kapseln.`