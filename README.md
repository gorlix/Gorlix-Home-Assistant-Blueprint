# Home Assistant Blueprints

Two Home Assistant blueprints for the Companion app: one sets real alarms on your Android phone from your calendar, the other sends notifications with buttons and acts on the reply.

| Blueprint | Type | Import |
|---|---|---|
| **[Calendar Alarms](#calendar-alarms)**<br>Sets real alarms on your Android phone, based on your calendar. | Automation | [![Import Calendar Alarms blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FHome-Assistant-Calendar-Alarm%2Fblob%2Fmain%2Fcalendar-auto-alarm.yaml) |
| **[Actionable Notification](#actionable-notification)**<br>Sends a notification with up to three buttons and runs actions on the reply. | Script | [![Import Actionable Notification blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FHome-Assistant-Calendar-Alarm%2Fblob%2Fmain%2Factionable-notification.yaml) |

Both blueprints need Home Assistant **2024.10** or later and the [Home Assistant Companion app](https://companion.home-assistant.io/) on your phone. Each one has full documentation inline, shown when you import or set it up.

---

## Calendar Alarms

[![Import Calendar Alarms blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FHome-Assistant-Calendar-Alarm%2Fblob%2Fmain%2Fcalendar-auto-alarm.yaml)

Source: [`calendar-auto-alarm.yaml`](calendar-auto-alarm.yaml)

Every evening it checks tomorrow morning's events. If it finds a wake-up event, it quietly sets one or more alarms in your phone's clock app. Nothing opens on your phone, the screen stays off, and you don't have to remember to set them.

Because the calendar is the source of truth, **anything that can write to a calendar can set your alarms**: an AI assistant, a Telegram bot, a shared family calendar or a script. None of them needs access to the Home Assistant API.

### Features

- **Calendar-driven.** Add an event called `Alarm 1` at 07:00, and the alarms get set. You decide the event title.
- **Silent.** Alarms are created in the background with `android.intent.extra.alarm.SKIP_UI:true`. The clock app doesn't open and the screen doesn't wake up.
- **Staggered alarms.** Set how many alarms you want and how far apart. With 4 alarms 10 minutes apart and an event at 07:00, you get 06:30, 06:40, 06:50 and 07:00. The last alarm always matches the event time.
- **Two independent groups.** For example, `Alarm 1` for a normal morning and `Alarm 2` for mornings you really can't miss. Each group has its own event title, alarm count, interval and label on the phone.
- **Confirmation notification.** After the alarms are set, you get a notification, so you know before going to sleep that everything is in place. You can change its title and text, or turn it off.

### Requirements

1. An **Android** phone with the Companion app.
2. The **Alarms & reminders** permission granted to the Companion app (Android Settings → Apps → Home Assistant → Alarms & reminders).
3. Any **calendar integration** in Home Assistant: Local Calendar, Google Calendar, CalDAV and so on.

### Setup

1. Import the blueprint with the button above.
2. Create an automation from it.
3. Choose your calendar and your Android phone. Set the notify service only if its name differs from the device name.
4. Adjust the event titles, alarm counts and intervals to your liking.

### Configuration

Inputs are grouped into collapsible sections: general settings, Group 1, Group 2 and Confirmation notification.

| Input | Default | Description |
|---|---|---|
| Run time | `23:00` | When the automation checks the calendar. Keep it before midnight. |
| Calendar | — | Calendar entity to scan. |
| Android phone | — | Phone with the Companion app, picked from your devices. |
| Notify service (optional) | — | Only if your service name differs from the device name, for example `notify.mobile_app_my_phone`. |
| Event title (Group 1) | `Alarm 1` | Exact title of the event that triggers Group 1. |
| Number of alarms (Group 1) | `4` | 1 to 10 alarms. |
| Minutes between alarms (Group 1) | `10` | 1 to 60 minutes. |
| Alarm label (Group 1) | `Alarm 1` | Label shown in the clock app. |
| Event title (Group 2) | `Alarm 2` | Exact title of the event that triggers Group 2. |
| Number of alarms (Group 2) | `3` | 1 to 10 alarms. |
| Minutes between alarms (Group 2) | `10` | 1 to 60 minutes. |
| Alarm label (Group 2) | `Alarm 2` | Label shown in the clock app. |
| Send confirmation | `on` | Send a notification after the alarms are set. |
| Notification title | `Alarms Set` | Title of the confirmation notification. |
| Notification message | `Your alarms for tomorrow morning have been set.` | Text of the confirmation notification. |

### How it works

1. At the run time you set, the automation calls `calendar.get_events` for the next morning, **00:00 to 12:00**.
2. For each group, it looks for the first event whose title **exactly** matches the configured one.
3. It works out the alarm times backwards from the event start.
4. It sends one `command_activity` notification per alarm with the `android.intent.action.SET_ALARM` intent, waiting 2 seconds between calls.
5. If no matching event is found, it does nothing. No event means no alarm, and you can sleep in.

### Good to know

- **Title matching is exact and case-sensitive.** `Alarm 1` and `alarm 1` are different titles.
- **Alarms are set once, in the evening.** If you delete or move the event afterwards, the alarms already on your phone stay where they are.
- **Only the first matching event per group is used.**
- **Keep the run time before midnight.** The search window always targets "tomorrow".
- **Android only.** iOS doesn't let apps create alarms in the Clock app.

---

## Actionable Notification

[![Import Actionable Notification blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FHome-Assistant-Calendar-Alarm%2Fblob%2Fmain%2Factionable-notification.yaml)

Source: [`actionable-notification.yaml`](actionable-notification.yaml)

A script blueprint that sends a notification to the Companion app, on Android or iOS, with up to three buttons. It waits for the reply, runs the actions of the button that was pressed, and returns the reply to whoever called it. Set it up once, then reuse it from any automation.

### Features

- **Up to three buttons**, each with its own actions or a link. Leave a title empty to hide a button.
- **Timeout** with its own actions. It can also clear the notification and treat a swipe-away as no reply.
- **Attachments**: camera snapshot or image URL.
- **Android options**: channel, importance, immediate delivery, persistent notification, lock screen visibility, icon and color, Android Auto.
- **iOS options**: interruption level up to critical, red button, unlock required.
- **Overridable fields**: `title`, `message`, `device`, `link`, `camera`, `image_url` and `timeout` can be passed when calling the script.
- **Returns the reply** to the caller: `button_1`, `button_2`, `button_3`, `timeout`, `dismissed` or `sent`.
- **Parallel runs**: each notification has its own tag, so several can wait for a reply at the same time.

### Requirements

1. An **Android** or **iOS** device with the Companion app.
2. The device must be able to reach Home Assistant, so button presses get back to it.

### Setup

1. Import the blueprint with the button above.
2. Create a script from it, for example named `Actionable notification`.
3. Choose the device and set the default content, buttons and their actions.
4. Call the script from your automations, overriding the fields you need.

### Example

Calling it from an automation and using the reply:

```yaml
- action: script.actionable_notification
  data:
    title: Front door
    message: Someone rang the bell. Unlock?
    camera: camera.front_door
    timeout: 120          # seconds, or hours/minutes/seconds
  response_variable: answer
- if: "{{ answer.reply == 'button_1' }}"
  then:
    - action: lock.unlock
      target:
        entity_id: lock.front_door
```

The script's entity ID depends on the name you give it when you create it.

### Configuration

Inputs are grouped into collapsible sections.

| Section | What you set |
|---|---|
| Target | Device, and optionally a notify service if its name differs from the device name. |
| Content | Title, message, subtitle, link on tap, camera snapshot or image URL. |
| Buttons | Title, actions and optional link for each of the three buttons. Red button and unlock required (iOS). |
| Waiting for a reply | Timeout on or off, duration, timeout actions, swipe-away as timeout, clear on timeout. |
| Delivery | Channel, importance, immediate delivery, persistent, visibility, icon and color (Android). Interruption level (iOS). Group and tag. |

### Good to know

- **Button links on Android** only open the link. They don't run the button's actions and don't stop the timeout. This is a limit of the Companion app.
- **Swipe-away detection** works only on Android.
- **Channel importance on Android** is fixed the first time a channel is used. Use a new channel name to change it.
- **Buttons do nothing?** In *Developer Tools → Events*, listen to `mobile_app_notification_action` and press a button. If no event arrives, the phone can't reach Home Assistant.

---

## Contributing

Issues and pull requests are welcome. If you have an idea for an improvement, open an issue.

## License

Released under the [MIT License](LICENSE).
