# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

GPUImage for Android is an Android image filter library that uses OpenGL ES 2.0 to apply real-time filters to images and videos. It's a port of the iOS GPUImage framework with matching vertex and fragment shaders.

**Requirements**: Android 2.2+ (OpenGL ES 2.0)

**Maven Coordinates**: `jp.co.cyberagent.android:gpuimage:2.x.x`

## Build Commands

```bash
# Build the library
./gradlew clean assemble

# Build the release variant
./gradlew assembleRelease
```

The library uses CMake for native code (YUV to RGBA conversion). The native module is at `library/src/main/cpp/`.

## Architecture

### Core Components

- **GPUImage** ([library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImage.java](library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImage.java)): Main entry point. Provides API for setting images, filters, cameras, and saving filtered images.

- **GPUImageRenderer** ([library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImageRenderer.java](library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImageRenderer.java)): Implements GLSurfaceView.Renderer. Handles texture loading, image scaling, rotation, and drawing pipeline.

- **GPUImageFilter** ([library/src/main/java/jp/co/cyberagent/android/gpuimage/filter/GPUImageFilter.java](library/src/main/java/jp/co/cyberagent/android/gpuimage/filter/GPUImageFilter.java)): Base class for all filters. Contains vertex/fragment shader strings and GL program management.

- **GPUImageView** ([library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImageView.java](library/src/main/java/jp/co/cyberagent/android/gpuimage/GPUImageView.java)): Custom View that combines GPUImage with GLSurfaceView/GLTextureView.

### Filter System

All filters extend `GPUImageFilter` and are located in [library/src/main/java/jp/co/cyberagent/android/gpuimage/filter/](library/src/main/java/jp/co/cyberagent/android/gpuimage/filter/). Each filter typically:
1. Defines custom vertex/fragment shaders
2. Overrides `onInit()` to get uniform locations
3. Overrides `onDrawArraysPre()` to set shader parameters via helper methods (`setFloat`, `setFloatVec2`, `setUniformMatrix4f`, etc.)

The `GPUImageFilterGroup` class chains multiple filters together.

### Rendering Pipeline

1. Image/Bitmap loaded via `GPUImage.setImage()` or `GPUImageRenderer.setImageBitmap()`
2. Textures created via `OpenGlUtils.loadTexture()`
3. Filter applied via `GPUImageFilter.onDraw()` which runs the GL program
4. Output rendered to GLSurfaceView or captured via PixelBuffer

### Native Code

The C library at `library/src/main/cpp/yuv-decoder.c` converts camera YUV preview data to RGBA using CMake build.

## Key Patterns

- **Thread Safety**: OpenGL operations must run on the GL thread. Use `GPUImageRenderer.runOnDraw()` to queue GL operations.
- **Filter Parameters**: Set parameters via `setFloat()`, `setFloatVec2()`, etc. - these queue tasks to run on the GL thread.
- **Scaling**: `GPUImage.ScaleType` controls image fitting (CENTER_CROP or CENTER_INSIDE).
- **Rotation**: Use `Rotation` enum from `jp.co.cyberagent.android.gpuimage.util`.

## Configuration

Version and SDK settings are in `library/gradle.properties` (VERSION_CODE, VERSION_NAME, COMPILE_SDK_VERSION, etc.).

## Notable Design Notes

- Filters use a queue-based task system to ensure thread-safe parameter updates on the GL thread
- Odd-width images require padding (handled in `GPUImageRenderer.setImageBitmap`)
- Camera preview uses deprecated Camera API with YUV data processed through native code