# Easeout scene: Lift, a loaded barbell (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

One screen inside a white sheet over a lime strip: logging one set of a barbell lift. Its hero is a barbell in side view, loaded with coloured weight plates. Everything that shows the weight follows one list, the plates on one side of the bar: the big number, the chip that compares with last week, and the labels under the barbell. It plays in five beats:

1. **Loading.** The sheet slides up and the content comes in piece by piece. Three pairs of plates slide onto the bar from its two ends, one pair after the other, and the big number counts up: 20, 70, 120, 140.
2. **A pair off.** A tap on the outline pill "Remove last": the outer pair slides off the ends of the bar, the clamps move in, the number rolls to 120 and the chip turns red, "−15 kg less than last week".
3. **A pair on.** A tap on the yellow 15 in the rack: a pair of yellow plates slides onto both ends, the number rolls to 150 and the chip turns green, "+15 kg more than last week".
4. **Logged.** A tap on the black pill button "Log set": it turns green with a small pop and says "Set logged" with a check, the barbell makes one small jump, and the chip at the top rolls to "3 of 5 done".
5. **Again.** The sheet slides away and it starts again.

`LiftSceneDemo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files.
- Create the five files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, plate sizes, durations, curves and delays. These numbers are the design.
- The big number, the chip, the text next to it and the labels under the barbell are all computed from the list of plates in `LiftSceneLoad`. Never write one of them by hand, and never let one of them lag behind the others.
- Plates slide along the bar, on from the outside and off to the outside. Do not replace this with a fade or a scale.
- Keep `.contentTransition(.numericText(value:))` on the weight, so the digits roll.
- The pill button has three looks: black = an action to take, pale grey = switched off, green = the action worked. Keep all three.
- Labels and buttons look different on purpose. `LiftSceneStatusChip` and `LiftSceneChangeChip` are labels. The outline pill "Remove last", the round plates of the rack and `LiftScenePillButton` are buttons. Do not restyle one as the other.
- No `matchedGeometryEffect`.
- Keep `.id(demo.round)` on `LiftSceneView`; it makes the screen come in fresh each time.
- Keep all timing inside `LiftSceneMotion`. Do not write durations in other places.
- Every type and every file of the scene starts with `LiftScene`, so nothing collides with my project, also when it is a fitness app with its own `LiftView`. Do not rename them.
- If my project already has an `@main` app, do not add a second one: leave out the file `LiftSceneApp.swift` (it holds nothing but the entry point), keep the other four files unchanged, and show `LiftSceneRootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- The scene respects Reduce Motion and stops its loop when the view goes away. Keep both.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to change.
3. Then ask me one question: "Should I connect this to your real workout data?" If yes, remove the auto-play (`demo.run()` and the timeline in `LiftSceneDemo`), fill the list of plates and last week's weight from my model, add a pair when a round plate of the rack is tapped, remove one with "Remove last", and save the set from the pill button. Keep every animation as it is.

## What I may ask you to change

- The example lift: the name, the bar's weight, the plates at the start and last week's weight live in `LiftSceneLoad` in `LiftSceneModel.swift`.
- The plates: weight, colour and size of each plate are one table in `LiftSceneLoad`. For pounds, change that table and the unit text.
- The colours of the strip: `LiftSceneInk`.

## Check yourself before you answer

- With the example data the screen shows 140 kg and "+5 kg" after loading, 120 kg and "−15 kg" after the pair is removed, and 150 kg and "+15 kg" after the yellow pair is added.
- The labels under the barbell always list exactly the plates that are on the bar, mirrored left and right, and 20 plus twice their sum is the big number.
- The left and the right side of the bar are always mirror images.
- The button is black before the tap and green after it.
- Light mode only; `LiftSceneRootView` sets the light colour scheme for its own content.

## The code

### `LiftSceneModel.swift`

```swift
import SwiftUI
import Observation

// Colours of the scene (flat look, no glass).
enum LiftSceneInk {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)
    static let secondary = Color(red: 0.459, green: 0.463, blue: 0.498)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)           // button that is switched off
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let successFill = Color(red: 0.133, green: 0.773, blue: 0.369)   // button after an action worked
    static let successText = Color.white
    static let stripLeft = hex(0xCDF78C)     // the strip behind the top of the sheet: its left end
    static let stripRight = hex(0x83CA1D)    // and its right end
    static let open = hex(0x94D908)          // chip dot while the set is open
    static let done = hex(0x22C55E)          // chip dot after the set was logged
    static let gainFill = hex(0xD8F8B0)      // change chip: more than last week
    static let gainText = hex(0x166534)
    static let lossFill = hex(0xFDD2D4)      // change chip: less than last week
    static let lossText = hex(0x940B0A)
    static let panel = hex(0xF0F0F5)         // behind the barbell
    static let bar = hex(0xC3C3CD)
    static let clamp = hex(0x605E63)
    static let outline = hex(0xD4D4D8)       // around the "Remove last" pill
    static let plateOutline = hex(0xC3C4C9)  // around the white 5 kg plate
    static let press = hex(0xE4E4E9)         // ring behind a rack disc that is being tapped
    static let pressDot = Color.black.opacity(0.14)   // touch dot on a white control

    static func hex(_ value: UInt32) -> Color {
        Color(red: Double((value >> 16) & 0xFF) / 255,
              green: Double((value >> 8) & 0xFF) / 255,
              blue: Double(value & 0xFF) / 255)
    }
}

/// All timing goes through here. Launch with `-slow 8` to run everything eight times slower:
/// the recording is then blended back to normal speed, which gives real motion blur.
/// `-warmup 16` waits first, so the home indicator has faded before the flow starts.
enum LiftSceneMotion {
    static let k: Double = {
        let v = UserDefaults.standard.double(forKey: "slow")
        return v > 0 ? v : 1
    }()
    static let warmup = UserDefaults.standard.double(forKey: "warmup")

    /// Soft ease-out, the main curve.
    static func smooth(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.2, 0.9, 0.2, 1, duration: d * k).delay(delay * k)
    }
    static func spring(_ d: Double, bounce: Double = 0, delay: Double = 0) -> Animation {
        .spring(duration: d * k, bounce: bounce).delay(delay * k)
    }
    /// Throws when the task is cancelled, so a flow that waits here stops at once.
    static func wait(_ seconds: Double) async throws {
        try await Task.sleep(for: .seconds(seconds * k))
    }
}

// Example data. It adds up: bar 20 + 2 x (25 + 25 + 10) = 140 kg, which is 5 more than last week's 135.
// Without the 10s it is 120 (15 less), with 15s in their place it is 150 (15 more).
enum LiftSceneLoad {
    /// How a plate of one weight looks: upright on the bar, and as a round disc in the rack.
    struct Plate: Sendable {
        let weight: Double
        let ink: Color               // on the bar
        let width: CGFloat           // on the bar
        let height: CGFloat          // on the bar
        let disc: Color              // in the rack
        let size: CGFloat            // diameter in the rack
        var outlined = false         // white with a grey outline
    }

    /// A pair of plates on the bar, one on each side. `id` is a counter, so SwiftUI knows
    /// which pair slides on and which one slides off.
    struct Pair: Identifiable, Equatable, Sendable {
        let id: Int
        let weight: Double
    }

    static let lift = "Back squat"
    static let bar = 20.0                            // the empty bar, in kg
    static let lastWeek = 135                        // last week's lift, in kg
    static let set = 3
    static let sets = 5
    /// The plates of one side at the start of the story, from the inside out.
    static let start: [Double] = [25, 25, 10]
    /// The pair that is added from the rack in the story.
    static let next = 15.0

    /// The plates of the rack, from heavy to light.
    static let plates: [Plate] = [
        Plate(weight: 25, ink: LiftSceneInk.hex(0xEF4241), width: 20.5, height: 100, disc: LiftSceneInk.hex(0xF1413D), size: 47),
        Plate(weight: 20, ink: LiftSceneInk.hex(0x1774FD), width: 19, height: 92, disc: LiftSceneInk.hex(0x1774FD), size: 43),
        Plate(weight: 15, ink: LiftSceneInk.hex(0xFAC003), width: 17.5, height: 84, disc: LiftSceneInk.hex(0xFFC805), size: 38),
        Plate(weight: 10, ink: LiftSceneInk.hex(0x22C55E), width: 16.5, height: 76, disc: LiftSceneInk.hex(0x22C55E), size: 35),
        Plate(weight: 5, ink: Color.white, width: 12, height: 56, disc: Color.white, size: 32, outlined: true),
        Plate(weight: 2.5, ink: LiftSceneInk.hex(0x1C1C1C), width: 9, height: 44, disc: LiftSceneInk.hex(0x1C1C1C), size: 31),
    ]

    static let chipOpen = "Set \(set) of \(sets)"
    static let chipDone = "\(set) of \(sets) done"
    static let barLabel = "Bar " + text(bar)

    /// The look of the plate with this weight.
    static func plate(_ weight: Double) -> Plate {
        plates.first { $0.weight == weight } ?? plates[0]
    }

    /// A weight as text without "kg": "25", "2.5".
    static func text(_ kg: Double) -> String {
        kg == kg.rounded() ? String(Int(kg)) : String(kg)
    }

    /// The whole barbell in kg: the bar and the plates of both sides. Pairs always give a whole number.
    static func total(_ side: [Double]) -> Int {
        Int((bar + 2 * side.reduce(0, +)).rounded())
    }

    /// Difference to last week in kg.
    static func change(_ side: [Double]) -> Int {
        total(side) - lastWeek
    }

    /// The text of the change chip: "+5 kg" or "−15 kg", with a real minus sign.
    static func changeText(_ side: [Double]) -> String {
        let d = change(side)
        let sign = d > 0 ? "+" : (d < 0 ? "\u{2212}" : "±")
        return "\(sign)\(abs(d)) kg"
    }

    /// The grey text next to the change chip.
    static func changeWords(_ side: [Double]) -> String {
        let d = change(side)
        return d > 0 ? "more than last week" : (d < 0 ? "less than last week" : "same as last week")
    }

    /// The label under the left plates: their weights from the outside in. Empty without plates.
    static func leftLabel(_ side: [Double]) -> String {
        side.reversed().map { text($0) }.joined(separator: " · ")
    }

    /// The label under the right plates: their weights from the inside out. Empty without plates.
    static func rightLabel(_ side: [Double]) -> String {
        side.map { text($0) }.joined(separator: " · ")
    }

    /// What VoiceOver says about the barbell: the total and the plates per side.
    static func spoken(_ side: [Double]) -> String {
        let load = side.isEmpty ? "no plates" : side.map { text($0) }.joined(separator: ", ") + " kg per side"
        return "\(total(side)) kg, \(load)"
    }
}

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
@MainActor @Observable
final class LiftSceneDemo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var round = 0                    // a new number makes the screen come in fresh
    var pressed: String? = nil       // control that is being "tapped": "main", "remove" or a rack disc ("15")
    var pairs: [LiftSceneLoad.Pair] = []   // the plate pairs on the bar, from the inside out
    var loaded = false               // the bar is loaded: the line with the change is shown
    var logged = false               // the set was logged

    /// Counter for the identity of the pairs.
    @ObservationIgnored private var counter = 0

    /// The plates of one side, from the inside out. Everything that shows the weight is computed from it.
    var side: [Double] { pairs.map { $0.weight } }
    /// The big number.
    var total: Int { LiftSceneLoad.total(side) }
    var gain: Bool { LiftSceneLoad.change(side) >= 0 }
    var change: String { LiftSceneLoad.changeText(side) }
    var words: String { LiftSceneLoad.changeWords(side) }
    var chip: String { logged ? LiftSceneLoad.chipDone : LiftSceneLoad.chipOpen }
    var chipDot: Color { logged ? LiftSceneInk.done : LiftSceneInk.open }
    var button: String { logged ? "Set logged" : "Log set" }

    private func tap(_ id: String) async throws {
        withAnimation(LiftSceneMotion.spring(0.2)) { pressed = id }
        try await LiftSceneMotion.wait(0.17)
        withAnimation(LiftSceneMotion.spring(0.35, bounce: 0.2)) { pressed = nil }
        try await LiftSceneMotion.wait(0.08)
    }

    /// Puts a pair on the bar. Every pair gets the next number of the counter as its identity.
    private func add(_ weight: Double, _ animation: Animation) {
        counter += 1
        let pair = LiftSceneLoad.Pair(id: counter, weight: weight)
        withAnimation(animation) { pairs.append(pair) }
    }

    func run() async {
        do {
            if LiftSceneMotion.warmup > 0 { try await Task.sleep(for: .seconds(LiftSceneMotion.warmup)) }
            while !Task.isCancelled {
                // Reset while the sheet is down.
                pressed = nil; sheetUp = false; pairs = []; loaded = false; logged = false
                round += 1
                try await LiftSceneMotion.wait(0.12)

                // 1. The sheet slides up, the pieces come in one after the other.
                sheetUp = true
                try await LiftSceneMotion.wait(0.85)

                // 2. The bar gets loaded, pair by pair. The big number counts up with it.
                //    Then the line with the change comes in.
                let start = LiftSceneLoad.start
                for (n, weight) in start.enumerated() {
                    add(weight, LiftSceneMotion.spring(0.5, bounce: 0.25))
                    try await LiftSceneMotion.wait(n + 1 < start.count ? 0.32 : 0.5)
                }
                loaded = true
                try await LiftSceneMotion.wait(1.3)

                // 3. "Remove last": the outer pair slides off the ends of the bar, the chip turns red.
                try await tap("remove")
                withAnimation(LiftSceneMotion.spring(0.5, bounce: 0.15)) { pairs = Array(pairs.dropLast()) }
                try await LiftSceneMotion.wait(1.2)

                // 4. A tap on the yellow 15 in the rack: a pair of 15s slides on, the chip turns green.
                try await tap(LiftSceneLoad.text(LiftSceneLoad.next))
                add(LiftSceneLoad.next, LiftSceneMotion.spring(0.55, bounce: 0.3))
                try await LiftSceneMotion.wait(1.3)

                // 5. "Log set": the button turns green, the barbell makes one small jump,
                //    the chip at the top says that the set is done.
                try await tap("main")
                logged = true
                try await LiftSceneMotion.wait(1.7)

                // 6. The sheet slides away, then everything starts again.
                sheetUp = false
                try await LiftSceneMotion.wait(0.5)
            }
        } catch {
            // Cancelled: the view went away, so the flow ends here.
        }
    }
}
```

### `LiftSceneParts.swift`

```swift
import SwiftUI

/// Flat pill button. `pressed` is driven by the demo, with a soft touch dot so the tap is visible in the clip.
/// The title rolls to the new text when it changes; `done` puts a small check in front of it.
/// Switched off, it is pale grey with grey text. With `success` it turns fresh green with white
/// text and a small pop:
/// the answer to an action that worked.
/// With Reduce Motion nothing scales: only the colour and the text change.
struct LiftScenePillButton: View {
    let title: String
    var enabled = true
    var success = false
    var done = false
    var pressed = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        let peak: CGFloat = success && !reduceMotion ? 1.04 : 1
        let checkIn: AnyTransition = reduceMotion ? .opacity : .scale(scale: 0.5).combined(with: .opacity)
        HStack(spacing: 8) {
            if done {
                Image(systemName: "checkmark")
                    .font(.system(size: 15, weight: .bold))
                    .transition(checkIn)
            }
            Text(title).contentTransition(.numericText())
        }
        .font(.system(size: 18, weight: .semibold))
        .foregroundStyle(success ? LiftSceneInk.successText : (enabled ? Color.white : LiftSceneInk.secondary))
        .frame(maxWidth: .infinity)
        .frame(height: 54)
        .background(Capsule().fill(success ? LiftSceneInk.successFill : (enabled ? LiftSceneInk.black : LiftSceneInk.off)))
        .overlay(LiftSceneTouchDot(on: pressed))
        .scaleEffect(pressed && !reduceMotion ? 0.965 : 1)
        .keyframeAnimator(initialValue: CGFloat(1), trigger: success) { view, pop in
            view.scaleEffect(pop)
        } keyframes: { _ in
            KeyframeTrack {
                SpringKeyframe(peak, duration: 0.14 * LiftSceneMotion.k, spring: .snappy(duration: 0.5 * LiftSceneMotion.k))
                SpringKeyframe(CGFloat(1), duration: 0.45 * LiftSceneMotion.k, spring: .bouncy(duration: 0.5 * LiftSceneMotion.k))
            }
        }
        .animation(LiftSceneMotion.spring(0.6), value: title)
        .animation(LiftSceneMotion.smooth(0.45), value: enabled)
        .animation(LiftSceneMotion.smooth(0.35), value: success)
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(title)
        .accessibilityAddTraits(.isButton)
    }
}

/// Element comes in from a soft blur, a little lower and smaller. Used with small delays for a stagger.
/// With Reduce Motion it only fades in.
struct LiftSceneReveal: ViewModifier {
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
            .animation(LiftSceneMotion.smooth(0.7, delay: delay), value: on)
    }
}

extension View {
    func liftSceneReveal(_ on: Bool, delay: Double = 0, y: CGFloat = 14, blur: CGFloat = 10, scale: CGFloat = 0.98) -> some View {
        modifier(LiftSceneReveal(on: on, delay: delay, y: y, blur: blur, scale: scale))
    }
}

/// Small grey capsule: a status label, not a button. With `dot` a coloured dot sits in front of the text.
struct LiftSceneStatusChip: View {
    let text: String
    var dot: Color? = nil
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        let dotIn: AnyTransition = reduceMotion ? .opacity : .scale(scale: 0.4).combined(with: .opacity)
        HStack(spacing: 8) {
            if let dot {
                Circle().fill(dot).frame(width: 10, height: 10)
                    .transition(dotIn)
            }
            Text(text)
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(LiftSceneInk.secondary)
                .contentTransition(.numericText())
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(LiftSceneInk.chip))
    }
}

/// Soft dot that shows a tap in the clip. It lies over a control and comes in while the demo "presses" it.
/// White for dark controls, a dark `ink` for white ones.
/// With Reduce Motion it only fades.
struct LiftSceneTouchDot: View {
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

/// Small chip with the change against last week: green for a gain, red for a loss.
struct LiftSceneChangeChip: View {
    let text: String
    var gain = true

    var body: some View {
        Text(text)
            .font(.system(size: 16, weight: .semibold))
            .monospacedDigit()
            .foregroundStyle(gain ? LiftSceneInk.gainText : LiftSceneInk.lossText)
            .contentTransition(.numericText())
            .lineLimit(1)
            .padding(.horizontal, 10)
            .frame(height: 26)
            .background(RoundedRectangle(cornerRadius: 9, style: .continuous)
                .fill(gain ? LiftSceneInk.gainFill : LiftSceneInk.lossFill))
    }
}

/// The hero: the barbell in its grey panel. Everything in it is computed from ONE list, the plate pairs
/// from the inside out: where each plate sits, where the clamps sit, and the labels underneath.
/// A new pair slides on from outside the ends of the bar, a removed pair slides off the same way.
/// With `jump` the whole barbell makes one small jump.
/// With Reduce Motion the plates only fade and nothing jumps.
struct LiftSceneBarbell: View {
    let pairs: [LiftSceneLoad.Pair]
    var jump = false                 // the set was logged
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    private static let height: CGFloat = 164
    private static let barY: CGFloat = 72
    private static let labelY: CGFloat = 143
    private static let firstEdge: CGFloat = 76       // inner edge of the first plate, from the middle
    private static let gap: CGFloat = 2.5            // between two plates, and before the clamp
    private static let clamp = CGSize(width: 9.5, height: 28)

    /// Distance from the middle of the bar to the inner edge of place `index` on one side.
    /// Place `pairs.count` is where the clamp sits.
    private func inner(_ index: Int) -> CGFloat {
        pairs.prefix(index).reduce(Self.firstEdge) { $0 + LiftSceneLoad.plate($1.weight).width + Self.gap }
    }

    var body: some View {
        let side: [Double] = pairs.map { $0.weight }
        let end: CGFloat = inner(pairs.count)                             // inner edge of the clamp
        let clampAt: CGFloat = end + Self.clamp.width / 2
        let platesAt: CGFloat = (Self.firstEdge + end - Self.gap) / 2     // middle of the plates of one side
        let lift: CGFloat = jump && !reduceMotion ? -9 : 0

        GeometryReader { g in
            let half: CGFloat = g.size.width / 2
            ZStack {
                // The bar, the clamps and the plates. They jump together.
                ZStack {
                    Capsule().fill(LiftSceneInk.bar)
                        .frame(width: max(g.size.width - 22, 0), height: 9)
                        .position(x: half, y: Self.barY)
                    clamp.position(x: half - clampAt, y: Self.barY)
                    clamp.position(x: half + clampAt, y: Self.barY)
                    plates(-1, half: half)
                    plates(1, half: half)
                }
                .keyframeAnimator(initialValue: CGFloat(0), trigger: jump) { view, y in
                    view.offset(y: y)
                } keyframes: { _ in
                    KeyframeTrack {
                        SpringKeyframe(lift, duration: 0.16 * LiftSceneMotion.k, spring: .snappy(duration: 0.3 * LiftSceneMotion.k))
                        SpringKeyframe(CGFloat(0), duration: 0.38 * LiftSceneMotion.k, spring: .bouncy(duration: 0.4 * LiftSceneMotion.k))
                    }
                }

                // The labels: the left one reads from the outside in, the right one from the inside out.
                label(LiftSceneLoad.leftLabel(side), x: half - platesAt)
                label(LiftSceneLoad.barLabel, x: half)
                label(LiftSceneLoad.rightLabel(side), x: half + platesAt)
            }
        }
        .frame(height: Self.height)
        .background(LiftSceneInk.panel)
        .clipShape(RoundedRectangle(cornerRadius: 20, style: .continuous))
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Barbell")
        .accessibilityValue(LiftSceneLoad.spoken(side))
    }

    private var clamp: some View {
        RoundedRectangle(cornerRadius: 3, style: .continuous)
            .fill(LiftSceneInk.clamp)
            .frame(width: Self.clamp.width, height: Self.clamp.height)
    }

    /// The plates of one side: `sign` is -1 for the left side and 1 for the right side.
    /// A plate comes from just outside the panel on its own side and leaves to there.
    private func plates(_ sign: CGFloat, half: CGFloat) -> some View {
        ForEach(Array(pairs.enumerated()), id: \.element.id) { index, pair in
            let width: CGFloat = LiftSceneLoad.plate(pair.weight).width
            let centre: CGFloat = inner(index) + width / 2
            let slide: AnyTransition = reduceMotion
                ? .opacity
                : .offset(x: sign * (half + width - centre)).combined(with: .opacity)
            LiftScenePlate(weight: pair.weight)
                .position(x: half + sign * centre, y: Self.barY)
                .zIndex(1)   // a leaving plate stays in front of the clamp
                .transition(slide)
        }
    }

    private func label(_ text: String, x: CGFloat) -> some View {
        Text(text)
            .font(.system(size: 15, weight: .medium))
            .monospacedDigit()
            .foregroundStyle(LiftSceneInk.secondary)
            .contentTransition(.numericText())
            .lineLimit(1)
            .fixedSize()
            .position(x: x, y: Self.labelY)
    }
}

/// A plate on the bar, seen from the side: an upright rounded rectangle.
/// Its size and its colour follow its weight.
struct LiftScenePlate: View {
    let weight: Double

    var body: some View {
        let plate = LiftSceneLoad.plate(weight)
        let shape = RoundedRectangle(cornerRadius: 5, style: .continuous)
        shape.fill(plate.ink)
            .overlay {
                if plate.outlined {
                    shape.strokeBorder(LiftSceneInk.plateOutline, lineWidth: 1.5)
                }
            }
            .frame(width: plate.width, height: plate.height)
            .accessibilityHidden(true)
    }
}

/// The rack: the six plates seen from the front, in six equal columns, each with its weight underneath.
/// `pressed` is the weight (as text) of the disc the demo "taps". That disc gets a soft grey ring
/// BEHIND it and shrinks a little, so its colour stays bright.
/// With Reduce Motion nothing scales: only the ring fades in.
struct LiftSceneRack: View {
    var pressed: String? = nil
    var shown = true                 // the columns have come in
    var lead: Double = 0             // delay of the first column
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    private static let rowHeight: CGFloat = 52

    var body: some View {
        let plates = LiftSceneLoad.plates
        HStack(alignment: .top, spacing: 0) {
            ForEach(plates.indices, id: \.self) { i in
                let name: String = LiftSceneLoad.text(plates[i].weight)
                cell(plates[i], name: name, pressed: pressed == name)
                    .frame(maxWidth: .infinity)
                    .liftSceneReveal(shown, delay: lead + 0.04 * Double(i))
            }
        }
    }

    private func cell(_ plate: LiftSceneLoad.Plate, name: String, pressed: Bool) -> some View {
        VStack(spacing: 8) {
            disc(plate, pressed: pressed)
            Text(name)
                .font(.system(size: 19, weight: .semibold))
                .monospacedDigit()
                .foregroundStyle(LiftSceneInk.primary)
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Add a pair of \(name) kg plates")
        .accessibilityAddTraits(.isButton)
    }

    /// A flat disc with a small white hole in its middle. All discs stand on one centre line.
    private func disc(_ plate: LiftSceneLoad.Plate, pressed: Bool) -> some View {
        let hole: CGFloat = plate.size * 0.26
        return ZStack {
            Circle().fill(LiftSceneInk.press)
                .frame(width: plate.size + 12, height: plate.size + 12)
                .scaleEffect(pressed || reduceMotion ? 1 : 0.8)
                .opacity(pressed ? 1 : 0)
            ZStack {
                Circle().fill(plate.disc)
                Circle().fill(Color.white)
                    .frame(width: hole, height: hole)
                if plate.outlined {
                    Circle().strokeBorder(LiftSceneInk.plateOutline, lineWidth: 1.5)
                    Circle().strokeBorder(LiftSceneInk.plateOutline, lineWidth: 1.5)
                        .frame(width: hole + 3, height: hole + 3)
                }
            }
            .frame(width: plate.size, height: plate.size)
            .scaleEffect(pressed && !reduceMotion ? 0.92 : 1)
        }
        .frame(height: Self.rowHeight)
    }
}
```

### `LiftSceneView.swift`

```swift
import SwiftUI

/// The lift screen. There is only this one screen; the list of plates on the bar changes its state:
/// the big number, the change chip and the labels under the barbell all follow that list.
/// Font sizes and heights are fixed numbers on purpose: this is a fixed showcase for a recording,
/// not a screen that follows Dynamic Type.
struct LiftSceneView: View {
    let demo: LiftSceneDemo
    @State private var shown = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text(LiftSceneLoad.lift)
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(LiftSceneInk.secondary)
                Spacer()
                LiftSceneStatusChip(text: demo.chip, dot: demo.chipDot)
                    .animation(LiftSceneMotion.spring(0.5), value: demo.chip)
            }
            .padding(.leading, 4)
            .frame(height: 32)
            .liftSceneReveal(shown, delay: lead)

            // The line box of such a large font is much taller than its digits.
            // The frame keeps only the height of the digits, so the next line can sit close.
            HStack(alignment: .firstTextBaseline, spacing: 6) {
                Text("\(demo.total)")
                    .font(.system(size: 104, weight: .bold))
                    .tracking(-3)
                    .foregroundStyle(LiftSceneInk.primary)
                    .contentTransition(.numericText(value: Double(demo.total)))
                Text("kg")
                    .font(.system(size: 34, weight: .semibold))
                    .foregroundStyle(LiftSceneInk.secondary)
            }
            .fixedSize()
            .frame(height: 78)
            .padding(.top, 18)
            .animation(LiftSceneMotion.spring(0.4), value: demo.total)
            .accessibilityElement(children: .ignore)
            .accessibilityLabel("\(demo.total) kilograms")
            .liftSceneReveal(shown, delay: lead + 0.06)

            // The change against last week. It comes in when the bar is loaded.
            HStack(spacing: 8) {
                LiftSceneChangeChip(text: demo.change, gain: demo.gain)
                Text(demo.words)
                    .font(.system(size: 18))
                    .foregroundStyle(LiftSceneInk.secondary)
                    .contentTransition(.numericText())
                    .lineLimit(1)
            }
            .padding(.leading, 4)
            .padding(.top, 8)
            .animation(LiftSceneMotion.spring(0.5), value: demo.total)
            .accessibilityElement(children: .combine)
            .accessibilityHidden(!demo.loaded)
            .liftSceneReveal(shown && demo.loaded)

            LiftSceneBarbell(pairs: demo.pairs, jump: demo.logged)
                .padding(.top, 16)
                .liftSceneReveal(shown, delay: lead + 0.12)

            HStack {
                Text("Add a pair")
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(LiftSceneInk.secondary)
                Spacer()
                removeLast
            }
            .padding(.leading, 4)
            .frame(height: 30)
            .padding(.top, 16)
            .liftSceneReveal(shown, delay: lead + 0.18)

            LiftSceneRack(pressed: demo.pressed, shown: shown, lead: lead + 0.24)
                .padding(.top, 16)
        }
        .onAppear { shown = true }
    }

    /// The white pill "Remove last". The demo "taps" it with a dark touch dot.
    private var removeLast: some View {
        let pressed = demo.pressed == "remove"
        return Text("Remove last")
            .font(.system(size: 16, weight: .semibold))
            .foregroundStyle(LiftSceneInk.primary)
            .padding(.horizontal, 16)
            .frame(height: 30)
            .background(Capsule().fill(Color.white))
            .overlay(Capsule().strokeBorder(LiftSceneInk.outline, lineWidth: 1.5))
            .overlay(LiftSceneTouchDot(on: pressed, ink: LiftSceneInk.pressDot, size: 38))
            .scaleEffect(pressed && !reduceMotion ? 0.965 : 1)
            .accessibilityElement(children: .ignore)
            .accessibilityLabel("Remove last pair")
            .accessibilityAddTraits(.isButton)
    }
}
```

### `LiftSceneRootView.swift`

```swift
import SwiftUI

/// The whole scene: the lime strip, the white sheet with the lift screen, and the pill button.
/// Show this view to see the scene. It starts the flow by itself and stops it when it goes away.
struct LiftSceneRootView: View {
    @State private var demo = LiftSceneDemo()
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    /// Height of the white sheet. The clip later shows only this lower part of the screen.
    private let sheetHeight: CGFloat = 640

    var body: some View {
        ZStack(alignment: .bottom) {
            // The strip behind the sheet. It does not change.
            LinearGradient(colors: [LiftSceneInk.stripLeft, LiftSceneInk.stripRight],
                           startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()
                .accessibilityHidden(true)

            VStack(spacing: 0) {
                LiftSceneView(demo: demo)
                    .id(demo.round)
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // Black = an action to take. After the set was logged it turns green: it worked.
                LiftScenePillButton(title: demo.button, success: demo.logged, done: demo.logged,
                                    pressed: demo.pressed == "main")
                    .liftSceneReveal(demo.sheetUp, delay: demo.sheetUp ? 0.5 : 0)
            }
            .padding(.horizontal, 18)
            .padding(.top, 32)
            .padding(.bottom, 40)
            .frame(maxWidth: .infinity)
            .frame(height: sheetHeight)
            .background(
                UnevenRoundedRectangle(topLeadingRadius: 40, topTrailingRadius: 40, style: .continuous)
                    .fill(Color.white)
                    .padding(.bottom, -200)   // white also below the screen edge while the sheet springs
            )
            // With Reduce Motion the sheet stays in place and fades instead of sliding.
            .offset(y: demo.sheetUp || reduceMotion ? 0 : sheetHeight + 60)
            .opacity(demo.sheetUp || !reduceMotion ? 1 : 0)
            .animation(reduceMotion ? LiftSceneMotion.smooth(0.35) : LiftSceneMotion.spring(0.85, bounce: 0.08),
                       value: demo.sheetUp)
        }
        .ignoresSafeArea(edges: .bottom)
        .environment(\.colorScheme, .light)   // the scene is drawn for light mode only
        .persistentSystemOverlays(.hidden)   // no home indicator line in the recording
        .task { await demo.run() }
    }
}
```

### `LiftSceneApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
// This file is nothing but the app's entry point. A project that already has one leaves this file out
// and shows `LiftSceneRootView()` itself.
@main
struct LiftSceneApp: App {
    var body: some Scene {
        WindowGroup {
            LiftSceneRootView()
                .preferredColorScheme(.light)
                .statusBarHidden(true)
        }
    }
}
```
