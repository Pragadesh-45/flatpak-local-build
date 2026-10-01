# flatpak-local-build

Build a Flatpak app from its manifest and install it for your user with a single command. It's handy for testing manifest changes or PRs to a Flathub repo before they're merged.

```sh
flatpak-local-build path/to/com.example.App.yml
flatpak run com.example.App//master
```

## What it does

1. Uses `flatpak-builder` from the host if it's installed. Otherwise it installs and uses `org.flatpak.Builder` from Flathub.
2. Builds the manifest, installing any missing runtime, SDK or base app from Flathub. It caches compiled code so rebuilds are fast.
3. Exports the build to a local repo, then installs it for your user **from the host**.

Step 3 is separate because apps that use `extra-data`, such as AppImage repackages, run `apply_extra` in their own bubblewrap sandbox. That sandbox can't start inside the `org.flatpak.Builder` sandbox, so `flatpak-builder --install` fails with:

```
bwrap: No permissions to create a new namespace ...
Error: Failed to install ...: While trying to apply extra data: apply_extra script failed
```

Installing from the host avoids this.

## Install

```sh
curl -fsSL -o ~/.local/bin/flatpak-local-build \
  https://raw.githubusercontent.com/Pragadesh-45/flatpak-local-build/main/flatpak-local-build
chmod +x ~/.local/bin/flatpak-local-build
```

Alternatively, clone the repo and run `./flatpak-local-build` directly. You need `flatpak` with the `flathub` remote added for your user.

## Usage

```
flatpak-local-build [options] <manifest>

  --run              build, install, then launch the app
  --uninstall        remove the locally installed build and exit
  --clean            delete the build directory and exit
  --id ID            app id (default: read from the manifest)
  --branch NAME      branch to build and install (default: master)
  --build-root DIR   where build files go
                     (default: ${XDG_CACHE_HOME:-~/.cache}/flatpak-local-build/<app id>)
  --deps-from REMOTE remote to install runtimes/SDKs from (default: flathub)
  -h, --help         show this help
```

The local build is installed on its own branch (`master` by default), so it can sit next to a Flathub `stable` install. Use `flatpak run <app id>//master` to make sure you launch the local build. `--uninstall` removes only that branch.

## Example: testing a Flathub PR

```sh
git clone https://github.com/flathub/com.usebruno.Bruno && cd com.usebruno.Bruno
gh pr checkout 129
flatpak-local-build --run com.usebruno.Bruno.yml
```

## License

MIT
