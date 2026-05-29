---
title: Prime Sieve
app_type: prime-sieve
wallet: 0x512ebDE3650630120cdbA40d2C0F213d682d1d31
---

A small page that finds every prime up to a number you give it. The visitor types a target, presses sieve, and a faint grid lights up with the primes inside. The whole sieve runs inside a WebAssembly module shipped in the source as a small array of bytes, instantiated at page boot. The same sieve also runs in plain JavaScript, side by side, so the visitor can see how much faster the wasm path is for any given target. There is no server. There is no library beyond what the standard browser ships.

## the why

A chalk colored page, the kind of dark you find inside a math classroom at the end of the day after the boards have been wiped. Slate green for the type, paper yellow reserved for the small primes that light up inside the grid, indigo for the active button and the timing numbers. Lora serif at 18px for body, at 22px italic for the small inline math expressions, weights 400 and 500. Sometype Mono at 12px for the numbers in the grid and the timing readouts. No icons except a small inline svg of a chalkboard eraser in the top right corner of the page, fourteen pixels across, two paths. The page is one column at most 760px wide, centered, 24px gutter on phones. Spacing is generous.

This page exists because finding primes is the cleanest small piece of code anyone has ever written and because the difference between writing the sieve once in JavaScript and once in WebAssembly is exactly the kind of small honest measurement that helps a person decide when to reach for wasm. The page is not a benchmark, it is not a guide, it is one example of one algorithm run twice. The visitor takes whatever they want from it.

A sieve of Eratosthenes works like this. Start with a flat array of booleans of size N plus one, all true. Set zero and one to false. Walk from two up. For each true index, mark every multiple of that index above it as false. When you reach the square root of N, stop. The indices that remain true are the primes. The whole thing is a few short loops. It is a hundred lines in any language, including the comments.

The interesting part for this page is that the inner mark loop is the kind of loop where wasm earns its keep. The JavaScript engine has to handle the array as a typed array of uint8 and do a bounds check on every write, which is fast on modern engines but not as fast as a wasm module writing into a shared linear memory with no bounds check beyond the page boundary. For a target of ten million, the difference is around two to four times faster on most laptops. For a target below a hundred thousand the wasm cost of instantiation wipes out the gain. The page lets the visitor watch this break even point happen by trying different targets.

The shape of one prime in the grid, written into the page, is a small Sometype Mono number in a 36 by 36 cell. The whole grid is laid out in rows of fifty cells, with the row number at the start of each row in slate green at thirty percent opacity. Primes light up in paper yellow with the number in indigo, composite numbers stay in slate green at twenty percent opacity. The grid scrolls vertically when it overflows the viewport, with a sticky header showing the target and the current count of primes found.

A small block of the wasm module's source, written as a comment in src/lib/wat.ts, looks roughly like this. The actual module ships as a byte array (a Uint8Array of around three hundred bytes), the wat text is here so the page makes its own work readable.

```
(module
  (memory (export "mem") 16)
  (func (export "sieve") (param $n i32)
    (local $i i32) (local $j i32) (local $sqrt i32)
    (local.set $sqrt (i32.trunc_f32_s (f32.sqrt (f32.convert_i32_s (local.get $n)))))
    (local.set $i (i32.const 2))
    (block $break (loop $continue
      (br_if $break (i32.gt_s (local.get $i) (local.get $sqrt)))
      (if (i32.load8_u (local.get $i))
        (then
          (local.set $j (i32.mul (local.get $i) (local.get $i)))
          (block $stop (loop $mark
            (br_if $stop (i32.gt_s (local.get $j) (local.get $n)))
            (i32.store8 (local.get $j) (i32.const 0))
            (local.set $j (i32.add (local.get $j) (local.get $i)))
            (br $mark)))))
      (local.set $i (i32.add (local.get $i) (i32.const 1)))
      (br $continue))))
```

The compiled bytes live inside src/lib/sieve.wasm.ts as a typed array literal. The App calls WebAssembly.instantiate with the byte array on boot, no fetch, no compile step from text, the page does not depend on a wat assembler at runtime. The exported sieve function takes the target, walks the memory, and returns when finished. The exported memory is read back by the App to color the grid.

## the how

The page has three regions stacked vertically.

The first region is the input bar. A serif label reads find every prime up to in 16px Lora. A single number input in 22px Sometype Mono indigo, with a default value of one thousand. The input accepts integers from 2 up to one billion. Above one hundred million the page shows a small italic 12px line saying the grid will only render the first hundred thousand primes, the rest are counted but not drawn. To the right of the input, a single text button in 14px Lora 500 indigo, reads sieve.

The second region is the timing strip. Two thin horizontal bars side by side, each 12px tall, labelled wasm and javascript in 11px Sometype Mono slate green. When a sieve runs, both bars start at zero width, the wasm bar reaches its full width when the wasm sieve completes, the javascript bar reaches its full width when the javascript sieve completes. The bars are normalized to the slower of the two runs, so the visitor immediately sees which path was faster and by how much. Below the bars, two small lines in Sometype Mono 12px showing the actual milliseconds for each path, formatted to one decimal place. A third line below in Lora 13px italic reads the ratio in plain text, like wasm ran 2.7 times faster than javascript.

The third region is the grid. A large vertically scrolling grid of cells, fifty cells per row, each cell 36 by 36 pixels, with the integer rendered in 11px Sometype Mono. Primes are paper yellow with indigo text, composites are slate green at twenty percent opacity. A sticky header at the top of the grid shows the target, the count of primes found, the index of the largest prime found, and a tiny text button reading copy the primes which writes the primes to the clipboard as a comma separated list. Tapping any prime in the grid pops a tiny tooltip in Lora 13px showing whether it is a twin prime (a prime with a sibling at plus or minus two), a Sophie Germain prime (a prime p where 2p plus 1 is also prime), or simply a prime with no extra label.

The sieve runs sequentially, wasm first, javascript second, with a tiny pause between them so the timing measurements are not contaminated by gc pressure from the previous run. Both paths run on the main thread, which keeps the comparison fair. The timing uses performance.now around the actual sieve call, not around the rendering, so the bars and numbers reflect the algorithm time only. Rendering the grid uses requestAnimationFrame to paint cells in chunks of two thousand at a time, so a target of one million does not lock the page during the paint.

A small line at the bottom of the page in Lora 13px italic reads the wasm path uses an exported i32 memory page, the javascript path uses a Uint8Array of the same size, the algorithms are otherwise identical. A second line beside it reads the wasm module ships as a 318 byte literal inside src/lib/sieve.wasm.ts, written by hand from the wat text above and verified at boot with a tiny self check that asks the module for the primes up to fifty and compares the result against a hardcoded list.

The visitor's last target, the timing of the most recent successful sieve, and the active path (wasm or javascript or both) are saved to localStorage under the key prime.sieve.v1. The localStorage write happens on idle, not on every keystroke. On boot the App reads the saved values, renders the most recent grid from a cached compact form, and restores the input field. If storage is unavailable the page falls back to memory and a small italic 11px line at the bottom right reads this run is not being kept on this device.

If WebAssembly is unavailable (the only browsers without it are very old), the page falls back to a javascript only mode, the timing strip shows only the javascript bar, the wasm timing line reads webassembly is not available in this browser. The sieve still runs. The grid still renders.

The sieve memory page is initialized with 0x01 (true) for every byte at sieve start, the wasm module exports a small fill function the App calls before each run, which avoids relying on host side initialization. After a sieve the memory is read with a single Uint8Array view over the exported buffer, walked once to collect prime indices into a flat array, and that array is what the grid renders. For very large targets, the collection step uses a small typed array growth strategy to avoid array reallocation pressure. The collection time is included in the displayed timing for both paths so the comparison stays honest.

The build is a single Vite project. React 18 with TypeScript. Tailwind CSS for layout and color, custom palette in tailwind.config.ts adding chalk, slate green, slate green muted, paper yellow, indigo. Vite as the build tool. State is plain useState and one useReducer for the run state machine (idle, sieving, done). No router, no global store, no context provider, no math library, no benchmark library, no icon pack. WebAssembly is used directly. performance.now is used directly. Every glyph including the eraser and the grid cells is inline svg or plain text in the component that renders it.

Files. index.html with the Lora and Sometype Mono links in the head. src/main.tsx as the React entry. src/App.tsx exporting one default component holding the target, the run state, the timing, and the prime list. src/components/InputBar.tsx for the top region. src/components/TimingStrip.tsx for the two bars and the readouts. src/components/Grid.tsx for the large grid of cells. src/components/Tooltip.tsx for the small twin and Sophie Germain pop. src/lib/sieve.wasm.ts holding the compiled wasm bytes as a typed array literal. src/lib/wat.ts holding the wat text as a comment for reference. src/lib/sieve.ts holding the WebAssembly.instantiate call, the run wrapper, and the javascript sieve. src/lib/primes.ts holding the twin and Sophie Germain checks. src/lib/storage.ts for the localStorage try catch. tailwind.config.ts. package.json with react, react dom, vite, tailwind only. README.md with two short paragraphs.

A first time visitor opens the page in chrome on a laptop. The chalk page paints, the input reads 1000, the empty grid sits below. They press sieve. The wasm bar fills almost instantly, the javascript bar fills a moment later, the ratio line reads wasm ran 1.6 times faster than javascript. The grid lights up with 168 primes in paper yellow. They change the target to 100000, press sieve again, the ratio jumps to about 2.4 times faster, the grid renders ten times more cells. They tap the prime 11, the tooltip reads twin prime with 13. They reload the page, the same target and the same grid wait for them.
