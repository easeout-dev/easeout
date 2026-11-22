# Easeout scene: Voice note (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

A voice note app inside a white sheet over a peach-to-coral gradient. It plays in five beats:

1. **Recording.** The sheet slides up and the content comes in piece by piece. A large timer counts. The waveform grows bar by bar: the newest bars are red and fade to black, a red line marks the current position, the part not recorded yet is a row of short grey ticks. In the grey card the transcript appears word by word; new words are grey first, then turn black.
2. **Stop.** A tap on the black pill button "Stop and summarize": the content melts into a blur and the finished note comes in. Its play button pops in, a calm waveform rises in a wave, and the three summary lines appear one after the other.
3. **Action items.** A tap on the second pill: a black capsule slides from "Summary" to "Action items" and the card swaps to a checklist, row by row.
4. **Tick.** The two open tasks get tapped one after the other: each time the black disc pops in, the check draws itself, and a line runs through the task.
5. **Share.** A tap on "Share notes": the title rolls to "Link copied" with a small check. Then the sheet slides away and it starts again.

`Demo` plays these beats by itself in a loop, so I can see the motion without touching anything.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files.
- Create the five files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, durations, curves and delays. These numbers are the design.
- Screens change in one way only: the old content leaves with the `blurOut` transition and the new content comes in with staggered `.reveal`. Do not replace this with slides or plain opacity fades.
- The only shared element is the black selection capsule in `TabPills`. Do not add more `matchedGeometryEffect`.
- Labels and buttons look different on purpose. `StatusChip` (pale grey, grey text) is a label. `TabPills` (white, clear outline, near-black text) and `PillButton` (black) are buttons. Do not restyle one as the other.
- Keep the order of the views in `RootView` and keep `.id(demo.round)` on `RecordingView`; it makes the screen come in fresh each time.
- Keep all timing inside `Motion`. Do not write durations in other places.
- If my project already has an `@main` app, do not add a second one: leave out `VoiceApp` and show `RootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- If a type name collides with one in my project (`Ink`, `Motion`, `Demo`, `Phase`, `Note`, `Levels`), rename ours with the prefix `Voice` and tell me.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to rename.
3. Then ask me one question: "Should I connect this to a real recording?" If yes, remove the auto-play (`demo.run()` and the timeline in `Demo`), feed `bars` and `Levels` from the microphone level, `words` and `firm` from the speech recogniser, and `seconds` from the recorder. Ask me before you add microphone or speech permissions to the project. Keep every animation as it is.

## What I may ask you to change

- The example note: all texts live in `Note` in `Model.swift`.
- The colours: the gradient is `Ink.peach` to `Ink.coral`, the live colour of the waveform is `Ink.red` (see `Ink.wave(age:)`).
- The number of bars: the count of `Levels.all`.

## Check yourself before you answer

- While recording, exactly the last three visible words are grey.
- The red line always stands right after the newest bar.
- Between two screens the sheet is almost empty for a moment: old content gone, new content not yet there.
- The black button never moves. Only its title changes.
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
    static let faint = Color(red: 0.667, green: 0.667, blue: 0.706)
    static let outline = Color(red: 0.831, green: 0.831, blue: 0.847)
    static let fill = Color(red: 0.957, green: 0.957, blue: 0.961)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let tickOff = Color(red: 0.855, green: 0.855, blue: 0.863)
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let peach = Color(red: 1.0, green: 0.788, blue: 0.659)
    static let coral = Color(red: 1.0, green: 0.42, blue: 0.341)
    static let red = Color(red: 0.937, green: 0.196, blue: 0.196)
    static let green = Color(red: 0.204, green: 0.78, blue: 0.349)

    /// Colour of a waveform bar. The newest bar is red, the eight newest fade to near black.
    static func wave(age: Int) -> Color {
        let t = min(max(Double(age) / 8, 0), 1)
        return Color(red: 0.937 + (0.067 - 0.937) * t, green: 0.196 + (0.067 - 0.196) * t, blue: 0.196 + (0.075 - 0.196) * t)
    }
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

// Example data.
enum Note {
    static let title = "Product sync"
    static let meta = "0:42 · 3 speakers"
    /// The spoken sentence, already broken into the three lines of the card.
    static let lines: [[String]] = [
        ["Let's", "ship", "the", "new", "onboarding"],
        ["on", "Friday", "and", "move", "the", "pricing"],
        ["test", "to", "next", "week."],
    ]
    static let wordCount = lines.reduce(0) { $0 + $1.count }
    /// Number of the word at line `l`, position `w`, counted through the whole sentence.
    static func index(_ l: Int, _ w: Int) -> Int { lines[..<l].reduce(0) { $0 + $1.count } + w }

    static let tabs = ["Summary", "Action items", "Decisions"]
    static let summary = ["Onboarding ships Friday.", "Pricing test moves to next week.",
                          "Mia shares the new designs tomorrow."]
    static let tasks: [(title: String, note: String)] = [
        ("Ship onboarding", "Leo · Friday"), ("Move pricing test", "You · next week"),
        ("Share new designs", "Mia · tomorrow"),
    ]
}

enum Phase: Equatable { case recording, note }

/// Plays the whole flow by itself, so the simulator can be recorded without touching it.
/// Screens change like this: the old content melts into a blur, the new content comes in piece by piece.
@MainActor @Observable
final class Demo {
    var sheetUp = false              // the white sheet slides up from the bottom
    var phase: Phase = .recording
    var round = 0                    // a new number makes the recording screen come in fresh
    var pressed: String? = nil       // control that is being "tapped": "main", "tab", "task1" or "task2"
    var button = "Stop and summarize"
    var copied = false               // small check in the button

    // Recording screen
    var seconds = 39                 // timer
    var bars = 0                     // recorded waveform bars
    var barsFrom = 0                 // bars before the last change, where the wave starts
    var live = false                 // the red line is shown and moves
    var words = 0                    // words of the transcript that are visible
    var firm = 0                     // words that have turned from grey to black

    // Note screen
    var lines = 0                    // visible summary lines
    var tab = 0                      // selected pill
    var checked = [true, false, false]

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
            phase = .recording; round += 1
            button = "Stop and summarize"; copied = false
            seconds = 39; bars = 0; barsFrom = 0; live = false; words = 0; firm = 0
            lines = 0; tab = 0; checked = [true, false, false]
            await Motion.wait(0.12)

            // 1. The sheet slides up. The first bars rise in a wave, then the recording runs:
            //    one bar after the other, the timer counts, the words appear.
            sheetUp = true
            await Motion.wait(0.35)
            barsFrom = 0; bars = 20
            await Motion.wait(0.35)
            live = true
            for step in 0..<30 {
                barsFrom = bars; bars += 1
                words = min(Note.wordCount, 1 + step / 2)
                firm = max(0, words - 3)                 // the last three words are still grey
                seconds = 39 + Int(0.7 + Double(step) * 0.11)
                await Motion.wait(0.11)
            }

            // 2. Stop: the note comes in, its summary lines appear one after the other.
            await tap("main")
            live = false
            button = "Share notes"
            withAnimation(Motion.smooth(0.5)) { phase = .note }
            await Motion.wait(0.8)
            for n in 1...Note.summary.count {
                lines = n
                await Motion.wait(0.28)
            }
            await Motion.wait(0.75)

            // 3. Action items: the black pill slides over, the card swaps to the checklist.
            await tap("tab")
            withAnimation(Motion.spring(0.5, bounce: 0.18)) { tab = 1 }
            await Motion.wait(0.95)

            // 4. The two open tasks get ticked, one after the other.
            await tap("task1")
            checked[1] = true
            await Motion.wait(0.5)
            await tap("task2")
            checked[2] = true
            await Motion.wait(0.8)

            // 5. Share, then the sheet slides away and everything starts again.
            await tap("main")
            copied = true
            button = "Link copied"
            await Motion.wait(0.85)
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

/// Small grey capsule with a coloured dot: a status label, not a button. With `pulse` the dot breathes.
struct StatusChip: View {
    let text: String
    let color: Color
    var pulse = false

    var body: some View {
        HStack(spacing: 8) {
            Circle().fill(color).frame(width: 10, height: 10)
                .phaseAnimator([1.0, pulse ? 0.3 : 1.0]) { dot, level in
                    dot.opacity(level)
                } animation: { _ in .easeInOut(duration: 0.6 * Motion.k) }
            Text(text)
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(Ink.secondary)
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(Ink.chip))
    }
}

/// Soft dot that shows where the demo "taps".
struct TouchDot: View {
    let on: Bool
    var light = false

    var body: some View {
        Circle()
            .fill(light ? Color.white.opacity(0.26) : Color.black.opacity(0.12))
            .frame(width: 40, height: 40)
            .scaleEffect(on ? 1 : 1.5)
            .opacity(on ? 1 : 0)
    }
}

/// Row of outline pill buttons. One can be selected: a black capsule slides behind its title.
struct TabPills: View {
    let titles: [String]
    var selected: Int? = nil
    var pressed: Int? = nil
    @Namespace private var ns

    var body: some View {
        HStack(spacing: 8) {
            ForEach(0..<titles.count, id: \.self) { i in
                let on = selected == i
                Text(titles[i])
                    .font(.system(size: 16, weight: .medium))
                    .foregroundStyle(on ? Color.white : Ink.primary)
                    .lineLimit(1)
                    .frame(maxWidth: .infinity)
                    .frame(height: 40)
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
                    .scaleEffect(pressed == i ? 0.96 : 1)
            }
        }
    }
}

/// Loudness of each bar, 0...1. Fixed, so the recording looks the same every time.
enum Levels {
    static let all: [CGFloat] = (0..<56).map { i in
        let peaks: [Double] = [0.5, 0.95, 0.6, 1.0, 0.45, 0.8, 0.55, 0.9, 0.65]   // how loud each "word" gets
        let x = Double(i) * 0.5 + 0.3                                             // one word every six bars or so
        let word = Int(x / .pi)
        let shape = pow(abs(sin(x)), 0.8)
        let jitter = Double((i * 37 + 11) % 13) / 13 * 0.18                       // small irregular part
        return CGFloat(min(1, 0.1 + peaks[word % peaks.count] * shape * 0.82 + jitter))
    }
}

/// Live waveform. Recorded bars stand up one after the other; the newest are red and fade to black.
/// A red line marks the current position, the part not recorded yet is a row of short grey ticks.
struct Waveform: View {
    let recorded: Int
    var from = 0                   // bars before the last change, where the wave starts
    var live = true

    private let slots = Levels.all.count
    private let barWidth: CGFloat = 2.6
    private let height: CGFloat = 72

    var body: some View {
        HStack(alignment: .center, spacing: 0) {
            ForEach(0..<slots, id: \.self) { i in
                let on = i < recorded
                Capsule()
                    .fill(on ? Ink.wave(age: live ? recorded - 1 - i : 99) : Ink.tickOff)
                    .frame(width: barWidth, height: on ? max(5, Levels.all[i] * 64) : 5)
                    .animation(Motion.spring(0.45, bounce: 0.35, delay: Double(max(0, i - from)) * 0.016), value: on)
                    .animation(Motion.smooth(0.3), value: recorded)
                    .animation(Motion.smooth(0.3), value: live)
                if i < slots - 1 { Spacer(minLength: 0) }
            }
        }
        .frame(height: height)
        .overlay {
            GeometryReader { g in
                let pitch = (g.size.width - barWidth) / CGFloat(slots - 1)
                Capsule().fill(Ink.red)
                    .frame(width: 2, height: height)
                    .position(x: (CGFloat(recorded) - 0.5) * pitch + barWidth / 2, y: g.size.height / 2)
                    .opacity(live && recorded > 0 ? 1 : 0)
                    .animation(Motion.smooth(0.22), value: recorded)
                    .animation(Motion.smooth(0.3), value: live)
            }
        }
    }
}

/// Calm waveform next to the play button. Its bars rise in a quick wave from left to right.
struct MiniWave: View {
    var shown = true
    var delay: Double = 0

    var body: some View {
        HStack(alignment: .center, spacing: 0) {
            ForEach(0..<46, id: \.self) { i in
                Capsule()
                    .fill(Ink.primary)
                    .frame(width: 2.4, height: shown ? max(4, Levels.all[i + 5] * 30) : 3)
                    .opacity(shown ? 1 : 0)
                    .animation(Motion.spring(0.5, bounce: 0.3, delay: delay + Double(i) * 0.012), value: shown)
                if i < 45 { Spacer(minLength: 0) }
            }
        }
        .frame(height: 34)
    }
}

/// Round checkbox. When it gets checked the black disc pops in and the check draws itself.
struct CheckCircle: View {
    let done: Bool

    var body: some View {
        ZStack {
            Circle().fill(Color.white)
            Circle().strokeBorder(Ink.outline, lineWidth: 1.8)
            Circle().fill(Ink.black)
                .scaleEffect(done ? 1 : 0.5)
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
        .animation(Motion.spring(0.45, bounce: 0.35), value: done)
    }
}

/// One task of the checklist. When done, the title turns grey and a line runs through it from the left.
struct TaskRow: View {
    let title: String
    let note: String
    let done: Bool
    var pressed = false

    var body: some View {
        HStack(spacing: 12) {
            CheckCircle(done: done)
                .overlay(TouchDot(on: pressed))
                .scaleEffect(pressed ? 0.92 : 1)
            Text(title)
                .font(.system(size: 17.5, weight: .medium))
                .foregroundStyle(done ? Ink.secondary : Ink.primary)
                .lineLimit(1)
                .layoutPriority(1)
                .overlay(alignment: .leading) {
                    Rectangle().fill(Ink.secondary)
                        .frame(height: 1.5)
                        .scaleEffect(x: done ? 1 : 0, anchor: .leading)
                }
                .animation(Motion.smooth(0.45, delay: 0.08), value: done)
            Spacer(minLength: 8)
            Text(note)
                .font(.system(size: 15))
                .lineLimit(1)
                .foregroundStyle(Ink.secondary)
        }
    }
}
```

### `RecordingView.swift`

```swift
import SwiftUI

/// The recording screen. While it records, the waveform grows bar by bar, the timer counts
/// and the transcript appears word by word (new words are grey first, then turn black).
struct RecordingView: View {
    let demo: Demo
    @State private var shown = false

    /// The sheet is still sliding up, so the pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("New note")
                    .font(.system(size: 17, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                StatusChip(text: "Recording", color: Ink.red, pulse: true)
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            Text("0:\(demo.seconds)")
                .font(.system(size: 72, weight: .heavy))
                .tracking(-2)
                .monospacedDigit()
                .foregroundStyle(Ink.primary)
                .contentTransition(.numericText(value: Double(demo.seconds)))
                .animation(Motion.spring(0.6), value: demo.seconds)
                .padding(.top, 20)
                .reveal(shown, delay: lead + 0.06)

            Text(Note.title)
                .font(.system(size: 20))
                .foregroundStyle(Ink.secondary)
                .padding(.leading, 4)
                .reveal(shown, delay: lead + 0.1)

            Waveform(recorded: demo.bars, from: demo.barsFrom, live: demo.live)
                .padding(.horizontal, 4)
                .padding(.top, 26)
                .reveal(shown, delay: lead + 0.16)

            transcript
                .padding(.top, 28)
                .reveal(shown, delay: lead + 0.22)

            TabPills(titles: Note.tabs)
                .padding(.top, 18)
                .reveal(shown, delay: lead + 0.28)
        }
        .onAppear { shown = true }
    }

    private var transcript: some View {
        VStack(alignment: .leading, spacing: 10) {
            Text("Live transcript")
                .font(.system(size: 15))
                .foregroundStyle(Ink.secondary)
            VStack(alignment: .leading, spacing: 5) {
                ForEach(0..<Note.lines.count, id: \.self) { l in
                    HStack(spacing: 5) {
                        ForEach(0..<Note.lines[l].count, id: \.self) { w in
                            let n = Note.index(l, w)
                            Text(Note.lines[l][w])
                                .font(.system(size: 20, weight: .medium))
                                .foregroundStyle(n < demo.firm ? Ink.primary : Ink.faint)
                                .animation(Motion.smooth(0.4), value: n < demo.firm)
                                .reveal(n < demo.words, y: 6, blur: 6)
                        }
                    }
                }
            }
        }
        .padding(.horizontal, 16)
        .padding(.vertical, 16)
        .frame(maxWidth: .infinity, alignment: .leading)
        .background(RoundedRectangle(cornerRadius: 24, style: .continuous).fill(Ink.fill))
    }
}
```

### `NoteView.swift`

```swift
import SwiftUI

/// The finished note. The play button pops in, the waveform rises in a wave, and the summary
/// appears line by line. The pills switch what the grey card shows.
struct NoteView: View {
    let demo: Demo
    @State private var shown = false

    /// The old screen is still melting away, so the new pieces wait a moment.
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text("Today, 9:41")
                    .font(.system(size: 17, weight: .medium))
                    .foregroundStyle(Ink.secondary)
                Spacer()
                StatusChip(text: "Saved", color: Ink.green)
            }
            .padding(.horizontal, 4)
            .reveal(shown, delay: lead)

            VStack(alignment: .leading, spacing: 3) {
                Text(Note.title)
                    .font(.system(size: 34, weight: .bold))
                    .foregroundStyle(Ink.primary)
                Text(Note.meta)
                    .font(.system(size: 19))
                    .foregroundStyle(Ink.secondary)
            }
            .padding(.leading, 4)
            .padding(.top, 32)
            .reveal(shown, delay: lead + 0.06)

            HStack(spacing: 14) {
                Image(systemName: "play.fill")
                    .font(.system(size: 18))
                    .foregroundStyle(Color.white)
                    .offset(x: 1.5)
                    .frame(width: 48, height: 48)
                    .background(Circle().fill(Ink.black))
                    .reveal(shown, delay: lead + 0.14, y: 0, scale: 0.6)
                MiniWave(shown: shown, delay: lead + 0.2)
            }
            .padding(.top, 38)

            TabPills(titles: Note.tabs, selected: demo.tab, pressed: demo.pressed == "tab" ? 1 : nil)
                .padding(.top, 34)
                .reveal(shown, delay: lead + 0.26)

            ZStack(alignment: .topLeading) {
                if demo.tab == 0 {
                    summary.transition(swap)
                } else {
                    TaskList(demo: demo).transition(swap)
                }
            }
            .padding(.horizontal, 16)
            .padding(.vertical, 22)
            .frame(maxWidth: .infinity, minHeight: 150, alignment: .topLeading)
            .background(RoundedRectangle(cornerRadius: 24, style: .continuous).fill(Ink.fill))
            .animation(Motion.smooth(0.5), value: demo.tab)
            .padding(.top, 26)
            .reveal(shown, delay: lead + 0.32)
        }
        .onAppear { shown = true }
    }

    private var swap: AnyTransition {
        .asymmetric(insertion: .modifier(active: Swap(y: 18), identity: Swap(y: 0)),
                    removal: .modifier(active: Swap(y: -18), identity: Swap(y: 0)))
    }

    private var summary: some View {
        VStack(alignment: .leading, spacing: 17) {
            ForEach(0..<Note.summary.count, id: \.self) { i in
                HStack(spacing: 12) {
                    Circle().fill(Ink.primary).frame(width: 6, height: 6)
                    Text(Note.summary[i])
                        .font(.system(size: 18.5, weight: .medium))
                        .foregroundStyle(Ink.primary)
                        .lineLimit(1)
                        .minimumScaleFactor(0.85)
                }
                .reveal(i < demo.lines, y: 8, blur: 8)
            }
        }
    }
}

/// The checklist inside the grey card. Its rows come in one after the other.
struct TaskList: View {
    let demo: Demo
    @State private var shown = false

    var body: some View {
        VStack(spacing: 17) {
            ForEach(0..<Note.tasks.count, id: \.self) { i in
                TaskRow(title: Note.tasks[i].title, note: Note.tasks[i].note, done: demo.checked[i],
                        pressed: demo.pressed == "task\(i)")
                    .reveal(shown, delay: 0.12 + Double(i) * 0.08, y: 10, blur: 8)
            }
        }
        .onAppear { shown = true }
    }
}
```

### `VoiceApp.swift`

```swift
import SwiftUI

// Scene for a design clip. Not a shipping app: it plays one flow by itself in a loop.
@main
struct VoiceApp: App {
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
            LinearGradient(colors: [Ink.peach, Ink.coral], startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()

            VStack(spacing: 0) {
                ZStack(alignment: .top) {
                    switch demo.phase {
                    case .recording:
                        RecordingView(demo: demo)
                            .id(demo.round)
                            .transition(AnyTransition.blurOut)
                    case .note:
                        NoteView(demo: demo)
                            .transition(AnyTransition.blurOut)
                    }
                }
                .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)

                // The button stays in place on every screen, only its title changes.
                PillButton(title: demo.button, done: demo.copied, pressed: demo.pressed == "main")
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
