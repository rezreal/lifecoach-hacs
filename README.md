# LifeCoach Home Satellite (HACS integration)

A Home Assistant **custom integration** that bridges an HA install and the
LifeCoach server. It is a thin push adapter — it runs **no inference** and
reasons about **no goals**. See [`docs/satellites/home.md`](../../docs/satellites/home.md)
for the full design.

## What it does

- **Device state** — forwards state changes of the entities you pick (a TV, a
  vacuum, switches, climate, motion/contact sensors) as generic
  `home.device_state` events. Durations and frequencies are derived server-side.
- **Presence** — motion/presence sensors also emit `home.presence`.
- **Camera snapshots** — on motion (with a per-area cooldown) or on a schedule,
  grabs a still and POSTs it to the server's `/v1/vision/frame`. The **server**
  runs the vision model and **discards the frame** — nothing is stored, and no
  frame ever touches HA disk beyond the in-memory grab.

## Privacy

Camera inference is off by default and gated by **two config-time consents**:

1. **You confirm you live alone** (single-occupancy attestation), and/or
2. **You select which areas may be captured** — other rooms are never
   photographed.

There is no per-frame person-count gate; the config-time consent above is the
safeguard. Each observation type (smoking, drinking…) is enabled server-side.

## Install

1. In HACS: **⋮ → Custom repositories**, add `https://github.com/rezreal/lifecoach-hacs`
   with category **Integration**, then download it. (Or copy `custom_components/lifecoach/`
   into your HA `config/custom_components/` manually.)
2. Restart Home Assistant.
3. **Settings → Devices & Services → Add Integration → LifeCoach**, then enter
   your server URL and the pairing code from LifeCoach onboarding.
4. Open the integration's **Configure** to pick entities/cameras and, if wanted,
   turn on camera inference (after ticking “I live alone”).

## Development

The pure core (`events.py`, `cooldown.py`, `buffer.py`) has no Home Assistant
imports and is unit-tested without an HA install:

```bash
cd satellites/home
python3 -m pytest        # runs tests/ against the pure core
```

The HA-facing modules (`__init__.py`, `config_flow.py`, `coordinator.py`,
`api.py`, `queue.py`) are exercised in a real HA instance.
