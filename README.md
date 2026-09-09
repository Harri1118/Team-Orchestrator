# Team-Orchestrator

An agentic project manager built on Claude Code that uses XP methodology to plan, scope, and ship software projects.

## What It Does

Takes a video, transcript, or project idea and produces:
- Structured project brief
- XP plan with user stories, architecture, and story map
- Prerequisites checklist (API keys, accounts, env vars)
- Linear tickets with full acceptance criteria
- Visual roadmap with mermaid diagrams (story map, Gantt, architecture)
- Implementation, QA, and validation pipeline

## Commands

| Command | What It Does |
|---------|-------------|
| `/vision <url>` | Ingest video/transcript, extract project brief |
| `/plan` | Generate XP plan with stories, architecture, story map |
| `/preflight` | Identify all prerequisites before work starts |
| `/ticket` | Create Linear tickets from user stories |
| `/roadmap` | Generate visual story map + release timeline |
| `/build <ticket>` | Implement a ticket with XP discipline |
| `/qa` | Adversarial code review (no changes, only findings) |
| `/validate` | Final pre-merge validation |

## Pipeline

```
/vision -> /plan -> /preflight -> /ticket -> /roadmap
                                     |
                              /build -> /qa -> fix loop -> /validate -> ship
```

## Setup

### Prerequisites

```bash
# Video transcript extraction
brew install yt-dlp

# Audio transcription (for uncaptioned videos)
pip install openai-whisper

# Linear integration
# 1. Get API key: Linear Settings > API > Personal API keys
# 2. Set in .mcp.json: LINEAR_API_KEY
```

### Install

```bash
git clone <this-repo>
cd Team-Orchestrator

# Configure Linear (edit .mcp.json with your API key)
# Then in Claude Code:
/mcp
```

## How It Works

### Phase 1: Project Discovery
```bash
# From a YouTube video
/vision https://youtube.com/watch?v=...

# From a local file
/vision /path/to/recording.mp4

# From a pasted transcript
/vision paste
```

### Phase 2: Planning
```bash
/plan          # Generates XP plan from the latest brief
/preflight     # Identifies what you need before coding
/ticket        # Creates Linear tickets
/roadmap       # Generates visual story map
```

### Phase 3: Implementation (per ticket)
```bash
/build TEAM-42    # Implement the ticket
/qa               # Get adversarial review
# Fix issues...
/qa               # Re-review
/validate         # Final check before merge
```

## XP Practices Enforced

- **Small iterations** (1-2 weeks)
- **User stories** as the unit of work
- **Test-first** development
- **Simple design** (YAGNI)
- **Continuous integration** gates
- **Pair programming** recommendations
- **Human gates** at every phase transition
- **Proof-of-work** at every stage
