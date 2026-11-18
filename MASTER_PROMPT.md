# Master prompt

Paste everything below the line into a coding agent (e.g. Claude Code) in an empty folder. It is meant to produce the whole project in one prompt. It has not been tested.

---

Build **Ring Drift**: a true-to-scale, explorable solar system in a single `index.html` (WebGL2, no libraries, no build step, no external assets except the planet maps listed below). Work autonomously, don't ask questions, and test it in a browser before you finish.

## Scale and precision
- One scale for everything: Earth radius = 40 units (`KM = 40/6371`, `AU = 149597870.7*KM`). True radii for every body (Sun ≈ 4372 units, Phobos ≈ 0.07) and true orbit distances for planets and moons.
- Keep body and camera positions in float64 on the CPU. Each frame send positions to the GPU **relative to the camera** (camera at origin in the shader), so 30 AU distances don't break float32.

## Bodies (32)
Sun; Mercury; Venus; Earth + Moon; Mars + Phobos, Deimos; Jupiter + Io, Europa, Ganymede, Callisto; Saturn + Titan, Rhea, Iapetus, Dione, Tethys, Enceladus, Mimas; Uranus + Titania, Oberon, Umbriel, Ariel, Miranda; Neptune + Triton (retrograde), Proteus; Pluto + Charon.
- Each body has: radius (km), orbit distance, real orbital period, kind label, a small facts table (radius, distance, day, year, moons, temperature…) and one interesting fact.

## Motion
- Simulation time = days since J2000 (`Date.UTC(2000,0,1,12)`), starting at `Date.now()`.
- Planets: circular orbits from J2000 mean longitude, period, inclination and ascending node; ecliptic → world with y = ecliptic north.
- Real pole directions (ecliptic lon/lat) and rotation periods for the Sun and planets (Venus and Uranus retrograde / tilted).
- Moons orbit in their parent's equatorial plane (Earth's Moon in the ecliptic) with real periods, and are **tidally locked** so the parent always sits over map longitude 0.
- Earth's spin is set so the subsolar longitude matches real UTC time.
- Time rates: real time, 1 min/s, 10 min/s, 1 h/s (default), 6 h/s, 1 day/s, 1 week/s, 1 month/s, 1 year/s, plus pause. `[` `]` `P` keys and on-screen − ⏸ + buttons; show the simulated UTC date/time.

## Rendering (one fullscreen fragment shader)
- Analytic ray–sphere over all bodies (uniform arrays: position+radius, axis+spin angle, two colours + shader type + param).
- **Anti-aliased silhouettes** from analytic edge coverage (`(R − missDistance) / pixelSize`).
- Bodies smaller than ~2 px are drawn as **points of light** (glow scaled by size, phase and inverse-square sunlight) instead of being shaded.
- Shade only the nearest hit per pixel. Lighting = true inverse-square sunlight times an eye-adaptation exposure based on the camera's distance from the Sun; ACES tonemap; dither.
- Body-local coordinates via axis + spin; equirectangular map lookup with explicit LOD from pixel footprint (`textureLod`) so there is no seam line.
- Procedural fallbacks per type when no map is loaded: sun (granulation), gas giant (bands, storms, Great Red Spot), ice giant, Earth (continents, oceans with specular, ice caps, clouds, city lights), cratered rock/ice (crater bump mapping), hazy (Titan/Venus), Mars, Io (volcanic), Europa (lineae), two-tone Iapetus.
- **Soft eclipse shadows** with penumbra. The CPU picks, each frame, only the bodies whose shadow can reach another body or Saturn's rings (max 8) and passes that list to the shader.
- **Atmospheres** (single-scattering Rayleigh + Mie, phase g = 0.76) for Earth, Venus, Mars, Titan with their own colours and scale heights; compute only the nearest one.
- **Saturn's rings**: analytic plane in Saturn's equatorial frame, radial density with C/B/A rings, Cassini division and Encke gap, noise detail faded by `fwidth`, lit/unlit side, ring shadow on Saturn and planet shadow on the rings.
- Sun corona glow and a lens glare that dims when the Sun is eclipsed.
- **Sky**: bake once at startup into a mipmapped RGBA8 cubemap — two star layers with colour variety, a Milky Way band with dust lanes and a core glow, one faint nebula.
- **Ring rocks**: instanced subdivided icospheres with vertex-noise displacement and random spin, placed in Saturn's ring frame on the sunlit side, drawn in a second pass that depth-tests against the ray-traced depth (`gl_FragDepth`); only drawn within ~2500 units of Saturn.

## Planet maps (real, free to use)
Use the Solar System Scope textures (CC BY 4.0) from Wikimedia Commons (`https://upload.wikimedia.org/wikipedia/commons/…`, which sends CORS headers): 2k maps for Sun, Mercury, Venus atmosphere, Earth day/night/clouds, Moon, Mars, Jupiter, Saturn, Uranus, Neptune. On High use sharper ones for Earth, Moon (3840 px thumbnails of the 8k files), Sun, Jupiter, Saturn (the 4096 px "8k" originals). Look up the exact file paths with the Commons API.
- Upload as `SRGB8_ALPHA8` with mipmaps. Load a map only when the camera is within 500 radii of its body; free it past 5000 radii. On a failed download fall back to 2k, then to the procedural look, and retry after 30 s.
- Credit Solar System Scope, CC BY 4.0, in the menu.

## Controls
- Desktop: pointer-lock mouse look, WASD, E/Q up/down, Shift boost ×8, wheel sets cruise speed. Speed scales with distance to the nearest surface; collisions keep you just above every surface.
- Touch: drag to look, one finger flies forward, two fingers boost, ≡ button opens the menu. `touch-action: none` on the canvas.
- T travel menu, 0–9 Sun and planets, R Saturn's rings view, M music, H stats (fps, resolution, speed, distance from Sun).
- Teleporting puts the camera at ~3.2 radii, three-quarter lit, and **follows** that body as it orbits.

## UI
- Menu: title, short tagline, Launch button, controls table, Time row (Slower / rate / Faster / Pause), Detail row (Ultra low / Low / Medium / High, saved to localStorage, reloads), credit line. Right column: travel list, each planet followed by its moon buttons (Saturn also gets “Rings”), “no moons” where none.
- Info card (bottom left) for the body you are nearest to (relative to its size): name, kind, facts table, fact.
- Dark glassy panels, thin borders, warm accent colour, system font.

## Detail levels and performance
| Level | Resolution | Rocks | Noise octaves | Atmosphere steps | Sky size | Maps |
|---|---|---|---|---|---|---|
| Ultra low | 45% | 0 | 2 | 6 | 512 | none |
| Low | 60% | 1500 | 3 | 8 | 1024 | 2k |
| Medium | 85% | 3500 | 4 | 12 | 1536 | 2k |
| High | 100% | 6000 | 5 | 16 | 2048 | 2k + sharper |
- Default to Low on touch devices or ≤4 GB memory, else High.
- Dynamic resolution between 50% and 100% to hold ~60 fps.

## Music
An original generative score written with WebAudio (no samples, nothing to license): slow D-Lydian pad chords (detuned saws through a slowly opening low-pass), a soft sine bass, and a sparse wandering FM-bell melody with occasional rising “question” intervals, through a feedback echo and a generated reverb, plus a faint low rumble. Look-ahead scheduler; off until the first Launch click; M toggles.

## Done when
It opens with no console errors, every body can be visited from the travel menu and looks right (real maps where available), Saturn's rings and rocks render, time controls and pause work, Ultra low renders, and the frame rate stays reasonable on integrated graphics.
