# ⛳ [Best Ball Golf](https://mikeharman89.github.io/bestball-golf/)

A live best ball golf pool web app built for friend groups and expanding audiences. Originally built for the Masters Tournament, designed to scale to any PGA Tour event.

---

## What Is Best Ball Golf?

Best Ball Golf is a tournament pool format where each participant selects one golfer from each of four tiers. During each round, the **lowest score among your four golfers on each individual hole** counts toward your team total. Scores are tracked hole-by-hole and updated live throughout the tournament.

---

## Features

### 🏆 Live Leaderboard
- Real-time hole-by-hole scoring pulled directly from the tournament's official score feed
- Running cumulative best ball score displayed across all 18 holes
- Round tabs: Round 1, Round 2, Round 3, Round 4, and Overall standings
- Overall leaderboard sums each team's best ball score across all four rounds
- Tiebreaker logic: in the event of a tie, scores are compared backwards from hole 18
- Percentage of round completed shown for each team
- Click any team to expand and see all four golfer scores
- Click any golfer to see their full 18-hole breakdown for that round
- Unpaid entries are excluded from the leaderboard and listed separately below it
- Auto-refreshes every 5 minutes while a tournament is active

### 📝 Pick Submission
- Two separate pick windows: **Thursday & Friday** and **Saturday & Sunday**
- Participants select one golfer per tier from alphabetically sorted dropdowns
- Email collection on Thursday & Friday submissions for tournament communications
- Picks are editable until the admin locks them
- Anonymous authentication — no account creation required for participants

### ⚙️ Admin Panel
- Create and manage tournaments with name, dates, score feed URL, buy-in amount, and status
- Assign golfers to tiers before picks open
- Switch picks phase between weekday and weekend
- Lock picks to prevent further edits
- Archive completed tournaments to preserve results
- **Manage Picks**: view all submitted entries, edit names, emails, and golfer selections, mark paid/unpaid, delete entries, and add new entries manually via dropdown
- **Total Purse**: automatically calculated as paid entries × buy-in amount, displayed on the leaderboard

### 💰 Payment Tracking
- Paid checkbox per entry in the admin panel
- Paid/unpaid counter at the top of the picks manager
- Unpaid list displayed below the leaderboard for participants to see who still owes

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| Database | Firebase Firestore |
| Authentication | Firebase Auth (Anonymous + Email/Password) |
| Score Data | Official tournament JSON feeds (e.g. masters.com) |
| Hosting | GitHub Pages |
| Fonts | Google Fonts (Playfair Display, DM Mono) |

---

## Project Structure

```
bestball-golf/
  index.html          ← entire app (frontend + Firebase logic)
  data/
    2026-masters.json ← archived final scores snapshot (coming soon)
  README.md
```

---

## How It Works

### Score Calculation
1. For each hole, the app fetches each golfer's raw stroke count from the score feed
2. Each stroke count is converted to a score vs par (e.g. birdie = -1, bogey = +1)
3. The **lowest** score vs par among the team's four golfers is selected for that hole
4. Those best-per-hole values are accumulated into a running cumulative total
5. The final cumulative total after 18 holes is the team's round score

### Example
| Hole | Tier 1 | Tier 2 | Tier 3 | Tier 4 | Best Ball | Running Total |
|------|--------|--------|--------|--------|-----------|---------------|
| 1    | E      | +1     | -1     | E      | -1        | -1            |
| 2    | -1     | E      | E      | +1     | -1        | -2            |
| 3    | E      | E      | -2     | E      | -2        | -4            |

### Pick Structure
- **Thursday & Friday**: uses `r1golfers` for Round 1 and Round 2
- **Saturday & Sunday**: uses `r2golfers` for Round 3 and Round 4
- Weekend picks are matched to weekday entries by name

---

## Firebase Data Model

```
tournaments/
  {tournamentId}/
    name:        "2026 Masters Tournament"
    dates:       "Apr 10-13, 2026"
    scoreFeed:   "https://www.masters.com/en_US/scores/feeds/2026/scores.json"
    startDate:   "2026-04-10"
    status:      "active" | "archived"
    locked:      false
    picksPhase:  "weekday" | "weekend"
    buyin:       25
    tiers: [
      { golfers: ["Scottie Scheffler", "Rory McIlroy", ...] },  // Tier 1
      { golfers: ["Ludvig Åberg", "Bryson DeChambeau", ...] },   // Tier 2
      { golfers: ["Tommy Fleetwood", "Justin Thomas", ...] },    // Tier 3
      { golfers: ["Shane Lowry", "Russell Henley", ...] }        // Tier 4
    ]
    picks/
      {pickId}/
        name:       "Mike Harman"
        email:      "mike@example.com"
        userId:     "anon-uid"
        paid:       true
        r1golfers:  ["Scottie Scheffler", "Ludvig Åberg", "Tommy Fleetwood", "Shane Lowry"]
        r2golfers:  ["Rory McIlroy", "Bryson DeChambeau", "Justin Thomas", "Russell Henley"]
        updatedAt:  timestamp
```

---

## Setup & Deployment

### Prerequisites
- A [Firebase](https://console.firebase.google.com) project with Firestore and Authentication enabled
- A [GitHub](https://github.com) account

### Firebase Configuration
1. Create a Firebase project
2. Enable **Firestore Database** and **Authentication** (Anonymous + Email/Password)
3. Add your Firebase config to `index.html`
4. Set your admin email in `index.html`:
   ```javascript
   const ADMIN_EMAIL = 'your@email.com';
   ```
5. Set Firestore security rules:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /tournaments/{id} {
         allow read: if true;
         allow write: if request.auth != null && request.auth.token.email == "your@email.com";
         match /picks/{pickId} {
           allow read: if true;
           allow write: if request.auth != null;
         }
       }
     }
   }
   ```

### GitHub Pages Deployment
1. Upload `index.html` to your GitHub repository
2. Go to **Settings → Pages → Source → main branch**
3. Your app will be live at `https://yourusername.github.io/bestball-golf`

### Custom Domain (Optional)
1. Buy a domain from Namecheap, GoDaddy, or similar
2. Add DNS records pointing to GitHub Pages servers
3. Add the custom domain in **Settings → Pages → Custom domain**
4. Enable **Enforce HTTPS**

---

## Adding a New Tournament

1. Log in as admin on the home screen
2. Click **+ New Tournament** in the Admin Panel
3. Fill in tournament name, dates, score feed URL, start date, buy-in, and status
4. Click **Tiers** to assign golfers to each tier (one golfer per line)
5. Set picks phase to **Weekday** to open Thursday & Friday submissions
6. Share the site link with participants
7. Switch picks phase to **Weekend** before Saturday picks open
8. Archive the tournament when it concludes

---

## Roadmap

- [ ] Snapshot archived tournament scores locally (so results persist if feed goes offline)
- [ ] Support for PGA Tour events via SportsDataIO or Sportradar API
- [ ] Multiple leagues/groups per tournament
- [ ] User accounts with pick history
- [ ] Mobile app (iOS/Android)
- [ ] Entry fee payment processing

---

*Built for the love of golf and friendly competition.*
