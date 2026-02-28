# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**TurfSynth AR** - A location-based AR game concept blending GTA/Schedule I turf dynamics with Pokémon Go collection mechanics. The core differentiator is **environmental synthesis**: the user's real-world surroundings generate visuals, animations, sounds, and creatures procedurally.

This repository contains:
- **Theoretical specifications** (`TurfSynth-AR-Concept.md`) - Product design, game mechanics, scaling estimates
- **Prototype apps** (from Google AI Studio) - Proof-of-concept spatial/map tools

## Repository Structure

```
my-block-warfare/
├── TurfSynth-AR-Concept.md      # Master product specification
├── specs/                        # Feature specifications (SDD approach)
├── mcp-maps-3d/                  # 3D map tool using Gemini + MCP
└── spatial-understanding/        # Spatial detection tool using Gemini
```

## Development Commands

Both prototype apps use Vite + TypeScript:

```bash
# mcp-maps-3d (LitElement + MCP + Google Maps)
cd mcp-maps-3d
npm install
npm run dev      # Start dev server
npm run build    # Production build

# spatial-understanding (React + Jotai + Tailwind)
cd spatial-understanding
npm install
npm run dev      # Start dev server
npm run build    # Production build
```

**Required**: Set `GEMINI_API_KEY` in each app's `.env.local` file.

## Architecture Patterns

### mcp-maps-3d
- **MCP (Model Context Protocol)** architecture: Server exposes map tools, client relays AI function calls
- LitElement web components (`MapApp`)
- In-memory transport linking MCP client/server
- Gemini 2.5 Flash for natural language → map actions

### spatial-understanding
- React 19 with Jotai for atomic state management
- Atoms defined in `atoms.tsx`, hooks in `hooks.tsx`
- Detection modes: 2D bounding boxes, 3D bounding boxes, segmentation masks, points
- Supports live screenshare and static image analysis

## Specification-Driven Development

This project uses the **speckit** methodology. Specifications live in `specs/` and drive implementation:

```bash
/speckit.specify <feature>    # Create feature specification
/speckit.plan <tech-context>  # Generate implementation plan
/speckit.tasks                # Generate executable task list
```

Key concept: Specifications are truth, code is generated output.

## Key Technical Decisions

- **Privacy-by-design**: Store/transmit only "Place Fingerprints" (low-dimensional vectors), never raw camera/audio
- **Geofencing**: Server-side exclusion of sensitive locations (schools, hospitals, private residences)
- **Anti-cheat**: GPS spoof detection, movement plausibility, influence submission validation
- **Synthlings**: Procedural creatures with attributes derived from environment scans (palette, geometry, motion, sound)

## Deployments

| Prototype | URL | Platform |
|-----------|-----|----------|
| `mcp-maps-3d` | https://turfsynth-mcp-maps.pages.dev | Cloudflare Pages |
| `spatial-understanding` | https://turfsynth-spatial.pages.dev | Cloudflare Pages |

<!-- ORGANVM:AUTO:START -->
## System Context (auto-generated — do not edit)

**Organ:** ORGAN-III (Commerce) | **Tier:** standard | **Status:** CANDIDATE
**Org:** `unknown` | **Repo:** `my-block-warfare`

### Edges
- **Produces** → `unknown`: unknown

### Siblings in Commerce
`classroom-rpg-aetheria`, `gamified-coach-interface`, `trade-perpetual-future`, `fetch-familiar-friends`, `sovereign-ecosystem--real-estate-luxury`, `public-record-data-scrapper`, `search-local--happy-hour`, `multi-camera--livestream--framework`, `universal-mail--automation`, `mirror-mirror`, `the-invisible-ledger`, `enterprise-plugin`, `virgil-training-overlay`, `tab-bookmark-manager`, `a-i-chat--exporter` ... and 11 more

### Governance
- Strictly unidirectional flow: I→II→III. No dependencies on Theory (I).

*Last synced: 2026-02-24T12:41:28Z*
<!-- ORGANVM:AUTO:END -->
