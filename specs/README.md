# TurfSynth AR Specifications

This directory contains feature specifications for TurfSynth AR, following the speckit methodology.

## Specification Structure

Each feature spec follows this structure:
```
specs/
├── README.md                    # This index
├── <feature>/
│   ├── spec.md                  # Feature specification
│   ├── plan.md                  # Implementation plan (generated)
│   └── tasks.md                 # Task breakdown (generated)
```

## Specification Workflow

1. **Specify**: Define feature requirements in `spec.md`
2. **Plan**: Generate implementation approach in `plan.md`
3. **Tasks**: Break down into executable work items in `tasks.md`

## Feature Index

| Feature | Status | Description |
|---------|--------|-------------|
| *None yet* | - | Run `/speckit.specify <feature>` to begin |

## Priority Features (from Concept Doc)

### MVP (Phase 1)
- [ ] `place-fingerprint` - Environmental synthesis from surroundings
- [ ] `turf-mechanics` - Territory claiming and influence
- [ ] `synthling-generation` - Procedural creature creation
- [ ] `safety-geofencing` - Location-based access controls

### Core (Phase 2)
- [ ] `ar-visualization` - ARKit/ARCore rendering layer
- [ ] `multiplayer-sync` - Real-time player interactions
- [ ] `audio-synthesis` - Procedural soundscapes

### Extended (Phase 3)
- [ ] `faction-system` - Team dynamics and alliances
- [ ] `economy` - Virtual resource management
- [ ] `events` - Time-limited gameplay modes

## Commands

```bash
/speckit.specify <feature>    # Create new feature specification
/speckit.plan <feature>       # Generate implementation plan
/speckit.tasks <feature>      # Generate task breakdown
```

## References

- [TurfSynth-AR-Concept.md](../TurfSynth-AR-Concept.md) - Master product specification
- [memory/constitution.md](../memory/constitution.md) - Architectural principles
