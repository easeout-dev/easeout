---
name: easeout
description: "Builds calm, polished, animated SwiftUI screens with the Easeout motion system (blur-out screen changes, staggered reveals, rolling numbers, tuned springs and curves, black/grey/green button states, Reduce Motion) and the Easeout screen layout (big and few: one huge number, one simple hero, real logos in white circles, a lot of white space, a pinned pill button), and installs the eight ready-made Easeout scenes (flight, voice, split, portfolio, weather, lift, subscriptions, workout). Use it whenever the user asks for SwiftUI animation, transitions, micro-interactions, motion polish, or a screen that should feel premium or iOS-native, even if they do not mention Easeout, and whenever they name an Easeout scene."
---

# Easeout

Easeout is a library of animated SwiftUI screens ("scenes"). They share one motion system and one screen layout: calm, big, premium, iOS-native. This skill carries the motion system, the layout, one worked example of the target look and the eight scenes.

It has two jobs. Decide first which one the request is.

| The user … | Job |
|---|---|
| wants an animated or polished SwiftUI screen, a transition, a micro-interaction, or says "make this feel better" | 1. Build with the motion system and the screen layout |
| names an Easeout scene ("add the Easeout flight scene") | 2. Install the scene |

## Job 1: build a screen with the motion system and the screen layout

Without this skill the usual result is a static screen, or `.animation(.default)` on everything. With only half of it the result is good motion on a busy screen: a ring, a panel with three values, a list of rows, logos on coloured tiles. The target has two halves.
- **Motion**: the screen plays in beats. Content arrives piece by piece, one thing moves at a time, numbers roll, the button answers, nothing slides sideways.
- **Layout**: calm and big. Four or five blocks on a screen, one huge number (62 pt), one simple hero, real logos in white circles, a lot of white space, one quiet row, a black pill button at the bottom. The approved example is `references/screens/subs-week.png`, `subs-review.png` and `subs-saved.png`.

**Step 1. Read and look.** Every time, also for a small screen and also inside an existing design:
- `references/screens/subs-week.png`, `subs-review.png`, `subs-saved.png`: open the images and look at them first, for any app screen, whatever its subject. Then read `references/examples/subscriptions.md`, the source that draws them. Build in this manner, with your own content: the same calm, the same sizes, the same small number of things. Do not copy its data or texts.
- `references/motion-system.md`, all of it, before you write code. It holds the code to copy and the numbers. The numbers are the design: do not round them and do not replace them with defaults.
- `references/screen-layout.md`, all of it: "big and few", the sheet and its blocks, `EaseLogoBadge`, the logo rules, the strip colour and the four screen patterns (overview, focus, result, live) with their code.
- Only when your hero is a ring, a chart, a route, a waveform, a curve or blocks: the one scene screenshot that has it (section 1 of the layout file says which). Take the hero from it, not the number of blocks.

**Step 2. Look at the project.**
- The kit needs iOS 17 (`spring(duration:bounce:)`, `keyframeAnimator`, `numericText(value:)`, `@Observable`). If the deployment target is lower, ask before you raise it.
- Search for `EaseMotion` and `EaseLogoBadge`. If the kit is already in the project, reuse it. Never add a second copy.
- Choose the colours. If the project has its own design system, keep its colours, fonts and components; "big and few", the screen patterns and the logo presentation still come from `references/screen-layout.md`. If the screen is new and stands alone, or the user wants the Easeout look, take the whole look: strip, white sheet, `EaseInk`.
- If the user's data names real companies (Netflix, Spotify …), use their real logos: get the official vector files now (section 4 of `references/screen-layout.md`). Leave them out only if the user said they do not want them.
- If the screen shows people, places, products or media (a profile, a playlist, a hotel, a meal), plan real pictures for them (section 4): the user's own images first, free photos for example data second.
- If the request is to tune one transition or one control of an existing screen, change only that. Everything you build or rebuild follows the layout.

**Step 3. Plan each screen: one hero, four or five blocks, then 3 to 5 beats.** Write the plan before any code. Do not wait for approval unless the request is unclear.
- One **source of truth**: the value or list from which every number, label and colour is computed.
- One **job per screen**: an overview, a focus, a live screen (something running now) or a result (section 5 of the layout file).
- The **strip colour** of each screen, taken from its content (the logo in focus, the item's own colour, the state of the value), never from the topic and never `sun` from the example. Write it in the plan per screen. A request with much content becomes several calm screens with a blur-out between them, never one full screen.
- The **one hero**: name it in one sentence ("each carrier's logo pinned above its day"), the data that draws it, and where its colour comes from (the logos).
- The **blocks**, top to bottom, at most five: title line, the one huge number with its chip, the hero, one quiet row, the button. Then write what you **left out** and on which screen it lives instead. No panel with three values, no ring, no chart, no second list, unless that is the hero.
- The **logos**: for every real brand the official file and its source; for invented things the symbol and colour of an `EaseIconTile`; for people the picture or `EaseAvatar`. No letter monograms.
- Per beat: the trigger (appear, tap, drag, new data), the **one big thing** that moves, what follows quietly (digits roll, a chip changes, the button title rolls), and the state it rests in.
- The main button in each beat: black (an action to take), pale grey (switched off) or green (it worked); for a choice a white outline pill next to the black one.

~~~text
Flow: Parcels (overview → focus → result). Looked at: subs-week.png, subs-review.png, subs-saved.png, examples/subscriptions.md. Strip: sky.
Source of truth: parcels (carrier, shop, day, state).
Week screen, 5 blocks: title "This week" · the count "3" at 62 pt + chip "1 arrives today" · HERO · "Next up" row (DHL, Tomorrow) · button "Check parcels".
Hero: a week strip; each carrier's logo in a white circle pinned above its delivery day, the next one larger, today's number in the black circle.
Logos: DHL, UPS, FedEx from Simple Icons, each in an EaseLogoBadge in its brand colour.
Left out: the map, a status panel, the list of all parcels (the focus cards show them one at a time).
1. Enter   Blocks reveal top to bottom, the badges drop onto their days one after another, the count rolls up. Button black: "Check parcels".
2. Focus   Tap: the screen blurs out, the first logo travels into its card. Buttons: "Not yet" (outline), "Arrived" (black).
3. Decide  Tap: the card leaves to its side, the next one rises, the counter rolls.
4. Result  After the last card: the check writes itself, "2 arrived this week", one chip per carrier. Button "Done", back to the week.
~~~

**Step 4. Build.**
- Copy the kit parts you need from `references/motion-system.md` (prefix `Ease`) and start from its skeleton (section 17). Copy `EaseLogoBadge` and the pattern of your screen (`EaseWeekStrip`, `EaseFocusCard` with `EaseCardStack`, `EaseResultHero`) from `references/screen-layout.md`; put your data in and keep the sizes.
- Build the planned blocks and nothing more. A `Spacer` above the button takes what is left; do not fill it.
- Prefix your own types and files with the screen's name (`TransferView`, `TransferModel`). No bare `Motion`, `Ink`, `Demo`, `Phase`, `RootView`.
- Enter: every block gets `.easeReveal(shown, delay:)`, top to bottom, the first at 0.14 s, the next ones 0.06 s apart, the main button at 0.5 s.
- Change screens inside a flow: `withAnimation(EaseMotion.smooth(0.5))`, old content leaves with `AnyTransition.easeBlurOut`, new content reveals. No slide, push or cross-fade. `matchedGeometryEffect` only for one logo that travels from the overview into the focus card, as in the example.
- Change values: every text that changes has `.contentTransition(.numericText())` and changes inside an `EaseMotion` animation.
- Hero: animate its data (which pins exist, the card in front, the value). Never cross-fade two pictures.
- Main button: fixed at the bottom. Only its title and colour change.
- All timing goes through `EaseMotion`. With Reduce Motion things only fade. Timed steps wait with `EaseMotion.wait` and stop when the view goes away.
- The screen reacts to real taps and drags. Build a self-playing loop only for a showcase, or when the user wants to watch the motion without touching.

**Step 5. Verify.** Build the project. If you can run the screen, take a screenshot and put it next to `subs-week.png` (or the pattern you used). Fix until all of these hold:
- Count the blocks from the top of the sheet to its bottom: five at most.
- One number at 54 to 62 pt; nothing else is larger than 21 pt. One hero that one sentence describes.
- Every real brand shows its official logo in a white circle (`EaseLogoBadge`), in its own colour. No coloured tile behind a logo, no letter anywhere.
- No ring, chart, panel with three values or second list that the plan did not name as the hero.
- On every screen of the flow, 60 to 120 pt stay white above the pinned pill button. More than 140 pt empty is a hole: make the hero taller until it fits. A live screen's hero is 240 to 300 pt.
- The strip colour comes from the content of that screen. Your screens do not reuse the example's `sun` strip.
- Side by side with the target, your screen is as calm.
- Then go through `references/checklist.md` line by line. If it, or `references/motion-system.md`, still asks for a full sheet, a stat panel or tiles behind logos, or forbids every `matchedGeometryEffect`, this file and `references/screen-layout.md` win.

**Step 6. Answer.** Keep it short: the beats you built, where the files are, how to open the screen, anything renamed, each logo and picture you added and where it came from, and anything you could not verify (say so plainly). Then ask one question: "Should I connect this to your real data?"

## Job 2: install a named scene

| Scene | File | The user may call it |
|---|---|---|
| Flight status | `scenes/flight.md` | flight, boarding pass |
| Voice note | `scenes/voice.md` | voice, recording, transcript |
| Split the bill | `scenes/split.md` | split, bill, ring chart |
| Portfolio chart | `scenes/portfolio.md` | portfolio, stocks, line chart |
| Weather, the sun's path | `scenes/weather.md` | weather, sun, golden hour |
| Lift, a loaded barbell | `scenes/lift.md` | lift, barbell, plates |
| Subscriptions | `scenes/subscriptions.md` | subscriptions, renewals, Netflix and Spotify week |
| Workout | `scenes/workout.md` | workout, workout tracker, timer, heart rate |

1. Open `scenes/<id>.md` and follow it exactly. It is a complete prompt with its own rules, code, checks and closing question.
2. The scene file wins over everything in `references/`. Create its files as given. Do not swap its timing type for the kit, do not restyle it, do not change a number, and rename only as the scene file says.
3. Check with the scene's own "Check yourself before you answer" list and report as its "After it runs" section says.
4. A second scene in the same project: flight, voice, split and portfolio share type names (`RootView`, `PillButton`, `Reveal`, `StatusChip`, `Ink`, `Motion`, `Demo`, and the `reveal` and `blurOut` extensions). Give the incoming scene its prefix on all of them, as its collision rule says, and report it. Weather and lift are prefixed already. Subscriptions and workout each bring a copy of the Easeout kit (`EaseMotion`, `EaseInk`, `EaseParts`): if the kit is already in the project, keep one copy and add only the members the new scene is missing, as its file says.
5. If the build fails in Swift 6 language mode, fix the isolation only, as shown in `references/motion-system.md` section 16. Do not touch the numbers.
6. If the user asks for a scene that does not exist, name the eight and offer to build the screen with job 1.
7. If the user wants to change an installed scene, use its "What I may ask you to change" section first. For more than that, apply job 1 and keep the scene's numbers.

## Which file to open when

| File | Open it |
|---|---|
| `references/screens/subs-week.png`, `subs-review.png`, `subs-saved.png` | Job 1, always, first: before the plan and again in step 5. |
| `references/examples/subscriptions.md` | Job 1, always, right after the three screenshots. |
| `references/motion-system.md` | Job 1, always, before any code. |
| `references/screen-layout.md` | Job 1, always, before the plan. |
| `references/screens/<scene>.png` | Job 1, only when your hero is one of the scenes' heroes. |
| `references/checklist.md` | Job 1, before you answer. |
| `scenes/<id>.md` | Job 2. In job 1 open at most one, for the code of its hero. Do not copy its data, its texts or its density. |

The layout guide was called `references/visual-style.md` before. If another file still names it, open `references/screen-layout.md`.

Worked examples by hero:
- `examples/subscriptions`: the target look. A week strip with pinned logos, a focus card with a stack, a result with a check; real taps; Reduce Motion; Swift 6.
- `flight`: two screens with blur-out, a card that rises.
- `voice`: live waveform, tab pills, checklist.
- `split`: ring chart.
- `portfolio`: a line chart that morphs, a finger on the chart.
- `weather`: one value drives colours and positions; Reduce Motion; Swift 6.
- `lift`: things slide on and off a list; Reduce Motion; a loop that is cancelled.
- `subscriptions`: the target look as a full scene, with real brand logos from Simple Icons.
- `workout`: a live screen with a tall hero (timer, heart-rate bars, goal bar); the strip colour follows the workout.

## Always

- iOS 17 or later, SwiftUI only, no packages. Image files only for official logos and for pictures as section 4 of `references/screen-layout.md` says.
- Never add a second `@main`. Show the screen from the app's navigation, a sheet, a preview or a debug entry, and say where.
- Big and few: at most four or five blocks per screen, one huge number, one hero, 60 to 120 pt white above the button (more than that: a taller hero, never another block). More content means another screen, not a fuller one.
- Logos and pictures: when the data names a real company, show its real logo in an `EaseLogoBadge` (white circle, hairline, soft shadow, the logo in its own colour). The official vector file (Simple Icons first, the brand's press page second) goes into the asset catalog with "Preserve Vector Data". `EaseIconTile` is only for invented things without a logo, `EaseAvatar` for people; never a letter monogram. People, places, products and media get real pictures (the user's own first). Never generate a logo with an image model and never redraw one from memory. No photos of real, identifiable people unless the user supplies them.
- Ask before you add a permission (microphone, speech, location) or a network service.
- Example data must add up. Compute totals, shares and percentages; do not type them by hand.
- Leave the app's own navigation and system sheets alone unless the user asks for a change there.
