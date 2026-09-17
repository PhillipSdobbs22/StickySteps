# StickySteps

**Procedure steps that stay on screen while you work in other apps.**

Free, open source, on-device.

---

## The problem

You ask an assistant how to change a setting on your iPhone. It gives you four
steps. You open Settings — and the steps are gone. You bounce back through the
app switcher to re-read step two, lose your scroll position, and start over.

iPhone has no split screen. There is no way to keep the instructions and the
destination on screen at the same time.

## The approach

StickySteps pins your steps to a **floating card** that stays on screen while
you work in any other app. All the steps are visible at once — no swiping, no
tapping, no losing your place. Controls to advance or dismiss live in a **Live
Activity** in Notification Center.

<!--https://github.com/user-attachments/assets/b58c59f4-e791-458f-933b-ec579358e6ef-->

## Status

**Proof of concept.** Both surfaces are built and verified on real hardware.
Capture, OCR, history and persistence are not built yet. See the roadmap below.

## What has been verified on real hardware

Tested on **iPhone 13 Pro (no Dynamic Island), iOS 26.6.2**:

| Question | Result |
|---|---|
| Can a third-party app float text over other apps? | **Yes** — via the Picture-in-Picture video-call API |
| Does it survive app switching? | Yes — held over Home Screen, Photos, YouTube, and a third-party app |
| Do buttons inside the floating window work? | **No** — iOS reserves those touches for its own controls |
| Is a Live Activity visible inside another app? | No, not on notched hardware |
| Do Live Activity buttons work from Notification Center? | Yes — without foregrounding the app |

Full log: [`docs/technical-findings.md`](docs/technical-findings.md)

## Architecture

Two surfaces, each doing what it is actually good at:

| Surface | Role | Framework |
|---|---|---|
| **Floating card** | Always-visible, read-only list of all steps | AVKit (Picture in Picture) |
| **Notification Center** | Next / Back / Done controls | ActivityKit + AppIntents |
| **Tap the card** | Returns to the app | — |

Both are public SDK. No private APIs, no special entitlements.

**Why the card has no buttons:** touches inside a Picture-in-Picture window are
reserved by iOS for its own controls and never reach app content. A button there
would misrepresent what the region does. The card is a pure reference display;
controls live where taps are proven to work.

**Why there is no current-step highlight:** with every step visible at once, the
reader finds their own place. All steps render identically in high-contrast
yellow on near-black, and type scales with step count so roughly eleven steps
fit before the remainder is summarized.

## What this deliberately does not attempt

Apple's own system HUDs — the dictation pill, the privacy indicators, the
screen-recording bar — are drawn by **SpringBoard**, the iOS system UI process,
using private frameworks and Apple-only entitlements. Third-party apps cannot do
that, and this project does not pretend otherwise.

The Picture-in-Picture route reaches a similar result through a supported door,
with the limits that door imposes. Those limits are documented above rather than
designed around.

## Known open question

Picture in Picture requires the **Background Modes** capability "Audio, AirPlay,
and Picture in Picture." Declaring a background mode an app does not genuinely
use is a documented App Store rejection cause. Teleprompter apps ship this exact
feature today, so precedent exists — but the mechanism they declare has not been
verified. **This is unresolved and blocks submission.**

## Requirements

- iOS 17.0 or later
- Xcode 15 or later
- A physical iPhone. The Dynamic Island presentation needs an iPhone 15 Pro or
  newer simulator to test.

## Build it

See [`SETUP.md`](SETUP.md) for step-by-step Xcode instructions, including the
widget extension target and the settings that most often cause a silent failure.

## Project layout

```
Shared/
  StepAttributes.swift   ActivityKit data contract (both targets)
  StepIntents.swift      Next / Back / Done as LiveActivityIntents (both targets)

StickySteps/
  StickyStepsApp.swift   App entry point
  ContentView.swift      Starts and ends the Live Activity
  StepModel.swift        Shared step state
  PiPManager.swift       Creates and controls the floating window
  PiPStepsView.swift     What the floating card displays
  PiPTestView.swift      Test harness for the PiP surface

StickyStepsWidget/
  StickyStepsWidgetBundle.swift    Extension entry point
  StickyStepsLiveActivity.swift    Lock Screen + Dynamic Island UI
```

## Roadmap

- [x] Live Activity with working Next / Back / Done
- [x] Floating card visible over other apps
- [ ] Merge both surfaces so Notification Center controls update the card
- [ ] Resolve the background-mode question
- [ ] Share sheet extension — pin steps straight from an assistant's reply
- [ ] Persistence so notes survive an app relaunch
- [ ] Screenshot capture with on-device text recognition (Vision framework)
- [ ] Automatic step splitting from pasted text
- [ ] iPad layout
- [ ] Expiring treatment for passcodes and other sensitive items

## Privacy

Everything stays on the device. No accounts, no servers, no analytics. Live
Activities here use `pushType: nil`, meaning they are never updated remotely.

## License

TBD — MIT or Apache 2.0.

---

Built by [Phillip-Shawn Dobbs](https://github.com/PhillipSdobbs22).
