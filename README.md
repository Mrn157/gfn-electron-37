For personal use, electron 35 is not on nixpkgs anymore:

https://github.com/Mrn157/nix-dotfiles

# How to Install

## Create a .nix file and place this code inside it:
```bash
{
  lib,
  buildNpmPackage,
  fetchFromGitHub,
  electron_37,
  nix-update-script,
  makeBinaryWrapper,
  python3,
}:
let
  electron = electron_37;
  version = "2.2.0";
in
buildNpmPackage {
  pname = "gfn-electron";
  inherit version;

  src = fetchFromGitHub {
    owner = "mrn157";
    repo = "gfn-electron-37";
    rev=  "bef8ee39b06c1ccb62a97fcc5ea84c2c856b33f7";
    hash = "sha256-g5z5dzZ64xyTCq40fdzfsMlgsMa+EgXlkuFnhxEQT2w=";
  };

  npmDepsHash = "sha256-C3un04tucPG7DNyggQ6LqQc+/p2t7lhIfOkpB4Gf8qw=";

  nativeBuildInputs = [
    makeBinaryWrapper
    # node_modules/node-gyp/gyp/pylib/gyp/input.py
    #   from distutils.version import StrictVersion
    # ModuleNotFoundError: No module named 'distutils'
    (python3.withPackages (ps: with ps; [ setuptools ]))
  ];

  env.ELECTRON_SKIP_BINARY_DOWNLOAD = true;

  # FIXME: Git dependency node_modules/register-scheme contains install scripts,
  # but has no lockfile, which is something that will probably break.
  forceGitDeps = true;

  makeCacheWritable = true;

  buildPhase = ''
    runHook preBuild

    # NOTE: Upstream explicitly opts to not build an ASAR as it would cause all
    # text to disappear in the app.
    npm exec electron-builder -- \
      --dir \
      -c.electronDist=${electron.dist} \
      -c.electronVersion=${electron.version}

    runHook postBuild
  '';

  installPhase = ''
    runHook preInstall

    mkdir -p $out
    cp -r dist/*-unpacked $out/dist

    install -Dm644 com.github.hmlendea.geforcenow-electron.desktop -t $out/share/applications
    install -Dm644 icon.png $out/share/icons/hicolor/512x512/apps/geforcenow-electron.png

    runHook postInstall
  '';

  postFixup = ''
    makeWrapper $out/dist/geforcenow-electron $out/bin/geforcenow-electron \
      --add-flags "--no-sandbox --disable-gpu-sandbox" \
      --add-flags "\''${NIXOS_OZONE_WL:+\''${WAYLAND_DISPLAY:+--ozone-platform-hint=auto --enable-features=WaylandWindowDecorations --enable-wayland-ime=true}}"

    substituteInPlace $out/share/applications/com.github.hmlendea.geforcenow-electron.desktop \
      --replace-fail "/opt/geforcenow-electron/geforcenow-electron" "geforcenow-electron" \
      --replace-fail "Icon=nvidia" "Icon=geforcenow-electron"
  '';

  passthru.updateScript = nix-update-script { };

  meta = {
    description = "Linux Desktop client for Nvidia's GeForce NOW game streaming service";
    homepage = "https://github.com/hmlendea/gfn-electron";
    license = with lib.licenses; [ gpl3Only ];
    platforms = lib.intersectLists lib.platforms.linux electron.meta.platforms;
    maintainers = with lib.maintainers; [ pluiedev ];
    mainProgram = "geforcenow-electron";
  };
}
```

## Add to somewhere in your configuration (like `environment.systemPackages`)
```bash
(pkgs.callPackage ./path/to/config.nix {})
```
Change `./path/to/config.nix` to the correct path

[![Donate](https://img.shields.io/badge/-%E2%99%A5%20Donate-%23ff69b4)](https://hmlendea.go.ro/fund.html) [![Build Status](https://github.com/hmlendea/gfn-electron/actions/workflows/node.js.yml/badge.svg)](https://github.com/hmlendea/gfn-electron/actions/workflows/node.js.yml) [![Latest GitHub release](https://img.shields.io/github/v/release/hmlendea/gfn-electron)](https://github.com/hmlendea/gfn-electron/releases/latest)

# NO LONGER DISCONTINUED

Hi everyone!

The owner of this repository had discontinued and archived the project a few weeks ago but the project is back now. I will be continuing to work on this project and we welcome as much help as possible from the community!

Thanks for everyone's support!

---

# About

Unofficial client for Nvidia's GeForce NOW game streaming service, providing a native Linux desktop experience and some additional features such as Discord rich presence.

## About us

## Disclaimer

This project and its contributors are not affiliated with Nvidia, nor its GeForce NOW product. This repository does not contain any Nvidia / GeForce NOW software. It is simply an Electron wrapper that loads the official GFN web application page, just as it would in a regular web browser.

## Developers

Founder & Owner: Horațiu Mlendea (https://github.com/hmlendea)

Maintainer: Goldy Yan (https://github.com/Cybertaco360)

# Installation

[![Get it from the AUR](https://raw.githubusercontent.com/hmlendea/readme-assets/master/badges/stores/aur.png)](https://aur.archlinux.org/packages/geforcenow-electron/) [![Get it from FlatHub](https://raw.githubusercontent.com/hmlendea/readme-assets/master/badges/stores/flathub.png)](https://flathub.org/apps/details/io.github.hmlendea.geforcenow-electron)

***Note**: The main version of this project, which receives the most support, is the flatpak version hosted on FlatHub!*

## Manual Installation

 - Go to the [latest release](https://github.com/hmlendea/gfn-electron/releases/latest).
 - Download the specific file that best fits your distro.

***Note**: Manual installations are possible but not supported. Please use the flatpak version if you have any trouble with the manual installation!*

# Usage

 - [Basic usage](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage)
   - [Keyboard shortcuts](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage#keyboard-shortcuts)
   - [Command-line arguments](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage#command-line-arguments)
   - [Changing the keyboard layout](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage#changing-the-keyboard-layout)
   - [Directly launching a game from the desktop](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage#directly-launching-a-game-from-the-desktop)
 - [Integrations](https://github.com/hmlendea/gfn-electron/wiki/Integrations)
   - [Discord](https://github.com/hmlendea/gfn-electron/wiki/Integrations#discord)
     - [Using native GFN + flatpak Discord](https://github.com/hmlendea/gfn-electron/wiki/Integrations#using-native-gfn--flatpak-discord)
     - [Disabling the Discord RPC](https://github.com/hmlendea/gfn-electron/wiki/Integrations#disabling-the-discord-rpc)
 - [Troubleshooting](https://github.com/hmlendea/gfn-electron/wiki/Troubleshooting)
   - [Gamepad controls are not detected](https://github.com/hmlendea/gfn-electron/wiki/Troubleshooting#gamepad-controls-are-not-detected)
   - [Steam Deck controls are not detected](https://github.com/hmlendea/gfn-electron/wiki/Troubleshooting#steam-deck-controls-are-not-detected)

# Building from source

## Requirements

You will need to install [npm](https://www.npmjs.com/), the Node.js package manager. On most distributions, the package is simply called `npm`.

## Cloning the source code

Once you have npm, clone the wrapper to a convenient location:

```bash
git clone https://github.com/hmlendea/gfn-electron.git
```

## Building

```bash
npm install
npm start
```

On subsequent runs, `npm start` will be all that's required.

## Updating the source code

Simply pull the latest version of master and install any changed dependencies:

```bash
git checkout master
git pull
npm install
```

# Links
 - [GeForce NOW](https://nvidia.com/en-eu/geforce-now)
 - [FlatHub release](https://flathub.org/apps/details/io.github.hmlendea.geforcenow-electron)
 - [FlatHub repository](https://github.com/flathub/io.github.hmlendea.geforcenow-electron)
 - [Basic usage](https://github.com/hmlendea/gfn-electron/wiki/Basic-usage)
 - [Troubleshooting](https://github.com/hmlendea/gfn-electron/wiki/Troubleshooting)
