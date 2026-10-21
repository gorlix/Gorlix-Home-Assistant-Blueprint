# Home Assistant Calendar Alarm

A Home Assistant blueprint that sets real alarms on your Android phone, based on your calendar.

Every evening it checks tomorrow morning's events. If it finds a wake-up event, it quietly sets one or more alarms in your phone's clock app. Nothing opens on your phone, the screen stays off, and you don't have to remember to set them.

Because the calendar is the source of truth, **anything that can write to a calendar can set your alarms**: an AI assistant, a Telegram bot, a shared family calendar or a script. None of them needs access to the Home Assistant API.

[![Open your Home Assistant instance and show the blueprint import dialog with a specific blueprint pre-filled.](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FHome-Assistant-Calendar-Alarm%2Fblob%2Fmain%2Fcalendar-auto-alarm.yaml)

## Features

- **Calendar-driven.** Add an event called `Alarm 1` at 07:00, and the alarms get set. You decide the event title.
- **Silent.** Alarms are created in the background with `android.intent.extra.alarm.SKIP_UI:true`. The clock app doesn't open and the screen doesn't wake up.
- **Staggered alarms.** Set how many alarms you want and how far apart. With 4 alarms 10 minutes apart and an event at 07:00, you get 06:30, 06:40, 06:50 and 07:00. The last alarm always matches the event time.
- **Two independent groups.** For example, `Alarm 1` for a normal morning and `Alarm 2` for mornings you really can't miss. Each group has its own event title, alarm count, interval and label on the phone.
- **Confirmation notification.** After the alarms are set, you get a notification, so you know before going to sleep that everything is in place. You can change its title and text, or turn it off.

## Requirements

1. An **Android** phone with the [Home Assistant Companion App](https://companion.home-assistant.io/).
2. The **Alarms & reminders** permission granted to the Companion App (Android Settings → Apps → Home Assistant → Alarms & reminders).
3. Any **calendar integration** in Home Assistant: Local Calendar, Google Calendar, CalDAV and so on.

## Installation

1. Click the **Import Blueprint** button above, or go to *Settings → Automations & Scenes → Blueprints → Import Blueprint* and paste:
   ```
   https://github.com/gorlix/Home-Assistant-Calendar-Alarm/blob/main/calendar-auto-alarm.yaml
   ```
2. Create an automation from the blueprint.
3. Choose your calendar and enter your phone's notify service, for example `notify.mobile_app_your_phone`. You can find it in *Developer Tools → Actions*.
4. Adjust the event titles, alarm counts and intervals to your liking.

## Configuration

Inputs are grouped into collapsible sections: general settings, Group 1, Group 2 and Confirmation notification.

| Input | Default | Description |
|---|---|---|
| Run time | `23:00` | When the automation checks the calendar. Keep it before midnight. |
| Calendar | — | Calendar entity to scan. |
| Android notify service | — | Notify service of your phone, for example `notify.mobile_app_your_phone`. |
| Event title (Group 1) | `Alarm 1` | Exact title of the event that triggers Group 1. |
| Number of alarms (Group 1) | `4` | 1 to 10 alarms. |
| Minutes between alarms (Group 1) | `10` | 1 to 60 minutes. |
| Alarm label (Group 1) | `Home Assistant` | Label shown in the clock app. |
| Event title (Group 2) | `Alarm 2` | Exact title of the event that triggers Group 2. |
| Number of alarms (Group 2) | `3` | 1 to 10 alarms. |
| Minutes between alarms (Group 2) | `10` | 1 to 60 minutes. |
| Alarm label (Group 2) | `Alarm 2` | Label shown in the clock app. |
| Send confirmation | `on` | Send a notification after the alarms are set. |
| Notification title | `Alarms Set` | Title of the confirmation notification. |
| Notification message | `Your alarms for tomorrow morning have been set.` | Text of the confirmation notification. |

## How it works

1. At the run time you set, the automation calls `calendar.get_events` for the next morning, **00:00 to 12:00**.
2. For each group, it looks for the first event whose title **exactly** matches the configured one.
3. It works out the alarm times backwards from the event start.
4. It sends one `command_activity` notification per alarm with the `android.intent.action.SET_ALARM` intent, waiting 2 seconds between calls.
5. If no matching event is found, it does nothing. No event means no alarm, and you can sleep in.

## Good to know

- **Title matching is exact and case-sensitive.** `Alarm 1` and `alarm 1` are different titles.
- **Alarms are set once, in the evening.** If you delete or move the event afterwards, the alarms already on your phone stay where they are.
- **Only the first matching event per group is used.**
- **Keep the run time before midnight.** The search window always targets "tomorrow".
- **Android only.** iOS doesn't let apps create alarms in the Clock app.

## Contributing

Issues and pull requests are welcome. If you have an idea for an improvement, open an issue.

## License

Released under the [MIT License](LICENSE).
