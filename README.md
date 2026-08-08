<body>

<h1>SMPlayer AppImage — GPU HW Accelerated 8K 60FPS</h1>

<div class="logo">
    <img src="images/logo.svg" alt="SMPlayer AppImage Logo">
</div>

<p class="intro">
    <strong>A portable, hardware-accelerated SMPlayer AppImage for Linux.</strong><br>
    Designed around a lightweight and cooperative AppImage architecture that
    uses the host system's multimedia stack whenever possible while providing
    compatible runtime libraries inside the AppImage as a fallback.
</p>

<h2>Overview</h2>

<p>
This project provides a custom portable build of
<strong>SMPlayer</strong> packaged as an AppImage and optimized for
hardware-accelerated video playback.
</p>

<p>
The goal is simple: keep the application portable without turning the
AppImage into an entire operating system. The application can use compatible
libraries provided by the host system, while bundled libraries are available
inside the AppImage when required.
</p>

<p>
This approach is particularly useful on Arch Linux, CachyOS, Manjaro and
other modern Linux distributions, while still providing a reasonable chance
of compatibility with other distributions.
</p>

<div class="success">
    <strong>Hardware acceleration verified.</strong>
    Playback has been tested with high-resolution H.264/AVC material,
    including 1080p, 4K and high-frame-rate video.
</div>

<h2>Highlights</h2>

<ul>
    <li><span class="feature">Portable AppImage</span> — no system-wide installation required.</li>
    <li><span class="feature">GPU hardware video decoding</span> through the host multimedia stack.</li>
    <li><span class="feature">4K 30 FPS playback tested.</span></li>
    <li><span class="feature">4K 60 FPS playback tested.</span></li>
    <li><span class="feature">High-resolution video playback including 8K testing.</span></li>
    <li><span class="feature">H.264 / AVC hardware decoding.</span></li>
    <li><span class="feature">Subtitles and translations supported.</span></li>
    <li><span class="feature">YouTube playback through yt-dlp.</span></li>
    <li><span class="feature">M3U / M3U8 playlist playback tested.</span></li>
    <li><span class="feature">IPTV playback tested with HD channels.</span></li>
    <li><span class="feature">Cooperative runtime design.</span></li>
    <li><span class="feature">Minimal AppDir layout.</span></li>
</ul>

<h2>SMPlayer Running High-Resolution Video</h2>

<div class="screenshot">
    <img src="images/screenshot1.png"
         alt="SMPlayer playing high-resolution video">
    <div class="caption">
        SMPlayer playing high-resolution video with hardware acceleration.
    </div>
</div>

<h2>GPU Hardware Acceleration Verification</h2>

<p>
Hardware acceleration can be monitored using the appropriate GPU monitoring
tools. For Intel graphics, <code>intel_gpu_top</code> can be used to verify
activity in the video engine during playback.
</p>

<div class="screenshot">
    <img src="images/screenshot2.png"
         alt="Intel GPU Top showing hardware video acceleration">
    <div class="caption">
        Intel GPU Top showing GPU activity during hardware-accelerated playback.
    </div>
</div>

<h2>Cooperative AppImage Architecture</h2>

<p>
This AppImage follows a <strong>cooperative runtime philosophy</strong>.
Instead of attempting to bundle an entire Linux userspace, it provides the
application and the libraries required for portability while allowing the
host system to provide compatible components when appropriate.
</p>

<p>
This is especially important for graphics and multimedia components.
Hardware acceleration depends heavily on the host kernel, GPU drivers,
VA-API/VDPAU stack and related system components.
</p>

<p>
The AppImage therefore does <strong>not</strong> attempt to replace the host
GPU driver stack.
</p>

<div class="note">
    <strong>Important:</strong>
    GPU drivers remain provided by the host operating system. The AppImage
    does not contain proprietary GPU drivers.
</div>

<h2>Runtime Libraries</h2>

<p>
The AppImage contains a minimal set of runtime libraries required by the
application. These include compatible Qt5 components and other runtime
dependencies required for the portable build.
</p>

<p>
The runtime is intentionally kept small. Unnecessary development packages,
compilers, debugging tools and duplicated system components are not included.
</p>

<p>
Depending on the distribution and its library versions, the host system may
provide a compatible version of a library while the AppImage provides a
fallback copy.
</p>

<h2>System Requirements</h2>

<p>
The recommended environment contains the following multimedia components:
</p>

<ul>
    <li><strong>MPV</strong></li>
    <li><strong>FFmpeg</strong></li>
    <li><strong>Qt5 / compatible Qt runtime</strong></li>
    <li><strong>GPU drivers supporting hardware video decoding</strong></li>
</ul>

<p>
The AppImage already contains several runtime components required for
portability. However, using the host versions may be preferable when they
are compatible with the system's optimized multimedia stack.
</p>

<h3>Arch Linux / CachyOS / Manjaro</h3>

<pre><code>sudo pacman -S mpv ffmpeg qt5-base</code></pre>

<h3>Debian / Ubuntu</h3>

<pre><code>sudo apt install mpv ffmpeg</code></pre>

<h3>Fedora</h3>

<pre><code>sudo dnf install mpv ffmpeg</code></pre>

<div class="warning">
    <strong>Note:</strong>
    Package names can differ between distributions. The AppImage is intended
    to remain portable, but multimedia hardware acceleration ultimately
    depends on the host system's GPU driver and multimedia stack.
</div>

<h2>Hardware Acceleration</h2>

<p>
Hardware decoding is strongly recommended for high-resolution video.
Without hardware acceleration, 4K and especially 8K playback can place a
very large load on the CPU.
</p>

<p>
For Intel GPUs, VA-API support can be checked with:
</p>

<pre><code>sudo pacman -S libva-utils
vainfo</code></pre>

<p>
Intel GPU activity can be monitored with:
</p>

<pre><code>sudo pacman -S intel-gpu-tools
intel_gpu_top</code></pre>

<p>
During hardware-accelerated playback, the GPU video engine should show
activity.
</p>

<h2>Portable Path Fix</h2>

<p>
The original SMPlayer source contains installation-path assumptions that
normally expect resources to be installed into fixed system locations.
</p>

<p>
This project modifies the path handling so that resources can be discovered
relative to the portable application layout.
</p>

<h3>Modified Source File</h3>

<p>
The primary portability modification is located in:
</p>

<pre><code>src/paths.cpp</code></pre>

<p>
The following implementation makes the application locate its shared data
relative to the executable:
</p>

<pre><code>QString Paths::dataPath() {
    return QDir(appPath()).absoluteFilePath("../share/smplayer");
}

QString Paths::translationPath() {
    return dataPath() + "/translations";
}

QString Paths::docPath() {
    return dataPath() + "/docs";
}

QString Paths::themesPath() {
    return dataPath() + "/themes";
}</code></pre>

<h2>Fallback Data Path</h2>

<p>
The final implementation also supports a fallback mechanism. If the portable
data directory exists, it is preferred. Otherwise, the original compiled
path can still be used when available.
</p>

<pre><code>QString Paths::dataPath() {
    QString appData =
        QDir(appPath()).absoluteFilePath("../share/smplayer");

    if (QDir(appData).exists()) {
        return appData;
    }

#ifdef DATA_PATH
    QString path = QString(DATA_PATH);

    if (!path.isEmpty()) {
        return path;
    }
#endif

    return appPath();
}</code></pre>

<h2>Build Configuration</h2>

<p>
The fixed installation-path definitions were removed from the build
configuration so that the runtime path logic can determine the correct
location dynamically.
</p>

<p>
In <code>src/smplayer.pro</code>, the following definitions are disabled:
</p>

<pre><code>#DEFINES += DATA_PATH=$(DATA_PATH)
#DEFINES += DOC_PATH=$(DOC_PATH)
#DEFINES += TRANSLATION_PATH=$(TRANSLATION_PATH)
#DEFINES += THEMES_PATH=$(THEMES_PATH)
#DEFINES += SHORTCUTS_PATH=$(SHORTCUTS_PATH)</code></pre>

<h2>Portable Resource Layout</h2>

<p>
The intended AppImage structure follows the standard Linux filesystem
layout:
</p>

<pre><code>AppDir/
├── AppRun
├── smplayer.desktop
├── smplayer.svg
└── usr/
    ├── bin/
    │   └── smplayer
    ├── lib/
    │   └── Qt5 runtime libraries
    └── share/
        └── smplayer/
            ├── translations/
            ├── themes/
            ├── shortcuts/
            └── docs/</code></pre>

<p>
Keeping resources under <code>usr/share/smplayer</code> preserves the normal
Linux filesystem organization while allowing the application to operate from
inside an AppImage.
</p>

<h2>Translations</h2>

<p>
SMPlayer translations are included in the portable resource tree and are
loaded relative to the application's portable data path.
</p>

<p>
This avoids depending exclusively on a fixed system installation such as:
</p>

<pre><code>/usr/share/smplayer</code></pre>

<p>
The portable application instead resolves its resources relative to the
AppImage filesystem.
</p>

<h2>YouTube Playback</h2>

<p>
The application can use <strong>yt-dlp</strong> for online video playback.
The portable environment can maintain its own user-local yt-dlp installation
without requiring a system-wide installation.
</p>

<p>
This allows SMPlayer to access online video services while keeping the main
application portable.
</p>

<h2>M3U / M3U8 and IPTV</h2>

<p>
Playlist playback has also been tested with M3U and M3U8 sources.
</p>

<p>
HD IPTV channels were successfully tested through SMPlayer, demonstrating
that the portable build can also function as a lightweight media player for
network streams.
</p>

<h2>Optimization Philosophy</h2>

<p>
This project intentionally avoids bundling unnecessary components.
</p>

<ul>
    <li>Unused development files removed.</li>
    <li>Unnecessary debugging components removed.</li>
    <li>Redundant runtime files removed.</li>
    <li>Unused Qt components removed where possible.</li>
    <li>Runtime binaries stripped.</li>
    <li>Only required Qt libraries retained.</li>
    <li>System GPU drivers are not bundled.</li>
    <li>Host multimedia components can be used when compatible.</li>
</ul>

<p>
The objective is not to create a miniature Linux distribution inside an
AppImage. The objective is to provide a practical portable application with
the smallest reasonable runtime footprint.
</p>

<h2>Why Hardware Acceleration Matters</h2>

<p>
High-resolution video can be extremely demanding when decoded entirely by
the CPU. Hardware decoding moves the appropriate video decoding workload to
the GPU, significantly reducing CPU utilization.
</p>

<p>
This is particularly interesting for older GPUs that still contain dedicated
video decoding hardware capable of efficiently decoding formats such as
H.264/AVC.
</p>

<p>
The project therefore focuses on using the hardware capabilities already
present in the host system rather than replacing them with bundled drivers.
</p>

<h2>Project Philosophy</h2>

<p>
<strong>Portable does not have to mean bloated.</strong>
</p>

<p>
This project follows a cooperative approach:
</p>

<ul>
    <li>Bundle what the application genuinely needs.</li>
    <li>Use compatible host libraries whenever appropriate.</li>
    <li>Keep GPU drivers in the host system.</li>
    <li>Avoid duplicating the entire operating system inside the AppImage.</li>
    <li>Keep the application easy to copy, execute and remove.</li>
</ul>

<p>
The result is a lightweight AppImage designed to behave like a portable
Linux application rather than a complete userspace distribution.
</p>

<h2>Source Code Modifications</h2>

<p>
This repository includes the modified source files used for the portability
work:
</p>

<pre><code>src mod/
├── LEEME.MD
├── paths.cpp
└── smplayer.pro</code></pre>

<p>
The modifications are intentionally small and focused primarily on resource
path discovery and portable filesystem layout.
</p>

<h2>Testing</h2>

<p>
The build has been tested with:
</p>

<ul>
    <li>H.264 / AVC video</li>
    <li>1080p video</li>
    <li>1080p 60 FPS</li>
    <li>4K 30 FPS</li>
    <li>4K 60 FPS</li>
    <li>High-resolution 8K testing</li>
    <li>Hardware-accelerated playback</li>
    <li>Subtitles</li>
    <li>SMPlayer translations</li>
    <li>YouTube playback through yt-dlp</li>
    <li>M3U playlists</li>
    <li>M3U8/network streams</li>
    <li>HD IPTV channels</li>
</ul>

<h2>Disclaimer</h2>

<p>
Hardware acceleration depends on the capabilities and configuration of the
host GPU, kernel, drivers and multimedia libraries. Successful playback on
one system does not guarantee identical results on every Linux distribution.
</p>

<p>
Bundled libraries are provided for portability, but distribution-specific
optimizations and library versions can affect runtime behavior.
</p>

<h2>License</h2>

<p>
SMPlayer remains licensed under its original open-source license.
</p>

<p>
This repository contains the AppImage/AppDir packaging work, portability
modifications, documentation and related helper files.
</p>

</body>
</html>
