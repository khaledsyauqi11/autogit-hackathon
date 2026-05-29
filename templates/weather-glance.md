---
title: Weather Glance
app_type: weather-glance
wallet: 0x63CAC3a278A58361a6134Ad44891A9Fe3f98bB29
---

A page that opens to a single glance of the weather. Current condition in one short phrase, the temperature in one big number, the next twelve hours as a small ribbon along the bottom. The page asks for the visitor's location once, fetches the weather from a public no key open meteo endpoint, caches the response through the Cache api so a quick reload does not refetch, and cancels stale requests with an AbortController when the visitor moves between hours.

═══ THE GLANCE ═══

A mist colored page, the gentle blue grey of a sky an hour after sunrise. Slate dark for body type. Sun yellow reserved for the temperature number when the current condition is clear and the small sun glyph in the corner. Cool blue reserved for the small numeric subscripts in the ribbon. Source Serif 4 at 32px for the current condition phrase at the top, Geist at 168px for the big temperature number, Geist at 14px 500 for the small labels and buttons, Geist Mono at 12px for the times under the ribbon hour cells. Three typefaces, three roles, no overlap. The layout is one full viewport, centered, no scroll.

The first thing the visitor sees is the condition phrase in Source Serif 4 at the top center, like a quiet morning, a clear cold afternoon, light rain after lunch. Below the phrase, the big temperature number, like 14 with a small superscript degree mark and a slightly smaller cardinal direction wind glyph to the right of the number in Geist Mono 12px. Below the number, the ribbon, twelve small cells each 56 by 96 pixels showing the next twelve hours, each cell stacked top to bottom: the hour in Geist Mono 12px, a small inline svg condition glyph (sun, partly sunny, cloudy, rain, snow, fog), the temperature in Geist 16px. The active hour cell (the current one) has a 2px sun yellow outline.

═══ THE LOCATION ═══

On first load, the page asks for navigator.geolocation through a small Geist 14px line at the top center reading the page would like your rough location to fetch the weather, with a single Geist 14px 500 text button reading allow once. Tapping it asks the browser's standard prompt. On grant, the App stores the resolved latitude longitude in localStorage under the key weather.glance.place.v1, with a precision of one decimal degree (about eleven kilometers), so the saved location does not pinpoint the visitor's apartment. On decline, the page shows a small input asking for a city name. The city name is converted to coordinates through an in source list of about three hundred world cities, the visitor picks the closest match from a small dropdown. The page does not call a geocoding api.

A small text button below the temperature reads change the place, which reopens the choice dialog with the same flow.

═══ THE FETCH ═══

The weather is fetched from a public no key endpoint at open meteo. The exact url, the parameters, and the response shape are below. The fetch is wrapped in an AbortController so a second fetch (triggered by the visitor changing the place or by the page resyncing after a long sleep) cancels any in flight request from the previous one, preventing a stale response from overwriting a fresh one.

```
const controller = new AbortController();
const url = new URL('https://api.open-meteo.com/v1/forecast');
url.searchParams.set('latitude',  String(lat));
url.searchParams.set('longitude', String(lon));
url.searchParams.set('current',   'temperature_2m,weather_code,wind_speed_10m,wind_direction_10m');
url.searchParams.set('hourly',    'temperature_2m,weather_code');
url.searchParams.set('timezone',  'auto');
url.searchParams.set('forecast_hours', '12');
fetch(url, { signal: controller.signal }).then(...);
```

The response is a small json object with current and hourly fields, the App reads only the keys the page renders. A typical response shape, written in src/lib/openmeteo.ts as a comment.

```
{
  current: {
    time:               '2026-05-29T13:00',
    temperature_2m:     14.2,
    weather_code:       2,
    wind_speed_10m:     12.4,
    wind_direction_10m: 224
  },
  hourly: {
    time:           ['2026-05-29T13:00', '2026-05-29T14:00', ...],
    temperature_2m: [14.2, 14.5, 14.7, ...],
    weather_code:   [2, 2, 3, 3, ...]
  }
}
```

The weather code is mapped to one of six condition kinds (clear, partly sunny, cloudy, rain, snow, fog) through a small in source table in src/lib/codes.ts. The kind drives both the inline svg glyph in the ribbon and the temperature number color.

═══ THE CACHE ═══

The fetch's response is cached through the standard Cache api under a cache named weather.glance.v1, keyed by the request url. On boot, before firing the network fetch, the App opens the cache and pulls the matching response if any. If a cached response is fresh (under fifteen minutes old), it is used directly and the network fetch is skipped. If it is stale, it is shown immediately while a background fetch refreshes it (a stale while revalidate strategy). The cache is populated by the fetch callback, every successful response replaces the previous entry under the same key.

If the visitor changes the place, the new url is fetched fresh and cached under its own key. The cache holds at most three entries, the oldest is evicted when a fourth place is set, through a small in source LRU pass at every cache write.

═══ THE ANCHOR ═══

A small detail panel appears when the visitor hovers or taps a ribbon cell, showing the full hour's weather code phrase, the wind speed and direction, and the precipitation probability if the response includes it. The panel is positioned through CSS Anchor Positioning, with the ribbon cell as the anchor and the panel positioned above the cell with a 12 pixel offset, falling back to inline coordinates when CSS Anchor Positioning is unavailable. The panel uses a CSS mask image to clip its corners to a small dovetail shape that visually points at the cell.

The mask is a single inline svg url, no external image. The CSS rule is below.

```
.detail-panel {
  position-anchor: --hour-cell;
  position: absolute;
  position-area: top span-x;
  mask-image: url("data:image/svg+xml,...");
}
```

The panel disappears on pointerleave or on a second tap.

═══ THE GLYPH ═══

Each condition has a small inline svg glyph drawn in the source. The glyphs are 24 by 24 pixels, drawn in cool blue strokes, with the sun yellow used only for the sun and the partly sunny glyphs (the sun behind the cloud). The glyphs are simple, four to eight paths each, no shading, no gradient. The glyph for the current condition appears at 56 by 56 pixels next to the big temperature number, the ribbon glyphs are 24 by 24. The glyphs are defined in src/components/Glyph.tsx as a single switch on the kind.

═══ THE EDGES ═══

A fetch that fails (no network, the api is down, the response is malformed) shows the cached value if any with a small italic 11px line at the bottom of the page reading the last refresh was 4 minutes ago, with the elapsed time updating every thirty seconds. The page never blanks the weather, the visitor always sees the most recent reading.

A fetch that succeeds but returns a current temperature below minus 60 or above 60 degrees celsius is treated as bad data, the cache is not updated, the page reads from the previous cached value, and a small italic line reads the upstream gave an implausible reading, retrying soon.

A visitor without internet at first load sees an empty page with a small italic 14px line reading no cached weather yet, the page is fetching for the first time. The page retries every fifteen seconds quietly until a fetch succeeds.

A visitor who declines geolocation and does not pick a city sees an empty page with the place prompt, the page does not infer location from any other signal.

═══ THE BUILD ═══

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding mist, slate dark, slate dark muted, sun yellow, cool blue. Vite as the build tool. State is plain useState and one useReducer for the place, the response, the fetch state, and the active ribbon cell. No router, no global store, no context provider, no weather library, no geocoding library, no svg library, no icon pack. fetch, AbortController, Cache, navigator.geolocation, Intl.DateTimeFormat, and CSS Anchor Positioning are all used directly. Every glyph is inline svg in the component that uses it.

Files. index.html with the Source Serif 4, Geist, and Geist Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the place, the response, the active ribbon cell, and the fetch controller. src/components/Condition.tsx for the top phrase. src/components/Temperature.tsx for the big number and the wind subscript. src/components/Ribbon.tsx for the twelve hour cells. src/components/HourDetail.tsx for the anchored panel. src/components/PlacePrompt.tsx for the location dialog. src/components/Glyph.tsx for the inline svg condition glyphs. src/lib/openmeteo.ts for the fetch wrapper and the AbortController. src/lib/cache.ts for the Cache api wrapper. src/lib/codes.ts for the weather code to condition kind table. src/lib/cities.ts for the small city dataset. src/lib/intl.ts for the Intl.DateTimeFormat helpers. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page on a laptop. The mist page paints, the place prompt sits at the top center. They tap allow once, accept the browser prompt. Within a second the page paints the condition phrase a quiet morning in Source Serif 4 at the top, the big number 14 with a small degree mark in Geist sun yellow below it, the twelve hour ribbon at the bottom with the current hour outlined. They hover the third cell of the ribbon, a small anchored panel appears showing partly sunny, wind from the southwest at twelve kilometers per hour. They close the tab. They reopen ten minutes later, the cached response paints immediately, a quiet background fetch refreshes it within a second.
