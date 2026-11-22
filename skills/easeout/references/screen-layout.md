# Easeout screen layout and imagery

The look to hit is calm and big: few things on a screen, one huge number, one simple hero, real logos in white circles, a lot of white, a black pill button. Read this file for every screen you build in job 1. A project with its own design system keeps its colours, fonts and components and still takes "big and few", the three screen patterns and the logo presentation from here. The code needs iOS 17, compiles in Swift 6 mode and uses `EaseInk`, `EaseMotion`, `.easeReveal`, `EaseSheet` and `EasePillButton` as `motion-system.md` defines them.

## 1. Look at the screenshots first

**For any app screen, open these three first**, together with `references/examples/subscriptions.md`, the source that draws them. They are the approved target: build in this manner, with your own content. Open them again next to your running screen. They are cropped to the sheet.

| File | Pattern | What to see |
|---|---|---|
| `references/screens/subs-week.png` | overview | Five blocks: a grey title; `$41.47` at 62 pt with one grey chip beside it; a week strip with three logos in white circles pinned above their days, the next one larger, today in a black circle; a hairline, "Next up" and one row; the black pill. About 90 pt of white above the button stay empty. |
| `references/screens/subs-review.png` | focus | A counter with three progress segments; one centred grey card with the logo in a 96 pt white circle, the name, the price at 54 pt, one chip, a hairline and two quiet detail lines; two more cards peek out below it; a white outline pill next to the black pill. |
| `references/screens/subs-saved.png` | result | Centred in the white: a black check circle, "You save", `$15.49` at 62 pt, "every month", three small chips with a logo and one word. One black pill. Nothing else. |

The six scene screenshots are for the **hero only**, when your data asks for one of theirs: `flight.png` (two codes joined by a route: travel, delivery, a transfer) · `voice.png` (a waveform: media, recording, timers) · `split.png` (a ring with the total inside: shares of one whole) · `portfolio.png` (a line chart with a soft fill: a value over time) · `weather.png` (a curve with a glowing marker on a gradient panel: one measured value over a day) · `lift.png` (colour blocks on a grey panel: counts, things you assemble). All are in `references/screens/`. The scenes are denser than the target: take the hero and its code (`scenes/<id>.md`), not the number of blocks. Their logos and portraits were replaced by tiles with two letters; never copy the letters.

## 2. Big and few

- **One job per screen**: an overview ("what is coming"), a focus ("decide this one") or a result ("this is what changed"). A screen that wants to do two of them becomes two screens with a blur-out between them.
- **Four or five blocks**, counted from the top of the sheet to its bottom: title line, the number, the hero, one quiet row, the button. The week screen has five, the review screen three (counter, card stack, buttons), the saved screen two (result block, button). Count yours before you build. A sixth block goes to another screen.
- **One huge number**: 62 bold, tracking −2 (54 bold, −1.5 inside a card). It is the one value the screen is about. Nothing else on the screen is larger than 21 pt. What belongs to the number (a count, a unit, a change) sits beside it as one small grey chip, never as a second number.
- **One simple hero** (section 5) that you can describe in one sentence: "logos pinned above the days of a week", "one card with a price", "a check and a sentence". Its colour comes from the logos; the rest is white, grey and black.
- **White space is part of the design, in one place and in one amount.** 60 to 120 pt of empty white above the button is right. Never add a block, a caption or a value to fill the sheet; a `Spacer` takes what is left. **More than 140 pt empty is a hole, not calm**: then the hero grows into it (a taller graph, a larger card, a bigger result block) until 60 to 120 pt remain. Check every screen of the flow, also the live one and the result; the screen with the fewest blocks is the one that ends up half empty.
- **No stat panel with three values, no ring, no chart, no legend**, unless the question of the screen is exactly that (shares of one whole: a ring; a value over time: a chart). Then it is the hero and replaces the other one. A panel with three values only when all three are needed to act, and then the row below it goes.
- **Rows only for "Next up" or a short list**: one row under a grey caption, at most three. The full list is its own screen. Details are quiet lines inside the card they belong to: grey label left, semibold value right, 16 pt, two of them.
- **Do not repeat.** What the hero shows (who renews on which day) is not listed again below it. What the focus card shows is not on the overview.
- **Test**: cover each block with a finger. If the screen still answers its question, delete that block.

## 3. The sheet and its blocks

| Block, top to bottom | Exact look (sizes in pt) |
|---|---|
| 1. Strip | Fills the screen behind the status bar; about 210 pt stay visible above the sheet. One gradient from leading to trailing, in the colour of what the screen is about (section 6). |
| 2. Sheet | White, 640 high, top corners 40 continuous, padding 22 left and right, 30 top, 40 bottom (set these in `EaseSheet`; installed scenes keep their own values). The white reaches below the screen edge. |
| 3. Title line | 18 medium, secondary, left ("This week"). On a focus screen a counter on the right ("2 of 3", rolling) and, 12 lower, one capsule segment per card: 6 high, gap 8, `EaseInk.hairline`, filled with `EaseInk.black` from the leading edge. |
| 4. The number | 2 under the title: 62 bold, −2, `monospacedDigit`, `numericText`. Beside it, 14 apart and centred on it, one chip: capsule `EaseInk.chip`, padding 14 × 8, 15 medium secondary ("3 renewals"). On a result screen the number is centred between two lines in 19 medium secondary. |
| 5. Hero | Section 5. Full width, 14 to 18 under the block above. Week strip 175 high; focus card about 330; result block about 290, centred between two spacers; live hero 240 to 300 (whatever leaves 60 to 120 pt white above the button). |
| 6. Quiet row | A 1 pt hairline 20 under the hero; 20 lower a caption (16 medium secondary, "Next up"); 12 lower one row: padding 12, 1 pt `EaseInk.hairline` outline, radius 22; a 50 pt white logo square, 14 gap, the name 19 semibold over one word 15 secondary, on the right the value 19 medium. |
| 7. Button | Pinned to the bottom of the sheet. One `EasePillButton`: black capsule, 54 high, full width, title 18 semibold white. For a choice two pills side by side, 12 apart: left the white outline pill for "no" (white capsule, 1.5 pt `EaseInk.hairline` outline, primary text), right the black pill for "yes" (`SubsPillStyle` in the example draws both). Pressed: scale 0.965. |

The sheet holds 570 pt (640 − 30 − 40). The week screen adds up to 487: title 22, number 76, strip 189, hairline 21, caption 39, row 86, button 54. The other 83 stay white above the button. Build with `VStack(alignment: .leading, spacing: 0)`, one `.padding(.top, …)` per block and one `Spacer(minLength: 12)` above the button; each block is one step of the stagger. If the content does not fit, remove a block; do not shrink the type. Check 375 pt width: the number and its chip stay on one line.

## 4. Logos and pictures

- **Real brands get their real logos, on white.** When the data names a real company (Netflix, Spotify, Lufthansa), show its official logo in an `EaseLogoBadge`: a white circle, a 1 pt hairline, a soft shadow, the logo in its own colour. Never on a coloured tile, never white on the brand colour. Do this without asking; leave logos out only if the user said so.
- **Sizes**: pinned on a timeline 52, the next or urgent one 68 · on a focus card 96 with `shadow: false` · in a row a 50 pt white rounded square (`corner: 14, shadow: false, scale: 0.52`) · in a chip the bare logo at 18 pt.
- **The official file**: Simple Icons first (`https://cdn.jsdelivr.net/npm/simple-icons@15/icons/<slug>.svg`; the slug is the lower-case name without spaces, for example `netflix`, `spotify`, `youtube`; the brand colour is in `simple-icons@15/data/simple-icons.json`), the brand's own press or media page second. Add the SVG or PDF to the asset catalog as a single-scale image set with "Preserve Vector Data" checked. A one-colour glyph (every Simple Icons file) is set to "Render As: Template Image" and gets the brand colour through `tint`; a full-colour logo gets no tint. Do not stretch, crop or recolour it in any other way. List each logo and its source in your answer, so that the user can check the brand's terms.
- **Never generate a logo with an image model, and never redraw one from memory** as a path, a shape or lookalike letters. A logo is the official file, or it is not there. If no official file exists or the user said no: the same white badge with an SF Symbol in the brand's colour (`logo: Image(systemName: "play.fill")`).
- **`EaseIconTile` is only for invented things without a logo**: an example brand ("Tunebox"), a category, an item. A filled SF Symbol, white, on one `EaseBrand` colour; neighbours get different colours; at most three tiles on a screen. On a screen that also shows real logos, put the symbol in the white badge instead, so that the screen has one kind of picture.
- **Never a letter monogram**: no letter in a tile or a circle, no empty grey square. Initials are for people without a photo, in `EaseAvatar`.
- **Pictures.** People, places, products and media get real pictures: the user's own images first; for example data free photos (Unsplash, under the Unsplash licence) in the asset catalog, each named with its photographer in your answer. Show them clipped to their shape with `.resizable().scaledToFill()` over a soft colour fallback. No photos of real, identifiable people unless the user supplies them.

~~~swift
struct EaseLogoBadge: View {                 // every real brand: white circle, hairline, soft shadow, the official logo in its own colour
    var logo: Image                          // the official vector from the asset catalog: Image("netflix")
    var tint: Color? = nil                   // brand colour for a one-colour glyph (template image); nil shows a full-colour logo as it is
    var scale: CGFloat = 0.54                // logo side as a share of the badge; 0.64 for a logo that is itself a disc (Spotify)
    var corner: CGFloat? = nil; var shadow = true    // corner 14: the white square of a row · shadow false on a grey card and in a row
    var body: some View {                    // fills the frame it is given
        GeometryReader { geo in
            let side = min(geo.size.width, geo.size.height)
            let shape = RoundedRectangle(cornerRadius: corner ?? side / 2, style: corner == nil ? .circular : .continuous)
            ZStack {
                shape.fill(Color.white).overlay(shape.strokeBorder(EaseInk.hairline, lineWidth: 1))
                logo.renderingMode(tint == nil ? nil : .template).resizable().scaledToFit()
                    .foregroundStyle(tint ?? EaseInk.black).frame(width: side * scale, height: side * scale)
            }
            .frame(width: geo.size.width, height: geo.size.height)
        }
        .shadow(color: .black.opacity(shadow ? 0.07 : 0), radius: 7, y: 3)
    }
}
struct EaseIconTile: View {                  // ONLY for invented things without a logo: an example brand, a category, an item
    var symbol: String; var color: Color; var size: CGFloat = 44   // a filled SF Symbol such as "music.note"; one EaseBrand colour
    var body: some View {
        RoundedRectangle(cornerRadius: size * 0.28, style: .continuous).fill(color.gradient).frame(width: size, height: size)
            .overlay { Image(systemName: symbol).font(.system(size: size * 0.46, weight: .semibold)).foregroundStyle(.white) }
    }
}
struct EaseAvatar: View {                    // a person without a photo: initials on a colour that follows the name
    var name: String; var size: CGFloat = 44
    var body: some View {
        Text(String(name.split(separator: " ").prefix(2).compactMap(\.first)).uppercased())
            .font(.system(size: size * 0.4, weight: .bold)).foregroundStyle(.white)
            .frame(width: size, height: size).background(EaseBrand.color(for: name).gradient, in: Circle())
    }
}
enum EaseBrand {                             // for tiles, avatars and symbols in a white badge; a real logo brings its own colour
    static let red = Color(red: 0.90, green: 0.22, blue: 0.23), orange = Color(red: 0.98, green: 0.45, blue: 0.09), amber = Color(red: 0.96, green: 0.62, blue: 0.04)
    static let green = Color(red: 0.13, green: 0.77, blue: 0.37), teal = Color(red: 0.08, green: 0.72, blue: 0.65), blue = Color(red: 0.15, green: 0.39, blue: 0.92)
    static let indigo = Color(red: 0.31, green: 0.27, blue: 0.90), violet = Color(red: 0.55, green: 0.36, blue: 0.96), pink = Color(red: 0.93, green: 0.28, blue: 0.60)
    static let warnFill = Color(red: 1.0, green: 0.906, blue: 0.824), warnText = Color(red: 0.851, green: 0.282, blue: 0.059)   // the one soft orange chip
    static let all = [red, orange, amber, green, teal, blue, indigo, violet, pink]
    static func color(for name: String) -> Color { all[name.unicodeScalars.reduce(0) { ($0 * 31 + Int($1.value)) % 9973 } % all.count] }
}
~~~

## 5. The four screens

Each pattern is one hero with the few blocks around it. Take the pattern that fits the job, put your own content in, keep the sizes. Overview, focus and result run in `references/examples/subscriptions.md`; the live screen is described at the end of this section. The snippets enter with `.easeReveal` and `EaseMotion.smooth`. The example adds three springs (badge drop 0.75 s, bounce 0.26 · next card 0.7 s, bounce 0.14 · check circle 0.7 s, bounce 0.28); take them from `EaseMotion`.

**Overview: a timeline with pinned logos** (`subs-week.png`). Question: what is coming, and when? Blocks: title · number with chip · strip · "Next up" row · button. Seven equal columns; each item's badge stands on a thin line above its day, the next one larger; today's number sits in a black circle. Hours of a day, stops of a route or weeks of a month work the same way. Use: `EaseWeekStrip(letters: l, numbers: n, today: 2, pins: [3: true, 4: false, 6: false], shown: shown) { day in EaseLogoBadge(logo: Image("netflix"), tint: red) }`.

~~~swift
struct EaseWeekStrip<Pin: View>: View {      // seven days; each item's badge is pinned above its day
    var letters: [String]; var numbers: [Int]            // seven each, from the calendar: "M" … and 30, 1, 2 …
    var today: Int; var pins: [Int: Bool]; var shown: Bool   // today's column · column → true for the next (larger) one · the enter flag
    @ViewBuilder var pin: (Int) -> Pin                   // the EaseLogoBadge for that column
    var body: some View {
        HStack(alignment: .bottom, spacing: 0) {
            ForEach(0..<7, id: \.self) { i in
                VStack(spacing: 0) {
                    Color.clear.frame(height: 100).overlay(alignment: .bottom) {   // an overlay: a wide badge cannot push the columns apart
                        if let next = pins[i] {
                            let order = Double(pins.keys.sorted().firstIndex(of: i) ?? 0)   // the badges arrive one after another
                            VStack(spacing: 0) {
                                pin(i).frame(width: next ? 68 : 52, height: next ? 68 : 52).easeReveal(shown, delay: 0.38 + order * 0.13)
                                Rectangle().fill(EaseInk.hairline).frame(width: 1.5, height: next ? 24 : 32)
                                    .scaleEffect(y: shown ? 1 : 0, anchor: .bottom).animation(EaseMotion.smooth(0.4).delay(0.6 + order * 0.13), value: shown)
                            }
                            .fixedSize()
                        }
                    }
                    VStack(spacing: 8) {
                        Text(letters[i]).font(.system(size: 16, weight: .medium)).foregroundStyle(EaseInk.secondary)
                        Text("\(numbers[i])").font(.system(size: 21, weight: .semibold)).foregroundStyle(i == today ? Color.white : EaseInk.primary)
                            .frame(width: 40, height: 40).background(i == today ? EaseInk.black : Color.clear, in: Circle())
                    }
                    .padding(.top, 8).easeReveal(shown, delay: 0.12 + Double(i) * 0.025)
                }
                .frame(maxWidth: .infinity).zIndex(pins[i] == true ? 2 : (pins[i] == nil ? 0 : 1))
            }
        }
    }
}
~~~

**Focus: one card, the next ones behind it** (`subs-review.png`). Question: what do I do with this one? Blocks: counter with progress segments · card stack · two pills. The card is centred: badge, name, the number, one chip (white; the soft orange only when it is urgent), a hairline, two detail lines. A decision sets the card's side inside `withAnimation(EaseMotion.smooth(0.65))` and raises `index` in the same tap: the card leaves to the left (no) or right (yes), the next one grows into place and its content fades in.

~~~swift
struct EaseFocusCard<Badge: View>: View {    // the content of one card; EaseCardStack draws the grey card around it
    var name: String; var value: String; var chip: String; var urgent = false   // "Spotify" · "$11.99" · "Renews Friday"
    var details: [(label: String, value: String)]        // two quiet lines, not more
    @ViewBuilder var badge: Badge                        // EaseLogoBadge(…, shadow: false)
    var body: some View {
        VStack(spacing: 0) {
            badge.frame(width: 96, height: 96)
            Text(name).font(.system(size: 21, weight: .semibold)).padding(.top, 14)
            Text(value).font(.system(size: 54, weight: .bold)).tracking(-1.5)
            Text(chip).font(.system(size: 15, weight: .medium)).foregroundStyle(urgent ? EaseBrand.warnText : EaseInk.secondary)
                .padding(.horizontal, 14).padding(.vertical, 7).background(urgent ? EaseBrand.warnFill : Color.white, in: Capsule()).padding(.top, 6)
            Rectangle().fill(EaseInk.hairline).frame(height: 1).padding(.top, 20)
            ForEach(Array(details.enumerated()), id: \.offset) { i, line in
                HStack { Text(line.label).foregroundStyle(EaseInk.secondary); Spacer(); Text(line.value).fontWeight(.semibold) }
                    .font(.system(size: 16)).padding(.top, i == 0 ? 16 : 10)
            }
        }
        .foregroundStyle(EaseInk.primary).padding(.horizontal, 22).padding(.top, 26).padding(.bottom, 22)
    }
}
struct EaseCardStack<Item: Identifiable, Card: View>: View {   // one card in front, the next two peek out below it
    var items: [Item]; var index: Int                    // index: the card in front
    var side: (Item) -> CGFloat?                         // nil: not decided · -1: it left to the left (no) · 1: to the right (yes)
    @ViewBuilder var card: (Item) -> Card                // an EaseFocusCard
    @Environment(\.accessibilityReduceMotion) private var reduceMotion
    var body: some View {
        let shape = RoundedRectangle(cornerRadius: 30, style: .continuous)
        ZStack(alignment: .top) {
            ForEach(Array(items.enumerated()), id: \.element.id) { i, item in
                let depth = CGFloat(min(max(i - index, 0), 2))
                let out = side(item)
                let fly = out != nil && !reduceMotion     // with Reduce Motion the card only fades
                let visible = i == index || out != nil    // cards behind stay blank until it is their turn
                card(item).opacity(visible ? 1 : 0).blur(radius: visible ? 0 : 10).animation(EaseMotion.smooth(0.55).delay(0.1), value: visible)
                    .frame(maxWidth: .infinity).background(shape.fill(EaseInk.fill).overlay(shape.strokeBorder(Color.white, lineWidth: 2)))
                    .scaleEffect(1 - depth * 0.055, anchor: .bottom).offset(y: depth * 13)
                    .rotationEffect(.degrees(fly ? Double(out ?? 0) * 10 : 0), anchor: .bottom)
                    .offset(x: fly ? (out ?? 0) * 470 : 0, y: fly ? 34 : 0).blur(radius: fly ? 12 : 0).opacity(out == nil ? 1 : 0)
                    .zIndex(Double(items.count - i))
            }
        }
    }
}
~~~

**Result: a check and one sentence** (`subs-saved.png`). Question: what did that change? Blocks: the result block, centred between two `Spacer`s · one button. One chip per item says what happened to it; the chip of an item that is gone is muted.

~~~swift
struct EaseResultHero<Chips: View>: View {   // a check, one sentence around the number, one small chip per item
    var lead: String; var value: Double; var trail: String; var shown: Bool   // "You save" · 15.49 · "every month"; set value 0.4 s after shown
    @ViewBuilder var chips: Chips
    var body: some View {
        VStack(spacing: 0) {
            ZStack {
                Circle().fill(EaseInk.black)
                EaseCheckShape().trim(from: 0, to: shown ? 1 : 0).stroke(Color.white, style: StrokeStyle(lineWidth: 5, lineCap: .round, lineJoin: .round))
                    .frame(width: 34, height: 26).animation(EaseMotion.smooth(0.5).delay(0.35), value: shown)
            }
            .frame(width: 84, height: 84).easeReveal(shown, delay: 0.05)
            Text(lead).padding(.top, 24).easeReveal(shown, delay: 0.18)
            Text(String(format: "$%.2f", value)).font(.system(size: 62, weight: .bold)).tracking(-2).monospacedDigit().foregroundStyle(EaseInk.primary)
                .contentTransition(.numericText(value: value)).animation(EaseMotion.smooth(0.9), value: value).fixedSize().easeReveal(shown, delay: 0.24)
            Text(trail).easeReveal(shown, delay: 0.3)
            HStack(spacing: 8) { chips }.padding(.top, 28).easeReveal(shown, delay: 0.5)
        }
        .font(.system(size: 19, weight: .medium)).foregroundStyle(EaseInk.secondary).frame(maxWidth: .infinity)
    }
}
struct EaseResultChip: View {                // the bare logo at 18 pt and one word; muted: the item is gone (cancelled, removed)
    var logo: Image; var tint: Color? = nil; var text: String; var muted = false
    var body: some View {
        HStack(spacing: 7) {
            logo.renderingMode(tint == nil ? nil : .template).resizable().scaledToFit().foregroundStyle(tint ?? EaseInk.black)
                .frame(width: 18, height: 18).saturation(muted ? 0 : 1).opacity(muted ? 0.55 : 1)
            Text(text).font(.system(size: 14, weight: .medium)).foregroundStyle(muted ? EaseInk.secondary : EaseInk.primary)
        }
        .padding(.horizontal, 12).padding(.vertical, 8).background(EaseInk.chip, in: Capsule()).fixedSize()
    }
}
struct EaseCheckShape: Shape {               // stroked with trim, so the check writes itself
    func path(in r: CGRect) -> Path {
        Path { p in p.move(to: CGPoint(x: r.minX, y: r.midY + r.height * 0.08)); p.addLine(to: CGPoint(x: r.minX + r.width * 0.36, y: r.maxY)); p.addLine(to: CGPoint(x: r.maxX, y: r.minY)) }
    }
}
~~~

**Live: something running right now** (a workout, a recording, an upload, a ride, a timer). Question: how is it going? Blocks: title line (the name of what runs, 18 medium secondary, with a small "Live" chip on the right: capsule `EaseInk.chip`, a 7 pt red dot that pulses in opacity, 14 medium) · the number (elapsed time or the value that runs, 62 bold, `numericText`, a grey chip beside it with the plan: "of 45 min") · the **tall hero** · one quiet progress row · the button. This screen has the fewest blocks, so the hero must be tall: **240 to 300 pt**, never a thin 60 pt band with the sheet empty under it.
- The hero is the live signal drawn large: a bar graph of the last minutes (bars 4 pt wide, gap 4, round caps, the newest bar in the accent, older bars `EaseInk.primary` fading to `EaseInk.hairline` toward the leading edge, a dashed target band behind them when the data has a target), a waveform, a route, a filling shape. Its current value sits inside the hero at its top left (21 semibold with a small symbol in the accent), not as a second row below it.
- The quiet row is progress toward the goal: grey caption left ("Goal"), value right ("18 of 45 min", 19 medium), 12 lower a capsule bar 8 high, `EaseInk.hairline` track, filled with the strip's main colour. No "Next up" here; the next item belongs to the result or the overview.
- Beats: the number rolls every second; the newest bar grows from the baseline with `EaseMotion.smooth(0.35)` while the oldest one leaves at the leading edge; the bar fill moves with the same value. The button is black ("Finish workout"), green with a check when the run is done, then the screen blurs out into the result.
- Sum for a 640 sheet: title 22, number 76, hero 270, row 70, button 54 = 492; 78 stay white above the button.

## 6. Colour, type and surfaces

~~~swift
enum EaseStrip {                             // one gradient per screen, from the left end to the right end
    static let sun = [Color(red: 1.0, green: 0.847, blue: 0.302), Color(red: 1.0, green: 0.541, blue: 0.239)]           // the subscriptions example
    static let sky = [Color(red: 0.557, green: 0.773, blue: 1.0), Color(red: 0.231, green: 0.510, blue: 0.965)], coral = [Color(red: 1.0, green: 0.788, blue: 0.659), Color(red: 1.0, green: 0.42, blue: 0.341)]
    static let mint = [Color(red: 0.655, green: 0.953, blue: 0.816), Color(red: 0.204, green: 0.827, blue: 0.6)], graphite = [Color(red: 0.122, green: 0.161, blue: 0.216), Color(red: 0.294, green: 0.333, blue: 0.388)]
    static let day = [Color(red: 0.604, green: 0.875, blue: 0.996), Color(red: 0.204, green: 0.788, blue: 0.992)], lime = [Color(red: 0.804, green: 0.969, blue: 0.549), Color(red: 0.514, green: 0.792, blue: 0.114)]
}
~~~

- **Strip: its colour comes from the content, not from the topic.** Take the colour of what the screen is about: the brand colour of the logo in focus (Netflix red, Spotify green), the colour your data gives the item in front (today's workout type, the category of the expense, the person's colour), or the state of the value (warm when it is late or high, cool when it is calm). Build it with `EaseStrip.from(color)` and draw it over white: `ZStack { Color.white; LinearGradient(colors: EaseStrip.from(c), startPoint: .leading, endPoint: .trailing) }`. When the item in focus changes, the strip changes with it inside the same `EaseMotion.smooth(0.5)` animation as the screen change (SwiftUI animates gradient colours). So two apps on the same topic do not look the same, and the screens of one flow differ.
- The presets (`sun`, `sky`, `coral`, `mint`, `graphite`, `day`, `lime`) are only for content that has no colour of its own. `sun` belongs to the subscriptions example: do not take it, and do not take `lime` just because the topic is health or sport. The sheet stays white.

~~~swift
extension EaseStrip {                        // a light end and the full colour, drawn over white
    static func from(_ c: Color) -> [Color] { [c.opacity(0.42), c.opacity(0.92)] }
}
~~~
 Flat: no glass, no materials. The only shadow is the one under a white badge (black 7 %, radius 7, y 3). Light mode only. Every rounded rectangle is `.continuous`.
- **Colour is rare.** Text is `EaseInk.primary` or `EaseInk.secondary`. Black belongs to the main button, to today's circle and to the check circle; pale grey (`EaseInk.off`) is the switched-off button, green (`successFill`) the button that worked. All other colour comes from the logos, the strip and one soft orange chip for what is urgent. No coloured panels, bars or tiles behind logos.
- **Type ramp**, system font only: the number 62 bold, −2 (54 bold, −1.5 in a card) · names 19–21 semibold · day numbers 21 semibold · the value in a row 19 medium · button 18 semibold · title and the lines around a centred number 18–19 medium secondary · caption, day letters, detail lines 16 · chips and sub lines 14–15 medium. Numbers that change use `.monospacedDigit()` and `.contentTransition(.numericText())`.
- **Surfaces**: focus card `EaseInk.fill`, radius 30, a 2 pt white outline; row outline 1 pt, radius 22; logo square radius 14; chips are capsules in `EaseInk.chip`; hairlines 1 pt. Never restyle a label as a button or a button as a label.
- **Dynamic Type**: in an app scale the small texts with `@ScaledMetric(relativeTo: .body)`; the number, the button height and the hero keep their size; one-line texts keep `.lineLimit(1)` with `.minimumScaleFactor(0.8)`.
