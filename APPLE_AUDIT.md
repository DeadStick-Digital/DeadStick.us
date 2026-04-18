# deadstick.net — Apple Developer / App Store audit

_Audit date: 2026-04-18. Based on App Store Review Guidelines, App Store Connect
requirements, and Apple's third-party trademark guidelines as of this date._

This audits **the website** against Apple's requirements for apps distributed
through the App Store and Mac App Store. It doesn't cover things that live
inside the apps themselves (Xcode settings, Info.plist entries, in-app privacy
disclosures, etc.) — those are out of scope, but I've flagged a few items at
the bottom that are worth checking on the app side.

## Legend

- ✅ **Pass** — meets Apple's requirement.
- ⚠️  **Fix recommended** — works today but has a concrete gap or risk.
- 🚧 **Action required outside the website** — Chance needs to do this in
  App Store Connect, in the app code, or in the Developer Program portal.

---

## 1. Required URLs (App Store Connect)

Apple requires every app submission to declare a **Privacy Policy URL** and a
**Support URL**. Marketing URL is optional.

| URL type          | Requirement | deadstick.net has it                              | Status |
|-------------------|-------------|--------------------------------------------------|--------|
| Privacy Policy    | Required    | `privacy.html`                                   | ✅ |
| Support URL       | Required    | `support.html`                                   | ✅ |
| Marketing URL     | Optional    | `index.html` (and per-app pages under `apps/`)   | ✅ |
| Privacy Choices   | Optional    | not provided                                     | n/a |

Per Apple, the Support URL must contain *up-to-date contact information* and
a link to the privacy policy. `support.html` now directs people to the new
contact form and links the Privacy Policy in context — ✅.

## 2. Privacy policy content (Guideline 5.1.1)

Apple requires the policy to cover: (a) what data is collected and how,
(b) all uses of that data, (c) third-party data-sharing obligations,
(d) retention/deletion, and (e) how users revoke consent or request deletion.

| Requirement                                      | Covered in `privacy.html`          | Status |
|--------------------------------------------------|------------------------------------|--------|
| Data collected & how                             | §3                                 | ✅ |
| All uses of the data                             | §4                                 | ✅ |
| Third parties & equal protection commitment      | §5 (Apple, Google, email, host, Web3Forms) | ✅ (after today's edit) |
| Retention & deletion policy                      | §6                                 | ✅ |
| User rights / how to revoke or delete            | §7 (CCPA/GDPR — now routed through contact form) | ✅ |

**Note on Web3Forms:** the contact form's submissions pass through Web3Forms
before reaching our mailbox. I added a line to §5 declaring them as a processor
and naming the data fields they touch (name, email, topic, message, IP, bot
score). Apple expects the policy to match reality, so leaving them out would be
a defect.

## 3. Contact-form backend (today's change)

The new `#contact` form posts to Web3Forms (`api.web3forms.com/submit`) with
three spam defenses:

1. **hCaptcha** via Web3Forms' native integration (`<div class="h-captcha" data-captcha="true">` + their client script).
2. **Honeypot checkbox** named `botcheck`, hidden off-screen and `aria-hidden`.
3. **Required-field validation** (email + topic + message).

**Before going live:** replace `REPLACE_WITH_WEB3FORMS_ACCESS_KEY` in
`index.html` (line ~205) with an access key generated at
<https://web3forms.com/> using contact@deadstick.net as the destination address.
Nothing else needs to change.

## 4. Apple trademark attribution (⚠️  fix recommended)

Apple's third-party guidelines require an attribution notice in the "credit
section" of any page that uses an Apple trademark. The modern form is:

> "Apple, the Apple logo, iPhone, iPad, Mac, macOS, and Apple Silicon are
> trademarks of Apple Inc., registered in the U.S. and other countries
> **and regions**. App Store is a service mark of Apple Inc."

### Gaps

- The footer on `index.html` already has this notice — ✅
  - Minor: add "**and regions**" and include "**Apple Silicon**" (used on
    `clawmelt.html`).
- `support.html`, `privacy.html`, `terms.html`, and the three pages under
  `apps/` all mention Apple marks (iPhone, iPad, Mac, macOS, App Store) but
  have a stripped-down footer with **no** attribution. The cleanest fix is to
  add the attribution line to the minimal footer on those pages, or move the
  notice to a shared partial.

## 5. VPN app review (CarrierPigeon) — Guideline 5.4 (⚠️  fix recommended + risk note)

CarrierPigeon is a VPN app for App Review purposes (NEPacketTunnelProvider on
iOS, VpnService on Android). Guideline 5.4 says:

> "VPN apps may not sell, use, or disclose to third parties any data for any
> purpose, and must commit to this in their privacy policy... Apps offering VPN
> services must be offered by developers enrolled as an organization."

### What's in good shape

- **Organization enrollment.** DeadStick Digital LLC is the developer entity
  — ✅.
- **Technical implementation.** The `apps/carrierpigeon.html` page describes
  using `NEPacketTunnelProvider` on iOS and `VpnService` on Android, which is
  what Apple expects.
- **No data collection.** The privacy policy (§10.3) says CarrierPigeon never
  sees traffic contents, destinations, or metadata.

### Gaps

- ⚠️  **The explicit 5.4 commitment is missing.** Apple wants the literal
  commitment: "we do not sell, use, or disclose to third parties any data
  collected by this app." Privacy §10.3 should add a sentence like:

  > "As required by Apple's VPN-app policy, DeadStick Digital does not sell,
  > use, or disclose to third parties any data collected by CarrierPigeon
  > for any purpose."

- ⚠️  **"Carrier tether detection" framing is a review risk.** The
  `carrierpigeon.html` hero says the traffic is *"indistinguishable from
  native on-device traffic… no hotspot fingerprint,"* and the "How it works"
  section mentions setting TTL to 65 *"so the phone's +1-hop decrement lands
  at the native-looking 64."* Apple reviewers have historically rejected apps
  whose framing suggests circumventing carrier tethering detection — it
  brushes against Guideline 5.5 (Developer Code of Conduct / third-party ToS).
  The disclaimer "You remain fully responsible for how you use the connection
  and for complying with your carrier's terms of service" is good and should
  stay, but consider softening the "indistinguishable from native" language
  in the public marketing copy. One option:

  > "CarrierPigeon keeps the connection efficient — no double-NAT, no extra
  > battery drain — and is fully within your control, which is why it's your
  > responsibility to use it in line with your carrier's terms."

  This is a risk judgment, not a hard rule. Worth knowing going into review.

## 6. Account-deletion rule (Guideline 5.1.1(v))

Apple requires apps that support account creation to offer in-app account
deletion. All three app pages declare **"Account: None required"**, so this
rule doesn't apply — ✅. If any app ever gains an account system, revisit.

## 7. Subscription disclosures (Guideline 3.1.2)

BillingBird's side-card lists *"One-time + optional Pro tier."* If that Pro
tier is an auto-renewing subscription, Apple requires clear disclosure of:

- Subscription length
- Price per billing period
- Renewal terms
- How to cancel

…both in the App Store listing **and** before purchase in the app. The
website itself isn't obligated to show pricing, but the phrase "optional Pro
tier" is slightly underspecified — once the tier ships, it's worth either
saying "optional Pro subscription" (accurate) or "optional Pro one-time
upgrade" (accurate) so the marketing matches what the App Store will show.

## 8. EULA

`support.html` and `terms.html` already point App Store installs at Apple's
standard Licensed Application EULA
(<https://www.apple.com/legal/internet-services/itunes/dev/stdeula/>) and your
own Terms for direct-download builds — ✅.

## 9. 🚧 Apple-side items (not a website fix)

These aren't on the website but are worth queuing up before Chance submits
anything to App Review:

1. **D-U-N-S Number** — required to enroll DeadStick Digital LLC as an
   organization in the Apple Developer Program. Free; look up or request at
   <https://developer.apple.com/enroll/duns-lookup/>.
2. **Developer contact address in App Store Connect.** The public-facing
   email has been removed from the site today, but contact@deadstick.net
   still needs to exist and be provided to Apple in App Store Connect for
   App Review correspondence (under *App Review Information* and the
   *Account* page). Removing it from the website doesn't remove it from
   Apple's communications.
3. **Privacy Manifest (`PrivacyInfo.xcprivacy`)** — mandatory in every iOS /
   iPadOS / visionOS / watchOS / tvOS app binary as of May 2024. Make sure
   BillingBird's iOS build and CarrierPigeon's iOS + macOS builds ship one
   that matches the privacy policy.
4. **iOS 26 SDK deadline.** Starting **April 28, 2026** (ten days from
   today), new App Store submissions must be built with the iOS 26 / iPadOS
   26 / tvOS 26 / visionOS 26 / watchOS 26 SDKs. Plan accordingly.
5. **In-app privacy link.** Guideline 5.1.1 also requires a link to the
   privacy policy inside the app, not just on the website. Each app's
   Settings → "About" or "Legal" screen should link to
   <https://www.deadstick.net/privacy.html>.
6. **App Privacy questionnaire.** When submitting each app, the answers in
   the App Privacy section of App Store Connect must match `privacy.html`.
   Mismatches are a common review-rejection reason.

## 10. Summary — what still to do

### Website changes (small)

- [ ] Paste the Web3Forms access key into `index.html` in place of
      `REPLACE_WITH_WEB3FORMS_ACCESS_KEY`.
- [ ] Add Apple trademark attribution to the footers of `support.html`,
      `privacy.html`, `terms.html`, and the three `apps/*.html` pages (or
      share a common footer). Update the main footer to read "…and other
      countries **and regions**" and include **Apple Silicon**.
- [ ] Add the literal 5.4 commitment to `privacy.html` §10.3 (CarrierPigeon).
- [ ] Consider softening the "indistinguishable from native traffic /
      no hotspot fingerprint / TTL 65" language on `apps/carrierpigeon.html`
      ahead of App Review — optional, but I'd do it.
- [ ] Clarify whether BillingBird's "optional Pro tier" is a subscription or
      a one-time upgrade, once decided.

### Developer-side setup (outside this repo)

- [ ] Get D-U-N-S Number and enroll DeadStick Digital LLC as an organization.
- [ ] Keep contact@deadstick.net active for App Store Connect contact fields.
- [ ] Build with iOS/iPadOS/macOS 26 SDKs by April 28, 2026.
- [ ] Ship a `PrivacyInfo.xcprivacy` manifest in every Apple-platform binary.
- [ ] Add in-app links to `privacy.html` and `support.html` in each app.
- [ ] Match App Privacy answers in App Store Connect to the privacy policy.
