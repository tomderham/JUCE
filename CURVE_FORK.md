# JUCE for Curve (unofficial fork)

This is an **unofficial fork** of [JUCE](https://github.com/juce-framework/JUCE), maintained for [Curve](https://github.com/tomderham/curve). It is not affiliated with or endorsed by Raw Material Software Limited (JUCE).

## Branch `curve/8.0.16`
Upstream tag [`8.0.16-security`](https://github.com/juce-framework/JUCE/tree/8.0.16-security), plus fixes cherry-picked from JUCE 9.0.4 and later upstream `develop` commits that are relevant to Curve on macOS: an audio plug-in host using VST3 and AU, CoreMIDI, network downloads and modal dialogs. Fixes for other platforms and new features are not included, and neither is JUCE 9's CoreAudio rewrite.

Each commit is a cherry-pick of the upstream commit listed below, which its commit message records as `(cherry picked from commit …)`, apart from the Curve-specific commits listed under [Curve-specific fixes](#curve-specific-fixes).

| Upstream commit | Change |
|---|---|
| `79c98faddd` | CoreMIDI: Create a single client per application |
| `3fa28d0a14` | VST3 Host: Set host context on the factory before matching class indices |
| `a18ae60e46` | VST3 Host: Avoid updating client with single-precision param values after receiving a double-precision performEdit call |
| `08c66825ff` | XmlDocument: Avoid inifinite loop when parsing document with a malformed header |
| `1c66556aed` | Network (macOS): Fix bug where cancel() wouldn't interrupt a connection that was still starting |
| `08ae49ec10` | Network (macOS): Fix bug where cancel() wouldn't interrupt a blocking read |
| `5fc8d3485d` | ModalComponentManager (macOS): Raise modal windows to the modal window level |
| `44632f19ec` | ModalComponentManager (macOS): Avoid sending mouse clicks to child views blocked by modal components |
| `3ac433b3fd` | ObjCClass: Stop disposing of the allocated class pairs |
| `8be8d27de5` | Timer: Don't post a message if the thread should be exiting |
| `23d329bb02` | Timer: Synchronously stop the TimerThread when shutting down |
| `f5b3b01f41` | Timer: Prevent data race |
| `c59fd49324` | MessageManager (macOS): Avoid posting to the system queue during shutdown |
| `5912be57d4` | TableListBox: Fix potential crash in destructor |
| `1ffaca0c60` | Thread (macOS): Fix a failed realtime start leaving the thread running |
| `3d9fde8db0` | MemoryBlock: Prevent calling memcmp with a nullptr |
| `3ac805ae41` | OutputStream: Prevent calling strlen with a nullptr |
| `1c4d38c48d` | MemoryBlock: Prevent leaving a MemoryBlock in an inconsistent state |
| `90db407245` | VST3 Host: Avoid calling checkSizeConstraint on non-resizable views (upstream `develop`, after 9.0.4) |

### Changes to the upstream code
- **Timer: Synchronously stop the TimerThread when shutting down:** JUCE 9's `stopThread()`, which waits indefinitely, became `stopThread (-1)`, its JUCE 8 equivalent. (2026-10-15)

Apart from that, the cherry-picked commits are unchanged.

### Curve-specific fixes
These commits are not from upstream.

- **ModalComponentManager (macOS): Don't lower popup menus when raising modal windows:** `5fc8d3485d` moves the frontmost modal window to the modal panel level. Popup menus are modal and sit above that level, so each open submenu dropped behind its parent menu, which is visible when the two overlap. A window now keeps its level if that level is already higher. As of 2026-10-30, upstream `develop` still has the original behaviour; drop this commit if upstream fixes it.
- **VST3 Host: Only send setContentScaleFactor when the scale changes:** the host window sends the scale before and after attaching the view, and some plug-ins mishandle the repeat. As of 2026-10-30, upstream `develop` still sends the repeat; drop this commit if upstream fixes it.

## Updating
When JUCE publishes a new 8.x tag, rebase this branch onto it, and drop any commit that the tag already contains.

## Licence
Unchanged from JUCE: dual-licensed under the AGPLv3 or the JUCE commercial licence (see [LICENSE.md](LICENSE.md)). Copyright notices in the source files are unchanged.
