# Checklist before you answer

Go through every line after the build. Fix what fails. What you could not check, say so in the answer.

## Build and project

- [ ] The project builds for iOS 17 or later. If its language mode is Swift 6, there are no new errors or warnings (`motion-system.md`, section 16).
- [ ] There is still exactly one `@main`. The screen opens from the app's navigation, a sheet, a preview or a debug entry, and you know where.
- [ ] The kit exists once. A search for `enum EaseMotion` finds one file.
- [ ] Every new type, file and extension member has a prefix (`Ease…` or the screen's name). No bare `Motion`, `Ink`, `Demo`, `Phase`, `RootView`.
- [ ] No packages. Every real brand in the data shows its official logo (source named in the answer), unless the user said no. Pictures come from the user or a free source that is named. No photos of real, identifiable people the user did not supply. No new permission and no network service without the user's yes.
- [ ] The app's own navigation and system sheets are untouched.

## Layout

- [ ] Every screen of the flow, the live one and the result included: five blocks at most, and 60 to 120 pt white above the button. More than 140 pt empty: the hero grows.
- [ ] Each strip takes its colour from that screen's content (`EaseStrip.from`). No `sun` from the example, no `lime` just because the topic is sport.

## Motion

- [ ] The screen plays in the beats you planned. Each beat has one big moving thing; the rest only rolls, pops or changes colour.
- [ ] Between two beats nothing moves for at least 0.5 s. No big motion starts while another is running.
- [ ] Enter: every block has `.easeReveal`, top to bottom, the first at 0.14 s, steps of 0.04 to 0.08 s, the main button at 0.5 s.
- [ ] Screens change only with `withAnimation(EaseMotion.smooth(0.5))` and `AnyTransition.easeBlurOut`. For a moment the sheet is almost empty. Nothing slides sideways.
- [ ] `matchedGeometryEffect` appears at most once: the selection capsule of tab pills.
- [ ] Every text that changes has `.contentTransition(.numericText…)` and changes inside an `EaseMotion` animation. Numbers in rows have `.monospacedDigit()`.
- [ ] The main button never moves or resizes. It is black for an action, pale grey when switched off, green with a check after success.
- [ ] The hero animates its data (a position, fractions, heights, a list). No cross-fade between two pictures, no chart library.
- [ ] The numbers are the ones from `motion-system.md`. None is rounded or replaced by a default.

## No timing outside `EaseMotion`

Run this in the folder of the new files. Hits are allowed only in `EaseMotion.swift`.

~~~text
grep -rnE "\.animation\(\.default|withAnimation \{|withAnimation\(\) |\.easeInOut\(|\.easeIn\(|\.easeOut\(|\.linear\(|\.spring\(\)|asyncAfter|Timer\.|Task\.sleep" --include="*.swift" .
~~~

- [ ] The search is clean.
- [ ] Every keyframe duration and every keyframe spring duration is multiplied by `EaseMotion.k`.
- [ ] With the launch argument `-slow 4` everything runs four times slower, pauses included.

## Data

- [ ] One source of truth. Every number, label, colour and button title is computed from it; nothing is typed twice.
- [ ] Example data adds up. Totals, shares and percentages are calculated. Negative numbers use `"\u{2212}"`.
- [ ] Texts and numbers that belong together change in the same frame. None lags behind.

## Reduce Motion and accessibility

- [ ] Every scale, offset, blur, pop, jump and slide has a Reduce Motion branch. With the setting on, the screen reaches the same states by fading and colour only, and the sheet fades in place.
- [ ] Controls are real `Button`s with a label. The hero graphic is one element with a label and a value. Decoration is hidden from VoiceOver.
- [ ] Pieces that are revealed later are `.accessibilityHidden` until then. A drag has an adjustable action.
- [ ] Dynamic Type: in an app screen the text sizes scale (`@ScaledMetric`). At the largest size nothing is cut off or overlaps, and the button stays reachable. A showcase with fixed sizes says so in a comment.
- [ ] At 375 pt width nothing is clipped or truncated, and on a 667 pt high screen the content fits above the button.
- [ ] The Easeout look sets light mode for its own content only (`.environment(\.colorScheme, .light)`). A project design keeps its dark mode.

## Tasks

- [ ] Every timed step runs in `.task`, `.task(id:)` or a stored `Task` that is cancelled in `.onDisappear`.
- [ ] Every pause is `try await EaseMotion.wait(…)` inside one `do`/`catch`. No `try?` in front of a wait inside a loop.
- [ ] A self-playing loop exists only if the user asked for a showcase. It checks `Task.isCancelled` and stops when the view goes away.
- [ ] Leave the screen during a beat and come back: nothing fires late, nothing is stuck halfway.

## The answer

- [ ] The beats you built, in one line each.
- [ ] Where the files are and how to open the screen.
- [ ] Anything renamed, and any number changed at the user's wish (which one, where).
- [ ] What you could not verify (no simulator, no device, Reduce Motion not tried), said plainly.
- [ ] One closing question: "Should I connect this to your real data?"
