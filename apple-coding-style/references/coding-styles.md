# Swift API Coding Style (from Swift.org API Design Guidelines)

Source: https://www.swift.org/documentation/api-design-guidelines/

---

## Core principles
- **Clarity at the point of use** is the top priority. Always judge an API by how it reads at the call site.
- **Clarity > brevity**. Avoid shortening names if it harms understanding.
- **Document every declaration**. Writing docs often reveals design issues early.

---

## Documentation comments
- Write docs in **Swift-flavored Markdown**.
- Start with a **one-line summary** describing what it *does/is*, ideally a **sentence fragment** ending with a period.
- Summaries should begin with:
  - Functions/methods: **Verb** describing the effect (e.g., “Returns…”, “Inserts…”, “Removes and returns…”).
  - Subscripts: **“Accesses …”**
  - Initializers: **“Creates …”**
- Use structured sections when helpful: `- Parameters:`, `- Returns:`, `- Throws:`, `- Note:`, `- SeeAlso:` (Xcode recognizes many of these).

---

## Naming rules

### Promote clear usage
- Include the words needed to **avoid ambiguity** (e.g., `remove(at:)` not `remove(_)`).

For example, consider a method that removes the element at a given position within a collection.

```swift
extension List {
  public mutating func remove(at position: Index) -> Element
}
employees.remove(at: x)
```

If we were to omit the word at from the method signature, it could imply to the reader that the method searches for and removes an element equal to x, rather than using x to indicate the position of the element to remove.

- **Omit needless words**—especially ones that repeat type info (e.g., `remove(_:)` instead of `removeElement(_:)`).

More words may be needed to clarify intent or disambiguate meaning, but those that are redundant with information the reader already possesses should be omitted. In particular, omit words that merely repeat type information.

```swift
// Bad
public mutating func removeElement(_ member: Element) -> Element?

allViews.removeElement(cancelButton)
```

In this case, the word Element adds nothing salient at the call site. This API would be better:

```swift
public mutating func remove(_ member: Element) -> Element?

allViews.remove(cancelButton) // clearer
```

Occasionally, repeating type information is necessary to avoid ambiguity, but in general it is better to use a word that describes a parameter’s role rather than its type. See the next item for details.

- Name things by their **role**, not by their type (e.g., `greeting` not `string`).

```swift
// bad
var string = "Hello"
protocol ViewController {
  associatedtype ViewType : View
}
class ProductionLine {
  func restock(from widgetFactory: WidgetFactory)
}
```

Repurposing a type name in this way fails to optimize clarity and expressivity. Instead, strive to choose a name that expresses the entity’s role.

```swift
// Good
var greeting = "Hello"
protocol ViewController {
  associatedtype ContentView : View
}
class ProductionLine {
  func restock(from supplier: WidgetFactory)
}
```

If an associated type is so tightly bound to its protocol constraint that the protocol name is the role, avoid collision by appending Protocol to the protocol name:

```swift
protocol Sequence {
  associatedtype Iterator : IteratorProtocol
}
protocol IteratorProtocol { ... }
```

- If type info is weak (`Any`, `NSObject`, `Int`, `String`, etc.), add role words for clarity (e.g., `addObserver(_:forKeyPath:)`).

Especially when a parameter type is NSObject, Any, AnyObject, or a fundamental type such as Int or String, type information and context at the point of use may not fully convey intent. In this example, the declaration may be clear, but the use site is vague.

```swift
// Bad
func add(_ observer: NSObject, for keyPath: String)

grid.add(self, for: graphics) // vague
```

To restore clarity, precede each weakly typed parameter with a noun describing its role:

```swift
func addObserver(_ observer: NSObject, forKeyPath path: String)
grid.addObserver(self, forKeyPath: graphics) // clear
```


### Strive for fluent usage
- Prefer names that form **grammatical English** at the call site (e.g., `x.insert(y, at: z)`).

```swift
x.insert(y, at: z)          “x, insert y at z”
x.subviews(havingColor: y)  “x's subviews having color y”
x.capitalizingNouns()       “x, capitalizing nouns”
```

- Factory methods should start with **`make`** (e.g., `makeIterator()`).
- Don’t force grammatical flow with the *first argument* of initializers/factories—labels typically handle this.

For example, the first arguments to these calls do not read as part of the same phrase as the base name:

```swift
let foreground = Color(red: 32, green: 64, blue: 128)
let newPart = factory.makeWidget(gears: 42, spindles: 14)
let ref = Link(target: destination)
```

### Name by side effects
- **No side effects** → noun phrase (e.g., `distance(to:)`, `successor()`).
- **Has side effects** → imperative verb phrase (e.g., `sort()`, `append(_)`).
- Mutating/nonmutating pairs:
  - Mutating: verb (`sort()`, `reverse()`)
  - Nonmutating: “-ed” or “-ing” form (`sorted()`, `reversed()`, `appending(_)`).

Prefer to name the nonmutating variant using the verb’s past participle (usually appending “ed”):

```swift
/// Reverses `self` in-place.
mutating func reverse()

/// Returns a reversed copy of `self`.
func reversed() -> Self
...
x.reverse()
let y = x.reversed()
```

When adding “ed” is not grammatical because the verb has a direct object, name the nonmutating variant using the verb’s present participle, by appending “ing.”

```swift
/// Strips all the newlines from `self`
mutating func stripNewlines()

/// Returns a copy of `self` with all the newlines stripped.
func strippingNewlines() -> String
...
s.stripNewlines()
let oneLine = t.strippingNewlines()
```

When the operation is naturally described by a noun, use the noun for the nonmutating method and apply the “form” prefix to name its mutating counterpart.

| Nonmutating              | Mutating                    |
|--------------------------|-----------------------------|
| `x = y.union(z)`         | `y.formUnion(z)`            |
| `j = c.successor(i)`     | `c.formSuccessor(&i)`       |

- Uses of Boolean methods and properties should read as assertions about the receiver when the use is nonmutating, e.g. `x.isEmpty`, `line1.intersects(line2)`.
- Protocols that describe what something is should read as nouns (e.g. `Collection`).
- Protocols that describe a capability should be named using the suffixes able, ible, **or** ing (e.g. `Equatable`, `ProgressReporting`).
- The names of other types, properties, variables, and constants should read as nouns.

---

## Use Terminology Well

Term of Artnoun - a word or phrase that has a precise, specialized meaning within a particular field or profession.

- Avoid obscure terms if a more common word conveys meaning just as well. Don’t say “epidermis” if “skin” will serve your purpose. Terms of art are an essential communication tool, but should only be used to capture crucial meaning that would otherwise be lost.
- Stick to the established meaning if you do use a term of art.
    The only reason to use a technical term rather than a more common word is that it precisely expresses something that would otherwise be ambiguous or unclear. Therefore, an API should use the term strictly in accordance with its accepted meaning.

Don’t surprise an expert: anyone already familiar with the term will be surprised and probably angered if we appear to have invented a new meaning for it.

Don’t confuse a beginner: anyone trying to learn the term is likely to do a web search and find its traditional meaning.

- Avoid abbreviations. Abbreviations, especially non-standard ones, are effectively terms-of-art, because understanding depends on correctly translating them into their non-abbreviated forms.
- Embrace precedent. Don’t optimize terms for the total beginner at the expense of conformance to existing culture.
    It is better to name a contiguous data structure Array than to use a simplified term such as List, even though a beginner might grasp the meaning of List more easily. Arrays are fundamental in modern computing, so every programmer knows—or will soon learn—what an array is. Use a term that most programmers are familiar with, and their web searches and questions will be rewarded.

Within a particular programming domain, such as mathematics, a widely precedented term such as sin(x) is preferable to an explanatory phrase such as verticalPositionOnUnitCircleAtOriginOfEndOfRadiusWithAngle(x). Note that in this case, precedent outweighs the guideline to avoid abbreviations: although the complete word is sine, “sin(x)” has been in common use among programmers for decades, and among mathematicians for centuries.

---

## Conventions (practical defaults)
- Prefer methods and properties to free functions. Free functions are used only in special cases:

1. When there’s no obvious self: `min(x, y, z)`
2. When the function is an unconstrained generic: `print(x)`
3. When function syntax is part of the established domain notation: `sin(x)`
- Use **lowerCamelCase** for methods, properties, variables; **UpperCamelCase** for types.

```swift
var utf8Bytes: [UTF8.CodeUnit]
var isRepresentableAsASCII = true
var userSMTPServer: SecureSMTPServer
```

Other acronyms should be treated as ordinary words:

```swift
var radarDetector: RadarScanner
var enjoysScubaDiving = true
```

- Prefer **Swift-native types and patterns**; keep APIs “Swifty” (optionals, value semantics, generics, etc.).
- Keep naming consistent across the module.

Methods can share a base name when they share the same basic meaning or when they operate in distinct domains.
For example, the following is encouraged, since the methods do essentially the same things:

```swift
extension Shape {
  /// Returns `true` if `other` is within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: Point) -> Bool { ... }

  /// Returns `true` if `other` is entirely within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: Shape) -> Bool { ... }

  /// Returns `true` if `other` is within the area of `self`;
  /// otherwise, `false`.
  func contains(_ other: LineSegment) -> Bool { ... }
}
```

And since geometric types and collections are separate domains, this is also fine in the same program:

```swift
extension Collection where Element : Equatable {
  /// Returns `true` if `self` contains an element equal to
  /// `sought`; otherwise, `false`.
  func contains(_ sought: Element) -> Bool { ... }
}
```

Avoid “overloading on return type” because it causes ambiguities in the presence of type inference.

```swift
// Bad
extension Box {
  /// Returns the `Int` stored in `self`, if any, and
  /// `nil` otherwise.
  func value() -> Int? { ... }

  /// Returns the `String` stored in `self`, if any, and
  /// `nil` otherwise.
  func value() -> String? { ... }
}
```

---

## Parameters & argument labels (common rules of thumb)
- Use labels to make calls read clearly.
- Choose labels that express the **role** of the argument.
- Avoid redundant labels when the type or context already makes it obvious.

Choose parameter names to serve documentation. Even though parameter names do not appear at a function or method’s point of use, they play an important explanatory role.

Choose these names to make documentation easy to read. For example, these names make documentation read naturally:

```swift
/// Return an `Array` containing the elements of `self`
/// that satisfy `predicate`.
func filter(_ predicate: (Element) -> Bool) -> [Generator.Element]

/// Replace the given `subRange` of elements with `newElements`.
mutating func replaceRange(_ subRange: Range, with newElements: [E])
```

- Take advantage of defaulted parameters when it simplifies common uses. Any parameter with a single commonly-used value is a candidate for a default.

```swift
// bad
let order = lastName.compare(
royalFamilyName, options: [], range: nil, locale: nil)

// Good
let order = lastName.compare(royalFamilyName)
```

Default arguments are generally preferable to the use of method families, because they impose a lower cognitive burden on anyone trying to understand the API.

```swift
extension String {
  /// ...description...
  public func compare(
     _ other: String, options: CompareOptions = [],
     range: Range? = nil, locale: Locale? = nil
  ) -> Ordering
}
```

- Prefer to locate parameters with defaults toward the end of the parameter list. Parameters without defaults are usually more essential to the semantics of a method, and provide a stable initial pattern of use where methods are invoked.

- If your API will run in production, prefer #fileID over alternatives. #fileID saves space and protects developers’ privacy. Use `#filePath` in APIs that are never run by end users (such as test helpers and scripts) if the full path will simplify development workflows or be used for file I/O. Use #file to preserve source compatibility with Swift 5.2 or earlier.

- Omit all labels when arguments can’t be usefully distinguished, e.g. `min(number1, number2)`, `zip(sequence1, sequence2)`.
- In initializers that perform value preserving type conversions, omit the first argument label, e.g. `Int64(someUInt32)`.
- When the first argument forms part of a prepositional phrase, give it an argument label. The argument label should normally begin at the preposition, e.g. `x.removeBoxes(havingLength: 12)`.
- Otherwise, if the first argument forms part of a grammatical phrase, omit its label, appending any preceding words to the base name, e.g. `x.addSubview(y)`.
