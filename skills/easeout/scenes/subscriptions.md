# Easeout scene: Subscriptions, logos pinned to a week (SwiftUI)

Add this ready-made SwiftUI scene to my project. The code below is complete and tested. Your job is to place it, make it build, and show it to me running. Do not redesign it.

## What you are building

Three screens inside a white sheet over a yellow-to-orange strip: a subscription tracker that shows what renews this week and lets me keep or cancel each one. Its hero is a week strip with the official logo of each subscription in a white circle, standing on a thin line above the day it renews. Everything that shows money or a count follows one list, `SubsModel.all`, and my choices in `SubsModel.choices`. It plays in five beats:

1. **This week.** The content comes in piece by piece, top to bottom. The three logos drop into place out of a blur, one after another, and each thin line grows up to its badge: Netflix (68 pt, it renews tomorrow) above Thursday 3, Spotify above Friday 4, YouTube above Sunday 6. Today, Wednesday 2, sits in a black circle. The amount rolls from $0.00 to $40.47 next to the chip "3 renewals". Below: "Next up" with Netflix, Tomorrow, $15.49, and the black pill button "Review this week".
2. **Review.** A press on the button: the week screen melts into a blur and the Netflix logo travels from its day into a grey card. The card shows the logo at 96 pt, "Netflix", $15.49, the soft orange chip "Renews tomorrow", "Last opened 34 days ago" and "Paid so far $185.88". Two more cards peek out below it. On top: "Review", "1 of 3" and three progress segments, the first one filling. At the bottom: "Cancel" (outline) and "Keep" (black).
3. **Decide.** A press on "Cancel": the Netflix card tilts 10° and flies out to the left with a blur. The Spotify card grows into its place ($10.99, Renews Friday, Today, $131.88), the counter rolls to "2 of 3" and the next segment fills. "Keep" sends Spotify out to the right, then "Keep" again for YouTube Premium ($13.99, Renews Sunday, Yesterday, $83.94).
4. **You save.** The review melts away. A black circle pops in and its check writes itself, then "You save", the amount rolling from $0.00 to $15.49, "every month", and one chip per subscription: a grey Netflix "Cancelled", Spotify "Kept", YouTube "Kept". Button: "Done".
5. **Back to the week.** A press on "Done": the week comes back without Netflix. The amount rolls from $40.47 to $24.98, the chip says "2 renewals" and "Next up" shows Spotify, Friday, $10.99.

`SubsModel` plays these beats once by itself when the screen appears (`playDemo()`), showing each press on the button with a small shrink and a soft dot, so I can see the motion without touching anything. Any real tap ends the demo and the flow goes on from there with real taps.

## Rules

- Target iOS 17 or later. SwiftUI only, no packages. The only image files are the three official logos described under "Logos" below.
- Create the nine files exactly as given. Keep every number as written: font sizes, paddings, corner radii, colours, badge sizes, durations, curves and delays. These numbers are the design.
- The week total, the chip "3 renewals", the pins on the strip, the "Next up" row, the saved amount and the chips on the last screen are all computed from `all` and `choices` in `SubsModel`. Never write one of them by hand, and never let one of them lag behind the others.
- Screens change in one way: the old screen leaves with `AnyTransition.easeBlurOut` and the new one brings its own staggered `.easeReveal`. The single exception is the first logo, which travels from the week strip into the first card with `matchedGeometryEffect`. Do not add a second one, and do not replace any of this with slides or plain fades.
- A decided card leaves to its side (left for "Cancel", right for "Keep"), tilted and blurred. Do not replace this with a fade or a scale.
- Keep `.contentTransition(.numericText(value:))` on the amounts and the counter, so the digits roll.
- The pill button has three looks: black = an action to take, white with an outline = the other choice, pale grey = switched off (when nothing is left to review). Keep all three. The grey capsules ("3 renewals", "Renews Friday", "Kept") are labels, not buttons. Do not restyle one as the other.
- Keep all timing inside `EaseMotion`. Do not write durations in other places. Launching with the argument `-slow 4` plays everything four times slower; keep that.
- The files starting with `Ease` are the shared Easeout kit. If my project already declares any of these types (`EaseMotion`, `EaseInk`, `EaseStrip`, `EaseBrand`, `EaseReveal`, `EaseBlurOut`, `EaseTouchDot`, `EaseLogoBadge`, `EaseCheckShape`, the `easeReveal` modifier or `AnyTransition.easeBlurOut`), for example from another Easeout scene or from the Easeout skill, keep one copy and do not add a second: use the declaration that is already there and add to it only the members this scene needs and it lacks (for example `EaseStrip.sun`, `EaseBrand.warnFill` and `EaseBrand.warnText`). If a member exists in both with a different value, keep mine and tell me which one.
- The scene's own types start with `Subs`. If one of them collides with a type in my project, rename ours with the prefix `SubsScene` everywhere and tell me.
- If my project already has an `@main` app, do not add a second one: leave out the file `SubscriptionsApp.swift` (it holds nothing but the entry point), keep the other eight files unchanged, and show `SubsRootView()` from a preview, a sheet or a debug menu entry, whichever fits my project. Tell me where you put it.
- The scene respects Reduce Motion (things only fade) and stops every waiting step when the view goes away. Keep both.
- Light mode only; `SubsRootView` sets the light colour scheme for its own content.
- Do not add comments that explain your changes, and do not delete the comments that are there.

## Logos

The code loads three logos from the asset catalog: `Image("netflix")`, `Image("spotify")` and `Image("youtube")`. Add them before you build:

- Download the official files from Simple Icons: `https://cdn.jsdelivr.net/npm/simple-icons@15/icons/netflix.svg`, `.../spotify.svg` and `.../youtube.svg` (the slug is the lower-case name without spaces).
- Add each SVG to the asset catalog as a single-scale image set named `netflix`, `spotify` and `youtube`, with "Preserve Vector Data" checked and "Render As: Template Image". They are one-colour glyphs; `SubsBrand.color` gives each its brand colour as Simple Icons lists it (#E50914, #1ED760, #FF0000) through the tint of `EaseLogoBadge`.
- Do not stretch, crop or recolour them in any other way.
- Never generate a logo with an image model and never redraw one from memory as a path, a shape or lookalike letters. If you cannot download the files, stop and tell me; do not ship the scene with drawn logos.
- In your answer, list the three logos and their source, so that I can check the brands' terms.

## After it runs

1. Build and run it. Fix build errors without changing the design numbers.
2. Tell me in three lines: where the files are, how to open the scene, anything you had to change.
3. Then ask me one question: "Should I connect this to your real subscriptions?" If yes, remove the demo (`playDemo()`, the `demo` task, `demoChoices` and the `pressed` state with what reads it), fill `SubsModel.all` from my data (name, brand logo, price, renewal day, last opened, months paid), and save what I keep or cancel. Keep every animation as it is.

## What I may ask you to change

- The example subscriptions, their prices, days and details: `SubsItem.examples` in `SubsModel.swift`.
- The days of the week strip and which one is today: `letters`, `numbers` and `today` in `SubsWeekView`.
- The colours of the strip: `EaseStrip.sun`.
- A new brand: add its official logo to the asset catalog as above, a case in `SubsBrand` and its brand colour.

## Check yourself before you answer

- With the example data the week screen shows $40.47 and "3 renewals"; 15.49 + 10.99 + 13.99 = 40.47.
- After the demo cancels Netflix and keeps the other two, the last screen says "You save $15.49 every month", and the week shows $24.98, "2 renewals" and Spotify under "Next up".
- "Paid so far" is the price times the months paid: $185.88, $131.88, $83.94.
- Only Netflix has the soft orange chip and the 68 pt badge, because it is the only one marked `urgent`.
- The three logos come from the asset catalog in their brand colours, on white circles. None of them is drawn in code.

## The code

### `SubsModel.swift`

```swift
import SwiftUI
import Observation

enum SubsBrand: String, Sendable {
    case netflix, spotify, youtube

    /// Brand colours as Simple Icons lists them. The logo files are template images and take this colour.
    var color: Color {
        switch self {
        case .netflix: return Color(red: 0.898, green: 0.035, blue: 0.078)   // #E50914
        case .spotify: return Color(red: 0.118, green: 0.843, blue: 0.376)   // #1ED760
        case .youtube: return Color(red: 1.0, green: 0.0, blue: 0.0)         // #FF0000
        }
    }
    /// The official logo from the asset catalog.
    var logo: Image { Image(rawValue) }
    /// The Spotify logo is a disc itself, so it sits a little larger.
    var logoScale: CGFloat { self == .spotify ? 0.64 : 0.54 }
}

struct SubsItem: Identifiable, Sendable {
    let brand: SubsBrand
    let name: String
    let price: Double
    let day: Int            // column in the week strip, 0 = Monday
    let dayWord: String
    let renewLabel: String
    let urgent: Bool
    let lastOpened: String
    let monthsPaid: Int
    var id: String { brand.rawValue }
    var paidSoFar: Double { price * Double(monthsPaid) }

    // Example data. The week adds up: 15.49 + 10.99 + 13.99 = 40.47.
    static let examples: [SubsItem] = [
        SubsItem(brand: .netflix, name: "Netflix", price: 15.49, day: 3, dayWord: "Tomorrow",
                 renewLabel: "Renews tomorrow", urgent: true, lastOpened: "34 days ago", monthsPaid: 12),
        SubsItem(brand: .spotify, name: "Spotify", price: 10.99, day: 4, dayWord: "Friday",
                 renewLabel: "Renews Friday", urgent: false, lastOpened: "Today", monthsPaid: 12),
        SubsItem(brand: .youtube, name: "YouTube Premium", price: 13.99, day: 6, dayWord: "Sunday",
                 renewLabel: "Renews Sunday", urgent: false, lastOpened: "Yesterday", monthsPaid: 6),
    ]
}

enum SubsPhase: Equatable { case week, review, summary }
enum SubsChoice: Equatable, Sendable { case keep, cancel }

/// Source of truth: `all` and `choices`. Every amount, count, pin and chip is computed from them.
@MainActor @Observable
final class SubsModel {
    let all = SubsItem.examples
    var phase: SubsPhase = .week
    var queue: [SubsItem] = []
    var index = 0
    var choices: [String: SubsChoice] = [:]
    var total: Double = 0
    var saved: Double = 0
    /// The control the demo is "tapping", so the press shows without a finger.
    var pressed: String? = nil

    @ObservationIgnored private var timed: Task<Void, Never>?
    @ObservationIgnored private var demo: Task<Void, Never>?

    /// The example choices of the demo, one per card.
    static let demoChoices: [SubsChoice] = [.cancel, .keep, .keep]

    var active: [SubsItem] { all.filter { choices[$0.id] != .cancel } }
    var weekTotal: Double { active.reduce(0) { $0 + $1.price } }
    var cancelledTotal: Double { queue.filter { choices[$0.id] == .cancel }.reduce(0) { $0 + $1.price } }
    var renewalsLabel: String { active.count == 1 ? "1 renewal" : "\(active.count) renewals" }

    static func money(_ value: Double) -> String { String(format: "$%.2f", value) }

    // MARK: Lifecycle

    /// The week screen is on: the amount rolls up, then the demo plays the flow once.
    func start() {
        roll(after: 0.4)
        demo?.cancel()
        demo = Task { [weak self] in await self?.playDemo() }
    }

    /// The view went away: no step keeps waiting.
    func stop() {
        timed?.cancel()
        demo?.cancel()
    }

    // MARK: Real taps (they end the demo)

    func userReview() { endDemo(); review() }
    func userDecide(_ choice: SubsChoice) { endDemo(); decide(choice) }
    func userDone() { endDemo(); done() }

    private func endDemo() {
        demo?.cancel()
        demo = nil
        pressed = nil
    }

    // MARK: The flow

    func review() {
        guard phase == .week, !active.isEmpty else { return }
        choices = choices.filter { $0.value == .cancel }
        queue = active
        index = 0
        withAnimation(EaseMotion.spring(0.7, bounce: 0.14)) { phase = .review }
    }

    func decide(_ choice: SubsChoice) {
        guard phase == .review, index < queue.count, choices[queue[index].id] == nil else { return }
        let sub = queue[index]
        withAnimation(EaseMotion.smooth(0.65)) { choices[sub.id] = choice }
        if index < queue.count - 1 {
            withAnimation(EaseMotion.spring(0.7, bounce: 0.14)) { index += 1 }
        } else {
            timed?.cancel()
            timed = Task { [weak self] in
                do {
                    try await EaseMotion.wait(0.3)
                    guard let self else { return }
                    self.saved = 0
                    withAnimation(EaseMotion.smooth(0.5)) { self.phase = .summary }
                    try await EaseMotion.wait(0.4)
                    self.saved = self.cancelledTotal
                } catch {
                    // Cancelled: the view went away.
                }
            }
        }
    }

    func done() {
        guard phase == .summary else { return }
        withAnimation(EaseMotion.smooth(0.5)) { phase = .week }
        roll(after: 0.45)
    }

    private func roll(after seconds: Double) {
        timed?.cancel()
        timed = Task { [weak self] in
            do {
                try await EaseMotion.wait(seconds)
                guard let self else { return }
                self.total = self.weekTotal
            } catch {
                // Cancelled: the view went away.
            }
        }
    }

    // MARK: Demo (plays once on launch)

    private func tap(_ id: String) async throws {
        withAnimation(EaseMotion.spring(0.2)) { pressed = id }
        try await EaseMotion.wait(0.17)
        withAnimation(EaseMotion.spring(0.35, bounce: 0.2)) { pressed = nil }
        try await EaseMotion.wait(0.08)
    }

    private func playDemo() async {
        do {
            try await EaseMotion.wait(2.6)           // enter, amount rolls, rest
            try await tap("review")
            review()
            for choice in Self.demoChoices {
                try await EaseMotion.wait(1.6)       // card settles, rest
                try await tap(choice == .keep ? "keep" : "cancel")
                decide(choice)
            }
            try await EaseMotion.wait(3.0)           // summary arrives, amount rolls, rest
            try await tap("done")
            done()
        } catch {
            // Cancelled by a real tap or because the view went away.
        }
    }
}
```

### `SubsParts.swift`

```swift
import SwiftUI

/// The bare official logo in its brand colour.
struct SubsBrandLogo: View {
    let brand: SubsBrand

    var body: some View {
        brand.logo
            .renderingMode(.template)
            .resizable()
            .scaledToFit()
            .foregroundStyle(brand.color)
            .accessibilityHidden(true)
    }
}

/// Flat pill: black, white with an outline, or switched off. It gives way while pressed.
/// `forced` is the demo's press: it also shows the touch dot, because no finger is there.
struct SubsPillStyle: ButtonStyle {
    enum Kind { case solid, outline, off }
    var kind: Kind = .solid
    var forced = false

    func makeBody(configuration: Configuration) -> some View {
        SubsPillBody(label: configuration.label, kind: kind,
                     pressed: configuration.isPressed || forced, dot: forced)
    }
}

struct SubsPillBody: View {
    let label: ButtonStyleConfiguration.Label
    let kind: SubsPillStyle.Kind
    let pressed: Bool
    let dot: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        label
            .font(.system(size: 18, weight: .semibold))
            .foregroundStyle(kind == .solid ? Color.white : (kind == .off ? EaseInk.secondary : EaseInk.primary))
            .frame(maxWidth: .infinity)
            .frame(height: 54)
            .background(Capsule().fill(kind == .solid ? EaseInk.black : (kind == .off ? EaseInk.off : Color.white)))
            .overlay(Capsule().strokeBorder(EaseInk.hairline, lineWidth: kind == .outline ? 1.5 : 0))
            .overlay(EaseTouchDot(on: dot, ink: kind == .solid ? Color.white.opacity(0.26) : Color.black.opacity(0.12)))
            .contentShape(Capsule())
            .scaleEffect(pressed && !reduceMotion ? 0.965 : 1)
            .animation(pressed ? EaseMotion.spring(0.2) : EaseMotion.spring(0.35, bounce: 0.2), value: pressed)
    }
}

/// Dollar amount whose digits roll softly to the new value.
struct SubsRollingMoney: View {
    let value: Double
    var body: some View {
        Text(SubsModel.money(value))
            .monospacedDigit()
            .contentTransition(.numericText(value: value))
            .animation(EaseMotion.spring(0.9), value: value)
    }
}

struct SubsHairline: View {
    var body: some View { Rectangle().fill(EaseInk.hairline).frame(height: 1).accessibilityHidden(true) }
}
```

### `SubsWeekView.swift`

```swift
import SwiftUI

/// Screen 1 (overview): what renews this week, each logo pinned to its day.
/// Fixed sizes, as in the approved showcase.
@MainActor
struct SubsWeekView: View {
    var model: SubsModel
    var ns: Namespace.ID
    @State private var shown = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    // The example week. In an app these come from the calendar.
    private let letters = ["M", "T", "W", "T", "F", "S", "S"]
    private let numbers = [30, 1, 2, 3, 4, 5, 6]
    private let today = 2

    var body: some View {
        VStack(alignment: .leading, spacing: 0) {
            Text("This week")
                .font(.system(size: 18, weight: .medium))
                .foregroundStyle(EaseInk.secondary)
                .easeReveal(shown)

            HStack(alignment: .center, spacing: 14) {
                SubsRollingMoney(value: model.total)
                    .font(.system(size: 62, weight: .bold))
                    .tracking(-2)
                    .foregroundStyle(EaseInk.primary)
                    .fixedSize()
                Text(model.renewalsLabel)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .contentTransition(.numericText())
                    .animation(EaseMotion.spring(0.5), value: model.renewalsLabel)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 8)
                    .background(Capsule().fill(EaseInk.chip))
                    .fixedSize()
            }
            .padding(.top, 2)
            .accessibilityElement(children: .combine)
            .easeReveal(shown, delay: 0.06)

            strip.padding(.top, 14)

            SubsHairline().padding(.top, 20).easeReveal(shown, delay: 0.2, y: 0)

            Text("Next up")
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(EaseInk.secondary)
                .padding(.top, 20)
                .easeReveal(shown, delay: 0.24)

            if let next = model.active.first {
                nextRow(next).padding(.top, 12).easeReveal(shown, delay: 0.29)
            }

            Spacer(minLength: 14)

            Button("Review this week") { model.userReview() }
                .buttonStyle(SubsPillStyle(kind: model.active.isEmpty ? .off : .solid,
                                           forced: model.pressed == "review"))
                .disabled(model.active.isEmpty)
                .easeReveal(shown, delay: 0.35)
        }
        .onAppear { shown = true }
    }

    private var strip: some View {
        HStack(alignment: .bottom, spacing: 0) {
            ForEach(0..<7, id: \.self) { i in
                let sub = model.active.first(where: { $0.day == i })
                VStack(spacing: 0) {
                    // An overlay, so a wide badge cannot push the seven columns apart.
                    Color.clear
                        .frame(height: 100)
                        .overlay(alignment: .bottom) {
                            if let sub { pin(sub) }
                        }
                    VStack(spacing: 8) {
                        Text(letters[i])
                            .font(.system(size: 16, weight: .medium))
                            .foregroundStyle(EaseInk.secondary)
                        ZStack {
                            if i == today { Circle().fill(EaseInk.black).frame(width: 40, height: 40) }
                            Text("\(numbers[i])")
                                .font(.system(size: 21, weight: .semibold))
                                .foregroundStyle(i == today ? Color.white : EaseInk.primary)
                        }
                        .frame(height: 40)
                    }
                    .padding(.top, 8)
                    .easeReveal(shown, delay: 0.12 + Double(i) * 0.025, y: 10, blur: 6)
                }
                .frame(maxWidth: .infinity)
                .zIndex(sub?.urgent == true ? 2 : (sub != nil ? 1 : 0))
            }
        }
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("Renewals this week")
        .accessibilityValue(model.active.map { "\($0.name), \($0.dayWord), \(SubsModel.money($0.price))" }
            .joined(separator: "; "))
    }

    /// Badge with its thin line down to the day. Badges drop in one after another, out of a blur.
    private func pin(_ sub: SubsItem) -> some View {
        let size: CGFloat = sub.urgent ? 68 : 52
        let order = Double(model.all.firstIndex(where: { $0.id == sub.id }) ?? 0)
        return VStack(spacing: 0) {
            EaseLogoBadge(logo: sub.brand.logo, tint: sub.brand.color, scale: sub.brand.logoScale)
                .matchedGeometryEffect(id: "logo-\(sub.id)", in: ns)
                .frame(width: size, height: size)
                .scaleEffect(shown || reduceMotion ? 1 : 0.55)
                .opacity(shown ? 1 : 0)
                .blur(radius: shown || reduceMotion ? 0 : 8)
                .offset(y: shown || reduceMotion ? 0 : -22)
                .animation(EaseMotion.spring(0.75, bounce: 0.26, delay: 0.38 + order * 0.13), value: shown)
            Rectangle()
                .fill(EaseInk.hairline)
                .frame(width: 1.5, height: sub.urgent ? 24 : 32)
                .scaleEffect(y: shown || reduceMotion ? 1 : 0, anchor: .bottom)
                .opacity(shown ? 1 : 0)
                .animation(EaseMotion.smooth(0.4, delay: 0.6 + order * 0.13), value: shown)
        }
        .fixedSize()
    }

    private func nextRow(_ sub: SubsItem) -> some View {
        HStack(spacing: 14) {
            EaseLogoBadge(logo: sub.brand.logo, tint: sub.brand.color, scale: 0.52, corner: 14, shadow: false)
                .frame(width: 50, height: 50)
            VStack(alignment: .leading, spacing: 2) {
                Text(sub.name)
                    .font(.system(size: 19, weight: .semibold))
                    .foregroundStyle(EaseInk.primary)
                Text(sub.dayWord)
                    .font(.system(size: 15))
                    .foregroundStyle(EaseInk.secondary)
            }
            Spacer()
            Text(SubsModel.money(sub.price))
                .font(.system(size: 19, weight: .medium))
                .monospacedDigit()
                .foregroundStyle(EaseInk.primary)
        }
        .padding(12)
        .overlay(RoundedRectangle(cornerRadius: 22, style: .continuous).strokeBorder(EaseInk.hairline, lineWidth: 1))
        .accessibilityElement(children: .combine)
        .accessibilityLabel("Next up: \(sub.name), \(sub.dayWord), \(SubsModel.money(sub.price))")
    }
}
```

### `SubsReviewView.swift`

```swift
import SwiftUI

/// Screen 2 (focus): one card per renewal, keep or cancel.
@MainActor
struct SubsReviewView: View {
    var model: SubsModel
    var ns: Namespace.ID
    @State private var shown = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        VStack(spacing: 0) {
            HStack {
                Text("Review")
                Spacer()
                Text("\(min(model.index + 1, model.queue.count)) of \(model.queue.count)")
                    .contentTransition(.numericText(value: Double(model.index)))
                    .monospacedDigit()
                    .animation(EaseMotion.spring(0.5), value: model.index)
            }
            .font(.system(size: 18, weight: .medium))
            .foregroundStyle(EaseInk.secondary)
            .easeReveal(shown)

            HStack(spacing: 8) {
                ForEach(0..<model.queue.count, id: \.self) { i in
                    Capsule()
                        .fill(EaseInk.hairline)
                        .overlay(alignment: .leading) {
                            GeometryReader { g in
                                Capsule()
                                    .fill(EaseInk.black)
                                    .frame(width: i <= model.index && shown ? g.size.width : 0)
                                    .animation(EaseMotion.smooth(0.6, delay: i == 0 ? 0.25 : 0.05), value: model.index)
                                    .animation(EaseMotion.smooth(0.6, delay: 0.25), value: shown)
                            }
                        }
                        .clipShape(Capsule())
                        .frame(height: 6)
                }
            }
            .padding(.top, 12)
            .accessibilityHidden(true)
            .easeReveal(shown, delay: 0.05)

            ZStack(alignment: .top) {
                ForEach(Array(model.queue.enumerated()), id: \.element.id) { i, sub in
                    card(sub, position: i)
                }
            }
            .padding(.top, 18)

            Spacer(minLength: 12)

            HStack(spacing: 12) {
                Button("Cancel") { model.userDecide(.cancel) }
                    .buttonStyle(SubsPillStyle(kind: .outline, forced: model.pressed == "cancel"))
                Button("Keep") { model.userDecide(.keep) }
                    .buttonStyle(SubsPillStyle(forced: model.pressed == "keep"))
            }
            .easeReveal(shown, delay: 0.3)
        }
        .onAppear { shown = true }
    }

    /// The decided card flies out to its side and blurs as it goes. With Reduce Motion it only fades.
    private func card(_ sub: SubsItem, position i: Int) -> some View {
        let depth = max(i - model.index, 0)
        let choice = model.choices[sub.id]
        let gone = choice != nil
        let fly = gone && !reduceMotion
        let side: CGFloat = choice == .keep ? 1 : -1
        return SubsReviewCard(sub: sub, showContent: i == model.index || gone, shown: shown,
                              hero: i == 0 && !reduceMotion, ns: ns)
            .scaleEffect(1 - CGFloat(depth) * 0.055, anchor: .bottom)
            .offset(y: CGFloat(depth) * 13)
            .rotationEffect(.degrees(fly ? Double(side) * 10 : 0), anchor: .bottom)
            .offset(x: fly ? side * 470 : 0, y: fly ? 34 : 0)
            .blur(radius: fly ? 12 : 0)
            .opacity(gone ? 0 : 1)
            .zIndex(Double(model.queue.count - i))
            .accessibilityHidden(i != model.index)
    }
}

struct SubsReviewCard: View {
    let sub: SubsItem
    let showContent: Bool
    let shown: Bool
    let hero: Bool
    var ns: Namespace.ID

    var body: some View {
        VStack(spacing: 0) {
            logo.frame(width: 96, height: 96)

            VStack(spacing: 0) {
                Text(sub.name)
                    .font(.system(size: 21, weight: .semibold))
                    .foregroundStyle(EaseInk.primary)
                    .padding(.top, 14)
                    .easeReveal(shown, delay: 0.14)

                Text(SubsModel.money(sub.price))
                    .font(.system(size: 54, weight: .bold))
                    .tracking(-1.5)
                    .monospacedDigit()
                    .foregroundStyle(EaseInk.primary)
                    .easeReveal(shown, delay: 0.18)

                Text(sub.renewLabel)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(sub.urgent ? EaseBrand.warnText : EaseInk.secondary)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 7)
                    .background(Capsule().fill(sub.urgent ? EaseBrand.warnFill : Color.white))
                    .padding(.top, 6)
                    .easeReveal(shown, delay: 0.22)

                SubsHairline().padding(.top, 20).easeReveal(shown, delay: 0.26, y: 0)

                row("Last opened", sub.lastOpened).padding(.top, 16).easeReveal(shown, delay: 0.3)
                row("Paid so far", SubsModel.money(sub.paidSoFar)).padding(.top, 10).easeReveal(shown, delay: 0.34)
            }
        }
        // Cards behind the top one keep their content hidden until it is their turn.
        .opacity(showContent ? 1 : 0)
        .blur(radius: showContent ? 0 : 10)
        .animation(EaseMotion.smooth(0.55, delay: 0.1), value: showContent)
        .padding(.horizontal, 22)
        .padding(.top, 26)
        .padding(.bottom, 22)
        .frame(maxWidth: .infinity)
        .background(
            RoundedRectangle(cornerRadius: 30, style: .continuous)
                .fill(EaseInk.fill)
                .overlay(RoundedRectangle(cornerRadius: 30, style: .continuous).strokeBorder(Color.white, lineWidth: 2))
                .opacity(shown ? 1 : 0)
                .scaleEffect(shown ? 1 : 0.96)
                .animation(EaseMotion.smooth(0.7, delay: 0.08), value: shown)
        )
        .accessibilityElement(children: .combine)
    }

    /// Only the first card shares its logo with the week strip, so that badge can travel into the card.
    @ViewBuilder private var logo: some View {
        let badge = EaseLogoBadge(logo: sub.brand.logo, tint: sub.brand.color, scale: sub.brand.logoScale, shadow: false)
        if hero {
            badge.matchedGeometryEffect(id: "logo-\(sub.id)", in: ns)
        } else {
            badge
        }
    }

    private func row(_ label: String, _ value: String) -> some View {
        HStack {
            Text(label)
                .font(.system(size: 16))
                .foregroundStyle(EaseInk.secondary)
            Spacer()
            Text(value)
                .font(.system(size: 16, weight: .semibold))
                .monospacedDigit()
                .foregroundStyle(EaseInk.primary)
        }
    }
}
```

### `SubsSummaryView.swift`

```swift
import SwiftUI

/// Screen 3 (result): what the review saves every month.
@MainActor
struct SubsSummaryView: View {
    var model: SubsModel
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
                Text("You save")
                    .font(.system(size: 19, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .padding(.top, 24)
                    .easeReveal(shown, delay: 0.18)

                SubsRollingMoney(value: model.saved)
                    .font(.system(size: 62, weight: .bold))
                    .tracking(-2)
                    .foregroundStyle(EaseInk.primary)
                    .fixedSize()
                    .easeReveal(shown, delay: 0.24)

                Text("every month")
                    .font(.system(size: 19, weight: .medium))
                    .foregroundStyle(EaseInk.secondary)
                    .easeReveal(shown, delay: 0.3)
            }
            .accessibilityElement(children: .combine)

            HStack(spacing: 8) {
                ForEach(Array(model.queue.enumerated()), id: \.element.id) { i, sub in
                    result(sub).easeReveal(shown, delay: 0.5 + Double(i) * 0.09)
                }
            }
            .padding(.top, 28)

            Spacer(minLength: 0)

            Button("Done") { model.userDone() }
                .buttonStyle(SubsPillStyle(forced: model.pressed == "done"))
                .easeReveal(shown, delay: 0.4)
        }
        .onAppear { shown = true }
    }

    private func result(_ sub: SubsItem) -> some View {
        let cancelled = model.choices[sub.id] == .cancel
        return HStack(spacing: 7) {
            SubsBrandLogo(brand: sub.brand)
                .frame(width: 18, height: 18)
                .saturation(cancelled ? 0 : 1)
                .opacity(cancelled ? 0.55 : 1)
            Text(cancelled ? "Cancelled" : "Kept")
                .font(.system(size: 14, weight: .medium))
                .foregroundStyle(cancelled ? EaseInk.secondary : EaseInk.primary)
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .background(Capsule().fill(EaseInk.chip))
        .fixedSize()
        .accessibilityElement(children: .ignore)
        .accessibilityLabel("\(sub.name): \(cancelled ? "cancelled" : "kept")")
    }
}
```

### `SubsRootView.swift`

```swift
import SwiftUI

/// The strip, the white sheet and the three screens. Plays the flow once by itself on launch;
/// any real tap takes over.
@MainActor
struct SubsRootView: View {
    @State private var model = SubsModel()
    @Namespace private var ns

    private let sheetHeight: CGFloat = 640

    var body: some View {
        ZStack(alignment: .bottom) {
            LinearGradient(colors: EaseStrip.sun, startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()
                .accessibilityHidden(true)

            ZStack {
                switch model.phase {
                case .week:
                    SubsWeekView(model: model, ns: ns).transition(AnyTransition.easeBlurOut)
                case .review:
                    SubsReviewView(model: model, ns: ns).transition(AnyTransition.easeBlurOut)
                case .summary:
                    SubsSummaryView(model: model).transition(AnyTransition.easeBlurOut)
                }
            }
            .padding(.horizontal, 22)
            .padding(.top, 30)
            .padding(.bottom, 40)
            .frame(maxWidth: .infinity)
            .frame(height: sheetHeight)
            .background(
                UnevenRoundedRectangle(topLeadingRadius: 40, topTrailingRadius: 40, style: .continuous)
                    .fill(Color.white)
                    .ignoresSafeArea(edges: .bottom)
            )
        }
        .ignoresSafeArea(edges: .bottom)
        .environment(\.colorScheme, .light)
        .onAppear { model.start() }
        .onDisappear { model.stop() }
    }
}

#Preview {
    SubsRootView()
}
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
}

/// One gradient per screen, from the left end to the right end.
enum EaseStrip {
    static let sun = [Color(red: 1.0, green: 0.847, blue: 0.302), Color(red: 1.0, green: 0.541, blue: 0.239)]
}

/// The one soft orange chip for what is urgent.
enum EaseBrand {
    static let warnFill = Color(red: 1.0, green: 0.906, blue: 0.824)
    static let warnText = Color(red: 0.851, green: 0.282, blue: 0.059)
}
```

### `EaseParts.swift`

```swift
import SwiftUI

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

/// Every real brand: white circle, hairline, soft shadow, the official logo in its own colour.
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
        .accessibilityHidden(true)
    }
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
```

### `SubscriptionsApp.swift`

```swift
import SwiftUI

@main
struct SubscriptionsApp: App {
    var body: some Scene {
        WindowGroup {
            SubsRootView()
        }
    }
}
```
