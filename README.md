# bl-raylib

Raylib 5.x–6.x bindings for the [Biscuit Language](https://biscuitlang.org/). Tested with Biscuit 0.15.0 and raylib 6.0.0.

This module follows Biscuit's `lib/bl/api/extra` module convention. It exposes
raylib's C ABI directly through `#extern`, including windows, timing, 2D drawing,
keyboard, mouse, colors, basic geometry, and the complete public audio API.

## Requirements

- Biscuit Language 0.15.x (`blc`; earlier versions may work but are not tested)
- raylib 5.x or 6.x installed and discoverable by the platform linker
- A desktop platform supported by raylib

The repository does **not** vendor raylib. This keeps the binding small and
lets applications choose a raylib build appropriate for their graphics backend.

## Module layout

```text
extra/raylib/module.yaml # canonical module metadata
extra/raylib/raylib.bl   # canonical C-ABI declarations
build.bl
examples/basic.bl
```

The repository already includes a checkout-local copy at `extra/raylib/`, so
`blc -build` can resolve the example without a global install. The build script
uses `set_module_dir(exe, ".")` to search the checkout first.

For a system-wide install, copy or symlink `extra/raylib/` to:

```text
<bl-install>/lib/bl/api/extra/raylib
```

Then import it from Biscuit code:

```bl
raylib :: #import "extra/raylib";
```

## Example

```bl
#import "std/string"
raylib :: #import "extra/raylib";

main :: fn () s32 {
raylib.InitWindow(800, 450, strtoc("Hello from Biscuit"));
defer raylib.CloseWindow();
raylib.SetTargetFPS(60);
loop !raylib.WindowShouldClose() {
raylib.BeginDrawing();
raylib.ClearBackground(raylib.RAYWHITE);
raylib.DrawText(strtoc("Hello, bl-raylib!"), 190, 200, 20, raylib.DARKGRAY);
raylib.EndDrawing();
}
return 0;
}
```

From this directory, build the example with:

```sh
blc -build
```

Run that command from the repository root. Biscuit's build pipeline discovers
`build.bl` in the current directory, and the local module manifest supplies the
raylib linker configuration.

## Audio

The binding includes raylib's audio device, wave, sound, music, and raw audio
stream APIs, including validity checks, memory loading, wave conversion/export,
sound aliases, playback controls, stream callbacks, and stream/mixer processors.
Audio data types mirror raylib's C layouts:

```bl
raylib.InitAudioDevice();
sound := raylib.LoadSound(strtoc("sound.wav"));
if raylib.IsSoundValid(sound) {
    raylib.SetSoundVolume(sound, 0.8f);
    raylib.PlaySound(sound);
    loop raylib.IsSoundPlaying(sound) {
        raylib.WaitTime(0.01);
    }
    raylib.UnloadSound(sound);
}
raylib.CloseAudioDevice();
```

`Wave.data`, `Music.ctxData`, `AudioStream.buffer`, and
`AudioStream.processor` use `*u8` as raw/opaque pointers. `AudioCallback` is
declared as `*fn (bufferData: *u8, frames: u32)` for stream and mixer callback
registration. Callback functions must follow that signature and obey raylib's
audio-thread safety requirements; callback registration is an advanced FFI use
case and should be checked against the compiler's function-pointer conversion
rules.

## Platform notes

- Linux links `raylib`, OpenGL, X11, and the usual system libraries.
- Windows expects `raylib.lib` plus `winmm`, `gdi32`, and `opengl32`.
- macOS links raylib and the Cocoa/OpenGL/IOKit/CoreVideo frameworks.

If raylib is installed in a non-standard location, pass the appropriate library
path through your local Biscuit/toolchain configuration. The module uses raw
linker options, so both static and shared raylib installations are supported
as long as the linker can resolve the listed libraries.

## Scope and ABI notes

The binding is intentionally direct: function names match raylib symbols, and
`Color`, `Vector2`, and `Rectangle` mirror raylib's C structs. Text parameters use `*s8`, matching Biscuit's `strtoc` C-string helper and
raylib's C `char *`; call `strtoc` for string literals so they are
NUL-terminated before crossing the FFI boundary. Raylib boolean results use Biscuit's `bool` type, matching raylib's C `bool`
return values and allowing them to be used directly in conditions.

This binding is not a generated copy of all raylib headers, but the public
raylib audio API is included alongside the initial window, drawing, input, and
geometry surface. More raylib subsystems can be added without changing the
module layout.
