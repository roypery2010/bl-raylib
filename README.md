# bl-raylib

Raylib 5.x–6.x bindings for the [Biscuit Language](https://biscuitlang.org/). Tested with Biscuit 0.15.0 and raylib 6.0.0.

This module follows Biscuit's `lib/bl/api/extra` module convention. It exposes
raylib's C ABI directly through `#extern`, with a small, practical starting API
for windows, timing, 2D drawing, keyboard, mouse, colors, and basic geometry.

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

This is an initial binding, not a generated copy of all raylib headers. More
raylib subsystems can be added without changing the module layout.
