# AI routes from words and images to sewing patterns

Research for the ticket **AI routes from words and images to sewing patterns**, which feeds **Choose the pattern-generation route and its AI model**. Licenses, prices and terms were checked on 2026-09-26. Facts link to the source that owns them; my own judgments are labelled **Assessment**.

## Answer in brief

- **No AI route produces a production-ready pattern today.** Every pattern-generating model I could inspect stops at *simulation-ready* pieces: seam lines plus which edges are stitched together, enough to drape in a cloth simulator, with no seam allowance, notches, grainlines, closures or pockets and no knit-specific drafting. The mesh-first route stops earlier, at a 3D shape that still has to be cut into pieces.
- **The pattern-first research mostly sits on one base, GarmentCode.** ChatGarment, Design2GarmentCode, AIpparel, SewingLDM and NGL either train on GarmentCodeData or drive GarmentCode directly, so its design space and drafting quality are their ceiling. GarmentCode is MIT and GarmentCodeData is CC BY 4.0.
- **ADR-0001 rules out most trained garment models.** DressCode and SewFormer have no license at all, ChatGarment was trained on research-only SHHQ images, GarmageNet is CC BY-NC-ND. AIpparel and SewingLDM pass, but neither runs on the GTX 1650, AIpparel is English-only, and both stay inside GarmentCode's design space.
- **A general vision-language model already does the "words and images" part well.** NGL (2026) had off-the-shelf models fill a structured garment description that code turns into a GarmentCode pattern, with no training, and it beat ChatGarment and Design2GarmentCode even when they were also given GPT-5. The pattern to copy: the AI translates the description into a design spec; deterministic code drafts the pattern.
- **Mesh-first is not a realistic route to patterns.** The open generators need 6 to 24 GB of GPU memory, TRELLIS pulls in non-commercial renderers, Hunyuan3D's license excludes the EU, UK and South Korea, and Meshy and Tripo need paid plans for API or commercial use. The best flattening research gives very different pieces for small changes in the mesh, which defeats grading.
- **Grading only works when the output is a program.** Outputs that are GarmentCode, FreeSewing or Seamly2D programs can be re-drafted for every row of a size chart; raw piece geometry or a mesh cannot.
- **Recommendation (assessment):** choose "language model fills a design spec, our own parametric pattern engine drafts it". Reuse GarmentCode (MIT) for its piece, stitch and 3D-placement model, and FreeSewing (MIT) for its production markings and knit ease. Build the rest: production markings, closures, pockets, pleats, knit drafting, Brazilian size charts, grading with the same points in every size, and DXF-AAMA/ASTM export.

## Terms used here

- **Seam line and seam allowance**: the seam line is where the stitches go; the seam allowance is the strip of fabric (often 1 cm) outside it. Research code draws only seam lines, and calls pattern pieces *panels*.
- **Notch**: a small mark on a piece's edge showing which points of two pieces meet when sewn.
- **Grainline**: an arrow showing how to lay a piece along the fabric's threads.
- **Dart**: a stitched wedge that shapes flat fabric around the bust, waist or hips.
- **Ease**: room added over body measurements. **Negative ease**: the pattern is smaller than the body, as for stretchy knits.
- **Made-to-measure drafting**: drawing a pattern from one set of body measurements. Running it once per size-chart row is one way to grade; **Grading methods for regular and plus sizes, woven and knit** compares it with rule-based grading.

To say how close a route gets, this report uses four levels:

1. **3D look only**: a mesh; no pieces.
2. **Simulation-ready**: pieces plus stitch pairs; drapes in a simulator.
3. **Sewable**: a pattern maker could cut a sample after adding allowances and markings.
4. **Production-ready**: our bar; a graded, marked pattern file a sample is cut and sewn from with no corrections.

## Route 1: pattern-first

### 1.1 GarmentCode and GarmentCodeData, the shared base

Facts:

- [GarmentCode](https://github.com/maria-korosteleva/GarmentCode) (SIGGRAPH Asia 2023) is a Python library in which each garment is a program built from panels, edges and stitch interfaces. Code is [MIT](https://github.com/maria-korosteleva/GarmentCode/blob/main/LICENSE); last push 2025-06-29.
- **Design space** ([default.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/design_params/default.yaml)): tops (shirt, fitted shirt, optional strapless), seven neckline shapes, collar pieces (turtleneck, simple lapel, two-panel hood), sleeves (sleeveless, armhole shape, length, cuffs, ruffles), straight or fitted waistbands, and bottoms (circle, asymmetric circle, godet, basic, many-panel, pencil with slits, tiered skirts; pants). Dresses and jumpsuits join a top and a bottom. The garment programs contain darts and ruffles (gathers) but no pleat, pocket, button, zip or placket construct (search of [garment_programs](https://github.com/maria-korosteleva/GarmentCode/tree/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/garment_programs)). [GarmentCodeRC](https://github.com/biansy000/GarmentCodeRC) (MIT), ChatGarment's fork, adds open-front tops, tighter pants and high waists; the ChatGarment paper says it models pleats but "cannot model some specific details such as zippers and pockets" ([paper](https://arxiv.org/html/2412.17811)).
- **Output**: a JSON pattern specification (panels with 3D placement, plus stitches), SVG and PNG images, and a printable PDF of bare outlines with panel names ([wrappers.py](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/pygarment/pattern/wrappers.py#L53-L86)). No DXF, seam allowance, notches or grainlines.
- **The authors' own limits** ([paper](https://igl.ethz.ch/projects/garmentcode/GarmentCode-Programming-Parametric-Sewing-Patterns-SIGGRAPH-Asia-2023-Korosteleva-and-Sorkine-Hornung.pdf), sections 5.4 and 6): panels cannot have holes, pieces sewn on top (pockets, flounces) are not supported, sharp folds have "limited support" and pleats need "more tools", and cuffs and turtlenecks can only be one layer where real patterns fold a double piece. Reproducing Mood Fabrics' "Birch dress" production pattern, they got the overall shape but smoother sleeve curves, misaligned darts and extra seams. They also note that simulation patterns are more fragmented than fabrication patterns, such as a stitch down the center front instead of a piece cut on the fold.
- **Drafting inputs**: 26 body measurements, including uncommon ones such as shoulder and hip inclination, bust-point distance and arm pose angle ([mean_female.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/bodies/mean_female.yaml)). Ease is fixed in code, for example 2.5 cm at the armhole ([body_params.py](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/bodies/body_params.py#L39)) and 5 cm at the leg, commented "2 inch ease: from pattern making book" ([pants.py](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/garment_programs/pants.py#L190)). There is no fabric or stretch input.
- **Rules that change with the body**: the back bodice gets waist darts only when the waist is small enough relative to the panel ([bodice.py](https://github.com/maria-korosteleva/GarmentCode/blob/d449629979028123a5c4dc9e732a2ec19b7fce31/assets/garment_programs/bodice.py#L137)).
- **Bodies**: two average SMPL meshes, plus bodies from the authors' own model ([bodies readme](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/Readme.md)), released as [GarmentMeasurements](https://github.com/mbotsch/GarmentMeasurements) (GPL-3.0) and "based on CAESAR" per the dataset record.
- **Hardware**: drafting is plain Python and runs on the CPU ([installation](https://github.com/maria-korosteleva/GarmentCode/blob/main/docs/Installation.md)). The draping simulator is a fork of NVIDIA Warp under the NVIDIA Source Code License, "for research or evaluation purposes only" ([LICENSE.md](https://github.com/maria-korosteleva/NvidiaWarp-GarmentCode/blob/main/LICENSE.md)); upstream Warp became Apache-2.0 in version 1.6.2 on 2025-03-07 ([changelog](https://github.com/NVIDIA/warp/blob/main/CHANGELOG.md)).
- **GarmentCodeData** (ECCV 2024): 115,000 garments (tops, shirts, dresses, jumpsuits, skirts, pants) on varied body shapes, with three textile materials applied when draping ([DataCite record](https://api.datacite.org/dois/10.3929/ethz-b-000690432)). Both [v1](https://doi.org/10.3929/ethz-b-000673889) and [v2](https://doi.org/10.3929/ethz-b-000690432) carry "Creative Commons Attribution 4.0 International" in the ETH repository's own record ([v2 OAI record](https://www.research-collection.ethz.ch/oai/request?verb=GetRecord&metadataPrefix=oai_dc&identifier=oai:www.research-collection.ethz.ch:20.500.11850/690432)).
- **SMPL**: the average SMPL bodies bundled with GarmentCode and DressCode are fixed meshes ("SMPL-Body"), which are CC BY 4.0 ([SMPL-Body license](https://smpl.is.tue.mpg.de/bodylicense.html)); the SMPL shape model itself is non-commercial and forbids training for commercial use ([SMPL-Model license](https://smpl.is.tue.mpg.de/modellicense.html)).

**Assessment:** GarmentCode reaches level 2 and is the broadest commercially usable open design space with stitch and 3D information that I found. Re-drafting it per size is possible, but needs all 26 measurements for each Brazilian size (size charts list far fewer), and its body-dependent rules can add or drop darts between sizes, while a graded pattern file needs the same points in every size.

### 1.2 Models trained or prompted to output patterns

| Model | Output | Licenses: code / weights / data | ADR-0001 | Owner's PC? |
|---|---|---|---|---|
| [DressCode](https://github.com/IHe-KaiI/DressCode) (SIGGRAPH 2024) | piece geometry from English keywords | none / [none](https://huggingface.co/IHe-KaiI/DressCode) / [CC BY 4.0](https://doi.org/10.5281/zenodo.5267549) plus GPT-4V captions | fails: no license | not assessed; simulation needs Autodesk Maya |
| [SewFormer](https://github.com/sail-sg/sewformer) (SIGGRAPH Asia 2023) | piece geometry from one photo | none / [none](https://huggingface.co/liulj/sewformer) / [SewFactory](https://github.com/sail-sg/sewformer/blob/main/SewFactory/ReadMe.md), no license stated | fails: no license | not assessed; simulation needs Maya |
| [ChatGarment](https://github.com/biansy000/ChatGarment) (CVPR 2025) | GarmentCodeRC parameters from photo or text | Apache-2.0 / [unlicensed download](https://github.com/biansy000/ChatGarment/blob/main/docs/Installation.md) / [no license](https://huggingface.co/datasets/sy000/ChatGarmentDataset), includes SHHQ | fails: SHHQ | no: 7B model; text path calls GPT-4o |
| [Design2GarmentCode](https://github.com/Style3D/design2garmentcode-impl) (CVPR 2025) | GarmentCode parameters from text, photo or sketch | MIT / license not stated / CC BY 4.0 plus GPT-4V descriptions | code passes; weights unclear | partly: hosted GPT-4o plus a 2B model |
| [AIpparel](https://github.com/georgeNakayama/AIpparel-Code) (CVPR 2025) | piece geometry from text, photo or edit instruction | MIT / [MIT on a Llama 2 base](https://huggingface.co/georgeNakayama/AIpparel) / CC BY 4.0 plus GPT-4o captions | passes, with caveats | no: 27 GB checkpoint |
| [SewingLDM](https://github.com/shengqiliu1/SewingLDM) (arXiv 2024-25) | piece geometry from text, sketches and body shape | Apache-2.0 / [Apache-2.0](https://huggingface.co/liusq/sewingldm) / CC BY 4.0 | passes | no: 3.5 GB model plus 19 GB text encoder |
| [GarmageNet](https://github.com/Style3D/garmagenet-impl) (SIGGRAPH Asia 2025) | pieces and stitches from text or sketch | CC BY-NC-ND 4.0 / - / [CC BY-NC-ND 4.0](https://huggingface.co/datasets/Style3D/GarmageSet) | fails | not assessed |
| [GarmentDiffusion](https://github.com/Shenfu-Research/GarmentDiffusion) (IJCAI 2025) | piece geometry | repository holds only a project page | no code | - |
| [NGL](https://arxiv.org/abs/2602.20700) (arXiv 2026) | GarmentCode pattern via a general model's structured description | code promised "for research use", not found | idea reusable | tested with GPT-5 and Qwen models of 3B to 235B |

Notes, per model:

- **DressCode** turns keyword prompts such as "dress, sleeveless, midi length" into pieces, optionally with ChatGPT as a chat front end ([README](https://github.com/IHe-KaiI/DressCode)). It covers the 2021 dataset's garments: dresses, jumpsuits, pants, 2, 4 and 8-panel skirts, tees and jackets ([Zenodo file list](https://doi.org/10.5281/zenodo.5267549)). With no license on code or weights, there is no permission to reuse either.
- **SewFormer** has no license file anywhere in its repository ([repo](https://github.com/sail-sg/sewformer)). Its data generator needs Autodesk Maya 2020 with the Qualoth cloth plugin and the SMPL-H body, and simulating its predictions needs SMPL body estimates from a separate network ([SewFactory readme](https://github.com/sail-sg/sewformer/blob/main/SewFactory/ReadMe.md)).
- **ChatGarment** fine-tunes LLaVA-1.5-7B to emit a JSON of GarmentCodeRC choices plus 76 numbers, trained on 4 x H100 GPUs; the paper notes it cannot model zippers or pockets ([paper](https://arxiv.org/abs/2412.17811)). Its training data includes 38,000 SHHQ photos, and SHHQ is "available for non-commercial research purposes only" and forbids exploiting "any portion of the derived data for commercial purposes" ([SHHQ terms](https://github.com/stylegan-human/StyleGAN-Human/blob/main/docs/Dataset.md)).
- **Design2GarmentCode**: the released code asks GPT-4o (any OpenAI-compatible API) to pick options from GarmentCode's menu, then a fine-tuned [Qwen2-VL-2B](https://huggingface.co/Qwen/Qwen2-VL-2B-Instruct) (Apache-2.0, trained further on GarmentCodeData) predicts the numbers ([README](https://github.com/Style3D/design2garmentcode-impl)); its weights sit on Google Drive with no license stated. The paper says it "cannot substantially alter GarmentCode's underlying structure", and shows one new component (a layered skirt) written by a Llama-3.2-3B fine-tuned on 583 description-to-code pairs ([paper](https://arxiv.org/html/2412.08603v3)).
- **AIpparel** predicts pieces as tokens plus regressed vertices, 2.1 s per garment on a 48 GB A6000; pockets need structures its representation cannot hold ([paper](https://arxiv.org/abs/2412.03937)). Its MIT weights are a fine-tune of LLaVA-1.5-7B, which is under the Llama 2 Community License ([LLaVA card](https://huggingface.co/liuhaotian/llava-v1.5-7b)): commercial use is allowed below 700 million monthly users ([Llama 2 license](https://github.com/meta-llama/llama/blob/main/LICENSE)), and LLaVA asks users to also respect OpenAI's terms for its GPT-written training data ([LLaVA README](https://github.com/haotian-liu/LLaVA)).
- **SewingLDM** conditions on text, front and back line sketches and a body-shape vector from GarmentCodeData's body model ([paper](https://arxiv.org/abs/2412.14453)). Its text encoder is T5-XXL from [PixArt-Sigma](https://huggingface.co/PixArt-alpha/pixart_sigma_sdxlvae_T5_diffusers) (tagged OpenRAIL, 19 GB in fp32).
- **GarmageNet** is the only model trained on professionally designed garments (14,801 in GarmageSet), but both code and data are CC BY-NC-ND 4.0 ([LICENSE](https://github.com/Style3D/garmagenet-impl/blob/main/LICENSE)).
- **NGL** asks GPT-5, Qwen-3-VL or Qwen-2.5-VL to describe a photographed garment in a structured "Natural Garment Language", which a parser turns into GarmentCode. With no training it beat ChatGarment and Design2GarmentCode on two benchmarks, even with both baselines also running on GPT-5, but the authors note it "cannot reconstruct sewing patterns outside" its predefined parameter space ([paper](https://arxiv.org/html/2602.20700)).
- The 2026 preprint [GarmentWeaver](https://arxiv.org/abs/2608.30550) (a vision-language model producing structured patterns) has no released code that I could find.

What they share (GarmageNet's output not inspected):

- **Markings**: none outputs seam allowance, notches or grainlines; the output is seam-line pieces plus stitch pairs (level 2).
- **Woven versus knit**: none drafts differently for knit. ChatGarment maps inferred materials only to simulator settings ([paper](https://arxiv.org/html/2412.17811v2)); GarmentCodeData varies materials only in simulation.
- **Language**: AIpparel's card lists English only and DressCode takes English keywords; the models that call GPT-4o or a general model inherit that model's language coverage.
- **Grading, assessment**: parameter outputs (ChatGarment, Design2GarmentCode, NGL) can be re-drafted per size through GarmentCode. Geometry outputs (DressCode, SewFormer, AIpparel, SewingLDM, GarmageNet) come in one size, with no named points to attach grade rules to.
- **Hardware, assessment**: nothing here runs on a 4 GB GTX 1650 as released; the 7B models would need heavy quantization or a slow CPU run, and the AIpparel checkpoint alone exceeds the owner's 19 GB of RAM before conversion (untested).

### 1.3 A general language model filling or writing parametric pattern code

Three open targets exist for a language model to fill or extend:

| Target | What the language model would write | Production markings built in | Grading | Export | License |
|---|---|---|---|---|---|
| GarmentCode | design parameters (YAML or JSON), or new Python components | none | re-draft per body from 26 measurements | JSON, SVG, PDF | MIT |
| [FreeSewing](https://codeberg.org/freesewing/freesewing) | option values for an existing design, or a new JavaScript design | [seam allowance, grainline, cut-on-fold, title, pleat, hem](https://freesewing.dev/reference/macros); [notches, buttons, buttonholes, snaps](https://freesewing.dev/reference/snippets) | drafted from measurements per person or size | [PDF and SVG](https://freesewing.eu/docs/about/site/draft/); its DXF-ASTM plugin is [deprecated](https://www.npmjs.com/package/@freesewing/plugin-export-dxf) | MIT |
| [Seamly2D](https://github.com/FashionFreedom/Seamly2D) (Valentina is its sibling) | XML pattern files whose points are formulas of measurements | [seam allowance, notches, grainline, piece labels](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vtools/dialogs/tools/piece/pattern_piece_dialog.ui) | [multisize measurement files](https://github.com/FashionFreedom/Seamly2D#readme), size picked at export | [DXF-AAMA, DXF, PDF, tiled PDF, SVG](https://manpages.debian.org/experimental/seamly2d/seamly2d.1.en.html) | GPL-3.0 |

- FreeSewing moved from GitHub ([archived notice](https://github.com/freesewing/freesewing)) to Codeberg, where it is MIT and had commits on 2026-09-26. Its womenswear-relevant designs ([designs folder](https://codeberg.org/freesewing/freesewing/src/branch/develop/designs)) include bodice blocks (Bella, Breanna, Noble), a knit top block (Bibi), a skirt block "based on Aldrich" (Sarah), fitted, wrap-style and button-down tops, a tee, pencil and circle skirts, a skater dress "with princess seams and pockets" (Sasha), a slip dress (Sophie), a trouser block, chinos, wrap pants and leggings. Knit handling is per design: the Lumira leggings default to -8% ease ([shape.mjs](https://codeberg.org/freesewing/freesewing/src/branch/develop/designs/lumira/src/shape.mjs)).
- **Evidence**: Design2GarmentCode and NGL (in 1.2) show that language models can fill GarmentCode's parameters. I found no published evaluation for FreeSewing or Seamly2D; an unlicensed agent tool driving Valentina and GarmentCode appeared in August 2026 ([garment-cad-mcp](https://github.com/0xZoharHuang/garment-cad-mcp)).
- **Assessment**: having the model fill a validated design spec that our engine drafts is reliable and testable. Having it write new drafting code on each request is not, because every new program would need a pattern maker's check. New details should enter the engine through us, not through the model at run time.

## Route 2: mesh-first

### 2.1 Text and image to 3D generators

| Generator | Input | License (ADR-0001) | Needs | Notes |
|---|---|---|---|---|
| [TRELLIS](https://github.com/microsoft/TRELLIS) | text or image; the README advises text to image first | code and weights MIT, but default submodules [diffoctreerast](https://github.com/JeffreyXiang/diffoctreerast/blob/master/LICENSE) (Inria and MPII, research only) and [nvdiffrast](https://github.com/NVlabs/nvdiffrast/blob/main/LICENSE.txt) (NVIDIA, research or evaluation only) | NVIDIA GPU with 16 GB+, Linux | meshes extracted with FlexiCubes (Apache-2.0) |
| [TRELLIS.2](https://github.com/microsoft/TRELLIS.2) | image | MIT, with nvdiffrast and nvdiffrec under their own licenses | 24 GB+ | advertises open surfaces "e.g., clothing" |
| [Hunyuan3D-2](https://github.com/Tencent-Hunyuan/Hunyuan3D-2) and [2.1](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1) | image | [community license](https://github.com/Tencent-Hunyuan/Hunyuan3D-2.1/blob/main/LICENSE): not valid in the EU, UK or South Korea; over 1 million monthly users needs Tencent's permission; outputs may not improve other AI models | 2.0: 6 GB for shape; 2.1: 10 GB for shape, 21 GB for texture | shape 0.6B "mini" variant exists, memory not stated |
| [TripoSG](https://github.com/VAST-AI-Research/TripoSG) | image | MIT | 8 GB+ | trained on image and signed-distance pairs |
| [Meshy](https://www.meshy.ai/pricing) | text or image, hosted | Free: 100 credits a month, outputs CC BY 4.0; Pro: 1,000 credits a month, API access, you own outputs | API keys need a paid plan ([docs](https://docs.meshy.ai/en/api/quick-start)); text-to-3D preview 20 credits, image-to-3D 30 ([API pricing](https://docs.meshy.ai/en/api/pricing)) | exports FBX, GLB, OBJ, USDZ, STL; its [clothing page](https://www.meshy.ai/use-cases/clothing-design) promises no patterns |
| [Tripo](https://www.tripo3d.ai/pricing) | text or image, hosted | Free: 200 credits a month, "Public Models, Non-Commercial Use"; Pro: US$20 a month billed yearly, commercial use | separate pay-as-you-go API at US$0.01 per credit: text-to-3D 10 to 20 credits, image-to-3D 20 to 30 ([API pricing](https://developers.tripo3d.ai/en/pricing)) | its [MetaTailor tie-in](https://www.tripo3d.ai/blog/metatailor-x-tripo) fits meshes onto game characters, not patterns |

By their own stated requirements, none of the open generators fits the owner's 4 GB GPU. TRELLIS and TRELLIS.2 also default to FlashAttention, which needs Ampere or newer ([README](https://github.com/Dao-AILab/flash-attention)), while the GTX 1650 is a Turing card (compute capability 7.5, as NVIDIA lists for the GTX 1650 Ti, [CUDA GPUs](https://developer.nvidia.com/cuda-gpus)).

### 2.2 Flattening a mesh into pattern pieces

- **Pietroni et al. 2022**, "Computational Pattern Making from 3D Garment Models" ([paper](https://arxiv.org/abs/2202.10272)), is the strongest method found. It cuts a garment mesh into pieces with darts, grain alignment, mirror-symmetric seams and a woven-fabric stretch model, and the authors sewed a neoprene wetsuit and spandex leggings with "an excellent fit". Its stated limitations: wrinkles in simulated or scanned meshes add noise, "a slight difference in the input garment or user constraints can result in a significantly different pattern", and seam allowance is future work. Its code states licenses only in READMEs, with no LICENSE file: GPL-3 for [parafashion](https://github.com/nicopietroni/parafashion), and an MIT text with the name left blank for [garment-flattening](https://github.com/CorentinDumery/garment-flattening), which also notes that generic UV unwrapping is "unfit for anisotropic materials" such as woven fabric.
- **[NeuralTailor](https://github.com/maria-korosteleva/Garment-Pattern-Estimation)** (MIT, SIGGRAPH 2022) recovers piece structure from 3D point clouds, trained only on the 2021 synthetic dataset.
- **[Garment3DGen](https://github.com/nsarafianos/Garment3DGen)** builds simulation-ready garment meshes from images, but is [CC BY-NC 4.0](https://github.com/nsarafianos/Garment3DGen/blob/main/LICENSE.md).
- **[Stitched Embeddings](https://arxiv.org/abs/2607.00829)** (July 2026) claims pattern recovery from meshes by linking 3D garments and 2D patterns in one learned space. Its [code](https://github.com/Andreus00/StitchedEmbeddings), published 2026-09-17, has no license file, is trained on GarmentCodeData and bundles Hunyuan3D-2.1 code.

### 2.3 Why the route stalls (assessment)

- Generators that extract iso-surfaces or learn signed distances (TRELLIS uses FlexiCubes; TripoSG trains on signed-distance data) return closed shells, while a garment is an open, thin sheet with neck, arm and waist openings. TRELLIS.2 advertises open surfaces "e.g., clothing" as a break from iso-surface methods, and it needs 24 GB.
- The mesh has no size: nothing ties it to a body of known measurements, so fit is unknown, and producing other sizes means flattening each size separately, which Pietroni's own limitation says is unstable.
- Closures, pockets, plackets and waistbands come out as surface bumps or texture, not as separate pieces.
- Net: level 1 from the generators, level 3 at best after manual mesh clean-up and a flattening step with no license-clean, maintained code, and no path to grading.

## Side by side

| Criterion | Trained garment models | Language model plus parametric engine | Mesh-first |
|---|---|---|---|
| Families and details | GarmentCode's space: tops, skirts, pants, dresses, jumpsuits; necklines, collars, hoods, sleeves, cuffs, darts, ruffles, slits; no closures or pockets | whatever the engine drafts; FreeSewing already marks buttons, buttonholes, snaps and pleats and has a dress with pockets | any shape the generator draws; details are not pieces |
| Woven versus knit | none; simulation material only | FreeSewing designs use negative ease; Seamly2D by formula; GarmentCode none | Pietroni models woven stretch; generators none |
| Output quality | level 2 | GarmentCode level 2; FreeSewing and Seamly2D reach level 3 with markings | level 1; level 3 at best after flattening |
| Can be graded | parameter outputs yes, geometry outputs no | yes, by re-drafting per size (Seamly2D has multisize files) | no |
| Licenses (ADR-0001) | most fail; AIpparel and SewingLDM pass with caveats | GarmentCode MIT, FreeSewing MIT, Seamly2D GPL-3.0; the language model is for the budget research | TRELLIS non-commercial submodules; Hunyuan3D territory limits; Meshy and Tripo need paid plans for API or commercial use |
| Hardware | none fits the GTX 1650 | engine on the CPU; the language model hosted or small (budget research) | 6 to 24 GB GPU, or a paid API |

## Is any route production-ready today?

No. **Assessment:** the best AI output is level 2, from the GarmentCode-based routes; level 3 exists only in hand-driven tools (FreeSewing, Seamly2D); nothing reaches level 4. The pattern-first models were built to feed simulators and datasets: none reports sewing a sample from its output (GarmentCode compared one design with a commercial pattern; AIpparel lists fabrication as future work), and none carries the markings, grading and file formats a cutting room needs. The realistic path is one where the AI never draws geometry: it turns words and images into a design spec, and code we control drafts, grades, marks and exports the pattern.

## The gap our own code must fill

1. **A design spec** (garment family, details, woven or knit and stretch, fit and ease, lengths) that the language model fills and edits from Portuguese or English words and images, validated against allowed values, as NGL does.
2. **Drafting rules** per family and detail, including what GarmentCode lacks: closures and plackets, pockets, pleats, facings and hem finishes, reviewed by the pattern maker.
3. **Knit drafting**: negative ease set from the fabric's stretch, and knit blocks separate from woven ones.
4. **Size charts**: every measurement the engine needs, for every Brazilian regular and plus size, deriving the ones the chart lacks.
5. **Grading**: every size drafted with the same set of points (no rules that add or drop darts between sizes), plus plus-size-specific rules.
6. **Production markings**: seam allowance per seam with proper corners, notches, grainlines, cut-on-fold, drill holes at dart points, and piece labels with name, size and cut count.
7. **Export** to DXF-AAMA/ASTM with grading, and PLT or PDF for plotting.
8. **Automated geometry checks**: stitched edges of equal length (or with intended ease), closed outlines, matching notch pairs, mirror-symmetric pieces.
9. **Stitch and 3D placement data** for the 3D preview, which a GarmentCode-based engine already has.

**Assessment on the engine base:** fork GarmentCode for its representation (panels, edges, stitch interfaces, 3D placement) and component system, and bring in FreeSewing's marking and ease conventions. Whether to keep GarmentCode's drafting rules or redraft each garment family with the pattern maker is for the route decision; I lean toward redrafting, because GarmentCode's rules were written for simulation and some change the piece structure between sizes.

## For other tickets

- **Choose the pattern-generation route and its AI model**: the choice becomes which general language model fills the design spec, and which base the pattern engine starts from, rather than which AI model generates patterns.
- **Cloth simulators and body models for the 3D preview**: GarmentCode's Warp fork is non-commercial, while upstream Warp is Apache-2.0 since 1.6.2; fixed SMPL meshes are CC BY 4.0 but the SMPL shape model is not; GarmentCode's own body model is GPL-3.0 and CAESAR-based.
- **AI models that fit a near-zero budget**: no garment-specific fine-tuned model is needed; the requirement is a model that reads images and returns schema-valid JSON in Portuguese and English.
- **Garment details v1 must support**: GarmentCode covers necklines, collars, hoods, sleeves, cuffs, darts, ruffles, slits and waistbands; it lacks closures and pockets, and its authors say pleats need more tools.
- **Grading methods for regular and plus sizes, woven and knit** and **Pattern file formats for apparel CAD and plotting**: FreeSewing exports only PDF and SVG; Seamly2D exports DXF-AAMA and reads multisize measurement files.

## Not verified

- Whether AIpparel, SewingLDM or ChatGarment could run quantized on 4 GB or on the CPU within 19 GB of RAM (not tested).
- Meshy Pro's list price: the pricing cards render by script; the page's referral text values one Pro month at US$20.
- Tripo API: free credits for new accounts (only secondary sources mention 300 credits valid two weeks) and the commercial terms for API outputs.
- Whether TRELLIS can produce meshes without its non-commercial submodules, and the memory needs of Hunyuan3D-2mini.
- The license of Design2GarmentCode's and ChatGarment's weight files, which none of the pages state.
- Whether the CAESAR-based body model had the rights needed for its GPL-3.0 release.
- Accuracy benchmarks: this report compares coverage, licenses and hardware, not measured pattern accuracy.
