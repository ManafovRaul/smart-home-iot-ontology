# Smart Home Ontology — Presentation Slides

13 slides. Each slide has its title, on-slide bullet content, and speaker notes written in presenting voice.

---

## Slide 1 — Title

**On the slide:**

- **A Smart Home Ontology in OWL 2**
- Modelling devices, users, rooms and automation in a residential IoT environment
- Raul — VR544459
- Knowledge Representation, MSc Data Science, University of Verona
- Prof. Matteo Cristani — June 2026

**Speaker notes:**
So, good morning. Today I'm presenting the project I built for this course: an ontology of a smart home, written in OWL 2 with Protégé. The idea is to model everything that lives in a connected house — the devices, the rooms they're in, the people who use them, the events they generate — and then let a reasoner do useful work on top of that model: classify devices automatically, catch inconsistencies, and answer queries that nobody asserted explicitly. I'll walk through why I chose this domain, how I designed it, the main OWL constructs I used, and what the HermiT reasoner actually infers when you run it.

---

## Slide 2 — Motivation & Use Case

**On the slide:**

- Smart homes are heterogeneous: many vendors, many protocols, no shared vocabulary
- Interoperability is the core problem — a thermostat and a motion sensor can't "talk" without shared semantics
- An ontology gives: common vocabulary, machine-readable semantics, automatic classification, consistency checking
- Real standards exist in this space: W3C SSN/SOSA, ETSI SAREF — this project is a compact, self-contained take on the same problem
- Concrete payoffs: "which devices need a battery change?", "is this lock controlled only by admins?" — answered by reasoning, not by code

**Speaker notes:**
So on this slide I want to motivate the domain choice. The smart home is a genuinely good fit for ontologies, not just a toy example. The real-world problem is fragmentation: you buy a Philips light, a Xiaomi sensor, a Nest thermostat, and they all describe themselves differently. The industry has actually responded with ontologies — the W3C has SSN and SOSA for sensors, ETSI has SAREF for smart appliances — so this isn't academic fantasy, it's how the field actually attacks interoperability. My project builds a compact ontology in the same spirit. And the payoff I want to stress is that once knowledge is in the ontology, questions like "which devices are running low on battery" or "did anyone who isn't an admin get control of the front door lock" stop being application code and become reasoning tasks. That's the core idea of knowledge representation: move logic from procedures into declarative axioms.

---

## Slide 3 — Design Methodology

**On the slide:**

- Followed the classic *Ontology Development 101* workflow (Noy & McGuinness)
- 1. Define scope via **competency questions** → 2. enumerate terms → 3. build class hierarchy (top-down) → 4. add properties → 5. add restrictions → 6. populate individuals → 7. iterate with the reasoner
- Tool: **Protégé 5.6**; reasoner: **HermiT**; format: RDF/XML
- Deliberate design choices documented (e.g. why `isConnectedTo` is *not* functional)
- Reasoner run after every major change — errors caught early

**Speaker notes:**
On this slide I want to show that the ontology wasn't built ad hoc. I followed the Ontology Development 101 methodology by Noy and McGuinness, which is the standard reference for this kind of project. I started by writing competency questions — the queries the ontology must be able to answer — because they define the scope and stop you from modelling things you don't need. Then I enumerated the important terms, built the class hierarchy top-down starting from six disjoint top categories, added object and data properties with proper domains and ranges, then the restrictions and defined classes, and finally populated the ABox with thirty-five individuals. One thing I want to emphasise is the iteration loop: I ran HermiT constantly during development. The reasoner caught real mistakes — for example, at one point a functional `isConnectedTo` clashed with the gateway being on two networks, and I had to make a documented design decision about it, which I'll come back to.

---

## Slide 4 — Class Hierarchy Overview

**On the slide:**

- **46 classes**, six disjoint top-level branches:
  - `Device` → Sensor, Actuator, Gateway, Controller, HybridDevice
  - `Sensor` → Temperature, Motion, Humidity, Smoke, Light, CO2 (all disjoint)
  - `Actuator` → SmartLight, SmartLock, Thermostat, SmartPlug, SecurityCamera, SmartBlinds
  - `Room` → Bedroom, Kitchen, LivingRoom, Bathroom, Garage, Hallway
  - `HomeNetwork` → WiFi, Zigbee, Bluetooth, Z-Wave
  - `User` → Admin, Guest, Child, NonAdmin · `Event` → Security, Environment, User, Device · `AutomationRule`
- Plus 8 **defined classes** (SmartDevice, HybridDevice, EnvironmentalSensor, SafetyDevice, NonAdminUser, ChildUser, BatteryPoweredDevice, LowBatteryDevice)

**Speaker notes:**
So this is the skeleton of the TBox. Six top-level categories, all mutually disjoint — a device can never be a room, an event can never be a user. Inside the Device branch, Sensor and Actuator are disjoint from each other, which models the intuition that sensing and acting are different capabilities. That disjointness immediately raises a question: what about devices that do both, like a climate station with a thermometer and a fan? I handle those with the HybridDevice class — it's a Device that *has components* which are sensors and actuators, so it sits beside Sensor and Actuator rather than violating their disjointness. The classes in green on my Protégé screenshot — the defined classes — are where the interesting semantics live, and they get their own slide. The point to take away here is depth: this isn't a flat list, the hierarchy goes three to four levels deep and every leaf is disjoint from its siblings.

---

## Slide 5 — Object Properties & Characteristics

**On the slide:**

- **18 object properties**, every one with explicit domain, range, label, comment
- Inverse pairs: `isLocatedIn`/`hasDevice`, `isConnectedTo`/`hasConnectedDevice`, `controlledBy`/`controls`, `hasOwner`/`owns`, `triggers`/`isTriggeredBy`, `monitors`/`isMonitoredBy`, `hasAutomationRule`/`appliesToDevice`, `hasComponent`/`isComponentOf`
- Characteristics:
  - `isLocatedIn` — **functional + asymmetric** (a device is in exactly one room; rooms aren't in devices)
  - `hasOwner`, plus several data properties — **functional**
  - `communicatesWith` — **symmetric**
  - `hasComponent` — **transitive**
- Design decision: `isConnectedTo` is **not** functional — `Gateway_Main` bridges two networks

**Speaker notes:**
On this slide I want to show the relational structure. Eighteen object properties, organised mostly in inverse pairs so the graph can be traversed in both directions — if a device is located in a room, the room has that device. The characteristics are where OWL gets expressive. `isLocatedIn` is functional, meaning the reasoner will flag an error if I ever assert a device in two rooms — that's a consistency guarantee, not just documentation. It's also asymmetric, ruling out nonsense like a room being located in a sensor. `communicatesWith` is symmetric because device communication is mutual. `hasComponent` is transitive — if the climate station has a sensor module and that module has a chip, the station has the chip. And here's the design decision I mentioned earlier: I initially made `isConnectedTo` functional, but then my gateway individual, which bridges Wi-Fi and Zigbee, made the ontology inconsistent. That's actually correct reasoner behaviour — gateways genuinely connect to multiple networks — so the fix was to drop functionality, and I document that as a modelling lesson: the reasoner forced me to align the axioms with reality.

---

## Slide 6 — Data Properties & XSD Datatypes

**On the slide:**

- **15 data properties**, all typed with XSD datatypes:
  - `hasIPAddress` → `xsd:string` (functional) · `hasBatteryLevel` → `xsd:integer` (functional)
  - `isOnline` → `xsd:boolean` · `hasTemperatureReading`, `hasHumidityReading`, `hasEnergyConsumption`, `hasFloorArea` → `xsd:float`
  - `hasInstallationDate`, `hasTimestamp` → `xsd:dateTime`
  - `hasManufacturer`, `hasFirmwareVersion`, `hasSSID` → `xsd:string`
  - `hasAge`, `hasPriority`, `hasMaxOccupancy` → `xsd:integer`
- Domains restricted where meaningful: `hasTemperatureReading` only on `TemperatureSensor`, `hasSSID` only on `WiFiNetwork`, `hasAge` only on `User`
- Datatypes feed into **datatype restrictions** in defined classes (next slides)

**Speaker notes:**
So this slide covers the attribute layer. Fifteen data properties, each with the correct XSD datatype — booleans for online status, floats for physical readings, dateTime for timestamps, integers for battery percentage and age. Two things I want to highlight. First, domains are as tight as they can sensibly be: a humidity reading can only attach to a humidity sensor, an SSID only to a Wi-Fi network. That keeps the data clean — Protégé will literally not offer the wrong property on the wrong individual. Second, and more interesting, these datatypes aren't passive labels. OWL 2 lets you build class expressions over datatypes with facet restrictions, and I use that twice: `ChildUser` is defined as a user whose age is below eighteen, and `LowBatteryDevice` as a device whose battery is at most twenty percent. So an integer value asserted in the ABox actually drives classification in the TBox — that's an OWL 2 feature specifically, it didn't exist in OWL 1.

---

## Slide 7 — Key OWL 2 Axioms (Manchester Syntax)

**On the slide:**

```
SmartDevice EquivalentTo
    Device and (isConnectedTo some HomeNetwork)

HybridDevice EquivalentTo
    Device and (hasComponent some Sensor)
           and (hasComponent some Actuator)

NonAdminUser EquivalentTo
    User and (not AdminUser)

LowBatteryDevice EquivalentTo
    Device and (hasBatteryLevel some xsd:integer[<= 20])
```

- Plus: `EnvironmentalSensor` and `SafetyDevice` as **unionOf** expressions; 8 groups of **AllDisjointClasses**

**Speaker notes:**
This slide is the heart of the TBox, so I want to take a moment on each example. The first one, SmartDevice, is the classic defined class: equivalence rather than subclass, which means the reasoner classifies *into* it — any device with an asserted network connection becomes a SmartDevice automatically, I never assert that type by hand. The second, HybridDevice, is an intersection of two existential restrictions — it captures composite devices and resolves the Sensor–Actuator disjointness elegantly. The third uses complement: NonAdminUser is literally every user that is not an admin, and from that single axiom the reasoner derives that both GuestUser and ChildUser are subclasses of NonAdminUser — I never stated those subsumptions. The fourth is the OWL 2 datatype restriction I mentioned: a facet on xsd:integer. And not on the slide in full, but I also have two union classes — EnvironmentalSensor as the union of the four ambient sensor types, and SafetyDevice as a union that deliberately crosses the Sensor–Actuator divide, grouping smoke sensors with smart locks because what unites them is *purpose*, not mechanism.

---

## Slide 8 — ABox: Individuals & Assertions

**On the slide:**

- **35 named individuals**: 6 rooms, 2 networks, 3 users, 19 devices, 4 events, 3 automation rules
- Examples:
  - `TempSensor_Kitchen` : TemperatureSensor — `isLocatedIn Kitchen_Main`, `isConnectedTo Zigbee_Mesh`, `hasBatteryLevel 87`, `hasTemperatureReading 22.5`
  - `Gateway_Main` : Gateway — connected to **both** `HomeWiFi_5G` and `Zigbee_Mesh`, `hasIPAddress "192.168.1.10"`
  - `Marco_Russo` : asserted only as `User`, `hasAge 9`
  - `ClimateStation_Bedroom` : asserted only as `Device`, with `hasComponent` a sensor and an actuator
- Several individuals are *deliberately under-typed* — to let the reasoner finish the job

**Speaker notes:**
So on this slide I want to show how the ontology is populated. Thirty-five individuals forming one coherent scenario: a real apartment with six rooms, two networks, a family of three users plus a guest, and a full set of devices with realistic values — battery levels, IP addresses, sensor readings, installation dates. The thing I want you to notice is the last bullet. Some individuals are deliberately under-specified. Marco is asserted just as a User with age nine — not as a ChildUser. The climate station is asserted just as a Device — not as a HybridDevice. The smoke event is asserted just as an Event. That's intentional: it sets up the demonstration on the next slide, where the reasoner derives all those missing types from the definitions. If I had asserted everything by hand, the defined classes would be decorative. This way they're doing real inferential work.

---

## Slide 9 — Defined Classes & the HermiT Reasoner

**On the slide:**

- HermiT 1.4: ontology **consistent**, zero unsatisfiable classes, classification < 2 s
- Inferred class hierarchy:
  - `GuestUser ⊑ NonAdminUser`, `ChildUser ⊑ NonAdminUser`
  - `LowBatteryDevice ⊑ BatteryPoweredDevice`
  - 4 sensor types ⊑ `EnvironmentalSensor`; 4 classes ⊑ `SafetyDevice`
- Inferred individual types:
  - 15 devices → `SmartDevice` · `MotionSensor_Hallway` (15%) → `LowBatteryDevice`
  - `Marco_Russo` → `ChildUser` · `ClimateStation_Bedroom` → `HybridDevice`

**Speaker notes:**
So this is where everything pays off. I run HermiT and three kinds of things happen. First, the sanity check: the ontology is consistent and every class is satisfiable — given eight groups of disjointness axioms plus functional and asymmetric properties, that's a non-trivial guarantee. Second, the inferred class hierarchy: subsumptions I never wrote appear, like GuestUser under NonAdminUser — that follows purely from the complement definition — and LowBatteryDevice under BatteryPoweredDevice, which follows because having a battery level of at most twenty entails having a battery level at all. Third, individual classification: all fifteen connected devices drop into SmartDevice; the hallway motion sensor at fifteen percent battery becomes a LowBatteryDevice; Marco, age nine, becomes a ChildUser and therefore also a NonAdminUser; and the climate station becomes a HybridDevice from its components alone. I want to underline the point: none of these facts is in the file. They are all consequences. That's the difference between an ontology and a database schema.

---

## Slide 10 — SWRL Rules

**On the slide:**

```
R1: MotionSensor(?s) ^ isLocatedIn(?s, ?r) ^ triggers(?s, ?e)
    ^ SecurityEvent(?e)  →  occursIn(?e, ?r)

R2: SmokeSensor(?s) ^ triggers(?s, ?e)  →  SecurityEvent(?e)

R3: SmartLock(?l) ^ controlledBy(?l, ?u)  →  AdminUser(?u)
```

- R1: events inherit the location of the sensor that detected them
- R2: smoke detection is always security-relevant (reclassifies events)
- R3: a security policy expressed declaratively — lock controllers must be admins
- All three fire in HermiT (DL-safe rules, no built-ins)

**Speaker notes:**
On this slide I want to show the SWRL layer, which goes beyond what pure OWL class expressions can do. The key limitation of OWL is that class restrictions can't relate *three* things at once — you can't say "the event's location is the location of the sensor that triggered it" with restrictions alone, because that requires joining variables across properties. Rule R1 does exactly that join: motion sensor in a room triggers a security event, therefore the event occurred in that room. In my ABox, that's how the system learns that the hallway motion event occurred in the hallway. R2 is a reclassification rule: anything a smoke sensor triggers is a security event — and indeed my smoke event, asserted as a plain Event, comes out as a SecurityEvent. R3 is the one I find most interesting conceptually: it encodes a security *policy*. If a smart lock is controlled by someone, that someone must be an admin. In my data the front door lock is controlled only by Alice, who is an admin, so everything is consistent — but if I added a controlledBy assertion to the guest, the rule would force Bob into AdminUser, which clashes with the disjointness between user types, and HermiT would report an inconsistency. So the rule acts as a declarative access-control check. All three rules are DL-safe, no built-ins, and they fire directly in HermiT inside Protégé.

---

## Slide 11 — Competency Questions

**On the slide:**

| # | Question | DL Query | Answer (inferred) |
|---|---|---|---|
| CQ1 | Which devices are smart? | `SmartDevice` | 15 devices |
| CQ2 | Which devices need battery service? | `LowBatteryDevice` | `MotionSensor_Hallway` |
| CQ3 | What monitors the kitchen? | `monitors value Kitchen_Main` | temp + smoke sensors |
| CQ4 | Where did security events occur? | `SecurityEvent and occursIn some Room` | `Event_Motion_001` → Hallway |
| CQ5 | Which users lack admin rights? | `NonAdminUser` | Bob (guest), Marco (child) |

**Speaker notes:**
So on this slide I close the loop with the methodology. These are the competency questions I wrote at the start, and now I show they're actually answerable — each one is a DL query you can paste into Protégé's DL Query tab after running the reasoner. The thing I want to point out is that *every single answer depends on inference*. CQ1's answer set is entirely inferred — SmartDevice has no asserted members. CQ2 depends on the datatype restriction over battery level. CQ4 depends on a SWRL rule, because no occursIn assertion exists in the file. CQ5 depends on the complement-based definition and on Marco being classified as a child from his age. So the evaluation isn't "the ontology contains the data" — it's "the ontology *derives* the answers". That's the standard evaluation method from Ontology Development 101, and I think it's the strongest evidence the design works.

---

## Slide 12 — Conclusions & Future Work

**On the slide:**

- Delivered: 46-class OWL 2 DL ontology, 18 object + 15 data properties, 35 individuals, 3 SWRL rules — consistent under HermiT
- OWL 2 features exercised: defined classes, union / intersection / complement, datatype facet restrictions, property characteristics, AllDisjointClasses
- The reasoner does real work: classification, policy checking, location propagation
- **Future work:**
  - Align with SSN/SOSA and SAREF for true interoperability
  - Temporal modelling of events (intervals, sequences)
  - SPARQL endpoint + live device data ingestion
  - More automation semantics: rule conflicts, priorities, triggers as first-class objects

**Speaker notes:**
So, to conclude. The project delivers a complete, consistent OWL 2 ontology of a smart home that exercises essentially the whole expressive toolbox we covered in the course — and, more importantly, uses each construct for a reason, not as decoration. The defined classes classify, the disjointness axioms catch errors, the rules encode policy. For future work, the honest next step would be alignment with the real standards, SSN/SOSA and SAREF, so the ontology could exchange data with actual commercial systems. Temporal reasoning is the second gap — right now events have timestamps but no interval semantics, so I can't say "motion *followed by* door opening within five minutes", which is what real security logic needs. And finally, connecting the ABox to live data through a SPARQL endpoint would turn this from a model into a running system. Thank you — happy to take questions on any of the axioms.

---

## Slide 13 — References

**On the slide:**

- Gruber, T. (1993). *A Translation Approach to Portable Ontology Specifications.* Knowledge Acquisition, 5(2).
- Noy, N. & McGuinness, D. (2001). *Ontology Development 101.* Stanford KSL Technical Report.
- W3C (2012). *OWL 2 Web Ontology Language Primer* (2nd ed.).
- Horrocks, I. et al. (2004). *SWRL: A Semantic Web Rule Language.* W3C Member Submission.
- Glimm, B. et al. (2014). *HermiT: An OWL 2 Reasoner.* Journal of Automated Reasoning, 53(3).
- Musen, M. (2015). *The Protégé Project: A Look Back and a Look Forward.* AI Matters, 1(4).
- Haller, A. et al. (2019). *The SOSA/SSN Ontology.* Semantic Web Journal, 10(1).
- ETSI (2020). *SAREF: Smart Applications REFerence ontology.* TS 103 264.
- Baader, F. et al. (2007). *The Description Logic Handbook* (2nd ed.). Cambridge UP.

**Speaker notes:**
And these are the references — the foundational ontology papers, the W3C specifications for OWL 2 and SWRL, the tools I used, and the two real-world smart home ontology standards that frame the future-work direction. Thanks again.
