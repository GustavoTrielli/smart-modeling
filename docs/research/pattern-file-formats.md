# Pattern file formats for apparel CAD and plotting

Research for the ticket "Pattern file formats for apparel CAD and plotting" on the map "Map: smart-modeling v1 spec".
All sources were read on 2026-09-26, and every price and term below was checked that day.
Facts carry a link to their source. My own judgments are labelled **Assessment**.

## Answer in brief

- **For apparel CAD, deliver DXF-ASTM (ASTM D6673) or its predecessor DXF-AAMA (ANSI/AAMA-292).** Both are plain DXF text files with agreed meanings: one *block* per pattern piece, numbered *layers* per kind of marking, and structured text for piece name, size and quantity. Audaces and Gerber AccuMark import and export both, GRAFIS writes both, and CLO and Richpeace write ASTM.
- **Neither standard is maintained.** ASTM withdrew D6673 in January 2019; AAMA-292 dates from 1993. D6673-10 is still sold for USD 86; I found no seller for AAMA-292. So "compliant" in practice means "the receiving CAD imports it cleanly", and the pattern maker's own CAD is the real test.
- **Grading travels as the base size plus a `.RUL` rule table** (each grade point's x/y offset in every size) **or as a graded nest** (every size as its own block). Only the graded nest carries each size's exact curves.
- **For plotting, apparel plotters and bureaus take PLT (HP-GL or HP-GL/2), and print shops take full-size vector PDF.** Both are plain drawings, with no piece structure or grading data.
- **Each piece must carry:** name, style reference, size, quantity to cut (left/right pairs), cut line with seam allowance, sew line, grain line, notches, drill holes, internal lines, fold line, material, units, and the grading.
- **Marker making (encaixe) is done by the factory's cutting room or a bureau**, because it needs fabric width, table length and quantities per size. **Assessment:** keep it out of v1.
- **Writers within ADR-0001:** ezdxf (MIT) writes everything these DXF files need; RUL and HP-GL are small text formats; reportlab (BSD) or jsPDF (MIT) write PDF. Valentina (GPL-3.0) has the most complete open AAMA/ASTM exporter and is best used as a reference; Seamly2D (GPL-3.0) writes AAMA only.

## Terms used here

- **Cut line** (linha de corte): where the fabric is cut; with seam allowance included, the allowance's outer edge.
- **Seam allowance** (margem de costura): the strip between cut line and sew line.
- **Sew line** (linha de costura): where the stitching goes.
- **Notch** (pique): a small cut on the edge marking where pieces meet or fold.
- **Grain line** (fio): how the piece sits relative to the fabric's lengthwise threads.
- **Drill hole** (furo): a point marked through the fabric, for dart tips, pockets or buttons.
- **Internal line** (linha interna): drawn but not cut, such as a dart (pence) or pleat (prega).
- **Rule table** (regras de gradação): each grade point's x/y move per size, in a `.RUL` file.
- **Graded nest**: all sizes of a piece in one file, one block per size.
- **Marker** (encaixe; the printed sheet is the risco): all pieces of one cutting job laid across the fabric width, printed full size.
- **Bureau** (bureau or birô de plotagem): a company that grades, makes markers and plots for factories.

Word clash: ASTM calls the base size the "sample size". In this project a *sample* is the sewn garment (peça piloto), so this report says **base size**.

## 1. The formats at a glance

| Format | What it holds | Grading | Who reads or writes it | Spec and cost | Open writers (license) |
|---|---|---|---|---|---|
| **DXF-ASTM** (ASTM D6673) | One block per piece; layers for cut line, 6 notch types, grain line, drill holes, internal lines, sew lines, text; piece and style text | Base size + `.RUL`, or graded nest | Read by Audaces and Gerber AccuMark; written by them and by GRAFIS, CLO, Richpeace, Valentina (§3) | Withdrawn 2019; D6673-10 PDF USD 86 at MyStandards, 790 SEK at SIS (§2.7) | ezdxf (MIT) plus our conventions; Valentina (GPL-3.0) |
| **DXF-AAMA** (ANSI/AAMA-292, 1993) | Same core layers, fewer notch types, looser rules | Base size + `.RUL`, or all sizes as outlines ("noRUL") | Read by Audaces, Gerber, Molde.me; written by Gerber, GRAFIS, Molde.me, Seamly2D, Valentina | No seller found (§2.7) | Same as above; Seamly2D (GPL-3.0) |
| **Plain DXF** | Lines only, no meaning | None | Any CAD, but the meaning is lost | Autodesk DXF reference, free | ezdxf and many others |
| **PLT** (HP-GL, HP-GL/2) | Pen moves and text for a plotter | Only as drawn outlines | Read by apparel plotters, bureaus and Molde.me (§4) | HP reference, free | Own code (tiny), vpype (MIT), Valentina (GPL-3.0) |
| **Full-size PDF** | Vector drawing at 1:1 scale | Only as drawn outlines | Read by print shops (gráficas), home sewers and Molde.me (§4) | Adobe PDF 1.7 reference, free | reportlab (BSD), jsPDF (MIT), pdf-lib (MIT) |

## 2. Inside DXF-ASTM and DXF-AAMA

Primary source: pages 1 to 4 (of 10) of ASTM D6673-10, free as a reseller's [preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf). For the other pages I used the [DH Patterns and Fit ASTM guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf) (Oslo, 2011, written from D6673-10 by an ASTM member), checked against two real exports, from Richpeace Design V9 and CLO 7.2 ([sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns), unlicensed, read as evidence only), and against the [Valentina](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/vdxfengine.cpp) and [Seamly2D](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vdxf/vdxfengine.cpp) exporters.

### 2.1 File skeleton

- A DXF file is ASCII text made of code/value pairs, with an optional header and tables, blocks and entities sections ([preview §4.1](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- D6673-10 names the AutoCAD Release 13 DXF specification ([preview §1.2](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)). AAMA-292 was built on Release 11 ([NISTIR 5969](https://archive.org/stream/datasharingimple5969leey/datasharingimple5969leey_djvu.txt), NIST, 1997).
- Each piece is a separate BLOCK, with no nested INSERTs, and one file holds a single style ([preview §4.3.1.3](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- Real exporters write minimal files in the old R12 style. The Richpeace V9 export has an empty header, no tables section, one BLOCK per piece, and an entities section holding the style text plus one INSERT per block; the CLO 7.2 export declares AC1006 (Release 10) ([sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns)). Valentina's changelog records a fix for "compatibility with Richpeace DXF-AAMA/ASTM R12" ([ChangeLog](https://gitlab.com/smart-pattern/valentina/-/blob/develop/ChangeLog.txt)).
- `Units: METRIC` means millimetres to two decimals; `ENGLISH` means inches to four ([preview §4.3.1.1](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- Text values may only use ASCII 7-bit characters ([preview §4.3.1.1](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)), and Gerber truncates style and piece names at 20 characters ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). A consequence: Portuguese names with accents, such as "Cós", must be written without them.

### 2.2 Layers

The layer list below is quoted from the standard ([preview §4.3](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)); the entity column comes from the DH guide and matches both real exports.

| Layer | ASTM meaning | Plain meaning | DXF entity |
|---|---|---|---|
| 1 | Piece boundary, plus style and piece system text | Cut line | Closed POLYLINE; TEXT |
| 2 | Turn points | Corners, used to rebuild lines | POINT |
| 3 | Curve points | Points along curves | POINT |
| 4 | V-notch and slit notch | Notches (piques) | POINT |
| 5 | Grade reference and alternate grade reference lines | Axis that grade rules follow | LINE |
| 6 | Mirror line | Fold line of a half piece | LINE |
| 7 | Grainline | Grain line (fio) | LINE |
| 8 | Internal lines | Darts, pleats, placement lines (drawn, not cut) | POLYLINE or LINE |
| 9, 10 | Stripe and plaid reference lines | Matching stripes and checks | LINE, optional POINT |
| 11 | Internal cutouts | Lines cut inside the piece | POLYLINE |
| 12 | Intentionally left blank | | |
| 13 | Drill holes | Drill holes (furos) | POINT |
| 14 | Sew lines | Sew line | POLYLINE |
| 15 | Annotation text | Printed notes | TEXT |
| 80, 81, 82, 83 | T-notch, castle notch, check notch, U-notch | Other notch shapes | POINT |
| 84 to 87 | Quality validation curves for layers 1, 8, 11, 14 | Dense copies of curves to check import accuracy | POLYLINE |

Rules from the standard: layer 1 is required and must be a closed polygon; layers 2 and 3 hold all turn and curve points of layers 1, 8, 11 and 14; layers 5, 6, 7, 9, 10 and 13 may not contain polylines ([preview §4.3](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).

### 2.3 System text

Written as `Identifier:<value>` TEXT on layer 1 ([preview §4.3.1.1 and §4.3.1.2](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).

| Level | Identifiers | Notes |
|---|---|---|
| Style (once, in the entities section) | `Style Name`, `Creation Date` (dd-mm-yyyy), `Creation Time`, `Author` (vendor;application;release), `Sample Size` (the base size), `Grade Rule Table`, `Units`, standard version, `Curve Tolerance` | `Sample Size` and `Grade Rule Table` must appear even when blank; `Curve Tolerance` appears only when validation curves are used |
| Piece (inside each block) | `Piece Name` (the only required one), `Quantity` (R,L, for example `1,1` for a mirrored pair), `Rotation`, `Flip`, `Tilt`, `Fold`, `Material` | `Fold` means the piece may be laid on the fold of tubular fabric; a piece cut on the fold of the pattern uses the mirror line on layer 6 instead |

### 2.4 How the markings are encoded

- **Seam allowance.** Layer 1 is the cut line; when allowances are included, the finished outline goes on layer 14 as the sew line ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). No field states the allowance width: it is the geometry between the two lines, which is also how Valentina and Seamly2D export.
- **Notches.** A POINT on the piece edge. The layer gives the shape (4 = slit or V, 80 = T, 81 = castle, 82 = check, 83 = U), code 30 (the Z value) the depth, code 39 the width at the edge, and code 50 the angle from the x axis ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). Valentina's writer ([libdxfrw.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/libdxfrw/libdxfrw.cpp)) and both real exports use the same encoding: Richpeace T-notches on layer 80 about 2.5 mm deep, CLO slit notches on layer 4 7 mm deep ([sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns)). Simpler options are to cut the notch into the outline or draw it on layer 8 ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)); Seamly2D writes short LINEs on layer 4.
- **Grain line.** A LINE on layer 7 ([preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf), both samples).
- **Drill holes.** A POINT on layer 13; the Z value may hold the diameter and an optional integer text may give a drill type ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)). Valentina writes the diameter in Z.
- **Internal lines, internal cutouts, sew lines.** Layers 8, 11 and 14 ([preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- **Fold (mirror) line.** A LINE on layer 6 says the piece is a half that must be mirrored; the text `NM` at the start of an element marks it as "not mirrored" ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)).
- **Annotation.** TEXT on layer 15, recommended 3.5 or 5 mm high ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)).
- **Turn and curve points and validation curves.** Points on layers 2 and 3 let the importer rebuild curves within the `Curve Tolerance`. The standard's wording makes the validation curves (layers 84 to 87) optional ("must exist and only exists when Quality Validation Curves are used", [preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)); the DH guide calls them mandatory. Both real exports include layers 84 and 85.

### 2.5 Grading

- **Grade rule identifier.** TEXT `# <id>` (optionally `# <id>, <alternate id>`) placed at the same XY as the point it grades; every point that must not move needs a zero-growth rule; rules may not sit on layers 2, 3, 6 or 84 to 87 ([preview §4.3.1.4](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- **Rule table (`.RUL`).** An ASCII file: a header, then one `RULE: DELTA <id>` block per rule with one x,y offset per size in `SIZE LIST` order, measured from the base size (0,0) along the grade reference line, and a closing `END` ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf), quoting a Gerber AccuMark file). The Richpeace and CLO files match, apart from comma use. Illustrative example (Brazilian sizes, invented values):

  ```text
  ASTM/D13 Proposal 1 Version: D 6673-10
  AUTHOR: smart-modeling
  CREATION DATE: 26-09-2026
  CREATION TIME: 14:00
  UNITS: METRIC
  GRADE RULE TABLE: VESTIDO01
  NUMBER OF SIZES: 5
  SIZE LIST: 36 38 40 42 44
  SAMPLE SIZE: 40
  RULE: DELTA 1
     -10.00, -5.00    -5.00, -2.50     0.00, 0.00     5.00, 2.50    10.00, 5.00
  RULE: DELTA 2
       0.00, 0.00      0.00, 0.00      0.00, 0.00     0.00, 0.00     0.00, 0.00
  END
  ```

- **Delivery.** The DXF and the RUL share a base name and must travel together, ideally zipped; names must not change after export; GRAFIS advises at most 8 characters, with no spaces or special characters ([GRAFIS](https://help.grafis.de/en-gb/Content/ExportIImport/Schrittfolge_fuer_den_Export.htm)).
- **Graded nest.** Several blocks with the same `Piece Name`, each with a `Size Name`, in size order, each with a grade reference line; every point and attribute is repeated in the same order, quantity and layer as the base size; graded sizes leave out layers 2, 3, 4, 80 to 83 and alternate grade lines. The blocks may be stacked or placed apart, and the receiving system "should be able to interpret" either layout ([preview §4.3.1.5](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- **Why the nest matters.** The standard defines no curve interpolation, so the curves of rule-graded pieces "cannot be validated"; only a graded nest guarantees them ([preview §4.3.1.6](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)).
- **Support.** GRAFIS exports AAMA with a RUL or as "noRUL" (all sizes as outlines in the DXF), and ASTM with a RUL or as a "GradedNest" ([GRAFIS](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)).

### 2.6 AAMA versus ASTM

- AAMA-292 was published in 1993, built on DXF Release 11, and defined 14 layers; it "does not define how to transfer grading information" beyond rule identifiers ([NISTIR 5969](https://archive.org/stream/datasharingimple5969leey/datasharingimple5969leey_djvu.txt)).
- D6673 was first approved in 2001 and revised in 2004 and 2010 ([preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)). Compared with AAMA, ASTM "contains a number of new notch types and supports descriptive text", and GRAFIS offers graded nests only in its ASTM export ([GRAFIS](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)).
- GRAFIS calls AAMA "currently the most widely used format" but "not clearly defined in all points", and ASTM "more standardised but not available in all CAD systems" ([GRAFIS](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm)). A Brazilian nesting vendor wrote that AAMA "sofre de divergências na sua implementação" ([Otimize Nesting](https://www.otimizenesting.com.br/post/2016/10/19/dxf-aama-e-astm), Joinville, 2016).
- Dialects differ in practice. CLO writes `PIECE NAME:` and `SIZE:` where the standard says `Piece Name:` and `Size Name:` ([sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns)). Valentina ships compatibility modes for Richpeace CAD V8, V9, V10 and CLO3D ([dxfdef.h](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/dxfdef.h)), and a Seamly2D code comment notes that "Optitex doesn't like" a text layer 19 ([dxiface.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vdxf/dxiface.cpp)).
- Unverified: I found no copy of the AAMA-292 text, so its exact layer list is unknown. The shared numbering of the core layers (1 to 15, with 12 blank in ASTM) suggests ASTM kept AAMA's numbers, but this is inference.

### 2.7 Getting the specifications

| Document | Status | Where and price (checked 2026-09-26) |
|---|---|---|
| ASTM D6673-10, 10 pages | Withdrawn; ASTM store says "Withdrawn 2019" and shows no stock ([ASTM](https://store.astm.org/d6673-10.html)); BSI records 17 January 2019 ([BSI](https://knowledge.bsigroup.com/products/standard-practice-for-sewn-products-pattern-data-interchange-data-format)) | USD 86 PDF at [MyStandards](https://www.mystandards.biz/standard/astm-d6673-10-1.1.2010.html); 790 SEK at [SIS](https://www.sis.se/en/produkter/external-categories/textiles-astm-vol-07/textiles-ii-d4393--latest-astm-vol-0702/astmd6673); pages 1 to 4 free as a [preview](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf) |
| ASTM D6673-04 (10 pages), D6673-01 (9 pages) | Historical | USD 86 each at the ASTM store ([2004](https://store.astm.org/d6673-04.html), [2001](https://store.astm.org/d6673-01.html)) |
| ASTM D6963-04, terminology, 3 pages | Historical | USD 77 at the [ASTM store](https://store.astm.org/d6963-04.html) |
| ANSI/AAMA-292 (1993) | A 2002 ANSI Standards Action notice is indexed as withdrawing it effective 13 November 2002, but the PDF was bot-protected, so this is unverified ([ANSI](https://share.ansi.org/Shared%20Documents/Standards%20Action/2002%20PDFs/SAV3341.pdf)). AAMA merged into AAFA in August 2000 ([AAFA](https://www.aafaglobal.org/AAFA/About/Who_We_Are.aspx)) | No seller found |
| DH Patterns and Fit ASTM guide | Secondary, written from D6673-10 | [Free PDF](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf) |
| AutoCAD DXF reference | Release 12 and 2012 editions as PDF, current edition online | Free from Autodesk ([R12](https://damassets.autodesk.net/content/dam/autodesk/www/developer-network/platform-technologies/autocad-dxf-archive/acad_r12_dxf.pdf), [2012](https://images.autodesk.com/adsk/files/autocad_2012_pdf_dxf-reference_enu.pdf), [online](https://help.autodesk.com/view/ACD/2022/ENU/?guid=GUID-3F0380A5-1C15-464D-BC66-2C5F094BCFB9)); no Release 13 edition found |

## 3. What CAD systems import and export

| System | Imports | Exports | Source |
|---|---|---|---|
| Audaces Moldes (Florianópolis, founded 1992, [ND+](https://ndmais.com.br/moda-e-beleza/audaces-a-empresa-de-florianopolis-que-mudou-a-historia-da-moda-no-brasil/)) | AutoCAD DXF; AAMA/ASTM "de acordo com as normas ANSI/AAMA-292 ... de 1993, e ASTM D6673-01"; Lectra MDL, VET, IBA; Gerber zip; Investronica | DXF (basic, by layer, by tool); ASTM D6673-01 ("antigo AAMA-DXF"), with a "Compatibilizar com o AAMA Gerber" option; PLT through "Plotar para arquivo" | Version 10 manual, vendor text on third-party hosts ([pdfcoffee](https://pdfcoffee.com/manual-digital-audaces-moldes-vs10pdf-pdf-free.html), [idoc](https://idoc.pub/documents/manual-digital-audaces-moldes-vs10pdf-vnd50m8g2wlx)); the current [product page](https://audaces.com/en/solutions/pattern) only says DXF import and export |
| Audaces Encaixe (marker software) | Audaces patterns, AAMA/ASTM DXF, Lectra Vet-iba, Investronica, Gerber zip | Plot files (PLT, HPG, HPA), ISO cutter files, EMF/WMF, PDF, XML | [Version 10 manual](https://pdfcoffee.com/manual-digital-audaces-encaixe-vs10-pdf-free.html) |
| Gerber AccuMark | Rule tables, pieces and markers from Assyst, Lectra, Investronica, AutoCAD DXF, AAMA/ASTM, TIIP, IGES | ASTM DXF, AAMA DXF, standard DXF, IGS; grade rule table | [Data Conversion Utility](https://help.gerbertechnology.com/AccuMark/AMX/Function_DCU.htm), [Export](https://help.gerbertechnology.com/AccuMark/PDS/File_Export_Tab_File.htm) |
| GRAFIS | (not checked) | AAMA/DXF with RUL or noRUL, ASTM/DXF with RUL or GradedNest, plain DXF | [GRAFIS help](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm) |
| CLO 3D | Not verified (help pages bot-protected) | ASTM D6673-04 DXF plus RUL, seen in a real CLO 7.2 export | [Sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns) |
| Richpeace Design | Not verified | ASTM D6673-04 DXF plus RUL, seen in a real V9 export | [Sample files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns) |
| Molde.me (Brazilian, web) | AAMA, DXF, RUL, EMF, GIF, JPG, PDF, PGM, PLT, PNG, SVG; best for automatic import: AAMA, PLT, DXF, DXF+RUL | DXF, EMF, PDF, PLT, PLT "modo compatibilidade", SVG, AAMA | [Import](https://molde.me/ajuda/importar-moldes), [export](https://www.molde.me/ajuda/exportar-molde) |
| MODA-01 (Brazilian) | DXF, HP-GL/2 | DXF, HP-GL/2 | [Freedom Atelier](http://www.modelagem.ind.br/servicos) |
| Lectra Modaris, Optitex, RZ CAD Têxtil | Not verified: first-party help is behind customer logins or does not list formats | | |

**Assessment:** every system above that documents apparel interchange speaks AAMA/ASTM DXF, and Audaces reads both. The Audaces manual I could read is the old version 10, so the current version's handling of graded nests and notch types must be tested, not assumed.

## 4. Plotting: PLT and full-size PDF

- **PLT is HP-GL:** plotter commands such as `IN` (initialize), `SP` (select pen), `PU` and `PD` (pen up, pen down), `PA` (plot absolute) and `LB` (label), in plotter units of 0.025 mm, 40 per millimetre ([HP-GL/2 reference](http://www.hp.com/ctg/Manual/bpl13211.pdf)).
- **Apparel plotters read it.** "As plotters comercializadas pela Audaces aceitam arquivos no formato PLT (plotagem) com o protocolo de comunicação HPGL e HPGL/2" ([Audaces, December 2025](https://audaces.com/pt-br/blog/plotter-audaces)); "a maioria dos equipamentos do mercado trabalha com os formatos HPGL e PLT", and a marker plotter costs R$ 15,000 to 40,000 ([Audaces, July 2026](https://audaces.com/pt-br/blog/plotter-risco)). Dialects vary: Molde.me offers both "PLT" and "PLT (modo compatibilidade)" ([Molde.me](https://molde.me/blog/exportacao-aama)).
- **PLT and PDF carry no data**, only lines and labels. Still, Molde.me says most AAMA, PLT and PDF files import "em tamanho real com os moldes e gradações geradas" ([Molde.me](https://molde.me/blog/boas-praticas-para-importar-moldes)), so nested PLT and PDF drawings apparently also circulate as pattern files in Brazil.
- **Print shops plot full-size PDF** on A0 (84.1 x 118.9 cm) or rolls; customers ask for "tamanho real (100%)" and "não ajustar à página", then measure a test square ([NS Moldes](https://nsmoldes.com.br/guia-de-impressao-de-moldes-em-pdf-a4-e-plotter-a0/)). A PDF page is limited to 200 x 200 inches (5.08 m) unless PDF 1.6's `UserUnit` is used ([Adobe PDF Reference 1.7, Appendix C](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/pdfreference1.7old.pdf)).
- **Bureaus rarely publish formats or prices.** Freedom Atelier (Flores da Cunha, RS) plots markers "de qualquer sistema que gere arquivos em BPS e HPHL2" (HP-GL/2, misspelt; BPS unidentified) on a pen plotter and delivers markers "na largura de até 183cm" ([Freedom Atelier](http://www.modelagem.ind.br/servicos), undated). Mold&Cad (Brás, São Paulo) sells plotting, marker making, grading and digitizing without listing formats or prices ([plotagem](https://moldecad.com.br/plotagem/), [encaixe](https://moldecad.com.br/encaixe/)). No per-metre prices were published; they need quotes.

**Assessment:** PDF is the easiest output to check (the owner can open it and measure it) and is what print shops take; PLT is what apparel plotters and bureaus take. Both come from the same drawing code. Start with the basic HP-GL commands above and test with the chosen bureau.

## 5. What a production-ready pattern file must contain

Sources: the standard's content list ([preview §4.1](https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf)), the "elementos do molde" in a [UDESC handout (2017)](https://www.udesc.br/arquivos/ceart/id_cpmenu/3787/Apostila_Modelagem_Avan_ada___2017_15206212782182_3787.pdf), the CAD pattern data in an [IFSC handout (2008)](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf), and the pattern properties in the [Audaces Moldes v10 manual](https://pdfcoffee.com/manual-digital-audaces-moldes-vs10pdf-pdf-free.html).

| Content | Portuguese | In DXF-ASTM | In PLT or PDF |
|---|---|---|---|
| Piece name | nome da peça | `Piece Name:` in the block (required; ASCII; 20 characters for Gerber) | Printed label |
| Style reference | referência do modelo | `Style Name:` | Label |
| Size | tamanho | `Sample Size:` plus the RUL `SIZE LIST`, or `Size Name:` per block in a graded nest | Label on every piece |
| Quantity to cut, left/right pairs | quantidade, cortar 2x (par) | `Quantity: R,L` | Label, for example "Cortar 2x" |
| Cut line with seam allowance | linha de corte, margem de costura | Layer 1 closed polyline | Outer line |
| Sew line | linha de costura | Layer 14 | Dashed inner line |
| Grain line | fio | Layer 7 | Line with arrows |
| Notches | piques | Layer 4 or 80 to 83 points (or cut into the outline) | Short lines or V shapes |
| Drill holes | furos | Layer 13 points | Small circles or crosses |
| Internal lines: darts, pleats, buttonholes | pences, pregas, casas | Layer 8 | Drawn lines |
| Fold line | dobra | Layer 6 on a half piece, or a full piece | Line labelled "dobra" |
| Centre front and back | centro da frente, centro das costas | Internal line or annotation | Label |
| Material | tecido, forro, entretela | `Material:` | Label |
| Units and scale | | `Units: METRIC` (mm, 2 decimals) | Exact 1:1 scale, test square |
| Grading | gradação | RUL with `# n` identifiers, or a graded nest | All sizes nested, or one file per size |
| Date and author | data, modelista | `Creation Date:`, `Author:` | Label |

The file format fixes where these go, not their values. How wide the allowances are, which notch goes where and where drill holes sit are pattern-making decisions for the pattern maker.

## 6. Marker making (encaixe): who does it

- A marker is paper at the fabric width and the cutting table's usable length, carrying the outlines and markings of every piece for the sizes and models being cut ([IFSC](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf)).
- It is prepared in the cutting room by the *riscador*, whose supervisor receives production orders from production planning ([IFSC](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf)).
- In CAD, "com a gradação pronta o operador indica a grade e a largura do tecido", and the marker is then plotted at full size ([IFSC](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf)). Marker software needs the fabric width, lay length, flat or tubular fabric, grain direction and the quantity of each size ([Audaces Encaixe v10 manual](https://pdfcoffee.com/manual-digital-audaces-encaixe-vs10-pdf-free.html)).
- After the sample is approved and graded, "a modelagem completa segue para a sala de corte" ([UDESC](https://www.udesc.br/arquivos/ceart/id_cpmenu/3787/Apostila_Modelagem_Avan_ada___2017_15206212782182_3787.pdf)).
- The DH guide lists complete marker laying and spreading information among the functions D6673 does not support ([DH guide](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)).
- Bureaus sell marker making as a separate service ([Mold&Cad](https://moldecad.com.br/encaixe/), [Freedom Atelier](http://www.modelagem.ind.br/servicos)).
- For sample garments, pieces are laid out by hand from full-size pattern pieces: "Utilizado para peças piloto" ([IFSC](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf)).

**Assessment:** marker making belongs to the factory or its bureau, and v1 cannot do it well without their production data. Keep it out of scope. v1 still needs a simple, non-optimized placement of pieces within a chosen width to produce the PDF and PLT files.

## 7. Libraries and tools that can write these formats

| Tool | License | What it writes | Fit with ADR-0001 | Notes |
|---|---|---|---|---|
| [ezdxf](https://github.com/mozman/ezdxf) 1.4.4 ([PyPI](https://pypi.org/project/ezdxf/), May 2026) | MIT | DXF R12 and R2000 to R2018; Release 13 and 14 input is saved as R2000 ([docs](https://ezdxf.readthedocs.io/en/stable/introduction.html)) | Yes | No apparel logic: we add the layers, system text, notch points and RUL. A throwaway check wrote every entity needed as handle-free R12 (`$HANDLING` set to 0), with notch depth in Z, width in code 39 and angle in code 50 |
| [dxfjs/writer](https://github.com/dxfjs/writer) | MIT | DXF with blocks, from TypeScript | Yes | Its header code writes AC1021, Release 2007 ([header.ts](https://github.com/dxfjs/writer/blob/next/src/header/header.ts)); I saw no R12 option; importer acceptance unknown |
| [js-dxf](https://github.com/tarikjabiri/js-dxf) | MIT | Points, polylines, text, layers | Yes | README lists no blocks, so it cannot hold one piece per block |
| [Valentina](https://gitlab.com/smart-pattern/valentina) | GPL-3.0 ([license](https://gitlab.com/smart-pattern/valentina/-/blob/develop/LICENSE_GPL.txt)) | DXF-AAMA and DXF-ASTM with all notch types, drill holes, turn and curve points and validation layers; compatibility modes; PLT (HP-GL, HP-GL/2); full and tiled PDF ([exporter](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vlayout/vlayoutexporter.cpp)) | Allows commercial use; copyleft applies if we ever distribute it | Command-line export ([vcmdexport.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/app/valentina/core/vcmdexport.cpp)) works only on patterns built in Valentina's own format; no rule table or graded nest writer found, so each export is one size |
| [Seamly2D](https://github.com/FashionFreedom/Seamly2D) | GPL-3.0 | DXF-AAMA only; ASTM entries are marked "For now not supported" ([source](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/app/seamly2d/mainwindowsnogui.cpp)); no drill holes or grading | Same as Valentina | The request to replace AAMA with ASTM is still open ([issue](https://github.com/FashionFreedom/Seamly2D/issues/59)) |
| libdxfrw | GPL-2.0-or-later ([header](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vdxf/libdxfrw/libdxfrw.h)) | Low-level DXF | Same as Valentina | Used inside Valentina and Seamly2D |
| [vpype](https://github.com/abey79/vpype) | MIT | HP-GL from vector drawings, with custom plotter profiles ([docs](https://vpype.readthedocs.io/en/latest/cookbook.html)) | Yes | Built for pen plotters; HP-GL is simple enough to write directly |
| [reportlab](https://pypi.org/project/reportlab/) | BSD | PDF (Python) | Yes | |
| [jsPDF](https://github.com/parallax/jsPDF), [pdf-lib](https://github.com/Hopding/pdf-lib) | MIT | PDF (JavaScript) | Yes | pdf-lib's repository was last pushed in July 2024 |
| [PyMuPDF](https://github.com/pymupdf/PyMuPDF) | AGPL-3.0 | PDF | Avoid | AGPL's network clause makes it risky for a hosted web app |

**Assessment on the GPL tools:** GPL-3.0 permits commercial use, so Valentina and Seamly2D pass the letter of ADR-0001. The cost is copyleft if the product is ever distributed rather than hosted, and adopting Valentina would mean building every pattern in its format. Both are better used as reference implementations (reading their code is fine) than embedded. This is a licensing judgment, not legal advice.

## 8. Recommendations for the decision tickets

All of this section is **Assessment**.

**Choose the export route**
1. One DXF writer with two flavours, ASTM by default and AAMA as a compatibility switch (as Valentina does): minimal R12-style ASCII, millimetres, no entity handles, ASCII-only text. Offer both a graded nest and a base size plus RUL, and pick the default after the first import trial in the pattern maker's CAD.
2. Plotting: full-size vector PDF first (exact 1:1 scale, a 10 cm test square, a label on every piece), PLT second (basic HP-GL). Place pieces within a width chosen per job (A0, or the bureau's roll width, such as 183 cm).
3. Libraries: ezdxf for a Python backend, otherwise a small custom writer (the format has only a few entity types); RUL and HP-GL written by hand; PDF with reportlab or jsPDF. Do not embed Valentina or Seamly2D.
4. Buy ASTM D6673-10 (USD 86) before implementing, to read the notch, drill hole and rule table sections first-hand.
5. Put marker making (encaixe) out of scope.

**Choose the grading approach**
- If each size is drafted from its own measurements (which suits plus sizes drafted for different proportions), the graded nest is the natural output and the only one that delivers each size's exact curves. A RUL can still be derived (each size's point minus the base size's point), but both forms need every size to keep the same points in the same order, so the pattern generator must keep one point structure per piece across sizes.
- A plus-size range with a different construction, such as an extra dart, has to be a separate style file.

**Define the production-ready acceptance test**
- Automated checks can come straight from the standard's rules:
  - every block has a closed layer 1 polyline and a `Piece Name`;
  - text is ASCII only;
  - layers 5, 6, 7, 9, 10 and 13 hold no polylines;
  - turn and curve points sit on vertices;
  - notch points sit on the edge;
  - every `# id` has a rule, and the base size's offsets are 0,0;
  - graded nest blocks have matching point counts and order;
  - PDF and PLT drawings match the model at 1:1.
- The decisive check is an import into the pattern maker's CAD (probably Audaces), comparing seam lengths, notches and sizes, plus a plotted PDF with its test square measured.

**Garment details v1 must support**
- The file can carry slit, V, T, castle, check and U notches, drill holes with diameters, internal lines (darts, pleats, buttonholes), fold lines, and stripe or plaid matching lines. Each detail v1 supports needs a mapping onto these layers.

## 9. Confidence and gaps

**Solid:**
- The ASTM layer table, system text, grade rule identifiers and graded nest rules come from the standard's own text.
- Withdrawal and prices come from ASTM, BSI and resellers.
- The notch and RUL encodings agree across the DH guide, Valentina's code and two real exports.
- The Gerber, GRAFIS, HP and Adobe facts come from first-party documents.

**Unverified or weak:**
- ASTM D6673-10 pages 5 to 10 (notch, drill hole and rule table sections) were read only through the DH guide.
- The AAMA-292 text and its exact layer list were not found, and its 2002 ANSI withdrawal date could not be opened.
- Audaces evidence comes from the old version 10 manual and blog posts. How today's Audaces imports graded nests, RUL files and typed notches is untested.
- CLO, Lectra, Optitex and RZ CAD import behaviour could not be checked (bot protection or customer logins).
- Brazilian bureau coverage is thin: two bureaus and print-shop guides, with no published prices.
- Whether handle-free R12 files from ezdxf import cleanly into Audaces is untested.

## Sources

Standards and references:
- ASTM D6673-10 preview, pages 1 to 4: https://www.normsplash.com/Samples/ASTM/191361149/ASTM-D6673-10-en.pdf
- ASTM store: [D6673-10](https://store.astm.org/d6673-10.html), [D6673-04](https://store.astm.org/d6673-04.html), [D6673-01](https://store.astm.org/d6673-01.html), [D6963-04](https://store.astm.org/d6963-04.html)
- [BSI Knowledge, D6673-10](https://knowledge.bsigroup.com/products/standard-practice-for-sewn-products-pattern-data-interchange-data-format); [SIS](https://www.sis.se/en/produkter/external-categories/textiles-astm-vol-07/textiles-ii-d4393--latest-astm-vol-0702/astmd6673); [MyStandards](https://www.mystandards.biz/standard/astm-d6673-10-1.1.2010.html)
- [NISTIR 5969, Y. Tina Lee, NIST, January 1997](https://archive.org/stream/datasharingimple5969leey/datasharingimple5969leey_djvu.txt)
- [ANSI Standards Action 2002, SAV3341](https://share.ansi.org/Shared%20Documents/Standards%20Action/2002%20PDFs/SAV3341.pdf) (not readable, bot-protected); [AAFA, Who We Are](https://www.aafaglobal.org/AAFA/About/Who_We_Are.aspx)
- [DH Patterns and Fit, ePattern ASTM Standard, 2011](https://dorthehansen.com/wp-content/uploads/2014/10/ePattern-ASTM-Standard.pdf)
- [HP-GL/2 Vector Graphics, HP](http://www.hp.com/ctg/Manual/bpl13211.pdf); [Adobe PDF Reference 1.7](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/pdfreference1.7old.pdf); [AutoCAD R12 DXF Reference](https://damassets.autodesk.net/content/dam/autodesk/www/developer-network/platform-technologies/autocad-dxf-archive/acad_r12_dxf.pdf)

Vendors and Brazilian practice:
- Audaces: [Moldes v10 manual](https://pdfcoffee.com/manual-digital-audaces-moldes-vs10pdf-pdf-free.html) ([second copy](https://idoc.pub/documents/manual-digital-audaces-moldes-vs10pdf-vnd50m8g2wlx)), [Encaixe v10 manual](https://pdfcoffee.com/manual-digital-audaces-encaixe-vs10-pdf-free.html), [Pattern product page](https://audaces.com/en/solutions/pattern), [Plotter Audaces](https://audaces.com/pt-br/blog/plotter-audaces), [Plotter de risco](https://audaces.com/pt-br/blog/plotter-risco); [ND+ on the company's history](https://ndmais.com.br/moda-e-beleza/audaces-a-empresa-de-florianopolis-que-mudou-a-historia-da-moda-no-brasil/)
- Gerber AccuMark: [Data Conversion Utility](https://help.gerbertechnology.com/AccuMark/AMX/Function_DCU.htm), [Export](https://help.gerbertechnology.com/AccuMark/PDS/File_Export_Tab_File.htm)
- GRAFIS: [Export formats](https://help.grafis.de/en-gb/Content/ExportIImport/Exportformate_und_deren_Besonderheiten.htm), [Export steps](https://help.grafis.de/en-gb/Content/ExportIImport/Schrittfolge_fuer_den_Export.htm)
- Molde.me: [import](https://molde.me/ajuda/importar-moldes), [export](https://www.molde.me/ajuda/exportar-molde), [AAMA export post](https://molde.me/blog/exportacao-aama), [import practices](https://molde.me/blog/boas-praticas-para-importar-moldes)
- [Freedom Atelier de Modelagem](http://www.modelagem.ind.br/servicos); Mold&Cad: [plotagem](https://moldecad.com.br/plotagem/), [encaixe](https://moldecad.com.br/encaixe/); [Otimize Nesting](https://www.otimizenesting.com.br/post/2016/10/19/dxf-aama-e-astm); [NS Moldes print guide](https://nsmoldes.com.br/guia-de-impressao-de-moldes-em-pdf-a4-e-plotter-a0/)
- [IFSC, Tecnologia da Confecção, 2008](https://wiki.ifsc.edu.br/mediawiki/images/7/73/Apostila_tecnologia_cris.pdf); [UDESC, Modelagem Avançada do Vestuário Feminino, 2017](https://www.udesc.br/arquivos/ceart/id_cpmenu/3787/Apostila_Modelagem_Avan_ada___2017_15206212782182_3787.pdf)
- Real exports read as evidence (no license, not reused): [CLO 7.2 and Richpeace V9 files](https://github.com/martaquintana/2023-tfm-migjrv/tree/main/T-shirt%20patterns)

Open source:
- Valentina: [repository](https://gitlab.com/smart-pattern/valentina), [vdxfengine.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/vdxfengine.cpp), [libdxfrw.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/libdxfrw/libdxfrw.cpp), [dxfdef.h](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vdxf/dxfdef.h), [vlayoutexporter.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/libs/vlayout/vlayoutexporter.cpp), [vcmdexport.cpp](https://gitlab.com/smart-pattern/valentina/-/blob/develop/src/app/valentina/core/vcmdexport.cpp), [ChangeLog](https://gitlab.com/smart-pattern/valentina/-/blob/develop/ChangeLog.txt)
- Seamly2D: [repository](https://github.com/FashionFreedom/Seamly2D), [vdxfengine.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vdxf/vdxfengine.cpp), [dxiface.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/libs/vdxf/dxiface.cpp), [mainwindowsnogui.cpp](https://github.com/FashionFreedom/Seamly2D/blob/develop/src/app/seamly2d/mainwindowsnogui.cpp), [issue on ASTM export](https://github.com/FashionFreedom/Seamly2D/issues/59)
- [ezdxf](https://github.com/mozman/ezdxf) ([docs](https://ezdxf.readthedocs.io/en/stable/introduction.html)); [dxfjs/writer](https://github.com/dxfjs/writer); [js-dxf](https://github.com/tarikjabiri/js-dxf); [vpype](https://github.com/abey79/vpype); [reportlab](https://pypi.org/project/reportlab/); [jsPDF](https://github.com/parallax/jsPDF); [pdf-lib](https://github.com/Hopding/pdf-lib); [PyMuPDF](https://github.com/pymupdf/PyMuPDF)
