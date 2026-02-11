[![ORGAN-III: Ergon](https://img.shields.io/badge/ORGAN--III-Ergon-1b5e20?style=flat-square)](https://github.com/organvm-iii-ergon)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.6-3178c6?style=flat-square&logo=typescript&logoColor=white)](https://www.typescriptlang.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow?style=flat-square)](LICENSE)
[![CI](https://img.shields.io/github/actions/workflow/status/organvm-iii-ergon/my-block-warfare/ci.yml?branch=main&style=flat-square&label=CI)](https://github.com/organvm-iii-ergon/my-block-warfare/actions)
[![Node.js](https://img.shields.io/badge/Node.js-≥20-339933?style=flat-square&logo=node.js&logoColor=white)](https://nodejs.org/)

# TurfSynth AR

[![CI](https://github.com/organvm-iii-ergon/my-block-warfare/actions/workflows/ci.yml/badge.svg)](https://github.com/organvm-iii-ergon/my-block-warfare/actions/workflows/ci.yml)
[![Coverage](https://img.shields.io/badge/coverage-pending-lightgrey)](https://github.com/organvm-iii-ergon/my-block-warfare)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](https://github.com/organvm-iii-ergon/my-block-warfare/blob/main/LICENSE)
[![Organ III](https://img.shields.io/badge/Organ-III%20Ergon-F59E0B)](https://github.com/organvm-iii-ergon)
[![Status](https://img.shields.io/badge/status-active-brightgreen)](https://github.com/organvm-iii-ergon/my-block-warfare)
[![TypeScript](https://img.shields.io/badge/lang-TypeScript-informational)](https://github.com/organvm-iii-ergon/my-block-warfare)


**A location-based augmented reality game where your real-world neighborhood procedurally generates the game around you.**

TurfSynth AR merges open-world turf dynamics with creature collection and environmental synthesis. Instead of overlaying static content onto the real world the way existing location-based games do, TurfSynth extracts compact "Place Fingerprints" from camera, microphone, and sensor data, then uses those fingerprints to procedurally generate creatures, soundscapes, visuals, and mission flavor that are unique to every block. The core pitch: *what if you could take over the turf in your own neighborhood, and the world you are standing in literally builds the game around you?*

This repository contains the authoritative backend implementation (Fastify + TypeScript), the Unity AR client scaffolding, two Gemini-powered prototype applications (3D maps and spatial understanding), and the full specification-driven feature library that governs ongoing development.

---

## Table of Contents

- [Product Overview](#product-overview)
- [Technical Architecture](#technical-architecture)
- [Installation and Quick Start](#installation-and-quick-start)
- [Game Mechanics and Features](#game-mechanics-and-features)
- [API Reference](#api-reference)
- [Project Structure](#project-structure)
- [Key Technical Decisions](#key-technical-decisions)
- [Privacy and Safety Design](#privacy-and-safety-design)
- [Specification-Driven Development](#specification-driven-development)
- [Development Commands](#development-commands)
- [Testing](#testing)
- [CI/CD Pipeline](#cicd-pipeline)
- [Scaling Estimates](#scaling-estimates)
- [Cross-Organ Context](#cross-organ-context)
- [Related Work](#related-work)
- [Roadmap](#roadmap)
- [Contributing](#contributing)
- [License](#license)
- [Author](#author)

---

## Product Overview

### The Problem

Location-based games today overlay static, artist-authored content on top of the real world. Every player who visits the same spot sees the same gym, the same spawn, the same assets. The world itself contributes nothing to the experience. Environmental context -- color, sound, geometry, motion, time of day -- is completely ignored.

### The Approach

TurfSynth AR introduces **environmental synthesis**: the player's actual surroundings become the creative engine. On-device pipelines extract a compact "Place Fingerprint" (a privacy-preserving feature vector under 400 bytes) from camera frames, ambient audio, motion sensors, and GPS locality. That fingerprint drives procedural generation of collectible creatures ("Synthlings"), ambient soundscapes, visual themes, and mission parameters. No raw camera or audio data ever leaves the device.

The game layer on top of this synthesis engine is a **turf-control system** where neighborhoods are divided into H3 hexagonal cells grouped into districts. Players form crews, earn influence through exploration and combat, deploy outposts, and execute asynchronous raids to contest territory. Influence decays over time, so holding turf requires continuous play rather than a one-time capture.

### The Outcome

A location-based game where every block sounds and looks like itself, every player's creature collection reflects the places they have actually visited, and neighborhood pride becomes a literal game mechanic. The result is a system where the real world and the game world are coupled at a level of specificity that no existing title achieves.

### Player Fantasy

You are a "Mapper" who samples reality to produce a personalized, stylized underworld layer: graffiti sigils, neon signage, ambient beats, faction banners, procedural creatures, and mission set dressing -- all generated from your surroundings without requiring artists to hand-author every block of every city.

---

## Technical Architecture

### System Topology

```
┌─────────────────────────────────────────────────────────────────┐
│                        Unity Client                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │ AR Session   │  │  Fingerprint │  │    Turf      │          │
│  │ Manager      │  │   Capture    │  │   Display    │          │
│  └──────────────┘  └──────────────┘  └──────────────┘          │
└───────────────────────────┬─────────────────────────────────────┘
                            │ REST API (shared/api-types.ts)
┌───────────────────────────┴─────────────────────────────────────┐
│                     Node.js Backend (Fastify 5)                 │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │
│  │  Geofencing  │  │  Fingerprint │  │     Turf     │          │
│  │   Service    │  │   Service    │  │   Service    │          │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │
│         │                 │                  │                  │
│  ┌──────┴─────────────────┴──────────────────┴───────┐         │
│  │          PostgreSQL 16 + PostGIS 3.4              │         │
│  │                    Redis 7                         │         │
│  └────────────────────────────────────────────────────┘         │
└─────────────────────────────────────────────────────────────────┘
```

### Backend Services

The backend is structured as three domain services, each with its own API route prefix, database tables, and specification document.

| Service | Route Prefix | Purpose | Key Modules |
|---------|-------------|---------|-------------|
| **Geofencing** | `/api/v1/location` | GPS validation, exclusion zones (schools, hospitals, government buildings), spoof detection, speed lockouts | `zone-checker`, `speed-validator`, `spoof-detector`, `h3-cache`, `zone-sync` |
| **Fingerprint** | `/api/v1/fingerprint` | Receive, validate, store, and query Place Fingerprints; award influence for submissions; calculate fingerprint similarity | `capture-manager`, `color-extractor`, `visual-pipeline`, `audio-pipeline`, `assembler`, `validation-gate` |
| **Turf** | `/api/v1/turf` | Territory control: influence scoring with time-decay, outpost deployment and module management, raid initiation and resolution, district leaderboards, crew management, contract generation | `influence-manager`, `outpost-manager`, `raid-engine` |

All three services share a PostgreSQL connection pool and a Redis instance. The server entry point (`src/server.ts`) registers Fastify plugins for CORS and rate limiting, mounts the three route prefixes, and exposes `/health` and `/ready` endpoints suitable for Kubernetes probes.

### Unity Client

The Unity project (`unity/`) targets Unity 2022.3 LTS with AR Foundation for cross-platform ARKit/ARCore support:

| Component | Purpose | Path |
|-----------|---------|------|
| **ARSessionManager** | AR Foundation setup, camera and sensor access | `unity/Assets/Scripts/AR/` |
| **FingerprintCapture** | On-device environmental feature extraction | `unity/Assets/Scripts/Fingerprint/` |
| **ApiClient** | REST client for backend communication | `unity/Assets/Scripts/Networking/` |
| **LocationService** | GPS management and location updates | `unity/Assets/Scripts/Networking/` |

C# API types can be generated from the shared TypeScript contract:

```bash
npx quicktype -s typescript -o unity/Assets/Scripts/Generated/ApiTypes.cs shared/api-types.ts
```

### Prototype Applications

Two Gemini-powered web prototypes provide proof-of-concept spatial tooling:

- **mcp-maps-3d** -- A 3D map tool using Model Context Protocol (MCP), LitElement web components, and Google Maps 3D Tiles. A Gemini 2.5 Flash model translates natural language queries into map actions via an in-memory MCP transport.
- **spatial-understanding** -- A React 19 app with Jotai state management that performs real-time spatial detection (2D/3D bounding boxes, segmentation masks, point detection) via Gemini vision. Supports both static images and live screenshare.

Both use Vite + TypeScript and require a `GEMINI_API_KEY` in `.env.local`.

### Database Schema

Three SQL migrations define the persistence layer:

1. **001 -- Geofencing** (`src/db/migrations/001_geofencing_schema.sql`): Exclusion zones with PostGIS geometry, H3 cell-to-zone cache, partitioned location validation audit log (monthly partitions with 6-month retention), speed lockout records, cumulative spoof scores, and helper functions (`check_exclusion_zone`, `get_cached_zones`, `update_h3_cache`).

2. **002 -- Fingerprints** (`src/db/migrations/002_fingerprint_schema.sql`): Fingerprint storage with JSONB columns for palette, geometry, motion, audio, and locality descriptors. User and area queries.

3. **003 -- Turf Mechanics** (`src/db/migrations/003_turf_mechanics_schema.sql`): Districts, outposts (one per cell, up to 4 module types), raids with JSONB results, contracts (5 types: capture, survey, patrol, raid, defend), spawn configuration per cell, and stored functions for outpost ticks, district control calculation, raid resolution, and daily contract generation.

---

## Installation and Quick Start

### Prerequisites

| Dependency | Minimum Version | Purpose |
|-----------|----------------|---------|
| Node.js | 20.0 | Runtime |
| PostgreSQL | 16 | Primary data store |
| PostGIS | 3.4 | Geospatial queries |
| Redis | 7 | Caching, rate limiting, speed lockout state |
| Unity | 2022.3 LTS | AR client (optional, for client development only) |

### Backend Setup

```bash
# Clone the repository
git clone https://github.com/organvm-iii-ergon/my-block-warfare.git
cd my-block-warfare

# Install dependencies
npm install

# Configure environment
cp .env.example .env.local
# Edit .env.local with your database credentials and Redis URL
```

### Environment Variables

```bash
# Database
DATABASE_URL=postgresql://localhost:5432/turfsynth
DATABASE_POOL_SIZE=20

# Redis
REDIS_URL=redis://localhost:6379
REDIS_CLUSTER_MODE=false

# Server
PORT=3000
HOST=0.0.0.0
NODE_ENV=development

# H3 Configuration
H3_RESOLUTION_STORAGE=7     # ~5km cells for storage
H3_RESOLUTION_GAMEPLAY=9    # ~175m cells for gameplay

# Rate Limiting
RATE_LIMIT_MAX=100
RATE_LIMIT_WINDOW_MS=60000

# Geofencing
SPEED_LOCKOUT_KMH=15
SPEED_WINDOW_SECONDS=30
SPOOF_VELOCITY_MAX_KMH=500

# Zone Data Sources
OSM_API_URL=https://overpass-api.de/api/interpreter
SAFEGRAPH_API_KEY=           # Optional

# Logging
LOG_LEVEL=info
```

### Database Initialization

```bash
# Ensure PostGIS extension is available
psql -d turfsynth -c "CREATE EXTENSION IF NOT EXISTS postgis;"
psql -d turfsynth -c "CREATE EXTENSION IF NOT EXISTS pg_trgm;"

# Run migrations
npm run db:migrate

# Seed development data (optional)
npm run db:seed
```

### Start Development Server

```bash
npm run dev
# Server starts at http://localhost:3000
# Health check: http://localhost:3000/health
# Readiness probe: http://localhost:3000/ready
```

### Prototype Apps

```bash
# 3D Maps (LitElement + MCP + Google Maps)
cd mcp-maps-3d && npm install && npm run dev

# Spatial Understanding (React + Gemini Vision)
cd spatial-understanding && npm install && npm run dev
```

Both require `GEMINI_API_KEY` set in their respective `.env.local` files.

---

## Game Mechanics and Features

### Core Loop

```
┌─────────────────────────────────────────────────────────────────┐
│   Player walks into cell                                        │
│        │                                                        │
│        ▼                                                        │
│   Location validated (Safety Geofencing) ─── Fails? ──▶ Blocked │
│        │                                                        │
│        ▼ Passes                                                 │
│   Extract fingerprint (Place Fingerprint)                       │
│        │                                                        │
│        ├──▶ Submit fingerprint ──▶ +10 Influence                │
│        │                                                        │
│        └──▶ Encounter Synthling (procedural creature)           │
│                  │                                              │
│                  ▼                                              │
│             Capture ──▶ +5 Influence + Add to Collection        │
│                                                                 │
│   [Passive] Influence decays hourly (48h half-life)             │
│   [Async]   Raid rival outposts                                 │
│   [Goal]    Control districts, evolve Synthlings                │
└─────────────────────────────────────────────────────────────────┘
```

### Place Fingerprints

The Place Fingerprint is the core technical innovation. It is a compact, privacy-preserving feature vector (under 400 bytes serialized) extracted entirely on-device from camera frames, ambient audio, motion sensors, and GPS locality. The fingerprint contains five descriptors:

| Descriptor | Size | Source | Contents |
|-----------|------|--------|----------|
| **Visual Palette** | ~80 bytes | Camera frame | 5-7 dominant colors with weights, overall brightness, saturation |
| **Geometry** | ~64 bytes | Scene understanding | Edge orientation histogram (8 buckets), surface type distribution (sky, vegetation, building, ground, water, road), vertical bias, complexity score |
| **Motion** | ~16 bytes | Accelerometer | Motion level, dominant direction, periodicity |
| **Audio** | ~48 bytes | Microphone | Spectral centroid, harmonic ratio, rhythm density, loudness, dominant frequency band |
| **Locality** | ~32 bytes | GPS + Clock | H3 cell index (resolution 7), time-of-day bucket, day type (weekday/weekend), season hint |

Raw capture inputs (pixel data, audio samples, precise coordinates) never leave the device. Only the extracted fingerprint vector is transmitted to the server.

### Territory System

The territory layer uses Uber's H3 hexagonal grid system:

- **Cells**: Resolution 9 hexagons (~175 meter diameter) are the atomic unit of territory control. Each cell tracks per-crew influence scores.
- **Districts**: Named aggregations of cells representing real-world neighborhoods. Districts have a controlling crew (the crew with the highest aggregate influence) and a control percentage.
- **Influence**: A numeric score per crew per cell. Influence is earned through fingerprint submissions (+10), Synthling captures (+5), contract completions (variable), outpost passive generation (hourly ticks), and successful raids (variable). Influence decays with a 48-hour half-life, processed every 15 minutes.
- **Control**: A cell is controlled by the crew with the highest influence score. A district is controlled by the crew that controls the most cells within it.

### Outposts

Players deploy outposts (one per cell) to project persistent influence. Each outpost supports up to four module types:

| Module | Effect | Max Level |
|--------|--------|-----------|
| **Scanner** | Increases Synthling spawn rate in the cell | 3 |
| **Amplifier** | Multiplies passive influence generation (0.25x per level) | 3 |
| **Shield** | Reduces incoming raid damage (5 defense per level) | 3 |
| **Beacon** | Attracts crew members (social signal) | 3 |

Outposts have health (0-100) and level (1-5). They generate influence passively via hourly ticks processed by a server-side stored function.

### Raid System

Raids are the primary conflict mechanic. They are cell-targeted (not player-targeted) and currently resolve immediately (asynchronous job-queue resolution planned for production):

1. **Initiation**: Attacker specifies a target cell and commits attack power (minimum 10). A 30-minute cooldown prevents raid spam.
2. **Defense Calculation**: Base defense equals the controlling crew's influence in the cell, plus outpost level bonus (10 per level), plus shield module bonus (5 per level), multiplied by a 1.2x defender advantage.
3. **Resolution**: If attack power exceeds defense power, the raid succeeds. Twenty percent of the power differential transfers as influence. Outposts take 20 base damage on successful raids.
4. **Rewards**: Attackers earn influence on success. Defenders earn a defense bonus (25 influence) on successful defense.

### Synthlings

Synthlings are procedurally generated creatures whose attributes derive from Place Fingerprints:

| Attribute | Source | Types |
|-----------|--------|-------|
| **Palette** | Visual fingerprint colors | RGB arrays mapped to skin/energy tones |
| **Geometry** | Edge histogram, surface distribution | Crystalline, Organic, Geometric, Fluid, Fractal |
| **Motion** | Motion descriptor | Float, Pulse, Spiral, Bounce, Swarm |
| **Sound** | Audio descriptor | Ambient, Melodic, Percussive, Harmonic, Noise |
| **Rarity** | Composite fingerprint rarity score | Common, Uncommon, Rare, Epic, Legendary |

Evolution requires visiting distinct micro-biomes (three different environment signatures within a district), coupling real-world exploration to creature progression.

### Contracts

Context-aware short missions available per district:

| Type | Objective | Typical Reward |
|------|-----------|----------------|
| **Capture** | Capture specific Synthlings | 50 influence |
| **Survey** | Submit fingerprints in target area | 50 influence |
| **Patrol** | Visit multiple cells in sequence | 50 influence |
| **Raid** | Execute a successful raid | Variable |
| **Defend** | Successfully defend against a raid | Variable |

Contracts expire after 24 hours and are generated daily via the `generate_district_contracts` stored function.

### Crews

Crews are 6-30 player factions identified by name, 3-5 character tag, and hex color. Crew-level progression unlocks district benefits (faster influence gain, better defense, cosmetic district themes). The "Heat" mechanic tracks rival attention: high heat yields higher rewards but makes holding turf harder.

---

## API Reference

### Location Validation

```http
POST /api/v1/location/validate
Content-Type: application/json

{
  "userId": "uuid",
  "sessionId": "uuid",
  "latitude": 37.7749,
  "longitude": -122.4194,
  "accuracy": 10,
  "timestamp": "2026-01-15T10:30:00Z",
  "platform": "ios"
}
```

Response includes validation result code (`valid`, `blocked_exclusion_zone`, `blocked_speed_lockout`, `blocked_spoof_detected`, `blocked_rate_limit`, `error`), the resolved H3 cell index, and detailed check results for zone, speed, and spoof analysis.

### Fingerprint Submission

```http
POST /api/v1/fingerprint/submit
Content-Type: application/json

{
  "h3Cell": "89283082813ffff",
  "colorPalette": [{"r": 120, "g": 85, "b": 200}],
  "dominantColor": {"r": 120, "g": 85, "b": 200},
  "brightness": 0.65,
  "colorTemperature": 5600,
  "planeCount": 4,
  "audioFeatures": {
    "ambientLevel": 0.4,
    "frequency": 440,
    "complexity": 0.3
  },
  "motionSignature": {
    "rotationX": 0.1,
    "rotationY": 0.02,
    "rotationZ": -0.05,
    "accelerationMagnitude": 1.02
  }
}
```

### Turf Endpoints

```http
GET  /api/v1/turf/cell/:h3Index          # Get cell territory status
GET  /api/v1/turf/district/:districtId/leaderboard  # District rankings
POST /api/v1/turf/raid                    # Initiate a raid
```

See `shared/api-types.ts` for complete TypeScript type definitions covering all request/response shapes, including outpost deployment, module installation, Synthling spawning and capture, and error responses.

---

## Project Structure

```
my-block-warfare/
├── src/                              # Backend source (TypeScript)
│   ├── server.ts                     # Fastify entry point
│   ├── config/                       # Environment configuration
│   ├── api/v1/                       # REST route handlers
│   │   ├── location.ts               # Geofencing endpoints
│   │   ├── fingerprint.ts            # Fingerprint endpoints
│   │   └── turf.ts                   # Territory endpoints
│   ├── services/                     # Domain services
│   │   ├── geofencing/               # Safety validation pipeline
│   │   │   ├── index.ts              # GeofencingService orchestrator
│   │   │   ├── zone-checker.ts       # Exclusion zone lookup
│   │   │   ├── speed-validator.ts    # Vehicle speed detection
│   │   │   ├── spoof-detector.ts     # GPS manipulation analysis
│   │   │   ├── h3-cache.ts           # H3-to-zone cache layer
│   │   │   └── zone-sync.ts          # External zone data sync
│   │   ├── fingerprint/              # Place Fingerprint processing
│   │   │   ├── index.ts              # FingerprintService
│   │   │   ├── capture-manager.ts    # Capture coordination
│   │   │   ├── color-extractor.ts    # Palette extraction
│   │   │   ├── visual-pipeline.ts    # Visual feature pipeline
│   │   │   ├── audio-pipeline.ts     # Audio feature pipeline
│   │   │   ├── assembler.ts          # Fingerprint assembly + similarity
│   │   │   └── validation-gate.ts    # Submission validation
│   │   └── turf/                     # Territory mechanics
│   │       ├── index.ts              # TurfService
│   │       ├── influence-manager.ts  # Influence scoring + decay
│   │       ├── outpost-manager.ts    # Outpost CRUD + ticks
│   │       └── raid-engine.ts        # Raid initiation + resolution
│   ├── db/                           # Database layer
│   │   ├── connection.ts             # PostgreSQL pool
│   │   ├── redis.ts                  # Redis client
│   │   └── migrations/               # SQL schema migrations
│   │       ├── 001_geofencing_schema.sql
│   │       ├── 002_fingerprint_schema.sql
│   │       └── 003_turf_mechanics_schema.sql
│   ├── types/                        # TypeScript type definitions
│   │   ├── geofencing.ts
│   │   ├── fingerprint.ts
│   │   ├── turf.ts
│   │   ├── synthling.ts
│   │   └── index.ts
│   ├── utils/                        # Shared utilities
│   │   └── logger.ts                 # Pino logger factory
│   └── __tests__/                    # Test suite
│       ├── setup.ts
│       └── unit/
│           ├── influence-manager.test.ts
│           ├── raid-engine.test.ts
│           └── zone-checker.test.ts
├── shared/                           # Cross-platform API contract
│   └── api-types.ts                  # TypeScript types (source of truth)
├── unity/                            # Unity AR client
│   ├── Assets/Scripts/
│   │   ├── AR/ARSessionManager.cs
│   │   ├── Fingerprint/FingerprintCapture.cs
│   │   └── Networking/
│   │       ├── ApiClient.cs
│   │       └── LocationService.cs
│   └── README.md
├── mcp-maps-3d/                      # Gemini + MCP 3D map prototype
├── spatial-understanding/            # Gemini vision spatial detector
├── specs/                            # Feature specifications (speckit)
│   ├── safety-geofencing/
│   ├── place-fingerprint/
│   ├── synthling-generation/
│   └── turf-mechanics/
├── docs/
│   ├── evaluation/                   # Project evaluation reports
│   ├── roadmap/                      # Execution roadmap
│   └── runbook/                      # Operations runbook
├── memory/
│   └── constitution.md               # Immutable architectural principles
├── TurfSynth-AR-Concept.md           # Master product specification
├── CLAUDE.md                         # AI assistant context
├── .github/workflows/ci.yml          # CI pipeline
├── .env.example                      # Environment template
├── package.json                      # Node.js manifest
├── tsconfig.json                     # TypeScript configuration
├── vitest.config.ts                  # Test runner configuration
└── eslint.config.js                  # ESLint flat config
```

---

## Key Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| **Spatial indexing** | H3 (Uber) | Consistent hexagonal cells with no shape distortion at edges, efficient `gridDisk` neighbor queries, deterministic cell IDs from coordinates |
| **Database** | PostgreSQL 16 + PostGIS 3.4 | Mature geospatial operators, JSONB for flexible per-cell influence maps, table partitioning for audit log retention |
| **Cache/real-time** | Redis 7 | Pub/sub for live updates, sorted sets for leaderboards, key expiry for speed lockouts, sub-millisecond lookups for hot-path geofence data |
| **API framework** | Fastify 5 | Schema-validated routes, plugin architecture, structured JSON logging via Pino, built-in rate limiting |
| **AR client** | Unity 2022.3 LTS + AR Foundation | Cross-platform ARKit/ARCore, C# type generation from shared TypeScript contract via quicktype |
| **Schema validation** | Zod | Runtime request validation with TypeScript type inference, composable schemas |
| **Test runner** | Vitest | Native ESM support, fast watch mode, Vite-aligned configuration |
| **Audit log partitioning** | Monthly PostgreSQL partitions | Efficient retention management (6-month policy), partition-level DROP instead of row-level DELETE |

---

## Privacy and Safety Design

Privacy and safety are constitutional constraints, not afterthoughts. The project constitution (`memory/constitution.md`) mandates Safety-Mandatory and Privacy-First as the two highest-priority gates.

### Privacy by Design

- **Place Fingerprints only**: The server never receives raw camera frames, audio samples, or precise GPS coordinates. Only compact feature vectors (under 400 bytes) and H3 cell indices are transmitted.
- **Cell-level location**: The audit log stores H3 cell indices, not latitude/longitude pairs. Location precision is intentionally degraded for privacy.
- **Hashed device IDs**: Device identifiers are hashed before inclusion in fingerprints.
- **No live tracking**: All multiplayer interactions (raids, territory contests) are cell-based and asynchronous. No player can see another player's real-time location.

### Safety Geofencing

The geofencing service performs three checks in order of computational cost (target: <100ms p95 latency):

1. **Zone check (~5ms with cache hit)**: Exclusion zones around schools, hospitals, government buildings, and private residences sourced from OpenStreetMap and optional SafeGraph data. Pre-computed H3-to-zone cache avoids PostGIS queries on the hot path.
2. **Speed validation (~2ms Redis lookup)**: Detects vehicle speeds (>15 km/h) and applies a cooldown lockout. Prevents gameplay while driving.
3. **Spoof detection (~10ms history analysis)**: Analyzes GPS history for impossible velocities, coordinate jitter, and implausible movement patterns. Cumulative spoof scores track repeat offenders.

### Content Safety

- All user-named tags and outposts pass through content filters.
- Freeform text is limited; preset naming and stylized emblems are preferred.
- Real-world behavior prompts: the game never instructs trespass; "public space only" UX reminders appear when gameplay intensifies.

---

## Specification-Driven Development

This project uses the **speckit** methodology: specifications are truth, code is generated output. Feature specifications live in `specs/` and follow a three-artifact structure:

| Artifact | Purpose | Example |
|----------|---------|---------|
| `spec.md` | Requirements, success criteria, constitution gates | `specs/safety-geofencing/spec.md` |
| `plan.md` | Implementation approach, architecture decisions | `specs/place-fingerprint/plan.md` |
| `tasks.md` | Executable work items with estimates | `specs/turf-mechanics/tasks.md` |

### MVP Feature Specifications

| Feature | Status | Engineering Estimate | Constitution Gates |
|---------|--------|---------------------|--------------------|
| Safety Geofencing | Specified + Implemented | 68h | Safety-Mandatory, Privacy-First |
| Place Fingerprint | Specified + Implemented | 92h | Privacy-First, Mobile-First, Environmental Authenticity |
| Synthling Generation | Specified | 192h (72h eng + 120h art) | Environmental Authenticity, Progressive Disclosure |
| Turf Mechanics | Specified + Implemented | 88h | Safety-Mandatory, Privacy-First |

### Feature Dependency Graph

```
Safety Geofencing ──────┐
       │                │ (constitution priority #1)
       ▼                │
Place Fingerprint ◄─────┘
       │
       ├──────────────────┐
       ▼                  ▼
Synthling Generation    Turf Mechanics
(hard dependency)       (soft dependency)
```

---

## Development Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start development server with hot reload (tsx watch) |
| `npm run build` | Compile TypeScript to `dist/` |
| `npm run start` | Run compiled production build |
| `npm run test` | Run test suite (Vitest) |
| `npm run test:coverage` | Run tests with coverage report |
| `npm run test:integration` | Run integration tests (requires running database) |
| `npm run lint` | Check code style (ESLint flat config) |
| `npm run lint:fix` | Auto-fix lint errors |
| `npm run format` | Format source with Prettier |
| `npm run typecheck` | Run TypeScript type checker (`tsc --noEmit`) |
| `npm run db:migrate` | Run database migrations |
| `npm run db:seed` | Seed development data |

---

## Testing

The test suite uses Vitest with 59 unit tests covering the three core services:

```bash
# Run all tests
npm run test

# Run with coverage
npm run test:coverage

# Run a specific test file
npm run test -- src/__tests__/unit/influence-manager.test.ts
```

### Test Files

| Test | Covers | Key Scenarios |
|------|--------|---------------|
| `influence-manager.test.ts` | Influence scoring, decay processing, cell control transitions | Fingerprint influence award, decay half-life, crew influence aggregation |
| `raid-engine.test.ts` | Raid lifecycle, defense calculation, reward distribution | Successful raid, failed raid, cooldown enforcement, outpost damage |
| `zone-checker.test.ts` | Exclusion zone lookup, H3 cache behavior | Zone block, zone allowance, cache hit/miss, expired zone handling |

---

## CI/CD Pipeline

The GitHub Actions workflow (`.github/workflows/ci.yml`) runs on pushes and pull requests to `main` and `develop`:

1. **Lint and Type Check** -- ESLint and `tsc --noEmit` on Ubuntu with Node.js 22
2. **Test** -- Full test suite with coverage against PostgreSQL 16 + PostGIS 3.4 and Redis 7 service containers. Coverage reports upload to Codecov.
3. **Build** -- TypeScript compilation with artifact upload (7-day retention)

Concurrency groups cancel in-progress runs for the same branch.

---

## Scaling Estimates

The concept document includes detailed scaling cost models using an Effort Unit (EU) framework where 1 EU = 1 person-week of productive work.

| Stage | Target Scale | Team Size | Annual People Cost | Monthly Infra |
|-------|-------------|-----------|-------------------|---------------|
| S0: MVP | Private alpha, <=10k users | 8-10 FTE | $2.0M-$3.4M | $5k-$25k |
| S1: City Pilot | 1 metro, 50k-200k MAU | 12-18 FTE | $3.5M-$6.1M | $25k-$120k |
| S2: Multi-City | 5-10 metros, 0.5M-2M MAU | 25-40 FTE | $7.8M-$13.5M | $120k-$450k |
| S3: National | US-wide, 2M-10M MAU | 50-80 FTE | $15.6M-$27.0M | $450k-$1.8M |
| S4: Global | Multi-region, 10M-50M MAU | 100-160 FTE | $31.2M-$54.1M | $1.8M-$6.0M |

Infrastructure cost bracket: $0.03-$0.20 per DAU per month (lower end for cached/async architecture, upper end for real-time sync with heavy anti-cheat).

---

## Cross-Organ Context

This repository lives within **ORGAN-III (Ergon)**, the commerce organ of the [organvm eight-organ system](https://github.com/organvm-iii-ergon). ORGAN-III houses SaaS, B2B, and B2C product repositories -- the applied, revenue-capable outputs of the broader creative-institutional project.

TurfSynth AR draws on work from adjacent organs:

| Organ | Relationship |
|-------|-------------|
| **ORGAN-I (Theoria)** | Theoretical foundations: the [recursive-engine](https://github.com/organvm-i-theoria/recursive-engine--generative-entity) provides the recursive self-reference patterns that inform the fingerprint similarity calculus and the influence decay model |
| **ORGAN-II (Poiesis)** | Creative precedents: the [metasystem-master](https://github.com/organvm-ii-poiesis/metasystem-master) project's generative art pipelines inform the Synthling visual synthesis approach |
| **ORGAN-IV (Taxis)** | Orchestration: the [agentic-titan](https://github.com/organvm-iv-taxis/agentic-titan) agent framework may coordinate future CI/CD and deployment automation for TurfSynth |
| **ORGAN-V (Logos)** | Public process: development decisions and design rationale will be documented as essays in the [public-process](https://github.com/organvm-v-logos/public-process) repository |

The dependency direction follows the organ system invariant: I -> II -> III only. ORGAN-III consumes theory and art but never introduces back-edges.

---

## Related Work

- **Pokemon Go** (Niantic) -- Pioneered mass-market location-based AR gaming. TurfSynth differentiates through environmental synthesis (the world generates content) rather than static POI-based spawning.
- **Ingress** (Niantic) -- Established territory-control mechanics in location gaming. TurfSynth's influence-decay model and cell-based raids are direct descendants, adapted for crew-scale play.
- **GTA Online** / **Schedule I** -- The turf-control fantasy and crew dynamics draw from open-world crime sandbox design, translated to safe, cell-based, non-confrontational AR mechanics.
- **H3** (Uber) -- The hexagonal hierarchical geospatial indexing system underpinning all spatial operations.
- **AR Foundation** (Unity) -- Cross-platform AR abstraction layer enabling ARKit/ARCore support from a single codebase.

---

## Roadmap

### Completed

- [x] Backend architecture (Fastify + TypeScript + PostgreSQL + Redis)
- [x] Safety Geofencing service (zone checker, speed validator, spoof detector)
- [x] Place Fingerprint service (submission, validation, storage, similarity)
- [x] Turf Mechanics service (influence, outposts, raids, districts, crews)
- [x] Shared API type contract (`shared/api-types.ts`)
- [x] Unit tests for core services (59 tests)
- [x] CI/CD pipeline with GitHub Actions (lint, typecheck, test, build)
- [x] Unity project structure with AR Foundation scaffolding
- [x] Gemini-powered spatial prototypes (mcp-maps-3d, spatial-understanding)
- [x] Feature specifications for all 4 MVP features (speckit methodology)
- [x] Database schema with 3 migrations

### In Progress

- [ ] Integration tests for API endpoints
- [ ] Unity API client implementation
- [ ] AR session management and fingerprint capture on device
- [ ] Synthling generation service (engineering + art assets)

### Upcoming

- [ ] Real-time influence updates (WebSocket / Server-Sent Events)
- [ ] AR visualization layer (ARKit/ARCore rendering)
- [ ] Multiplayer sync for live showdowns
- [ ] Procedural audio synthesis engine
- [ ] Faction system with alliance mechanics
- [ ] Virtual economy and resource management
- [ ] Time-limited event system
- [ ] Alpha pilot deployment

---

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Write tests for new functionality
4. Ensure all tests pass (`npm run test`)
5. Run lint and typecheck (`npm run lint && npm run typecheck`)
6. Commit changes with a descriptive message (`git commit -m 'Add amazing feature'`)
7. Push to your branch (`git push origin feature/amazing-feature`)
8. Open a Pull Request

All contributions must comply with the project constitution (`memory/constitution.md`), particularly the Safety-Mandatory and Privacy-First gates.

---

## License

MIT

---

## Author

**@4444j99** -- [github.com/4444J99](https://github.com/4444J99)

Part of the [ORGAN-III: Ergon](https://github.com/organvm-iii-ergon) commerce organ within the [organvm eight-organ system](https://github.com/meta-organvm).
