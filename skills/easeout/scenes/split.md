# Easeout scene: Split the bill (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

One screen inside a white sheet over a mint-to-green gradient: a restaurant bill that three people split. Its hero is a ring chart. Each person has a colour, and the ring shows who pays how much. It plays in four beats:

1. **Start.** The sheet slides up and the content comes in piece by piece. The ring draws itself in grey: nothing is assigned yet. All three shares say "$0.00", every dish has a small plus button, and the pill button "Request" is switched off (pale grey).
2. **Assign.** Four taps on the plus buttons. Each tap gives a dish to the people who had it: a coloured dot pops in on the row, that person's colour grows on the ring, their portrait jumps onto the middle of their arc, and their share rolls up. The last dish is shared by all three, so all three arcs grow and the portraits ride along the ring to their new places. Then the button turns black and its title rolls to "Request $57.60".
3. **Request.** A tap on the button: it turns green with a small pop and the title rolls to "Requested" with a small check. Green is the answer to an action that worked. The two friends pay back one after the other: a green check pops onto each portrait and the amount in the middle of the ring rolls up to what has been paid back.
4. **Settled.** The label at the top says "Settled" with a green dot and the button turns black again and says "Done". Then the sheet slides away and it starts again.

`Demo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files.
- Create the four files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, durations, curves and delays. These numbers are the design.
- The ring is drawn by `RingArc`, whose two ends animate, and the portraits are placed by `Orbit`, whose turn animates. Keep both: this is why the arcs grow smoothly and the portraits travel along the ring instead of cutting across it.
- Keep `.contentTransition(.numericText(value:))` on every amount, so the digits roll.
- The pill button has three looks: black = an action to take, pale grey = switched off, green = the action worked. Keep all three.
- Labels and buttons look different on purpose. `StatusChip` (pale grey, grey text) is a label. The plus circles (white, clear outline) and `PillButton` (black, or pale grey when switched off) are buttons. Do not restyle one as the other.
- Keep `.id(demo.round)` on `SplitView`; it makes the screen come in fresh each time.
- Keep all timing inside `Motion`. Do not write durations in other places.
- If my project already has an `@main` app, do not add a second one: leave out `SplitApp` and show `RootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- If a type name collides with one in my project (`Ink`, `Motion`, `Demo`, `Bill`, `Person`, `Dish`), rename ours with the prefix `Split` and tell me.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## Photos

The scene needs no image files. `Portrait` is a neutral placeholder: a soft disc in the person's colour with their initial.

- Do not add photos of people on your own, and never take photos of real people from the web.
- If I ask for photos, ask me first where they come from: my own files, or generated ones.
- My own files: add them to the asset catalog and show them in `Portrait` with `Image(person.image).resizable().scaledToFill().clipShape(Circle())`, keeping the white border.
- Generated ones: only if an image tool is connected and I said yes. Make fictional people, and tell me that the faces are generated.
- If I say no, keep the placeholder.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to rename.
3. Then ask me one question: "Should I connect this to your real bills?" If yes, remove the auto-play (`demo.run()` and the timeline in `Demo`), fill `Bill` from my model, and let real taps change `assigned`, `requested` and `paid`. Keep every animation as it is.

## What I may ask you to change

- The example bill: people, dishes, prices and tip live in `Bill` in `Model.swift`. Shares and the amount to request are calculated from them, do not write them by hand.
- The colours: one colour per person in `Bill.people`, the gradient is `Ink.mint` to `Ink.emerald`.
- More people or dishes: add them to `Bill`; the ring, the legend and the list follow.

## Check yourself before you answer

- With the example data the shares are $28.80, $32.40 and $25.20, the total is $86.40 and the request is $57.60.
- Before any tap the ring is one closed grey ring. After all four taps there is no grey left.
- Each portrait sits on the middle of its own colour.
- The button is pale grey until all dishes are assigned.
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
    static let track = Color(red: 0.902, green: 0.902, blue: 0.918)     // grey ring, nothing assigned
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)       // button that is switched off
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let mint = Color(red: 0.655, green: 0.953, blue: 0.816)
    static let emerald = Color(red: 0.204, green: 0.827, blue: 0.6)
    static let green = Color(red: 0.133, green: 0.773, blue: 0.369)     // "paid" check and "Settled" dot
    static let successFill = Color(red: 0.133, green: 0.773, blue: 0.369)   // button after an action worked: fresh green
    static let successText = Color.white                                    // … with white text
    static let orange = Color(red: 0.984, green: 0.573, blue: 0.235)
    static let blue = Color(red: 0.376, green: 0.647, blue: 0.98)
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

struct Person {
    let name: String
    let initials: String
    let image: String
    let color: Color
}

struct Dish {
    let name: String
    let price: Double
    let eaters: [Int]            // who had it, as positions in `Bill.people`
}

// Example data. It adds up: the dishes are $72.00, plus 20 % tip is $86.40.
// Each share is the own dish plus a third of the tiramisu, plus tip: 28.80 + 32.40 + 25.20 = 86.40.
enum Bill {
    static let title = "Dinner at Nonna"
    static let tip = 0.2
    static let people = [
        Person(name: "You", initials: "Y", image: "you", color: Ink.emerald),
        Person(name: "Mia", initials: "M", image: "mia", color: Ink.orange),
        Person(name: "Leo", initials: "L", image: "leo", color: Ink.blue),
    ]
    static let dishes = [
        Dish(name: "Lasagne", price: 21, eaters: [0]),
        Dish(name: "Truffle pasta", price: 24, eaters: [1]),
        Dish(name: "Margherita", price: 18, eaters: [2]),
        Dish(name: "Tiramisu", price: 9, eaters: [0, 1, 2]),
    ]
    static let total = dishes.reduce(0) { $0 + $1.price } * (1 + tip)

    /// What one person pays for the dishes assigned so far, tip included.
    static func share(of person: Int, assigned: Set<Int>) -> Double {
        var sum = 0.0
        for d in assigned where dishes[d].eaters.contains(person) {
            sum += dishes[d].price / Double(dishes[d].eaters.count)
        }
        return sum * (1 + tip)
    }

    static func money(_ value: Double) -> String { String(format: "$%.2f", value) }
}

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
@MainActor @Observable
final class Demo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var round = 0                    // a new number makes the screen come in fresh
    var pressed: String? = nil       // control that is being "tapped": "main" or "dish0" … "dish3"
    var ringDrawn: Double = 0        // how much of the grey ring is drawn
    var assigned: Set<Int> = []      // dishes that have been given to their people
    var requested = false            // the request has been sent
    var paid: Set<Int> = []          // people who have paid back

    var shares: [Double] { Bill.people.indices.map { Bill.share(of: $0, assigned: assigned) } }
    var complete: Bool { assigned.count == Bill.dishes.count }
    var settled: Bool { paid.count == Bill.people.count - 1 }
    /// What the others owe you: everything except your own share.
    var owed: Double { shares.dropFirst().reduce(0, +) }
    var received: Double { paid.reduce(0) { $0 + shares[$1] } }

    var button: String {
        if settled { return "Done" }
        if requested { return "Requested" }
        return complete ? "Request " + Bill.money(owed) : "Request"
    }

    private func tap(_ id: String) async {
        withAnimation(Motion.spring(0.2)) { pressed = id }
        await Motion.wait(0.17)
        withAnimation(Motion.spring(0.35, bounce: 0.2)) { pressed = nil }
        await Motion.wait(0.08)
    }

    func run() async {
        if Motion.warmup > 0 { try? await Task.sleep(for: .seconds(Motion.warmup)) }
        while !Task.isCancelled {
            // Reset while the sheet is down.
            assigned = []; requested = false; paid = []; ringDrawn = 0
            round += 1
            await Motion.wait(0.12)

            // 1. The sheet slides up, the grey ring draws itself.
            sheetUp = true
            await Motion.wait(0.45)
            ringDrawn = 1
            await Motion.wait(0.85)

            // 2. Four taps. Each one gives a dish to its people: a coloured dot pops in,
            //    the colour grows on the ring, the picture jumps onto it and the share rolls up.
            for d in Bill.dishes.indices {
                await tap("dish\(d)")
                assigned.insert(d)
                await Motion.wait(d == Bill.dishes.count - 1 ? 1.35 : 0.55)
            }

            // 3. Request: the two friends pay back one after the other.
            await tap("main")
            requested = true
            await Motion.wait(0.7)
            paid.insert(1)
            await Motion.wait(0.75)
            paid.insert(2)
            await Motion.wait(1.5)

            // 4. Done: the sheet slides away, then everything starts again.
            await tap("main")
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

/// One arc of the ring, from `start` to `end` (both 0...1, clockwise from the top).
/// Both ends can animate. The arc is drawn a little shorter than asked, so that neighbours
/// with round ends leave a small white gap between them.
struct RingArc: Shape {
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
struct Orbit: ViewModifier, Animatable {
    var turn: Double
    var radius: CGFloat

    var animatableData: Double {
        get { turn }
        set { turn = newValue }
    }

    func body(content: Content) -> some View {
        let angle = turn * 2 * Double.pi - Double.pi / 2
        content.offset(x: radius * CGFloat(cos(angle)), y: radius * CGFloat(sin(angle)))
    }
}

/// Small green check that pops onto a portrait when that person has paid.
struct CheckBadge: View {
    let on: Bool

    var body: some View {
        ZStack {
            Circle().fill(Color.white)
            Circle().fill(Ink.green).padding(2)
            Path { p in
                p.move(to: CGPoint(x: 0, y: 3.6))
                p.addLine(to: CGPoint(x: 2.8, y: 6.4))
                p.addLine(to: CGPoint(x: 7.6, y: 0.4))
            }
            .trim(from: 0, to: on ? 1 : 0)
            .stroke(Color.white, style: StrokeStyle(lineWidth: 1.8, lineCap: .round, lineJoin: .round))
            .frame(width: 7.6, height: 6.8)
        }
        .frame(width: 19, height: 19)
        .scaleEffect(on ? 1 : 0.4)
        .opacity(on ? 1 : 0)
        .animation(Motion.spring(0.5, bounce: 0.4), value: on)
    }
}

/// The hero: a ring that shows who pays how much of the bill.
/// `fractions` is each person's part of the whole bill. What is not assigned yet stays grey.
/// A person's portrait sits on the middle of their arc and rides along when the arcs change.
struct RingChart: View {
    let fractions: [Double]
    var drawn: Double = 1          // 0...1, how much of the grey ring is drawn
    var paid: Set<Int> = []

    private let size: CGFloat = 196
    private let line: CGFloat = 19

    var body: some View {
        let starts = fractions.indices.map { i in fractions[..<i].reduce(0, +) }
        let used = min(fractions.reduce(0, +), 1)
        ZStack {
            RingArc(start: used, end: used + (1 - used) * drawn, line: line)
                .stroke(Ink.track, style: StrokeStyle(lineWidth: line, lineCap: .round))
                .opacity(used < 0.999 ? 1 : 0)
            ForEach(fractions.indices, id: \.self) { i in
                RingArc(start: starts[i], end: starts[i] + fractions[i], line: line)
                    .stroke(Bill.people[i].color, style: StrokeStyle(lineWidth: line, lineCap: .round))
                    .opacity(fractions[i] > 0 ? 1 : 0)
            }
            ForEach(fractions.indices, id: \.self) { i in
                Portrait(person: Bill.people[i])
                    .overlay(alignment: .bottomTrailing) {
                        CheckBadge(on: paid.contains(i)).offset(x: 4, y: 4)
                    }
                    .frame(width: 47, height: 47)
                    .scaleEffect(fractions[i] > 0 ? 1 : 0.5)
                    .opacity(fractions[i] > 0 ? 1 : 0)
                    .modifier(Orbit(turn: starts[i] + fractions[i] / 2, radius: (size - line) / 2))
            }
        }
        .frame(width: size, height: size)
    }
}

/// Round portrait with a white border.
/// This is a neutral placeholder: the person's colour as a soft disc with their initial.
/// Real photos are the user's choice, see "Photos" in the prompt.
struct Portrait: View {
    let person: Person

    var body: some View {
        GeometryReader { g in
            let s = min(g.size.width, g.size.height)
            ZStack {
                Circle().fill(Color.white)
                Circle().fill(person.color.opacity(0.22)).padding(3)
                Text(person.initials)
                    .font(.system(size: s * 0.4, weight: .bold))
                    .foregroundStyle(Ink.primary)
            }
            .frame(width: g.size.width, height: g.size.height)
        }
        .shadow(color: Color.black.opacity(0.1), radius: 3, y: 1)
    }
}
```

### `SplitView.swift`

```swift
import SwiftUI

/// The split-the-bill screen. There is only this one screen; taps change its state:
/// dishes get assigned (colour grows on the ring, shares roll up), then the friends pay back.
struct SplitView: View {
    let demo: Demo
    @State private var shown = false

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        let shares = demo.shares

        VStack(spacing: 0) {
            HStack {
                Text(Bill.title)
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                StatusChip(text: demo.settled ? "Settled" : "\(Bill.people.count) people",
                           dot: demo.settled ? Ink.green : nil)
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            RingChart(fractions: shares.map { $0 / Bill.total }, drawn: demo.ringDrawn, paid: demo.paid)
                .overlay { centre }
                .padding(.top, 16)
                .reveal(shown, delay: lead + 0.06)

            HStack(alignment: .top) {
                ForEach(Bill.people.indices, id: \.self) { i in
                    if i > 0 { Spacer(minLength: 0) }
                    legend(i, shares[i])
                }
            }
            .padding(.horizontal, 10)
            .padding(.top, 24)
            .reveal(shown, delay: lead + 0.14)

            VStack(spacing: 0) {
                ForEach(Bill.dishes.indices, id: \.self) { d in
                    row(d)
                    if d < Bill.dishes.count - 1 {
                        Rectangle().fill(Ink.hairline).frame(height: 1).padding(.horizontal, 6)
                    }
                }
            }
            .background(RoundedRectangle(cornerRadius: 20, style: .continuous).strokeBorder(Ink.hairline, lineWidth: 1.5))
            .padding(.top, 24)
            .reveal(shown, delay: lead + 0.22)
        }
        .animation(Motion.spring(0.75, bounce: 0.16), value: demo.assigned)
        .animation(Motion.spring(0.7, bounce: 0.1), value: demo.paid)
        .animation(Motion.smooth(1.0), value: demo.ringDrawn)
        .onAppear { shown = true }
    }

    /// Middle of the ring: the bill total, later what has been paid back.
    private var centre: some View {
        let back = !demo.paid.isEmpty
        let amount = back ? demo.received : Bill.total
        return VStack(spacing: 1) {
            Text(Bill.money(amount))
                .font(.system(size: 33, weight: .bold))
                .foregroundStyle(Ink.primary)
                .monospacedDigit()
                .contentTransition(.numericText(value: amount))
            Text(back ? "paid back" : "incl. \(Int(Bill.tip * 100))% tip")
                .font(.system(size: 15))
                .foregroundStyle(Ink.secondary)
                .contentTransition(.numericText())
        }
    }

    private func legend(_ i: Int, _ share: Double) -> some View {
        VStack(alignment: .leading, spacing: 3) {
            HStack(spacing: 7) {
                Circle().fill(Bill.people[i].color).frame(width: 11, height: 11)
                Text(Bill.people[i].name)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(Ink.secondary)
            }
            Text(Bill.money(share))
                .font(.system(size: 21, weight: .bold))
                .foregroundStyle(Ink.primary)
                .monospacedDigit()
                .contentTransition(.numericText(value: share))
        }
    }

    /// One dish. Until it is assigned it shows a plus button, afterwards the dots of the people who had it.
    private func row(_ d: Int) -> some View {
        let dish = Bill.dishes[d]
        let on = demo.assigned.contains(d)
        let pressed = demo.pressed == "dish\(d)"
        return HStack(spacing: 0) {
            Text(dish.name)
                .font(.system(size: 17, weight: .medium))
                .foregroundStyle(Ink.primary)
            Spacer(minLength: 8)
            ZStack(alignment: .trailing) {
                HStack(spacing: 3) {
                    ForEach(dish.eaters.indices, id: \.self) { n in
                        Circle().fill(Bill.people[dish.eaters[n]].color)
                            .frame(width: 12, height: 12)
                            .scaleEffect(on ? 1 : 0.3)
                            .opacity(on ? 1 : 0)
                            .animation(Motion.spring(0.5, bounce: 0.45, delay: Double(n) * 0.07), value: on)
                    }
                }
                Image(systemName: "plus")
                    .font(.system(size: 11, weight: .bold))
                    .foregroundStyle(Ink.primary)
                    .frame(width: 24, height: 24)
                    .background(Circle().fill(Color.white))
                    .overlay(Circle().strokeBorder(Ink.outline, lineWidth: 1.5))
                    .overlay(TouchDot(on: pressed))
                    .scaleEffect(pressed ? 0.9 : (on ? 0.6 : 1))
                    .opacity(on ? 0 : 1)
                    .blur(radius: on ? 4 : 0)
                    .animation(Motion.smooth(0.3), value: on)
                    .offset(x: 6)
            }
            Text(Bill.money(dish.price))
                .font(.system(size: 17, weight: .semibold))
                .foregroundStyle(Ink.primary)
                .monospacedDigit()
                .frame(width: 84, alignment: .trailing)
        }
        .padding(.horizontal, 16)
        .frame(height: 36)
    }
}
```

### `SplitApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
@main
struct SplitApp: App {
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
            LinearGradient(colors: [Ink.mint, Ink.emerald], startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()

            VStack(spacing: 0) {
                SplitView(demo: demo)
                    .id(demo.round)
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // Switched off until every dish is assigned, then black with the amount to request.
                // After the request went out it is green until everybody has paid.
                PillButton(title: demo.button, enabled: demo.complete,
                           success: demo.requested && !demo.settled,
                           done: demo.requested && !demo.settled, pressed: demo.pressed == "main")
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
