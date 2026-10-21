# Home Assistant Blueprints

Home Assistant blueprints for the Companion app: real Android alarms from your calendar, notifications with buttons, and Do Not Disturb you can pause from a notification.

| Blueprint | Type | Import |
|---|---|---|
| **[Calendar Alarms](#calendar-alarms)**<br>Sets real alarms on your Android phone, based on your calendar. | Automation | [![Import Calendar Alarms blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fcalendar-auto-alarm.yaml) |
| **[Actionable Notification](#actionable-notification)**<br>Sends a notification with up to three buttons and runs actions on the reply. | Script | [![Import Actionable Notification blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Factionable-notification.yaml) |
| **[Android DND Controller](#android-dnd-controller)**<br>Turns DND on and off from a toggle, with a notification to pause it for a while. | Automation | [![Import Android DND Controller blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fandroid-dnd-controller.yaml) |
| **[Android DND Auto Activation](#android-dnd-auto-activation)**<br>Turns the DND toggle on and off based on zones, charging, Wi-Fi and a calendar. | Automation | [![Import Android DND Auto Activation blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fandroid-dnd-auto-activation.yaml) |

All blueprints need Home Assistant **2024.10** or later and the [Home Assistant Companion app](https://companion.home-assistant.io/) on your phone. Each one has full documentation inline, shown when you import or set it up.

---

## Calendar Alarms

[![Import Calendar Alarms blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fcalendar-auto-alarm.yaml)

Source: [`calendar-auto-alarm.yaml`](calendar-auto-alarm.yaml)

Every evening it checks tomorrow morning's events. If it finds a wake-up event, it quietly sets one or more alarms in your phone's clock app. Nothing opens on your phone, the screen stays off, and you don't have to remember to set them.

Because the calendar is the source of truth, **anything that can write to a calendar can set your alarms**: an AI assistant, a Telegram bot, a shared family calendar or a script. None of them needs access to the Home Assistant API.

### Features

- **Calendar-driven.** Add an event called `Alarm 1` at 07:00, and the alarms get set. You decide the event title.
- **Silent.** Alarms are created in the background with `android.intent.extra.alarm.SKIP_UI:true`. The clock app doesn't open and the screen doesn't wake up.
- **Staggered alarms.** Set how many alarms you want and how far apart. With 4 alarms 10 minutes apart and an event at 07:00, you get 06:30, 06:40, 06:50 and 07:00. The last alarm always matches the event time.
- **Two independent groups.** For example, `Alarm 1` for a normal morning and `Alarm 2` for mornings you really can't miss. Each group has its own event title, alarm count, interval and label on the phone.
- **Confirmation notification.** After the alarms are set, you get a notification, so you know before going to sleep that everything is in place. You can change its title, text, link, channel and other Android options, or turn it off.
- **Reliable delivery.** Alarm commands are sent with high priority, so a phone in Doze mode gets them right away.

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
| Link on tap | — | Opened when the confirmation is tapped, for example `/lovelace/home`. |

The **Android notification options** section controls how the confirmation is delivered and shown:

| Input | Default | Description |
|---|---|---|
| Channel | `Alarms` | Notification channel. Its sound and importance are fixed when it is first used. |
| Importance | `Low` | Sound and pop-up behaviour, applied when the channel is created. |
| Deliver immediately | `on` | High priority and no time-to-live, so a phone in Doze mode gets it right away. |
| Lock screen visibility | `Private` | Public, private or secret. |
| Status bar icon | `mdi:alarm-check` | Icon in the status bar. |
| Icon color | — | Color of the icon. |
| Remove after | `0` | Remove the notification automatically after this time. 0 keeps it. |
| Keep after tap | `off` | Keep the notification when it is tapped. |
| Alert once | `on` | No new sound when the notification is replaced. |
| Vibration pattern | — | For example `100, 300, 100, 300`, applied when the channel is created. |
| Tag | `calendar_alarms` | A new confirmation replaces the previous one. |
| Group | — | Stack notifications with the same group together. |
| Show on Android Auto | `off` | Also show the notification in Android Auto. |

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

[![Import Actionable Notification blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Factionable-notification.yaml)

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

## Android DND Controller

[![Import Android DND Controller blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fandroid-dnd-controller.yaml)

Source: [`android-dnd-controller.yaml`](android-dnd-controller.yaml)

Turns Do Not Disturb on your Android phone on and off with a toggle helper, and lets you **pause it for a while** straight from a notification. When the pause ends, DND comes back on by itself. Anything can drive the toggle: a dashboard button, a voice assistant, another automation or [Android DND Auto Activation](#android-dnd-auto-activation).

### Features

- **DND level of your choice**: priority only, alarms only or total silence.
- **Ringer mode** when DND turns on and off, for example vibrate and normal. Optional.
- **Pause from the notification**: up to three buttons with their own duration, for example 30 seconds and 10 minutes.
- **Live countdown** while paused, with a button to resume DND early.
- **Extra actions** when DND turns on or off, for example a focus mode on your computer. Optional.
- **Reliable delivery**: commands are sent with high priority, so a phone in Doze mode gets them right away.
- **Safe with several phones**: button IDs are unique per automation, so instances never react to each other.

### Requirements

1. An **Android** phone with the Companion app.
2. **Do Not Disturb access** granted to the Companion app (Android Settings → Apps → Special app access → Do Not Disturb access → Home Assistant).
3. Two helpers, created in *Settings → Devices & services → Helpers*:
   - a **toggle** (`input_boolean`): the DND switch;
   - a **timer**: the pause. Turn on *Restore* in its settings, so a pause survives a restart.

### Setup

1. Create the two helpers.
2. Import the blueprint with the button above and create an automation from it.
3. Choose the toggle, the timer and your phone.
4. Turn the toggle on: DND turns on and the notification with the pause buttons appears.

### Configuration

| Section | What you set |
|---|---|
| Target | DND switch, pause timer, phone, and optionally a notify service if its name differs from the device name. |
| Do Not Disturb | DND level, ringer mode when DND turns on and when it turns off. |
| Pause buttons | Title and duration of up to three pause buttons, title of the resume button. |
| Notification | Titles and messages while on and while paused (`{time}` becomes the end time), link on tap, channel, importance, icon, cannot be swiped away, keep after tap. |
| Extra actions | Actions to run when DND turns on and when it turns off, pauses included. |

### Good to know

- **The toggle is the source of truth.** If you change DND by hand on the phone, Home Assistant doesn't know about it.
- **Pressing a pause button while paused** restarts the pause with the new duration, and the countdown updates.
- **Turning the toggle off during a pause** cancels the pause, so DND doesn't come back later.
- **Channel importance on Android** is fixed the first time a channel is used. Use a new channel name to change it.

---

## Android DND Auto Activation

[![Import Android DND Auto Activation blueprint](https://my.home-assistant.io/badges/blueprint_import.svg)](https://my.home-assistant.io/redirect/blueprint_import/?blueprint_url=https%3A%2F%2Fgithub.com%2Fgorlix%2FGorlix-Home-Assistant-Blueprint%2Fblob%2Fmain%2Fandroid-dnd-auto-activation.yaml)

Source: [`android-dnd-auto-activation.yaml`](android-dnd-auto-activation.yaml)

Turns the DND toggle on and off by itself, based on where you are, whether your phone is charging, your Wi-Fi and a DND calendar. It pairs with [Android DND Controller](#android-dnd-controller): this blueprint decides **when**, the controller does the work on the phone.

### Features

- **Charging in a zone**: switch on when the phone starts charging in one of your zones, for example on your desk at university.
- **Wi-Fi check**: charging counts only if the phone is not on Wi-Fi. Optional.
- **Unplugged or left**: switch off when the phone is unplugged in a zone or when you leave all zones.
- **Entering a zone**: switch on just by arriving. Optional, off by default.
- **DND calendar**: switch on when an event starts, off when it ends. You can require being in a zone.
- **Every rule has its own toggle**, and every sensor is optional.
- **No useless writes**: the toggle changes only when it needs to.

### Requirements

1. The same **toggle** (`input_boolean`) used in Android DND Controller.
2. A **person** and one or more **zones**.
3. Optional: the Companion app **charging** sensor, a **Wi-Fi connection** sensor and a **calendar** dedicated to DND events.

### Setup

1. Import the blueprint with the button above and create an automation from it.
2. Choose the toggle, your person and the zones.
3. Add the sensors and calendar you want to use, and turn the rules on or off.

### Configuration

| Section | What you set |
|---|---|
| Target | DND switch, person, zones. |
| Sensors | Charging sensor, Wi-Fi sensor and DND calendar, all optional. |
| Rules | On when charging, off when unplugged, off when leaving, on when entering, calendar rules, require a zone for calendar events. |

### Good to know

- **Rules react to changes.** A restart of Home Assistant doesn't fire them again, and if you are already charging in a zone when you create the automation, nothing happens until the next change.
- **"In a zone"** means your person is listed in the `persons` attribute of at least one of the chosen zones.
- **A Wi-Fi sensor that is unavailable** counts as not connected.
- **Any event on the calendar counts**, so use a calendar dedicated to DND.

---

## Contributing

Issues and pull requests are welcome. If you have an idea for an improvement, open an issue.

## License

Released under the [MIT License](LICENSE).
