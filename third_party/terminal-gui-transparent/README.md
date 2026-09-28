# Patched Terminal.Gui — transparent (default) background

podliner renders its UI through [Terminal.Gui](https://github.com/gui-cs/Terminal.Gui) `1.19.0`.
Terminal.Gui 1.x paints every cell with an explicit background colour, so on a
translucent / glass terminal (Omarchy "liquid glass", `alpha`/`background-opacity`,
etc.) the UI is an opaque block even though the rest of the terminal is see-through.

cmatrix fixes the same problem by calling ncurses' `use_default_colors()` and using
`-1` (terminal default) as the background. Terminal.Gui has no such concept, so this
patch adds it: whenever the background colour is `Color.Black`, the driver passes
`-1` to `init_pair()` instead of `COLOR_BLACK`, which makes the terminal draw its own
default background (including transparency/glass).

Everything else is untouched — coloured highlights (focus, progress bars, volume)
still paint normally.

## What is committed

- `default-background.patch` — the source change, against Terminal.Gui `v1.19.0`
  (`Terminal.Gui/ConsoleDrivers/CursesDriver/CursesDriver.cs`).
- `../../local-nuget/Terminal.Gui.1.19.0-transparent.nupkg` — the prebuilt package,
  picked up by the repository `NuGet.config`. `dotnet build` / `dotnet publish` work
  with no extra steps.

## Rebuilding the package (e.g. for a newer Terminal.Gui)

```bash
git clone --depth 1 --branch v1.19.0 https://github.com/gui-cs/Terminal.Gui.git /tmp/terminal.gui
cd /tmp/terminal.gui
git apply /path/to/default-background.patch
dotnet pack Terminal.Gui/Terminal.Gui.csproj -c Release \
  -p:TargetFrameworks=net8.0 -p:DebugType=None \
  -p:Version=1.19.0-transparent -p:PackageVersion=1.19.0-transparent \
  -o /path/to/podliner/local-nuget
```
