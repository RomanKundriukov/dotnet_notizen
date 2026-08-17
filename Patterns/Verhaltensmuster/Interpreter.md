Das **Interpreter Pattern** ist ein Verhaltensmuster, das eine einfache Sprache oder Grammatik durch Klassen abbildet und Ausdrücke dieser Sprache anschließend interpretiert.

Dabei wird jede wichtige Regel der Sprache durch eine eigene Klasse repräsentiert.

> [!info] Grundidee  
> Eine Sprache besteht aus Regeln.
> 
> Das Interpreter Pattern bildet diese Regeln als Objekte ab und wertet daraus zusammengesetzte Ausdrücke aus.

Ein einfaches Beispiel wäre eine Sprache für mathematische Ausdrücke:

```text
x + y - z
```

Dabei könnten Klassen existieren für:

```text
Variable
Addition
Subtraktion
Konstante
```

---

# Wann sollte man Interpreter verwenden?

Das Pattern eignet sich besonders:

- wenn eine kleine, klar definierte Sprache ausgewertet werden soll;
    
- wenn Regeln oder Ausdrücke häufig wiederholt interpretiert werden;
    
- wenn die Grammatik relativ einfach ist;
    
- wenn einzelne Grammatikregeln als Klassen dargestellt werden sollen;
    
- wenn neue Regeln leicht ergänzt werden sollen;
    
- wenn ein Ausdruck als Baum aus einzelnen Teil-Ausdrücken aufgebaut werden kann.
    

Typische Beispiele:

- mathematische Ausdrücke;
    
- Filterausdrücke;
    
- Suchbedingungen;
    
- kleine DSLs;
    
- einfache Regelwerke;
    
- Berechtigungsregeln;
    
- Konfigurationssprachen.
    

---

# Wann sollte man Interpreter nicht verwenden?

Bei einer großen und komplexen Sprache wird das Pattern schnell unübersichtlich.

Zum Beispiel bei:

```text
komplexer Programmiersprache
+
vielen Grammatikregeln
+
Operator-Prioritäten
+
Scopes
+
Funktionen
+
Typensystem
+
Fehlerdiagnostik
```

sind spezialisierte Parser- und Compiler-Techniken normalerweise besser geeignet.

> [!warning]  
> Interpreter eignet sich besonders für **kleine und überschaubare Grammatiken**.

---

# Grundstruktur

```text
                     IExpression
                          ▲
              ┌───────────┴───────────┐
              │                       │
     TerminalExpression     NonterminalExpression
              │                       │
              │              ┌────────┴────────┐
              │              ▼                 ▼
              │         Expression         Expression
              │
              ▼
           Context
```

---

# Teilnehmer des Patterns

## AbstractExpression

`AbstractExpression` beziehungsweise ein Interface definiert den gemeinsamen Vertrag aller Ausdrücke.

Typischerweise:

```csharp
public interface IExpression
{
    int Interpret(Context context);
}
```

Jeder Ausdruck kann also interpretiert werden.

---

## TerminalExpression

Ein **TerminalExpression** repräsentiert einen elementaren Ausdruck, der nicht weiter zerlegt wird.

Zum Beispiel:

```text
x
y
z
5
10
```

Eine Variable wie:

```text
x
```

kann direkt über den `Context` ausgewertet werden.

---

## NonterminalExpression

Ein **NonterminalExpression** kombiniert andere Expressions.

Zum Beispiel:

```text
x + y
```

besteht aus:

```text
linke Expression
+
rechte Expression
```

oder:

```text
x - y
```

besteht aus:

```text
linke Expression
-
rechte Expression
```

Diese Expressions können wiederum weitere Expressions enthalten.

Dadurch entsteht eine rekursive Struktur.

---

## Context

Der `Context` enthält Informationen, die bei der Interpretation benötigt werden.

Zum Beispiel:

```text
x = 5
y = 8
z = 2
```

Die Expressions können auf diese Werte zugreifen.

---

## Client

Der Client:

1. erstellt den Context;
    
2. definiert Variablen;
    
3. baut den Ausdruck als Objektstruktur auf;
    
4. ruft `Interpret()` auf.
    

---

# Allgemeine Struktur in C#

```csharp
// Enthält Informationen,
// die für die Interpretation benötigt werden.
public class Context
{
}


// Gemeinsamer Vertrag für alle Expressions.
public interface IExpression
{
    void Interpret(Context context);
}


// Terminaler Ausdruck.
// Repräsentiert einen elementaren Bestandteil der Grammatik.
public class TerminalExpression : IExpression
{
    public void Interpret(Context context)
    {
        // Terminal auswerten.
    }
}


// Nicht-terminaler Ausdruck.
// Kombiniert mehrere andere Expressions.
public class NonterminalExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public NonterminalExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public void Interpret(Context context)
    {
        // Untergeordnete Expressions interpretieren.
        _left.Interpret(context);
        _right.Interpret(context);
    }
}
```

---

# Terminal und Nonterminal einfach erklärt

Das ist einer der wichtigsten Punkte beim Interpreter Pattern.

## Terminal

Ein Terminal ist ein Ausdruck, der nicht weiter zerlegt wird.

Zum Beispiel:

```text
x
```

oder:

```text
5
```

---

## Nonterminal

Ein Nonterminal besteht aus anderen Ausdrücken.

Zum Beispiel:

```text
x + y
```

Die Addition besitzt:

```text
left  → x
right → y
```

Also:

```text
      +
     / \
    x   y
```

`x` und `y` sind Terminal-Ausdrücke.

`+` ist ein Nonterminal-Ausdruck.

---

# Praktisches Beispiel: mathematische Ausdrücke

Wir möchten folgende Ausdrücke interpretieren:

```text
x + y - z
```

Mit:

```text
x = 5
y = 8
z = 2
```

Erwartetes Ergebnis:

```text
5 + 8 - 2 = 11
```

---

# Vereinfachte Grammatik

Unsere kleine Sprache könnte ungefähr so beschrieben werden:

```text
Expression ::=
    VariableExpression
    | ConstantExpression
    | AddExpression
    | SubtractExpression
```

Addition:

```text
AddExpression ::=
    Expression + Expression
```

Subtraktion:

```text
SubtractExpression ::=
    Expression - Expression
```

Variable:

```text
VariableExpression ::=
    x | y | z | ...
```

Konstante:

```text
ConstantExpression ::=
    0 | 1 | 2 | 3 | ...
```

---

# Expression Interface

```csharp
// Gemeinsamer Vertrag für alle Ausdrücke.
public interface IExpression
{
    // Wertet den Ausdruck anhand des Context aus.
    int Interpret(Context context);
}
```

---

# Context

Der Context speichert die Werte unserer Variablen.

```csharp
public class Context
{
    // Speichert Variablen und ihre aktuellen Werte.
    private readonly Dictionary<string, int> _variables =
        new();

    // Gibt den Wert einer Variable zurück.
    public int GetVariable(string name)
    {
        return _variables[name];
    }

    // Fügt eine Variable hinzu
    // oder aktualisiert ihren Wert.
    public void SetVariable(
        string name,
        int value)
    {
        _variables[name] = value;
    }
}
```

---

# Terminal Expression: Variable

Eine einzelne Variable ist ein terminaler Ausdruck.

```csharp
public class VariableExpression : IExpression
{
    private readonly string _name;

    public VariableExpression(string name)
    {
        _name = name;
    }

    public int Interpret(Context context)
    {
        // Holt den aktuellen Wert der Variable
        // aus dem Context.
        return context.GetVariable(_name);
    }
}
```

Beispiel:

```csharp
IExpression x =
    new VariableExpression("x");
```

Bei:

```text
x = 5
```

liefert:

```csharp
x.Interpret(context);
```

den Wert:

```text
5
```

---

# Terminal Expression: Konstante

Auch feste Zahlen können als TerminalExpression modelliert werden.

```csharp
public class ConstantExpression : IExpression
{
    private readonly int _value;

    public ConstantExpression(int value)
    {
        _value = value;
    }

    public int Interpret(Context context)
    {
        // Eine Konstante benötigt den Context nicht.
        return _value;
    }
}
```

Beispiel:

```csharp
IExpression five =
    new ConstantExpression(5);
```

---

# Nonterminal Expression: Addition

```csharp
public class AddExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public AddExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        // Beide Teilausdrücke werden zunächst
        // rekursiv ausgewertet.
        int leftValue =
            _left.Interpret(context);

        int rightValue =
            _right.Interpret(context);

        // Anschließend werden die Ergebnisse addiert.
        return leftValue + rightValue;
    }
}
```

---

# Nonterminal Expression: Subtraktion

```csharp
public class SubtractExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public SubtractExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        int leftValue =
            _left.Interpret(context);

        int rightValue =
            _right.Interpret(context);

        return leftValue - rightValue;
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
        Context context =
            new Context();

        // Werte der Variablen definieren.
        context.SetVariable("x", 5);
        context.SetVariable("y", 8);
        context.SetVariable("z", 2);

        // Ausdruck erstellen:
        //
        // x + y - z
        IExpression expression =
            new SubtractExpression(
                new AddExpression(
                    new VariableExpression("x"),
                    new VariableExpression("y")),
                new VariableExpression("z"));

        int result =
            expression.Interpret(context);

        Console.WriteLine(
            $"Ergebnis: {result}");
    }
}
```

Ausgabe:

```text
Ergebnis: 11
```

---

# Was passiert hier genau?

Dieser Code:

```csharp
new AddExpression(
    new VariableExpression("x"),
    new VariableExpression("y"))
```

repräsentiert:

```text
x + y
```

Dann wird dieses Ergebnis in:

```csharp
new SubtractExpression(
    ...,
    new VariableExpression("z"))
```

eingebaut.

Dadurch entsteht:

```text
(x + y) - z
```

---

# Abstract Syntax Tree

Der Ausdruck wird intern als Baum dargestellt.

Das nennt man:

**Abstract Syntax Tree (AST)**

Für:

```text
x + y - z
```

entsteht:

```text
          -
         / \
        +   z
       / \
      x   y
```

Genauer:

```text
SubtractExpression
│
├── Left:
│   AddExpression
│   ├── VariableExpression("x")
│   └── VariableExpression("y")
│
└── Right:
    VariableExpression("z")
```

---

# Wie wird der Baum ausgewertet?

Wir starten bei:

```text
SubtractExpression
```

Diese benötigt:

```text
linkes Ergebnis
-
rechtes Ergebnis
```

Also muss zunächst:

```text
AddExpression
```

ausgewertet werden.

---

## Schritt 1

```text
VariableExpression("x")
```

liest:

```text
x = 5
```

aus dem Context.

---

## Schritt 2

```text
VariableExpression("y")
```

liest:

```text
y = 8
```

---

## Schritt 3

`AddExpression` berechnet:

```text
5 + 8 = 13
```

---

## Schritt 4

```text
VariableExpression("z")
```

liest:

```text
z = 2
```

---

## Schritt 5

`SubtractExpression` berechnet:

```text
13 - 2 = 11
```

---

# Rekursive Interpretation

Der wichtige Mechanismus lautet:

```csharp
_left.Interpret(context);
_right.Interpret(context);
```

Jede Expression kann wiederum andere Expressions enthalten.

Dadurch funktioniert die Auswertung rekursiv.

Zum Beispiel:

```text
(x + y) - (z + 10)
```

könnte so dargestellt werden:

```text
            -
          /   \
         +     +
        / \   / \
       x   y z  10
```

---

# Beispiel für komplexeren Ausdruck

```csharp
IExpression expression =
    new SubtractExpression(
        new AddExpression(
            new VariableExpression("x"),
            new VariableExpression("y")),
        new AddExpression(
            new VariableExpression("z"),
            new ConstantExpression(10)));
```

Das entspricht:

```text
(x + y) - (z + 10)
```

---

# Neue Grammatikregel hinzufügen

Angenommen, wir möchten Multiplikation unterstützen:

```text
x * y
```

Dann erstellen wir einfach eine neue Expression:

```csharp
public class MultiplyExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public MultiplyExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        return
            _left.Interpret(context)
            *
            _right.Interpret(context);
    }
}
```

Danach:

```csharp
IExpression expression =
    new MultiplyExpression(
        new VariableExpression("x"),
        new VariableExpression("y"));
```

repräsentiert:

```text
x * y
```

---

# Division hinzufügen

Genauso:

```csharp
public class DivideExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public DivideExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        int divisor =
            _right.Interpret(context);

        if (divisor == 0)
        {
            throw new DivideByZeroException();
        }

        return
            _left.Interpret(context)
            /
            divisor;
    }
}
```

Das zeigt einen wichtigen Vorteil:

> Neue Regeln können häufig durch neue Expression-Klassen ergänzt werden.

---

# Interpreter und Open/Closed Principle

Angenommen, wir besitzen:

```text
AddExpression
SubtractExpression
VariableExpression
ConstantExpression
```

und möchten:

```text
MultiplyExpression
```

hinzufügen.

Dann müssen bestehende Expressions normalerweise nicht verändert werden.

Wir ergänzen lediglich eine neue Klasse.

Das unterstützt häufig das:

**Open/Closed Principle**

> Offen für Erweiterung, möglichst geschlossen für Änderung.

---

# Ein wichtiges Problem des ursprünglichen Beispiels

Im ursprünglichen Code hieß die Variable-Expression:

```csharp
NumberExpression
```

obwohl sie eigentlich:

```text
x
y
z
```

repräsentiert.

Ein Name wie:

```csharp
VariableExpression
```

ist deshalb verständlicher.

Eine echte Zahl wie:

```text
10
```

wird dagegen besser durch:

```csharp
ConstantExpression
```

dargestellt.

---

# Komplettes modernes Beispiel

```csharp
// Gemeinsamer Vertrag für alle Expressions.
public interface IExpression
{
    int Interpret(Context context);
}


// Enthält Variablen und deren Werte.
public class Context
{
    private readonly Dictionary<string, int> _variables =
        new();

    public void SetVariable(
        string name,
        int value)
    {
        _variables[name] = value;
    }

    public int GetVariable(string name)
    {
        if (!_variables.TryGetValue(
                name,
                out int value))
        {
            throw new InvalidOperationException(
                $"Variable '{name}' wurde nicht definiert.");
        }

        return value;
    }
}


// Terminal:
// repräsentiert eine Variable.
public class VariableExpression : IExpression
{
    private readonly string _name;

    public VariableExpression(string name)
    {
        _name = name;
    }

    public int Interpret(Context context)
    {
        return context.GetVariable(_name);
    }
}


// Terminal:
// repräsentiert einen festen Zahlenwert.
public class ConstantExpression : IExpression
{
    private readonly int _value;

    public ConstantExpression(int value)
    {
        _value = value;
    }

    public int Interpret(Context context)
    {
        return _value;
    }
}


// Nonterminal:
// Addition zweier Expressions.
public class AddExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public AddExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        return
            _left.Interpret(context)
            +
            _right.Interpret(context);
    }
}


// Nonterminal:
// Subtraktion zweier Expressions.
public class SubtractExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public SubtractExpression(
        IExpression left,
        IExpression right)
    {
        _left = left;
        _right = right;
    }

    public int Interpret(Context context)
    {
        return
            _left.Interpret(context)
            -
            _right.Interpret(context);
    }
}
```

Verwendung:

```csharp
public class Program
{
    public static void Main()
    {
        Context context =
            new Context();

        context.SetVariable("x", 5);
        context.SetVariable("y", 8);
        context.SetVariable("z", 2);

        // Repräsentiert:
        //
        // (x + y) - z
        IExpression expression =
            new SubtractExpression(
                new AddExpression(
                    new VariableExpression("x"),
                    new VariableExpression("y")),
                new VariableExpression("z"));

        int result =
            expression.Interpret(context);

        Console.WriteLine(
            $"Ergebnis: {result}");
    }
}
```

---

# Wichtig: Interpreter ist nicht automatisch ein Parser

In unserem Beispiel erstellen wir den Ausdruck manuell:

```csharp
new SubtractExpression(
    new AddExpression(...),
    ...)
```

Wir übergeben also nicht einfach:

```csharp
"x + y - z"
```

als String.

Das bedeutet:

> Der Beispielcode interpretiert bereits aufgebaute Expression-Objekte.

Wenn wir tatsächlich einen String:

```text
x + y - z
```

einlesen möchten, benötigen wir vorher zusätzlich einen **Parser**.

---

# Parser vs. Interpreter

## Parser

Der Parser nimmt Text:

```text
"x + y - z"
```

und erzeugt daraus beispielsweise:

```text
          -
         / \
        +   z
       / \
      x   y
```

also einen Syntaxbaum.

---

## Interpreter

Der Interpreter nimmt diesen Baum:

```text
          -
         / \
        +   z
       / \
      x   y
```

und berechnet:

```text
11
```

Vereinfacht:

```text
Text
 │
 ▼
Parser
 │
 ▼
AST
 │
 ▼
Interpreter
 │
 ▼
Ergebnis
```

---

# Interpreter vs. Strategy

## Strategy

Strategy kapselt verschiedene alternative Algorithmen.

Zum Beispiel:

```text
CompressionStrategy
├── Zip
├── GZip
└── Brotli
```

Der Client wählt einen Algorithmus.

---

## Interpreter

Interpreter bildet eine **Sprache beziehungsweise Grammatik** aus mehreren Expressions ab.

```text
Expression
├── Variable
├── Constant
├── Add
└── Subtract
```

Die Expressions werden miteinander kombiniert.

---

# Interpreter vs. Composite

Interpreter und Composite sehen strukturell oft sehr ähnlich aus.

Zum Beispiel:

```text
Expression
├── Terminal
└── Nonterminal
      ├── Expression
      └── Expression
```

Das ist gleichzeitig eine Baumstruktur.

Der Unterschied liegt in der Absicht.

## Composite

> Objekte zu einer Baumstruktur zusammensetzen.

## Interpreter

> Eine Grammatik beziehungsweise Sprache mit dieser Struktur auswerten.

Interpreter verwendet daher häufig eine Struktur, die dem Composite Pattern ähnelt.

---

# Vorteile

- Grammatikregeln werden klar in Klassen getrennt;
    
- neue einfache Regeln können leicht ergänzt werden;
    
- rekursive Ausdrücke lassen sich elegant darstellen;
    
- Syntaxbaum und Auswertungslogik passen natürlich zusammen;
    
- kann kleine DSLs sehr verständlich modellieren;
    
- unterstützt häufig das Open/Closed Principle.
    

---

# Nachteile

- für große Grammatiken entstehen sehr viele Klassen;
    
- komplexe Sprachen werden schnell unübersichtlich;
    
- Parsing ist nicht automatisch Bestandteil des Patterns;
    
- Operator-Prioritäten und Syntaxfehler können zusätzlichen Aufwand erzeugen;
    
- bei großen Expression-Bäumen kann die Performance relevant werden.
    

---

# Wann ist Interpreter sinnvoll?

Gut geeignet:

```text
kleine Grammatik
+
wenige Regeln
+
häufige Auswertung
+
baumartige Expressions
```

Zum Beispiel:

```text
age > 18 AND country == "DE"
```

oder:

```text
x + y - 10
```

oder:

```text
Admin AND Active
```

---

# Wann eher nicht?

Bei:

```text
vollständiger Programmiersprache
+
vielen hundert Grammatikregeln
+
komplexem Parsing
```

sollte man eher auf:

```text
Parser Generator
Compiler-Techniken
ANTLR
Roslyn
andere spezialisierte Parser
```

setzen.

---

# Vereinfachte Struktur

```text
                    IExpression
                         ▲
               ┌─────────┴─────────┐
               │                   │
           Terminal           Nonterminal
               │                   │
         ┌─────┴─────┐       ┌─────┴──────┐
         │           │       │            │
     Variable    Constant   Add       Subtract
```

---

# Unser Beispiel als Baum

```text
                 Subtract
                 /      \
              Add        z
             /   \
            x     y
```

Dabei:

```text
x, y, z
→ TerminalExpressions

Add, Subtract
→ NonterminalExpressions
```

---

# Das solltest du dir merken

Typische Struktur:

```csharp
public interface IExpression
{
    int Interpret(Context context);
}
```

Terminal:

```csharp
public class VariableExpression : IExpression
{
    public int Interpret(Context context)
    {
        return context.GetVariable(...);
    }
}
```

Nonterminal:

```csharp
public class AddExpression : IExpression
{
    private readonly IExpression _left;
    private readonly IExpression _right;

    public int Interpret(Context context)
    {
        return
            _left.Interpret(context)
            +
            _right.Interpret(context);
    }
}
```

---

# Merksatz

> **Interpreter stellt Regeln einer einfachen Sprache als Klassen dar und wertet daraus zusammengesetzte Ausdrücke aus.**

Noch einfacher:

```text
Terminal:
"Ich bin ein einzelner Wert."

Nonterminal:
"Ich kombiniere andere Ausdrücke."

Context:
"Ich liefere zusätzliche Informationen."

Interpreter:
"Ich werte den Ausdruck aus."
```

Oder ganz kurz:

```text
Interpreter
=
Grammatik
+
Expression-Baum
+
Interpret()
```

> [!summary] Zusammenfassung  
> Das **Interpreter Pattern** ist ein **Verhaltensmuster**.
> 
> Es bildet eine kleine Sprache beziehungsweise Grammatik durch Klassen ab.
> 
> Typischerweise gibt es:
> 
> ```text
> IExpression
> ├── TerminalExpression
> └── NonterminalExpression
> ```
> 
> Terminal-Ausdrücke repräsentieren elementare Bestandteile:
> 
> ```text
> x
> y
> 10
> ```
> 
> Nonterminal-Ausdrücke kombinieren andere Expressions:
> 
> ```text
> x + y
> x - y
> ```
> 
> Dadurch entsteht ein **Abstract Syntax Tree**:
> 
> ```text
>       -
>      / \
>     +   z
>    / \
>   x   y
> ```
> 
> Die Auswertung erfolgt rekursiv über:
> 
> ```csharp
> Interpret(context);
> ```
> 
> Wichtig: Das Interpreter Pattern ist besonders für **kleine Grammatiken** geeignet. Bei komplexen Sprachen sollte man spezialisierte Parser- oder Compiler-Techniken verwenden.
> 
> **Kurz gesagt:**  
> `Interpreter = Regeln einer Sprache als Objekte darstellen und Ausdrücke dieser Sprache auswerten.`