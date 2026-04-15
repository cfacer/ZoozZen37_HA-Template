# Zooz ZEN37 800LR — Hold-to-Dim for Home Assistant

Smooth, proportional hold-to-dim automation for the
[Zooz ZEN37 800LR Wall Remote](https://www.support.getzooz.com/kb/article/1514-how-to-program-your-zen37-wall-remote-on-home-assistant/)
using [Z-Wave JS](https://www.home-assistant.io/integrations/zwave_js/) in Home Assistant.

| Button | Action | Result |
|--------|--------|--------|
| **B1** (top large) — hold | `KeyHeldDown` | Brightness ramps **up** |
| **B1** — release | `KeyReleased` | Brightness **freezes** |
| **B2** (middle large) — hold | `KeyHeldDown` | Brightness ramps **down** |
| **B2** — release | `KeyReleased` | Brightness **freezes** |

---

## How it works

The ZEN37 fires **one** `KeyHeldDown` event when you start holding and **one** `KeyReleased` when you let go — not a continuous stream.  This automation handles that gracefully:

1. **Hold detected** — `light.turn_on` is called with a proportional `transition` duration targeting `100 %` (B1) or `1 %` (B2).  The transition is proportional: if you're already at 50 %, reaching 100 % takes only half the configured time.
2. **`wait_for_trigger`** — the automation pauses, waiting for the matching release event.
3. **Release detected** — brightness at the moment of release is **calculated mathematically** via linear interpolation over elapsed time.  No bulb-state polling is required, so accuracy is independent of whether the bulb reports mid-transition brightness.
4. **Freeze** — `light.turn_on` is called at the computed value with `transition: 0.1` to halt the ramp smoothly.  If the downward ramp reached minimum, `light.turn_off` is called instead.

`mode: parallel` ensures B1 and B2 hold events are always handled independently with no queuing.

---

## Requirements

| Requirement | Version |
|-------------|---------|
| Home Assistant | **2024.1.0+** |
| Z-Wave JS integration | any current |
| Zooz ZEN37 800LR | firmware 10.20+ recommended |

---

## Option A — Blueprint (recommended)

Copy the blueprint file into your HA config directory and import it through the UI.

### 1. Install

```bash
# From your HA config directory:
mkdir -p blueprints/automation/zooz
cp blueprints/automation/zooz/zen37_800lr_dimmer.yaml  blueprints/automation/zooz/
```

Or use **Settings → Automations → Blueprints → Import Blueprint** and paste the raw GitHub URL:

```
https://raw.githubusercontent.com/cfacer/ZoozZen37_HA-Template/main/blueprints/automation/zooz/zen37_800lr_dimmer.yaml
```

### 2. Configure

Go to **Settings → Automations → Blueprints**, find *Zooz ZEN37 800LR — Hold-to-Dim*, click **Create Automation**, and fill in:

| Field | Description |
|-------|-------------|
| ZEN37 Remote | Select your ZEN37 device from the Z-Wave JS device list |
| Light | The light entity to control |
| Full-Range Transition Time | Seconds for 0 %→100 % (default: **5 s**) |

---

## Option B — Direct Automation

For paste-in use without the blueprint UI.

### 1. Find your ZEN37 device ID

1. Go to **Developer Tools → Events**.
2. Listen for: `zwave_js_value_notification`
3. Press any button on the ZEN37 remote.
4. Copy the `device_id` field from the event data.

### 2. Edit the automation

Open `automations/zen37_dimmer.yaml` and replace all occurrences of `YOUR_ZEN37_DEVICE_ID` with the value you copied.  Adjust `_light` and `_ttime` variables at the top if needed.

### 3. Import

Paste the YAML into **Settings → Automations → Create Automation → Edit in YAML**, or add it to your `automations.yaml` / `configuration.yaml`.

---

## Button map (ZEN37 scene numbers)

| Button | `property_key_name` | Physical location |
|--------|---------------------|--------------------|
| B1 | `"001"` | Large top button |
| B2 | `"002"` | Large middle button |
| B3 | `"003"` | Small bottom-left |
| B4 | `"004"` | Small bottom-right |

Key attribute values used:

| Value | Meaning |
|-------|---------|
| `KeyHeldDown` | Button is being held (fires **once** at hold start) |
| `KeyReleased` | Button was released (fires **once**) |
| `KeyPressed` | Single tap |
| `KeyPressed2x` – `KeyPressed5x` | Multi-tap |

---

## Tuning tips

- **Transition time** — default `5` seconds covers the full 0–100 % range.  Shorter values feel snappier; longer values are better for very gradual mood-lighting changes.
- **Minimum brightness** — the downward ramp targets `brightness_pct: 1` (not 0), so the bulb doesn't turn off mid-transition.  If `_freeze_pct` reaches `≤ 1` at release, `light.turn_off` is called instead of `light.turn_on`.
- **Multiple lights** — in the blueprint, point the *Light* selector at a light group entity to control several lights simultaneously.

---

## Acknowledgements

Inspired by:
- [irakhlin's ZEN37 800LR Blueprint Gist](https://gist.github.com/irakhlin/5b605729f55916830082888a464f20f4)
- [Zooz ZEN37 HA Programming Guide](https://www.support.getzooz.com/kb/article/1514-how-to-program-your-zen37-wall-remote-on-home-assistant/)
- [HA Community ZEN37 800LR thread](https://community.home-assistant.io/t/zwave-js-zooz-zen37-800lr-wall-remote/676731)

---

## License

MIT — see [LICENSE](LICENSE).  Personal, non-commercial use.
