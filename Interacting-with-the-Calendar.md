## Overview

This guide outlines how to programmatically interact with a user's calendar using intents or through the `CalendarProvider`.

(**Needs Attention**)

### Using Intents to Manage the Calendar

For general usage of intents for interacting with the Calendar which is generally simpler but more limited when compared to content providers, check out [this calendar intents guide](http://www.grokkingandroid.com/intents-of-androids-calendar-app/).

See [this tutsplus guide](http://code.tutsplus.com/tutorials/android-essentials-adding-events-to-the-users-calendar--mobile-8363) for details on how to add an event to the users calendar using intents.

### Interacting with Calendar Content Provider 

The [Calendar Provider](http://developer.android.com/guide/topics/providers/calendar-provider.html) is a repository for a user's calendar events. The Calendar Provider API allows you to perform query, insert, update, and delete operations on calendars, events, attendees, reminders, and so on.

You can check out the [official docs](http://developer.android.com/guide/topics/providers/calendar-provider.html) for an overview. You can also check out [this post for retrieving calendars](http://www.emberex.com/using-androids-built-in-calendarprovider-part-one-retrieving-calendars/) and [this second post](http://www.emberex.com/using-androids-built-in-calendarprovider-part-two-retrieving-events/) for retrieving calendar events.

## References

* <http://developer.android.com/guide/topics/providers/calendar-provider.html>
* <http://code.tutsplus.com/tutorials/android-essentials-adding-events-to-the-users-calendar--mobile-8363>
* <http://www.emberex.com/using-androids-built-in-calendarprovider-part-one-retrieving-calendars/>
* <http://www.emberex.com/using-androids-built-in-calendarprovider-part-two-retrieving-events/>
* <http://www.grokkingandroid.com/intents-of-androids-calendar-app/>