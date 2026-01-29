# There & Back Again: TurfSynth AR Development Roadmap

**Version**: 2.0
**Last Updated**: 2026-01-29
**Status**: Active Development
**Document Owner**: Engineering Lead

---

## Executive Summary

This document outlines the complete development journey for TurfSynth AR, from initial foundation through city-scale deployment. The roadmap follows a "There & Back Again" structure: **THERE** represents the build phase (Weeks 0-16), while **& BACK** represents sustained operation and scaling (Weeks 17+).

### Key Metrics

| Metric | Target | Timeline |
|--------|--------|----------|
| First Playable | Week 10 | Core loop functional |
| Private Alpha | Week 13 | 10k users, 1-2 cities |
| City Pilot | Week 24 | 200k MAU, multi-city |
| Production Scale | Week 36+ | 1M+ MAU |

### Total Resource Estimate

| Phase | Engineering | Art/Content | Ops/QA | Total FTE-Weeks |
|-------|-------------|-------------|--------|-----------------|
| MVP (0-10) | 320h | 120h | 40h | ~12 |
| Alpha (11-16) | 160h | 80h | 80h | ~8 |
| Scale (17-24) | 240h | 120h | 160h | ~13 |
| **Total** | **720h** | **320h** | **280h** | **~33** |

---

## The Journey Map

```
                              THERE (Build)
    ════════════════════════════════════════════════════════════════════════►

    Week 0        Week 4        Week 8        Week 12       Week 16       Week 24
    ───┼────────────┼────────────┼─────────────┼─────────────┼─────────────┼───
       │            │            │             │             │             │
       ▼            ▼            ▼             ▼             ▼             ▼
    ┌──────┐    ┌──────┐    ┌──────┐     ┌──────┐     ┌──────┐     ┌──────┐
    │ PRE  │───▶│ FOUN │───▶│ CORE │────▶│ MVP  │────▶│ ALPHA│────▶│SCALE │
    │FLIGHT│    │DATION│    │ GEN  │     │      │     │      │     │      │
    └──────┘    └──────┘    └──────┘     └──────┘     └──────┘     └──────┘
       │            │            │             │             │             │
       ▼            ▼            ▼             ▼             ▼             ▼
    DevOps      Safety +     Synthlings    Full Loop    Private       Multi-City
    Setup       Fingerprint  + Turf        Working      10k Users     200k MAU

    ◄════════════════════════════════════════════════════════════════════════
                            & BACK (Operate & Scale)
```

---

## Phase 0: Pre-Flight Checklist (Week 0)

### Objective
Establish development infrastructure, tooling, and team alignment before active development begins.

### Deliverables

| Deliverable | Owner | Status | Notes |
|-------------|-------|--------|-------|
| Development environment setup | DevOps | ✅ Complete | Node.js 22, TypeScript 5.6 |
| CI/CD pipeline | DevOps | ✅ Complete | GitHub Actions |
| Database provisioning | DevOps | ✅ Complete | PostgreSQL 16 + PostGIS |
| Redis cluster | DevOps | ✅ Complete | Redis 7 |
| Monitoring stack | DevOps | ⏳ Pending | Pino logging only |
| H3 + zone data accounts | Engineering | ✅ Complete | OSM Overpass API |
| Unity project scaffold | Engineering | ✅ Complete | AR Foundation 5.x |

### Infrastructure Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           DEVELOPMENT ENVIRONMENT                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐         │
│  │   Local Dev     │    │    Staging      │    │   Production    │         │
│  │                 │    │                 │    │                 │         │
│  │ • npm run dev   │    │ • Preview PRs   │    │ • Blue/Green    │         │
│  │ • Docker Compose│    │ • Integration   │    │ • Auto-scaling  │         │
│  │ • Hot reload    │    │ • E2E tests     │    │ • CDN           │         │
│  └─────────────────┘    └─────────────────┘    └─────────────────┘         │
│           │                      │                      │                   │
│           └──────────────────────┼──────────────────────┘                   │
│                                  │                                          │
│                      ┌───────────▼───────────┐                              │
│                      │    GitHub Actions     │                              │
│                      │                       │                              │
│                      │ • Lint & Typecheck    │                              │
│                      │ • Unit Tests (59)     │                              │
│                      │ • Integration Tests   │                              │
│                      │ • Build Artifacts     │                              │
│                      │ • Coverage Reports    │                              │
│                      └───────────────────────┘                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Team Structure (Phase 0)

| Role | Allocation | Responsibilities |
|------|------------|------------------|
| Senior Engineer | Full-time | Architecture, core services |
| DevOps Engineer | Part-time (20%) | Infrastructure setup |

### Exit Criteria

- [x] Local dev builds run on simulators
- [x] Staging environment accessible
- [x] CI pipeline passes lint/typecheck/test
- [ ] Geofencing DB seeded with test city data
- [ ] APM/monitoring dashboards configured

### Risks & Mitigations

| Risk | Probability | Impact | Mitigation |
|------|-------------|--------|------------|
| Cloud costs exceed budget | Medium | Medium | Use spot instances, set billing alerts |
| Team unfamiliar with H3 | Low | Medium | Allocate learning time, use h3-js docs |
| Unity license constraints | Low | High | Evaluate Unity Personal tier limits early |

---

## Phase 1: Foundation (Weeks 1-4)

### Objective
Build the safety-critical geofencing system and place fingerprint extraction pipeline. These are the foundational services that all gameplay depends upon.

### Phase 1A: Safety Geofencing (Weeks 1-2)

**Effort Estimate**: 68 hours

#### Week-by-Week Breakdown

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WEEK 1                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-2: Database Schema (8h)                                              │
│  ├── CREATE TABLE exclusion_zones (id, geometry, category, buffer_meters)   │
│  ├── CREATE TABLE zone_cache (h3_index, zone_ids[], expires_at)             │
│  └── CREATE INDEX on exclusion_zones USING GIST(geometry)                   │
│                                                                              │
│  Day 3-4: H3 Cache Layer (12h)                                              │
│  ├── Three-tier cache: Local Map → Redis → PostgreSQL                       │
│  ├── Resolution 9 cells (~174m diameter) for gameplay                       │
│  ├── TTL: 5min local, 1hr Redis, persistent DB                              │
│  └── Cache invalidation on zone updates                                     │
│                                                                              │
│  Day 5: Zone Checker Service (8h)                                           │
│  ├── checkLocation(lat, lng) → { allowed, blockedBy? }                      │
│  ├── PostGIS ST_Intersects for geometry checks                              │
│  └── Category-based blocking (school, hospital, government, residential)    │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 2                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-2: Speed Validator (12h)                                             │
│  ├── 30-second rolling window of positions                                  │
│  ├── 15 km/h lockout threshold (prevents drive-by scanning)                 │
│  ├── Graduated warning system: warn → soft lock → hard lock                 │
│  └── 5-minute cooldown after speed violation                                │
│                                                                              │
│  Day 3-4: Spoof Detector (16h)                                              │
│  ├── Impossible velocity detection (>500 km/h jumps)                        │
│  ├── Coordinate jitter analysis (GPS noise vs synthetic)                    │
│  ├── Movement plausibility scoring (Bayesian model)                         │
│  └── Behavioral scoring (not auto-ban, flags for review)                    │
│                                                                              │
│  Day 5: Zone Data Sync (12h)                                                │
│  ├── OSM Overpass API integration                                           │
│  ├── Query: amenity=school|hospital, boundary=protected_area                │
│  ├── Nightly differential sync                                              │
│  └── Admin override system for manual zone adjustments                      │
│                                                                              │
│  Day 6: Integration & Load Testing (8h)                                     │
│  ├── POST /api/v1/location/validate endpoint                                │
│  ├── 10k req/s target, <100ms p95 latency                                   │
│  └── Chaos testing: Redis failure, DB failover                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Technical Architecture

```
                              Location Validation Flow
                              ========================

    ┌──────────┐     ┌──────────┐     ┌──────────┐     ┌──────────┐
    │  Client  │────▶│   Rate   │────▶│  Speed   │────▶│  Spoof   │
    │ Request  │     │  Limiter │     │ Validator│     │ Detector │
    └──────────┘     └──────────┘     └──────────┘     └──────────┘
                           │                │                │
                           ▼                ▼                ▼
                     ┌─────────────────────────────────────────┐
                     │           Zone Checker                   │
                     │                                          │
                     │  ┌─────────┐  ┌─────────┐  ┌─────────┐  │
                     │  │ Local   │─▶│  Redis  │─▶│ PostGIS │  │
                     │  │ Cache   │  │  Cache  │  │  Query  │  │
                     │  └─────────┘  └─────────┘  └─────────┘  │
                     │       5min        1hr        Persistent  │
                     └─────────────────────────────────────────┘
                                          │
                                          ▼
                                   ┌──────────────┐
                                   │   Response   │
                                   │              │
                                   │ • valid      │
                                   │ • h3Cell     │
                                   │ • blockedBy? │
                                   └──────────────┘
```

#### Key Files

| File | Purpose | Lines (Est.) |
|------|---------|--------------|
| `src/services/geofencing/zone-checker.ts` | Core validation logic | 180 |
| `src/services/geofencing/h3-cache.ts` | Multi-tier caching | 220 |
| `src/services/geofencing/speed-validator.ts` | Movement speed checks | 150 |
| `src/services/geofencing/spoof-detector.ts` | GPS spoof detection | 200 |
| `src/services/geofencing/zone-sync.ts` | OSM data synchronization | 250 |
| `src/db/migrations/001_geofencing.sql` | Database schema | 80 |

#### Milestone 1.1
**Geofencing API returns valid/invalid for any GPS coordinate with <100ms p95 latency**

### Phase 1B: Place Fingerprint (Weeks 2-4)

**Effort Estimate**: 92 hours

#### Week-by-Week Breakdown

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           WEEK 2 (Overlap with 1A)                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1: Type Definitions (4h)                                               │
│  ├── PlaceFingerprint interface (<400 bytes serialized)                     │
│  ├── ColorPalette, AudioFeatures, MotionSignature                           │
│  └── FingerprintSubmission, ValidationResult                                │
│                                                                              │
│  Day 2-3: Capture Manager (12h)                                             │
│  ├── Coordinate camera, microphone, motion sensor capture                   │
│  ├── Temporal alignment (ensure all data from same moment)                  │
│  ├── Debounce: max 1 capture per 30 seconds per cell                        │
│  └── Battery-aware capture (reduce frequency when <20% battery)             │
│                                                                              │
│  Day 4-5: Color Extractor (12h)                                             │
│  ├── K-means clustering (k=5) in LAB color space                            │
│  ├── Dominant color calculation (largest cluster centroid)                  │
│  ├── Color temperature estimation (warm/cool classification)                │
│  └── Brightness normalization (0-1 scale)                                   │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 3                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-2: Visual Pipeline (16h)                                             │
│  ├── Image segmentation (sky, ground, structures, vegetation)               │
│  ├── Sobel edge detection → histogram bucketing                             │
│  ├── Depth estimation (ARKit/ARCore LiDAR or ML-based)                      │
│  └── Plane count and orientation extraction                                 │
│                                                                              │
│  Day 3-4: Audio Pipeline (16h)                                              │
│  ├── FFT spectral analysis (256 bins, 44.1kHz sample rate)                  │
│  ├── Onset detection for rhythmic classification                            │
│  ├── Ambient level normalization (dB SPL estimate)                          │
│  └── Frequency centroid and spread calculation                              │
│                                                                              │
│  Day 5: Motion Analyzer (8h)                                                │
│  ├── Gyroscope rotation capture (X, Y, Z euler angles)                      │
│  ├── Accelerometer magnitude (device stability indicator)                   │
│  └── Walking vs stationary classification                                   │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 4                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-2: Assembler (12h)                                                   │
│  ├── Combine all pipeline outputs into PlaceFingerprint                     │
│  ├── Normalize and quantize values for compact representation               │
│  ├── Generate deterministic hash for fingerprint identity                   │
│  └── Ensure total payload <400 bytes                                        │
│                                                                              │
│  Day 3: Validation Gate (8h)                                                │
│  ├── Rate limiting (max 100 submissions/user/hour)                          │
│  ├── Anomaly detection (impossible values, repeated fingerprints)           │
│  ├── Geofencing cross-check (must have valid location first)                │
│  └── Cooldown enforcement per cell                                          │
│                                                                              │
│  Day 4: Offline Queue (8h)                                                  │
│  ├── IndexedDB storage for offline captures                                 │
│  ├── Background sync when connectivity restored                             │
│  ├── Conflict resolution (newer fingerprint wins)                           │
│  └── Queue size limits (max 50 pending submissions)                         │
│                                                                              │
│  Day 5: Integration Testing (8h)                                            │
│  ├── End-to-end capture → submit → store flow                               │
│  ├── Device matrix testing (iOS 14+, Android 10+)                           │
│  ├── Performance profiling (<500ms total capture time)                      │
│  └── Battery drain measurement                                              │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Fingerprint Data Structure

```typescript
interface PlaceFingerprint {
  // Location (16 bytes)
  h3Cell: string;           // 15 chars = 15 bytes + null

  // Visual (48 bytes)
  colorPalette: RGB[];      // 5 colors × 3 bytes = 15 bytes
  dominantColor: RGB;       // 3 bytes
  brightness: number;       // 4 bytes (float32)
  colorTemperature: number; // 4 bytes (float32)
  edgeHistogram: number[];  // 8 buckets × 2 bytes = 16 bytes
  planeCount: number;       // 1 byte
  segmentation: number[];   // 4 categories × 1 byte = 4 bytes

  // Audio (24 bytes)
  ambientLevel: number;     // 4 bytes (float32)
  frequencyCentroid: number;// 4 bytes (float32)
  frequencySpread: number;  // 4 bytes (float32)
  onsetRate: number;        // 4 bytes (float32)
  spectralFlux: number;     // 4 bytes (float32)
  complexity: number;       // 4 bytes (float32)

  // Motion (16 bytes)
  rotationX: number;        // 4 bytes (float32)
  rotationY: number;        // 4 bytes (float32)
  rotationZ: number;        // 4 bytes (float32)
  accelerationMag: number;  // 4 bytes (float32)

  // Metadata (8 bytes)
  timestamp: number;        // 8 bytes (int64 unix ms)

  // Total: ~112 bytes (plus overhead for serialization)
}
```

#### Privacy Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                         PRIVACY-BY-DESIGN                                    │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌───────────────────────────────────────────────────────────────────────┐  │
│  │                        ON-DEVICE PROCESSING                            │  │
│  │                                                                        │  │
│  │   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐          │  │
│  │   │  Camera  │   │   Mic    │   │  Motion  │   │   GPS    │          │  │
│  │   │  Frame   │   │  Buffer  │   │  Sensors │   │ Position │          │  │
│  │   └────┬─────┘   └────┬─────┘   └────┬─────┘   └────┬─────┘          │  │
│  │        │              │              │              │                 │  │
│  │        ▼              ▼              ▼              ▼                 │  │
│  │   ┌──────────────────────────────────────────────────────────┐       │  │
│  │   │              Feature Extraction (On-Device ML)            │       │  │
│  │   │                                                           │       │  │
│  │   │  • K-means color clustering                               │       │  │
│  │   │  • FFT spectral analysis                                  │       │  │
│  │   │  • Edge detection histogram                               │       │  │
│  │   │  • Motion signature computation                           │       │  │
│  │   └──────────────────────────────────────────────────────────┘       │  │
│  │                              │                                        │  │
│  │                              ▼                                        │  │
│  │                    ┌──────────────────┐                               │  │
│  │                    │ Place Fingerprint│                               │  │
│  │                    │   (~400 bytes)   │                               │  │
│  │                    └────────┬─────────┘                               │  │
│  └─────────────────────────────┼─────────────────────────────────────────┘  │
│                                │                                             │
│                                │ ONLY fingerprint transmitted                │
│                                │ (no raw camera/audio data)                  │
│                                ▼                                             │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │                            SERVER                                        ││
│  │                                                                          ││
│  │  • Cannot reconstruct original image/audio from fingerprint             ││
│  │  • Fingerprint is low-dimensional (lossy compression)                   ││
│  │  • Location stored as H3 cell, not exact coordinates                    ││
│  │  • No biometric or PII in fingerprint                                   ││
│  └─────────────────────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Key Files

| File | Purpose | Lines (Est.) |
|------|---------|--------------|
| `src/services/fingerprint/assembler.ts` | Combine pipeline outputs | 150 |
| `src/services/fingerprint/color-extractor.ts` | K-means color clustering | 180 |
| `src/services/fingerprint/audio-pipeline.ts` | FFT spectral analysis | 200 |
| `src/services/fingerprint/visual-pipeline.ts` | Edge detection, segmentation | 220 |
| `src/services/fingerprint/capture-manager.ts` | Coordinate sensor capture | 160 |
| `src/services/fingerprint/validation-gate.ts` | Rate limiting, anomaly detection | 140 |

#### Milestone 1.2
**Extract fingerprint from device camera + microphone in <500ms on iPhone 12 / Pixel 6**

### Phase 1 Exit Criteria

| Criterion | Target | Measurement |
|-----------|--------|-------------|
| Geofencing latency | <100ms p95 | Load test with k6 |
| Geofencing throughput | 10k req/s | Sustained 5 minutes |
| Fingerprint extraction | <500ms | Device profiling |
| Cross-integration | Pass | Fingerprint calls geofencing |
| Privacy audit | Pass | No raw data in network logs |
| Unit test coverage | ≥70% | Coverage report |
| CI pipeline | Green | All checks pass |

---

## Phase 2: Core Generation (Weeks 5-8)

### Objective
Build the gameplay systems: procedural Synthling creatures from fingerprints and territorial turf mechanics with influence decay.

### Phase 2A: Synthling Generation (Weeks 5-7)

**Effort Estimate**: 72h engineering + 120h art

#### Synthling Concept

Synthlings are procedurally generated creatures whose attributes derive from the Place Fingerprint of their spawn location. Each Synthling is unique to its birthplace.

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                    FINGERPRINT → SYNTHLING MAPPING                           │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Fingerprint Input              Synthling Attribute                          │
│  ─────────────────              ───────────────────                          │
│                                                                              │
│  colorPalette[0..4]      ───▶   Body color, accent colors, glow tint        │
│  brightness              ───▶   Luminosity, transparency                     │
│  colorTemperature        ───▶   Warm (fire-like) vs Cool (ice-like)         │
│                                                                              │
│  edgeHistogram           ───▶   Body geometry complexity                     │
│  planeCount              ───▶   Number of appendages/facets                  │
│  segmentation            ───▶   Elemental affinity (sky→air, ground→earth)  │
│                                                                              │
│  ambientLevel            ───▶   Size (louder = larger)                       │
│  frequencyCentroid       ───▶   Voice pitch                                  │
│  onsetRate               ───▶   Movement speed, animation tempo              │
│  complexity              ───▶   Behavioral complexity                        │
│                                                                              │
│  rotationX/Y/Z           ───▶   Idle animation pose preferences              │
│  accelerationMag         ───▶   Energy level, aggression                     │
│                                                                              │
│  h3Cell                  ───▶   Spawn location (for evolution chains)        │
│  timestamp               ───▶   Time-of-day variant (nocturnal/diurnal)     │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Archetype System

30 base archetypes organized into 5 elemental families:

| Family | Archetypes | Visual Theme | Sound Theme |
|--------|------------|--------------|-------------|
| **Lumina** | 6 | Light, crystalline, geometric | Chimes, harmonics |
| **Terra** | 6 | Earthy, organic, textured | Rumbles, thuds |
| **Aqua** | 6 | Fluid, translucent, flowing | Bubbles, waves |
| **Aero** | 6 | Wispy, feathered, light | Whistles, whooshes |
| **Void** | 6 | Dark, abstract, fractal | Drones, glitches |

#### Evolution System

```
                           EVOLUTION CHAINS
                           ================

    Stage 1 (Common)              Stage 2 (Rare)              Stage 3 (Legendary)
    ────────────────              ──────────────              ───────────────────

    Base archetype         Requires 3 unique           Requires 5 unique
    from fingerprint       locations + 50 influence    locations + 200 influence

    ┌─────────────┐        ┌─────────────┐             ┌─────────────┐
    │  Sparklet   │───────▶│  Gleamling  │────────────▶│  Radiant    │
    │  (Lumina)   │        │  (Lumina)   │             │  (Lumina)   │
    └─────────────┘        └─────────────┘             └─────────────┘
         │                       │                           │
         │                       │                           │
    Visual changes:        Visual changes:             Visual changes:
    • +1 facet             • +2 facets                 • +3 facets
    • Glow intensity 1.2x  • Glow intensity 1.5x      • Glow intensity 2.0x
    • Size +10%            • Size +25%                 • Size +50%

    Stat changes:          Stat changes:               Stat changes:
    • Base stats           • All stats +20%            • All stats +50%
    • 1 ability            • 2 abilities               • 3 abilities + ultimate
```

#### Week-by-Week Breakdown

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WEEK 5                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1: Archetype Schema (6h)                                               │
│  ├── Base archetype definitions (30 archetypes)                             │
│  ├── Family/element classification                                          │
│  ├── Stat ranges and scaling formulas                                       │
│  └── Evolution chain definitions                                            │
│                                                                              │
│  Day 2: Registry Service (6h)                                               │
│  ├── Load archetype definitions from config                                 │
│  ├── Query by family, element, rarity                                       │
│  └── Cache for fast lookup                                                  │
│                                                                              │
│  Day 3: Spawn Resolver (10h)                                                │
│  ├── Determine spawn eligibility for cell                                   │
│  ├── Rarity calculation based on cell control                               │
│  ├── Time-of-day modifiers                                                  │
│  └── Cooldown enforcement (1 spawn per cell per 30min)                      │
│                                                                              │
│  Day 4-5: Attribute Deriver (14h)                                           │
│  ├── Map fingerprint values to archetype selection                          │
│  ├── Deterministic variation within archetype bounds                        │
│  ├── Color palette application                                              │
│  └── Stat calculation with fingerprint modifiers                            │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 6                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-3: Visual Generator (24h)                                            │
│  ├── Shader parameter mapping from attributes                               │
│  ├── Procedural mesh deformation                                            │
│  ├── Particle system configuration                                          │
│  └── LOD (Level of Detail) system for performance                           │
│                                                                              │
│  Day 4-5: Audio Generator (16h)                                             │
│  ├── Procedural voice synthesis parameters                                  │
│  ├── Ambient sound loop generation                                          │
│  ├── Interaction sound effects                                              │
│  └── Spatial audio positioning                                              │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 7                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1-2: Evolution Manager (14h)                                           │
│  ├── Track evolution progress per Synthling                                 │
│  ├── Location diversity validation                                          │
│  ├── Influence cost calculation                                             │
│  └── Evolution animation triggers                                           │
│                                                                              │
│  Day 3-4: Integration Tests (14h)                                           │
│  ├── Fingerprint → Synthling determinism tests                              │
│  ├── Same fingerprint = same Synthling verification                         │
│  ├── Evolution chain progression tests                                      │
│  └── Visual/audio asset loading tests                                       │
│                                                                              │
│  Day 5: Performance Tuning (8h)                                             │
│  ├── GPU profiling on target devices                                        │
│  ├── Draw call batching                                                     │
│  ├── Texture atlas optimization                                             │
│  └── Target: <100ms generation, 60fps rendering                             │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Art Pipeline (Parallel Track)

| Week | Deliverable | Asset Count | Hours |
|------|-------------|-------------|-------|
| 5-6 | Base archetype meshes | 10 archetypes | 40h |
| 6-7 | Remaining archetypes | 20 archetypes | 80h |
| 7 | Evolution variants | 60 variants | Derived procedurally |
| 7 | Particle effects | 15 effect types | 20h |
| 7 | UI elements | Icons, frames | 20h |

#### Key Files

| File | Purpose | Lines (Est.) |
|------|---------|--------------|
| `src/services/synthling/registry.ts` | Archetype definitions | 300 |
| `src/services/synthling/spawn-resolver.ts` | Spawn eligibility | 180 |
| `src/services/synthling/attribute-deriver.ts` | Fingerprint → attributes | 250 |
| `src/services/synthling/evolution-manager.ts` | Evolution tracking | 200 |
| `unity/Assets/Scripts/Synthlings/SynthlingRenderer.cs` | Visual generation | 400 |
| `unity/Assets/Scripts/Synthlings/SynthlingAudio.cs` | Audio generation | 250 |

#### Milestone 2.1
**Generate visually distinct Synthlings from fingerprints with <100ms GPU time**

### Phase 2B: Turf Mechanics (Weeks 6-8)

**Effort Estimate**: 88 hours

#### Turf System Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          TURF MECHANICS OVERVIEW                             │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│                              DISTRICT                                        │
│                         (Collection of cells)                                │
│                                                                              │
│         ┌─────────────────────────────────────────────┐                     │
│         │     ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲      │                     │
│         │    ╱A ╲  ╱A ╲  ╱A ╲  ╱B ╲  ╱B ╲  ╱B ╲     │                     │
│         │   ╱────╲╱────╲╱────╲╱────╲╱────╲╱────╲    │                     │
│         │   ╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱╲    ╱    │                     │
│         │    ╲A ╱  ╲A ╱  ╲? ╱  ╲B ╱  ╲B ╱  ╲B ╱     │                     │
│         │     ╲╱    ╲╱    ╲╱    ╲╱    ╲╱    ╲╱      │                     │
│         │      H3 Resolution 9 cells (~174m)         │                     │
│         └─────────────────────────────────────────────┘                     │
│                                                                              │
│  Legend:  A = Crew Alpha (controller)                                        │
│           B = Crew Beta (controller)                                         │
│           ? = Contested (no clear controller)                                │
│                                                                              │
│  Control = Crew with highest influence in cell                               │
│  Contested = Top 2 crews within 10% of each other                           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Influence System

```
                         INFLUENCE ECONOMY
                         =================

    ┌─────────────────────────────────────────────────────────────────────────┐
    │                        INFLUENCE SOURCES                                 │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                                                                          │
    │  Source                      Base Amount    Cooldown    Notes            │
    │  ──────                      ───────────    ────────    ─────            │
    │  Fingerprint submission      +10            30 sec      Per cell         │
    │  Synthling capture           +5             None        Per capture      │
    │  Contract completion         +25            Daily       Per contract     │
    │  Outpost passive             +5/hour        Continuous  Per outpost      │
    │  Raid success                +50            10 min      Per raid         │
    │  Raid defense                +25            None        On defense       │
    │                                                                          │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                        INFLUENCE DECAY                                   │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                                                                          │
    │  Formula: I(t) = I₀ × e^(-λt)                                           │
    │                                                                          │
    │  Where:                                                                  │
    │    I₀ = Initial influence                                               │
    │    λ  = ln(2) / half_life                                               │
    │    t  = Time elapsed (hours)                                            │
    │    half_life = 48 hours (configurable)                                  │
    │                                                                          │
    │  Example: 100 influence after 48h = 50, after 96h = 25                  │
    │                                                                          │
    │  Decay processed hourly by background job                               │
    │                                                                          │
    └─────────────────────────────────────────────────────────────────────────┘
```

#### Outpost System

```
                              OUTPOST MODULES
                              ===============

    ┌─────────────────┬────────────────────────────────────────────────────────┐
    │     Module      │                    Effect                              │
    ├─────────────────┼────────────────────────────────────────────────────────┤
    │                 │                                                        │
    │  SCANNER        │  Increases Synthling spawn rate in cell               │
    │  Lv1: +10%      │  Lv2: +25%     Lv3: +50%                              │
    │                 │                                                        │
    ├─────────────────┼────────────────────────────────────────────────────────┤
    │                 │                                                        │
    │  AMPLIFIER      │  Increases passive influence generation               │
    │  Lv1: +20%      │  Lv2: +50%     Lv3: +100%                             │
    │                 │                                                        │
    ├─────────────────┼────────────────────────────────────────────────────────┤
    │                 │                                                        │
    │  SHIELD         │  Reduces raid damage taken                            │
    │  Lv1: -10%      │  Lv2: -25%     Lv3: -40%                              │
    │                 │                                                        │
    ├─────────────────┼────────────────────────────────────────────────────────┤
    │                 │                                                        │
    │  BEACON         │  Increases fingerprint influence bonus                │
    │  Lv1: +5        │  Lv2: +10      Lv3: +20                               │
    │                 │                                                        │
    └─────────────────┴────────────────────────────────────────────────────────┘

    Outpost Limits:
    • 1 outpost per cell per crew
    • Max 3 modules per outpost
    • Outpost has 100 HP base
    • Destroyed at 0 HP (respawn cooldown: 24h)
```

#### Raid System

```
                              RAID MECHANICS
                              ==============

    ┌─────────────────────────────────────────────────────────────────────────┐
    │                          RAID INITIATION                                 │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                                                                          │
    │  Requirements:                                                           │
    │  • Attacker must be in adjacent cell                                    │
    │  • Attacker must have ≥50 influence in origin cell                      │
    │  • Target cell must have enemy outpost OR ≥100 enemy influence          │
    │  • 10-minute cooldown between raids from same origin                    │
    │                                                                          │
    │  Attack Power Calculation:                                              │
    │  • Base: Attacker's influence in origin cell                            │
    │  • Bonus: +10% per crew member in origin cell                           │
    │  • Bonus: +5% per Synthling in attacker's squad                         │
    │  • Cap: 500 max attack power                                            │
    │                                                                          │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                          RAID RESOLUTION                                 │
    ├─────────────────────────────────────────────────────────────────────────┤
    │                                                                          │
    │  Defense Power Calculation:                                             │
    │  • Base: Defender's influence in target cell                            │
    │  • Bonus: +50% if outpost present                                       │
    │  • Bonus: Shield module reduction applied                               │
    │                                                                          │
    │  Outcome:                                                               │
    │  • If Attack > Defense × 1.2: Decisive Victory                          │
    │    → Transfer 30% of defender influence                                 │
    │    → Deal 40 damage to outpost                                          │
    │                                                                          │
    │  • If Attack > Defense: Victory                                         │
    │    → Transfer 15% of defender influence                                 │
    │    → Deal 20 damage to outpost                                          │
    │                                                                          │
    │  • If Attack > Defense × 0.8: Stalemate                                 │
    │    → No influence transfer                                              │
    │    → Both sides lose 5% influence                                       │
    │                                                                          │
    │  • If Attack ≤ Defense × 0.8: Defeat                                    │
    │    → Attacker loses 10% influence in origin                             │
    │    → Defender gains +25 influence (defense bonus)                       │
    │                                                                          │
    │  Resolution Time: 5 minutes (async, not real-time)                      │
    │                                                                          │
    └─────────────────────────────────────────────────────────────────────────┘
```

#### Week-by-Week Breakdown

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                              WEEK 6                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1: Database Schema (6h)                                                │
│  ├── turf_cells (h3_index, district_id, controlling_crew_id, ...)          │
│  ├── influence_events (id, cell_h3, crew_id, user_id, source, amount, ...)  │
│  ├── outposts (id, cell_h3, crew_id, modules[], health, ...)               │
│  └── raids (id, attacker_crew_id, target_cell, status, ...)                │
│                                                                              │
│  Day 2: Data Models (6h)                                                    │
│  ├── TurfCell, InfluenceEvent, Outpost, Raid TypeScript interfaces         │
│  ├── Zod validation schemas                                                 │
│  └── Database row ↔ domain object mappers                                   │
│                                                                              │
│  Day 3: H3 Cell Setup (8h)                                                  │
│  ├── Resolution 9 cell generation for target area                          │
│  ├── District boundary assignment                                           │
│  ├── Neighbor relationship caching                                          │
│  └── Cell metadata (zone type, spawn modifiers)                             │
│                                                                              │
│  Day 4-5: Influence Manager (16h)                                           │
│  ├── awardInfluence(cell, crew, user, source, multiplier)                   │
│  ├── updateCellControl(cell) → controllingCrewId                            │
│  ├── getCellInfluence(cell) → { crews: Map<crewId, influence> }             │
│  ├── getCrewTotalInfluence(crew) → number                                   │
│  └── getCellLeaderboard(cell, limit) → LeaderboardEntry[]                   │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 7                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1: Decay Processor (8h)                                                │
│  ├── processDecay() → number (cells processed)                              │
│  ├── Hourly cron job scheduling                                             │
│  ├── Batch processing (1000 cells per iteration)                            │
│  └── Control recalculation after decay                                      │
│                                                                              │
│  Day 2-3: Outpost Manager (16h)                                             │
│  ├── deployOutpost(cell, crew, user) → Outpost                              │
│  ├── installModule(outpostId, moduleType, level)                            │
│  ├── damageOutpost(outpostId, damage) → remainingHealth                     │
│  ├── destroyOutpost(outpostId) → void                                       │
│  └── processOutpostTick() → influence generated                             │
│                                                                              │
│  Day 4: District Aggregation (6h)                                           │
│  ├── calculateDistrictControl(districtId) → controllingCrewId              │
│  ├── getDistrictLeaderboard(districtId) → LeaderboardEntry[]                │
│  └── District-level bonuses for full control                                │
│                                                                              │
│  Day 5: Raid Engine (16h)                                                   │
│  ├── initiateRaid(attackerCrew, targetCell, attackPower) → Raid             │
│  ├── calculateDefensePower(cell) → number                                   │
│  ├── resolveRaid(raidId) → RaidResult                                       │
│  └── Async resolution with 5-minute delay                                   │
│                                                                              │
├─────────────────────────────────────────────────────────────────────────────┤
│                              WEEK 8                                          │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  Day 1: Spawn Seeder (6h)                                                   │
│  ├── Synthling spawn rate based on cell control                             │
│  ├── Rarity modifiers (controlled cells = rarer spawns)                     │
│  ├── Time-of-day spawn patterns                                             │
│  └── Integration with Synthling generation                                  │
│                                                                              │
│  Day 2-3: API Endpoints (16h)                                               │
│  ├── GET /api/v1/turf/cell/:h3Index                                         │
│  ├── GET /api/v1/turf/district/:districtId                                  │
│  ├── GET /api/v1/turf/district/:districtId/leaderboard                      │
│  ├── POST /api/v1/turf/outpost/deploy                                       │
│  ├── POST /api/v1/turf/outpost/:id/module                                   │
│  ├── POST /api/v1/turf/raid                                                 │
│  └── GET /api/v1/turf/raid/:id                                              │
│                                                                              │
│  Day 4: Integration Tests (8h)                                              │
│  ├── Influence award → control change flow                                  │
│  ├── Decay → control flip scenarios                                         │
│  ├── Raid initiation → resolution → outcome                                 │
│  └── Outpost deployment → passive generation                                │
│                                                                              │
│  Day 5: Load Testing (8h)                                                   │
│  ├── 10k influence updates/second target                                    │
│  ├── Raid resolution <500ms                                                 │
│  ├── Concurrent raid handling                                               │
│  └── Database query optimization                                            │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

#### Key Files

| File | Purpose | Lines (Est.) |
|------|---------|--------------|
| `src/services/turf/influence-manager.ts` | Influence operations | 280 |
| `src/services/turf/outpost-manager.ts` | Outpost CRUD | 220 |
| `src/services/turf/raid-engine.ts` | Raid mechanics | 300 |
| `src/services/turf/decay-processor.ts` | Hourly decay | 120 |
| `src/services/turf/district-aggregator.ts` | District stats | 150 |
| `src/db/migrations/003_turf.sql` | Turf schema | 100 |

#### Milestone 2.2
**Territory changes hands through influence + raids with 10k updates/sec throughput**

### Phase 2 Exit Criteria

| Criterion | Target | Measurement |
|-----------|--------|-------------|
| Synthling generation | <100ms GPU | Device profiling |
| Deterministic output | 100% | Same fingerprint = same Synthling |
| Influence throughput | 10k/sec | Load test |
| Raid resolution | <500ms | API latency |
| Spawn integration | Pass | Rarity affected by control |
| Art assets | 30 archetypes | Asset manifest |

---

## Phase 3: MVP Integration (Weeks 9-10)

### Objective
Wire all systems together into a playable end-to-end experience. Validate the core loop on real devices.

### The Core Loop (Implemented)

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            THE CORE LOOP                                     │
│                                                                              │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  1. WALK                                                               │ │
│   │     Player physically moves to new H3 cell                             │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼                                         │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  2. VALIDATE                                                           │ │
│   │     Location checked against geofencing                                │ │
│   │     • Exclusion zone check (school, hospital, etc.)                    │ │
│   │     • Speed validation (15 km/h limit)                                 │ │
│   │     • Spoof detection (behavioral scoring)                             │ │
│   │                                                                        │ │
│   │     IF BLOCKED → Show message, no gameplay in this cell               │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼ (Allowed)                               │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  3. SCAN                                                               │ │
│   │     Extract Place Fingerprint from environment                         │ │
│   │     • Camera → Color palette, brightness, edges                        │ │
│   │     • Microphone → Ambient level, frequency, complexity                │ │
│   │     • Sensors → Motion signature, orientation                          │ │
│   │                                                                        │ │
│   │     Submit fingerprint → +10 Influence to crew                         │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼                                         │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  4. ENCOUNTER                                                          │ │
│   │     Synthling may spawn based on:                                      │ │
│   │     • Cell control status (controlled = rarer spawns)                  │ │
│   │     • Outpost scanner bonus                                            │ │
│   │     • Time of day                                                      │ │
│   │     • RNG with seeded fingerprint hash                                 │ │
│   │                                                                        │ │
│   │     IF SPAWN → Show AR Synthling derived from fingerprint              │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼                                         │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  5. CAPTURE (if Synthling present)                                     │ │
│   │     Tap to capture (minigame optional)                                 │ │
│   │     • Add Synthling to collection                                      │ │
│   │     • +5 Influence                                                     │ │
│   │     • Track location for evolution progress                            │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼                                         │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  6. CONTROL                                                            │ │
│   │     View cell/district control on map                                  │ │
│   │     • See crew influence breakdown                                     │ │
│   │     • Deploy outpost if controlling                                    │ │
│   │     • Initiate raid on adjacent enemy cells                            │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                    │                                         │
│                                    ▼                                         │
│   ┌───────────────────────────────────────────────────────────────────────┐ │
│   │  BACKGROUND PROCESSES                                                  │ │
│   │     • Influence decays hourly (48h half-life)                          │ │
│   │     • Outposts generate passive influence                              │ │
│   │     • Raids resolve asynchronously (5 min)                             │ │
│   │     • Push notifications for control changes                           │ │
│   └───────────────────────────────────────────────────────────────────────┘ │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Integration Tasks

| Task | Effort | Owner | Status |
|------|--------|-------|--------|
| Full flow smoke tests | 8h | QA | Pending |
| Cross-service contract validation | 4h | Engineering | Pending |
| Error handling at boundaries | 4h | Engineering | Pending |
| Device matrix testing | 16h | QA | Pending |
| Battery drain profiling | 8h | Engineering | Pending |

### Device Test Matrix

| Device | OS Version | AR Support | Priority |
|--------|------------|------------|----------|
| iPhone 12 | iOS 16+ | ARKit | P0 |
| iPhone 14 Pro | iOS 17+ | ARKit + LiDAR | P0 |
| Pixel 6 | Android 13+ | ARCore | P0 |
| Pixel 8 | Android 14+ | ARCore | P1 |
| Samsung S23 | Android 14+ | ARCore | P1 |
| iPhone SE (3rd) | iOS 16+ | ARKit (limited) | P2 |
| Pixel 4a | Android 12+ | ARCore | P2 |
| Samsung A54 | Android 13+ | ARCore | P2 |

### Performance Targets

| Metric | Target | Acceptable | Current |
|--------|--------|------------|---------|
| App cold start | <3s | <5s | TBD |
| Location validation | <100ms | <200ms | ✅ |
| Fingerprint extraction | <500ms | <750ms | TBD |
| Synthling render | <100ms | <150ms | TBD |
| Battery drain | <15%/hr | <20%/hr | TBD |
| Memory usage | <200MB | <300MB | TBD |
| Network data | <5MB/hr | <10MB/hr | TBD |

### Phase 3 Exit Criteria

| Criterion | Target | Status |
|-----------|--------|--------|
| Complete loop playable | Pass | Pending |
| No crashes in 1hr session | Pass | Pending |
| All specs pass acceptance | Pass | Pending |
| API documentation | Complete | Pending |
| Runbook | Complete | ✅ |

---

## Phase 4: Alpha Pilot (Weeks 11-16)

### Objective
Launch a private alpha with real users in 1-2 test cities. Gather data on real-world behavior, balance, and technical issues.

### Phase 4A: Production Hardening (Weeks 11-12)

#### Tasks

| Task | Effort | Priority | Owner |
|------|--------|----------|-------|
| Admin dashboard (zone management) | 16h | P0 | Engineering |
| Analytics integration | 8h | P0 | Engineering |
| Crash reporting (Sentry) | 4h | P0 | Engineering |
| Rate limiting refinement | 8h | P1 | Engineering |
| Feature flags (LaunchDarkly) | 4h | P1 | Engineering |
| App store preparation | 8h | P0 | Engineering |
| Privacy policy & ToS | 8h | P0 | Legal |
| Accessibility audit | 8h | P1 | QA |

#### Admin Dashboard

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          ADMIN DASHBOARD                                     │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  ZONE MANAGEMENT                                                         ││
│  │                                                                          ││
│  │  • Add/edit/remove exclusion zones                                       ││
│  │  • Override OSM data with manual boundaries                              ││
│  │  • Set buffer distances per zone category                                ││
│  │  • View zone coverage map                                                ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  USER MANAGEMENT                                                         ││
│  │                                                                          ││
│  │  • View flagged users (spoof detection alerts)                           ││
│  │  • Issue warnings / temporary bans                                       ││
│  │  • Review appeal queue                                                   ││
│  │  • Crew moderation (rename, disband)                                     ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  LIVE METRICS                                                            ││
│  │                                                                          ││
│  │  • DAU / MAU / Sessions                                                  ││
│  │  • Influence generated (total, by source)                                ││
│  │  • Raids initiated / resolved                                            ││
│  │  • Synthlings captured                                                   ││
│  │  • API latency percentiles                                               ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
│  ┌─────────────────────────────────────────────────────────────────────────┐│
│  │  BALANCE CONTROLS                                                        ││
│  │                                                                          ││
│  │  • Influence decay rate (half-life slider)                               ││
│  │  • Spawn rate multipliers                                                ││
│  │  • Raid damage scaling                                                   ││
│  │  • Event toggles (2x XP weekend, etc.)                                   ││
│  └─────────────────────────────────────────────────────────────────────────┘│
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Phase 4B: Private Alpha (Weeks 13-16)

#### Rollout Plan

| Week | Milestone | User Count | Cities |
|------|-----------|------------|--------|
| 13 | Friends & Family | 100 | 1 (SF) |
| 14 | Closed Beta | 1,000 | 1 (SF) |
| 15 | Expanded Beta | 5,000 | 2 (SF + LA) |
| 16 | Alpha Complete | 10,000 | 2 |

#### What We Learn

| Category | Questions | Measurement |
|----------|-----------|-------------|
| **Engagement** | Do players return daily? | DAU/MAU ratio |
| **Engagement** | How long are sessions? | Avg session length |
| **Balance** | Is influence decay too fast/slow? | Control flip frequency |
| **Balance** | Are raids too powerful? | Win rate distribution |
| **Technical** | GPS accuracy in urban areas? | Location validation errors |
| **Technical** | Device diversity issues? | Crash-free rate by device |
| **Privacy** | Do users understand fingerprinting? | Support ticket themes |
| **Safety** | Do exclusion zones work? | False positive rate |

#### Success Metrics

| Metric | Target | Critical Threshold |
|--------|--------|-------------------|
| DAU/MAU | >30% | >20% |
| Sessions/day | >2.5 | >1.5 |
| Crash-free sessions | >99% | >97% |
| Avg session length | >8 min | >5 min |
| Synthlings captured/session | >3 | >1 |
| Fingerprints submitted/session | >5 | >2 |
| 7-day retention | >40% | >25% |
| NPS | >30 | >10 |

### Phase 4 Exit Criteria

| Criterion | Target | Status |
|-----------|--------|--------|
| 10k users onboarded | Pass | Pending |
| No P0 bugs for 2 weeks | Pass | Pending |
| Balance tuning complete | Pass | Pending |
| Zone coverage >99% | Pass | Pending |
| Scaling runway confirmed | Pass | Pending |

---

## Phase 5: Scale to S1 (Weeks 17-24)

### Objective
Scale from 10k alpha users to 200k MAU across multiple cities. Prepare infrastructure for production load.

### Infrastructure Scaling

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                      INFRASTRUCTURE EVOLUTION                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ALPHA (10k users)                 S1 (200k MAU)                            │
│  ─────────────────                 ────────────                              │
│                                                                              │
│  PostgreSQL                        PostgreSQL (Primary)                      │
│  └── Single instance    ────▶     ├── 2x Read replicas                      │
│                                   ├── PgBouncer connection pooling          │
│                                   └── Partition by district_id              │
│                                                                              │
│  Redis                             Redis Cluster                             │
│  └── Single instance    ────▶     ├── 3 master nodes                        │
│                                   ├── 3 replica nodes                       │
│                                   └── Persistence enabled                   │
│                                                                              │
│  API Server                        API Server Cluster                        │
│  └── 2 instances        ────▶     ├── Kubernetes deployment                 │
│                                   ├── HPA (2-10 pods)                       │
│                                   └── Regional load balancing               │
│                                                                              │
│  Background Jobs                   Job Queue                                 │
│  └── Cron on single     ────▶     ├── BullMQ with Redis                     │
│      instance                     ├── Dedicated worker pods                 │
│                                   └── Priority queues                       │
│                                                                              │
│  CDN                               Global CDN                                │
│  └── None               ────▶     ├── CloudFront / Fastly                   │
│                                   ├── Asset caching                         │
│                                   └── API caching for GET                   │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Team Scaling

| Role | Alpha | S1 | Notes |
|------|-------|-----|-------|
| Senior Engineer | 1 | 2 | Add mobile specialist |
| Backend Engineer | 0 | 1-2 | Scale infrastructure |
| QA Engineer | 0.5 | 1 | Dedicated testing |
| Content/Art | 0.5 | 1-2 | New archetypes, events |
| Community Manager | 0 | 1 | User support, Discord |
| Trust & Safety | 0 | 0.5 | Moderation, appeals |

### Additional Features (S1)

| Feature | Effort | Priority | Notes |
|---------|--------|----------|-------|
| AR Visualization layer | 40h | P0 | Full AR Synthling rendering |
| Multiplayer presence | 60h | P1 | See crew members on map |
| Real-time events | 40h | P1 | Live influence updates |
| Audio synthesis | 30h | P2 | Ambient soundscapes |
| Crew chat | 40h | P2 | In-app communication |
| Seasonal events | 30h | P2 | Limited-time content |

### Cost Projections

| Component | Alpha (10k) | S1 (200k) | Notes |
|-----------|-------------|-----------|-------|
| PostgreSQL | $200/mo | $800/mo | RDS db.r6g.large + replicas |
| Redis | $50/mo | $300/mo | ElastiCache cluster |
| API Compute | $100/mo | $600/mo | ECS/EKS |
| CDN | $0 | $200/mo | Asset delivery |
| Monitoring | $50/mo | $200/mo | Datadog/NewRelic |
| **Total** | **$400/mo** | **$2,100/mo** | |
| **Per DAU** | $0.04 | $0.035 | Improving with scale |

### Phase 5 Exit Criteria

| Criterion | Target | Status |
|-----------|--------|--------|
| 100k MAU sustained | Pass | Pending |
| Infrastructure costs <$0.05/DAU | Pass | Pending |
| 99.9% uptime (30 days) | Pass | Pending |
| Support volume <50 tickets/day | Pass | Pending |
| All S1 features shipped | Pass | Pending |

---

## & Back: Sustainable Operations

### Ongoing Operations Cadence

#### Daily
- Monitor error rates and latency
- Review spoof detection alerts
- Process user support queue

#### Weekly
- Zone data sync verification
- Balance telemetry review
- Team standup / retrospective

#### Monthly
- Security audit
- Performance regression testing
- Cost optimization review
- Content refresh planning

#### Quarterly
- Major feature release
- Infrastructure capacity review
- Team growth planning
- Strategic roadmap update

### Content Velocity

| Content Type | Frequency | Effort per Unit |
|--------------|-----------|-----------------|
| New archetypes | 2/month | 12h engineering + 8h art |
| Events | 1/month | 40h total |
| Balance patches | 2/month | 8h |
| Bug fixes | Weekly | 8h/week avg |
| Infrastructure updates | Monthly | 16h |

### Long-Term Roadmap (Post-S1)

| Quarter | Focus | Key Features |
|---------|-------|--------------|
| Q3 | Social | Crew leagues, tournaments, leaderboards |
| Q4 | Depth | Trading, breeding, advanced evolution |
| Q1+1 | Expansion | New regions, localization, partnerships |
| Q2+1 | Platform | API for community tools, modding support |

---

## Risk Matrix

| Risk | Impact | Probability | Mitigation |
|------|--------|-------------|------------|
| Zone data gaps | High | Medium | Multiple sources, user reporting |
| GPS spoof waves | High | Medium | Behavioral scoring, manual review |
| Model too slow on devices | Medium | Low | Tiered quality, CPU fallback |
| Privacy audit failure | Critical | Low | Pre-launch adversarial review |
| App store rejection | High | Low | Early guideline review |
| Player harassment | High | Medium | Async-only, no live tracking |
| Team burnout | High | Medium | Sustainable pace, buffer time |
| Competitor launches | Medium | Medium | Focus on differentiation |
| Cloud cost overrun | Medium | Medium | Budget alerts, auto-scaling limits |

---

## Appendices

### Appendix A: Technology Decisions

| Decision | Choice | Alternatives Considered | Rationale |
|----------|--------|-------------------------|-----------|
| Spatial indexing | H3 (Uber) | S2 (Google), Geohash | Hexagonal cells, efficient neighbors |
| Database | PostgreSQL + PostGIS | MongoDB, CockroachDB | Mature, PostGIS, JSONB |
| Cache | Redis | Memcached, KeyDB | Pub/sub, persistence |
| API framework | Fastify | Express, Hono | Performance, TypeScript |
| AR Client | Unity + AR Foundation | Unreal, React Native | AR quality, 3D |
| Fingerprint ML | On-device | Cloud inference | Privacy, latency |

### Appendix B: Glossary

| Term | Definition |
|------|------------|
| H3 Cell | Hexagonal spatial index cell (~174m at resolution 9) |
| Place Fingerprint | Compact vector representing environmental features |
| Synthling | Procedurally generated creature from fingerprint |
| Influence | Currency representing crew control of territory |
| Turf | Territory/cell controlled by a crew |
| Raid | Async attack on enemy-controlled cell |
| Outpost | Player-deployed structure generating influence |
| Crew | Player team/guild |
| District | Collection of H3 cells forming a neighborhood |

### Appendix C: References

- [H3 Documentation](https://h3geo.org/)
- [AR Foundation Manual](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@5.1/manual/)
- [PostGIS Reference](https://postgis.net/documentation/)
- [Fastify Guides](https://fastify.dev/docs/latest/)

---

*Document Version: 2.0 | Last Updated: 2026-01-29*
