# smart-modeling

A tool that turns a garment idea described in plain language into a 3D preview and a production-ready, graded pattern file.

## Language

### Patterns

**Pattern**:
The set of flat pieces a garment is cut from, with seam lines, notches and grain lines (Portuguese: modelagem).
_Avoid_: Mold, template

**Grading**:
Producing a pattern in every size of a size range, starting from one base size (Portuguese: gradação).
_Avoid_: Sizing, scaling

**Base size**:
The size a pattern is first drafted in, before grading (Portuguese: tamanho base).
_Avoid_: Sample size (the pattern-file standard's name for it, which clashes with **Sample**)

**Size chart**:
The body measurements for every size in a size range, which grading works from (Portuguese: tabela de medidas).
_Avoid_: Size table, measurement table

**Plus size**:
A size above the regular range, drafted for different body proportions rather than as a scaled-up regular size.

**Pattern file**:
The deliverable: a digital file holding a garment's pattern graded to every size, in a format that apparel CAD systems and plotters read.
_Avoid_: Technical file, tech file

**Production-ready**:
The quality bar for a pattern file: a sample can be cut and sewn straight from it, with no corrections by a pattern maker.

**Plotting**:
Printing a pattern at full size on a wide-format plotter, ready for cutting (Portuguese: plotagem).

**Tech pack**:
A production document with flat sketches, a measurement table per size, materials and sewing instructions; a different thing from the pattern file (Portuguese: ficha técnica).
_Avoid_: Technical file

**Pattern maker**:
A professional who drafts and grades patterns (Portuguese: modelista).

**Sample**:
A garment cut and sewn from a pattern file to check fit and construction before production (Portuguese: peça piloto).
_Avoid_: Prototype, toile

**Fit model**:
A person whose body matches one size of the size chart, who tries samples on at a fitting (Portuguese: modelo de prova).
_Avoid_: Mannequin (a dress form, which is not a person)

**Automated check**:
A test the tool runs on a pattern file at export. A hard error makes the pattern unsewable and blocks the export; a warning flags a quality issue and lets it through.
_Avoid_: Validation, lint

### Fabric

**Woven**:
Fabric made of interlaced threads that barely stretches, such as denim or linen (Portuguese: tecido plano).

**Knit**:
Fabric made of interlocking loops that stretches, such as t-shirt jersey; its patterns are drafted smaller than the body, by an amount set by the stretch (Portuguese: malha).

### Design

**Design**:
One garment the owner creates and keeps in the library, together with its brief, design spec and versions; a dress counts as one garment.
_Avoid_: Project, model, outfit

**Brief**:
The owner's own description of an intended garment: words in Portuguese or English, plus optional reference images.
_Avoid_: Prompt, description

**Design spec**:
The structured set of choices and measurements a pattern is drafted from; the tool fills it from the brief, and chat and controls edit it.
_Avoid_: Pattern parameters, design parameters

**Version**:
A saved state of a design; any version can be reopened or exported.
_Avoid_: Revision, snapshot

**Detail**:
A design feature of a garment within its family, such as a neckline, sleeve, collar, closure, pocket or pleat. A shape detail only changes the outline of existing pieces; a construction detail adds pieces, markings or sewing steps.
_Avoid_: Feature, option

**Proven**:
The status of a detail, in a fabric and size range, once the pattern maker has reviewed it and, for a construction detail, a sample has passed; a design is production-ready when all its details are proven and its file passes the automated checks. A later change to a proven piece's shape removes the status until it is reviewed again.
_Avoid_: Certified, approved

**Finish**:
How a raw edge such as a neckline, armhole, hem or waist is completed: a facing, binding, band or hem (Portuguese: acabamento).
_Avoid_: Edge treatment

**Garment family**:
A category of garments drafted with its own pattern logic, such as tops, skirts, pants or dresses.
_Avoid_: Category, garment type

**3D preview**:
The garment simulated on a virtual body, used to judge and adjust a design; it is not a deliverable.
_Avoid_: 3D model
