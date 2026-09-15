# Phone Manager — 2026-09-15 diagnostic follow-up

## Source

OnePlus 10 Pro NE2213, OxygenOS 16.0.0. Diagnostics run through Shizuku/Rish from Termux.

## New observations

### Services

`dumpsys activity services com.oplus.phonemanager` returned no matching active service records. This is useful: although the package declares many services, the queried dump did not show a currently running service matching the filters.

### Receivers / event integration

The package declares substantial event-driven integration:

- `RealTimeMonitorReceiver` handles package-added/removed/incremental events and `SECURE_PAY_VIRUS_SCAN`.
- `BootCompleteReceiver` handles Oplus boot completion.
- `IdleTimeChangeReceiver` handles boot/date/time/timezone changes.
- AndroidX WorkManager receivers handle battery, storage, connectivity, power and rescheduling events.
- Virus services include `RemoteAppScanService`, `RemoteVirusService`, and `AutoTestScanApksService`.
- Cleanup services include `RemoteCustomScanService`, `RemoteClearService`, and `RemoteScanService`.
- Push services are present for HeyTap/OPPO message delivery.
- Providers include IdleOptimize, PhoneManager, Virus/RiskData, tracking (`nearx.track`), balance/analytics, AdDefender, Facebook Audience Network, Google Mobile Ads initialisation, and other OEM providers.

These declarations establish capability/integration, not proof that every component is active or costly during idle.

### CPU

A 100.439-second CPU interval showed Phone Manager at 35% of the interval's reported CPU metric. A following 12.737-second interval showed it at 330%, and it was not among the top entries in the subsequent 44.762-second interval.

This is strong evidence that Phone Manager can generate short, high CPU bursts while enabled. It is NOT evidence that it continuously consumes 35%/330% CPU: these are interval snapshots, and the process was under active investigation/use.

The 12.737-second interval also had `kswapd0` at 49%, demonstrating that memory-reclaim activity was occurring simultaneously. Causal attribution between Phone Manager and that reclaim activity is not established.

### Wakelocks

BatteryStats currently lists:

- `com.oplus.phonemanager:virusdetect` wakelock
- `*launch*`
- `Icing`
- `NotificationManagerService:post:com.oplus.phonemanager`
- `*job*r/com.oplus.phonemanager/.safejob.SafeCenterJobService`

The filtered output did not expose durations for the Phone Manager virusdetect wakelock, so its actual cost remains unresolved.

The SafeCenter job remains known to be tiny: 120 ms realtime across two runs and 83 ms background in the earlier detailed accounting.

## Assessment

**Decision: INVESTIGATE FURTHER.**

Evidence now supports that Phone Manager is not merely a dormant UI package. It has broad OEM system integration, event receivers, security/virus monitoring capabilities, idle-optimisation components, push infrastructure and analytics/advertising providers. It can also produce significant short CPU bursts.

However, there is still insufficient evidence to say that keeping it enabled improves system performance, battery life, thermal behaviour, security, or required functionality enough to offset its potential overhead. The CPU snapshots are not clean idle measurements, and the `virusdetect` wakelock duration is unresolved.

## Next experiment

Do not change the package state yet. First obtain a clean, quantitative comparison of enabled versus disabled Phone Manager under comparable conditions, with special attention to CPU time, battery current/charge change, temperature, wakelock time and memory pressure. The test should avoid opening Phone Manager during the measurement window.
