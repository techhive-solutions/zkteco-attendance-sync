# Admin guide

This guide is for the people who run the attendance devices in Odoo. It covers pairing rules, device users,
enrolling and template sync, departures, clearing device logs, roles and troubleshooting. It assumes you have
finished the [setup guide](setup-guide.md).

## Where things are

| Menu | What it's for |
|---|---|
| **Attendances > Attendance Devices > Devices** | Your devices, their connection and status, and the device buttons |
| **Attendances > Attendance Devices > Device Users** | The people on each device and the employee each is mapped to |
| **Attendances > Attendance Devices > Punches** | Every raw punch, with how it was verified and its state |
| **Attendances > Attendance Devices > Sync History** | One row per pull, user download or upload, and log clear |
| **Settings > Attendances > Biometric Devices** | Pairing rules, per company |

## Device buttons

| Button | What it does |
|---|---|
| **Test Connection** | Connects and reads back the serial, firmware, platform and record counts. |
| **Pull Attendance** | Pulls punches now, instead of waiting for the schedule. |
| **Download Users** | Reads the users on the device and maps them to employees. |
| **Upload Users** | Writes the device users already mapped to an employee on this device back to it. It doesn't copy users from other devices by itself. |
| **Sync Time to Device** | Sets the device clock. |
| **Reboot Device** | Restarts the device. |
| **Clear Device Logs** | Clears the device's attendance log, after the checks below. |
| **Sync Templates** | Copies fingerprint templates from this device to others. |

## Punch pairing

Each pull pairs the punches of all its devices together, in time order, so an employee can check in on one device
and out on another. It also pairs any punch an earlier pull stored but didn't get to pair.

Set the rules per company under **Settings > Attendances > Biometric Devices**:

| Setting | Default | What it does |
|---|---|---|
| **Punch Pairing Strategy** | Device Punch State | How a punch becomes a check-in or check-out. See the modes below. |
| **Max Shift Hours** | 16 | A check-in still open after this long is closed with no worked hours when the next shift starts, and flagged in its chatter, rather than paired to a later punch. |
| **Min Punch Gap (minutes)** | 1 | Collapses the double taps fingerprint devices produce. |
| **Punch Backfill Days** | 30 | How far back a pull accepts punches. |
| **Sync Run Retention Days** | 90 | Older sync history is deleted. |
| **Device User IDs Are Shared Across Devices** | Off | See [Device users](#device-users). |

| Mode | Behaviour |
|---|---|
| **Device Punch State** | Uses the device's own check-in, check-out or overtime state. |
| **Alternate In/Out** | Odd punch is in, even punch is out, per employee, ordered by time. Use this for single-button devices. |
| **First/Last of Day** | The first punch of the device-local day is the check-in, the last is the check-out. |

### What pairing never touches

- An attendance a person has edited. When someone changes the check-in or check-out of a device attendance, that
  side's mode becomes manual, and pairing never changes it again.
- The placeholder attendances Odoo's absence management creates.

### A wrong device clock

A punch stamped more than 10 minutes in the future is stored as an **Error** punch and never paired. Fix the clock
with **Sync Time to Device**. The punch stays on the device, so deleting it in Odoo only brings it back on the next
pull; it goes away when the device log is cleared.

## Device users

A punch from someone not yet linked to an employee is never dropped. It creates a row under **Device Users** with
the state **Unmapped**, and the person's punches are processed once you map them.

Users are mapped automatically, in this order:

1. An existing mapping.
2. The same device user ID already mapped on another device of the same company, only when **Device User IDs Are
   Shared Across Devices** is on.
3. An employee whose badge number matches the device user ID.
4. An employee with exactly that name (case doesn't matter).

If several employees have the name, nothing is guessed: the row stays unmapped for you to resolve.

Leave the shared-IDs setting off unless every device really gives each person the same ID. Devices number their
users 1, 2, 3... by default, so user 5 on a new device is usually not user 5 on another.

## Enrolling on devices

On the employee form, open the **Attendance Devices** tab and click **Enroll on Devices**. Pick one or more devices
and click **Enroll**. The employee gets the same device ID on each device. If one device is unreachable, the
wizard reports it and still enrolls on the others.

## Fingerprint template sync

Enroll a finger once, then copy it to your other devices.

1. On the source device, click **Sync Templates**.
2. Pick the **Target Devices** and the **Employees**.
3. Click **Sync**.

Each template is read back after it is written, so a device that accepted a finger but didn't store it is reported.
Each target device succeeds or fails on its own; one bad target doesn't block the others. An employee who isn't
enrolled on the source device gets a clear message.

Face templates aren't copied between devices yet; fingerprints are.

## Departures

When you register an employee's departure, **Remove from Attendance Devices** is ticked by default. The employee is
deleted from every device they are on, and their device user rows are archived. If a device can't be reached, its
row stays active and the failure is posted on the employee, so you can retry.

## Clearing a device log

**Clear Device Logs** is guarded:

1. You type the device name to confirm.
2. Odoo pulls from the device first. If that pull fails, nothing is cleared.
3. Odoo checks that every record on the device inside the backfill window is already a punch in Odoo. If one isn't,
   clearing is blocked.

## Roles

| Group | Can |
|---|---|
| **Attendance Devices: User** | View devices, test them and pull attendance. |
| **Attendance Devices: Manager** | Everything above, plus change devices, sync clocks, reboot, enroll, sync templates and clear logs. Device managers also get the offline alerts. |
| **Attendance Devices: Delete Device-Sourced Attendance** | Delete an attendance that came from a device. |

## Offline alerts

The connectivity check runs every 5 minutes. When a device drops, device managers get an activity. When it is back,
the device posts a note. A device that stays down isn't re-alerted on every run.

## Troubleshooting

- **A device shows Offline**: check the host, port and firewall first. The device flips back to Online on the next
  successful probe.
- **Punches aren't turning into attendances**: check the punch's state under **Punches**. **Error** and **Ignored**
  punches can be processed again with **Re-process** once fixed, for example after mapping the device user.
- **Start with Sync History**: every pull, user download and upload, and log clear leaves a row with its counts and
  any error message.

Need help? Email info@techhivesolutions.com
