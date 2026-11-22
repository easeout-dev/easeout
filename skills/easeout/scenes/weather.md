# Easeout scene: Weather, the sun's path (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

One screen inside a white sheet over a sky-coloured strip: today's weather. Its hero is a sky panel with the path of the sun. Everything follows one number, the position of the sun: the sky, the big temperature, and the grey tile in the row of moments under the panel. It plays in five beats:

1. **Sunrise.** The sheet slides up and the content comes in piece by piece. The sun rises along its bow to where it is now, the line behind it grows, and the big temperature counts up.
2. **Golden hour.** A finger takes the sun and drags it along the bow to the golden hour. The sky and the strip above the sheet turn amber and coral, the lines turn white, the temperature rolls, the grey line shows the time of the forecast, and the grey tile in the row below slides from "Now" to "Golden".
3. **Sunset.** The finger drags on to the end of the bow. The sky turns indigo, the clouds fade, a moon and eight stars pop in, and the tile moves on to "Sunset".
4. **Back.** The finger lets go. One spring brings the sun back to now, the sky clears, and the tile returns to "Now".
5. **Reminder.** A tap on the black pill button "Remind me at golden hour": it turns green with a small pop and says "Reminder set for 19:47" with a check, and a small bell pops into the tag in the panel. Then the sheet slides away and it starts again.

`WeatherSceneDemo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files.
- Create the five files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, durations, curves and delays. These numbers are the design.
- The sky never cross-fades between two pictures. `WeatherSceneSkyPanel`, `WeatherSceneSkyStrip` and `WeatherSceneHourRow` are `Animatable`: they get the sun's position and work out every colour and every position from it, frame by frame. Keep that.
- Keep `.contentTransition(.numericText(value:))` on the temperature, so the digits roll.
- The pill button has three looks: black = an action to take, pale grey = switched off, green = the action worked. Keep all three.
- Labels and buttons look different on purpose. `WeatherSceneStatusChip` is a label, `WeatherScenePillButton` is a button. Do not restyle one as the other.
- No `matchedGeometryEffect`.
- Keep `.id(demo.round)` on `WeatherSceneView`; it makes the screen come in fresh each time.
- Keep all timing inside `WeatherSceneMotion`. Do not write durations in other places.
- Every type and every file of the scene starts with `WeatherScene`, so nothing collides with my project, also when it is a weather app with its own `WeatherView`. Do not rename them.
- If my project already has an `@main` app, do not add a second one: leave out the file `WeatherSceneApp.swift` (it holds nothing but the entry point), keep the other four files unchanged, and show `WeatherSceneRootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- The scene respects Reduce Motion and stops its loop when the view goes away. Keep both.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to change.
3. Then ask me one question: "Should I connect this to real weather data?" If yes, remove the auto-play (`demo.run()` and the timeline in `WeatherSceneDemo`), fill `WeatherSceneDay` from my forecast (sunrise, sunset and the temperatures of the day), and move the sun with a drag gesture on the sky panel. Ask me before you add a location permission or a weather service to the project. Keep every animation as it is.

## What I may ask you to change

- The example day: all values live in `WeatherSceneDay` in `WeatherSceneModel.swift`. The temperature, the time under the big number, the chip and the numbers in the row are calculated from them, do not write them by hand.
- The colours of the three skies (day, golden hour, dusk): they live in `WeatherSceneInk`.
- The place: the title "Munich" in `WeatherSceneView`.

## Check yourself before you answer

- With the example data the screen shows 21° at rest, 18° and "Forecast for 19:47" at the golden hour, and 17° and "Forecast for 20:47" at sunset.
- While the sun moves, sky, strip, lines, clouds and sun change together. Nothing jumps and nothing fades out and in.
- The grey tile in the row under the panel sits exactly on "Now" at rest, on "Golden" (18°) at 19:47 and on "Sunset" (17°) at 20:47. The number in the tile's column equals the big temperature at those moments.
- The button is black before the tap and green after it.
- Light mode only; `WeatherSceneRootView` sets the light colour scheme for its own content.

## The code

### `WeatherSceneModel.swift`

```swift
import SwiftUI
import Observation

// Colours of the scene (flat look, no glass).
enum WeatherSceneInk {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)
    static let secondary = Color(red: 0.459, green: 0.463, blue: 0.498)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)           // button that is switched off
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let successFill = Color(red: 0.133, green: 0.773, blue: 0.369)   // button after an action worked
    static let successText = Color.white
    static let tile = hex(0xE7E6EB)          // behind the "Now" column
    static let sunny = hex(0xFECC04)         // chip dot by day, and the small sun icons
    static let golden = hex(0xFF963E)        // chip dot at golden hour
    static let sunset = hex(0xF778BC)        // chip dot at sunset
    static let bell = hex(0xFF8818)
    static let moon = hex(0xFBE79A)          // crescent in the dusk sky
    static let iconCloud = hex(0xC7C7CD)

    /// One colour for each of the three skies: day (phase 0), golden hour (1) and dusk (2).
    /// `at` mixes the two neighbours of a phase in plain RGB, so a colour can follow the sun smoothly.
    struct Tone: Sendable {
        let day: UInt32
        let golden: UInt32
        let dusk: UInt32

        init(_ day: UInt32, _ golden: UInt32, _ dusk: UInt32) {
            self.day = day
            self.golden = golden
            self.dusk = dusk
        }

        func at(_ phase: Double) -> Color {
            let p = min(max(phase, 0), 2)
            let from = p < 1 ? day : golden
            let to = p < 1 ? golden : dusk
            let f = p < 1 ? p : p - 1
            func channel(_ shift: UInt32) -> Double {
                let a = Double((from >> shift) & 0xFF)
                let b = Double((to >> shift) & 0xFF)
                return (a + (b - a) * f) / 255
            }
            return Color(red: channel(16), green: channel(8), blue: channel(0))
        }
    }

    static func hex(_ value: UInt32) -> Color { Tone(value, value, value).at(0) }

    /// The sky in the panel: four stops from top to bottom.
    static let sky: [(at: CGFloat, tone: Tone)] = [
        (0, Tone(0x19C4FD, 0xFEBB20, 0x2F2A86)),
        (0.5, Tone(0x6DD5FE, 0xFC9467, 0x6639CF)),
        (0.78, Tone(0x9BE0FD, 0xF97E7F, 0x9D4AE5)),
        (1, Tone(0xBAE9FD, 0xFA7387, 0xE86CC5)),
    ]
    /// The strip behind the top of the sheet, from left to right.
    static let stripLeft = Tone(0x9ADFFE, 0xFEBD29, 0x382F92)
    static let stripRight = Tone(0x34C9FD, 0xF97786, 0x8141F3)
    /// The solid line behind the sun: its left end and its end at the sun.
    static let lineStart = Tone(0xFECC03, 0xFFFFFF, 0xFFFFFF)
    static let lineEnd = Tone(0xFD9124, 0xFFFFFF, 0xFFFFFF)
    /// The golden-hour piece at the end of the bow: where it starts and where it ends.
    static let goldStart = Tone(0xFCB370, 0xFEF48B, 0xFEF48B)
    static let goldEnd = Tone(0xFC5093, 0xFEF48B, 0xFEF48B)
    static let sunDisc = Tone(0xFEC804, 0xFEFCB5, 0xFD9135)
    static let sunRing = Tone(0xFEEB9A, 0xFFFFFF, 0xFBD9CA)
    static let cloud = Tone(0xFCFBFE, 0xFEE5E9, 0xFEE5E9)
}

/// All timing goes through here. Launch with `-slow 8` to run everything eight times slower:
/// the recording is then blended back to normal speed, which gives real motion blur.
/// `-warmup 16` waits first, so the home indicator has faded before the flow starts.
enum WeatherSceneMotion {
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

// Example data. It adds up: sunrise 06:12 + 875 minutes of daylight = sunset 20:47.
// Now it is 14:10 (478 minutes after sunrise), the golden hour starts at 19:47 (815 minutes after sunrise).
enum WeatherSceneDay {
    /// What the sky looks like in a column of the row under the panel.
    enum Sky: Sendable {
        case sun, sunCloud, golden, sunset

        var label: String {
            switch self {
            case .sun: return "Sunny"
            case .sunCloud: return "Partly cloudy"
            case .golden: return "Golden hour"
            case .sunset: return "Sunset"
            }
        }
    }

    /// One column of the row under the panel: a moment of the day, from now until sunset.
    struct Hour: Sendable {
        let title: String
        let sky: Sky
        let minute: Int              // minutes after midnight
    }

    static let city = "Munich"
    static let sunrise = 6 * 60 + 12                 // minutes after midnight
    static let sunset = 20 * 60 + 47
    static let nowMinute = 14 * 60 + 10
    static let goldenMinute = 19 * 60 + 47
    static let daylight = Double(sunset - sunrise)   // 875 minutes

    // The sun's position on its bow is one number: 0 = sunrise, 1 = sunset.
    static let now = Double(nowMinute - sunrise) / daylight            // 478 / 875
    static let goldenHour = Double(goldenMinute - sunrise) / daylight  // 815 / 875
    /// From this position on the day sky starts to turn golden.
    static let skyTurns = 0.80

    /// Temperature along the day, linear between these points.
    static let temperatures: [(minute: Int, degrees: Double)] = [
        (sunrise, 12), (9 * 60, 15), (12 * 60, 19), (nowMinute, 21), (15 * 60, 22),
        (16 * 60, 24), (18 * 60, 20), (goldenMinute, 18), (sunset, 17),
    ]
    /// "Feels like" along the day, linear between these points. It stays close to the temperature.
    static let feels: [(minute: Int, degrees: Double)] = [
        (sunrise, 13), (9 * 60, 16), (12 * 60, 21), (nowMinute, 23), (15 * 60, 24),
        (16 * 60, 25), (18 * 60, 20), (goldenMinute, 18), (sunset, 16),
    ]
    static let high = Int(temperatures.map { $0.degrees }.max() ?? 0)
    static let low = Int(temperatures.map { $0.degrees }.min() ?? 0)

    /// The row under the panel: the moments that matter from now until sunset, one column each.
    /// Their temperatures are read from the curve: 21° now, 24° at 16:00 (the high of the day),
    /// 20° at 18:00, 18° at the golden hour and 17° at sunset.
    static let hours = [
        Hour(title: "Now", sky: .sun, minute: nowMinute),
        Hour(title: "16:00", sky: .sun, minute: 16 * 60),
        Hour(title: "18:00", sky: .sunCloud, minute: 18 * 60),
        Hour(title: "Golden", sky: .golden, minute: goldenMinute),
        Hour(title: "Sunset", sky: .sunset, minute: sunset),
    ]

    /// Rounded temperature at a minute of the day.
    static func temperature(atMinute minute: Int) -> Int {
        Int(value(of: temperatures, at: Double(minute)).rounded())
    }

    /// Where the sun's moment lies in the row of columns: 0 = the first column, 1 = the second, and so on,
    /// with fractions in between. Before "now" it stays on the first column, so the tile rests on "Now"
    /// while the sun rises.
    static func column(at t: Double) -> Double {
        let m = minute(at: t)
        guard let first = hours.first, m > Double(first.minute) else { return 0 }
        for i in 1..<hours.count where m <= Double(hours[i].minute) {
            let a = Double(hours[i - 1].minute)
            let b = Double(hours[i].minute)
            return Double(i - 1) + (m - a) / (b - a)
        }
        return Double(hours.count - 1)
    }

    /// Sky phase for a sun position: 0 = day, 1 = golden hour, 2 = dusk.
    /// Day until `skyTurns`, then softly to 1 at the start of the golden hour, then softly to 2 at sunset.
    static func phase(at t: Double) -> Double {
        func ramp(_ a: Double, _ b: Double) -> Double {
            let u = min(max((t - a) / (b - a), 0), 1)
            return u * u * (3 - 2 * u)
        }
        return ramp(skyTurns, goldenHour) + ramp(goldenHour, 1)
    }

    /// Minutes after midnight at sun position `t`.
    static func minute(at t: Double) -> Double {
        Double(sunrise) + min(max(t, 0), 1) * daylight
    }

    /// Rounded temperature at sun position `t`.
    static func temperature(at t: Double) -> Int {
        Int(value(of: temperatures, at: minute(at: t)).rounded())
    }

    /// Rounded "feels like" at sun position `t`.
    static func feelsLike(at t: Double) -> Int {
        Int(value(of: feels, at: minute(at: t)).rounded())
    }

    /// Clock time at sun position `t`, for example "19:47".
    static func time(at t: Double) -> String {
        let m = Int(minute(at: t).rounded())
        return String(format: "%02d:%02d", m / 60, m % 60)
    }

    /// Value on the line through `points` at `minute`. Before the first and after the last point it stays flat.
    private static func value(of points: [(minute: Int, degrees: Double)], at minute: Double) -> Double {
        guard let first = points.first, let last = points.last else { return 0 }
        if minute <= Double(first.minute) { return first.degrees }
        for (a, b) in zip(points, points.dropFirst()) where minute <= Double(b.minute) {
            let f = (minute - Double(a.minute)) / Double(b.minute - a.minute)
            return a.degrees + (b.degrees - a.degrees) * f
        }
        return last.degrees
    }
}

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
@MainActor @Observable
final class WeatherSceneDemo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var round = 0                    // a new number makes the screen come in fresh
    var pressed: String? = nil       // control that is being "tapped": "main"
    var sun: Double = 0              // the sun's position on its bow: 0 = sunrise, 1 = sunset
    var holding = false              // a finger holds the sun
    var reminded = false             // the reminder was set

    /// The big number: the temperature where the sun is.
    var temperature: Int { WeatherSceneDay.temperature(at: sun) }
    /// The finger holds the sun somewhere else than "now": the screen shows a forecast.
    var forecast: Bool { holding && abs(sun - WeatherSceneDay.now) > 0.0005 }
    /// The grey line under the big number. It belongs to the same moment as the number: while the sun rises
    /// it counts up with it, and while a finger holds the sun it names the time of the forecast.
    var line: String {
        if forecast {
            return "Forecast for \(WeatherSceneDay.time(at: sun)) · Feels like \(WeatherSceneDay.feelsLike(at: sun))°"
        }
        return "Feels like \(WeatherSceneDay.feelsLike(at: sun))° · H \(WeatherSceneDay.high)° L \(WeatherSceneDay.low)°"
    }
    /// What the chip says: 0 = day, 1 = golden hour, 2 = sunset.
    private var stage: Int {
        let phase = WeatherSceneDay.phase(at: sun)
        return phase < 0.5 ? 0 : (phase <= 1.5 ? 1 : 2)
    }
    var chip: String { ["Sunny", "Golden hour", "Sunset"][stage] }
    var chipDot: Color { [WeatherSceneInk.sunny, WeatherSceneInk.golden, WeatherSceneInk.sunset][stage] }
    var button: String {
        reminded ? "Reminder set for " + WeatherSceneDay.time(at: WeatherSceneDay.goldenHour) : "Remind me at golden hour"
    }

    private func tap(_ id: String) async throws {
        withAnimation(WeatherSceneMotion.spring(0.2)) { pressed = id }
        try await WeatherSceneMotion.wait(0.17)
        withAnimation(WeatherSceneMotion.spring(0.35, bounce: 0.2)) { pressed = nil }
        try await WeatherSceneMotion.wait(0.08)
    }

    /// Moves the sun along its bow in small steps, slow at both ends.
    private func move(from a: Double, to b: Double, steps: Int, step: Double) async throws {
        for n in 0...steps {
            let u = Double(n) / Double(steps)
            let eased = u * u * (3 - 2 * u)
            withAnimation(WeatherSceneMotion.smooth(0.16)) { sun = a + (b - a) * eased }
            try await WeatherSceneMotion.wait(step)
        }
    }

    func run() async {
        do {
            if WeatherSceneMotion.warmup > 0 { try await Task.sleep(for: .seconds(WeatherSceneMotion.warmup)) }
            while !Task.isCancelled {
                // Reset while the sheet is down.
                pressed = nil; sheetUp = false; sun = 0; holding = false; reminded = false
                round += 1
                try await WeatherSceneMotion.wait(0.12)

                // 1. The sheet slides up, the pieces come in one after the other.
                sheetUp = true
                try await WeatherSceneMotion.wait(0.55)

                // 2. The sun rises along its bow to "now". The big number counts up with it.
                try await move(from: 0, to: WeatherSceneDay.now, steps: 26, step: 0.04)
                try await WeatherSceneMotion.wait(0.75)

                // 3. A finger takes the sun and drags it to the start of the golden hour.
                withAnimation(WeatherSceneMotion.spring(0.3, bounce: 0.2)) { holding = true }
                try await WeatherSceneMotion.wait(0.25)
                try await move(from: WeatherSceneDay.now, to: WeatherSceneDay.goldenHour, steps: 28, step: 0.04)
                try await WeatherSceneMotion.wait(0.8)

                // 4. The finger drags on to sunset: the sky turns to dusk.
                try await move(from: WeatherSceneDay.goldenHour, to: 1, steps: 12, step: 0.04)
                try await WeatherSceneMotion.wait(0.95)

                // 5. The finger lets go. One spring brings the sun back to "now".
                withAnimation(WeatherSceneMotion.spring(0.9, bounce: 0.1)) {
                    holding = false
                    sun = WeatherSceneDay.now
                }
                try await WeatherSceneMotion.wait(1.25)

                // 6. "Remind me at golden hour": the button turns green, the bell pops into the tag.
                try await tap("main")
                reminded = true
                try await WeatherSceneMotion.wait(1.6)

                // 7. The sheet slides away, then everything starts again.
                sheetUp = false
                try await WeatherSceneMotion.wait(0.5)
            }
        } catch {
            // Cancelled: the view went away, so the flow ends here.
        }
    }
}
```

### `WeatherSceneParts.swift`

```swift
import SwiftUI

/// Flat pill button. `pressed` is driven by the demo, with a soft touch dot so the tap is visible in the clip.
/// The title rolls to the new text when it changes; `done` puts a small check in front of it.
/// Switched off, it is pale grey with grey text. With `success` it turns fresh green with white
/// text and a small pop:
/// the answer to an action that worked.
/// With Reduce Motion nothing scales: only the colour and the text change.
struct WeatherScenePillButton: View {
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
        .foregroundStyle(success ? WeatherSceneInk.successText : (enabled ? Color.white : WeatherSceneInk.secondary))
        .frame(maxWidth: .infinity)
        .frame(height: 54)
        .background(Capsule().fill(success ? WeatherSceneInk.successFill : (enabled ? WeatherSceneInk.black : WeatherSceneInk.off)))
        .overlay(
            Circle()
                .fill(Color.white.opacity(0.26))
                .frame(width: 44, height: 44)
                .scaleEffect(pressed || reduceMotion ? 1 : 1.5)
                .opacity(pressed ? 1 : 0)
        )
        .scaleEffect(pressed && !reduceMotion ? 0.965 : 1)
        .keyframeAnimator(initialValue: CGFloat(1), trigger: success) { view, pop in
            view.scaleEffect(pop)
        } keyframes: { _ in
            KeyframeTrack {
                SpringKeyframe(peak, duration: 0.14 * WeatherSceneMotion.k, spring: .snappy(duration: 0.5 * WeatherSceneMotion.k))
                SpringKeyframe(CGFloat(1), duration: 0.45 * WeatherSceneMotion.k, spring: .bouncy(duration: 0.5 * WeatherSceneMotion.k))
            }
        }
        .animation(WeatherSceneMotion.spring(0.6), value: title)
        .animation(WeatherSceneMotion.smooth(0.45), value: enabled)
        .animation(WeatherSceneMotion.smooth(0.35), value: success)
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(title)
        .accessibilityAddTraits(.isButton)
    }
}

/// Element comes in from a soft blur, a little lower and smaller. Used with small delays for a stagger.
/// With Reduce Motion it only fades in.
struct WeatherSceneReveal: ViewModifier {
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
            .animation(WeatherSceneMotion.smooth(0.7, delay: delay), value: on)
    }
}

extension View {
    func weatherSceneReveal(_ on: Bool, delay: Double = 0, y: CGFloat = 14, blur: CGFloat = 10, scale: CGFloat = 0.98) -> some View {
        modifier(WeatherSceneReveal(on: on, delay: delay, y: y, blur: blur, scale: scale))
    }
}

/// Small grey capsule: a status label, not a button. With `dot` a coloured dot sits in front of the text.
struct WeatherSceneStatusChip: View {
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
                .foregroundStyle(WeatherSceneInk.secondary)
                .contentTransition(.numericText())
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(WeatherSceneInk.chip))
    }
}

/// The gradient strip behind the top of the sheet. It follows the sky: the sun's position `t` is its
/// animatable data, so the colours are mixed anew for every value in between.
/// `animatableData` is `nonisolated`: SwiftUI reads and writes it while it animates, and it only touches `t`.
struct WeatherSceneSkyStrip: Animatable {
    var t: Double

    nonisolated var animatableData: Double {
        get { t }
        set { t = newValue }
    }
}

extension WeatherSceneSkyStrip: View {
    var body: some View {
        let phase = WeatherSceneDay.phase(at: t)
        LinearGradient(colors: [WeatherSceneInk.stripLeft.at(phase), WeatherSceneInk.stripRight.at(phase)],
                       startPoint: .leading, endPoint: .trailing)
    }
}

/// The hero: the sky with the sun on its bow. Everything in it is computed from ONE number, the sun's
/// position `t` (0 = sunrise, 1 = sunset). `t` is the animatable data, so SwiftUI hands `body` every
/// value in between: no colour and no position jumps, whether the finger moves the sun in small steps
/// or one spring brings it back.
/// `animatableData` is `nonisolated`: SwiftUI reads and writes it while it animates, and it only touches `t`.
struct WeatherSceneSkyPanel: Animatable {
    var t: Double
    var held = false                 // a finger holds the sun
    var bell = false                 // the reminder is set: a bell sits in the tag
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    nonisolated var animatableData: Double {
        get { t }
        set { t = newValue }
    }

    private static let height: CGFloat = 162
    /// Stars of the dusk sky: place in a panel of 366 points width, and size.
    private static let stars: [(x: CGFloat, y: CGFloat, size: CGFloat)] = [
        (73, 14, 2), (124, 13, 3), (89, 33, 2), (21, 53, 2),
        (203, 22, 2), (216, 12, 2), (284, 41, 3), (340, 41, 2),
    ]

    /// The moon and the stars pop in one after the other and fade together.
    private func pop(_ night: Bool, order: Int) -> Animation {
        night ? WeatherSceneMotion.spring(0.4, bounce: 0.35, delay: 0.05 * Double(order)) : WeatherSceneMotion.smooth(0.3)
    }
}

extension WeatherSceneSkyPanel: View {
    var body: some View {
        let phase = WeatherSceneDay.phase(at: t)
        let golden = WeatherSceneDay.goldenHour
        let night = phase > 1.5                                             // dusk: the moon and the stars are out
        let cloudFade: Double = 1 - min(max((phase - 1) * 2, 0), 1)         // clouds are gone before the moon comes
        let ringFade: Double = 1 - 0.45 * max(1 - abs(phase - 1), 0)        // the ring is translucent at golden hour
        let cloudInk: Color = WeatherSceneInk.cloud.at(phase)
        let bellIn: AnyTransition = reduceMotion ? .opacity : .scale(scale: 0.3).combined(with: .opacity)
        let skyStops: [Gradient.Stop] = WeatherSceneInk.sky.map { Gradient.Stop(color: $0.tone.at(phase), location: $0.at) }
        let spoken: String = "\(WeatherSceneDay.time(at: t)), \(WeatherSceneDay.temperature(at: t))°"

        GeometryReader { g in
            let w: CGFloat = max(g.size.width, 1)
            let sun: CGPoint = WeatherSceneBow.point(at: t, in: g.size)
            let leftX: CGFloat = WeatherSceneBow.point(at: 0, in: g.size).x
            let goldenX: CGFloat = WeatherSceneBow.point(at: golden, in: g.size).x
            let rightX: CGFloat = WeatherSceneBow.point(at: 1, in: g.size).x
            let lineInk = LinearGradient(colors: [WeatherSceneInk.lineStart.at(phase), WeatherSceneInk.lineEnd.at(phase)],
                                         startPoint: UnitPoint(x: leftX / w, y: 0.5),
                                         endPoint: UnitPoint(x: max(sun.x, leftX + 1) / w, y: 0.5))
            let goldInk = LinearGradient(colors: [WeatherSceneInk.goldStart.at(phase), WeatherSceneInk.goldEnd.at(phase)],
                                         startPoint: UnitPoint(x: goldenX / w, y: 0.5),
                                         endPoint: UnitPoint(x: rightX / w, y: 0.5))

            ZStack(alignment: .topLeading) {
                // Two clouds. They fade out as the sky turns to dusk.
                Group {
                    WeatherSceneCloud().fill(cloudInk)
                        .frame(width: 37, height: 22)
                        .offset(x: 20, y: 13)
                    WeatherSceneCloud().fill(cloudInk)
                        .frame(width: 21, height: 13)
                        .offset(x: 87, y: 12)
                }
                .opacity(cloudFade)
                .accessibilityHidden(true)

                // The moon, where the larger cloud was.
                WeatherSceneIcon.Crescent().fill(WeatherSceneInk.moon)
                    .frame(width: 22, height: 22)
                    .scaleEffect(night || reduceMotion ? 1 : 0.4)
                    .opacity(night ? 1 : 0)
                    .offset(x: 27.5, y: 13)
                    .animation(pop(night, order: 0), value: night)
                    .accessibilityHidden(true)

                // The stars.
                ForEach(Self.stars.indices, id: \.self) { i in
                    let star = Self.stars[i]
                    Circle().fill(Color.white)
                        .frame(width: star.size, height: star.size)
                        .scaleEffect(night || reduceMotion ? 1 : 0.2)
                        .opacity(night ? 1 : 0)
                        .position(x: star.x * w / 366, y: star.y)
                        .animation(pop(night, order: i + 1), value: night)
                }
                .accessibilityHidden(true)

                // The bow. The dashes are counted from the golden hour back to the sun, so they stand
                // still while the sun moves. The solid line is drawn last and covers the golden piece
                // as the sun passes.
                Group {
                    WeatherSceneBow(from: golden, to: min(t, golden))
                        .stroke(Color.white.opacity(0.9), style: StrokeStyle(lineWidth: 2, lineCap: .round, dash: [4, 7]))
                    WeatherSceneBow(from: max(t, golden), to: 1)
                        .stroke(goldInk, style: StrokeStyle(lineWidth: 5, lineCap: .round, lineJoin: .round))
                    WeatherSceneBow(from: 0, to: t)
                        .stroke(lineInk, style: StrokeStyle(lineWidth: 5, lineCap: .round, lineJoin: .round))
                }
                .accessibilityHidden(true)

                // The sun: the ring and the disc. While the finger holds it, a wide soft white circle
                // lies around it. It shows the touch without dulling the sun.
                ZStack {
                    Circle().fill(Color.white.opacity(0.3))
                        .frame(width: 62, height: 62)
                        .scaleEffect(held || reduceMotion ? 1 : 0.6)
                        .opacity(held ? 1 : 0)
                    Circle().fill(WeatherSceneInk.sunRing.at(phase).opacity(ringFade))
                        .frame(width: 41, height: 41)
                    Circle().fill(WeatherSceneInk.sunDisc.at(phase))
                        .frame(width: 32, height: 32)
                }
                .position(sun)
            }
        }
        .frame(height: Self.height)
        .overlay(alignment: .topTrailing) {
            // The tag. It is fixed at its right end, so it grows to the left when the bell comes in.
            HStack(spacing: 4) {
                if bell {
                    Image(systemName: "bell.fill")
                        .font(.system(size: 10, weight: .semibold))
                        .foregroundStyle(WeatherSceneInk.bell)
                        .transition(bellIn)
                }
                Text("Golden hour")
                    .foregroundStyle(WeatherSceneInk.secondary)
                Text(WeatherSceneDay.time(at: golden))
                    .fontWeight(.semibold)
                    .foregroundStyle(WeatherSceneInk.primary)
            }
            .font(.system(size: 11.5, weight: .medium))
            .padding(.horizontal, 8)
            .frame(height: 20)
            .background(Capsule().fill(Color.white))
            .padding(.trailing, 7)
            .padding(.top, 10)
            .animation(WeatherSceneMotion.spring(0.5, bounce: 0.3), value: bell)
        }
        .background(LinearGradient(stops: skyStops, startPoint: .top, endPoint: .bottom))
        .clipShape(RoundedRectangle(cornerRadius: 20, style: .continuous))
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Sun position")
        .accessibilityValue("\(spoken), golden hour at \(WeatherSceneDay.time(at: golden))\(bell ? ", reminder set" : "")")
        .accessibilityAddTraits(.updatesFrequently)
    }
}

/// A piece of the sun's path: a parabola from the lower left to the lower right of the panel.
/// `from` and `to` are positions on it (0 = sunrise, 1 = sunset). The piece is drawn from `from` to `to`,
/// also backwards, which decides where a dash pattern starts.
struct WeatherSceneBow: Shape {
    var from: Double = 0
    var to: Double = 1

    /// Point of the bow at position `t` in a panel of this size. `t` is linear in x.
    /// The ends keep 30 and 26 points to the panel's edges, so the whole sun stays inside at sunrise and sunset.
    static func point(at t: Double, in size: CGSize) -> CGPoint {
        let u = CGFloat(min(max(t, 0), 1))
        let base = size.height - 26
        let top = size.height * 0.27
        return CGPoint(x: 30 + (size.width - 60) * u,
                       y: base - (base - top) * 4 * u * (1 - u))
    }

    func path(in rect: CGRect) -> Path {
        var path = Path()
        let a = min(max(from, 0), 1)
        let b = min(max(to, 0), 1)
        guard a != b else { return path }
        let steps = max(2, Int(abs(b - a) * 80))
        for n in 0...steps {
            let p = Self.point(at: a + (b - a) * Double(n) / Double(steps), in: rect.size)
            let q = CGPoint(x: rect.minX + p.x, y: rect.minY + p.y)
            if n == 0 { path.move(to: q) } else { path.addLine(to: q) }
        }
        return path
    }
}

/// A cloud: a flat base with three round bumps.
struct WeatherSceneCloud: Shape {
    func path(in rect: CGRect) -> Path {
        let w = rect.width
        let h = rect.height
        /// A bump: centre in parts of the width and height, radius in parts of the height.
        func bump(_ x: CGFloat, _ y: CGFloat, _ r: CGFloat) -> Path {
            Path(ellipseIn: CGRect(x: rect.minX + x * w - r * h, y: rect.minY + y * h - r * h,
                                   width: 2 * r * h, height: 2 * r * h))
        }
        let base = Path(roundedRect: CGRect(x: rect.minX, y: rect.minY + 0.46 * h, width: w, height: 0.54 * h),
                        cornerRadius: 0.27 * h)
        return base
            .union(bump(0.27, 0.58, 0.31))
            .union(bump(0.52, 0.40, 0.40))
            .union(bump(0.77, 0.60, 0.29))
    }
}

/// The row under the panel: the moments from now until sunset, one column each. A grey tile marks the
/// moment the sun stands for. It rests on "Now", follows the sun between the columns and lands exactly
/// on a column when the sun reaches that moment. The sun's position `t` is the animatable data, so the
/// tile stays in step with the sun, also while one spring brings the sun back.
/// `animatableData` is `nonisolated`: SwiftUI reads and writes it while it animates, and it only touches `t`.
struct WeatherSceneHourRow: Animatable {
    var t: Double
    var shown = true                 // the columns have come in
    var lead: Double = 0             // delay of the first column

    nonisolated var animatableData: Double {
        get { t }
        set { t = newValue }
    }

    private static let height: CGFloat = 82
}

extension WeatherSceneHourRow: View {
    var body: some View {
        let hours = WeatherSceneDay.hours
        let place = CGFloat(WeatherSceneDay.column(at: t))
        GeometryReader { g in
            let w: CGFloat = g.size.width / CGFloat(max(hours.count, 1))
            ZStack(alignment: .topLeading) {
                RoundedRectangle(cornerRadius: 14, style: .continuous)
                    .fill(WeatherSceneInk.tile)
                    .frame(width: w, height: Self.height)
                    .offset(x: w * place)
                    .weatherSceneReveal(shown, delay: lead)
                    .accessibilityHidden(true)
                HStack(spacing: 0) {
                    ForEach(hours.indices, id: \.self) { i in
                        cell(hours[i])
                            .frame(width: w, height: Self.height)
                            .weatherSceneReveal(shown, delay: lead + 0.05 * Double(i))
                    }
                }
            }
        }
        .frame(height: Self.height)
    }

    private func cell(_ hour: WeatherSceneDay.Hour) -> some View {
        VStack(spacing: 4) {
            Text(hour.title)
                .font(.system(size: 15, weight: .medium))
                .foregroundStyle(WeatherSceneInk.secondary)
            WeatherSceneIcon(sky: hour.sky)
            Text("\(WeatherSceneDay.temperature(atMinute: hour.minute))°")
                .font(.system(size: 19, weight: .semibold))
                .monospacedDigit()
                .foregroundStyle(WeatherSceneInk.primary)
        }
        .accessibilityElement(children: .combine)
    }
}

/// Small weather icon of a column, drawn from plain shapes.
struct WeatherSceneIcon: View {
    let sky: WeatherSceneDay.Sky
    var size: CGFloat = 26

    var body: some View {
        ZStack {
            switch sky {
            case .sun:
                Circle().fill(WeatherSceneInk.sunny)
                    .frame(width: size * 0.96, height: size * 0.96)
            case .sunCloud:
                Circle().fill(WeatherSceneInk.sunny)
                    .frame(width: size * 0.8, height: size * 0.8)
                    .offset(x: -size * 0.36, y: -size * 0.2)
                WeatherSceneCloud().fill(WeatherSceneInk.iconCloud)
                    .frame(width: size * 1.42, height: size * 0.78)
                    .offset(x: size * 0.04, y: size * 0.14)
            case .golden:
                // The low sun of the golden hour: an orange disc just above the horizon.
                Circle().fill(WeatherSceneInk.golden)
                    .frame(width: size * 0.78, height: size * 0.78)
                    .offset(y: -size * 0.1)
                Capsule().fill(WeatherSceneInk.golden.opacity(0.45))
                    .frame(width: size * 1.3, height: 3)
                    .offset(y: size * 0.44)
            case .sunset:
                // The sun half below the horizon.
                Circle().trim(from: 0.5, to: 1).fill(WeatherSceneInk.sunset)
                    .frame(width: size * 0.9, height: size * 0.9)
                    .offset(y: size * 0.2)
                Capsule().fill(WeatherSceneInk.sunset.opacity(0.45))
                    .frame(width: size * 1.3, height: 3)
                    .offset(y: size * 0.26)
            }
        }
        .frame(width: size * 1.5, height: size)
        .accessibilityElement(children: .ignore)
        .accessibilityLabel(sky.label)
    }

    /// Crescent moon: a disc with a second disc cut out of its upper right.
    struct Crescent: Shape {
        func path(in rect: CGRect) -> Path {
            let r = min(rect.width, rect.height) / 2
            let disc = Path(ellipseIn: CGRect(x: rect.midX - r, y: rect.midY - r, width: 2 * r, height: 2 * r))
            let cut = Path(ellipseIn: CGRect(x: rect.midX - 0.49 * r, y: rect.midY - 1.05 * r,
                                             width: 1.7 * r, height: 1.7 * r))
            return disc.subtracting(cut)
        }
    }
}
```

### `WeatherSceneView.swift`

```swift
import SwiftUI

/// The weather screen. There is only this one screen; the sun's position changes its state:
/// the sky changes between day, golden hour and dusk, the big number follows the sun.
/// Font sizes and heights are fixed numbers on purpose: this is a fixed showcase for a recording,
/// not a screen that follows Dynamic Type.
struct WeatherSceneView: View {
    let demo: WeatherSceneDemo
    @State private var shown = false

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text(WeatherSceneDay.city)
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(WeatherSceneInk.secondary)
                Spacer()
                WeatherSceneStatusChip(text: demo.chip, dot: demo.chipDot)
                    .animation(WeatherSceneMotion.spring(0.5), value: demo.chip)
            }
            .padding(.leading, 4)
            .frame(height: 32)
            .weatherSceneReveal(shown, delay: lead)

            // The line box of such a large font is much taller than its digits.
            // The frame keeps only the height of the digits, so the next line can sit close.
            Text("\(demo.temperature)°")
                .font(.system(size: 104, weight: .bold))
                .tracking(-3)
                .foregroundStyle(WeatherSceneInk.primary)
                .contentTransition(.numericText(value: Double(demo.temperature)))
                .fixedSize()
                .frame(height: 78)
                .padding(.top, 18)
                .animation(WeatherSceneMotion.spring(0.4), value: demo.temperature)
                .weatherSceneReveal(shown, delay: lead + 0.06)

            Text(demo.line)
                .font(.system(size: 18))
                .monospacedDigit()
                .foregroundStyle(WeatherSceneInk.secondary)
                .contentTransition(.numericText())
                .lineLimit(1)
                .padding(.leading, 4)
                .padding(.top, 8)
                .animation(WeatherSceneMotion.spring(0.5), value: demo.forecast)
                .weatherSceneReveal(shown, delay: lead + 0.1)

            WeatherSceneSkyPanel(t: demo.sun, held: demo.holding, bell: demo.reminded)
                .padding(.top, 22)
                .weatherSceneReveal(shown, delay: lead + 0.16)

            HStack(alignment: .top) {
                label("Sunrise", WeatherSceneDay.time(at: 0), .leading)
                Spacer()
                label("Sunset", WeatherSceneDay.time(at: 1), .trailing)
            }
            .padding(.horizontal, 5)
            .padding(.top, 10)
            .weatherSceneReveal(shown, delay: lead + 0.22)

            WeatherSceneHourRow(t: demo.sun, shown: shown, lead: lead + 0.28)
                .padding(.top, 14)
        }
        .onAppear { shown = true }
    }

    private func label(_ word: String, _ time: String, _ alignment: HorizontalAlignment) -> some View {
        VStack(alignment: alignment, spacing: 0) {
            Text(word)
                .font(.system(size: 15, weight: .medium))
                .foregroundStyle(WeatherSceneInk.secondary)
            Text(time)
                .font(.system(size: 17, weight: .semibold))
                .monospacedDigit()
                .foregroundStyle(WeatherSceneInk.primary)
        }
        .accessibilityElement(children: .combine)
    }
}
```

### `WeatherSceneRootView.swift`

```swift
import SwiftUI

/// The whole scene: the sky strip, the white sheet with the weather screen, and the pill button.
/// Show this view to see the scene. It starts the flow by itself and stops it when it goes away.
struct WeatherSceneRootView: View {
    @State private var demo = WeatherSceneDemo()
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    /// Height of the white sheet. The clip later shows only this lower part of the screen.
    private let sheetHeight: CGFloat = 640

    var body: some View {
        ZStack(alignment: .bottom) {
            // The strip behind the sheet has the colours of the sky in the panel.
            WeatherSceneSkyStrip(t: demo.sun)
                .ignoresSafeArea()
                .accessibilityHidden(true)

            VStack(spacing: 0) {
                WeatherSceneView(demo: demo)
                    .id(demo.round)
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // Black = an action to take. After the reminder was set it turns green: it worked.
                WeatherScenePillButton(title: demo.button, success: demo.reminded, done: demo.reminded,
                                  pressed: demo.pressed == "main")
                    .weatherSceneReveal(demo.sheetUp, delay: demo.sheetUp ? 0.5 : 0)
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
            .animation(reduceMotion ? WeatherSceneMotion.smooth(0.35) : WeatherSceneMotion.spring(0.85, bounce: 0.08),
                       value: demo.sheetUp)
        }
        .ignoresSafeArea(edges: .bottom)
        .environment(\.colorScheme, .light)   // the scene is drawn for light mode only
        .persistentSystemOverlays(.hidden)   // no home indicator line in the recording
        .task { await demo.run() }
    }
}
```

### `WeatherSceneApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
// This file is nothing but the app's entry point. A project that already has one leaves this file out
// and shows `WeatherSceneRootView()` itself.
@main
struct WeatherSceneApp: App {
    var body: some Scene {
        WindowGroup {
            WeatherSceneRootView()
                .preferredColorScheme(.light)
                .statusBarHidden(true)
        }
    }
}
```
