# Worked example: subscriptions (the target look)

This is the source of the three screenshots `references/screens/subs-week.png`, `subs-review.png` and `subs-saved.png`. The owner approved exactly this look. For any app screen, build in this manner, with your own content: the same calm, the same sizes, the same small number of things per screen. Do not copy its data or texts.

**Logos.** The screens show Netflix, Spotify and YouTube. The code loads them as `Image("netflix")`, `Image("spotify")` and `Image("youtube")` from the asset catalog. Fetch the official files as `references/screen-layout.md` section 4 says: Simple Icons, slugs `netflix`, `spotify`, `youtube`; one image set each with "Preserve Vector Data" and "Render As: Template Image". `SubsBrand.color` holds the three brand colours as Simple Icons lists them. Name the three logos and their source in your answer. Never draw them yourself.

**What differs from the prototype.** The prototype played itself for a recording. Here every step is a real tap, and after "Done" a second review goes through what is still active. Left out because they were only for the clip: the slow factor, the warm-up, the idle phase, the touch dot on the buttons, the hidden status bar and home indicator. Added: Reduce Motion (things only fade) and the switched-off button when nothing is left. Every size, colour, delay and curve of the design is unchanged.

**In a project that has the Easeout kit**, do not keep two copies of the same part: use `EaseMotion`, `EaseInk`, `.easeReveal`, `AnyTransition.easeBlurOut` and `EaseLogoBadge` in place of `SubsMotion`, `SubsInk`, `.subsReveal`, `.subsBlurOut` and `SubsLogoBadge`, and keep the numbers of this file. As printed here the example is complete: iOS 17, no packages, Swift 6 safe.

**How to show it.** `SubsRootView()` fills the screen. Show it from the app's `WindowGroup`, a `.fullScreenCover` or the preview at the end of `SubsRootView.swift`. There is no `@main` in the example.

## The screens

| Screen | Pattern | Blocks, top to bottom | Hero |
|---|---|---|---|
| Week (`SubsWeekView`) | overview | "This week" · `$41.47` at 62 pt with the chip "3 renewals" · week strip · hairline, "Next up" and one row · "Review this week" | Seven day columns; each subscription's logo in a white circle on a thin line above the day it renews; the next one 68 pt, the others 52 pt; today in a black circle. |
| Review (`SubsReviewView`) | focus | "Review" with "1 of 3" and three progress segments · card stack · "Cancel" (outline) and "Keep" (black) | One grey card: logo in a 96 pt white circle, name, price at 54 pt, a chip (soft orange when it renews tomorrow), a hairline, "Last opened" and "Paid so far". Two more cards peek out below. |
| Saved (`SubsSummaryView`) | result | result block, centred · "Done" | A black check circle, "You save", `$15.49` at 62 pt, "every month", one chip per subscription: "Cancelled" in grey or "Kept". |

Source of truth: `SubsModel.all` (the subscriptions) and `SubsModel.choices` (keep or cancel). The week total, the count in the chip, the pins, the "Next up" row, the saved amount and the chips are all computed from these two. The example prices add up: 15.49 + 11.99 + 13.99 = 41.47.

What is not on these screens, on purpose: no ring, no panel with three values, no list of all subscriptions, no second number. The week strip already says who renews when; the cards show the details one at a time.

## The beats

1. **Enter** (appear). The week screen reveals top to bottom: title 0 s, number 0.06, the days 0.12 and 0.025 apart, hairline 0.2, caption 0.24, row 0.29, button 0.35. The badges drop in out of a blur one after another (spring 0.75 s, bounce 0.26, from 0.38 s, 0.13 apart), each thin line grows up to its badge (0.4 s, from 0.6 s). After 0.4 s the amount rolls from $0.00 to $41.47. Rest: button black, "Review this week".
2. **Review** (tap the button). The week screen blurs out. The first logo travels from its day into the card (the only `matchedGeometryEffect`, spring 0.7 s, bounce 0.14). The card's lines reveal from 0.14 to 0.34 s, the first progress segment fills. Rest: "Cancel" and "Keep".
3. **Decide** (tap "Cancel" or "Keep"). The button gives way (scale 0.965). The card leaves to the left (cancel) or right (keep): 470 pt, 10° about its bottom edge, 34 pt down, blur 12, 0.65 s. The next card grows into place (spring 0.7 s, bounce 0.14), its content fades in out of a blur (0.55 s after 0.1 s). The counter rolls and the next segment fills.
4. **Result** (0.3 s after the last card). The review blurs out. The check circle pops (spring 0.7 s, bounce 0.28), the check writes itself (0.5 s after 0.35 s), the lines reveal, the chips follow 0.09 s apart from 0.5 s. After 0.4 s the amount rolls from $0.00 to what was cancelled. Rest: button black, "Done".
5. **Back** (tap "Done"). The result blurs out and the week screen reveals again without the cancelled logo. After 0.45 s the amount rolls from $41.47 to $25.98, the chip says "2 renewals", "Next up" shows Spotify.

Every screen change is `withAnimation` on `phase`: the old screen leaves with `subsBlurOut`, the new one brings its own stagger. Nothing slides or cross-fades.

## What to take over, and what to change

- Keep: five blocks or fewer per screen, the 62 pt number, the white circles, the white space above the button, the two-pill choice, the delays and curves.
- Change: the data, the texts, the strip colours, the unit of the timeline (days here; hours, stops or weeks elsewhere), the two detail lines.
- For a showcase that plays itself, call `review()`, `decide(_:)` and `done()` from one task with `SubsMotion.wait` between them, and cancel that task when the view goes away.

## The files

#### SubsModel.swift

```swift
import SwiftUI
import Observation

// Colours taken from the approved design pictures (flat look, no glass).
enum SubsInk {
    static let primary = Color(red: 0.067, green: 0.067, blue: 0.075)
    static let secondary = Color(red: 0.557, green: 0.557, blue: 0.576)
    static let hairline = Color(red: 0.898, green: 0.898, blue: 0.918)
    static let fill = Color(red: 0.961, green: 0.961, blue: 0.965)
    static let chip = Color(red: 0.949, green: 0.949, blue: 0.953)
    static let black = Color(red: 0.122, green: 0.122, blue: 0.129)
    static let sun = Color(red: 1.0, green: 0.847, blue: 0.302)
    static let ember = Color(red: 1.0, green: 0.541, blue: 0.239)
    static let warnFill = Color(red: 1.0, green: 0.906, blue: 0.824)
    static let warnText = Color(red: 0.851, green: 0.282, blue: 0.059)
}

/// All timing goes through here.
enum SubsMotion {
    /// Soft ease-out, the main curve.
    static func smooth(_ d: Double, delay: Double = 0) -> Animation {
        .timingCurve(0.2, 0.9, 0.2, 1, duration: d).delay(delay)
    }
    static func spring(_ d: Double, bounce: Double = 0, delay: Double = 0) -> Animation {
        .spring(duration: d, bounce: bounce).delay(delay)
    }
    /// Timed steps wait here. In a cancelled task it returns at once, so the steps after it land in their rest state.
    static func wait(_ seconds: Double) async {
        try? await Task.sleep(for: .seconds(seconds))
    }
}

enum SubsBrand: String, Sendable {
    case netflix, spotify, youtube

    /// The brand colour as Simple Icons lists it. The logo files are template images and take this colour.
    var color: Color {
        switch self {
        case .netflix: return Color(red: 0.898, green: 0.035, blue: 0.078)   // #E50914
        case .spotify: return Color(red: 0.118, green: 0.843, blue: 0.376)   // #1ED760
        case .youtube: return Color(red: 1.0, green: 0.0, blue: 0.0)         // #FF0000
        }
    }
}

struct SubsItem: Identifiable, Sendable {
    let brand: SubsBrand
    let name: String
    let price: Double
    let day: Int            // column in the week strip, 0 = Monday
    let dayWord: String     // shown in the "Next up" row
    let renewLabel: String  // chip on the review card
    let urgent: Bool
    let lastOpened: String
    let paidSoFar: String
    var id: String { brand.rawValue }

    // Example data. The prices add up: 15.49 + 11.99 + 13.99 = 41.47.
    static let examples: [SubsItem] = [
        SubsItem(brand: .netflix, name: "Netflix", price: 15.49, day: 3, dayWord: "Tomorrow",
                 renewLabel: "Renews tomorrow", urgent: true, lastOpened: "34 days ago", paidSoFar: "$185.88"),
        SubsItem(brand: .spotify, name: "Spotify", price: 11.99, day: 4, dayWord: "Friday",
                 renewLabel: "Renews Friday", urgent: false, lastOpened: "Today", paidSoFar: "$143.88"),
        SubsItem(brand: .youtube, name: "YouTube Premium", price: 13.99, day: 6, dayWord: "Sunday",
                 renewLabel: "Renews Sunday", urgent: false, lastOpened: "Yesterday", paidSoFar: "$83.94"),
    ]
}

enum SubsPhase: Equatable { case week, review, summary }
enum SubsChoice: Equatable { case keep, cancel }

/// The source of truth. Every amount, count and chip on the three screens is computed from `all` and `choices`.
@MainActor @Observable
final class SubsModel {
    let all = SubsItem.examples
    var phase: SubsPhase = .week
    var queue: [SubsItem] = []           // the cards of the running review
    var index = 0                        // review card on top
    var choices: [String: SubsChoice] = [:]
    var total: Double = 0                // amount shown on the week screen
    var saved: Double = 0                // amount shown on the summary
    @ObservationIgnored private var timed: Task<Void, Never>?

    var active: [SubsItem] { all.filter { choices[$0.id] != .cancel } }
    var weekTotal: Double { active.reduce(0) { $0 + $1.price } }
    var cancelledTotal: Double { queue.filter { choices[$0.id] == .cancel }.reduce(0) { $0 + $1.price } }
    var renewalsLabel: String { active.count == 1 ? "1 renewal" : "\(active.count) renewals" }

    /// The week screen is on: a moment later the amount rolls up.
    func start() { roll(after: 0.4) }

    /// "Review this week": go through everything that is still active.
    func review() {
        guard phase == .week, !active.isEmpty else { return }
        choices = choices.filter { $0.value == .cancel }
        queue = active
        index = 0
        withAnimation(SubsMotion.spring(0.7, bounce: 0.14)) { phase = .review }
    }

    /// "Keep" or "Cancel" for the card on top. After the last card the summary comes in.
    func decide(_ choice: SubsChoice) {
        guard phase == .review, index < queue.count, choices[queue[index].id] == nil else { return }
        let sub = queue[index]
        withAnimation(SubsMotion.smooth(0.65)) { choices[sub.id] = choice }
        if index < queue.count - 1 {
            withAnimation(SubsMotion.spring(0.7, bounce: 0.14)) { index += 1 }
        } else {
            timed?.cancel()
            timed = Task {
                await SubsMotion.wait(0.3)
                saved = 0
                withAnimation(SubsMotion.smooth(0.5)) { phase = .summary }
                await SubsMotion.wait(0.4)
                saved = cancelledTotal
            }
        }
    }

    /// "Done": back to the week, then the amount rolls to the new total.
    func done() {
        guard phase == .summary else { return }
        withAnimation(SubsMotion.smooth(0.5)) { phase = .week }
        roll(after: 0.45)
    }

    /// The view went away: no step keeps waiting.
    func stop() { timed?.cancel() }

    private func roll(after seconds: Double) {
        timed?.cancel()
        timed = Task {
            await SubsMotion.wait(seconds)
            total = weekTotal
        }
    }
}
```

#### SubsParts.swift

```swift
import SwiftUI

/// The official logo from the asset catalog, in the brand's own colour.
/// The three image sets are template images. For a full-colour logo file, leave out `renderingMode` and `foregroundStyle`.
struct SubsBrandLogo: View {
    let brand: SubsBrand

    var body: some View {
        image
            .renderingMode(.template)
            .resizable()
            .scaledToFit()
            .foregroundStyle(brand.color)
    }

    private var image: Image {
        switch brand {
        case .netflix: return Image("netflix")
        case .spotify: return Image("spotify")
        case .youtube: return Image("youtube")
        }
    }
}

/// Plain white circle with a hairline and the logo inside. It fills whatever frame it is given.
struct SubsLogoBadge: View {
    let brand: SubsBrand
    var shadow = true

    var body: some View {
        GeometryReader { g in
            let s = min(g.size.width, g.size.height)
            let logo = s * (brand == .spotify ? 0.64 : 0.54)   // the Spotify logo is a disc itself, so it sits a little larger
            ZStack {
                Circle().fill(.white)
                Circle().strokeBorder(SubsInk.hairline, lineWidth: 1)
                SubsBrandLogo(brand: brand)
                    .frame(width: logo, height: logo)
            }
            .frame(width: g.size.width, height: g.size.height)
        }
        .shadow(color: .black.opacity(shadow ? 0.07 : 0), radius: 7, y: 3)
    }
}

/// Flat pill button: black, white with an outline, or switched off. It gives way a little while it is pressed.
struct SubsPillStyle: ButtonStyle {
    enum Kind { case solid, outline, off }
    var kind: Kind = .solid

    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .font(.system(size: 18, weight: .semibold))
            .foregroundStyle(kind == .solid ? Color.white : (kind == .off ? SubsInk.secondary : SubsInk.primary))
            .frame(maxWidth: .infinity)
            .frame(height: 54)
            .background(Capsule().fill(kind == .solid ? SubsInk.black : (kind == .off ? SubsInk.chip : Color.white)))
            .overlay(Capsule().strokeBorder(SubsInk.hairline, lineWidth: kind == .outline ? 1.5 : 0))
            .contentShape(Capsule())
            .scaleEffect(configuration.isPressed ? 0.965 : 1)
            .animation(configuration.isPressed ? SubsMotion.spring(0.2) : SubsMotion.spring(0.35, bounce: 0.2),
                       value: configuration.isPressed)
    }
}

/// Dollar amount whose digits roll softly to the new value.
struct SubsRollingMoney: View {
    let value: Double
    var body: some View {
        Text(String(format: "$%.2f", value))
            .monospacedDigit()
            .contentTransition(.numericText(value: value))
            .animation(SubsMotion.spring(0.9), value: value)
    }
}

/// Element comes in from a soft blur, a little lower and smaller. Used with small delays for a stagger.
/// With Reduce Motion it only fades.
struct SubsReveal: ViewModifier {
    let on: Bool
    var delay: Double = 0
    var y: CGFloat = 14
    var blur: CGFloat = 10
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        content
            .opacity(on ? 1 : 0)
            .blur(radius: on || reduceMotion ? 0 : blur)
            .scaleEffect(on || reduceMotion ? 1 : 0.98)
            .offset(y: on || reduceMotion ? 0 : y)
            .animation(SubsMotion.smooth(0.7, delay: delay), value: on)
    }
}

extension View {
    func subsReveal(_ on: Bool, delay: Double = 0, y: CGFloat = 14, blur: CGFloat = 10) -> some View {
        modifier(SubsReveal(on: on, delay: delay, y: y, blur: blur))
    }
}

/// A screen leaves by melting into a blur. New screens bring their own staggered reveal.
struct SubsBlurOut: ViewModifier {
    let active: Bool
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    func body(content: Content) -> some View {
        content
            .opacity(active ? 0 : 1)
            .blur(radius: active && !reduceMotion ? 16 : 0)
            .scaleEffect(active && !reduceMotion ? 0.97 : 1)
            .offset(y: active && !reduceMotion ? -8 : 0)
    }
}

extension AnyTransition {
    @MainActor static var subsBlurOut: AnyTransition {
        .asymmetric(insertion: .identity,
                    removal: .modifier(active: SubsBlurOut(active: true), identity: SubsBlurOut(active: false)))
    }
}

struct SubsHairline: View {
    var body: some View { Rectangle().fill(SubsInk.hairline).frame(height: 1) }
}
```

#### SubsWeekView.swift

```swift
import SwiftUI

/// Screen 1: what leaves your account this week, with each logo pinned to the day it renews.
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
                .foregroundStyle(SubsInk.secondary)
                .subsReveal(shown)

            HStack(alignment: .center, spacing: 14) {
                SubsRollingMoney(value: model.total)
                    .font(.system(size: 62, weight: .bold))
                    .tracking(-2)
                    .foregroundStyle(SubsInk.primary)
                    .fixedSize()
                Text(model.renewalsLabel)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(SubsInk.secondary)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 8)
                    .background(Capsule().fill(SubsInk.chip))
                    .fixedSize()
            }
            .padding(.top, 2)
            .subsReveal(shown, delay: 0.06)

            strip.padding(.top, 14)

            SubsHairline().padding(.top, 20).subsReveal(shown, delay: 0.2, y: 0)

            Text("Next up")
                .font(.system(size: 16, weight: .medium))
                .foregroundStyle(SubsInk.secondary)
                .padding(.top, 20)
                .subsReveal(shown, delay: 0.24)

            if let next = model.active.first {
                nextRow(next).padding(.top, 12).subsReveal(shown, delay: 0.29)
            }

            Spacer(minLength: 14)

            Button("Review this week") { model.review() }
                .buttonStyle(SubsPillStyle(kind: model.active.isEmpty ? .off : .solid))
                .disabled(model.active.isEmpty)
                .subsReveal(shown, delay: 0.35)
        }
        .onAppear { shown = true }
    }

    private var strip: some View {
        HStack(alignment: .bottom, spacing: 0) {
            ForEach(0..<7, id: \.self) { i in
                let sub = model.active.first(where: { $0.day == i })
                VStack(spacing: 0) {
                    // The badge sits in an overlay so a wide badge cannot push the seven day columns apart.
                    Color.clear
                        .frame(height: 100)
                        .overlay(alignment: .bottom) {
                            if let sub { pin(sub) }
                        }
                    VStack(spacing: 8) {
                        Text(letters[i])
                            .font(.system(size: 16, weight: .medium))
                            .foregroundStyle(SubsInk.secondary)
                        ZStack {
                            if i == today { Circle().fill(SubsInk.black).frame(width: 40, height: 40) }
                            Text("\(numbers[i])")
                                .font(.system(size: 21, weight: .semibold))
                                .foregroundStyle(i == today ? Color.white : SubsInk.primary)
                        }
                        .frame(height: 40)
                    }
                    .padding(.top, 8)
                    .subsReveal(shown, delay: 0.12 + Double(i) * 0.025, y: 10, blur: 6)
                }
                .frame(maxWidth: .infinity)
                .zIndex(sub?.urgent == true ? 2 : (sub != nil ? 1 : 0))
            }
        }
    }

    /// Logo badge with its thin line down to the day. Badges drop in one after another, out of a blur.
    private func pin(_ sub: SubsItem) -> some View {
        let size: CGFloat = sub.urgent ? 68 : 52
        let order = Double(model.all.firstIndex(where: { $0.id == sub.id }) ?? 0)
        return VStack(spacing: 0) {
            SubsLogoBadge(brand: sub.brand)
                .matchedGeometryEffect(id: "logo-\(sub.id)", in: ns)
                .frame(width: size, height: size)
                .scaleEffect(shown || reduceMotion ? 1 : 0.55)
                .opacity(shown ? 1 : 0)
                .blur(radius: shown || reduceMotion ? 0 : 8)
                .offset(y: shown || reduceMotion ? 0 : -22)
                .animation(SubsMotion.spring(0.75, bounce: 0.26, delay: 0.38 + order * 0.13), value: shown)
            Rectangle()
                .fill(SubsInk.hairline)
                .frame(width: 1.5, height: sub.urgent ? 24 : 32)
                .scaleEffect(y: shown ? 1 : 0, anchor: .bottom)
                .animation(SubsMotion.smooth(0.4, delay: 0.6 + order * 0.13), value: shown)
        }
        .fixedSize()
    }

    private func nextRow(_ sub: SubsItem) -> some View {
        HStack(spacing: 14) {
            ZStack {
                RoundedRectangle(cornerRadius: 14, style: .continuous).fill(.white)
                RoundedRectangle(cornerRadius: 14, style: .continuous).strokeBorder(SubsInk.hairline, lineWidth: 1)
                SubsBrandLogo(brand: sub.brand).frame(width: 26, height: 26)
            }
            .frame(width: 50, height: 50)
            VStack(alignment: .leading, spacing: 2) {
                Text(sub.name)
                    .font(.system(size: 19, weight: .semibold))
                    .foregroundStyle(SubsInk.primary)
                Text(sub.dayWord)
                    .font(.system(size: 15))
                    .foregroundStyle(SubsInk.secondary)
            }
            Spacer()
            Text(String(format: "$%.2f", sub.price))
                .font(.system(size: 19, weight: .medium))
                .foregroundStyle(SubsInk.primary)
        }
        .padding(12)
        .overlay(RoundedRectangle(cornerRadius: 22, style: .continuous).strokeBorder(SubsInk.hairline, lineWidth: 1))
    }
}
```

#### SubsReviewView.swift

```swift
import SwiftUI

/// Screen 2: one card per renewal, keep or cancel.
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
            }
            .font(.system(size: 18, weight: .medium))
            .foregroundStyle(SubsInk.secondary)
            .subsReveal(shown)

            HStack(spacing: 8) {
                ForEach(0..<model.queue.count, id: \.self) { i in
                    Capsule()
                        .fill(SubsInk.hairline)
                        .overlay(alignment: .leading) {
                            GeometryReader { g in
                                Capsule()
                                    .fill(SubsInk.black)
                                    .frame(width: i <= model.index && shown ? g.size.width : 0)
                                    .animation(SubsMotion.smooth(0.6, delay: i == 0 ? 0.25 : 0.05), value: model.index)
                                    .animation(SubsMotion.smooth(0.6, delay: 0.25), value: shown)
                            }
                        }
                        .clipShape(Capsule())
                        .frame(height: 6)
                }
            }
            .padding(.top, 12)
            .subsReveal(shown, delay: 0.05)

            ZStack(alignment: .top) {
                ForEach(Array(model.queue.enumerated()), id: \.element.id) { i, sub in
                    card(sub, position: i)
                }
            }
            .padding(.top, 18)

            Spacer(minLength: 12)

            HStack(spacing: 12) {
                Button("Cancel") { model.decide(.cancel) }
                    .buttonStyle(SubsPillStyle(kind: .outline))
                Button("Keep") { model.decide(.keep) }
                    .buttonStyle(SubsPillStyle())
            }
            .subsReveal(shown, delay: 0.3)
        }
        .onAppear { shown = true }
    }

    /// Cards sit in a small stack. The decided card flies out to its side and blurs as it goes.
    /// With Reduce Motion it only fades.
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
                    .foregroundStyle(SubsInk.primary)
                    .padding(.top, 14)
                    .subsReveal(shown, delay: 0.14)

                Text(String(format: "$%.2f", sub.price))
                    .font(.system(size: 54, weight: .bold))
                    .tracking(-1.5)
                    .foregroundStyle(SubsInk.primary)
                    .subsReveal(shown, delay: 0.18)

                Text(sub.renewLabel)
                    .font(.system(size: 15, weight: .medium))
                    .foregroundStyle(sub.urgent ? SubsInk.warnText : SubsInk.secondary)
                    .padding(.horizontal, 14)
                    .padding(.vertical, 7)
                    .background(Capsule().fill(sub.urgent ? SubsInk.warnFill : Color.white))
                    .padding(.top, 6)
                    .subsReveal(shown, delay: 0.22)

                SubsHairline().padding(.top, 20).subsReveal(shown, delay: 0.26, y: 0)

                row("Last opened", sub.lastOpened).padding(.top, 16).subsReveal(shown, delay: 0.3)
                row("Paid so far", sub.paidSoFar).padding(.top, 10).subsReveal(shown, delay: 0.34)
            }
        }
        // Cards behind the top one keep their text hidden until it is their turn.
        .opacity(showContent ? 1 : 0)
        .blur(radius: showContent ? 0 : 10)
        .animation(SubsMotion.smooth(0.55, delay: 0.1), value: showContent)
        .padding(.horizontal, 22)
        .padding(.top, 26)
        .padding(.bottom, 22)
        .frame(maxWidth: .infinity)
        .background(
            RoundedRectangle(cornerRadius: 30, style: .continuous)
                .fill(SubsInk.fill)
                .overlay(RoundedRectangle(cornerRadius: 30, style: .continuous).strokeBorder(Color.white, lineWidth: 2))
                .opacity(shown ? 1 : 0)
                .scaleEffect(shown ? 1 : 0.96)
                .animation(SubsMotion.smooth(0.7, delay: 0.08), value: shown)
        )
    }

    /// Only the first card shares its logo with the week strip, so that badge can fly into the card.
    @ViewBuilder private var logo: some View {
        if hero {
            SubsLogoBadge(brand: sub.brand, shadow: false)
                .matchedGeometryEffect(id: "logo-\(sub.id)", in: ns)
        } else {
            SubsLogoBadge(brand: sub.brand, shadow: false)
        }
    }

    private func row(_ label: String, _ value: String) -> some View {
        HStack {
            Text(label)
                .font(.system(size: 16))
                .foregroundStyle(SubsInk.secondary)
            Spacer()
            Text(value)
                .font(.system(size: 16, weight: .semibold))
                .foregroundStyle(SubsInk.primary)
        }
    }
}
```

#### SubsSummaryView.swift

```swift
import SwiftUI

/// Screen 3: what the review saved.
@MainActor
struct SubsSummaryView: View {
    var model: SubsModel
    @State private var shown = false
    @Environment(\.accessibilityReduceMotion) private var reduceMotion

    var body: some View {
        VStack(spacing: 0) {
            Spacer(minLength: 0)

            ZStack {
                Circle().fill(SubsInk.black)
                SubsCheckShape()
                    .trim(from: 0, to: shown ? 1 : 0)
                    .stroke(Color.white, style: StrokeStyle(lineWidth: 5, lineCap: .round, lineJoin: .round))
                    .frame(width: 34, height: 26)
                    .animation(SubsMotion.smooth(0.5, delay: 0.35), value: shown)
            }
            .frame(width: 84, height: 84)
            .scaleEffect(shown || reduceMotion ? 1 : 0.6)
            .opacity(shown ? 1 : 0)
            .blur(radius: shown || reduceMotion ? 0 : 10)
            .animation(SubsMotion.spring(0.7, bounce: 0.28, delay: 0.05), value: shown)

            Text("You save")
                .font(.system(size: 19, weight: .medium))
                .foregroundStyle(SubsInk.secondary)
                .padding(.top, 24)
                .subsReveal(shown, delay: 0.18)

            SubsRollingMoney(value: model.saved)
                .font(.system(size: 62, weight: .bold))
                .tracking(-2)
                .foregroundStyle(SubsInk.primary)
                .fixedSize()
                .subsReveal(shown, delay: 0.24)

            Text("every month")
                .font(.system(size: 19, weight: .medium))
                .foregroundStyle(SubsInk.secondary)
                .subsReveal(shown, delay: 0.3)

            HStack(spacing: 8) {
                ForEach(Array(model.queue.enumerated()), id: \.element.id) { i, sub in
                    result(sub).subsReveal(shown, delay: 0.5 + Double(i) * 0.09)
                }
            }
            .padding(.top, 28)

            Spacer(minLength: 0)

            Button("Done") { model.done() }
                .buttonStyle(SubsPillStyle())
                .subsReveal(shown, delay: 0.4)
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
                .foregroundStyle(cancelled ? SubsInk.secondary : SubsInk.primary)
        }
        .padding(.horizontal, 12)
        .padding(.vertical, 8)
        .background(Capsule().fill(SubsInk.chip))
        .fixedSize()
    }
}

struct SubsCheckShape: Shape {
    func path(in rect: CGRect) -> Path {
        var p = Path()
        p.move(to: CGPoint(x: rect.minX, y: rect.midY + rect.height * 0.08))
        p.addLine(to: CGPoint(x: rect.minX + rect.width * 0.36, y: rect.maxY))
        p.addLine(to: CGPoint(x: rect.maxX, y: rect.minY))
        return p
    }
}
```

#### SubsRootView.swift

```swift
import SwiftUI

/// The strip, the white sheet and the three screens. Show it from the app's own entry point; it has no `@main`.
@MainActor
struct SubsRootView: View {
    @State private var model = SubsModel()
    @Namespace private var ns

    /// Height of the white sheet.
    private let sheetHeight: CGFloat = 640

    var body: some View {
        ZStack(alignment: .bottom) {
            LinearGradient(colors: [SubsInk.sun, SubsInk.ember], startPoint: .leading, endPoint: .trailing)
                .ignoresSafeArea()

            ZStack {
                switch model.phase {
                case .week:
                    SubsWeekView(model: model, ns: ns).transition(AnyTransition.subsBlurOut)
                case .review:
                    SubsReviewView(model: model, ns: ns).transition(AnyTransition.subsBlurOut)
                case .summary:
                    SubsSummaryView(model: model).transition(AnyTransition.subsBlurOut)
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
        .onAppear { model.start() }
        .onDisappear { model.stop() }
    }
}

#Preview {
    SubsRootView()
}
```
