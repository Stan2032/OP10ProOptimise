# OP10ProOptimise

Central evidence log and working state for empirical optimisation of the OnePlus 10 Pro (NE2213).

## Purpose

This repository is the persistent centre for the optimisation work. Findings, measurements, decisions, reversions, experiments and unresolved questions should be recorded here so that future agents can continue from the same evidence base rather than restarting or relying on conversational memory.

## Working rules

- Evidence first; do not change a setting merely because it is commonly recommended.
- Distinguish established facts, device-specific measurements, plausible interpretations, contested claims and speculation.
- Optimise the whole device, not idle battery alone: responsiveness, active-use efficiency, sustained performance, thermals, RAM/ZRAM, gaming/Winlator, background activity, connectivity, security/privacy, OTA/update reliability and required functionality all matter.
- A measured lack of benefit is a valid result.
- Make one meaningful change at a time when testing a configuration change; batch independent read-only diagnostics when that avoids unnecessary repetition.
- Preserve prior findings. Accumulate rather than overwrite history.
- Never treat a transient CPU snapshot, battery percentage change, or historical BatteryStats total as proof of sustained behaviour without controlled comparison.
- Do not disable core services blindly. Required functionality and security/recovery capabilities take precedence over small theoretical savings.
- Record the exact device/build state and exact commands/results where practical.

## Device baseline

- Device: OnePlus 10 Pro NE2213
- SoC: Qualcomm Snapdragon 8 Gen 1 / taro
- Android: 16
- OxygenOS: 16.0.0 (`V16.0.0`)
- Incremental build: `S.3c11fd8-181a3a9-17c6449`
- Security patch: 2026-06-01
- OTA package observed: `NE2213_11.J.94_4940_202606122010`
- Shizuku + Termux available for shell diagnostics

## Current decision categories

1. **KEEP** — evidence supports the current state.
2. **KEEP — DIFFERENT REASON** — current state is justified, but for a reason different from the original hypothesis.
3. **REVERT** — evidence does not justify the trade-off.
4. **INVESTIGATE** — evidence is insufficient to decide safely.
5. **UNTESTED OPTIMISATION** — plausible target not yet properly measured.

## Current known package decisions

| Package | Current state | Working assessment |
|---|---|---|
| `com.google.android.gms` | Enabled | KEEP; core Google functionality and Find Hub evidence do not justify disabling |
| `com.google.android.youtube` | Disabled | KEEP; user uses an alternative client |
| `com.oneplus.gallery` | Disabled | KEEP provisionally; user does not use it; security claims about Private Safe remain unverified |
| `net.oneplus.weather` | Disabled | KEEP; user does not use it |
| `com.google.android.projection.gearhead` | Disabled | KEEP; user does not use Android Auto |
| `com.oplus.aiunit` | Disabled | KEEP provisionally; user does not want OEM AI; unrelated core-function impact not fully tested |
| `com.google.android.devicelockcontroller` | Disabled | KEEP provisionally; AOSP evidence identifies device-financing/subsidy functionality, not a general performance service |
| `com.oplus.phonemanager` | **Enabled for investigation** | **INVESTIGATE**; privileged OEM management/security component with idle optimisation, virus/security, cleanup, push and tracking integration; actual benefit/cost is not yet established |

## Major evidence already established

- Phantom-process monitoring remains enabled by default/observed configuration; only two phantom processes were observed and neither showed meaningful CPU use. No reason to disable the restriction.
- RAM/ZRAM shows active reclaim/swap but no observed allocation stalls or OOM kills in the clean screen-off test. No basis yet to alter RAM Expansion/ZRAM.
- GMS has substantial historical BLE scan time associated with Find Hub/Fast Pair-related UUIDs, but direct Find Hub wakelock time is small and a short screen-off test did not show continuing scan-counter growth. Do not disable Find Hub or manipulate BLE scanning without stronger evidence.
- Wi-Fi multicast accounting was dominated by UID 1000 `AdbMulticastLock`, consistent with the Shizuku/ADB diagnostic environment; Wi-Fi radio sleep was effectively 100% in that snapshot. Do not infer a normal GMS Wi-Fi multicast problem from that accounting.
- Phone Manager is a privileged OEM component with meaningful system integration. Its presence of idle optimisation, security/virus detection, cleanup, WorkManager, boot/time/storage receivers and multiple privileged providers means disabling it may remove OEM functions; however, its tracking/ads/push components also create a plausible resource/privacy cost. Controlled measurement is required.

## Repository structure

- `README.md` — persistent operating rules, baseline and current decisions.
- `findings/` — detailed evidence records, one topic at a time.
- `experiments/` — controlled A/B test records and raw/derived measurements.
- `device/` — stable device/build baseline and configuration snapshots.
- `logs/` — chronological decision/change log when useful.

## Continuity rule for future agents

Before making recommendations or changes, read this README and the relevant `findings/` and `experiments/` records. Add new evidence rather than silently replacing older conclusions. If a new measurement contradicts an earlier conclusion, record the contradiction and update the decision explicitly.
