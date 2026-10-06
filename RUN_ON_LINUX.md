# Running Legends Online on Ubuntu / Linux

**Legends Online**: https://playlegends.online/

Client internals: `silkroad.exe` is the game launcher,
`sro_client.exe` is the actual 32-bit DirectX 9 game client, and
`D3DX9_43.dll` / `legends.dll` / `vsroplus_lib.dll` are bundled alongside it
(typical for Silkroad).

## 0. Create an account and get the client

1. Register an account: https://playlegends.online/auth/register.html if not already
2. Download the client (Google Drive): https://drive.google.com/file/d/1kLyqBW3Z5d4ub5R1vZ723q_MebJfsYJ8/view
3. Extract the archive to a folder, e.g. `~/Games/LegendsOnline`.
4. If you hit install/account issues, Discord is our support channel:
   https://discord.gg/legendsonline1


## Reference versions (this machine, 2026-10-06)

| Component  | Version                                      |
|------------|-----------------------------------------------|
| OS         | Ubuntu 24.04.4 LTS (kernel 7.0.0-34-generic)  |
| Wine       | 9.0 (`wine-9.0 Ubuntu 9.0~repack-4build3`)    |
| Winetricks | 20240105-2                                    |
| Lutris     | 0.5.18 (optional, also installed)              |
| GPU        | Intel Iris Xe (Alder Lake-UP3), Mesa driver   |
| Client     | 32-bit PE (Intel 80386), DirectX 9            |

If you set this up on another machine, install at least these versions or newer.

## 1. Install Wine with 32-bit support

Once you have the client downloaded, proceed with installing Wine with 32-bit support.

```bash
sudo dpkg --add-architecture i386
sudo apt update
sudo apt install wine wine32:i386 wine64 winetricks
```

Verify:

```bash
wine --version   # wine-9.0 (or newer)
```

## 2. Install required Windows runtime components

The client links against `mfcm80.dll` (MFC80) and `mscoree.dll` (.NET), so install:

```bash
winetricks corefonts vcrun2005
```

- `corefonts` avoids missing-font/garbled-text issues in the UI.
- `vcrun2005` provides the MFC80 runtime the client needs.
- `D3DX9_43.dll` is already shipped next to the exe in this folder, so Wine will
  pick it up automatically — you normally **don't** need `winetricks d3dx9` unless
  you still get a "d3dx9_43.dll missing" error.
- If the client shows a one-time prompt to install **Wine Mono** or **Gecko** on
  first launch, accept it — that satisfies the `mscoree.dll` dependency.

## 3. (Optional) Use a dedicated Wine prefix

Keeps this game isolated from other Wine apps:

```bash
WINEARCH=win32 WINEPREFIX=~/.wine-legends winetricks corefonts vcrun2005
```

If you skip this, the default `~/.wine` prefix works fine too, since
`wine32:i386` is installed system-wide.

## 4. Launch the game

From this folder:

```bash
cd ~/Games/LegendsOnline
wine silkroad.exe
```

(or, if using a dedicated prefix: `WINEPREFIX=~/.wine-legends wine silkroad.exe`)

If the launcher window misbehaves (the Silkroad launcher sometimes skips
patch/update UI), launch the actual client
directly instead:

```bash
wine sro_client.exe
```

## 5. Optional: run it via Lutris instead

Since you might want a GUI or have Lutris already installed, you can wrap this in a Lutris entry:

1. Open Lutris → **+ Add game** → **Add locally installed game**
2. Runner: **Wine**. Executable: point to `silkroad.exe` (or `sro_client.exe`)
3. Working directory: ~/Games/LegendsOnline
4. Wine version: 9.0 (system)
5. Save, then launch from Lutris — it manages its own prefix per game

## 6. Troubleshooting

- **"d3dx9_43.dll not found"** → run `winetricks d3dx9_43` inside the same prefix
- **MFC/mfcm80 errors** → re-run `winetricks vcrun2005`
- **Crashes** → this folder already has a `Dump/` directory with `.dmp` crash
  files from past sessions; check the timestamp of the newest one after a crash
- **Black screen / resolution issues** → try forcing a windowed virtual desktop:
  `wine explorer /desktop=1920x1080 silkroad.exe`
- **Blurry/scaled UI on HiDPI or Wayland** → run your session under Xorg, or set
  the Wine prefix to disable DPI scaling via `winecfg` → Graphics tab
- **Poor performance on the Intel Iris Xe iGPU** → ensure `mesa-vulkan-drivers`
  is installed; wined3d (Wine's default DX9 backend) is normally sufficient for
  a game this old, so DXVK is optional, not required

## Notes

- Log into the game client with the same username/password you registered at
  https://playlegends.online/auth/register.html
- If the launcher won't connect or you suspect the server needs something
  this guide doesn't cover, ask in Discord: https://discord.gg/legendsonline1
