# The Runner - Notes For Next Codex Session

This file is for Codex, not for the user. The user may paste it into a new chat on another computer so the next session can quickly understand the current project state.

## Project Location

Main project directory:

`D:\GRY\the Runner`

The game is still mostly a static HTML prototype. No build step, no package manager, no server required.

Important files:

- `index.html` - main game.
- `stage-editor.html` - local editor for runs, stages, characters, HP, avatars and dialogs.
- `stage-data.js` - external stage/run/character/dialog data loaded by the game and editor.
- `asset-manifest.js` - list of image paths available in editor dropdowns.
- `runner_files/` - images and avatars.

## Current Data Flow

`index.html` loads:

```html
<script src="asset-manifest.js"></script>
<script src="stage-data.js"></script>
```

`stage-data.js` defines:

```js
window.RUNNER_STAGE_DATA = { runs: [...], characters: [...] };
```

The game converts this editor data into the legacy in-game `STAGES` shape with `buildStagesFromEditorData(window.RUNNER_STAGE_DATA)`.

The editor also loads the same `asset-manifest.js` and `stage-data.js`. However, `stage-editor.html` first checks its own `localStorage` save (`the-runner-stage-editor-v1`). If the editor shows stale data, use the editor button `Load File Data` to reload from `stage-data.js`.

## User Workflow For Data Changes

The user edits in `stage-editor.html`, then:

1. Click `Save`.
2. Click `Export JS Config`.
3. Click `Copy Export`.
4. Paste the full text into Codex.
5. Codex replaces `stage-data.js` with that export.

Important: `Save` only writes to browser `localStorage`; it does not write `stage-data.js`.

When replacing `stage-data.js`, preserve or restore `rewardSoft` and `rewardXp` if the export lacks them. The editor sometimes exports only data it knows about, but the game uses rewards.

## Stage Data Shape

Run:

```js
{
  id: 1,
  name: "Run 1",
  subtitle: "Day as usual",
  rewardSoft: 420,
  rewardXp: 120,
  introDialogs: [
    { speakerId: "boss", text: "..." },
    { speakerId: "player", text: "..." }
  ],
  stages: [
    {
      id: "1-1",
      label: "Stage 1-1",
      characterId: "nitro",
      hp: 6500,
      preFightDialogs: [
        { speakerId: "nitro", text: "..." }
      ]
    }
  ]
}
```

Character:

```js
{
  id: "nitro",
  name: "Nitro",
  suit: "diamonds",
  art: "runner_files/Karo_mechanic_01.png",
  avatar: "runner_files/Karo_mechanic_01_avatar.png",
  role: "garage",
  flavor: "..."
}
```

Special speaker ids:

- `player` maps to right-side/player SMS bubbles.
- anything else maps to left-side/contact SMS bubbles.
- `nitro` is displayed on home thread as `NITRO Car Repair`.

## Current Story/Data State

Run 1:

- subtitle: `Day as usual`
- intro SMS is the original Boss/Kurier job conversation.
- Stage 1-1 Nitro, HP 6500, pre-fight mechanic dialog.
- Stage 1-2 Bump, HP 7100.
- Stage 1-3 Roxy, HP 7700.
- Stage 1-4 Security Guy, HP 8400.
- Stage 1-5 Blake Castro, HP 9400.

Run 2:

- subtitle: `Pimp my Car`
- intro SMS from Nitro / player.
- Stage 2-1 Local Thug, HP 7600.
- Stage 2-2 Nitro, HP 8000.
- Stage 2-3 Corupted Policeman, HP 8600.

Run 3:

- subtitle: `Good Stuff`
- intro SMS from Blake Castro.
- currently no stages.
- The game was adjusted so runs without stages are still visible in the test run picker and can play intro SMS without crashing.

## Recent UI/Game Changes

Home/SMS:

- Home screen was changed into a dark-mode messenger; old background illustration removed.
- Home thread contact is dynamic based on current run `introDialogs`.
- Run 2 first contact is Nitro and displayed as `NITRO Car Repair`.
- Old run conversations appear below as greyed archived threads.
- Bottom test run selector is below the bottom nav. It is intentionally small.
- Selector does not switch instantly; choose run, then press `GO`.
- `GO` resets current run state, HP, battle/reward/upgrade state and home messages so selected run starts with its intro SMS.

Battle dialogs:

- Pre-fight speech bubbles appear before first cards are dealt.
- Cards are not dealt until all configured `preFightDialogs` are clicked through.
- Bubbles show only spoken text, no speaker name, no `Tap anywhere`.

Rewards:

- End-of-run card pack uses flip cards with backs.
- Cards reveal on click.
- Continue appears only after all reward cards are revealed.

Run upgrades:

- Run upgrades stack. Picking the same upgrade again adds its value again.
- Map upgrade chips show `x2`, `x3`, etc.
- Battle damage summary uses yellow stars instead of a yellow diamond/square.
- Number of stars reflects upgrade stack count.

AI:

- Enemy is smarter after player pass. If enemy is already winning/tied after player pass, it should not risk extra draws.

HP:

- Player base HP was reduced to `28000`.
- Player HP per level is `3000`.

## Asset Manifest

Editor dropdowns use `asset-manifest.js`:

```js
window.RUNNER_ASSET_MANIFEST = [
  "runner_files/Karo_bump_01.png",
  ...
];
```

If the user adds new images to `runner_files/`, add their paths to `asset-manifest.js` so they appear in editor dropdowns.

Current known avatar files include:

- `Karo_mechanic_01_avatar.png`
- `Karo_bump_01_avatar.png`
- `hearts_girl_02_avatar.png`
- `clubs_security_01_avatar.png`
- `spades_boss_02_avatar.png`
- `spades_entrepeneur_01_avatar.png`

## Important Implementation Details

- `HOME_CHAT_STEPS` still exists as fallback, but normal home messages come from `getHomeChatSteps()` using `RUNNER_STAGE_DATA.runs[].introDialogs`.
- `getRunContact(runId)` chooses the home thread contact from the first non-player `introDialogs` speaker.
- `renderArchivedRunThreads()` shows completed/lower-id run threads as grey archive entries.
- `createBattleIntroDialogs(stageId, nodeIndex)` first tries `getConfiguredBattleIntroDialogs()`, then falls back to old hardcoded sample dialogs.
- `buildStagesFromEditorData()` must not filter out runs with empty `stages`, because Run 3 currently has intro SMS only.
- `renderMap()` has a guard for empty-stage runs and shows a simple card instead of trying to divide by zero.

## Validation

After editing JS/HTML, validate by parsing inline scripts. Example with Node:

```js
const fs = require("fs");
new Function(fs.readFileSync("stage-data.js", "utf8"));
const html = fs.readFileSync("index.html", "utf8");
new Function(html.match(/<script>([\s\S]*)<\/script>/)[1]);
```

In this Codex environment, use available tools rather than `require` if needed. Previous validation passed for:

- `stage-data.js`
- `index.html`
- `stage-editor.html`

