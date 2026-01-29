# TurfSynth AR Constitution

Immutable architectural principles that govern all implementation decisions. These are non-negotiable gates that every feature must pass.

---

## 1. Privacy-First

**Principle**: The system never stores or transmits raw camera, audio, or biometric data.

### Requirements
- Only "Place Fingerprints" (low-dimensional feature vectors) leave the device
- On-device scene understanding before any network transmission
- No facial recognition or individual identification capabilities
- User location precision degraded for storage (city-block level max)

### Verification Gate
> "Can this feature function with only Place Fingerprints and degraded location data?"

---

## 2. Mobile-First

**Principle**: The primary experience runs on smartphones using native AR frameworks.

### Requirements
- ARKit (iOS) and ARCore (Android) as primary rendering targets
- On-device ML models for scene understanding (Core ML, TensorFlow Lite)
- Offline-capable core gameplay loop
- Battery-conscious design (<15% drain per hour active play)

### Verification Gate
> "Does this feature work on a 3-year-old flagship device with intermittent connectivity?"

---

## 3. Safety-Mandatory

**Principle**: Player and public safety supersedes all gameplay considerations.

### Requirements
- Server-side geofencing excludes sensitive locations (schools, hospitals, private residences, government buildings)
- Movement plausibility validation (GPS spoof detection, speed governors)
- Mandatory heads-up attention breaks every 5 minutes of AR mode
- No gameplay incentives that encourage dangerous behavior (traffic areas, cliffs, private property)
- Anti-harassment: blocking, reporting, no unsolicited contact defaults

### Verification Gate
> "Could a player get hurt, harassed, or trespass while using this feature?"

---

## 4. Specification-Driven

**Principle**: Code is generated output; specifications are the source of truth.

### Requirements
- Every feature begins with a specification in `specs/`
- Implementation plans derive from specifications
- Tests validate specification compliance
- Code changes require spec updates first

### Verification Gate
> "Is there a specification that defines this behavior?"

---

## 5. Progressive Disclosure

**Principle**: Complexity reveals itself as players demonstrate mastery.

### Requirements
- Core loop understandable in <2 minutes
- Advanced mechanics unlock through gameplay, not tutorials
- UI surfaces only contextually relevant information
- No overwhelming dashboards or stat screens for new players

### Verification Gate
> "Can a new player enjoy this without reading documentation?"

---

## 6. Environmental Authenticity

**Principle**: Synthlings and game elements derive from real environmental input, not random generation.

### Requirements
- Place Fingerprints encode real visual, audio, and spatial characteristics
- Generated creatures reflect their origin environment (colors, shapes, behaviors)
- Same location produces recognizably similar (but not identical) outputs
- Environmental changes (weather, time, seasons) affect generation

### Verification Gate
> "Would a player recognize where this Synthling came from based on its characteristics?"

---

## Decision Record

When these principles conflict, priority order is:
1. Safety-Mandatory
2. Privacy-First
3. Mobile-First
4. Environmental Authenticity
5. Progressive Disclosure
6. Specification-Driven

---

## Amendment Process

These principles may only be amended through:
1. Written proposal with rationale
2. Analysis of impact on existing specifications
3. Explicit documentation of the change and date

*Last amended: 2026-01-29 (Initial constitution)*
