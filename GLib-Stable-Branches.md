This is the page for handling WebKitGTK and WPEWebKit stable branches.
We should merge not overly intrusive patches that improve stability or
performance, fix build issues, etc.

# 2.40

## Information

* Branch: https://github.com/WebKit/WebKit/commits/webkitglib/2.40
* Branch point: [260527@main](https://commits.webkit.org/260527@main)

## Proposed merges

* [ ] https://commits.webkit.org/261635@main - [GStreamer] Unmute doesn't work
* [ ] https://commits.webkit.org/261961@main - [GTK] Slow scroll adjustment when using a mouse wheel
* [ ] https://commits.webkit.org/261956@main - [GLib] webkit_user_content_manager_register_script_message_handler() world_name parameter should be nullable
* [ ] https://commits.webkit.org/261839@main - [GLib] No render update when seeking outside of network buffer in fullscreen
* [ ] https://commits.webkit.org/261810@main - REGRESSION(261320@main): [GLib] Broke WebKitUserContentManager::script-message-received
* [ ] https://commits.webkit.org/260572@main - [JSC] Fix new BBQ's address materialization
* [ ] https://commits.webkit.org/260597@main - [JSC] Some misc cleanup in new BBQ
* [ ] https://commits.webkit.org/260700@main - [JSC] Fix SIMD in new BBQ
* [ ] https://commits.webkit.org/261048@main - [JSC] x64 CCall returnValueGPR is not in m_validGPRs
* [ ] https://commits.webkit.org/261060@main - [JSC] Remove m_dataScratch register in WasmBBQJIT
* [ ] https://commits.webkit.org/261498@main - Unreviewed build fixes for RISCV64
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
