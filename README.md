# OpenCASCADE Corresponding Source

This repository exists for one reason: to satisfy the LGPL-2.1 obligation that
attaches to the **Open CASCADE Technology (OCCT)** library that Saimek
distributes, in compiled WebAssembly form, inside **Saimek3D**.

It is a compliance artifact. It is not a product, it is not maintained as one,
and nothing here is written or owned by Saimek. Everything in this repository is
third-party source, vendored verbatim, plus this README.

If you have a copy of Saimek3D and you want the source for the LGPL library
inside it, you are in the right place. The tree under [`occt/`](occt/) is that
source.

## Why this repository exists rather than a link

LGPL-2.1 places the obligation to supply Corresponding Source on the party doing
the **distributing** — that is Saimek, not the upstream project. A link to
`opencascade.org` would discharge nothing: if upstream reorganises, renames a
tag, or removes a snapshot, the link dies and the recipient is left with no
source. So the bytes live here, in a repository Saimek controls.

## The chain, from the binary in your app to the source in this repository

Each step below was established by measurement, not by reading version strings.

**1. The shipped binary.**

    Saimek3D.app/Contents/Resources/web/replicad/opencascade.wasm
    size    10,855,736 bytes
    sha256  2e07c45b83267b38d3102ec411fad11e7ce2ed71854084e461c5ab5fee94aaff

**2. That file is byte-identical to `src/replicad_single.wasm` from the npm
package `replicad-opencascadejs@0.23.0`.**

All 27 published versions of that package (0.5.0 through 1.0.0) were downloaded
and every WebAssembly file in each was hashed. Exactly one hash matched:
`src/replicad_single.wasm` in 0.23.0. The match is on the full file hash above,
not on a version number. Note that 1.0.0 relocated its output to `dist/`; both
files at that path were hashed and neither matches.

**3. That package is built by the recipe in
[`build-recipe/replicad-opencascadejs-0.23.0/`](build-recipe/replicad-opencascadejs-0.23.0/).**

The recipe is vendored from the upstream monorepo at

    github.com/sgenoud/replicad
    commit 0a2e81e3dea02291343557b63f93fdb58136eba2
    path   packages/replicad-opencascadejs   (package version 0.23.0)

The built `src/replicad_single.wasm` present at that upstream commit hashes to
the same `2e07c45b…` as the shipped binary, which is what ties this specific
recipe to this specific binary.

Its `buildSingle` step is:

    cd build-config
    docker run -it --rm -v $(pwd):/src -u $(id -u):$(id -g) \
      donalffons/opencascade.js custom_build_single.yml

**4. That Docker image is built from
[`build-recipe/opencascade.js-5ff2b750/`](build-recipe/opencascade.js-5ff2b750/).**

Vendored from

    github.com/donalffons/opencascade.js
    commit 5ff2b750ba4b9a9fdfbff8842712cbb562e78ce7   (repository tip, 2023-03-27)

Its `Dockerfile` pins OCCT by commit, not by tag:

    ENV OCCT_COMMIT_HASH_FULL bb368e271e24f63078129283148ce83db6b9670a

**5. That commit is OCCT 7.6.2, and its source is vendored at
[`occt/`](occt/).**

    github.com/Open-Cascade-SAS/OCCT
    commit bb368e271e24f63078129283148ce83db6b9670a
    tag    V7_6_2

The `occt/` directory is that commit's tree, vendored verbatim as ordinary git
content. It was verified at the git-object level: the tree object of `occt/` in
this repository is `806cdcfc9f3701110e1e836727c84a31d0f924b0`, which is exactly
the tree object of upstream commit `bb368e27…`. Every blob hash therefore
matches upstream. It is not a re-tarred or re-normalised copy.

## Reproducibility: a rebuild is functionally equivalent, NOT bit-identical

This must be stated plainly, because the chain above might otherwise be read as
a promise of byte-reproducibility. It is not one.

**The builder image is pulled by floating tag, not by digest.** The recipe runs
`donalffons/opencascade.js` with no tag and no `@sha256:` digest, so it resolves
to whatever `latest` pointed at on the day the build ran. That specific image is
not identified anywhere in the published artifact.

**The OCCT identity survives this; the toolchain identity does not.** The OCCT
pin `bb368e27…` was introduced upstream on 2022-04-30 and was never changed
again through the repository's final commit on 2023-03-27, so any `latest` image
from that window carries the same OCCT commit. The Emscripten pin has no such
stability — across that same window the base image moved through
`emscripten/emsdk` 3.1.9, 3.1.10, 3.1.11, 3.1.10 (reverted), 3.1.12, 3.1.13 and
3.1.14. Which of those compiled the shipped binary cannot be determined from the
pin history alone.

**Nor can it be recovered from the binary.** The shipped `opencascade.wasm` has
been stripped: parsing its section table yields zero custom sections, so the
`producers` section that would name the Emscripten revision is absent.

Consequently: rebuilding from this repository yields a WebAssembly module that is
**functionally equivalent** to the shipped one — same OCCT version, same source,
same binding configuration — but it will **not** be bit-identical, and no claim
of bit-identity is made here.

## Contents

    occt/                                       OCCT source at bb368e27… (tag V7_6_2)
    build-recipe/replicad-opencascadejs-0.23.0/ binding config and build scripts
    build-recipe/opencascade.js-5ff2b750/       Dockerfile and the scripts it invokes
    LICENSE                                     GNU LGPL 2.1
    OCCT_LGPL_EXCEPTION.txt                     Open CASCADE exception to the LGPL

Built artifacts have been omitted from the vendored recipes: they are outputs,
not source, and the relevant one is the file already in your copy of the app.
Upstream CI workflow definitions have also been omitted, as they are not part of
the build recipe.

## Licensing

OCCT is licensed under the **GNU Lesser General Public License, version 2.1**,
with the Open CASCADE exception. Both texts are included at the root of this
repository, and the originals remain in place at `occt/LICENSE_LGPL_21.txt` and
`occt/OCCT_LGPL_EXCEPTION.txt`.

The two build recipes carry their own upstream licences, retained alongside
them: `replicad-opencascadejs` is MIT, and `opencascade.js` is LGPL-2.1.

No Saimek source code is present in this repository, and none of the third-party
material here has been modified.
