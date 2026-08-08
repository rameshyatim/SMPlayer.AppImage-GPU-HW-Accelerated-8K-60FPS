SMPlayer Portability Patch

This directory contains the source-code modifications used to make SMPlayer work correctly as a portable AppImage.

The modification is intentionally small and focused on the application's resource path handling.

1. src/paths.cpp — Portable Resource Paths

The main portability change is located in:

src/paths.cpp

The original application expects its resources to be installed through traditional system paths.

The first version of the portability patch changed the resource paths to resolve relative to the executable:

QString Paths::dataPath() {
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
}

This makes the following AppImage/AppDir layout possible:

AppDir/
└── usr/
    ├── bin/
    │   └── smplayer
    └── share/
        └── smplayer/
            ├── translations/
            ├── themes/
            └── docs/

The important part is:

QDir(appPath()).absoluteFilePath("../share/smplayer")

The resource directory is therefore discovered relative to the executable rather than requiring a hardcoded /usr/share/smplayer location.

2. Final dataPath() Implementation

The dataPath() implementation was subsequently improved to preserve compatibility with traditional installations.

The final version is:

QString Paths::dataPath() {
    QString appData = QDir(appPath()).absoluteFilePath("../share/smplayer");

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
}

This provides a simple fallback hierarchy:

1. AppImage-relative ../share/smplayer
2. DATA_PATH, if defined and valid
3. appPath()

This is preferable to blindly forcing an AppImage-specific path because it preserves compatibility with other ways of running SMPlayer.

3. translationPath()

Translations are resolved from the dynamically detected application data directory:

QString Paths::translationPath() {
    return dataPath() + "/translations";
}

Therefore the AppImage can contain:

usr/share/smplayer/translations/

without requiring the system installation path.

4. docPath()

Documentation follows the same mechanism:

QString Paths::docPath() {
    return dataPath() + "/docs";
}
5. themesPath()

Themes are also resolved relative to the detected application data directory:

QString Paths::themesPath() {
    return dataPath() + "/themes";
}
6. src/smplayer.pro

The compile-time path macros were disabled so that the runtime path implementation can determine the resource location.

The following definitions were commented out:

#DEFINES += DATA_PATH=$(DATA_PATH)
#DEFINES += DOC_PATH=$(DOC_PATH)
#DEFINES += TRANSLATION_PATH=$(TRANSLATION_PATH)
#DEFINES += THEMES_PATH=$(THEMES_PATH)
#DEFINES += SHORTCUTS_PATH=$(SHORTCUTS_PATH)

This is important because otherwise the build system can continue embedding the traditional installation paths into the application.

Result

The modification allows SMPlayer to operate from an AppImage while keeping the resource structure clean:

AppDir/
└── usr/
    ├── bin/
    │   └── smplayer
    └── share/
        └── smplayer/
            ├── translations/
            ├── themes/
            └── docs/

The patch does not require rewriting the SMPlayer application or replacing its resource system.

It simply changes how the application discovers its data directory.

This makes the modification small, auditable and easy for other developers to reproduce.

Why the Source Patch Is Included

The modified source is provided openly so that other developers can inspect exactly what was changed.

The objective is to demonstrate a lightweight approach to making existing Linux applications more AppImage-friendly without unnecessarily bundling an entire operating-system userspace.
