
# Persian calendar
## WPF calendar with support for Gregorian and solar calendars

- A free Persian calendar with the ability to support the Gregorian calendar that can be used in WPF
- You can write any style you like for the controls.
- Language change support and other resources

Please see the test project for further guidance.


## Date and calendar APIs

The date helpers can be used without instantiating a WPF control. The package still targets the Windows frameworks listed in the project. Examples below assume `DateTime date` and `DateTime other` are available; asynchronous examples belong in an async method or command.

### Persian parsing, formatting and components

```csharp
using System;
using PersianDateTimeWPFTools;
using PersianDateTimeWPFTools.Extensions.Persian;

bool parsedDate = PersianDateTime.TryParse("1405/07/30", out DateTime valueDate);
bool parsedDateTime = PersianDateTime.TryParse("1405/07/30 15:30", out DateTime valueDateTime);
bool exact = PersianDateTime.TryParseExact("1405/07/30", "yyyy/MM/dd", out DateTime exactValue);
string text = date.ToPersianString("yyyy/MM/dd");
string persianDigits = date.ToPersianString("yyyy/MM/dd", usePersianDigits: true);
int year = date.GetPersianYear();
string monthName = date.GetPersianMonthName();
string dayName = date.GetPersianDayName();
var parts = date.GetPersianDateTimeParts();
TimeSpan exactTime = parts.TimeOfDay;
DateTimeKind kind = parts.Kind;
```
### Parser Default Formats Supported
|Predefined format| Format | Result |
|--|--|--|
| d | yyyy/MM/dd |1405/07/30|
| D | dddd dd MMMM yyyy |پنجشنبه 30 مهر 1405|
| t | HH:mm |00:49|
| T | HH:mm:ss |00:50:23|
| g | yyyy/MM/dd HH:mm |1405/07/30 00:51|
| G | yyyy/MM/dd HH:mm:ss |1405/07/30 00:51:05|
| s | yyyy-MM-ddTHH:mm:ss |1405-07-30T00:51:05|


`PersianDateTimeParts` is immutable and includes year, month, day, hour, minute, second, millisecond, the original time of day at tick precision, and `Kind`. No time zone conversion is performed. `Kind` is not a time zone identifier or an offset.

```csharp
DateTime nextMonth = date.AddPersianMonths(1);
DateTime changedYear = date.WithPersianYear(1404);
DateTime changedMonth = date.WithPersianMonth(7, InvalidDayHandling.Throw);
DateTime changedDay = date.WithPersianDay(15);
bool sameMonth = date.IsInSamePersianMonth(other);

foreach (DateTime item in date.EnumeratePersianMonthDates())
{
    // Dates retain the input time and Kind.
}
DateTime secondTuesday;
bool found = date.TryGetNthWeekdayOfPersianMonth(PersianDayOfWeek.Tuesday, 2, out secondTuesday);
DateTime lastFriday = date.GetLastWeekdayOfPersianMonth(PersianDayOfWeek.Friday);
```

Year/month replacement defaults to `InvalidDayHandling.Clamp`, which limits an absent day to the final day of the target month. `Throw` rejects that day. Invalid years, months and policy values are always rejected. Day replacement rejects an invalid day. Editing preserves the complete time and `Kind`.

Weekday occurrence is one-based, from 1 through 5. An absent occurrence returns `false` and `default(DateTime)`; invalid arguments throw. Successful month queries preserve time and `Kind`. Enumeration is repeatable; Persian source dates are validated before returning the sequence.

### Periods and half-open ranges

```csharp
DateTime start = date.StartOfPersianMonth();
PersianDateRange month = date.GetPersianMonthRange();
PersianDateRange currentWeek = PersianDateRanges.CurrentWeek();
PersianDateRange previousQuarter = PersianDateRanges.PreviousQuarter();
bool contains = month.Contains(date);
```

Ranges use `[StartInclusive, EndExclusive)`. The Persian week starts on Saturday; week 1 contains Farvardin 1. Start-of-period operations return midnight. Editing and month queries retain the original time. Navigation requiring a date outside the supported range throws.

### Focused Persian services

| Service | Responsibility |
|---|---|
| `IPersianDateProvider` | Components, calendar information, month/year arithmetic and date construction |
| `IPersianDatePeriodProvider` | Period boundaries, week numbering and quarters |
| `IPersianDateComparisonProvider` | Equality at day, month and year precision |
| `IPersianDateEditor` | Component replacement |
| `IPersianDateQueryProvider` | Month predicates, enumeration and weekday queries |
| `IPersianDateRangeProvider` | Explicit-date and clock-relative half-open ranges |

The core provider has 12 members. `ToDateTime(year, month, day)` constructs midnight with `Kind=Unspecified`. Default period, comparison, editor and query services use the injected core provider, including its date construction behavior.

```csharp
var services = PersianDateTimeConfiguration.Services;
DateTime start = services.DatePeriodProvider.StartOfMonth(date);
DateTime edited = services.DateEditor.WithMonth(date, 7);
PersianDateRange range = services.DateRangeProvider.GetMonthRange(date);
```

`PersianDateTimeConfiguration` configures Persian services and the existing clock integration. Advanced operations can be replaced independently using `DatePeriodProviderFactory`, `DateComparisonProviderFactory`, `DateEditorFactory` and `DateQueryProviderFactory`; each receives the configured core provider. Public default implementations expose virtual operations so a subclass can override only the required behavior.

Formatter, parser and range factories receive `PersianDateTimeServiceContext`, including the resolved Persian operation services. Holiday and working-day policy is configured separately, as shown below. A factory returning `null` rejects the configuration. `Reset()` restores defaults. Retained derived services stay bound to their original core provider.

### Native DateTime helpers

Use native .NET APIs for existing Gregorian operations: `date.Year`, `date.Month`, `date.Day`, `date.AddMonths(...)` and `date.AddYears(...)`.

Additional helpers use short names in the shared extension namespace:

```csharp
using PersianDateTimeWPFTools.Extensions;

DateTime start = date.StartOfMonth();
DateTime nextQuarter = date.StartOfNextQuarter();
DateTime changed = date.WithMonth(2, InvalidDayHandling.Throw);
foreach (DateTime item in date.EnumerateMonthDates())
{
    // Gregorian month dates, with the input time and Kind.
}
DateTime secondTuesday;
bool found = date.TryGetNthWeekdayOfMonth(DayOfWeek.Tuesday, 2, out secondTuesday);
DateTime lastFriday = date.GetLastWeekdayOfMonth(DayOfWeek.Friday);
```

Calendar calculations with unqualified names on `DateTime` are Gregorian, matching its native components. Persian-specific counterparts keep `Persian` in their method names and remain opt-in through `Extensions.Persian`. Working-day and holiday helpers are policy-based and do not imply a calendar.

The 18 additional date helpers comprise ten period operations, three component-editing operations and five month queries. `StartOf...` returns midnight and preserves `Kind`; editing and month queries preserve full time precision and `Kind`. No time zone conversion occurs. Gregorian editing and weekday-query failure policies match the Persian APIs above.

Gregorian configuration remains explicit and independent:

```csharp
using System.Globalization;
using PersianDateTimeWPFTools.Services.DateTimeServices;

GregorianDateTimeConfiguration.Configure(options =>
{
    options.DatePeriodProvider = new DefaultGregorianDatePeriodProvider(
        DayOfWeek.Sunday, CalendarWeekRule.FirstDay);
});
```

`GregorianDateTimeConfiguration.Services` exposes `IGregorianDatePeriodProvider`, `IGregorianDateEditor` and `IGregorianDateQueryProvider`. Replace implementations independently or subclass their public defaults and override selected methods.

Gregorian weeks default to Monday with `FirstFourDayWeek`. Week numbering uses `Calendar.GetWeekOfYear`, not an ISO week-number/year pair. This configures helper calculations, not WPF control properties.

### Asynchronous holidays and working days

```csharp
using System.Threading;
using PersianDateTimeWPFTools.Extensions;

bool holiday = await date.IsHolidayAsync(cancellationToken);
bool working = await date.IsWorkingDayAsync(cancellationToken);
DateTime next = await date.NextWorkingDayAsync(cancellationToken);
DateTime previous = await date.PreviousWorkingDayAsync(cancellationToken);
DateTime afterFive = await date.AddWorkingDaysAsync(5, cancellationToken);
```

`IsHolidayAsync` directly uses the configured `IHolidayProvider`; the working-day extensions use `IWorkingDayProvider`. These contracts are async-only and accept optional cancellation tokens. A provider can use a database, HTTP service, stream, cache or in-memory rules. Returning `Task.FromResult` for an already-known result is supported; asynchronous signatures do not imply thread-pool execution.

The default `FridayHolidayProvider` marks only Fridays as holidays; it is not a country's public-holiday list. `DefaultWorkingDayProvider` treats non-holidays as working days. Navigation excludes the starting day, supports negative offsets and preserves time and `Kind`; zero returns the original date. Cancellation is cooperative and checked even for a zero offset and between navigation steps. Holiday-source failures propagate rather than being interpreted as working days.

Working-day policy has its own configuration:

```csharp
using PersianDateTimeWPFTools.Services.WorkingDays;

WorkingDayConfiguration.Configure(options =>
{
    options.HolidayProvider = new FridayHolidayProvider();
    // Optional; the default already binds to the selected holiday provider.
    options.WorkingDayProviderFactory = holidays => new DefaultWorkingDayProvider(holidays);
});
```

Implement `IHolidayProvider.IsHolidayAsync(DateTime, CancellationToken)` to supply your own data. For in-memory rules, a complete example is:

```csharp
using System;
using System.Threading;
using System.Threading.Tasks;
using PersianDateTimeWPFTools.Abstractions;

public sealed class WeekendHolidayProvider : IHolidayProvider
{
    public Task<bool> IsHolidayAsync(DateTime date, CancellationToken cancellationToken = default)
    {
        if (cancellationToken.IsCancellationRequested)
            return Task.FromCanceled<bool>(cancellationToken);
        return Task.FromResult(date.DayOfWeek == DayOfWeek.Saturday ||
                               date.DayOfWeek == DayOfWeek.Sunday);
    }
}
```

Configure it with `WorkingDayConfiguration.Configure(o => o.HolidayProvider = new WeekendHolidayProvider());`. The working-day service is rebuilt against that policy. A custom factory receives the selected `IHolidayProvider`; a factory returning `null` rejects the configuration. Subclass `DefaultWorkingDayProvider` to override individual operations, or implement `IWorkingDayProvider` directly.

`WorkingDayConfiguration.Services` exposes the holiday and working-day services from the same snapshot. Capture that snapshot when several related queries must use the same policy. Configuring or resetting Persian, Gregorian or working-day services does not reset the other configurations.

For I/O-backed navigation, implementations can prefetch/cache a date range instead of issuing one remote request per day. For WPF presentation, load the visible-date facts asynchronously outside the day-button rendering loop, then project them into `DayMetadata`. No automatic holiday styling or asynchronous render-loop calls are introduced by these helpers.

### Digits and provider facades

```csharp
using PersianDateTimeWPFTools.Extensions.Persian;

string persian = "1405/07/28".ToPersianDigits();
string english = persian.ToEnglishDigits();
bool success = persian.TryParsePersianDateTime(out DateTime parsed);
```

Provider facades in `PersianDateTimeWPFTools.Providers` expose configured services when direct access is needed. `HolidayProvider.Current` and `WorkingDayProvider.Current` use `WorkingDayConfiguration`; calendar operation facades use their respective configurations. No setup is required for the default behavior.



Persian DateTime Extensions
## Persian DateTime Extensions

All examples below use:

```csharp

var date = new DateTime(2026, 10, 22, 15, 30, 45); // 1405/07/30 15:30:45

```

### Formatting

| Method | Result |
|----------|----------|
| `ToPersianString()` | `1405/07/30 15:30:45` |
| `ToPersianString(usePersianDigits: true)` | `۱۴۰۵/۰۷/۳۰ ۱۵:۳۰:۴۵` |
| `ToPersianString("d")` | `1405/07/30` |
| `ToPersianString("d", usePersianDigits: true)` | `۱۴۰۵/۰۷/۳۰` |

### Date Components

| Method | Result |
|----------|----------|
| `GetPersianYear()` | `1405` |
| `GetPersianMonth()` | `7` |
| `GetPersianDay()` | `30` |
| `GetPersianMonthName()` | `مهر` |
| `GetPersianDayName()` | `پنجشنبه` |
| `GetPersianDayOfWeek()` | `Thursday` |
| `GetPersianDayOfYear()` | `216` |
| `GetPersianWeekOfYear()` | `31` |
| `GetPersianQuarter()` | `3` |
| `GetPersianDateTimeParts()` | `PersianDateTimeParts` |

### Calendar Information

| Method | Result |
|----------|----------|
| `IsPersianLeapYear()` | `false` |
| `GetDaysInPersianMonth()` | `30` |
| `GetDaysInPersianYear()` | `365` |

### Date Arithmetic

| Method | Result |
|----------|----------|
| `AddPersianMonths(1)` | `1405/08/30 15:30:45` |
| `AddPersianYears(1)` | `1406/07/30 15:30:45` |
| `WithPersianYear(1407)` | `1407/07/30 15:30:45` |
| `WithPersianMonth(10)` | `1405/10/30 15:30:45` |
| `WithPersianDay(10)` | `1405/07/10 15:30:45` |

### Month Operations

| Method | Result |
|----------|----------|
| `StartOfPersianMonth()` | `1405/07/01 00:00:00` |
| `StartOfNextPersianMonth()` | `1405/08/01 00:00:00` |
| `GetPersianMonthRange()` | `[1405/07/01, 1405/08/01)` |
| `EnumeratePersianMonthDates()` | `IEnumerable<DateTime>` |

### Week Operations

| Method | Result |
|----------|----------|
| `StartOfPersianWeek()` | `1405/07/25 00:00:00` |
| `StartOfNextPersianWeek()` | `1405/08/02 00:00:00` |
| `GetPersianWeekRange()` | `[1405/07/25, 1405/08/02)` |

### Year Operations

| Method | Result |
|----------|----------|
| `StartOfPersianYear()` | `1405/01/01 00:00:00` |
| `StartOfNextPersianYear()` | `1406/01/01 00:00:00` |
| `GetPersianYearRange()` | `[1405/01/01, 1406/01/01)` |

### Quarter Operations

| Method | Result |
|----------|----------|
| `StartOfPersianQuarter()` | `1405/07/01 00:00:00` |
| `StartOfNextPersianQuarter()` | `1405/10/01 00:00:00` |
| `GetPersianQuarterRange()` | `[1405/07/01, 1405/10/01)` |

### Date Comparison

| Method | Result |
|----------|----------|
| `IsInSamePersianDay(otherDate)` | `false` |
| `IsInSamePersianMonth(otherDate)` | `true` |
| `IsInSamePersianYear(otherDate)` | `true` |
| `IsFirstDayOfPersianMonth()` | `false` |
| `IsLastDayOfPersianMonth()` | `false` |

### Advanced Operations

| Method | Result |
|----------|----------|
| `TryGetNthWeekdayOfPersianMonth(PersianDayOfWeek.Friday, 2, out var result)` | `1405/07/10 00:00:00` |
| `GetLastWeekdayOfPersianMonth(PersianDayOfWeek.Friday)` | `1405/07/24 00:00:00` |


## Native DateTime Extensions (Gregorian)

All examples below use:

```csharp

var date = new DateTime(2026, 10, 22, 15, 30, 45);

```

### Comparison

| Method | Result |
|----------|----------|
| `IsBetween(new DateTime(2026, 10, 01), new DateTime(2026, 11, 01))` | `true` |
| `HasExpired(new DateTime(2026, 10, 20))` | `false` |
| `new DateTime(2026, 10, 21).HasExpired()` | `true` |

### Month Operations

| Method | Result |
|----------|----------|
| `StartOfMonth()` | `2026-10-01 00:00:00` |
| `StartOfNextMonth()` | `2026-11-01 00:00:00` |
| `IsFirstDayOfMonth()` | `false` |
| `IsLastDayOfMonth()` | `false` |
| `EnumerateMonthDates()` | `IEnumerable<DateTime>` |
| `TryGetNthWeekdayOfMonth(DayOfWeek.Friday, 2, out var result)` | `2026-10-09 00:00:00` |
| `GetLastWeekdayOfMonth(DayOfWeek.Friday)` | `2026-10-30 00:00:00` |

### Week Operations

| Method | Result |
|----------|----------|
| `StartOfWeek()` | `2026-10-18 00:00:00` |
| `StartOfNextWeek()` | `2026-10-25 00:00:00` |
| `GetWeekOfYear()` | `43` |

### Quarter Operations

| Method | Result |
|----------|----------|
| `GetQuarter()` | `4` |
| `StartOfQuarter()` | `2026-10-01 00:00:00` |
| `StartOfNextQuarter()` | `2027-01-01 00:00:00` |

### Year Operations

| Method | Result |
|----------|----------|
| `StartOfYear()` | `2026-01-01 00:00:00` |
| `StartOfNextYear()` | `2027-01-01 00:00:00` |

### Date Editing

| Method | Result |
|----------|----------|
| `WithYear(2028)` | `2028-10-22 15:30:45` |
| `WithMonth(12)` | `2026-12-22 15:30:45` |
| `WithDay(10)` | `2026-10-10 15:30:45` |

### Invalid Day Handling

| Method | Result |
|----------|----------|
| `new DateTime(2024, 02, 29).WithYear(2025)` | `2025-02-28 00:00:00` |
| `new DateTime(2024, 01, 31).WithMonth(2)` | `2024-02-29 00:00:00` |
| `WithMonth(2, InvalidDayHandling.Throw)` | `ArgumentOutOfRangeException` |

## Holiday Extensions

| Method | Result |
|----------|----------|
| `IsHolidayAsync()` | `Task<bool>` |

## Working Day Extensions

| Method | Result |
|----------|----------|
| `IsWorkingDayAsync()` | `Task<bool>` |
| `AddWorkingDaysAsync(5)` | `Task<DateTime>` |
| `NextWorkingDayAsync()` | `Task<DateTime>` |
| `PreviousWorkingDayAsync()` | `Task<DateTime>` |


## Persian String Extensions

### Parsing

| Method | Result |
|----------|----------|
| `"1405/07/30".TryParsePersianDateTime(out var result)` | `true` |
| `"1405/07/30".TryParsePersianDateTimeExact("yyyy/MM/dd", out var result)` | `true` |
| `"invalid-date".TryParsePersianDateTime(out var result)` | `false` |
| `"1405/7/30".TryParsePersianDateTimeExact("yyyy/MM/dd", out var result)` | `false` |

### Digit Conversion

| Method | Result |
|----------|----------|
| `"1234567890".ToPersianDigits()` | `۱۲۳۴۵۶۷۸۹۰` |
| `"Order #123".ToPersianDigits()` | `Order #۱۲۳` |
| `"۱۲۳۴۵۶۷۸۹۰".ToEnglishDigits()` | `1234567890` |
| `"شماره ۱۲۳".ToEnglishDigits()` | `شماره 123` |



## Controls
- Clock

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/clock.png)
- PersianCalendar

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pc.png)
- PersianCalendarWithClock

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdpwc.png)
- PersianDatePicker

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdp.png)
- PersianDateTimePicker

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdtp.png)


You can manually change the Culture of the control using the following code in XAML or C#.

**XAML**
```
CustomCultureName="fa-IR"
```

**C#**
```
pcwc1.CustomCulture = CultureInfo.CreateSpecificCulture("en-US");
```
If you do not select Culture, it will be automatically selected based on the software's Culture.

## ⚙️How to use?
Step 1: Add the following resources in the App.xaml file

```
<Application 
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools"
             >
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <pdtt:InitResources />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

Step 2:
Use in XAML (WPF) files
You can use below namespace in xaml (wpf)

xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools"


Sample:
```
<Window 
    xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools">
    <Grid>
        <pdtt:PersianCalendar />
    </Grid>
</Window>
```

## Ability to write custom styles for all controls
![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/Persian-calendar-Custom-Style.png)


## Dependency Properties and Features
| Name        | Description           | Default  |
| :------------- |:-------------| :-----|
| CustomCulture      | Selecting a different culture than the current software culture | Based on the current software culture |
| CustomCultureName     | Choosing a culture name that is different from the current software culture (such as fa-IR or en-US)      |   Based on the current software culture |
| AllowSelectBlackedOutDay | Selectable holidays      |    False |
| ShowTodayButton | Show a button to move to the current day   |    False |
| ShowConfirmButton | Show a button to display confirm button   |    False |
| DisplayDate | Displays the current date   |    ? |
| DisplayDateStart | Start displaying the date   |    ? |
| DisplayDateEnd | End of displaying date   |    ? |
| DisplayMode | Date display type (like Month,Year,Decade)   |    Month |
| FirstDayOfWeek | First day of the week  |    Sunday |
| SelectionMode | Type of selection  |    SingleDate |
| IsTodayHighlighted | Show current day as highlights  |    True |
| ✨ DayMetadata | Introduce DayMetadata API for unified per-day customization such as Tooltip, DayIndicators, DayIndicatorStyle, etc.  |     |
| ✨ DayToolTips | Show Tooltip for calendar days  |     |
| ✨ DayToolTipTemplate | A template for a tooltip  |     |
| ✨ DayIndicators | Display Indicator for calendar days  |     |
| ✨ DayIndicatorStyle | A style for the Indicator  |     |
| ✨ ISystemClock | A static Clock provider to replace direct DateTime usage  |  new SystemClock()  |



## ❔ What is DayMetadata?

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/dayMetadataUse.png)

introduce DayMetadata API for unified per-day customization
- Added CalendarDayInfo model to describe per-day metadata
- Introduced DayMetadata dictionary to centralize day configuration
- Unified tooltip, indicator, and disabled-day logic under a single API
- Ensured backward compatibility with existing DayToolTips and DayIndicators
- Enabled tooltips to be shown on disabled days for better UX
- Implemented virtualization-safe updates across month navigation

## 📝 A note about the DayMetadata execution priority
DayMetadata

↓

DayToolTips / DayIndicators

↓

Default behavior

## ⚙️How to use DayMetadata?

First you need to familiarize yourself with the properties of the CalendarDayInfo class.


| Name        | Description           | Default  |
| :------------- |:-------------| :-----|
| ToolTip      | Show Tooltip for calendar days |  |
| ToolTipTemplate      | A template for a tooltip. |  |
| HasIndicator      | Display Indicator for calendar days | False |
| IndicatorStyle      | A style for the Indicator (You can create a unique style for each day.) |  |
| IsDisabled      | Disable specific day| False |
| ShowToolTipWhenDisabled      | Show Tooltip for disabled days | True |

C#

```
persianCalendar.DayMetadata = new Dictionary<DateTime, CalendarDayInfo>
{
    [new DateTime(2025, 12, 29)] = new CalendarDayInfo
    {
        ToolTip = "Meeting with the marketing team",
    },
    [new DateTime(2025, 12, 31)] = new CalendarDayInfo
    {
        ToolTip = "Important meeting with the development team",
        HasIndicator = true,
        IndicatorStyle = Resources["DayIndicatorRedStyle"] as Style,
    },
    [new DateTime(2025, 12, 21)] = new CalendarDayInfo
    {
        ToolTip = "My love's birthday",
        HasIndicator = true,
        IsDisabled = true,
        ShowToolTipWhenDisabled = true,
        IndicatorStyle = this.Resources["DayIndicatorGreenStyle"] as Style,
        ToolTipTemplate = Resources["FancyDayToolTipTemplate"] as DataTemplate,
    }
};
```


## ✨Tip 1
The DateTime object should only contain dates.

## ✨Tip 2
If you want to update DayMetadata you need to do something like the following

```
var preDayMetadata = persianCalendar.DayMetadata;

preDayMetadata.Add(DateTime.Today.AddDays(2), new CalendarDayInfo() { ToolTip = "Don't forget about renting a house.", HasIndicator = true });

persianCalendar.DayMetadata = preDayMetadata;
```

💫Creating a Style for the Day Indicator

```
<Style
    x:Key="DayIndicatorRedStyle"
    TargetType="Ellipse">
    <Setter
        Property="Width"
        Value="10" />
    <Setter
        Property="Height"
        Value="10" />
    <Setter
        Property="Fill"
        Value="Red" />
    <Setter
        Property="VerticalAlignment"
        Value="Top" />
    <Setter
        Property="HorizontalAlignment"
        Value="Right" />
    <Setter
        Property="Margin"
        Value="2,2,2,4" />
</Style>
```
## ⚙️How to use Day ToolTips?

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/dayTooltip.png)

C#

```
persianCalendar.DayToolTips = new Dictionary<DateTime, object>
{
            { DateTime.Today,"It is today."},
            { DateTime.Today.AddDays(1),"It is tomorrow."},
};
```

## ✨Tip 1
The DateTime object should only contain dates.

## ✨Tip 2
If you want to update Day ToolTips you need to do something like the following
```
var preTooltips = persianCalendar.DayToolTips;
preTooltips.Add(DateTime.Today.AddDays(2), "Important business meeting");
persianCalendar.DayToolTips = preTooltips;
```
## You can also create a Template for Day Tooltip.
XAML
```
<DataTemplate
    x:Key="FancyDayToolTipTemplate">
    <Border
        Background="#222"
        Padding="8"
        CornerRadius="6">
        <StackPanel>
            <TextBlock
                Text="{Binding}"
                Foreground="White"
                FontWeight="Bold" />
            <TextBlock
                Text="More..."
                Foreground="LightGray"
                FontSize="11" />
        </StackPanel>
    </Border>
</DataTemplate>
```
And in your control

```
DayToolTipTemplate="{StaticResource FancyDayToolTipTemplate}"
```
## ⚙️How to use Day Indicators?
![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/dayIndicator.png)

C#

```
persianCalendar.DayIndicators = new Dictionary<DateTime, bool>
{
            [new DateTime(2025, 12, 23)] = true,
            [new DateTime(2025, 12, 25)] = true
};
```

## ✨Tip 1
The DateTime object should only contain dates.

## ✨Tip 2
If you want to update Day Indicators you need to do something like the following
```
var preDayIndicators = persianCalendar.DayIndicators;
preDayIndicators.Add(DateTime.Today.AddDays(2), true);
persianCalendar.DayIndicators = preDayIndicators;
```
## You can also create a Style for Day Indicators.
```
<Style
    x:Key="RedDotIndicatorStyle"
    TargetType="Ellipse">
    <Setter
        Property="HorizontalAlignment"
        Value="Right" />
    <Setter
        Property="VerticalAlignment"
        Value="Top" />
    <Setter
        Property="Width"
        Value="10" />
    <Setter
        Property="Height"
        Value="10" />
    <Setter
        Property="Fill"
        Value="Red" />
</Style>
```
And in your control

```
DayIndicatorStyle="{StaticResource RedDotIndicatorStyle}"
```

## ⚙️How to change the theme
In the **App.xaml** file, you can select one of the following themes by selecting the **SelectedTheme** property in the InitResources class.
- Default
- DarkModern1
- LightModern1
```
<Application 
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools"
             >
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <pdtt:InitResources
                    SelectedTheme="LightModern1"
                />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

## ⚙️How to change theme in C#
You can do the following in the constructor of the App.xaml.cs class:
```
public partial class App : Application
    {        
        public App()
        {
            var init = new InitResources();
            init.SelectedTheme = PersianDateTimeWPFTools.Themes.BaseThemeName.DarkModern1;
        }
    }
```
or
```
public partial class App : Application
    {        
        public App()
        {
            InitResources.SetTheme(new ThemeDarkModern1());
        }
    }
```
## ⚙️How to change the control language?
```
public partial class App : Application
    {        
        public App()
        {
            new InitResources()
            .ChangeLanguage("fa"); // en
        }
    }
```

## ⚙️How to change the language of controls with our own resources
To read the language resources, please visit the link below.
[Github Link](https://github.com/Behnam2064/PersianDateTimeWPFTools/blob/main/PersianDateTimeWPFTools/PersianDateTimeWPFTools/Resources/Lang/Lang.en.xaml)
```
public partial class App : Application
    {        
        public App()
        {
            new InitResources()
            .ChangeLanguage(null, "pack://application:,,,/TestPersianCalendar;component/Lang.es.xaml"); // Your resource address
        }
    }
```
## ISystemClock and ClockProvider
`ISystemClock` provides a replaceable time source for testability and for scenarios where WPF calendar controls must use a custom value for the current time.

For new code, prefer configuring the clock together with the other services at application startup:

```csharp
PersianDateTimeConfiguration.Configure(options =>
{
    options.Clock = new MySystemClock();
});
```

`ClockProvider` remains available in its existing namespace for backward compatibility. Existing applications that set `ClockProvider.Current` can continue to use that API.

```csharp
ClockProvider.Current = new SystemClock();
```

### Mouse Wheel Interaction (Updated)
The Clock control now follows a standardized modifier-based wheel interaction model:

- **Ctrl** → Change hour  
- **Shift** → Change minute  
- **Ctrl + Shift** → Change second  
- Regular wheel scrolling (no modifiers) continues to adjust minutes.

This update removes the previous Alt-based behavior due to WPF system-level AccessKey/MenuMode limitations and ensures consistent, predictable interaction across all UI elements.

## Current Persian date/time and interval comparisons

```csharp
using PersianDateTimeWPFTools;
using PersianDateTimeWPFTools.Extensions;

PersianDateTimeParts now = PersianDateTime.Now;
string dateText = now.ToString(); // yyyy/MM/dd, invariant Latin digits
TimeSpan time = now.TimeOfDay;    // full tick precision is retained
bool inside = value.IsBetween(startInclusive, endExclusive);
bool expired = deadline.HasExpired(referenceTime);
bool expiredAgainstClock = deadline.HasExpired();
```

`PersianDateTime` is a static facade; parsing returns `System.DateTime` and `Now` returns an immutable `PersianDateTimeParts` snapshot. `Now` uses one configured service snapshot and reads the configured clock once. Default clock time is local. The date-only text does not discard the captured time or `Kind`, and does not read later configuration changes.

`IsBetween` uses `[startInclusive, endExclusive)`, matching `PersianDateRange.Contains`; equal boundaries form an empty interval and reversed boundaries throw. Use the existing `Contains`, not `IsContains`, on a range. `HasExpired` is true at the deadline (`referenceTime >= deadline`). Comparisons retain tick precision, do not convert time zones and require inputs on the same time basis. For a UTC deadline, pass an explicit UTC reference time. The overload without a reference reads `ClockProvider.Current.Now` once.
