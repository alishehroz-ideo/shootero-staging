# Shootero — WebGL staging build

Unity WebGL build of Shootero wired to the **GameBull staging API**
(`https://api.staging.g-b.store`).

Served by GitHub Pages from the repository root:
<https://alishehroz-ideo.github.io/shootero-staging/>

## What is in here

Unity's build output, verbatim, plus four files that live in this repo and **not** in
Unity's output:

| file | why it matters |
|---|---|
| `.nojekyll` | without it Pages runs Jekyll, which drops any path beginning with an underscore |
| `.gitattributes` | `* -text` — stops Git rewriting line endings inside the build artifacts |
| `.gitignore` | scratch files that must never be committed alongside a build |
| `README.md` | this file |

Unity rebuilds this folder from scratch, so those four are **deleted from the working tree by
every build**. Read `git status` for DELETIONS before committing a new build, and restore them:

```
git checkout HEAD -- .nojekyll .gitattributes .gitignore README.md
```

`git add -A` stages their removal without comment, and the symptom afterwards is a Pages site
that mysteriously stops serving parts of the build.

## Build

Both halves of the environment have to be switched together — the C# `GAMEBULL_PRODUCTION`
define picks the URL the *game* talks to, and the WebGL template's own `apiBase` picks the one
the *loading screen* queries for the per-game icon. Changing one alone is a silent mismatch.

With Unity **closed** (an open editor holds the project lock):

```
Unity.exe -quit -batchmode -nographics -projectPath <proj> \
  -executeMethod GameBull.EditorTools.GameBullBuildSetup.UseStaging

Unity.exe -quit -batchmode -nographics -buildTarget WebGL -projectPath <proj> \
  -executeMethod WebGLBuilder.BuildWebGLDeploy
```

Output goes to `builds/webgl-staging`. **That folder name is load-bearing** — Unity names its
artifacts after it, so it is what produces `webgl-staging.loader.js`, the exact filename
`index.html` asks for. Build to a differently-named folder and the page 404s on its own loader.

Use `BuildWebGLDeploy`, never `BuildWebGL`. The localhost entry point forces
`compressionFormat = Disabled` and `exceptionSupport = FullWithStacktrace`, both of which are
persisted to `ProjectSettings.asset` and outlive the build. On this game that is 117 MB instead
of 61 MB, with the wasm going 8.2 MB → 46 MB.

## Size

Brotli, with Unity's JS decompression fallback on — GitHub Pages sends no `Content-Encoding`
of its own, so the fallback is what makes the compressed files load at all.

| | |
|---|---|
| `webgl-staging.data.unityweb` | 54.7 MB |
| `webgl-staging.wasm.unityweb` | 8.2 MB |
| total | ~61 MB |

Compare the wasm against the last build before pushing — the loading screen was tuned around
that number.
