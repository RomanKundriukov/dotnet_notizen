Das **Flyweight Pattern** ist ein Strukturmuster, mit dem eine große Anzahl ähnlicher Objekte effizient gespeichert und verwendet werden kann, indem gemeinsame Daten **zwischen mehreren Objekten geteilt** werden.

Das Hauptziel ist normalerweise:

> [!info] Grundidee  
> **Speicherverbrauch reduzieren**, indem identische oder unveränderliche Daten nicht für jedes Objekt separat gespeichert werden.

Statt:

```
Objekt 1
├── gemeinsame Daten
└── individuelle Daten

Objekt 2
├── gemeinsame Daten
└── individuelle Daten

Objekt 3
├── gemeinsame Daten
└── individuelle Daten
```

werden gemeinsame Daten nur einmal gespeichert:

```
                  Flyweight
              gemeinsame Daten
                  ▲   ▲   ▲
                  │   │   │
               Obj1 Obj2 Obj3
                │    │    │
             eigene individuelle Daten
```

---

# Wann sollte man Flyweight verwenden?

Das Flyweight Pattern ist besonders sinnvoll, wenn:

- sehr viele ähnliche Objekte existieren;
- diese Objekte viel Speicher benötigen;
- ein großer Teil ihrer Daten identisch ist;
- der gemeinsame Zustand unveränderlich oder zumindest kontextunabhängig ist;
- individuelle Daten aus den Objekten ausgelagert werden können;
- durch Teilen gemeinsamer Objekte tatsächlich ein relevanter Speichergewinn entsteht.

Typische Situation:

```
sehr viele Objekte
+
viele identische Daten
+
wenige individuelle Daten
=
Flyweight kann sinnvoll sein
```

---

# Klassisches Beispiel: Text

Ein Text kann Millionen von Zeichen enthalten.

Zum Beispiel:

```
Hallo Welt
```

enthält einzelne Zeichen:

```
H
a
l
l
o
...
```

Man könnte theoretisch für jedes Zeichen ein vollständiges Objekt erzeugen:

```
CharacterObject
├── Character = 'l'
├── Font = Arial
├── GlyphData = ...
├── PositionX = 120
└── PositionY = 50
```

Bei Millionen Zeichen wäre das sehr teuer.

Viele Informationen sind aber identisch.

Zum Beispiel:

```
Zeichen 'A'
Font Arial
Schriftgröße 12
Glyph-Daten
```

könnten von sehr vielen Zeicheninstanzen gemeinsam verwendet werden.

Individuell ist möglicherweise nur:

```
Position
Zeile
Spalte
Farbe
```

---

# Wichtigstes Konzept: Intrinsic und Extrinsic State

Das Flyweight Pattern basiert auf der Trennung des Zustands in zwei Kategorien:

```
Intrinsic State
Extrinsic State
```

Diese Unterscheidung ist der wichtigste Punkt des gesamten Patterns.

---

# Intrinsic State

Der **Intrinsic State** ist der interne, gemeinsame Zustand eines Flyweights.

Er:

- hängt nicht vom aktuellen Verwendungskontext ab;
- kann zwischen mehreren Clients geteilt werden;
- wird normalerweise im Flyweight selbst gespeichert;
- sollte möglichst unveränderlich sein.

Beispiel eines Gebäudetyps:

```
Typ      = Plattenbau
Etagen   = 16
Material = Betonplatten
Grundriss = Typ A
```

Diese Informationen sind für alle Häuser desselben Typs gleich.

---

# Extrinsic State

Der **Extrinsic State** ist der externe, individuelle Zustand.

Er:

- hängt vom konkreten Kontext ab;
- unterscheidet sich zwischen einzelnen Verwendungen;
- wird nicht dauerhaft im gemeinsamen Flyweight gespeichert;
- wird häufig beim Methodenaufruf übergeben.

Zum Beispiel:

```
Haus 1:
Latitude  = 48.8
Longitude = 9.2

Haus 2:
Latitude  = 48.9
Longitude = 9.3
```

Beide Häuser können denselben Gebäudetyp verwenden.

---

# Intrinsic vs. Extrinsic State

```
Intrinsic State
→ gemeinsam
→ unabhängig vom Kontext
→ im Flyweight gespeichert


Extrinsic State
→ individuell
→ abhängig vom Kontext
→ von außen bereitgestellt
```

---

# Beispiel

Angenommen, wir haben 1000 identische Häuser.

Ohne Flyweight:

```
House 1
├── Floors = 16
├── Material = Panel
├── Architecture = Type A
├── Latitude
└── Longitude

House 2
├── Floors = 16
├── Material = Panel
├── Architecture = Type A
├── Latitude
└── Longitude

...

House 1000
```

Die gemeinsamen Daten werden 1000-mal gespeichert.

---

# Mit Flyweight

```
PanelHouseFlyweight
├── Floors = 16
├── Material = Panel
└── Architecture = Type A
```

wird nur einmal erstellt.

Die einzelnen Häuser speichern nur noch:

```
House 1
├── Flyweight → PanelHouseFlyweight
├── Latitude
└── Longitude

House 2
├── Flyweight → PanelHouseFlyweight
├── Latitude
└── Longitude
```

---

# Grundstruktur

```
                         Flyweight
                            ▲
               ┌────────────┴────────────┐
               │                         │
     ConcreteFlyweight       UnsharedConcreteFlyweight
               ▲
               │
        FlyweightFactory
               │
               ▼
          Flyweight-Pool
```

Der Client fragt die Factory nach einem Flyweight:

```
Client
  │
  │ GetFlyweight(key)
  ▼
FlyweightFactory
  │
  ├── vorhanden?
  │      │
  │      ├── Ja → vorhandenes Objekt zurückgeben
  │      │
  │      └── Nein → erzeugen und speichern
  │
  ▼
Flyweight
```

---

# Teilnehmer des Patterns

## Flyweight

`Flyweight` definiert die gemeinsame Schnittstelle für geteilte Objekte.

Beispiel:

```
public interface IFlyweight
{
    void Operation(ExtrinsicState state);
}
```

Der externe Zustand wird häufig als Parameter übergeben.

---

# ConcreteFlyweight

Der `ConcreteFlyweight` enthält den **intrinsischen Zustand**.

Zum Beispiel:

```
public sealed class ConcreteFlyweight : IFlyweight
{
    private readonly string _intrinsicState;

    public ConcreteFlyweight(string intrinsicState)
    {
        _intrinsicState = intrinsicState;
    }

    public void Operation(ExtrinsicState state)
    {
        // Der interne Zustand gehört zum Flyweight.
        //
        // Der externe Zustand wird vom Client
        // für die aktuelle Verwendung übergeben.
    }
}
```

---

# FlyweightFactory

Die Factory verwaltet bereits erzeugte Flyweights.

Typisch ist beispielsweise:

```
Dictionary<TKey, TFlyweight>
```

Sie sorgt dafür, dass ein identisches Flyweight nicht immer wieder neu erzeugt wird.

---

# Client

Der Client:

- fordert Flyweights von der Factory an;
- speichert oder berechnet den extrinsischen Zustand;
- übergibt diesen Zustand beim Verwenden des Flyweights.

---

# Formales modernes Beispiel

```
public interface IFlyweight
{
    void Operation(int extrinsicState);
}
```

ConcreteFlyweight:

```
public sealed class ConcreteFlyweight : IFlyweight
{
    private readonly string _intrinsicState;

    public ConcreteFlyweight(string intrinsicState)
    {
        _intrinsicState = intrinsicState;
    }

    public void Operation(int extrinsicState)
    {
        Console.WriteLine(
            $"Intrinsic: {_intrinsicState}, " +
            $"Extrinsic: {extrinsicState}");
    }
}
```

---

# Factory

```
public sealed class FlyweightFactory
{
    // Bereits erzeugte Flyweights werden
    // anhand eines Schlüssels wiederverwendet.
    private readonly Dictionary<string, IFlyweight> _flyweights =
        new();

    public IFlyweight GetFlyweight(string key)
    {
        if (!_flyweights.TryGetValue(
                key,
                out IFlyweight? flyweight))
        {
            // Nur wenn noch kein passendes
            // Flyweight existiert, wird eines erzeugt.
            flyweight =
                new ConcreteFlyweight(key);

            _flyweights[key] =
                flyweight;
        }

        return flyweight;
    }
}
```

---

# Verwendung

```
FlyweightFactory factory =
    new FlyweightFactory();

IFlyweight flyweightA1 =
    factory.GetFlyweight("A");

IFlyweight flyweightA2 =
    factory.GetFlyweight("A");

IFlyweight flyweightB =
    factory.GetFlyweight("B");
```

Hier gilt:

```
flyweightA1
und
flyweightA2

→ dasselbe gemeinsame Flyweight
```

während:

```
flyweightB
```

ein anderes Objekt ist.

---

# Referenz prüfen

```
Console.WriteLine(
    ReferenceEquals(
        flyweightA1,
        flyweightA2));
```

Ergebnis:

```
True
```

Die Factory gibt also dasselbe Objekt zurück.

---

# Die Factory ist sehr wichtig

Ohne Factory könnte ein Client schreiben:

```
new ConcreteFlyweight("A");
new ConcreteFlyweight("A");
new ConcreteFlyweight("A");
```

Dann würden wieder mehrere identische Objekte entstehen.

Die FlyweightFactory verhindert das:

```
"A"
 │
 ▼
ein einziges Flyweight
```

---

# Flyweight-Pool

Die Factory verwaltet praktisch einen Pool:

```
Dictionary
│
├── "A" → Flyweight A
├── "B" → Flyweight B
├── "C" → Flyweight C
└── ...
```

Beim Zugriff:

```
factory.GetFlyweight("A");
```

wird zuerst geprüft:

```
Existiert "A" bereits?
```

Wenn ja:

```
bestehendes Objekt zurückgeben
```

Wenn nein:

```
neues Objekt erzeugen
+
im Pool speichern
```

---

# Praktisches Beispiel: Stadt mit Häusern

Angenommen, wir simulieren eine große Stadt.

Die Stadt enthält:

```
100.000 Häuser
```

Es gibt aber nur wenige Gebäudetypen:

```
Plattenbau
Backsteinhaus
Hochhaus
Einfamilienhaus
```

Viele Häuser basieren auf demselben Bauplan.

---

# Intrinsic State des Hauses

Gemeinsam für alle Häuser eines bestimmten Typs:

```
Etagenzahl
Baumaterial
Bauplan
Dachtyp
Grundriss
```

Diese Informationen können im Flyweight liegen.

---

# Extrinsic State

Individuell für jedes konkrete Haus:

```
Latitude
Longitude
Hausnummer
Farbe
Grundstück
```

Diese Informationen gehören nicht in das gemeinsame Flyweight.

---

# HouseType als Flyweight

Eine moderne Modellierung wäre:

```
public sealed class HouseType
{
    public string Name { get; }

    public int Floors { get; }

    public string Material { get; }

    public HouseType(
        string name,
        int floors,
        string material)
    {
        Name = name;
        Floors = floors;
        Material = material;
    }

    public void Build(
        double latitude,
        double longitude)
    {
        Console.WriteLine(
            $"{Name} mit {Floors} Etagen " +
            $"aus {Material} wird gebaut. " +
            $"Position: {latitude}, {longitude}");
    }
}
```

Hier sind:

```
Name
Floors
Material
```

der **intrinsische Zustand**.

---

# Extrinsischer Zustand

Beim Aufruf:

```
houseType.Build(
    latitude,
    longitude);
```

sind:

```
latitude
longitude
```

extrinsischer Zustand.

Sie werden nicht dauerhaft im `HouseType` gespeichert.

---

# HouseTypeFactory

```
public sealed class HouseTypeFactory
{
    private readonly Dictionary<string, HouseType> _houseTypes =
        new();

    public HouseType GetOrCreate(
        string key,
        Func<HouseType> factory)
    {
        if (!_houseTypes.TryGetValue(
                key,
                out HouseType? houseType))
        {
            // Der Gebäudetyp wird nur einmal erzeugt.
            houseType =
                factory();

            _houseTypes[key] =
                houseType;
        }

        return houseType;
    }
}
```

---

# Gebäudetypen erzeugen

```
HouseTypeFactory factory =
    new HouseTypeFactory();

HouseType panelHouse =
    factory.GetOrCreate(
        "Panel",
        () => new HouseType(
            "Plattenbau",
            16,
            "Betonplatten"));

HouseType brickHouse =
    factory.GetOrCreate(
        "Brick",
        () => new HouseType(
            "Backsteinhaus",
            5,
            "Ziegel"));
```

Es existieren nur:

```
2 HouseType-Objekte
```

obwohl später möglicherweise tausende Häuser gebaut werden.

---

# Viele Häuser bauen

```
double latitude = 55.74;
double longitude = 37.61;

for (int i = 0; i < 5; i++)
{
    HouseType type =
        factory.GetOrCreate(
            "Panel",
            () => new HouseType(
                "Plattenbau",
                16,
                "Betonplatten"));

    type.Build(
        latitude,
        longitude);

    latitude += 0.1;
    longitude += 0.1;
}
```

Obwohl fünf Häuser gebaut werden:

```
Haus 1
Haus 2
Haus 3
Haus 4
Haus 5
```

existiert weiterhin nur **ein gemeinsames `HouseType`-Objekt** für den Plattenbau.

---

# Darstellung

```
                    PanelHouseType
                Floors = 16
                Material = Panel
                    ▲
          ┌─────────┼─────────┐
          │         │         │
        Haus1     Haus2     Haus3
          │         │         │
       Position  Position  Position
```

---

# Noch sauberere Modellierung

In realen Anwendungen würde man häufig zwischen:

```
House
```

und:

```
HouseType
```

unterscheiden.

`HouseType` ist der Flyweight.

`House` enthält den extrinsischen Zustand.

---

# House

```
public sealed class House
{
    private readonly HouseType _type;

    public double Latitude { get; }

    public double Longitude { get; }

    public House(
        HouseType type,
        double latitude,
        double longitude)
    {
        _type = type;
        Latitude = latitude;
        Longitude = longitude;
    }

    public void Build()
    {
        // Der individuelle Zustand des Hauses
        // wird an das gemeinsame Flyweight übergeben.
        _type.Build(
            Latitude,
            Longitude);
    }
}
```

---

# Struktur

```
House
├── Latitude
├── Longitude
└── HouseType ───────┐
                     │
House                 │
├── Latitude          │
├── Longitude         │
└── HouseType ────────┤
                     │
House                 │
├── Latitude          │
├── Longitude         │
└── HouseType ────────┘
                     │
                     ▼
              gemeinsames HouseType
```

Das entspricht dem Flyweight-Gedanken noch deutlicher.

---

# Beispiel mit 1000 Häusern

```
HouseType panelType =
    factory.GetOrCreate(
        "Panel",
        () => new HouseType(
            "Plattenbau",
            16,
            "Betonplatten"));

List<House> houses =
    new();

for (int i = 0; i < 1000; i++)
{
    houses.Add(
        new House(
            panelType,
            latitude: 50 + i * 0.001,
            longitude: 9 + i * 0.001));
}
```

Es existieren:

```
1000 House-Objekte
```

aber nur:

```
1 HouseType-Objekt
```

für die gemeinsamen schweren Daten.

---

# Warum spart das Speicher?

Angenommen, der Bauplan eines Gebäudetyps benötigt:

```
5 MB
```

und wir haben:

```
10.000 identische Häuser
```

Ohne Flyweight wäre das theoretisch:

```
10.000 × 5 MB
=
50.000 MB
≈ 50 GB
```

nur für denselben Bauplan.

Mit Flyweight:

```
1 × 5 MB
+
kleine individuelle Daten pro Haus
```

Das kann einen enormen Unterschied machen.

---

# Ein besseres reales Beispiel: Spiele

Flyweight ist besonders interessant bei Spielen.

Angenommen, ein Wald besitzt:

```
1.000.000 Bäume
```

Jeder Baum besitzt:

```
Position
Größe
Alter
```

aber viele Bäume derselben Art teilen:

```
Textur
3D-Modell
Material
Shader
Baumart
```

---

# Ohne Flyweight

```
Tree 1
├── Position
├── Model
├── Texture
└── Material

Tree 2
├── Position
├── Model
├── Texture
└── Material

Tree 3
├── Position
├── Model
├── Texture
└── Material
```

Das 3D-Modell und die Textur würden ständig dupliziert.

---

# Mit Flyweight

```
TreeType
├── Model
├── Texture
└── Material
```

wird geteilt.

Einzelne Bäume:

```
Tree
├── X
├── Y
├── Z
└── TreeType
```

---

# C# Beispiel: TreeType

```
public sealed class TreeType
{
    public string Species { get; }

    public string Texture { get; }

    public string Model { get; }

    public TreeType(
        string species,
        string texture,
        string model)
    {
        Species = species;
        Texture = texture;
        Model = model;
    }

    public void Render(
        double x,
        double y,
        double z)
    {
        Console.WriteLine(
            $"{Species} wird an " +
            $"({x}, {y}, {z}) dargestellt.");
    }
}
```

---

# Tree

```
public sealed class Tree
{
    private readonly TreeType _type;

    private readonly double _x;
    private readonly double _y;
    private readonly double _z;

    public Tree(
        double x,
        double y,
        double z,
        TreeType type)
    {
        _x = x;
        _y = y;
        _z = z;
        _type = type;
    }

    public void Render()
    {
        _type.Render(
            _x,
            _y,
            _z);
    }
}
```

---

# TreeFactory

```
public sealed class TreeFactory
{
    private readonly Dictionary<string, TreeType> _types =
        new();

    public TreeType GetTreeType(
        string species,
        string texture,
        string model)
    {
        string key =
            $"{species}|{texture}|{model}";

        if (!_types.TryGetValue(
                key,
                out TreeType? type))
        {
            // Schwere gemeinsame Daten werden
            // nur einmal pro Baumtyp erzeugt.
            type =
                new TreeType(
                    species,
                    texture,
                    model);

            _types[key] =
                type;
        }

        return type;
    }
}
```

---

# Verwendung

```
TreeFactory factory =
    new TreeFactory();

TreeType oakType =
    factory.GetTreeType(
        "Eiche",
        "oak.png",
        "oak.obj");

Tree tree1 =
    new Tree(
        10,
        20,
        0,
        oakType);

Tree tree2 =
    new Tree(
        50,
        70,
        0,
        oakType);

Tree tree3 =
    new Tree(
        80,
        15,
        0,
        oakType);
```

Alle drei Bäume verwenden dasselbe:

```
oakType
```

---

# Intrinsic und Extrinsic beim Baum

## Intrinsic

```
Species
Texture
Model
```

Diese Daten befinden sich in:

```
TreeType
```

und werden geteilt.

---

## Extrinsic

```
X
Y
Z
```

Diese Daten gehören zu jedem einzelnen Baum.

---

# Wichtig: Flyweight sollte möglichst immutable sein

Da ein Flyweight von mehreren Objekten geteilt wird, ist veränderlicher Zustand gefährlich.

Angenommen:

```
Tree 1 ──┐
Tree 2 ──┼──► TreeType
Tree 3 ──┘
```

und `Tree 1` verändert:

```
treeType.Texture = "red.png";
```

Dann sehen plötzlich auch:

```
Tree 2
Tree 3
```

die neue Textur.

Deshalb sollte gemeinsamer Flyweight-Zustand möglichst unveränderlich sein.

---

# Gute Variante

```
public sealed class TreeType
{
    public string Species { get; }

    public string Texture { get; }

    public string Model { get; }

    // ...
}
```

Nicht:

```
public string Texture { get; set; }
```

wenn diese Information wirklich gemeinsamer intrinsischer Zustand ist.

---

# `record` als Flyweight

Für reine Daten kann auch ein unveränderliches `record` interessant sein:

```
public sealed record HouseType(
    string Name,
    int Floors,
    string Material);
```

Damit lässt sich gemeinsamer Zustand kompakt modellieren.

---

# FlyweightFactory und moderne Dictionaries

Die ursprüngliche Quelle verwendet:

```
Hashtable
```

In modernem C# würde man meistens generische Collections verwenden:

```
Dictionary<string, Flyweight>
```

Vorteile:

```
Typsicherheit
kein Casting
klarere API
bessere Lesbarkeit
```

---

# Alt

```
Hashtable flyweights =
    new Hashtable();
```

Modern:

```
Dictionary<string, IFlyweight> flyweights =
    new();
```

---

# Factory kann Lazy Creation verwenden

Es ist nicht notwendig, alle Flyweights beim Start zu erzeugen.

Besser kann sein:

```
public HouseType GetHouseType(string key)
{
    if (!_types.TryGetValue(
            key,
            out HouseType? type))
    {
        type =
            CreateType(key);

        _types[key] =
            type;
    }

    return type;
}
```

Dadurch wird ein Flyweight erst erzeugt, wenn er tatsächlich gebraucht wird.

---

# Factory = Factory + Cache

Eine FlyweightFactory kombiniert im Grunde zwei Aufgaben:

```
Objekt erzeugen
+
bereits erzeugte Objekte wiederverwenden
```

Vereinfacht:

```
Get("Panel")
     │
     ├── vorhanden → zurückgeben
     │
     └── nicht vorhanden
              │
              ▼
            Create
              │
              ▼
            Cache
              │
              ▼
           zurückgeben
```

---

# Ist jeder Cache ein Flyweight?

Nein.

Ein Cache kann beliebige Ergebnisse speichern:

```
HTTP Response
Datenbankabfrage
Berechnung
Bild
```

Flyweight hat eine speziellere Absicht:

> Viele logische Objekte sollen **denselben gemeinsamen Objektzustand teilen**, um Ressourcen zu sparen.

---

# Flyweight vs. Cache

```
Cache
→ teure Daten nicht erneut laden/berechnen


Flyweight
→ gemeinsamen Zustand vieler Objekte teilen
```

Eine FlyweightFactory verwendet intern häufig einen Cache beziehungsweise Pool.

Aber das macht nicht jeden Cache automatisch zum Flyweight Pattern.

---

# Flyweight vs. Singleton

Auch diese Patterns teilen Objekte, aber auf unterschiedliche Weise.

## Singleton

```
eine Klasse
→ genau eine Instanz
```

Beispiel:

```
ApplicationConfiguration
```

---

## Flyweight

```
viele logische Objekte
→ kleine Menge gemeinsam genutzter Instanzen
```

Beispiel:

```
1.000.000 Bäume
→ vielleicht 10 TreeType-Objekte
```

---

# Kurzvergleich

```
Singleton
→ eine globale Instanz


Flyweight
→ mehrere geteilte Instanzen,
  jeweils für bestimmte gemeinsame Zustände
```

---

# Flyweight vs. Prototype

## Prototype

Prototype erzeugt neue Objekte durch Kopieren:

```
Prototype
   │
   ▼
Clone
   │
   ▼
neues Objekt
```

---

## Flyweight

Flyweight versucht gerade, **weniger neue Objekte zu erzeugen**:

```
Client A ─┐
Client B ─┼──► gemeinsames Flyweight
Client C ─┘
```

---

# Kurzvergleich

```
Prototype
→ Objekt kopieren


Flyweight
→ Objekt teilen
```

Fast gegensätzliche Ziele.

---

# Flyweight vs. Object Pool

Auch ein Object Pool wiederverwendet Objekte.

Der Zweck unterscheidet sich aber.

## Object Pool

Objekte werden:

```
ausleihen
verwenden
zurückgeben
```

Beispiel:

```
Database Connection Pool
```

Ein Objekt wird normalerweise nicht gleichzeitig von allen Clients gemeinsam verwendet.

---

## Flyweight

Ein Flyweight kann von vielen Clients gleichzeitig geteilt werden:

```
Client A ─┐
Client B ─┼──► Flyweight
Client C ─┘
```

---

# Kurzvergleich

```
Object Pool
→ teure Objekte wiederverwenden


Flyweight
→ gemeinsamen unveränderlichen Zustand teilen
```

---

# Flyweight vs. Proxy

## Proxy

```
Client
  ↓
Proxy
  ↓
RealSubject
```

Ziel:

```
Zugriff kontrollieren
```

---

## Flyweight

```
viele Clients
    ↓
gemeinsames Objekt
```

Ziel:

```
Speicher sparen
```

---

# Flyweight vs. Composite

Flyweight und Composite können zusammen eingesetzt werden.

Ein Composite könnte beispielsweise eine große grafische Baumstruktur darstellen:

```
Scene
├── Tree
├── Tree
├── House
└── Tree
```

und jedes `Tree`-Objekt könnte intern ein gemeinsames:

```
TreeType Flyweight
```

verwenden.

Also:

```
Composite
→ Struktur organisieren


Flyweight
→ gemeinsame Daten effizient teilen
```

---

# Flyweight vs. Factory Method

Die FlyweightFactory ähnelt zunächst einer Factory.

Aber der Zweck ist unterschiedlich.

## Factory Method

```
Welches konkrete Objekt
soll erzeugt werden?
```

## FlyweightFactory

```
Existiert dieses gemeinsame Objekt bereits?

Ja
→ wiederverwenden

Nein
→ erzeugen und speichern
```

---

# Ein wichtiger Unterschied

Bei normaler Objekt-Erzeugung:

```
new HouseType(...);
```

erhalten wir jedes Mal eine neue Instanz.

Bei Flyweight:

```
factory.GetHouseType(...);
```

kann dieselbe Instanz zurückgegeben werden.

---

# Interner Zustand muss korrekt gewählt werden

Die größte Designaufgabe beim Flyweight Pattern ist normalerweise nicht die Factory.

Es ist die Frage:

> **Welche Daten können wirklich geteilt werden?**

Angenommen:

```
HouseType
├── Floors
├── Material
├── Color
└── Position
```

Sind `Color` und `Position` wirklich für alle Häuser gleich?

Wenn nein, gehören sie nicht in den Flyweight.

---

# Falsch

```
public class HouseType
{
    public int Floors { get; }

    public string Material { get; }

    public double Latitude { get; set; }

    public double Longitude { get; set; }
}
```

Wenn dasselbe `HouseType` geteilt wird, würden alle Häuser dieselben Koordinaten verwenden.

---

# Richtig

```
public class HouseType
{
    public int Floors { get; }

    public string Material { get; }

    public void Build(
        double latitude,
        double longitude)
    {
        // Individuelle Daten werden
        // nur für diese Verwendung übergeben.
    }
}
```

---

# Flyweight lohnt sich nicht automatisch

Zusätzliche Abstraktion bringt ebenfalls Kosten.

Die Factory benötigt beispielsweise:

```
Dictionary
Lookup
Schlüsselverwaltung
zusätzliche Klassen
```

Wenn wir nur:

```
10 kleine Objekte
```

haben, spart Flyweight praktisch nichts und macht den Code nur komplizierter.

---

# Wann lohnt es sich?

Besonders bei Größenordnungen wie:

```
100.000 Objekte
1.000.000 Objekte
10.000.000 Objekte
```

und einem großen gemeinsamen Zustand.

---

# Beispielrechnung

Angenommen:

```
jedes Objekt:
10 KB gemeinsamer Zustand
+
24 Byte individueller Zustand
```

bei:

```
100.000 Objekten
```

Ohne Sharing ungefähr:

```
100.000 × 10 KB
≈ 1 GB
```

Mit Flyweight und beispielsweise zehn unterschiedlichen Typen:

```
10 × 10 KB
+
100.000 × kleine individuelle Daten
```

Der Unterschied kann enorm sein.

---

# Nachteile

Der Speichergewinn hat einen Preis.

Flyweight kann:

- das Design komplizierter machen;
- intrinsischen und extrinsischen Zustand schwerer nachvollziehbar machen;
- mehr Parameter bei Methoden erfordern;
- Factory- und Lookup-Logik notwendig machen;
- Debugging erschweren;
- bei falscher Verwendung zu gemeinsam verändertem Zustand führen.

---

# Suche im Flyweight-Pool

Die ursprüngliche Quelle erwähnt einen möglichen Nachteil:

Wenn sehr viele Flyweights existieren, muss die Factory das passende Objekt finden.

Bei modernen:

```
Dictionary<TKey, TValue>
```

ist der Zugriff typischerweise sehr effizient.

Trotzdem entstehen zusätzliche:

```
Lookup-Kosten
Speicher für Dictionary
Schlüsselverwaltung
```

Deshalb muss der Speichergewinn relevant genug sein.

---

# Thread Safety

Wenn eine FlyweightFactory von mehreren Threads verwendet wird, muss gegebenenfalls darauf geachtet werden, dass nicht gleichzeitig mehrere identische Flyweights erzeugt werden.

Zum Beispiel könnten zwei Threads gleichzeitig feststellen:

```
Key existiert nicht
```

und beide ein Objekt erzeugen.

Je nach Anwendung kann man beispielsweise:

```
ConcurrentDictionary<TKey, TValue>
```

verwenden.

---

# Beispiel

```
public sealed class FlyweightFactory
{
    private readonly ConcurrentDictionary<string, HouseType> _types =
        new();

    public HouseType GetOrCreate(
        string key,
        Func<string, HouseType> factory)
    {
        return _types.GetOrAdd(
            key,
            factory);
    }
}
```

> [!note]  
> Thread-Sicherheit gehört nicht direkt zum GoF-Pattern, ist aber bei gemeinsam verwendeten Flyweight-Factories praktisch relevant.

---

# Strings in .NET als verwandte Idee

Ein bekanntes Beispiel für das Teilen gleicher Daten ist **String Interning**.

Konzeptionell:

```
"Hello"
"Hello"
"Hello"
```

müssen unter bestimmten Bedingungen nicht zwangsläufig drei vollständig getrennte identische String-Daten repräsentieren.

Das ist dem Flyweight-Prinzip sehr ähnlich:

```
gleicher unveränderlicher Inhalt
→ gemeinsame Repräsentation
```

> [!note]  
> Man sollte String Interning und das GoF Flyweight Pattern nicht völlig gleichsetzen. Es ist aber ein gutes Beispiel für die zugrunde liegende Idee des Teilens unveränderlicher Daten.

---

# Typische reale Einsatzgebiete

Flyweight eignet sich besonders für:

```
Texteditoren

Schriftzeichen / Glyphen

Games

Partikelsysteme

Bäume und Vegetation

3D-Objekte

Karten und Map-Objekte

Icons

wiederverwendete Styles

grafische Elemente

große Simulationen
```

---

# Beispiel: Partikelsystem

Eine Explosion kann:

```
100.000 Partikel
```

erzeugen.

Alle Partikel könnten dieselbe:

```
Textur
Shader
Materialdefinition
```

verwenden.

Individuell bleiben:

```
Position
Geschwindigkeit
Lebensdauer
Rotation
```

Also:

```
ParticleType
├── Texture
├── Shader
└── Material
```

geteilt.

Während:

```
Particle
├── X
├── Y
├── Velocity
├── Lifetime
└── ParticleType
```

für jedes Partikel existiert.

---

# Warum Flyweight ein Strukturmuster ist

Flyweight beschreibt nicht hauptsächlich:

```
wie ein Algorithmus ausgeführt wird
```

sondern:

```
wie Objekte strukturiert
und miteinander geteilt werden
```

Deshalb gehört es zu den:

**Strukturmustern**.

---

# Klassische Struktur

```
                       FlyweightFactory
                              │
                              ▼
                      Flyweight-Pool
                    /        |        \
                   /         |         \
                  ▼          ▼          ▼
            Flyweight A Flyweight B Flyweight C
                  ▲          ▲
                  │          │
             viele Clients teilen
             dieselben Objekte
```

---

# Moderne Struktur des Stadt-Beispiels

```
                         HouseTypeFactory
                                │
                                ▼
                  ┌─────────────┴─────────────┐
                  ▼                           ▼
            PanelHouseType              BrickHouseType
                  ▲                           ▲
          ┌───────┼───────┐            ┌─────┼─────┐
          │       │       │            │     │     │
       House1  House2  House3       House4 House5 House6
```

Dabei enthalten die einzelnen `House`-Objekte nur ihre individuellen Daten.

---

# Das solltest du dir merken

Flyweight:

```
public sealed class Flyweight
{
    // Gemeinsamer intrinsischer Zustand.
    public string SharedState { get; }

    public Flyweight(string sharedState)
    {
        SharedState = sharedState;
    }

    public void Operation(
        string uniqueState)
    {
        // SharedState:
        // wird zwischen Objekten geteilt.
        //
        // uniqueState:
        // gehört zum konkreten Kontext.
    }
}
```

Factory:

```
public sealed class FlyweightFactory
{
    private readonly Dictionary<string, Flyweight> _flyweights =
        new();

    public Flyweight GetFlyweight(string key)
    {
        if (!_flyweights.TryGetValue(
                key,
                out Flyweight? flyweight))
        {
            flyweight =
                new Flyweight(key);

            _flyweights[key] =
                flyweight;
        }

        return flyweight;
    }
}
```

Verwendung:

```
Flyweight a =
    factory.GetFlyweight("A");

Flyweight b =
    factory.GetFlyweight("A");
```

Dann:

```
ReferenceEquals(a, b);
```

liefert:

```
true
```

---

# Die wichtigste Frage beim Pattern

Wenn du Flyweight erkennst oder selbst einsetzen möchtest, frage:

> **Welche Daten sind für viele Objekte gleich und können deshalb einmal gespeichert und gemeinsam verwendet werden?**

Danach:

> **Welche Daten sind individuell und müssen außerhalb des Flyweights bleiben?**

---

# Entscheidungsregel

```
Habe ich sehr viele ähnliche Objekte?
              │
         ┌────┴────┐
        Nein       Ja
         │          │
         ▼          ▼
    kein Flyweight  Gibt es viel
                    gemeinsamen Zustand?
                         │
                    ┌────┴────┐
                   Nein       Ja
                    │          │
                    ▼          ▼
               eher kein    Kann dieser Zustand
                Flyweight   sicher geteilt werden?
                                  │
                             ┌────┴────┐
                            Nein       Ja
                             │          │
                             ▼          ▼
                        kein Flyweight Flyweight
                                      sinnvoll
```

---

# Merksatz

> **Flyweight reduziert den Speicherverbrauch, indem viele logische Objekte denselben gemeinsamen, kontextunabhängigen Zustand teilen.**

Noch einfacher:

```
Gemeinsame Daten
→ einmal speichern

Individuelle Daten
→ pro Objekt speichern
```

Oder:

```
Intrinsic State
→ im Flyweight


Extrinsic State
→ außerhalb des Flyweights
```

Noch kürzer:

```
Flyweight
=
Objekte teilen
+
gemeinsamen Zustand auslagern
+
Speicher sparen
```

---

# Kurzvergleich wichtiger Strukturmuster

```
Flyweight
→ gemeinsamen Zustand teilen


Proxy
→ Zugriff kontrollieren


Bridge
→ Abstraktion und Implementierung trennen


Composite
→ Teil-Ganzes-Baumstruktur


Decorator
→ Verhalten erweitern


Adapter
→ Schnittstelle anpassen


Facade
→ komplexes Subsystem vereinfachen
```

---

> [!summary] Zusammenfassung  
> Das **Flyweight Pattern** ist ein **Strukturmuster**, dessen Hauptziel die Reduzierung des Speicherverbrauchs bei einer sehr großen Anzahl ähnlicher Objekte ist.
> 
> Dafür wird der Zustand aufgeteilt:
> 
> ```
> Intrinsic State
> → gemeinsam
> → kontextunabhängig
> → wird im Flyweight gespeichert
> 
> 
> Extrinsic State
> → individuell
> → kontextabhängig
> → wird außerhalb gespeichert
> ```
> 
> Ein klassisches Beispiel sind tausende Häuser:
> 
> ```
> HouseType
> ├── Etagen
> ├── Material
> └── Bauplan
> ```
> 
> wird geteilt.
> 
> Dagegen besitzt jedes konkrete Haus:
> 
> ```
> Position
> Hausnummer
> individuelle Eigenschaften
> ```
> 
> selbst.
> 
> Die `FlyweightFactory` sorgt dafür, dass identische Flyweights nicht mehrfach erzeugt werden:
> 
> ```
> if (!_flyweights.TryGetValue(
>         key,
>         out Flyweight? flyweight))
> {
>     flyweight =
>         new Flyweight(key);
> 
>     _flyweights[key] =
>         flyweight;
> }
> 
> return flyweight;
> ```
> 
> Dadurch können beispielsweise:
> 
> ```
> 100.000 logische Objekte
> ```
> 
> nur:
> 
> ```
> 10 gemeinsame Flyweight-Objekte
> ```
> 
> für den schweren gemeinsamen Zustand verwenden.
> 
> Besonders wichtig ist, dass der gemeinsame Zustand möglichst **unveränderlich** sein sollte, da sonst eine Änderung alle Nutzer desselben Flyweights beeinflussen würde.
> 
> Der wichtigste Unterschied zu Singleton:
> 
> ```
> Singleton
> → genau eine Instanz einer Klasse
> 
> Flyweight
> → wenige gemeinsame Instanzen
>   für sehr viele logische Objekte
> ```
> 
> **Kurz gesagt:**  
> `Flyweight = Gleiche Daten nicht tausendmal speichern, sondern einmal erzeugen und gemeinsam verwenden.`