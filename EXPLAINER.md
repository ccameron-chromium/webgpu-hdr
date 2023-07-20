# WebGPU Extended Range Brightness

## Introduction

This proposal introduces a mechanism through which WebGPU can draw colors
brighter than white (`#FFFFFF`).

## Background

### Existing WebGPU Behavior

WebGPU can currently create a canvas with a `GPUTextureFormat` of
"rgba16float". When this is done, values that are outside of the [0, 1]
interval are [clamped in luminance but not in chrominance](https://www.w3.org/TR/webgpu/#canvas-color-space).

### High dynamic range headroom of a display

A display is considered high dynamic range (HDR) if it can display colors
brighter than white (#FFFFFF). The HDR capability of a display is parameterized
by its HDR headroom, which defined as the ratio between the maximum brightness
that the display can currently produce to the brightness of white.

A display which is not HDR (has an HDR headroom of 1) is called standard dynamic
range (SDR).

### High dynamic range headroom of content

The HDR headroom of content is the ratio between the maximum brightness of the
content to the brightness of white in that content.

## Goal

The goal of this proposal is to provide a mechanism through which a WebGPU
canvas can indicate that its luminance should not be clamped.

## API proposal

Add the following new dictionaries.

```webidl
dictionary ExtendedRangeMetadata {
  required float headroom;
};

dictionary CanvasColorMetadata {
  ExtendedRangeMetadata extendedRange;
};

partial dictionary GPUCanvasConfiguration {
  CanvasColorMetadata colorMetadata;
};
```

When a canvas is created with a `GPUCanvasConfiguration` that specifies a
`colorMetadata` which specifies an `extendedRange`, colors that have a linear
value less than `headroom` and are capable of being displayed by the display
device should not be clamped in luminance.

If the display device cannot display brightness up to the specified `headroom`,
then all colors that are outside of the capabilities of the display device will
be clamped to the displayable color range of the display device. All colors in
the canvas that are within the capabilities of the display device should not be
altered.

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

