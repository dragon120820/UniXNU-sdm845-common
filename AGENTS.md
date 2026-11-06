# UniXNU-sdm845-common

Shared Qualcomm SDM845 (Napali) resources for UniXNU device repositories.

## Ownership

This repository owns everything that is the same for every SDM845 handset:

- CPU topology and MPIDR affinities, the architectural timer frequency, PSCI.
- Chip-wide controllers: GICv3, the UFS host/PHY/ICE window, the applications SMMU,
  TLMM, the ARMv8 timer block.
- The `soc.toml` the profile loader reads, and the SoC config fragment.

The device repository keeps what differs between handsets: which CPUs and peripherals
are enabled, RAM and reserved regions, wiring, firmware selection, power policy,
display and partition layout. Peripheral nodes are declared in `status = "disabled"`
here and turned on by the board DTS.

## Sources

Every register window in `dts/sdm845.dtsi` is read back from a live OnePlus 6 (A6003)
device tree (`/sys/firmware/fdt`) and cross-checked against `/proc/iomem` where the
running kernel exposes the same window. Nothing is copied from a vendor dtsi.

Do not add a memory node or `reserved-memory` here: DRAM size differs per SKU and is a
device-repository input.

## Rules

- Use the chip's standard identifier: `sdm845`, `qcom,sdm845`.
- Keep runtime source file names, class names, symbols, bundle IDs and protocol IDs.
- A component ID must exist in UniXNU's `components.toml`; this repository currently
  includes `ArmCpu`, `ArmDTPlatform` and `ArmSmmu4K`.
- `ArmSmmu4K` is registered because the applications SMMU is a real gap, not because a
  driver exists: SDM845 uses `qcom,qsmmu-v500`, which `ArmSmmu4K` does not implement.
- Describe current behaviour in short, plain English. No invented measurements.
