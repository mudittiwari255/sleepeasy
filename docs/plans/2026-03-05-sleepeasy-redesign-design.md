# SleepEasy v2 Design Document

**Project:** SleepEasy - Indian City Atlas Sleep Game
**Date:** March 5, 2026
**Version:** 2.0
**Status:** Design Approved

---

## Executive Summary

SleepEasy is a voice-driven sleep hypnosis web application that uses an atlas-style city naming game with Indian cities. Users take turns naming cities with the AI, which progressively slows down speech to create a hypnotic, sleep-inducing experience. This redesign addresses critical reliability issues, adds new sleep-enhancing features, and introduces a comprehensive soothing UI system.

**Key Changes:**
- Fix voice selection bug on Android
- Replace white noise with high-quality brown noise
- Improve speech recognition reliability
- 100% Indian cities database (500+ cities)
- New features: intensity levels, sleep timer, guided meditation
- Daily session management
- Complete UI redesign with soothing, sleep-appropriate aesthetics

---

## Table of Contents

1. [Architecture Overview](#1-architecture-overview)
2. [Core Features](#2-core-features)
3. [Technical Specifications](#3-technical-specifications)
4. [UI/UX Design System](#4-uiux-design-system)
5. [Critical Fixes](#5-critical-fixes)
6. [New Features](#6-new-features)
7. [Data Flow & State Management](#7-data-flow--state-management)
8. [Error Handling & Edge Cases](#8-error-handling--edge-cases)
9. [Testing Strategy](#9-testing-strategy)
10. [Implementation Roadmap](#10-implementation-roadmap)

---

## 1. Architecture Overview

### 1.1 Technology Stack

**Core Technologies:**
- Vanilla HTML/CSS/JavaScript (no frameworks)
- Web Speech API (SpeechRecognition + SpeechSynthesis)
- Web Audio API (brown noise generation)
- LocalStorage (session persistence)

**Optional Enhancement:**
- Tailwind CSS (for styling, if needed)

### 1.2 Application Structure

```
┌─────────────────────────────────────────────────────────────┐
│                      SleepEasy v2                          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │ VoiceEngine  │  │SpeechRecognizer│ │CityValidator│     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │AmbientSound  │  │GameController│ │MeditationMode│     │
│  │   Engine     │  │              │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│                                                             │
│  ┌──────────────┐  ┌──────────────┐                        │
│  │SettingsPanel │  │SessionTimer  │                        │
│  └──────────────┘  └──────────────┘                        │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

### 1.3 Platform Support

| Platform | Browser | Status |
|----------|---------|--------|
| Android | Chrome | ✅ Full Support |
| iOS | Safari | ✅ Full Support |
| Desktop | Chrome, Edge, Safari | ✅ Full Support |
| Desktop | Firefox | ⚠️ Speech API limited |

---

## 2. Core Features

### 2.1 Feature Matrix

| Feature | v1 | v2 | Notes |
|---------|----|----|-------|
| Atlas City Game | ✅ | ✅ | Enhanced |
| Progressive Slowdown | ✅ | ✅ | Customizable rates |
| Ghost Mode | ✅ | ✅ | Improved |
| Brown Noise | ❌ | ✅ | Replaces white noise |
| City Memory | ❌ | ✅ | New: track used cities |
| Intensity Levels | ❌ | ✅ | New: 3 presets |
| Sleep Timer | ❌ | ✅ | New: with fade-out |
| Guided Meditation | ❌ | ✅ | New mode |
| Daily Sessions | ❌ | ✅ | Fresh start daily |
| Voice Selection | ⚠️ | ✅ | Fixed Android bug |
| Indian Cities Only | ⚠️ | ✅ | 500+ comprehensive |

### 2.2 Preserved Features

All existing v1 features are preserved and enhanced:
- Breathing animation (visual feedback)
- Starfield background
- Wake Lock API (screen stays on)
- Session state tracking
- Turn-based gameplay

---

## 3. Technical Specifications

### 3.1 Component Responsibilities

| Component | Responsibility |
|-----------|----------------|
| **VoiceEngine** | TTS playback, voice selection, progressive slowdown, device-specific voice handling |
| **SpeechRecognizer** | STT, noise handling, retry logic, "already used" detection |
| **CityValidator** | Indian city lookup, used-city checking, letter validation |
| **AmbientSoundEngine** | High-quality brown noise generation, volume control, smooth fades |
| **GameController** | Turn management, ghost mode, session lifecycle |
| **MeditationMode** | Guided meditation scripts, separate from atlas mode |
| **SettingsPanel** | Intensity, timer, ambient sound selection UI |
| **SessionTimer** | Countdown timer with gradual audio fade-out |

### 3.2 Data Structures

```javascript
// Session State
sessionState = {
  // Mode Selection
  mode: 'atlas' | 'meditation',

  // Game State
  usedCities: [],           // All cities used this session
  lastCity: '',             // Last spoken city
  lastLetter: '',           // Last letter for next turn
  turnCount: 0,

  // Hypnotic Settings
  speechRate: 0.95,         // Current TTS speed
  hypnoticDelay: 1500,      // Current delay (ms)
  intensity: 'low' | 'medium' | 'high',

  // Session Management
  sleepTimer: null,         // Remaining seconds
  isGhostMode: false,
  sessionStartTime: null,

  // Audio Settings
  ambientSound: 'brownNoise',
  ambientVolume: 0.4,
  currentVoice: null,

  // Daily Session
  lastSessionDate: null     // For detecting new day
}

// Indian City Database
indianCities = [
  { name: "Mumbai", state: "Maharashtra" },
  { name: "Chennai", state: "Tamil Nadu" },
  { name: "Kolkata", state: "West Bengal" },
  // ~500+ cities from all Indian states
]
```

---

## 4. UI/UX Design System

### 4.1 Design Philosophy

**Core Principle:** Every UI decision prioritizes calming the user and preparing them for sleep.

- Dark, muted tones reduce eye strain
- Slow, organic animations never jar the user
- Minimal interface reduces cognitive load
- Large touch targets work for tired fingers
- Gentle gradients create depth without harshness

### 4.2 Color Palette

```css
/* Background Layers */
--bg-deep: #0a0e1a;        /* Deepest night sky */
--bg-surface: #111827;     /* Main surface */
--bg-elevated: #1f2937;    /* Cards, panels */
--bg-subtle: #374151;      /* Hover states */

/* Text */
--text-primary: #e2e8f0;   /* Soft white - reduced brightness */
--text-secondary: #94a3b8; /* Muted text */
--text-muted: #64748b;     /* De-emphasized */

/* Accents - Gentle, Calming */
--accent-primary: #818cf8;   /* Soft lavender-blue */
--accent-secondary: #a78bfa; /* Gentle lavender */
--accent-success: #6ee7b7;   /* Soft mint - for positive states */
--accent-warm: #fbbf24;      /* Muted amber - rare use */

/* Special States */
--listening-glow: #6366f1;   /* Indigo when listening */
--speaking-glow: #818cf8;    /* Lavender when speaking */
--ghost-mode: #a78bfa;       /* Soft purple for ghost mode */

/* Gradient Overlays */
--gradient-night: linear-gradient(180deg, #0f172a 0%, #0a0e1a 100%);
--gradient-glow: radial-gradient(circle, rgba(129,140,248,0.15) 0%, transparent 70%);
```

### 4.3 Typography

**Font Stack:**
```css
font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
```

**Type Scale:**

| Usage | Size | Weight | Line Height | Color |
|-------|------|--------|-------------|-------|
| Display (App Title) | 32px | 500 (Medium) | 1.2 | `--accent-primary` |
| Headline (Session) | 24px | 500 (Medium) | 1.3 | `--text-primary` |
| City Name (Large) | 48px | 400 (Regular) | 1.1 | `--text-primary` |
| Body | 16px | 400 (Regular) | 1.5 | `--text-secondary` |
| Caption (Timer, Status) | 14px | 400 (Regular) | 1.4 | `--text-muted` |
| Button | 16px | 500 (Medium) | 1 | `--bg-deep` |

### 4.4 Screen Designs

#### Home Screen

```
┌─────────────────────────────────────────┐
│                                         │
│              🌙 SleepEasy               │
│                                         │
│                                         │
│    ┌───────────────────────────────┐   │
│    │                               │   │
│    │   Start Tonight's Session     │   │
│    │                               │   │
│    └───────────────────────────────┘   │
│                                         │
│         Voice: Indian English           │
│         Intensity: Medium               │
│              [Settings ▸]              │
│                                         │
└─────────────────────────────────────────┘
```

**Key Elements:**
- Centered layout with moon icon
- Large, inviting CTA button (280x80px minimum)
- Quick settings summary
- Minimal distractions

#### Voice Selection Screen

```
┌─────────────────────────────────────────┐
│  ← Back              Select Voice       │
├─────────────────────────────────────────┤
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  ★ Google Indian English        │   │
│  │  [Preview ▶]                    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │  ★ Natural Online Voice         │   │
│  │  [Preview ▶]                    │   │
│  └─────────────────────────────────┘   │
│                                         │
│  ┌─────────────────────────────────┐   │
│  │    Microsoft Ravi               │   │
│  │  [Preview ▶]                    │   │
│  └─────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

#### Settings Panel

```
┌─────────────────────────────────────────┐
│               Settings                  │
├─────────────────────────────────────────┤
│                                         │
│  Intensity                              │
│  ○ Low   ● Medium   ○ High             │
│                                         │
│  Sleep Timer                            │
│  ○ Off   ○ 30m   ● 45m   ○ 60m         │
│                                         │
│  Ambient Sound                          │
│  ● Brown Noise                          │
│  Volume: ████████░░ 80%                │
│                                         │
│  Voice                                  │
│  [Change Voice ▸]                       │
│                                         │
│  [Save]                   [Cancel]      │
│                                         │
└─────────────────────────────────────────┘
```

#### Active Session Screen (Atlas Mode)

```
┌─────────────────────────────────────────┐
│  Mumbai                   [⏸] [✕]      │
│                                         │
│         ◉  BREATHING ANIMATION  ◉       │
│     (softly pulses with listening-glow) │
│                                         │
│  🎤 Listening...                        │
│                                         │
│  12 cities used          Timer: 38:22   │
│                                         │
│  Ghost mode in 45s                      │
│                                         │
└─────────────────────────────────────────┘
```

**Key Elements:**
- **City Name**: Center, large (48px), glowing text
- **Breathing Core**: Animated circle that expands/contracts (3s cycle)
- **Status Indicator**: Small icon + text below
- **Timer/Stats**: Subtle, bottom corners
- **Controls**: Minimal, icon-only, 44x44px touch targets

#### Active Session Screen (Meditation Mode)

```
┌─────────────────────────────────────────┐
│  Body Scan Meditation      [⏸] [✕]     │
│                                         │
│         ◉  BREATHING ANIMATION  ◉       │
│                                         │
│  "Focus on your feet. Notice any        │
│   sensations without judging them..."   │
│                                         │
│  3:45 / 10:00          Timer: 35:15     │
│                                         │
└─────────────────────────────────────────┘
```

### 4.5 Animation Standards

**Timing Standards:**
```css
:root {
  --duration-instant: 150ms;   /* Button feedback */
  --duration-quick: 300ms;     /* Hover states */
  --duration-smooth: 500ms;    /* Page transitions */
  --duration-slow: 1000ms;     /* Breathing animation */
  --duration-very-slow: 3000ms; /* Ghost mode transitions */

  --easing-gentle: cubic-bezier(0.4, 0, 0.2, 1);
  --easing-breathe: cubic-bezier(0.3, 0, 0.3, 1);
}
```

**Key Animations:**

1. **Breathing Core (Active Session)**
```css
@keyframes breathe {
  0%, 100% {
    transform: scale(1);
    opacity: 0.6;
  }
  50% {
    transform: scale(1.15);
    opacity: 0.8;
  }
}

.breathing-core {
  animation: breathe 3s cubic-bezier(0.3, 0, 0.3, 1) infinite;
}
```

2. **Status Glow (Listening/Speaking)**
```css
@keyframes pulse-glow {
  0%, 100% { box-shadow: 0 0 20px var(--listening-glow); }
  50% { box-shadow: 0 0 40px var(--listening-glow); }
}

.status-listening {
  animation: pulse-glow 2s ease-in-out infinite;
}
```

3. **Fade In (Page Transitions)**
```css
@keyframes fade-in {
  from { opacity: 0; transform: translateY(8px); }
  to { opacity: 1; transform: translateY(0); }
}

.fade-in {
  animation: fade-in 500ms cubic-bezier(0.4, 0, 0.2, 1);
}
```

### 4.6 Accessibility

- **High contrast** for all text (WCAG AA compliant)
- **Large touch targets** (minimum 44x44px for mobile)
- **Reduced motion** support (`prefers-reduced-motion` query)
- **Screen reader announcements** for state changes
- **Focus indicators** visible but subtle

```css
@media (prefers-reduced-motion: reduce) {
  *,
  *::before,
  *::after {
    animation-duration: 0.01ms !important;
    transition-duration: 0.01ms !important;
  }
}
```

---

## 5. Critical Fixes

### 5.1 Voice Selection Bug on Android

**Problem:** Voice selection UI shows multiple options but all produce the same voice.

**Root Cause:** Code likely stores voice index/ID but doesn't re-apply correctly when `speechSynthesis.speak()` is called. Android's implementation may return voices in different order or reset the voice.

**Solution:**
```javascript
// Store voice URI (not index) as unique identifier
selectedVoiceURI = voiceObj.voiceURI;
selectedVoiceName = voiceObj.name;

// Before speaking, explicitly set voice by URI
const utterance = new SpeechSynthesisUtterance(text);
const allVoices = speechSynthesis.getVoices();
utterance.voice = allVoices.find(v => v.voiceURI === selectedVoiceURI);

// Fallback: match by name if URI fails
if (!utterance.voice) {
  utterance.voice = allVoices.find(v => v.name === selectedVoiceName);
}

// Android-specific: Re-fetch voices after each utterance
utterance.onend = () => {
  speechSynthesis.getVoices(); // Refresh voice list
};
```

**Additional Safeguards:**
- Add voice preview button in settings
- Log voice selection for debugging
- Store voice preference in localStorage
- Validate voice still exists after page reload

### 5.2 Speech Recognition Failures

**Problem:** App fails to recognize user speech consistently.

**Root Cause:** Web Speech Recognition is sensitive to background noise, pauses, and inconsistent audio levels.

**Solutions:**

1. **Better Configuration:**
```javascript
recognition.continuous = false;        // One phrase at a time
recognition.interimResults = false;     // Only final results
recognition.maxAlternatives = 3;        // Get multiple guesses
recognition.lang = 'en-IN';             // Indian English
```

2. **Ambient Sound Ducking:**
```javascript
// Lower ambient volume by 50% when listening
ambientSoundEngine.setVolume(0.2); // Down from 0.4

// Restore after recognition completes
recognition.onresult = () => {
  ambientSoundEngine.setVolume(0.4);
};
```

3. **Confidence Threshold:**
```javascript
recognition.onresult = (event) => {
  const results = event.results[0];
  const transcript = results[0].transcript;
  const confidence = results[0].confidence;

  if (confidence < 0.6) {
    // Low confidence - re-prompt
    speakMessage("I couldn't hear you clearly. Please say the city name again.");
    return;
  }

  processCity(transcript);
};
```

4. **Smart Retry:**
```javascript
let retryCount = 0;

recognition.onerror = (event) => {
  if (event.error === 'no-speech' && retryCount < 2) {
    retryCount++;
    speakMessage("Please try speaking again.");
    startListening();
  } else {
    handleRecognitionFailure();
  }
};
```

5. **Fuzzy Matching:**
```javascript
// If confidence is low but city name sounds similar
function findSimilarCity(spoken) {
  const similarities = indianCities.map(city => ({
    city: city.name,
    similarity: levenshteinDistance(spoken, city.name.toLowerCase())
  }));

  const bestMatch = similarities.sort((a, b) => a.similarity - b.similarity)[0];

  if (bestMatch.similarity <= 2) { // Close match
    speakMessage(`Did you say ${bestMatch.city}?`);
    // Confirm with user
  }
}
```

### 5.3 White Noise Issues

**Problem:** White noise is too loud and has poor quality (harsh, sibilant).

**Solution: Switch to Brown Noise**

Brown noise has -6dB/octave rolloff - much softer, deeper, more natural than white noise. Think heavy rainfall or distant thunder.

**Implementation:**
```javascript
// Generate high-quality brown noise
const bufferSize = 2 * context.sampleRate;
const noiseBuffer = context.createBuffer(1, bufferSize, context.sampleRate);
const output = noiseBuffer.getChannelData(0);

let lastOut = 0;
for (let i = 0; i < bufferSize; i++) {
  const white = Math.random() * 2 - 1;
  output[i] = (lastOut + (0.02 * white)) / 1.02;
  lastOut = output[i];
  output[i] *= 3.5; // Gain compensation
}

// Lower default volume (40% instead of 70%)
brownNoiseGain = 0.4;

// Add soft attack/release to prevent clicks
const gainNode = context.createGain();
gainNode.gain.setValueAtTime(0, context.currentTime);
gainNode.gain.linearRampToValueAtTime(0.4, context.currentTime + 0.5); // 500ms attack
```

---

## 6. New Features

### 6.1 City Memory System

**Feature:** Track all cities used in current session and reject repeats.

**Implementation:**
```javascript
function validateCity(cityName) {
  const normalized = cityName.toLowerCase().trim();

  // Check if already used
  if (sessionState.usedCities.includes(normalized)) {
    speakMessage(`${cityName} already used in this session. Please try another city.`);
    startListening(); // Re-prompt for input
    return false;
  }

  // Check if valid Indian city
  const isValid = indianCities.some(city =>
    city.name.toLowerCase() === normalized
  );

  if (!isValid) {
    speakMessage(`I don't have ${cityName} in my Indian cities list. Try another city.`);
    startListening();
    return false;
  }

  // Check correct starting letter
  const firstLetter = normalized[0];
  if (firstLetter !== sessionState.lastLetter) {
    speakMessage(`That doesn't start with ${sessionState.lastLetter}. Please try again.`);
    startListening();
    return false;
  }

  // Valid city - add to used list
  sessionState.usedCities.push(normalized);
  return true;
}
```

### 6.2 Intensity Levels

**Feature:** Three presets that affect hypnotic parameters.

| Setting | Speech Slowdown Rate | Hypnotic Delay Increase | Brown Noise Volume | Max Delay |
|---------|---------------------|------------------------|-------------------|-----------|
| **Low** | -0.03 every 3 turns | +0.3s per turn | 30% | 8s |
| **Medium** | -0.05 every 3 turns | +0.5s per turn | 40% | 5s |
| **High** | -0.08 every 3 turns | +0.7s per turn | 50% | 3s |

**Implementation:**
```javascript
const intensityPresets = {
  low: {
    slowdownRate: 0.03,
    delayIncrement: 300,
    ambientVolume: 0.3,
    maxDelay: 8000
  },
  medium: {
    slowdownRate: 0.05,
    delayIncrement: 500,
    ambientVolume: 0.4,
    maxDelay: 5000
  },
  high: {
    slowdownRate: 0.08,
    delayIncrement: 700,
    ambientVolume: 0.5,
    maxDelay: 3000
  }
};

function applyIntensity(level) {
  const preset = intensityPresets[level];
  sessionState.intensity = level;
  ambientSoundEngine.setVolume(preset.ambientVolume);
}
```

### 6.3 Sleep Timer with Fade-Out

**Feature:** User can set timer (15/30/45/60 min) with graceful audio fade.

**Implementation:**
```javascript
class SessionTimer {
  constructor() {
    this.remaining = null;
    this.interval = null;
    this.fadeStarted = false;
  }

  start(minutes) {
    this.remaining = minutes * 60;
    this.fadeStarted = false;

    this.interval = setInterval(() => {
      this.remaining--;
      updateTimerDisplay(this.remaining);

      // Start fade at 90% of timer
      if (this.remaining <= (this.total * 0.1) && !this.fadeStarted) {
        this.startFadeOut();
        this.fadeStarted = true;
      }

      if (this.remaining <= 0) {
        this.endSession();
      }
    }, 1000);
  }

  startFadeOut() {
    // Fade voice volume over 60 seconds
    const fadeDuration = 60000;
    const fadeSteps = 60;
    const volumeStep = 1.0 / fadeSteps;

    let currentStep = 0;
    const fadeInterval = setInterval(() => {
      currentStep++;
      voiceEngine.setVolume(1.0 - (currentStep * volumeStep));

      if (currentStep >= fadeSteps) {
        clearInterval(fadeInterval);
        this.fadeBrownNoise();
      }
    }, fadeDuration / fadeSteps);
  }

  fadeBrownNoise() {
    // Fade brown noise over 30 seconds
    const fadeDuration = 30000;
    ambientSoundEngine.fadeTo(0, fadeDuration);

    setTimeout(() => {
      ambientSoundEngine.stop();
      speakMessage("Session complete. Rest well.");
    }, fadeDuration);
  }

  endSession() {
    clearInterval(this.interval);
    // Show completion screen
  }
}
```

### 6.4 Guided Meditation Mode

**Feature:** Separate mode with scripted meditations (body scan, breathing exercises).

**Meditation Scripts:**
```javascript
const meditations = {
  bodyScan: {
    name: "Body Scan Relaxation",
    duration: 600, // 10 minutes
    script: [
      { time: 0, text: "Close your eyes and take a deep breath..." },
      { time: 30, text: "Bring your attention to your feet..." },
      { time: 60, text: "Notice any sensations without judgment..." },
      // ... more script segments
    ]
  },
  breathing: {
    name: "4-7-8 Breathing",
    duration: 300, // 5 minutes
    script: [
      { time: 0, text: "Inhale for 4 counts..." },
      { time: 4, text: "Hold for 7 counts..." },
      { time: 11, text: "Exhale for 8 counts..." },
      // ... repeat pattern
    ]
  }
};
```

**UI Flow:**
1. User selects "Meditation" mode on home screen
2. Chooses meditation type (Body Scan, Breathing, etc.)
3. Selects duration (5, 10, 15 minutes)
4. Session starts with guided audio
5. Breathing animation syncs with script
6. Timer counts down
7. Gentle completion message

### 6.5 Daily Session Management

**Feature:** Fresh session each day with "Start Tonight's Session" button.

**Implementation:**
```javascript
function checkForNewDay() {
  const lastSessionDate = localStorage.getItem('lastSessionDate');
  const today = new Date().toDateString();

  if (lastSessionDate !== today) {
    // New day - fresh start
    localStorage.setItem('lastSessionDate', today);
    localStorage.removeItem('sessionState');
    return true;
  }

  return false;
}

function startNewSession() {
  sessionState = {
    // ... fresh state
  };
  localStorage.setItem('sessionState', JSON.stringify(sessionState));
  localStorage.setItem('lastSessionDate', new Date().toDateString());
}

// On app load
if (checkForNewDay()) {
  showWelcomeScreen(); // "Start Tonight's Session"
}
```

---

## 7. Data Flow & State Management

### 7.1 Typical Game Turn Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                         USER SPEAKS                             │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│              SPEECH RECOGNIZER STARTS                            │
│  • Duck brown noise volume 50%                                  │
│  • Start listening with timeout (8 seconds)                     │
└─────────────────────────────────────────────────────────────────┘
                              │
                    ┌─────────┴─────────┐
                    ▼                   ▼
              SUCCESS              FAILURE/TIMEOUT
                    │                   │
                    ▼                   ▼
          ┌─────────────────┐   ┌──────────────────┐
          │ VALIDATE INPUT  │   │ Re-prompt user:  │
          └─────────────────┘   │ "Try again..."   │
                    │           └──────────────────┘
                    ▼
          ┌─────────────────────────────────────┐
          │  Is city in usedCities array?       │
          └─────────────────────────────────────┘
                    │
          ┌─────────┴─────────┐
          ▼                   ▼
        YES                  NO
          │                   │
          ▼                   ▼
┌──────────────────┐  ┌─────────────────────┐
│ "Already used,   │  │ Valid city!         │
│  try another"    │  │ • Add to usedCities │
└──────────────────┘  │ • Extract last letter│
                      │ • Increment turn    │
                      └─────────────────────┘
                                   │
                                   ▼
                      ┌─────────────────────────────┐
                      │     AI SPEAKS NEXT CITY     │
                      │  • Calculate new speechRate │
                      │  • Calculate new delay      │
                      │  • Speak after delay        │
                      └─────────────────────────────┘
```

### 7.2 State Transitions

```
IDLE → VOICE_SELECTION → SETTINGS → ACTIVE_SESSION → COMPLETED
                                    ↓
                               GHOST_MODE
```

### 7.3 Persistence Strategy

| Data | Storage | Duration |
|------|---------|----------|
| Session State | localStorage | Until new day |
| Voice Preference | localStorage | Persistent |
| Settings | localStorage | Persistent |
| Last Session Date | localStorage | Persistent |
| Indian Cities Database | Embedded | Permanent |

---

## 8. Error Handling & Edge Cases

| Scenario | Handling |
|----------|----------|
| **No voice selected** | Auto-select best available voice, show toast confirming which |
| **Speech API not supported** | Show friendly message: "This browser doesn't support speech. Try Chrome/Safari." |
| **No microphone permission** | Request permission with explanation, show fallback UI button |
| **Recognition returns empty** | "I couldn't hear you. Please say the city name clearly." |
| **City not in database** | "I don't have that city in my list. Try another Indian city." |
| **Wrong starting letter** | "That doesn't start with [letter]. Please try again." |
| **Ghost mode needed** | After 60s silence: "I'll continue for a while..." then AI plays alone |
| **User never speaks** | After 45s: "Still with me?" (gentle check) |
| **Page refresh during session** | Save state to localStorage, offer "Resume last session?" on reload |
| **Timer reaches zero** | Fade all audio, show "Session complete. Sleep well." |
| **Network error during meditation** | Show cached meditation, continue offline |
| **Voice unavailable after reload** | Fall back to default voice, notify user |

---

## 9. Testing Strategy

### 9.1 Critical Test Cases

| # | Test Case | Platform | Expected Result |
|---|-----------|----------|-----------------|
| 1 | Voice selection on Android phone | Android/Chrome | Different voices sound different |
| 2 | Voice selection on laptop | Desktop/Chrome | Same voice selected sounds consistent |
| 3 | Speech recognition in quiet room | All | Accurate city recognition |
| 4 | Speech recognition with brown noise | All | Accurate recognition with ambient |
| 5 | Repeat city detection | All | Immediate rejection with message |
| 6 | Progressive slowdown after 10+ turns | All | Noticeable speech slowing |
| 7 | Ghost mode activation after 60s | All | AI continues alone |
| 8 | Sleep timer fade-out | All | No abrupt cutoffs |
| 9 | All intensity levels | All | Different behaviors |
| 10 | Meditation mode playback | All | Clear, calm guidance |
| 11 | Daily session reset | All | Fresh start each day |
| 12 | Brown noise quality | All | Deep, rumbly, not harsh |

### 9.2 Device Testing Matrix

| Device | OS | Browser | Priority |
|--------|-------|---------|----------|
| Android Phone | Android 14+ | Chrome | ⭐⭐⭐ Critical |
| iPhone | iOS 17+ | Safari | ⭐⭐⭐ Critical |
| Laptop | macOS/Windows | Chrome | ⭐⭐ Important |
| Tablet | iPadOS | Safari | ⭐⭐ Important |
| Laptop | macOS/Windows | Safari/Edge | ⭐ Nice-to-have |

### 9.3 User Scenarios

1. **First-time user:** Opens app, selects voice, starts session, plays for 5 minutes
2. **Returning user:** Opens app next day, starts new session, uses timer
3. **Insomnia user:** Uses high intensity, 60-minute timer, ghost mode activates
4. **Meditation user:** Skips atlas mode, uses guided meditation
5. **Mobile user:** Uses app in bed, phone locks (wake lock keeps on)

---

## 10. Implementation Roadmap

### Phase 1: Critical Fixes (Week 1)

**Priority: P0 - Blocks all other work**

| Task | Estimate | Owner |
|------|----------|-------|
| Fix Android voice selection bug | 2 days | |
| Replace white noise with brown noise | 1 day | |
| Improve speech recognition reliability | 2 days | |
| Add city memory with "already used" rejection | 1 day | |

**Deliverables:**
- Voices work correctly on Android
- Brown noise is pleasant and not too loud
- Speech recognition works 90%+ of the time in quiet room
- Cities are tracked and repeats are rejected

### Phase 2: New Core Features (Week 2)

**Priority: P1 - Core v2 features**

| Task | Estimate | Owner |
|------|----------|-------|
| Build comprehensive Indian cities database (500+) | 2 days | |
| Implement intensity level settings | 1 day | |
| Add sleep timer with fade-out | 2 days | |
| Create guided meditation mode | 2 days | |
| Implement daily session management | 1 day | |

**Deliverables:**
- Indian cities database covering all states
- Intensity presets work correctly
- Timer fades audio gracefully
- Meditation mode is functional
- Daily sessions reset properly

### Phase 3: UI/UX Implementation (Week 2-3)

**Priority: P1 - Complete visual overhaul**

| Task | Estimate | Owner |
|------|----------|-------|
| Implement new color system | 1 day | |
| Redesign home screen | 1 day | |
| Redesign voice selection screen | 1 day | |
| Redesign settings panel | 1 day | |
| Redesign active session screen | 2 days | |
| Add breathing animation | 1 day | |
| Add all screen transitions | 1 day | |

**Deliverables:**
- Soothing dark theme implemented
- All screens redesigned to spec
- Breathing animation works smoothly
- Transitions are calm and gentle

### Phase 4: Polish & Enhancements (Week 3)

**Priority: P2 - Important for quality**

| Task | Estimate | Owner |
|------|----------|-------|
| Add session save/resume to localStorage | 1 day | |
| Improve UI for voice selection (preview) | 1 day | |
| Add meditation script content | 2 days | |
| Performance optimization | 1 day | |
| Accessibility audit & fixes | 1 day | |

**Deliverables:**
- Sessions persist across refresh
- Voice preview works
- Multiple meditation scripts available
- App loads quickly
- WCAG AA compliant

### Phase 5: Testing & Launch (Week 3-4)

**Priority: P0 - Must pass before launch**

| Task | Estimate | Owner |
|------|----------|-------|
| Full test suite execution | 2 days | |
| Bug fixes from testing | 2 days | |
| Device-specific testing | 1 day | |
| Documentation | 1 day | |
| Deployment | 1 day | |

**Deliverables:**
- All test cases pass
- No known critical bugs
- Tested on all priority devices
- User documentation complete
- Deployed to production

---

## Appendix A: Indian Cities Database Structure

**Coverage:** All 28 states + 8 union territories

**Categories:**
- Major cities (population > 1M)
- State capitals
- Important districts
- Historic/cultural cities
- Tier-2 cities

**Sample Data:**
```javascript
{
  name: "Mumbai",
  state: "Maharashtra",
  population: 12442373,
  type: "metropolitan"
}

{
  name: "Jaipur",
  state: "Rajasthan",
  population: 3073350,
  type: "capital"
}
```

**Minimum Target:** 500 cities
**Stretch Goal:** 1000 cities

---

## Appendix B: Meditation Scripts

**Initial Scripts:**

1. **Body Scan Relaxation** (5 min, 10 min, 15 min versions)
2. **4-7-8 Breathing Exercise** (5 min)
3. **Progressive Muscle Relaxation** (10 min)
4. **Sleep Visualization** ("Peaceful Lake") (10 min)
5. **Gratitude Meditation** (8 min)

**Script Format:**
```javascript
{
  id: "body-scan-10",
  name: "Body Scan Relaxation",
  duration: 600,
  segments: [
    { time: 0, text: "Find a comfortable position...", voiceSpeed: 1.0 },
    { time: 30, text: "Take a deep breath in...", voiceSpeed: 0.98 },
    // ...
  ]
}
```

---

## Appendix C: Glossary

| Term | Definition |
|------|------------|
| **Atlas Game** | Turn-based word game where players name cities starting with the last letter |
| **Brown Noise** | Deep, rumbly sound with -6dB/octave rolloff; softer than white noise |
| **Ghost Mode** | AI continues playing alone after user stops responding (60s silence) |
| **Progressive Slowdown** | Speech gradually slows down to create hypnotic effect |
| **Hypnotic Delay** | Increasing pause between turns to deepen relaxation |
| **Intensity Level** | Preset affecting slowdown rate, delay, and ambient volume |

---

## Sign-off

**Design Version:** 2.0
**Date:** March 5, 2026
**Status:** ✅ Approved - Ready for Implementation Planning

**Next Step:** Invoke writing-plans skill to create detailed implementation plan.

---

*This design document is the source of truth for SleepEasy v2 development. All implementation decisions should reference this document.*
