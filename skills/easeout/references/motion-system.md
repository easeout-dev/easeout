# Easeout motion system

The motion of the six Easeout scenes, as code to copy. The numbers are the design: use them as written. Each part names the scene it comes from; open that scene in `scenes/` to see it at work.

## Contents

1. How to use this file
2. Rules of thumb
3. Timing: `EaseMotion`
4. Which timing for what
5. Colours: `EaseInk`
6. Enter: staggered reveal
7. Change screens: blur out, then reveal
8. Swap content in place
9. Text and numbers that roll
10. Buttons: press, three states, selection
11. Small things pop
12. One value drives everything
13. Drags
14. Things that slide on and off
15. Reduce Motion
16. Swift 6 notes
17. Skeleton: a whole screen

## 1. How to use this file

- The snippets form one kit with the prefix `Ease`. Add it to the project once: `EaseMotion.swift` (sections 3 and 5) and `EaseParts.swift` (the parts the screen uses). Search the project for `EaseMotion` first and reuse the kit if it is there.
- Every snippet needs `import SwiftUI`, targets iOS 17, compiles in Swift 6 language mode, and may use the snippets above it.
- Names: every type, every file and every extension member you add starts with a prefix. Use `Ease` for the kit and the screen's name for the screen (`CheckoutView`, `CheckoutModel`). The scenes show why. Flight, voice, split and portfolio use bare names (`Ink`, `Motion`, `Demo`, `Phase`) and need a rename rule for collisions. Weather and lift prefix every type, every file and even the reveal method (`WeatherSceneMotion`, `.liftSceneReveal`) and never collide. For one self-contained showcase you may replace `Ease` with `<Name>Scene`, as they do.
- A project with its own palette keeps the `EaseInk` names and points them at its own colours. The motion stays.
- Do not "improve" a number. If the user asks for faster or slower motion, change it in one place and say which.

## 2. Rules of thumb

1. **One big moving thing per beat.** A ticket rises (flight), a capsule slides (voice), arcs grow (split), a line morphs (portfolio), the sun travels (weather), a pair of plates slides on (lift). Everything else answers quietly: digits roll, a chip changes, a title rolls.
2. **Let it rest.** After a beat nothing moves for 0.5 to 1.5 s. Never start the next big motion while the last one is still running.
3. **Screens change in one way only.** The old content melts into a blur, the sheet is almost empty for a moment, the new content comes in piece by piece. Nothing slides sideways, nothing morphs, no plain cross-fade (flight, voice).
4. **No `matchedGeometryEffect` for screen changes**, and none for moving content from one screen to another. The only shared element in the scenes is the black selection capsule behind tab pills (voice, portfolio). Flight and split use none; weather and lift forbid it.
5. **What stays, stays still.** The main button never moves; only its title and colour change. A screen keeps one layout for all its states; only the data changes (flight: one layout for boarding, in the air, landed). Hidden pieces keep their space.
6. **Things move along their real path.** The plane along the route (flight), portraits along the ring (split), the sun along its bow (weather), plates along the bar (lift). Animate the position on the path, not x and y. Do not replace such a move with a fade or a scale.
7. **Data graphics never cross-fade between two pictures.** Animate the data: the heights of a line (portfolio), the ends of an arc (split), one position from which every colour is mixed (weather).
8. **One source of truth.** Every number, label and colour is computed from one value or one list, in the same frame: the plates on the bar (lift), the sun's position (weather), the assigned dishes (split). Nothing is typed by hand and nothing lags behind.
9. **Every text that changes rolls.** `numericText` on values, titles, chips and labels.
10. **In slow, out fast.** Content comes in over 0.7 s, staggered. It leaves in 0.3 to 0.5 s, all at once.
11. **Bounce by size.** Text, digits, reveals and presses: 0. Big surfaces: 0.08 to 0.12 (sheet, chart, the sun coming back). Objects that land: 0.15 to 0.3 (arcs, capsule, plates, bars). Small things that pop: 0.35 to 0.45 (check, dots, stars). Nothing above 0.45.
12. **Three steps of feedback.** Press: the control gives way (scale 0.965). Answer, on the control: the title rolls, it turns green with a pop, a check appears. Consequence, in the hero: the amount rolls, the barbell jumps, a bell pops in.
13. **Colour has a job.** Black: an action to take. Pale grey: switched off. Green: the action worked. Labels are grey and never look like buttons.
14. **Reduce Motion: fades and colours only** (section 15).

## 3. Timing: `EaseMotion`

From every scene (`Motion`, `WeatherSceneMotion`, `LiftSceneMotion`). `travel` is from flight. The throwing `wait` is from weather and lift. `pulse` holds the one curve that voice writes inline, so no curve lives outside this type.

~~~swift
import SwiftUI

/// All timing goes through here. Launch with `-slow 4` to run everything four times slower
/// and look at the motion.
enum EaseMotion {
    static let k: Double = {
        let v = UserDefaults.standard.double(forKey: "slow")
        return v > 0 ? v : 1
    }()

    /// Soft ease-out, the main curve.
    static func smooth(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.2, 0.9, 0.2, 1, duration: d * k).delay(delay * k)
    }
    /// Ease-in-out for things that travel across the screen.
    static func travel(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.6, 0, 0.15, 1, duration: d * k).delay(delay * k)
    }
    static func spring(_ d: Double, bounce: Double = 0, delay: Double = 0) -> Animation {
        .spring(duration: d * k, bounce: bounce).delay(delay * k)
    }
    /// Slow ease-in-out for a dot that breathes.
    static func pulse(_ d: Double) -> Animation {
        .easeInOut(duration: d * k)
    }
    /// Throws when the task is cancelled, so a flow that waits here stops at once.
    static func wait(_ seconds: Double) async throws {
        try await Task.sleep(for: .seconds(seconds * k))
    }
}
~~~

What "all timing goes through here" means:

- Every `Animation` is made by `EaseMotion.smooth`, `.travel`, `.spring` or `.pulse`. Durations and delays are arguments of these calls, as in the scenes: `EaseMotion.smooth(0.7, delay: delay)`.
- Every pause is `try await EaseMotion.wait(…)`.
- Every keyframe duration is multiplied by `EaseMotion.k` (see the pill button in section 10).
- Never use `.animation(.default, …)`, `withAnimation { }` without an animation, `.easeInOut(duration:)`, `.linear`, a bare `.spring()`, `DispatchQueue.main.asyncAfter` or `Timer`.
- Attach `.animation(_, value:)` to the smallest view that changes with that value. Use `withAnimation` when the change inserts or removes views: a new screen, a new list item, a selected tab.
- Use `smooth` for things that appear, leave, fade or change colour. Use `spring` for things that move, grow, roll or pop. Use `travel` for one object that crosses the screen.

## 4. Which timing for what

Pick from these tables. All calls are `EaseMotion.…`.

**Enter and leave**

| What | Timing | Values | Scene |
|---|---|---|---|
| Sheet slides up or away | `spring(0.85, bounce: 0.08)` | from `height + 60` below | all six |
| A piece reveals | `smooth(0.7, delay:)` | y 14, blur 10, scale 0.98 | all six |
| First piece waits | 0.14 s (`lead`) | then 0.06 s per block (0.04 to 0.08) | all six |
| Main button reveals | delay 0.5 on the way in, 0 on the way out | | all six |
| Columns of a row | +0.05 per column; +0.04 when there are six | | weather; lift |
| Rows of a list | 0.12 + 0.08 × row | y 10, blur 8 | voice |
| Words; lines | reveal, one line every 0.28 s | y 6, blur 6; y 8, blur 8 | voice |
| A card rises | reveal | y 90, blur 14, scale 0.94; its content 0.14 s later with y 10 | flight |
| A round control pops in | reveal | y 0, scale 0.6 | voice |
| A screen leaves | `withAnimation(smooth(0.5))` | blur 16, scale 0.97, y −8 | flight, voice |
| Content swaps in place | `smooth(0.5)`; per column `smooth(0.6, delay: i × 0.09)` | y ±18, blur 8 | voice; flight |
| A card drops away | move `smooth(0.6)`, fade `smooth(0.25, delay: 0.3)` | y 96, scale 0.8 from the bottom, 2.5°, blur 6 | flight |

**Text**

| What | Timing | Scene |
|---|---|---|
| Button title rolls | `spring(0.6)` | all six |
| Chip text rolls | `spring(0.5)` | weather, lift |
| Number that follows a finger or counts fast | `spring(0.4)` | weather, lift |
| Timer | `spring(0.6)` | voice |
| Number that makes one step | `spring(1.1)` | flight |
| Amounts after a tap | `spring(0.75, bounce: 0.16)`; `spring(0.7, bounce: 0.1)`; `spring(0.8)` | split (assign); split (paid); portfolio, flight |
| Text turns from grey to black | `smooth(0.4)` | voice |
| Line through a finished task | `smooth(0.45, delay: 0.08)` | voice |

**Press and answer**

| What | Timing | Scene |
|---|---|---|
| Press down | `spring(0.2)`, held 0.17 s in the demos | all six (`tap()`) |
| Release | `spring(0.35, bounce: 0.2)`, the result shows 0.08 s later | all six (`tap()`) |
| Button switches between off and black | `smooth(0.45)` | split |
| Button turns green | colour `smooth(0.35)`; pop to 1.04 in 0.14 s (snappy), back in 0.45 s (bouncy) | split, portfolio, weather, lift |
| Check in the button | scale 0.5 with opacity, on the title's `spring(0.6)` | all six |
| Hero jumps once | −9 pt in 0.16 s (snappy), back in 0.38 s (bouncy) | lift |

**Pops**

| What | Timing | From scale | Scene |
|---|---|---|---|
| Coloured dot | `spring(0.5, bounce: 0.45, delay: n × 0.07)` | 0.3 | split |
| Check badge | `spring(0.5, bounce: 0.4)` | 0.4 | split |
| Checkbox, check draws itself | `spring(0.45, bounce: 0.35)` | 0.5 | voice |
| Icon into a tag | `spring(0.5, bounce: 0.3)` | 0.3 | weather |
| Moon and stars come out | `spring(0.4, bounce: 0.35, delay: 0.05 × order)` | 0.4 and 0.2 | weather |
| Moon and stars leave | `smooth(0.3)`, together | | weather |
| A control leaves | `smooth(0.3)`, to scale 0.6 and blur 4 | | split |
| A finger takes hold | `spring(0.3, bounce: 0.2)` | halo from 0.6 | weather |
| Live dot breathes | `pulse(0.6)`, opacity between 1 and 0.3 | | voice |

**Hero and data**

| What | Timing | Scene |
|---|---|---|
| Selection capsule slides | `spring(0.5, bounce: 0.18)`; `spring(0.7, bounce: 0.12)` together with a chart | voice; portfolio |
| Chart line morphs | `spring(0.7, bounce: 0.12)` | portfolio |
| Ring arcs grow, portraits ride along | `spring(0.75, bounce: 0.16)` | split |
| A line draws itself | `smooth(1.0)`; a chart `smooth(1.1)` | flight, split; portfolio |
| An object crosses the screen | `travel(1.5)`; the trail behind it `smooth(0.6)` | flight |
| A value follows the finger | `smooth(0.16)` | portfolio, weather |
| The finger lets go | `spring(0.9, bounce: 0.1)` | weather |
| An object slides on | `spring(0.5, bounce: 0.25)`; after a tap `spring(0.55, bounce: 0.3)` | lift |
| An object slides off | `spring(0.5, bounce: 0.15)` | lift |
| A wave of bars | `spring(0.45, bounce: 0.35, delay: i × 0.016)`; `spring(0.5, bounce: 0.3, delay: i × 0.012)` | voice |
| Bars print themselves | `spring(0.5, bounce: 0.3, delay: 0.55 + i × 0.009)`, from height 0.15 at the bottom | flight |
| Ticks light up | `smooth(0.3, delay: i × 0.026)`; each stretches to 1.9 in 0.13 s, back in 0.42 s | flight |

**Rests** (from the scenes' own timelines)

| Between | Wait |
|---|---|
| Sheet starts to rise, hero starts to move | 0.35 to 0.85 s |
| A hero motion, the next tap | 0.75 to 2.2 s |
| A small answer (a tick, a dot), the next tap | 0.5 to 0.55 s |
| Success, leaving | 1.5 to 1.7 s |
| Sheet away, start again | 0.5 s |

## 5. Colours: `EaseInk`

The colours the scenes share. `references/screen-layout.md` explains their roles.

~~~swift
/// Colours of the Easeout look: flat, light mode only.
enum EaseInk {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)       // titles, numbers
    static let secondary = Color(red: 0.459, green: 0.463, blue: 0.498)     // labels, captions
    static let faint = Color(red: 0.667, green: 0.667, blue: 0.706)         // text that is not final yet
    static let fill = Color(red: 0.957, green: 0.957, blue: 0.961)          // grey card
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)          // status chip
    static let hairline = Color(red: 0.898, green: 0.898, blue: 0.914)      // dividers, outline of a list card
    static let outline = Color(red: 0.831, green: 0.831, blue: 0.847)       // outline of a white button
    static let track = Color(red: 0.902, green: 0.902, blue: 0.918)         // empty part of a ring or a bar
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)         // button: an action to take
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)           // button: switched off
    static let successFill = Color(red: 0.133, green: 0.773, blue: 0.369)   // button: the action worked
    static let successText = Color.white
    static let up = Color(red: 0.133, green: 0.773, blue: 0.369)            // gain
    static let down = Color(red: 0.937, green: 0.267, blue: 0.267)          // loss
    static let upFill = Color(red: 0.863, green: 0.988, blue: 0.906)        // chip behind a gain
    static let upText = Color(red: 0.086, green: 0.502, blue: 0.239)
    static let downFill = Color(red: 0.996, green: 0.886, blue: 0.886)      // chip behind a loss
    static let downText = Color(red: 0.725, green: 0.11, blue: 0.11)
    static let live = Color(red: 0.937, green: 0.196, blue: 0.196)          // recording, live
}
~~~

## 6. Enter: staggered reveal

From all six scenes (`Reveal`). The Reduce Motion branch is from weather and lift.

~~~swift
/// Element comes in from a soft blur, a little lower and smaller. Used with small delays for a stagger.
/// With Reduce Motion it only fades in.
struct EaseReveal: ViewModifier {
    let on: Bool
    var delay: Double = 0
    var y: CGFloat = 14
    var blur: CGFloat = 10
    var scale: CGFloat = 0.98
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        let still = on || reduceMotion
        content
            .opacity(on ? 1 : 0)
            .blur(radius: still ? 0 : blur)
            .scaleEffect(still ? 1 : scale)
            .offset(y: still ? 0 : y)
            .animation(EaseMotion.smooth(0.7, delay: delay), value: on)
    }
}

extension View {
    func easeReveal(_ on: Bool, delay: Double = 0, y: CGFloat = 14, blur: CGFloat = 10, scale: CGFloat = 0.98) -> some View {
        modifier(EaseReveal(on: on, delay: delay, y: y, blur: blur, scale: scale))
    }
}
~~~

How to stagger:

- The screen owns `@State private var shown = false` and sets it in `.onAppear { shown = true }`.
- Reveal blocks, not single labels: a row, a number, a card, a graphic. Five to seven per screen.
- Delays run from top to bottom: `lead` (0.14), then `lead + 0.06`, `lead + 0.12`, and so on. The scenes use steps between 0.04 and 0.08. The last block starts at most about 0.35 s after the first.
- Columns inside a row add 0.05 each (weather), or 0.04 when there are six (lift). Rows of a list use `0.12 + 0.08 × row` with `y: 10, blur: 8` (voice).
- Small pieces travel less: words `y: 6, blur: 6`, lines `y: 8, blur: 8` (voice).
- A card that should be seen arriving travels more: `y: 90, blur: 14, scale: 0.94`. Its content follows 0.14 s later with `y: 10` (flight ticket).
- A round control pops: `y: 0, scale: 0.6` (voice play button).
- The main button follows the sheet: `.easeReveal(sheetUp, delay: sheetUp ? 0.5 : 0)`. It is the last in and the first out.
- Reveal works both ways and keeps the space. `.easeReveal(shown && loaded)` brings a line in later without moving its neighbours (lift). Give such a piece `.accessibilityHidden(!loaded)`.
- To show pieces one by one, count: `.easeReveal(i < visibleLines, y: 8, blur: 8)` and raise `visibleLines` every 0.28 s (voice).
- To play the entrance again, give the screen `.id(round)` and raise `round` (all scenes).

## 7. Change screens: blur out, then reveal

From flight and voice (`BlurOut`, `blurOut`). Those two scenes have no Reduce Motion branch; the one here follows the rule of weather and lift: fade only.

~~~swift
/// A screen leaves by melting into a blur. New screens bring their own staggered reveal.
/// With Reduce Motion it only fades.
struct EaseBlurOut: ViewModifier {
    let active: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        let moving = active && !reduceMotion
        content
            .opacity(active ? 0 : 1)
            .blur(radius: moving ? 16 : 0)
            .scaleEffect(moving ? 0.97 : 1)
            .offset(y: moving ? -8 : 0)
    }
}

extension AnyTransition {
    @MainActor static var easeBlurOut: AnyTransition {
        .asymmetric(insertion: .identity,
                    removal: .modifier(active: EaseBlurOut(active: true), identity: EaseBlurOut(active: false)))
    }
}
~~~

How to use it (section 17 shows it complete):

- Both screens sit in one `ZStack(alignment: .top)` with a `switch`. Each case ends with `.transition(AnyTransition.easeBlurOut)`.
- Change the screen with `withAnimation(EaseMotion.smooth(0.5)) { phase = .next }`.
- Insertion is `.identity`: the new screen is there at once, with every piece still hidden. Its own reveal (`lead` 0.14) brings the pieces in while the old content melts. For a moment the sheet is almost empty. That pause is wanted.
- Whatever stays (strip, sheet, main button) lives outside the `ZStack` and does not move. Draw the button after the screens, so a card can drop behind it (flight).
- This is for the steps of one flow inside a screen or a sheet. Do not rebuild the app's `NavigationStack` with it.

## 8. Swap content in place

From flight (`CellsRow`: three columns swap one after the other) and voice (the grey card swaps between summary and checklist). The old content leaves upward in a blur, the new one comes up from below. The Reduce Motion branch follows weather and lift.

~~~swift
/// In and out state for content that is swapped in place: moved by `y`, blurred and invisible.
/// With Reduce Motion it only fades.
struct EaseSwap: ViewModifier {
    let y: CGFloat
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        content
            .opacity(y == 0 ? 1 : 0)
            .blur(radius: y == 0 || reduceMotion ? 0 : 8)
            .offset(y: reduceMotion ? 0 : y)
    }
}

extension AnyTransition {
    @MainActor static var easeSwap: AnyTransition {
        .asymmetric(insertion: .modifier(active: EaseSwap(y: 18), identity: EaseSwap(y: 0)),
                    removal: .modifier(active: EaseSwap(y: -18), identity: EaseSwap(y: 0)))
    }
}

/// A label and a value. When the label changes, the old pair leaves upward and the new one comes
/// up from below. When only the value changes, its digits roll.
struct EaseSwapCell: View {
    let label: String
    let value: String
    var column = 0

    var body: some View {
        ZStack {
            VStack(spacing: 5) {
                Text(label)
                    .font(.system(size: 15))
                    .foregroundStyle(EaseInk.secondary)
                Text(value)
                    .font(.system(size: 24, weight: .bold))
                    .foregroundStyle(EaseInk.primary)
                    .monospacedDigit()
                    .lineLimit(1)
                    .minimumScaleFactor(0.8)
                    .contentTransition(.numericText())
            }
            .id(label)
            .transition(AnyTransition.easeSwap)
        }
        .frame(maxWidth: .infinity)
        .animation(EaseMotion.smooth(0.6, delay: Double(column) * 0.09), value: label)
        .animation(EaseMotion.spring(1.1), value: value)
    }
}
~~~

For a whole card with two contents (voice): put `if` and `else` in a `ZStack(alignment: .topLeading)`, give both `.transition(AnyTransition.easeSwap)`, and put `.animation(EaseMotion.smooth(0.5), value: tab)` on the card. Give the card a fixed `minHeight`, so it does not jump.

## 9. Text and numbers that roll

From all six scenes.

- Every text that changes gets `.contentTransition(.numericText())`: values, button titles, chips and labels ("Boards in" to "Lands in", flight).
- Numbers get `.contentTransition(.numericText(value: Double(x)))`, so the digits roll up when the value grows and down when it shrinks. A countdown passes the negative value: `numericText(value: Double(-minutes))` (flight).
- Numbers in rows, columns, timers and amounts get `.monospacedDigit()`, so their neighbours do not shake. The free-standing 104 pt number of weather and lift goes without it and uses `.fixedSize()`.
- The change must happen inside an animation: `.animation(EaseMotion.spring(…), value: x)` on the text or on a parent, or `withAnimation`. Without one the text jumps.
- Compute the text from the model. To count up at the start, change the model in steps (lift: 20, 70, 120, 140) or once (flight: 60 to 42).
- Use a real minus sign, `"\u{2212}"` (portfolio, lift).

The hero number of weather and lift:

~~~swift
/// A very large number with a small unit. Its digits roll when the value changes.
struct EaseHeroNumber: View {
    let value: Int
    var unit = "kg"

    var body: some View {
        // The line box of such a large font is much taller than its digits.
        // The frame keeps only the height of the digits, so the next line can sit close.
        HStack(alignment: .firstTextBaseline, spacing: 6) {
            Text("\(value)")
                .font(.system(size: 104, weight: .bold))
                .tracking(-3)
                .foregroundStyle(EaseInk.primary)
                .contentTransition(.numericText(value: Double(value)))
            Text(unit)
                .font(.system(size: 34, weight: .semibold))
                .foregroundStyle(EaseInk.secondary)
        }
        .fixedSize()
        .frame(height: 78)
        .animation(EaseMotion.spring(0.4), value: value)
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("\(value) \(unit)")
    }
}
~~~

The status chip, a label whose text rolls (all scenes; the breathing dot is from voice, the dot that pops in from split, the Reduce Motion branch from weather and lift). Where its text changes, add `.animation(EaseMotion.spring(0.5), value: text)` at the call site.

~~~swift
/// Small grey capsule: a status label, not a button. With `dot` a coloured dot sits in front of the text.
/// With `pulse` the dot breathes.
struct EaseStatusChip: View {
    let text: String
    var dot: Color? = nil
    var pulse = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        let dotIn: AnyTransition = reduceMotion ? .opacity : .scale(scale: 0.4).combined(with: .opacity)
        let low: Double = pulse && !reduceMotion ? 0.3 : 1
        HStack(spacing: 8) {
            if let dot {
                Circle().fill(dot).frame(width: 10, height: 10)
                    .phaseAnimator([1.0, low]) { circle, level in
                        circle.opacity(level)
                    } animation: { _ in EaseMotion.pulse(0.6) }
                    .transition(dotIn)
            }
            Text(text)
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(EaseInk.secondary)
                .contentTransition(.numericText())
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(EaseInk.chip))
    }
}
~~~

## 10. Buttons: press, three states, selection

**Press.** The numbers come from `tap()` in every scene's demo: down with `spring(0.2)`, release with `spring(0.35, bounce: 0.2)`. The scenes draw the press themselves; here it is a `ButtonStyle`, so real taps get the same feedback. The touch dot is from lift (`LiftSceneTouchDot`).

~~~swift
/// Soft dot that shows a press. White for dark controls, a dark `ink` for white ones.
/// With Reduce Motion it only fades.
struct EaseTouchDot: View {
    let on: Bool
    var ink = Color.white.opacity(0.26)
    var size: CGFloat = 44
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        Circle()
            .fill(ink)
            .frame(width: size, height: size)
            .scaleEffect(on || reduceMotion ? 1 : 1.5)
            .opacity(on ? 1 : 0)
            .accessibilityHidden(true)
    }
}

/// Press feedback: the control shrinks a little and a soft dot comes in.
/// With Reduce Motion nothing scales: only the dot fades in.
struct EasePressStyle: ButtonStyle {
    var scale: CGFloat = 0.965
    var dot = Color.white.opacity(0.26)
    var dotSize: CGFloat = 44

    func makeBody(configuration: Configuration) -> some View {
        EasePressBody(label: configuration.label, pressed: configuration.isPressed,
                      scale: scale, dot: dot, dotSize: dotSize)
    }
}

struct EasePressBody: View {
    let label: ButtonStyleConfiguration.Label
    let pressed: Bool
    let scale: CGFloat
    let dot: Color
    let dotSize: CGFloat
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        label
            .overlay(EaseTouchDot(on: pressed, ink: dot, size: dotSize))
            .scaleEffect(pressed && !reduceMotion ? scale : 1)
            .animation(pressed ? EaseMotion.spring(0.2) : EaseMotion.spring(0.35, bounce: 0.2), value: pressed)
    }
}
~~~

Scale by size of the control:

| Control | Pressed scale | Touch dot | Scene |
|---|---|---|---|
| Main pill button | 0.965 | white 26 %, 44 pt | all six |
| Outline pill, height 40 | 0.96 | black 12 %, 40 pt | voice |
| Outline pill, height 34 | 0.95 | black 12 %, 40 pt | portfolio |
| Outline pill, height 30 | 0.965 | black 14 %, 38 pt | lift |
| Small round control (checkbox, rack disc) | 0.92 | black 12 %, 40 pt | voice, lift |
| 24 pt plus | 0.9 | black 12 %, 40 pt | split |

**The main button and its three looks.** From split, portfolio, weather and lift (`PillButton`); the Reduce Motion branches are from weather and lift.

~~~swift
/// Flat pill button. The title rolls to the new text when it changes; `done` puts a small check in front of it.
/// Switched off, it is pale grey with grey text. With `success` it turns fresh green with white text
/// and a small pop: the answer to an action that worked.
/// With Reduce Motion nothing scales: only the colour and the text change.
struct EasePillButton: View {
    let title: String
    var enabled = true
    var success = false
    var done = false
    let action: () -> Void
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        let peak: CGFloat = success && !reduceMotion ? 1.04 : 1
        let checkIn: AnyTransition = reduceMotion ? .opacity : .scale(scale: 0.5).combined(with: .opacity)
        Button(action: action) {
            HStack(spacing: 8) {
                if done {
                    Image(systemName: "checkmark")
                        .font(.system(size: 15, weight: .bold))
                        .transition(checkIn)
                }
                Text(title).contentTransition(.numericText())
            }
            .font(.system(size: 18, weight: .semibold))
            .foregroundStyle(success ? EaseInk.successText : (enabled ? Color.white : EaseInk.secondary))
            .frame(maxWidth: .infinity)
            .frame(height: 54)
            .background(Capsule().fill(success ? EaseInk.successFill : (enabled ? EaseInk.black : EaseInk.off)))
            .contentShape(Capsule())
        }
        .buttonStyle(EasePressStyle())
        .disabled(!enabled)
        .keyframeAnimator(initialValue: CGFloat(1), trigger: success) { view, pop in
            view.scaleEffect(pop)
        } keyframes: { _ in
            KeyframeTrack {
                SpringKeyframe(peak, duration: 0.14 * EaseMotion.k, spring: .snappy(duration: 0.5 * EaseMotion.k))
                SpringKeyframe(CGFloat(1), duration: 0.45 * EaseMotion.k, spring: .bouncy(duration: 0.5 * EaseMotion.k))
            }
        }
        .animation(EaseMotion.spring(0.6), value: title)
        .animation(EaseMotion.smooth(0.45), value: enabled)
        .animation(EaseMotion.smooth(0.35), value: success)
        .accessibilityLabel(title)
    }
}
~~~

- Black is an action to take. Pale grey is switched off, until the screen is ready (split: until every dish is assigned). Green is the answer to an action that worked, with `done` for the check. For the next action it goes back to black (split: "Done").
- The button never moves and never changes size. Only its title and colour change.
- Write titles so that they share letters and roll well: "Request", "Request $57.60", "Requested" (split); "Log set", "Set logged" (lift); "Add money", "Added $50.00" (portfolio).
- For a self-playing showcase, where no finger presses, use the scene version with a `pressed:` parameter (see `scenes/lift.md`).

**Selection.** From voice and portfolio (`TabPills`). This black capsule is the only place for `matchedGeometryEffect`. Those two scenes have no Reduce Motion branch; here the selection changes without animation then.

~~~swift
/// Row of outline pill buttons. One is selected: a black capsule slides behind its title.
struct EaseTabPills: View {
    let titles: [String]
    @Binding var selected: Int
    @Namespace private var ns
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        HStack(spacing: 8) {
            ForEach(titles.indices, id: \.self) { i in
                let on = selected == i
                Button {
                    if reduceMotion {
                        selected = i
                    } else {
                        withAnimation(EaseMotion.spring(0.5, bounce: 0.18)) { selected = i }
                    }
                } label: {
                    Text(titles[i])
                        .font(.system(size: 16, weight: .medium))
                        .foregroundStyle(on ? Color.white : EaseInk.primary)
                        .lineLimit(1)
                        .frame(maxWidth: .infinity)
                        .frame(height: 40)
                        .background {
                            ZStack {
                                Capsule().fill(Color.white)
                                Capsule().strokeBorder(EaseInk.outline, lineWidth: 1.5)
                                if on {
                                    Capsule().fill(EaseInk.black).matchedGeometryEffect(id: "selection", in: ns)
                                }
                            }
                        }
                        .contentShape(Capsule())
                }
                .buttonStyle(EasePressStyle(scale: 0.96, dot: Color.black.opacity(0.12), dotSize: 40))
            }
        }
    }
}
~~~

When the selection also changes a chart, use `spring(0.7, bounce: 0.12)` for both, so the capsule and the line move as one (portfolio). The content under the pills swaps as in section 8.

## 11. Small things pop

From split (dots, check badge), voice (checkbox), weather (bell, moon, stars), lift (the jump) and flight (ticks). Small things are the only ones with a bounce of 0.35 to 0.45. They answer a tap or follow the hero; they are never the big thing of a beat.

**A thing that keeps its place.** It is always in the layout and only scaled and faded, so nothing around it moves. The way out (`smooth(0.3)`, no bounce, all together) is from weather.

~~~swift
/// A small thing pops: it grows from `from` with a bouncy spring and keeps its place while it is hidden.
/// It leaves quickly and without bounce. With Reduce Motion it only fades.
struct EasePop: ViewModifier {
    let on: Bool
    var from: CGFloat = 0.3
    var duration: Double = 0.5
    var bounce: Double = 0.45
    var delay: Double = 0
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        content
            .scaleEffect(on || reduceMotion ? 1 : from)
            .opacity(on ? 1 : 0)
            .animation(on ? EaseMotion.spring(duration, bounce: bounce, delay: delay) : EaseMotion.smooth(0.3), value: on)
    }
}

extension View {
    func easePop(_ on: Bool, from: CGFloat = 0.3, duration: Double = 0.5, bounce: Double = 0.45, delay: Double = 0) -> some View {
        modifier(EasePop(on: on, from: from, duration: duration, bounce: bounce, delay: delay))
    }
}

extension AnyTransition {
    /// For a small thing that is inserted: a symbol in a tag, a dot in a chip, a check in a button.
    @MainActor static func easePop(from scale: CGFloat, reduceMotion: Bool) -> AnyTransition {
        reduceMotion ? .opacity : .scale(scale: scale).combined(with: .opacity)
    }
}
~~~

| Thing | Call | Scene |
|---|---|---|
| Coloured dot number `n` on a row | `.easePop(on, delay: Double(n) * 0.07)` | split |
| Check badge on a portrait | `.easePop(on, from: 0.4, bounce: 0.4)` | split |
| Moon | `.easePop(night, from: 0.4, duration: 0.4, bounce: 0.35)` | weather |
| Star number `n` (the moon is 0) | `.easePop(night, from: 0.2, duration: 0.4, bounce: 0.35, delay: 0.05 * Double(n))` | weather |

The control that gives way to such a thing leaves faster than the thing arrives: scale 0.6, opacity 0, blur 4 with `EaseMotion.smooth(0.3)` (the plus button in split). Both sit in one `ZStack(alignment: .trailing)`, so the row keeps its width.

**A thing that is inserted.** From weather (the bell in the tag). The container animates, the thing brings the transition.

~~~swift
/// Small white tag on a coloured panel: a label and a value. With `icon` a symbol pops in at its front.
/// Fix the tag at its trailing end, so it grows to the left when the symbol comes in.
struct EaseTag: View {
    let label: String
    let value: String
    var icon: String? = nil
    var tint = EaseInk.primary
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        HStack(spacing: 4) {
            if let icon {
                Image(systemName: icon)
                    .font(.system(size: 10, weight: .semibold))
                    .foregroundStyle(tint)
                    .transition(AnyTransition.easePop(from: 0.3, reduceMotion: reduceMotion))
            }
            Text(label)
                .foregroundStyle(EaseInk.secondary)
            Text(value)
                .fontWeight(.semibold)
                .foregroundStyle(EaseInk.primary)
        }
        .font(.system(size: 11.5, weight: .medium))
        .padding(.horizontal, 8)
        .frame(height: 20)
        .background(Capsule().fill(Color.white))
        .animation(EaseMotion.spring(0.5, bounce: 0.3), value: icon)
    }
}
~~~

**A checkbox.** From voice (`CheckCircle`, `TaskRow`).

~~~swift
/// Round checkbox. When it gets checked the black disc pops in and the check draws itself.
/// With Reduce Motion the disc only fades in.
struct EaseCheckCircle: View {
    let done: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        ZStack {
            Circle().fill(Color.white)
            Circle().strokeBorder(EaseInk.outline, lineWidth: 1.8)
            Circle().fill(EaseInk.black)
                .scaleEffect(done || reduceMotion ? 1 : 0.5)
                .opacity(done ? 1 : 0)
            Path { p in
                p.move(to: CGPoint(x: 0, y: 5))
                p.addLine(to: CGPoint(x: 4, y: 9))
                p.addLine(to: CGPoint(x: 11, y: 0.5))
            }
            .trim(from: 0, to: done ? 1 : 0)
            .stroke(Color.white, style: StrokeStyle(lineWidth: 2.2, lineCap: .round, lineJoin: .round))
            .frame(width: 11, height: 9.5)
        }
        .frame(width: 24, height: 24)
        .animation(EaseMotion.spring(0.45, bounce: 0.35), value: done)
        .accessibilityHidden(true)
    }
}

/// A line runs through a finished text from the left, a moment after the check.
/// Set the text's colour before this modifier, so it turns grey on the same curve.
/// With Reduce Motion the line only fades in.
struct EaseStrike: ViewModifier {
    let on: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        content
            .overlay(alignment: .leading) {
                Rectangle().fill(EaseInk.secondary)
                    .frame(height: 1.5)
                    .scaleEffect(x: on || reduceMotion ? 1 : 0, anchor: .leading)
                    .opacity(on || !reduceMotion ? 1 : 0)
            }
            .animation(EaseMotion.smooth(0.45, delay: 0.08), value: on)
    }
}

extension View {
    func easeStrike(_ on: Bool) -> some View {
        modifier(EaseStrike(on: on))
    }
}
~~~

- The title of a task is `Text(title).foregroundStyle(done ? EaseInk.secondary : EaseInk.primary).lineLimit(1).easeStrike(done)`.
- The circle is the control: wrap it in a `Button` with `EasePressStyle(scale: 0.92, dot: Color.black.opacity(0.12), dotSize: 40)` and give the button an accessibility label and value.
- After one tick the next tap waits 0.5 s (voice).

**The hero jumps once.** From lift (the barbell after "Log set"). It is the consequence of the tap, in the hero, while the button turns green.

~~~swift
/// The view makes one small jump when `trigger` turns true. With Reduce Motion nothing jumps.
struct EaseJump: ViewModifier {
    let trigger: Bool
    var height: CGFloat = 9
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        let lift: CGFloat = trigger && !reduceMotion ? -height : 0
        content
            .keyframeAnimator(initialValue: CGFloat(0), trigger: trigger) { view, y in
                view.offset(y: y)
            } keyframes: { _ in
                KeyframeTrack {
                    SpringKeyframe(lift, duration: 0.16 * EaseMotion.k, spring: .snappy(duration: 0.3 * EaseMotion.k))
                    SpringKeyframe(CGFloat(0), duration: 0.38 * EaseMotion.k, spring: .bouncy(duration: 0.4 * EaseMotion.k))
                }
            }
    }
}

extension View {
    func easeJump(_ trigger: Bool, height: CGFloat = 9) -> some View {
        modifier(EaseJump(trigger: trigger, height: height))
    }
}
~~~

**A wave of small pops.** From flight (`TickRow`). The delay of each piece counts from where the last change ended, so a second wave starts at the first new piece and not at the left edge.

~~~swift
/// Row of tick marks as a progress bar. When more ticks light up they do it one after the other,
/// each with a small pop, like a wave that starts at `from`.
/// With Reduce Motion the ticks only change their colour.
struct EaseTickRow: View {
    let lit: Int
    var from = 0                   // lit ticks before the last change, where the wave starts
    var count = 44
    var color = EaseInk.up
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        let peak: CGFloat = reduceMotion ? 1 : 1.9
        HStack(spacing: 0) {
            ForEach(0..<count, id: \.self) { i in
                let on = i < lit
                let delay = Double(max(0, i - from)) * 0.026
                Capsule()
                    .fill(on ? color : EaseInk.track)
                    .frame(width: 3, height: 11)
                    .keyframeAnimator(initialValue: CGFloat(1), trigger: on) { view, stretch in
                        view.scaleEffect(y: stretch)
                    } keyframes: { _ in
                        KeyframeTrack {
                            LinearKeyframe(CGFloat(1), duration: max(delay, 0.01) * EaseMotion.k)
                            SpringKeyframe(peak, duration: 0.13 * EaseMotion.k, spring: .snappy(duration: 0.5 * EaseMotion.k))
                            SpringKeyframe(CGFloat(1), duration: 0.42 * EaseMotion.k, spring: .bouncy(duration: 0.5 * EaseMotion.k))
                        }
                    }
                    .animation(EaseMotion.smooth(0.3, delay: delay), value: on)
                if i < count - 1 { Spacer(minLength: 0) }
            }
        }
        .frame(height: 11)
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Progress")
        .accessibilityValue("\(lit) of \(count)")
    }
}
~~~

- The model keeps both numbers and sets them together: `ticksFrom = ticks; ticks = count` (flight: 13 of 44, then 26, then 44). Flight colours each lit tick from light to full green with its own `Ink.tick(_:)`.
- The same rule, `delay = Double(max(0, i - from)) * step`, drives every wave. Steps: 0.026 s for ticks (flight), 0.016 s for live bars and 0.012 s for a calm waveform (voice), 0.009 s after a start of 0.55 s for bars that print themselves (flight).
- Bars grow in height, not in scale: from 5 pt to their level with `spring(0.45, bounce: 0.35)` (voice), or from `scaleEffect(y: 0.15, anchor: .bottom)` with `spring(0.5, bounce: 0.3)` (flight barcode).

## 12. One value drives everything

From weather (the sun's position), lift (the plates on the bar), split (the assigned dishes), portfolio (the range and the finger) and flight (the stage and the plane's position).

How to set it up:

- The model stores as little as it can: one position (weather: `sun`), one list (lift: `pairs`), one set (split: `assigned`), one index and one optional (portfolio: `range`, `scrub`), one enum (flight: `stage`).
- Everything the screen shows is a computed property of the model: the big number, the chip and its dot, the line under the number, the button's title, the fractions of a ring, the heights of a chart. The scenes' check lists test exactly this (lift: "20 plus twice their sum is the big number").
- One layout for all states. Flight's `Stage` gives every state its chip, labels, values, tick count, plane position and button title; the views never change.
- The views get the value. One `.animation(_, value:)` on the screen, or one `withAnimation`, moves the hero and rolls the numbers in the same transaction: `.animation(EaseMotion.spring(0.75, bounce: 0.16), value: model.assigned)` on the whole `VStack` (split).
- Where the frames in between must be computed (a place on a curve, a mixed colour, the shape of a line), the value becomes `animatableData`. SwiftUI then hands `body` or `path(in:)` every value in between, and nothing can cut a corner or cross-fade.

**Arcs and places on a ring.** From split (`RingArc`, `Orbit`, `RingChart`).

~~~swift
/// One arc of the ring, from `start` to `end` (both 0...1, clockwise from the top).
/// Both ends can animate. The arc is drawn a little shorter than asked, so that neighbours
/// with round ends leave a small white gap between them.
struct EaseRingArc: Shape {
    var start: Double
    var end: Double
    var line: CGFloat

    var animatableData: AnimatablePair<Double, Double> {
        get { AnimatablePair(start, end) }
        set { start = newValue.first; end = newValue.second }
    }

    func path(in rect: CGRect) -> Path {
        let radius = min(rect.width, rect.height) / 2 - line / 2
        let whole = start <= 0.0005 && end >= 0.9995
        let inset = whole ? 0 : (Double(line) / 2 + 3.2) / (2 * Double.pi * Double(radius))
        let a = start + inset
        let b = end - inset
        var path = Path()
        guard b > a else { return path }
        path.addArc(center: CGPoint(x: rect.midX, y: rect.midY), radius: radius,
                    startAngle: .degrees(a * 360 - 90), endAngle: .degrees(b * 360 - 90), clockwise: false)
        return path
    }
}

/// Puts a view on a circle. `turn` is the position, 0...1 clockwise from the top.
/// Because the turn animates (not the x and y), the view travels along the ring.
/// `animatableData` is `nonisolated`: SwiftUI reads and writes it while it animates, and it only touches `turn`.
struct EaseOrbit: Animatable {
    var turn: Double
    var radius: CGFloat

    nonisolated var animatableData: Double {
        get { turn }
        set { turn = newValue }
    }
}

extension EaseOrbit: ViewModifier {
    func body(content: Content) -> some View {
        let angle = turn * 2 * Double.pi - Double.pi / 2
        content.offset(x: radius * CGFloat(cos(angle)), y: radius * CGFloat(sin(angle)))
    }
}

/// A ring that shows parts of a whole. `fractions` are the parts, one colour each; their sum is
/// at most 1. What is not used yet stays grey.
struct EaseRingChart: View {
    let fractions: [Double]
    let colors: [Color]
    var drawn: Double = 1          // 0...1, how much of the grey ring is drawn
    var size: CGFloat = 196
    var line: CGFloat = 19

    var body: some View {
        let starts = fractions.indices.map { i in fractions[..<i].reduce(0, +) }
        let used = min(fractions.reduce(0, +), 1)
        ZStack {
            EaseRingArc(start: used, end: used + (1 - used) * drawn, line: line)
                .stroke(EaseInk.track, style: StrokeStyle(lineWidth: line, lineCap: .round))
                .opacity(used < 0.999 ? 1 : 0)
            ForEach(fractions.indices, id: \.self) { i in
                EaseRingArc(start: starts[i], end: starts[i] + fractions[i], line: line)
                    .stroke(colors[i % colors.count], style: StrokeStyle(lineWidth: line, lineCap: .round))
                    .opacity(fractions[i] > 0 ? 1 : 0)
            }
        }
        .frame(width: size, height: size)
        .animation(EaseMotion.spring(0.75, bounce: 0.16), value: fractions)
        .animation(EaseMotion.smooth(1.0), value: drawn)
    }
}
~~~

- The ring enters grey: `drawn` goes from 0 to 1 about 0.45 s after the sheet starts to rise (split).
- A marker rides on the middle of its arc: `.modifier(EaseOrbit(turn: starts[i] + fractions[i] / 2, radius: (size - line) / 2))`. While its fraction is 0 it is hidden at scale 0.5. Split's portraits are 47 pt.
- The number in the middle (33 pt bold) and the legend roll on the same spring. When the middle changes its meaning, its caption rolls too ("incl. 20% tip" to "paid back") and the number uses `spring(0.7, bounce: 0.1)`.

**A line that changes its shape.** From portfolio (`AnimatableVector`, `ChartShape`, `ChartView`). Line, area, base line and end dot use the same heights, so they morph as one. Every set of data must have the same number of points (portfolio: 48); resample real data to that count.

~~~swift
/// A row of numbers that SwiftUI can animate as one value. It lets the chart line change its shape smoothly.
struct EaseVector: VectorArithmetic, Sendable {
    var values: [Double]

    static var zero: EaseVector { EaseVector(values: []) }
    static func + (a: EaseVector, b: EaseVector) -> EaseVector { combine(a, b, +) }
    static func - (a: EaseVector, b: EaseVector) -> EaseVector { combine(a, b, -) }

    private static func combine(_ a: EaseVector, _ b: EaseVector,
                                _ op: (Double, Double) -> Double) -> EaseVector {
        let n = max(a.values.count, b.values.count)
        return EaseVector(values: (0..<n).map { i in
            op(i < a.values.count ? a.values[i] : 0, i < b.values.count ? b.values[i] : 0)
        })
    }

    mutating func scale(by rhs: Double) { values = values.map { $0 * rhs } }
    var magnitudeSquared: Double { values.reduce(0) { $0 + $1 * $1 } }
}

/// The parts of the chart as shapes that share one set of heights (0 = bottom, 1 = top).
/// Because the heights are animatable, line, area, base line and end dot all morph together.
struct EaseChartShape: Shape {
    enum Part: Sendable { case line, area, base, dot(CGFloat) }

    var heights: EaseVector
    var part: Part

    var animatableData: EaseVector {
        get { heights }
        set { heights = newValue }
    }

    func path(in rect: CGRect) -> Path {
        let v = heights.values
        guard v.count > 1 else { return Path() }
        func point(_ i: Int) -> CGPoint {
            CGPoint(x: rect.minX + rect.width * CGFloat(i) / CGFloat(v.count - 1),
                    y: rect.maxY - rect.height * CGFloat(v[i]))
        }
        var path = Path()
        switch part {
        case .base:
            path.move(to: CGPoint(x: rect.minX, y: point(0).y))
            path.addLine(to: CGPoint(x: rect.maxX, y: point(0).y))
        case .dot(let radius):
            let c = point(v.count - 1)
            path.addEllipse(in: CGRect(x: c.x - radius, y: c.y - radius, width: radius * 2, height: radius * 2))
        case .line, .area:
            path.move(to: point(0))
            for i in 1..<v.count { path.addLine(to: point(i)) }
            if case .area = part {
                path.addLine(to: CGPoint(x: rect.maxX, y: rect.maxY))
                path.addLine(to: CGPoint(x: rect.minX, y: rect.maxY))
                path.closeSubpath()
            }
        }
        return path
    }
}

/// A line chart with a soft area under it, a dotted line for the starting value and a dot at the end.
/// `drawn` draws the line from left to right. With `scrub` a finger is on the chart: a marker stands at that
/// place, the part to the right of it fades, and `label` is shown in a small pill above the marker.
struct EaseLineChart: View {
    let heights: [Double]
    var color = EaseInk.up
    var drawn: Double = 1
    var scrub: Double? = nil
    var label = ""

    private let height: CGFloat = 100

    var body: some View {
        let vec = EaseVector(values: heights)
        GeometryReader { g in
            let w = g.size.width
            let at = scrub ?? 1
            let on = scrub != nil
            let x = w * at
            let y = height - height * level(at)
            ZStack(alignment: .topLeading) {
                EaseChartShape(heights: vec, part: .base)
                    .stroke(EaseInk.outline, style: StrokeStyle(lineWidth: 1.6, lineCap: .round, dash: [0.5, 6]))

                chart(vec).opacity(on ? 0.26 : 0)
                chart(vec).mask(alignment: .leading) {
                    Rectangle().frame(width: on ? x : w + 30)
                }

                EaseChartShape(heights: vec, part: .dot(10)).fill(color.opacity(0.22))
                    .opacity(!on && drawn >= 1 ? 1 : 0)
                EaseChartShape(heights: vec, part: .dot(6.5)).fill(Color.white)
                    .opacity(!on && drawn >= 1 ? 1 : 0)
                EaseChartShape(heights: vec, part: .dot(4.5)).fill(color)
                    .opacity(!on && drawn >= 1 ? 1 : 0)

                // The finger's marker.
                Group {
                    Rectangle().fill(EaseInk.primary)
                        .frame(width: 1.5, height: height + 6)
                        .position(x: x, y: height / 2 - 3)
                    Circle().fill(EaseInk.primary)
                        .frame(width: 11, height: 11)
                        .overlay(Circle().strokeBorder(Color.white, lineWidth: 2))
                        .position(x: x, y: y)
                    Text(label)
                        .font(.system(size: 13, weight: .semibold))
                        .monospacedDigit()
                        .foregroundStyle(Color.white)
                        .padding(.horizontal, 9)
                        .frame(height: 23)
                        .background(Capsule().fill(EaseInk.black))
                        .fixedSize()
                        .position(x: min(max(x, 28), w - 28), y: -18)
                }
                .opacity(on ? 1 : 0)
            }
        }
        .frame(height: height)
        .animation(EaseMotion.spring(0.7, bounce: 0.12), value: heights)
        .animation(EaseMotion.smooth(1.1), value: drawn)
        .animation(EaseMotion.smooth(0.16), value: scrub)
    }

    private func chart(_ vec: EaseVector) -> some View {
        ZStack {
            EaseChartShape(heights: vec, part: .area)
                .fill(LinearGradient(colors: [color.opacity(0.26), color.opacity(0)], startPoint: .top, endPoint: .bottom))
                .opacity(drawn)
            EaseChartShape(heights: vec, part: .line)
                .trim(from: 0, to: drawn)
                .stroke(color, style: StrokeStyle(lineWidth: 2.4, lineCap: .round, lineJoin: .round))
        }
    }

    /// Height of the line at position 0...1, between two points.
    private func level(_ at: Double) -> CGFloat {
        guard heights.count > 1 else { return 0 }
        let p = min(max(at, 0), 1) * Double(heights.count - 1)
        let i = min(Int(p), heights.count - 2)
        return CGFloat(heights[i] + (heights[i + 1] - heights[i]) * (p - Double(i)))
    }
}
~~~

- Leave 30 pt above the chart for the marker's pill (portfolio: `.padding(.top, 30)`).
- The colour is data too: `color: loss ? EaseInk.down : EaseInk.up`. The percent chip, the note and the rows change inside the same `spring(0.7, bounce: 0.12)` as the line and the selection capsule.
- Set `drawn` to 1 about 0.5 s after the sheet starts to rise. Never swap in a chart library and never cross-fade two charts.
- Give the chart `.accessibilityElement(children: .ignore)` with a label and a value at the call site.

**Colours that follow a value.** From weather (`Tone`, `WeatherSceneSkyStrip`, `WeatherSceneSkyPanel`). A colour is not animated from A to B; it is mixed anew from the value in every frame.

~~~swift
/// One colour for each state of a screen (weather: day, golden hour, dusk), as 0xRRGGBB.
/// `at` mixes the two neighbours of a phase in plain RGB, so a colour can follow a value smoothly.
struct EaseTone: Sendable {
    let stops: [UInt32]

    init(_ stops: UInt32...) {
        self.stops = stops
    }

    /// Phase 0 is the first state, 1 the second, and so on, with fractions in between.
    func at(_ phase: Double) -> Color {
        guard !stops.isEmpty else { return .clear }
        let p = min(max(phase, 0), Double(stops.count - 1))
        let i = min(Int(p), max(stops.count - 2, 0))
        let from = stops[i]
        let to = stops[min(i + 1, stops.count - 1)]
        let f = p - Double(i)
        func channel(_ shift: UInt32) -> Double {
            let a = Double((from >> shift) & 0xFF)
            let b = Double((to >> shift) & 0xFF)
            return (a + (b - a) * f) / 255
        }
        return Color(red: channel(16), green: channel(8), blue: channel(0))
    }
}

/// A gradient that follows one value. The phase `t` is its animatable data, so the colours are
/// mixed anew for every value in between.
/// `animatableData` is `nonisolated`: SwiftUI reads and writes it while it animates, and it only touches `t`.
struct EaseToneStrip: Animatable {
    var t: Double
    let left: EaseTone
    let right: EaseTone

    nonisolated var animatableData: Double {
        get { t }
        set { t = newValue }
    }
}

extension EaseToneStrip: View {
    var body: some View {
        LinearGradient(colors: [left.at(t), right.at(t)], startPoint: .leading, endPoint: .trailing)
    }
}
~~~

- Weather's strip is `EaseToneStrip(t: phase, left: EaseTone(0x9ADFFE, 0xFEBD29, 0x382F92), right: EaseTone(0x34C9FD, 0xF97786, 0x8141F3))`.
- Weather hands its views the sun's position, not the phase, and computes the phase inside `body` with two soft ramps (`WeatherSceneDay.phase(at:)`: day until 0.80, golden hour at 815/875, dusk at 1). Do the same when your phase is not linear in your value.
- Build every view that follows the value this way: the sky panel (four gradient stops, lines, clouds, sun), the strip behind the sheet, the row whose grey tile slides to the column of the moment. They all get the same `t`, so they stay in step, also while one spring brings the value back.
- A marker on a curve needs this too. Give the curve one function, as weather's `WeatherSceneBow.point(at:in:)`, use it for the `Shape` and for the marker's `.position`, and put the marker in the `Animatable` view. Without it SwiftUI moves the marker in a straight line between two points.
- Things that switch at a threshold (moon and stars when the phase passes 1.5, the chip's text when it passes 0.5) pop or roll as in sections 9 and 11.

**A marker on a line.** From flight (`RouteLine`). This is the one place for `EaseMotion.travel`. Flight draws the dashes with its own `Ink.dash`; the kit uses `EaseInk.faint`.

~~~swift
/// A line between two dots with a marker on it. `progress` moves the marker (0 = start, 1 = end).
/// `drawn` draws the dashed line. With `trail` the part behind the marker turns solid.
struct EaseRouteLine: View {
    var progress: Double
    var trail = false
    var drawn: Double = 1          // 0...1, how much of the dashed line is drawn
    var symbol = "airplane"
    var gap: Color = .white        // colour behind the marker, so the line pauses around it

    var body: some View {
        GeometryReader { g in
            let w = g.size.width
            let mid = g.size.height / 2
            let inset: CGFloat = 15
            let x = inset + (w - 2 * inset) * progress
            let arrived = progress > 0.97
            ZStack(alignment: .leading) {
                Path { p in
                    p.move(to: CGPoint(x: 3, y: mid))
                    p.addLine(to: CGPoint(x: w - 3, y: mid))
                }
                .trim(from: 0, to: drawn)
                .stroke(EaseInk.faint, style: StrokeStyle(lineWidth: 1.3, lineCap: .round, dash: [3.5, 4.5]))

                Capsule().fill(EaseInk.primary)
                    .frame(width: max(x, 1), height: 1.8)
                    .opacity(trail ? 1 : 0)

                Circle().fill(trail ? EaseInk.primary : EaseInk.faint)
                    .frame(width: 6, height: 6)
                    .opacity(drawn > 0 ? 1 : 0)
                Circle().fill(EaseInk.faint)
                    .frame(width: 6, height: 6)
                    .position(x: w - 3, y: mid)
                    .opacity(drawn >= 1 && !arrived ? 1 : 0)

                Image(systemName: symbol)
                    .font(.system(size: 22, weight: .bold))
                    .foregroundStyle(EaseInk.primary)
                    .padding(.horizontal, 5)
                    .background(gap)
                    .position(x: x, y: mid)
                    .opacity(drawn > 0 ? 1 : 0)
            }
        }
        .frame(height: 30)
        .animation(EaseMotion.travel(1.5), value: progress)
        .animation(EaseMotion.smooth(1.0), value: drawn)
        .animation(EaseMotion.smooth(0.6), value: trail)
        .accessibilityHidden(true)
    }
}
~~~

- Flight sets `drawn = 1` and the first `progress` 0.55 s after the sheet starts to rise, and lights the ticks 0.4 s later. One big thing at a time.
- While the marker travels, the numbers next to it make one step with `spring(1.1)` (60 to 42 minutes).

**Live data.** From voice (`Waveform`). Each new bar stands up with `spring(0.45, bounce: 0.35)`. The age of a bar gives its colour, from red to near black over eight bars, with `smooth(0.3)`. The red position line follows with `smooth(0.22)`. The timer rolls with `spring(0.6)`. New words reveal with `y: 6, blur: 6` and turn from `EaseInk.faint` to `EaseInk.primary` with `smooth(0.4)` three words later. Open `scenes/voice.md` for the whole view.

## 13. Drags

From portfolio (a finger on the chart) and weather (a finger takes the sun). A drag does not animate a view; it changes the one value of section 12, and everything follows.

**A finger reads a value.** From portfolio. While the finger is down the value is its place from 0 to 1; without a finger it is `nil`.

~~~swift
extension View {
    /// A finger on a graphic. While it is down, `value` is its place from 0 (left) to 1 (right)
    /// and follows it softly. When it lets go, `value` goes back to `nil`.
    func easeScrub(_ value: Binding<Double?>) -> some View {
        overlay {
            GeometryReader { g in
                Color.clear
                    .contentShape(Rectangle())
                    .gesture(
                        DragGesture(minimumDistance: 0)
                            .onChanged { drag in
                                let x = min(max(drag.location.x / max(g.size.width, 1), 0), 1)
                                withAnimation(EaseMotion.smooth(0.16)) { value.wrappedValue = Double(x) }
                            }
                            .onEnded { _ in
                                withAnimation(EaseMotion.smooth(0.16)) { value.wrappedValue = nil }
                            }
                    )
            }
        }
    }
}
~~~

- Use it as `EaseLineChart(heights: h, scrub: model.scrub, label: model.time).easeScrub($model.scrub)` with `@Bindable var model`.
- The model computes from the finger: the amount (`scrub.map { value(at: $0) } ?? now`), the change, the percent, and the note ("today" becomes "at 11:42"). All of them have `numericText` and roll with the same `smooth(0.16)`.
- While the finger is down: a 1.5 pt marker line with an 11 pt dot, a small black pill with the time above it, the part to the right at 26 %, the end dot hidden (all in `EaseLineChart`).
- Nothing springs while a finger moves. Bounce belongs to the moment it lets go.
- `minimumDistance: 0` takes every touch. Inside a `ScrollView`, start the drag after a short long-press instead, or the list cannot scroll there.

**A finger takes a thing and lets it go.** From weather: hold with `spring(0.3, bounce: 0.2)`, follow with `smooth(0.16)`, and one `spring(0.9, bounce: 0.1)` brings everything back.

~~~swift
/// A wide soft circle around a thing that a finger holds. It shows the touch without dulling the thing.
/// With Reduce Motion it only fades.
struct EaseHoldHalo: View {
    let held: Bool
    var size: CGFloat = 62
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        Circle().fill(Color.white.opacity(0.3))
            .frame(width: size, height: size)
            .scaleEffect(held || reduceMotion ? 1 : 0.6)
            .opacity(held ? 1 : 0)
            .accessibilityHidden(true)
    }
}

extension View {
    /// A finger takes a value away from its rest and drags it between 0 and 1.
    /// When it lets go, one spring brings the value back to `rest`.
    /// `inset` is the space the path keeps to the left and right edge.
    func easeHoldDrag(_ value: Binding<Double>, held: Binding<Bool>, rest: Double, inset: CGFloat = 30) -> some View {
        overlay {
            GeometryReader { g in
                Color.clear
                    .contentShape(Rectangle())
                    .gesture(
                        DragGesture(minimumDistance: 0)
                            .onChanged { drag in
                                if !held.wrappedValue {
                                    withAnimation(EaseMotion.spring(0.3, bounce: 0.2)) { held.wrappedValue = true }
                                }
                                let span = max(g.size.width - 2 * inset, 1)
                                let t = min(max((drag.location.x - inset) / span, 0), 1)
                                withAnimation(EaseMotion.smooth(0.16)) { value.wrappedValue = Double(t) }
                            }
                            .onEnded { _ in
                                withAnimation(EaseMotion.spring(0.9, bounce: 0.1)) {
                                    held.wrappedValue = false
                                    value.wrappedValue = rest
                                }
                            }
                    )
            }
        }
    }
}
~~~

- The halo sits behind the thing, in the same `ZStack` and at the same `.position` (weather: 62 pt halo, 41 pt ring, 32 pt disc).
- While the value is away from its rest, the screen says so: weather's grey line rolls from "Feels like 23° · H 24° L 12°" to "Forecast for 19:47 · Feels like 18°" with `spring(0.5)`, and the big number follows with `spring(0.4)`.
- The way back is one spring for everything: position, colours, the tile in the row, the number. This works only because every view takes the same value as `animatableData` (section 12).
- If the drag should stay where the finger left it, leave out the reset of `value` and keep the `held` change.
- A drag needs a second way in for VoiceOver: give the graphic a label and a value and add `.accessibilityAdjustableAction` that moves the same value in steps.
- A showcase moves the value itself, in small eased steps: 0.04 s apart, each inside `withAnimation(EaseMotion.smooth(0.16))` (weather's `move`, portfolio's `drag`).

## 14. Things that slide on and off

From lift (pairs of plates on a bar) and flight (the ticket drops behind the button). The thing arrives from where it would really come from and leaves the same way.

**The list is the truth.** From lift (`LiftSceneLoad.Pair`, `LiftSceneDemo.add`). The model needs `import Observation`.

~~~swift
/// One block of a row. `id` is a counter, so SwiftUI knows which block slides on and which one slides off,
/// also when two blocks have the same value.
struct EaseBlock: Identifiable, Equatable, Sendable {
    let id: Int
    let value: Double
}

/// The list is the source of truth: the total and every label are computed from it.
@MainActor @Observable
final class EaseBlockList {
    private(set) var blocks: [EaseBlock] = []
    /// Counter for the identity of the blocks.
    @ObservationIgnored private var counter = 0

    var total: Double { blocks.reduce(0) { $0 + $1.value } }

    /// `byTap` is true when a finger asked for the block: it lands with a little more bounce.
    func add(_ value: Double, byTap: Bool = true) {
        counter += 1
        let block = EaseBlock(id: counter, value: value)
        let landing = byTap ? EaseMotion.spring(0.55, bounce: 0.3) : EaseMotion.spring(0.5, bounce: 0.25)
        withAnimation(landing) { blocks.append(block) }
    }

    func removeLast() {
        withAnimation(EaseMotion.spring(0.5, bounce: 0.15)) { blocks = Array(blocks.dropLast()) }
    }
}
~~~

**The row.** From lift (`LiftSceneBarbell`). Lift draws the same list twice, mirrored from the middle of the bar with a sign of −1 and 1; this row runs from the left and takes new blocks from the right.

~~~swift
/// A row of blocks that slide on from the trailing edge and off the same way. Every place is
/// computed from the list, so the blocks that stay move over by themselves. Clip the container,
/// so a block disappears at its edge. With Reduce Motion the blocks only fade.
struct EaseSlideRow<Item: Identifiable, Block: View>: View {
    let items: [Item]
    let width: (Item) -> CGFloat
    let inset: CGFloat
    let gap: CGFloat
    private let block: (Item) -> Block
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    init(items: [Item], width: @escaping (Item) -> CGFloat, inset: CGFloat = 16, gap: CGFloat = 2.5,
         @ViewBuilder block: @escaping (Item) -> Block) {
        self.items = items
        self.width = width
        self.inset = inset
        self.gap = gap
        self.block = block
    }

    /// Leading edge of place `index`. Place `items.count` is where the next block will sit.
    private func edge(_ index: Int) -> CGFloat {
        items.prefix(index).reduce(inset) { $0 + width($1) + gap }
    }

    var body: some View {
        GeometryReader { g in
            ZStack {
                ForEach(Array(items.enumerated()), id: \.element.id) { index, item in
                    let w: CGFloat = width(item)
                    let centre: CGFloat = edge(index) + w / 2
                    let slide: AnyTransition = reduceMotion
                        ? .opacity
                        : .offset(x: g.size.width + w - centre).combined(with: .opacity)
                    block(item)
                        .frame(width: w)
                        .position(x: centre, y: g.size.height / 2)
                        .zIndex(1)   // a leaving block stays in front of what moves in behind it
                        .transition(slide)
                }
            }
        }
    }
}

/// Usage: a grey panel with upright blocks. Buttons call `list.add(10)` and `list.removeLast()`.
struct EaseSlideShelf: View {
    let list: EaseBlockList

    var body: some View {
        EaseSlideRow(items: list.blocks, width: { _ in 20.5 }) { _ in
            RoundedRectangle(cornerRadius: 5, style: .continuous)
                .fill(EaseInk.primary)
                .frame(height: 100)
        }
        .frame(height: 164)
        .background(EaseInk.fill)
        .clipShape(RoundedRectangle(cornerRadius: 20, style: .continuous))
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Shelf")
        .accessibilityValue("\(list.blocks.count) blocks")
    }
}
~~~

- Identity is a counter, never the index and never the value. Insert and remove inside `withAnimation`, because views come and go.
- The transition is an offset to just outside the container on the thing's own side, combined with opacity. Do not replace it with a fade or a scale.
- Whatever depends on the list moves on the same spring: lift's clamps sit at place `pairs.count` and move in when a pair leaves; the labels under the plates roll.
- The numbers follow in the same frame: the total with `spring(0.4)`, the change chip and its words with `spring(0.5)`. The chip turns from green to red by its data, not by a second animation.
- Loading at the start: one thing every 0.32 s with `spring(0.5, bounce: 0.25)`, then 0.5 s of rest before the line that depends on it reveals with `.easeReveal(shown && loaded)` (lift).
- After a change the next tap waits 1.2 to 1.3 s (lift).

**A card drops away.** From flight (`Drop`): the ticket goes into the wallet button. It moves at once and fades only at the end, so the eye sees where it goes. The Reduce Motion branch follows weather and lift.

~~~swift
/// A card drops down behind whatever sits below it. It moves at once and fades only at the end.
/// With Reduce Motion it only fades.
struct EaseDrop: ViewModifier {
    let active: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        let moving = active && !reduceMotion
        content
            .opacity(active ? 0 : 1)
            .animation(EaseMotion.smooth(0.25, delay: reduceMotion ? 0 : 0.3), value: active)
            .scaleEffect(moving ? 0.8 : 1, anchor: .bottom)
            .rotationEffect(.degrees(moving ? 2.5 : 0))
            .offset(y: moving ? 96 : 0)
            .blur(radius: moving ? 6 : 0)
            .animation(EaseMotion.smooth(0.6), value: active)
    }
}
~~~

- The order of the two `.animation` lines matters: the first one covers only the opacity above it.
- The target must be drawn after the card, so the card passes behind it. In the sheet of section 17 the main button is drawn after the screens for this reason.
- In the same tap the target answers: the title rolls to "Added" with a check (flight). The next beat starts 0.65 s later.

## 15. Reduce Motion

From weather and lift, which read `@Environment(\.accessibilityReduceMotion)` in every part that scales, slides or pops. Flight, voice, split and portfolio have no such branches; the kit gives their parts one by the same rule: **fades and colours only**.

| Part | Normally | With Reduce Motion | Source of the branch |
|---|---|---|---|
| Reveal | fade, y 14, blur 10, scale 0.98 | fade | weather, lift |
| Sheet | slides from `height + 60`, `spring(0.85, bounce: 0.08)` | stays in place and fades with `smooth(0.35)` | weather, lift |
| Press | scale 0.965, dot shrinks from 1.5 | the dot fades in | weather, lift |
| Button turns green | colour, pop to 1.04 | colour and text | weather, lift |
| Check in the button, dot in a chip, symbol in a tag | scale with opacity | opacity | weather, lift |
| Moon, stars, halo, ring behind a pressed disc | scale from 0.2 to 0.8 | fade | weather, lift |
| Things slide on and off | offset with opacity | opacity | lift |
| Hero jumps | −9 pt | nothing | lift |
| Screen leaves, content swaps, card drops | blur, scale, offset | fade | kit, by the same rule |
| Selection capsule | slides | changes at once | kit, by the same rule |
| Tick stretches, live dot breathes, strike line grows | scale or pulse | colour or fade only | kit, by the same rule |

What stays the same, as in weather and lift:

- Digits and titles still roll (`numericText`), colours still change, staggers keep their delays, beats keep their rests.
- Data keeps moving, because the position is the information: the sun travels along its bow, a tile follows it, a chart takes its new shape, a value follows the finger.

How to write a branch (three forms, all in the kit):

- A thing that keeps its place: `.scaleEffect(on || reduceMotion ? 1 : from)` and leave the opacity alone (`EasePop`, `EaseReveal`).
- A thing that is inserted: `let slide: AnyTransition = reduceMotion ? .opacity : …` (`EaseSlideRow`, `EaseTag`).
- Keyframes: make the target equal to the rest value, `let peak: CGFloat = success && !reduceMotion ? 1.04 : 1` (`EasePillButton`, `EaseJump`).
- Never remove the state change itself. With Reduce Motion the screen reaches the same states; it only gets there by fading.

What weather and lift also do for VoiceOver; do the same:

- Controls are real `Button`s with a label. A drawn control has `.accessibilityElement(children: .ignore)`, a label and `.accessibilityAddTraits(.isButton)`.
- The hero graphic is one element with a label and a value computed from the source of truth ("Barbell", "150 kg, 25, 25, 15 kg per side"). A value that follows a finger adds `.updatesFrequently`.
- Decoration is hidden: strip, clouds, stars, touch dots, tiles (`.accessibilityHidden(true)`).
- A piece that is revealed later is hidden until then: `.accessibilityHidden(!loaded)`.
- A label and its value read as one: `.accessibilityElement(children: .combine)`.

## 16. Swift 6 notes

From weather and lift, which compile in Swift 6 language mode. The kit in this file follows them.

- **Model.** `@MainActor @Observable final class`, held by the view in `@State`. The file needs `import Observation`. Counters and caches that the views do not show are `@ObservationIgnored` (lift).
- **Flows.** A flow is `func run() async` on the model. Inside, every pause is `try await EaseMotion.wait(…)`, and one `do { } catch { }` around the whole flow ends it when the task is cancelled. Start it with `.task { await model.run() }` or `.task(id:)`; SwiftUI cancels both when the view goes away. Do not write `try?` in front of a wait inside a loop: the loop would run on without pauses after a cancel.
- **Static data.** Example data and tables live in an `enum` as `static let`, never `static var`. Give the value types you put there `Sendable` (weather: `Tone`, `Hour`; lift: `Plate`, `Pair`).
- **Transitions.** A static transition that creates view modifiers is `@MainActor` (sections 7, 8 and 11).
- **Animatable views and modifiers.** Declare the type with `Animatable` only, give it `nonisolated var animatableData`, and add `View` or `ViewModifier` in an extension (`EaseOrbit`, `EaseToneStrip`). The data must touch nothing but plain stored values.
- **Shapes.** A `Shape` needs no annotation. Everything it stores must be `Sendable` (`EaseVector`, `EaseChartShape.Part`).
- **No escape hatches.** Do not answer an error with `@unchecked Sendable`, `nonisolated(unsafe)`, `@preconcurrency import`, `DispatchQueue.main` or `Task.detached`.

**The four older scenes.** Flight, voice, split and portfolio do not mark these places. If the project builds in Swift 6 language mode and the compiler stops there, change the isolation only. Every number stays.

~~~swift
// Flight and voice: the transition creates view modifiers, and those belong to the main actor.
extension AnyTransition {
    @MainActor static var blurOut: AnyTransition {
        .asymmetric(insertion: .identity,
                    removal: .modifier(active: BlurOut(active: true), identity: BlurOut(active: false)))
    }
}

// Split: SwiftUI reads and writes `animatableData` while it animates, and it only touches `turn`.
struct Orbit: Animatable {
    var turn: Double
    var radius: CGFloat

    nonisolated var animatableData: Double {
        get { turn }
        set { turn = newValue }
    }
}

extension Orbit: ViewModifier {
    func body(content: Content) -> some View {
        let angle = turn * 2 * Double.pi - Double.pi / 2
        content.offset(x: radius * CGFloat(cos(angle)), y: radius * CGFloat(sin(angle)))
    }
}
~~~

- If the compiler names a `static let` in `Flight`, `Note`, `Bill` or `Market`, add `Sendable` to the struct it holds (`Cell`, `Person`, `Dish`, `Holding`).
- The scenes' `Motion.wait` does not throw and their loops check `Task.isCancelled` once per round. Leave that as it is when you install a scene. In your own code use the throwing wait.
- Report every such change in your answer, as the scene's "After it runs" section asks.

## 17. Skeleton: a whole screen

The frame is from all six scenes (`RootView`), its Reduce Motion branch from weather and lift. The screen is the "Send money" example of `SKILL.md`: hero and source of truth are the amount; four beats (enter, type, send, receipt).

**The sheet.** Add it to `EaseParts.swift`.

~~~swift
/// The Easeout frame: a coloured strip, a white sheet that slides up over it, the screens of the
/// flow, and the main button at the bottom.
/// With Reduce Motion the sheet stays in place and fades instead of sliding.
struct EaseSheet<Strip: View, Content: View, Footer: View>: View {
    let up: Bool
    let height: CGFloat
    private let strip: Strip
    private let content: Content
    private let footer: Footer
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    init(up: Bool, height: CGFloat = 640,
         @ViewBuilder strip: () -> Strip, @ViewBuilder content: () -> Content, @ViewBuilder footer: () -> Footer) {
        self.up = up
        self.height = height
        self.strip = strip()
        self.content = content()
        self.footer = footer()
    }

    var body: some View {
        ZStack(alignment: .bottom) {
            strip
                .ignoresSafeArea()
                .accessibilityHidden(true)

            VStack(spacing: 0) {
                // The screens of the flow. Each one leaves with `easeBlurOut`.
                ZStack(alignment: .top) { content }
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // The button stays in place on every screen. It is drawn after the screens,
                // so a card can drop behind it. It is the last in and the first out.
                footer
                    .easeReveal(up, delay: up ? 0.5 : 0)
            }
            .padding(.horizontal, 18)
            .padding(.top, 32)
            .padding(.bottom, 40)
            .frame(maxWidth: .infinity)
            .frame(height: height)
            .background(
                UnevenRoundedRectangle(topLeadingRadius: 40, topTrailingRadius: 40, style: .continuous)
                    .fill(Color.white)
                    .padding(.bottom, -200)   // white also below the screen edge while the sheet springs
            )
            .offset(y: up || reduceMotion ? 0 : height + 60)
            .opacity(up || !reduceMotion ? 1 : 0)
            .animation(reduceMotion ? EaseMotion.smooth(0.35) : EaseMotion.spring(0.85, bounce: 0.08), value: up)
        }
        .ignoresSafeArea(edges: .bottom)
        .environment(\.colorScheme, .light)   // the Easeout look is drawn for light mode only
    }
}
~~~

**The screen.** One file per type in a real project; shown together here.

~~~swift
import SwiftUI
import Observation

enum TransferInk {
    static let stripLeft = Color(red: 0.655, green: 0.953, blue: 0.816)
    static let stripRight = Color(red: 0.204, green: 0.827, blue: 0.6)
}

enum TransferPhase: Equatable { case form, receipt }

/// Source of truth: the amount and whether it was sent. Every text on the screen is computed from them.
@MainActor @Observable
final class TransferModel {
    // Example data. It adds up: 842.50 − 20.00 = 822.50.
    static let recipient = "Mia"
    static let steps: [Double] = [5, 20, 50]
    let balanceBefore = 842.50

    var phase: TransferPhase = .form
    var amount = 0.0
    var sent = false

    var balance: Double { balanceBefore - (sent ? amount : 0) }
    var ready: Bool { amount > 0 }
    /// Green while the form still shows: the answer to the tap.
    var success: Bool { sent && phase == .form }
    var button: String {
        if phase == .receipt { return "Done" }
        if sent { return "Sent" }
        return ready ? "Send " + Self.money(amount) : "Send"
    }

    static func money(_ value: Double) -> String { String(format: "$%.2f", value) }

    func add(_ step: Double) {
        guard !sent, amount + step <= balanceBefore else { return }
        amount += step
    }

    func send() {
        guard ready, !sent else { return }
        sent = true
    }

    /// After the success has rested, the form melts away and the receipt comes in.
    func showReceipt() async {
        guard sent, phase == .form else { return }
        do {
            try await EaseMotion.wait(1.5)
            withAnimation(EaseMotion.smooth(0.5)) { phase = .receipt }
        } catch {
            // Cancelled: the view went away, so the flow ends here.
        }
    }
}

/// The whole screen: strip, sheet, the two steps of the flow and the main button.
struct TransferView: View {
    var onDone: () -> Void = {}
    @State private var model = TransferModel()
    @State private var sheetUp = false

    var body: some View {
        EaseSheet(up: sheetUp) {
            LinearGradient(colors: [TransferInk.stripLeft, TransferInk.stripRight],
                           startPoint: .leading, endPoint: .trailing)
        } content: {
            switch model.phase {
            case .form:
                TransferForm(model: model)
                    .transition(AnyTransition.easeBlurOut)
            case .receipt:
                TransferReceipt(model: model)
                    .transition(AnyTransition.easeBlurOut)
            }
        } footer: {
            // Pale grey until there is an amount, black with the amount, green after the tap, black for "Done".
            EasePillButton(title: model.button, enabled: model.ready, success: model.success, done: model.success) {
                if model.phase == .receipt { onDone() } else { model.send() }
            }
        }
        .onAppear { sheetUp = true }
        // The timed step. SwiftUI cancels it when the view goes away.
        .task(id: model.sent) { await model.showReceipt() }
    }
}

/// The form. It comes in piece by piece from top to bottom.
struct TransferForm: View {
    let model: TransferModel
    @State private var shown = false

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Send money")
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                Spacer()
                EaseStatusChip(text: "To " + TransferModel.recipient)
            }
            .padding(.horizontal, 4)
            .easeReveal(shown, delay: lead)

            // The hero: the amount. Its digits roll with every tap.
            Text(TransferModel.money(model.amount))
                .font(.system(size: 45, weight: .bold))
                .tracking(-1)
                .monospacedDigit()
                .foregroundStyle(EaseInk.primary)
                .contentTransition(.numericText(value: model.amount))
                .animation(EaseMotion.spring(0.8), value: model.amount)
                .padding(.leading, 3)
                .padding(.top, 12)
                .easeReveal(shown, delay: lead + 0.06)

            // The consequence of "Send": the balance rolls down.
            Text("Balance " + TransferModel.money(model.balance))
                .font(.system(size: 17))
                .monospacedDigit()
                .foregroundStyle(EaseInk.secondary)
                .contentTransition(.numericText(value: model.balance))
                .animation(EaseMotion.spring(0.8), value: model.sent)
                .padding(.leading, 4)
                .padding(.top, 6)
                .easeReveal(shown, delay: lead + 0.12)

            HStack(spacing: 8) {
                ForEach(TransferModel.steps, id: \.self) { step in
                    Button {
                        model.add(step)
                    } label: {
                        Text("+" + TransferModel.money(step))
                            .font(.system(size: 16, weight: .medium))
                            .foregroundStyle(EaseInk.primary)
                            .lineLimit(1)
                            .frame(maxWidth: .infinity)
                            .frame(height: 40)
                            .background(Capsule().fill(Color.white))
                            .overlay(Capsule().strokeBorder(EaseInk.outline, lineWidth: 1.5))
                            .contentShape(Capsule())
                    }
                    .buttonStyle(EasePressStyle(scale: 0.96, dot: Color.black.opacity(0.12), dotSize: 40))
                    .accessibilityLabel("Add " + TransferModel.money(step))
                }
            }
            .padding(.top, 30)
            .easeReveal(shown, delay: lead + 0.18)
        }
        .onAppear { shown = true }
    }
}

/// The receipt. It reveals itself while the form is still melting away.
struct TransferReceipt: View {
    let model: TransferModel
    @State private var shown = false

    /// The old screen is still melting away, so the new pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Receipt")
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                Spacer()
                EaseStatusChip(text: "Sent", dot: EaseInk.successFill)
            }
            .padding(.horizontal, 4)
            .easeReveal(shown, delay: lead)

            Text(TransferModel.money(model.amount))
                .font(.system(size: 45, weight: .bold))
                .tracking(-1)
                .monospacedDigit()
                .foregroundStyle(EaseInk.primary)
                .padding(.leading, 3)
                .padding(.top, 12)
                .easeReveal(shown, delay: lead + 0.06)

            HStack(spacing: 0) {
                EaseSwapCell(label: "To", value: TransferModel.recipient)
                Rectangle().fill(EaseInk.hairline).frame(width: 1, height: 56)
                EaseSwapCell(label: "Balance", value: TransferModel.money(model.balance), column: 1)
            }
            .frame(height: 90)
            .background(RoundedRectangle(cornerRadius: 24, style: .continuous).fill(EaseInk.fill))
            .padding(.top, 30)
            .easeReveal(shown, delay: lead + 0.12)
        }
        .onAppear { shown = true }
    }
}

#Preview {
    TransferView()
}
~~~

What the skeleton shows, and what to keep when you change it:

- **Enter.** `sheetUp` raises the sheet; each block has `.easeReveal(shown, delay:)` from `lead` downward; the button comes last, at 0.5 s.
- **One source of truth.** `amount` and `sent`. The button's title and look, the balance and the receipt are computed. Nothing is typed twice.
- **Three steps of feedback.** Press (the pill gives way), answer (green, "Sent", a check), consequence (the balance rolls down).
- **Rest, then leave.** 1.5 s after the success, `withAnimation(EaseMotion.smooth(0.5))` changes the phase. The form melts, the sheet is almost empty, the receipt reveals. The button stays and rolls to "Done".
- **The timed step can be cancelled.** It lives in `.task(id:)` and waits with the throwing `EaseMotion.wait`.
- **Real taps.** Nothing plays by itself.
- Show the screen from the app: a `.fullScreenCover`, a navigation destination, a preview or a debug entry. Never add a second `@main`.
- Sizes are fixed here, as in the scenes. For an app screen see the Dynamic Type notes in `references/screen-layout.md`.
- Look at it once with the launch argument `-slow 4`.

**A showcase that plays itself.** From weather and lift (`run`, `tap`). Only when the user wants to watch the motion without touching. The loop ends as soon as the view goes away.

~~~swift
/// Plays the beats by itself. Start it with `.task { await demo.run() }` on the root view.
@MainActor @Observable
final class TransferDemo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var round = 0                    // a new number makes the screen come in fresh: `.id(demo.round)`
    var pressed: String? = nil       // control that is being "tapped"

    private func tap(_ id: String) async throws {
        withAnimation(EaseMotion.spring(0.2)) { pressed = id }
        try await EaseMotion.wait(0.17)
        withAnimation(EaseMotion.spring(0.35, bounce: 0.2)) { pressed = nil }
        try await EaseMotion.wait(0.08)
    }

    func run() async {
        do {
            while !Task.isCancelled {
                // Reset while the sheet is down.
                pressed = nil; sheetUp = false
                round += 1
                try await EaseMotion.wait(0.12)

                // 1. The sheet slides up, the pieces come in one after the other.
                sheetUp = true
                try await EaseMotion.wait(0.55)

                // 2. One beat: a tap, the change in the model, a rest.
                try await tap("main")
                try await EaseMotion.wait(1.5)

                // 3. The sheet slides away, then everything starts again.
                sheetUp = false
                try await EaseMotion.wait(0.5)
            }
        } catch {
            // Cancelled: the view went away, so the flow ends here.
        }
    }
}
~~~

- A showcase draws the press itself, because no finger is there: its controls take `pressed:` and show the touch dot, as the scene parts do (`scenes/lift.md`). Its state lives in the demo, and the model's values change between the taps.
- Rests between the beats come from the table in section 4.

When the screen stands, hold it against the rules of thumb in section 2, then go through `references/checklist.md`.
