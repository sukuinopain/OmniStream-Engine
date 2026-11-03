# OmniStream

A spatio-temporal video grounding engine for natural language action localization in surveillance streams (evaluated on the VIRAT dataset).

---

## Technical Architecture & Implemented Components

### 1. Vision-Language Alignment
- **High-Resolution Backbone (`SigLIP-384` @ FP16):** Deployed `google/siglip-base-patch16-384` in half-precision to enhance spatial feature extraction on low-resolution and distant targets while reducing memory footprint.
- **Letterbox Context Preservation:** Replaced bounding-box stretching with a 22% contextual margin and uniform letterbox padding, preserving anatomical proportions during inference.
- **Calibrated Contrastive Scoring:** Eliminated hardcoded negative thresholds in favor of learned temperature scaling and relative Softmax ranking against scene background embeddings.

### 2. Tracking & Pipeline Optimization
- **Multi-Class Detection & Association:** Integrated `YOLOv8m` with `ByteTrack` configured across primary surveillance categories (pedestrians, vehicles, bicycles, and carried items).
- **Single-Pass In-Memory Caching:** Replaced disk/stream seek operations (`cap.set`) with an in-memory priority queue that stores optimal crops per track during the initial pass, cutting processing latency by >60%.
- **Kinetic Saliency Fallback:** Integrated differential motion contour tracking to retain spatial localization when bounding boxes drop out during physical contact or occlusion.

### 3. Analytical 8D EMA Kinematics
- **Exponential Moving Average (EMA) Filtering:** Attenuated high-frequency detector jitter on bounding-box coordinates prior to numerical differentiation ($\alpha = 0.30$).
- **8D State Vector:** Computes and tracks instantaneous position, dimensions, velocity, and acceleration per tube:
  $$\mathbf{s}_t = [x_c, y_c, w, h, v_x, v_y, a_x, a_y]^T$$
- **Net Normalized Displacement:** Evaluates total trajectory displacement relative to bounding-box diagonal scale to mathematically distinguish real movement from stationary vibration.

### 4. Dynamic Temporal Boundary Detection
- **Full-Width at Half-Maximum (FWHM) Slicing:** Replaced fixed temporal buffers with dynamic boundary estimation derived from the similarity curve profile around the peak timestamp.
- **Adaptive Kinetic-Semantic Weighting:** Automatically adjusts the fusion ratio between visual semantics and differential motion energy when motion predicates are detected in the query.

### 5. Serving & Visualization Infrastructure
- **Tactical HUD Rendering:** Generates trajectory history trails, instantaneous velocity vectors, and coordinate telemetry on target entities.
- **Web-Safe Transcoding Pipeline:** Standardized slice export to H.264 / `yuv420p` via FFmpeg and sandboxed caching within execution directories to prevent Gradio path and codec exceptions.
