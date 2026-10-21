# FindMyAdv API reference

## `FindMyAdvConfig`

| Field | Default | Meaning |
| --- | ---: | --- |
| `appleAdvertisementKeyBase64` | `""` | 28-byte Apple public advertisement key encoded as 40 Base64 characters; empty disables Apple |
| `googleAdvertisementEidHex` | `""` | 20-byte Google advertisement EID encoded as 40 hexadecimal characters; empty disables Google |
| `advertisingIntervalMs` | `2000` | Stationary BLE interval: valid range 20–10240 ms; out-of-range values reset to 2000 ms, not the nearest bound |
| `txPowerDbm` | `0` | Requested advertising power, mapped to the nearest ESP controller level |
| `providerWindowMs` | `6000` | Requested provider window; clamped to at least three stationary advertisement intervals (and at least 1 second) |
| `accelerometerEnabled` | `false` | Enables LIS3DH/LIS2DH/LIS2DH12 motion detection |
| `accelerometerWire` | `nullptr` | Null selects `Wire` when the built-in driver is enabled; pass `&Wire1` or another bus when required |
| `initializeAccelerometerBus` | `true` | Calls `begin(SDA, SCL)` on the selected bus; set false if the host firmware already initialized it |
| `accelerometerSda` | `8` | I2C SDA pin |
| `accelerometerScl` | `9` | I2C SCL pin |
| `accelerometerAddress` | `0` | `0` probes `0x18`/`0x19`; otherwise forces the supplied address |
| `motionThreshold` | `320` | Motion delta threshold; raise it to reject small vibrations |
| `motionIntervalMs` | `200` | Motion BLE interval: valid range 20–10240 ms; out-of-range values reset to 200 ms |
| `motionHoldMs` | `30000` | Time to keep the motion interval after the last detected movement |
| `motionDetectedCallback` | `nullptr` | Optional no-argument callback returning true on motion; supports any external sensor or application state |
| `backgroundTask` | `true` | Runs the scheduler in an internal FreeRTOS task |
| `schedulerTaskStackBytes` | `3072` | Runtime stack reserved for the optional scheduler task; values below 2048 are raised to 2048 |

## Methods

### `bool begin(const FindMyAdvConfig &config)`

Copies the complete configuration, initializes the appropriate BLE backend,
starts advertising, and optionally creates the scheduler task. Returns `false`
when neither key is valid, BLE initialization fails, or the task cannot start.

Calling `begin()` while already running is a no-op that returns `true`. Use
`reconfigure()` to replace a live configuration.

After `end()`, an immediate `begin()` waits for the previous private scheduler
to exit before starting a replacement, including when switching to manual
polling. Shutdown waiting is bounded at one second; no second task is created
while the old task is still present. Serialize lifecycle calls in your application.

### `bool reconfigure(const FindMyAdvConfig &config)`

Stops advertising, safely copies the new configuration, and restarts the
library. Use this for live changes to keys, intervals, power, pins, or motion
settings.

### `bool setKeys(const char *apple, const char *google)`

Keeps the current radio and motion settings and replaces only the two provider
keys. Pass an empty string to disable one provider. At least one valid key must
remain.

### `void poll()`

Runs one scheduler iteration. Call it at least every 100 ms only when
`backgroundTask` is `false`. It is not required in the default mode.

### `void end()`

Stops FindMyAdv advertising and asks the private scheduler task to exit. It
does not tear down the shared Bluetooth controller, so another application
component can keep using the initialized stack.

### Status methods

- `bool isRunning()`
- `bool isAppleEnabled()`
- `bool isGoogleEnabled()`
- `bool isAccelerometerDetected()`
- `bool isMotionModeActive()`
- `uint32_t activeAdvertisingIntervalMs()`
- `FindMyAdvStatus status()`
- `const char *lastError()`

`FindMyAdvStatus` values are `Stopped`, `Running`, `NoValidKey`,
`BleInitializationFailed`, and `TaskCreationFailed`.

## Compile-time options

| Define | Default | Meaning |
| --- | ---: | --- |
| `FINDMYADV_DISABLE_BUILTIN_ACCELEROMETER` | `0` | Set to `1` to compile out `Wire` and the built-in ST LIS driver; custom motion callbacks still work |
| `FINDMYADV_USE_BLUEDROID` | `0` | Set to `1` only when the host firmware must use the legacy Bluedroid backend |

The broadcaster-only NimBLE flags are documented in the main README. They are
global stack settings rather than FindMyAdv API options and must not be used
when another module needs scanning, client, or GATT-server roles.

## Threading and ownership

Configuration updates stop and restart the private scheduler. Do not call
`reconfigure()`, `setKeys()`, and `end()` concurrently from multiple tasks;
serialize them in the host application when several configuration sources are
present. The library owns legacy advertising while it is running but does not
deinitialize a Bluetooth stack that another component may share.
