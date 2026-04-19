# Calibrating the Stop Coordinates

The Echo-Tour app triggers a video when your phone comes within a certain
radius of a stop. The current coordinates inside `index.html` are
approximations from memory — to make the auto-trigger land at the right
spot on the walk, Miguel (or anyone) needs to replace them with real
GPS readings taken on-site.

---

## Where the coordinates live

All eight stops are defined in one place — the `HOTSPOTS` array near the
top of the `<script>` block in **`index.html`**.

Look for the block that starts with:

```js
const HOTSPOTS = [
  {
    id: '00',
    title: 'Prologue',
    subtitle: 'Before the Walk',
    era: '',
    duration: '—',
    coords: [38.71380, -9.16085],     // ← edit this line
    description: '...',
    video: 'videos/00.mp4',
    isIntro: true
  },
  {
    id: '01',
    title: 'Jardim da Estrela',
    ...
    coords: [38.71388, -9.16125],    // ← and this
    ...
  },
  ...
];
```

Each `coords` entry is `[latitude, longitude]` — **latitude first**,
decimal degrees, six digits of precision is plenty (six decimals ≈
11 cm on the ground).

---

## Getting accurate readings on-site

You need to stand at the exact spot where you want the video to start
and record your GPS position. Three ways to do it, easiest first.

### Option A — Use the browser console (recommended)

This is the most accurate and doesn't require installing anything.
You need to be connected to the internet and able to open the app.

1. Open the deployed app on your phone: **https://zeah-lda.github.io/Echo-Tour/**
2. Stand exactly where you want the video to trigger.
3. Open the browser's developer console:
   - **iPhone Safari:** Settings → Safari → Advanced → Web Inspector (ON).
     Then connect the phone to a Mac running Safari, open
     *Develop → [your iPhone] → echo page*. You'll see the console there.
   - **Easier:** use a web-based console bookmarklet like
     [Eruda](https://github.com/liriliri/eruda). Paste the Eruda
     bootstrap line as a bookmark URL, tap it while on the app page,
     and a console opens in-page.
4. Paste this line into the console and press Enter:

   ```js
   navigator.geolocation.getCurrentPosition(p =>
     prompt('Copy these coords', `${p.coords.latitude.toFixed(6)}, ${p.coords.longitude.toFixed(6)}`)
   );
   ```

5. You'll see a prompt with the coordinates you can copy, e.g.
   `38.713912, -9.161378`.
6. Repeat at every stop.

For extra accuracy, stand still for ~20 seconds before reading — the
GPS needs a moment to average. The **accuracy** reading
(`p.coords.accuracy`) tells you the radius, in metres, the phone is
confident about. Under 10 m is good.

### Option B — Use a GPS-coordinate app

Any compass / "my-location" app works. Some good ones:
- **iOS:** the built-in *Compass* app shows lat/long at the bottom.
- **Android:** *GPS Coordinates* (Google Play, free).
- **Any phone:** Google Maps — long-press the blue dot, and it shows
  your exact location in decimal degrees.

Stand still, note the values, move on.

### Option C — Google Maps on desktop (least accurate but quick)

If you know the spot visually, open Google Maps on a desktop,
right-click the exact location, and the coordinates appear at the top
of the menu. Decent for rough placement, but GPS taken on-site is
always more accurate because Google Maps pins satellite imagery which
can be a few metres off.

---

## Editing the coordinates

Once you have the real readings, open `index.html` and paste them into
the matching `coords` field. The format is:

```js
coords: [LATITUDE, LONGITUDE],
```

**Example — before:**

```js
{
  id: '03',
  title: 'Liceu Pedro Nunes',
  ...
  coords: [38.71650, -9.16130],
  ...
}
```

**After — replaced with a real reading of `38.716428, -9.161275`:**

```js
{
  id: '03',
  title: 'Liceu Pedro Nunes',
  ...
  coords: [38.716428, -9.161275],
  ...
}
```

Save the file. No rebuild step — it's pure HTML/CSS/JS.

---

## The trigger radius

A stop fires when you step within **30 metres** of its coordinates
(the default). You can change this at runtime in the app:

**Settings drawer → Trigger Radius** — arrows to increase or decrease
in 10 m steps, range 10–200 m.

Guideline:
| Terrain                    | Radius     |
|----------------------------|------------|
| Open square, gardens       | 25–30 m    |
| Narrow Alfama-style street | 15–20 m    |
| GPS-weak alley / archway   | 40–50 m    |

If you want the default to change for everyone, not just your device,
edit `state.settings.triggerRadius: 30` in `index.html` and push.

---

## Testing a new reading

Two ways:

1. **Walk to the spot.** Open the app, walk to the coordinates, see
   if the trigger fires at the right moment.
2. **Browser DevTools sensor spoof.**
   - Chrome/Edge desktop: DevTools → three-dot menu (top-right of the
     DevTools panel) → **More tools → Sensors**.
   - Set *Location* → *Other* → paste your latitude and longitude.
   - Reload the page. The compass will act as though you're there.

---

## Pushing the changes back to GitHub

After saving `index.html`:

```bash
cd "C:/Users/hugom/Documents/App miguel"
git add index.html
git commit -m "Refine coords for stops 01, 03, 05 from on-site readings"
git push
```

GitHub Pages will republish the site within ~1 minute. The live URL
picks up the new coordinates on next page load.

---

## A couple of notes

- **Latitude first, longitude second.** Mixing them up sends you to a
  completely different part of the world.
- **Negative longitudes** for Portugal (you're west of Greenwich).
  Double-check the minus sign is there: `-9.16125`, not `9.16125`.
- **6 decimals = 11 cm**, **5 decimals = 1.1 m**, **4 decimals = 11 m**.
  Four is enough for outdoor stops; six gives perfect precision if
  your GPS agrees.
- **Indoor stops** (like inside Casa Fernando Pessoa) will never get a
  great GPS fix. Put the coordinates at the entrance/doorway and bump
  the trigger radius for that stop.
