# NFL Win Total Picker

A one-page website for picking every 2026 NFL regular-season game by win percentage. It shows projected win totals, playoff seeds, playoff odds and a record of how the picks have done. Final scores update automatically.

It's a static site: plain HTML, CSS and JavaScript in one file. There's no build step, no framework, no packages, no server code and no API keys.

## Files

| File | What it is |
|---|---|
| `index.html` | The whole app. Includes the 2026 schedule, team data, playoff tiebreaker logic, and the owner's current picks as the starting set. |
| `picks.json` | Backup copy of the owner's 179 picks (made Oct 20, 2026). These are already built into `index.html`; this file is for safekeeping and can be loaded with **Restore picks**. |
| `README.md` | This file. |

## Deploy (GitHub + Vercel)

1. Create a new GitHub repo (suggested name: `nfl-win-picker`) and commit all three files at the repo root.
2. In Vercel, import the repo with these settings:
   - Framework preset: **Other**
   - Build command: *(leave empty)*
   - Output directory: **`.`** (the repo root)
   - Install command: *(leave empty)*
3. Deploy, then open the site. Go to the **Totals** tab and scroll to the bottom. The **Scores** row should say "Scores through Week N, checked today …". That means the live score feed is working.

Keep `index.html` as one file and don't run it through a bundler or framework. Everything it needs is inline except the Google Fonts stylesheet and the score feed below.

## How results refresh

- The page downloads the public nflverse schedule file in the browser:
  `https://raw.githubusercontent.com/nflverse/nfldata/master/data/games.csv`
  This file is public, allows cross-site requests (CORS), and is updated automatically many times a day. Final scores show up there after games end. It's about 2 MB.
- **When it checks:**
  - when the page opens, if the last check was more than 30 minutes ago
  - when the tab comes back into view after 30 minutes
  - whenever the **Refresh** button is tapped (Totals tab settings, and the My picks tab)
- **What a final score does:**
  - The finished game counts 100% for the winner in win totals, playoff seeds and odds.
  - The original pick stays saved, and the **My picks** tab grades it right or wrong.
  - New spreads, odds and kickoff-time changes come in at the same time.
- **If the feed can't be reached**, the page uses the last scores saved in that browser. If there are none, it uses the scores built into `index.html`, which run through Week 7 (as of Oct 28, 2026).
- There's no cron job, server or scheduled task to set up.

## Where picks are saved

- Picks save in the browser's `localStorage` under the key `nfl-win-picker-2026`. Each browser and device keeps its own copy.
- The first time the site opens in a new browser, it loads the owner's built-in picks.
- **To move picks to another device or back them up**, go to the Totals tab:
  - **Back up picks → Download** saves a `.json` file.
  - **Restore picks → Choose file** loads one. `picks.json` from this repo works here too.
- **Format:** `{"picks": {"<game_id>": <home team win %>}, "useResults": true}`
  - The percentage is 0–100 in steps of 10.
  - `game_id` is the nflverse id. For example, `2026_08_CAR_GB` is Carolina at Green Bay, and a value of `30` means Green Bay 30%, Carolina 70%.

### Changing the built-in picks (optional)

To make a new picks file the default for new browsers, replace the single line in `index.html` that starts with `const SEED = ` with:

```js
const SEED = {"picks":{ ...the "picks" object from the file... },"useResults":true};
```

Browsers that already have saved picks keep their own. For those, use **Restore picks**.

## What's in the app

- **Pick**: one game at a time. Tap a team's 60–100% button, or Toss-up for 50/50, and it moves to the next unpicked game. Games that are already final drop out of the list.
- **Totals**: projected wins by division or conference, plus the settings (count real results, scores, backup, restore, start over).
  - Each game adds its win percentage to the team's total, so a 70% pick adds 0.7 wins.
  - Unpicked games count as 50/50.
- **Playoffs**: seeds 1–7 per conference using the NFL's rules.
  - Division champions take seeds 1–4, and the three best other teams are wild cards.
  - Ties are broken with the official division and wild-card tiebreakers.
  - Also shows the Wild Card Round matchups and playoff, division and 1-seed odds from 10,000 simulated seasons.
  - Point-based tiebreakers are replaced by a coin toss, because simulated games have no scores.
- **My picks**: right–wrong record, expected right from the percentages, hit rate at each confidence level, and every pick by week. Upcoming picks link back to the picker.

Teams are shown as color badges with their abbreviations, so there are no image files to host.

## Next season

The 2026 schedule and team list are built into `index.html`. The score feed only updates games already in the file, so a 2027 version needs the file rebuilt with the new schedule.
