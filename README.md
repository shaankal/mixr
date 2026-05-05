# MIXR
### Your playlist. DJ'd by AI.

---

## The Problem

Spotify's Mix feature auto-blends between songs, but it does a pretty bad job of it.

Transitions fire on a timer. They don't care if the beat is in the wrong place, if the keys clash, or if two songs that have nothing in common sonically just got jammed next to each other. The result sounds like a playlist — not a mix.

Real DJs don't work that way. They sequence tracks so transitions are naturally easy, then execute each handoff at exactly the right moment — beat-locked, harmonically smooth, with EQ dialed so nothing sounds like a mess.

---

## What Mixr Does

Mixr is the AI system that does what a DJ does, automatically.

You give it a playlist or album. It gives you back the same songs, reordered for maximum flow, with every transition executed cleanly. **Every song still plays in full — Mixr only touches the seams.**

### Step 1 — It reorders your playlist

Mixr analyzes every track for BPM, key, and energy. It solves for the sequence where every adjacent pair is as compatible as possible — harmonically, rhythmically, and emotionally. No more jarring key jumps. No more whiplash energy drops.

### Step 2 — It executes every transition

At the end of each song, Mixr figures out exactly how to hand off to the next one. It picks the overlap length, the volume curve, the EQ, and whether to nudge the tempo. Everything is beat-locked — transitions start and end on bar boundaries, never mid-bar.

The result sounds like a DJ mixed it. Because an AI did.

---

## How It Works

### 1. Audio analysis
Every track gets fingerprinted: BPM, beat grid, Camelot key, energy curve, and the exact timestamps where the intro ends and the outro begins. This runs once per track and is cached.

### 2. Compatibility scoring
Mixr scores every possible pair of tracks across four dimensions:

| Factor | Weight | Method |
|--------|--------|--------|
| Harmonic fit | 40% | Camelot wheel distance |
| BPM delta | 35% | Linear penalty, 0–15% range |
| Energy match | 15% | Cosine similarity at seam points |
| Vibe / mood | 10% | Claude API reasoning on genre and mood metadata |

### 3. Smart ordering
With a full compatibility matrix, Mixr solves for the optimal track sequence — the ordering that maximizes total transition quality across the whole playlist. Exact solution for playlists up to 12 tracks (Held-Karp), beam search for larger ones.

### 4. Transition execution
The mixing engine runs each transition:
- **Beat-locked crossfade** — starts and ends on a bar boundary, never mid-beat
- **EQ crossover** — outgoing track gets a low-cut, incoming gets a low-boost; bass never stacks
- **Tempo nudge** — if BPM delta is ≤ 6%, the outgoing track is time-stretched to close the gap
- **Gain normalization** — all tracks normalized to -14 LUFS before mixing

---

## The Hard Rule

**Every song plays completely through. Always.**

Mixr does not skip to the drop. It does not cut songs short. It does not chop out verses to get to the hook faster. You hear the full track, every time. The AI earns its keep by making the moments *between* songs sound great — not by butchering the songs themselves.

---

## Mixr vs Spotify Auto

| | Spotify Auto | Mixr |
|--|--|--|
| Track ordering | Fixed or random shuffle | AI-optimized for flow |
| Transition timing | Timer-based | Beat-locked to bar boundaries |
| Key awareness | Limited | Full Camelot wheel routing |
| BPM handling | No tempo matching | Tempo nudge within 6% delta |
| EQ at transitions | Basic | Low-frequency crossover |
| Vibe reasoning | None | Claude mood coherence layer |
| Full songs | Sometimes cuts | Always plays full tracks |

---

## API

Base URL: `https://api.mixr.io/v1`

### `POST /analyze`
Upload an audio file. Get back its full fingerprint.

```json
// Response
{
  "track_id": "uuid",
  "bpm": 142.3,
  "key_camelot": "12A",
  "tail_start_ms": 158000,
  "intro_end_ms": 8400,
  "duration_ms": 168000
}
```

### `POST /mix`
Send a list of analyzed track IDs. Get back a mixed audio file.

```json
// Request
{
  "track_ids": ["uuid1", "uuid2", "uuid3"],
  "metadata": [
    { "genre": "trap", "mood": "aggressive", "energy_level": 8 }
  ],
  "reorder": true,
  "energy_arc": false,
  "output_format": "mp3"
}
```

```
// Response
Binary audio file (MP3/WAV)
Header: X-Mix-Manifest-URL → /v1/mix/{id}/manifest
```

### `GET /mix/{id}/manifest`
Full breakdown of every decision Mixr made.

```json
{
  "ordered_tracks": ["uuid2", "uuid1", "uuid3"],
  "transitions": [
    {
      "from_id": "uuid2",
      "to_id": "uuid1",
      "score": 87.4,
      "overlap_bars": 4,
      "curve_type": "blend",
      "tempo_warped": true,
      "bpm_delta_pct": 2.8,
      "key_from": "12A",
      "key_to": "1A",
      "eq_applied": true
    }
  ]
}
```

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| API | FastAPI (Python) |
| Audio analysis | librosa + Essentia |
| Mixing engine | Pedalboard + pydub |
| AI reasoning | Claude API (Sonnet) |
| Task queue | Celery + Redis |
| Storage | Cloudflare R2 |
| Deployment | Docker + Fly.io |

---

## Status

Mixr is in active development.

- **v1** — Batch processing API, full mixing pipeline, Claude integration, manifest output
- **v2** — Real-time streaming, Spotify Web API integration, direct playlist import

---

*Mixr — Built different.*
