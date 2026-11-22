# Easeout scene: Portfolio chart (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

One screen inside a white sheet over a dark grey gradient: the value of an investment portfolio. Its hero is a line chart that changes its shape. It plays in five beats:

1. **Start.** The sheet slides up and the content comes in piece by piece. The chart line draws itself from left to right in green, with a soft green area under it, a dotted line for yesterday's close and a dot at the end.
2. **Finger on the chart.** A finger drags along the line. A thin marker with a dot follows it, a small black pill above the marker shows the time, the part of the chart to the right of the finger fades, and the big amount, the percent chip and the text next to it follow the finger with rolling digits.
3. **A losing month.** A tap on "1M": a black capsule slides to that pill, the line morphs into the shape of the past month and turns red, the chip turns pale red, and the two holdings roll to their change for the month.
4. **Back.** A tap on "1D": everything morphs back to the green day.
5. **Add money.** A tap on the black pill button "Add money": it turns green with a small pop and says "Added $50.00" with a check, and the big amount rolls up. Then the sheet slides away and it starts again.

`Demo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files.
- Create the four files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, durations, curves and delays. These numbers are the design.
- The chart morphs because `ChartShape` animates its heights through `AnimatableVector`. Line, area, dotted base line and end dot use the same heights. Keep that: do not swap in a chart library and do not cross-fade between two charts.
- Both lines of data must keep the same number of points (48). If you change one, change the other.
- Keep `.contentTransition(.numericText(value:))` on every amount and percent, so the digits roll.
- The pill button has three looks: black = an action to take, pale grey = switched off, green = the action worked. Keep all three.
- Labels and buttons look different on purpose. `StatusChip` and `ChangeChip` are labels. `TabPills` (white, clear outline, black when selected) and `PillButton` are buttons. Do not restyle one as the other.
- The only shared element is the black selection capsule in `TabPills`.
- Keep `.id(demo.round)` on `PortfolioView`; it makes the screen come in fresh each time.
- Keep all timing inside `Motion`. Do not write durations in other places.
- If my project already has an `@main` app, do not add a second one: leave out `PortfolioApp` and show `RootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- If a type name collides with one in my project (`Ink`, `Motion`, `Demo`, `Market`, `Holding`), rename ours with the prefix `Portfolio` and tell me.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## Logos

The scene needs no image files. `HoldingLogo` is a neutral placeholder: a white tile with the first letter of the ticker.

- Do not add real company logos on your own. Brand logos are trademarks.
- If I ask for real logos, ask me first: "Do you want the real logos of these companies in the list?"
- If I say yes: find the official vector logo yourself (the company's press page first, an open icon set such as Simple Icons second), add it to the asset catalog as an SVG with "Preserve Vector Data", and show it inside `HoldingLogo` with the same tile around it. Tell me the source you used.
- If I say no, or you cannot find an official vector: keep the placeholder.
- Never generate a logo with an image model and never redraw one from memory.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to rename.
3. Then ask me one question: "Should I connect this to your real data?" If yes, remove the auto-play (`demo.run()` and the timeline in `Demo`), fill `Market` from my model (resample each range to 48 points), drive `scrub` from a drag gesture on the chart and `range` from taps on the pills. Keep every animation as it is. This is example data, not financial advice, and nothing in it places an order.

## What I may ask you to change

- The example numbers: all values live in `Market` in `Model.swift`. Change and percent are calculated from them, do not write them by hand.
- The colours: gain is `Ink.up`, loss is `Ink.down`, the gradient is `Ink.charcoal` to `Ink.graphite`.
- More ranges: add a line of 48 heights to `Market` and return it from `Market.heights(_:)`.

## Check yourself before you answer

- With the example data the day shows $12,480.36, +2.67% and +$324.18. At 11:42 the finger shows $12,391.20 and +1.93%. The month shows −3.12% and −$401.94.
- When the range changes, the line bends into its new shape. It never jumps and never fades out and in.
- While the finger is on the chart, the dot at the end of the line is hidden.
- The button is black before the tap and green after it.
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
    static let outline = Color(red: 0.831, green: 0.831, blue: 0.847)
    static let hairline = Color(red: 0.898, green: 0.898, blue: 0.914)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)       // button that is switched off
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let charcoal = Color(red: 0.122, green: 0.161, blue: 0.216)
    static let graphite = Color(red: 0.294, green: 0.333, blue: 0.388)
    static let up = Color(red: 0.133, green: 0.773, blue: 0.369)        // gain: chart, percent, dots
    static let down = Color(red: 0.937, green: 0.267, blue: 0.267)      // loss
    static let upFill = Color(red: 0.863, green: 0.988, blue: 0.906)    // chip behind a gain
    static let upText = Color(red: 0.086, green: 0.502, blue: 0.239)
    static let downFill = Color(red: 0.996, green: 0.886, blue: 0.886)  // chip behind a loss
    static let downText = Color(red: 0.725, green: 0.11, blue: 0.11)
    static let successFill = up                                         // button after an action worked
    static let successText = Color.white
}

/// All timing goes through here. Launch with `-slow 8` to run everything eight times slower:
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
    static func spring(_ d: Double, bounce: Double = 0, delay: Double = 0) -> Animation {
        .spring(duration: d * k, bounce: bounce).delay(delay * k)
    }
    static func wait(_ seconds: Double) async {
        try? await Task.sleep(for: .seconds(seconds * k))
    }
}

struct Holding {
    let name: String
    let ticker: String
    let shares: Int
    let value: Double
    let day: Double              // change in percent, today
    let month: Double            // change in percent, past month
}

// Example data. It adds up: 12,156.18 (yesterday) + 324.18 = 12,480.36, which is +2.67 %.
// A month ago it was 12,882.30, so the month is −401.94 or −3.12 %.
enum Market {
    static let ranges = ["1D", "1W", "1M", "1Y", "All"]
    static let open = 12_156.18          // yesterday's close
    static let now = 12_480.36
    static let monthAgo = 12_882.30
    static let deposit = 50.0
    static let holdingCount = 6
    static let holdings = [
        Holding(name: "Microsoft", ticker: "MSFT", shares: 8, value: 3_412.80, day: 1.24, month: 4.08),
        Holding(name: "Netflix", ticker: "NFLX", shares: 3, value: 2_049.33, day: -0.42, month: -9.75),
    ]

    /// Today's gain at 48 moments from the open until now. 0 = yesterday's close, 1 = the value now.
    static let dayGain: [Double] = [
        0.00000, 0.04445, 0.04869, 0.08721, 0.10875, 0.16094, 0.21296, 0.23115,
        0.24847, 0.24655, 0.29288, 0.31063, 0.34137, 0.37861, 0.39319, 0.41191,
        0.41881, 0.46728, 0.47030, 0.53592, 0.51934, 0.54155, 0.59317, 0.62322,
        0.66645, 0.67432, 0.72497, 0.71275, 0.70299, 0.68519, 0.62643, 0.60558,
        0.61643, 0.61622, 0.64823, 0.74163, 0.78426, 0.76601, 0.77640, 0.81649,
        0.86246, 0.87844, 0.90061, 0.91465, 0.89989, 0.97264, 0.97270, 1.00000,
    ]
    /// The past month as heights in the chart. 0 = bottom, 1 = top.
    static let monthLine: [Double] = [
        0.915, 0.874, 0.902, 0.859, 0.841, 0.843, 0.798, 0.822,
        0.766, 0.737, 0.755, 0.741, 0.756, 0.786, 0.794, 0.774,
        0.790, 0.766, 0.752, 0.725, 0.681, 0.683, 0.662, 0.608,
        0.619, 0.570, 0.530, 0.564, 0.532, 0.584, 0.580, 0.573,
        0.627, 0.608, 0.609, 0.613, 0.560, 0.530, 0.511, 0.472,
        0.464, 0.418, 0.396, 0.383, 0.337, 0.261, 0.223, 0.156,
    ]

    /// Heights of the chart line for a range. Only "1D" (0) and "1M" (2) have data in this example.
    static func heights(_ range: Int) -> [Double] {
        range == 2 ? monthLine : dayGain.map { 0.08 + 0.84 * $0 }
    }

    /// Value of the portfolio at position `x` of today's chart (0 = open, 1 = now).
    static func value(at x: Double) -> Double {
        let p = min(max(x, 0), 1) * Double(dayGain.count - 1)
        let i = min(Int(p), dayGain.count - 2)
        let gain = dayGain[i] + (dayGain[i + 1] - dayGain[i]) * (p - Double(i))
        return open + (now - open) * gain
    }

    /// Clock time at position `x` of today's chart: the market opened at 9:30, now it is 13:30.
    static func time(at x: Double) -> String {
        let minutes = 9 * 60 + 30 + Int(min(max(x, 0), 1) * 240)
        return String(format: "%d:%02d", minutes / 60, minutes % 60)
    }

    static func money(_ value: Double) -> String {
        let f = NumberFormatter()
        f.numberStyle = .decimal
        f.locale = Locale(identifier: "en_US")
        f.minimumFractionDigits = 2
        f.maximumFractionDigits = 2
        return "$" + (f.string(from: NSNumber(value: abs(value))) ?? "")
    }
    /// "+$324.18" or "−$401.94".
    static func signedMoney(_ value: Double) -> String { (value < 0 ? "−" : "+") + money(value) }
    /// "+2.67%" or "−3.12%".
    static func percent(_ value: Double) -> String { (value < 0 ? "−" : "+") + String(format: "%.2f%%", abs(value)) }
}

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
@MainActor @Observable
final class Demo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var round = 0                    // a new number makes the screen come in fresh
    var pressed: String? = nil       // control that is being "tapped": "main", "range0" or "range2"
    var drawn: Double = 0            // how much of the chart line is drawn
    var range = 0                    // selected pill: 0 = 1D, 2 = 1M
    var scrub: Double? = nil         // where the finger is on the chart, 0...1; nil = no finger
    var added = false                // money was added

    /// The big amount: follows the finger while it is on the chart.
    var amount: Double {
        if let scrub { return Market.value(at: scrub) }
        return Market.now + (added ? Market.deposit : 0)
    }
    /// Change in money for the selected range (or up to the finger).
    var change: Double {
        if range == 2 { return Market.now - Market.monthAgo }
        return (scrub.map { Market.value(at: $0) } ?? Market.now) - Market.open
    }
    var percent: Double { change / (range == 2 ? Market.monthAgo : Market.open) * 100 }
    var note: String {
        if range == 2 { return "past month" }
        return scrub.map { "at " + Market.time(at: $0) } ?? "today"
    }
    var button: String { added ? "Added " + Market.money(Market.deposit) : "Add money" }

    private func tap(_ id: String) async {
        withAnimation(Motion.spring(0.2)) { pressed = id }
        await Motion.wait(0.17)
        withAnimation(Motion.spring(0.35, bounce: 0.2)) { pressed = nil }
        await Motion.wait(0.08)
    }

    /// Moves the finger along the chart in small steps, slow at both ends.
    private func drag(from a: Double, to b: Double, steps: Int, step: Double) async {
        for n in 0...steps {
            let t = Double(n) / Double(steps)
            let eased = t * t * (3 - 2 * t)
            scrub = a + (b - a) * eased
            await Motion.wait(step)
        }
    }

    func run() async {
        if Motion.warmup > 0 { try? await Task.sleep(for: .seconds(Motion.warmup)) }
        while !Task.isCancelled {
            // Reset while the sheet is down.
            range = 0; scrub = nil; added = false; drawn = 0
            round += 1
            await Motion.wait(0.12)

            // 1. The sheet slides up, the line draws itself from left to right.
            sheetUp = true
            await Motion.wait(0.5)
            drawn = 1
            await Motion.wait(1.5)

            // 2. A finger drags along the line to 11:42 and back. The amount follows it.
            await drag(from: 0.92, to: 26.0 / 47.0, steps: 24, step: 0.04)
            await Motion.wait(0.7)
            await drag(from: 26.0 / 47.0, to: 1, steps: 14, step: 0.035)
            scrub = nil
            await Motion.wait(0.45)

            // 3. "1M": the line changes its shape into the losing month and turns red.
            await tap("range2")
            withAnimation(Motion.spring(0.7, bounce: 0.12)) { range = 2 }
            await Motion.wait(1.5)

            // 4. "1D": back to the green day.
            await tap("range0")
            withAnimation(Motion.spring(0.7, bounce: 0.12)) { range = 0 }
            await Motion.wait(1.15)

            // 5. "Add money": the button turns green, the amount rolls up.
            await tap("main")
            added = true
            await Motion.wait(1.5)

            // 6. The sheet slides away, then everything starts again.
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
/// Switched off, it is pale grey with grey text. With `success` it turns fresh green with white
/// text and a small pop:
/// the answer to an action that worked.
struct PillButton: View {
    let title: String
    var enabled = true
    var success = false
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
        .foregroundStyle(success ? Ink.successText : (enabled ? Color.white : Ink.secondary))
        .frame(maxWidth: .infinity)
        .frame(height: 54)
        .background(Capsule().fill(success ? Ink.successFill : (enabled ? Ink.black : Ink.off)))
        .overlay(
            Circle()
                .fill(Color.white.opacity(0.26))
                .frame(width: 44, height: 44)
                .scaleEffect(pressed ? 1 : 1.5)
                .opacity(pressed ? 1 : 0)
        )
        .scaleEffect(pressed ? 0.965 : 1)
        .keyframeAnimator(initialValue: CGFloat(1), trigger: success) { view, pop in
            view.scaleEffect(pop)
        } keyframes: { _ in
            KeyframeTrack {
                SpringKeyframe(CGFloat(success ? 1.04 : 1), duration: 0.14 * Motion.k, spring: .snappy)
                SpringKeyframe(CGFloat(1), duration: 0.45 * Motion.k, spring: .bouncy)
            }
        }
        .animation(Motion.spring(0.6), value: title)
        .animation(Motion.smooth(0.45), value: enabled)
        .animation(Motion.smooth(0.35), value: success)
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

/// Small grey capsule: a status label, not a button. With `dot` a coloured dot sits in front of the text.
struct StatusChip: View {
    let text: String
    var dot: Color? = nil

    var body: some View {
        HStack(spacing: 8) {
            if let dot {
                Circle().fill(dot).frame(width: 10, height: 10)
                    .transition(.scale(scale: 0.4).combined(with: .opacity))
            }
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

/// Soft dot that shows where the demo "taps".
struct TouchDot: View {
    let on: Bool

    var body: some View {
        Circle()
            .fill(Color.black.opacity(0.12))
            .frame(width: 40, height: 40)
            .scaleEffect(on ? 1 : 1.5)
            .opacity(on ? 1 : 0)
    }
}

/// Row of outline pill buttons. One is selected: a black capsule slides behind its title.
struct TabPills: View {
    let titles: [String]
    var selected: Int? = nil
    var pressed: Int? = nil
    @Namespace private var ns

    var body: some View {
        HStack(spacing: 7) {
            ForEach(0..<titles.count, id: \.self) { i in
                let on = selected == i
                Text(titles[i])
                    .font(.system(size: 15, weight: .semibold))
                    .foregroundStyle(on ? Color.white : Ink.primary)
                    .lineLimit(1)
                    .frame(maxWidth: .infinity)
                    .frame(height: 34)
                    .background {
                        ZStack {
                            Capsule().fill(Color.white)
                            Capsule().strokeBorder(Ink.outline, lineWidth: 1.5)
                            if on {
                                Capsule().fill(Ink.black).matchedGeometryEffect(id: "selection", in: ns)
                            }
                        }
                    }
                    .overlay(TouchDot(on: pressed == i))
                    .scaleEffect(pressed == i ? 0.95 : 1)
            }
        }
    }
}

/// Small chip with the change in percent: pale green for a gain, pale red for a loss.
struct ChangeChip: View {
    let percent: Double

    var body: some View {
        let up = percent >= 0
        Text(Market.percent(percent))
            .font(.system(size: 16, weight: .semibold))
            .monospacedDigit()
            .foregroundStyle(up ? Ink.upText : Ink.downText)
            .contentTransition(.numericText(value: percent))
            .padding(.horizontal, 9)
            .frame(height: 26)
            .background(RoundedRectangle(cornerRadius: 9, style: .continuous).fill(up ? Ink.upFill : Ink.downFill))
    }
}

/// A row of numbers that SwiftUI can animate as one value. It lets the chart line change its shape smoothly.
struct AnimatableVector: VectorArithmetic {
    var values: [Double]

    static var zero: AnimatableVector { AnimatableVector(values: []) }
    static func + (a: AnimatableVector, b: AnimatableVector) -> AnimatableVector { combine(a, b, +) }
    static func - (a: AnimatableVector, b: AnimatableVector) -> AnimatableVector { combine(a, b, -) }

    private static func combine(_ a: AnimatableVector, _ b: AnimatableVector,
                                _ op: (Double, Double) -> Double) -> AnimatableVector {
        let n = max(a.values.count, b.values.count)
        return AnimatableVector(values: (0..<n).map { i in
            op(i < a.values.count ? a.values[i] : 0, i < b.values.count ? b.values[i] : 0)
        })
    }

    mutating func scale(by rhs: Double) { values = values.map { $0 * rhs } }
    var magnitudeSquared: Double { values.reduce(0) { $0 + $1 * $1 } }
}

/// The parts of the chart as shapes that share one set of heights (0 = bottom, 1 = top).
/// Because the heights are animatable, line, area, base line and end dot all morph together.
struct ChartShape: Shape {
    enum Part { case line, area, base, dot(CGFloat) }

    var heights: AnimatableVector
    var part: Part

    var animatableData: AnimatableVector {
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

/// The hero: a line chart with a soft area under it, a dotted line for the starting value and a dot at the end.
/// `drawn` draws the line from left to right. With `scrub` a finger is on the chart: a marker stands at that
/// place, the part to the right of it fades, and `label` is shown in a small pill above the marker.
struct ChartView: View {
    let heights: [Double]
    let color: Color
    var drawn: Double = 1
    var scrub: Double? = nil
    var label = ""

    private let height: CGFloat = 100

    var body: some View {
        let vec = AnimatableVector(values: heights)
        GeometryReader { g in
            let w = g.size.width
            let at = scrub ?? 1
            let on = scrub != nil
            let x = w * at
            let y = height - height * level(at)
            ZStack(alignment: .topLeading) {
                ChartShape(heights: vec, part: .base)
                    .stroke(Ink.outline, style: StrokeStyle(lineWidth: 1.6, lineCap: .round, dash: [0.5, 6]))

                chart(vec).opacity(on ? 0.26 : 0)
                chart(vec).mask(alignment: .leading) {
                    Rectangle().frame(width: on ? x : w + 30)
                }

                ChartShape(heights: vec, part: .dot(10)).fill(color.opacity(0.22))
                    .opacity(!on && drawn >= 1 ? 1 : 0)
                ChartShape(heights: vec, part: .dot(6.5)).fill(Color.white)
                    .opacity(!on && drawn >= 1 ? 1 : 0)
                ChartShape(heights: vec, part: .dot(4.5)).fill(color)
                    .opacity(!on && drawn >= 1 ? 1 : 0)

                // The finger's marker.
                Group {
                    Rectangle().fill(Ink.primary)
                        .frame(width: 1.5, height: height + 6)
                        .position(x: x, y: height / 2 - 3)
                    Circle().fill(Ink.primary)
                        .frame(width: 11, height: 11)
                        .overlay(Circle().strokeBorder(Color.white, lineWidth: 2))
                        .position(x: x, y: y)
                    Text(label)
                        .font(.system(size: 13, weight: .semibold))
                        .monospacedDigit()
                        .foregroundStyle(Color.white)
                        .padding(.horizontal, 9)
                        .frame(height: 23)
                        .background(Capsule().fill(Ink.black))
                        .fixedSize()
                        .position(x: min(max(x, 28), w - 28), y: -18)
                    TouchDot(on: on).position(x: x, y: y)
                }
                .opacity(on ? 1 : 0)
            }
        }
        .frame(height: height)
        .animation(Motion.spring(0.7, bounce: 0.12), value: heights)
        .animation(Motion.smooth(1.1), value: drawn)
        .animation(Motion.smooth(0.16), value: scrub)
    }

    private func chart(_ vec: AnimatableVector) -> some View {
        ZStack {
            ChartShape(heights: vec, part: .area)
                .fill(LinearGradient(colors: [color.opacity(0.26), color.opacity(0)], startPoint: .top, endPoint: .bottom))
                .opacity(drawn)
            ChartShape(heights: vec, part: .line)
                .trim(from: 0, to: drawn)
                .stroke(color, style: StrokeStyle(lineWidth: 2.4, lineCap: .round, lineJoin: .round))
        }
    }

    /// Height of the line at position 0...1, between two points.
    private func level(_ at: Double) -> CGFloat {
        let p = min(max(at, 0), 1) * Double(heights.count - 1)
        let i = min(Int(p), heights.count - 2)
        return CGFloat(heights[i] + (heights[i + 1] - heights[i]) * (p - Double(i)))
    }
}

/// Small logo tile in front of a holding.
/// This is a neutral placeholder: the first letter of the ticker on a white tile.
/// Real brand logos are the user's choice, see "Logos" in the prompt.
struct HoldingLogo: View {
    let ticker: String

    var body: some View {
        GeometryReader { g in
            Text(String(ticker.prefix(1)))
                .font(.system(size: min(g.size.width, g.size.height) * 0.44, weight: .bold))
                .foregroundStyle(Ink.primary)
                .frame(width: g.size.width, height: g.size.height)
        }
        .background(RoundedRectangle(cornerRadius: 11, style: .continuous).fill(Color.white))
        .overlay(RoundedRectangle(cornerRadius: 11, style: .continuous).strokeBorder(Ink.hairline, lineWidth: 1.5))
    }
}
```

### `PortfolioView.swift`

```swift
import SwiftUI

/// The portfolio screen. There is only this one screen; taps and the finger change its state:
/// the chart morphs between ranges, the amount follows the finger, the colours switch between gain and loss.
struct PortfolioView: View {
    let demo: Demo
    @State private var shown = false

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        let month = demo.range == 2

        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Portfolio")
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                StatusChip(text: "Market open", dot: Ink.up)
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            Text(Market.money(demo.amount))
                .font(.system(size: 45, weight: .bold))
                .tracking(-1)
                .monospacedDigit()
                .foregroundStyle(Ink.primary)
                .contentTransition(.numericText(value: demo.amount))
                .padding(.leading, 3)
                .padding(.top, 12)
                .reveal(shown, delay: lead + 0.06)

            HStack(spacing: 10) {
                ChangeChip(percent: demo.percent)
                Text(Market.signedMoney(demo.change) + " " + demo.note)
                    .font(.system(size: 17))
                    .monospacedDigit()
                    .foregroundStyle(Ink.secondary)
                    .contentTransition(.numericText())
                    .lineLimit(1)
            }
            .padding(.leading, 4)
            .padding(.top, 6)
            .reveal(shown, delay: lead + 0.1)

            ChartView(heights: Market.heights(demo.range), color: month ? Ink.down : Ink.up,
                      drawn: demo.drawn, scrub: demo.scrub, label: demo.scrub.map { Market.time(at: $0) } ?? "")
                .padding(.horizontal, 2)
                .padding(.top, 30)
                .reveal(shown, delay: lead + 0.16)

            TabPills(titles: Market.ranges, selected: demo.range,
                     pressed: demo.pressed == "range2" ? 2 : (demo.pressed == "range0" ? 0 : nil))
                .padding(.top, 18)
                .reveal(shown, delay: lead + 0.22)

            HStack {
                Text("Top holdings")
                    .font(.system(size: 15))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                HStack(spacing: 4) {
                    Text("See all \(Market.holdingCount)")
                        .font(.system(size: 15, weight: .semibold))
                    Image(systemName: "chevron.right")
                        .font(.system(size: 11, weight: .bold))
                }
                .foregroundStyle(Ink.primary)
            }
            .padding(.horizontal, 4)
            .padding(.top, 20)
            .reveal(shown, delay: lead + 0.26)

            VStack(spacing: 0) {
                ForEach(Market.holdings.indices, id: \.self) { i in
                    row(Market.holdings[i], month: month)
                    if i < Market.holdings.count - 1 {
                        Rectangle().fill(Ink.hairline).frame(height: 1).padding(.horizontal, 12)
                    }
                }
            }
            .background(RoundedRectangle(cornerRadius: 20, style: .continuous).strokeBorder(Ink.hairline, lineWidth: 1.5))
            .padding(.top, 12)
            .reveal(shown, delay: lead + 0.3)
        }
        .animation(Motion.spring(0.7, bounce: 0.12), value: demo.range)
        .animation(Motion.smooth(0.16), value: demo.scrub)
        .animation(Motion.spring(0.8), value: demo.added)
        .onAppear { shown = true }
    }

    private func row(_ h: Holding, month: Bool) -> some View {
        let change = month ? h.month : h.day
        return HStack(spacing: 12) {
            HoldingLogo(ticker: h.ticker)
                .frame(width: 42, height: 42)
            VStack(alignment: .leading, spacing: 2) {
                Text(h.name)
                    .font(.system(size: 17, weight: .semibold))
                    .foregroundStyle(Ink.primary)
                Text("\(h.ticker) · \(h.shares) shares")
                    .font(.system(size: 14))
                    .foregroundStyle(Ink.secondary)
            }
            Spacer(minLength: 8)
            VStack(alignment: .trailing, spacing: 2) {
                Text(Market.money(h.value))
                    .font(.system(size: 17, weight: .semibold))
                    .foregroundStyle(Ink.primary)
                    .monospacedDigit()
                Text(Market.percent(change))
                    .font(.system(size: 14, weight: .medium))
                    .foregroundStyle(change >= 0 ? Ink.up : Ink.down)
                    .monospacedDigit()
                    .contentTransition(.numericText(value: change))
            }
        }
        .padding(.horizontal, 12)
        .frame(height: 62)
    }
}
```

### `PortfolioApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
@main
struct PortfolioApp: App {
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
            LinearGradient(colors: [Ink.charcoal, Ink.graphite], startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()

            VStack(spacing: 0) {
                PortfolioView(demo: demo)
                    .id(demo.round)
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // Black = an action to take. After the money was added it turns green: it worked.
                PillButton(title: demo.button, success: demo.added, done: demo.added,
                           pressed: demo.pressed == "main")
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
