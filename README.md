# UniXNU-sdm845-common

Shared SoC layer for Qualcomm SDM845 (Napali): Snapdragon 845.
Unofficial; not affiliated with Qualcomm or Apple.

## Status

Skeleton. `dts/sdm845.dtsi` compiles with dtc 1.8.1 and drives UniXNU's
`devicetree.generate()` to a complete ADT, but no kernel has been built or booted.

## Layout

| Path | Contents |
| --- | --- |
| `soc.toml` | SoC profile: name, description, source provenance, component list |
| `dts/sdm845.dtsi` | Chip resources: CPUs, PSCI, GICv3, UFS, TLMM, timer, apps SMMU |
| `configs/sdm845.config` | SoC config fragment, deliberately empty of switches |
| `patches/xnu/series` | SoC-level kernel patch series, currently comment-only |

## Data provenance

Every address below is read from the live device tree of a OnePlus 6 (A6003) and, where
the running Android kernel exposes the same window in `/proc/iomem`, confirmed twice.

| Resource | Window | Note |
| --- | --- | --- |
| GICv3 distributor | `0x17a00000` + `0x10000` | `arm,gic-v3`, `#interrupt-cells = 3` |
| GICv3 redistributor | `0x17a60000` + `0x100000` | `redistributor-stride = 0x20000`, one region |
| UFS host | `0x1d84000` + `0x2500` | `qcom,ufshc`, SPI 265, `lanes-per-direction = 2` |
| UFS PHY | `0x1d87000` + `0xda8` | `qcom,ufs-phy-qmp-v3` |
| UFS ICE | `0x1d90000` + `0x8000` | `qcom,ice`; disabled for the first round |
| TLMM | `0x3400000` + `0xc00000` | **one 12 MiB window**, not three 3 MiB blocks |
| Applications SMMU | `0x15000000` + `0x80000`, TCU `0x150c2000` | `qcom,qsmmu-v500`, **not** MMU-500 |
| Architectural timer | `clock-frequency = 19200000` | also `timebase-frequency` on `/cpus` |

### The TLMM window

Mu-Silicium's `MemoryMapLib.c` maps `0x03500000` + `0x03900000` + `0x03D00000`, three
3 MiB blocks totalling 11 MiB. The device reports `03400000-03ffffff : pinctrl_regs`
(12 MiB) and the device tree gives `reg = <0x03400000 0x00c00000>`. The firmware layout
drops the low 1 MiB — which is exactly where GPIO 0–15 live. Use the single 12 MiB window.

### The UFS SMMU question

The device tree has **no** `iommus` property on `ufshc`, and no node in the tree carries
stream ID `0x100`. Community kernel arguments that set `native-ufs-smmu-stream=0x100`,
`mask=0xf` or `adopt=1` therefore have no device-tree basis; whether UFS traffic goes
through the applications SMMU has to be decided from hardware, not from the arguments.

## Building

This repository is a feed for a device repository, not a standalone build:

```sh
# in the UniXNU checkout, with a device repository that lists this SoC
./unixnu fetch
./unixnu status --json
./unixnu defconfig <device>_defconfig -D ../UniXNU-<device>
./unixnu build kernel
```

## License

BSD-2-Clause-Patent. See `LICENSE`.
