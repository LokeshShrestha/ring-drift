# Ring Drift

A true-to-scale solar system you can fly through in the browser. One HTML file, no build step, no libraries.

**Live:** https://REPLACE_USER.github.io/ring-drift/

Built with [Claude Code](https://claude.com/claude-code). [`MASTER_PROMPT.md`](MASTER_PROMPT.md) is a single prompt written to rebuild the whole thing in one go.

## What it does

- **True scale.** Every radius and distance uses the same scale (Earth radius = 40 units, 1 unit ≈ 159 km). The Sun is 109 Earths wide; Neptune is 30 AU out.
- **Real positions.** Planets start where they are today (circular orbits from J2000 mean elements, accurate to a few degrees) and keep orbiting. Real axis tilts and day lengths.
- **32 bodies.** The Sun, 8 planets, Pluto, and 21 major moons, each with a facts card. Teleport to any of them.
- **Real maps** for the Sun, Mercury, Venus, Earth, the Moon, Mars, Jupiter, Saturn, Uranus and Neptune. Earth has city lights and clouds, and turns to match the real UTC time. Moons are tidally locked.
- **Atmospheres** for Earth, Venus, Mars and Titan (single-scattering Rayleigh + Mie).
- **Saturn's rings** with gaps, ring shadows, and thousands of instanced rocks you can fly between.
- **Soft eclipse shadows** between bodies, anti-aliased planet edges, and far planets drawn as points of light.
- **Time control:** real time up to 1 year per second, or paused.
- **Generative music** made in code (no recordings, nothing to license).
- **Touch controls** on phones and tablets.

## Controls

| Action | Keys | Touch |
|---|---|---|
| Look | Mouse | Drag |
| Fly | W A S D, E / Q up/down | Hold one finger |
| Boost | Hold Shift | Two fingers |
| Speed | Mouse wheel | |
| Travel menu | T | ≡ button |
| Sun / planets | 0 – 9 | |
| Saturn's rings | R | |
| Time slower / faster / pause | [ / ] / P | − + ⏸ buttons |
| Music / stats | M / H | ♪ button |

## Detail levels

| Level | Resolution | Ring rocks | Maps |
|---|---|---|---|
| Ultra low | 45% | none | none (procedural look) |
| Low | 60% | 1,500 | 2k |
| Medium | 85% | 3,500 | 2k |
| High | 100% | 6,000 | sharper Earth, Moon, Sun, Jupiter, Saturn |

Resolution also scales down on its own to hold about 60 fps.

## How it works

- The whole scene is ray-traced in one fullscreen WebGL2 fragment shader: analytic ray–sphere hits, analytic ring plane.
- Positions are kept in 64-bit floats on the CPU and sent to the GPU relative to the camera, so true-scale distances don't lose precision.
- Only the body under each pixel gets shaded. Bodies smaller than 2 pixels become glows, and the CPU picks which bodies can cast shadows each frame.
- The star field and Milky Way are baked into a cubemap once at startup.
- Maps download when you get within 500 radii of a body and are freed from GPU memory past 5,000 radii.

## Run locally

Open `index.html` in Chrome, Edge or Firefox. An internet connection is needed for the planet maps; without one, bodies use the procedural look.

## Credits and license

- Code: [MIT](LICENSE).
- Planet maps: [Solar System Scope](https://www.solarsystemscope.com/textures/), based on NASA data, [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/), loaded from Wikimedia Commons. They are not stored in this repo.
- Inspired by [Noniv/abyssal-garden](https://github.com/Noniv/abyssal-garden).
