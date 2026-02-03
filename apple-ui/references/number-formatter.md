# Swift Number formatter

## Integer Format Style
A structure that converts between integer values and their textual representations.

Instances of IntegerFormatStyle create localized, human-readable text from BinaryInteger numbers and parse string representations of numbers into instances of BinaryInteger types. All of the Swift standard library’s integer types, such as Int and UInt32, conform to BinaryInteger, and therefore work with this format style.

IntegerFormatStyle includes two nested types, IntegerFormatStyle.Percent and IntegerFormatStyle.Currency, for working with percentages and currencies. Each format style includes a configuration that determines how it represents numeric values, for things like grouping, displaying signs, and variant presentations like scientific notation. 

IntegerFormatStyle and IntegerFormatStyle.Percent include a NumberFormatStyleConfiguration, and IntegerFormatStyle.Currency includes a CurrencyFormatStyleConfiguration. You can customize numeric formatting for a style by adjusting its backing configuration. The system automatically caches unique configurations of a format style to enhance performance.

### Formatting Integer

Use the `formatted()` method to create a string representation of an integer using the default IntegerFormatStyle configuration.

```swift
let formattedDefault = 123456.formatted()
// formattedDefault is "123,456" in en_US locale.
// Other locales may use different separator and grouping behavior.
```

You can specify a format style by providing an argument to the `formatted(_:)` method. The following example shows the number 12345 represented in each of the available styles, in the en_US locale:

```swift
let number = 123456

let formattedNumber = number.formatted(.number)
// formattedNumber is "123,456".

let formattedPercent = number.formatted(.percent)
// formattedPercent is "123,456%".

let formattedCurrency = number.formatted(.currency(code: "USD"))
// formattedCurrency is "$123,456.00".
```

Each style provides methods for updating its numeric configuration, including the number of significant digits, grouping length, and more. You can specify a numeric configuration by calling as many of these methods as you need in any order you choose. The following example shows the same number with default and custom configurations:

```swift
let exampleNumber = 123456

let defaultFormatting = exampleNumber.formatted(.number)
// defaultFormatting is "125 000" for the "fr_FR" locale
// defaultFormatting is "125000" for the "jp_JP" locale
// defaultFormatting is "125,000" for the "en_US" locale

let customFormatting = exampleNumber.formatted(
    .number
    .grouping(.never)
    .sign(strategy: .always()))
// customFormatting is "+123456"
```

### Creating an integer format style instance

The previous examples use static factory methods like number to create format styles within the call to the `formatted(_:)` method. You can also create an IntegerFormatStyle instance and use it to repeatedly format different values with the `format(_:)` method:

```swift
let percentFormatStyle = IntegerFormatStyle<Int>.Percent()

percentFormatStyle.format(50) // "50%"
percentFormatStyle.format(85) // "85%"
percentFormatStyle.format(100) // "100%"
```

### Parsing integers
You can use IntegerFormatStyle to parse strings into integer values. You can define the format style within the type’s initializer or pass in a format style you create prior to calling the method, as shown here:

```swift
let price = try? Int("$123,456", format: .currency(code: "USD")) // 123456

let priceFormatStyle = IntegerFormatStyle<Int>.Currency(code: "USD")
let salePrice = try? Int("$120,000", format: priceFormatStyle) // 120000
```

### Matching regular expressions

Along with parsing numeric values in strings, you can use the Swift regular expression domain-specific language to match and capture numeric substrings. The following example defines a currency format style to match and capture a currency value using US dollars and en_US numeric conventions. The rest of the regular expression ignores any characters prior to a ": " sequence that precedes the currency substring.

```swift
import RegexBuilder

let source = "Payment due: $123,456"
let matcher = Regex {
    OneOrMore(.any)
    ": "
    Capture {
        One(.localizedIntegerCurrency(code: Locale.Currency("USD"),
                                      locale: Locale(identifier: "en_US")))
    }
}
let match = source.firstMatch(of: matcher)
let localizedInteger = match?.1 // 123456
```

## Floating Point Format Style
A structure that converts between floating-point values and their textual representations.

### Formatting floating-point values
Use the `formatted()` method to create a string representation of a floating-point value using the default FloatingPointFormatStyle configuration.

```swift
let formattedDefault = 12345.67.formatted()
// formattedDefault is "12,345.67" in the en_US locale.
// Other locales may use different separator and grouping behavior.
```

### Creating a floating-point format style instance

The previous examples use static factory methods like number to create format styles within the call to the `formatted(_:)` method. You can also create a `FloatingPointFormatStyle` instance and use it to repeatedly format different values, with the `format(_:)` method:

```swift
let percentFormatStyle = FloatingPointFormatStyle<Double>.Percent()

percentFormatStyle.format(0.5) // "50%"
percentFormatStyle.format(0.855) // "85.5%"
percentFormatStyle.format(1.0) // "100%"
```

### Parsing floating-point values
You can use `FloatingPointFormatStyle` to parse strings into floating-point values. You can define the format style within the type’s initializer or pass in a format style created outside the function, as shown here:

```swift
let price = try? Double("$3,500.63", format: .currency(code: "USD")) // 3500.63

let priceFormatStyle = FloatingPointFormatStyle<Double>.Currency(code: "USD")
let salePrice = try? Double("$731.67", format: priceFormatStyle) // 731.67
```

## Decimal Format Style
A structure that converts between decimal values and their textual representations.

### Formatting decimal values

Use the `formatted()` method to create a string representation of a decimal value using the default `Decimal.FormatStyle` configuration:

```swift
let formattedDefault = Decimal(12345.67).formatted()
// formattedDefault is "12,345.67" in en_US locale.
// Other locales may use different separator and grouping behavior.
```

You can specify a format style by providing an argument to the `formatted(_:)` method. The following example shows the decimal 0.1 represented in each of the available styles in the en_US locale:

```swift
let number: Decimal = 0.1

let formattedNumber = number.formatted(.number)
// formattedNumber is "0.1"

let formattedPercent = number.formatted(.percent)
// formattedPercent is "10%"

let formattedCurrency = number.formatted(.currency(code: "USD"))
// formattedCurrency is "$0.10"
```

Each style provides methods for updating its numeric configuration, including the number of significant digits, grouping length, and more. You can specify a numeric configuration by calling as many of these methods as you need in any order you choose. The following example shows the same number with default and custom configurations:

```swift
let exampleNumber: Decimal = 125000.12

let defaultFormatting = exampleNumber.formatted(.number)
// defaultFormatting is "125 000,12" for the "fr_FR" locale
// defaultFormatting is "125,000.12" for the "en_US" locale

let customFormatting = exampleNumber.formatted(
    .number
    .grouping(.never)
    .sign(strategy: .always()))
// customFormatting is "+125000.12"
```

### Creating a decimal format style instance

The previous examples use static instances like number to create format styles within the call to the `formatted(_:)` method. You can also create a `Decimal.FormatStyle` instance and use it to repeatedly format different values by using the `format(_:)` method, as shown here:

```swift
let percentFormatStyle = Decimal.FormatStyle.Percent()

percentFormatStyle.format(0.5) // "50%"
percentFormatStyle.format(0.855) // "85.5%"
percentFormatStyle.format(1.0) // "100%"
```

### Parsing decimal values

You can use `Decimal.FormatStyle` to parse strings into decimal values. You can define the format style within the type’s initializer or pass in a format style created outside the function. The following demonstrates both approaches:

```swift
let price = try? Decimal("$3,500.63", format: .currency(code: "USD")) // 3500.63

let priceFormatStyle = Decimal.FormatStyle.Currency(code: "USD")
let salePrice = try? Decimal("$731.67", format: priceFormatStyle) // 731.67
```

### Matching regular expressions
Along with parsing numeric values in strings, you can use the Swift regular expression domain-specific language to match and capture numeric substrings. The following example defines a currency format style to match and capture a currency value using US dollars and en_US numeric conventions. The rest of the regular expression ignores any characters prior to a ": " sequence that precedes the currency substring.

```swift
import RegexBuilder
let source = "Payment due: $49,525.99"
let matcher = Regex {
    OneOrMore(.any)
    ": "
    Capture {
        One(.localizedCurrency(code:Locale.Currency("USD"),
                               locale:Locale(identifier: "en_US")))
    }
}
let match = source.firstMatch(of: matcher)
let localizedDecimal = match?.1 // 49525.99
```