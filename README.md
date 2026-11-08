# The Mountain Climber

A climbing game by Greta. Open `index.html` in a web browser to play.

## How to play

- Arrow keys or A / D to run, Space or Up to jump, press jump again in the air to double jump.
- Climb through the Meadow, the Mountains, the Icy Peaks and Outer Space to reach the top.
- Blue flag platforms are checkpoints. Press R to go back to your last one.
- Purple platforms move, orange ones crumble, yellow ones bounce, light blue ones are icy.
- Jump on goats, birds and aliens to bonk them. Stay away from bears!
- Reach the top as fast as you can and put your name on the leaderboard.

## Changing the game

All the settings are at the top of `index.html` in the SETTINGS section. You can change
how high you jump, how fast you run, how many platforms there are, your jacket color, and
the level number (which gives you a brand new tower).

---

## For grown-ups: putting the game on the internet

### Step 1: host the game (free)

The game is a single `index.html` file, so GitHub Pages works well:

1. Push this folder to a GitHub repository.
2. In the repository, go to Settings, then Pages.
3. Under "Build and deployment" choose "Deploy from a branch", pick `main` and the root folder, and save.
4. After a minute the game is live at `https://<username>.github.io/<repository>/`.

### Step 2: the shared online leaderboard (optional, free)

Out of the box the leaderboard only remembers times in each player's own browser.
To share one leaderboard between everyone, the game can store times in a free
Firebase Firestore database. No server code is needed. Setup takes about ten minutes.

1. Go to https://console.firebase.google.com and sign in with a Google account.
2. Click "Add project", give it a name (for example `mountain-climber`), and turn off
   Google Analytics when asked. Finish creating the project.
3. In the left menu open Build, then Firestore Database, and click "Create database".
   Pick a location near you and start in **production mode**.
4. Open the Rules tab and replace the rules with the ones below, then click Publish.
   They let anyone read the scores and add a score, but nobody can change or delete
   scores, and a score must have a short name and a whole number for the time.

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /scores/{scoreId} {
         allow read: if true;
         allow create: if request.resource.data.keys().hasOnly(['name', 'frames', 'date'])
                       && request.resource.data.name is string
                       && request.resource.data.name.size() > 0
                       && request.resource.data.name.size() <= 12
                       && request.resource.data.frames is int
                       && request.resource.data.frames > 0
                       && request.resource.data.date is string;
         allow update, delete: if false;
       }
     }
   }
   ```

5. Click the gear icon next to "Project Overview", then Project settings. Under
   "Your apps" click the web icon (`</>`), register an app with any nickname, and you
   will see a config block. Copy the `projectId` and `apiKey` values from it.
6. Open `index.html`, find the line near the top that says

   ```
   const ONLINE_LEADERBOARD = null;
   ```

   and change it to (using your own values):

   ```
   const ONLINE_LEADERBOARD = { projectId: 'mountain-climber-1234', apiKey: 'AIza...' };
   ```

7. Save, push to GitHub, and the leaderboard is now shared by everyone who plays.

Notes:

- The Firebase API key is meant to be public in a web page. The rules above are what
  protect the data. If you ever want to clear the leaderboard, delete the documents in the
  `scores` collection in the Firebase console.
- Optional but recommended: in the Google Cloud console you can restrict the API key so it
  only works from your GitHub Pages address (APIs & Services, Credentials, pick the key,
  "Website restrictions").
- The free Firestore plan allows tens of thousands of reads a day, which is plenty for a
  family and friends leaderboard.
