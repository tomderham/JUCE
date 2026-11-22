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
| `4d458dbb9a` | MIDI (Apple): Avoid constructing synthetic endpoints for devices already discovered through the system UMP endpoint manager |
| `27f89b5dd3` | macOS: Fix a bug dismissing modal alerts |
| `6975e67eac` | String: Fix a bug when comparing multi-byte characters |
| `f8a5c5d80f` | XmlDocument: Only load external entities through a caller-supplied InputSource |
| `3342d40911` | XmlDocument: Limit entity expansion and element nesting depth |
| `94410931f4` | XmlDocument: Fix double expansion of escaped references inside entity markup |
| `bf695ff985` | RectangleList: Avoid infinite loops when consolidating floating-point lists |
| `28432d8ed3` | Component: Retarget mouse events to the nearest surviving ancestor when the original target is deleted before delivery |
| `486906b640` | JSON: Limit the nesting depth of parsed objects and arrays |

### Changes to the upstream code
- **Timer: Synchronously stop the TimerThread when shutting down:** JUCE 9's `stopThread()`, which waits indefinitely, became `stopThread (-1)`, its JUCE 8 equivalent. (2026-10-15)
- **String: Fix a bug when comparing multi-byte characters:** the jassert it brings into `CharacterFunctions::compareUpTo()` was resolved as upstream has it. (2026-11-22)
- **XmlDocument and JSON hardening:** upstream's `BREAKING_CHANGES.md` entries for these weren't taken, as the file differs between JUCE 8 and 9. In short, an XmlDocument created from text or a stream no longer loads external entities unless given an InputSource, and documents and JSON nested deeper than the new limits fail to parse. (2026-11-22)

Apart from that, the cherry-picked commits are unchanged.

### Curve-specific fixes
These commits are not from upstream.

- **ModalComponentManager (macOS): Don't lower popup menus when raising modal windows:** `5fc8d3485d` moves the frontmost modal window to the modal panel level. Popup menus are modal and sit above that level, so each open submenu dropped behind its parent menu, which is visible when the two overlap. A window now keeps its level if that level is already higher. As of 2026-10-30, upstream `develop` still has the original behaviour; drop this commit if upstream fixes it.
- **VST3 Host: Only send setContentScaleFactor when the scale changes:** the host window sends the scale before and after attaching the view, and some plug-ins mishandle the repeat. As of 2026-10-30, upstream `develop` still sends the repeat; drop this commit if upstream fixes it.
- **AU Host: Allow auxiliary output buses to be disabled:** `isBusesLayoutSupported` rejected every disabled bus, so the Enabled toggle in a host's I/O configuration could never switch off the extra outputs of a multi-output instrument. A disabled output bus other than the main one now keeps the AU's stream format and is not rendered. As of 2026-10-30, upstream `develop` still rejects disabled buses; drop this change if upstream fixes it.
- **VST3 Host: Give inactive buses scratch buffers instead of nullptrs:** `HostBufferMapper` reported an inactive bus's full channel count but passed a nullptr for each channel, so a plug-in that writes to every bus without checking whether it is active crashed in its process call, e.g. an instrument with its main output disabled. Inactive buses now get scratch channels, sized in `prepare()` and cleared every block, and are still flagged as silent. Upstream's `HostBufferMapper` test expected the nullptrs and now expects silent scratch channels. As of 2026-11-05, upstream `develop` still passes nullptrs; drop this change if upstream fixes it.

- **AudioProcessorGraph: Don't pass MIDI through a suspended node:** a suspended node's audio was cleared but its MIDI buffer, which holds the events routed to it, passed on unprocessed. A host that suspends a node to mute it, as Curve does, couldn't stop MIDI going through it. Only note releases pass now, including those from the MIDI inputs when the MIDI Input node is suspended, so the nodes after it aren't left holding notes, and a node that takes MIDI and resumes first gets sustain off, all notes off and all sound off on every channel. Nodes without MIDI are left alone. It also adds `Node::setOutputMuted()`, which keeps a node processing, on silence and with only note releases for MIDI, releases the notes it's holding as it starts, and passes on nothing but note releases, so a host can let a muted node's tails ring out to nothing before suspending it, as Curve does. As of 2026-11-22, upstream `develop` still passes the MIDI through; drop this change if upstream fixes it.
- **AU Host: Implement reset():** `AudioProcessor::reset()` did nothing for an AU, so a host could only clear its tails and voices with `releaseResources()` and `prepareToPlay()`, which uninitialise and initialise it. It now resets the AU's buses and global scope, as `releaseResources()` does, under the callback lock as VST3's `reset()` is. As of 2026-11-22, upstream `develop` still has no `reset()` for AUs; drop this change if upstream adds one.

## Updating
When JUCE publishes a new 8.x tag, rebase this branch onto it, and drop any commit that the tag already contains.

## Licence
Unchanged from JUCE: dual-licensed under the AGPLv3 or the JUCE commercial licence (see [LICENSE.md](LICENSE.md)). Copyright notices in the source files are unchanged.
