
# Touch Tilt FM iOS

This project is a mobile-first browser instrument that uses multitouch and device tilt to control a three-voice FM synthesizer in Csound WebAssembly.

## Goals
This project integrates real-time touch interaction and motion sensors to control a three-voice FM synth:

1. Load and initialize the Csound WASM engine  
2. Compile a polyphonic FM synthesis instrument  
3. Handle multitouch input to control pitch and amplitude  
4. Use device tilt (via DeviceOrientation) to modulate FM depth and ratio  
5. Keep the FM range stable enough for iOS playback without harsh foldover noise

## CsoundObj API Usage
This app uses the following CsoundObj methods:

1. `Csound()` – Create a new Csound engine object  
2. `.setOption()` – Configure real-time audio and buffer sizes  
3. `.compileOrc()` – Compile FM synth instrument code  
4. `.start()` – Start the audio engine  
5. `.inputMessage()` – Schedule a persistent instrument (`i1 0 -1`)  
6. `.setControlChannel()` – Dynamically control FM parameters  

## JavaScript Overview
All application logic is contained in the `<script type="module">` section of `index.html`. No external build tools or frameworks are required. This includes:

- Asynchronous initialization of Csound  
- UI event listeners for touch and slider inputs  
- Motion permission and sensor event handling  

### Touch Handling
```javascript
pad.addEventListener("touchstart", handleTouch);
pad.addEventListener("touchmove", handleTouch);
pad.addEventListener("touchend", handleTouchEnd);
```
Each touch is mapped to a fixed musical scale (Y axis → pitch) and X axis (amplitude). Touches are assigned to one of three voices. When the touch ends, the corresponding amplitude fades out smoothly via `portk()`.

### Tilt Mapping
```javascript
const gammaNorm = Math.min(Math.abs(gamma) / 75, 1);
const betaNorm = (beta + 90) / 180;
const fmIndex = gammaNorm * 0.55;
const ratios = [0.5, 0.75, 1, 1.25, 1.5, 2, 2.5];
const fmRatio = ratios[Math.min(Math.floor(betaNorm * ratios.length), ratios.length - 1)];
```
- Gamma (left/right tilt) controls FM **index**  
- Beta (forward/backward tilt) selects one of several moderated FM **ratios**

### Csound Instrument
```csound
instr 1
  kamp1 = portk(chnget:k("amp1"), krelease)
  kfreq1 = chnget:k("freq1")
  ...
  aleft1, aright1 Voice kamp1, kfreq1, 0.18
  aleft = aleft1 + aleft2 + aleft3
  aleft = tanh(aleft * 1.35) * 0.72
```
The modulation ratio is dynamically linked to the carrier, but the ratio and index are clamped before synthesis. The synth uses `foscil`, low-pass filtering, DC blocking, lower per-voice gain, and soft limiting to reduce the noisy behaviour caused by extreme tilt-driven FM deviation.

### Stereo Panning
```csound
kpan1 = 0.18
kpan2 = 0.5
kpan3 = 0.82
```
The voices are placed across a stable stereo field. This avoids extra random movement in the signal path while keeping the multitouch layers spatially separated.

## HTML Layout
```html
<div id="starter">
  <button id="startAllBtn">Start Synth + Tilt</button>
  <input id="releaseSlider" type="range" min="0.01" max="0.3" />
</div>
<div id="xy-pad"></div>
```
A button starts the synth and requests motion permissions. A slider controls the release smoothing time for amplitude changes.

## Permissions
iOS requires:
- **User gesture** to enable motion sensing (via `DeviceOrientationEvent.requestPermission()`)  
- **HTTPS origin** for motion access

These are handled as soon as the user presses the start button.

## Deployment
You can host this app as a static website:

### GitHub Pages
1. Push your files to a new repo  
2. Go to **Settings > Pages** and choose the main branch root  
3. Visit `https://your-username.github.io/your-repo-name`  

## Testing & Debugging
Open the browser console to test real-time values and issue direct Csound messages:
```js
csound.inputMessage("i1 0 1")
csound.setControlChannel("fmIndex", 0.5)
```

## Conclusion
This app is a compact iOS-oriented FM instrument powered by WebAssembly and gesture input. You can touch and tilt to explore harmonics, motion-based modulation, and spatial sound from a single browser page.
