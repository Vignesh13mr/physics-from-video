# 🎯 Physics from Video

Film something real. Get the physics out of it.

Point a phone camera at a swinging pendulum, a thrown ball or a bouncing spring (or record a
plucked string) and this tool tracks it, converts pixels to metres, fits the physics
equations, and tells you things like **g**, **spring constant**, **damping** and **pitch**.

Every result is checked against ground truth: the demo generator creates videos with *known*
physics, and the test-suite verifies the tool recovers it.

## Experiments (modules)

| Module | Input | Discovers |
|---|---|---|
| `pendulum` | video | period, damping, length *or* g |
| `projectile` | video | g, launch speed, launch angle, max height |
| `spring` | video | frequency, spring constant k, damping |
| `sound` | .wav | fundamental frequency, note, harmonics, string wave speed |

## Quick start

```bash
git clone https://github.com/Vignesh13mr/physics-from-video.git
cd physics-from-video
python -m venv venv
venv\Scripts\activate          # Windows   (Mac/Linux: source venv/bin/activate)
pip install -r requirements.txt

python examples/make_demo_videos.py        # creates demo files with known physics
python run.py pendulum examples/pendulum.mp4 --m-per-px 0.002 --length 0.6
streamlit run app/streamlit_app.py         # web interface
pytest                                      # run the tests
```

## Use it on your own video

1. Film an object with a **bright, solid colour** (orange ball, yellow tape) against a plain background.
2. Keep the phone **steady** (tripod or leaned on something) and film **straight-on**.
3. Include something of **known length** in the frame (a metre stick, a ruler) and measure how many
   pixels it spans, e.g. 1 m = 500 px → `--ref-px 500 --ref-m 1.0`.
4. Run, for example:

```bash
python run.py pendulum my_swing.mp4 --ref-px 500 --ref-m 1.0 --colour green --length 0.6
python run.py projectile my_throw.mp4 --m-per-px 0.004 --start 0.5 --end 1.4
```

If the object isn't found, change `--colour`, or give exact `--lower H S V --upper H S V` values.

## How it works

```
video ──► tracker (OpenCV colour mask) ──► pixels ──► calibration ──► metres
      ──► clean/resample ──► physics module (curve fit with SciPy) ──► parameters + plot
```

* `core/` – tracking, calibration, filtering, curve fitting, audio FFT
* `modules/` – one file per physics experiment (all share the same interface)
* `app/` – Streamlit web app
* `tests/` – recover known physics from synthetic data and generated videos

Damped oscillators (pendulum, spring) are fitted to  x(t) = A·e^(−γt)·cos(ωt + φ) + c.
Projectiles are fitted with a quadratic in y(t) and a line in x(t).

## Add your own experiment

1. Create `modules/my_experiment.py` with a class that extends `PhysicsModule` and returns a `Result`.
2. Register it in `modules/__init__.py`.
3. Add a test in `tests/`.

Ideas: inclined plane friction, collisions (momentum), rolling disk, falling in fluid, Torricelli draining.

## Limitations

* Tracking uses colour, so lighting and background matter.
* Models are simple (small-angle pendulum, no air drag for projectiles).
* Accuracy depends on camera steadiness and a good scale reference.

## Roadmap

- [ ] Inclined-plane friction module
- [ ] Collision (momentum) module
- [ ] Air-drag projectile model
- [ ] Automatic equation discovery with SINDy

