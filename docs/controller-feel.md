# Xbox controller feel adaptation

Adapted from ZacharyLeahan/goldeneye-pc-port-the-finals revision
4753a95d19afd11fa48f09903325a5669ca79f5b, specifically
mods/the-finals/precision-aim.h and apply-controls.py. The reference is MIT
licensed; its license is retained in controller-reference-LICENSE.txt.
This borrows Zach's tuning choices, not a measured reproduction of The Finals.

The Xbox build now uses a 4000-unit radial deadzone instead of the former
10000-unit axial deadzone. Movement remains on the native game collision,
gait and acceleration paths, with normalized diagonal stick magnitude.
The stick-click sprint and melee additions are described below; slide, jump and
weapon-wheel changes are not included.

The default PC-style dual-stick Xbox scheme reads full-resolution physical
right-stick values at the gameplay tick. A quadratic response supplies native
rotation with 200/170 degrees-per-second horizontal/vertical base rates at
60-degree vertical FOV. ADS applies 60% gain and tangent-based FOV scaling.
Beyond 95% horizontal travel, hip-fire turning adds up to 220 degrees/second,
ramped over half a second. The ramp is integrated over each tick; releasing,
reversing or aiming clears it. There is no added smoothing tail or base
acceleration. The GoldenEye profile's 65/60 sensitivity settings represent
neutral multipliers for these rates in its newer precision adapter.

The native pitch preference is retained. Gameplay look bypasses the second
N64 deadzone and integer conversion. ADS uses the camera rather than an
independent floating crosshair, and left-stick analog movement remains enabled
at 90% while aiming. Native camera rotation/collision and pitch clamps remain.

State is per player and resets on owner/level/frame-counter changes. Paused,
dead, endscreen, active-menu, scripted-camera, EyeSpy, special-vision, bike,
grab and non-default stick-layout contexts retain their existing behavior.
The fine-look path requires the PC control scheme with swapped, dual analog
sticks (the existing Xbox default). The radial compatibility pad also serves
menus and alternate schemes; those still use N64-range values.

## Validation

Run the response checks with:

    clang -Iport/include tests/xbox-controller-feel.c -o build-xbox/controller-feel-test
    build-xbox/controller-feel-test

Passed: deadzone, normalized diagonal output, small input, immediate stop,
base/ADS rates, scoped FOV scaling, boost reversal and 30/60/144 Hz integration.
Release Xbox build succeeded. XEMU final-candidate smoke test reached dataDyne
Defection and exercised pitch, yaw, movement, firing and pause. A debugger read
of xboxLookOwners confirmed the new path was active for player one.

The candidate is build-xbox/default-controls.xbe (SHA-256
 e9b8fc77f499763dd4d924190b60c19bf9888304198bb4c01cf17bfcd1d18e84).
An installable local tree is build-xbox/controls/Perfect Dark X.
The native walking flags are important: unk14 enables analog strafing, and
canlookahead also enables analog forward/back movement despite its name.
disablelookahead independently disables camera recentering. These gates are
set from the movement stick, including while aiming. ADS input is scaled to
63/70 native movement units (90% maximum) at full stick travel.

The corrected build was tested in XEMU with aim held together with forward
movement, both strafe directions, look and firing. Keyboard controller LT was
temporarily bound to Shift to permit native modifier combinations. These are
functional emulator checks, not physical stick-feel measurements.

The final correction was installed on the physical Xbox and its SHA-256
verified by FTP readback. The original c612f4fe build remains backed up as
F:\Games\Perfect Dark X\default-before-controls.xbe; its hash was also
verified by readback. The intermediate controls build is separately retained
as controls-intermediate.xbe.

The physical-console test passed in dataDyne Defection on Agent difficulty:
LT plus left/right strafing, LT plus forward/back movement, and simultaneous
LT, analog strafe, right-stick turn and RT fire. Held-input captures showed the
aiming reticle; scene parallax and landmark scale changes established movement.
The magazine decreased from 8 to 6 during the combined firing test. Start then
opened the pause menu. All controls were released and live capture closed.

Private evidence is in the ESP-KVM checkout's observations/pdx-controls-ads-strafe,
observations/pdx-controls-ads-forward-back and observations/pdx-controls-final-reviewed.
Generic runner output remains INCONCLUSIVE automatically; the final review
records actual visual inspection. This is a bounded rooftop functional test,
not a full mission, full enemy orbit, multiplayer, variable-zoom scope or
subjective human stick-feel qualification.

## Finals face-button layout

The Xbox default PC-style mapping now uses Y for interact, B for full-crouch/stand toggle,
and X for reload. A cycles weapons (the former Y action), while menu A accepts
and menu B cancels through separate UI bindings. Desktop/N64 bindings are
unchanged. The full revision is installed as the default XBE on the physical Xbox.

## Stick-click combat controls

Left-stick click toggles sprint while moving forward. It adds 35% to native
movement, ramped over 0.15 seconds; stopping, aiming, firing, crouching, melee
or leaving ordinary on-foot gameplay cancels it. Clicking again also stops it.
The default modern control path replaces the old automatic delayed run boost
so the two bonuses cannot stack. Native gait, collision and movement remain
responsible for actual displacement.

Right-stick click performs one native unarmed primary strike, then returns to
the previous weapon. It uses the game's native lowering/equipping/attack
animations and hit handling. A held click does not repeat. During the action,
fire and weapon-cycle/reload input are suppressed. The previous gun function,
weapon-cycle history and dual-wield state are restored. Clip restoration moves
rounds between the native clip and reserve rather than replacing a saved total,
so pickups survive and the action does not provide a free reload. Inventory
checks prevent resurrecting a weapon or second gun lost during the action.

Final XBE: build-xbox/default-finals-combat.xbe, SHA-256
9dfe0e5090fd0eef6d36c7c040f39e7965b5832c5b8f2cc3c9e238db94dca747.
Installed default.xbe matched this hash on FTP readback. The original remains
in default-before-controls.xbe, and the previous verified aiming correction
(e9b8fc77...) is retained in default-before-finals-combat.xbe.

Native API boundary checks in tests/xbox-combat.c passed: one strike per held
click, clip/reserve conservation, pickups during the action, dual-wield restore,
disarm fallback, menu/death boundaries, sprint cap, and aim/fire/stop cancellation.
These compile the actual combat module against the game's actual types and
stub only its native API boundary; they do not qualify native animations or
collision. Run after configuring the game build:

    clang -D_LANGUAGE_C=1 -DVERSION=2 -Wno-macro-redefined \
      -Iinclude -Isrc/include -Iinclude/PR -Isrc/lib/ultra/audio \
      -Iport/include -Ibuild-xbox/src/generated/ntsc-final \
      -Ibuild-xbox/port/include tests/xbox-combat.c \
      port/src/xbox/combat_xbox.c -o build-xbox/combat-test
    build-xbox/combat-test

Release nxdk build and the existing look-response tests passed. XEMU exercised
native unarmed selection and return to a partially loaded pistol, X reload and
B crouch. That combat candidate also booted in XEMU. Physical Xbox evidence captured
the extended unarmed hand during a held RS click, then the Falcon 2 restored
with the original 7/72 ammunition. X reloaded to 8/71, and B lowered the camera
without cycling the weapon. Bounded walk, LS+forward, and LS+LT+forward/strafe
inputs changed the scene; LT displayed the aim reticle. This is functional
movement evidence, not a measured 35% physical-console speed benchmark.
The combat smoke tests used build 91ee7c4ca32b; the final 9dfe0e5090fd build
changes only the B binding from native three-position crouch cycling to the
native full-crouch/stand toggle. Combined LT+strafe+right-stick turning passed
in both directions around a helipad support in observations/pdx-controls-combat-circle.
Physical-console enemy melee damage is verified in the controlled test below. Y opening a door has not been separately qualified.

Private evidence: ESP-KVM observations/pdx-controls-combat-slap,
observations/pdx-controls-combat-run and observations/pdx-20261116-combat-reload-crouch.
The generic scenario runner still labels ordinary game tests INCONCLUSIVE;
the observations above record manual visual review.


The final 9dfe build was then booted into the mission on the Xbox and retested:
one B press fully crouched, the second returned to the original standing height.
After one RT shot, RS restored the pistol with the same 7/72 magazine/reserve.
Start opened the pause menu. Evidence: observations/pdx-20261116-combat-final-review.
All controls were released, and capture was closed at handoff.


## Enemy damage check (2026-11-16)

XEMU confirmed actual enemy damage in Combat Simulator Challenge 1 (Temple),
using only right-stick melee input and no RT shots. A separate diagnostic XBE
(d59f67ff6b1a976ff4d3a49ac62101b27854672613e5763d614280828b567977)
protected the player and passively observed NPC damage. It did not modify NPC
health, shields, hit detection or damage calculation. The production combat
module was restored immediately after building this separate artifact; the
normal installed default.xbe remains the verified 9dfe0e5090fd build.

Positive-damage telemetry fired for enemy chr 0 after contact. A read-only
XEMU monitor check then read the native damage/maxdamage fields as
0x3f800000 / 0x41000000, which are float 1.0 / 8.0. This confirms accumulated
real damage, not just a hand animation. No kill or per-strike balance value is
claimed. The diagnostic HUD's float placeholders were blank in nxdk snprintf,
so numeric values came from the native memory read rather than that HUD text.
The native 32-bit struct offsets were obtained with the actual nxdk compiler.

Physical Xbox attempts reached a live shooting simulant, but did not establish
an enemy health decrease or kill. The separate diagnostic then encountered
"no controller in controller socket 1" at boot; a bounded controller reset did
not restore usable gameplay. Manual DEVICE USB reseating and returning the
console to its dashboard were requested. That attempt was inconclusive; the subsequent physical test below completed qualification.
The protected diagnostic was never the installed default game.

Private details and the exact native memory read are in
build-xbox/melee-damage-validation.json; the temporary diagnostic patch and
reproduction builder are retained under ignored build-xbox/. Physical attempts
are saved under ESP-KVM observations/pdx-controls-enemy-* and
observations/pdx-20261116-enemy-* with explicit input releases.


## Physical Xbox enemy damage confirmed

The subsequent controlled test passed on the original Xbox. Two separate
right-stick clicks increased the existing live enemy chr 0's native damage
from 0 to 0.4, then 0.8, against maxdamage 8.0 (remaining health 8.0, 7.6, 7.2).
Both the captured HUD and the on-disk Xbox log recorded these changes during
combat phase 2. No RT input or gunfire from the player was used.

The separate fixture XBE SHA-256 was
40100b054e4ef6759fab2e64930c7a72547ea6ecfcc51857cd94997da0a3c351.
It protected the player and used chrMoveToPos to hold the existing live
simulant at 50 native units in front while LS was held. RS then invoked the
unchanged combat module and native attack/hit/damage routines. The fixture
never wrote NPC health or shields or invoked a damage function directly.
This verifies actual melee damage under controlled positioning, not only the
animation; it is not a kill test or a general balance measurement.

Native log (damage values multiplied by 1000 for nxdk integer formatting):

    MELEE TEST chr 0: damage 0 -> 400 / 8000 (x1000) phase 2
    MELEE TEST chr 0: damage 400 -> 800 / 8000 (x1000) phase 2

Evidence is in ESP-KVM observations/pdx-controls-enemy-hw3-measured-slaps.
The native pd.log was downloaded before restarting the normal game, preserved
as build-xbox/hardware-test/enemy-damage-xbox.log. Copies of the two reviewed
screenshots are slap-damage-first.jpg and slap-damage-second.jpg beside it.
The normal installed default.xbe was verified again as 9dfe0e5090fd, and the
temporary diagnostic XBE was removed from the Xbox after testing.

After cleanup, the normal installed build was relaunched and its Perfect Dark X boot intro was visually verified on the physical Xbox.
