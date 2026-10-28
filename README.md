# Persian calendar
## WPF date and time controls with support for eight calendars
![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/all-suppored-calendar.png)
- A free WPF calendar library supporting Persian, Gregorian, Hijri, UmAlQura, Hebrew, Thai Buddhist, Japanese, and Taiwan calendars.
- You can write any style you like for the controls.
- Language change support and other resources

## Quick start

Install the package in a WPF application:

```shell
dotnet add package PersianDateTimeWPFTools
```

Add the package namespace and place a control in your window:

```xml
<Window x:Class="MyApp.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools"
        Title="Calendar" Width="400" Height="450">
    <Grid>
        <pdtt:PersianCalendar />
    </Grid>
</Window>
```

Replace `MyApp.MainWindow` with your window's class name. Default styles load automatically through `Themes/Generic.xaml`. You do not need to merge `InitResources` for the default appearance.

## Supported calendars

The WPF date controls support eight calendars through `CustomCulture.DateTimeFormat.Calendar`:

| Calendar | .NET calendar class | Culture used in the examples |
|---|---|---|
| Persian (Solar Hijri / Shamsi) | `System.Globalization.PersianCalendar` | `fa-IR` |
| Gregorian | `GregorianCalendar` | `en-US` |
| Hijri | `HijriCalendar` | `ar-SA` |
| UmAlQura | `UmAlQuraCalendar` | `ar-SA` |
| Hebrew | `HebrewCalendar` | `he-IL` |
| Thai Buddhist | `ThaiBuddhistCalendar` | `th-TH` |
| Japanese | `JapaneseCalendar` | `ja-JP` |
| Taiwan | `TaiwanCalendar` | `zh-TW` |

This applies to `PersianCalendar`, `PersianDatePicker`, `PersianCalendarWithClock`, and `PersianDateTimePicker`. Their existing names and selection properties are retained. Calendar selection on these controls does not change the calendar-specific contracts of the Persian/Gregorian services and extension methods documented below.

### Select a calendar explicitly

**`CustomCultureName` selects a culture and its default calendar. It does not guarantee a particular calendar implementation.** For example, `he-IL` can use a Gregorian calendar with Hebrew month/day names. Likewise, `ja-JP`, `ar-SA`, and `zh-TW` alone do not explicitly select Japanese, Hijri, or Taiwan calendars.

To choose a calendar explicitly, create a matching culture, assign its `DateTimeFormat.Calendar`, and pass the **whole culture object** to `CustomCulture`:

```csharp
using System.Globalization;

var hebrewCulture = new CultureInfo("he-IL", false);
hebrewCulture.DateTimeFormat.Calendar = new HebrewCalendar();

calendar.CustomCulture = hebrewCulture;
datePicker.CustomCulture = hebrewCulture;
calendarWithClock.CustomCulture = hebrewCulture;
dateTimePicker.CustomCulture = hebrewCulture;
```

The names `calendar`, `datePicker`, `calendarWithClock`, and `dateTimePicker` refer to controls in your window. Apply the assignment to the controls you use.

> Assigning `CustomCultureName = hebrewCulture.Name` would create a culture again using its default calendar. The `HebrewCalendar` assigned above would not be carried over. Assign `CustomCulture = hebrewCulture` instead, and avoid setting both properties for the same selection.

Use the same pattern for the other calendars:

```csharp
using System.Globalization;

static CultureInfo CreateCalendarCulture(
    string cultureName, System.Globalization.Calendar calendar)
{
    var culture = new CultureInfo(cultureName, false);
    culture.DateTimeFormat.Calendar = calendar;
    return culture;
}

var persian = CreateCalendarCulture(
    "fa-IR", new System.Globalization.PersianCalendar());
var gregorian = CreateCalendarCulture("en-US", new GregorianCalendar());
var hijri = CreateCalendarCulture(
    "ar-SA", new HijriCalendar { HijriAdjustment = 0 });
var umAlQura = CreateCalendarCulture("ar-SA", new UmAlQuraCalendar());
var hebrew = CreateCalendarCulture("he-IL", new HebrewCalendar());
var buddhist = CreateCalendarCulture("th-TH", new ThaiBuddhistCalendar());
var japanese = CreateCalendarCulture("ja-JP", new JapaneseCalendar());
var taiwan = CreateCalendarCulture("zh-TW", new TaiwanCalendar());

// Select one configured culture.
datePicker.CustomCulture = hijri;
```

`HijriAdjustment = 0` explicitly selects the unadjusted Hijri calculation. Set this property on your calendar instance when a different adjustment is required.

For culture defaults, the name-based XAML property remains available:

```xml
<pdtt:PersianCalendar CustomCultureName="fa-IR" />
<pdtt:PersianDatePicker CustomCultureName="en-US" />
```

Without a custom culture, the controls use the culture resolved from their WPF `Language` or the current culture. To change calendar or number-format settings later, assign a newly configured `CultureInfo` to `CustomCulture`; editing the internals of an already assigned instance does not itself raise a WPF dependency-property change notification.

### DateTime values and calendar boundaries

Selected values remain ordinary `DateTime` values:

| Controls | Selection property |
|---|---|
| `PersianCalendar`, `PersianDatePicker` | `DateTime? SelectedDate` |
| `PersianCalendarWithClock`, `PersianDateTimePicker` | `DateTime? SelectedDateTime` |
| `Clock`, `ClockPicker` | `DateTime? SelectedTime` |

`DateTime.Year`, `Month`, and `Day` expose Gregorian components even when a control displays another calendar. Use the active calendar to obtain its components or construct a date:

```csharp
var hebrewCalendar = new HebrewCalendar();
DateTime value = hebrewCalendar.ToDateTime(5784, 1, 15, 0, 0, 0, 0);

datePicker.SelectedDate = value;
int year = hebrewCalendar.GetYear(value);       // 5784
int month = hebrewCalendar.GetMonth(value);     // 1: the selected month
int months = hebrewCalendar.GetMonthsInYear(5784); // 13: months in this year
```

Each calendar has its own supported range. Read `MinSupportedDateTime` and `MaxSupportedDateTime` from that calendar rather than assuming `DateTime.MinValue` and `DateTime.MaxValue` are supported. These boundaries are also `DateTime` values: for the Hebrew calendar, their Gregorian years are 1583 and 2239, while `GetYear` returns Hebrew years 5343 and 5999.

Changing the display culture does not change the selected date/time value into a different `DateTime`. If an existing selection falls outside a requested calendar's range, the date picker controls report a date validation error and retain the previous accepted selection/culture.

### Calendar-specific behavior

- **Hijri and UmAlQura:** Each uses its own .NET calendar calculations and supported range. Explicitly assign the desired calendar even though both examples use `ar-SA`.
- **Hebrew:** Common years have 12 months and leap years have 13. The year view adds a fourth row for the extra month and returns to three rows for a common year with the default templates. The thirteenth month is selectable, and month navigation accounts for the actual number of months. Long-date formatting may use Hebrew letters for numbers as defined by .NET's culture formatting.
- **Thai Buddhist:** Year display uses the Buddhist calendar. Short and long input support five-digit years near the upper `DateTime` boundary.
- **Japanese:** Short dates include the era. If a long-date or month-heading pattern omits the era, it is included in the control's display. For example, April 30, 2019 displays as `平成 31/04/30`, while May 1, 2019 displays as `令和 01/05/01`. A transition year's heading shows both eras; its year view keeps the twelve months in the same chronological year. Year and decade navigation remain chronological across era changes. Era names come from the runtime's calendar data.
- **Taiwan:** Months and leap days follow the Gregorian calendar with a different year numbering; for example, Gregorian 2026 is Taiwan year 115.

For manual input, the short date format is `yyyy/MM/dd`, with an era prefix for Japanese dates. Long dates follow the effective culture's long-date pattern, with the calendar-specific handling described above. Keep the era when entering or copying historical Japanese dates.

## Native digits for dates and times

Set `UseNativeDigits="True"` to display digits from the effective culture's `NumberFormat.NativeDigits`. The default is **`false`**, so Latin digits are not replaced.

The property is available on `PersianCalendar`, `PersianDatePicker`, `PersianCalendarWithClock`, `PersianDateTimePicker`, `Clock`, and `ClockPicker`.

```xml
<pdtt:PersianCalendar CustomCultureName="fa-IR" UseNativeDigits="True" />
<pdtt:PersianDatePicker CustomCultureName="fa-IR" UseNativeDigits="True" />
<pdtt:Clock CustomCultureName="fa-IR" UseNativeDigits="True" />
<pdtt:ClockPicker CustomCultureName="fa-IR"
                  UseNativeDigits="True"
                  TimeFormat="HH:mm" />

<pdtt:PersianCalendarWithClock CustomCultureName="fa-IR" UseNativeDigits="True" />
<pdtt:PersianDateTimePicker CustomCultureName="fa-IR" UseNativeDigits="True" />
```

These examples use culture defaults. For explicit calendar selection, assign the configured `CustomCulture` as shown above, then enable the option:

```csharp
datePicker.CustomCulture = hebrew;
datePicker.UseNativeDigits = true;

// Restore Latin digit display without changing the selected value.
datePicker.UseNativeDigits = false;
```

The setting covers day numbers, calendar headings, year/decade labels, date fields, clock dial numbers, and formatted time text. Composite controls forward the culture and digit option to their nested calendar and clock; `ClockPicker` also applies them to its popup clock. `Clock` and `ClockPicker` inherit `CustomCulture`, `CustomCultureName`, and `UseNativeDigits` from `ClockBase`.

- Toggling digit display does not change the selected date/time or raise a selection-change event by itself.
- Native input digits are normalized before parsing. Persian, Arabic-Indic, and Latin input remain accepted; custom native digits from the assigned culture are supported too.
- Clock calculations and dial selection continue to use numeric values rather than the localized labels.
- Committing unchanged `ClockPicker` text preserves precision hidden by `TimeFormat`, including seconds and fractional ticks. An actual time edit uses the existing parsed-field behavior.
- A culture's native digits may already be Latin. In that case, enabling the option produces no digit-shape change. The property uses `NativeDigits`; it does not invent a separate numeral system.

Calendar culture and digit display are separate from the built-in `fa`/`en` language-resource dictionaries. Labels such as Today, Confirm, and AM/PM still use the resource and property mechanisms described below; selecting a calendar culture does not translate those resource dictionaries.

## Calendar events

Calendar events are optional, calculated offline, and independent of the calendar used to display a date. The feature follows the existing configuration/service/provider pattern:

- `CalendarEventConfiguration.Configure(...)`, `.Services`, and `.Reset()` in the root namespace.
- `CalendarEventConfigurationOptions.EventProvider` for replacing the implementation.
- `ICalendarEventProvider` in `PersianDateTimeWPFTools.Abstractions`.
- `CalendarEventDefinition`, rules, and occurrences in `PersianDateTimeWPFTools.Events`.
- Default providers in `PersianDateTimeWPFTools.Services.CalendarEvents`.
- `PersianDateTimeWPFTools.Providers.CalendarEventProvider.Current` as a provider facade.

### Enable the Iranian preset

Configure the data source, then opt in on the date controls that should display it:

```csharp
using PersianDateTimeWPFTools;
using PersianDateTimeWPFTools.Services.CalendarEvents;

CalendarEventConfiguration.Configure(options =>
{
    options.EventProvider = new IranianCalendarEventProvider();
});
```

```xml
<pdtt:PersianCalendar ShowCalendarEvents="True" />
<pdtt:PersianDatePicker ShowCalendarEvents="True" />
<pdtt:PersianCalendarWithClock ShowCalendarEvents="True" />
<pdtt:PersianDateTimePicker ShowCalendarEvents="True" />
```

Use the package's existing `pdtt` XML namespace. `ShowCalendarEvents` defaults to `false`. The global default provider is empty: no region is assumed and there are no provider queries while the feature is disabled.

The included Iranian starter set has 41 recurring occasions. Persian dates use `PersianCalendar`; lunar dates use `HijriCalendar { HijriAdjustment = 0 }` by default. Definitions repeat across years without annual date files, network access, or a new package version. Titles and month/day definitions are editable by copying the preset below. This is calculated event display, not an automatic change to official holiday or working-day policies.

To use a different lunar calculation for the preset:

```csharp
using System.Globalization;

CalendarEventConfiguration.Configure(options =>
{
    options.EventProvider = new IranianCalendarEventProvider(
        new HijriCalendar { HijriAdjustment = 1 });
    // Alternatively: new IranianCalendarEventProvider(new UmAlQuraCalendar())
});
```

### Define your own events

```csharp
using System;
using System.Globalization;
using PersianDateTimeWPFTools.Events;
using PersianDateTimeWPFTools.Services.CalendarEvents;

var definitions = new[]
{
    new CalendarEventDefinition("my.nowruz", "Nowruz",
        new AnnualCalendarEventRule(new System.Globalization.PersianCalendar(), 1, 1)),
    new CalendarEventDefinition("my.ashura", "Ashura",
        new AnnualCalendarEventRule(new HijriCalendar { HijriAdjustment = 0 }, 1, 10)),
    new CalendarEventDefinition("my.meeting", "Personal meeting",
        new FixedCalendarEventRule(new DateTime(2026, 3, 21)))
};

CalendarEventConfiguration.Configure(options =>
    options.EventProvider = new RuleBasedCalendarEventProvider(definitions));
```

An annual rule takes **calendar, month, day**. Built-in annual rules support Persian, Gregorian, Hijri, and UmAlQura source calendars. The calendar is cloned when the rule is constructed; subsequent changes to the caller's calendar do not change existing rules.

Events from all these source calendars can be displayed on any of the eight supported display calendars. Japanese, Hebrew, Buddhist, and Taiwan display does not require annual rules in those calendars. For other recurrence systems, implement `CalendarEventRule` or supply a custom `ICalendarEventProvider`.

`FixedCalendarEventRule` is a one-date event. Event dates are date-only: time-of-day and time-zone conversion are outside this API.

Invalid annual dates are skipped by default, for example February 29 in a common year or Esfand 30 in a non-leap Persian year. Clamping must be explicit:

```csharp
var rule = new AnnualCalendarEventRule(new GregorianCalendar(), 2, 29,
    InvalidCalendarEventDateBehavior.ClampToLastDay);
```

The preset uses explicit clamping for occasions defined at the end of a lunar month. No silent adjustment is applied to ordinary annual rules. Impossible month/day definitions are rejected unless clamping is requested.

### Edit or remove predefined occasions

Definitions are immutable snapshots with stable, case-sensitive IDs. Copy the set to customize it; the preset and other controls are not mutated:

```csharp
using System.Linq;

var definitions = IranianCalendarEventProvider.CreateDefaultEvents()
    .Where(e => e.Id != "ir.lunar.ashura")
    .ToList();

definitions.Add(new CalendarEventDefinition(
    "ir.lunar.ashura", "My Ashura title",
    new AnnualCalendarEventRule(new HijriCalendar { HijriAdjustment = 0 }, 1, 10)));

CalendarEventConfiguration.Configure(options =>
    options.EventProvider = new RuleBasedCalendarEventProvider(definitions));
```

Inspect `CreateDefaultEvents()` or the provider's `Events` property to discover IDs. IDs must be unique within a `RuleBasedCalendarEventProvider`. Replacing a title does not require changing the ID. Definitions can also be replaced with fixed-date or custom rules.

### Query without a WPF control

```csharp
var services = CalendarEventConfiguration.Services;
var todayEvents = services.GetEvents(DateTime.Today);
var monthEvents = services.GetEvents(
    new DateTime(2026, 3, 1), new DateTime(2026, 4, 1));
```

Ranges are start-inclusive and end-exclusive. Start is normalized to its date; the exclusive end must be midnight. `DateTime.MaxValue` is a special final boundary so that `GetEvents(DateTime.MaxValue.Date)` is supported. All occurrences contain a date and their definition, ID, and title. Multiple events on one date are retained and ordered by descending `Priority`, then ordinal ID.

Custom providers implement the same range contract and must be safe for concurrent reads. Their `GetEvents` method is synchronous; load external data before supplying it to the display path. The built-in providers do not perform I/O.

`Configure` publishes a new service snapshot; loaded calendars refresh on their UI dispatcher. Hidden popup calendars refresh when loaded. `Reset()` returns to the empty provider and refreshes loaded calendars. For a custom provider whose internal data changes without a configuration update, call `RefreshCalendarEvents()` on the relevant date control from its UI thread.

### Tooltip and event highlight

Existing `DayMetadata` objects and legacy indicator APIs remain available. When events are enabled:

- Existing tooltip content appears first, followed by the day's event titles.
- A `CalendarDayInfo.ToolTipTemplate` applies to the original tooltip content, not to the event list. The legacy `DayToolTipTemplate` is also honored during composition.
- Existing user indicators remain independent of the new event highlight.
- Events do not alter `SelectedDate`, `SelectedDateTime`, blackout dates, working-day services, or enabled state.
- Disabled-day tooltip behavior continues to follow `CalendarDayInfo.ShowToolTipWhenDisabled`.

Arbitrary tooltip data and templates are supported. An explicit `ToolTip` object's body is temporarily presented inside the combined tooltip while open and restored when it closes; the metadata dictionary and original tooltip reference are not replaced. Application-specific tooltip subclasses with custom opening behavior should be tested with composition or displayed through a custom event template.

| Property on the four date controls | Default | Purpose |
|---|---|---|
| `ShowCalendarEvents` | `false` | Enable event lookup and presentation |
| `ShowCalendarEventToolTips` | `true` | Append events to the existing tooltip |
| `ShowCalendarEventHighlights` | `true` | Show the independent layer behind the day number |
| `CalendarEventHighlightStyle` | `null` | Optional style targeting `Border` |
| `CalendarEventTemplate` | `null` | Optional tooltip item template for each `CalendarEventOccurrence` |

The two presentation switches have an effect only while `ShowCalendarEvents` is enabled.

```xml
<pdtt:PersianCalendar ShowCalendarEvents="True">
    <pdtt:PersianCalendar.CalendarEventHighlightStyle>
        <Style TargetType="Border">
            <Setter Property="Background" Value="#182196F3" />
            <Setter Property="CornerRadius" Value="5" />
            <Setter Property="Margin" Value="3" />
        </Style>
    </pdtt:PersianCalendar.CalendarEventHighlightStyle>
    <pdtt:PersianCalendar.CalendarEventTemplate>
        <DataTemplate>
            <TextBlock Text="{Binding Title}" TextWrapping="Wrap" />
        </DataTemplate>
    </pdtt:PersianCalendar.CalendarEventTemplate>
</pdtt:PersianCalendar>
```

All five built-in themes have the event layer between their backgrounds and day text. Existing custom day-button styles are not replaced. A fully custom day-button `ControlTemplate` must add its own layer or use the attached state:

- `CalendarDayButtonExtensions.CalendarEvents`: the day's occurrence list.
- `CalendarDayButtonExtensions.ShowCalendarEventHighlight`: whether the built-in highlight should be shown.
- `CalendarDayButtonExtensions.CalendarEventHighlightStyle`: the selected highlight style.

No legacy API is marked obsolete by this feature.

## ClockPicker

`ClockPicker` combines an editable time field with the existing `Clock` in a popup. It derives from `ClockBase` and keeps the same `DateTime? SelectedTime`, `DateTime DisplayTime`, `TimeFormat`, `SelectedTimeChanged` and `Confirmed` contracts.

```xml
<pdtt:ClockPicker SelectedTime="{Binding AppointmentTime}"
                  TimeFormat="HH:mm:ss"
                  Watermark="Select a time" />
```

Use a `DateTime?` view-model property; selection binds two-way by default. The control edits the time of day while preserving the selected/displayed date and `Kind`. Default styles work without initialization, and all five existing themes are supported.

- Type a time and press Enter, or open the popup and use the clock. Seconds are optional: `14:30` is accepted with the default `HH:mm:ss` format and displayed as `14:30:00`.
- Clock interactions update the selection immediately. Escape restores the value captured on opening; closing by clicking outside keeps the current value.
- The confirmation button or Enter in the popup confirms the displayed time and closes it. `ShowConfirmButton="False"` hides the footer.
- Empty text clears the selection. Invalid text raises `TimeValidationError` and restores the previous formatted value.
- Supported formats include `HH:mm`, `HH:mm:ss`, `hh:mm tt`, `HH:mm:ss.fff`, `t` and `T`. Formatting/parsing use the effective culture selected by `CustomCulture`, `CustomCultureName`, or the culture fallback. Persian and Arabic-Indic input digits are accepted; `UseNativeDigits` enables localized digit display. Date-bearing formats are rejected.
- `IsReadOnly`, `ClockStyle`, `ConfirmButtonContent` and `Watermark` support customization. Explicit styles keep WPF's normal precedence.
- F4 or Alt+Down toggles the popup. Escape cancels popup changes; Tab continues focus navigation.
- UI Automation Value and ExpandCollapse patterns are supported.

`Text` validates immediately when set programmatically; editor typing is committed on Enter, focus loss or opening the popup. Text that already matches the formatted selection retains hidden precision; changed text uses the parsed fields (for example, editing an `HH:mm` value resets seconds to zero).

For localized AM/PM and confirmation labels, change the language on your merged `InitResources` instance. New `ClockBase.AmDesignator` and `PmDesignator` properties provide English fallbacks when no language resources are merged. The default confirmation label is also available without initialization.

## Themes and resources

### Select a startup theme

To select an application-wide theme, merge one `InitResources` instance into `App.xaml`:

```xml
<Application x:Class="MyApp.App"
             xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
             xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
             xmlns:pdtt="https://github.com/Behnam2064/PersianDateTimeWPFTools"
             StartupUri="MainWindow.xaml">
    <Application.Resources>
        <ResourceDictionary>
            <ResourceDictionary.MergedDictionaries>
                <pdtt:InitResources SelectedTheme="LightModern1" />
            </ResourceDictionary.MergedDictionaries>
        </ResourceDictionary>
    </Application.Resources>
</Application>
```

Replace `MyApp.App` with your application's class name. Omit `SelectedTheme` to keep the default theme. `InitResources` also exposes the package's named styles and language resources. Merge it once; do not also merge `Resources.xaml` directly.

| Theme | Enum value | Theme class |
|---|---|---|
| Default | `BaseThemeName.Default` | `ThemeDefault` |
| Dark Modern | `BaseThemeName.DarkModern1` | `ThemeDarkModern1` |
| Light Modern | `BaseThemeName.LightModern1` | `ThemeLightModern1` |
| Dark Classic | `BaseThemeName.DarkClassic` | `ThemeDarkClassic` |
| Light Classic | `BaseThemeName.LightClassic` | `ThemeLightClassic` |

### Switch themes at runtime

Call `SetTheme` on the application's UI thread after the WPF `Application` has been created, for example in a button handler or `OnStartup`:

```csharp
using PersianDateTimeWPFTools.Themes;
using PersianDateTimeWPFTools.Tools;

InitResources.SetTheme(new ThemeDarkModern1());

// Switch again without recreating controls.
InitResources.SetTheme(new ThemeLightClassic());

// Return to the default appearance.
InitResources.SetTheme(new ThemeDefault());
```

These calls demonstrate alternatives; call the one matching the user's selection. Existing and newly created controls use the selected theme through implicit styles. Theme changes replace the library's active resource dictionary and preserve the selected language.

`SetTheme` works without merging `InitResources` first. It requires a WPF `Application` and its UI thread. Passing `null` or an invalid theme is rejected. `SelectedTheme` and `InitResources.Theme` represent application-wide state; setting `SelectedTheme` on a merged instance uses the same theme-switching API. Independent themes for individual windows are not provided by this API.

### Custom styles and resource precedence

The controls no longer assign a local `Style` in their constructors. Explicit styles and implicit styles defined closer to a control retain WPF's normal precedence. Direct entries in `Application.Resources` take precedence over merged dictionaries; later merged dictionaries take precedence over earlier ones.

To follow application-wide theme changes, leave the control's `Style` unset. An explicitly assigned named style stays assigned when the theme changes. Existing named style keys remain available through `InitResources` for applications that intentionally choose a fixed style.

For example, this explicit style preserves the default template while customizing a property:

```xml
<Window.Resources>
    <Style x:Key="LargeCalendar" TargetType="{x:Type pdtt:PersianCalendar}">
        <Setter Property="FontSize" Value="18" />
    </Style>
</Window.Resources>

<!-- Place inside the window's content. -->
<pdtt:PersianCalendar Style="{StaticResource LargeCalendar}" />
```

The example's style remains in place during theme changes. For customization that follows the active theme, prefer setting individual control properties instead of assigning a fixed `Style`.

### Theme notifications

```csharp
using System;
using PersianDateTimeWPFTools.Tools;

InitResources.OnTheme += HandleThemeChanged;

// Unsubscribe when the subscriber no longer needs notifications.
InitResources.OnTheme -= HandleThemeChanged;

void HandleThemeChanged(object sender, ThemeChangedEventArgs e)
{
    Console.WriteLine($"Theme changed from {e.OldTheme.Theme} to {e.NewTheme.Theme}");
}
```

`OnTheme` is a static event, raised after a successful theme change. Its sender is `null`. Manage subscriptions for the subscriber's lifetime. The controls update through resource lookup and do not need to subscribe to this event.

## Control language resources

Built-in language codes are `fa` and `en`. When constructed, `InitResources` selects Persian if `CultureInfo.CurrentUICulture.TwoLetterISOLanguageName` is `fa`; otherwise it selects English. Without `InitResources`, the default resource fallback is English.

To change the language at runtime, update the same instance that is merged into the application's resources. With the `App.xaml` example above:

```csharp
using System.Linq;
using System.Windows;
using PersianDateTimeWPFTools.Tools;

var calendarResources = Application.Current.Resources.MergedDictionaries
    .OfType<InitResources>()
    .Single();

calendarResources.ChangeLanguage("fa");
// Or: calendarResources.ChangeLanguage("en");
```

This example assumes exactly one `InitResources` instance is merged directly into `Application.Resources`. Keep a reference to the instance yourself if your resource structure is different.

For initialization entirely in code, create and merge the instance before using it:

```csharp
using System.Windows;
using PersianDateTimeWPFTools.Tools;

public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);

        // Use this approach when InitResources is not already merged in App.xaml.
        var calendarResources = new InitResources();
        calendarResources.ChangeLanguage("fa");
        Resources.MergedDictionaries.Add(calendarResources);
    }
}
```

Constructing `InitResources` alone does not modify `Application.Resources`. Calling `new InitResources().ChangeLanguage("fa")` without merging that instance therefore does not change the application's controls. Language changes replace only the language dictionary owned by that instance.

Language resources supply labels such as AM, PM, Today and Confirm. They are separate from `CustomCulture` and `CustomCultureName`, which configure date/time presentation and, for date controls, the calendar. Change those control properties separately when required.

### Custom language dictionaries

Create a WPF resource dictionary with the same keys as the built-in language resources. For example, add `Lang.es.xaml` to your application with build action `Page`:

```xml
<ResourceDictionary
    xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
    xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
    xmlns:sys="clr-namespace:System;assembly=mscorlib">
    <sys:String x:Key="AmText">a. m.</sys:String>
    <sys:String x:Key="PmText">p. m.</sys:String>
    <sys:String x:Key="ConfirmText">Confirmar</sys:String>
    <sys:String x:Key="TodayText">Hoy</sys:String>
</ResourceDictionary>
```

Load it through the merged instance, replacing `MyApp` with your application's assembly name:

```csharp
calendarResources.ChangeLanguage(
    "es",
    "pack://application:,,,/MyApp;component/Lang.es.xaml");
```

A language other than `fa` or `en` requires `lanAddress`. Run language changes on the application UI thread. Switching themes does not reset the selected language dictionary.

## Migration: resources and theming

- Default styles now load automatically. Remove initialization used only to make the default controls visible.
- Keep one merged `InitResources` instance if you need language selection or named styles. Do not depend on its constructor to modify application resources.
- Replace language changes on temporary, unmerged instances with changes on the merged instance.
- `OnTheme` is now an event: subscribe with `+=` and unsubscribe with `-=`. Direct assignment and invocation from consumer code are no longer supported; the sender is now `null`.
- `ThemeChangedEventArgs.OldTheme` now contains the previous theme.
- `InitResources(bool noInit)` is obsolete. Use the parameterless constructor.
- Review explicitly assigned styles if controls should follow runtime theme changes; those styles retain their normal precedence.


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
| `TryGetNthWeekdayOfPersianMonth(PersianDayOfWeek.Friday, 2, out var result)` | `1405/07/10 15:30:45` |
| `GetLastWeekdayOfPersianMonth(PersianDayOfWeek.Friday)` | `1405/07/24 15:30:45` |


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
| `StartOfWeek()` | `2026-10-19 00:00:00` |
| `StartOfNextWeek()` | `2026-10-26 00:00:00` |
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

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/clock.png)
- PersianCalendar

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pc.png)
- PersianCalendarWithClock

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdpwc.png)
- PersianDatePicker

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdp.png)
- PersianDateTimePicker

![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/pdtp.png)

- ClockPicker 
editable time field with a popup clock; see the ClockPicker section above.
![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/cpdl.png)

For calendar selection and culture configuration, see [Supported calendars](#supported-calendars). Use `CustomCultureName` for a culture's default calendar, or assign a configured `CustomCulture` to select a specific calendar. For localized date and clock digits, see [Native digits for dates and times](#native-digits-for-dates-and-times).

## Ability to write custom styles for all controls
![IMAGE_DESCRIPTION](https://raw.githubusercontent.com/Behnam2064/PersianDateTimeWPFTools.doc/refs/heads/main/Persian-calendar-Custom-Style.png)


## Dependency Properties and Features
| Name        | Description           | Default  |
| :------------- |:-------------| :-----|
| CustomCulture | A configured `CultureInfo`; its `DateTimeFormat.Calendar` selects the date calendar | Resolved from `Language` or the current culture |
| CustomCultureName | Selects a culture by name and uses its default calendar; use `CustomCulture` for an explicit calendar | Resolved from `Language` or the current culture |
| UseNativeDigits | Display culture-native digits in date and time controls | `false` |
| ShowCalendarEvents | Enable configured event lookup and display on date controls | `false` |
| ShowCalendarEventToolTips | Append events after existing tooltip content | `true` |
| ShowCalendarEventHighlights | Display a separate layer behind event-day numbers | `true` |
| CalendarEventHighlightStyle | Optional event-layer style targeting `Border` | `null` |
| CalendarEventTemplate | Tooltip item template for `CalendarEventOccurrence` | `null` |
| AllowSelectBlackedOutDay | Selectable holidays      |    False |
| ShowTodayButton | Show a button to move to the current day   |    False |
| ShowConfirmButton | Show a button to display confirm button   |    False |
| DisplayDate | Displays the current date   |    ? |
| DisplayDateStart | Start displaying the date   |    ? |
| DisplayDateEnd | End of displaying date   |    ? |
| DisplayMode | Date display type (like Month,Year,Decade)   |    Month |
| FirstDayOfWeek | First day of the week; can be explicitly overridden | Effective culture's first day of the week |
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



