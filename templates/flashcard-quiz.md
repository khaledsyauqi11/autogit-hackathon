---
title: Flashcard Quiz App
app_type: flashcard-quiz-app
wallet: 0x7d8369c99339e1b9fd1087db50d24926ed06095c
---

You are building a Flashcard Quiz App called "BrainDeck" using React + Vite + Tailwind CSS. Generate a complete, fully functional single-page application.

## App Requirements

### Pre-loaded Deck
Include 10 pre-loaded flashcards on the topic "General Knowledge" with question on front and answer on back.

### Flashcard Viewer
- Show one card at a time with a large card UI in the center
- Card flip animation (CSS 3D transform rotateY) when user clicks the card
- Front: shows question with a "?" icon
- Back: shows answer with a "✓" icon
- Navigation: Previous / Next buttons

### Quiz Mode
- "Start Quiz" button shuffles the deck
- After flipping to see the answer, show two buttons: "Got it ✓" (green) and "Missed ✗" (red)
- Track score: X correct out of Y answered
- At the end of the deck: show results screen with score percentage and "Retry" button

### Deck Management
- Simple form to add a custom card: Question input + Answer input + "Add Card" button
- Show total card count: "X cards in deck"

### Visual Design
- Rich indigo/purple theme (#4f46e5 primary, #7c3aed accent)
- Cards have white background with subtle box-shadow
- Flip animation must be smooth (0.6s ease)
- Font: "Poppins" from Google Fonts
- Confetti emoji burst (rendered as animated spans) when quiz score >= 80%

### Technical Rules
- CSS 3D flip using transform-style: preserve-3d — no libraries
- Single App.jsx
- No backend
