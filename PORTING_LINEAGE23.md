# LineageOS 23 TrebleDroid patch-port findings

## Verdict

**COMPLETE: LineageOS 23.2 / Android 16 TrebleDroid GSI builds, boots, and passes C10 runtime acceptance.**

The original 306-row A14→A16 review reached a terminal verdict at confidence >= 90: 265 patches retained and 41 verified omissions. Thirty evidence-backed integration/runtime patches were subsequently added through real builds and hardware testing, producing a current tree of 295 patch files. Every added fix is recorded in `INTEGRATION_FIXES_LINEAGE23.md` and rows 307–336 of `PORTING_PATCH_MANIFEST.csv`.

Current published inputs and artifact:

- Patch fork: `MeisterLone/lineage_patches_unified`, branch `lineage-23-td`, head `f0e2c55`
- Build-wrapper fork: `MeisterLone/lineage_build_unified`, branch `lineage-23-td`, head `b84b50f`
- Current patch count: 295 (265 reviewed baseline + 30 integration/runtime patches)
- Baseline 265-patch aggregate: `167a9ab4e2618131985d9d336017782412acbf458c9578d7b04eed34dd9c6e1b`
- Current 295-patch aggregate: `2dffe6b2b7ed9ab317d668ac61bf23b5b8d858a7540b2decbe9ac6da61d30cea`
- Final signed image: `lineage-23.2-20260812-UNOFFICIAL-arm64_bgN-signed.img`
- Size: 3,225,112,576 bytes
- SHA-256: `1ff235092605a79ac4b44e8eb8ca2cf53260c2352424cabbfbf4c43f2f4460c2`
- Final image flashed to `system_a`; userdata/metadata wiped; Android 16 boot completed with no ANR

## Scope and revisions

- Build host: `192.168.1.44`, pinned host key `SHA256:dtRtfEvjwb5TTxCSpO8iCuOLPZli5NVzEvGQUoNtCi8`
- A14 reference: `/root/custom_android/lineage21`
- Canonical A14 patch stack: `lineage-21-td`, commit `b0d95291662e6707af8bcc24458d98f6a6aa33da`
- A16 platform target: `/root/custom_android/lineage23`, branch `lineage-23.2`, Android `android-16.0.0_r4`
- A16 TD targets: `device/phh/treble` `android-16.0`; `treble_app` master
- Repository-specific baseline commits are recorded in each job's verifier `result.json`.

## Totals

### Port-stage outcomes

| Outcome | Rows |
|---|---:|
| CLEAN_AM | 148 |
| REBASED | 114 |
| DROPPED_OBSOLETE | 11 |
| SKIPPED_IRRELEVANT | 32 |
| SKIPPED_RETIRED | 1 |
| Unresolved / blocked | 0 |

Opus subsequently restored or fixed several port-stage drops/rebases. Final artifact totals therefore differ from the port-stage categories.

### Final Opus outcomes

| Outcome | Rows |
|---|---:|
| VERIFIED_HIGH | 289 |
| FIXED_VERIFIED_HIGH | 17 |
| Confidence below 90 | 0 |
| Retained baseline patch artifacts | 265 |
| Verified omissions | 41 |
| Added integration/runtime patch artifacts | 30 |
| Current patch artifacts | 295 |

The 306 original rows remain the immutable baseline review ledger. Manifest rows 307–336 are the later integration/runtime fixes and are all build-verified; device-facing rows are additionally hardware-verified.

## Final dropped or superseded rows

- **#1 Bluetooth disabled-commands revert** — its target behavior is absent after the A16 controller refactor; there is nothing to revert.
- **#33 TelephonyMetrics shim** — `TelephonyMetrics.java` and the legacy clearcut metrics path were deleted upstream; no caller remains.
- **#107 VNDK-28 sepolicy selector** — omitted together with #163. VNDK-28 policy is unreachable for this VNDK-33 device; selector without data breaks Soong and data without selector is inert.
- **#216 Display Cutout Error** — equivalent fix is already upstream as AOSP commit `911464cc30f6`.
- **#220/#221/#224 notification and QS color-series changes** — superseded by A16 notification redesign and `materialColor`/`customColor` theming. Partial reverts would create an incoherent visual stack.
- **#229 SplitShade header colors** — behavior is absorbed into A16 `ShadeHeaderController.updateColors()`.
- **#237 QSCarrier visibility fixup** — its target method is gone and its parent #229 is omitted.

Port-stage drops **#4** and **#134** were rejected by Opus: the fingerprint-cleanup gate migrated to AIDL `FingerprintProvider.java`. Both were restored and verified. Personal small-clock counterpart **#285** was also restored to stay coherent with #276.

## Verified skips

- Retired identity patch **#260** remains omitted; retail model `C10` is supplied by the fixed #262 patch.
- Device-specific personal TD rows **#304-306** remain omitted to preserve the known-good C10 build configuration.
- Frameworks/base FOD/UDFPS group **#3/#146/#154/#159** is omitted after payload review; all touched behavior is UDFPS-only and the C10 has no fingerprint hardware.
- Most frameworks/base personal UI rows **#278-299** remain omitted after individual payload review. **#285** is the exception and was restored.
- Personal telephony SPN **#267** and Chinese OEM captive-portal override **#273** remain omitted.
- VNDK-28 policy data **#163** is omitted with selector #107.

## Opus fixes incorporated

The promoted tree uses verifier artifacts for all fixed rows:

- **#4, #134** — restored fingerprint cleanup reverts at the migrated AIDL provider.
- **#16** — repaired MTK GED KPI port: malformed escapes, raw NUL, invalid binder conversion, and missing `NO_BINDER` guards.
- **#22** — rustfmt-compliant no-BPF Rust logging hunk.
- **#35** — restored an unrelated A16 radio diagnostic lost during conflict resolution.
- **#99** — guarded V7 metadata conversion so `libaudiohal@2.0` compiles.
- **#151** — fixed illegal Java capture of a newly non-final brightness variable.
- **#166** — converted all A16 kernel-version hard exits, not only one, to the intended no-BPF failure flag.
- **#171** — added the missing A2DP sysbta client-interface path.
- **#177** — moved the stray HCI command guard to the actually reachable assertion.
- **#184** — repaired author/Change-Id metadata without changing code.
- **#198** — removed an unintended residual blank line from the Messaging chain.
- **#211** — restored the navbar ContentObserver attach/detach lifecycle.
- **#262** — replaced the build-breaking A14 identity mechanism with A16 Soong build-property overrides; exact FancyDay/C10US/C10 identity retained.
- **#263** — updated the restored `/sbin` PATH to preserve A16 `/apex/com.android.virt/bin`.
- **#264** — removed personal `CGMod` branding while retaining the LineageOS self-built property mechanism.
- **#285** — restored the SystemUI small-clock counterpart to retained ThemePicker #276.

## A16 project and mechanism moves

| A14 target | A16 handling |
|---|---|
| `packages/apps/Trebuchet` | Retargeted to `packages/apps/Launcher3`; apply-directory normalization verified. |
| `packages/apps/Nfc` | Mainlined into `packages/modules/Nfc`; #266 retargeted to the resource default. |
| `system/nfc` | Mainlined into `packages/modules/Nfc`; #44 retargeted to `searchConfigPath()`. Co-apply with #266 is order-independent. |
| `build/make` build-info scripts | Product/system build props moved to `build/soong/scripts/gen_build_prop.py`; #108/#264 retargeted there. |
| C++ `system/bpf` bpfloader | Split into Rust `loader/bpfloader.rs`, C++ `loader/Loader.cpp`, and mainline `netbpfload`; all no-BPF failure analogs were mapped. |
| `system/core/rootdir/Android.mk` | Root-structure logic moved to `create_root_structure.mk`/Soong-generated environment rc. |
| `Fingerprint21.java` cleanup gate | Migrated to AIDL `FingerprintProvider.java`; #4/#134 restored at the new site. |

## Cross-project constraints

These rows must be promoted/applied as a unit; the promoted tree contains every member:

- frameworks/native **#15** with hardware/interfaces **#190** (`vendor800_1`).
- telephony **#34/#35/#40** with frameworks/base **#150/#158** and packages/services/Telephony **#82**.
- build/Soong **#264** with vendor/lineage **#269**.
- LineageParts **#199** with lineage-sdk **#200**.
- Securize removal **#258** with TrebleApp **#249**.
- VNDK-28 selector/data **#107/#163** remain omitted together.

## Security and identity checks

- `9001-model-allwinner-a523.patch` remains retired.
- #262 uses the retail identity: brand/manufacturer `FancyDay`, device/model `C10`, name `C10US`, exact dump-derived fingerprint/description/display ID, tablet characteristics.
- #261 contains only the intended self-signed `f74ca33f.0` certificate; no private key and no unrelated trust material changes.
- Securize 1/2 and 2/2 remove the dangerous relock path while preserving unrelated `rw-system.sh` and TrebleApp behavior.
- No secrets were added to the promoted patch tree.

## Completed integration and runtime acceptance

The full build pipeline has completed system image generation, target-files packaging, APK/APEX release-key signing, and signed image extraction. Later source-level fixes were compiled incrementally with `64GN nosync`; the wrapper gates validated generated identity and hardware-feature files before signing.

Final on-device results:

- Android 16 / SDK 36 boots from `system_a`; boot completes with `lmkd` running and no ActivityManager ANR.
- Rooted ADB starts automatically as uid 0 on the userdebug GSI.
- Runtime identity resolves to FancyDay / C10US / C10 across system, product, and system_ext; exact retail fingerprint, description, tablet characteristics, and display ID `863C_C10_20240619` are present while SDK/security patch remain truthful.
- Display/HWC, touch, Wi-Fi, Bluetooth, speaker audio, accelerometer/rotation, GPS, and both cameras work.
- Both cameras capture photos and video. 4K30 playback is smooth; 4K60 plays with some lag.
- TrebleApp runtime package remains `me.phh.treble.app`; namespace is supplied by Gradle.
- The C10 is a Wi-Fi-only tablet. False GSM/IMS declarations and the implied `android.software.telecom` feature are absent. Confirmed location/GPS/network features are present.
- Static RE plus fresh Play/GMS captures proved Play restriction 9 was server-side device incompatibility computed from the Android-ID checkin profile. GMS had uploaded the stale feature list verbatim. A same-ID live feature correction removed nine false cellular/telecom features, added three location features, and immediately restored normal Chrome discovery/download/install in Play Store.
- Play Store reports the device certified; Google Earth, YouTube, Chrome, and other tested applications install normally.

The final signed image itself was inspected after signing:

- absent: `android.hardware.telephony.gsm.xml`, `android.hardware.telephony.ims.xml`
- present: `android.hardware.location.xml`, `android.hardware.location.gps.xml`
- present: C10 identity/display properties, rooted-ADB default, and lmkd compatibility code

## Build commands

Production/full replay:

```bash
cd /root/custom_android/lineage23
export USE_CCACHE=1 CCACHE_EXEC=$(command -v ccache)
setsid nohup bash lineage_build_unified/buildbot_unified.sh treble 64GN \
  > /root/custom_android/build-los23.log 2>&1 < /dev/null &
```

Fast source iteration only:

```bash
setsid nohup bash lineage_build_unified/buildbot_unified.sh treble 64GN nosync \
  > /root/custom_android/build-los23.log 2>&1 < /dev/null &
```

Use unsigned `systemimage` builds for future rapid hardware/profile experiments; run release-key target-files signing only for acceptance artifacts. Reuse the existing `~/.android-certs` keyset.
