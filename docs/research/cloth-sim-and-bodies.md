# Cloth simulators and body models for the 3D preview

Research for the ticket "Cloth simulators and body models for the 3D preview". It feeds the decision ticket "Choose the 3D preview engine and body model".

- Licenses, prices and terms were checked on **2026-09-26**.
- Numbers marked **measured** were taken the same day on the owner's PC (WSL2 on Windows, GTX 1650 with 4 GB, 16 CPU threads). The appendix says how.
- Everything else is cited to its source. My own judgments are marked **assessment**.

## Answer in brief

**Facts**

- **GarmentCode's simulator is research-only.** It needs a fork of NVIDIA Warp that still carries NVIDIA's old license, which allows use "for research or evaluation purposes only" ([fork LICENSE](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/LICENSE.md)). GarmentCode's own code, including the step that turns a pattern into a simulation mesh, is MIT ([LICENSE](https://github.com/maria-korosteleva/GarmentCode/blob/main/LICENSE)).
- **Newton is the strongest free engine found.** It is Apache 2.0 and a Linux Foundation project ([README](https://github.com/newton-physics/newton)). It installed with `pip` and ran on the GTX 1650 under WSL2. Its Style3D cloth solver has separate stiffness along the warp, the weft and the bias, and handles cloth-on-cloth collisions. Measured: a women's skirt (36.5k triangles) on a female avatar ran at 5.6 simulated frames per second, about 11 s per simulated second, using about 0.3 GB of video memory.
- **Blender** (GPL, free, CPU only) sews flat pieces with "sewing springs", but uses one stiffness for all directions. Measured on the same skirt: 2.0 frames per second, about 2.8 times slower than Newton.
- **CLO and Style3D do not fit.** CLO costs $450 a year for an individual. Its terms limit API plug-ins to internal use and forbid access from other computers. Style3D's free version is for "personal learning and use" only, and its SDK price is not published.
- **No browser library sews pattern pieces onto a body.** WebGPU makes browser cloth possible, but this would be a build from scratch.
- **Body models:** the SMPL family is non-commercial; commercial licenses come from Max Planck Innovation at unpublished prices. Free and allowing commercial use: Anny, MakeHuman/MPFB, Meta's MHR, NVIDIA's SOMA, and MetaHuman, which bans use for AI training.
- **Bodies from a size chart need our own fitting code.** No free tool takes a size chart and returns a body. Measured: Anny's adult female bodies reach about 123 cm hip and 115-128 cm bust with its main controls. With MakeHuman's measurement controls at maximum, they reach about 142 cm hip and 145-157 cm bust. SOMA's random bodies spanned 82-128 cm hip (98% of samples, men and women mixed).

**Recommendation (assessment)**

- Simulate on the owner's PC with Newton, fed by GarmentCode's MIT pattern-to-mesh step, and show the result in the browser. Keep Blender as the fallback.
- Use Anny as the body model, with a small optimizer that fits it to each size of the size chart. In the preview prototype, compare its plus-size bodies with SOMA or MHR.

## Terms used here

- **Drape**: let the simulated fabric settle on the body under gravity.
- **Sewing in a simulator**: the flat pattern pieces start around the body, and the simulator pulls each **seam pair** closed. A seam pair is two piece edges that get sewn together.
- **Warp, weft and bias** (Portuguese: urdume, trama, viés): the two thread directions of a woven fabric, and the diagonal. A woven barely stretches along warp or weft but gives on the bias; a knit stretches a lot. A simulator needs separate stiffness per direction to tell them apart.
- **Body model**: a program that makes a body mesh from a few numbers. Its **shape space** is the range of bodies it can make. **Scan-based** models are learned from 3D scans; **artist-made** ones (MakeHuman, Anny) are sculpted.

## 1. Simulators

### 1.1 Comparison

| Option | License and cost (ADR-0001) | Sews 2D pieces from seam pairs? | Runs on | Speed | Woven vs knit |
|---|---|---|---|---|---|
| **GarmentCode + its Warp fork** | Fork: research or evaluation only. **Fails.** GarmentCode itself: MIT | Yes: joins the pieces into one "box mesh" by merging each seam pair's vertices | NVIDIA GPU; fork built from source with the CUDA Toolkit | 30 s per garment on an RTX 3090, about 30k triangles ([paper](https://arxiv.org/html/2405.17609)) | One stiffness for all directions; three bending levels |
| **Newton** (Style3D, VBD, XPBD solvers) | Apache 2.0, free. **Passes** | Partly: Style3D takes an assembled 3D garment plus the flat 2D pieces; VBD and XPBD take springs that can close seam pairs. No pattern importer | NVIDIA GPU (compute capability 5.0+), Linux or Windows; CPU fallback | Measured: 5.6 frames/s (skirt) and 3.9 (sweatshirt) on GPU; 0.23 on CPU | Style3D: separate warp, weft and bias stiffness for stretch and bending |
| **Blender cloth** | GPL, free; what you make is yours. **Passes** | Yes, with sewing springs | CPU only; headless | Measured: 2.0 frames/s (skirt) | One stiffness for all directions |
| **CLO** (desktop + API) | $450/year individual; plug-ins for internal use only. **Fails** for a web app used by others | Yes; imports DXF | Windows or macOS desktop | Not published | Separate weft, warp, bias stretch and bending |
| **Style3D** (Studio, SDK) | Free version: personal learning only; SDK price unpublished. **Fails** | Yes, in Studio | Desktop; SDK not checked | Not published | Not checked in Studio; its Newton solver has it |
| **Browser (WebGPU)** | MIT demos; no garment library | No | Viewer's GPU | Study: 10k cloth points over a 50k-triangle body at 30 frames/s on an RTX 4070 Ti ([paper](https://arxiv.org/html/2507.11794v1)) | Demos: one stiffness |
| Neural (HOOD, ContourCraft) | MIT code, but built on SMPL bodies. **Fails** | No | GPU | Not checked | Learned |

### 1.2 Notes per option

**GarmentCode.** GarmentCode tells users to install its "own version of the NVIDIA warp simulator" ([Installation](https://github.com/maria-korosteleva/GarmentCode/blob/main/docs/Installation.md)). The fork is "based on Warp v.1.0.0-beta.6". It adds the following ([fork README](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode)):

- self-collisions
- attachments, for example "fixing the skirt placement on the waist area"
- a drag that frees pieces stuck in the body
- body collision

Upstream Warp moved to a license allowing commercial use in 0.13.0 and to Apache 2.0 in 1.6.2 ([changelog](https://github.com/NVIDIA/warp/blob/main/CHANGELOG.md)). The fork did not. Its fabric is two single numbers, `tri_ke` and `tri_ka`, the same in every direction ([model.py](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/warp/sim/model.py)).

Only GarmentCode's simulation modules (`garment.py`, `simulation.py` and `datasim_utils.py`) use the fork. The box-mesh builder `boxmeshgen.py` does not ([pygarment/meshgen](https://github.com/maria-korosteleva/GarmentCode/tree/main/pygarment/meshgen)). It triangulates with the CGAL bindings, which are GPLv3+ ([PyPI](https://pypi.org/project/cgal/)).

The paper reports ([GarmentCodeData](https://arxiv.org/html/2405.17609)):

- garments of about 30k triangles at about 1 cm spacing, on a 47.5k-triangle body
- "30 seconds to simulate on average" on an RTX 3090, capped at 2,400 frames or 5 minutes
- about 72% of generated garments passing quality checks
- "three different textile materials", differing in bending only

The authors warn that Warp "includes only basic functionality, not specifically targeted for garment draping". Layers, sharp pleats, zippers and buttons are out of scope.

**Newton.** Newton "extends and generalizes Warp's (deprecated) `warp.sim` module". It was started by Disney Research, Google DeepMind and NVIDIA ([README](https://github.com/newton-physics/newton)). It needs an NVIDIA GPU of compute capability 5.0+ and driver 545+, but no CUDA Toolkit ([installation guide](https://newton-physics.github.io/newton/latest/guide/installation.html)); the GTX 1650 is 7.5. Version 1.0 shipped on 2026-04-13 and 1.6.0 on 2026-09-10 ([releases](https://github.com/newton-physics/newton/releases)).

- **Style3D solver**: a "projective dynamics based cloth solver" ([source](https://github.com/newton-physics/newton/blob/main/newton/_src/solvers/style3d/solver_style3d.py)), co-written by an engineer from Linctex, Style3D's company ([commits](https://github.com/newton-physics/newton/commits/main/newton/_src/solvers/style3d/solver_style3d.py)).
  - Its stiffness settings `tri_aniso_ke` and `edge_aniso_ke` hold separate stretch and bending values for warp, weft and bias.
  - It resolves cloth-on-cloth contact.
  - Its input is an assembled 3D garment plus "2D panel-space coordinates per vertex"; the flat pieces define the rest shape ([cloth.py](https://github.com/newton-physics/newton/blob/main/newton/_src/solvers/style3d/cloth.py)).
  - Its example garments (women's skirt, T-shirt and sweatshirt) and female avatar are Style3D Studio exports ([newton-assets](https://github.com/newton-physics/newton-assets/tree/main/style3d)).
- **VBD** ([Vertex Block Descent](https://arxiv.org/abs/2403.06321)) and **XPBD** simulate springs, so zero-length springs can close seam pairs, as in Blender.

**Assessment:** GarmentCode's box mesh already has the Style3D solver's input shape: pieces merged at the seams, with each piece's flat 2D coordinates kept. Joining the two looks natural but is unproven. Sewing from pieces that start apart is harder than the pre-assembled garments I timed.

**Blender.** Sewing springs "are created by adding extra edges to a cloth mesh that are not included in any faces", with a "Max Sewing Force" to keep the first frames stable ([manual](https://docs.blender.org/manual/en/latest/physics/cloth/settings/shape.html)). Stiffness is one value each for tension, compression, shear and bending, with no warp or weft ([manual](https://docs.blender.org/manual/en/latest/physics/cloth/settings/physical_properties.html)). Cloth is "CPU-intensive" ([manual](https://docs.blender.org/manual/en/latest/physics/cloth/introduction.html)). Blender 5.x adds an experimental XPBD "Cloth Dynamics" node that lacks self-collision ([manual](https://docs.blender.org/manual/en/latest/modeling/geometry_nodes/simulation/cloth_dynamics.html)). "What you create with Blender is your sole property", and scripts must be GPL-compatible only "if published" ([license](https://www.blender.org/about/license/)).

**CLO.** Individual plans cost "$50 USD" a month or "$450 USD" a year ([pricing FAQ](https://www.clo3d.com/en/pricing)). The Additional Terms, updated 2 Sep 2024 ([terms](https://legal.clo-set.com/additional-clo)), say:

- The API may be used only "to develop plug-ins ... for (i) your Internal Business Needs", which "in no event" include "providing or making available the Licensed Materials to any third party".
- A Standalone license runs "on a single computer ... which cannot be accessed from any other computer".

Plug-ins run inside the Windows or macOS desktop app. They cover simulation, auto-hanging and DXF import ([API docs](https://developer.clo3d.com/)). CLO fabrics have separate weft, warp, shear and bending settings ([stretch](https://support.clo3d.com/hc/en-us/articles/115000483087-Adjust-Stretch-Weft-Warp-Shear), [bending](https://support.clo3d.com/hc/en-us/articles/115000483187-Adjust-Bending-Weft-Warp)). A "Headless API" is documented ([docs](https://www.clo3d.com/en/docs/headless/restAPI)), but I found no public terms or price for it.

**Style3D.** "Free Version is only for your personal learning and use. Use of Free Version for commercial use or any other profit purposes is prohibited" ([Terms of Service, 5 June 2024](https://studio.style3d.com/docs/en/terms_of_service.pdf), 6.1). Style3D documents a "Simulator SDK" ([help center](https://help.style3d.com/simulator/en)) but publishes no price. A reference site outside Style3D's domain lists "SDK Service $32,000/yr" ([netlify page](https://style3d-dev-reference.netlify.app/)). That figure is **unverified**. The free route to Style3D's solver is Newton.

**Browser.** WebGPU shipped in Chrome 113 and Firefox 141 on Windows, and in Safari 26 ([MDN data](https://github.com/mdn/browser-compat-data/blob/main/api/GPU.json)). Building blocks that allow reuse:

- three.js (MIT) has a WebGPU "verlet system" cloth demo, a 30x30 grid over a sphere ([example](https://github.com/mrdoob/three.js/blob/dev/examples/webgpu_compute_cloth.html)).
- "Ten Minute Physics" has MIT cloth and self-collision demos ([file](https://github.com/matthias-research/pages/blob/master/tenMinutePhysics/15-selfCollision.html)).

The WebGPU simulator that GarmentCodeData cites has no license, so it cannot be reused ([repo](https://github.com/ccincotti3/webgpu_cloth_simulator)). The 2025 study in the table used simple springs and did not handle self-collision. **Assessment:** the GTX 1650 has a small fraction of an RTX 4070 Ti's compute, and a garment also needs sewing and fabric properties.

**Also checked.** HOOD and ContourCraft need SMPL body files ([HOOD](https://github.com/Dolorousrtur/HOOD), [ContourCraft](https://github.com/Dolorousrtur/ContourCraft)). Codim-IPC (Apache 2.0, [repo](https://github.com/ipc-sim/Codim-IPC)) targets accuracy over speed; I did not time it.

### 1.3 Measured on the owner's PC

Frames are simulated at 60 per simulated second, so 6 frames/s means 10 s of waiting per simulated second. The GPU was shared with the Windows desktop.

| Run | Garment | Result |
|---|---|---|
| Newton 1.6.0, Style3D solver, GPU | Women's sweatshirt: 24.5k vertices, 48.8k triangles (about 6.5 mm spacing), on a 53k-vertex avatar | **3.93 frames/s** (472 frames in 120 s) |
| Newton, Style3D solver, GPU | Women's skirt: 18.4k vertices, 36.5k triangles | **5.56 frames/s** (334 in 60 s) |
| Newton, Style3D solver, CPU only | Same skirt | **0.23 frames/s** (28 in 122 s) |
| Blender 5.1.1, CPU, no cloth-on-cloth collisions | Same skirt and avatar | **2.90 frames/s** (60 in 20.7 s) |
| Blender 5.1.1, CPU, with cloth-on-cloth collisions | Same skirt and avatar | **2.01 frames/s** (60 in 29.8 s); stable, vertices moved 22 mm median, 70 mm max |

- **Video memory:** GPU memory in use went from 1,677 to 1,977 MB, so the drape used about 0.3 GB. Windows alone held 1.7 to 2.6 GB of the 4 GB during the session.
- **First run:** compiling Newton's GPU code took about 40 s; later runs use a cache.
- **Caveat:** these garments come pre-assembled on the avatar, so sewing time is not included. Solver settings differ between engines, so treat the ratios as rough.

**Assessment:** GarmentCodeData's RTX 3090 has 10,496 CUDA cores; the GTX 1650 has 896 ([RTX 3090](https://www.nvidia.com/en-us/geforce/graphics-cards/30-series/rtx-3090-3090ti/), [GTX 1650](https://www.nvidia.com/en-ph/geforce/graphics-cards/gtx-1650/)). A full sew-and-drape at dataset resolution would take minutes here. A coarser preview mesh (1 to 1.5 cm spacing), restarted from the previous drape after a small change, could plausibly take tens of seconds. That needs a prototype.

## 2. Body models

### 2.1 Comparison

| Model | License: code / model data / exported meshes | Built from | Measurement controls | ADR-0001 |
|---|---|---|---|---|
| **SMPL, SMPL-X, STAR** (Max Planck) | Non-commercial only; bans "production of other artefacts for commercial purposes". Single fixed "SMPL-Body" meshes are CC BY 4.0 | Scans | None; fitting needs own code | **Fails** without a paid license (price unpublished) |
| **MakeHuman 1.x / MPFB2** | Code AGPL-3.0 / GPL-3.0; assets CC0; exports unrestricted | Artist-made | Bust, underbust, waist, hips, neck, arms, legs, shown in cm | **Passes** |
| **Anny** (NAVER LABS Europe) | Code Apache 2.0; MakeHuman assets CC0; optional "smplx" layout non-commercial (avoid) | Artist-made (MakeHuman assets) | 11 overall controls (gender, age, muscle, weight, height, proportions, cup size, firmness, 3 ancestry mixes) plus 254 local ones, including MakeHuman's measurement controls. Differentiable (PyTorch) | **Passes** |
| **MHR** (Meta) | Apache 2.0, model files included | 7,110 scans; 20 body shape numbers | None | **Passes** |
| **SOMA** (NVIDIA SOMA-X) | Apache 2.0 code and model files; its SMPL options need licensed files (avoid) | 9,326 SizeUSA scans, 303 other scans, GarmentMeasurements bodies; 128 shape numbers | None | **Passes** |
| **GarmentCodeData bodies** | Code GPL-3.0; 5,000 bodies with measurements, CC BY 4.0 | 1,700 European CAESAR scans | Measures bodies; cannot build one from measurements | **Passes on paper**; CAESAR terms not checked |
| **MetaHuman** (Epic) | Unreal Engine license: free under $1M revenue, royalty-free in any engine; no AI training | Scans | Creator can "adjust or constrain measurements" | **Passes with conditions**; needs Unreal Editor |
| GHUM (Google) | Academic request form only | Scans | - | **Fails** |
| CLO and Style3D avatars | Only inside the paid apps | - | Avatar editors | **Fails** for our use |

### 2.2 License details that matter

- **SMPL family.** Use is allowed only for "non-commercial scientific research, non-commercial education, or non-commercial artistic projects". "Any other use ... is prohibited", including "production of other artefacts for commercial purposes" ([SMPL](https://smpl.is.tue.mpg.de/modellicense.html); same for [SMPL-X](https://smpl-x.is.tue.mpg.de/modellicense.html) and [STAR](https://star.is.tue.mpg.de/license.html)).
  - "SMPL-Body", which "excludes the shape blendshapes", is CC BY 4.0 ([SMPL-Body](https://smpl.is.tue.mpg.de/bodylicense.html)). So a fixed average body can be used with credit, as GarmentCode does ([bodies Readme](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/Readme.md)), but new shapes cannot be made.
  - Epic Games bought Meshcapade, and Max Planck Innovation "will now directly take over the licensing of the SMPL technology" ([Max Planck, 18 Feb 2026](https://www.mpg.de/26082348/max-planck-spin-off-meshcapade-draws-epic-games-to-tuebingen)). Meshcapade's online platforms "have been shut down" since April 18 ([meshcapade.com](https://meshcapade.com/)).
- **MakeHuman and MPFB.** The assets (base mesh, targets, modifiers, textures, rigs and more) "have been released under CC0 1.0 Universal". The MakeHuman team "makes no claim whatsoever over output such as" file exports and renderings ([MPFB2 LICENSE](https://github.com/makehumancommunity/mpfb2/blob/master/LICENSE.md)). The MakeHuman 1.x app is AGPL ([LICENSE](https://github.com/makehumancommunity/makehuman/blob/master/LICENSE.md)). **Assessment:** use the CC0 assets through Anny rather than put AGPL code in a web service.
- **Anny.** The code is Apache 2.0, and its MakeHuman assets "are licensed under the CC0 1.0 Universal License" ([README](https://github.com/naver/anny)).
  - A "smplx" layout is "for non-commercial use only", and "the free install may download non-commercial only assets when needed". Pin the default layout and check what it downloads.
  - The paper warns that its controls "encode by design stereotypes of MakeHuman artists", and that "SMPL-X better captures fine skin-fold details of high-BMI bodies compared with Anny" ([paper](https://arxiv.org/html/2511.03589v3)).
- **MHR.** Apache 2.0 ([README](https://github.com/facebookresearch/MHR)). The `assets.zip` of release v1.0.1 contains an Apache 2.0 `LICENSE.txt` ([release](https://github.com/facebookresearch/MHR/releases/tag/v1.0.1)). The shape space comes from "7110 scans" ([paper](https://arxiv.org/html/2511.15586)).
- **SOMA.** Apache 2.0, though "optional third-party models and dependencies retain their own license terms" ([README](https://github.com/NVlabs/SOMA-X)). The Hugging Face files are tagged `apache-2.0` and not gated ([nvidia/SOMA-X](https://huggingface.co/nvidia/SOMA-X)). The shape space is "trained on 9,326 SizeUSA body scans, 303 Triplegangers photogrammetry scans, and samples distilled from the GarmentMeasurements PCA model" ([paper](https://arxiv.org/html/2603.16858)). I did not check the scan providers' terms. Version 0.3.1 needed two workarounds to run (appendix).
- **GarmentCodeData bodies.** They come from a PCA of "the European subset of the CAESAR database, consisting of 1700 3D scans", sampled at σ = 0.7. The authors note this subset "further worsens" CAESAR's bias ([paper](https://arxiv.org/html/2405.17609)). The dataset is CC BY 4.0 per ETH's record ([dataset](https://doi.org/10.3929/ethz-b-000690432)).
- **MetaHuman.** "MetaHumans can be used with any engine or creative software", with no royalties outside Unreal. But "you may not use MetaHuman characters ... to build or enhance any database or train or test artificial intelligence" ([license](https://www.metahuman.com/license)). Since version 5.6 its scan-based body can "adjust or constrain measurements for height, chest, waist, leg length, and many more parameters" ([5.6 news](https://www.metahuman.com/news/metahuman-leaves-early-access-with-a-feature-packed-new-release)).
- **GHUM** needs a request from an academic institution ([GHUM](https://github.com/google-research/google-research/tree/master/ghum)).

### 2.3 Building a body from size-chart measurements

- **MakeHuman / MPFB (by hand or scripted).** The Measure sliders cover neck, arms, bust, underbust, waist, hips, legs and more ([sliders](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/data/modifiers/measurement_sliders.json)). Each reads in cm along a fixed loop of mesh vertices ([Ruler](https://github.com/makehumancommunity/makehuman/blob/master/makehuman/plugins/0_modeling_a_measurement.py)). A script can adjust each slider until it matches the chart.
- **Anny (optimizer).** Anny has the same measurement controls and is differentiable, so a short gradient-descent fitter can match height, bust, underbust, waist and hip together. Its built-in measures are only height, waist, volume, mass and BMI ([anthropometry.py](https://github.com/naver/anny/blob/main/src/anny/anthropometry.py)). Bust and hip can reuse MakeHuman's vertex loops, as below. Measured: 25 ms per body on the CPU, after a one-time 27 s cache build.
- **MHR or SOMA (optimizer).** These have no measurement controls, so fitting also needs measurement loops defined on their meshes.
- **MetaHuman Creator (by hand).** I did not check whether it can be scripted.

### 2.4 How large can the bodies get? (measured)

I generated adult female Anny bodies (gender 1.0, age 0.67, other controls 0.5 unless stated). I measured them as a tape measure would: the outline, seen from above, of MakeHuman's bust, underbust, waist and hip loops. Values in cm.

| Anny setting | Bust | Underbust | Waist | Hip |
|---|---|---|---|---|
| weight 0 | 87.7 | 72.8 | 69.1 | 92.4 |
| weight 0.5 (default) | 90.2 | 74.7 | 74.7 | 99.5 |
| weight 1 | 95.5 | 78.9 | 83.0 | 107.8 |
| weight 1, muscle 0 | 115.1 | 105.0 | 114.5 | 123.2 |
| weight 1, muscle 0, cup size 1 | 127.5 | 106.0 | 114.5 | 123.2 |
| weight 1, muscle 0, bust/underbust/waist/hip controls at +1 | 145.2 | 135.4 | 128.3 | 141.9 |
| same, plus cup size 1 | 156.9 | 136.3 | 128.3 | 141.9 |

Height is separate; these bodies are 180 cm tall, so a real fit would lower it. **Assessment**, from front and side silhouettes: the bodies read as plausible regular and plus-size women. The heavy settings default to an apple shape with a full stomach, the "+1" body has lumpy transitions, and a pear shape needs the hip controls.

SOMA mixes men and women, so I measured 500 random bodies at one standard deviation and 500 at two, using horizontal slices of the T-posed mesh.

| SOMA random bodies | Bust | Waist | Hip | Height |
|---|---|---|---|---|
| 1 SD: 1st / median / 99th percentile | 75 / 104 / 133 | 53 / 85 / 114 | 82 / 103 / 128 | 148 / 171 / 192 |
| 2 SD: 99th percentile | 163 | 147 | 158 | 216 |

At two standard deviations some bodies are broken (waists near 11 cm, heights over 2.2 m). So random sampling cannot test the upper range fairly; a fit to the largest sizes would.

**Assessment:** these are the ranges the models reach, not what Brazilian sizes need. Compare them with the chart from the ticket "Brazilian women's size charts, regular and plus". None of the models found was built from Brazilian scans.

## 3. Where the simulation can run

- **Owner's PC, GPU.** Works under WSL2; Warp saw `"NVIDIA GeForce GTX 1650" (4 GiB, sm_75)` with no CUDA Toolkit installed. A drape needs about 0.3 GB, but Windows already holds 1.7 to 2.6 GB. A browser drawing the preview on the same GPU competes for the rest.
- **Server without a GPU.** Newton on CPU was 24 times slower (0.23 frames/s). Blender managed 2.0 frames/s on 16 threads. **Assessment:** free-tier servers have far fewer cores, so previews would take many minutes.
- **Browser.** WebGPU runs on the owner's GPU in Chrome or Edge on Windows. Sewing, self-collision and fabric properties would all be new code.

**Assessment:** the owner's PC with a GPU engine is the only free route that is fast enough. The browser should display the draped mesh (a few MB, for example glTF in three.js), not simulate it.

## 4. Recommendations (assessment)

1. **Simulator: Newton, plus our own glue between pattern and simulator.**
   - Reuse GarmentCode's pattern format and box-mesh builder (MIT).
   - Rebuild the simulator features GarmentCodeData needed (waistband attachment, freeing stuck pieces, body collision) without copying fork code. Alternatively, ask the fork's author, Maria Korosteleva, to relicense her changes.
   - Prefer the Style3D solver, which can tell woven from knit; VBD is second choice.
   - Replace the GPLv3 triangulation only if the software may ever be distributed. A hosted web app does not distribute it.
   - Prototype sewing from separated pieces first; it is the riskiest step.
2. **Fallback: Blender** from the command line or as a Python module. It sews and needs no GPU, but it is about 3 times slower here and has one stiffness for all directions.
3. **Avoid:**
   - GarmentCode's Warp fork.
   - SMPL and anything that needs it: HOOD, ContourCraft, Anny's "smplx" layout, and SOMA's SMPL options.
   - CLO and Style3D apps as engines.
4. **Body: Anny by default.** The license is clean, it has MakeHuman's measurement controls, and it is differentiable.
   - Write a fitter that builds one body per size from the size chart: bust, underbust, waist, hip and height first, then lengths.
   - In the prototype, compare its plus-size bodies with SOMA (mostly SizeUSA scans) and MHR.
   - Ask the pattern maker whether each size looks right.
   - MetaHuman is a manual fallback if realism matters more than automation, but it must never feed AI training.
5. **Browser simulation: not in v1.** Revisit it if the preview must work without the owner's PC.

## 5. Confidence and gaps

**Solid:**

- Each license and term was read from the LICENSE file, license page or terms document cited next to it.
- Timings and body ranges were measured on the owner's PC.

**Unverified or open:**

- Whether Newton can sew GarmentCode box meshes from separated pieces, reliably and fast enough. Only pre-assembled garments were timed.
- Prices: CLO Headless API terms and price; Style3D SDK and plan prices (only an unofficial page); SMPL commercial price.
- Scan terms: CAESAR's terms for derived bodies, and the scan providers' terms behind SOMA and MHR. We rely on each publisher's grant.
- Body realism: whether plus-size bodies look right for judging fit, and how far MHR and MetaHuman reach (not probed).
- Brazilian plus-size measurement targets, which the size-chart research will supply.

## Appendix: how the measurements were taken

- **Newton:** `pip install "newton[examples]"` (Newton 1.6.0, Warp 1.17.0), then `python -m newton.examples cloth_style3d --benchmark 120` (10 substeps, 4 iterations per frame). The skirt runs used the same example with `garment_usd_name = "Women_Skirt"`, `--benchmark 60`, and `--device cpu` for CPU. Memory was polled with `nvidia-smi`.
- **Blender:** the `bpy` 5.2.2 wheel lacked X11 libraries on this NixOS-WSL machine, so I ran the nixpkgs Blender 5.1.1 build headless. Same skirt and avatar, exported to OBJ. Settings: cloth quality 5, mass 0.15, collision distance 3 mm, self-collision distance 2 mm.
- **Anny:** Anny 0.6.0, CPU PyTorch, `topology="makehuman"` so MakeHuman's vertex loops apply. MakeHuman's own loop lengths ran 0 to 3 cm above the tape-measure values (8 cm for underbust at cup size 1).
- **SOMA:** py-soma-x 0.3.1, native `soma` shape model, CPU. Workarounds: pass `data_root` from `soma.assets.get_assets_dir()`, and switch off pose correctives, which failed to load under PyTorch 2.14. Slices keep the largest loop: bust = largest at 68-73.5% of height, waist = smallest at 58-66%, hip = largest at 46-56%.
