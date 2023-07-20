# WebGPU Extended Range Brightness

## Introduction

This proposal introduces a mechanism through which WebGPU can draw colors
brighter than `white` (`#FFFFFF`).

## Background

## Existing WebGPU Behavior

WebGPU can currently create a canvas with a `GPUTextureFormat` of
"rgba16float". When this is done, values that are outside of the [0, 1]
interval are clamped to the [standard dynamic range of the display device](https://www.w3.org/TR/webgpu/#canvas-color-space).

## Goal

The goal of this proposal is to provide a mechanism through which a WebGPU
canvas can indicate that its colors should not be clamped to standard
dynamic range.

## API proposal

Add the following new dictionaries.

```webidl
enum CanvasColorMode {
  "default",
  "extended",
};

dictionary CanvasColorMetadata {
  CanvasColorMode mode = "default";
};

partial dictionary GPUCanvasConfiguration {
  CanvasColorMetadata colorMetadata;
};
```

When a canvas is created with a `GPUCanvasConfiguration` which specifies a
`colorMetadata` which specifies a `mode` of `"extended"`, then colors will not
be clamped to the standard dynamic range, but rather will be clamped to full the
dynamic range of the display device.

## Example use

The following code would configure the canvas for high dynamic range, using the
Display P3 color space.

```
  context.configure({
    device: device,
    format: "rgba16float",
    usage: GPUTextureUsage.RENDER_ATTACHMENT,
    alphaMode: "opaque",
    colorSpace: "display-p3",
    colorMetadata: { mode:"extended" }
  });
```

## Notes on potential future metadata extensions

This proposal does not introduce Perceptual Quantizer (PQ) and Hybrid-Log Gamma
(HLG) transfer functions from ITU-R BT.2100 and ISO 22028-5. The natural
proposal to include this would update `PredefinedColorSpace` to include
`"rec2100-display-linear"` (for use with `"rgba16float"` formats) and
potentially `"rec2100-hlg"` and `"rec2100-pq"` (for use with `"rgb10a2unorm"`
formats, should they become available).

This proposal does not introduce any of the standard HDR metadata types.
These include the mastering display color volume (MDCV, from SMPTE ST
2086:2014), content light level information (CLLI, from CTA 861.3),
content color volume (CCV), nominal diffuse white luminance (NDWL),
and reference viewing environment (REVE).

The natural proposal to include these would add new members to
`CanvasColorMetadata` corresponding to the various types of metadata.

This proposal does not introduce any non-trivial (clamping) tone mapping
behaviors. Additional tone mapping behaviors can be introduced by adding
corresponding new members to CanvasColorMode.

For example, to specify HDR10 tone mapping with a default MDCV metadata
and with CLLI metadata that indicates an average of 200 nits and maxmium of
1200 nits, one could specify:

```
  context.configure({
    ...
    colorSpace: "rec2100-display-linear",
    colorMetadata: { mode:"hdr10", clli:{maxFALL:200, maxCLL:1200} }
  });
```

## Notes on analogous WebGL proposal

For WebGL, the API would be shaped similarly to how the color space
is specified (the `drawingBufferColorSpace` attribute). The following
shows how to configure the canvas the same way as is done in the WebGPU
example.

```
  gl.drawingBufferStorage(gl.RGBA16F,
                          gl.drawingBufferWidth,
                          gl.drawingBufferHeight);
  gl.drawingBufferColorSpace = "display-p3";
  gl.drawingBufferColorMetadata = { mode:"extended" };
```

