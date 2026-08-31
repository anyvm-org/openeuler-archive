# openeuler-archive

Re-hosted **openEuler virtual-machine images** for the
[anyvm-org](https://github.com/anyvm-org) image builders.

## Why this repo exists

`openeuler-builder` starts every build by downloading an ~940 MB
`.qcow2.xz` VM image. Getting those bytes onto a GitHub Actions runner has
been the single most expensive step of the build, and no upstream source
solves it:

| Source | Throughput on the CI runner | Time for the 938 MB image |
|--------|------------------------------|---------------------------|
| `repo.openeuler.org` (origin) | 1.14 MB/s | 13m43s |
| `mirrors.aliyun.com` (mirror) | 4.08 MB/s, but unstable | 3m50s in a good run, still going at 19m in a bad one |
| release asset of this repo | runner and asset are in the same cloud | seconds |

The origin is **server-side rate limited**, not far away: it delivers the
same ~1.2 MB/s to a Hong Kong VPS a few hundred kilometres from it as it
does to a US runner, so picking a geographically closer mirror does not
help. `mirrors.aliyun.com` is the only mirror with an overseas CDN edge
(a US client resolves it to a node in Atlanta) and it did cut the download
to 3m50s, but its throughput varies a lot between runs.

Hosting the images as GitHub release assets removes the variable entirely:
the runner and the asset live in the same cloud.

This is the same approach as
[netbsd-archive](https://github.com/anyvm-org/netbsd-archive), which
re-hosts NetBSD images that `archive.netbsd.org` refuses to serve to CI
ranges.

## What is hosted here

Byte-for-byte, **unmodified** copies of the official openEuler VM images
published under
`https://repo.openeuler.org/openEuler-<release>/virtual_machine_img/<arch>/`.
Nothing is repackaged or patched -- this is a download-reliability mirror,
not a fork.

Each image is uploaded together with its upstream `.sha256sum`, and the
sync job verifies the digest before uploading, so an asset here either
matches upstream exactly or never gets published.

| Release | Arches |
|---------|--------|
| 25.09 | `x86_64`, `aarch64`, `riscv64` |
| 24.03-LTS-SP4 | `x86_64`, `aarch64`, `loongarch64` |
| 22.03-LTS-SP4 | `x86_64`, `aarch64` |

Asset names keep the upstream filename, including the `.qcow2.xz` suffix,
so the builder picks the right decompress path:
`openEuler-<release>-<arch>.qcow2.xz`.

See the [releases](../../releases) page for the authoritative list.

## How the builders use it

Point the builder conf's image link at a release asset here:

```sh
VM_VHD_LINK="https://github.com/anyvm-org/openeuler-archive/releases/download/<tag>/openEuler-25.09-x86_64.qcow2.xz"
```

Note that this only covers the IMAGE download. The guest's own `dnf` still
talks to a mirror at build time; `openeuler-builder`'s
`hooks/vm_postBuild.sh` points that at `mirrors.aliyun.com`.

## Adding or refreshing an image

The published release images are immutable -- `openEuler-25.09` stays
`openEuler-25.09` -- so a sync only has to be repeated when a NEW
release/arch is added.

1. Add the `{ release, arch }` pair to the matrix in
   `.github/workflows/sync.yml`.
2. Run the **Sync upstream images** workflow (`workflow_dispatch`) with the
   release tag to publish under. It downloads from upstream, verifies the
   `.sha256sum`, and uploads both files as release assets.
3. Point the builder conf's `VM_VHD_LINK` at the new asset.

## License / provenance

These are unmodified openEuler distribution images. openEuler is released
under the [Mulan PSL v2](http://license.coscl.org.cn/MulanPSL2); the
original copyright and distribution terms apply unchanged. This repository
only re-hosts the bytes for download reliability on CI.
