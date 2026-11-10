# W3 Patient mobile builder report

## STATUS
done (all five screens built and driven in the iOS Simulator; two verification gaps listed under RISKS).

## SUMMARY
Expo SDK 58 / Expo Router app in `apps/mobile`, "Clinical paper" direction. Flow: language pick (creates the session) -> intake sheet (ruled lines, ledger of earlier answers, one follow-up at a time, "Question n of up to 3") -> done status, or red-flag full-bleed carmine interrupt, or offline/error notice that always tells the patient to go to the desk. Fonts per language via `mobileFont()` (Newsreader / Noto Serif Thai / IBM Plex Sans / Plex Sans Thai Looped). Tokens only, no raw hex. Template demo screens, tabs, glow/badge assets and the reset script removed.

## FILES
Created: `apps/mobile/src/i18n.ts` (th/it/en string table), `src/api.ts` (fetch, 20 s timeout, `EXPO_PUBLIC_API_URL` default `http://localhost:3000`), `src/store.tsx` (context + `screenFor(session)` routing), `src/components/ui.tsx` (T, Button, Rule, Page), `src/app/{_layout,index,intake,done,redflag}.tsx`.
Modified: `apps/mobile/app.json` (name, slug `medx-bridge`, light only, web output `single`, paper splash), `package.json`/`package-lock.json` (expo-haptics + 4 `@expo-google-fonts/*` via `npx expo install`; removed `reset-project` script).
Deleted: template `components/`, `hooks/`, `constants/`, `global.css`, `scripts/`, `explore.tsx`, unused template images.
Not committed. Left `ios/`/`android/` absent (CNG).

## TESTS
- `npx tsc --noEmit`: pass.
- `npx expo lint`: 0 errors, 0 warnings.
- `npx expo export --platform web`: builds (static SPA export, index.html + per-route bundles).
- Backend curl: Italian headache -> `collecting` with `nextQuestion`; Thai chest-pain text -> `escalated` with Thai `patientMessage`.
- Simulator (iPhone 17, iOS 27, Expo Go SDK 58): manual drive, see EVIDENCE. No automated UI tests written.

## EVIDENCE (docs/reports/screens/)
- `mobile-2-intake-it-typed.png` Italian intake, "Ho mal di testa da ieri sera" on ruled lines.
- `mobile-3-followup-it.png` first answer in the ledger, "Domanda 1 di massimo 3", follow-up as next form line.
- `mobile-4-done-it.png` done: "Un infermiere leggerà le sue risposte." + code B-014 + next steps (reached after 3 follow-ups).
- `mobile-5-redflag-th.png` Thai session, red flag: full-bleed carmine, Thai message (first sentence display serif, rest lead), code B-015 at 64px, button "ฉันอยู่ที่เคาน์เตอร์แล้ว".
- `mobile-6-offline-error.png` API unreachable (`EXPO_PUBLIC_API_URL=http://localhost:3999`): inline notice + Try again; start screen stays usable.

## ASSUMPTIONS
- "Question n of up to 3" counts follow-ups only (the opening "What brings you in" is not numbered). After 3 follow-ups the backend returns a non-collecting status and the app routes to done.
- Language is chosen once, per session (server fixes `patientLang`); no in-session language switch (DIRECTION 6e control not built).
- `autoCorrect`/`spellCheck` are off on the answer field: iOS autocorrect mangled Italian in testing ("mal di" -> "mail do"), which would corrupt clinical text.
- Red-flag screen shows the first emergency hit's `patientMessage`, else the first hit.
- The English headline on the language screen is deliberate (rows are in each language); the privacy/disclaimer note is English only on that screen.

## RISKS
- Thai typed input was NOT verified end to end in the simulator (the tool pasted Thai to the clipboard but the field had no Paste menu). The Thai red-flag run used the English phrase "severe chest pain and sweating" in a Thai session; the Thai-keyword path is covered only by the backend curl. Thai rendering of UI strings and server messages is verified visually.
- Mid-intake failure (turn POST fails, session code known) uses the same notice with the code ("Please go to the desk and show code B-0xx") but was not screenshot; only the session-create failure was captured.
- Mock backend returns the same severity question for follow-ups 1 to 3 (seen in screenshots); that is backend, not the app.
- "I'm at the desk" is local only (no endpoint to tell the clinician): it swaps the text to "keep this screen open and show it to staff" and then offers "Start a new demo intake". DIRECTION 6c says no resume path; this starts a new session, not a resume, but the reviewer may want it hidden for the real demo.
- Not done: VoiceOver walk-through (labels, `alert` role, `announceForAccessibility` are in code, not exercised with VoiceOver), dynamic type at max size, reduced-motion toggle test (`useReducedMotion` wired). Android and physical device untested. Haptic fires in code (`notificationAsync(Error)` once); the simulator cannot show it.
- Native-speaker review of th/it strings in `i18n.ts` still pending (DIRECTION says to confirm before demo).
- "I need help finding the desk" quiet button from DIRECTION omitted: no desk location is defined in the data.
- Expo Go dev FAB (gear) is visible in screenshots; not part of the app. A hardware keyboard typing into an unfocused screen opens the Expo dev menu (dev only). Input is autofocused and refocused after each question.
- Unused template deps left in package.json (`@expo/ui`, `expo-glass-effect`, `expo-symbols`, `expo-image`, `expo-device`, `expo-web-browser`); harmless, remove if bundle size matters.

## OPEN QUESTIONS / CONTRACT CHANGE REQUESTS
- CR (optional): an endpoint or turn flag for "patient is at the desk" so the clinician queue can show arrival on escalated sessions.
- Decide whether a patient on `needs_human` should see an extra line from the backend (`abstainReason`); currently generic copy.
- Desk location text for the pilot clinic, if the "help finding the desk" affordance is wanted.

## NEXT
1. Native-speaker pass on `src/i18n.ts`.
2. VoiceOver + Dynamic Type + reduced-motion check on a device, and a Thai-keyboard run.
3. Point `EXPO_PUBLIC_API_URL` at the deployed API for the judged demo (a LAN IP for a physical phone; `localhost` works only in the simulator and the web build on the same host). Run: `cd apps/mobile && npx expo start`.
