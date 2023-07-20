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
canvas can indicate that its luminance should not be clamped.

## API proposal

Add the following new dictionaries.

```webidl
dictionary CanvasColorMetadata {
  bool extendedRange = false;
};

partial dictionary GPUCanvasConfiguration {
  CanvasColorMetadata colorMetadata;
};
```

When a canvas is created with a `GPUCanvasConfiguration` which specifies a
`colorMetadata` which specifies `extendedRange` as `true`, then colors will not
be clamped to the standard dynamic range, but rather will be clamped to full the
dynamic range of the display device.

## Non goals and future work

This proposal does not introduce Perceptual Quantizer (PQ) and Hybrid-Log Gamma
(HLG) transfer functions from ITU-R BT.2100 and ISO 22028-5 HLG. The natural
proposal to include this would update `PredefinedColorSpace` to include
`'rec2100-hlg'` and `'rec2100-pq'`.

This proposal does not introduce any of the metadata types discussed in ISO
22028-5. These metadata types include mastering display color volume (MDCV),
content colour volume (CCV) metadata, content light level (CLL), diffuse white
luminance metadata (NDWL), and reference viewing environment (REVE). The natural
proposal to include these would add new members to `CanvasColorMetadata`
corresponding to the various types of metadata.

This proposal does not introduce any non-trivial (clamping) tone mapping
behaviors. Additional tone mapping behaviors can be introduced by adding
corresponding new members to CanvasColorMetadata.

