# Collide Together

A starter sketch for two computers. Each player has three objects. Click one of yours to
select it and drag it around; let go while it's moving to throw it. The controls above the
canvas edit the selected object: its message, **push** or **pull** with a force from 0 to 10,
its look, and whether it leaves a **trail**.

When one of your objects touches one of your partner's, both messages pop up in speech bubbles
and **each object's force moves the other object**: a push knocks it away, a pull flings it
towards and through. A held object stays in your hand, so dragging a strong pusher into a
resting object sends that object flying. Each object is labelled "you" or "them".

**Try it now:** https://geitenkaas.github.io/CreativeTech_CollideTogether/?room=pick-a-name
(open it on two computers, using the same room name).

## Get your own copy

1. On this repo's GitHub page, click **Use this template → Create a new repository**, and make
   it public. Work in pairs: one of you makes the copy and adds the other as a collaborator, so
   you both run the same sketch.
2. Clone it and open the folder in VS Code.
3. To put it online, go to **Settings → Pages**, choose **Deploy from a branch**, **main**,
   **/ (root)**, and save. After a minute, your sketch is at
   `https://<your-username>.github.io/<repo-name>/`.

## Run it

```bash
python -m http.server 8000
```

Open `http://localhost:8000/?room=pick-a-name` on both computers, using the same room name.
Until someone joins, two grey dummies sit in the middle so you can practise on your own. To test
alone, open the page in two different browsers.

Across two computers, the page has to be reachable from both. The simplest way is to put it on
GitHub Pages and have both of you open the same link. It doesn't use the camera, so it needs no
certificates.

## How it works (`sketch.js`)

| Section | What it does |
|---|---|
| **LOOKS** | The ways an object can be drawn. Each look has `r` (its hit radius) and `draw(o)`, which draws it centred on 0, 0. The three examples are a vector circle, a PNG (`assets/star.png`) and an 8-frame sprite sheet (`assets/blob-sheet.png`). |
| **OBJECTS** | `move()`: a dragged object follows the mouse; a free one slides, slows down (`FRICTION`) and bounces off the edges. Your partner's objects glide towards the positions that arrive over the network (`follow()`). |
| **MOUSE** | `mousePressed()` picks up the top-most of *your* objects under the mouse and selects it. You can't pick up your partner's objects. |
| **COLLISION** | `collides(a, b)`: two objects touch when the distance between their centres is less than their two radii added together. `checkEncounters()` tests each of your objects against each of theirs and fires `encounter(a, b)` once, on the frame a pair *starts* touching, with a one-second cooldown per pair. `knock(target, hitter)` applies the hitter's push or pull. |
| **DRAWING** | Trails redraw the last 25 positions, more faintly the older they are. The selected object has a dashed ring. |
| **CONTROLS** | `pick(o)` copies the selected object's settings into the controls, and `readControls()` copies them back every frame. |
| **NETWORK** | [p5LiveMedia](https://github.com/vanevery/p5LiveMedia) in DATA mode. Each browser sends a list of its objects as JSON 20 times a second. |

**Who moves what:** each computer only ever moves its own objects. Both screens spot the
encounter. Your screen applies their object's force to your object, and their screen applies
your object's force to theirs. So your force does move their object, but their computer does
the moving. That way the two screens can't disagree about where anything is. The dummies have no
owner, so your screen knocks them itself.

## Make it yours

- **Add a look**: add an entry to the object returned by `makeLooks()`. Load any images
  with `await loadImage(...)` at the top of it. It appears in the dropdown automatically. Set
  `r` to roughly half the drawing's width. Press **H** to show the hit circles while you
  adjust it.
- **Change the encounter**: everything that happens on contact is in `encounter()`. You could
  play a sound, change colour, swap looks, spin, or keep score.
- **More objects**: change `HOW_MANY`.
- **Send more**: anything you add to each object in `sendMine()` arrives on the other computer in
  `gotData()`.
- Both players need the same copy of the sketch, because a look the other copy doesn't have is
  drawn as a circle.

## Limits

- Two players per room. A third player's updates are ignored.
- The connection relies on the public p5LiveMedia server at `p5livemedia.itp.io`.
- Data goes peer to peer over WebRTC. Some networks (often campus Wi-Fi) block this, and then
  the partner never appears. Try a phone hotspot.
- The canvas is a fixed 960 × 600 so that positions mean the same thing on both screens.
