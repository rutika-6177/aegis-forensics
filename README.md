# AegisForensics

**Academic Digital Image Forensics and Duplicate Detection**

A planned web-based academic project for analyzing image collections, detecting exact duplicates, finding visually similar images, and exploring relationships through an interactive graph.

## Project Goals

- Detect exact duplicate images.
- Identify potential visual matches using perceptual similarity.
- Explore image relationships through an interactive graph.
- Support graph traversal using BFS and DFS.
- Maintain an image registry and export analysis results.
- Keep image files local while optionally synchronizing metadata through Supabase.

## Planned Technology

- **Frontend:** React + TypeScript
- **Backend:** C++17
- **Graph visualization:** Cytoscape.js
- **Storage:** Local-first, with optional Supabase metadata synchronization.

The frontend framework is selected. The build tool and setup commands will be confirmed during the environment setup phase.

## Documentation

| Document | Purpose |
|---|---|
| [PRD.md](PRD.md) | Product requirements and scope |
| [Architecture.md](Architecture.md) | System architecture and confirmed frontend decision |
| [Rules.md](Rules.md) | Project rules and constraints |
| [Phases.md](Phases.md) | Development roadmap |
| [Design.md](Design.md) | UI and visual design direction |

## Project Status

**Current phase:** Planning and environment setup.

The project specifications are committed to GitHub. Application implementation has not yet begun; listed features are goals, not claims of existing functionality.

## Academic Purpose

This project supports academic learning and experimentation in digital image forensics, duplicate detection, graph algorithms, and web application development.