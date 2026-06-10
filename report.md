# A Smart Home Ontology in OWL 2: Design, Implementation and Reasoning

**Raul — VR544459**
Knowledge Representation, MSc in Data Science, University of Verona
Prof. Matteo Cristani — June 2026

---

## Abstract

This report presents the design, implementation and evaluation of an OWL 2 ontology modelling a residential smart home environment. The ontology covers the principal entities of a connected household — devices, sensors, actuators, rooms, users, home networks, events and automation rules — and formalises their relationships through a hierarchy of 46 classes, 18 object properties, 15 data properties and 35 named individuals. The design exercises a broad portion of the OWL 2 expressive toolbox: defined classes built from existential and datatype restrictions, union, intersection and complement expressions, eight groups of disjointness axioms, and property characteristics including functionality, transitivity, symmetry and asymmetry. Three DL-safe SWRL rules extend the ontology with inferences that pure class expressions cannot capture, including the propagation of event locations and a declarative access-control policy on smart locks. The ontology was developed in Protégé following the Ontology Development 101 methodology and validated with the HermiT reasoner, which confirms consistency, finds no unsatisfiable classes, and derives a substantial set of non-asserted facts: subsumptions between user categories, automatic classification of connected devices, age-based classification of users, and rule-driven event reclassification. Evaluation against five competency questions shows that every answer is obtained by inference rather than retrieval, demonstrating that the ontology performs genuine knowledge-representation work.

---

## 1. Introduction

Knowledge representation is concerned with encoding information about a domain in a form that supports automated reasoning: not merely storing facts, but deriving consequences from them. Among the formalisms developed for this purpose, ontologies occupy a central position. In Gruber's classic formulation, an ontology is "an explicit specification of a conceptualization" (Gruber, 1993): it fixes a shared vocabulary for a domain and constrains the meaning of its terms through logical axioms, so that both humans and machines can interpret data unambiguously.

The Web Ontology Language, OWL 2, is the W3C standard for authoring such ontologies (W3C, 2012). Its design is grounded in description logics, a family of decidable fragments of first-order logic, which means that an OWL 2 DL ontology comes with a precise model-theoretic semantics and with reasoning services — consistency checking, classification, instance retrieval — that are guaranteed to terminate. Compared with its predecessor, OWL 2 adds expressive means that this project uses directly, most notably datatype restrictions with facets (for example, "an integer at most 20") and richer property characteristics such as asymmetry.

An OWL ontology is conventionally divided into a TBox, the terminological component containing classes, properties and axioms about them, and an ABox, the assertional component containing individuals and facts about them. A reasoner such as HermiT (Glimm et al., 2014) operates on both: it checks that the axioms admit a model at all, computes the inferred class hierarchy, and classifies individuals into every class whose definition they satisfy.

This project applies these tools to the smart home domain. The remainder of the report describes the domain and the motivation for choosing it (Section 2), the methodology followed (Section 3), the TBox (Section 4) and ABox (Section 5) in detail, the SWRL rule layer (Section 6), the evaluation by reasoning and competency questions (Section 7), and conclusions with directions for future work (Section 8).

## 2. Domain Description

A smart home is a residence equipped with networked devices that sense the environment (temperature, motion, humidity, smoke, light, air quality), act on it (lights, locks, thermostats, plugs, cameras, blinds), and coordinate through home networks and automation rules under the control of users with different privileges. The domain was chosen for three reasons.

First, it is a domain where ontologies are not an academic exercise but the industry's actual answer to a real problem. The smart home market is severely fragmented: devices from different vendors speak different protocols and describe their capabilities in incompatible vocabularies. Standardisation bodies have responded precisely with ontologies — the W3C's SSN/SOSA ontology for sensors and observations (Haller et al., 2019) and ETSI's SAREF reference ontology for smart appliances (ETSI, 2020) are the most prominent examples. Building a compact ontology in the same spirit therefore connects the coursework to genuine practice.

Second, the domain naturally exercises a wide range of OWL constructs. It contains crisp taxonomies (kinds of sensors, kinds of rooms) that motivate disjointness axioms; cross-cutting categories (devices that are "safety-relevant" regardless of whether they sense or act) that motivate union classes; threshold-based categories (low battery, minor users) that motivate datatype restrictions; composite artefacts (a climate station containing both a sensor and an actuator) that motivate intersection definitions and transitive part-whole properties; and access-control policies that motivate rules.

Third, the domain yields competency questions with practical value. "Which devices need a battery change?", "which devices monitor the kitchen?", "did a security event occur, and where?", "does anyone without administrative rights control the front door lock?" are questions a real home-automation system must answer, and in this project they are answered by logical inference rather than by application code.

The scope was deliberately limited to a single household at a single point in time. Temporal evolution of readings, multi-home deployments and streaming data are out of scope and discussed as future work.

## 3. Methodology

The ontology was developed following the Ontology Development 101 methodology of Noy and McGuinness (2001), which structures the work into iterative steps.

**Step 1 — Scope and competency questions.** Before any modelling, five competency questions (listed in full in Section 7) were written down. They define what the ontology must be able to answer and acted throughout the project as the criterion for deciding whether a candidate class or property earned its place.

**Step 2 — Term enumeration.** The important terms of the domain were listed: device, sensor, actuator, gateway, the individual sensor and actuator kinds, room and room kinds, network and network technologies, user roles, events, automation rules, and the attributes attached to each (battery level, IP address, readings, ages, timestamps, priorities).

**Step 3 — Class hierarchy.** The hierarchy was built top-down, starting from six mutually disjoint top-level categories and refining each into subclasses, with sibling disjointness asserted at every level where it holds.

**Step 4 — Properties.** Object properties were introduced in inverse pairs with explicit domains and ranges, and property characteristics (functional, transitive, symmetric, asymmetric) were asserted where the domain semantics justified them. Data properties were typed with appropriate XSD datatypes and given the tightest sensible domains.

**Step 5 — Restrictions and defined classes.** The classes whose membership should be computed rather than asserted — SmartDevice, HybridDevice, LowBatteryDevice, ChildUser and others — were converted into defined classes using equivalence axioms over restrictions.

**Step 6 — Individuals.** The ABox was populated with 35 individuals forming one coherent household scenario, with several individuals deliberately under-typed so that the reasoner's classification work would be visible.

**Step 7 — Iteration with the reasoner.** HermiT was run after every substantial change. This loop caught real modelling errors during development. The most instructive one concerned the property `isConnectedTo`: it was initially declared functional, on the intuition that a device joins one network, but the gateway individual — which by its nature bridges the Wi-Fi and the Zigbee networks — then made the ontology inconsistent, because functionality forced the two distinct networks to be identified while other axioms kept them apart. The inconsistency was correct reasoner behaviour exposing a wrong axiom; the fix was to drop functionality from `isConnectedTo` and document the decision. This episode illustrates the central methodological benefit of working with a reasoner: axioms are tested against the data continuously, and mistakes surface as logical contradictions rather than as silent modelling debt.

The implementation tool was Protégé 5.6 (Musen, 2015), the de-facto standard ontology editor; the serialisation format is RDF/XML; the reasoner is HermiT 1.4, which implements a hypertableau calculus for OWL 2 DL (Glimm et al., 2014). The ontology IRI is `http://www.semanticweb.org/raul/ontologies/2026/smarthome`. Every class and every property carries an `rdfs:label` and an `rdfs:comment`, so the ontology is self-documenting when browsed in Protégé.

## 4. The TBox: Classes, Properties and Axioms

### 4.1 Class hierarchy

The TBox contains 46 classes organised under six mutually disjoint top-level categories, declared pairwise disjoint through an `owl:AllDisjointClasses` axiom: `Device`, `Room`, `User`, `Event`, `HomeNetwork` and `AutomationRule`. The disjointness is semantically essential: without it, OWL's open-world assumption would permit a model in which, say, a particular device is also a room, and several of the error-detection guarantees discussed below would be lost.

The `Device` branch is the deepest. `Device` has five direct subclasses: `Sensor`, `Actuator`, `Gateway`, `Controller` and `HybridDevice`, with the first four mutually disjoint. The disjointness of `Sensor` and `Actuator` encodes the design decision that sensing and acting are distinct capabilities; composite devices are handled separately, as discussed in Section 4.3. `Sensor` specialises into six mutually disjoint subclasses (`TemperatureSensor`, `MotionSensor`, `HumiditySensor`, `SmokeSensor`, `LightSensor`, `CO2Sensor`) and `Actuator` into six (`SmartLight`, `SmartLock`, `Thermostat`, `SmartPlug`, `SecurityCamera`, `SmartBlinds`). `SecurityCamera` is placed under `Actuator` as a deliberate simplification — cameras are modelled here by their controllable aspect (pan, record, arm) rather than their sensing aspect, a choice that the `monitors` property compensates for, as explained below.

`Room` has six disjoint subclasses (`Bedroom`, `Kitchen`, `LivingRoom`, `Bathroom`, `Garage`, `Hallway`); `HomeNetwork` has four (`WiFiNetwork`, `ZigbeeNetwork`, `BluetoothNetwork`, `ZWaveNetwork`); `Event` has four (`SecurityEvent`, `EnvironmentEvent`, `UserEvent`, `DeviceEvent`); and `User` has the asserted subclasses `AdminUser`, `GuestUser` and `ChildUser` (the first mutually disjoint from the other two) plus the defined class `NonAdminUser` discussed next. In total the ontology contains eight `AllDisjointClasses` groups.

### 4.2 Defined classes

Eight classes are *defined*, i.e. introduced through `owl:equivalentClass` axioms rather than mere subclass axioms, so that the reasoner computes their membership. They are the semantic core of the ontology. In Manchester syntax:

```
SmartDevice        EquivalentTo  Device and (isConnectedTo some HomeNetwork)
HybridDevice       EquivalentTo  Device and (hasComponent some Sensor)
                                        and (hasComponent some Actuator)
EnvironmentalSensor EquivalentTo TemperatureSensor or HumiditySensor
                                  or LightSensor or CO2Sensor
SafetyDevice       EquivalentTo  SmokeSensor or CO2Sensor
                                  or SmartLock or SecurityCamera
NonAdminUser       EquivalentTo  User and (not AdminUser)
ChildUser          EquivalentTo  User and (hasAge some xsd:integer[< 18])
BatteryPoweredDevice EquivalentTo Device and (hasBatteryLevel some xsd:integer)
LowBatteryDevice   EquivalentTo  Device and (hasBatteryLevel some xsd:integer[<= 20])
```

These eight definitions jointly exercise intersection (`and`), union (`or`), complement (`not`), existential restriction (`some`) and OWL 2 datatype restrictions with facets. Several remarks are in order.

`SmartDevice` is the canonical defined class of the ontology: no individual is ever asserted to belong to it; membership follows entirely from an asserted network connection. `HybridDevice` resolves the tension created by the Sensor–Actuator disjointness: a composite device such as a climate station is not simultaneously a sensor and an actuator (which would be inconsistent), but a device that *has components* of both kinds. `SafetyDevice` is a union that deliberately crosses the Sensor–Actuator divide, grouping smoke and CO2 sensors with locks and cameras: what unites its members is purpose, not mechanism, and a union class is exactly the right tool for such cross-cutting categories. `NonAdminUser` uses complement, and from its definition alone the reasoner derives — without any explicit subclass axiom — that `GuestUser` and `ChildUser` are subclasses of `NonAdminUser`, since both are disjoint from `AdminUser`. Finally, the pair `BatteryPoweredDevice` / `LowBatteryDevice` demonstrates inference between datatype restrictions: having a battery level of at most 20 logically entails having a battery level, so HermiT infers `LowBatteryDevice ⊑ BatteryPoweredDevice`, again without that subsumption ever being stated.

### 4.3 Object properties

The ontology declares 18 object properties, almost all in inverse pairs, each with an explicit domain and range:

| Property | Inverse | Domain → Range | Characteristics |
|---|---|---|---|
| `isLocatedIn` | `hasDevice` | Device → Room | functional, asymmetric |
| `isConnectedTo` | `hasConnectedDevice` | Device → HomeNetwork | — (deliberately not functional) |
| `controlledBy` | `controls` | Device → User | — |
| `hasOwner` | `owns` | Device → User | functional |
| `triggers` | `isTriggeredBy` | Device → Event | — |
| `monitors` | `isMonitoredBy` | Device → Room | — |
| `hasAutomationRule` | `appliesToDevice` | Device → AutomationRule | — |
| `hasComponent` | `isComponentOf` | Device → Device | transitive |
| `communicatesWith` | (itself) | Device → Device | symmetric |
| `occursIn` | — | Event → Room | — |

The characteristics carry real semantic weight. `isLocatedIn` is functional — a device occupies exactly one room — which turns double location assertions into detectable inconsistencies given the disjointness of room individuals' classes; it is also asymmetric, ruling out cycles such as a room located in a device. `hasOwner` is functional for the same reason: ownership is modelled as unique. `communicatesWith` is symmetric, since device communication is mutual, and is its own inverse. `hasComponent` is transitive, so the part-whole structure of composite devices propagates: a chip inside a sensor module inside a climate station is a component of the station.

Two design decisions deserve explicit mention. The first, already discussed in Section 3, is that `isConnectedTo` is *not* functional, because gateways bridge multiple networks; the initial functional declaration was refuted by the reasoner. The second concerns `monitors`: its domain is `Device` rather than `Sensor`, so that a security camera — placed under `Actuator` in this ontology — can still be said to monitor a room. Narrowing the domain to `Sensor` would have caused the reasoner to classify the camera as a sensor, contradicting the Sensor–Actuator disjointness; widening the domain keeps the model consistent while preserving the intended use of the property. Both decisions exemplify how domain and range axioms in OWL are not input constraints but inference triggers, and must be set with the reasoner's behaviour in mind.

`occursIn` is declared but barely asserted: it is populated mainly by SWRL rule R1 (Section 6), which is precisely the point of having a rule layer.

### 4.4 Data properties

Fifteen data properties attach literal values to individuals, each typed with the appropriate XSD datatype: `hasIPAddress` (string, functional), `hasBatteryLevel` (integer, functional), `isOnline` (boolean, functional), `hasTemperatureReading` and `hasHumidityReading` (float, with domains narrowed to the corresponding sensor classes), `hasManufacturer` and `hasFirmwareVersion` (string), `hasInstallationDate` and `hasTimestamp` (dateTime), `hasEnergyConsumption` and `hasFloorArea` (float), `hasAge` (integer, functional, domain `User`), `hasSSID` (string, domain `WiFiNetwork`), `hasPriority` (integer, domain `AutomationRule`) and `hasMaxOccupancy` (integer, domain `Room`).

Domains were made as tight as the semantics allow — an SSID belongs only to a Wi-Fi network, a humidity reading only to a humidity sensor — which both documents intent and lets Protégé guide correct data entry. More importantly, two of these properties (`hasBatteryLevel`, `hasAge`) participate in the datatype restrictions of the defined classes of Section 4.2, so the data layer directly drives classification: asserting `hasAge 9` on a plain `User` is sufficient for the reasoner to conclude `ChildUser`, and asserting `hasBatteryLevel 15` on a sensor is sufficient to conclude `LowBatteryDevice`. This interplay between ABox literals and TBox definitions is an OWL 2 capability (datatype facets) that did not exist in OWL 1.

## 5. The ABox: Individuals

The ABox contains 35 named individuals forming a single coherent scenario: one apartment, two networks, one family.

The apartment has six rooms — `Kitchen_Main`, `LivingRoom_Main`, `Bedroom_Master`, `Bathroom_Main`, `Garage_Main`, `Hallway_Entrance` — with floor areas and maximum occupancies. Two networks are present: `HomeWiFi_5G`, a `WiFiNetwork` with SSID `"SmartHome_5G"`, and `Zigbee_Mesh`, a `ZigbeeNetwork`. The users are `Alice_Russo` (an `AdminUser`, age 38), `Bob_Verdi` (a `GuestUser`, age 29) and `Marco_Russo`, who is asserted *only* as a `User` with age 9.

Nineteen devices populate the rooms. The sensors include `TempSensor_Kitchen` (battery 87, reading 22.5 °C), `MotionSensor_Hallway` (battery 15, which triggers `Event_Motion_001`), `HumiditySensor_Bathroom` (reading 71.5 %), `SmokeSensor_Kitchen` (which triggers `Event_Smoke_001`), `CO2Sensor_Garage` and `LightSensor_LivingRoom` (with `isOnline false`, modelling a disconnected unit). The actuators include `SmartLight_LivingRoom`, `SmartLock_FrontDoor` — controlled exclusively by Alice, a fact whose significance emerges with rule R3 — `Thermostat_LivingRoom`, `SmartPlug_Bedroom` (drawing 1450 W from a connected heater), `SecurityCamera_Garage` and `SmartBlinds_Bedroom`. `Gateway_Main` is connected to *both* networks and has IP address 192.168.1.10; `Controller_VoiceHub` is the voice assistant. Three `AutomationRule` individuals (`Rule_EveningLights`, `Rule_NightTemperature`, `Rule_MorningBlinds`) carry priorities and are linked to the devices they govern. Four events complete the picture: two motion events, a smoke event, and a humidity event.

A deliberate feature of the ABox is *under-typing*. `Marco_Russo` is not asserted to be a `ChildUser`; `ClimateStation_Bedroom` is asserted only as a `Device`, with `hasComponent` assertions to an internal temperature sensor and an internal fan actuator; `Event_Smoke_001` is asserted only as an `Event`; and no individual is asserted into `SmartDevice`, `BatteryPoweredDevice` or `LowBatteryDevice`. These omissions are intentional: they leave the classification work to the reasoner, so that the defined classes of Section 4.2 demonstrably earn their keep. Had every type been asserted by hand, the equivalence axioms would be logically present but practically decorative.

## 6. SWRL Rules

Pure OWL class expressions, for all their expressivity, cannot relate more than two individuals through a chain of joined variables: there is no class restriction stating "this event occurred in the room where the sensor that triggered it is located", because that statement joins an event, a sensor and a room. The Semantic Web Rule Language (Horrocks et al., 2004) fills this gap with Horn-style rules over OWL vocabulary. The ontology contains three DL-safe rules (no built-ins), all of which fire in HermiT within Protégé:

```
R1: MotionSensor(?s) ∧ isLocatedIn(?s,?r) ∧ triggers(?s,?e) ∧ SecurityEvent(?e)
        → occursIn(?e,?r)

R2: SmokeSensor(?s) ∧ triggers(?s,?e) → SecurityEvent(?e)

R3: SmartLock(?l) ∧ controlledBy(?l,?u) → AdminUser(?u)
```

R1 performs the three-way join just described: in the ABox, the hallway motion sensor triggers `Event_Motion_001`, and the rule derives `occursIn(Event_Motion_001, Hallway_Entrance)` — a fact present nowhere in the file. R2 is a reclassification rule encoding the policy that smoke detection is always security-relevant: `Event_Smoke_001`, asserted as a plain `Event`, is inferred to be a `SecurityEvent`.

R3 is conceptually the most interesting: it is a declarative access-control policy. It states that whoever controls a smart lock must be an administrator. In the current ABox the front-door lock is controlled only by Alice, an `AdminUser`, so the rule is satisfied and the ontology consistent. But the rule also acts as a guard: if a `controlledBy` assertion from the lock to `Bob_Verdi` were added, R3 would force `AdminUser(Bob_Verdi)`, which contradicts the asserted disjointness between `AdminUser` and `GuestUser`, and HermiT would report the ontology inconsistent. A security violation thus manifests as a logical contradiction — the policy is enforced by the semantics, not by application code. This pattern, using rules together with disjointness axioms as integrity constraints, is in the author's view the clearest illustration in the project of what knowledge representation buys over plain data storage.

## 7. Evaluation

The ontology was evaluated along two complementary lines: reasoner-based verification and competency questions.

### 7.1 Reasoning results

HermiT 1.4 classifies the ontology in under two seconds and reports it **consistent**, with **zero unsatisfiable classes** — a non-trivial guarantee given the eight disjointness groups, the functional and asymmetric property axioms, and the interaction between rules and class definitions.

Beyond the sanity check, the reasoner derives a substantial body of non-asserted knowledge, verified in this project run by run:

*Inferred subsumptions.* `GuestUser ⊑ NonAdminUser` and `ChildUser ⊑ NonAdminUser` (from the complement definition together with disjointness from `AdminUser`); `LowBatteryDevice ⊑ BatteryPoweredDevice` (from entailment between datatype restrictions); `TemperatureSensor`, `HumiditySensor`, `LightSensor`, `CO2Sensor ⊑ EnvironmentalSensor` and `SmokeSensor`, `CO2Sensor`, `SmartLock`, `SecurityCamera ⊑ SafetyDevice` (from the union definitions). None of these subclass axioms appears in the file.

*Inferred individual types.* All fifteen network-connected devices are classified into `SmartDevice`; every device with a battery-level assertion is classified into `BatteryPoweredDevice`; `MotionSensor_Hallway`, at 15 %, is classified into `LowBatteryDevice`; `Marco_Russo`, asserted only as a `User` with age 9, is classified into `ChildUser` and consequently into `NonAdminUser`; `ClimateStation_Bedroom`, asserted only as a `Device`, is classified into `HybridDevice` (from its sensor and actuator components) and into `SmartDevice` (from its network connection).

*Rule-derived facts.* `SecurityEvent(Event_Smoke_001)` via R2, and `occursIn(Event_Motion_001, Hallway_Entrance)` via R1.

### 7.2 Competency questions

The five competency questions written at the start of the project were posed to the classified ontology as DL queries in Protégé's DL Query tab. All five are answered correctly, and — the central point — every answer depends on inference:

**CQ1 — Which devices in the home are smart devices?** Query: `SmartDevice`. Answer: fifteen individuals, from `TempSensor_Kitchen` to `Gateway_Main`. The answer set is entirely inferred, since `SmartDevice` has no asserted members.

**CQ2 — Which devices need their battery serviced?** Query: `LowBatteryDevice`. Answer: `MotionSensor_Hallway`. The answer follows from the datatype facet restriction over `hasBatteryLevel`.

**CQ3 — Which devices monitor the kitchen?** Query: `monitors value Kitchen_Main`. Answer: the kitchen temperature and smoke sensors. This query exercises the `monitors` property and its widened domain.

**CQ4 — Where did security events occur?** Query: `SecurityEvent and (occursIn some Room)`. Answer: `Event_Motion_001`, located in `Hallway_Entrance`. Both the event's classification context and its location depend on the SWRL layer: no `occursIn` assertion exists in the file.

**CQ5 — Which users lack administrative rights?** Query: `NonAdminUser`. Answer: `Bob_Verdi` and `Marco_Russo`. The first follows from the inferred subsumption `GuestUser ⊑ NonAdminUser`; the second additionally requires Marco's age-based classification into `ChildUser`.

The evaluation thus shows not merely that the ontology *contains* the relevant information, but that it *derives* it — which is the criterion an ontology, as opposed to a database schema, should be judged by.

## 8. Conclusions and Future Work

This project delivered a complete, consistent OWL 2 DL ontology of a smart home environment: 46 classes in a hierarchy three to four levels deep, 18 object properties and 15 data properties with full domain/range typing and meaningful characteristics, 35 individuals forming a coherent scenario, eight defined classes and three SWRL rules — all annotated, all validated by HermiT. The expressive constructs were chosen for semantic reasons rather than coverage for its own sake: disjointness axioms turn modelling errors into detectable contradictions, defined classes move classification from human assertion to automated inference, union classes capture cross-cutting purpose-based categories, datatype facets let literal data drive classification, and rules extend the reach of the formalism to multi-variable joins and declarative policies.

Several limitations point to future work. First, interoperability: a production ontology should align with the existing standards in the field, importing or mapping to SSN/SOSA (Haller et al., 2019) and SAREF (ETSI, 2020), so that data could be exchanged with commercial systems rather than living in a private vocabulary. Second, temporal modelling: events currently carry timestamps but no interval semantics, so sequential patterns — "motion followed by door opening within five minutes" — cannot be expressed; an extension with a temporal ontology pattern or with SWRL temporal built-ins would address this. Third, deployment: exposing the ABox through a SPARQL endpoint and feeding it from live device telemetry would turn the model into a running system, with the reasoner acting as a continuous monitoring component. Finally, the automation layer could be deepened: rule conflict detection between `AutomationRule` individuals, priority semantics, and the representation of rule triggers as first-class event patterns are natural next steps.

The broader lesson of the project, however, is methodological. The most valuable moments of the development were the ones where the reasoner *disagreed* — the functional `isConnectedTo` refuted by the two-network gateway, the camera nearly classified as a sensor through a careless domain axiom. In each case the contradiction was not an obstacle but information: the formal semantics confronted the model with the domain and won. That feedback loop, unavailable in informal modelling, is the practical content of the claim that ontologies are knowledge *representation* and not merely knowledge storage.

## References

Baader, F., Calvanese, D., McGuinness, D., Nardi, D., & Patel-Schneider, P. (Eds.) (2007). *The Description Logic Handbook: Theory, Implementation and Applications* (2nd ed.). Cambridge University Press.

ETSI (2020). *SAREF: Smart Applications REFerence ontology.* ETSI TS 103 264 V3.1.1.

Glimm, B., Horrocks, I., Motik, B., Stoilos, G., & Wang, Z. (2014). HermiT: An OWL 2 Reasoner. *Journal of Automated Reasoning*, 53(3), 245–269.

Gruber, T. R. (1993). A Translation Approach to Portable Ontology Specifications. *Knowledge Acquisition*, 5(2), 199–220.

Haller, A., Janowicz, K., Cox, S. J. D., Lefrançois, M., Taylor, K., Le Phuoc, D., Lieberman, J., García-Castro, R., Atkinson, R., & Stadler, C. (2019). The Modular SSN Ontology: A Joint W3C and OGC Standard Specifying the Semantics of Sensors, Observations, Sampling, and Actuation. *Semantic Web*, 10(1), 9–32.

Horrocks, I., Patel-Schneider, P. F., Boley, H., Tabet, S., Grosof, B., & Dean, M. (2004). *SWRL: A Semantic Web Rule Language Combining OWL and RuleML.* W3C Member Submission.

Musen, M. A. (2015). The Protégé Project: A Look Back and a Look Forward. *AI Matters*, 1(4), 4–12.

Noy, N. F., & McGuinness, D. L. (2001). *Ontology Development 101: A Guide to Creating Your First Ontology.* Stanford Knowledge Systems Laboratory Technical Report KSL-01-05.

W3C OWL Working Group (2012). *OWL 2 Web Ontology Language Primer* (2nd ed.). W3C Recommendation.
