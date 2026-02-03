# Swift Date formatter

A structure that creates a locale-appropriate string representation of a date instance and converts strings of dates and times into date instances.

## Date Format
A date format style shares the date and time formatting pattern preferred by the user’s locale for formatting and parsing.
- When you want to apply a specific formatting style to a single Date instance, use Date.FormatStyle. For other instances, use the following:
- When working with date representations in ISO 8601 format, use `Date.ISO8601FormatStyle`.
- To represent an interval between two date instances, use `Date.RelativeFormatStyle`.
- To represent two dates as a pair, for example to get output that looks like 10/21/1985 1:45 PM - 9/13/2015 6:33 PM, use `Date.IntervalFormatStyle`.

### Formatting String Representations of Dates and Times

`Date.FormatStyle` provides a variety of localized presets and configuration options to create user-visible representations of dates and times from instances of Date.

When displaying a date to a user, use the `formatted(date:time:)` instance method. Set the date and time styles of the date format style separately, according to your particular needs.

For example, to create a string with a full date and no time representation, set the `Date.FormatStyle.DateStyle` to complete and the `Date.FormatStyle.TimeStyle` to omitted. Conversely, to create a string representing only the time for the current locale and time zone, set the date style to omitted and the time style to complete, as the following code illustrates:

```swift
let birthday = Date()

birthday.formatted(date: .complete, time: .omitted) // Sunday, January 17, 2021
birthday.formatted(date: .omitted, time: .complete) // 4:03:12 p.m. CST
```

The results shown are for locale set to en_US and time zone set to CST.

You can create string representations of a Date instance with various levels of brevity using preset date and time styles. The following example shows date styles of long, abbreviated, and numeric, and time styles of shortened, standard, and complete:

```swift
let birthday = Date()

birthday.formatted(date: .long, time: .shortened) // January 17, 2021, 4:03 PM
birthday.formatted(date: .abbreviated, time: .standard) // Jan 17, 2021, 4:03:12 PM
birthday.formatted(date: .numeric, time: .complete) // 1/17/2021, 4:03:12 PM CST

birthday.formatted() // Jan 17, 2021, 4:03 PM
```

The default date style is `abbreviated` and the default time style is `shortened`.

For full customization of the string representation of a date, use the formatted(_:) instance method of Date and provide a `Date.FormatStyle` instance.

You can apply more customization of the date and time components and their representation in your app by appying a series of convenience modifiers to your format style. The following example applies a series of modifiers to the format style to precisely define the formatting of the year, month, day, hour, minute, and timezone components of the resulting string. The ordering of the date and time modifiers has no impact on the string produced.

```swift
// Call the .formatted method on an instance of Date passing in an instance of Date.FormatStyle.
let birthday = Date()

birthday.formatted(
    Date.FormatStyle()
        .year(.defaultDigits)
        .month(.abbreviated)
        .day(.twoDigits)
        .hour(.defaultDigits(amPM: .abbreviated))
        .minute(.twoDigits)
        .timeZone(.identifier(.long))
        .era(.wide)
        .dayOfYear(.defaultDigits)
        .weekday(.abbreviated)
        .week(.defaultDigits)
) 
// Sun, Jan 17, 2021 Anno Domini (week: 4), 11:18 AM America/Chicago
```

`Date.FormatStyle` provides a convenient factory variable, `dateTime`, used to shorten the syntax when applying date and time modifiers to customize the format, as in the following example:

```swift
let localeArray = ["en_US", "sv_SE", "en_GB", "th_TH", "fr_BE"]
for localeID in localeArray {
    let date = meetingDate.formatted(.dateTime
             .day(.twoDigits)
             .month(.wide)
             .weekday(.short)
             .hour(.conversationalTwoDigits(amPM: .wide))
             .locale(Locale(identifier: localeID)))
}

// Th, November 12, 7 PM
// to 12 november 19
// Th 12 November, 19
// พฤ. 12 พฤศจิกายน 19
// je 12 novembre, 19 h
```

To parse a Date instance from an input string, use a date parse strategy. For example:

```swift
let inputString = "Archive for month 8, archived on day 23 - complete."
let strategy = Date.ParseStrategy(format: "Archive for month \(month: .defaultDigits), archived on day \(day: .twoDigits) - complete.", locale: Locale(identifier: "en_US"), timeZone: TimeZone(abbreviation: "CDT")!)
if let date = try? Date(inputString, strategy: strategy) {
    date.formatted() // "Aug 23, 2000 at 12:00 AM"
}
```

The time defaults to midnight local time unless explicitly defined.
The parse instance method attempts to parse a provided string into an instance of date using the source date format style. The function throws an error if it can’t parse the input string into a date instance.

You can use `Date.FormatStyle` for round-trip formatting and parsing in a locale-aware manner. This date format style guides parsing the date instance from an input string, as the following code demonstrates:

```swift
let birthdayFormatStyle = Date.FormatStyle()
    .year(.defaultDigits)
    .month(.abbreviated)
    .day(.twoDigits)
    .hour(.defaultDigits(amPM: .abbreviated))
    .minute(.twoDigits)
    .timeZone(.identifier(.long))
    .era(.abbreviated)
    .weekday(.abbreviated)

let yourBirthdayString = "Mon, Feb 17, 1997 AD, 1:27 AM America/Chicago"

// Create a date instance from a string representation of a date.
let yourBirthday = try? birthdayFormatStyle.parse(yourBirthdayString)
// Feb 17, 1997 at 1:27 AM
```

The following round-trip date formatting example uses a date format style to create a locale-aware string representation of a date instance. Then, the date format style guides parsing the newly created string into a new date instance.

```swift
let myFormat = Date.FormatStyle()
    .year()
    .day()
    .month()
    .locale(Locale(identifier: "en_US"))
    
let dateString = Date().formatted(myFormat)
// "Feb 17, 2021" for the "en_US" locale

print(dateString) // Feb 17, 2021

if let anniversary = try? Date(dateString, strategy: myFormat) {
    print(anniversary.formatted(myFormat)) // Feb 17, 2021
    print(anniversary.formatted()) // 2/17/2021, 12:00 AM
} else {
    print("Can't parse string into date with this format.")
}
```

Once you create a date format style, you can use it to format dates multiple times.
You can use a format style to parse a set of date instances from a set of string representations of dates. Then, use another format style, applied repeatedly, to produce more detailed string representations of those dates for a different locale. For example:

```swift
func formatIntroDates() {
   let inputFormat = Date.FormatStyle()
      .locale(Locale(identifier: "en_GB"))
      .year()
      .month()
      .day()
    // Parse string inputs into date instances.
    guard let productIntroDate = try? Date("9 Jan 2007", strategy: inputFormat) else { return }
    guard let anotherIntroDate = try? Date("27 Jan 2010", strategy: inputFormat) else { return }
    guard let conferenceDate = try? Date("7 Jun 2021", strategy: inputFormat) else { return }


    let outputFormat = Date.FormatStyle() // Define format style for string output.
        .locale(Locale(identifier: "en_US"))
        .year()
        .month(.wide)
        .day(.twoDigits)
        .weekday(.abbreviated)


    // Apply the output format on the three dates below.
    print(outputFormat.format(conferenceDate)) // Mon, June 07, 2021
    print(outputFormat.format(anotherIntroDate)) // Wed, January 27, 2010
    print(outputFormat.format(productIntroDate)) // Tue, January 09, 2007
}
```

## Date Relative Format Style

A format style that forms locale-aware string representations of a relative date or time.

Use the strings that the format style produces, such as “1 hour ago”, “in 2 weeks”, “yesterday”, and “tomorrow” as standalone strings. Embedding them in other strings may not be grammatically correct.
Express relative date formats in either numeric or named styles. For example:

```swift
if let past = Calendar.current.date(byAdding: .day, value: -7, to: Date()) {
    var formatStyle = Date.RelativeFormatStyle()
    
    formatStyle.presentation = .numeric
    past.formatted(formatStyle) // "1 week ago"
    
    formatStyle.presentation = .named
    past.formatted(formatStyle) // "last week"
}
```

Use the convenient static factory method `relative(presentation:unitsStyle:)` to shorten the syntax when applying presentation and units style modifiers to customize the format. For example:

```swift
if let past = Calendar.current.date(byAdding: .day, value: 7, to: Date()) {


    past.formatted(.relative(presentation: .numeric)) // "in 1 week"
    past.formatted(.relative(presentation: .named)) // "next week"


    past.formatted(.relative(presentation: .named, unitsStyle: .wide)) // "next week"
    past.formatted(.relative(presentation: .named, unitsStyle: .narrow)) // "next wk."
    past.formatted(.relative(presentation: .named, unitsStyle: .abbreviated)) // "next wk."
    past.formatted(.relative(presentation: .named, unitsStyle: .spellOut)) // "next week"
    past.formatted(.relative(presentation: .numeric, unitsStyle: .wide)) // "in 1 week"
    past.formatted(.relative(presentation: .numeric, unitsStyle: .narrow)) // "in 1 wk."
    past.formatted(.relative(presentation: .numeric, unitsStyle: .abbreviated)) // "in 1 wk."
    past.formatted(.relative(presentation: .numeric, unitsStyle: .spellOut)) // "in one week"
}
```

The `format(_:)` instance method generates a string from the provided relative date. Once you create a style, you can use it to format relative dates multiple times.
The following example applies a format style repeatedly to produce string representations of relative dates:

```swift
if let pastWeek = Calendar.current.date(byAdding: .day, value: -7, to: Date()), 
  let pastDay = Calendar.current.date(byAdding: .day, value: -1, to: Date()) {

    let formatStyle = Date.RelativeFormatStyle(
        presentation: .named,
        unitsStyle: .spellOut,
        locale: Locale(identifier: "en_GB"),
        calendar: Calendar.current,
        capitalizationContext: .beginningOfSentence)
        
    formatStyle.format(pastDay) // "Yesterday"
    formatStyle.format(pastWeek) // "Last week"
}
```

## Date ISO8601 Format Style

A type that converts between dates and their ISO-8601 string representations.

The `Date.ISO8601FormatStyle` type generates and parses string representations of dates following the ISO-8601 standard, like 2024-04-01T12:34:56.789Z. Use this type to create ISO-8601 representations of dates and create dates from text strings in ISO 8601 format. For other formatting conventions, like human-readable, localized date formats, use`Date.FormatStyle`.
Instance modifier methods applied to an ISO-8601 format style customize the formatted output, as the following example illustrates.

```swift
let now = Date()
print(now.formatted(Date.ISO8601FormatStyle().dateSeparator(.dash)))
// 2021-06-21T211015Z
```

Use the static factory property FormatStyle/iso8601 to create an instance of `Date.ISO8601FormatStyle`. Then apply instance modifier methods to customize the format, as in the example below.

```swift
let meetNow = Date()
let formatted = meetNow.formatted(.iso8601
    .year()
    .month()
    .day()
    .timeZone(separator: .omitted)
    .time(includingFractionalSeconds: true)
    .timeSeparator(.colon)
) // "2022-06-10T12:34:56.789Z"
```

## Date Interval Format Style

A format style that creates string representations of date intervals.

Use a date interval format style to create user-readable strings in the form of <start> - <end> for your app’s interface, where <start> and <end> are date values that you supply. The format style uses locale and language information, along with custom formatting options, to define the content of the resulting string.
`Date.IntervalFormatStyle` provides a variety of localized presets and configuration options to create user-visible representations of date intervals. When displaying a date interval to a user, use the `formatted(date:time:)` instance method of Range<Date>. Set the date and time styles of the date interval format style separately, according to your particular needs.
For example, to create a date interval string with a full date and no time representation, set the `Date.IntervalFormatStyle.DateStyle` to `complete` and the `Date.IntervalFormatStyle.TimeStyle` to `omitted`. The following example creates a formatted interval string with this style:

```swift
if let today = Calendar.current.date(byAdding: .day, value: -120, to: Date()),
    let thirtyDaysBeforeToday = Calendar.current.date(byAdding: .day, value: -30, to: today) {
    // today: June 5, 2023
    // thirtyDaysBeforeToday: May 6, 2023

    // Create a Range<Date>.
    let last30days = thirtyDaysBeforeToday..<today

    let formatted = last30days.formatted(date: .complete, time: .omitted)
    // "Saturday, January 30 – Monday, March 1, 2021"
}
```

You can create string representations of date intervals with various levels of brevity using a variety of preset date and time styles. The following example shows date styles of long, abbreviated, and numeric, and time styles of shortened, standard, and complete:

```swift
if let today = Calendar.current.date(byAdding: .day, value: -120, to: Date()),
   let thirtyDaysBeforeToday = Calendar.current.date(byAdding: .day, value: -30, to: today) {
   // today: Mar 1, 2021 at 8:01 PM
   // thirtyDaysBeforeToday: Jan 30, 2021 at 8:01 PM

   // Create a Range<Date>.
   let last30days = thirtyDaysBeforeToday..<today

   print(last30days.formatted(date: .long, time: .shortened))
   // January 30, 2021, 8:01 PM – March 1, 2021, 8:01 PM

   print(last30days.formatted(date: .abbreviated, time: .standard))
   // Jan 30, 2021, 8:01:49 PM – Mar 1, 2021, 8:01:49 PM

   print(last30days.formatted(date: .numeric, time: .complete))
   // 1/30/2021, 8:01:49 PM CST – 3/1/2021, 8:01:49 PM CST

   print(last30days.formatted())
   // 1/30/21, 8:01 PM – 3/1/21, 8:01 PM
}
```

The default date style is abbreviated and the default time style is shortened.

For full customization of the string representation of a date interval, use the formatted(_:) instance method of Range<Date> and provide a Date.IntervalFormatStyle instance.

You can achieve any customization of date and time representation your app requires by appying a series of convenience modifiers to your format style. The following example applies a series of modifiers to the format style to precisely define the formatting of the year, month, day, hour, minute, and time zone components of the resulting string:

```swift
if let today = Calendar.current.date(byAdding: .day, value: -140, to: Date()),
   let sevenDaysAfterToday = Calendar.current.date(byAdding: .day, value: 7, to: today) {

    // Create a Range<Date>.
    let weekFromNow = today..<sevenDaysAfterToday
    
    // Call the .formatted method on a Range<Date> and pass in an instance of Date.IntervalFormatStyle.
    weekFromNow.formatted(
        Date.IntervalFormatStyle()
            .year()
            .month(.abbreviated)
            .day()
            .hour(.defaultDigits(amPM: .narrow))
            .weekday(.abbreviated)
    ) //  Wed, Feb 10, 2021, 3 p – Wed, Feb 17, 2021, 3 p
}
```

`Date.IntervalFormatStyle` provides a convenient factory variable, interval, to shorten the syntax when applying date and time modifiers to customize the format.

```swift
if let today = Calendar.current.date(byAdding: .day, value: -140, to: Date()),
   let sevenDaysBeforeToday = Calendar.current.date(byAdding: .day, value: -7, to: today) {

    // Create a Range<Date>.
    let weekBefore = sevenDaysBeforeToday..<today

    let localeArray = ["en_US", "sv_SE", "en_GB", "th_TH", "fr_BE"]
    for localeID in localeArray {
        // Call the .formatted method on a Range<Date> and pass in an instance of Date.IntervalFormatStyle.
        print(weekBefore.formatted(.interval
                 .day()
                 .month(.wide)
                 .weekday(.short)
                 .hour(.conversationalTwoDigits(amPM: .wide))
                 .locale(Locale(identifier: localeID))))
    }
}
// We, February 3, 3 PM – We, February 10, 3 PM
// on 3 februari 15 – on 10 februari 15
// We 3 February, 15 – We 10 February, 15
// พ. 3 กุมภาพันธ์ 15 – พ. 10 กุมภาพันธ์ 15
// me 3 février, 15 h – me 10 février, 15 h
```
