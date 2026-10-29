# exampleMetadata-xdi

The 55 XDI files in
[`XAS-CDIF/exampleData`](https://github.com/CDIF-4-XAS/XAS-CDIF/tree/main/exampleData),
converted by **this** pipeline.

The same files converted by the production RML pipeline are in
`XAS-CDIF/exampleMetadata`. Both sets are kept so the two
implementations can be compared on identical inputs — that comparison is
the point of this directory, and it is what
`cdif-xas-UKDS/CONVERGENCE-PROPOSAL.md` argues from.

Regenerate from a checkout of this repository with the XAS-CDIF corpus
alongside; see the repository README.

| | `exampleMetadata/` | this directory |
|---|---|---|
| Producer | `cdif-xas-UKDS` | `CDIF-4-XAS/cdifnexmetadata` |
| Mapping | RML (`mapping_dds.ttl`) run by rmlmapper | SSSOM crosswalk + Python |
| Runtime | FastAPI + Java + pyld | pure Python |
| Validates against `xasDocument` | 55 / 55 | **55 / 55** |

**`XAS-CDIF/exampleMetadata/` remains the reference output.** It is the
production pipeline, it is reviewed, and it validates completely. This
directory exists to show what the second implementation currently
produces from the same bytes, not to replace it.

## Why this was generated

The SSSOM crosswalk gained four rows (`Column.ifluor`, `Column.mufluor`,
`Column.murefer`, `Beamline.xray_source`) and the XDI binding gained
four derivations (probe, detection mode, reflection plane, d-spacing
units). None of that reaches the RML pipeline, which does not read the
crosswalk — there is no reference to SSSOM anywhere in `cdif-xas-UKDS`.
So regenerating `exampleMetadata/` would have shown no change at all,
and this is where the crosswalk work actually shows up.

## Sentinels for what the files omit

Two properties the profile requires are missing from parts of the
corpus: the Diamond B18 series (`262875_PtSn_*` and siblings) gives
`Mono.name` and no `Mono.d_spacing` at all, and files such as
`feo_rt1.xdi` write a source type under neither `Facility` nor
`Beamline`.

An omitted property makes the instrument undescribable and fails
validation, so both are emitted, each with a description saying it was
not recorded in the source file:

- a missing d-spacing is `:unav`, DataCite's "value unavailable";
- a missing source type is `Synchrotron X-ray Source`, because the
  technique supplies it: an XAS measurement is made at a synchrotron.
  The description says so, so a consumer can tell it from a value the
  beamline recorded.

## Reading the difference

Both directories validate 55/55, so the pass rate says nothing. The
difference is in what each says where a file is silent, and on these
properties the two now agree on the values:

| where the file omits | `exampleMetadata/` (RML) | this directory |
|---|---|---|
| `Mono.d_spacing` | `:unav` | `:unav`, with a description |
| a source type | `Synchrotron X-ray Source` | `Synchrotron X-ray Source`, with a description saying the technique supplied it |
| a reflection plane (read from `Mono.name`, e.g. `Si(311)`) | `3,1,1` | `3 1 1` |

The RML output once wrote `1,1,1` for every file, including the eight
whose `Mono.name` says `Si(311)`; both now read the reflection out of
`Mono.name`. What still differs is the description: this directory marks
a value it supplied, and the RML output does not, so there a default
cannot be told from a reading.
