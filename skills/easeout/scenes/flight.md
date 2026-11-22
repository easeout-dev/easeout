# Easeout scene: Flight status (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

A flight screen inside a white sheet over a blue gradient. It plays in five beats:

1. **Status.** The sheet slides up from the bottom and the content comes in piece by piece, top to bottom: airline row, two large airport codes with a dashed route line, a grey card with gate, seat and boarding time, a row "Boards in" and a row of 44 tick marks as the progress bar. The plane flies to the middle of the line, the minutes roll from 60 down to 42, and 13 ticks light up green one after the other, each with a small pop.
2. **Boarding pass.** A tap on the black pill button: the status content melts into a blur, the ticket rises as one card, and its barcode prints itself bar by bar. The button title rolls to "Add to Wallet".
3. **Into the wallet.** A second tap: the ticket drops down behind the button and the title rolls to "Added" with a small check.
4. **In the air.** The status screen comes back. The plane flies from the start to 62 % of the line, the line behind it turns solid, altitude and speed climb, 26 ticks light up.
5. **Landed.** The plane reaches the destination, the chip says "Landed", the card swaps its three values column by column, every tick is lit. Then the sheet slides away and it starts again.

`Demo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages.
- Create the five files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, durations, curves and delays. These numbers are the design.
- Screens change in one way only: the old content leaves with the `blurOut` transition and the new content comes in with staggered `.reveal`. Do not replace this with slides, plain opacity fades or `matchedGeometryEffect`.
- Keep `.contentTransition(.numericText())` on every text that changes.
- Keep the order of the views in `RootView`: the button is drawn after the screens, which is why the ticket can drop behind it. Keep `.id(demo.round)` on `StatusView`; it makes the screen come in fresh each time.
- Keep all timing inside `Motion`. Do not write durations in other places.
- If my project already has an `@main` app, do not add a second one: leave out `FlightApp` and show `RootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- If a type name collides with one in my project (`Ink`, `Motion`, `Demo`, `Phase`, `Stage`, `Cell`), rename ours with the prefix `Flight` and tell me.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## Logos and images

The scene needs no image files. The airline badge is a neutral placeholder drawn in code.

- Do not add a real airline logo on your own. Brand logos are trademarks.
- If I ask for a real logo, or my data names a real airline, ask me first: "Do you want the real logo of this airline in the badge?"
- If I say yes: find the official vector logo yourself (the brand's press page first, an open icon set such as Simple Icons second), add it to the asset catalog as an SVG with "Preserve Vector Data", and show it inside `AirlineBadge` at 73 % of the badge size. Tell me the source you used.
- If I say no, or you cannot find an official vector: keep the placeholder.
- Never generate a logo with an image model and never redraw one from memory.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to rename.
3. Then ask me one question: "Should I connect this to your real data?" If yes, remove the auto-play (`demo.run()` and the timeline in `Demo`) and drive `stage`, `plane`, `ticks`, `minutes`, `button` and `phase` from my model. Keep every animation as it is.

## What I may ask you to change

- The example flight: all texts live in `Flight` and `Stage` in `Model.swift`.
- The colours: the gradient is `Ink.skyLight` to `Ink.sky`, the lit ticks come from `Ink.tick(_:)`.
- The number of ticks: `Flight.tickCount`.

## Check yourself before you answer

- The status screen shows 13 green ticks of 44, later 26, then all 44.
- Between two screens the sheet is almost empty for a moment: old content gone, new content not yet there. Nothing slides sideways and nothing morphs.
- The button never moves. Only its title changes.
- The ticket disappears behind the button, not in front of it.
- Light mode only; the scene sets `.preferredColorScheme(.light)`.

## The code

### `Model.swift`

```swift
import SwiftUI
import Observation

// Colours taken from the approved design pictures (flat look, no glass).
enum Ink {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)
    static let secondary = Color(red: 0.459, green: 0.463, blue: 0.498)
    static let hairline = Color(red: 0.875, green: 0.875, blue: 0.894)
    static let dash = Color(red: 0.698, green: 0.702, blue: 0.729)
    static let fill = Color(red: 0.957, green: 0.957, blue: 0.961)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let tickOff = Color(red: 0.855, green: 0.855, blue: 0.863)
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let skyLight = Color(red: 0.557, green: 0.773, blue: 1.0)
    static let sky = Color(red: 0.231, green: 0.510, blue: 0.965)
    static let green = Color(red: 0.016, green: 0.804, blue: 0.373)
    static let blue = Color(red: 0.118, green: 0.522, blue: 0.980)

    /// Colour of a lit tick: light green on the left, the green of the status dot on the right.
    static func tick(_ t: Double) -> Color {
        Color(red: 0.298 + (0.020 - 0.298) * t, green: 0.851 + (0.706 - 0.851) * t, blue: 0.529 + (0.329 - 0.529) * t)
    }
}

/// All timing goes through here. Launch with `-slow 4` to run everything four times slower:
/// the recording is then blended back to normal speed, which gives real motion blur.
/// `-warmup 16` waits first, so the home indicator has faded before the flow starts.
enum Motion {
    static let k: Double = {
        let v = UserDefaults.standard.double(forKey: "slow")
        return v > 0 ? v : 1
    }()
    static let warmup = UserDefaults.standard.double(forKey: "warmup")

    /// Soft ease-out, the main curve.
    static func smooth(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.2, 0.9, 0.2, 1, duration: d * k).delay(delay * k)
    }
    /// Ease-in-out for things that travel across the screen, like the plane.
    static func travel(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.6, 0, 0.15, 1, duration: d * k).delay(delay * k)
    }
    static func spring(_ d: Double, bounce: Double = 0, delay: Double = 0) -> Animation {
        .spring(duration: d * k, bounce: bounce).delay(delay * k)
    }
    static func wait(_ seconds: Double) async {
        try? await Task.sleep(for: .seconds(seconds * k))
    }
}

struct Cell: Equatable {
    let label: String
    let value: String
}

/// The status screen has three states. Only the data changes, the layout stays the same.
enum Stage: Equatable {
    case boarding, air, landed

    var chip: String { ["On time", "In the air", "Landed"][index] }
    var chipColor: Color { self == .air ? Ink.blue : Ink.green }
    var duration: String { self == .landed ? "11 h 37 min" : "11 h 45 min" }
    var arrival: String { self == .landed ? "13:12" : "13:20" }
    var rowLabel: String { ["Boards in", "Lands in", "Arrived"][index] }
    var rowValue: String { ["42 min", "4 h 38 min", "8 min early"][index] }
    var ticks: Int { [13, 26, Flight.tickCount][index] }
    /// Where the plane sits on the route line, 0 = Munich, 1 = San Francisco.
    var plane: Double { [0.5, 0.62, 1.0][index] }
    var button: String { ["Boarding pass", "Share arrival", "Get a ride"][index] }
    var cells: [Cell] {
        switch self {
        case .boarding: return [Cell(label: "Gate", value: "K12"), Cell(label: "Seat", value: "24A"),
                                Cell(label: "Boarding", value: "09:55")]
        case .air: return [Cell(label: "Altitude", value: "11.3 km"), Cell(label: "Speed", value: "905 km/h"),
                           Cell(label: "Outside", value: "−56 °C")]
        case .landed: return [Cell(label: "Terminal", value: "G"), Cell(label: "Belt", value: "7"),
                              Cell(label: "Bags", value: "13:40")]
        }
    }
    /// Values the numbers start from before they roll to `cells` (the plane is still climbing).
    var startCells: [Cell] {
        self == .air ? [Cell(label: "Altitude", value: "7.9 km"), Cell(label: "Speed", value: "684 km/h"),
                        Cell(label: "Outside", value: "−31 °C")] : cells
    }
    private var index: Int { self == .boarding ? 0 : (self == .air ? 1 : 2) }
}

// Example data. The times add up: 10:35 + 11 h 45 min is 13:20 in San Francisco (nine hours behind Munich).
enum Flight {
    static let airline = "Lufthansa"
    static let number = "LH 458"
    static let from = "MUC"
    static let to = "SFO"
    static let fromCity = "Munich"
    static let toCity = "San Francisco"
    static let departure = "10:35"
    static let passenger = "Lena Brandt"
    static let travelClass = "LH 458 · Economy"
    static let tickCount = 44
}

enum Phase: Equatable { case status, pass }

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
/// Screens change like this: the old content melts into a blur, the new content comes in piece by piece.
@MainActor @Observable
final class Demo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var phase: Phase = .status
    var round = 0                    // a new number makes the status screen come in fresh
    var stage: Stage = .boarding
    var settled = true               // false while the numbers of a stage still show their start values
    var pressed = false              // the button is being "tapped"
    var added = false                // small check in the button
    var stored = false               // the ticket has dropped into the wallet button
    var button = Stage.boarding.button
    var plane: Double = 0            // position of the plane on the route line
    var routeDrawn: Double = 0       // how much of the dashed route line is drawn
    var ticks = 0                    // lit tick marks
    var ticksFrom = 0                // lit tick marks before the last change, where the wave starts
    var minutes = 60                 // "Boards in" counts down to 42

    var cells: [Cell] { settled ? stage.cells : stage.startCells }

    private func tap() async {
        withAnimation(Motion.spring(0.2)) { pressed = true }
        await Motion.wait(0.17)
        withAnimation(Motion.spring(0.35, bounce: 0.2)) { pressed = false }
        await Motion.wait(0.08)
    }

    private func light(_ count: Int) {
        ticksFrom = ticks
        ticks = count
    }

    func run() async {
        if Motion.warmup > 0 { try? await Task.sleep(for: .seconds(Motion.warmup)) }
        while !Task.isCancelled {
            // Reset while the sheet is down.
            stage = .boarding; settled = true; added = false; stored = false
            button = Stage.boarding.button
            plane = 0; routeDrawn = 0; ticks = 0; ticksFrom = 0; minutes = 60
            phase = .status; round += 1
            await Motion.wait(0.12)

            // 1. The sheet slides up. Status: the plane flies to the middle, the ticks light up, the minutes count down.
            sheetUp = true
            await Motion.wait(0.55)
            routeDrawn = 1
            plane = Stage.boarding.plane
            await Motion.wait(0.4)
            light(Stage.boarding.ticks)
            minutes = 42
            await Motion.wait(1.6)

            // 2. Boarding pass: the ticket comes up as a card and the barcode prints itself.
            await tap()
            button = "Add to Wallet"
            withAnimation(Motion.smooth(0.5)) { phase = .pass }
            await Motion.wait(1.9)
            await tap()
            stored = true                       // the ticket drops behind the button
            added = true
            button = "Added"
            await Motion.wait(0.65)

            // 3. A few hours later: in the air. The plane flies from Munich, the numbers climb.
            stage = .air; settled = false; added = false
            button = Stage.air.button
            plane = 0; routeDrawn = 1; ticks = 0; ticksFrom = 0
            round += 1
            withAnimation(Motion.smooth(0.5)) { phase = .status }
            await Motion.wait(0.5)
            plane = Stage.air.plane
            settled = true
            light(Stage.air.ticks)
            await Motion.wait(2.0)

            // 4. Landed: the plane reaches San Francisco, the card swaps its values.
            stage = .landed
            button = Stage.landed.button
            plane = Stage.landed.plane
            light(Stage.landed.ticks)
            await Motion.wait(2.2)

            // 5. The sheet slides away, then everything starts again.
            sheetUp = false
            await Motion.wait(0.5)
        }
    }
}
```

### `Parts.swift`

```swift
import SwiftUI

/// Flat pill button. `pressed` is driven by the demo, with a soft touch dot so the tap is visible in the clip.
/// The title rolls to the new text when it changes; `done` puts a small check in front of it.
struct PillButton: View {
    let title: String
    var done = false
    var pressed = false

    var body: some View {
        HStack(spacing: 8) {
            if done {
                Image(systemName: "checkmark")
                    .font(.system(size: 15, weight: .bold))
                    .transition(.scale(scale: 0.5).combined(with: .opacity))
            }
            Text(title).contentTransition(.numericText())
        }
        .font(.system(size: 18, weight: .semibold))
        .foregroundStyle(Color.white)
        .frame(maxWidth: .infinity)
        .frame(height: 54)
        .background(Capsule().fill(Ink.black))
        .overlay(
            Circle()
                .fill(Color.white.opacity(0.26))
                .frame(width: 44, height: 44)
                .scaleEffect(pressed ? 1 : 1.5)
                .opacity(pressed ? 1 : 0)
        )
        .scaleEffect(pressed ? 0.965 : 1)
        .animation(Motion.spring(0.6), value: title)
    }
}

/// Element comes in from a soft blur, a little lower and smaller. Used with small delays for a stagger.
struct Reveal: ViewModifier {
    let on: Bool
    var delay: Double = 0
    var y: CGFloat = 14
    var blur: CGFloat = 10
    var scale: CGFloat = 0.98

    func body(content: Content) -> some View {
        content
            .opacity(on ? 1 : 0)
            .blur(radius: on ? 0 : blur)
            .scaleEffect(on ? 1 : scale)
            .offset(y: on ? 0 : y)
            .animation(Motion.smooth(0.7, delay: delay), value: on)
    }
}

extension View {
    func reveal(_ on: Bool, delay: Double = 0, y: CGFloat = 14, blur: CGFloat = 10, scale: CGFloat = 0.98) -> some View {
        modifier(Reveal(on: on, delay: delay, y: y, blur: blur, scale: scale))
    }
}

/// A screen leaves by melting into a blur. New screens bring their own staggered reveal.
struct BlurOut: ViewModifier {
    let active: Bool
    func body(content: Content) -> some View {
        content
            .opacity(active ? 0 : 1)
            .blur(radius: active ? 16 : 0)
            .scaleEffect(active ? 0.97 : 1)
            .offset(y: active ? -8 : 0)
    }
}

extension AnyTransition {
    static var blurOut: AnyTransition {
        .asymmetric(insertion: .identity,
                    removal: .modifier(active: BlurOut(active: true), identity: BlurOut(active: false)))
    }
}

/// Small grey capsule with a coloured dot. Text and dot colour change softly.
struct StatusChip: View {
    let text: String
    let color: Color

    var body: some View {
        HStack(spacing: 8) {
            Circle().fill(color).frame(width: 10, height: 10)
            Text(text)
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(Ink.secondary)
                .contentTransition(.numericText())
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(Ink.chip))
    }
}

/// Route between two airports: a dashed line with a dot at each end and a plane on it.
/// `progress` moves the plane (0 = start, 1 = end). With `trail` the part behind the plane turns solid.
struct RouteLine: View {
    var progress: Double
    var trail = false
    var drawn: Double = 1          // 0...1, how much of the dashed line is drawn
    var gap: Color = .white        // colour behind the plane, so the line pauses around it

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
                .stroke(Ink.dash, style: StrokeStyle(lineWidth: 1.3, lineCap: .round, dash: [3.5, 4.5]))

                Capsule().fill(Ink.primary)
                    .frame(width: max(x, 1), height: 1.8)
                    .opacity(trail ? 1 : 0)

                Circle().fill(trail ? Ink.primary : Ink.dash)
                    .frame(width: 6, height: 6)
                    .opacity(drawn > 0 ? 1 : 0)
                Circle().fill(Ink.dash)
                    .frame(width: 6, height: 6)
                    .position(x: w - 3, y: mid)
                    .opacity(drawn >= 1 && !arrived ? 1 : 0)

                Image(systemName: "airplane")
                    .font(.system(size: 22, weight: .bold))
                    .foregroundStyle(Ink.primary)
                    .padding(.horizontal, 5)
                    .background(gap)
                    .position(x: x, y: mid)
                    .opacity(drawn > 0 ? 1 : 0)
            }
        }
        .frame(height: 30)
        .animation(Motion.travel(1.5), value: progress)
        .animation(Motion.smooth(1.0), value: drawn)
        .animation(Motion.smooth(0.6), value: trail)
    }
}

/// Row of tick marks as a progress bar. Lit ticks take a colour from light to full green.
/// When more ticks light up they do it one after the other, each with a small pop, like a wave.
struct TickRow: View {
    let lit: Int
    var from = 0                   // lit ticks before the last change, where the wave starts

    var body: some View {
        HStack(spacing: 0) {
            ForEach(0..<Flight.tickCount, id: \.self) { i in
                let on = i < lit
                let delay = Double(max(0, i - from)) * 0.026
                Capsule()
                    .fill(on ? Ink.tick(Double(i) / Double(Flight.tickCount - 1)) : Ink.tickOff)
                    .frame(width: 3, height: 11)
                    .keyframeAnimator(initialValue: CGFloat(1), trigger: on) { view, stretch in
                        view.scaleEffect(y: stretch)
                    } keyframes: { _ in
                        KeyframeTrack {
                            LinearKeyframe(CGFloat(1), duration: max(delay, 0.01) * Motion.k)
                            SpringKeyframe(CGFloat(1.9), duration: 0.13 * Motion.k, spring: .snappy)
                            SpringKeyframe(CGFloat(1), duration: 0.42 * Motion.k, spring: .bouncy)
                        }
                    }
                    .animation(Motion.smooth(0.3, delay: delay), value: on)
                if i < Flight.tickCount - 1 { Spacer(minLength: 0) }
            }
        }
        .frame(height: 11)
    }
}

/// One-dimensional barcode. With `shown` the bars grow from the bottom one after the other, like being printed.
struct Barcode: View {
    var shown = true

    /// Bar width and the gap after it, in units. Fixed seed, so the code looks the same every time.
    static let bars: [(w: CGFloat, gap: CGFloat)] = {
        var seed: UInt32 = 7
        func next() -> Int {
            seed = seed &* 1664525 &+ 1013904223
            return Int(seed >> 16)
        }
        let widths: [CGFloat] = [1, 1, 2, 3, 1, 2, 4]
        let gaps: [CGFloat] = [1, 1, 2, 1, 3]
        return (0..<66).map { _ in (w: widths[next() % widths.count], gap: gaps[next() % gaps.count]) }
    }()

    var body: some View {
        GeometryReader { g in
            let bars = Self.bars
            let total = bars.reduce(0) { $0 + $1.w + $1.gap } - bars[bars.count - 1].gap
            let u = g.size.width / total
            HStack(spacing: 0) {
                ForEach(0..<bars.count, id: \.self) { i in
                    Rectangle().fill(Ink.primary)
                        .frame(width: bars[i].w * u)
                        .scaleEffect(y: shown ? 1 : 0.15, anchor: .bottom)
                        .opacity(shown ? 1 : 0)
                        .animation(Motion.spring(0.5, bounce: 0.3, delay: 0.55 + Double(i) * 0.009), value: shown)
                        .padding(.trailing, i == bars.count - 1 ? 0 : bars[i].gap * u)
                }
            }
        }
    }
}

/// Rounded card with a half-round notch on the left and right edge, like a paper ticket.
struct TicketShape: Shape {
    var notchFromBottom: CGFloat
    var radius: CGFloat = 26
    var notch: CGFloat = 10

    func path(in rect: CGRect) -> Path {
        var p = Path(roundedRect: rect, cornerRadius: radius, style: .continuous)
        let y = rect.maxY - notchFromBottom
        guard y > rect.minY + radius else { return p }
        for x in [rect.minX, rect.maxX] {
            p = p.subtracting(Path(ellipseIn: CGRect(x: x - notch, y: y - notch, width: notch * 2, height: notch * 2)))
        }
        return p
    }
}

/// Three label and value pairs side by side with thin dividers.
/// When a label changes, the old pair leaves upward in a blur and the new one comes up from below,
/// one column after the other. When only the value changes, its digits roll.
struct CellsRow: View {
    let cells: [Cell]

    var body: some View {
        HStack(spacing: 0) {
            ForEach(0..<cells.count, id: \.self) { i in
                ZStack {
                    VStack(spacing: 5) {
                        Text(cells[i].label)
                            .font(.system(size: 15))
                            .foregroundStyle(Ink.secondary)
                        Text(cells[i].value)
                            .font(.system(size: 24, weight: .bold))
                            .foregroundStyle(Ink.primary)
                            .monospacedDigit()
                            .lineLimit(1)
                            .minimumScaleFactor(0.8)
                            .contentTransition(.numericText())
                    }
                    .id(cells[i].label)
                    .transition(.asymmetric(
                        insertion: .modifier(active: Swap(y: 18), identity: Swap(y: 0)),
                        removal: .modifier(active: Swap(y: -18), identity: Swap(y: 0))))
                }
                .frame(maxWidth: .infinity)
                .animation(Motion.smooth(0.6, delay: Double(i) * 0.09), value: cells[i].label)
                .animation(Motion.spring(1.1), value: cells[i].value)
                if i < cells.count - 1 {
                    Rectangle().fill(Ink.hairline).frame(width: 1, height: 56)
                }
            }
        }
    }
}

/// In and out state for content that is swapped in place: moved by `y`, blurred and invisible.
struct Swap: ViewModifier {
    let y: CGFloat
    func body(content: Content) -> some View {
        content
            .opacity(y == 0 ? 1 : 0)
            .blur(radius: y == 0 ? 0 : 8)
            .offset(y: y)
    }
}

/// A card drops down behind whatever sits below it (here: the ticket goes into the wallet button).
/// It moves at once and fades only at the end, so you see where it goes.
struct Drop: ViewModifier {
    let active: Bool
    func body(content: Content) -> some View {
        content
            .opacity(active ? 0 : 1)
            .animation(Motion.smooth(0.25, delay: 0.3), value: active)
            .scaleEffect(active ? 0.8 : 1, anchor: .bottom)
            .rotationEffect(.degrees(active ? 2.5 : 0))
            .offset(y: active ? 96 : 0)
            .blur(radius: active ? 6 : 0)
            .animation(Motion.smooth(0.6), value: active)
    }
}

/// Airline badge: white circle with a hairline, inside it a soft disc with the two-letter airline code.
/// This is a neutral placeholder. A real brand logo is the user's choice, see "Logos and images" in the prompt.
struct AirlineBadge: View {
    var code = "LH"

    var body: some View {
        GeometryReader { g in
            let s = min(g.size.width, g.size.height)
            ZStack {
                Circle().fill(Color.white)
                Circle().strokeBorder(Ink.hairline, lineWidth: 1)
                Circle().fill(Ink.fill)
                    .frame(width: s * 0.73, height: s * 0.73)
                Text(code)
                    .font(.system(size: s * 0.27, weight: .bold))
                    .foregroundStyle(Ink.primary)
            }
            .frame(width: g.size.width, height: g.size.height)
        }
    }
}
```

### `StatusView.swift`

```swift
import SwiftUI

/// The flight status screen. One layout for three states (boarding, in the air, landed).
/// It comes in piece by piece from top to bottom. When `demo.stage` changes, the plane flies on,
/// the card swaps its values, the texts roll and more ticks light up.
struct StatusView: View {
    let demo: Demo
    @State private var shown = false

    private let code = Font.system(size: 50, weight: .heavy)
    /// The old screen is still melting away, so the new pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        let stage = demo.stage

        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Today")
                    .font(.system(size: 17, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                StatusChip(text: stage.chip, color: stage.chipColor)
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            HStack(spacing: 13) {
                AirlineBadge()
                    .frame(width: 66, height: 66)
                VStack(alignment: .leading, spacing: 1) {
                    Text(Flight.airline)
                        .font(.system(size: 23, weight: .bold))
                        .foregroundStyle(Ink.primary)
                    Text(Flight.number)
                        .font(.system(size: 18, weight: .medium))
                        .foregroundStyle(Ink.secondary)
                }
            }
            .padding(.top, 24)
            .reveal(shown, delay: lead + 0.06)

            HStack(spacing: 9) {
                Text(Flight.from)
                    .font(code).tracking(-1)
                    .foregroundStyle(Ink.primary)
                    .fixedSize()
                RouteLine(progress: demo.plane, trail: stage != .boarding, drawn: demo.routeDrawn)
                    .overlay(alignment: .top) {
                        Text(stage.duration)
                            .font(.system(size: 14.5))
                            .foregroundStyle(Ink.secondary)
                            .contentTransition(.numericText())
                            .fixedSize()
                            .offset(y: -17)
                    }
                Text(Flight.to)
                    .font(code).tracking(-1)
                    .foregroundStyle(Ink.primary)
                    .fixedSize()
            }
            .padding(.top, 44)
            .padding(.horizontal, 3)
            .reveal(shown, delay: lead + 0.12)

            HStack(alignment: .top) {
                place(Flight.fromCity, Flight.departure, .leading)
                Spacer()
                place(Flight.toCity, stage.arrival, .trailing)
            }
            .padding(.horizontal, 4)
            .padding(.top, 2)
            .reveal(shown, delay: lead + 0.18)

            CellsRow(cells: demo.cells)
                .frame(height: 90)
                .background(RoundedRectangle(cornerRadius: 24, style: .continuous).fill(Ink.fill))
                .padding(.top, 42)
                .reveal(shown, delay: lead + 0.24)

            HStack {
                Text(stage.rowLabel)
                    .font(.system(size: 18))
                    .foregroundStyle(Ink.secondary)
                    .contentTransition(.numericText())
                Spacer()
                Text(stage == .boarding ? "\(demo.minutes) min" : stage.rowValue)
                    .font(.system(size: 18, weight: .semibold))
                    .foregroundStyle(Ink.primary)
                    .monospacedDigit()
                    .contentTransition(.numericText(value: Double(-demo.minutes)))
                    .animation(Motion.spring(1.1), value: demo.minutes)
            }
            .padding(.horizontal, 4)
            .padding(.top, 36)
            .reveal(shown, delay: lead + 0.3)

            TickRow(lit: demo.ticks, from: demo.ticksFrom)
                .padding(.horizontal, 4)
                .padding(.top, 15)
                .reveal(shown, delay: lead + 0.34)
        }
        .animation(Motion.spring(0.8), value: stage)
        .onAppear { shown = true }
    }

    private func place(_ city: String, _ time: String, _ side: HorizontalAlignment) -> some View {
        VStack(alignment: side, spacing: 3) {
            Text(city)
                .font(.system(size: 18))
                .foregroundStyle(Ink.secondary)
            Text(time)
                .font(.system(size: 21, weight: .semibold))
                .foregroundStyle(Ink.primary)
                .monospacedDigit()
                .contentTransition(.numericText())
        }
    }
}
```

### `PassView.swift`

```swift
import SwiftUI

/// The boarding pass. The ticket comes up from below as one card, its content follows piece by piece,
/// and the barcode prints itself bar by bar. With `demo.stored` the ticket drops behind the button.
struct PassView: View {
    let demo: Demo
    @State private var shown = false

    /// Height of the part below the perforation. The notches sit right above it.
    private let lower: CGFloat = 148
    private let code = Font.system(size: 40, weight: .heavy)
    /// The old screen is still melting away, so the new pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Boarding pass")
                    .font(.system(size: 17, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                Image(systemName: "xmark")
                    .font(.system(size: 12, weight: .bold))
                    .foregroundStyle(Ink.primary)
                    .frame(width: 32, height: 32)
                    .background(Circle().fill(Ink.chip))
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            VStack(spacing: 0) {
                VStack(alignment: .leading, spacing: 0) {
                    HStack(spacing: 12) {
                        AirlineBadge()
                            .frame(width: 54, height: 54)
                        VStack(alignment: .leading, spacing: 1) {
                            Text(Flight.passenger)
                                .font(.system(size: 21, weight: .semibold))
                                .foregroundStyle(Ink.primary)
                            Text(Flight.travelClass)
                                .font(.system(size: 16))
                                .foregroundStyle(Ink.secondary)
                        }
                    }
                    .reveal(shown, delay: lead + 0.22, y: 10)

                    HStack(spacing: 9) {
                        Text(Flight.from)
                            .font(code).tracking(-0.8)
                            .foregroundStyle(Ink.primary)
                            .fixedSize()
                        RouteLine(progress: 0.5, gap: Ink.fill)
                        Text(Flight.to)
                            .font(code).tracking(-0.8)
                            .foregroundStyle(Ink.primary)
                            .fixedSize()
                    }
                    .padding(.top, 26)
                    .reveal(shown, delay: lead + 0.28, y: 10)

                    CellsRow(cells: Stage.boarding.cells)
                        .frame(height: 74)
                        .padding(.top, 22)
                        .reveal(shown, delay: lead + 0.34, y: 10)
                }
                .padding(.horizontal, 20)
                .padding(.top, 24)
                .padding(.bottom, 14)

                // Perforation between the two notches.
                GeometryReader { g in
                    Path { p in
                        p.move(to: CGPoint(x: 18, y: g.size.height / 2))
                        p.addLine(to: CGPoint(x: g.size.width - 18, y: g.size.height / 2))
                    }
                    .stroke(Ink.dash.opacity(0.7), style: StrokeStyle(lineWidth: 1.2, dash: [5, 5]))
                }
                .frame(height: 20)

                VStack(spacing: 11) {
                    Barcode(shown: shown)
                        .frame(height: 78)
                    Text("Scan at the gate")
                        .font(.system(size: 15))
                        .foregroundStyle(Ink.secondary)
                        .reveal(shown, delay: lead + 0.9, y: 6)
                }
                .padding(.horizontal, 22)
                .frame(height: lower)
            }
            .background(TicketShape(notchFromBottom: lower + 10).fill(Ink.fill))
            // The ticket itself: a card that rises with a soft spring.
            .reveal(shown, delay: lead + 0.08, y: 90, blur: 14, scale: 0.94)
            .modifier(Drop(active: demo.stored))
            .padding(.top, 22)
        }
        .onAppear { shown = true }
    }
}
```

### `FlightApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
@main
struct FlightApp: App {
    var body: some Scene {
        WindowGroup {
            RootView()
                .preferredColorScheme(.light)
                .statusBarHidden(true)
        }
    }
}

struct RootView: View {
    @State private var demo = Demo()

    /// Height of the white sheet. The clip later shows only this lower part of the screen.
    private let sheetHeight: CGFloat = 640

    var body: some View {
        ZStack(alignment: .bottom) {
            LinearGradient(colors: [Ink.skyLight, Ink.sky], startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()

            VStack(spacing: 0) {
                ZStack(alignment: .top) {
                    switch demo.phase {
                    case .status:
                        StatusView(demo: demo)
                            .id(demo.round)
                            .transition(AnyTransition.blurOut)
                    case .pass:
                        PassView(demo: demo)
                            .transition(AnyTransition.blurOut)
                    }
                }
                .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // The button stays in place on every screen, only its title changes.
                // It is drawn after the screens, so the ticket can drop behind it.
                PillButton(title: demo.button, done: demo.added, pressed: demo.pressed)
                    .reveal(demo.sheetUp, delay: demo.sheetUp ? 0.5 : 0)
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
            .offset(y: demo.sheetUp ? 0 : sheetHeight + 60)
            .animation(Motion.spring(0.85, bounce: 0.08), value: demo.sheetUp)
        }
        .ignoresSafeArea(edges: .bottom)
        .persistentSystemOverlays(.hidden)   // no home indicator line in the recording
        .task { await demo.run() }
    }
}
```
