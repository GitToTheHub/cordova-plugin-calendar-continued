# cordova-plugin-calendar-continued

[![npm - Latest](https://img.shields.io/npm/v/cordova-plugin-calendar-continued/latest?label=Latest%20Release%20(npm))](https://npmjs.com/package/cordova-plugin-calendar-continued)
[![GitHub](https://img.shields.io/github/package-json/v/gittothehub/cordova-plugin-calendar-continued?label=Development%20(Git))](https://github.com/GitToTheHub/cordova-plugin-calendar-continued)

This plugin allows you to add events to the Calendar of the mobile device.

> [!NOTE]
> This is a fork of the [Calendar-PhoneGap-Plugin](https://github.com/EddyVerbruggen/Calendar-PhoneGap-Plugin) by [EddyVerbruggen](https://github.com/EddyVerbruggen), which will continue the support of this plugin to the community. If you like, you can support me on my [GitHub Sponsor page](https://github.com/sponsors/GitToTheHub).

## Supported platforms

- Android
- iOS

## iOS Specifics

### Calendar Usage Description

You need to provide a reason to the user for Calendar access. This plugin adds the key `NSCalendarsUsageDescription` to your App's Info.plist with the default text `This app uses your calendar`, which you can override with the preference key `CALENDAR_USAGE_DESCRIPTION`. To do so, pass the following variable when installing the plugin:

```bash
cordova plugin add cordova-plugin-calendar-continued --variable CALENDAR_USAGE_DESCRIPTION="My custom calendar usage reason"
```

### Contacts usage description

You need to provide a reason to the user for contacts access. This is needed in some iOS versions when searching for locations and invitees using the interactive mode like `createEventInteractively`. This plugin adds the key `NSContactsUsageDescription` to your App's Info.plist with the defaul text `This app uses contacts when searching for locations and invitees when using the calendar functionality`, which you can override with the preference key `CONTACTS_USAGE_DESCRIPTION`. To do so, pass the following variable when installing the plugin:

```bash
cordova plugin add cordova-plugin-calendar-continued --variable CONTACTS_USAGE_DESCRIPTION="My custom contacts usage reason"
```

## Android Specifics

### Android Permissions

Since Android 6 permissions need to be requested to use the Calendar at runtime when targeting API level 23+. Since plugin version 4.5.0 this is handled automatically in a just-in-time manner. If you call `createEvent` the plugin will pop up a permission dialog, after the user granted access to his calendar the event will be created.

You can also manually manage and check permissions if that's your thing.
Note that the `hasPermission` functions will return `true` when:

- You're running on iOS
- You're targeting on Android an API level lower than 23 (Android 6)
- You've already granted permission.

```js
  function hasReadWritePermission() {
    window.plugins.calendar.hasReadWritePermission(
      function(result) {
        // if this is 'false' you probably want to call 'requestReadWritePermission' now
        alert(result);
      }
    )
  }

  function requestReadWritePermission() {
    // no callbacks required as this opens a popup which returns async
    window.plugins.calendar.requestReadWritePermission();
  }
```

There are similar methods for Read and Write access only (`hasReadPermission`, etc),
although it looks like that if you request read permission you can write as well,
so you might as well stick with the example above.

Note that backward compatibility was added by checking for read or write permission in the relevant plugins functions.
If permission is needed the plugin will now show the permission request popup.
The user will then need to allow access and invoke the same method again after doing so.

## Installation

From npm (Release version):

```bash
cordova plugin add cordova-plugin-calendar-continued
```

From GitHub (Development version):

```bash
cordova plugin add https://github.com/GitToTheHub/cordova-plugin-calendar-continued
```

## Usage

### Calendar Methods

These methods are for handling a calendar itself and not the events on it.

| Method | Support |
| ---    | ---     |
| [createCalendar](#create-calendar) | Android, iOS |
| [deleteCalendar](#delete-calendar) | Android, iOS |
| [listCalendars](#list-calendars) | Android, iOS |
| [openCalendar](#open-calendar-app) | Android, iOS |

#### Create Calendar

The `createCalendar` method will take as its first parameter a calender name or an options object.

If the calender did not already exist, the success callback will deliver the calendar id, which will be on Android an int like `1` or on iOS an UUID like `C50FED73-B77E-4DEF-91C2-5DA4E5191162`. If the calender already exists, the result will a string message like `OK, Calendar already exists` on iOS, or on Android it will be just `null`. 

##### Create Calendar By Name

```js
window.plugins.calendar.createCalendar(
  "My Calendar Name",
  (result) => {
    console.log(`Calendar created, result=${result}`)
  },
  (errorMessage) => {
    console.error(`Error creating calendar: ${errorMessage}`)
  }
)
```

##### Create Calendar By Options

```js
const createCalOptions = window.plugins.calendar.getCreateCalendarOptions()
// Android: Set the local account name where the local calendar
// is created under. If you don't set it, the app name will be
// used as account name.
createCalOptions.accountName = "My Account name"
// The calendar name, Android and iOS
createCalOptions.calendarName = "My Calendar Name"
// Optional, set the calendar color by a hex color like `#FF0000"`, default is null, so the OS picks a color.
createCalOptions.calendarColor = "#FF0000"

// Create a calendar with options
window.plugins.calendar.createCalendar(
  createCalOptions,
  (result) => {
    console.log(`Calendar created, result=${result}`)
  },
  (errorMessage) => {
    console.error(`Error creating calendar, errorMessage=${errorMessage}`);
  }
)
```

##### Android Specifics

On Android a local calendar will be created in a local account which get not synced and which is since 2025 not visible by default in the Google Calendar App. A user have to enable the local account, which Google calls a `non-Google account` in the settings of the app, see for this [Turn on a non-Google account](https://support.google.com/calendar/answer/13322290?sjid=17399977447742721718-EU#enable-non-google-account).

#### Delete Calendar

```js
window.plugins.calendar.deleteCalendar(
  "My Calendar Name", 
  (message) => {
    console.log(`Calender deleted successfully, message=${message}`)
  },
  (errorMessage) => {
    console.log(`Error during deleting calendar, errorMessage=${errorMessage}`)
  }
)
```

#### List Calendars

Get a list of all calendars. The success callback returns an array of calendar objects.

```js
window.plugins.calendar.listCalendars(
  // Success callback with an array of calendar objects.
  (calendars) => {
    console.log(`Successfully could get list of calendars, calendars=${JSON.stringify(calendars)}`)
  },
  (errorMessage) => {
    console.error(`Could not get list of calendars, errorMessage=${errorMessage}`)
  }
)
```

A calendar object contains these properties:

| Property | Support | Description |
| --- | --- | --- |
| `id` | Android, iOS | Calendar identifier. On Android this is a numeric string like `"1"`. On iOS this is a UUID string. |
| `name` | Android, iOS | Calendar name. |
| `displayname` | Android | Calendar display name. |
| `isPrimary` | Android | Whether this is the primary calendar. |
| `type` | iOS | Calendar type, for example `Local`, `CalDAV`, `Exchange`, `Subscription`, `Birthday`, or `Mail`. |

Example calendar object on iOS:

```js
{
  id: "258B0D99-394C-4189-9250-9488F75B399D",
  name: "standard calendar",
  type: "Local"
}
```

#### Open Calendar App

You can open the calendar app by calling `openCalendar`. Calling it will no parameter will open it at today:

```js
window.plugins.calendar.openCalendar()
```

You can open the calendar app at a specific date. The success and error callback are optional.

```js
// Open the calendar app at a specific date, here today + 3 days
const date = new Date(new Date().getTime() + 3 * 24 * 60 * 60 * 1000)
window.plugins.calendar.openCalendar(
  date,
  () => {
    console.log(`Calenar app sucessfully opened`)
  },
  (errorMessage) => {
    console.error(`Could not open calendar app, errorMessage=${errorMessage}`)
  }
)
```

### Event Methods

These methods are for handling the calendar events.

| Method | Support |
| ---    | ---     |
| [getCalendarOptions](#get-calendar-options) | Android, iOS |
| [createEvent](#create-event-without-options) | Android, iOS |
| [createEventWithOptions](#create-event-with-options) | Android, iOS |
| [createEventInteractively](#create-event-interactively) | Android, iOS |
| [createEventInteractivelyWithOptions](#create-event-interactively-with-options) | Android, iOS |
| [findEvent](#find-event) | Android, iOS |
| [findEventWithOptions](#find-event-with-options) | Android, iOS |
| [listEventsInRange](#list-events-in-range) | Android, iOS |
| [findAllEventsInNamedCalendar](#find-all-events-in-named-calendar) | iOS |
| [modifyEvent](#modify-event) | iOS |
| [modifyEventWithOptions](#modify-event-with-options) | iOS |
| [deleteEvent](#delete-event) | Android, iOS |
| [deleteEventFromNamedCalendar](#delete-event-from-named-calendar) | iOS |
| [deleteEventById](#delete-event-by-id) | Android, iOS |

#### Create event

There a two methods for creating an event: `createEvent` and `createEventWithOptions`. You should prefer using `createEventWithOptions` where you can choose the calendar where the events should be created. On `createEvent` you cannot choose the calendar and defaults will be used.

The success callback of the create event methods will deliver the event id. On Android this will be an int like `1`. On iOS this will be a calendar/event identifier string containing UUID segments like `B12B87C3-6721-4FE7-AE1C-B71600B09AB7:9C9E6CF8-3335-4A7D-B81F-C0ED34B7B54D`.

##### Create Event Without Options

Calling `createEvent` will choose a default calendar where the event will be created which can lead to unspecific behaviour, prefere using [createEventWithOptions](#create-event-with-options).

```js
window.plugins.calendar.createEvent(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  // Note: On JavaScript, the first Month starts at 0, the last month ends at 11.
  // So 7 means August, not July
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  (eventId) => {
    console.log(`Event created, eventId=${eventId}`)
  },
  (errorMessage) => {
    console.error(`Error creating event, errorMessage=${errorMessage}`);
  }
);
```

##### Create Event With Options

`createEventWithOptions` will let you choose the calendar where the event will be created. You have to create an options object by [`getCalendarOptions`](#get-calendar-options) where you can set more options for the event, for e.g. you can set the calendar where the event should be created on.

```js
// Create the options object first
const options = window.plugins.calendar.getCalendarOptions();
// Modify the options object
// Example: Change the first reminder from 1 hour to 2 hours before
options.firstReminderMinutes = 120;
// Set calender name which applies only on iOS
options.calendarName = "My Calendar Name";
// Set calendar id which applies only on Android
options.calendarId = 1;

window.plugins.calendar.createEventWithOptions(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  // Note: On JavaScript, the first Month starts at 0, the last month
  // ends at 11. So 7 means August, not July
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // Add here your options you created by `getCalendarOptions`
  options,
  (eventId) => {
    console.log(`Event created, eventId=${eventId}`)
  },
  (errorMessage) => {
    console.error(`Error creating event, errorMessage=${errorMessage}`);
  }
);
```

##### Create All-Day Event

On iOS, an event is treated as all-day when the date range covers full days from midnight to midnight. On Android, set `options.allday = true`; for silent event creation Android also checks that the event duration is a whole number of days. Use midnight-to-midnight dates to avoid timezone surprises.

```js
// Set the start date to midnight and the end date to midnight the next day.
const startDate = new Date(2027, 2, 15, 0, 0, 0, 0, 0)
const endDate = new Date(2027, 2, 16, 0, 0, 0, 0, 0)

const options = window.plugins.calendar.getCalendarOptions()
// Android only: mark this event as all-day.
options.allday = true

window.plugins.calendar.createEventWithOptions(
  "My all-day event",
  "Home",
  "Some notes about this event.",
  startDate,
  endDate,
  options,
  () => {
    console.log("All-day event created")
  },
  (errorMessage) => {
    console.error(`Error creating all-day event, errorMessage=${errorMessage}`)
  }
)
```

##### Create Multi-Day All-Day Event

```js
// Set the start date to midnight and the end date to midnight 3 days later.
const startDate = new Date(2027, 2, 24, 0, 0, 0, 0, 0)
const endDate = new Date(2027, 2, 27, 0, 0, 0, 0, 0)

const options = window.plugins.calendar.getCalendarOptions()
// Android only: mark this event as all-day.
options.allday = true

window.plugins.calendar.createEventWithOptions(
  "My 3-day event",
  "Home",
  "Some notes about this event.",
  startDate,
  endDate,
  options,
  () => {
    console.log("3-day event created")
  },
  (errorMessage) => {
    console.error(`Error creating 3-day event, errorMessage=${errorMessage}`)
  }
)
```

##### Get Calendar Options

`getCalendarOptions` returns the default options object used by the event methods that accept options:

- `createEventWithOptions`
- `createEventInteractivelyWithOptions`
- `findEventWithOptions`
- `modifyEventWithOptions`

Start with this object and override only the properties you need. The JavaScript wrapper merges your options with these defaults before sending them to the native platform code. Not every method and platform uses every property: most properties configure event creation, while `findEventWithOptions` only uses event id and calendar filtering options and `modifyEventWithOptions` uses one options object to find the existing event and another options object for the new event values.

`getCalendarOptions` returns these properties:

| Property | Default | Support | Description |
| --- | --- | --- | --- |
| `firstReminderMinutes` | `60` | Android, iOS | Adds the first reminder this many minutes before the event. Set to `null` to disable it. Not supported by Android's interactive event editor. |
| `secondReminderMinutes` | `null` | Android, iOS | Adds a second reminder this many minutes before the event. Not supported by Android's interactive event editor. |
| `recurrence` | `null` | Android, iOS | Makes the event recurring. Supported values are `daily`, `weekly`, `monthly`, and `yearly`. |
| `recurrenceInterval` | `1` | Android, iOS | Repeat interval for `recurrence`, for example `2` with `monthly` means every 2 months. |
| `recurrenceWeekstart` | `"MO"` | Android only | RRULE `WKST` value, for example `MO` or `SU`. Supported by silent event creation only. |
| `recurrenceByDay` | `null` | Android only | RRULE `BYDAY` value, for example `MO,WE,FR`. Supported by silent event creation only. |
| `recurrenceByMonthDay` | `null` | Android only | RRULE `BYMONTHDAY` value, for example `15`. Supported by silent event creation only. |
| `recurrenceEndDate` | `null` | Android, iOS | JavaScript `Date` at which the recurrence ends. Leave `null` for no end date. |
| `recurrenceCount` | `null` | Android only | RRULE `COUNT` value, for example `5` to repeat 5 times. Supported by silent event creation only. |
| `allday` | `null` | Android only | Set to `true` for an all-day event. For silent event creation, Android also checks that the event duration is a whole number of days. Use midnight-to-midnight dates to avoid timezone surprises. iOS ignores this option and detects all-day events from the date range. |
| `calendarName` | `null` | iOS | Selects the calendar by name or identifier. If omitted, iOS uses the default calendar for new events. Android ignores this for event creation. |
| `calendarId` | `null` | Android | Selects the Android calendar id. If omitted during event creation, Android defaults to `1`. iOS ignores this. |
| `url` | `null` | Android, iOS | Adds a URL to the event. iOS stores it as the event URL. Android appends it to the event notes because Android calendar events do not expose the same URL field. |

Example response from `getCalendarOptions`:

```js
{
  firstReminderMinutes: 60,
  secondReminderMinutes: null,
  recurrence: null,
  recurrenceInterval: 1,
  recurrenceWeekstart: "MO",
  recurrenceByDay: null,
  recurrenceByMonthDay: null,
  recurrenceEndDate: null,
  recurrenceCount: null,
  allday: null,
  calendarName: null,
  calendarId: null,
  url: null
}
```

##### Create Event Interactively

`createEventInteractively` opens the native calendar UI with the event fields prefilled. The user can review, change, save, or cancel the event in the calendar UI.

```js
window.plugins.calendar.createEventInteractively(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // Success callback
  (result) => {
    console.log(`Event UI closed, result=${result}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error creating event interactively, errorMessage=${errorMessage}`)
  }
);
```

##### Create Event Interactively With Options

`createEventInteractivelyWithOptions` opens the native calendar UI with the event fields and an options object created by [`getCalendarOptions`](#get-calendar-options). Some options are platform-specific or ignored by Android's interactive event editor; see the `getCalendarOptions` table for details.

```js
// Create the options object first
const options = window.plugins.calendar.getCalendarOptions();
// iOS: select calendar by name
options.calendarName = "My Calendar Name"; // iOS
// Android: select calendar by id
options.calendarId = 1; // Android
// Add a URL to the event
options.url = "https://www.google.com";

window.plugins.calendar.createEventInteractivelyWithOptions(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // Options created by getCalendarOptions
  options,
  // Success callback
  (result) => {
    console.log(`Event UI closed, result=${result}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error creating event interactively, errorMessage=${errorMessage}`)
  }
);
```

##### Create Event In Named Calendar

`createEventInNamedCalendar` creates an event in a named iOS calendar. This method is deprecated; use [`createEventWithOptions`](#create-event-with-options) with `options.calendarName` instead.

```js
window.plugins.calendar.createEventInNamedCalendar(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // Calendar name
  "My Calendar Name",
  // Success callback
  (eventId) => {
    console.log(`Event created, eventId=${eventId}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error creating event, errorMessage=${errorMessage}`)
  }
);
```

##### Find Event

`findEvent` searches events by title, location, notes, start date, and end date. The success callback receives an array of matching events.

```js
window.plugins.calendar.findEvent(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 0, 0, 0, 0, 0),
  // End date
  new Date(2026, 7, 16, 0, 0, 0, 0, 0),
  // Success callback
  (events) => {
    console.log(`Found events, events=${JSON.stringify(events)}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error finding events, errorMessage=${errorMessage}`)
  }
);
```

##### Find Event With Options

`findEventWithOptions` searches events with an options object created by [`getCalendarOptions`](#get-calendar-options). On iOS, `options.id` can search by event id and `options.calendarName` limits the search to one calendar. On Android, `options.id` can search by event id and `options.calendarId` limits the search to one calendar.

```js
// Create the options object first
const options = window.plugins.calendar.getCalendarOptions();
// iOS: limit search to this calendar
options.calendarName = "My Calendar Name"; // iOS
// Android: limit search to this calendar id
options.calendarId = 1; // Android
// Optional event id filter
options.id = "D9B1D85E-1182-458D-B110-4425F17819F1"; // optional

window.plugins.calendar.findEventWithOptions(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 0, 0, 0, 0, 0),
  // End date
  new Date(2026, 7, 16, 0, 0, 0, 0, 0),
  // Options created by getCalendarOptions
  options,
  // Success callback
  (events) => {
    console.log(`Found events, events=${JSON.stringify(events)}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error finding events, errorMessage=${errorMessage}`)
  }
);
```

The success callback of `findEvent`, `findEventWithOptions`, and `findAllEventsInNamedCalendar` returns an array of event objects.

An iOS event object can contain these properties:

| Property | Description |
| --- | --- |
| `id` | Event identifier. |
| `calendar` | Calendar name. |
| `title` | Event title. |
| `location` | Event location, when set. |
| `message` | Event notes, when set. |
| `startDate` | Event start date formatted as `yyyy-MM-dd HH:mm:ss`. |
| `endDate` | Event end date formatted as `yyyy-MM-dd HH:mm:ss`. |
| `lastModifiedDate` | Last modification date formatted as `yyyy-MM-dd HH:mm:ss`. |
| `attendees` | Attendee objects, when available. |
| `rrule` | Recurrence rule object, when available. |

Example event object on iOS:

```js
{
  calendar: "Kalender",
  endDate: "2016-06-10 23:59:59",
  id: "0F9990EB-05A7-40DB-B082-424A85B59F90",
  lastModifiedDate: "2016-06-13 09:14:02",
  location: "",
  message: "my description",
  startDate: "2016-06-10 00:00:00",
  title: "myEvent"
}
```

##### List Events In Range

`listEventsInRange` returns all events in the given date range.

```js
window.plugins.calendar.listEventsInRange(
  // Start date
  new Date(2026, 7, 15, 0, 0, 0, 0, 0),
  // End date
  new Date(2026, 7, 16, 0, 0, 0, 0, 0),
  // Success callback
  (events) => {
    console.log(`Events in range, events=${JSON.stringify(events)}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error listing events, errorMessage=${errorMessage}`)
  }
);
```

##### Find All Events In Named Calendar

`findAllEventsInNamedCalendar` finds all future events in the first iOS calendar with the given name.

```js
window.plugins.calendar.findAllEventsInNamedCalendar(
  // Calendar name
  "My Calendar Name",
  // Success callback
  (events) => {
    console.log(`Events in calendar, events=${JSON.stringify(events)}`)
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error finding events, errorMessage=${errorMessage}`)
  }
);
```

##### Modify Event

`modifyEvent` changes the first event matching the original title, location, notes, start date, and end date. This method is iOS only.

```js
window.plugins.calendar.modifyEvent(
  // Title of the event to modify
  "My nice event",
  // Location of the event to modify
  "Home",
  // Notes of the event to modify
  "Some notes about this event.",
  // Start date of the event to modify
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date of the event to modify
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // New title
  "My changed event",
  // New location
  "Office",
  // New notes
  "Some changed notes.",
  // New start date
  new Date(2026, 7, 15, 20, 0, 0, 0, 0),
  // New end date
  new Date(2026, 7, 15, 21, 0, 0, 0, 0),
  // Success callback
  () => {
    console.log("Event modified")
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error modifying event, errorMessage=${errorMessage}`)
  }
);
```

##### Modify Event With Options

`modifyEventWithOptions` changes the first event matching the original event fields and `options`. Use `newOptions` to set the changed calendar, reminders, recurrence, or URL. This method is iOS only.

```js
// Options used to find the existing event
const options = window.plugins.calendar.getCalendarOptions();
// Calendar containing the existing event
options.calendarName = "My Calendar Name";
// Optional event id filter
options.id = "D9B1D85E-1182-458D-B110-4425F17819F1"; // optional

// Options used for the modified event
const newOptions = window.plugins.calendar.getCalendarOptions();
// Calendar where the modified event should be stored
newOptions.calendarName = "My Changed Calendar Name";
// New first reminder in minutes before the event
newOptions.firstReminderMinutes = 120;

window.plugins.calendar.modifyEventWithOptions(
  // Title of the event to modify
  "My nice event",
  // Location of the event to modify
  "Home",
  // Notes of the event to modify
  "Some notes about this event.",
  // Start date of the event to modify
  new Date(2026, 7, 15, 18, 30, 0, 0, 0),
  // End date of the event to modify
  new Date(2026, 7, 15, 19, 30, 0, 0, 0),
  // New title
  "My changed event",
  // New location
  "Office",
  // New notes
  "Some changed notes.",
  // New start date
  new Date(2026, 7, 15, 20, 0, 0, 0, 0),
  // New end date
  new Date(2026, 7, 15, 21, 0, 0, 0, 0),
  // Options used to find the existing event
  options,
  // Options used for the modified event
  newOptions,
  // Success callback
  () => {
    console.log("Event modified")
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error modifying event, errorMessage=${errorMessage}`)
  }
);
```

##### Delete Event

`deleteEvent` deletes matching events in the given date range. You can pass `null` for title, location, or notes when they should not be used for matching. The dates are mandatory.

```js
window.plugins.calendar.deleteEvent(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 0, 0, 0, 0, 0),
  // End date
  new Date(2026, 7, 16, 0, 0, 0, 0, 0),
  // Success callback
  () => {
    console.log("Event deleted")
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error deleting event, errorMessage=${errorMessage}`)
  }
);
```

##### Delete Event From Named Calendar

`deleteEventFromNamedCalendar` deletes matching events from a specific iOS calendar. This method is iOS only.

```js
window.plugins.calendar.deleteEventFromNamedCalendar(
  // Title
  "My nice event",
  // Event location
  "Home",
  // Notes
  "Some notes about this event.",
  // Start date
  new Date(2026, 7, 15, 0, 0, 0, 0, 0),
  // End date
  new Date(2026, 7, 16, 0, 0, 0, 0, 0),
  // Calendar name
  "My Calendar Name",
  // Success callback
  () => {
    console.log("Event deleted")
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error deleting event, errorMessage=${errorMessage}`)
  }
);
```

##### Delete Event By Id

`deleteEventById` deletes an event by id. `fromDate` is optional and can be `null`. If the event is recurring, all future instances are deleted unless `fromDate` is set; then deletion starts from that date.

```js
window.plugins.calendar.deleteEventById(
  // Event id
  "D9B1D85E-1182-458D-B110-4425F17819F1",
  // Optional start date for recurring event deletion
  null,
  // Success callback
  () => {
    console.log("Event deleted")
  },
  // Error callback
  (errorMessage) => {
    console.error(`Error deleting event, errorMessage=${errorMessage}`)
  }
);
```

## Promises

If you like to use promises instead of callbacks, or struggle to create a lot of
events asynchronously with this plugin then I encourage you to take a look at
[this awesome wrapper](https://github.com/poetic-labs/native-calender-api) for
this plugin. Kudos to [John Rodney](https://github.com/JohnRodney) for this piece of art!

## Credits

This plugin was enhanced for Plugman / PhoneGap Build by [Eddy Verbruggen](http://www.x-services.nl). He fixed some issues in the native code (mainly for iOS) and changed the JS-Native functions a little in order to make a universal JS API for both platforms.
* Inspired by [this nice blog of Devgirl](http://devgirl.org/2013/07/17/tutorial-how-to-write-a-phonegap-plugin-for-android/).
* Credits for the original iOS code go to [Felix Montanez](https://github.com/felixactv8/Phonegap-Calendar-Plugin-ios).
* Credits for the original Android code go to [Ten Forward Consulting](https://github.com/tenforwardconsulting/Phonegap-Calendar-Plugin-android) and [twistandshout](https://github.com/twistandshout/phonegap-calendar-plugin).
* Special thanks to [four32c.com](http://four32c.com) for sponsoring part of the implementation, while keeping the plugin opensource.

## License

[The MIT License (MIT)](http://www.opensource.org/licenses/mit-license.html)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
