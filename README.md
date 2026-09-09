# Atlas spike

Answers one question before Phase 4 is built: can a phone hold the real anatomy? Two data sets
share one page.

```
node fetch.mjs     # version 1: BodyParts3D via human-atlas, ~26 MB into ./models (git-ignored)
node server.js     # http://localhost:8102, and http://<LAN ip>:8102 from a phone
```

| URL | Data | What it has |
|---|---|---|
| `/` | BodyParts3D 4.0 (human-atlas chunks) | All 2,234 parts: 296 bones, 402 muscles, 639 arteries, 404 veins, 139 nerves, the organs. No joint ligaments, no lat, no rectus abdominis. |
| `/?atlas=za` (menu) | Z-Anatomy, full resolution | 1291 structures: bones, muscles, ligaments, menisci, discs, capsules, bursae. |
| `/?atlas=za-lite` (menu) | Z-Anatomy, decimated | Same structures, muscles and bones reduced for the phone budget. |

Below the set switch, eight layer toggles hide or show bones, muscles, joints and ligaments,
fascia, arteries, veins, nerves and organs without reloading. The last four only ever light up on
the BodyParts3D set: Z-Anatomy’s whole cardiovascular collection is 22 meshes, seventeen of them
the heart, so it has no vascular tree to show (measured 2026-09-09). Each layer has its own colour
— red arteries, blue veins, yellow nerves — because in one clay colour they are indistinguishable: a per-vertex system slot and a `discard` in the fragment shader. Picking
skips hidden layers, so muscles off means a tap on the thigh names the femur. Fascia starts hidden;
the 82 fasciae wrap the muscles like a skin.

The "Myofascial lines" menu highlights one of Thomas Myers' lines (superficial back and front,
lateral, spiral, functional, deep front, arm lines front and back) in blue and dims the rest. The
lines are name-pattern lists in `index.html`, so they work on both data sets; bursae and tendon
sheaths are excluded. The pick bar names the lines a tapped structure belongs to. Z-Anatomy has no
patellar ligament body, only its attachment footprints in the "Muscular insertions" collection,
which the export does not take.

Drag to orbit, pinch or scroll to zoom, tap a structure to select it. The HUD reports triangles,
buffer size, draw calls and heap; the bar at the bottom names the structure and times the pick.

## Building the Z-Anatomy sets

Blender is not installed on the machine; the portable zip works, unpacked into `zanatomy/`
(git-ignored, ~1 GB with the source file):

```
curl -L -o zanatomy/Z-Anatomy.zip https://raw.githubusercontent.com/Z-Anatomy/Models-of-human-anatomy/master/Z-Anatomy.zip
curl -L -o zanatomy/TA2.csv     https://raw.githubusercontent.com/Z-Anatomy/Models-of-human-anatomy/master/TA2.csv
unzip zanatomy/Z-Anatomy.zip -d zanatomy          # -> zanatomy/Z-Anatomy/Startup.blend, 307 MB
curl -L -o blender.zip https://download.blender.org/release/Blender5.2/blender-5.2.1-windows-x64.zip
unzip blender.zip -d zanatomy && rm blender.zip
B=zanatomy/blender-5.2.1-windows-x64/blender.exe
$B -b zanatomy/Z-Anatomy/Startup.blend --python export-zanatomy.py -- --inspect
$B -b zanatomy/Z-Anatomy/Startup.blend --python export-zanatomy.py -- --out za                                  # 20 s
$B -b zanatomy/Z-Anatomy/Startup.blend --python export-zanatomy.py -- --out za-lite --decimate 0.35 --min-tris 3000   # 2 min
```

`export-zanatomy.py` takes the meshes of the "Skeletal system", "Joints" and "Muscular system"
collections, drops the helpers (labels `.t`, groups `.g`, joint markers `.j`, the muscular
insertion surfaces), evaluates modifiers (Z-Anatomy leans on subdivision and solidify), converts
Blender's Z-up metres to the Y-up frame of the first set with the feet on y=0, and writes the same
`atlas.json` + `models/body-N.bin.gz` layout the page already reads. Every part records its
Z-Anatomy name, side, TA2 id where the English name matches `TA2.csv` (1188 of 1373), and a system:
`skeletal`, `muscular`, `articular` (ligaments, cartilage, menisci, discs, capsules, bursae, fat
pads) or `fascial`.

Not app code. It is the throwaway that proved the approach, kept because the numbers in
`../../docs/MERGE-PLAN.md` §9.4 came from it and someone will want to re-measure on a newer phone.

## Licences

- Version 1: BodyParts3D 4.0, CC BY 4.0, The Database Center for Life Science. Loader approach
  from ashemag/human-atlas (MIT).
- Z-Anatomy sets: "Z-Anatomy - The libre 3D atlas of anatomy - CC-BY-SA 4.0", derived from
  "BodyParts3D - The Database Center for Life Science". ShareAlike: the exported geometry and
  `atlas.json` stay under CC BY-SA 4.0 wherever they ship, with both attributions. Z-Anatomy also
  bundles a kidney and an inner ear under non-commercial licences; they are not in the
  musculoskeletal collections, so the export never touches them.
