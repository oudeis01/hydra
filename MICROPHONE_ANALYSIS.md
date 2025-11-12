# Microphone Functionality Analysis by Platform

## Summary
Based on code analysis, microphone functionality (getUserMedia) will work on all platforms but with different permission behaviors.

## How Hydra Uses Microphone

The hydra-synth library uses the following Web APIs:
- `navigator.mediaDevices.getUserMedia({ audio: true })` for microphone access
- `AudioContext` for audio processing
- `createMediaStreamSource()` to connect microphone stream
- Meyda library for FFT analysis (amplitude, frequency, etc.)

Location: `node_modules/hydra-synth/dist/hydra-synth.js`

## Platform-by-Platform Analysis

### Linux (Native Build)

**Status**: Should work with permission prompt

**Details**:
- WebView: WebKitGTK (uses Chromium backend)
- Permission Handling: Browser-like permission prompt via WebKitGTK
- getUserMedia: Fully supported
- No additional Tauri configuration required

**User Experience**:
1. User calls `s0.initAudio()` in Hydra
2. System shows permission dialog: "Allow app to use your microphone?"
3. User clicks "Allow"
4. Microphone starts working

**Potential Issues**:
- PulseAudio/PipeWire must be properly configured
- User must grant permission via system dialog

**Testing Commands**:
```bash
npm run tauri:dev    # Test in dev mode
npm run tauri:build  # Build for production
```

---

### Windows (Cross-compiled)

**Status**: Should work with permission prompt

**Details**:
- WebView: WebView2 (Microsoft Edge Chromium engine)
- Permission Handling: WebView2 provides browser-like permission prompts
- getUserMedia: Fully supported in WebView2
- No additional Tauri configuration required

**User Experience**:
1. User calls `s0.initAudio()` in Hydra
2. WebView2 shows permission prompt: "Allow this app to use your microphone?"
3. User clicks "Allow"
4. Microphone starts working

**Potential Issues**:
- WebView2 Runtime must be installed (usually pre-installed on Windows 10+)
- User must grant permission via prompt

**Build Target**: `x86_64-pc-windows-gnu`

---

### macOS (If built)

**Status**: Should work BUT requires additional configuration

**Details**:
- WebView: WKWebView (Safari engine)
- Permission Handling: macOS requires Info.plist entry for microphone
- getUserMedia: Supported in WKWebView
- **REQUIRES**: Additional configuration in tauri.conf.json

**Required Changes for macOS**:

Add to `src-tauri/tauri.conf.json`:

```json
{
  "bundle": {
    "macOS": {
      "entitlements": null,
      "exceptionDomain": null,
      "frameworks": [],
      "providerShortName": null,
      "signingIdentity": null,
      "infoPlist": {
        "NSMicrophoneUsageDescription": "This app needs microphone access for audio-reactive visuals."
      }
    }
  }
}
```

**User Experience**:
1. First time: macOS system dialog asks for microphone permission
2. User must allow in System Preferences > Security & Privacy > Microphone
3. After permission granted, works normally

**Potential Issues**:
- Without `NSMicrophoneUsageDescription`: App will crash when requesting microphone
- User might deny permission initially

---

## Current Tauri Configuration Status

### What's Already Configured
```json
{
  "security": {
    "csp": null,  // No Content Security Policy blocking media APIs
    "assetProtocol": {
      "enable": true,
      "scope": ["**"]
    }
  }
}
```

### What's NOT Configured
- No microphone-specific permissions in `capabilities/default.json`
- No macOS Info.plist microphone usage description

### Why It Still Works (on Linux/Windows)
Tauri v2's WebView delegates media permissions to the underlying WebView engine:
- **Linux**: WebKitGTK handles `getUserMedia` natively
- **Windows**: WebView2 handles `getUserMedia` natively
- **No Tauri-level permission needed** because it's a browser API

---

## Recommendations

### For Linux/Windows Users
No changes needed. Microphone should work out of the box with permission prompt.

### For macOS Support (Future)
Add the macOS Info.plist configuration shown above.

### For Testing
To verify microphone functionality:

```javascript
// In Hydra console

// 1. Show audio monitor
a.show()

// 2. Set FFT bins
a.setBins(6)

// 3. Test audio reactivity
osc(10, 0, () => a.fft[0] * 4).out()

// Or simpler test:
solid(1, 0, 0).mult(osc(10, 0, () => a.fft[0] * 4)).out()
```

Expected behavior:
- Permission prompt appears on first use
- Visual should react to audio/microphone input
- `a.fft` array should show changing values

---

## Technical Details

### Web APIs Used by Hydra
```javascript
// From hydra-synth source code:

// 1. Request microphone access
navigator.mediaDevices.getUserMedia({ 
  video: false, 
  audio: true 
})

// 2. Create audio context
const context = new AudioContext()

// 3. Create media stream source
const audioStream = context.createMediaStreamSource(stream)

// 4. Analyze with Meyda
Meyda.createMeydaAnalyzer({
  audioContext: context,
  source: audioStream,
  bufferSize: 512,
  featureExtractors: ['amplitudeSpectrum'],
  // ...
})
```

### Tauri v2 Permission Model
Tauri v2 uses a capability-based permission system (`capabilities/default.json`), but:
- **Browser APIs** (like getUserMedia) are **NOT controlled by Tauri capabilities**
- They are controlled by the **WebView engine's permission system**
- Tauri capabilities only control **Tauri-specific APIs** (fs, dialog, etc.)

---

## Conclusion

**Linux**: ✅ Should work (with permission prompt)  
**Windows**: ✅ Should work (with permission prompt)  
**macOS**: ⚠️ Requires Info.plist configuration

The microphone functionality relies entirely on standard Web APIs that are supported by all modern WebView engines. No additional Tauri plugins or Rust code is needed.
