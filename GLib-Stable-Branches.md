This is the page for handling WebKitGTK and WPEWebKit stable branches.
We should merge not overly intrusive patches that improve stability or
performance, fix build issues, etc.

# 2.44

## Information

* Branch: https://github.com/WebKit/WebKit/commits/webkitglib/2.44
* Branch point: [274313@main](https://commits.webkit.org/274313@main)

## Proposed merges

  * [x] https://commits.webkit.org/276282@main - REGRESSION([274077@main](https://commits.webkit.org/274077@main)): failure to build on i586 (and likely other 32bit arches): static assertion failed: Timer should stay small
  * [x] https://commits.webkit.org/275934@main - [Clang] GeneratedSerializers.cpp(3716,11): error: offset of on non-standard-layout type 'WebKit::AudioTrackPrivateRemoteConfiguration' [-Werror,-Winvalid-offsetof]
  * [x] https://commits.webkit.org/275926@main - [JSC] DatePrototype.cpp(337,29): error: 'snprintf' will always be truncated; specified size is 28, but format string expands to at least 29 [-Werror,-Wformat-truncation]
  * [x] https://commits.webkit.org/276012@main and https://commits.webkit.org/276019@main - [WPE][GTK] Warning: WebKit2: Couldn't find 'run_async_javascript_function_in_world_finish' for the corresponding async function: 'run_async_javascript_function_in_world'
  * [x] https://commits.webkit.org/275168@main - [Nicosia] Add support for translate/rotate/scale animations
  * [x] https://commits.webkit.org/274475@main - [GLib] Enable WebCodecs
  * [x] https://commits.webkit.org/275063@main - Web process cache should expire old web processes sooner
  * [x] https://commits.webkit.org/275032@main - [GStreamer] Crash after 10 seconds on watchdog thread do to loop when destroying ~ImageDecoderGStreamerSample
  * [x] https://commits.webkit.org/275835@main - Atlassian Confluence blocks Epiphany's user agent

# 2.42

## Information

* Branch: https://github.com/WebKit/WebKit/commits/webkitglib/2.42
* Branch point: [266719@main](https://commits.webkit.org/266719@main)

## Proposed merges

 * [ ] https://commits.webkit.org/273818@main - [GTK] WebView empty with GTK Vulkan renderer - (If cleanly applies)
 * [ ] https://commits.webkit.org/272287@main - [TextureMapper] right side of for-loop condition must be constant for ES SL 1.0
 * [x] https://commits.webkit.org/272009@main - [GTK][WPE] Random incorrect image displayed as the background of a div
 * [x] https://commits.webkit.org/271861@main - [GStreamer] Misc leak fixes
 * [x] https://commits.webkit.org/271864@main - [GStreamer] HTTP source element leaks
 * [x] https://commits.webkit.org/266809@main - Revert Resizing video on YouTube can result in aliasing
 * [x] https://commits.webkit.org/271007@main - [FreeType] Do not special case the "sans" font family name
 * [x] https://commits.webkit.org/270977@main - Build fails with libxml2 version 2.12.0 due to API change
 * [x] https://commits.webkit.org/265870.537@safari-7616-branch - Security hardening for SincResampler
 * [x] https://commits.webkit.org/270274@main - REGRESSION([266247@main](https://commits.webkit.org/266247@main)): PDF "Save" button does nothing, "Print" function also broken
 * [x] https://commits.webkit.org/269255@main - Unable to scroll results.webkit.org results using the scrollbars
 * [x] https://commits.webkit.org/269223@main - Element application crashes in WebCore::Path::isEmpty()
 * [x] https://commits.webkit.org/269169@main - [WPE][GTK] Bump Safari version in user agent header
 * [x] https://commits.webkit.org/269068@main - [GStreamer][MSE] video playback uses GstVA, except on YouTube
 * [x] https://commits.webkit.org/268085@main - [GTK4] NativeWebWheelEvent crashes on wheel event tests
 * [x] https://commits.webkit.org/268142@main - [GTK][WPE] Use enable-html5-database runtime flag to control IndexedDB API
 * [x] https://commits.webkit.org/268137@main - GLContextX11.cpp:89:66: error: invalid cast from type 'long unsigned int' to type 'EGLNativePixmapType' {aka 'unsigned int'}
 * [x] https://commits.webkit.org/267995@main - Updated Swedish translation
 * [x] https://commits.webkit.org/267038@main - [GLib] Process launching hangs if xdg-dbus-proxy is not installed
 * [x] https://commits.webkit.org/267070@main - [GTK][WPE] Pass GBM_BO_USE_RENDERING to gbm_bo_create
 * [x] https://commits.webkit.org/267560@main - [JSC] Unreviewed RISCV64 build fix

# 2.40

## Information

* Branch: https://github.com/WebKit/WebKit/commits/webkitglib/2.40
* Branch point: [260527@main](https://commits.webkit.org/260527@main)

## Proposed merges

* [x] https://commits.webkit.org/265527@main - MemoryPressureMonitor (cgroupV1) honors memory.memsw.usage_in_bytes if exist
* [ ] ~~https://commits.webkit.org/262970@main - [WPE] Do not skip generic touch event handling for axis event gesturing~~
* [x] https://commits.webkit.org/264193@main - [GStreamer] Audio sinks created by media players leak
* [x] https://commits.webkit.org/264198@main - [GLib] Remove obsolete documentation from WebKitWebsiteDataManager
* [x] https://commits.webkit.org/264064@main - REGRESSION(262138@main): [GStreamer] Broke video rendering when GL is disabled
* [x] https://commits.webkit.org/264017@main - [GStreamer] Constant CPU usage on autoplaying videos, even when out of viewport
* [x] https://commits.webkit.org/263921@main - [GStreamer] video.loop cannot reliably be set on a paused pipeline
* [x] https://commits.webkit.org/263860@main - [GStreamer] Looped video is not seamless (flicker inbetween loops)
* [ ] ~~https://commits.webkit.org/263791@main - [GStreamer]\[MSE] Decoder sometimes receives caps event before stream-start~~ depends on [263585@main](https://commits.webkit.org/263585@main) which has many conflicts
* [x] https://commits.webkit.org/263134@main - [GStreamer] Critical warnings when browsing cnn.com
* [x] https://commits.webkit.org/262066@main - [GStreamer] Harness: Support for output stream caps changes
* [x] https://commits.webkit.org/261635@main - [GStreamer] Unmute doesn't work
* [x] https://commits.webkit.org/261629@main - [GStreamer]\[MSE] Version check for a GStreamer bug fixed in 1.20.6
* [ ] ~~https://commits.webkit.org/263176@main - [GTK] Build fix for Debian Stable after 263061@main~~ Files mentioned in patch only exist in the `main` branch
* [x] https://commits.webkit.org/263085@main - Images are not drawn even after they are completely loaded from a slow server
  - [x] https://commits.webkit.org/261700@main - [GPU Process] Have one copy of NativeImage when it is shared between WebProcess and GPUProcess
* [x] https://commits.webkit.org/262434@main - Fix build of SourceBrush.cpp
* [x] https://commits.webkit.org/262664@main - Fix build with GCC 13 -Werror
* [x] https://commits.webkit.org/262163@main - Fallback to elogind when systemd is unavailable at build time
* [x] https://commits.webkit.org/261961@main - [GTK] Slow scroll adjustment when using a mouse wheel
* [x] https://commits.webkit.org/261956@main - [GLib] webkit_user_content_manager_register_script_message_handler() world_name parameter should be nullable
* [x] https://commits.webkit.org/261839@main - [GLib] No render update when seeking outside of network buffer in fullscreen
* [x] https://commits.webkit.org/261810@main - REGRESSION(261320@main): [GLib] Broke WebKitUserContentManager::script-message-received
* [x] https://commits.webkit.org/261498@main - Unreviewed build fixes for RISCV64
* [x] https://commits.webkit.org/261060@main - [JSC] Remove m_dataScratch register in WasmBBQJIT
* [x] https://commits.webkit.org/261048@main - [JSC] x64 CCall returnValueGPR is not in m_validGPRs
* [x] https://commits.webkit.org/260700@main - [JSC] Fix SIMD in new BBQ
* [x] https://commits.webkit.org/260597@main - [JSC] Some misc cleanup in new BBQ
* [x] https://commits.webkit.org/260572@main - [JSC] Fix new BBQ's address materialization
* [x] https://commits.webkit.org/261474@main - REGRESSION(261349@main): [GLib] Problems with installed headers
* [x] https://commits.webkit.org/261433@main - [GLib] Warning on WebKitNetworkSession documentation; old API version should not build migration docs for new API version
* [x] https://commits.webkit.org/261432@main - [GTK] [l10n] Updated Turkish translation of WebKitGTK
* [x] https://commits.webkit.org/261349@main - [GLib] Rename WebKitWebExtension to WebKitWebProcessExtension
* [x] https://commits.webkit.org/261320@main - [GLib] Remove WebKitJavascriptResult
* [x] https://commits.webkit.org/261280@main - [GTK][l10n] Updated Polish translation of WebKitGTK for 2.40
* [x] https://commits.webkit.org/261228@main - [CoordinatedGraphics] Initialize WebCore::DisplayUpdate in ThreadedDisplayRefreshMonitor
* [x] https://commits.webkit.org/261199@main - [GTK] Update Korean translations - Mar 3, 2023
* [x] https://commits.webkit.org/261196@main - TestFeatures.h:40:37: error: use of undeclared identifier 'uint32_t'
* [x] https://commits.webkit.org/261118@main - [GLib] Need API for asynchronously handling WebKitDownload::decide-destination
* [x] https://commits.webkit.org/261018@main - REGRESSION(260511@main): [GLib] Fix HSTS storage directory
* [x] https://commits.webkit.org/261011@main - [GLib] Ensure no final classes have public class structs
* [x] https://commits.webkit.org/261008@main - Add Cancel/Unknown/Clear Hardware Keycodes
* [x] https://commits.webkit.org/261002@main - \[GTK]\[WPE] WebKitDownload destination should be a path instead of a URI
* [x] https://commits.webkit.org/260989@main - [GLib] Bump Safari version in user agent header
* [x] https://commits.webkit.org/260973@main - [GTK] Updated Swedish translation
* [x] https://commits.webkit.org/260949@main - [GLib] New API to get the request body of WebKitURISchemeRequest
* [x] https://commits.webkit.org/260875@main - REGRESSION(260082@main): \[GStreamer]\[1.20] YT broken
* [x] https://commits.webkit.org/260931@main - Non-unified build fixes, late February 2023 edition
* [x] https://commits.webkit.org/260818@main - [CMake] Rework decision to enable -gsplit-dwarf by default
* [x] https://commits.webkit.org/260790@main - [Linux] DMABufObject modifiers should default to being not-present #10517
* [x] https://commits.webkit.org/260584@main - LLIntAssembly.h:38532:23: error: ‘g_superSamplerCount’ was not declared in this scope

# 2.38

## Information

* Branch: https://github.com/WebKit/WebKit/commits/webkitglib/2.38
* Branch point: [253173@main](https://commits.webkit.org/253173@main)

## Proposed merges

* [ ] https://commits.webkit.org/265527@main - MemoryPressureMonitor (cgroupV1) honors memory.memsw.usage_in_bytes if exist
* [x] https://commits.webkit.org/260875@main REGRESSION(260082@main): [GStreamer][1.20] YT broken
* [x] https://commits.webkit.org/259999@main [GLIB] always update the active uri of the frame
* [x] https://commits.webkit.org/259434@main HTMLInputElement::setValueForUser should dispatch an input event
* [x] https://commits.webkit.org/258293@main [GStreamer] ImageDecoder fixes
* [x] https://commits.webkit.org/257912@main [GStreamer] Video element keeps changing the aspect ratio randomly (when the orientation information is in video's metadata)
* [x] https://commits.webkit.org/257838@main [GStreamer][MSE] Fix a caps leak in AppendPipeline
* [x] https://commits.webkit.org/257775@main Fix build with Ruby 3.2
* [x] https://commits.webkit.org/256566@main Network process crash in WebResourceLoadStatisticsStore::registrableDomains
* [x] https://commits.webkit.org/256486@main Crash in pas_segregated_page_switch_lock_and_rebias_while_ineligible_impl
* [x] https://commits.webkit.org/256395@main [GStreamer][WebRTC] Video encoder/decoder stats support
* [x] https://commits.webkit.org/255632@main [GStreamer] Critical warnings in appsink workaround thing
* [ ] ~~https://commits.webkit.org/255211@main [GStreamer][WebRTC] Set MTU to 1200 on RTP payloaders~~ commit is before branchpoint
* [ ] ~~https://commits.webkit.org/256654@main [GLIB] Fix MPRIS in flatpak sandbox~~ not a good backport candidate currently, see bug #247527
* [x] https://commits.webkit.org/256225@main [GStreamer][WebRTC] Media rendering improvements
* [x] https://commits.webkit.org/256149@main [GStreamer][WebRTC] Events forwarding between end-point and its consumers
* [x] https://commits.webkit.org/255721@main [SOUP] Spammed by 0-byte downloads on imgur.com
* [x] https://commits.webkit.org/255530@main [GTK] D-Bus proxy quietly fails if host bus address is not mounted in xdg-dbus-proxy's sandbox
* [x] https://commits.webkit.org/255218@main [GLib] D-Bus proxy quietly fails if host session bus address is an abstract socket
* [x] https://commits.webkit.org/255325@main REGRESSION(254232@main): Causes process launching to use fork + exec instead of posix_spawn
* [x] https://commits.webkit.org/255071@main [JSC][ARMv7] Fix clang compiler errors Constexpr if with a non-bool condition
* [x] https://commits.webkit.org/254673@main [GLIB] Bump Safari version in user agent header for Safari 16
* [x] https://commits.webkit.org/254163@main WebNotificationManager: fix build if SERVICE_WORKER=OFF
* [x] https://commits.webkit.org/254509@main [Nicosia] Async Scrolling: some elements are jumpy in gitlab
* [x] https://commits.webkit.org/254293@main Use a single xdg-dbus-proxy process
* [x] https://commits.webkit.org/254373@main [GStreamer][Debug] 2 mediastream tests hitting asserts
* [x] https://commits.webkit.org/254223@main [GStreamer] WebAudio drums demo makes WebKit GStreamer based ports crash
* [x] https://commits.webkit.org/254142@main [GStreamer] MediaPlayerPrivateGStreamer: Abort stale tasks on flushes
* [x] https://commits.webkit.org/254121@main [GLib] Fix build with CMake &lt 3.17
* [x] https://commits.webkit.org/254099@main [WPE] Kinetic scrolling doesn't work in overflow scrolling
* [x] https://commits.webkit.org/254097@main [GLIB] WheelEvent (phase=ended) has to be relayed to the scrollingTree if user scroll is in progress
* [x] https://commits.webkit.org/254093@main [GStreamer][MediaStream] Build failing for GStreamer versions &lt 1.18
* [x] https://commits.webkit.org/253980@main [GStreamer][WebRTC] End-point pipeline improvements
* [x] https://commits.webkit.org/253943@main [GStreamer][WebRTC] Video encoder improvements
* [x] https://commits.webkit.org/253940@main [GStreamer][MediaStream] Racy deadlock upon track removal request
* [x] https://commits.webkit.org/253936@main [GStreamer][WebRTC] Capabilities tweaks
* [x] https://commits.webkit.org/253897@main [GStreamer][MediaStream] Deadlock when disposing player while handling rotation tag
* [x] https://commits.webkit.org/253859@main (in case it's not fixed before 2.38.0) Unreviewed, reverting r251332@main.
* [x] https://commits.webkit.org/253679@main [GStreamer][WebRTC] Minor improvements in incoming media sources
* [x] https://commits.webkit.org/253678@main [GStreamer][WebRTC] Prepare for ICE selected candidate pair notifications
* [x] https://commits.webkit.org/253677@main [GStreamer][WebRTC] Align vpx capabilities with LibWebRTC provider
* [x] https://commits.webkit.org/253676@main [GStreamer][MediaStream] Skip video track configuration for RTP payloads
* [x] https://commits.webkit.org/253675@main [GStreamer][WebRTC] Misc clean-ups in stats gathering support
* [x] https://commits.webkit.org/253467@main [GStreamer][MediaStream] Additional fixes for disabled video track handling
* [x] https://commits.webkit.org/253297@main [GStreamer][WebRTC] Capabilities probing support
* [x] https://commits.webkit.org/253296@main [GStreamer] REGRESSION(253289@main): Broke debug builds
* [x] https://commits.webkit.org/253289@main [GStreamer][MediaStream] Video resizing fixes
* [x] https://commits.webkit.org/253287@main [WPE][GTK] Create GStreamerWebRTCProvider
* [x] https://commits.webkit.org/253257@main [WebRTC] Refactor generic code of LibWebRTCProvider into new WebRTCProvider
* [x] https://commits.webkit.org/253201@main [GStreamer][WebRTC] Remote RTP stats fixing
* [x] https://commits.webkit.org/253200@main [GStreamer][MediaStream] Lift malloc restriction for stream collection posting
* [x] https://commits.webkit.org/253807@main [CoordinatedGraphics] Cache and reuse image-based backing stores
* [X] https://commits.webkit.org/253605@main Assertion failure when using evaluated empty catch block
* [x] https://commits.webkit.org/254213@main Revert 252943@main for causing constant flickering on aa.com
* [x] https://commits.webkit.org/254237@main [GTK4] UI process hang when opening HTML select elements (combo boxes)

# 2.36 and older

The data for that releases is in the old Trac WiKi at https://trac.webkit.org/wiki/WebKitGTK/StableRelease#Listofreleases