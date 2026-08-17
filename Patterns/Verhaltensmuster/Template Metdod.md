> [!important]  
> **Template Method gehört zu den Verhaltensmustern**, nicht zu den Erzeugungsmustern.

Das **Template Method Pattern** definiert die feste Struktur eines Algorithmus in einer Basisklasse.

Einzelne Schritte dieses Algorithmus können von Unterklassen unterschiedlich implementiert oder überschrieben werden, **ohne dass sich die grundsätzliche Reihenfolge des Algorithmus ändert**.

> [!info] Grundidee  
> Die Basisklasse bestimmt:
> 
> **Welche Schritte werden ausgeführt und in welcher Reihenfolge?**
> 
> Die Unterklassen bestimmen:
> 
> **Wie werden einzelne Schritte konkret ausgeführt?**

---

# Wann sollte man Template Method verwenden?

Das Pattern eignet sich besonders:

- wenn mehrere Klassen grundsätzlich **denselben Ablauf** besitzen;
    
- wenn sich nur einzelne Schritte dieses Ablaufs unterscheiden;
    
- wenn gemeinsamer Code nicht in mehreren Klassen dupliziert werden soll;
    
- wenn die Reihenfolge der Schritte zentral festgelegt werden soll;
    
- wenn Unterklassen bestimmte Schritte anpassen dürfen, aber nicht den gesamten Algorithmus verändern sollen.
    

---

# Grundidee

Angenommen, ein Algorithmus besteht aus:

```text
1. OperationA()
2. OperationB()
3. OperationC()
```

Die Reihenfolge soll immer gleich bleiben:

```text
OperationA()
     ↓
OperationB()
     ↓
OperationC()
```

Aber einzelne Klassen dürfen beispielsweise `OperationA()` und `OperationC()` unterschiedlich implementieren.

---

# Allgemeine Struktur

```text
              AbstractClass
                    │
                    │
          TemplateMethod()
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
      StepA()    StepB()    StepC()
          ▲                   ▲
          │                   │
          └──── ConcreteClass ┘
```

---

# Allgemeines C#-Beispiel

```csharp
public abstract class AbstractClass
{
    // Template Method:
    // Definiert die feste Reihenfolge des Algorithmus.
    public void Execute()
    {
        Operation1();
        Operation2();
    }

    // Diese Schritte müssen von Unterklassen
    // konkret implementiert werden.
    protected abstract void Operation1();

    protected abstract void Operation2();
}
```

Konkrete Implementierung:

```csharp
public class ConcreteClass : AbstractClass
{
    protected override void Operation1()
    {
        Console.WriteLine("Operation 1 wird ausgeführt.");
    }

    protected override void Operation2()
    {
        Console.WriteLine("Operation 2 wird ausgeführt.");
    }
}
```

Verwendung:

```csharp
AbstractClass algorithm =
    new ConcreteClass();

algorithm.Execute();
```

---

# Warum `protected`?

Die einzelnen Schritte:

```csharp
protected abstract void Operation1();
```

sind normalerweise keine Funktionen, die der Client direkt aufrufen soll.

Der Client soll hauptsächlich:

```csharp
algorithm.Execute();
```

verwenden.

Die einzelnen Schritte sind interne Bestandteile des Algorithmus.

Deshalb ist in diesem Pattern häufig:

```csharp
protected
```

sinnvoller als:

```csharp
public
```

---

# Teilnehmer des Patterns

## AbstractClass

Die abstrakte Basisklasse definiert:

1. den gesamten Algorithmus;
    
2. seine Reihenfolge;
    
3. abstrakte oder virtuelle Einzelschritte.
    

Beispiel:

```csharp
public abstract class AbstractClass
{
    public void TemplateMethod()
    {
        Step1();
        Step2();
        Step3();
    }

    protected abstract void Step1();

    protected virtual void Step2()
    {
        Console.WriteLine("Standardimplementierung");
    }

    protected abstract void Step3();
}
```

---

## ConcreteClass

Eine konkrete Unterklasse implementiert oder überschreibt einzelne Schritte:

```csharp
public class ConcreteClass : AbstractClass
{
    protected override void Step1()
    {
        Console.WriteLine("Eigener Schritt 1");
    }

    protected override void Step3()
    {
        Console.WriteLine("Eigener Schritt 3");
    }
}
```

Die Reihenfolge:

```text
Step1
  ↓
Step2
  ↓
Step3
```

bleibt unverändert.

---

# Praktisches Beispiel: Bildung

Angenommen, wir haben zwei Bildungswege:

```text
Schule
Universität
```

Beide besitzen ungefähr denselben Ablauf:

```text
Aufnahme
   ↓
Lernen
   ↓
Prüfungen
   ↓
Abschlussdokument erhalten
```

Die konkreten Schritte unterscheiden sich jedoch.

---

# Ohne Template Method

Man könnte zunächst zwei unabhängige Klassen erstellen.

## Schule

```csharp
public class School
{
    public void Enter()
    {
        // Einschulung
    }

    public void Study()
    {
        // Unterricht
    }

    public void PassExams()
    {
        // Abschlussprüfungen
    }

    public void GetCertificate()
    {
        // Abschlusszeugnis
    }
}
```

## Universität

```csharp
public class University
{
    public void Enter()
    {
        // Immatrikulation
    }

    public void Study()
    {
        // Vorlesungen
    }

    public void Practice()
    {
        // Praktikum
    }

    public void PassExams()
    {
        // Abschlussprüfungen
    }

    public void GetDiploma()
    {
        // Hochschulabschluss
    }
}
```

Das Problem:

Beide Klassen besitzen praktisch denselben übergeordneten Algorithmus.

```text
Enter()
Study()
PassExams()
GetDocument()
```

Es entsteht unnötige Duplizierung.

---

# Lösung mit Template Method

Wir erstellen eine gemeinsame abstrakte Klasse:

```csharp
public abstract class Education
{
    // Template Method.
    // Sie definiert den vollständigen Ablauf der Ausbildung.
    public void Learn()
    {
        Enter();
        Study();
        PassExams();
        GetDocument();
    }

    // Muss von jeder Unterklasse implementiert werden.
    protected abstract void Enter();

    // Muss von jeder Unterklasse implementiert werden.
    protected abstract void Study();

    // Besitzt bereits eine Standardimplementierung.
    // Eine Unterklasse kann sie bei Bedarf überschreiben.
    protected virtual void PassExams()
    {
        Console.WriteLine(
            "Die Abschlussprüfungen werden abgelegt.");
    }

    // Muss von jeder Unterklasse implementiert werden.
    protected abstract void GetDocument();
}
```

Der komplette Algorithmus befindet sich jetzt an **einer Stelle**:

```csharp
public void Learn()
{
    Enter();
    Study();
    PassExams();
    GetDocument();
}
```

---

# School

```csharp
public class School : Education
{
    protected override void Enter()
    {
        Console.WriteLine(
            "Das Kind wird in die erste Klasse eingeschult.");
    }

    protected override void Study()
    {
        Console.WriteLine(
            "Der Schüler besucht den Unterricht und macht Hausaufgaben.");
    }

    // PassExams() wird nicht überschrieben.
    // Deshalb wird die Standardimplementierung
    // aus Education verwendet.

    protected override void GetDocument()
    {
        Console.WriteLine(
            "Der Schüler erhält sein Abschlusszeugnis.");
    }
}
```

---

# University

```csharp
public class University : Education
{
    protected override void Enter()
    {
        Console.WriteLine(
            "Der Student schreibt sich an der Universität ein.");
    }

    protected override void Study()
    {
        Console.WriteLine(
            "Der Student besucht Vorlesungen.");

        Console.WriteLine(
            "Der Student absolviert ein Praktikum.");
    }

    // Die Universität benötigt eine andere
    // Art der Abschlussprüfung.
    protected override void PassExams()
    {
        Console.WriteLine(
            "Der Student legt die Abschlussprüfung im Fachgebiet ab.");
    }

    protected override void GetDocument()
    {
        Console.WriteLine(
            "Der Student erhält seinen Hochschulabschluss.");
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
        Education school =
            new School();

        Education university =
            new University();

        Console.WriteLine("Schule:");
        school.Learn();

        Console.WriteLine();

        Console.WriteLine("Universität:");
        university.Learn();
    }
}
```

Ausgabe:

```text
Schule:
Das Kind wird in die erste Klasse eingeschult.
Der Schüler besucht den Unterricht und macht Hausaufgaben.
Die Abschlussprüfungen werden abgelegt.
Der Schüler erhält sein Abschlusszeugnis.

Universität:
Der Student schreibt sich an der Universität ein.
Der Student besucht Vorlesungen.
Der Student absolviert ein Praktikum.
Der Student legt die Abschlussprüfung im Fachgebiet ab.
Der Student erhält seinen Hochschulabschluss.
```

---

# Was passiert genau?

Bei:

```csharp
school.Learn();
```

wird die Methode aus `Education` ausgeführt:

```csharp
public void Learn()
{
    Enter();
    Study();
    PassExams();
    GetDocument();
}
```

Da das tatsächliche Objekt aber:

```csharp
new School()
```

ist, werden bei abstrakten beziehungsweise virtuellen Methoden die Implementierungen von `School` verwendet.

Also:

```text
Education.Learn()
       │
       ├── School.Enter()
       │
       ├── School.Study()
       │
       ├── Education.PassExams()
       │
       └── School.GetDocument()
```

Bei der Universität:

```text
Education.Learn()
       │
       ├── University.Enter()
       │
       ├── University.Study()
       │
       ├── University.PassExams()
       │
       └── University.GetDocument()
```

Die Struktur ist gleich.

Die Implementierungen unterscheiden sich.

---

# `abstract` vs. `virtual`

Beim Template Method Pattern werden häufig beide Varianten verwendet.

## `abstract`

```csharp
protected abstract void Enter();
```

Bedeutet:

> Die Basisklasse besitzt keine Standardimplementierung.

Jede konkrete Unterklasse **muss** diese Methode implementieren.

---

## `virtual`

```csharp
protected virtual void PassExams()
{
    Console.WriteLine(
        "Die Abschlussprüfungen werden abgelegt.");
}
```

Bedeutet:

> Es existiert bereits eine Standardimplementierung.

Eine Unterklasse **kann**, muss sie aber nicht überschreiben.

Beispiel:

```text
School
→ verwendet Standardimplementierung

University
→ überschreibt PassExams()
```

---

# Template Method selbst nicht `virtual`

Die Template Method sollte normalerweise nicht überschrieben werden können.

Deshalb:

```csharp
public void Learn()
```

und nicht:

```csharp
public virtual void Learn()
```

Denn die zentrale Idee ist:

> Die Unterklassen dürfen **Schritte ändern**, aber nicht die Struktur des Algorithmus.

---

# Problem mit `new`

Eine Unterklasse könnte theoretisch eine gleichnamige Methode definieren:

```csharp
public class School : Education
{
    public new void Learn()
    {
        Console.WriteLine(
            "Ich möchte nicht lernen.");
    }

    // ...
}
```

Das ist allerdings **kein echtes Überschreiben**.

Es handelt sich um:

**Method Hiding**

also das Verbergen einer geerbten Methode.

---

# Warum ist Method Hiding problematisch?

Betrachten wir:

```csharp
School school = new School();

school.Learn();
```

Hier wird die versteckte Methode von `School` verwendet.

Dagegen:

```csharp
Education education =
    new School();

education.Learn();
```

Hier wird:

```csharp
Education.Learn()
```

verwendet.

Das Verhalten hängt also vom statischen Variablentyp ab.

Das kann verwirrend sein und sollte normalerweise vermieden werden.

> [!warning]  
> `new` ist **kein Ersatz für `override`**.
> 
> Es versteckt lediglich ein Member der Basisklasse.

---

# `sealed override`

Eine interessante Situation entsteht, wenn `Education` selbst von einer anderen Klasse erbt.

Zum Beispiel:

```csharp
public abstract class Learning
{
    public abstract void Learn();
}
```

Dann muss `Education` diese Methode überschreiben:

```csharp
public abstract class Education : Learning
{
    public override void Learn()
    {
        Enter();
        Study();
        PassExams();
        GetDocument();
    }

    // ...
}
```

Damit weitere Unterklassen `Learn()` nicht erneut überschreiben können, kann man:

```csharp
sealed override
```

verwenden.

---

# Beispiel mit `sealed override`

```csharp
public abstract class Learning
{
    public abstract void Learn();
}
```

Dann:

```csharp
public abstract class Education : Learning
{
    // Der Algorithmus wird hier endgültig festgelegt.
    // Weitere Unterklassen dürfen Learn()
    // nicht mehr überschreiben.
    public sealed override void Learn()
    {
        Enter();
        Study();
        PassExams();
        GetDocument();
    }

    protected abstract void Enter();

    protected abstract void Study();

    protected virtual void PassExams()
    {
        Console.WriteLine(
            "Die Abschlussprüfungen werden abgelegt.");
    }

    protected abstract void GetDocument();
}
```

Jetzt ist Folgendes nicht erlaubt:

```csharp
public class School : Education
{
    // ❌ Compilerfehler:
    // Learn() ist in Education sealed.
    public override void Learn()
    {
    }
}
```

---

# `sealed` bedeutet

```text
override
→ Ich überschreibe eine Methode der Basisklasse.

sealed override
→ Ich überschreibe sie hier,
  aber weitere Unterklassen dürfen sie
  nicht mehr überschreiben.
```

---

# Hook Methods

Beim Template Method Pattern gibt es häufig noch sogenannte **Hooks**.

Ein Hook ist ein optionaler Schritt mit einer Standardimplementierung.

Beispiel:

```csharp
public abstract class DataProcessor
{
    public void Process()
    {
        LoadData();
        ProcessData();

        if (ShouldSave())
        {
            SaveData();
        }
    }

    protected abstract void LoadData();

    protected abstract void ProcessData();

    protected abstract void SaveData();

    // Hook:
    // Kann von einer Unterklasse überschrieben werden,
    // muss aber nicht.
    protected virtual bool ShouldSave()
    {
        return true;
    }
}
```

Eine Unterklasse kann beispielsweise entscheiden:

```csharp
protected override bool ShouldSave()
{
    return false;
}
```

Dadurch kann ein Teil des Algorithmus beeinflusst werden, ohne den gesamten Algorithmus zu überschreiben.

---

# Beispiel mit Hook

```csharp
public abstract class ReportGenerator
{
    public void Generate()
    {
        LoadData();
        CreateReport();

        if (ShouldExport())
        {
            Export();
        }
    }

    protected abstract void LoadData();

    protected abstract void CreateReport();

    protected abstract void Export();

    // Optionaler Erweiterungspunkt.
    protected virtual bool ShouldExport()
    {
        return true;
    }
}
```

Eine konkrete Implementierung:

```csharp
public class PreviewReport : ReportGenerator
{
    protected override void LoadData()
    {
        Console.WriteLine("Daten werden geladen.");
    }

    protected override void CreateReport()
    {
        Console.WriteLine("Bericht wird erstellt.");
    }

    protected override void Export()
    {
        Console.WriteLine("Bericht wird exportiert.");
    }

    // Bei einer Vorschau soll kein Export erfolgen.
    protected override bool ShouldExport()
    {
        return false;
    }
}
```

---

# Template Method reduziert Code-Duplizierung

Ohne Template Method:

```text
ClassA:
Load()
Validate()
Process()
Save()

ClassB:
Load()
Validate()
Process()
Save()

ClassC:
Load()
Validate()
Process()
Save()
```

Die Struktur wird dreimal wiederholt.

Mit Template Method:

```text
          BaseClass
              │
              ▼
Load()
Validate()
Process()
Save()
              │
      ┌───────┼───────┐
      ▼       ▼       ▼
   ClassA   ClassB   ClassC
```

Der Ablauf existiert nur einmal.

---

# Template Method und Vererbung

Template Method basiert stark auf **Vererbung**.

```text
AbstractClass
      ▲
      │
ConcreteClass
```

Die Basisklasse definiert den Algorithmus und ruft Methoden auf, die von Unterklassen implementiert werden.

Das Prinzip lautet oft:

> **Don't call us, we'll call you.**

Das bedeutet:

Die Unterklasse ruft nicht selbst den gesamten Algorithmus auf.

Stattdessen kontrolliert die Basisklasse den Ablauf und ruft die Implementierungen der Unterklasse zum passenden Zeitpunkt auf.

Dieses Prinzip wird auch:

**Hollywood Principle**

genannt.

---

# Template Method vs. Strategy

Template Method und Strategy lösen teilweise ähnliche Probleme, aber auf unterschiedliche Weise.

## Template Method

Verwendet:

```text
Vererbung
```

Die Basisklasse definiert den Algorithmus:

```text
Algorithm
├── Step1
├── Step2
└── Step3
```

Unterklassen verändern bestimmte Schritte.

---

## Strategy

Verwendet:

```text
Komposition
```

Ein komplettes Verhalten wird in ein separates Objekt ausgelagert:

```text
Context
   │
   ▼
IStrategy
   ▲
 ┌─┴─────────┐
 │           │
StrategyA StrategyB
```

---

# Template Method vs. Strategy

|Template Method|Strategy|
|---|---|
|basiert auf Vererbung|basiert auf Komposition|
|Grundalgorithmus liegt in Basisklasse|Algorithmus liegt in separater Strategie|
|Unterklasse ändert einzelne Schritte|komplette Strategie kann ausgetauscht werden|
|Verhalten meist zur Compile-Zeit festgelegt|Verhalten kann einfach zur Laufzeit gewechselt werden|

Merksatz:

```text
Template Method
→ gleichen Algorithmus vererben
  und einzelne Schritte verändern

Strategy
→ kompletten Algorithmus austauschen
```

---

# Vorteile

- reduziert Code-Duplizierung;
    
- zentralisiert die Struktur eines Algorithmus;
    
- einzelne Schritte können angepasst werden;
    
- Reihenfolge des Algorithmus bleibt kontrolliert;
    
- gemeinsame Standardimplementierungen sind möglich;
    
- unterstützt Wiederverwendung von Code.
    

---

# Nachteile

- starke Bindung an Vererbung;
    
- viele abstrakte Schritte können Unterklassen kompliziert machen;
    
- Änderungen an der Basisklasse können viele Unterklassen beeinflussen;
    
- zu viele Hooks und `virtual`-Methoden können den Ablauf schwer nachvollziehbar machen;
    
- weniger flexibel als Komposition, wenn ganze Algorithmen dynamisch ausgetauscht werden sollen.
    

---

# Wann Template Method nicht sinnvoll ist

Wenn zwei Klassen völlig unterschiedliche Algorithmen besitzen:

```text
ClassA:
A → B → C

ClassB:
X → Y → Z
```

gibt es kaum eine gemeinsame Vorlage.

Dann bringt Template Method wenig.

Auch wenn das Verhalten häufig zur Laufzeit gewechselt werden soll, ist beispielsweise **Strategy** oft geeigneter.

---

# Vereinfachte Struktur

```text
               Education
                   │
                   │ Learn()
                   ▼
          ┌─────────────────┐
          │ 1. Enter()      │
          │ 2. Study()      │
          │ 3. PassExams()  │
          │ 4. GetDocument()│
          └─────────────────┘
                   ▲
             ┌─────┴─────┐
             │           │
           School    University
```

---

# Der wichtigste Punkt

Der Algorithmus:

```csharp
public void Learn()
{
    Enter();
    Study();
    PassExams();
    GetDocument();
}
```

ist die **Template Method**.

Die Methoden:

```csharp
Enter();
Study();
PassExams();
GetDocument();
```

sind die einzelnen **Schritte des Algorithmus**.

Unterklassen dürfen diese Schritte unterschiedlich implementieren:

```text
                Learn()
                  │
        feste Reihenfolge
                  │
        ┌─────────┴─────────┐
        ▼                   ▼
      School            University
        │                   │
 eigene Schritte       eigene Schritte
```

---

# Das solltest du dir merken

Typische Struktur:

```csharp
public abstract class BaseAlgorithm
{
    // Template Method:
    // legt die Reihenfolge fest.
    public void Execute()
    {
        Step1();
        Step2();
        Step3();
    }

    // Muss implementiert werden.
    protected abstract void Step1();

    // Besitzt Standardverhalten.
    protected virtual void Step2()
    {
    }

    // Muss implementiert werden.
    protected abstract void Step3();
}
```

---

# Merksatz

> **Template Method definiert die feste Struktur eines Algorithmus, während Unterklassen einzelne Schritte dieses Algorithmus anpassen können.**

Noch einfacher:

```text
Basisklasse:
"Diese Schritte werden in dieser Reihenfolge ausgeführt."

Unterklasse:
"Ich entscheide, wie einzelne Schritte ausgeführt werden."
```

Oder ganz kurz:

```text
Template Method
=
fester Ablauf
+
austauschbare Einzelschritte
```

> [!summary] Zusammenfassung  
> Das **Template Method Pattern** ist ein **Verhaltensmuster**.
> 
> Die Basisklasse definiert in einer Template Method den vollständigen Ablauf eines Algorithmus.
> 
> Die einzelnen Schritte werden als `abstract` oder `virtual` definiert:
> 
> - `abstract` → Unterklasse **muss** den Schritt implementieren.
>     
> - `virtual` → es gibt eine Standardimplementierung, die Unterklasse **kann** sie überschreiben.
>     
> 
> Die Template Method selbst sollte normalerweise nicht `virtual` sein, damit die Struktur des Algorithmus erhalten bleibt.
> 
> Falls sie bereits eine geerbte abstrakte oder virtuelle Methode überschreibt, kann `sealed override` verwendet werden:
> 
> ```csharp
> public sealed override void Learn()
> ```
> 
> **Kurz gesagt:**  
> `Template Method = gleicher Ablauf, unterschiedliche Implementierung einzelner Schritte.`