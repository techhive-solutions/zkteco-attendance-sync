# Setup guide

This guide takes you from installing ZKTeco Attendance Sync to your first punches in Odoo Attendances. It works on
Odoo 18.0 Community and Enterprise, on Odoo.sh or your own server. It does not run on Odoo Online (SaaS), which
can't reach a device on your network.

## Before you start

- Each device must be reachable from your Odoo server: on your LAN, over a VPN, or at a static IP. The module pulls
  from the device; the device doesn't push to Odoo (ADMS / iclock isn't supported).
- The device speaks the ZK protocol on port 4370, over TCP or UDP. Tested models: ZKTeco K40 / K40 Pro, F18, MB160,
  MB460, iFace 702, uFace 800 and SpeedFace-V5L.
- No Python packages to install. The module depends only on Odoo's Attendances app.

## 1. Install the module

1. Add the module to your addons path, as you would any other Odoo app.
2. In Odoo, open **Apps**, click **Update Apps List**, search for **ZKTeco Attendance Sync**, and install it.

Installing contacts no device. There are none until you create one.

## 2. Add a device

1. Open **Attendances > Attendance Devices > Devices** and click **New**.
2. Fill in:
   - **Host / IP**: the device's address.
   - **Port**: 4370 by default.
   - **Protocol**: **TCP** or **UDP**.
   - **Comm Key**: only if the device has one set.
   - **Timeout**: in seconds, 30 by default. An unreachable device fails its own pull within this time; it never
     holds up the other devices.
   - **Device Timezone**: the device's own wall-clock timezone. Punches are read as the device's local time and
     converted using this field, so get it right, or every punch will be off by the difference.
   - **Location**: optional, a work location.
3. Save.

## 3. Test the connection

Click **Test Connection**. On success, the device's serial, firmware, platform and record counts are read back onto
the device form.

## 4. Let it run

Three scheduled actions ship switched on:

| Scheduled action | Runs | What it does |
|---|---|---|
| **Attendance Devices: Pull Attendance** | Every 30 minutes | Pulls punches from every device and pairs them into attendances. |
| **Attendance Devices: Check Connectivity** | Every 5 minutes | Marks devices Online or Offline and alerts device managers. |
| **Attendance Devices: Sync Run Cleanup** | Daily | Deletes sync history older than the retention period. |

The pull and the connectivity check cover every device with **Sync Enabled** ticked, which is the default for a new
device. Untick it to keep a device out of the schedule. Click **Pull Attendance** on a device to pull now.

## 5. Map your people

1. On the device, click **Download Users**. Device users are linked to employees automatically by badge number, or
   by name when exactly one employee has that name.
2. Open **Attendances > Attendance Devices > Device Users** and use the **Unmapped** filter. Pick the employee for
   anyone left unmapped.

Punches from unmapped people are kept, not dropped. See the [admin guide](admin-guide.md#device-users) for how
mapping works.

## Troubleshooting

| What you see | What to do |
|---|---|
| The device shows **Offline** | Check the host, port and firewall first. The connectivity check flips it back to Online on the next successful probe and posts a note. |
| Punches aren't turning into attendances | Open **Punches** and check the punch's state. Fix the cause, such as mapping the device user, then click **Re-process** on **Error** or **Ignored** punches. |
| Every punch is off by a few hours | Check the **Device Timezone** on the device form. |
| Something else failed | Open **Sync History**. Every pull, user download and upload, and log clear leaves a row with its counts and any error message. |

Need help connecting your devices, or a VPN to a branch? Email info@techhivesolutions.com
