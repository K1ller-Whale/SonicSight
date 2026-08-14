# SonicSight: Requirements and Analysis Study

### Analysis Report

**Fill the table below before submission.**

| Field | Value |
|---|---|
| Course | <<< COURSE >>> |
| Institution | <<< INSTITUTION >>> |
| Supervisor | <<< SUPERVISOR >>> |
| Submission date | <<< SUBMISSION DATE >>> |

**Team members and contribution split.** Placeholder — to be completed by the
team before submission. No names are stated in this document until supplied.

| Team member | Contribution |
|---|---|
| <<< NAME >>> | <<< CONTRIBUTION >>> |
| <<< NAME >>> | <<< CONTRIBUTION >>> |

**Documented repository state.** Every statement in this document was verified
against the following repository heads, re-confirmed clean on 2026-08-08:

| Repository | Branch | Head | Last commit |
|---|---|---|---|
| SonicSightBackend | `main` | `9da8709119dc510497a4a5e5b77e47396cd0b358` | 2026-08-05 |
| SonicSightMobile | `master` | `cfbe64f096c85f3902535f7c98793e537df85c77` | 2026-08-05 |
| multisensory | `master` | `424b85bdded0e2f7e860640511f82bd37d7d0513` | 2026-08-04 |

These are the same heads documented by the companion technical report
(`docs/TECHNICAL_REPORT.md`, front matter). Where this document and the
technical report describe the same fact, they describe the same working trees.

**Document status.** This document is being produced in phases. Sections 1–9
and §10.1–10.2 (with Appendix B, Figures 1–5, `docs/nfr/nfr_targets.yaml`,
the eight ADRs in `docs/adr/`, and the load suite in
`SonicSightBackend/loadtest/`) are complete. The Phase 2 thresholds were set
before the Phase 4 suite was built, and the suite has **not been run**:
targets demonstrably precede results, which arrive in Phase 5 (§10.3).
Sections marked *(pending — Phase N)* are placeholders whose planned content
is stated but not yet written.

---

## Contents

1. [Introduction](#1-introduction)
2. [Project context and scope](#2-project-context-and-scope)
3. [Stakeholder analysis](#3-stakeholder-analysis)
4. [Functional requirements](#4-functional-requirements)
5. [Non-functional requirements](#5-non-functional-requirements)
6. [Use case model](#6-use-case-model)
7. [Scope gap analysis](#7-scope-gap-analysis)
8. [Architecture and design rationale](#8-architecture-and-design-rationale)
9. [Verification and validation strategy](#9-verification-and-validation-strategy)
10. [Backend performance verification](#10-backend-performance-verification) *(§10.3 measured results pending — Phase 5)*
11. Mobile compatibility verification *(pending — Phase 6)*
12. Risk analysis *(pending — Phase 7)*
13. Ethical and privacy considerations *(pending — Phase 7)*
14. Conclusions and future work *(pending — Phase 7)*

References *(initial; completed in Phase 7)*
Appendix A — Requirements traceability matrix *(pending — Phase 7)*
[Appendix B — Use case specifications](#appendix-b--use-case-specifications)
Appendix C — Measured results *(pending — Phase 5)*
Appendix D — Evidence for claims in this document *(initial)*
Appendix E — Glossary *(initial)*

**Figures.**
Figure 1 — System context with actors and environments (§2.3).
Figure 2 — Stakeholder map (§3).
Figure 3 — Use case diagram, end-user cases (§6).
Figure 4 — Use case diagram, operator and developer cases (§6).
Figure 5 — Architectural decisions mapped to the NFRs they serve (§8.11).

---

## 1. Introduction

### 1.1 Purpose

This document is the requirements and analysis study for the SonicSight
project. It states what the software is required to do and how well, who has
an interest in it, which of those requirements the delivered system satisfies,
and how each claim of satisfaction is verified. It exists so that the project's
non-functional properties are *verifiable rather than asserted*: every
quantitative requirement in Section 5 carries a metric, a threshold, a
statistic, and a measurement condition, and Section 10 will report
measurements against those thresholds from a committed test suite.

### 1.2 Scope of this document

The study covers the three repositories named in the front matter: the Android
client, the Python backend, and the migrated multisensory model repository. It
covers requirements engineering (Sections 4–6), the scope gap analysis
(Section 7), architecture rationale (Section 8), verification
(Sections 9–11), and risk, ethics, and privacy (Sections 12–13). It does not restate the system's internal design; that is
the technical report's job (§1.4).

### 1.3 Intended audience

The academic supervisor and examiners assessing the project; the project team;
and any future maintainer who needs to know not just what the system does but
what it was *required* to do and which of those requirements are demonstrated.

### 1.4 Relationship to the technical report

`docs/TECHNICAL_REPORT.md` (cited throughout as "TR") documents the system as
built: architecture, protocol, pipeline, measurements, and limitations. This
document is its requirements-side companion. The two are deliberately
consistent: both describe the same repository heads, both use the same
three-label status taxonomy (§1.5), and where this document cites a measured
number it cites the same record the TR cites. Where a discrepancy between the
two is discovered, it is flagged in the text rather than silently resolved.

### 1.5 Conventions

**Status taxonomy.** This document reuses the TR's three capability labels
(TR §1.4): **implemented and validated** (the code runs and a recorded
measurement or test result exists), **implemented, not validated** (the code
exists and builds but has not been run end to end under the conditions it was
designed for), and **designed, not implemented** (a plan exists, no code).
For *requirements* — which, unlike capabilities, can be contradicted by
implemented behaviour — Sections 4–5 extend the taxonomy with two labels:
**partially satisfied** (the behaviour holds with a recorded exception) and
**not satisfied** (the implemented behaviour contradicts the requirement; a
finding, carried into Section 12). Planned work is never described in the
present tense.

**Evidence discipline.** Code is ground truth; project documentation is
treated as stale until confirmed against the working trees. Claims about the
system carry `file:line` evidence, collected in Appendices B and D and in
the Evidence sections of the Architecture Decision Records rather than
inline, so the body reads as prose but remains checkable. No measured number
is stated without its conditions; every measurement condition resolves to a
named reference environment in §2.3.

**Requirement identifiers.** Functional requirements are numbered `FR-###`
(in subsystem blocks, §4.0) and non-functional requirements
`NFR-<CATEGORY>-###` (Section 5); both sets were assigned in Phase 2 and are
stable — identifiers never change once assigned. Use cases are numbered
`UC-##` (assigned in Phase 1); Appendix B's *Requirements exercised* rows
reference the Section 4–5 identifiers.

**Placeholders.** Text in `<<< >>>` is a deliberate placeholder for
information only the team can supply. Nothing in `<<< >>>` is invented.

**Acronyms** are defined at first use and collected in Appendix E.

---

## 2. Project context and scope

### 2.1 Problem domain and motivation

A microphone records a single mixture of every sound source in a scene; a
listener who needs one source — one speaker among several, one instrument in
an ensemble — receives all of them summed. Audio-visual source separation uses
the video of the scene as a conditioning signal to decompose that mixture:
pixels indicate which sources are present and where, and a neural network uses
that indication to assign time-frequency energy to per-source output tracks.

SonicSight is a working end-to-end system built on two published,
pretrained models of this kind. An Android handset captures synchronized video
and audio and streams them over a local network to a Python server, which runs
one of two separation models and streams back separated audio tracks and a
spatial map of where the model attributes the sound; the handset plays the
tracks and renders the map as a live overlay on the camera preview. Three
selectable listening modes are deployed on this machinery (TR §3.5): a
music/instrument mode that separates the left and right halves of the scene
(Sound of Pixels [1]), a speech mode that separates on-screen from off-screen
sound (the multisensory model [2]), and a touch mode in which the user taps a
region of the scene to hear it in isolation (Sound of Pixels features, reused).

The motivation for studying this particular system is assistive: people who
use hearing aids, and d/DHH (deaf and hard-of-hearing) individuals more
broadly, are the population for whom "isolate the source I am looking at" is
not a convenience but the difference between following a conversation and
losing it. Whether visually-guided separation *as it exists in published
research today* could plausibly serve that population — at what latency, at
what quality, on what hardware — is a question that can only be answered by
building and measuring, which is what this project did.

### 2.2 Case-study framing

**This project is a case study, not a product.** It is a feasibility
investigation into whether visually-guided audio-visual source separation
could support people who use hearing aids or are d/DHH. It is not an assistive
device, not a clinical tool, and not a shippable application. This framing is
structural in this document: every requirement in Sections 4–5 is scoped
either **in scope (case study)** or **out of scope (would be required of a
product)**, and Section 7 assembles the second category into a gap analysis.

**Objectives.** The case study's objectives, against which its requirements
are written:

- **O1** — Demonstrate an end-to-end live pipeline: handset capture, network
  transport, server-side neural separation, and return playback with a
  localization overlay, on commodity hardware.
- **O2** — Integrate two architecturally different pretrained models
  (PyTorch and TensorFlow) behind one selection mechanism, both resident in
  one process on one consumer GPU.
- **O3** — Measure the pipeline honestly: latency decomposed into its causes,
  memory under stated conditions, and correctness checks designed to fail
  when the system is wired wrong.
- **O4** — Establish, from those measurements, how far the current state of
  this technique is from live conversational assistance, and identify what
  bounds it (Section 2.2.3).
- **O5** — Leave a maintainable record: a registry contract for adding
  models, deterministic validation harnesses, and documentation traceable to
  code.

**Explicit non-goals.** Stated to prevent scope creep in the requirements and
to be honest with the reader; each reappears in Section 7 as a product gap:

- No model training, fine-tuning, or dataset collection was performed; both
  checkpoints are pretrained artefacts from published research (TR §1.3).
- No formal user study was conducted, and no participant from the prospective
  beneficiary population has used the system (TR §1.3).
- No evaluation against a labelled separation benchmark was performed; the
  system reports no signal-to-distortion-ratio numbers of its own (TR §10.6).
- No transport security or authentication: the channel is plaintext on a
  trusted LAN by deliberate posture (TR §12.1).
- No real-time conversational assistance is attempted or claimed (§2.2.3).

#### 2.2.1 This is not a medical device

SonicSight is not a medical device and makes no clinical claim. It has not
been evaluated for safety, efficacy, or benefit to any person with hearing
loss, and nothing in this document or the software asserts otherwise.

The boundary matters because assistive hearing devices are regulated products
in most jurisdictions: in the European Union, hearing aids fall under the
Medical Device Regulation (EU MDR 2017/745); in the United States, hearing
aids — including the over-the-counter category created in 2022 — are regulated
by the Food and Drug Administration (FDA) under 21 CFR Part 800 ⟦regulatory
citations to be verified against primary sources in the Phase 7 references
pass⟧. A product intended to compensate for hearing impairment would sit
inside those regimes and carry their conformity, clinical-evaluation, and
post-market obligations. This case study sits entirely outside them: it is a
laboratory investigation of a signal-processing technique, run by its own
developers, on its own developers. Section 7 records what entering the
regulated space would require; Section 12 records the risk of a user treating
the prototype as more than it is.

#### 2.2.2 What the two models actually do

Neither model was designed or validated for hearing assistance, and this
document does not stretch them:

- **Sound of Pixels** [1] separates **musical instruments**; its published
  training and evaluation are on the MUSIC dataset of instrument
  performances. In this system it powers the "Music & Instruments" halves
  mode and the touch mode.
- **The multisensory model** [2] separates **on-screen from off-screen
  speech**; its separation head is fine-tuned on VoxCeleb. It powers the
  "Speech" mode. It is the only component even adjacent to hearing
  assistance, and its published validation is source separation quality, not
  assistive benefit.

The SonicSight working trees do not record the provenance of the Sound of
Pixels checkpoint (TR §15.2 flags the attribution gap); the MUSIC-dataset
attribution above is from the published paper, not from the checkpoint.

#### 2.2.3 The latency reality, stated up front

The system's estimated end-to-end perceived lag is **≈ 1.2 s** in the speech
mode and **≈ 3.0 s** in the music and touch modes. Both figures are
*estimates* — half the analysis window plus measured or budgeted inference
time — and the TR labels them "estimate, not a measurement" (TR Table 12);
no stopwatch measurement exists yet. The dominant term is not engineering
overhead but the analysis window each model requires: 5.94 s of audio for
Sound of Pixels, 2.10 s for the multisensory model, of which half is
unavoidable lookahead by construction.

Hearing-device processing delay tolerances reported in the audiology
literature are on the order of tens of milliseconds at most. The Stone and
Moore *Tolerable Hearing Aid Delays* series reports disturbance thresholds
in the range of roughly 5 to 30 ms depending on condition — own-voice
disturbance beyond about 15–20 ms, disturbance from as little as 5–6 ms
under some gain conditions, and a commonly cited working bound of ≈ 10 ms
for acceptable processing delay [5][6] ⟦per-condition attribution to
individual series parts to be pinned in the Phase 7 references pass⟧.
Audio-visual desynchrony tolerances are looser but still far below one
second: ITU-R BT.1359-1 puts the detectability of audio lag against vision
at about 125 ms (45 ms for audio lead) and acceptability at about 185 ms
lag / 90 ms lead [7]. Against the hearing-aid tolerances, the system's
estimated lags of ≈ 1.2 s and ≈ 3.0 s sit between roughly **60×** (1.2 s
against the most lenient ≈ 20 ms bound) and **600×** (3.0 s against the
strictest ≈ 5 ms bound) — **two to three orders of magnitude** from live
conversational assistance; even against the far looser broadcast
audio-visual acceptability bound of 185 ms the lags are ≈ 6–16× over. The
gap is imposed by the models' window requirements, not by the
implementation.

This is the study's central feasibility finding, and it is stated here as
one rather than buried as a limitation: the architecture demonstrates that
visually-guided separation runs on commodity hardware with a phone as the
capture device; the window length required by both models is what puts
real-time assistance out of reach today. The contribution of this work is
positioned there — on demonstrated system feasibility and an honest account
of the latency wall — not on any claim of assistive benefit.

### 2.3 Reference environment

Every measured figure in this document refers to one of the named
environments below. Requirements in Section 5 cite an environment identifier
(E-M, E-D, E-A, E-N) instead of restating conditions. Facts marked *probed
2026-08-08* were re-verified on the current machine during the preparation of
this document; the remainder are quoted from the project's measurement records
(HANDOFF §6.1; TR §9.3, §14.1).

#### E-M — Measurement environment (GPU performance record)

The environment behind every GPU latency, VRAM, and numerical-equivalence
figure in the project record (TR Tables 13–14, §10.3–10.4; recorded
2026-08-03).

| Item | Value |
|---|---|
| GPU | NVIDIA GeForce GTX 1660 Ti, 6 GB (Turing TU116, no tensor cores) |
| Driver / CUDA | 566.14 (CUDA 12.7) |
| Host OS | Windows; GPU runs measured inside WSL2 |
| WSL distribution | Ubuntu 26.04 LTS |
| Python | 3.12 (via uv) |
| TensorFlow | 2.21.0 with the `[and-cuda]` extra, `TF_USE_LEGACY_KERAS=1` |
| Repository location | `~/multisensory` on the WSL ext4 filesystem (not `/mnt/d`) |
| Workload | Multisensory separation checkpoint, 2.135 s window, 44 144 samples at 21 000 Hz, 63 frames; a single video clip, same clip and start offset across runs |

**Availability caveat.** E-M is not the machine this document was prepared
on. Its continued availability is unverified, and no figure recorded on E-M is
reproducible on E-D as configured (different GPU, TensorFlow absent). Any
Phase 5 measurement that requires the multisensory engine requires either
access to E-M or an equivalent provisioned environment, and will say which it
used.

**Unrecorded-conditions caveat.** The Sound of Pixels streaming measurements
in the project record (mask renormalisation and audio-normalisation effects,
TR Table 7) predate the environment discipline above; their hardware and date
are **not recorded** (TR Table 1, item 1). They are the reason this document
insists on named environments for everything measured from Phase 5 onward.

#### E-D — Documentation and unit-test environment (current machine)

The machine this document was prepared on; also the machine of the TR's
recorded unit-suite run (66 passed, 1 skipped, 2 collection errors,
2026-08-05, TR §10.5) and the intended host for the Phase 4–5 load suite.
All rows probed 2026-08-08.

| Item | Value |
|---|---|
| Host OS | Windows 11 Pro, build 10.0.26200 |
| CPU / RAM | Intel Core i7-9750H, 16 GiB |
| GPU | NVIDIA GeForce GTX 1650, 4 GB (WDDM) |
| Driver | 595.97 (driver-supported CUDA up to 13.2) |
| Python | 3.11.9 |
| PyTorch | 2.6.0+cu124 |
| NumPy / grpcio / protobuf | 2.3.5 / 1.78.0 / 6.33.6 |
| TensorFlow | **not installed** |
| WSL distribution | Ubuntu 24.04.1 LTS (no multisensory environment present; no `.wslconfig`, mirrored networking not configured) |
| ffmpeg / Node.js / mermaid-cli | 8.0.1 / 22.17.0 / 11.16.0 |
| Checkpoints on disk | Sound of Pixels `src/ckpt/`: 121 145 339 + 45 356 134 + 652 bytes; multisensory `results/nets/sep/full/net.tf-160000`: 1 364 209 708 + 13 016 751 + 25 545 bytes — byte-identical to the sizes the TR records (TR §9.4) |

**Consequence for Phase 5.** With TensorFlow absent and a 4 GB GPU, E-D can
exercise the `sonicsight` and `sonicsight-pixel` engines but not the
`multisensory` engine. The load suite will therefore measure the PyTorch
modes on E-D and state plainly which multisensory-dependent thresholds remain
unmeasured unless an E-M-equivalent environment is provisioned.

#### E-A — Android device environment

**Not recorded — a known gap.** No document in the three repositories names
the handset model or Android version used in any live session, including the
2026-08-05 touch-mode calibration session whose constants now ship in the
code. What is known is what the code requires and configures:

| Item | Value (from code, not from a device record) |
|---|---|
| SDK levels | minSdk 28, targetSdk 36, compileSdk 36 |
| Camera | CameraX 1.3.1, back camera, `ImageAnalysis` target 1280×720, keep-only-latest |
| Audio capture | `AudioRecord` at 44 100 Hz mono PCM16, decimated in-app to the wire rate (11 025 or 22 050 Hz) |
| Orientation | `sensorLandscape`, locked |
| Permissions | CAMERA, RECORD_AUDIO, INTERNET |

The Phase 6 mobile verification plan will begin by defining the target
device population precisely because this row of the reference environment is
currently empty. `<<< HANDSET MODEL AND ANDROID VERSION USED IN LIVE
SESSIONS — to be supplied by the team >>>`

#### E-N — Network environment

The transport is gRPC bidirectional streaming, plaintext, port 50051, over a
local Wi-Fi network; the client's compiled-in default server address is a
private LAN address, overridable in-app. Maximum message size 16 MB; client
keepalive 30 s. **The physical network conditions (Wi-Fi band, access point,
topology, competing traffic) behind the project's throughput estimates are
not recorded** — a gap the Phase 4 load suite will close by capturing
network metadata with every run, and the Phase 6 plan will address by
injecting controlled latency, jitter, and loss.

**Figure 1 — System context: actors, client, server, environments.**
The bystander is dashed: present in the captured field, party to no
interaction. (Source: `docs/diagrams/ar-figure-01-system-context.mmd`;
rendered: `docs/diagrams/svg/ar-figure-01-system-context.svg`.)

```mermaid
flowchart LR
    LST(("Listener"))
    BYS(("Bystander"))
    OPR(("Operator"))
    DEV(("Developer"))

    subgraph HANDSET["Android handset — environment E-A"]
        APP["SonicSight client<br/>CameraX capture, AudioRecord 44100 Hz,<br/>overlay, dual AudioTrack playback"]
    end

    subgraph SERVER["Server host — environments E-M / E-D"]
        GRPC["gRPC server :50051<br/>asyncio, plaintext"]
        REST["FastAPI :8000<br/>legacy /predict"]
        REG["Model registry<br/>sonicsight / sonicsight-pixel / multisensory"]
        GPU["NVIDIA GPU<br/>PyTorch + TensorFlow, one process"]
        CKPT[("Checkpoints<br/>Sound of Pixels 158.8 MiB<br/>multisensory 1.27 GiB")]
    end

    LST --- APP
    BYS -. "captured incidentally<br/>(camera and microphone field)" .-> APP
    APP <-- "StreamChunk / StreamResult<br/>gRPC bidirectional, LAN" --> GRPC
    GRPC --> REG
    REG --> GPU
    GPU --- CKPT
    OPR --- GRPC
    OPR --- REST
    DEV --- REG

    classDef incidental stroke-dasharray: 5 5;
    class BYS incidental;
```

---

## 3. Stakeholder analysis

The stakeholders of a study differ from the stakeholders of a product, and
conflating them would repeat the exact overstatement §2.2 exists to prevent.
The distinction drawn here: *direct participants* interact with the project
today; *prospective beneficiaries* are the population the feasibility
question is about, who have had no involvement in this study; *incidentally
affected parties* are exposed to the system without choosing to be.

**Figure 2 — Stakeholder map.** Dashed nodes have no direct relationship
with the project today. (Source:
`docs/diagrams/ar-figure-02-stakeholder-map.mmd`; rendered:
`docs/diagrams/svg/ar-figure-02-stakeholder-map.svg`.)

```mermaid
flowchart TB
    CS["SonicSight case study"]

    subgraph DIRECT["Direct participants"]
        TM["Project team<br/>(develops, operates, listens)"]
        SUP["Academic supervisor"]
        EXAM["Institution and examiners"]
    end

    subgraph BENEF["Prospective beneficiaries — no direct involvement in this study"]
        HA["People who use hearing aids"]
        DHH["d/DHH individuals"]
    end

    subgraph INCID["Incidentally affected"]
        BY["Bystanders in the captured<br/>audio and video field"]
    end

    subgraph UPSTR["Upstream parties"]
        AUTH["Authors of the two models<br/>(attribution, licensing)"]
        DATA["Subjects of the training datasets<br/>(AudioSet, VoxCeleb, MUSIC)"]
    end

    subgraph BOUND["Out-of-scope oversight"]
        REG["Medical device regulators<br/>(boundary only — no claim made)"]
    end

    TM --- CS
    SUP --- CS
    EXAM --- CS
    CS -. "findings may inform<br/>future assistive work" .-> HA
    CS -. "findings may inform<br/>future assistive work" .-> DHH
    CS -- "capture without<br/>consent mechanism" --- BY
    AUTH --- CS
    DATA -.- CS
    REG -. "scope boundary,<br/>not a relationship" .- CS

    classDef outside stroke-dasharray: 5 5;
    class HA,DHH,REG,DATA outside;
```

**Table 3.1 — Stakeholders and their interests.**

| # | Stakeholder | Relationship | Interests and needs | Where addressed |
|---|---|---|---|---|
| S1 | People who use hearing aids; d/DHH individuals | Prospective beneficiaries. **No member of this population has used the system**; no user study was conducted (TR §1.3). | An honest feasibility verdict, stated without overclaim: what latency and separation quality the technique achieves today and what would have to change before it could help them. Protection from the implication that an unvalidated prototype is assistive. | §2.2.3 (finding), §7 (gaps), §12 (over-reliance risk) |
| S2 | Project team | Direct: develops, operates, and — to date — is the only listener population. Dual role as developer and test subject is itself a validity limitation. | A system that can be measured and extended; documentation traceable to code; a defensible submission. | Whole document |
| S3 | Academic supervisor and examiners | Direct: assess the work. | Verifiable claims: requirements with metrics, measurements with conditions, rationale with alternatives and costs, limitations stated plainly. | §§4–5, 8–11; Appendices A–D |
| S4 | Future developers and maintainers | Direct: inherit three repositories. | A stable model-integration contract (registry + engine adapter + client profile), deterministic harnesses, and requirements that say which behaviour is load-bearing. | §4, §8, §9; UC-18–UC-23 |
| S5 | Bystanders in the captured field | Incidental: anyone whose voice or image enters the handset's microphone and camera range. They have not consented and have no interface to the system. | Not being recorded beyond what the function requires; not having their audio or image leave the device unprotected; retention limits. Current posture: plaintext LAN transport by design (TR §12.1) and an always-on device-side raw microphone dump with no user-facing control (TR §11.4, defect 1) — both directly adverse to this stakeholder. | §12, §13 |
| S6 | Upstream model authors (Sound of Pixels; multisensory) | Upstream: their published work and pretrained weights are the system's core. | Attribution and licence compliance. The TR records an attribution gap: the Sound of Pixels model definitions carry no upstream reference, both repositories ship placeholder licence files, and the checkpoint's provenance is unrecorded (TR §15.2). | §7, §13; References |
| S7 | Subjects of the training datasets (AudioSet, VoxCeleb, MUSIC) | Upstream, involuntary: people whose recordings trained the checkpoints. | Noted for completeness: this project cannot discharge obligations owed by the upstream training efforts, but inherits their ethical surface when deploying the weights. | §13 |
| S8 | Medical device regulators | Boundary, not a relationship: no regulated claim is made (§2.2.1). | That products entering their regime conform to it. Listed to make the scope boundary explicit, not because this study interacts with them. | §2.2.1, §7 |

Two structural observations the rest of the document acts on. First, the
stakeholder with the most at stake (S1) has had no voice in the project so
far; any future user testing with d/DHH participants would require ethics
approval, which Section 13 records as an explicit placeholder decision.
Second, the interests of S5 (bystanders) are currently served *worst* by the
system as built — the plaintext transport is at least a documented posture,
but the unconditional microphone dump is an open defect — and the
requirements in Sections 4–5 must not paper over that.

---

## 4. Functional requirements

### 4.0 Conventions

Requirements are numbered `FR-###` in blocks by subsystem (capture 001–,
streaming 010–, inference 020–, separation 030–, localization 040–, playback
050–, model selection 060–, configuration 070–, diagnostics 080–); gaps in
the numbering are reserved for insertions and identifiers never change once
assigned. Every requirement in §§4.1–4.9 is **in scope (case study)**; §4.10
lists functional requirements a *product* would carry that this case study
deliberately does not, feeding the Section 7 gap analysis.

Columns: **Source** names whose need the requirement traces to — the
case-study objectives (O1–O5, §2.2), a model constraint (MC — something the
checkpoint or paper imposes), a platform constraint (PC — something Android,
gRPC, CUDA, or WSL2 imposes), or a stakeholder (S1–S8, Table 3.1).
**Verification** is one of test (T), analysis (A), inspection (I), or
demonstration (D). **Status** uses the §1.5 taxonomy, abbreviated: **I+V**
implemented and validated, **I,nv** implemented not validated, **PS**
partially satisfied, **NS** not satisfied (implemented behaviour contradicts
the requirement). A requirement whose status is NS is a finding, not an
error in this document; the two NS entries below are the recorded defects
that Section 12 will carry as risks.

### 4.1 Capture

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-001 | **Dual-modality capture.** The client shall capture camera frames and microphone audio concurrently, with capture rates, geometry, and cadence set by the active model profile. | The models are audio-visual; both modalities must exist per window (O1). | Must | T, D | I+V for the halves mode (conditions unrecorded); I,nv for speech and touch |
| FR-002 | **Guaranteed-rate audio capture.** The client shall capture audio at 44 100 Hz mono PCM16 and decimate in-app (linear-phase FIR) to the profile's wire rate of 11 025 or 22 050 Hz. | 44 100 Hz is the only rate Android guarantees on every device; capturing at the wire rate directly would be a portability defect (PC; S4). | Must | T | I,nv (decimator has no unit test; device behaviour is a Phase 6 probe target) |
| FR-003 | **Profile-defined frame geometry.** The client shall produce either two 224×224 centre-cropped half-frames (halves mode) or one 224×224 mid-grey letterboxed full frame (speech, touch), JPEG quality 90. | Each model's input geometry is checkpoint-bound (MC). | Must | T, I | I,nv |
| FR-004 | **Profile-defined frame cadence.** The client shall throttle analysis frames to the profile rate (8 fps halves/touch; 30 fps nominal for speech via a 33 ms throttle interval, the camera delivering its nearest supported cadence) with keep-only-latest backpressure. | Frame rate is a model constraint; stale frames are worthless (MC). | Must | T | I,nv |
| FR-005 | **Permission gating.** The client shall not capture without granted CAMERA and RECORD_AUDIO permissions, and shall inform the user when they are refused. | Platform requirement and baseline privacy expectation (PC; S5). | Must | T | I,nv (on refusal the app informs and exits; graceful re-request is a product gap, §4.10) |
| FR-006 | **Capture bound to the session.** Capture shall run only while a session the user started is active, and shall stop when the session stops or the app leaves the foreground. | The microphone and camera must not outlive the user's intent (S5, S1). | Must | T | **NS** — no `onPause`/`onStop` handling exists; backgrounding continues capture (TR §11.4, defect 2) |

### 4.2 Streaming

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-010 | **Bidirectional streaming transport.** Client and server shall exchange timestamped `StreamChunk`/`StreamResult` messages over one gRPC bidirectional stream per session. | Continuous capture up, continuous results down, one connection (O1). | Must | T | I+V (halves; conditions unrecorded) |
| FR-011 | **Loss policy per modality.** Audio chunks shall be delivered losslessly (suspending emission, 125 ms nominal blocks of streamRate/8 samples); frame chunks may be dropped under backpressure. | A dropped audio block is an audible artefact; a dropped frame only ages the visual conditioning (O1). | Must | T, I | I,nv |
| FR-012 | **Model named at stream open.** The client shall name the model in the `sonicsight-model` call metadata; a stream with no metadata shall run the default `sonicsight` model. | Per-stream selection is the switchable-models contract; the default preserves legacy clients (O2). | Must | T | I+V (unit) |
| FR-013 | **Explicit teardown.** The client shall end a session with an `is_last` chunk; the server shall then emit the flushed overlap-add tail before closing. | Deterministic stream end; no truncated final audio (O1). | Must | T | I,nv |
| FR-014 | **Message size bound.** Both endpoints shall enforce a 16 MB message cap; an oversized message shall fail that message's stream, not the process. | Bounds memory per message; protects the server from malformed clients (PC). | Must | T | I,nv (cap configured; oversize behaviour untested — Phase 5 failure injection) |
| FR-015 | **Connection keepalive.** The client shall keep the channel alive across idle gaps (30 s keepalive, permitted without calls). | Session restarts should not pay reconnection latency (O1). | Could | I | I,nv |
| FR-016 | **In-band touch controls.** In touch mode the chunk stream shall carry pixel queries (at most 4 per chunk), the level-triggered freeze flag, the cluster-request latch, and the sticky-clear signal. | The touch interaction is in-band, ordered with capture (O1; MC). | Must | T | I,nv |
| FR-017 | **Result metadata.** Every result shall carry sequence number, echoed model id, buffering flag, and server timing fields (inference, post-processing, total ms). | Client-side staleness filtering, gap detection, and all Phase 5 measurement depend on these fields (O3). | Must | T | I,nv |

### 4.3 Inference and session management

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-020 | **Per-stream state isolation.** All per-session mutable state — audio buffer, frame ring, overlap-add buffers, touch caches — shall be instantiated per stream from the selected model's specification, never shared across streams. | Two concurrent sessions with different models must not collide (O2). | Must | T, A | **PS** — buffers are per-stream, but normalisation EMAs live on process-wide singletons, shared across concurrent same-mode streams: the input-gain EMA (`_norm_gain_ema`, TR §11.4 defect 8) always; the mask EMA (`_mask_ema`, `src/inference.py:1184-1193`) under non-default `SONICSIGHT_MASK_SMOOTH`; the pixel-gain EMA (`src/inference.py:1375-1381`) across touch streams; the CAM-scale EMA (`src/engines/multisensory_engine.py:146,253-259`) across speech streams |
| FR-021 | **Window selection.** The server shall select, per inference cycle, the newest frame for which a full audio window exists centred on it, with the window start aligned to the 256-sample STFT hop and at least the specification's `window_min_advance` between consecutive windows. | Phase alignment and monotonic progress are correctness constraints of the reconstruction (MC). | Must | T | I,nv (mechanics unit-tested via buffer tests; end-to-end timing ungated) |
| FR-022 | **Early inference (halves only).** Below one full window of audio, the server shall produce provisional results on a zero-padded window, marked as buffering and excluded from overlap-add. | Cuts perceived dead time during the 5.9 s fill (O1). | Should | T | I,nv |
| FR-023 | **Serialised GPU access.** All GPU inference across all sessions and all request paths shall be serialised through one process-wide lock. | One consumer GPU; concurrent kernels from two frameworks would contend unpredictably (PC). | Must | A, T | **PS** — both gRPC paths lock; the legacy FastAPI `/predict` path runs GPU inference without it (`src/main.py:102-108`; no in-tree caller — feeds G13). Also the direct cause of the low concurrency ceiling NFR-PERF-007 will measure |
| FR-024 | **Engine load tolerance.** At startup the server shall attempt to load every registered engine, tolerate individual load failures, and serve the models that loaded. | The server must run on hosts without TensorFlow (O2; PC). | Must | T | I+V (unit; demonstrated on E-D where TensorFlow is absent) |
| FR-025 | **Frame ring discipline.** The server shall cap the frame ring per the model specification and drop stale or out-of-order frames. | Bounded memory; frames older than the window are useless (O1). | Must | T | I,nv |
| FR-026 | **Audio ring discipline.** The server shall cap buffered audio at 15 s at the wire rate, discarding oldest samples while advancing the absolute base-sample index. | Bounded memory under a slow consumer (O1). | Must | T | I,nv |

### 4.4 Separation

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-030 | **Halves separation.** In halves mode the server shall return two independently separated tracks — left and right screen halves — at 11 025 Hz. | The deployed Sound of Pixels scene division (O1; MC). | Must | T, D | I+V (measured effects recorded; conditions unrecorded) |
| FR-031 | **Speech separation.** In speech mode the server shall return on-screen and off-screen tracks at 22 050 Hz wire rate, resampling to and from the model's 21 000 Hz internally at the exact 21:20 ratio. | The multisensory model's contract (MC). | Must | T | I,nv (adapter never run against the validated probe — TR Table 1, item 4) |
| FR-032 | **Reconstruction continuity.** Overlap-add output shall be gapless and duplicate-free on an absolute sample timeline, with raised-cosine crossfades between window estimates, and shall count gap-fills, splice fallbacks, and lagging windows. | Stitched windows must form one continuous signal; the counters make violations observable (O3). | Must | T | I+V (unit — 9 tests) |
| FR-033 | **Drain safety cap.** A single result's audio payload shall not exceed 1.5 s of samples, protecting the client's jitter buffer after a stall. | A multi-second drain would overflow playback buffering (O1). | Must | T | **PS** — cap correct on the halves path; the speech branch trims to 0.75 s because the cap is denominated in samples, not seconds (TR §11.4, defect 3) |
| FR-034 | **Output level normalisation.** The server shall apply smoothed RMS normalisation to output audio (target RMS 0.10, gain cap 20×, smoothing 0.3), adjustable by environment variable. | Raw separated stems vary wildly in level; measured effect on usability recorded (O1). | Should | T | I,nv (effects measured, conditions unrecorded) |
| FR-035 | **Region synthesis from cache.** In touch mode the server shall synthesize a queried region's audio from the cached window features — region-pooled mask, unwarp before renormalisation, no new forward pass — returning a 2.0 s tail with a relative energy value. | The touch interaction must answer at query rate, not inference rate (O1; MC). | Must | T | I,nv (18 unit tests; numeric gates unexecuted) |
| FR-036 | **Honest silence.** A query on a region whose cells all fall below the measured contrast threshold shall return zero energy rather than synthesized leakage. | Synthesizing "~40 % of everything" for a silent region misleads the listener (S1). | Must | T | I,nv (threshold calibrated once, 2026-08-05; marked provisional in code) |

### 4.5 Localization

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-040 | **Halves maps.** Halves-mode results shall carry two square uint8 heatmaps at the 56×56 wire convention, side inferable from byte count. | The client's decoder infers geometry; the convention is the contract (O1). | Must | T | I,nv |
| FR-041 | **Speech map.** Speech-mode results shall carry one full-frame map derived from the model's class activation map by the recorded post-processing chain (clamp negatives, time-mean, EMA-scaled normalisation, bilinear 7×7→56×56). | The validated localization evidence is this chain, exactly (MC; O3). | Must | T | I,nv (chain unit-tested TensorFlow-free) |
| FR-042 | **Confidence gate.** When the class activation map's positive fraction is below 0.10 the server shall withhold the map (empty bytes) while reporting the raw confidence value. | The alignment head is not a saliency map; an all-negative map means "audio does not match video" and must not be rendered as localization (MC; S1). | Must | T | I,nv (unit-tested) |
| FR-043 | **Touch energy map.** Touch-mode results shall carry the native 14×14 energy map with explicit wire dimensions, EMA-smoothed server-side. | Native grid honesty — no fake upsampling of a 14×14 signal (O3; S1). | Must | T | I,nv |
| FR-044 | **Source discovery.** On request the server shall cluster active cells into three sources (`N_CLUSTERS = 3`) plus a silence class, with identity and colour persistent across consecutive windows within a session. | Chip-level solo needs stable source identity (O1). | Should | T | I,nv (12 unit tests; thresholds provisional) |
| FR-045 | **Overlay registration.** The client shall register every map onto the camera preview by inverting the letterbox/crop transform of the active profile. | A misregistered map points at the wrong object — worse than none (S1). | Must | T | I+V (unit — coordinate-map JVM tests) |

### 4.6 Playback

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-050 | **Dual-track playback.** The client shall play the two returned tracks on two mono `AudioTrack`s at the profile's rate, hard-panned left and right. | Simultaneous audition of both streams (O1). | Must | T, D | I,nv |
| FR-051 | **Adaptive jitter buffering.** Each playback track shall buffer 200 ms initially, adapt upward by 50 ms after three consecutive underruns to a 500 ms ceiling, over a 1 500 ms ring, inserting silence when dry. | Network arrival variance must not cause audible underruns (O1). | Must | T | I,nv |
| FR-052 | **Gap concealment.** Missing result sequence numbers shall be concealed with equivalent-duration silence. | A skipped result must shift nothing; timeline integrity (O1). | Must | T | I,nv |
| FR-053 | **Track solo.** The user shall be able to solo either track or play both, under the profile's semantic labels; the control shall be absent in touch mode. | The core listening gesture; labels must match mode semantics (O1; S1). | Must | T | I,nv |
| FR-054 | **Query audio priority.** Touch-mode query audio shall play on a separate static track while the live mixture ducks to 0.15× and restores afterwards, generation-tagged against stale restores. | The queried region is the answer; the mixture is context (O1). | Must | T | I,nv |
| FR-055 | **Headphone hint.** The client shall hint once that headphones prevent feedback. | Speaker playback re-enters the microphone (O1). | Could | I | I,nv |

### 4.7 Model selection

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-060 | **Registry of frozen specifications.** Models shall be registered as frozen specification objects (window, hop, rates, frame rule, heatmap convention) resolved by id; three entries exist. | One authoritative record per model of every constant the stream shape depends on (O2; O5). | Must | T | I+V (unit — 13 registry tests) |
| FR-061 | **Selection failure semantics.** An unknown model id, or a known id whose engine is not loaded, shall abort the stream with `FAILED_PRECONDITION` and a message naming the model. | The client needs a machine-distinguishable "wrong model" signal (O2). | Must | T | I+V (unit) |
| FR-062 | **Switching is cancel-and-reopen.** A mode change during a session shall stop the stream, wait a settle delay (700 ms), and open a new stream under the new metadata; nothing shall attempt an in-stream switch. | The capture profiles are incompatible; a half-filled buffer for one model is garbage to the other (O2; MC). | Must | T, D | I,nv (device-level switch gate never run) |
| FR-063 | **Staleness by model echo.** The client shall drop any result whose echoed model id differs from the current selection. | In-flight results from the old stream must not render after a switch (O2). | Must | T | I,nv |
| FR-064 | **Both engines resident.** A model switch shall not unload or reload checkpoints; all loaded engines stay resident. | Switch cost is a stream restart, not a multi-second checkpoint reload (O2). | Must | A, T | I,nv — residency implemented; the both-frameworks memory footprint has never been measured (TR Table 1, item 8; NFR-PERF-010) |
| FR-065 | **Client profile mirroring.** The client shall hold one profile per model id mirroring the server specification: capture rates, frame kind and cadence, playback rate, labels, overlay form. | The wire contract is only coherent if both ends agree per mode (O2). | Must | T, I | I,nv |
| FR-066 | **No tunable model constants.** Per-model constants shall not be exposed as configuration — not environment variables, not flags. | Deviations from locked constants produced confident, plausible, wrong output twice on this project (MC; O3). | Must | T, I | I+V (unit — frozen dataclass; registry tests assert invariants) |

### 4.8 Configuration

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-070 | **Behavioural environment variables.** Mask style, renormalisation, separation mode, top-k, smoothing, audio normalisation, and dump controls shall be environment-configurable, read at server start, with the measured configuration as defaults. | Operator experimentation without code changes; defaults are the validated settings (O3; S4). | Should | T, I | I,nv |
| FR-071 | **Server address configuration.** The client shall let the user set the server host, persist it across restarts, and refuse changes during an active session; the port is fixed at 50051. | Different LANs, one build (O1). | Must | T | I,nv |
| FR-072 | **Multisensory environment overrides.** Checkpoint root and path, target GPU, memory growth, and memory fraction shall be environment-configurable. | Deployment differences between hosts must not require code edits (S4; PC). | Should | I | I,nv |
| FR-073 | **cuDNN workspace cap default.** The server shall default `TF_CUDNN_WORKSPACE_LIMIT_IN_MB` to 512. | Uncapped, the allocator holds ≈2.9 GB for a ≈0.5 GB working set; the cap costs no measurable latency (measured, E-M) (MC; PC). | Must | I | I+V (measured on E-M, 2026-08-03) |
| FR-074 | **Fixed service endpoints.** The server shall serve gRPC on 0.0.0.0:50051 and HTTP on 0.0.0.0:8000. | A fixed, documented endpoint on a trusted LAN; deliberate posture (O1; TR §12.1). | Must | I | I,nv |

### 4.9 Diagnostics

| ID | Requirement | Rationale (source) | MoSCoW | Ver. | Status |
|---|---|---|---|---|---|
| FR-080 | **Health reporting.** The server shall answer a unary health check with an any-model-loaded flag, a device string, and the loaded model ids. | Operators and harnesses need readiness without opening a stream (O3; S4). | Must | T | I,nv (no client caller; the device string omits the TensorFlow engine's placement — recorded gap) |
| FR-081 | **Opt-in server capture dumps.** When and only when explicitly enabled, the server shall record per-session diagnostic bundles (input audio, identity-mask control, output, per-cycle CSV). | Streaming claims need evidence; capture retention needs a switch (O3; S5). | Should | T | I,nv |
| FR-082 | **Periodic runtime diagnostics.** The server shall log input amplitude, drain statistics, and per-cycle summaries, flagging cycles over 140 ms. | The observable symptoms of falling behind (O3). | Should | I | I,nv |
| FR-083 | **Opt-in device capture dumps.** Any client-side persistence of captured audio or video shall be gated behind an explicit, default-off control. | Unconditional retention of microphone audio is adverse to S5 and S1 and serves no user function (S5; S1). | Must | T, I | **NS** — the client writes raw and decimated microphone WAVs unconditionally on every session, ungated by build type or setting (TR §11.4, defect 1) |
| FR-084 | **Client performance logging.** The client shall log encode-path performance summaries periodically. | Device-side cost visibility (O3). | Could | I | I,nv |
| FR-085 | **Measurement-grade timing fields.** Server timing fields (FR-017) shall be populated on every non-buffering result. | The Phase 5 suite measures the server through its own wire protocol (O3). | Must | T | I,nv |

### 4.10 Out of scope: functional requirements a product would carry

Recorded per §2.2; each row feeds Section 7. None of these is implemented,
and none is claimed.

| ID | Product requirement | Why out of scope here | What it would take |
|---|---|---|---|
| FR-P01 | Transport security and endpoint authentication (TLS, identity, authorisation) | Deliberate trusted-LAN posture for a lab study (TR §12.1) | TLS on the channel, credential provisioning, threat model |
| FR-P02 | Graceful permission recovery and onboarding | Refusal currently exits the app; acceptable for the operating team, not for users | Rationale UI, re-request flow, degraded modes |
| FR-P03 | On-device or offline operation | Client–server split is a founding constraint (GPU, dual frameworks) | Model compression/porting; see Section 8 decision 1 |
| FR-P04 | Background/lifecycle policy (calls, screen-off, interruptions) | Single-foreground-session lab use; FR-006 already fails without it | Lifecycle handling, audio focus, foreground service rules |
| FR-P05 | Consent, retention, and data-subject controls for captured media | No persistence should exist beyond opt-in diagnostics (FR-081/083) | Retention policy, consent UI, deletion controls |
| FR-P06 | Accessibility conformance to a named standard | Only content descriptions ship; full TalkBack UX is designed, not implemented (UC-24) | WCAG 2.2 / Android accessibility conformance work and audit |
| FR-P07 | Distribution, update, and support channels | Sideloaded debug builds on the team's devices | Signing, store distribution, crash reporting, update path |
| FR-P08 | Multi-user or concurrent-device service | One GPU serialises inference (FR-023); the design targets one active listener | Scale-out inference service; scheduling; see NFR-PERF-007 |

---

## 5. Non-functional requirements

### 5.0 Framework, conventions, and the single source of truth

Non-functional requirements are organised by the quality characteristics of
**ISO/IEC 25010:2023** [8]: functional suitability, performance efficiency,
compatibility, interaction capability, reliability, security, maintainability,
and flexibility. The framework is used because it forces coverage: a
characteristic with no requirements must be explained, not silently omitted.
The 2023 revision's ninth characteristic, *safety*, carries no NFRs here —
this case study makes no safety claim and runs no scenario in which the
system's output guards a person from harm; the corresponding hazard (a user
treating it as if it did) is handled as a risk in Section 12, not as a
quality requirement. That is this document's one deliberately empty category.

**Every threshold in this section is defined in
[`docs/nfr/nfr_targets.yaml`](nfr/nfr_targets.yaml)** — the single source of
truth. The load-test suite's assertions (Phase 4) will read that file; the
tables below quote it; a mechanical consistency check between this section
and the file will ship with the suite so the two cannot drift (its scope is
thresholds, statistics, and units — statuses and prose are outside it,
NFR-MAINT-004). Threshold values were set in this phase, *before* the suite
exists, from reasoned expectation grounded in the existing record — each
entry in the YAML carries a `basis` field saying where its number comes
from. Phase 5 will measure against them and reconcile in the open: a
measurement is allowed to force a target to change, and when it does the
*target* is revised with a recorded reason in the YAML's `reconciliation`
field — never the test, never silently.

**Statistics** are named exactly per assertion (e.g. p95, p99, max,
proportion, count, least-squares slope, max-over-steady-state); "average" is
not used. **Conditions** resolve to the §2.3 environments; scenario terms
(baseline, load, stress, spike, soak, switching, failure-injection, smoke)
are defined in the YAML's `meta.scenarios` block. Requirements whose
measurement needs the multisensory engine are tagged **E-M-conditional**:
they cannot be measured on E-D (TensorFlow absent, 4 GB GPU) and remain
unmeasured unless an E-M-equivalent host is available in Phase 5.
Mobile-side requirements against environment E-A are tagged
**matrix-conditional**: their device population is pinned by the Phase 6
matrix and they are unmeasurable before it. Both limitations are declared
now, not discovered later. **Status** in this phase is *target set —
unmeasured* unless an existing record or test already bears on the
requirement; two entries are already **NS** (not satisfied) on present
evidence and say so.

### 5.1 Performance efficiency

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-PERF-001 | Time to first non-buffering result on the halves mode shall be ≤ **8.0 s** at p95 over ≥ 20 stream openings, measured from stream establishment, single session, replay driver. | T | **Measured — pass** (Phase 5): E-D p95 6.34 s; E-M p95 6.22 s. Basis: 5.944 s window fill + inference + first-window margin |
| NFR-PERF-002 | Time to first non-buffering result on the speech mode shall be ≤ **3.5 s** at p95 over ≥ 20 stream openings, single session. **E-M-conditional.** | T | **Measured — pass** (Phase 5, E-M): p95 2.77 s. Basis: 2.102 s fill + 142.6 ms measured inference + margin |
| NFR-PERF-003 | Steady-state result cadence on the halves mode: the deviation of inter-result intervals from the 125 ms hop shall be ≤ **62.5 ms** at p95 (window: after first non-buffering result, ≥ 5 min), single session. | T | **Measured — pass** (Phase 5): E-D p95 18.0 ms; E-M p95 3.2 ms. Basis: hop 1378 samples; real window advances quantise to 1280/1536/1792 samples (TR §4.4) |
| NFR-PERF-004 | Per-window inference cost on the halves mode (`inference_time_ms`, timed around the forward pass) shall be ≤ **140 ms** at p95, ≤ **250 ms** at p99, and ≤ **1500 ms** at max over a ≥ 5-minute post-first-result window per run, single session; launch-to-yield time (`total_server_time_ms`) is reported as characterisation. | T | **Measured — target revised in the open, twice recorded** (Phase 5): (a) the original max ≤ 500 ms gate failed otherwise clean runs through rare isolated 1.2–1.5 s stalls (~1 per 10 min, both hosts) while every percentile held with ≥ 30 % headroom — p99 gate added, max relaxed to 1500 ms; (b) after the D-P5-1 overlap fix, `total_server_time_ms` spans the deliberate ingest/inference overlap, so the assertions were re-targeted to `inference_time_ms` at the same bounds (both reasons in the YAML `reconciliation:` field). Campaign-3 re-measurement in `EM_RESULTS.md` |
| NFR-PERF-005 | Per-window inference cost on the speech mode (`inference_time_ms`) shall be ≤ **200 ms** at p95 over a ≥ 5-minute post-first-result window per run, against the 250 ms hop; launch-to-yield time is reported as characterisation. **E-M-conditional.** | T | **Measured — pass** (Phase 5, E-M): pre-overlap record p95 152 ms; re-targeted to `inference_time_ms` after the D-P5-1 overlap fix for the same reason as NFR-PERF-004 (recorded reconciliation); campaign-3 re-measurement in `EM_RESULTS.md`. First-window cuDNN autotune (~4.8 s) sits in warmup, outside the assertion window |
| NFR-PERF-006 | Event-loop lag of the asyncio server, recorded continuously at ≥ 10 Hz in every scenario, shall be ≤ **100 ms** at p99 and ≤ **500 ms** at max at the single-session baseline; multi-session lag is reported as characterisation. | T | **Measured — pass** (Phase 5): E-D p99 13 / max 44 ms; E-M in-run p99 ≈ 2 / max 41 ms. One pre-serving exception recorded: model loading blocks the loop ~341 s before the port opens |
| NFR-PERF-007 | Concurrent capacity: with **2** simultaneous halves sessions, NFR-PERF-003 and NFR-PERF-004 shall still hold. The stress scenario shall additionally report the measured ceiling — the first concurrency at which any threshold breaches — as characterisation, not pass/fail. | T | **Measured — pass on E-M, not satisfied on E-D** (Phase 5): E-M stress held 2/2 at n = 2 with a ceiling of 3; E-D held 0/2 (host capacity, GTX 1650). The earlier E-D "ceiling = 1" is superseded as a measurement artifact (P5_VALIDATION A.2) |
| NFR-PERF-008 | Touch-mode query round trip (driver-observed, query sent → matching `PixelAudio` received) shall be ≤ **250 ms** at p95 over ≥ 100 queries, single session, loopback network. | T | **Measured — pass on E-M, failed on E-D** (Phase 5): E-M p95 1.56 ms; E-D 945–1183 ms under host saturation — an environment finding, not a protocol defect |
| NFR-PERF-009 | GPU memory on E-D with the PyTorch engine serving a halves session: device memory in use shall be ≤ **3 584 MiB** (≥ 512 MiB of the 4 096 MiB card free) at steady state, defined as after the first 5 minutes of the session. | T | Target set — unmeasured (the recorded E-D load run used 2 sessions, outside this target's single-session condition) |
| NFR-PERF-010 | GPU memory with **both** engines resident and a session active: ≤ **5 632 MiB** on a 6 144 MiB card (≥ 512 MiB free) at steady state (after the first 5 minutes). **E-M-conditional.** | T | **Measured — pass** (Phase 5, E-M): steady ≈ 2.3 GiB during the single-session speech load run; the full 3-hour campaign peaked at 2 650 MiB used / 3 317 MiB free. The TR's "largest unquantified risk" (TR §9.4) is now quantified |
| NFR-PERF-011 | Host memory stability: over a 30-minute single-session soak, server RSS slope over the final 20 minutes shall be ≤ **1.0 MiB/min** (least-squares); asserted from a no-dump soak (the diagnostic recorder buffers ~5.3 MiB/min by design). | T | **Mixed** (Phase 5, E-M): speech soak 0.086 MiB/min (pass); halves no-dump soak 1.03 MiB/min — 3 % over, within slope-estimator noise on a 20-minute tail. Target deliberately not relaxed; a ≥ 2 h soak decides |
| NFR-PERF-012 | GPU memory stability: over the same soak, device memory in use shall grow ≤ **128 MiB** after the first 5 minutes. | T | **NS — open defect D-P5-5** (§10.4): sustained *halves* streaming grows device memory ~34 MiB/min (+676 MiB/55 min; +1038 MiB/30 min on the no-dump soak) — would exhaust the 6 GiB card in ~2 h if unbounded. The speech soak measured only +93 MiB (its 64→128 MiB revision stands, reason recorded). Target not relaxed |
| NFR-PERF-013 | Result continuity: missing sequence numbers shall be ≤ **0.5 %** of results over every consecutive non-overlapping 5-minute window after the first non-buffering result, per measured session. | T | **Measured — pass** (Phase 5): 0.000 in every window on both hosts. Caveat: the metric counts gaps only across strictly increasing sequence pairs, so its passes are weak evidence (P5_VALIDATION C.10) |
| NFR-PERF-014 | Spike recovery: after 4 simultaneous stream openings against an idle server, at least **3** sessions shall survive, and each survivor shall reach its first compliant 10-second window (p95 absolute cadence deviation ≤ 62.5 ms, NFR-PERF-003's bound applied to a 10 s sliding window) within ≤ **30 s** of the burst. | T | **Measured — pass on E-M, failed on E-D** (Phase 5): E-M 4/4 survivors, worst compliant window at 11.8 s; on E-D survivors never reached compliance (host capacity) |
| NFR-PERF-015 | Steady-state result cadence on the speech mode: the deviation of inter-result intervals from the 250 ms hop shall be ≤ **125 ms** at p95 (window: after first non-buffering result, ≥ 5 min), single session. **E-M-conditional.** *(Phase 5 addition: the first E-M measurement exposed a served stride the Phase 2 set had no target to catch.)* | T | **Measured — pass after fix** (Phase 5, E-M): D-P5-1 (§10.4) was root-caused to inline-inference serialization plus a boundary-slack-unreachable advance guard, fixed the same day; re-measured interval p50 250.0 ms, dev p95 1.18 ms (was p50 373.5 / dev p95 128.8), throughput restored to the designed 4 results/s |
| NFR-PERF-016 | Concurrent capacity, speech: with **1** speech session, NFR-PERF-015 and NFR-PERF-005 shall hold. Two concurrent speech sessions are expected not to fit (≈ 0.6 GPU duty each) and are reported by stress as characterisation. **E-M-conditional.** *(Phase 5 addition.)* | T | **Measured — pass after fix** (Phase 5, E-M): 1/1 sessions meet NFR-PERF-015 and NFR-PERF-005 after the D-P5-1 fix (inference p95 157 ms, cadence dev p95 1.17 ms) |

### 5.2 Reliability

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-REL-001 | Malformed input (truncated PCM, invalid JPEG, zero-length chunk, wrong-field payloads): **100 %** of injected cases shall yield a per-stream error result or stream abort with the server process alive; a concurrent healthy session's NFR-PERF-003 shall keep holding during an injection phase of ≥ 5 minutes (long enough to evaluate its cadence window). | T | **Measured — pass** (Phase 5, E-M): 7/7 injections contained, healthy co-session cadence held throughout |
| NFR-REL-002 | Abrupt client disconnect mid-stream, repeated **50** times: after a 30 s settle, server RSS within **+5 %** of the pre-test baseline and GPU memory within **+64 MiB**; stream-open and stream-close log lines balance exactly (the orphaned-loop detector). | T | **Measured — pass, fully discharged** (Phase 5, E-M): RSS −0.15 %, GPU −13 MiB after 50 cycles; open/close log parity re-measured after the D-P5-4 close-line fix: 0 imbalance over 67 opens including the 50 abrupt disconnects |
| NFR-REL-003 | Oversized message (> 16 MB): **100 %** of attempts rejected at the transport layer; the process survives; the next session succeeds. | T | **Measured — pass** (Phase 5, E-M) |
| NFR-REL-004 | Degenerate media (wrong sample rate, black frames, silent audio, full-scale clipping audio): **100 %** of injections produce defined behaviour — results or clean error, no crash — with p95 cycle time during injection ≤ **280 ms** (2 × the NFR-PERF-004 p95 bound). | T | **Measured — pass** (Phase 5, E-M): 7/7 defined behaviour, p95 84 ms during injection |
| NFR-REL-005 | Soak: a 60-minute single halves session shall complete with **0** crashes, **0** unhandled exceptions in server logs, sequence-gap rate ≤ **0.5 %**, and splice fallbacks ≤ **1 %** of windows. | T | **Measured — pass** (Phase 5, E-M), discharged across two soaks: the 60-minute halves soak under `SONICSIGHT_DUMP_STREAM=1` (0 crashes, 0 unhandled exceptions, gap rate 0.000, splice rate 0.000) and a 30-minute no-dump soak reconfirming cleanliness without the recorder resident |
| NFR-REL-006 | Model-switch churn: **100** cancel-and-reopen cycles alternating the loadable models: **100 %** of results echo the model id of their own stream; RSS and GPU memory after the run within **+5 % / +64 MiB** of baseline. | T | **Measured — pass** (Phase 5, E-M, the harder cross-framework pair `sonicsight` ↔ `multisensory`): echo 1.000, RSS −0.04 %, GPU +11 MiB. The E-D run of the PyTorch-only pair failed its GPU delta at +250 MiB (P5_VALIDATION B.8) — not reproduced on E-M; E-D re-measurement open |
| NFR-REL-007 | GPU out-of-memory during a session (induced via memory-fraction squeeze where feasible): the affected stream shall deliver an error result and close within **5 s**; a subsequent session shall succeed without server restart. | T or A | Target set — unmeasured; injection remains manual (memory-fraction squeeze). Analysis fallback, if injection proves infeasible: a documented code-path review of the OOM handler with the reviewed lines cited, recorded as analysis-verified |

### 5.3 Functional suitability

Quality-of-output requirements. This case study measures *sanity*, not
benchmark quality — no signal-to-distortion evaluation exists or is claimed
(§2.2, non-goals); these three requirements are the wired-wrong detectors.

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-FUNC-001 | Replay-harness separation sanity: on an operator-supplied reference clip **within the model's domain (musical instruments — §2.2)**, SHA-256 recorded in run metadata, same clip and offset across runs, three identical runs must agree: left/right output Pearson correlation shall be ≤ **0.35** (the harness's own "healthy" verdict bound; 0.35–0.45 is marginal, > 0.45 is a defect), halves mode. | T | **Measured — pass** (Phase 5, E-M): on the in-domain guitar+cello clip (SHA `f33e3e700cc86f64`), streaming Pearson 0.187/0.187/0.187 over three deterministic replays; offline path 0.097; unchanged (0.1872) after the D-P5-1 loop restructure. The earlier 0.961 was an out-of-domain speech clip — §10.4, D-P5-2, reclassified |
| NFR-FUNC-002 | Localization cross-check: the masked-frame protocol shall PASS with margin ≥ **+0.30**. **E-M-conditional.** | T | Previously measured **+0.8248 PASS** on E-M (2026-08-03) by the standalone probe protocol; the load suite does not produce this metric and reports it honestly as not-measured — probe integration is future suite work |
| NFR-FUNC-003 | Confidence-gate correctness: for synthetic class-activation inputs with positive fraction < 0.10, **100 %** of results shall carry an empty map with the raw confidence reported (FR-042). | T | **Measured — pass** (Phase 5): asserted by `tests/test_confidence_gate.py`, green on both hosts |
| NFR-FUNC-004 | Speech-mode streaming result contract: **100 %** of non-buffering speech-mode results shall echo `model_id=multisensory` and carry exactly one heatmap slot — a 3 136-byte 56 × 56 map or an empty map under the confidence gate — with `cam_confidence` in [0, 1] and non-identical foreground/background audio when both are present. Semantic separation quality is deliberately not asserted on synthetic content. **E-M-conditional.** *(Phase 5 addition: the structural contract the suite can measure live, complementing NFR-FUNC-002's offline protocol.)* | T | **Measured — pass** (Phase 5, E-M): 1.000 over every result of the speech baseline |

### 5.4 Compatibility

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-COMPAT-001 | The two `sonicsight.proto` copies shall be byte-identical; **0** differing bytes, asserted mechanically. | T | Holds as of 2026-08-08 (Appendix D); to be asserted in the suite's smoke subset |
| NFR-COMPAT-002 | A stream opened with no model metadata shall run the `sonicsight` model — legacy-client compatibility, **100 %** of such streams. | T | Existing unit test bears on it (selection-branch tests) |
| NFR-COMPAT-003 | On a host without TensorFlow, the server shall start, serve both PyTorch modes (≥ **2** models served), and report a loaded-model list omitting `multisensory`; **100 %** of health checks consistent with the loaded set. | T | Demonstrated implicitly on E-D; to be asserted explicitly |

### 5.5 Interaction capability

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-INT-001 | Buffering feedback: the client shall present session-fill progress within **500 ms** of the record-control tap, p95 over 20 instrumented launches, E-A. **Matrix-conditional.** | T | Target set — unmeasured (Phase 6 instrumented test) |
| NFR-INT-002 | Semantic label integrity: in **100 %** of mode/label combinations, track labels shall match the mode's semantics (never "Left/Right" on the speech mode). | T, I | Profile-defined today (FR-065); to be asserted as a unit test |
| NFR-INT-003 | Screen-reader labelling: **0** missing-label findings from Accessibility Scanner on the main screen's interactive controls, E-A. **Matrix-conditional.** | T | Target set — unmeasured (Phase 6). Full TalkBack traversal remains designed-not-implemented (UC-24) and is *not* claimed by this requirement |
| NFR-INT-004 | Failure-message actionability: **100 %** of the branches of the client's error-mapping function shall name the configured host and a remedy (settings or retry), or a specific cause with its remedy. | T, I | Implemented behaviour (UC-07); to be asserted as a unit test |

### 5.6 Security

The posture is deliberate (TR §12.1): plaintext, unauthenticated, trusted
LAN, lab use. The in-scope requirements below bound that posture; everything
stronger is FR-P01 (product scope, Section 7).

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-SEC-001 | Confinement of exposure: the system shall communicate with **0** endpoints other than the user-configured server on the LAN — no telemetry, no third-party services, verified by inspection of both codebases and (Phase 5) by traffic observation scoped to the app and server processes (LAN = the configured RFC 1918 server address; OS-level DNS and platform telemetry excluded). | I, T | Holds on inspection to date; traffic-level assertion pending |
| NFR-SEC-002 | Capture persistence is opt-in everywhere: **0** code paths shall persist captured audio or video without an explicit, default-off control (server FR-081 conforms; device FR-083 does not). | I, T | **NS** — the client's unconditional microphone WAV dump violates this today (TR §11.4, defect 1). Recorded as a finding; fix is out of this study's permitted changes (§13 working rules) |

### 5.7 Maintainability

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-MAINT-001 | The backend unit suite shall pass with **0** failures and **0** collection errors. | T | **Measured — pass** (Phase 5): 81 passed, 1 skipped, 0 errors on E-M. The 2 recorded collection errors (TR §10.5, defect 6) were pytest miscollecting the manual live-server script `test_grpc_client.py`; it is now excluded from collection via `tests/conftest.py` with the classification recorded — a categorisation fix, not a hidden failure |
| NFR-MAINT-002 | Model extensibility: integrating a new model shall require **0** changes to the named core layers — `src/inference.py` buffering, `src/overlap_add_buffer.py`, the `src/grpc_server.py` stream loops, and the protocol other than additively — registry entry, engine adapter, client profile only. | A | Held for both historical integrations (multisensory, pixel); assessed by analysis of those change sets |
| NFR-MAINT-003 | Constant immutability: **100 %** of the constants in the named inventory — the frozen `ModelSpec` fields plus the engine adapters' LOCKED constant blocks — shall have no environment or flag override (FR-066). | T, I | Existing registry tests bear on it |
| NFR-MAINT-004 | Threshold single-sourcing: the load suite shall read **100 %** of its assertion thresholds from `docs/nfr/nfr_targets.yaml` — **0** hard-coded threshold literals in suite code (the replay harness's historical verdict bounds are adopted *by* the YAML via NFR-FUNC-001, and the suite reads them from the YAML); and the §5 tables shall match the YAML on thresholds, statistics, and units (mechanical check). | T, I | Constitutive requirement of this study's method; asserted from Phase 4 onward |

### 5.8 Flexibility

| ID | Requirement | Ver. | Status |
|---|---|---|---|
| NFR-FLEX-001 | Degraded-mode portability: the backend shall start and serve the two PyTorch modes on a Windows host with no TensorFlow (E-D) and the full set on a WSL2/Linux host with TensorFlow (E-M-class); asserted as absolute counts so a load failure cannot shrink the denominator: ≥ **2** models served on E-D, exactly **3** on an E-M-class host. | T, D | **Measured — pass, both halves** (Phase 5): 2 models on E-D; 3 on E-M with all engines loaded |
| NFR-FLEX-002 | Device compatibility: the client shall install and run a halves-mode session to ≥ 1 non-buffering result with clean teardown on **100 %** of the Phase 6 emulator matrix across API 28–36, E-A. **Matrix-conditional.** | T | Target set — matrix defined in Phase 6 |
| NFR-FLEX-003 | Audio-capture portability: capture at 44 100 Hz with in-app decimation (FR-002) shall initialise successfully on **100 %** of the Phase 6 device matrix, E-A (**matrix-conditional**); the capability probe shall record `getMinBufferSize` behaviour on every matrix device. | T | Target set — Phase 6 |

### 5.9 The latency requirement that is deliberately absent

There is no NFR of the form "end-to-end perceived lag shall be ≤ *T* for
assistive use," and its absence is a statement: §2.2.3 establishes with cited
tolerances [5][6][7] that assistive-grade latency (tens of milliseconds, per
the hearing-device tolerance literature) is two to three orders of magnitude
below what these models' windows permit (≈ 1.2 s and ≈ 3.0 s, estimated). Writing such an NFR against this system
would manufacture a guaranteed failure to no informative end; the honest
requirement set measures what the system *is* (NFR-PERF-001…014) and states
what it is *not* (Section 7). What Phase 5 adds is the missing measurement
itself: a stopwatch-grade estimate of true perceived lag to replace the
arithmetic estimate — recorded in Appendix C as a characterisation, not
thresholded.

---

## 6. Use case model

### 6.1 Actors

| Actor | Kind | Description |
|---|---|---|
| **Listener** | Primary, human | The person holding the handset: starts sessions, selects modes, listens, touches. In this case study the listener has always been a member of the project team (§3, S2) — deliberately not a member of the prospective beneficiary population. |
| **Operator** | Primary, human | The person who deploys, configures, and monitors the backend. In practice the same team, but a distinct role with distinct use cases. |
| **Developer** | Primary, human | The person who validates, extends, and maintains the system: harnesses, test suites, protocol regeneration, model integration. |
| **Backend server** | Secondary, system | The gRPC service the client converses with; participant in every streaming use case. |
| **GPU / CUDA stack** | Secondary, system | The device both engines share; participant in deployment, diagnostics, and harness use cases. |

A use case model for this system with only an end-user actor would be
incomplete to the point of dishonesty: roughly half of what the software
demonstrably does — deterministic replay, A/B harnesses, health and
diagnostics, environment provisioning across a WSL2 boundary — exists for the
operator and developer roles.

### 6.2 Use case inventory

Use cases are derived from what the software does at the documented heads,
not from ambition. Each carries the §1.5 status label; the *Requirements
exercised* rows of Appendix B reference the Section 4–5 identifiers assigned
in Phase 2. Full specifications for every use case are in Appendix B.

**Table 6.1 — Use case inventory.**

| ID | Name | Primary actor | Status |
|---|---|---|---|
| UC-01 | Start and stop a listening session | Listener | Implemented; validated end to end only for the `sonicsight` mode, with unrecorded measurement conditions |
| UC-02 | Select a listening mode | Listener | Implemented, not validated (selection branch unit-tested; on-device switch ungated) |
| UC-03 | Switch mode mid-session | Listener | Implemented, not validated |
| UC-04 | View the localization overlay | Listener | Implemented; speech-mode confidence gate implemented, not validated |
| UC-05 | Solo one separated track | Listener | Implemented, not validated |
| UC-06 | Configure the server address | Listener | Implemented, not validated |
| UC-07 | Recover from server unavailability | Listener | Implemented, not validated |
| UC-08 | Tap or drag to hear a region (touch mode) | Listener | Implemented, not validated (ran live once, 2026-08-05; numeric gates unexecuted) |
| UC-09 | Follow a region — sticky long-press (touch mode) | Listener | Implemented, not validated |
| UC-10 | Freeze the energy map (touch mode) | Listener | Implemented, not validated |
| UC-11 | Solo a discovered source (touch mode) | Listener | Implemented, not validated (thresholds provisional) |
| UC-12 | Deploy and launch the backend | Operator | Implemented and validated (launch procedure exercised; recorded runs exist) |
| UC-13 | Provision checkpoints and the TensorFlow environment | Operator | Implemented and validated on E-M; not reproduced on E-D |
| UC-14 | Configure separation behaviour via environment variables | Operator | Implemented; measured effects recorded for mask/normalisation variables, conditions unrecorded |
| UC-15 | Check backend health | Operator | Implemented, not validated (no client caller; manual exercise only) |
| UC-16 | Capture diagnostics and monitor GPU residency | Operator | Implemented, not validated |
| UC-17 | Bridge the WSL2 network boundary | Operator | Documented procedure; never exercised with both models resident |
| UC-18 | Run the deterministic replay validation harness | Developer | Implemented and validated (harness runs; verdict thresholds defined). Halves mode only |
| UC-19 | Run the pixel-mode A/B harness | Developer | Implemented; its design questions remain open pending a human listening decision |
| UC-20 | Run the automated unit suites | Developer | Implemented and validated (66 passed, 1 skipped, 2 collection errors, 2026-08-05, E-D) |
| UC-21 | Regenerate protocol stubs after a contract change | Developer | Implemented and validated (both proto copies byte-identical, verified 2026-08-08) |
| UC-22 | Add a new model behind the registry | Developer | Implemented contract (MODELS.md); exercised twice (multisensory, pixel) |
| UC-23 | Run the multisensory migration suite T0–T5 | Developer | Implemented; result artefacts absent from the tree (TR §10.2) |
| UC-24 | TalkBack explore-by-touch traversal | Listener | **Designed, not implemented** (in-code design note; only content descriptions ship today) |
| UC-25 | Alternative region shapes (thirds, quadrants, freeform) | Listener | **Designed, not implemented** (explicitly deferred in the model contract) |

**Retired path, recorded but not modelled.** A client-streaming file-upload
flow (`ProcessVideo` RPC → `ResultActivity` result screen) is implemented and
unreachable: no UI path triggers it at the documented heads. It is legacy
kept for compatibility, not a current use case; it appears in Section 7 as a
scope decision to be made (retire or maintain), not in the diagrams. The
FastAPI `/predict` endpoint has no consumer in any of the three repositories
and is treated the same way.

**Other planned work, not use cases.** The principal-component ambient colour
layer and the PyTorch port of the multisensory model are designed-not-
implemented items (TR Table 1, items 11–12) with no actor-facing behaviour.
The port reappears in the Section 7 gap analysis (G4, as the missing
precondition for on-device paths); the colour layer is deferred design work
that Section 14 will list under future work.

### 6.3 Use case diagrams

Mermaid has no native use case diagram type, so both figures use the
documented `flowchart` construction: actors as circles outside the system
boundary, use cases as stadium nodes inside a subgraph, undirected
associations, and dotted labelled links for «include»/«extend» relationships
(rendered as the words "includes"/"extends" for portability). Dashed borders
mark designed-not-implemented use cases; Figure 3 carries the legend.

**Figure 3 — Use case diagram: end-user cases.** UC-08–UC-11 are the touch
mode group (implemented, not validated); UC-24 and UC-25 are planned.
(Source: `docs/diagrams/ar-figure-03-usecase-user.mmd`; rendered:
`docs/diagrams/svg/ar-figure-03-usecase-user.svg`.)

```mermaid
flowchart LR
    LST(("Listener"))
    SRV(("Backend<br/>server"))

    subgraph SS["SonicSight"]
        UC01(["UC-01 Start and stop<br/>a listening session"])
        UC02(["UC-02 Select a<br/>listening mode"])
        UC03(["UC-03 Switch mode<br/>mid-session"])
        UC04(["UC-04 View the<br/>localization overlay"])
        UC05(["UC-05 Solo one<br/>separated track"])
        UC06(["UC-06 Configure the<br/>server address"])
        UC07(["UC-07 Recover from<br/>server unavailability"])

        UC08(["UC-08 Tap or drag<br/>to hear a region"])
        UC09(["UC-09 Follow a region<br/>(sticky long-press)"])
        UC10(["UC-10 Freeze the<br/>energy map"])
        UC11(["UC-11 Solo a<br/>discovered source"])

        UC24(["UC-24 TalkBack<br/>explore-by-touch"])
        UC25(["UC-25 Alternative<br/>region shapes"])
    end

    LST --- UC01
    LST --- UC02
    LST --- UC03
    LST --- UC04
    LST --- UC05
    LST --- UC06
    LST --- UC08
    LST --- UC09
    LST --- UC10
    LST --- UC11

    UC01 -. "includes" .-> UC02
    UC03 -. "includes" .-> UC02
    UC07 -. "extends" .-> UC01
    UC09 -. "extends" .-> UC08

    UC01 --- SRV
    UC03 --- SRV
    UC08 --- SRV

    LGD["Legend: dashed border =<br/>designed, not implemented"]

    classDef planned stroke-dasharray: 5 5;
    class UC24,UC25,LGD planned;
```

**Figure 4 — Use case diagram: operator and developer cases.** (Source:
`docs/diagrams/ar-figure-04-usecase-operator-developer.mmd`; rendered:
`docs/diagrams/svg/ar-figure-04-usecase-operator-developer.svg`.)

```mermaid
flowchart LR
    OPR(("Operator"))
    DEV(("Developer"))
    GPU(("GPU /<br/>CUDA stack"))

    subgraph SS["SonicSight backend and tooling"]
        UC12(["UC-12 Deploy and launch<br/>the backend"])
        UC13(["UC-13 Provision checkpoints<br/>and the TF environment"])
        UC14(["UC-14 Configure separation<br/>via environment variables"])
        UC15(["UC-15 Check backend health"])
        UC16(["UC-16 Capture diagnostics,<br/>monitor GPU residency"])
        UC17(["UC-17 Bridge the WSL2<br/>network boundary"])

        UC18(["UC-18 Run the replay<br/>validation harness"])
        UC19(["UC-19 Run the pixel-mode<br/>A/B harness"])
        UC20(["UC-20 Run the unit suites"])
        UC21(["UC-21 Regenerate<br/>protocol stubs"])
        UC22(["UC-22 Add a model<br/>behind the registry"])
        UC23(["UC-23 Run the migration<br/>suite T0 to T5"])
    end

    OPR --- UC12
    OPR --- UC13
    OPR --- UC14
    OPR --- UC15
    OPR --- UC16
    OPR --- UC17

    DEV --- UC18
    DEV --- UC19
    DEV --- UC20
    DEV --- UC21
    DEV --- UC22
    DEV --- UC23

    UC12 -. "includes" .-> UC13
    UC22 -. "includes" .-> UC21
    UC12 --- GPU
    UC16 --- GPU
    UC19 --- GPU
```

---

## 7. Scope gap analysis

This section assembles what a *product* would require that this case study
deliberately does not deliver. It is built structurally: from the §2.2
non-goals, the §4.10 out-of-scope functional requirements (FR-P01–P08), and
the costs recorded against the Section 8 decisions. Naming these gaps is
part of the study's claim to rigour — a requirements study that shows it
knows what it is not delivering is stronger than one that quietly omits it.
Gaps are ordered roughly by how far they sit from the current system.

**Table 7.1 — Product gaps.**

| # | Gap | Why out of scope here | What it would take |
|---|---|---|---|
| G1 | **Clinical validation of assistive benefit** | No member of the beneficiary population has used the system; no user study, no outcome measure, no claim (§2.2, §2.2.1) | Ethics approval, study design with d/DHH participants, validated outcome instruments, and — before any of that — closing G2 |
| G2 | **Latency reduction by two to three orders of magnitude** | The lag floor is the models' analysis windows (≈ half of 5.9 s / 2.1 s), not engineering slack (§2.2.3) | Different models: causal/streaming separation architectures with tens-of-milliseconds windows; nothing in this system's engineering can bridge it |
| G3 | **Regulatory conformity** | A product compensating for hearing impairment enters medical-device regimes (EU MDR 2017/745; US FDA OTC hearing aid rules) — this laboratory study sits outside them (§2.2.1) | Classification, conformity assessment, clinical evaluation, post-market surveillance; a regulatory strategy predating the product design |
| G4 | **On-device or edge inference; offline operation** | Foreclosed by D1 (ADR-0001): model sizes, GPU need, dual frameworks (FR-P03). The only step taken toward it — the multisensory PyTorch port — is a checkpoint-less skeleton (TR Table 1, item 12) | Validated ports (TFLite/ONNX/ExecuTorch), quantisation with revalidation against the correctness experiments, or new models per G2 |
| G5 | **Transport security and authentication** | Deliberate plaintext trusted-LAN posture (TR §12.1; FR-P01; NFR-SEC-001 bounds it) | TLS on the channel, endpoint identity, credential provisioning; a threat model covering the captured-media path |
| G6 | **Consent, retention, and data-subject controls** | No persistence should exist beyond opt-in diagnostics; the device-side dump defect (FR-083 NS) shows the machinery is absent (FR-P05) | Retention policy, consent surfaces, deletion controls — and fixing FR-083 first |
| G7 | **Accessibility conformance to a named standard** | Only content descriptions ship; the TalkBack explore-by-touch UX is designed, not implemented (UC-24; FR-P06). NFR-INT-003 asserts labelling only, deliberately short of conformance | WCAG 2.2 / Android accessibility conformance work, audit, and assistive-technology user testing — weighted heavily given the study's stated motivation |
| G8 | **Application lifecycle and interruption policy** | Single-foreground-session lab use; FR-006 is *not satisfied* without it (FR-P04) | Foreground-service policy, audio focus, call/screen-off handling, permission recovery (FR-P02) |
| G9 | **Sustained-session thermal and battery behaviour** | Unmeasured on any device; E-A is an unfilled environment row | Phase 6 will define a sustained-session probe (a 15-minute low-tier session recording frame-rate decay, per the study brief) as a first step; a product needs sustained-session budgets per device tier |
| G10 | **Multi-user service and scale** | One GPU serialises all inference (FR-023); the ceiling is expected to be low — single digits — and NFR-PERF-007's stress scenario will measure it (FR-P08) | Scale-out inference service, scheduling, session isolation — a different backend architecture |
| G11 | **Internationalisation** | The client ships a single English locale: one strings resource set (`values/strings.xml`) and no locale-qualified resources (`values-night/` is a theme qualifier only) — verified by inspection 2026-08-08 | Externalised strings audit, locale resource sets, RTL layout verification |
| G12 | **Distribution, update, and support channels** | Sideloaded debug builds on the team's devices (FR-P07) | Signing, store distribution, crash reporting, update path |
| G13 | **Retire-or-maintain decision on the legacy paths** | The upload→`ResultActivity` flow is implemented and unreachable; FastAPI `/predict` has no in-tree consumer (§6.2) — and runs GPU inference outside the process-wide lock (FR-023 PS) | A decision: delete (and simplify the contract) or document as supported API and bring under the lock; carrying dead paths is a maintenance liability either way |
| G14 | **Separation-quality benchmark evaluation** | §2.2 non-goal: no evaluation against a labelled benchmark; the system reports no signal-to-distortion figures of its own (TR §10.6). The in-scope quality checks (NFR-FUNC-001/002) are wired-wrong detectors, not quality claims | Evaluation against a labelled corpus per mode (e.g. speech mixtures for the speech branch) with standard separation metrics, on the shipped pipeline rather than the bare models |
| G15 | **Model training, fine-tuning, or domain adaptation** | §2.2 non-goal: both checkpoints are unmodified published artefacts; nothing was trained or adapted to this system's capture conditions (phone microphone, JPEG round-trip, LAN) | Training data and infrastructure, adaptation to the deployed capture chain, and revalidation of every locked-constant correctness experiment afterwards |
| G16 | **Automatic mode selection / out-of-domain safeguard** | The user carries the mode choice and nothing detects an out-of-domain scene (D7 cost, §8.9; TR §11.1) — acceptable for the operating team, not for end users | A validated scene/domain classifier or a per-mode competence estimate; today only the speech branch has any competence gate (FR-042) |

Two of these deserve emphasis because they interact. G2 is not an
engineering backlog item: no amount of optimisation of this pipeline
reaches assistive-grade latency while the models require seconds-long
windows — which is precisely the study's central finding (§2.2.3). And G1
is gated behind G2: testing assistive benefit on a system two to three
orders of magnitude outside latency tolerance would measure the wrong
thing. The honest product path runs through different models first, and
this study's contribution is establishing that with measurements rather
than opinion.

---

## 8. Architecture and design rationale

This section answers three questions for each architectural decision: what
was chosen, what else was considered, and which requirement it serves —
rationale that does not terminate in a requirement identifier is treated as
decoration and excluded. Costs are stated alongside benefits for every
decision; a rationale that lists only benefits is advocacy, and Section 8
is not that. Each decision is recorded in full as an Architecture Decision
Record in `docs/adr/` (standard form: context, decision, status,
consequences positive *and* negative); the subsections below summarise.
All ADRs are **retrospective**: they document decisions visible in the
working trees and the measurement record, dated to when they were recorded,
not when they were made.

### 8.1 The patterns as they exist in the code

Named from the code, not aspirationally; deviations are stated as facts.

**Backend layering.** Transport (`grpc_server.py`: service methods, message
codecs, per-stream loops) → per-session objects (`StreamingBuffer`,
`OverlapAddBuffer` instances, touch caches — constructed per stream from the
selected specification) → model registry and engine adapters
(`model_registry.py`, `engines/`) → inference internals (`inference.py`,
`pixel_cache.py`) → checkpoints on disk. Each layer's constants come from
the layer above it exactly once, at stream open.

**Strategy/plugin arrangement.** The registry-plus-engine mechanism is a
Strategy pattern with a plugin flavour: frozen `ModelSpec` objects resolved
by id select an engine implementing a common adapter contract; the stream
loop branches once, on `spec.mode`. Engines are process-wide singletons
loaded once (ADR-0005).

**Event-driven streaming with one lock.** The server is a single asyncio
process; each stream is an async generator loop; GPU work from both gRPC
request paths (`StreamProcess`, `ProcessVideo`) serialises through one
process-wide lock (FR-023). One recorded deviation: the legacy FastAPI
`/predict` endpoint runs GPU inference in the same process *without* taking
that lock (`src/main.py:102-108`) — no in-tree client exercises it, but it
narrows FR-023's status to *partially satisfied* and strengthens the G13
retire-or-maintain case. Concurrency is cooperative between chunks and
strictly serial at the device — which is why event-loop lag is this
system's collapse-predictor metric (NFR-PERF-006).

**State ownership.** Per-session: audio buffer, frame ring, overlap-add
buffers, touch cache ring, freeze/sticky/cluster latches. Process-wide: the
engine singletons, the inference lock — and, as a recorded deviation,
normalisation EMAs on the engine singletons that are shared across
concurrent same-mode streams: the input-gain and mask EMAs on the halves
path, the pixel-gain EMA across concurrent touch streams
(`src/inference.py:1375-1381`), and the CAM-scale EMA across concurrent
speech streams (`src/engines/multisensory_engine.py:146,253-259`). Together
these are the FR-020 *partially satisfied* finding — a direct cost of the
singleton choice (ADR-0005).

**Mobile: MVVM with Repository, and where it holds.** `MainViewModel` +
`GrpcVideoRepository` (cold Kotlin Flow over the generated stub) +
`GrpcModule` (channel and host preference). The wire-facing logic — model-id
staleness filtering, profile mirroring, sequence-gap handling — lives in the
ViewModel/Repository seam and is JVM-testable. Unidirectional data flow
holds for UI state and results; it factually breaks in two places: the
entire media pipeline (CameraX analysis, JPEG encode, `AudioRecord` loop,
`AudioTrack`/jitter-buffer lifecycle, chunk construction) lives in
`MainActivity`, and two `@Volatile` ViewModel fields are read directly by
the Activity outside observation (ADR-0008).

### 8.2 Decision summary

**Table 8.1 — The eight decisions.** Full records in `docs/adr/`.

| # | Decision | ADR | Principally serves | Principal cost |
|---|---|---|---|---|
| D1 | Client–server inference, not on-device | ADR-0001 | O1, O2; NFR-PERF-004/005, NFR-FLEX-001 | Network dependency, privacy exposure (S5), one-GPU concurrency ceiling |
| D2 | gRPC bidirectional streaming | ADR-0002 | FR-010–017; NFR-PERF-003/013, NFR-COMPAT-001 | Toolchain weight, TCP head-of-line under loss, client-side jitter buffering |
| D3 | Protocol Buffers, not JSON | ADR-0003 | NFR-COMPAT-001, NFR-PERF-003/013; UC-21 | Two-copy contract discipline, lite-runtime debuggability, comment/code drift |
| D4 | One process, both frameworks | ADR-0004 | NFR-PERF-010, FR-064; NFR-REL-006, NFR-COMPAT-003, NFR-FLEX-001 | Single failure domain, WSL2 coupling, loop-contention risk, unmeasured combined VRAM |
| D5 | Registry + engine interface | ADR-0005 | FR-060–066; NFR-MAINT-002/003, NFR-COMPAT-003 | Indirection, code-change-per-variation, singleton cross-session EMA state |
| D6 | Sliding window + overlap-add | ADR-0006 | FR-021/032; NFR-PERF-003, NFR-REL-005 | ≈ 47.6× compute multiplication (halves), half-window latency floor |
| D7 | User-selected switching, not fusion | ADR-0007 | FR-062–064; NFR-REL-006, NFR-INT-002 | User must pick the right mode; switch costs a refill (≈ 6 s halves) |
| D8 | MVVM + Repository on mobile | ADR-0008 | FR-045/052/063/065; NFR-INT-002 | Media pipeline still Activity-bound; UDF bypasses; no DI |

### 8.3 D1 — Client–server inference rather than on-device

The models decide this one. The multisensory checkpoint is ≈ 1.27 GiB of
TensorFlow-1-era graph running under `tf.compat.v1`; Sound of Pixels is
≈ 158.8 MiB of PyTorch; the multisensory forward pass costs 142.6 ms on a
GTX 1660 Ti and 1 991 ms on a desktop CPU (TR Table 13) — phone-class
silicon is on the wrong side of that 14× spread, before thermal budget.
Neither model has a validated mobile port (the PyTorch port skeleton has no
checkpoint, TR Table 1 item 12).

| Alternative | Why rejected |
|---|---|
| On-device inference (TFLite/ONNX port + quantisation) | No validated port exists; porting risk to model validity is the exact risk the TR §8.2 locked-constants record exists to avoid |
| Hybrid split (features on device, synthesis on server) | Still puts sustained neural compute on the phone; doubles the contract surface |
| Dedicated edge box beside the phone | Re-creates the server with worse logistics; no requirement served |

**Serves** O1, O2; NFR-PERF-004, NFR-PERF-005, NFR-FLEX-001. **Costs:**
network dependency (UC-07); a transport term in the lag budget; captured
audio and video leaving the device in plaintext — the S5 exposure and the
NFR-SEC-001/002 posture exist because of this decision; the one-GPU
serialisation ceiling (NFR-PERF-007); on-device and offline operation
foreclosed (FR-P03).

### 8.4 D2 — gRPC bidirectional streaming rather than REST polling, WebSocket, or RTP

A session is one ordered, bidirectional conversation: capture up at 8–38
messages/s, results down at 4–8/s, touch controls in-band with capture.

| Alternative | Why rejected |
|---|---|
| REST polling | No server push; per-call overhead at result cadence; touch controls need a second channel; no cross-call ordering |
| WebSocket | Untyped: schema, framing, status codes, and two-language codegen all hand-rolled — recreates gRPC badly |
| RTP/WebRTC media transport | Payloads are model-shaped, not codec frames; needs custom formats plus signalling; loss tolerance is wrong for an audio path that must be lossless (FR-011) |

**Serves** FR-010–FR-017, FR-061; NFR-PERF-003, NFR-PERF-013,
NFR-COMPAT-001/002. **Costs:** protoc/Gradle toolchain and stub-regeneration
discipline (UC-21); TCP head-of-line blocking under Wi-Fi loss (unmeasured —
an E-N gap); no media clocking, so pacing is the client's problem — the
adaptive jitter buffer (FR-051) is this decision's bill; plaintext is an
explicit channel setting (FR-P01).

### 8.5 D3 — Protocol Buffers rather than JSON

Payloads are mostly binary — PCM blocks, JPEGs, uint8 maps; a halves result
is ≈ 11.8 kB before compression at 8/s (TR §9.5). JSON would base64-inflate
every binary field by a third and parse strings at 16 messages/s uplink;
MessagePack/CBOR are binary-clean but schemaless (no codegen, no field
discipline); FlatBuffers buys zero-copy this system does not need at these
sizes.

**Serves** NFR-COMPAT-001 (byte-identical contract copies, mechanically
checkable), NFR-PERF-003/013, UC-21, FR-012/017. **Costs:** the two-copy
discipline itself; the Android `lite` runtime drops reflection; proto
comments are documentation, not code — the TR records seven comment/code
divergences (TR §6.4, Table 10).

### 8.6 D4 — One process hosting both frameworks

TensorFlow and PyTorch each allocate a CUDA context and cuDNN kernel images
— ≈ 1–1.3 GB of overhead per framework per process (TR §9.4). One process
pays it once; two processes pay it twice, which on a 6 GB card is the
difference between both models fitting and not. The earlier sidecar design
(separate services on ports 5556/5557) was abandoned and deleted for
exactly this arithmetic (HANDOFF §3.1); unload-on-switch was rejected
because a checkpoint reload costs the observed ≈ 4.2–4.5 s cuDNN autotune;
GPU partitioning does not exist on consumer Turing.

**Serves** NFR-PERF-010, FR-023, FR-024, FR-064; NFR-REL-006,
NFR-COMPAT-003, NFR-FLEX-001. **Costs:** one failure domain for all modes;
framework-version coupling that chains the full server to WSL2 (E-D serves
only the PyTorch modes); event-loop contention risk in one process —
NFR-PERF-006 exists to watch it; and the combined resident footprint has
never been measured — the TR's single largest unquantified risk, forced
into measurement by NFR-PERF-010.

### 8.7 D5 — Registry plus engine interface rather than conditional branching

Every per-stream object differs per model; encoding that as `if model ==`
branches would couple every model to every layer. The stronger argument is
evidential: deviating from locked constants produced confident,
plausible-looking, wrong output twice (TR §8.2), so per-model constants
must be frozen data, not tunable configuration — the registry is where they
freeze.

| Alternative | Why rejected |
|---|---|
| Conditional branching per model | N models × M touch points; every addition edits the core loop |
| Config/env-tunable model parameters | Invites exactly the "reasonable" deviations proven to produce silent wrongness |
| Inheritance without a registry | Still needs the id→class map; the registry is that map made explicit and testable |

**Serves** FR-060–FR-066, FR-024; NFR-MAINT-002 (two historical additions,
zero core-layer changes), NFR-MAINT-003, NFR-COMPAT-003. **Costs:**
indirection (spec → factory → singleton → module); every geometry variation
is a release; and engine singletons carry cross-session EMA state — the
FR-020 *partially satisfied* finding is this decision's recorded bill.

### 8.8 D6 — Sliding window with overlap-add rather than independent chunk inference

Both models are window-locked (5.944 s / 2.135 s); nothing shorter is a
valid input, and the record shows a changed window produces confidently
wrong output. Independent chunk inference therefore means either one result
per window (unusable cadence) or invalid inputs (not an option). The
deployed mechanism advances a full window by a small hop (125 ms / 250 ms)
and stitches outputs on an absolute sample timeline with raised-cosine
crossfades and violation counters.

**Serves** FR-021, FR-032, FR-033; NFR-PERF-003 (8 results/s from a 5.9 s
window — cadence decoupled from window length), NFR-REL-005. **Costs:**
compute multiplication — every output second is inferred ≈ 47.6× on the
halves branch (65 536/1 378) and ≈ 8.4× on speech, the duty-cycle bill on
speech (57–59 %, derived from the measured inference time) that caps
concurrency (NFR-PERF-007); and the
half-window lookahead that dominates the §2.2.3 lag estimates — inherited
from the models, not created here, but not removable here either.

### 8.9 D7 — User-selected switching rather than automatic fusion or gating

An earlier fusion design (the multisensory CAM gating Sound of Pixels
masks) was abandoned and its code deleted (HANDOFF §9.5). Three facts:
the domains are disjoint (instruments versus speech), so a gate blends a
valid answer with an out-of-domain one and no principled arbiter exists;
the capture profiles are incompatible, so one phone stream cannot feed both
models at once; and running both would double GPU work on a card already at
57–59 % duty on one branch. An automatic scene classifier was also
rejected: it adds an unvalidated third model whose silent-wrong-mode
failure is worse than an occasional wrong user pick.

**Serves** O2, O3; FR-062–FR-064; NFR-REL-006, NFR-INT-002 (labels can be
mode-truthful only because modes never mix). **Costs:** the user carries
the mode choice, and nothing detects an out-of-domain scene (TR §11.1) —
carried into the gap analysis as G16; a switch costs teardown + 700 ms
settle + a fresh window fill (≈ 6.6 s on halves); the model-id
echo-and-drop machinery exists only because of switching.

### 8.10 D8 — MVVM with Repository on the mobile side

The alternatives — Activity-owns-everything (no testable seam), MVI/Redux
(reducer ceremony wrong for continuous binary streams), Compose (a view
rewrite orthogonal to this app's problems), full Clean Architecture
(layering overhead for one screen) — were each worse fits. The honest
account is that the chosen pattern **partially holds**: contract logic
lives in the testable ViewModel/Repository seam, while the media pipeline
remains Activity-bound and two `@Volatile` fields bypass observation
(§8.1). Those deviations are recorded in ADR-0008 as consequences, and one
of them is load-bearing for a finding: FR-006's missing lifecycle handling
sits exactly in the layer the pattern left inside the Activity.

**Serves** FR-045, FR-052, FR-063, FR-065; NFR-INT-002; O5. **Costs:**
NFR-INT-001 is instrumented-only because the relevant path is not
JVM-testable; no dependency injection makes test substitution awkward.

### 8.11 Decision-to-requirement mapping

**Figure 5 — Architectural decisions mapped to the NFRs they principally
serve.** Full serves/costs lists per decision are in Table 8.1 and the
ADRs. (Source: `docs/diagrams/ar-figure-05-decisions-to-nfrs.mmd`;
rendered: `docs/diagrams/svg/ar-figure-05-decisions-to-nfrs.svg`.)

```mermaid
flowchart LR
    D1["D1 Client-server<br/>inference (ADR-0001)"]
    D2["D2 gRPC bidirectional<br/>streaming (ADR-0002)"]
    D3["D3 Protocol Buffers<br/>(ADR-0003)"]
    D4["D4 One process, both<br/>frameworks (ADR-0004)"]
    D5["D5 Registry + engine<br/>interface (ADR-0005)"]
    D6["D6 Sliding window +<br/>overlap-add (ADR-0006)"]
    D7["D7 User-selected<br/>switching (ADR-0007)"]
    D8["D8 MVVM +<br/>Repository (ADR-0008)"]

    P3(["NFR-PERF-003<br/>cadence"])
    P4(["NFR-PERF-004<br/>halves cycle time"])
    P5(["NFR-PERF-005<br/>speech cycle time"])
    P10(["NFR-PERF-010<br/>both-resident VRAM"])
    P13(["NFR-PERF-013<br/>result continuity"])
    R5(["NFR-REL-005<br/>soak stability"])
    R6(["NFR-REL-006<br/>switch churn"])
    C1(["NFR-COMPAT-001<br/>proto identity"])
    C3(["NFR-COMPAT-003<br/>TF-absent service"])
    I2(["NFR-INT-002<br/>label integrity"])
    M2(["NFR-MAINT-002<br/>model isolation"])
    M3(["NFR-MAINT-003<br/>frozen constants"])
    F1(["NFR-FLEX-001<br/>degraded portability"])

    D1 --> P4
    D1 --> P5
    D1 --> F1
    D2 --> P3
    D2 --> P13
    D2 --> C1
    D3 --> C1
    D3 --> P3
    D3 --> P13
    D4 --> P10
    D4 --> R6
    D4 --> C3
    D4 --> F1
    D5 --> M2
    D5 --> M3
    D5 --> C3
    D6 --> P3
    D6 --> R5
    D7 --> R6
    D7 --> I2
    D8 --> I2
```

---

## 9. Verification and validation strategy

### 9.1 Method per requirement class

Every requirement in Sections 4–5 names its verification method (test,
analysis, inspection, demonstration — §4.0). The strategy behind those
assignments:

- **Automated tests** carry everything a machine can decide: the existing
  backend unit suites (UC-20), the load suite of Section 10 for every
  measurable NFR, and — Phase 6 — the mobile automated layers. The load
  suite's assertions are generated from `docs/nfr/nfr_targets.yaml` at run
  time, so the requirement text, the threshold file, and the test verdict
  cannot drift apart (NFR-MAINT-004; the smoke scenario mechanically
  re-checks the §5 tables against the YAML on every run).
- **Analysis** carries what only reasoning over artefacts can decide:
  NFR-MAINT-002 (model-addition isolation, assessed over the two historical
  change sets) and NFR-REL-007's documented fallback if OOM injection
  proves infeasible.
- **Inspection** carries structural facts: frozen-constant discipline,
  endpoint confinement, capture-persistence gating, licence and posture
  statements.
- **Demonstration** carries end-to-end behaviour on real devices where
  instrumentation would perturb the thing observed (live sessions;
  degraded-mode startup on E-D).

### 9.2 Automated versus manual, and what stays unverified

Automated now (E-D): the smoke subset (proto identity, unit suite,
report↔YAML match, default-model and health probes) and the seven load-suite
scenarios for the two PyTorch modes. Automated later (Phase 6): the mobile
layers (JVM, Robolectric, instrumented, matrix). Manual by design: listening
judgements — the pixel-mode A/B decisions explicitly await a human listen
(UC-19), and Phase 6 specifies a structured manual listening protocol
modelled on the paper's own human evaluation.

Deliberately unverified, stated per the §1.5 discipline: every
E-M-conditional target while no TensorFlow host is available
(NFR-PERF-002/005/010, NFR-FUNC-002, NFR-FLEX-001's full-set half); every
matrix-conditional target until the Phase 6 matrix is pinned
(NFR-INT-001/003, NFR-FLEX-002/003); separation *quality* beyond the
wired-wrong detectors (G14); and assistive benefit of any kind (G1).

### 9.3 The verification chain for measured claims

A measured number enters this document only through the chain: scenario run
→ `results/<runid>/results.json` (with automatic run metadata: repository
heads, checkpoint hashes, GPU/driver, package versions, YAML hash) →
generated `summary.md` → citation in Section 10 with its run id. A result
without metadata is not citable; targets and measurements reconcile in the
open per §5.0.

---

## 10. Backend performance verification

### 10.1 The suite

The load and stress suite exists at `SonicSightBackend/loadtest/` (built in
Phase 4; run in Phase 5 on both reference hosts — §10.3).
It is an asyncio driver over the project's own generated stubs,
generalising the validated `replay_client.py` protocol to N concurrent
sessions, failure injection, and model switching. The tool question was
weighed deliberately: `ghz` cannot express paced two-message-per-tick
bidirectional capture or per-result metric extraction; `k6` requires the
xk6-grpc extension build and models streams awkwardly; Locust is HTTP-first
and adds distributed-worker machinery a single-host study does not need.
The custom driver reads its capture profiles from the server's own registry
(`loadtest/driver.py`), so the harness cannot drift from the wire contract;
the full justification paragraph sits at the top of that file.

Design properties that carry the study's claims:

- **Thresholds are read, never written.** Every assertion is constructed at
  run time from `docs/nfr/nfr_targets.yaml`; the suite contains no
  threshold literal (NFR-MAINT-004 — the no-literal half is discharged by
  inspection, declared in the YAML rather than faked as a self-reported
  count), and its smoke scenario mechanically counts §5↔YAML mismatches on
  every run. The matcher was exercised standalone while building the suite
  and surfaced two real drifts, both fixed; the citable count comes from
  the Phase 5 run.
- **Event-loop lag is measured from inside the server's loop** without any
  server source change: `loadtest/server_wrapper.py` runs the unmodified
  server with a 20 Hz sleep-drift sampler task on the same asyncio loop
  (NFR-PERF-006's condition), writing a timestamped CSV.
- **Run metadata is automatic**: repository heads and dirty flags,
  checkpoint sizes and hashes, GPU/driver identification, Python and
  framework versions, the YAML's own hash, timestamp, and full scenario
  parameters land in every `results.json`; §9.3 makes metadata a
  precondition of citability.
- **Deterministic media**: scenarios use a seeded synthetic source
  (procedural tones and moving shapes at each profile's exact rates); the
  one requirement needing real content (NFR-FUNC-001) uses an
  operator-supplied reference clip whose SHA-256 enters the metadata.
- **Conditional targets are skipped loudly.** E-M-conditional and
  matrix-conditional assertions appear in every summary as *skipped with
  reason*, never dropped.
- **The perceived-lag proxy is defined, not implied**: result arrival time
  minus send time of the capture chunk whose timestamp the result echoes —
  capture-to-result age at the driver, excluding playback buffering;
  reported as characterisation per §5.9.

### 10.2 Scenario-to-requirement map

| Scenario (`loadtest.run` name) | Exercises |
|---|---|
| `smoke` | NFR-COMPAT-001/002/003, NFR-MAINT-001, NFR-MAINT-004, NFR-FLEX-001 (both halves; the E-M half asserts the full served set) |
| `baseline` | NFR-PERF-001/003/004/006/008/013; NFR-FUNC-001 (with clip); E-M: PERF-002/005/015, FUNC-004 (FUNC-002 stays with its offline probe protocol) |
| `load` | NFR-PERF-007 (floor), NFR-PERF-009/010/013; NFR-SEC-001 (traffic observation); E-M: PERF-016 |
| `stress` | NFR-PERF-007 (ceiling characterisation — the knee) |
| `spike` | NFR-PERF-014 |
| `soak` | NFR-PERF-011/012, NFR-REL-005, NFR-SEC-001 |
| `failure-injection` | NFR-REL-001/002/003/004 (NFR-REL-007 semi-manual, per its fallback) |
| `switching` | NFR-REL-006 |

The expected concurrency ceiling is low — one GPU and a process-wide lock
serialise all inference (FR-023, ADR-0004) — and the stress scenario's job
is to name it precisely rather than impressively.

Targets whose scenario lies outside the load suite (`unit`, `inspection`,
`analysis`, `mobile-*`: NFR-FUNC-003, NFR-INT-001–004, NFR-SEC-002,
NFR-MAINT-002/003, NFR-FLEX-002/003) are appended to every run's summary as
*not-evaluable-by-this-suite* rather than vanishing; the unit-scenario
subset (FUNC-003, INT-002, INT-004, MAINT-003) will be discharged by
dedicated unit-test scaffolding in Phase 5, and the mobile subset by
Phase 6.

### 10.3 Measured results

Two campaigns exist. The E-D campaign (2026-08-08/09, GTX 1650 host; runs
`loadtest/results/p5-*`) is audited in `docs/nfr/P5_VALIDATION.md`: its
measured values stand, but six of nine runs were asserted against a
superseded targets file that was later edited mid-campaign, its stress
run's "ceiling = 1" is a documented measurement artifact, and all nine runs
record a dirty working tree whose harness state corresponds to the
later-committed `59adf8a`. The E-M campaign (2026-08-10, GTX 1660 Ti /
WSL2 / TF 2.21 host; runs `loadtest/results/em-*`, one fresh server
process throughout, ~3.5 h) is summarised per scenario, with the full
verdict table and staging data, in `docs/nfr/EM_RESULTS.md`. The §5 status
cells above quote both. Headline results:

- **Halves mode passes every baseline target on E-M with wide margins**
  (TTFR p95 6.22 s; cadence dev p95 3.2 ms; server p95 50 ms; loop lag
  p99 ≈ 2 ms) and the **concurrency floor of 2 holds; the measured
  ceiling is 3** (`em-stress`: 2/2 at n = 2; 0/3 at n = 3, breached by
  tail spikes at server-time p95 124 ms — the knee is spike-shaped, not
  throughput-shaped; median GPU utilisation stayed ≈ 32 %).
- **Speech mode meets its time-to-first-result and cycle-time targets**
  (p95 2.77 s and 152 ms) and its structural contract, and the
  both-engines-resident VRAM risk (TR §9.4) is quantified and closed
  (campaign peak 2 650 MiB used / 3 317 MiB free); **its cadence target
  is failed by a located server defect** (§10.4, D-P5-1) — the effective
  stride is 371.5 ms against the 250 ms design hop.
- **The reliability set passed its first-ever execution end to end**
  (containment, oversized rejection, degenerate media, disconnect
  recovery, cross-framework switch churn; 30-minute speech soak with RSS
  slope 0.086 MiB/min and zero unhandled exceptions).
- **The replay separation-sanity check failed** with a deterministic
  Pearson 0.961 (§10.4, D-P5-2) — the study's most consequential
  measured defect.
- Two targets were **revised in the open** after measurement, with
  reasons recorded in the YAML (`NFR-PERF-004`, `NFR-PERF-012`; §5
  status cells carry the summaries); the speech cadence/capacity targets
  were deliberately **not** softened — their FAILs pin D-P5-1.
- Environment-dependence finding: NFR-PERF-007/008/014 pass on E-M and
  fail on E-D — the 4 GiB card saturates, the 6 GiB card does not. The
  E-D failures are host-capacity characterisations, not architecture
  defects.

Raw multi-megabyte captures (server log, loop-lag CSV) are summarised in
`EM_RESULTS.md` rather than committed, per the working rules; every
committed `results.json` carries its full metadata block including the
exact `nfr_targets.yaml` hash it asserted against.

### 10.4 Defects found by measurement (Phase 5)

| ID | Defect | Evidence | Consequence | Disposition |
|---|---|---|---|---|
| D-P5-1 | Speech-mode window stride was 371.5 ms, not the designed 250 ms — two interacting causes, isolated by instrumented stride dumps (`window_delta` histograms) after an initial single-cause hypothesis (a 256-alignment argument) was refuted by measurement: (1) inference was awaited inline in the ingest loop, so each cycle's wall cost was inference (~150 ms) plus the serialized drain of every message queued during it, leaving no headroom at the 250 ms hop; (2) window candidates land on the 2 756-sample audio-chunk grid *minus* up to ~990 samples of slack (30 fps frame-timestamp graininess + the 256-sample STFT snap), so a minimum-advance guard at the hop itself rejected the 2-chunk (250 ms) candidate about 2 of 3 times, degrading the stride to 3 chunks | Stride dumps `stride-dump`/`stride-dump2`/`stride-dump3` (`cycles.csv`); `em-baseline-ms` p50 373.5 ms before, 250.0 ms after | Speech throughput was 2.67 results/s vs 4 designed (−33 %); NFR-PERF-015/016 NS before the fix | **Fixed 2026-08-10, verified**: inference now runs as a single in-flight task overlapping ingest (`grpc_server.py`), and `window_min_advance` is 4 480 (= hop − worst-case slack; still forbids the 1-chunk runaway). Measured after: interval p50 250.0 ms, cadence dev p95 1.8 ms. Campaign-3 re-measurement under full scenario conditions recorded in `EM_RESULTS.md` |
| D-P5-2 | *(Reclassified 2026-08-10 after systematic investigation — measurement artifact, not a pipeline defect.)* The Pearson 0.961 replay measurements (clip SHA `e2aa923e9318af7a`) were taken on a **non-conforming reference clip**: a two-speaker speech-debate video, outside the Sound of Pixels model's domain (musical instruments, MUSIC dataset — §2.2's own scoping). Investigation: the offline path on current code reproduces the same failure (Pearson 0.984) on this clip, so no streaming-vs-offline divergence exists; the frames show two talking heads whose pooled vision features are legitimately near-identical (cosine 0.946, no saturation); and the "+0.263 offline reference" hard-coded in `replay_client.py` describes a *different, tonal-music* clip by its own spectral profile (flatness 0.183, 81 % energy in 150–600 Hz) — that clip is not present on either host. The alternative pixel-level conditioning mode was also tested and did not separate this content either (0.995) — consistent with out-of-domain input, and reverted | `em-baseline-replay`; offline reproduction and frame inspection 2026-08-10 | NFR-FUNC-001 reverts to *unmeasured*: no conforming (two-instrument) reference clip exists on either host, so the requirement is undischargeable until the operator supplies one | No code change (none warranted by evidence). Action: obtain an in-domain two-instrument clip and re-run the three-replay protocol; `replay_client.py`'s hard-coded reference block should be regenerated for that clip |
| D-P5-3 | Rare isolated 1.2–1.5 s server-time stalls, ~once per 10–12 min of streaming, on both reference hosts, while p99 stays under 110 ms | `p5-stress` (1 338/1 476 ms), `em-load` (1 258 ms among 11 426 results) | Breached the original PERF-004 max gate on otherwise clean runs | Bounded by the recorded PERF-004 reconciliation; cause unlocalised — future work |
| D-P5-4 | Server emitted no stream-close log line, making REL-002's open/close parity unmeasurable | `em-failure` parity outcome *not-measured* | Orphan detection blind | **Fixed 2026-08-10, verified**: close line via `context.add_done_callback` (fires on every termination); parity re-measured at 0 imbalance over 67 opens incl. 50 abrupt disconnects (`em2-failure`) |
| D-P5-5 | **Open.** Sustained halves-mode streaming grows GPU device memory ~34 MiB/min: +676 MiB over the 55-minute dump soak and +1038 MiB over the 30-minute no-dump soak (recorder exonerated — its effect is host RSS, not VRAM). The speech soak grew only +93 MiB, so the growth is specific to the SoP/torch path. Unbounded, it would exhaust the 6 GiB card in ~2 h of continuous streaming | `em2-soak-halves`, `em3-soak-clean` resource series | NFR-PERF-012 NS for the halves path; long-session viability bounded | Recorded open; next step is a ≥ 2 h instrumented soak (`torch.cuda.memory_summary` sampling) to separate allocator-pool plateau from unbounded growth, then the allocator fix if unbounded |

---

## 11. Mobile compatibility verification

*(pending — Phase 6. Summarises `docs/MOBILE_VERIFICATION_PLAN.md`.)*

---

## 12. Risk analysis

*(pending — Phase 7. Includes the over-reliance risk identified in §3, and
the defects catalogued while preparing this study.)*

---

## 13. Ethical and privacy considerations

*(pending — Phase 7. Bystander capture, data leaving the device, retention,
the trust boundary, the always-on microphone dump defect, and the ethics
approval any future d/DHH user testing would require — the latter an explicit
placeholder decision.)*

---

## 14. Conclusions and future work

*(pending — Phase 7.)*

---

## References

*(Initial list; completed and verified in Phase 7. Entries marked ⟦pending⟧
have not yet been verified against primary sources and are not cited as fact
in the body.)*

1. H. Zhao, C. Gan, A. Rouditchenko, C. Vondrick, J. McDermott, and
   A. Torralba, "The Sound of Pixels," ECCV 2018.
2. A. Owens and A. A. Efros, "Audio-Visual Scene Analysis with
   Self-Supervised Multisensory Features," ECCV 2018 (arXiv:1804.03641v2).
3. Regulation (EU) 2017/745 (Medical Device Regulation). ⟦pending
   verification⟧
4. U.S. FDA, Medical Devices; Ear, Nose, and Throat Devices; Establishing
   Over-the-Counter Hearing Aids, 21 CFR Parts 800/801/808. ⟦pending
   verification⟧
5. M. A. Stone and B. C. J. Moore, "Tolerable Hearing Aid Delays. I.
   Estimation of Limits Imposed by the Auditory Path Alone Using Simulated
   Hearing Losses," *Ear and Hearing*, vol. 20, no. 3, pp. 182–192, 1999.
6. M. A. Stone and B. C. J. Moore, "Tolerable Hearing Aid Delays. II.
   Estimation of Limits Imposed During Speech Production," *Ear and
   Hearing*, vol. 23, no. 4, pp. 325–338, 2002. (Series parts III–V, 2003–
   2008, extend these findings; the working bounds cited in §2.2.3 are the
   series' consensus figures.)
7. Recommendation ITU-R BT.1359-1, "Relative Timing of Sound and Vision for
   Broadcasting," International Telecommunication Union, 1998.
8. ISO/IEC 25010:2023, "Systems and software engineering — Systems and
   software Quality Requirements and Evaluation (SQuaRE) — Product quality
   model." ⟦edition details to be confirmed in the Phase 7 references pass⟧

---

## Appendix A — Requirements traceability matrix

*(pending — Phase 7. One row per requirement: requirement ID → use case →
verification method → test identifier → implementing code.)*

---

## Appendix B — Use case specifications

Full-form specifications, one per use case. *Requirements exercised* rows
reference the Section 4 and Section 5 identifiers assigned in Phase 2.
Stakeholder identifiers (S1–S8) refer
to Table 3.1. *Primary evidence* rows give the `file:line` anchors from which
the behaviour was read; paths are relative to the repository named.

---

### UC-01 — Start and stop a listening session

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server; GPU/CUDA stack |
| Stakeholders and interests | S2 (working demonstrator), S3 (the central demonstrated capability), S5 (capture begins — bystander exposure starts here) |
| Preconditions | CAMERA and RECORD_AUDIO granted; backend reachable at the configured address with the selected model loaded; app in foreground, landscape |
| Postconditions | Stream closed after a final `is_last` chunk; server has flushed the overlap-add tail; playback tracks and capture released; UI returns to idle |
| Frequency | Several times per interactive session; the system's core loop |
| Status | Implemented. End-to-end validated only for the `sonicsight` mode, and the conditions of those recordings are unrecorded (TR Table 1, item 1); `multisensory` and `sonicsight-pixel` end-to-end runs are ungated |

**Main success scenario** (halves mode; speech and touch modes differ per
UC-02's capture profiles):

1. The Listener taps the record control.
2. The client creates fresh outbound flows and opens a gRPC bidirectional
   stream, naming the selected model in call metadata (UC-02).
3. The client captures frames (8 fps, split into two 224×224 half-frame
   JPEGs) and audio (44 100 Hz mono, decimated in-app to 11 025 Hz), emitting
   one audio chunk every 125 ms and one frame chunk per captured frame.
4. The server accumulates chunks in a per-stream buffer; until a full 65 536-
   sample window exists it returns buffering results, which the client
   renders as a progress hairline over the expected fill time (≈ 6 s).
5. For each full window the server runs inference (serialised by a global
   lock), stitches output through per-stream overlap-add buffers, and streams
   results carrying separated PCM, two heatmaps, and a sequence number.
6. The client feeds left/right PCM through adaptive jitter buffers (200 ms
   initial, adapting to 500 ms, over a 1 500 ms ring) into two hard-panned
   `AudioTrack`s, and renders the overlay (UC-04).
7. The Listener taps stop; the client sends a final chunk with `is_last`,
   the server yields the flushed tail, and both sides tear down.

**Extensions.** 3a. Sequence gaps in results are patched with silence chunks.
5a. Early-inference mode (halves mode only) produces provisional results on a
zero-padded window before the buffer fills; the server skips overlap-add for
these to protect the client's jitter buffer.

**Exceptions.** E1: permissions denied — the app shows a toast and exits
(UC-24's accessibility concerns notwithstanding, there is no recovery UI).
E2: no audio source can be opened — capture proceeds video-only with no
user-facing error, logged only (recorded as a defect, TR §11.4). E3: server
unreachable — UC-07. E4: GPU out of memory — the server returns an error
result; the client surfaces it and auto-stops. E5: the app has no
`onPause`/`onStop` handling — backgrounding does not stop capture (recorded
defect, TR §11.4).

**Requirements exercised.** FR-001–FR-006, FR-010, FR-011, FR-013, FR-017, FR-020–FR-026, FR-030, FR-032, FR-033, FR-050–FR-052; NFR-PERF-001, NFR-PERF-003, NFR-PERF-004, NFR-PERF-013, NFR-REL-005, NFR-REL-007, NFR-INT-001.

**Primary evidence.** SonicSightMobile `MainActivity.kt:133-139` (toggle),
`:976-984,1069-1106` (audio loop), `:852-870` (frames), `:1140-1144` (stop);
`MainViewModel.kt:227-235` (stream), `:304-313` (gap fill);
`JitterBuffer.kt:30,37,45,243-247`; SonicSightBackend
`src/grpc_server.py:83-119` (per-stream state), `:243-260` (buffering),
`:396-423` (final flush); `src/inference.py:258-317` (early mode).

---

### UC-02 — Select a listening mode

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server (validates the selection at stream open) |
| Stakeholders and interests | S2, S3 (the switchable-models decision is the architecture's centre); S1 (mode labels must not misdescribe what is heard) |
| Preconditions | App open; a session may or may not be active |
| Postconditions | The active profile — capture rates, frame geometry, playback rate, overlay form, track labels — matches the selection; the next stream opens with the matching metadata |
| Frequency | A few times per session |
| Status | Implemented, not validated: the server selection branch has 13 unit tests, but the on-device end-to-end switch is ungated (TR Table 1, item 5) |

**Main success scenario:**

1. The Listener chooses one of three modes: Music & Instruments
   (`sonicsight`), Speech (`multisensory`), or Touch (`sonicsight-pixel`).
2. The client activates the matching profile: 8 fps half-frames at 11 025 Hz;
   30 fps letterboxed full frames at 22 050 Hz; or 8 fps letterboxed full
   frames at 11 025 Hz, respectively.
3. The playback labels update from the profile ("Left/Right" versus
   "On-screen/Off-screen"), because the speech mode's two tracks are not
   spatial and labelling them spatially would be false (TR §3.5).
4. At the next stream open, the client sends the profile's id under the
   `sonicsight-model` metadata key; the server resolves it against the
   registry and shapes all per-stream state from the matching specification.

**Extensions.** 1a. Selection during an active session triggers UC-03.

**Exceptions.** E1: the id is unknown or its engine is not loaded — the
server aborts the stream with `FAILED_PRECONDITION`; the client maps this to
a plain-language message naming the other model (UC-07 path). E2: no
metadata sent (legacy client) — the server defaults to `sonicsight`.

**Requirements exercised.** FR-012, FR-060, FR-065, FR-066; NFR-COMPAT-002, NFR-INT-002.

**Primary evidence.** SonicSightMobile `MainActivity.kt:142-151`,
`ModelProfile.kt:51-99`, `GrpcVideoRepository.kt:49-60`; SonicSightBackend
`src/model_registry.py:21-25,92-189`, `src/grpc_server.py:54-67`.

---

### UC-03 — Switch mode mid-session

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S2, S3; S4 (the cancel-and-reopen contract is what keeps per-model state simple) |
| Preconditions | A session is active (UC-01) |
| Postconditions | Old stream closed; new stream open under the new model id; no result from the old model is rendered after the switch |
| Frequency | Occasional |
| Status | Implemented, not validated end to end on a device |

**Main success scenario:**

1. The Listener selects a different mode while streaming.
2. The client stops the current stream (as UC-01 step 7) — the two models'
   capture profiles are incompatible, so nothing buffered is reusable and
   switching is by design a cancel-and-reopen (TR §3.4).
3. The client waits a 700 ms settle delay.
4. The client reopens the stream with the new model's metadata (UC-02).
5. Results still in flight from the old stream carry the old model id; the
   client discards any result whose echoed `model_id` does not match the
   current selection.

**Exceptions.** E1: the new model is not loaded on the server —
`FAILED_PRECONDITION`, surfaced with a message suggesting the other model;
the session stays stopped.

**Requirements exercised.** FR-061–FR-064; NFR-REL-006.

**Primary evidence.** SonicSightMobile `MainActivity.kt:191-202`,
`MainViewModel.kt:275-278`; SonicSightBackend `src/grpc_server.py:52-67`;
TR §3.4, Figure 8.

---

### UC-04 — View the localization overlay

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S1 (the overlay is the "where" answer an assistive reading would depend on — and the reason honesty about its coarseness matters), S2 |
| Preconditions | Session active; results arriving |
| Postconditions | Overlay reflects the newest result at the mode's map rate (8 Hz halves/touch, 4 Hz speech) |
| Frequency | Continuous during every session |
| Status | Implemented; the speech mode's confidence gate is implemented, not validated |

**Main success scenario** (varies by mode):

1. *Halves:* the client receives two 56×56 uint8 maps (side inferred from
   byte count), renders each through the magma colour ramp with a
   gamma-shaped alpha so low activations vanish, stitches them side by side,
   and registers the result over the camera preview.
2. *Speech:* one full-frame map arrives; the client renders it the same way.
3. *Touch:* a native 14×14 energy map with explicit wire dimensions arrives,
   optionally tinted per discovered cluster, and is registered onto the
   letterboxed preview via an affine mapping that inverts the letterbox.

**Extensions.** 2a. **Confidence gate:** when the speech model's class
activation map (CAM) confidence falls below 0.10, the server withholds the
map (sends it empty, reporting the raw confidence); the client clears the
overlay and shows a "no reliable localization" label while audio continues.
The threshold exists because the alignment head is not a saliency map and an
all-negative CAM means "this audio does not match this video" (TR §8.3).

**Exceptions.** E1: malformed or absent map bytes — the overlay is simply
not updated for that result.

**Requirements exercised.** FR-040–FR-043, FR-045; NFR-FUNC-002, NFR-FUNC-003.

**Primary evidence.** SonicSightMobile `MaskProcessor.kt:100-147,160-197`,
`MainViewModel.kt:377-380,394-399`, `MainActivity.kt:215-250,747-757`;
SonicSightBackend `src/engines/multisensory_engine.py:44-50,242-259`.

---

### UC-05 — Solo one separated track

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | — |
| Stakeholders and interests | S1 (isolating one source is the assistive gesture in miniature), S2 |
| Preconditions | Session active in a two-track mode (halves or speech); the control is hidden in touch mode |
| Postconditions | One track audible, or both |
| Frequency | Frequent within a session |
| Status | Implemented, not validated |

**Main success scenario:**

1. The Listener selects Both, or one of the two named tracks; the names come
   from the active profile ("Left/Right" or "On-screen/Off-screen").
2. The client sets per-track volume on the two `AudioTrack`s accordingly;
   in Both, the two mono tracks play hard-panned left and right.

**Exceptions.** None beyond session loss (UC-07).

**Requirements exercised.** FR-053; NFR-INT-002.

**Primary evidence.** SonicSightMobile `MainActivity.kt:174-183,533-551,
592-608,624-636`.

---

### UC-06 — Configure the server address

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | — |
| Stakeholders and interests | S2; S5 (the address determines where captured audio and video are sent) |
| Preconditions | No session active — the dialog refuses changes while recording |
| Postconditions | New host persisted; the channel is rebuilt lazily on next use; port remains fixed at 50051 |
| Frequency | Rare — once per network change |
| Status | Implemented, not validated |

**Main success scenario:**

1. The Listener opens settings and enters a host address.
2. The client persists it in `SharedPreferences` and tears down the cached
   channel so the next stream connects to the new host.

**Exceptions.** E1: attempted while recording — the dialog is blocked with
"Stop listening before changing the server."

**Requirements exercised.** FR-071; NFR-SEC-001.

**Primary evidence.** SonicSightMobile `MainActivity.kt:464-482`,
`GrpcModule.kt:12-44` (compiled-in default `192.168.1.87`, port 50051).

---

### UC-07 — Recover from server unavailability

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server (by its absence) |
| Stakeholders and interests | S2; S1 (an assistive reading would make graceful degradation safety-relevant — recorded in §12) |
| Preconditions | The Listener attempts to start or continue a session and the server cannot be reached, or rejects the model |
| Postconditions | Session stopped; a plain-language message names the failing host and the remedy; the Listener can retry or change the address |
| Frequency | Whenever the LAN, the WSL2 bridge (UC-17), or the server is down |
| Status | Implemented, not validated |

**Main success scenario:**

1. The stream fails with `UNAVAILABLE`, an I/O exception, or a deadline.
2. The client maps the raw error to "Can't reach the server at ⟨host⟩. Check
   Wi-Fi, or change the address in settings," auto-stops the session, and
   returns to idle.
3. The Listener corrects the cause — network, server (UC-12), or address
   (UC-06) — and starts again (UC-01).

**Extensions.** 1a. `FAILED_PRECONDITION` ("not loaded" / "Unknown model") —
the message becomes "⟨model⟩ isn't loaded on this server. Try the other
model." 1b. Any other error — a generic message carrying the server's text.

**Requirements exercised.** FR-061; NFR-INT-004.

**Primary evidence.** SonicSightMobile `MainActivity.kt:691-694,776-788`;
`MainViewModel.kt:229-231,280-285`.

---

### UC-08 — Tap or drag to hear a region (touch mode)

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S1 (hear-what-you-point-at is the study's most direct assistive gesture), S2, S3 |
| Preconditions | Touch mode streaming (UC-01 under `sonicsight-pixel`); at least one analysis window cached server-side |
| Postconditions | The region's synthesized audio (a 2.0 s tail) has played; the live mixture ducks during playback and restores after |
| Frequency | The touch mode's core interaction; many times per session |
| Status | Implemented, not validated: the path ran live once (2026-08-05, producing the calibration constants now in the code) but none of the numeric gates in the test plan have been executed (TR Table 1, item 6) |

**Main success scenario:**

1. The Listener taps a point on the preview.
2. The client sends a `PixelQuery` (normalised coordinates, disc radius,
   `window_id` 0 = newest) on the outgoing stream — at most 4 queries per
   chunk.
3. The server answers from the per-window feature cache with no new forward
   pass: a region-pooled synthesis over the cached spectrogram, returning
   the last 2.0 s as PCM with a relative energy value.
4. The client ducks the live mixture to 0.15 volume, plays the region audio
   on a separate static track, and restores the mixture afterwards.

**Extensions.** 1a. *Drag:* the client issues one query per 14×14 grid cell
the finger crosses, sweeping the scene. 3a. *Silence gate:* if the region's
measured contrast is below the silence threshold the server returns zero
energy instead of leakage, and the client shows "No sound detected here."

**Exceptions.** E1: more than 4 queries in one chunk — the server answers
with a query-cap error. E2: the queried window has been evicted from the
3-slot ring — "window not available"; the client invites the Listener to
touch again. E3: the client-side silence threshold (`QUERY_SILENCE_ENERGY`)
is explicitly provisional — behaviour at the margin is uncalibrated.

**Requirements exercised.** FR-016, FR-035, FR-036, FR-054; NFR-PERF-008.

**Primary evidence.** SonicSightMobile `MainActivity.kt:262-332,341-354,
363-395`; SonicSightBackend `sonicsight.proto:96-120`,
`src/grpc_server.py:496-501,579-663`, `src/pixel_cache.py:172-249`.

---

### UC-09 — Follow a region: sticky long-press (touch mode)

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S1, S2 |
| Preconditions | As UC-08 |
| Postconditions | While sticky follow is active, the streamed "mixture" track is replaced by the selected region's synthesis, window after window |
| Frequency | Occasional within a touch session |
| Status | Implemented, not validated |

**Main success scenario:**

1. The Listener long-presses (450 ms) a region.
2. The client marks the query sticky; the server records the region's
   weights and, for every subsequent window, replaces the live mixture
   output with the region synthesis.
3. The Listener releases the follow via the client's clear-sticky control;
   the server reverts to streaming the plain mixture.

**Extensions.** 2a. Combined with freeze (UC-10), the followed region is
evaluated against the pinned window rather than the advancing one.

**Requirements exercised.** FR-016, FR-035; NFR-PERF-008.

**Primary evidence.** SonicSightMobile `MainActivity.kt:262-332`;
SonicSightBackend `sonicsight.proto:104-109` (`sticky`), `:184-186`
(`clear_sticky`), `src/grpc_server.py:529-533,704-716`.

---

### UC-10 — Freeze the energy map (touch mode)

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S1 (a frozen map makes deliberate exploration possible), S2 |
| Preconditions | Touch mode streaming |
| Postconditions | The displayed energy map and the query target are pinned to one window while frozen; live capture continues underneath |
| Frequency | Occasional |
| Status | Implemented, not validated |

**Main success scenario:**

1. The Listener toggles freeze.
2. The client stamps the level-triggered `freeze` flag on every outgoing
   chunk.
3. On the rising edge the server pins the newest cached window — pinned
   entries are exempt from ring eviction — and keeps the pin alive under a
   16-chunk time-to-live latch; queries resolve against the pinned window.
4. The Listener unfreezes; the pin is released and the map resumes
   advancing.

**Requirements exercised.** FR-016, FR-043.

**Primary evidence.** SonicSightMobile `MainActivity.kt:153-158,925,1099`;
SonicSightBackend `src/grpc_server.py:516-518,555-566`,
`src/pixel_cache.py:85-137`.

---

### UC-11 — Solo a discovered source (touch mode)

| Field | Content |
|---|---|
| Primary actor | Listener |
| Supporting actors | Backend server |
| Stakeholders and interests | S1 (source-level solo without knowing where to tap), S2 |
| Preconditions | Touch mode streaming; source discovery returning clusters |
| Postconditions | The chosen cluster's region audio is soloed; chips reflect cluster identity and colour persistently across windows |
| Frequency | Occasional |
| Status | Implemented, not validated — the silence and match thresholds are explicitly provisional, carrying a first calibration from one live session (TR Table 1, item 7) |

**Main success scenario:**

1. The client requests clustering (a time-to-live-latched flag on outgoing
   chunks).
2. The server gates silent cells, clusters the active cells (fixed cluster
   count plus a silence class), assigns persistent colours, and returns
   centroids, energies, colours, and per-cell labels.
3. The client renders one chip per cluster on the source rail, tinting the
   overlay cells toward each cluster's colour.
4. The Listener taps a chip; the client solos that cluster's region through
   the query machinery (UC-08).

**Requirements exercised.** FR-016, FR-044.

**Primary evidence.** SonicSightMobile `MainActivity.kt:400-446`;
SonicSightBackend `sonicsight.proto:122-128,170-176`,
`src/grpc_server.py:718-734`, `src/clustering.py` (via
`tests/test_clustering.py`).

---

### UC-12 — Deploy and launch the backend

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | GPU/CUDA stack |
| Stakeholders and interests | S2, S4 |
| Preconditions | Python 3.11+ environment with `requirements.txt` installed; checkpoints provisioned (UC-13); GPU visible to PyTorch (and to TensorFlow, if the speech mode is wanted) |
| Postconditions | gRPC serving on 0.0.0.0:50051 and FastAPI on 0.0.0.0:8000; every loadable engine resident; health reports the loaded set |
| Frequency | Once per working session |
| Status | Implemented and validated — the launch procedure is exercised and recorded runs exist; the both-frameworks-resident memory footprint has never been measured (TR Table 1, item 8) |

**Main success scenario:**

1. The Operator runs `python src/run_servers.py`.
2. The process loads every registered engine; a per-engine load failure
   (typically TensorFlow absent) logs a warning and skips that engine
   instead of killing the process.
3. FastAPI starts on a daemon thread (port 8000); the gRPC server runs on
   the main asyncio loop (port 50051, 16 MB message cap, gzip level 1).
4. The Operator confirms readiness via UC-15.

**Extensions.** 1a. gRPC only: `python src/grpc_server.py`. 1b. FastAPI
only: `python src/main.py`. There are no command-line flags; all tuning is
via environment variables (UC-14).

**Exceptions.** E1: no engine loads — the server still runs; every stream
open fails with `FAILED_PRECONDITION`.

**Requirements exercised.** FR-024, FR-074, FR-080; NFR-COMPAT-003, NFR-FLEX-001.

**Primary evidence.** SonicSightBackend `src/run_servers.py:23-55`,
`src/model_registry.py:202-213`, `src/grpc_server.py:958-981`,
`src/main.py:134-137`, `README.md:163-167`.

---

### UC-13 — Provision checkpoints and the TensorFlow environment

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | GPU/CUDA stack |
| Stakeholders and interests | S2, S4, S6 (checkpoints are the upstream authors' artefacts) |
| Preconditions | The three repositories cloned as siblings (the backend resolves `../multisensory` by default) |
| Postconditions | Sound of Pixels weights in `src/ckpt/`; multisensory checkpoint under `results/nets/sep/full/`; for GPU TensorFlow, a WSL2 environment that passes the GPU-visibility check |
| Frequency | Once per machine |
| Status | Implemented and validated on E-M; not reproduced on E-D (TensorFlow absent there by record and by probe) |

**Main success scenario:**

1. The Operator places `sound_best.pth`, `frame_best.pth`, and
   `synthesizer_best.pth` in `src/ckpt/`.
2. The Operator fetches the multisensory checkpoint (`download_models.sh`
   in the multisensory repository) so `net.tf-160000.*` sits under
   `results/nets/sep/full/`.
3. For the speech mode: inside WSL2, the Operator builds a Python 3.12
   environment with `tensorflow[and-cuda]==2.21.0`, `tf_keras`, `tf_slim`,
   applies the `LD_LIBRARY_PATH` activation fix for the NVIDIA wheel
   libraries and the WSL driver projection, and verifies
   `tf.config.list_physical_devices('GPU')` is non-empty.
4. Optional overrides: `MULTISENSORY_ROOT`, `MULTISENSORY_CHECKPOINT`,
   `MS_GPU`, `MS_GPU_ALLOW_GROWTH`, `MS_GPU_MEM_FRACTION`. The cuDNN
   workspace cap (`TF_CUDNN_WORKSPACE_LIMIT_IN_MB=512`) is a shipped default
   and must not be removed — uncapped, the allocator holds ≈ 2.9 GB for a
   ≈ 0.5 GB working set (HANDOFF §5.2).

**Exceptions.** E1: the environment sits on the Windows-mounted filesystem
(`/mnt/d`) — checkpoint loading and frame extraction degrade badly; the
record's rule is to copy to the WSL home filesystem. E2: an NVIDIA driver
installed inside WSL breaks the projected Windows driver.

**Requirements exercised.** FR-072, FR-073; NFR-FLEX-001, NFR-PERF-010.

**Primary evidence.** SonicSightBackend `README.md:141-159`; HANDOFF §6;
multisensory `README.md:22-26`; checkpoint presence and byte sizes verified
on E-D 2026-08-08 (Appendix D).

---

### UC-14 — Configure separation behaviour via environment variables

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | — |
| Stakeholders and interests | S2, S4 |
| Preconditions | Server not yet started (variables are read at import/start) |
| Postconditions | The configured behaviour applies to all subsequent sessions |
| Frequency | Rare; defaults are the measured configuration |
| Status | Implemented; the mask and audio-normalisation effects are measured but with unrecorded conditions (TR Table 7, Table 1 item 1) |

**Main success scenario:**

1. The Operator sets any of the documented variables —
   `SONICSIGHT_BINARY_MASK`, `SONICSIGHT_MASK_RENORM`,
   `SONICSIGHT_MASK_RENORM_FLOOR`, `SONICSIGHT_SEPARATION_MODE`,
   `SONICSIGHT_SEP_TOPK`, `SONICSIGHT_MASK_SMOOTH`, the
   `SONICSIGHT_AUDIO_NORM*` family, `SONICSIGHT_DUMP_STREAM`,
   `SONICSIGHT_DUMP_DIR` — and starts the server (UC-12).
2. Per-model constants (windows, hops, frame counts) are deliberately *not*
   configurable: they live in frozen registry specifications because
   "reasonable" changes to window length and map resolution both produced
   confident, plausible, wrong output on this project (TR §8.2).

**Requirements exercised.** FR-034, FR-070.

**Primary evidence.** SonicSightBackend `src/config.py:20-115`;
`src/model_registry.py:3-8,28-77`.

---

### UC-15 — Check backend health

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | — |
| Stakeholders and interests | S2, S4 |
| Preconditions | Server running |
| Postconditions | None (read-only) |
| Frequency | After every launch; ad hoc during diagnosis |
| Status | Implemented, not validated — and **the Android client never calls it**; the client learns model availability only from `FAILED_PRECONDITION` at stream open (TR §4.7) |

**Main success scenario:**

1. The Operator invokes the unary `HealthCheck` RPC (manual script or
   generic gRPC tool).
2. The response reports whether any engine is loaded, the device string of
   the PyTorch engine, and the loaded model ids in registry order.

**Extensions.** 1a. HTTP alternative: `GET /` on port 8000 returns
`{"status": "active", "model_loaded": …}`.

**Exceptions.** E1: the device string reflects only the PyTorch singleton;
the multisensory engine's device placement is not reported — a known
reporting gap.

**Requirements exercised.** FR-080; NFR-COMPAT-003.

**Primary evidence.** SonicSightBackend `src/grpc_server.py:40-47`,
`src/model_registry.py:215-224`, `src/main.py:58-60`; TR §4.7.

---

### UC-16 — Capture diagnostics and monitor GPU residency

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | GPU/CUDA stack |
| Stakeholders and interests | S2, S3 (diagnostic bundles are how streaming claims get evidence), S5 (dumps contain captured audio — retention is a privacy surface) |
| Preconditions | Server running; for dumps, `SONICSIGHT_DUMP_STREAM=1` set before launch |
| Postconditions | Per-session diagnostic bundle on disk; a VRAM observation against the 6 GB budget |
| Frequency | During validation and diagnosis |
| Status | Implemented, not validated; the VRAM protocol (test plan T3) has never been executed with both models resident |

**Main success scenario:**

1. With dumping enabled, each streaming session writes a bundle at stream
   end: microphone input as received, an identity-mask control track, output
   PCM, and a per-cycle CSV, in a timestamped directory.
2. The server's periodic log diagnostics run regardless: input amplitude
   every 20 chunks, drain statistics every 10 cycles, and a cycle summary
   whenever a cycle exceeds 140 ms.
3. The Operator samples `nvidia-smi` during a session and compares against
   the budget: steady state must fit 6 GB with at least 500 MiB free; the
   pressure-relief knob is `MS_GPU_MEM_FRACTION`.

**Requirements exercised.** FR-081, FR-082, FR-085; NFR-PERF-009–NFR-PERF-012, NFR-SEC-002.

**Primary evidence.** SonicSightBackend `src/grpc_server.py:104-112,137-138,
335-341,384-394,442-454`, `src/config.py:106-115`; TESTPLAN `T3:63-81`.

---

### UC-17 — Bridge the WSL2 network boundary

| Field | Content |
|---|---|
| Primary actor | Operator |
| Supporting actors | — |
| Stakeholders and interests | S2; S5 (the bridge exposes the plaintext port to the LAN) |
| Preconditions | The server runs inside WSL2 (required for GPU TensorFlow); the handset is on the LAN |
| Postconditions | The handset reaches port 50051 on the Windows host's address |
| Frequency | Once per machine configuration |
| Status | Documented procedure; never exercised with both models resident (TR §11.3). E-D currently has no `.wslconfig`, so mirrored networking is not configured there |

**Main success scenario:**

1. The Operator either sets `networkingMode=mirrored` in the Windows-side
   `.wslconfig`, or adds a `netsh interface portproxy` rule forwarding
   port 50051 to the WSL address.
2. The Listener's configured host (UC-06) is the Windows host's LAN
   address.

**Requirements exercised.** FR-074; NFR-SEC-001.

**Primary evidence.** SonicSightBackend `README.md:169-170`; HANDOFF §6.4;
absence of `.wslconfig` on E-D probed 2026-08-08 (Appendix D).

---

### UC-18 — Run the deterministic replay validation harness

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | Backend server; GPU/CUDA stack |
| Stakeholders and interests | S3, S4 (the harness is the project's instrument for separating streaming bugs from acoustics) |
| Preconditions | Server running with the `sonicsight` engine; a test video file; ffmpeg available |
| Postconditions | Replayed separation WAVs and a report with metrics and a plain-language verdict |
| Frequency | After any change touching buffering, windowing, or reconstruction |
| Status | Implemented and validated (the harness runs; verdict thresholds are defined). **Halves mode only** — it sends no model metadata, so it cannot exercise the speech or touch streams; the test plan records that gap |

**Main success scenario:**

1. The Developer runs `python replay_client.py --video ⟨clip⟩` (options:
   `--host`, `--out`, `--speed`, `--max-seconds`, `--no-analyze`).
2. The harness extracts clean 11 025 Hz mono audio and 8 fps half-frame
   JPEGs from the file — mirroring the phone's chunking, one audio and one
   frame chunk per 125 ms tick, real-time paced — and streams them through
   the live `StreamProcess` path.
3. It writes the returned left/right tracks and the input reference, then
   compares replayed separation against the one-shot `/predict` result:
   left/right Pearson correlation at or below 0.35 indicates healthy
   separation, above 0.45 indicates a reproduced streaming defect; the
   report states the verdict in plain language.
4. Interpretation: replay ≈ one-shot means the streaming code is correct
   and any live-session problem is acoustic capture; replay worse means a
   real streaming bug, now reproducible deterministically.

**Requirements exercised.** FR-085; NFR-FUNC-001.

**Primary evidence.** SonicSightBackend `replay_client.py:1-28,85-91,
117-215,251-341`; TESTPLAN `T1:25-38`, gap at `:93-96`.

---

### UC-19 — Run the pixel-mode A/B harness

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | GPU/CUDA stack |
| Stakeholders and interests | S3, S4 |
| Preconditions | Checkpoints local; ffmpeg; GPU (or a patient CPU) |
| Postconditions | Paired audio and map artefacts for a human listening decision |
| Frequency | When a pixel-mode design question is open |
| Status | Implemented; the design questions it exists to answer (ratio-versus-binary tap default; energy-versus-activation map) remain open pending a human listen — the harness's own docstring forbids changing defaults without one |

**Main success scenario:**

1. The Developer runs `python pixel_ab_harness.py --video ⟨clip⟩`
   (options: `--start`, `--out`, `--tap x,y`).
2. Offline, with no server, it extracts one full analysis window, mirrors
   the client's halves-crop and letterbox transforms, and renders: (A) the
   deployed halves-from-crops separation, (B) region-pooled halves from the
   full-frame cache, tap synthesis through both mask styles, and both map
   variants.
3. Outputs — mixture, A/B/tap WAVs, map images and arrays, and a report
   with RMS and A-versus-B correlations — feed a human listening decision.

**Requirements exercised.** FR-035, FR-036, FR-043 (design validation for these).

**Primary evidence.** SonicSightBackend `pixel_ab_harness.py:1-24,77-104,
144-150,173-231`.

---

### UC-20 — Run the automated unit suites

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | — |
| Stakeholders and interests | S3, S4 |
| Preconditions | Backend: Python environment (no GPU, checkpoints, or TensorFlow needed — suites are designed to import without them). Mobile: Gradle toolchain |
| Postconditions | Pass/fail record |
| Frequency | Per change; the fast layer of verification |
| Status | Implemented and validated. Recorded run: 66 passed, 1 skipped, 2 collection errors (`test_grpc_client.py`), 32 s, 2026-08-05, on E-D (TR §10.5); the collection errors are a recorded defect |

**Main success scenario:**

1. The Developer runs `python -m pytest tests/` in the backend: nine files
   covering clustering, the registry, the multisensory engine's
   TensorFlow-free helpers, overlap-add mechanics, the pixel cache, the
   preprocessor, the model-selection branch, and gRPC client behaviour.
2. On the mobile side, JVM unit tests (coordinate mapping) run via Gradle;
   each integration phase also gated on `assembleDebug`.

**Requirements exercised.** NFR-MAINT-001.

**Primary evidence.** SonicSightBackend `tests/` (nine files);
TR §10.5; SonicSightMobile `README.md:26-27`; TESTPLAN `:11-14`.

---

### UC-21 — Regenerate protocol stubs after a contract change

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | — |
| Stakeholders and interests | S4 (the two proto copies drifting apart would be a silent wire break) |
| Preconditions | The contract change made identically in both copies of `sonicsight.proto` |
| Postconditions | Backend and mobile stubs regenerated from byte-identical sources |
| Frequency | Per protocol change |
| Status | Implemented and validated — the two copies were verified byte-identical on 2026-08-08 (Appendix D) |

**Main success scenario:**

1. Backend: `python -m grpc_tools.protoc -I. --python_out=src
   --grpc_python_out=src sonicsight.proto`.
2. Mobile: the Gradle protobuf plugin regenerates Java/Kotlin lite stubs
   from `app/src/main/proto/sonicsight.proto` on build.
3. The Developer confirms the copies match (byte comparison) and that field
   numbering only ever extends — existing numbers are never reused.

**Requirements exercised.** NFR-COMPAT-001.

**Primary evidence.** SonicSightBackend `README.md:86-89`; SonicSightMobile
`app/build.gradle.kts:87-119`, `README.md:99-103`; byte-identity probe,
Appendix D.

---

### UC-22 — Add a new model behind the registry

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | GPU/CUDA stack |
| Stakeholders and interests | S4 (this contract is the system's main extensibility claim), S3 |
| Preconditions | A candidate model with a pretrained checkpoint |
| Postconditions | The model is a registry entry selectable per stream, with its own capture profile on the client |
| Frequency | Rare; exercised twice to date (multisensory, pixel mode) |
| Status | Implemented contract (`MODELS.md`), demonstrated by its two uses |

**Main success scenario:**

1. The Developer adds a frozen `ModelSpec` to the registry: window, hop,
   rates, frame rule, heatmap convention — constants, not knobs.
2. The Developer writes an `EngineAdapter` whose machine-learning framework
   imports live inside `load()`, so the server runs without that framework
   installed; the buffer and overlap-add layers run at the wire rate and
   the engine resamples internally.
3. The Developer mirrors a `ModelProfile` on the client (capture rates,
   frame geometry, labels) and, only if the model needs a capture format
   that does not exist, extends the protocol (UC-21).
4. Per the contract, the model ships only with a wired-wrong-detecting
   correctness experiment, a measured latency, and a video-memory record.

**Requirements exercised.** FR-024, FR-060, FR-065, FR-066; NFR-MAINT-002, NFR-MAINT-003.

**Primary evidence.** SonicSightBackend `MODELS.md:33-157`;
`src/model_registry.py:28-77`; TR §3.4.

---

### UC-23 — Run the multisensory migration suite T0–T5

| Field | Content |
|---|---|
| Primary actor | Developer |
| Supporting actors | GPU/CUDA stack; Docker (for the TensorFlow 1.8 oracle) |
| Stakeholders and interests | S3 (the migration's validity chain), S6 |
| Preconditions | multisensory repository provisioned (UC-13); for T3/T4, Docker with the pinned `tensorflow/tensorflow:1.8.0-py3` reference image |
| Postconditions | A layer-by-layer verdict that the Python 3 port matches the original within tolerance |
| Frequency | Once per migration change; historically once |
| Status | Implemented; the suite exists and the record cites its outcomes, but the result artefacts are absent from the tree (TR §10.2) — a recorded evidence gap |

**Main success scenario:**

1. T0 smoke (`t0_smoke.sh`), T1 Python-3 audit (`t1_py3_audit.py src/`).
2. T2 pretext self-oracle: `t2_pretext_oracle.py --video ⟨clip⟩ --trials
   200` — alignment accuracy near 50 % means broken, above ≈ 57 % is
   significant with 200 trials.
3. T3/T4: dump activations in the migrated code and in the TensorFlow 1.8
   Docker oracle, compare layer-wise at tolerance 1e-4.
4. T5: compare separation quality direction against the paper's published
   figures.

**Requirements exercised.** Supports the evidence chain behind FR-031 and FR-041 (migration validity); no requirement is exercised directly.

**Primary evidence.** multisensory `README.md:36-135`, `Dockerfile.tf18`,
`t0_smoke.sh`, `t1_py3_audit.py`, `t2_pretext_oracle.py`,
`t3_dump_activations.py`, `t4_compare_dumps.py`; TR §10.2.

---

### UC-24 — TalkBack explore-by-touch traversal *(designed, not implemented)*

| Field | Content |
|---|---|
| Primary actor | Listener |
| Status | **Designed, not implemented.** An in-code design note defers the full TalkBack explore-by-touch experience; what ships today is accessibility-importance flags and content descriptions only |
| Rationale for inclusion | The study's motivation is assistive; an accessibility use case is therefore load-bearing even as a plan, and its absence from the implementation is a Section 7 gap |
| Specification | Not written — a specification would presuppose design decisions (focus order, touch interception versus explore-by-touch pass-through for the query gesture) that are explicitly still open |

**Primary evidence.** SonicSightMobile `MainActivity.kt:643-651`.

---

### UC-25 — Alternative region shapes *(designed, not implemented)*

| Field | Content |
|---|---|
| Primary actor | Listener |
| Status | **Designed, not implemented.** The model contract names region modes beyond the disc — thirds, quadrants, freeform selection — as natural extensions and explicitly defers them |
| Rationale for inclusion | Marks the boundary of the touch mode's current interaction vocabulary |
| Specification | Not written; the contract records the extension point (`region_disc` is one implementation of a region primitive) |

**Primary evidence.** SonicSightBackend `MODELS.md:141-148`; TR Table 1,
item 10.

---

## Appendix C — Measured results

*(pending — Phase 5. Raw results with run metadata: commit SHAs of both
repositories, checkpoint identifiers, GPU model, driver and CUDA versions,
Python and framework versions, host specification, timestamp, and scenario
parameters, per the load-suite design.)*

---

## Appendix D — Evidence for claims in this document (initial)

Evidence anchors for Sections 2–3 and 7–8 claims not already carried by
Appendix B or the ADR Evidence sections. Probes were run on E-D on
2026-08-08.

| Claim | Evidence |
|---|---|
| Repository heads and cleanliness | `git rev-parse HEAD` / `git status --porcelain` per repository, probed 2026-08-08; matches TR front matter |
| E-M environment facts | HANDOFF (2).md §6.1; TR:2429-2433 (conditions block), TR:3007-3020 (Table 17) |
| E-M figures recorded 2026-08-03 | TR:2424-2427 |
| Sound of Pixels streaming measurement conditions unrecorded | TR:186 (Table 1, item 1) |
| E-D hardware, OS, driver | `nvidia-smi`, `Win32_Processor`, `Win32_ComputerSystem` probes; matches TR:3022-3034 (Table 18) |
| E-D Python/package versions | `python --version`, `pip show torch numpy grpcio protobuf` probes |
| TensorFlow absent on E-D | TR:2638-2641; `pip show` probe (not installed) |
| WSL on E-D: Ubuntu 24.04.1, no multisensory environment, no `.wslconfig` | `wsl -l -v`, `/etc/os-release`, home-directory and `%USERPROFILE%\.wslconfig` probes |
| Checkpoint byte sizes on disk match TR | Directory listing probe vs TR:2470-2473 |
| Both proto copies byte-identical | `fc.exe /b SonicSightBackend\sonicsight.proto SonicSightMobile\app\src\main\proto\sonicsight.proto` — no differences |
| ffmpeg, Node.js, mermaid-cli present on E-D | `ffmpeg -version`, `node --version`, `mmdc --version` probes |
| Perceived-lag figures are estimates, not measurements | TR:2407 (Table 12 note), TR:2412-2415 |
| Three deployed modes and their labels | TR:602-619 (Table 4); SonicSightBackend `src/model_registry.py:92-189` |
| Client default host and fixed port | SonicSightMobile `GrpcModule.kt:12-15` |
| Always-on device-side microphone dump (S5 concern) | SonicSightMobile `MainActivity.kt:1037-1063`; TR §11.4 defect 1 |
| Plaintext, unauthenticated transport by design | TR §12.1; SonicSightMobile `GrpcModule.kt:51`, `AndroidManifest.xml:19` |
| Attribution gap for Sound of Pixels artefacts | TR:3181,3190-3196 |
| Client never calls HealthCheck | TR:1002-1007; grep of mobile source (no caller) |
| Legacy upload path unreachable | SonicSightMobile `MainViewModel.kt:469`, `MainActivity.kt:696` |
| Single English locale, no locale-qualified resources (G11) | Directory listing of `SonicSightMobile\app\src\main\res` — `values/` + `values-night/` (theme qualifier) only |
| FastAPI `/predict` runs GPU inference outside the process lock (FR-023 PS) | SonicSightBackend `src/main.py:102-108` (no `_inference_lock` acquisition) |
| Shared per-mode EMAs on singletons (FR-020 PS) | SonicSightBackend `src/inference.py:561-567,1184-1193,1375-1381`; `src/engines/multisensory_engine.py:146,253-259` |

---

## Appendix E — Glossary (initial)

*(Extended in Phase 7.)*

| Term | Definition |
|---|---|
| CAM | Class activation map — the multisensory network's spatial evidence tensor, used here as the localization map after confidence gating. |
| d/DHH | Deaf and hard-of-hearing. |
| gRPC | A remote procedure call framework; the client–server transport, used in bidirectional streaming mode. |
| Halves mode | The `sonicsight` listening mode: the frame is split down the middle and each half is separated independently. |
| Jitter buffer | Client-side buffer absorbing network arrival variance before audio playback. |
| Letterbox | Padding a 16:9 frame into a square input with grey bars rather than cropping. |
| MoSCoW | Must/Should/Could/Won't — the prioritisation scheme used in Sections 4–5. |
| OLA | Overlap-add — stitching overlapping analysis windows into one continuous output stream with crossfades. |
| Perceived lag | Time from a real-world event to the corresponding separated audio at the ear; ≈ half the analysis window plus processing (an estimate in this project — no stopwatch measurement exists). |
| SoP | Sound of Pixels, the PyTorch separation model [1]. |
| TR | The companion technical report, `docs/TECHNICAL_REPORT.md`. |
| Touch (pixel) mode | The `sonicsight-pixel` listening mode: per-window features are cached and the user's touch selects a region to synthesize. |
| VRAM | Video (GPU) memory. |
| WSL2 | Windows Subsystem for Linux 2 — the Linux environment required for GPU TensorFlow on the server host. |
