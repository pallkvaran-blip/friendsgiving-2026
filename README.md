# Friendsgiving potluck board

Live at https://pallkvaran-blip.github.io/friendsgiving-2026/

- `index.html`: the whole page (HTML, CSS and JavaScript in one file). No build step.
- `board.json`: the starting board. Used only while no shared database is set.

GitHub Pages serves the `gh-pages` branch. Keep it in step with `main`:

    git push origin main main:gh-pages

## Shared editing

GitHub Pages only serves files, so edits are kept in a Firebase Realtime Database.
In `index.html`, set `DB_URL` to the database address, for example

    var DB_URL = 'https://friendsgiving-5cff6-default-rtdb.firebaseio.com';

With it set, anyone who opens the page can edit. Changes save by themselves about a second
after each edit, and every open page re-reads the board every 3 seconds. With `DB_URL` empty
the page is view-only and re-reads `board.json` every 5 seconds.

Database rules (Firebase console, Realtime Database, Rules):

    {
      "rules": {
        ".read": false,
        ".write": false,
        "board": {
          ".read": true,
          ".write": "newData.exists()",
          ".validate": "newData.hasChildren(['rev', 'json'])",
          "rev": { ".validate": "newData.isNumber() && (!data.exists() || newData.val() === data.val() + 1)" },
          "json": { ".validate": "newData.isString() && newData.val().length < 60000" },
          "$other": { ".validate": false }
        }
      }
    }

The `rev` rule is what stops two people saving at the same moment from overwriting each
other: the second save is refused, and that page loads the first one's board, replays its
own change on top and saves again.

Anyone with the page link can edit the board, and nobody can delete it outright.

## Shape of the board

- `event`: `kicker` (small heading), `line` (date, time, place), `start` (ISO time for the countdown).
- `guests`: one entry per seat. `id` is any unique string, `members` is the list of names sharing
  that seat, `note` is optional.
- `dishes`: `id`, `name`, `emoji`, `cat` (`main`, `side`, `sweet` or `extra`), and `by` set to a
  guest `id` when someone has claimed it. `core: true` counts toward the pie meter and
  `wanted: true` puts the dish in the "Most wanted" group. `custom: true` marks a dish someone added.
