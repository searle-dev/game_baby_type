# Baby Keyboard Game - Design Document

## Overview

A single-page web game for 2-year-old toddlers. The baby presses keyboard keys and sees
large, colorful visual feedback on screen with corresponding emoji, words, and animations.
The game is eye-friendly (soft colors, no flashing) and provides positive reinforcement
through a combination of free-play mode and occasional challenge mode.

## Technical Approach

Single `index.html` file with embedded `<style>` and `<script>`. Zero external dependencies.
Pure vanilla HTML/CSS/JS.

## Page Structure

- Full-screen game area (captures keyboard events)
- Central display area (letter card, emoji, word)
- Particle/animation layer (CSS animations)
- Challenge mode overlay (target letter prompt)
- Parent control corner (top-right gear icon, 3-second long-press to open)

## Visual Design

- **Background**: Soft gradient (cream to lavender), low saturation
- **Letter card**: Large rounded card (~40% of screen), each letter has a fixed soft color
- **Emoji**: Displayed below the letter card with the English word
- **Animations**: Bounce-in for letter card, colorful particle burst around it
- **Font**: System rounded font (`-apple-system`), large and friendly
- **No flashing**: All animations are smooth transitions, no sudden brightness changes

## Key Response Logic

### Letters A-Z
- Display large letter card + corresponding emoji + English word
- Web Speech API announces letter and word (e.g., "A, Apple")
- Particle animation burst

### Numbers 0-9
- Display large number + repeated emoji (e.g., 3 -> three stars)
- Web Speech API announces number

### Other keys (space, symbols, function keys)
- Trigger random "surprise" visual effect (bubbles, stars, sparkles)
- Play a cheerful tone
- Ensures every key press gets positive feedback

### Throttling
- 200ms throttle to prevent animation stacking from held/rapid keys

## Challenge Mode

- **Trigger**: After ~15-20 consecutive key presses in free mode, randomly trigger
- **Mechanic**: Show a target letter with a glowing halo, invite baby to press it
- **Correct**: Super reward animation (more particles, bigger bounce, cheer sound)
- **Wrong**: Normal feedback, target stays visible
- **Timeout**: Target fades after 10 seconds, returns to free mode
- **Difficulty**: Target chosen from recently pressed keys (higher success chance)

## Sound Design

- **Speech**: Web Speech API for letter/number pronunciation
- **Effects**: Web Audio API generated tones (no audio files needed)
  - Letter/number: Clear ding
  - Surprise: Rising scale
  - Challenge success: Happy chord
- All sounds are soft and moderate volume

## Parent Control Panel

- **Entry**: Gear icon top-right, requires 3-second long press
- **Features**:
  - Mute toggle
  - Challenge mode toggle
  - Language switch (reserved for future, English only in v1)
- **Close**: Tap outside panel

## Technical Details

- **Responsive**: `vw/vh` units, works on phone/tablet/desktop
- **Touch support**: Tapping screen triggers random colorful bubble feedback
- **Performance**: CSS `@keyframes` animations, DOM nodes removed after animation ends
- **Offline**: Pure static file, Speech API degrades silently without network
- **Safety**: Block default browser shortcuts (Ctrl+W, etc.) to prevent accidental closure

## Data Mapping (A-Z)

| Key | Emoji | Word |
|-----|-------|------|
| A | apple | Apple |
| B | bear | Bear |
| C | cat | Cat |
| D | dog | Dog |
| E | elephant | Elephant |
| F | fish | Fish |
| G | grape | Grape |
| H | horse | Horse |
| I | ice cream | Ice Cream |
| J | jellyfish | Jellyfish |
| K | kite | Kite |
| L | lion | Lion |
| M | moon | Moon |
| N | nut | Nut |
| O | octopus | Octopus |
| P | penguin | Penguin |
| Q | queen | Queen |
| R | rainbow | Rainbow |
| S | sun | Sun |
| T | tiger | Tiger |
| U | umbrella | Umbrella |
| V | violin | Violin |
| W | whale | Whale |
| X | xylophone | Xylophone |
| Y | yarn | Yarn |
| Z | zebra | Zebra |

## Numbers 0-9

Each number displays that many star emojis.
