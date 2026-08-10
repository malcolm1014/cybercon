# Adding to CYBER-SEC CON

You don't need to write logic to grow this game. Everything below is fill-in-the-blank editing inside `cybercon.html`. Open it in any text editor.

**The golden rule: after any change, load the game and type `VALIDATE` in the terminal.** It checks every room, exit, hotspot and puzzle and tells you in plain English what's broken. If it says `0 errors`, your content works.

---

## 1. Adding art (no code at all)

Make a folder called `scenes` next to `cybercon.html`. Name each picture after the room id:

```
cybercon.html
scenes/
  talkhall.jpg
  chillout.jpg
  mynewroom.jpg
```

That's it. The game looks for `scenes/<room id>.jpg` (also `.jpeg`, `.png`, `.webp`) and uses it automatically. If a picture isn't there, the room falls back to its coloured glow and nothing breaks — so you can add art gradually.

To find a room's id, walk into it and type `VALIDATE NOTES`; it lists rooms with no art by id.

Landscape images around 900px wide look best. The neon colour wash is layered on top automatically.

---

## 2. Adding a room

Two entries, both near the top of the big `<script>` block.

**A. The room's look and clickable things** — add to the `NEWSCENES` list:

```js
{id:'voting', name:'Voting Village', look:'blue',
 hs:[{x:50, y:50, type:'item', label:'Examine the machine',
      note:'A decommissioned voting machine, casing off, three people arguing about it.'},
     {x:15, y:85, type:'nav', to:'atrium', label:'Back to the atrium'}]},
```

- `id` — short, lowercase, no spaces. This is also your image filename.
- `look` — one of `neon`, `neon2`, `amber`, `violet`, `blue`, `redblue`, `graffiti`, `dawn`.
- `hs` — the clickable dots. `x`/`y` are percentages across the picture.
- Hotspot types: `nav` (go somewhere, needs `to`), `item` (needs `note`; add `gives:'Thing'` to hand over an item), `puzzle` (needs `id`), `door` (needs `door`).

**B. The room's text description** — add to `NEWROOMS`:

```js
voting:{ desc:"Decommissioned voting machines on folding tables, casings off.",
         exits:{s:'atrium', out:'atrium'} },
```

**C. Connect it** — add one line to the `LINKS` list inside `wireExpansion()`:

```js
['atrium', 'ne', 'voting', 'Voting Village'],
```

That reads as: *from the atrium, going northeast reaches voting, and put a dot labelled "Voting Village" in the atrium.* The way back is created for you, and the map position is worked out automatically.

Write a nudge for it in `HINTS` too:

```js
voting:'Look at the machine, then talk to the people arguing.',
```

---

## 3. Adding a puzzle

Add one `definePuzzle({...})` block. You get the pop-up panel, the typed command, the reward, the teaching note, and progress tracking for free. Four kinds:

**Pick the one right answer:**

```js
definePuzzle({
  id:'ports', kind:'choice', room:'cablefarm', hotspot:[60,40,'A switch with a labelled port'],
  title:'Which Port?',
  intro:'Which of these is the standard port for secure shell?',
  options:['21','22','80','3389'],
  correct:1,
  reward:'Networking Ribbon',
  lesson:'22 is SSH. 21 is FTP, 80 is unencrypted HTTP, 3389 is remote desktop.',
  verb:'port', argHint:'<number>',
  hint:'It is one number away from the old file transfer port.'
});
```

**Pick everything that applies** — use `kind:'multi'` and mark each option:

```js
options:[ {text:'Reused password', yes:true}, {text:'Long passphrase', yes:false} ],
```

**Click things in order** — use `kind:'sequence'` with `options` (the buttons) and `order` (the correct sequence).

**Type the answer** — use `kind:'text'` with `answers:['HACK THE PLANET']` and optionally `display:'the string to show'`. Answers ignore case and punctuation.

Optional fields on any puzzle: `footnote` (a second line of flavour), `wrongNote` (shown on a wrong answer), `wrongToast`.

Then add its id to the `ALL_PUZZLES` list and give it a friendly name in `PUZZLE_NAMES` so it counts toward the `BADGE` progress command.

---

## 4. Adding history to the ARCHIVE

One line in `LORE`:

```js
voting:`<b>The Voting Village</b> — launched in 2017; its reports fed into real legislation.`,
```

Players reach it by typing `ARCHIVE voting`.

---

## 5. Keeping puzzles honest

The one rule that matters for engagement: **never put the answer in `HELP` or in a `hint`.** Point at *where the clue is* instead. Good hint: "Did you watch the presentation on the Main Stage?" Bad hint: "The order is circle, triangle, square, diamond."

Put the actual clue somewhere in the world — an `item` note in another room is the classic move. That's what makes players explore.

---

## 6. Commands for you, the author

| Type this | What it does |
|---|---|
| `VALIDATE` | Full check of every room, exit, hotspot and puzzle |
| `VALIDATE NOTES` | Optional notes: rooms missing art or hints |
| `BADGE` | Progress: what's solved, what's untouched |
| `MAP` | Visual check that your new room connected properly |
| `ARCHIVE` | Index of history topics |

---

## 7. If it does break

`VALIDATE` messages are written to be self-explanatory. The most common ones:

- *"has an exit pointing at X, which does not exist"* — typo in a room id.
- *"cannot be reached from the entrance"* — you added the room but not the `LINKS` line.
- *"has no description entry"* — you added it to `NEWSCENES` but not `NEWROOMS`.
- *"opens puzzle X but nothing defines it"* — hotspot id doesn't match your `definePuzzle` id.

Ids must match **exactly** in all three places: `NEWSCENES`, `NEWROOMS`, and your image filename.

---

## 8. The badge meta-challenge

Six village puzzles each drop a stamped coupon; collected in slot order they spell **AGENCY**, which the player enters at the Badge Challenge Terminal (east of the Goon Command Post) to win the Black Badge and the top ending.

**To make one of your new puzzles drop a coupon**, add a line to `FRAGMENTS`:

```js
ports: {slot:7, letter:'X', from:'the switch',
        flavour:'A coupon is taped under the switch, stamped 7:X'},
```

The key must match your puzzle's `id`. Coupons are handed out automatically when the puzzle is solved — you don't wire anything up, because every puzzle calls `markSolved()` and that's where the hook lives.

If you add a seventh coupon you must also change `META_ANSWER` to the new word the letters spell. Keep the `slot` numbers sequential from 1.

**To add another ending tier**, push a function onto `ENDING_HOOKS`. Return a string to claim the ending, or `null` to pass. Later hooks win, so the most exclusive ending should be added last:

```js
ENDING_HOOKS.push(()=> state.inventory.includes('Golden Ticket')
  ? 'Your closing line here. <b>[SOME ENDING]</b>' : null);
```

## 9. One thing that will bite you

If you add a puzzle **without** `definePuzzle` (i.e. you hand-write the panel), you must add its id to `PUZZLE_REGISTRY` or `VALIDATE` will report it as undefined. `definePuzzle` registers for you.

---

## 10. Adding NPCs (the hallway track)

NPCs are the easiest content to add and the best place for real-world learning, because a person can explain something a room cannot. Add one entry to `NPCS`:

```js
lockpicker: {
  name:'TOOOL volunteer', room:'hwvillage', at:[70,60],
  look:'Someone with a cutaway padlock in each hand.',
  greeting:'"Want to see why that lock is theatre?"',
  topics:{
    pins:['pin tumblers',
      '"First line of the answer."',
      '"Optional second paragraph."'],
  },
  fallback:'"Ask me about pin tumblers."'
},
```

- `at:[x,y]` — position as percentages; the green dot is placed for you.
- Each topic is an array: `[button label, first line, optional more lines...]`.
- `TALK TO`, `ASK ... ABOUT`, `WHO`, and the clickable panel all work automatically.
- Players earn ribbons at 8 and 20 topics discussed, so more topics means more reward.

**Keep it factual.** The NPCs in this build discuss real, verifiable research — the DEF CON 34 badge and Baochip, IRIS chip inspection, autonomous-agent CTFs, AWS IAM escalation paths, hospital ransomware attribution. Invent the person, not the facts, and add an `ARCHIVE` entry (`LORE`) for anything you want players to be able to look up afterwards.
