---
title: Music Playlist Manager
app_type: music-playlist-manager
wallet: 0x35147fd220a13779ce9d0f3c38e51aca23aa5876
---

You are an expert React developer. Generate a complete, production-ready React application for a music playlist manager.

Requirements:

- Header with app title "GrooveList" and a now-playing bar at the bottom showing the currently selected song
- A left sidebar with 3 hardcoded playlists: Chill Vibes, Workout Mix, Late Night — clicking switches the main view
- Each playlist displays a grid of song cards, pre-populated with 5 songs each
- Each song card shows: song title, artist name, duration, genre tag, and a colored album art placeholder using a gradient div
- Clicking a song card marks it as "Now Playing" with a pulsing indicator and updates the bottom now-playing bar
- A like button (heart icon using inline SVG) on each card that toggles filled or outline state
- A search bar at the top that filters songs by title or artist across all playlists
- Dark music-app aesthetic with deep black, dark gray, and green accent colors

Technical:

- React 18 with TypeScript
- Tailwind CSS for all styling
- Use useState for active playlist, selected song, liked songs, and search state
- Export a single default App component
- No external UI libraries, icon packs, or audio functionality
- Use inline SVG for heart and play icons
- Fully responsive layout
