# Cremanic

<div align="center">

# 🎙️ CREMANIC
### Sovereign Browser-Native Neural Voice Studio & High-Fidelity Acoustic Synthesis

[![License: Apache 2.0](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://opensource.org/licenses/Apache-2.0)
[![Audio: 48kHz Stereo](https://img.shields.io/badge/Audio-48kHz%20Stereo-emerald.svg)](#acoustic-fidelity--audio-specifications)
[![Runtime: 100% Client--Side](https://img.shields.io/badge/Runtime-100%25%20In--Browser-violet.svg)](#the-zero-data-privacy-manifesto)
[![Engine: MOSS--TTS--Nano + Kokoro](https://img.shields.io/badge/Engine-MOSS--TTS--Nano%20%7C%20Lite-cyan.svg)](#dual-engine-architecture-matrix)
[![Languages: 20+](https://img.shields.io/badge/Languages-20%2B%20Supported-amber.svg)](#supported-multilingual-roster)
[![Zero-Shot Cloning](https://img.shields.io/badge/Voice%20Cloning-Zero--Shot%20Instant-rose.svg)](#zero-shot-custom-voice-cloning)

<br />

**Cremanic** is a client-side neural speech workstation that generates **48kHz broadcast-grade stereo voice synthesis** and **zero-shot custom voice cloning** entirely inside your browser tab. 

**Zero servers. Zero API keys. Zero audio leaves your device.**

</div>

---

## ⚡ Highlights & Innovations

- 🎧 **Studio Engine (MOSS-TTS-Nano)**: Client-side neural synthesis combining an autoregressive ~100M parameter global transformer with the ~20M parameter `MOSS-Audio-Tokenizer-Nano`, delivering true **48,000 Hz dual-channel stereo** output.
- ⚡ **Lite Engine (Fast Fallback)**: Lightweight Kokoro-82M engine delivering instantaneous 24kHz audio synthesis on lower-spec hardware or constrained mobile devices.
- 🧬 **Zero-Shot Voice Cloning**: Clone any voice in real-time from a 3–15 second microphone recording or audio file. No model fine-tuning or cloud processing required.
- 🌐 **20+ Supported Languages**: Multilingual speech synthesis with native SentencePiece tokenization and automated Pinyin-tone phonemization for Mandarin Chinese.
- 🛡️ **Complete Hardware Sovereignty**: Synthesis runs on your device's CPU/GPU via WebAssembly (SIMD + multi-threading) and WebGPU through `onnxruntime-web`.
- 🎛️ **Device-Capability Tiering & Auto-Fallback**: Intelligent hardware profiling, probe generation benchmarks, and automatic graceful fallback to Lite mode if resource limits are exceeded.
- 📦 **Compact INT8 Footprint**: Total runtime download of only **~231.3 MB**, cached permanently offline via `CacheStorage`.

---

## 🏛️ Dual-Engine Architecture Matrix

Cremanic features an adaptive dual-engine architecture designed to balance studio-grade acoustic fidelity with universal browser compatibility:

| Feature | 🎙️ **Studio Engine (MOSS-TTS-Nano)** | ⚡ **Lite Engine (Fallback)** |
|:---|:---|:---|
| **Architecture** | Global Transformer (~100M) + Audio Tokenizer (~20M) | Compact Feedforward Neural TTS (82M) |
| **Output Sample Rate** | **48,000 Hz Stereo (Dual-Channel)** | 24,000 Hz Mono (Single-Channel) |
| **Acoustic Quality** | Studio Broadcast / Master Grade | High Intelligibility / Standard Voice |
| **Voice Cloning** | **Zero-Shot Instant (from 3–15s clip)** | Not Supported (Curated Presets Only) |
| **Conditioning Method** | Discrete Acoustic Codebooks (`[T, 16]`) | Pre-calculated Acoustic Embeddings |
| **Execution Runtime** | `onnxruntime-web` (WASM SIMD + Multithreading / WebGPU) | `onnxruntime-web` (WASM SIMD) |
| **Runtime Download** | **~231.3 MB** (INT8 Quantized) | ~85 MB |
| **Permanent Cache** | `CacheStorage` (`cremanic-moss-onnx-v1`) | Browser HTTP / CacheStorage |
| **Hardware Gate** | Concurrency ≥ 2, Memory ≥ 2 GB, Fast Network | Any standard browser / low-end mobile |
| **Default Mode** | High-end Laptops, Desktops, Modern Tablets | Mobile devices, constrained hardware |

---

## 🧬 Zero-Shot Custom Voice Cloning

Cremanic brings zero-shot instant voice cloning into the browser without any remote server assistance:

```
[ Microphone / Audio File ]
            │
            ▼
[ Client-Side Audio Pre-flight Validation ]
   ├── Duration Enforcement (3.0s – 15.0s)
   ├── RMS Energy & Silence Gate (> 0.008 RMS)
   ├── Peak Normalization (-1 dBFS)
   └── Resampling to 48kHz Stereo
            │
            ▼
[ On-Demand Codec Encoder (WASM/SIMD) ]
   └── Generates Discrete Prompt Audio Codes: [T, 16]
            │
            ▼
[ Autoregressive Neural Generation Loop ]
   └── Synthesizes 48kHz stereo speech in the target timbre
            │
            ▼
[ Local Persistence in IndexedDB ]
   └── Voices survive session refreshes; 100% private to your browser
```

### Cloning Guardrails & Safety
1. **Duration Gate**: Enforces clips between 3.0s and 15.0s to guarantee sufficient acoustic context without exhausting client memory.
2. **Silence & Noise Detection**: Computes real-time Root Mean Square (RMS) energy. Recordings with RMS < 0.008 are rejected with actionable guidance.
3. **On-Demand Encoder**: The heavy acoustic encoder (`moss_audio_tokenizer_encode.onnx`, ~43.2 MB) is loaded **only** when cloning is initiated. Standard preset voices require **zero** runtime audio encoding.

---

## 🎛️ Intelligent Device-Capability Tiering

To prevent low-resource devices from crashing or freezing, Cremanic implements a multi-tiered capability gate before launching Studio mode:

```
                  [ App Launch / Hardware Check ]
                                 │
                 ┌───────────────┴───────────────┐
                 ▼                               ▼
       [ Static Hardware Gates ]       [ Connection Check ]
       • Concurrency < 2 ?             • Save-Data header active?
       • Device Memory < 2 GB?         • Effective connection 2G/3G?
                 │                               │
                 └───────────────┬───────────────┘
                                 │
                     Passes Static Gates?
                     ├── NO  ──► Auto-select Lite Engine (24kHz)
                     └── YES ──► [ Throwaway Generation Probe ]
                                       │
                                Run 5–8s Probe
                                ├── Timeout / Error ──► Default to Lite Engine
                                └── Success ──────────► Studio Mode Ready (48kHz)
```

### Resilience & Fail-Safe Mechanisms
- **Manual Override Warning**: Users on constrained devices who manually toggle into Studio mode receive an explicit safety dialog requiring confirmation before loading heavy weights.
- **Runtime Fallback**: If a Studio generation fails or times out in real-time, Cremanic automatically re-synthesizes the text via the Lite engine and presents a non-alarming notification banner with a one-click switch back.
- **Cached Profiles**: Probe benchmark results are stored in `localStorage` (`cremanic-capability-probe-v1`) to avoid re-running benchmark probes on every page load.

---

## 🌐 Supported Multilingual Roster

Cremanic's Studio engine supports speech generation across **20+ world languages**:

| Language Group | Supported Languages | Acoustic & Phonetic Processing |
|:---|:---|:---|
| **East Asian** | Mandarin Chinese (`zh`), Japanese (`ja`), Korean (`ko`) | SentencePiece WASM + `pinyin-pro` tone mapping for standard Mandarin |
| **Western Germanic** | English (`en-US`, `en-GB`), German (`de`) | SentencePiece subword tokenization with natural pacing |
| **Romance** | Spanish (`es`), French (`fr`), Italian (`it`), Portuguese (`pt`) | Acoustic prosody conditioning across European and Latin American cadences |
| **Slavic & Eastern European** | Russian (`ru`), Polish (`pl`), Czech (`cs`), Hungarian (`hu`) | Native phonetic token mapping |
| **Middle Eastern** | Modern Standard Arabic (`ar`), Persian / Farsi (`fa`), Hebrew (`he`) | Diacritized and non-diacritized tokenization handling |
| **Southern European & Other** | Greek (`el`), Turkish (`tr`), Danish (`da`), Swedish (`sv`) | Balanced European cadence modeling |

> **Note on Hindi**: Hindi is not included in the MOSS-TTS-Nano parameter tier. Support for Hindi requires the separate 31GB model, which is excluded from client-side browser deployment by design.

---

## 📊 Acoustic Fidelity & Audio Specifications

| Parameter | Cremanic Studio Engine | Standard Browser TTS / Cloud APIs |
|:---|:---|:---|
| **Sampling Rate** | **48,000 Hz** (48 kHz) | 16,000 Hz – 24,000 Hz |
| **Channels** | **2 Channels (True Stereo)** | 1 Channel (Mono) |
| **Bit Depth** | **16-bit PCM (Linear WAV)** | Compressed MP3 / Opus |
| **Spatial Imaging** | Yes (Stereo Soundstage) | Centered Mono |
| **Frequency Ceiling** | **24 kHz** (Full Audible Spectrum) | 8 kHz – 12 kHz (Muffled Highs) |
| **Voice Naturalness** | Conversational Intonation & Emotion | Robotic or Highly Compressed |
| **Latency** | Real-time streaming after warm-up | 500ms – 2,500ms cloud roundtrip |

---

## 🔒 The Zero-Data Privacy Manifesto

1. **100% In-Browser Execution**: All model weights, neural graph evaluations, and audio decoders execute directly in your browser's Web Worker thread.
2. **No Audio Leaves Your Machine**: When you record your voice for cloning or synthesize speech from text, 0 bytes of audio are sent over the network.
3. **No Account Required**: No logins, no tracking cookies, no subscription paywalls, no tokens.
4. **Permanent Local Caching**: Model weights are stored in your browser's dedicated `CacheStorage` so you can use the studio offline.
5. **Full Local Data Control**: Custom voice clones can be deleted with a single click from IndexedDB.

---

## 🛠️ Technology Stack (Under the Hood)

- **Neural Runtimes**: `onnxruntime-web` with WASM SIMD, multi-threading, and WebGPU backend.
- **Audio Synthesis**: MOSS-TTS-Nano 100M Global Transformer, MOSS-TTS-Nano Local Frame Sampler, MOSS-Audio-Tokenizer-Nano 48kHz Stereo Decoder.
- **Fallback Engine**: Kokoro-82M ONNX.
- **Phonetics & Text Processing**: SentencePiece WASM Sandbox, Pinyin-Pro, Web Audio API.
- **Client Storage**: IndexedDB (custom voices), CacheStorage (ONNX models), LocalStorage (tiering flags).

---

## 📄 License & Attribution

- **MOSS-TTS-Nano** & **MOSS-Audio-Tokenizer-Nano**: Developed by OpenMOSS / MOSI.AI, licensed under [Apache 2.0](https://opensource.org/licenses/Apache-2.0).
- **Kokoro**: Developed by Hexgrad, weights licensed under Apache 2.0.
- **Cremanic Showcase Portal**: Licensed under the Apache License, Version 2.0.

---

<div align="center">

**Cremanic — The Sovereign Browser-Native Neural Voice Studio**  
*Built for absolute privacy, acoustic precision, and zero-compromise client-side AI.*

</div>
