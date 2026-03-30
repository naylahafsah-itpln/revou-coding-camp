# Implementation Plan: Interactive Pet Cat

## Overview

Build a single `index.html` file containing all HTML, CSS (Tailwind via CDN), and vanilla JavaScript for the interactive pet cat app. Tasks are ordered so each step integrates cleanly into the previous one, ending with a fully wired, working app.

## Tasks

- [x] 1. Scaffold the HTML file with layout and static cat display
  - Create `index.html` with Tailwind CDN link and pastel gradient background
  - Add centered glassmorphism container panel (translucent bg, backdrop-blur, soft shadow)
  - Render the cat as an emoji/ASCII element inside the panel
  - Add mood label element beneath the cat and pet counter element visible on the page
  - Use relative units (rem, %, vh/vw) for sizing; ensure layout works 320px–2560px
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 7.1, 9.1, 9.2, 9.3, 9.4_

- [x] 2. Implement CSS keyframe animations
  - [x] 2.1 Define the four CSS keyframe animations in a `<style>` block
    - `anim-bounce`: vertical bounce
    - `anim-shake`: horizontal shake
    - `anim-spin`: full 360° rotation
    - `anim-pulse`: scale up/down
    - All animations complete within 1000ms
    - _Requirements: 4.1, 4.2, 4.3, 4.4_

  - [ ]* 2.2 Write property test for animation timing (Property 7)
    - **Property 7: Animation completes within 1000 ms**
    - **Validates: Requirements 4.2, 4.3**

- [x] 3. Implement core data models and app state
  - [x] 3.1 Define mood and reaction data structures in JavaScript
    - Define `AppState` singleton with `petCount`, `currentMoodId`, `reactionInProgress`, `lastReactionId`, `audioContextInitialized`
    - Define all 4 moods (happy, sleepy, excited, grumpy) each with ≥ 2 reactions (≥ 8 total)
    - Each reaction has `id`, `message`, `animation`, `sound` fields
    - _Requirements: 3.1, 6.1_

  - [ ]* 3.2 Write property test for reaction pool coverage (Property 9)
    - **Property 9: Reaction pool coverage — total reactions ≥ 8, per-mood ≥ 2**
    - **Validates: Requirements 3.1, 6.1**

- [x] 4. Implement PetCounter and UIUpdater
  - [x] 4.1 Implement `PetCounter` module
    - `PetCounter.increment()` → increments `AppState.petCount`, returns new count
    - `PetCounter.get()` → returns current count
    - _Requirements: 2.3, 7.3_

  - [ ]* 4.2 Write property test for pet counter increment (Property 1)
    - **Property 1: Pet increments counter — for any initial count, increment → count + 1**
    - **Validates: Requirements 2.3**

  - [x] 4.3 Implement `UIUpdater` module
    - `UIUpdater.setCounter(n)` → updates counter DOM element immediately
    - `UIUpdater.setMood(mood)` → updates mood label DOM element
    - `UIUpdater.triggerMoodTransition()` → applies brief color-shift/glow CSS class to container, removes it after transition
    - _Requirements: 6.3, 6.4, 7.2_

- [x] 5. Implement MoodSystem
  - [x] 5.1 Implement `MoodSystem` module
    - `MoodSystem.current()` → returns current mood object
    - `MoodSystem.checkTransition(count)` → if count > 0 and count % 10 === 0, pick a random mood different from current, update `AppState.currentMoodId`, return new mood; otherwise return null
    - `MoodSystem.getReactions()` → returns reactions array for current mood
    - _Requirements: 6.1, 6.2, 6.5_

  - [ ]* 5.2 Write property test for mood transition (Properties 5 & 6)
    - **Property 5 + 6: For any pet count that is a multiple of 10, new mood ≠ old mood**
    - **Validates: Requirements 6.2, 6.5**

- [x] 6. Implement AnimationController and SpeechBubbleController
  - [x] 6.1 Implement `AnimationController` module
    - `AnimationController.play(catElement, animationName)` → adds CSS class, returns a Promise that resolves after 1000ms and removes the class
    - _Requirements: 4.1, 4.2, 4.3, 4.4_

  - [x] 6.2 Implement `SpeechBubbleController` module
    - `SpeechBubbleController.show(message)` → sets bubble text, makes bubble visible adjacent to cat, schedules auto-hide after 2000ms
    - `SpeechBubbleController.hide()` → fades out and hides bubble
    - If `show()` is called while bubble is visible, replace message immediately and reset the 2000ms timer
    - Apply glassmorphism styling to the speech bubble element
    - _Requirements: 3.3, 5.1, 5.2, 5.3, 5.4_

  - [ ]* 6.3 Write property test for speech bubble auto-hide (Property 4)
    - **Property 4: Speech bubble hidden after 2000 ms**
    - **Validates: Requirements 3.3, 5.3**

- [x] 7. Implement SoundEngine
  - [x] 7.1 Implement `SoundEngine` module using Web Audio API
    - `SoundEngine.supported` → boolean, set at init time based on `AudioContext` availability
    - `SoundEngine.init()` → lazily creates `AudioContext` on first pet; no-op if already initialized or unsupported
    - `SoundEngine.play(soundType)` → no-op if unsupported; plays synthesized tone for the given key (purr, chirp, hiss, meow) at volume ≤ 0.5
    - Implement each sound type as a short synthesized burst (sine/triangle wave)
    - _Requirements: 8.1, 8.2, 8.3, 8.4_

  - [ ]* 7.2 Write property test for sound engine graceful degradation (Property 8)
    - **Property 8: No error thrown when Web Audio API unavailable**
    - **Validates: Requirements 8.4**

- [x] 8. Implement ReactionEngine and EventHandler
  - [x] 8.1 Implement `ReactionEngine` module
    - `ReactionEngine.trigger(mood)` → picks a random reaction from mood's pool, enforces no-immediate-repeat (reaction id ≠ `AppState.lastReactionId`), stores selected id in `AppState.lastReactionId`, then calls `AnimationController.play`, `SpeechBubbleController.show`, and `SoundEngine.play` in parallel; sets `reactionInProgress = false` when animation resolves
    - _Requirements: 2.2, 3.2, 3.4, 3.5_

  - [ ]* 8.2 Write property test for no immediate reaction repeat (Property 3)
    - **Property 3: Two consecutive reactions for the same mood are always different**
    - **Validates: Requirements 3.5**

  - [x] 8.3 Implement `EventHandler` module
    - Attach `click`/`tap` listener to cat element
    - If `AppState.reactionInProgress === true`, ignore the event
    - Otherwise: set `reactionInProgress = true`, call `SoundEngine.init()`, call `PetCounter.increment()`, call `MoodSystem.checkTransition(count)` (update UI if mood changed), call `ReactionEngine.trigger(currentMood)`
    - _Requirements: 2.1, 2.4, 8.3_

  - [ ]* 8.4 Write property test for reaction lock (Property 2)
    - **Property 2: Rapid clicks during a reaction increment counter by exactly 1**
    - **Validates: Requirements 2.4**

- [x] 9. Checkpoint — wire everything together and verify
  - Ensure `EventHandler.init()` is called on `DOMContentLoaded`
  - Verify mood label and counter are rendered correctly on load
  - Ensure all modules reference the shared `AppState` singleton
  - Ensure all tests pass, ask the user if questions arise.
  - _Requirements: 1.4, 7.1, 7.2_

- [x] 10. Final checkpoint — full integration validation
  - Confirm clicking the cat triggers animation, speech bubble, and sound together
  - Confirm mood transitions at pet counts 10, 20, 30, … with visual glow
  - Confirm speech bubble replaces correctly on rapid pets
  - Confirm layout is correct at 320px and 1440px viewport widths
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property tests use **fast-check** (`fc.assert(fc.property(...))`) with ≥ 100 iterations each
- Each property test comment must include: `// Feature: interactive-pet-cat, Property N: <property text>`
- All code lives in a single `index.html` file — no build step, no external assets
