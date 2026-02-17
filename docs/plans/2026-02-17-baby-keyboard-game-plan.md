# Baby Keyboard Game - Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build a single-page web game where toddlers press keyboard keys and see large animated letters, emoji, and hear pronunciations, with a challenge mode for guided learning.

**Architecture:** Single `index.html` file with embedded CSS and JS. No dependencies. Keyboard events drive all interactions. CSS animations for visuals, Web Audio API for sound effects, Web Speech API for pronunciation.

**Tech Stack:** Vanilla HTML5, CSS3 (animations, flexbox), JavaScript (ES6+), Web Audio API, Web Speech API

---

### Task 1: HTML skeleton + CSS base styles + background

**Files:**
- Create: `index.html`

**Step 1: Create the HTML skeleton with all structural elements**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Baby Type!</title>
  <style>
    /* Base styles */
  </style>
</head>
<body>
  <div id="game">
    <div id="card-container">
      <div id="letter-card">
        <div id="letter"></div>
        <div id="emoji-word"></div>
      </div>
    </div>
    <div id="particles"></div>
    <div id="challenge-overlay"></div>
    <div id="parent-btn">&#9881;</div>
    <div id="parent-panel">
      <h3>Settings</h3>
      <label><input type="checkbox" id="mute-toggle"> Mute</label>
      <label><input type="checkbox" id="challenge-toggle" checked> Challenge Mode</label>
    </div>
    <div id="prompt">Press any key!</div>
  </div>
  <script>
    // JS will go here
  </script>
</body>
</html>
```

Add CSS for:
- `*` reset (margin/padding 0, box-sizing border-box)
- `html, body` full height, overflow hidden, user-select none
- `#game` full viewport, soft gradient background (`#FFF8F0` to `#F0E6FF`), centered flexbox
- `#card-container` centered, flex column, align center
- `#letter-card` large rounded card (min 280px), white semi-transparent background, border-radius 2rem, box-shadow, opacity 0 by default
- `#letter` font-size ~20vw, font-weight bold, system rounded font
- `#emoji-word` font-size ~4vw
- `#particles` absolute overlay, pointer-events none
- `#challenge-overlay` fixed center, hidden by default
- `#parent-btn` fixed top-right, small gray gear, opacity 0.3
- `#parent-panel` fixed right panel, hidden by default, white background, padding, border-radius
- `#prompt` bottom center, soft gray hint text, font-size 2vw
- Responsive: use vw/vh/vmin units, media queries for small screens

**Step 2: Open in browser and verify**

Run: `open index.html` (macOS)
Expected: Soft cream-to-lavender gradient background, centered empty card area, gear icon top-right barely visible, "Press any key!" hint at bottom.

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add HTML skeleton with base styles and soft gradient background"
```

---

### Task 2: Letter/number data mapping + key event handler + card display

**Files:**
- Modify: `index.html` (script section)

**Step 1: Add the data mapping object**

In the `<script>` section, add:

```javascript
const LETTER_MAP = {
  a: { emoji: '🍎', word: 'Apple', color: '#E07070' },
  b: { emoji: '🐻', word: 'Bear', color: '#C0956C' },
  c: { emoji: '🐱', word: 'Cat', color: '#E8A060' },
  d: { emoji: '🐶', word: 'Dog', color: '#C09860' },
  e: { emoji: '🐘', word: 'Elephant', color: '#A0A0B0' },
  f: { emoji: '🐟', word: 'Fish', color: '#70A0D0' },
  g: { emoji: '🍇', word: 'Grape', color: '#9070B0' },
  h: { emoji: '🐴', word: 'Horse', color: '#B08860' },
  i: { emoji: '🍦', word: 'Ice Cream', color: '#E0B0C0' },
  j: { emoji: '🪼', word: 'Jellyfish', color: '#B070D0' },
  k: { emoji: '🪁', word: 'Kite', color: '#70B0D0' },
  l: { emoji: '🦁', word: 'Lion', color: '#D0A040' },
  m: { emoji: '🌙', word: 'Moon', color: '#E0D070' },
  n: { emoji: '🥜', word: 'Nut', color: '#C0A070' },
  o: { emoji: '🐙', word: 'Octopus', color: '#D07090' },
  p: { emoji: '🐧', word: 'Penguin', color: '#606080' },
  q: { emoji: '👑', word: 'Queen', color: '#D0B040' },
  r: { emoji: '🌈', word: 'Rainbow', color: '#D08080' },
  s: { emoji: '☀️', word: 'Sun', color: '#E0C040' },
  t: { emoji: '🐯', word: 'Tiger', color: '#E0A040' },
  u: { emoji: '☂️', word: 'Umbrella', color: '#7080C0' },
  v: { emoji: '🎻', word: 'Violin', color: '#A07050' },
  w: { emoji: '🐋', word: 'Whale', color: '#5090B0' },
  x: { emoji: '🎵', word: 'Xylophone', color: '#C07090' },
  y: { emoji: '🧶', word: 'Yarn', color: '#D08090' },
  z: { emoji: '🦓', word: 'Zebra', color: '#707070' }
};

const NUMBER_EMOJI = '⭐';
```

**Step 2: Add the keydown event handler with throttle**

```javascript
let lastKeyTime = 0;
const THROTTLE_MS = 200;

document.addEventListener('keydown', (e) => {
  e.preventDefault();
  const now = Date.now();
  if (now - lastKeyTime < THROTTLE_MS) return;
  lastKeyTime = now;

  const key = e.key.toLowerCase();

  if (LETTER_MAP[key]) {
    showLetterCard(key);
  } else if (key >= '0' && key <= '9') {
    showNumberCard(key);
  } else {
    showSurprise();
  }
});
```

**Step 3: Implement showLetterCard and showNumberCard**

```javascript
const letterEl = document.getElementById('letter');
const emojiWordEl = document.getElementById('emoji-word');
const letterCard = document.getElementById('letter-card');
const promptEl = document.getElementById('prompt');

function showLetterCard(key) {
  const data = LETTER_MAP[key];
  letterEl.textContent = key.toUpperCase();
  letterEl.style.color = data.color;
  emojiWordEl.textContent = `${data.emoji} ${data.word}`;
  triggerCardAnimation();
  promptEl.style.display = 'none';
}

function showNumberCard(key) {
  const num = parseInt(key);
  letterEl.textContent = key;
  letterEl.style.color = '#E0C040';
  emojiWordEl.textContent = num === 0 ? '🌟' : NUMBER_EMOJI.repeat(num);
  triggerCardAnimation();
  promptEl.style.display = 'none';
}

function showSurprise() {
  // Will be implemented in Task 4
  promptEl.style.display = 'none';
}

function triggerCardAnimation() {
  letterCard.classList.remove('show');
  // Force reflow to restart animation
  void letterCard.offsetWidth;
  letterCard.classList.add('show');
}
```

**Step 4: Add the card animation CSS**

In `<style>`, add:

```css
@keyframes bounceIn {
  0% { transform: scale(0.3); opacity: 0; }
  50% { transform: scale(1.08); opacity: 1; }
  70% { transform: scale(0.95); }
  100% { transform: scale(1); opacity: 1; }
}

#letter-card.show {
  animation: bounceIn 0.5s cubic-bezier(0.34, 1.56, 0.64, 1) forwards;
}
```

**Step 5: Verify in browser**

Open `index.html`, press letter keys and number keys. Each should show the letter/number in the card with emoji and a bounce animation.

**Step 6: Commit**

```bash
git add index.html
git commit -m "feat: add letter/number data mapping and key event handler with card display"
```

---

### Task 3: Particle animation system

**Files:**
- Modify: `index.html` (style + script sections)

**Step 1: Add particle CSS**

```css
@keyframes particleFly {
  0% { transform: translate(0, 0) scale(1); opacity: 1; }
  100% { transform: translate(var(--dx), var(--dy)) scale(0); opacity: 0; }
}

.particle {
  position: absolute;
  border-radius: 50%;
  pointer-events: none;
  animation: particleFly 0.8s ease-out forwards;
}
```

**Step 2: Add spawnParticles function**

```javascript
const particlesEl = document.getElementById('particles');

function spawnParticles(count, x, y) {
  for (let i = 0; i < count; i++) {
    const p = document.createElement('div');
    p.className = 'particle';
    const size = Math.random() * 12 + 6;
    const angle = (Math.PI * 2 * i) / count + (Math.random() - 0.5) * 0.5;
    const dist = Math.random() * 120 + 60;
    const dx = Math.cos(angle) * dist;
    const dy = Math.sin(angle) * dist;
    const hue = Math.random() * 360;
    p.style.cssText = `
      width: ${size}px; height: ${size}px;
      left: ${x}px; top: ${y}px;
      background: hsl(${hue}, 70%, 70%);
      --dx: ${dx}px; --dy: ${dy}px;
    `;
    particlesEl.appendChild(p);
    p.addEventListener('animationend', () => p.remove());
  }
}
```

**Step 3: Call spawnParticles from showLetterCard and showNumberCard**

After `triggerCardAnimation()` in both functions, add:

```javascript
const rect = letterCard.getBoundingClientRect();
const cx = rect.left + rect.width / 2;
const cy = rect.top + rect.height / 2;
spawnParticles(12, cx, cy);
```

Extract this into a helper to avoid duplication:

```javascript
function emitCardParticles(count = 12) {
  requestAnimationFrame(() => {
    const rect = letterCard.getBoundingClientRect();
    const cx = rect.left + rect.width / 2;
    const cy = rect.top + rect.height / 2;
    spawnParticles(count, cx, cy);
  });
}
```

Call `emitCardParticles()` from showLetterCard and showNumberCard.

**Step 4: Verify in browser**

Press keys, confirm colorful particles burst from the card center and fade away.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add particle animation system with colorful burst on key press"
```

---

### Task 4: Surprise effects for non-letter/number keys

**Files:**
- Modify: `index.html` (style + script sections)

**Step 1: Add surprise effect styles**

```css
@keyframes floatUp {
  0% { transform: translateY(0) scale(1); opacity: 1; }
  100% { transform: translateY(-200px) scale(0.5); opacity: 0; }
}

.surprise-bubble {
  position: absolute;
  font-size: 3rem;
  pointer-events: none;
  animation: floatUp 1.2s ease-out forwards;
}
```

**Step 2: Implement showSurprise**

```javascript
const SURPRISE_EMOJIS = ['🎈', '⭐', '🎉', '✨', '🦋', '🌸', '💫', '🎵', '🫧', '🌟'];

function showSurprise() {
  // Hide the letter card
  letterCard.classList.remove('show');
  letterEl.textContent = '';
  emojiWordEl.textContent = '';

  // Spawn 5-8 random emoji bubbles at random positions
  const count = Math.floor(Math.random() * 4) + 5;
  for (let i = 0; i < count; i++) {
    const bubble = document.createElement('div');
    bubble.className = 'surprise-bubble';
    bubble.textContent = SURPRISE_EMOJIS[Math.floor(Math.random() * SURPRISE_EMOJIS.length)];
    bubble.style.left = Math.random() * 80 + 10 + 'vw';
    bubble.style.top = Math.random() * 60 + 20 + 'vh';
    particlesEl.appendChild(bubble);
    bubble.addEventListener('animationend', () => bubble.remove());
  }
}
```

**Step 3: Verify in browser**

Press space, enter, symbols, etc. Colorful emoji should float up from random positions.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add surprise emoji bubbles for non-letter/number keys"
```

---

### Task 5: Sound system (Web Audio API + Web Speech API)

**Files:**
- Modify: `index.html` (script section)

**Step 1: Create AudioContext and sound functions**

```javascript
let audioCtx = null;
let isMuted = false;

function getAudioCtx() {
  if (!audioCtx) audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  return audioCtx;
}

function playDing(freq = 800) {
  if (isMuted) return;
  try {
    const ctx = getAudioCtx();
    const osc = ctx.createOscillator();
    const gain = ctx.createGain();
    osc.type = 'sine';
    osc.frequency.value = freq;
    gain.gain.setValueAtTime(0.15, ctx.currentTime);
    gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.4);
    osc.connect(gain);
    gain.connect(ctx.destination);
    osc.start();
    osc.stop(ctx.currentTime + 0.4);
  } catch (e) { /* silent fail */ }
}

function playRisingScale() {
  if (isMuted) return;
  try {
    const ctx = getAudioCtx();
    const notes = [523, 659, 784];
    notes.forEach((freq, i) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.1, ctx.currentTime + i * 0.12);
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + i * 0.12 + 0.3);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start(ctx.currentTime + i * 0.12);
      osc.stop(ctx.currentTime + i * 0.12 + 0.3);
    });
  } catch (e) { /* silent fail */ }
}

function playCheer() {
  if (isMuted) return;
  try {
    const ctx = getAudioCtx();
    const chord = [523, 659, 784, 1047];
    chord.forEach((freq) => {
      const osc = ctx.createOscillator();
      const gain = ctx.createGain();
      osc.type = 'sine';
      osc.frequency.value = freq;
      gain.gain.setValueAtTime(0.08, ctx.currentTime);
      gain.gain.exponentialRampToValueAtTime(0.001, ctx.currentTime + 0.8);
      osc.connect(gain);
      gain.connect(ctx.destination);
      osc.start();
      osc.stop(ctx.currentTime + 0.8);
    });
  } catch (e) { /* silent fail */ }
}

function speak(text) {
  if (isMuted) return;
  try {
    const u = new SpeechSynthesisUtterance(text);
    u.rate = 0.8;
    u.pitch = 1.2;
    u.volume = 0.7;
    speechSynthesis.cancel();
    speechSynthesis.speak(u);
  } catch (e) { /* silent fail */ }
}
```

**Step 2: Integrate sounds into existing display functions**

In `showLetterCard`: add `playDing(600 + Math.random() * 400)` and `speak(key.toUpperCase() + ', ' + data.word)`

In `showNumberCard`: add `playDing(700)` and `speak(key)`

In `showSurprise`: add `playRisingScale()`

**Step 3: Verify in browser**

Press keys, hear ding sounds and letter pronunciation. Press symbols, hear rising scale.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add sound system with Web Audio API tones and Web Speech API pronunciation"
```

---

### Task 6: Challenge mode

**Files:**
- Modify: `index.html` (style + script sections)

**Step 1: Add challenge mode CSS**

```css
@keyframes glowPulse {
  0%, 100% { box-shadow: 0 0 20px rgba(255, 200, 100, 0.3); }
  50% { box-shadow: 0 0 40px rgba(255, 200, 100, 0.6); }
}

#challenge-overlay {
  position: fixed;
  top: 10vh;
  left: 50%;
  transform: translateX(-50%);
  font-size: 3vw;
  color: #887050;
  opacity: 0;
  transition: opacity 0.5s;
  pointer-events: none;
  text-align: center;
}

#challenge-overlay.active {
  opacity: 1;
}

#challenge-overlay .target-letter {
  display: inline-block;
  font-size: 10vw;
  font-weight: bold;
  padding: 1rem 2rem;
  border-radius: 1.5rem;
  background: rgba(255, 240, 220, 0.9);
  animation: glowPulse 1.5s infinite;
  margin-bottom: 0.5rem;
}
```

**Step 2: Implement challenge mode logic**

```javascript
let keyPressCount = 0;
let challengeActive = false;
let challengeTarget = null;
let challengeTimer = null;
let recentKeys = [];
let challengeEnabled = true;
const CHALLENGE_TRIGGER = 15 + Math.floor(Math.random() * 6); // 15-20

const challengeOverlay = document.getElementById('challenge-overlay');

function trackKeyPress(key) {
  if (/^[a-z]$/.test(key) && !recentKeys.includes(key)) {
    recentKeys.push(key);
    if (recentKeys.length > 8) recentKeys.shift();
  }
  keyPressCount++;

  if (challengeEnabled && !challengeActive && keyPressCount >= CHALLENGE_TRIGGER && recentKeys.length >= 3) {
    startChallenge();
    keyPressCount = 0;
  }
}

function startChallenge() {
  challengeTarget = recentKeys[Math.floor(Math.random() * recentKeys.length)];
  challengeActive = true;
  const data = LETTER_MAP[challengeTarget];
  challengeOverlay.innerHTML = `
    <div>Can you find...</div>
    <div class="target-letter" style="color: ${data.color}">${challengeTarget.toUpperCase()}</div>
    <div>${data.emoji}</div>
  `;
  challengeOverlay.classList.add('active');

  challengeTimer = setTimeout(() => {
    endChallenge(false);
  }, 10000);
}

function checkChallenge(key) {
  if (!challengeActive) return false;
  if (key === challengeTarget) {
    clearTimeout(challengeTimer);
    endChallenge(true);
    return true;
  }
  return false;
}

function endChallenge(success) {
  challengeActive = false;
  challengeTarget = null;
  challengeOverlay.classList.remove('active');

  if (success) {
    playCheer();
    emitCardParticles(30);
  }
}
```

**Step 3: Integrate into keydown handler**

Modify the keydown handler to call `trackKeyPress(key)` and `checkChallenge(key)`.

If `checkChallenge` returns true for a letter key, show the letter card AND the super reward animation.

**Step 4: Verify in browser**

Press 15-20 letter keys, then a challenge should appear. Press the target letter, see super reward. Wait 10 seconds, challenge fades.

**Step 5: Commit**

```bash
git add index.html
git commit -m "feat: add challenge mode with target letter, glow animation, and super reward"
```

---

### Task 7: Parent control panel

**Files:**
- Modify: `index.html` (style + script sections)

**Step 1: Add parent panel interaction CSS**

```css
#parent-panel.open {
  display: block;
}
```

**Step 2: Implement long-press gear button**

```javascript
const parentBtn = document.getElementById('parent-btn');
const parentPanel = document.getElementById('parent-panel');
const muteToggle = document.getElementById('mute-toggle');
const challengeToggle = document.getElementById('challenge-toggle');
let pressTimer = null;

parentBtn.addEventListener('mousedown', () => {
  pressTimer = setTimeout(() => {
    parentPanel.classList.toggle('open');
  }, 3000);
});
parentBtn.addEventListener('mouseup', () => clearTimeout(pressTimer));
parentBtn.addEventListener('mouseleave', () => clearTimeout(pressTimer));

// Touch events for mobile
parentBtn.addEventListener('touchstart', (e) => {
  e.preventDefault();
  pressTimer = setTimeout(() => {
    parentPanel.classList.toggle('open');
  }, 3000);
});
parentBtn.addEventListener('touchend', () => clearTimeout(pressTimer));

// Close panel when clicking outside
document.addEventListener('click', (e) => {
  if (!parentPanel.contains(e.target) && e.target !== parentBtn) {
    parentPanel.classList.remove('open');
  }
});

// Toggle handlers
muteToggle.addEventListener('change', () => {
  isMuted = muteToggle.checked;
  if (isMuted) speechSynthesis.cancel();
});

challengeToggle.addEventListener('change', () => {
  challengeEnabled = challengeToggle.checked;
  if (!challengeEnabled && challengeActive) {
    clearTimeout(challengeTimer);
    endChallenge(false);
  }
});
```

**Step 3: Verify in browser**

Long-press gear icon for 3 seconds, panel opens. Toggle mute, confirm sounds stop. Toggle challenge mode off. Click outside to close.

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add parent control panel with mute and challenge mode toggles"
```

---

### Task 8: Touch support + browser shortcut blocking + polish

**Files:**
- Modify: `index.html` (script section)

**Step 1: Add touch event for screen taps**

```javascript
document.getElementById('game').addEventListener('touchstart', (e) => {
  if (e.target.closest('#parent-btn') || e.target.closest('#parent-panel')) return;
  e.preventDefault();

  for (const touch of e.changedTouches) {
    const bubble = document.createElement('div');
    bubble.className = 'surprise-bubble';
    bubble.textContent = SURPRISE_EMOJIS[Math.floor(Math.random() * SURPRISE_EMOJIS.length)];
    bubble.style.left = touch.clientX + 'px';
    bubble.style.top = touch.clientY + 'px';
    particlesEl.appendChild(bubble);
    bubble.addEventListener('animationend', () => bubble.remove());
  }
  playDing(500 + Math.random() * 300);
});
```

**Step 2: Block dangerous browser shortcuts**

```javascript
document.addEventListener('keydown', (e) => {
  // Block Ctrl+W, Ctrl+Q, Ctrl+N, etc.
  if (e.ctrlKey || e.metaKey || e.altKey) {
    e.preventDefault();
    return;
  }
}, { capture: true });
```

Note: This needs to be the FIRST keydown listener (capture phase), placed before the game keydown handler. Restructure: put the blocking listener first with capture, and the game handler second.

**Step 3: Verify in browser**

- Tap screen on mobile/touch device: emoji bubbles appear at touch point
- Press Ctrl+W: page does NOT close
- Press F5 or other function keys: get surprise effect instead of browser action

**Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add touch support, block browser shortcuts, final polish"
```

---

### Task 9: Final review and cleanup

**Files:**
- Modify: `index.html`

**Step 1: Review the complete file**

Read through the entire file and verify:
- All CSS animations are smooth (no sudden opacity jumps)
- All DOM cleanup happens (animationend removes particles/bubbles)
- Color palette is consistently soft and low-saturation
- Font sizes use relative units (vw/vh/vmin)
- No console errors in browser dev tools

**Step 2: Test all interactions end-to-end**

- Press every letter A-Z: correct emoji, word, color, sound, particles
- Press numbers 0-9: correct star count, sound
- Press space, enter, symbols: surprise bubbles
- Wait for challenge mode: target appears, correct key gives super reward
- Parent panel: long-press opens, toggles work, click-outside closes
- Touch screen: bubbles appear at touch point

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: complete baby keyboard game v1"
```
