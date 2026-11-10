# Unofficial fork builds

This fork's `Fork upstream sync and binaries` GitHub Actions workflow checks
KhronosGroup/SPIRV-Tools every six hours. It merges upstream `main` into the
fork's `main` without force-pushing. Independently, it queries upstream GitHub
Releases and selects the highest numbered published, non-prerelease version
matching `v<year>.<number>`, then builds that exact tagged
commit, not upstream HEAD or fork-specific changes. A successful build publishes
a GitHub release with the same version tag, with Windows x64, Linux x64, and macOS Intel x64
ZIPs. Each ZIP contains the CMake install tree, license, and build provenance.
These are unofficial binaries of numbered releases, not Vulkan SDK binaries. Tests are
disabled; the workflow smoke-tests the validator and optimizer.

The first run builds the latest stable numbered version (currently `v2026.4`),
not every historical release. Subsequent runs publish only when the latest numbered
version has no published fork release. Ordinary upstream commits, release
candidates (`.rc*`), development tags, unpublished tags, draft releases,
prereleases, and Vulkan SDK tags do not trigger binary
releases. Fork source syncing continues even when no new version is available.
Old `build-<commit>` releases created by the previous workflow are not removed
automatically; this workflow will not create any more of them.

## Setup

1. Push the workflow to the fork's default `main` branch and enable Actions.
   Forks may have Actions disabled by default. Allow workflow read/write
   permissions in the repository's Actions settings.
2. For reliable upstream sync, create a repository secret `FORK_SYNC_TOKEN`
   containing a token with write access to this fork's contents **and workflow
   files** (a fine-grained token needs Contents and Workflows read/write; a
   classic token needs `repo` and `workflow`). Without it, the built-in token
   can sync ordinary changes but cannot push changes to workflow files.
   Branch rules must also allow the token to push to `main`.
3. Run the workflow manually from Actions for the initial release. Later runs
   are scheduled; GitHub may delay scheduled runs or disable them after
   repository inactivity. A workflow-file push also triggers a run.

If a merge conflicts or a push is rejected, the run fails without force-pushing;
resolve the conflict or branch permission issue and run it again. If any build
fails, no new release is published. Failed or incomplete draft releases are
retried on the next run. Existing published builds are not rebuilt. The
built-in token's pushes do not trigger other push workflows; the optional sync
token can trigger them according to its permissions.