# Audio Design Rules · huashu-design

> The audio application recipe for all animation demos. Used together with `sfx-library.md` (asset list).
> Battle-hardened: huashu-design hero v1-v9 release iterations · in-depth Gemini analysis of three official Anthropic films · 8000+ A/B comparisons.

---

## Core Principle · Audio Dual-Track System (iron rule)

Animation audio **must be designed as two independent layers**, you cannot do only one:

| Layer | Function | Time scale | Relationship to visuals | Frequency band occupied |
|---|---|---|---|---|
| **SFX (beat layer)** | Mark each visual beat | 0.2-2 second short bursts | **Strong sync** (frame-level alignment) | **High frequency 800Hz+** |
| **BGM (atmosphere bed)** | Mood foundation, sound field | Continuous 20-60 seconds | Weak sync (section level) | **Mid-low frequency <4kHz** |

**An animation with only BGM is crippled** — viewers subconsciously perceive "the picture is moving but there's no sound response", and that's the root of the cheap feeling.

---

## Gold Standard · The Golden Ratios

These values are **engineering hard parameters** derived from real measurements of Anthropic's three official films + comparison with our own v9 final, plug-and-play:

### Volume
- **BGM volume**: `0.40-0.50` (relative to full scale 1.0)
- **SFX volume**: `1.00`
- **Loudness difference**: BGM is **-6 to -8 dB lower** than SFX peak (it's not the SFX's absolute loudness that makes it stand out, it's the loudness difference)
- **amix parameter**: `normalize=0` (never use normalize=1, it flattens the dynamic range)

### Frequency band isolation (P1 hard optimization)
The Anthropic secret is not "SFX volume is loud", it's **frequency band layering**:

```bash
[bgm_raw]lowpass=f=4000[bgm]      # Restrict BGM to mid-low frequency below 4kHz
[sfx_raw]highpass=f=800[sfx]      # Push SFX to mid-high frequency above 800Hz
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]
```

Why: The human ear is most sensitive to the 2-5kHz range (the "presence band"). If SFX are all in this range and BGM covers the full spectrum, **the SFX get masked by the high-frequency portion of the BGM**. Use highpass to push SFX up + lowpass to push BGM down, so the two occupy different sides of the spectrum, and SFX clarity goes up a tier.

### Fade
- BGM in: `afade=in:st=0:d=0.3` (0.3s, avoid hard cuts)
- BGM out: `afade=out:st=N-1.5:d=1.5` (1.5s long tail, sense of conclusion)
- SFX have built-in envelopes, no extra fade needed

---

## SFX cue design rules

### Density (how many SFX per 10 seconds)
The SFX density of Anthropic's three films measured in three tiers:

| Film | SFX per 10s | Product personality | Scenario |
|---|---|---|---|
| Artifacts (ref-1) | **~9 per 10s** | Feature-dense, info-heavy | Complex tool demo |
| Code Desktop (ref-2) | **0** | Pure ambience, meditative | Dev tool focused state |
| Word (ref-3) | **~4 per 10s** | Balanced, office rhythm | Productivity tool |

**Heuristics**:
- Product personality calm/focused → low SFX density (0-3 per 10s), BGM dominant
- Product personality lively/info-heavy → high SFX density (6-9 per 10s), SFX-driven rhythm
- **Don't fill every visual beat** — empty space is more sophisticated than density. **Removing 30-50% of cues makes the remaining ones more dramatic**.

### Cue selection priority
Not every visual beat needs SFX. Choose by this priority:

**P0 must include** (omitting causes incongruity):
- Typing (terminal/input)
- Click/select (user decision moment)
- Focus shift (visual subject transfer)
- Logo reveal (brand conclusion)

**P1 recommended**:
- Element entrance/exit (modal / card)
- Completion/success feedback
- AI generation start/end
- Major transitions (scene switches)

**P2 optional** (too many gets messy):
- hover / focus-in
- Progress tick
- Decorative ambient

### Timestamp alignment precision
- **Same-frame alignment** (0ms error): Click / focus shift / logo landing
- **Lead by 1-2 frames** (-33ms): Fast whoosh (gives viewer psychological anticipation)
- **Lag by 1-2 frames** (+33ms): Object landing/impact (matches real physics)

---

## BGM Selection Decision Tree

The huashu-design skill ships with 6 BGM tracks (`assets/bgm-*.mp3`):

```
What is the animation's personality?
├─ Product launch / tech demo → bgm-tech.mp3 (minimal synth + piano)
├─ Tutorial explanation / tool usage → bgm-tutorial.mp3 (warm, instructional)
├─ Education / principle explanation → bgm-educational.mp3 (curious, thoughtful)
├─ Marketing ad / brand promotion → bgm-ad.mp3 (upbeat, promotional)
└─ Same style needs variant → bgm-*-alt.mp3 (alternative version of each)
```

### No-BGM scenarios (worth considering)
Reference Anthropic Code Desktop (ref-2): **0 SFX + pure Lo-fi BGM** can also be very high-end.

**When to choose no BGM**:
- Animation duration < 10s (BGM can't establish itself)
- Product personality is "focused/meditative"
- The scene itself has ambient sound / narration
- When SFX density is very high (avoid auditory overload)

---

## Scenario Recipes (out-of-the-box)

### Recipe A · Product launch hero (same as huashu-design v9)
```
Duration: 25 seconds
BGM: bgm-tech.mp3 · 45% · band <4kHz
SFX density: ~6 per 10s

cues:
  Terminal typing → type × 4 (interval 0.6s)
  Enter           → enter
  Card converge   → card × 4 (staggered 0.2s)
  Selected        → click
  Ripple          → whoosh
  4 focus shifts  → focus × 4
  Logo            → thud (1.5s)

Volume: BGM 0.45 / SFX 1.0 · amix normalize=0
```

### Recipe B · Tool feature demo (reference Anthropic Code Desktop)
```
Duration: 30-45 seconds
BGM: bgm-tutorial.mp3 · 50%
SFX density: 0-2 per 10s (very few)

Strategy: Let BGM + narration voiceover drive, SFX only at **decisive moments** (file save / command execution complete)
```

### Recipe C · AI generation demo
```
Duration: 15-20 seconds
BGM: bgm-tech.mp3 or no BGM
SFX density: ~8 per 10s (high density)

cues:
  User input → type + enter
  AI starts processing → magic/ai-process (1.2s loop)
  Generation complete → feedback/complete-done
  Result presented → magic/sparkle
  
Highlight: ai-process can loop 2-3 times throughout the entire generation
```

### Recipe D · Pure ambient long shot (reference Artifacts)
```
Duration: 10-15 seconds
BGM: none
SFX: Use 3-5 carefully designed cues independently

Strategy: Each SFX is the protagonist, no BGM "smearing" issue.
Suitable for: single-product slow shots, close-up displays
```

---

## ffmpeg Composition Templates

### Template 1 · Layer single SFX onto video
```bash
ffmpeg -y -i video.mp4 -itsoffset 2.5 -i sfx.mp3 \
  -filter_complex "[0:a][1:a]amix=inputs=2:normalize=0[a]" \
  -map 0:v -map "[a]" output.mp4
```

### Template 2 · Multi-SFX timeline composition (aligned by cue time)
```bash
ffmpeg -y \
  -i sfx-type.mp3 -i sfx-enter.mp3 -i sfx-click.mp3 -i sfx-thud.mp3 \
  -filter_complex "\
[0:a]adelay=1100|1100[a0];\
[1:a]adelay=3200|3200[a1];\
[2:a]adelay=7000|7000[a2];\
[3:a]adelay=21800|21800[a3];\
[a0][a1][a2][a3]amix=inputs=4:duration=longest:normalize=0[mixed]" \
  -map "[mixed]" -t 25 sfx-track.mp3
```
**Key parameters**:
- `adelay=N|N`: First is left channel delay (ms), second is right channel; write twice to ensure stereo alignment
- `normalize=0`: Preserve dynamic range, critical!
- `-t 25`: Truncate to specified duration

### Template 3 · Video + SFX track + BGM (with band isolation)
```bash
ffmpeg -y -i video.mp4 -i sfx-track.mp3 -i bgm.mp3 \
  -filter_complex "\
[2:a]atrim=0:25,afade=in:st=0:d=0.3,afade=out:st=23.5:d=1.5,\
     lowpass=f=4000,volume=0.45[bgm];\
[1:a]highpass=f=800,volume=1.0[sfx];\
[bgm][sfx]amix=inputs=2:duration=first:normalize=0[a]" \
  -map 0:v -map "[a]" -c:v copy -c:a aac -b:a 192k final.mp4
```

---

## Failure Mode Quick Reference

| Symptom | Root cause | Fix |
|---|---|---|
| Can't hear SFX | BGM high-frequency portion is masking it | Add `lowpass=f=4000` to BGM + `highpass=f=800` to SFX |
| SFX too loud and harsh | SFX absolute volume too high | Drop SFX volume to 0.7, simultaneously drop BGM to 0.3, maintain the difference |
| BGM and SFX rhythms conflict | Wrong BGM choice (used music with strong beat) | Switch to ambient / minimal synth BGM |
| BGM cuts off abruptly when animation ends | No fade out done | `afade=out:st=N-1.5:d=1.5` |
| SFX overlap into mush | Cues too dense + each SFX too long | Keep SFX duration under 0.5s, cue interval ≥ 0.2s |
| WeChat Official Account mp4 has no sound | WeChat sometimes mutes auto-play | Don't worry, users get sound when they tap to open; gif has no sound by design |

---

## Coupling with visuals (advanced)

### SFX timbre must match visual style
- Warm rice/paper-like visuals → SFX use **wood/soft** timbres (Morse, paper snap, soft click)
- Cold high-tech visuals → SFX use **metallic/digital** timbres (beep, pulse, glitch)
- Hand-drawn/whimsical visuals → SFX use **cartoon/exaggerated** timbres (boing, pop, zap)

Our current `apple-gallery-showcase.md` warm rice base color → paired with `keyboard/type.mp3` (mechanical) + `container/card-snap.mp3` (soft) + `impact/logo-reveal-v2.mp3` (cinematic bass)

### SFX can drive visual rhythm
Advanced technique: **Design the SFX timeline first, then adjust visual animation to align with the SFX** (not the other way around).
Because each SFX cue is a "clock tick", visual animation adapting to SFX rhythm will be very stable — conversely, SFX chasing visuals often feels off if you're ±1 frame off.

---

## Quality Checklist (self-check before release)

- [ ] Loudness diff: SFX peak - BGM peak = -6 to -8 dB?
- [ ] Frequency: BGM lowpass 4kHz + SFX highpass 800Hz?
- [ ] amix normalize=0 (preserve dynamic range)?
- [ ] BGM fade-in 0.3s + fade-out 1.5s?
- [ ] Is SFX count appropriate (density chosen by scenario personality)?
- [ ] Each SFX aligned to visual beat at same frame (within ±1 frame)?
- [ ] Logo reveal SFX duration sufficient (1.5s recommended)?
- [ ] Listen with BGM muted: are SFX alone rhythmically sufficient?
- [ ] Listen with SFX muted: does BGM alone have emotional rise and fall?

Either layer alone should be self-consistent. If it only sounds good when both are stacked, you haven't done it right.

---

## References

- SFX asset list: `sfx-library.md`
- Visual style reference: `apple-gallery-showcase.md`
- In-depth audio analysis of Anthropic's three films: `AUDIO-BEST-PRACTICES.md` (in the original author's local workspace; not included in this repository)
- huashu-design v9 case study: `hero-animation-v9-final.mp4` (in the original author's local workspace; not included in this repository)
