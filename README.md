# Smart Home Ontology (OWL 2)

An OWL 2 ontology modelling a residential smart home / IoT environment: devices, sensors, actuators, rooms, users, home networks, events and automation rules. Built with Protégé for the Knowledge Representation course, MSc in Data Science, University of Verona (Prof. Matteo Cristani).

**Author:** Raul (Student ID VR544459)
**IRI:** `http://www.semanticweb.org/raul/ontologies/2026/smarthome`
**Format:** RDF/XML (OWL 2 DL)
**Reasoner:** HermiT 1.4 — consistent, zero unsatisfiable classes

---

## Repository structure

```
├── smart_home_ontology.owl    # The ontology (RDF/XML)
├── report.docx                # Full written report (Word)
├── report.md                  # Full written report (Markdown)
├── Raul-Manafov.pptx          # Presentation slides
├── presentation_slides.md     # Slide content (Markdown)
└── README.md                  # This file
```

## Ontology metrics

| Metric | Count |
|---|---|
| Classes | 46 |
| Object properties | 18 |
| Data properties | 15 |
| Named individuals | 35 |
| SWRL rules | 3 |
| Defined (equivalent) classes | 8 |
| Disjointness axiom groups | 8 |

Every class and property carries an `rdfs:label` and an `rdfs:comment`.

## How to open it

1. Download [Protégé 5.6+](https://protege.stanford.edu/).
2. `File → Open` and select `smart_home_ontology.owl`. It loads with no errors or missing imports (the ontology is fully self-contained).
3. To run the reasoner: `Reasoner → HermiT`, then `Reasoner → Start reasoner`. It classifies in under two seconds and reports a consistent ontology.
4. To view the SWRL rules: `Window → Tabs → SWRLTab` (enable the SWRLTab plugin from `File → Check for plugins` if it is not installed).

## What the reasoner infers

After classification, HermiT derives (among others):

- **Class hierarchy:** `GuestUser ⊑ NonAdminUser`, `ChildUser ⊑ NonAdminUser`, `LowBatteryDevice ⊑ BatteryPoweredDevice`, all four environmental sensor types under `EnvironmentalSensor`, and `SmokeSensor`/`CO2Sensor`/`SmartLock`/`SecurityCamera` under `SafetyDevice`.
- **Individual classification:** every device connected to a network becomes a `SmartDevice`; `MotionSensor_Hallway` (battery 15%) is classified as a `LowBatteryDevice`; `Marco_Russo` (age 9, asserted only as `User`) is classified as a `ChildUser` via a datatype restriction; `ClimateStation_Bedroom` (asserted only as `Device`) is classified as a `HybridDevice` because it has both a sensor and an actuator component.
- **SWRL-driven facts:** `Event_Smoke_001` is inferred to be a `SecurityEvent` (rule R2); `occursIn(Event_Motion_001, Hallway_Entrance)` is inferred from the location of the sensor that triggered it (rule R1).

## Competency questions

The ontology was evaluated against five competency questions, each answerable as a DL query in Protégé:

| # | Question | DL query |
|---|---|---|
| CQ1 | Which devices are smart devices? | `SmartDevice` |
| CQ2 | Which devices need their battery replaced? | `LowBatteryDevice` |
| CQ3 | Which devices monitor the kitchen? | `monitors value Kitchen_Main` |
| CQ4 | Where did security events occur? | `SecurityEvent and (occursIn some Room)` |
| CQ5 | Which users lack administrative rights? | `NonAdminUser` |

## SWRL rules

```
R1: MotionSensor(?s) ^ isLocatedIn(?s,?r) ^ triggers(?s,?e) ^ SecurityEvent(?e) → occursIn(?e,?r)
R2: SmokeSensor(?s) ^ triggers(?s,?e) → SecurityEvent(?e)
R3: SmartLock(?l) ^ controlledBy(?l,?u) → AdminUser(?u)
```

## Design notes

- `SmartDevice` is a defined class (`Device and isConnectedTo some HomeNetwork`), so membership is never asserted — it is always inferred.
- `isConnectedTo` is deliberately **not** functional: `Gateway_Main` bridges the Wi-Fi and Zigbee networks simultaneously.
- `isLocatedIn` is functional and asymmetric; `communicatesWith` is symmetric; `hasComponent` is transitive.
- Top-level categories (`Device`, `Room`, `User`, `Event`, `HomeNetwork`, `AutomationRule`) are mutually disjoint, as are the sibling sets inside each branch.

## License

Released for academic purposes. Feel free to reuse with attribution.
