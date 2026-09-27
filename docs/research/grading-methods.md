# Grading methods for regular and plus sizes, woven and knit

Research for the ticket [Grading methods for regular and plus sizes, woven and knit](https://github.com/GustavoTrielli/smart-modeling/issues/6) on the map "Map: smart-modeling v1 spec". It feeds the decisions "Choose the grading approach" and "Define the production-ready acceptance test".

- Researched on 2026-09-26. Licenses, prices and terms were checked on 2026-09-26.
- Facts carry an inline link to the source that owns them. Paragraphs marked **Assessment** are my judgment.
- Terms follow `CONTEXT.md`. Pattern-making terms that are not in the glossary yet are explained in section 1.

## Answer in brief

- **Two ways to make every size.** *Rule-based grading* moves chosen points of one approved base size by stored X/Y amounts per size. *Drafting each size* recomputes the whole pattern from each size's row in the size chart. Gerber, Audaces and CLO grade by rules; Grafis and all the open tools reviewed here draft each size.
- **Traditional rules do not follow bodies.** None of the seven assumptions behind bodice grade rules from 17 sources held up against body measurement data.
- **Plus is its own size range.** Plus bodies change proportion, not just size. Rules can change the step at a *size break*, but moving points cannot change shape, so plus needs its own base size and drafting.
- **Knit is drafted smaller than the body** (*negative ease*), by an amount set by the fabric's measured stretch, usually in a dartless block, and its size steps shrink too. Sources disagree on the percentages.
- **What CAD expects in a file.** DXF-AAMA/ASTM carries sizes either as the base size with grade points tagged `# <rule number>` plus a `.RUL` text table of X/Y moves per size, or as a *graded nest* holding every size's geometry. ASTM D6673 was withdrawn in 2019 without replacement but vendors still use it, each a little differently.
- **Open tools.** Valentina (GPL-3.0-or-later) and Seamly2D (GPL-3.0) draft every size from a multisize measurement file but export one size at a time without grade data. FreeSewing (MIT) and GarmentCode (MIT) draft from measurements but export only SVG/PDF, and GarmentCode lacks seam allowances, notches and grain lines. All allow commercial use (ADR-0001). The graded-file writer must be built; ezdxf (MIT) is a good base.
- **Recommendation (assessment).** Draft each size from its measurements: the AI produces a parametric pattern program, evaluated per size against the Brazilian size chart, with regular and plus as separate ranges and knit stretch as an input. Export a graded nest plus a derived rule table, and check the whole nest automatically. Grafis, a commercial CAD, works this way.
- **Why.** Each AI design is new, so nobody has written grade rules for its points. Drafting keeps joined seams, notches and piece closure consistent in every size, and plus proportions come from the size chart instead of uniform steps.

## 1. Terms used here

| Term | Meaning |
|---|---|
| Grade point | A point on a pattern piece (a corner, a notch, the end of a curve) that moves from size to size. |
| Grade rule | How far a grade point moves in X and Y between sizes. |
| Rule table | The grade rules for a size range. In pattern files it is a `.RUL` text file. |
| Sample size | The name DXF-ASTM files use for the base size. |
| Graded nest | All sizes of a piece stacked on one reference line. Pattern makers use it to check grading; in files it means sending every size as its own geometry ("nested sizes"). |
| Size break | A size where the grade step changes, for example where the plus range starts. |
| Ease | The difference between the pattern and the body. Positive ease adds room; negative ease makes the garment smaller than the body so a stretch fabric hugs it. |
| Stretch factor | How much a knit stretches when worn, used to reduce the pattern below body size. |
| Block | A basic, fitted pattern (bodice, skirt, trousers) that designs are developed from. |
| Dart | A stitched, tapering fold that shapes flat fabric over a curve such as the bust. |
| Seam allowance | The margin outside the stitching line that the seam is sewn through. |

## 2. Two ways to produce every size

### 2.1 Rule-based grading

How it works:

- A pattern maker drafts and fits the base size, then gives each grade point a rule: "the amounts of growth in the X and Y directions at the grade point between two grading sizes" (NIST's definition of a grade delta, [NISTIR 5969](https://nvlpubs.nist.gov/nistpubs/Legacy/IR/nistir5969.pdf)).
- CAD stores rules in rule tables, either as steps between sizes or as totals from the base, and supports size breaks ([Gerber: Grade Options](http://help.gerbertechnology.com/accumark/pds/Function_Grade_Options.htm), [Gerber: Break Sizes](https://help.gerbertechnology.com/AccuMark/PDS/Function_Break_Sizes.htm)). Audaces, a Brazilian apparel CAD: "By entering grading values at one point on the base size, the system replicates the logic across the entire size set" ([Audaces Pattern](https://audaces.com/en/solutions/pattern)).
- Points without a rule are "graded proportional to its neighboring grade points" ([DH Patterns and Fit, summary of ASTM D6673-10](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). Every size inherits the base shape; Grafis notes that "grade rule patterns cannot be graded made-to-measure" ([Grafis: Grade rule grading](https://help.grafis.de/en-gb/Content/Sprungwertgradieren/Sprungwertgradieren.htm)).

Evidence on fit:

- Schofield and LaBat examined the bodice grade rules of 17 sources, found seven assumptions behind them and tested those against body measurements of US Army women: "None of the assumptions were supported. Use of these assumptions results in sized garments that do not reflect the measurements and proportions of the human body" ([Clothing and Textiles Research Journal, 2005](https://experts.umn.edu/en/publications/defining-and-testing-the-assumptions-used-in-current-apparel-grad-2/)).
- Alexander, Pisut and Ivanescu: "Apparel patterns are made for hourglass-shaped women and are graded from an average size, assuming that women's measurements increase proportionally as size increases." In the SizeUSA survey, "different hip shapes exists within a given apparel size" for plus sizes 14W to 32W ([IJFDTE, 2012](https://digitalcommons.montclair.edu/appliedmath-stats-facpubs/76/)).

### 2.2 Drafting each size from its own measurements

How it works:

- The pattern is kept as a construction: drafting steps written as formulas of body measurements plus ease, recalculated for each size (made-to-measure is the same calculation with one person's measurements). Grafis: "The construction principle does not require grading increments. The basis for generating the production patterns in the different sizes are measurement charts. Each size is represented by a specific measurement chart" ([Grafis: Grading with measurement charts](https://help.grafis.de/en-gb/Content/Gradieren/Masstabellen.htm)).
- All the open tools in section 6 work this way: Valentina and Seamly2D (formulas over a multisize measurement file), FreeSewing (JavaScript drafting code) and GarmentCode (Python programs; its paper shows "retargeting garments conditioned on body measurements across different body shapes", [Korosteleva and Sorkine-Hornung 2023](https://arxiv.org/abs/2306.03642)).
- Gill and colleagues compared the two CAD styles: the parametric approach "captures both geometric shapes and block logic" and is slower to set up but faster to modify, while the traditional one "leads to static blocks necessitating recreation for each new wearer" ([IJFDTE 2023, CC BY](https://doi.org/10.1080/17543266.2023.2260829)).

### 2.3 Side by side

| | Rule-based grading | Drafting each size |
|---|---|---|
| What a person writes | Base pattern plus a rule per grade point | Drafting logic, ease and a size chart |
| A new design | New rules for every new point (style lines, pleats, pockets) | The same logic, evaluated again |
| Seams that join | Match only if the rules on both edges agree | Match wherever the logic ties them (for example a sleeve cap computed from the armhole) |
| Plus sizes | Size breaks change the steps; the shape stays the base's | Follows the size chart; the logic can change shape per range |
| Smooth growth | Smooth by design | Only as smooth as the size chart |
| Extreme sizes | Errors grow with distance from the base | Formulas tuned on one size can break (assessment) |
| What pattern files hold natively | Base plus rules | Must be converted to a graded nest or to derived rules |
| Who uses it | Gerber, Audaces; CLO stores "displacements, specified by designers for each vertex of garment panels individually, for each size" ([GarmentCode paper](https://arxiv.org/abs/2306.03642)) | Grafis; Valentina, Seamly2D, FreeSewing, GarmentCode |

## 3. Plus sizes: break points and shape changes

Facts:

- Standards publish plus sizes as their own figure type. ASTM D6960/D6960M lists "body measurements of adult female plus women's figure type" for US sizes 14W to 40W ([ASTM, US$80](https://store.astm.org/d6960_d6960m-16r23.html)). Brazil's ABNT NBR 16933 (2021) gives women's measurement references per body type, "biótipos retângulo e colher" ([Target catalog](https://www.target.com.br/produtos/normas-tecnicas/45826/nbr16933-vestuario-referenciais-de-medidas-do-corpo-humano-vestibilidade-para-mulheres-biotipos-retangulo-e-colher)); its plus coverage and status belong to the size chart research.
- Proportional grading from an average hourglass size misfits plus bodies, and hip shape varies within one plus size (Alexander et al., above).
- Pattern maker Liz Haywood: after about a 42-inch bust "the body begins to change proportions"; shoulders and necklines stop getting wider, armholes follow a different curve, sleeve heads get steeper at the front, and a full bust adjustment becomes the norm ([The Craft of Clothes, 2019](https://lizhaywood.com.au/what-happens-after-size-16-a-discussion-on-plus-size-grading/)).
- Rule tables can carry uneven steps because every size has its own value. In a rule table exported by CLO, one rule grows 10 mm per size from XS to L and then 7.5 mm per size to XL and 2XL (section 5.3).

**Assessment.** "Break point" covers two things. A step change (bigger increments in the plus range) is easy either way. A shape change (more dart intake, a different armhole and sleeve cap, more front length over bust and belly) cannot come from moving the points of a regular base; it needs a plus base size drafted and fitted on plus proportions, as the glossary's definition of plus size already says. So v1 should treat regular and plus as two size ranges, each with its own base size, size chart rows and pattern maker review. Brazilian increments per range (blogs quote 4 cm per size up to size 42 or 44, then 6 cm) are unverified and belong to the size chart research.

## 4. Knit: negative ease and stretch factor

Facts:

- Stretch is measured with a fabric test, not guessed. ASTM D2594/D2594M-21 (stretch properties of knitted fabrics having low power) measures fabric stretch and fabric growth, which "are useful in selection of fabrics that are required to stretch, but also recover to their original shape", for both comfort-stretch and form-fitting (semi-support) apparel ([ASTM, US$72](https://store.astm.org/d2594_d2594m-21.html)).
- The reduction applied varies by source and garment:
  - FreeSewing's Lily leggings default to 40% fabric stretch and -4% ease at waist, seat and knee (range -20% to 0). The code suggests an ease of minus a tenth of the stretch and carries a note to "implement stretch setting to replace ease" ([code](https://codeberg.org/freesewing/freesewing/src/branch/develop/designs/lily/src/back.mjs)).
  - A pattern-making tutorial site lists no reduction for stable knits and 2%, 3% and 5% for moderate, stretchy and super-stretchy knits, and says knit blocks are dartless ([Dresspatternmaking](https://dresspatternmaking.com/blog/stretch-basics-for-patternmaking-2-blocks/)).
  - Kim and Choi drafted leggings straight from 3D-scan measurements and compared applying a fixed and a graduated "application percentage" of the stretch ([Journal of Engineered Fibers and Fabrics 2023, CC BY](https://doi.org/10.1177/15589250231163773)). An early example of a pattern system for stretch fabric is Ziegert and Keil's body-contouring system ([CTRJ, 1988](https://doi.org/10.1177/0887302X8800600408)).
- Grading knits: industry pattern maker Kathleen Fasanella writes that graders traditionally guess that a 2-inch grade should be 3/4 or 7/8 inch per front and back instead of a full inch, and describes deriving the grade from the fabric's width and length stretch instead ([Fashion-Incubator](https://fashion-incubator.com/grading-stretch-knit-patterns/)).

**Assessment.** For a drafting engine, knit is a set of inputs, not a separate grading method: stretch per direction (measured per fabric), negative ease per body zone (a pattern maker decision per garment, since leggings differ from a t-shirt) and a knit block variant, usually dartless. Size steps then shrink automatically because every size is drafted from reduced measurements. Since the sources disagree, the values must come from the pattern maker and fabric tests, and at least one sewn sample should be knit.

## 5. What grading data a DXF-AAMA/ASTM file carries

### 5.1 The standards and their status

- ANSI/AAMA-292 (1993) added apparel conventions to AutoCAD R11 DXF. NIST noted in 1997 that it "does not define how to transfer grading information, it uses a grading rule identification to refer to information that is presented elsewhere" ([NISTIR 5969](https://nvlpubs.nist.gov/nistpubs/Legacy/IR/nistir5969.pdf)).
- ASTM D6673 added the rule table: it "uses the DXF file format for pattern piece data exchange and a specially formatted ASCII file format for grade rule tables" and specifies AutoCAD Version 13 DXF. ASTM lists D6673-10 as "Withdrawn 2019" with "No replacement" ([ASTM store](https://store.astm.org/d6673-10.html)).
- It is still the working format. Grafis calls AAMA "currently the most widely used format" and ASTM "more standardised but not available in all CAD systems" ([Grafis: Export formats](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)); CLO wrote a D6673-04 rule table in 2023 (section 5.3); the Audaces Moldes v10 manual lists import to ANSI/AAMA-292 and ASTM D6673-01 and export as "ASTM D6673-01/2001 (antigo AAMA-DXF)" ([third-party copy of the manual](https://idoc.pub/documents/manual-digital-audaces-moldes-vs10pdf-vnd50m8g2wlx)).

### 5.2 File layout

One DXF file holds one style, and each pattern piece is a DXF block. Content is split across numbered layers; from Valentina's implementation notes: 1 piece boundary, 2 turn points, 3 curve points, 4 V-notches and slit notches, 5 "grade reference and alternate grade reference line(s)", 6 mirror line, 7 grain line, 8 internal lines, 11 internal cutouts, 13 drill holes, 14 sew lines, 15 annotation text, 80 to 83 other notch types, 84 to 87 quality validation curves ([dxiface.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/dxiface.cpp)). Text for the whole style records the sample size, the rule table name and the units ([DH summary](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)).

### 5.3 Carrier A: base size plus a rule table

The full ASTM text is withdrawn and was paywalled, so the details below come from a summary written by an ASTM member ([DH Patterns and Fit](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)) and match a real file exported by CLO:

- The DXF holds the base size only. Each grade point is tagged with the text `# <ID>` at the point's coordinates; the ID is its rule number.
- The `.RUL` file is plain text: a header (version, author, date, units, rule table name, sample size, number of sizes, size list from small to large), then for each rule a line `RULE: DELTA <n>` followed by one X,Y pair per size, "measured from sample size" (0,0 for the base). Metric files use millimetres with two decimals.
- X and Y follow the piece's grade reference line (layer 5) when present, otherwise the drawing's axes.
- The DXF and the RUL file must keep the same name and travel together ([Grafis: export steps](https://help.grafis.de/en-gb/Content/ExportIImport/Schrittfolge_fuer_den_Export.htm)).

Excerpt of a rule table exported by CLO Standalone 7.2.116 in 2023 ([public thesis repository](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns)):

```text
ASTM/D13 Proposal 1 Version: D 6673-04
AUTHOR: CLO Virtual Fashion Inc.
UNITS: METRIC
GRADE RULE TABLE: W SS
SAMPLE SIZE: XS
NUMBER OF SIZES: 6
SIZE LIST: XS S M L XL 2XL
RULE: DELTA 0
     0.00     0.00     10.00     0.00     20.00     0.00     30.00     0.00
     37.50     0.00     45.00     0.00
```

The pairs are X,Y for XS, S, M, L, XL and 2XL; the step drops from 10 mm to 7.5 mm after L. In the paired DXF, the tags (`# 1` and so on) are text on layer 1 at each point, and every piece carries `SIZE: XS`.

### 5.4 Carrier B: graded nest

ASTM also allows sending every size as geometry: "multiple blocks with the same piece name form a collection of different sizes within a DXF file", in size order, each with its size name, and "Blocks repeat all points and attribute definitions (ATTDEF) in the same order, quantity and layer as sample size" ([DH summary](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). Grafis offers this as "noRUL" for AAMA, where "all sizes are contained in the DXF file as perimeters", and as "GradedNest" for ASTM ([Grafis: Export formats](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)).

### 5.5 Caveats that matter here

- CAD systems read the formats differently. Grafis warns that AAMA "is not clearly defined in all points so that significant differences may occur during interpretation by different CAD systems" and that the right export configuration "has to be tested" ([Grafis](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)). For example, CLO puts its grade tags on layer 1, while the DH summary places them on the point's own layer.
- Base plus rules loses information when the source drafts each size. Grafis: "all pattern stacks are reduced to a base size with grade rules. As all sizes are constructed individually in Grafis, this leads to a loss of information. The target system calculates the contours of the different sizes from these reduced data with its own mathematical algorithms. Therefore, the shape can differ from the original shape in Grafis, particularly in extreme sizes" ([Grafis](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)).
- Brazilian CAD: Molde.me, a Brazilian web CAD, imports AAMA and DXF plus RUL "com o molde e a gradação gerada" ([Molde.me help](https://molde.me/ajuda/importar-moldes)). Whether Audaces imports RUL grading or graded nests is **unverified**; the file format research and a real import test must settle it.

## 6. Open tools, checked against ADR-0001

| Tool (version checked) | License (file read) | How it makes sizes | Every size in one file? | Export formats | Knit and plus | Assessment |
|---|---|---|---|---|---|---|
| [Valentina](https://gitlab.com/smart-pattern/valentina) 1.1.1, 18 Sep 2026 | GPL-3.0-or-later | Formulas over a multisize file: 1 to 3 dimensions (height, size, waist, hip), base value plus a fixed shift per step, plus per-size corrections | No. One size per export (GUI or command line); AAMA/ASTM output has no grade layer and no RUL | SVG, PDF, tiled PDF, PNG, TIF, OBJ, PS, EPS, flat DXF, DXF-AAMA, DXF-ASTM (R12), HPGL, HPGL/2, PLT | Ease and stretch only through formulas; a plus range via corrections or a separate file | Strongest open drafting engine for hand-built blocks; a Qt desktop app, scriptable once per size |
| [Seamly2D](https://github.com/FashionFreedom/Seamly2D) v2026.9.21.146 | GPL-3.0 | Formulas over a multisize file: size and height only, base value plus a fixed increment per step, no corrections | No. ASTM export is marked "For now not supported" in code | SVG, PDF, tiled PDF, PNG, JPG, BMP, PPM, TIF, OBJ, PS, EPS, flat DXF, DXF-AAMA | As Valentina, but fixed steps cannot follow a size break within one file | Weaker than Valentina for real size charts |
| [FreeSewing](https://codeberg.org/freesewing/freesewing) v4.10.2 | MIT | JavaScript drafting code over body measurements (38 in its test data) | No. It can overlay several measurement sets in one SVG for checking | SVG, PDF (one untiled page, or tiled on A4 to A0, Letter, Legal, Tabloid, Arch D and E), JSON/YAML settings | Per design (Lily leggings: stretch and negative ease options) | Good MIT reference for drafting code; no DXF |
| [GarmentCode](https://github.com/maria-korosteleva/GarmentCode) / pygarment 2.0.2 | MIT | Python programs over 26 body measurements plus design parameters | No. Run again per size | JSON, SVG, PNG, PDF | No stretch input; default widths never go below body size | Closest to AI-written pattern programs, but built for simulation: no seam allowances, notches or grain lines |
| [ezdxf](https://github.com/mozman/ezdxf) 1.4.4 | MIT | Not a drafting tool | Could write either carrier with our code | DXF | Not applicable | Base for our own exporter |
| [JBlockCreator](https://github.com/aharwood2/JBlockCreator) 1.6 | GPL-3.0 (University of Manchester) | Java drafts of textbook blocks from body-scan measurements | No | DXF (an ASTM attempt that Lectra reads as AAMA) | No | Research reference; inactive since 2022 |

Notes that bear on the decisions (the license and source-code evidence for each row is in the appendix):

- **GPL in a web app.** GPL-3.0 allows selling ("You may charge any price or no price for each copy that you convey"), and running software on a server is not distribution: "Mere interaction with a user through a computer network, with no transfer of a copy, is not conveying" ([GPL-3.0 text](https://gitlab.com/smart-pattern/valentina/-/blob/develop/LICENSE_GPL.txt)). Obligations start only if GPL code is shipped to users.
- **Libraries.** On 2026-09-26 I found no maintained package on PyPI or npm that writes AAMA/ASTM grade rules; GitHub has only small, new projects that read RUL files (for example [seamer-studio](https://github.com/ahzs645/seamer-studio), MIT, no stars).
- **Costs.** All tools above are free. ASTM D6673-10 is withdrawn and listed out of stock; ASTM D6960 costs US$80 (US sizes, not needed); ASTM D2594 costs US$72 (only if we want the official knit test procedure); ABNT NBR 16933 costs R$197.10 at the Target reseller (the size chart research owns this). All checked on 2026-09-26.

## 7. Recommendation

**Assessment: draft each size from its own measurements, and export the result in both carriers.**

1. The AI produces (or chooses and fills in) a *parametric pattern program*: garment logic plus design parameters, written against named body measurements and ease. Each size is one evaluation of that program with one size chart row.
2. Regular and plus are two size ranges, each with its own base size, size chart rows and pattern maker review. The program may change shape logic at the break (dart intake, armhole and sleeve cap), not only the step size.
3. Woven and knit share the garment logic; knit adds measured stretch per direction, negative ease per zone and a knit block variant.
4. Every size keeps the same points in the same order (curves sampled at fixed positions, not adaptively), so the output can be written as a graded nest (the exact geometry of every size, the primary production file) and as base size plus a rule table computed by subtracting each size's points from the base (for CAD users who edit rules). Both go through an import test with the factory's CAD.
5. Every size is checked automatically: same points in every size, seams that join have equal lengths, notches line up, pieces close, no self-intersections, and smooth growth except at declared size breaks. The pattern maker reviews each base size and the graded nest; sewn samples cover the base of each range and the extreme sizes.

Why:

- Rule-based grading presumes a person writes rules for every new point. For AI-generated designs nobody has. Transferring an arbitrary garment to other body sizes automatically is still a research subject (the GarmentCode paper surveys optimization-based retargeting work), and I found no open tool that assigns production grade rules to arbitrary pattern geometry.
- Production-ready needs joined seams and notches to agree in every size. Drafting logic that ties them keeps them consistent by construction; with rules, consistency depends on how the rules on both edges were assigned.
- Plus proportions and knit reductions enter as data and logic instead of uniform steps, which the grading studies above show do not follow bodies.
- One engine serves both the 3D preview at any size and the pattern file.
- CAD compatibility is kept: Grafis drafts every size and still exports AAMA/ASTM, either as base plus rules or as a graded nest.

Costs and risks:

- It depends on a Brazilian size chart with full values for every size and every measurement the logic uses, for regular and plus.
- The drafting logic must hold across the whole range, and the plus extremes are where formulas break. This needs pattern maker input for each garment family.
- The exporter must be written (small, on ezdxf) and tested against the target CAD. The rule table carrier can be redrawn differently at extreme sizes by the receiving CAD, so the graded nest should be the reference.

## 8. What this means for the decision tickets

- **Choose the pattern-generation route and its AI model:** this favours routes that output a parametric program that can be evaluated again per size, over routes that output one size of geometry (a flattened mesh or raw panel shapes), which would force automatic rule grading of arbitrary shapes.
- **Choose the grading approach:** the recommendation above; still open is which drafting engine to use (Valentina-style formulas, FreeSewing-style JavaScript, GarmentCode-style Python, or our own) and who writes the drafting logic.
- **Define the production-ready acceptance test:** add the whole-nest checks from section 7 and cover several sizes, including plus and knit, in the sewn samples.
- **Choose the export route:** decide graded nest, rule table or both, AAMA or ASTM, and the DXF version, and plan an import test with the factory's CAD.
- **Brazilian women's size charts, regular and plus:** the drafting engine needs every size's value for every measurement it uses, not only increments.

## 9. Confidence and gaps

- **Solid:** licenses (license files read), what each open tool can and cannot export (checked in source code), the ASTM withdrawal, how Grafis, Gerber and Audaces describe their grading, and the structure of a real CLO rule table and DXF.
- **Secondary:** the detailed ASTM rule table syntax and graded nest rules come from one practitioner summary of the withdrawn, paywalled standard; they agree with Valentina's code and the CLO files.
- **Unverified:** whether Audaces imports RUL grading and graded nests; how current CLO versions export grading (only a 2023 file was checked); Brazilian plus-size increments; exact knit reduction percentages; whether GarmentCode and Valentina keep identical point lists across sizes once curves are turned into points.

## Appendix: evidence behind the tool table

- **Licenses (files read).** Valentina [LICENSE_GPL.txt](https://gitlab.com/smart-pattern/valentina/-/blob/develop/LICENSE_GPL.txt), with the README granting "version 3 of the License, or (at your option) any later version"; Seamly2D [LICENSE](https://github.com/FashionFreedom/Seamly2D/blob/develop/LICENSE); FreeSewing [LICENSE](https://codeberg.org/freesewing/freesewing/src/branch/develop/LICENSE) (the project moved to Codeberg; its [GitHub repository](https://github.com/freesewing/freesewing) is archived); GarmentCode [LICENSE](https://github.com/maria-korosteleva/GarmentCode/blob/main/LICENSE) and [pygarment](https://pypi.org/project/pygarment/); GarmentCode's body-measuring companion [GarmentMeasurements](https://github.com/mbotsch/GarmentMeasurements) (GPL-3.0); [ezdxf](https://pypi.org/project/ezdxf/); [JBlockCreator](https://github.com/aharwood2/JBlockCreator).
- **Valentina.** A measurement's value is its base value plus, for each dimension, a shift times the number of steps from the base, plus any correction stored for that exact size ([vmeasurement.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vpatterndb/variables/vmeasurement.cpp), [schema v0.6.2](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/ifc/schema/multisize_measurements/v0.6.2.xsd)). The `--dimensionA`, `--dimensionB` and `--dimensionC` options pick the size for a command-line export ([commandoptions.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vmisc/commandoptions.cpp)); formats are in [vlayoutdef.h](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vlayout/vlayoutdef.h). Its ASTM layer list leaves layer 5 (grade reference) commented out and the DXF engine has no rule table code ([dxiface.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/dxiface.cpp), [vdxfengine.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/vdxfengine.cpp)). A request for a grade nest view has been open since 2024 ([issue 203](https://gitlab.com/smart-pattern/valentina/-/issues/203)).
- **Seamly2D.** Each measurement has only `base`, `size_increase` and `height_increase` ([schema v0.4.5](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/ifc/schema/multi_size_measurements/v0.4.5.xsd)); `--gsize` and `--gheight` pick the size ([commandoptions.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vmisc/commandoptions.cpp)); ASTM export hits `Q_UNREACHABLE(); // For now not supported` ([mainwindowsnogui.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/app/seamly2d/mainwindowsnogui.cpp)); formats are in [def.h](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vmisc/def.h).
- **FreeSewing.** Its bundled sizes are "Body measurements data used to test FreeSewing designs", extrapolated from one average body "by keeping the same proportions"; the authors add "we are not in the business of standard sizes" ([README](https://codeberg.org/freesewing/freesewing/src/branch/develop/packages/models/README.md), [neckstimate.mjs](https://codeberg.org/freesewing/freesewing/src/branch/develop/packages/models/src/neckstimate.mjs)). See also the [sampler](https://codeberg.org/freesewing/freesewing/src/branch/develop/packages/core/src/pattern/pattern-sampler.mjs) and [export code](https://codeberg.org/freesewing/freesewing/src/branch/develop/packages/react/components/Editor/lib/export/index.mjs).
- **GarmentCode.** Output formats are in [wrappers.py](https://github.com/maria-korosteleva/GarmentCode/blob/main/pygarment/pattern/wrappers.py); a code search of the repository on 2026-09-26 found no seam allowance, notch or grain line handling. Body-relative widths in its default design space start at 1.0 (shirt 1.0 to 1.3, pants 1.0 to 1.5, waistband 1.0 to 2; [default.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/design_params/default.yaml)). Its body inputs include values that size charts rarely list, such as head length, hip and shoulder inclination and arm pose angle ([mean_all.yaml](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/mean_all.yaml)); its bodies are SMPL-based averages or its own body model ([bodies Readme](https://github.com/maria-korosteleva/GarmentCode/blob/main/assets/bodies/Readme.md)), a licensing question for the body model research, since drafting needs only measurements. The paper lists limits that matter for production: no panels with holes, limited pleats, and a wish "to further accommodate the differences between sewing patterns for garment fabrication vs. simulation" ([paper](https://arxiv.org/abs/2306.03642)).
