---

name: arduino-uno-q-app-lab

description: Use when the user is building, modifying, scaffolding, or porting code targeting the Arduino UNO Q and the Arduino App Lab ecosystem — creating apps, creating or converting bricks, or integrating an Edge Impulse model. Covers the Linux MPU side, the STM32 MCU sketch, the RouterBridge IPC bridge between them, and the Edge Impulse Linux runtime (.eim).

---

# Arduino UNO Q + App Lab skill

The Arduino UNO Q is a hybrid board: a Qualcomm Dragonwing-class Linux MPU running Debian and an STM32 MCU on one PCB. The two halves talk through Arduino's **RouterBridge**. Software is built and deployed via **Arduino App Lab**, an ecosystem of **apps** (full programs) composed from **bricks** (reusable units — sensors, ML models, web UIs, GPIO, etc.).

The Arduino App Lab specification evolves quickly. Treat every manifest field, CLI flag, and folder name in this skill as a starting point, not as ground truth. The canonical docs win.


## Step 1 — Always fetch the canonical docs first

Before generating any manifest, file layout, or CLI invocation, fetch the live docs and reconcile your output against them. Use WebFetch (or `curl` / `gh`) to read at least:

**App Lab**

- Apps overview: https://docs.arduino.cc/software/app-lab/apps/about-apps
- Developing apps: https://docs.arduino.cc/software/app-lab/apps/develop-apps
- Bricks overview: https://docs.arduino.cc/software/app-lab/bricks/about-bricks
- Using bricks: https://docs.arduino.cc/software/app-lab/bricks/use-bricks/
- Examples: https://docs.arduino.cc/software/app-lab/getting-started/examples/


**UNO Q hardware / OS**

- User manual: https://docs.arduino.cc/tutorials/uno-q/user-manual/
- Debian guide: https://docs.arduino.cc/tutorials/uno-q/debian-guide
- Remote access: https://docs.arduino.cc/tutorials/uno-q/remote-access
- Security hardening: https://docs.arduino.cc/tutorials/uno-q/security-hardening-guide
- RouterBridge (MPU↔MCU IPC): https://docs.arduino.cc/tutorials/uno-q/routerbridge-multilanguage


**Edge Impulse**

- Run on App Lab: https://docs.edgeimpulse.com/hardware/deployments/run-arduino-app-lab
- UNO Q board page: https://docs.edgeimpulse.com/hardware/boards/arduino-uno-q
- Reference example app: https://github.com/edgeimpulse/example-arduino-app-lab-object-detection-using-flask


If a template in this skill disagrees with the current docs, **the docs win — update the template before generating output for the user.**
 

## Step 2 — Pick the workflow 

Match the user's request to one of these. Each workflow has a template directory and a reference doc.

| User wants… | Use template | Read reference |
|---|---|---|
| Create a new App Lab app from scratch | `templates/app/` | `references/app-structure.md` |
| Create a new brick or convert an existing module into one | `templates/brick/` | `references/brick-structure.md` |
| Add an Edge Impulse model to an existing app | `templates/edge-impulse-brick/` | `references/edge-impulse-integration.md` |
| Make the MCU sketch talk to the Linux side | (snippet in `references/routerbridge.md`) | `references/routerbridge.md` |
| Port an existing standalone Python/Flask app | combine `templates/app/` + brick refactor | `references/porting-checklist.md` |
 

Working principles for any workflow:


- **Don't invent manifest fields.** If the live docs only show `name`, `version`, `entry`, stop there. No drive-by `description`/`author`/`category`/`icon` additions.
- **Pin dependencies** in `requirements.txt`. The UNO Q runs Debian — prefer `apt`-available native libs (`python3-opencv`, `python3-numpy`) over building from source.
- **Push compute to Linux.** Keep the MCU sketch deterministic and small; do ML, web, networking, file I/O on the MPU side.
- **Linux APIs, not Arduino APIs, on the MPU side.** Camera → V4L2 / `/dev/video0`. I²C → `smbus2` or `/dev/i2c-*`. GPIO → `gpiod`. Reserve Arduino-style APIs (`digitalRead`, `analogRead`) for the sketch.
- **Edge Impulse: prefer the `.eim` Linux runtime** (no on-device compile) unless latency/memory forces the C++ SDK.
- **Never commit model binaries.** Provide `fetch_model.sh` or document the deploy step in the README.
- **Make bricks self-describing.** A brick should advertise its inputs and outputs in its manifest and README so other bricks can consume it without reading source.

 

## Step 3 — Scaffold

Copy the matching template directory into the user's project, then rename and fill placeholders. Replace tokens of the form `{{ ... }}` everywhere — they exist in manifests, code, and READMEs.

If the user is porting an existing app, *don't overwrite* — read the existing entry point and tests first, then propose a diff that adds the manifest and reorganizes files into the brick layout. Confirm before moving files. 


## Step 4 — Validate before declaring done

Run `python scripts/validate.py <path>` to lint the structure (checks manifest filename, required fields, entry-file presence, no committed model binaries). It's a coarse check, not a substitute for the docs.

Manual checklist:

1. Manifest parses; required fields match the live docs schema.
2. Entry runs locally without the board (mock RouterBridge / hardware).
3. RouterBridge calls have a dev-mode fallback if any.
4. No hardcoded laptop paths.
5. README documents: install on UNO Q, run, inputs/outputs of each brick, where the model comes from.
6. For Edge Impulse: licence/terms of the EI project noted; `.eim` not committed.


## What this skill is NOT

- Not a substitute for the App Lab CLI's own scaffolder. If the Arduino CLI provides `arduino-app-lab new app` (or similar), prefer that — then layer the EI / porting work on top.
- Not a way to bypass the docs. Always fetch them first.
- Not a hardware-test harness. Functional verification on real UNO Q hardware is the user's job.

