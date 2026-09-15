# Phone Manager (`com.oplus.phonemanager`)

## Status

**INVESTIGATE** — currently enabled for controlled investigation.

## Device/build context

- OnePlus 10 Pro NE2213
- Android 16 / OxygenOS 16.0.0
- Build: `S.3c11fd8-181a3a9-17c6449`
- Package version observed: 16.13.3

## Evidence

Phone Manager is a privileged OEM system-management/security package, not merely a user-facing cleaner. Its declared components include:

- idle optimisation (`idleoptimize`)
- real-time/virus detection (`virusdetect`)
- boot/time/storage receivers
- WorkManager infrastructure
- cleanup/scan services
- push services
- communication/voice-call components
- multiple privileged providers
- tracking/analytics providers (`com.oplus.nearx.track...`)
- Google Mobile Ads provider
- Facebook Audience Network provider
- AdDefender provider

The current receiver/service dump showed:

- `RealTimeMonitorReceiver` reacts to package added/removed events and `SECURE_PAY_VIRUS_SCAN`.
- `IdleTimeChangeReceiver` reacts to boot/date/time/timezone changes.
- `BootCompleteReceiver` reacts to Oplus boot events.
- WorkManager receives battery/storage/connectivity/time changes.
- Virus/security services expose remote scan/virus abilities.
- `SafeCenterJobService` runs periodically.

No Phone Manager service was listed as currently running in the `dumpsys activity services com.oplus.phonemanager` query, despite the package being enabled.

### SafeCenter job

Previously measured:

- periodic interval: 12h
- standby bucket: EXEMPTED
- last successful run: 2026-09-15 10:57:08
- execution observed: about 120ms
- BatteryStats: 120ms realtime (2 times), 83ms background (1 time)
- successful completions: 2

This job is real but individually lightweight. It does not explain sustained resource use by itself.

### CPU evidence

A later CPU interval captured:

- `2026-09-15 00:37:39.651–00:39:20.090`: Phone Manager ~35% of one CPU during the 100.439s interval.
- `2026-09-15 00:39:20.090–00:39:32.827`: Phone Manager ~330% of one CPU-equivalent aggregate during the 12.737s interval.
- `2026-09-15 00:39:32.827–00:40:17.589`: Phone Manager was absent from the displayed top CPU processes.

The second interval is significant but transient. It proves Phone Manager can perform substantial active work, but does **not** prove that it continuously consumes CPU while idle.

The earlier BatteryStats accounting period showed approximately:

- UID total: 15.1mAh
- CPU: 6.47mAh
- total CPU time: ~2m30s
- foreground: ~3m02s
- background: ~5.6s
- cached: ~4m01s
- foreground activity: ~3m02s

Because the accounting period included user/diagnostic interaction with Phone Manager, these values cannot be used as a clean idle-background cost.

### Wakelocks

Observed names:

- `com.oplus.phonemanager:virusdetect`
- `*launch*`
- `Icing`
- `NotificationManagerService:post:com.oplus.phonemanager`
- `*job*r/com.oplus.phonemanager/.safejob.SafeCenterJobService`

The command output did not expose individual durations for these locks. Therefore their energy cost is **not yet established**.

## Interpretation

Established:

1. Phone Manager is deeply integrated into the OEM software stack.
2. It contains genuine security/virus detection and idle optimisation functionality.
3. It has scheduled/background triggers and privileged interfaces.
4. It can perform bursts of substantial CPU work.
5. The SafeCenter periodic job is individually lightweight.

Not established:

1. Whether disabling Phone Manager improves idle battery life on this specific build.
2. Whether its OEM idle optimisation materially improves active/sustained performance or thermals.
3. Whether disabling it breaks or degrades any required OEM security, payment, notification, call, backup, storage or system-maintenance functionality.
4. The actual duration/energy cost of its `virusdetect` wakelock.
5. Whether its analytics/advertising providers perform meaningful ongoing work while the package is otherwise idle.

## Current decision

**Do not disable yet solely from this evidence.** The package is a credible optimisation target, but a controlled A/B comparison is required because it also provides real OEM security/maintenance functions.

## Next diagnostic

Measure the actual Phone Manager wakelock durations and current process state without opening the Phone Manager UI. Then, if the result is still ambiguous, perform a controlled disable/enable comparison using matched screen-off idle periods and battery/thermal/CPU measurements.
