# Safety Rules Definition

## Overview

This document defines every safety rule enforced by the Construction Safety Monitor.
Rules were designed following OSHA 29 CFR 1926 (Construction Industry Standards)
and the UK CDM Regulations 2015 as reference frameworks.

---

## Rule Definitions

### R1 - Hard Hat / Safety Helmet

**Rule:** Every worker present in an active construction zone must wear an approved safety helmet at all times.

**Rationale:** Head injuries are among the leading causes of construction fatalities. Hard hats protect against falling objects, bumping against fixed structures, and electrical shock.

**What counts as a violation:**
- Worker detected without any head protection (`no_hardhat` class)
- Helmet detected nearby but not on the worker's head (spatial mismatch)
- Partial detection with confidence < 0.30 treated as uncertain and flagged conservatively

**What does NOT count as a violation:**
- Worker in a designated rest area (out-of-scope for this model version)
- Visitor zones with explicit exemption signage (out-of-scope)

**Illustrative violation:** A worker in PPE vest but no helmet visible → `no_hardhat` bounding box detected around head region.

---

### R2 — High-Visibility (Hi-Vis) Vest

**Rule:** Every worker in an active work zone or near moving machinery/vehicles must wear a high-visibility vest.

**Rationale:** Hi-vis vests make workers visible to vehicle operators, crane operators, and other workers — critical in busy or low-light construction environments.

**What counts as a violation:**
- Worker detected without a vest (`no_vest` class)
- Vest detected off-body (held or on the ground) — modelled implicitly by detecting `person` without co-located `vest`

**What does NOT count as a violation:**
- Workers in enclosed offices or welfare cabins (not captured in outdoor/site imagery)

**Illustrative violation:** Worker wearing a helmet but no vest visible on torso → `no_vest` bounding box around torso region.

---

### R3 — PPE Must Be Correctly Worn

**Rule:** PPE that is partially worn, unfastened, or improperly fitted is flagged as a violation.

**How this is modelled:** The model uses confidence scores as a proxy for correct wear. A detection of `hardhat` with confidence < 0.30 suggests the helmet is present but not clearly on the head (e.g. pushed back, tilted heavily, or unfastened chinstrap). The system applies a minimum confidence gate (`MIN_CONF = 0.30`) and treats sub-threshold detections conservatively.

**Limitation:** Full pose-based assessment (e.g. chinstrap fastening) requires keypoint detection and is noted as a future enhancement.

---

### R4 — Scene-Level Safety Ruling

**Rule:** Any single violation in a frame renders the **entire scene UNSAFE**.

**Rationale:** Construction safety is not averaged — one unprotected worker represents a real risk regardless of how many compliant workers are present.

**Output:** The system produces both a scene-level SAFE/UNSAFE verdict and individual violation records so supervisors can identify the specific non-compliant individual.

---

## Violation Categories Summary

| Category | Rule | Class Detected | Severity |
|----------|------|----------------|----------|
| Missing hard hat | R1 | `no_hardhat` | HIGH |
| Missing hi-vis vest | R2 | `no_vest` | HIGH |
| Uncertain PPE wear | R3 | Low-conf detection | MEDIUM |
| Any of the above | R4 | Scene-level | UNSAFE |

---

## Future Rule Extensions (Documented, Not Yet Implemented)

| Rule ID | Description |
|---------|-------------|
| R5 | Fall protection — workers at height (>2m) without visible harness |
| R6 | Zone-based rules — stricter PPE in demolition or electrical zones |
| R7 | Footwear — safety boots required (requires fine-grained foot detection) |
| R8 | Tool handling — prohibited tool use postures near live cables |

These extensions would require additional labelled classes and are documented as the next iteration roadmap.

---

## Confidence Score Interpretation

The system surfaces a **confidence score** (0–1) alongside every ruling:

| Score Range | Interpretation |
|-------------|----------------|
| 0.85–1.00 | High confidence — strong detections, clear violations or clear compliance |
| 0.60–0.84 | Moderate confidence — proceed but consider manual review |
| 0.30–0.59 | Low confidence — scene may be ambiguous (occlusion, distance, lighting) |
| < 0.30 | Very uncertain — no reliable detections; flag for human inspection |
