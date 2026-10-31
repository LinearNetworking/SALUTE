# SALUTE Report - ATAK plugin

Companion to the Node-RED "SALUTE -> ATAK" flow. Adds a toolbar button that opens a SALUTE (+H.I.M.) form.

- **Server set:** report is POSTed as JSON to `<server>/salute` (the flow turns it into CoT and, optionally, a mission data package).
- **No server set, or server unreachable:** report can be saved as a PDF on the device in `atak/tools/salute/`.
- **Settings button** (top right of the form): enter the server base URL, e.g. `http://192.168.1.50:1880`. "Test" calls `/salute/config`. Blank = offline / PDF-only mode. `http://user:pass@host:1880` sends Basic auth (for Node-RED `httpNodeAuth`).

## Flow endpoints used
| Plugin action | Endpoint |
|---|---|
| Send report | `POST /salute` |
| Mission dropdown / Refresh | `GET /salute/missions[?refresh=1]` |
| Settings > Test | `GET /salute/config` |

The JSON fields are exactly those the flow reads: callsign, size, activity, lat, lon, unit, time, equipment,
habits, intentions, morale, affiliation, mission, package, staleMinutes. No flow changes are needed.

## Building
No Gradle files are included on purpose: ATAK plugins must be built with the build.gradle, signing config and
`atakApiVersion` from the ATAK-CIV SDK (tak.gov) matching your ATAK version.

1. Copy the SDK's `plugin-examples/plugintemplate` to a new folder.
2. Replace its `app/src/main` (java, res, assets, AndroidManifest.xml) with this project's.
3. In the template's build.gradle set the applicationId / namespace to `com.atakmap.android.salute.plugin`
   (it must equal the manifest package so `R` resolves).
4. `./gradlew assembleCivDebug` (or your flavor), sideload the APK, then enable it in ATAK > Settings > Plugins.

## API points to verify against your SDK version
The code follows the 4.x/5.x plugin template pattern but was written without compiling against the SDK. If the
compiler complains, compare against the template shipped with your SDK:
- `SaluteLifecycle` constructor (`IServiceController` vs `Context`) and `SaluteTool` constructor (`Context`)
- `PluginContextProvider`, `AbstractPluginTool`, `PluginLayoutInflater` package names
- `MapView.getCenterPoint().get()` (used by "Map center"; older versions use `getPoint()`)
- `FileSystemUtils.getItem("tools/salute")` (PDF output folder)
- Code uses Java 8 lambdas; the template's compileOptions should be 1.8.

## Notes
- The server is the Node-RED flow's HTTP address, not TAK Server itself. The flow holds the TAK Server connection and certificates.
- Plain `http://` works from inside ATAK on most builds; if your build blocks cleartext, front Node-RED with HTTPS. Self-signed HTTPS certificates are not accepted by this plugin.
- "Map center": pan the ATAK map so the crosshair is on the target, open the form, tap Map center.
