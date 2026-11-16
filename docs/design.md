# go-nest: the design

Research into Google Nest thermostats and Nest Protect smoke and CO alarms, done on
2026-11-15 from public sources, to decide what a Go module for them would be. Nothing is
built and nothing here was run against a device. The decisions at the end are open.

In short: the two are different cases. A thermostat of the two newest generations speaks
Matter and can be driven locally; the earlier ones are reached through Google's
documented cloud API, or no longer at all. A Protect has no local interface and is in no
official API: the only way to read one is the undocumented API of Google's own app, with
a credential of the whole Google account.

## The devices

- **Thermostats** reach Google's cloud over Wi-Fi. Which generation decides everything
  below: the Learning Thermostat of 2011 and 2012, the third generation, the Thermostat E,
  the Nest Thermostat of 2020, the Learning Thermostat fourth generation of 2024.
- **Protects** talk to each other over Nest's Weave protocol on an 802.15.4 radio, and
  use Wi-Fi for the app, notifications and updates. The interconnect and the alarms work
  without Wi-Fi ([Google's help page](https://support.google.com/googlenest/answer/9232615)).
  Production of the Protect ended in 2025.

## Ways in: the thermostat

### Matter (local)

The Nest Thermostat of 2020 and the Learning Thermostat fourth generation support Matter,
over Wi-Fi. The thermostat is set up in Google Home first, then shared with another
controller by a pairing code, under linked Matter apps and services
([Google's help page](https://support.google.com/googlenest/answer/13127223)). Users of
Home Assistant report what it carries: the mode, the current temperature, the heating and
cooling targets. Not eco mode, schedules, remote sensors or the fan timer. Earlier
generations have no Matter.

### The Smart Device Management API (official cloud)

- Google's [Device Access program](https://developers.google.com/nest/device-access/registration):
  a one-time fee of 5 US dollars, a consumer Google account, a Google Cloud project with
  an OAuth client, and a Pub/Sub topic for events.
- It covers every Nest thermostat model, with traits for temperature, humidity, mode,
  eco, targets, the HVAC state and the fan timer
  ([thermostat traits](https://developers.google.com/nest/device-access/api/thermostat)).
- Its [limits](https://developers.google.com/nest/device-access/project/limits), for a
  personal project: 10 reads or commands a minute, and for a thermostat 5 commands a
  minute or 100 an hour.

### The first two generations

Google [ended their support](https://support.google.com/googlenest/answer/16233096) on
2025-10-25: they "will no longer connect to or work in the Google Nest app or Google Home
app". They keep their schedules and their controls on the wall. What reaches them now is
replacement firmware flashed over USB
([NoLongerEvil](https://github.com/codykociemba/NoLongerEvil-Thermostat),
[unfr0zen](https://github.com/unfr0zen/unfr0zen)).

### Google's Home APIs

SDKs for Android and iOS only; nothing a command on a computer can call
([the Home APIs](https://developers.home.google.com/apis)).

## Ways in: the Protects

- **Local: none.** No Matter, no HomeKit, and no public work on the Weave mesh.
- **Official cloud: none.** The Smart Device Management API's
  [supported devices](https://developers.google.com/nest/device-access/supported-devices)
  do not include it.
- **The app's API (unofficial).** [ha-nest-protect](https://github.com/iMicknl/ha-nest-protect)
  reads smoke, CO and heat states, battery and occupancy "via an undocumented and
  unofficial Nest API". It authenticates with cookies taken from a browser session at
  home.nest.com: "Only cookie authentication is supported as Google removed the API key
  authentication method." [nest_legacy](https://github.com/tronikos/nest_legacy) covers
  more of the same API. The credential is a session of the Google account, kept on the
  machine, and it ends when that session does.

## The ways in, compared

| | Thermostat over Matter | Thermostat over the official cloud API | Protect over the app's API |
|---|---|---|---|
| Supported by Google | yes | yes | no |
| Local at run time | yes | no | no |
| Needs | a 2020 or fourth-generation thermostat, a pairing code from Google Home | the fee, a Cloud project, OAuth | a browser session's cookies of the Google account |
| Gives | mode, temperature, targets | those and eco, humidity, fan, events | smoke, CO, heat, battery |
| Risk | one more controller at a time on a shared device; a Matter controller in Go | a cloud in the path; quotas | an API that can break; a credential of the whole account |

## What this module would be

Not decided, and it is two modules in one name:

1. **The thermostat.** Over Matter where the generation has it: this module would be a
   client of a Matter controller, not a Matter stack of its own. Over the Smart Device
   Management API otherwise: REST and OAuth, small.
2. **The Protects.** A port of the app's API from the two Python projects, or nothing.
   There is no supported route, and the alarms are no longer made.

Go clients of the cloud API exist, none established
([go-google-nest-sdm](https://github.com/portpowered/go-google-nest-sdm)); none of the
app's API.

## Open questions

- The thermostat's generation. It decides between Matter, the cloud API, and
  replacement firmware.
- Whether keeping a Google account's session on the machine is acceptable, for the
  Protects.
- Whether the Device Access fee and a Google Cloud project are acceptable, for the
  thermostat.

## Not verified

- Nothing above was observed on a device.
- Which thermostat generations have Matter, and what it carries, come from press and
  from users' reports, not from a Google specification.
- Whether a Google hub is needed to share a thermostat over Matter.
- That the Protect's replacement, sold by another maker, has no Matter: from press.

## Sources

- Google: [how Protects connect](https://support.google.com/googlenest/answer/9232615),
  [end of support for the first two thermostat generations](https://support.google.com/googlenest/answer/16233096),
  [sharing a Matter device](https://support.google.com/googlenest/answer/13127223).
- Device Access: [registration](https://developers.google.com/nest/device-access/registration),
  [supported devices](https://developers.google.com/nest/device-access/supported-devices),
  [thermostat traits](https://developers.google.com/nest/device-access/api/thermostat),
  [limits](https://developers.google.com/nest/device-access/project/limits).
- [The Home APIs](https://developers.home.google.com/apis): mobile SDKs only.
- [ha-nest-protect](https://github.com/iMicknl/ha-nest-protect) and
  [nest_legacy](https://github.com/tronikos/nest_legacy): the app's API, and how they
  authenticate.
- [NoLongerEvil](https://github.com/codykociemba/NoLongerEvil-Thermostat) and
  [unfr0zen](https://github.com/unfr0zen/unfr0zen): replacement firmware for the older
  thermostats.
- [go-google-nest-sdm](https://github.com/portpowered/go-google-nest-sdm): a Go client of
  the cloud API.
