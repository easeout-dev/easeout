# Easeout scene: Workout, a live timer and heart rate (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

Three screens inside a white sheet over a coloured strip: a workout tracker for one week, one live workout and the streak that follows it. The hero of the live screen is the heart rate, drawn large: the current beats per minute, one bar per minute of the workout and the target zone dashed behind them. Everything that shows minutes, counts or the streak follows one source: the plan in `WorkoutModel.plan`, what is logged in `logged`, and the seconds in `elapsed`. The strip takes the colour of what the screen is about. It plays in five beats:

1. **This week.** The white sheet slides up over the strip and the content comes in piece by piece, top to bottom. Three badges drop into place out of a blur, each on a thin line above its day: Run on Monday 9 with a small green check (already done), Strength on Wednesday 11 (today, in a black circle, the larger 68 pt badge because it is next), Yoga on Friday 13. The minutes roll from 0 to 32 next to the chip "1 of 3 done". Below: "Today", Strength, Upper body, 45 min. The strip is orange, the colour of Strength. Button: "Start strength".
2. **Live.** A press on the button: the week melts into a blur and the live screen comes in. The strip turns red. "Strength" with a "Live" chip whose red dot breathes, the timer 0:00 next to "of 45 min", the heart card with 88 bpm, a beating heart and "Target 120–150", and "Goal 0 of 45 min" with an empty red bar. Button: "Finish workout".
3. **The workout.** The demo fast-forwards: once a second the timer jumps five minutes, nine times, up to 45:00. The digits roll, five new bars grow from the baseline each time (the newest red, the older ones fading from near black to grey), the bpm rolls, and the goal bar and "… of 45 min" fill.
4. **Finished.** A press on "Finish workout": the button turns green with a small pop and a check, "Finished", and the chip says "Done" with a green dot that stops breathing. 1.5 s later the result comes in: the strip turns green, a black circle pops in and its check writes itself, "Your streak" rolls from 11 to 12, "workouts in a row", and one chip per workout: Run "32 min", Strength "45 min", and a muted Yoga "Friday". Button: "Done".
5. **Back to the week.** A press on "Done": the week comes back. The strip turns violet (Yoga is next), the minutes roll from 32 to 77, the chip says "2 of 3 done", Strength now has its green check, Yoga is the larger badge, "Next up" shows Yoga, Slow flow, 30 min, and the button is pale grey and switched off: "Done for today".

`WorkoutRootView(playsDemo: true)` plays these beats once by itself when it appears (`runDemo()`), showing each press on the button with a small shrink and a soft dot, so I can see the motion without touching anything. Taps are blocked while the demo plays. After it, the screen works with real taps, and the live timer counts one second per second.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages, no image files. The workout badges are SF Symbols.
- Create the eight files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, badge sizes, bar widths, the heart-rate scale and target zone, durations, curves and delays. These numbers are the design.
- The minutes, the chip "1 of 3 done", the green checks, the larger badge, the "Today" or "Next up" row, the timer, the bars, the bpm, the goal row, the streak, the result chips, the strip colour and the button title are all computed from `WorkoutModel`. Never write one of them by hand, and never let one of them lag behind the others.
- Screens change in one way: the old screen leaves with `AnyTransition.easeBlurOut` and the new one brings its own staggered `.easeReveal`. No `matchedGeometryEffect`, no slides, no plain fades between screens.
- New bars grow from the baseline (`AnyTransition.workoutGrow`). Do not replace this with a fade or a scale from the centre.
- Keep `.contentTransition(.numericText(value:))` on the minutes, the timer, the bpm, the goal and the streak, so the digits roll.
- The pill button `EasePillButton` has three looks: black = an action to take, pale grey = switched off, green with a check = the action worked. Keep all three. `EaseStatusChip` ("Live", "Done") and the grey capsules ("1 of 3 done", "of 45 min", the result chips) are labels, not buttons. Do not restyle one as the other.
- Keep all timing inside `EaseMotion`. Do not write durations in other places. Launching with the argument `-slow 4` plays everything four times slower; keep that.
- The files starting with `Ease` are the shared Easeout kit. If my project already declares any of these types (`EaseMotion`, `EaseInk`, `EaseBrand`, `EaseStrip`, `EaseReveal`, `EaseBlurOut`, `EaseTouchDot`, `EasePressStyle`, `EasePressBody`, `EasePillButton`, `EaseStatusChip`, `EasePop`, `EaseLogoBadge`, `EaseCheckShape`, `EaseSheet`, the `easeReveal` and `easePop` modifiers or `AnyTransition.easeBlurOut`), for example from another Easeout scene or from the Easeout skill, keep one copy and do not add a second: use the declaration that is already there and add to it only the members this scene needs and it lacks (for example `EaseInk.live`, `EaseBrand.violet` or `EaseStrip.from(_:)`). If a member exists in both with a different value, keep mine and tell me which one.
- The scene's own types start with `Workout`. A fitness app may already have a `WorkoutModel`, `WorkoutKind` or `WorkoutPlan`: if one of them collides with a type in my project, rename ours with the prefix `WorkoutScene` everywhere and tell me.
- If my project already has an `@main` app, do not add a second one: leave out the file `WorkoutApp.swift` (it holds nothing but the entry point), keep the other seven files unchanged, and show `WorkoutRootView(playsDemo: true)` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- The scene respects Reduce Motion (things only fade, the sheet does not slide) and stops every waiting step when the view goes away. Keep both.
- Light mode only; `EaseSheet` sets the light colour scheme for its own content.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to change.
3. Then ask me one question: "Should I connect this to your real workout data?" If yes, show `WorkoutRootView(playsDemo: false)` and remove the demo (`runDemo()`, `demoStep` and the `pressed` state with what reads it), fill the plan, the logged minutes and the streak from my model, feed the heart rate from my source in place of the example curve `WorkoutModel.bpm(atMinute:)`, and save the workout when "Finish workout" is pressed. Keep every animation as it is.

## What I may ask you to change

- The example week: the workouts, their days, minutes and details live in `WorkoutModel.plan`; today, the day letters and numbers, and the streak before today sit right below it.
- The heart-rate example and the target zone: `WorkoutModel.bpm(atMinute:)` and `low`, `high`, `zoneLow`, `zoneHigh` in `WorkoutHeartCard`.
- The colour of each workout: `WorkoutKind.color`. The strip uses it while that workout is next; red is the live screen, green the result.

## Check yourself before you answer

- With the example data the week shows 32 min and "1 of 3 done" before the workout, 77 min and "2 of 3 done" after it; 32 + 45 = 77.
- The live screen starts at 0:00 and 88 bpm and ends at 45:00, "45 of 45 min", with a full goal bar and 136 bpm. The heart card draws one bar per minute, as many of the newest as fit its width.
- The streak goes from 11 to 12 only after the workout is finished.
- The button is black while there is something to do, green after "Finish workout" is pressed, and pale grey once today's workout is logged.
- The strip is orange on the first week screen, red while live, green on the result and violet on the week after it.

## The code

### `WorkoutModel.swift`

```swift
import SwiftUI
import Observation

enum WorkoutKind: String, Sendable {
    case run, strength, yoga

    var name: String {
        switch self {
        case .run: return "Run"
        case .strength: return "Strength"
        case .yoga: return "Yoga"
        }
    }
    var symbol: String {
        switch self {
        case .run: return "figure.run"
        case .strength: return "dumbbell.fill"
        case .yoga: return "figure.mind.and.body"
        }
    }
    /// The workout's own colour: its badge, and the strip while it is in focus.
    var color: Color {
        switch self {
        case .run: return EaseBrand.blue
        case .strength: return EaseBrand.orange
        case .yoga: return EaseBrand.violet
        }
    }
}

struct WorkoutPlan: Identifiable, Sendable {
    let kind: WorkoutKind
    let day: Int            // column of the week strip, 0 = Monday
    let dayWord: String
    let planMinutes: Int
    let detail: String
    var id: String { kind.rawValue }
}

/// One bar of the heart-rate graph: one minute of the workout.
struct WorkoutBeat: Identifiable, Sendable {
    let id: Int
    let bpm: Int
}

enum WorkoutPhase: Equatable { case week, live, result }

/// Source of truth: the plan, what is logged, and the elapsed seconds of today's workout.
@MainActor @Observable
final class WorkoutModel {
    // Example week. Monday's run is already logged.
    static let plan: [WorkoutPlan] = [
        WorkoutPlan(kind: .run, day: 0, dayWord: "Monday", planMinutes: 30, detail: "Easy 5 km"),
        WorkoutPlan(kind: .strength, day: 2, dayWord: "Today", planMinutes: 45, detail: "Upper body"),
        WorkoutPlan(kind: .yoga, day: 4, dayWord: "Friday", planMinutes: 30, detail: "Slow flow"),
    ]
    static let today = 2
    static let todayPlan = plan[1]
    static let letters = ["M", "T", "W", "T", "F", "S", "S"]
    static let numbers = [9, 10, 11, 12, 13, 14, 15]
    static let streakBefore = 11
    /// In the demo each tick fast-forwards five minutes, so the 45 minutes play in nine seconds.
    static let demoStep = 300

    var phase: WorkoutPhase = .week
    var logged: [String: Int] = ["run": 32]     // minutes per finished workout
    var elapsed = 0                             // seconds of today's workout
    var finished = false
    var shownMinutes = 0                        // rolls to weekMinutes
    var shownStreak = WorkoutModel.streakBefore // rolls to streak
    var pressed: String? = nil
    var demoRunning: Bool
    @ObservationIgnored private var timed: Task<Void, Never>?

    init(demo: Bool) { demoRunning = demo }

    // MARK: Computed from the source of truth

    func isDone(_ p: WorkoutPlan) -> Bool { logged[p.id] != nil }
    var todayPending: Bool { !isDone(Self.todayPlan) }
    var weekMinutes: Int { logged.values.reduce(0, +) }       // 32 + 45 = 77 after today
    var doneCount: Int { Self.plan.filter { isDone($0) }.count }
    var doneLabel: String { "\(doneCount) of \(Self.plan.count) done" }
    var next: WorkoutPlan? { Self.plan.first { !isDone($0) && $0.day >= Self.today } }
    var streak: Int { Self.streakBefore + (todayPending ? 0 : 1) }
    var goalSeconds: Int { Self.todayPlan.planMinutes * 60 }
    var elapsedMinutes: Int { elapsed / 60 }
    var timerText: String { String(format: "%d:%02d", elapsed / 60, elapsed % 60) }
    var currentBPM: Int { Self.bpm(atMinute: Double(elapsed) / 60) }
    var beats: [WorkoutBeat] {
        (0..<elapsedMinutes).map { WorkoutBeat(id: $0, bpm: Self.bpm(atMinute: Double($0) + 0.5)) }
    }

    /// Example heart rate: a warm-up curve, a peak with every set, a slow drift.
    static func bpm(atMinute m: Double) -> Int {
        let warm = 38 * (1 - exp(-m / 5))
        let sets = 14 * max(0, sin(m * 1.3))
        let drift = 4 * sin(m * 0.37)
        return Int((88 + warm + sets + drift).rounded())
    }

    var stripColor: Color {
        switch phase {
        case .week: return (next ?? Self.todayPlan).kind.color
        case .live: return EaseBrand.red        // the heart rate is what this screen is about
        case .result: return EaseBrand.green    // the streak grew
        }
    }

    var buttonTitle: String {
        switch phase {
        case .week: return todayPending ? "Start \(Self.todayPlan.kind.name.lowercased())" : "Done for today"
        case .live: return finished ? "Finished" : "Finish workout"
        case .result: return "Done"
        }
    }
    var buttonEnabled: Bool { phase != .week || todayPending }
    var success: Bool { phase == .live && finished }

    // MARK: Actions

    func appear() { later(0.4) { [weak self] in self?.shownMinutes = self?.weekMinutes ?? 0 } }

    func mainAction() {
        switch phase {
        case .week: start()
        case .live: finish()
        case .result: done()
        }
    }

    func start() {
        guard phase == .week, todayPending else { return }
        elapsed = 0
        finished = false
        withAnimation(EaseMotion.smooth(0.5)) { phase = .live }
    }

    func tick(by seconds: Int) {
        guard phase == .live, !finished else { return }
        withAnimation(EaseMotion.smooth(0.35)) { elapsed += seconds }
    }

    func finish() {
        guard phase == .live, !finished else { return }
        let minutes = max(1, Int((Double(elapsed) / 60).rounded()))
        withAnimation(EaseMotion.smooth(0.35)) {
            finished = true
            logged[Self.todayPlan.id] = minutes
        }
        timed?.cancel()
        timed = Task {
            do {
                try await EaseMotion.wait(1.5)
                shownStreak = Self.streakBefore
                withAnimation(EaseMotion.smooth(0.5)) { phase = .result }
                try await EaseMotion.wait(0.4)
                shownStreak = streak
            } catch {
                // Cancelled: the view went away.
            }
        }
    }

    func done() {
        guard phase == .result else { return }
        withAnimation(EaseMotion.smooth(0.5)) { phase = .week }
        later(0.45) { [weak self] in self?.shownMinutes = self?.weekMinutes ?? 0 }
    }

    func stop() { timed?.cancel() }

    private func later(_ seconds: Double, _ step: @escaping @MainActor () -> Void) {
        timed?.cancel()
        timed = Task {
            do {
                try await EaseMotion.wait(seconds)
                step()
            } catch {}
        }
    }

    /// The live clock for real use: one second per tick. Ends when the live view goes away.
    func runClock() async {
        do {
            while !Task.isCancelled {
                try await EaseMotion.wait(1)
                if !demoRunning { tick(by: 1) }
            }
        } catch {}
    }

    // MARK: Demo, played once on launch

    private func tap(_ id: String) async throws {
        withAnimation(EaseMotion.spring(0.2)) { pressed = id }
        try await EaseMotion.wait(0.17)
        withAnimation(EaseMotion.spring(0.35, bounce: 0.2)) { pressed = nil }
        try await EaseMotion.wait(0.08)
    }

    func runDemo() async {
        guard demoRunning else { return }
        defer { demoRunning = false }
        do {
            try await EaseMotion.wait(2.4)                 // the week comes in and rests
            try await tap("main"); start()
            try await EaseMotion.wait(1.2)                 // the live screen reveals
            while elapsed < goalSeconds {
                try await EaseMotion.wait(1.0)
                tick(by: min(Self.demoStep, goalSeconds - elapsed))
            }
            try await EaseMotion.wait(1.2)
            try await tap("main"); finish()
            try await EaseMotion.wait(1.5 + 0.4 + 2.4)     // green rests, result comes in and rests
            try await tap("main"); done()
        } catch {
            // Cancelled: the view went away.
        }
    }
}
```

### `WorkoutWeekView.swift`

```swift
import SwiftUI

/// Overview: minutes trained this week, each workout pinned above its day, today's workout below.
struct WorkoutWeekView: View {
    let model: WorkoutModel
    @State private var shown = false
    private let lead = 0.14

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            Text("This week")
                .font(.system(size: 18, weight: .medium))
                .foregroundStyle(EaseInk.secondary)
                .easeReveal(shown, delay: lead)

            HStack(alignment: .center, spacing: 14) {
                HStack(alignment: .firstTextBaseline, spacing: 6) {
                    Text("\(model.shownMinutes)")
                        .font(.system(size: 62, weight: .bold))
                        .tracking(-2)
                        .monospacedDigit()
                        .foregroundStyle(EaseInk.primary)
                        .contentTransition(.numericText(value: Double(model.shownMinutes)))
                        .animation(EaseMotion.spring(0.9), value: model.shownMinutes)
                    Text("min")
                        .font(.system(size: 21, weight: .semibold))
                        .foregroundStyle(EaseInk.secondary)
                }
                .fixedSize()
                Text(model.doneLabel)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .contentTransition(.numericText())
                    .padding(.horizontal, 14)
                    .padding(.vertical, 8)
                    .background(Capsule().fill(EaseInk.chip))
                    .fixedSize()
            }
            .padding(.top, 2)
            .accessibilityElement(children: .combine)
            .accessibilityLabel("\(model.weekMinutes) minutes trained this week, \(model.doneLabel)")
            .easeReveal(shown, delay: lead + 0.06)

            WorkoutWeekStrip(model: model, shown: shown)
                .padding(.top, 14)

            Rectangle().fill(EaseInk.hairline).frame(height: 1)
                .padding(.top, 20)
                .easeReveal(shown, delay: lead + 0.18, y: 0)

            if let next = model.next {
                Text(next.day == WorkoutModel.today ? "Today" : "Next up")
                    .font(.system(size: 16, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .padding(.top, 20)
                    .easeReveal(shown, delay: lead + 0.18)

                row(next)
                    .padding(.top, 12)
                    .easeReveal(shown, delay: lead + 0.24)
            }
        }
        .onAppear { shown = true }
    }

    private func row(_ p: WorkoutPlan) -> some View {
        HStack(spacing: 14) {
            EaseLogoBadge(logo: Image(systemName: p.kind.symbol), tint: p.kind.color,
                          scale: 0.52, corner: 14, shadow: false)
                .frame(width: 50, height: 50)
            VStack(alignment: .leading, spacing: 2) {
                Text(p.kind.name)
                    .font(.system(size: 19, weight: .semibold))
                    .foregroundStyle(EaseInk.primary)
                Text(p.detail)
                    .font(.system(size: 15))
                    .foregroundStyle(EaseInk.secondary)
            }
            Spacer()
            Text("\(p.planMinutes) min")
                .font(.system(size: 19, weight: .medium))
                .monospacedDigit()
                .foregroundStyle(EaseInk.primary)
        }
        .padding(12)
        .overlay(RoundedRectangle(cornerRadius: 22, style: .continuous).strokeBorder(EaseInk.hairline, lineWidth: 1))
        .accessibilityElement(children: .combine)
    }
}

/// Seven days; each workout's badge stands on a thin line above its day, the next one larger.
struct WorkoutWeekStrip: View {
    let model: WorkoutModel
    let shown: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        HStack(alignment: .bottom, spacing: 0) {
            ForEach(0..<7, id: \.self) { i in
                let plan = WorkoutModel.plan.first { $0.day == i }
                VStack(spacing: 0) {
                    Color.clear.frame(height: 100).overlay(alignment: .bottom) {
                        if let plan { pin(plan) }
                    }
                    VStack(spacing: 8) {
                        Text(WorkoutModel.letters[i])
                            .font(.system(size: 16, weight: .medium))
                            .foregroundStyle(EaseInk.secondary)
                        Text("\(WorkoutModel.numbers[i])")
                            .font(.system(size: 21, weight: .semibold))
                            .foregroundStyle(i == WorkoutModel.today ? Color.white : EaseInk.primary)
                            .frame(width: 40, height: 40)
                            .background(i == WorkoutModel.today ? EaseInk.black : Color.clear, in: Circle())
                    }
                    .padding(.top, 8)
                    .easeReveal(shown, delay: 0.26 + Double(i) * 0.025, y: 10, blur: 6)
                }
                .frame(maxWidth: .infinity)
                .zIndex(plan?.id == model.next?.id && plan != nil ? 2 : (plan == nil ? 0 : 1))
            }
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("This week's plan")
        .accessibilityValue(WorkoutModel.plan.map { "\($0.kind.name) \($0.dayWord), \(model.isDone($0) ? "done" : "planned")" }
            .joined(separator: "; "))
    }

    /// Badges drop in out of a blur one after another; a finished one gets a small green check.
    private func pin(_ p: WorkoutPlan) -> some View {
        let isNext = p.id == model.next?.id
        let size: CGFloat = isNext ? 68 : 52
        let order = Double(WorkoutModel.plan.firstIndex { $0.id == p.id } ?? 0)
        return VStack(spacing: 0) {
            EaseLogoBadge(logo: Image(systemName: p.kind.symbol), tint: p.kind.color, scale: 0.5)
                .frame(width: size, height: size)
                .overlay(alignment: .bottomTrailing) {
                    if model.isDone(p) {
                        Image(systemName: "checkmark")
                            .font(.system(size: 10, weight: .bold))
                            .foregroundStyle(Color.white)
                            .frame(width: 20, height: 20)
                            .background(EaseInk.successFill, in: Circle())
                            .overlay(Circle().strokeBorder(Color.white, lineWidth: 2))
                            .offset(x: 3, y: 3)
                            .easePop(shown, from: 0.4, bounce: 0.4, delay: 0.75 + order * 0.13)
                    }
                }
                .scaleEffect(shown || reduceMotion ? 1 : 0.55)
                .opacity(shown ? 1 : 0)
                .blur(radius: shown || reduceMotion ? 0 : 8)
                .offset(y: shown || reduceMotion ? 0 : -22)
                .animation(EaseMotion.spring(0.75, bounce: 0.26, delay: 0.44 + order * 0.13), value: shown)
            Rectangle()
                .fill(EaseInk.hairline)
                .frame(width: 1.5, height: isNext ? 24 : 32)
                .scaleEffect(y: shown ? 1 : 0, anchor: .bottom)
                .animation(EaseMotion.smooth(0.4, delay: 0.66 + order * 0.13), value: shown)
        }
        .fixedSize()
    }
}
```

### `WorkoutLiveView.swift`

```swift
import SwiftUI

/// Live: the timer, the heart rate drawn large, progress toward the goal.
struct WorkoutLiveView: View {
    let model: WorkoutModel
    @State private var shown = false
    private let lead = 0.14

    var body: some View {
        let plan = WorkoutModel.todayPlan
        VStack(alignment: .leading, spacing: 0) {
            HStack {
                Text(plan.kind.name)
                    .font(.system(size: 18, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                Spacer()
                EaseStatusChip(text: model.finished ? "Done" : "Live",
                               dot: model.finished ? EaseInk.successFill : EaseInk.live,
                               pulse: !model.finished)
                    .animation(EaseMotion.spring(0.5), value: model.finished)
            }
            .easeReveal(shown, delay: lead)

            HStack(alignment: .center, spacing: 14) {
                Text(model.timerText)
                    .font(.system(size: 62, weight: .bold))
                    .tracking(-2)
                    .monospacedDigit()
                    .foregroundStyle(EaseInk.primary)
                    .contentTransition(.numericText(value: Double(model.elapsed)))
                    .animation(EaseMotion.spring(0.6), value: model.elapsed)
                    .fixedSize()
                Text("of \(plan.planMinutes) min")
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 8)
                    .background(Capsule().fill(EaseInk.chip))
                    .fixedSize()
            }
            .padding(.top, 2)
            .accessibilityElement(children: .combine)
            .easeReveal(shown, delay: lead + 0.06)

            WorkoutHeartCard(bpm: model.currentBPM, beats: model.beats, accent: EaseBrand.red)
                .frame(height: 250)
                .padding(.top, 16)
                .easeReveal(shown, delay: lead + 0.12)

            goalRow(plan)
                .padding(.top, 20)
                .easeReveal(shown, delay: lead + 0.18)
        }
        .onAppear { shown = true }
        .task { await model.runClock() }
    }

    private func goalRow(_ plan: WorkoutPlan) -> some View {
        let fraction = min(Double(model.elapsed) / Double(max(model.goalSeconds, 1)), 1)
        return VStack(spacing: 12) {
            HStack {
                Text("Goal")
                    .font(.system(size: 16, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                Spacer()
                Text("\(model.elapsedMinutes) of \(plan.planMinutes) min")
                    .font(.system(size: 19, weight: .medium))
                    .monospacedDigit()
                    .foregroundStyle(EaseInk.primary)
                    .contentTransition(.numericText(value: Double(model.elapsedMinutes)))
                    .animation(EaseMotion.spring(0.6), value: model.elapsedMinutes)
            }
            GeometryReader { g in
                Capsule().fill(EaseInk.hairline)
                    .overlay(alignment: .leading) {
                        Capsule().fill(EaseBrand.red).frame(width: g.size.width * fraction)
                    }
            }
            .frame(height: 8)
        }
        .accessibilityElement(children: .combine)
    }
}

/// The hero: the current heart rate at its top left, one bar per minute below, the target zone dashed behind.
struct WorkoutHeartCard: View {
    let bpm: Int
    let beats: [WorkoutBeat]
    let accent: Color
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    private let low = 70.0, high = 170.0, zoneLow = 120.0, zoneHigh = 150.0

    var body: some View {
        VStack(alignment: .leading, spacing: 14) {
            HStack(spacing: 8) {
                Image(systemName: "heart.fill")
                    .font(.system(size: 17, weight: .semibold))
                    .foregroundStyle(accent)
                    .phaseAnimator([CGFloat(1), reduceMotion ? 1 : 1.14]) { heart, s in
                        heart.scaleEffect(s)
                    } animation: { _ in EaseMotion.pulse(0.6) }
                Text("\(bpm) bpm")
                    .font(.system(size: 21, weight: .semibold))
                    .monospacedDigit()
                    .foregroundStyle(EaseInk.primary)
                    .contentTransition(.numericText(value: Double(bpm)))
                    .animation(EaseMotion.spring(0.4), value: bpm)
                Spacer()
                Text("Target \(Int(zoneLow))–\(Int(zoneHigh))")
                    .font(.system(size: 14, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
            }

            GeometryReader { g in
                let w = g.size.width, h = g.size.height
                let cap = max(Int((w + 4) / 8), 1)
                let visible = Array(beats.suffix(cap))
                let yHigh = y(zoneHigh, h), yLow = y(zoneLow, h)
                ZStack(alignment: .topLeading) {
                    Rectangle().fill(accent.opacity(0.07))
                        .frame(width: w, height: yLow - yHigh)
                        .offset(y: yHigh)
                    WorkoutDash().stroke(accent.opacity(0.5), style: StrokeStyle(lineWidth: 1, dash: [3, 4]))
                        .frame(width: w, height: 1).offset(y: yHigh)
                    WorkoutDash().stroke(accent.opacity(0.5), style: StrokeStyle(lineWidth: 1, dash: [3, 4]))
                        .frame(width: w, height: 1).offset(y: yLow)

                    HStack(alignment: .bottom, spacing: 4) {
                        ForEach(Array(visible.enumerated()), id: \.element.id) { i, beat in
                            let age = visible.count - 1 - i
                            Capsule()
                                .fill(age == 0 ? accent : Self.fade(age, cap))
                                .frame(width: 4, height: max(h - y(Double(beat.bpm), h), 6))
                                .transition(AnyTransition.workoutGrow(reduceMotion: reduceMotion))
                        }
                    }
                    .frame(width: w, height: h, alignment: .bottomTrailing)
                }
            }
        }
        .padding(18)
        .background(RoundedRectangle(cornerRadius: 30, style: .continuous).fill(EaseInk.fill))
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Heart rate")
        .accessibilityValue("\(bpm) beats per minute, target \(Int(zoneLow)) to \(Int(zoneHigh))")
        .accessibilityAddTraits(.updatesFrequently)
    }

    private func y(_ value: Double, _ h: CGFloat) -> CGFloat {
        let f = min(max((value - low) / (high - low), 0), 1)
        return h - h * CGFloat(f)
    }

    /// Older bars fade from near black to the hairline grey toward the leading edge.
    static func fade(_ age: Int, _ cap: Int) -> Color {
        let f = min(Double(age) / Double(max(cap - 1, 1)), 1)
        return Color(red: 0.067 + (0.898 - 0.067) * f,
                     green: 0.067 + (0.898 - 0.067) * f,
                     blue: 0.075 + (0.914 - 0.075) * f)
    }
}

struct WorkoutDash: Shape {
    func path(in r: CGRect) -> Path {
        Path { p in
            p.move(to: CGPoint(x: r.minX, y: r.midY))
            p.addLine(to: CGPoint(x: r.maxX, y: r.midY))
        }
    }
}

/// A new bar grows from the baseline. With Reduce Motion it only fades.
struct WorkoutGrow: ViewModifier {
    let on: Bool
    func body(content: Content) -> some View {
        content
            .scaleEffect(x: 1, y: on ? 1 : 0.02, anchor: .bottom)
            .opacity(on ? 1 : 0)
    }
}

extension AnyTransition {
    @MainActor static func workoutGrow(reduceMotion: Bool) -> AnyTransition {
        reduceMotion ? .opacity
            : .asymmetric(insertion: .modifier(active: WorkoutGrow(on: false), identity: WorkoutGrow(on: true)),
                          removal: .opacity)
    }
}
```

### `WorkoutResultView.swift`

```swift
import SwiftUI

/// Result: a check, the streak, one chip per workout of the week.
struct WorkoutResultView: View {
    let model: WorkoutModel
    @State private var shown = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        VStack(spacing: 0) {
            Spacer(minLength: 0)

            ZStack {
                Circle().fill(EaseInk.black)
                EaseCheckShape()
                    .trim(from: 0, to: shown ? 1 : 0)
                    .stroke(Color.white, style: StrokeStyle(lineWidth: 5, lineCap: .round, lineJoin: .round))
                    .frame(width: 34, height: 26)
                    .animation(EaseMotion.smooth(0.5, delay: 0.35), value: shown)
            }
            .frame(width: 84, height: 84)
            .scaleEffect(shown || reduceMotion ? 1 : 0.6)
            .opacity(shown ? 1 : 0)
            .blur(radius: shown || reduceMotion ? 0 : 10)
            .animation(EaseMotion.spring(0.7, bounce: 0.28, delay: 0.05), value: shown)
            .accessibilityHidden(true)

            VStack(spacing: 0) {
                Text("Your streak")
                    .padding(.top, 24)
                    .easeReveal(shown, delay: 0.18)
                Text("\(model.shownStreak)")
                    .font(.system(size: 62, weight: .bold))
                    .tracking(-2)
                    .monospacedDigit()
                    .foregroundStyle(EaseInk.primary)
                    .contentTransition(.numericText(value: Double(model.shownStreak)))
                    .animation(EaseMotion.spring(0.9), value: model.shownStreak)
                    .fixedSize()
                    .easeReveal(shown, delay: 0.24)
                Text("workouts in a row")
                    .easeReveal(shown, delay: 0.3)
            }
            .font(.system(size: 19, weight: .medium))
            .foregroundStyle(EaseInk.secondary)
            .accessibilityElement(children: .ignore)
            .accessibilityLabel("Streak: \(model.streak) workouts in a row")

            HStack(spacing: 8) {
                ForEach(Array(WorkoutModel.plan.enumerated()), id: \.element.id) { i, p in
                    chip(p).easeReveal(shown, delay: 0.5 + Double(i) * 0.09)
                }
            }
            .padding(.top, 28)

            Spacer(minLength: 0)
        }
        .frame(maxWidth: .infinity, maxHeight: .infinity)
        .onAppear { shown = true }
    }

    /// The symbol and one word: minutes when done, the day when still planned (muted).
    private func chip(_ p: WorkoutPlan) -> some View {
        let minutes = model.logged[p.id]
        let muted = minutes == nil
        return HStack(spacing: 7) {
            Image(systemName: p.kind.symbol)
                .resizable().scaledToFit()
                .foregroundStyle(p.kind.color)
                .frame(width: 18, height: 18)
                .saturation(muted ? 0 : 1)
                .opacity(muted ? 0.55 : 1)
            Text(minutes.map { "\($0) min" } ?? p.dayWord)
                .font(.system(size: 14, weight: .medium))
                .monospacedDigit()
                .foregroundStyle(muted ? EaseInk.secondary : EaseInk.primary)
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .background(EaseInk.chip, in: Capsule())
        .fixedSize()
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("\(p.kind.name), " + (minutes.map { "\($0) minutes" } ?? "planned \(p.dayWord)"))
    }
}
```

### `WorkoutRootView.swift`

```swift
import SwiftUI

/// Strip, white sheet, the three screens and the pinned button.
struct WorkoutRootView: View {
    @State private var model: WorkoutModel
    @State private var sheetUp = false

    init(playsDemo: Bool = true) {
        _model = State(initialValue: WorkoutModel(demo: playsDemo))
    }

    var body: some View {
        EaseSheet(up: sheetUp) {
            ZStack {
                Color.white
                LinearGradient(colors: EaseStrip.from(model.stripColor), startPoint: .leading, endPoint: .trailing)
            }
        } content: {
            switch model.phase {
            case .week:
                WorkoutWeekView(model: model).transition(AnyTransition.easeBlurOut)
            case .live:
                WorkoutLiveView(model: model).transition(AnyTransition.easeBlurOut)
            case .result:
                WorkoutResultView(model: model).transition(AnyTransition.easeBlurOut)
            }
        } footer: {
            EasePillButton(title: model.buttonTitle, enabled: model.buttonEnabled,
                           success: model.success, done: model.success,
                           pressed: model.pressed == "main") {
                model.mainAction()
            }
        }
        .allowsHitTesting(!model.demoRunning)
        .onAppear {
            sheetUp = true
            model.appear()
        }
        .onDisappear { model.stop() }
        .task { await model.runDemo() }
    }
}

#Preview("Demo") { WorkoutRootView(playsDemo: true) }
#Preview("Taps") { WorkoutRootView(playsDemo: false) }
```

### `EaseMotion.swift`

```swift
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

/// Colours of the Easeout look: flat, light mode only.
enum EaseInk {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)
    static let secondary = Color(red: 0.459, green: 0.463, blue: 0.498)
    static let faint = Color(red: 0.667, green: 0.667, blue: 0.706)
    static let fill = Color(red: 0.957, green: 0.957, blue: 0.961)
    static let chip = Color(red: 0.945, green: 0.945, blue: 0.949)
    static let hairline = Color(red: 0.898, green: 0.898, blue: 0.914)
    static let outline = Color(red: 0.831, green: 0.831, blue: 0.847)
    static let track = Color(red: 0.902, green: 0.902, blue: 0.918)
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let off = Color(red: 0.929, green: 0.929, blue: 0.937)
    static let successFill = Color(red: 0.133, green: 0.773, blue: 0.369)
    static let successText = Color.white
    static let up = Color(red: 0.133, green: 0.773, blue: 0.369)
    static let down = Color(red: 0.937, green: 0.267, blue: 0.267)
    static let live = Color(red: 0.937, green: 0.196, blue: 0.196)
}
```

### `EaseParts.swift`

```swift
import SwiftUI

// MARK: - Reveal

/// Element comes in from a soft blur, a little lower and smaller. With Reduce Motion it only fades in.
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

// MARK: - Blur out

/// A screen leaves by melting into a blur. With Reduce Motion it only fades.
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

// MARK: - Press

/// Soft dot that shows a press. With Reduce Motion it only fades.
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

// MARK: - Pill button

/// Flat pill button: black (an action), pale grey (switched off), green with a check (it worked).
/// `pressed` draws a press while the demo plays and no finger is there.
struct EasePillButton: View {
    let title: String
    var enabled = true
    var success = false
    var done = false
    var pressed = false
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
            .overlay(EaseTouchDot(on: pressed))
            .contentShape(Capsule())
        }
        .buttonStyle(EasePressStyle())
        .scaleEffect(pressed && !reduceMotion ? 0.965 : 1)
        .animation(pressed ? EaseMotion.spring(0.2) : EaseMotion.spring(0.35, bounce: 0.2), value: pressed)
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

// MARK: - Status chip

/// Small grey capsule: a status label, not a button. With `pulse` the dot breathes.
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
                Circle().fill(dot).frame(width: 7, height: 7)
                    .phaseAnimator([1.0, low]) { circle, level in
                        circle.opacity(level)
                    } animation: { _ in EaseMotion.pulse(0.6) }
                    .transition(dotIn)
            }
            Text(text)
                .font(.system(size: 14, weight: .medium))
                .foregroundStyle(EaseInk.secondary)
                .contentTransition(.numericText())
        }
        .padding(.horizontal, 13)
        .frame(height: 32)
        .background(Capsule().fill(EaseInk.chip))
    }
}

// MARK: - Pop

/// A small thing pops and keeps its place while hidden. With Reduce Motion it only fades.
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

// MARK: - Badges, colours, strip

/// White circle (or rounded square), hairline, soft shadow, the picture in its own colour.
struct EaseLogoBadge: View {
    var logo: Image
    var tint: Color? = nil
    var scale: CGFloat = 0.54
    var corner: CGFloat? = nil
    var shadow = true

    var body: some View {
        GeometryReader { geo in
            let side = min(geo.size.width, geo.size.height)
            let shape = RoundedRectangle(cornerRadius: corner ?? side / 2, style: corner == nil ? .circular : .continuous)
            ZStack {
                shape.fill(Color.white).overlay(shape.strokeBorder(EaseInk.hairline, lineWidth: 1))
                logo.renderingMode(tint == nil ? nil : .template).resizable().scaledToFit()
                    .foregroundStyle(tint ?? EaseInk.black)
                    .frame(width: side * scale, height: side * scale)
            }
            .frame(width: geo.size.width, height: geo.size.height)
        }
        .shadow(color: .black.opacity(shadow ? 0.07 : 0), radius: 7, y: 3)
    }
}

enum EaseBrand {
    static let red = Color(red: 0.90, green: 0.22, blue: 0.23)
    static let orange = Color(red: 0.98, green: 0.45, blue: 0.09)
    static let green = Color(red: 0.13, green: 0.77, blue: 0.37)
    static let blue = Color(red: 0.15, green: 0.39, blue: 0.92)
    static let violet = Color(red: 0.55, green: 0.36, blue: 0.96)
}

enum EaseStrip {
    /// A light end and the full colour, drawn over white.
    static func from(_ c: Color) -> [Color] { [c.opacity(0.42), c.opacity(0.92)] }
}

/// Stroked with trim, so the check writes itself.
struct EaseCheckShape: Shape {
    func path(in r: CGRect) -> Path {
        Path { p in
            p.move(to: CGPoint(x: r.minX, y: r.midY + r.height * 0.08))
            p.addLine(to: CGPoint(x: r.minX + r.width * 0.36, y: r.maxY))
            p.addLine(to: CGPoint(x: r.maxX, y: r.minY))
        }
    }
}

// MARK: - Sheet

/// Coloured strip, a white sheet that slides up over it, the screens of the flow, the pinned button.
/// With Reduce Motion the sheet stays in place and fades.
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
                ZStack(alignment: .top) { content }
                    .frame(maxWidth: .infinity, maxHeight: .infinity, alignment: .top)
                footer
                    .easeReveal(up, delay: up ? 0.5 : 0)
            }
            .padding(.horizontal, 22)
            .padding(.top, 30)
            .padding(.bottom, 40)
            .frame(maxWidth: .infinity)
            .frame(height: height)
            .background(
                UnevenRoundedRectangle(topLeadingRadius: 40, topTrailingRadius: 40, style: .continuous)
                    .fill(Color.white)
                    .padding(.bottom, -200)
            )
            .offset(y: up || reduceMotion ? 0 : height + 60)
            .opacity(up || !reduceMotion ? 1 : 0)
            .animation(reduceMotion ? EaseMotion.smooth(0.35) : EaseMotion.spring(0.85, bounce: 0.08), value: up)
        }
        .ignoresSafeArea(edges: .bottom)
        .environment(\.colorScheme, .light)
    }
}
```

### `WorkoutApp.swift`

```swift
import SwiftUI

@main
struct WorkoutApp: App {
    var body: some Scene {
        WindowGroup {
            // Plays the flow once by itself on launch, then real taps take over.
            WorkoutRootView(playsDemo: true)
        }
    }
}
```
