# Google Calendar Solutions for SkedPal

Solutions to support the workflow based on Google Calendar and [SkedPal](https://skedpal.com/).

Based on the [Google Calendar Corrections](https://github.com/scriptPilot/google-calendar-correction) and [Google Calendar Synchronization](https://github.com/scriptPilot/google-calendar-synchronization) you are able to solve most of the use cases you have between Google Calendar and SkedPal.

## Example: Timemaps Calendar

SkedPal supports to [fill time maps based on calendar events](https://skedpal.com/knowledge-base/amending-a-time-map-with-calendar-events/). Downside of this solution is that the required format `Event [Timemap Name]` is cluttering the calendar and and time-consuming to write and the events are not presented anymore within SkedPal.

Here comes a solution to split any of your calendars into an `Events` and `Timemaps` calendars to be used in SkedPal - simply by adding an emoji 🥳.

1. Setup the [Google Calendar Synchronization](https://github.com/scriptPilot/google-calendar-synchronization).

2. Create two more calendars in Google for the Events and Timemaps.

    ![Calendar Selection](docs/calendars.png)

3. Copy and paste this code to your `onCalendarUpdate` script file:

    ```js
    function onCalendarUpdate() {
      const timemaps = {
        '💻': 'Office Work',
        '🏡': 'Chores'
      }
      runOneWaySync('Personal', 'SkedPal Events', 0, 21, (targetEvent, sourceEvent) => {
        Object.keys(timemaps).forEach(emoji => {
          const regexp = new RegExp(`^(${emoji})?(.+?)?(${emoji})?$`)
          const summaryWithoutEmoji = targetEvent.summary.replace(regexp, '$2').trim()
          if (summaryWithoutEmoji !== targetEvent.summary) {        
            if (summaryWithoutEmoji === '') {
              targetEvent.status = 'cancelled'
            } else {
              targetEvent.summary = summaryWithoutEmoji
              targetEvent.transparency = 'transparent'
              targetEvent.description = sourceEvent.description
              targetEvent.location = sourceEvent.location
              targetEvent.colorId = 0
            }
          }
        })   
        return targetEvent
      })
      runOneWaySync('Personal', 'SkedPal Timemaps', 0, 21, (targetEvent, sourceEvent) => {   
        targetEvent.status = 'cancelled'
        Object.keys(timemaps).forEach(emoji => {
          const regexp = new RegExp(`^(${emoji})?(.+?)?(${emoji})?$`)
          const summaryWithoutEmoji = targetEvent.summary.replace(regexp, '$2').trim()
          if (summaryWithoutEmoji !== targetEvent.summary) {        
            targetEvent.status = 'confirmed'
            targetEvent.summary = `[${timemaps[emoji]}]`
            targetEvent.colorId = 0
          }
        })
        return targetEvent
      })
    }
    ```
    
4. Modify the timemaps, calendar names and other options according to your needs.
   - time maps must be created in SkedPal before
   - [Google Calendar Synchronization Documentation](https://github.com/scriptPilot/google-calendar-synchronization)

5. Save and run the `onCalendarUpdate` function once to grant access to your calendar to this script.
   
6. Create a trigger for the function `onCalendarUpdate`, based on calendar updates for your calendar id.

### Usage

Manage events in your personal calendar only.

Based on the timemaps definition:

```js
const timemaps = {
  '💻': 'Office Work',
  '🏡': 'Chores'
}
```  

When you not prefix or suffix any emoji:
  - an event is created in the events calendar as "busy"

![Event Only](docs/eventOnly.png)

When you prefix or suffix a configured emoji:
  - an event is created in the events calendar as "free" (to allow task to be scheduled)
  - an event is created in the timemaps calendar to be used by SkedPal to extend the timemap

![eventAndTimemap](docs/eventAndTimemap.png)

When you create an event only with an emoji in the summary:
  - an event is created in the timemaps calendar to be used by SkedPal to extend the timemap

![Timemap Only](docs/timemapOnly.png)
