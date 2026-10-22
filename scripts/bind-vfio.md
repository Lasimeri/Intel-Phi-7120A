# bind-vfio.sh

Hands the Xeon Phi cards to `vfio-pci`, or takes them back.

```
sudo scripts/bind-vfio.sh              # bind every 8086:225c device
sudo scripts/bind-vfio.sh bind BDF     # one card
sudo scripts/bind-vfio.sh unbind [BDF|all]
```

For each card: unbind whatever driver holds it, set `driver_override` to
`vfio-pci`, reprobe, and check that `vfio-pci` took it; then print the
IOMMU group and its `/dev/vfio/<group>` node, which the udev rule from
`setup-arch.sh` makes group `phi` writable.

This is the by-hand path. After `sudo scripts/setup-arch.sh` the modprobe
option `options vfio-pci ids=8086:225c` claims every card at boot without
it (`docs/results/2026-09-19-boot-without-login.md`), which is also how a
second card added later comes up bound with nothing typed: the ids match
is per device, not per card.

## Only Xeon Phi devices

An explicit BDF is checked against `vendor:device` first and refused unless it
is 8086:225c (7120 series) or 8086:225d (3120 series). Reason: a card whose
PCIe link does not train is absent from the bus, and on a bifurcated CPU slot
its address goes to the neighbour. On 2026-10-21, after a host restart with
the 7120A not enumerating, 0000:2f:00.0 was the RTX 3090 Ti (10de:2203,
`lspci -nn`); binding it would have taken the display GPU from its driver.
