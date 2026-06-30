# Campus Delivery Robot — Project Notes (v1)

**Reman Dey | First-Year, Engineering Physics | IIT Mandi | June 2026**

---

## What this is

A low-cost skid-steer delivery robot for fixed campus routes, built around parts the
Robotronics Club already has or that are cheap to buy. This is a first-year v1 build
— the goal is to get something that actually moves and follows ArUco markers, not to
build a perfect system.

Full write-up is in `iit_mandi_evaluation.pdf` / `iit_mandi_evaluation.tex`.

---

## Hardware at a Glance

| Component | What I'm using | Why |
|---|---|---|
| Motor driver | Cytron iMD16 (×1) | Club already has it; handles 12V, 16A cont. |
| Compute | Raspberry Pi 4 (4 GB) | Cheap, can run OpenCV + Python fine |
| Camera | USB webcam (720p) | All I need for ArUco detection |
| Motors | 12V geared DC (×4) | Paired left/right into the two iMD16 channels |
| Battery | 12V 7Ah lead-acid | Cheap, club may already have a spare |
| Localisation | Printed ArUco markers | No GPS needed on a fixed route |
| Chassis | Laser-cut plywood/acrylic | Fab Lab if free for club use |

**Estimated total cost: ~Rs. 17,650** (including 40% contingency for mistakes)  
If the Pi and iMD16 come from club stock, out-of-pocket drops to ~Rs. 7,000–8,000.

---

## How the motor control works (short version)

The iMD16 takes a PWM signal + a direction pin per channel. The Pi's 3.3V GPIO works
directly — no level shifter needed since the iMD16 accepts 3–30V logic.

Left-side motors are wired together into CH1, right-side into CH2.

Given a desired forward speed `v` (m/s) and turn rate `omega` (rad/s):

```
v_R = v + b * omega      # right side
v_L = v - b * omega      # left side
```

where `b` = half the track width (metres). Convert to duty cycle by dividing by max
wheel speed. That's basically the whole control loop.

---

## Software plan

Plain Python + OpenCV, no ROS 2 (yet). Four simple nodes:

```
webcam_node   →  grabs frames from the USB webcam
aruco_node    →  detects DICT_6X6_250 markers, estimates rough pose
motor_node    →  translates (v, omega) command to PWM on the iMD16
main_loop     →  if marker visible → steer toward it
                 if no marker → creep forward slowly and look
```

Will move to ROS 2 in a v2 once I'm more comfortable with the codebase.

---

## File list

```
iit_mandi_evaluation.pdf   ← Full proposal (compiled, read this)
iit_mandi_evaluation.tex   ← LaTeX source
abc.png               ← H-bridge figure
README.md                  ← This file
```

> **Note:** `h_bridge.png` is a placeholder. Swap in a real H-bridge diagram before
> final submission.

---

## What I cut from the original (expensive) version

| Dropped | Replaced with | Saving |
|---|---|---|
| NVIDIA Jetson Orin Nano (Rs. 28,000) | Raspberry Pi 4 (Rs. 5,500) | ~Rs. 22,500 |
| Luxonis OAK-D Pro (Rs. 32,000) | USB webcam (Rs. 800) | ~Rs. 31,200 |
| u-blox ZED-F9P GPS + RTK (Rs. 16,000) | Nothing (not needed) | ~Rs. 16,000 |
| 4× iMD16 drivers (Rs. 12,800) | 1× iMD16 reused from club | ~Rs. 12,800 |
| 24V 30Ah LiFePO₄ (Rs. 22,000) | 12V 7Ah lead-acid (Rs. 1,400) | ~Rs. 20,600 |

Original estimate: ~Rs. 1,75,000 → Revised: ~Rs. 17,650

---

## Stuff I'm planning to add later (not in v1)

- Wireless kill switch / proper e-stop
- Per-wheel current sensing (ACS712 module, ~Rs. 150 each)
- ROS 2 migration
- Better battery (LiFePO₄) once the chassis design is proven
- Maybe a second iMD16 for independent per-wheel control

---

*Submitted to Robotronics Club for review — feedback welcome.*
