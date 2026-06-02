# DP65 ZMK firmware scaffold

This repository contains a ZMK user-config style firmware scaffold for DP65. The active ZMK config lives at the repository root (`config/` and `build.yaml`) because ZMK's reusable GitHub Actions workflow expects `west init` and `west update` to run from the repo root.

## Hardware target

- Split 65% keyboard
- nice!nano / nice!nano v2 controller footprint
- nice!view display
- AS5600 magnetic encoder/scrubber target

## Do we need a ZMK git submodule?

No. For ZMK keyboard firmware, the clean approach is **not** to add ZMK as a git submodule and not to vendor a full ZMK clone inside this hardware repository.

This scaffold uses the standard ZMK user-config pattern:

- `config/west.yml` tells `west` to clone ZMK and its dependencies outside this repo during setup/build.
- `build.yaml` describes the GitHub Actions build matrix.
- `.github/workflows/zmk-build.yml` calls ZMK's reusable user-config build workflow.

That keeps this repository focused on DP65 hardware and DP65-specific firmware files.

## Current firmware files

```text
build.yaml
config/
├── west.yml
├── dp65.keymap
└── boards/shields/dp65/
    ├── Kconfig.defconfig
    ├── Kconfig.shield
    ├── dp65.conf
    ├── dp65.dtsi
    ├── dp65.keymap
    ├── dp65.zmk.yml
    ├── dp65_left.overlay
    └── dp65_right.overlay
```

## nice!view support

nice!view support is enabled by stacking ZMK's built-in `nice_view_adapter` and `nice_view` shields with the DP65 shield:

```yaml
include:
  - board: nice_nano_v2
    shield: "dp65_left nice_view_adapter nice_view"
  - board: nice_nano_v2
    shield: "dp65_right nice_view_adapter nice_view"
```

## AS5600 support status

The AS5600 is an I2C magnetic angle sensor, not a normal quadrature EC11 encoder. ZMK currently ships common support for quadrature encoders such as EC11, but a DP65 AS5600 scrubber needs an AS5600 driver/module that converts absolute angle deltas into ZMK sensor/input events.

This scaffold reserves the AS5600 hardware integration point in `dp65.dtsi`:

```dts
&pro_micro_i2c {
    status = "okay";

    as5600: as5600@36 {
        compatible = "ams,as5600";
        reg = <0x36>;
        status = "disabled";
    };
};
```

It is deliberately disabled for now so the base keyboard/display scaffold can build without pretending AS5600 support is complete.

Version 2 should add either:

1. an out-of-tree ZMK module that provides an `ams,as5600` compatible driver and emits ZMK sensor events, or
2. a custom behavior/input driver if the scrubber should do something richer than encoder-style increment/decrement.

## Local build

From the repository root:

```bash
python3 -m venv .venv-zmk
. .venv-zmk/bin/activate
pip install west
west init -l config
west update
west zephyr-export
pip install -r zephyr/scripts/requirements.txt
```

Build the left half:

```bash
west build -s zmk/app -d build/dp65_left -b nice_nano_v2 -- \
  -DSHIELD="dp65_left nice_view_adapter nice_view" \
  -DZMK_CONFIG="$PWD/config"
```

Build the right half:

```bash
west build -s zmk/app -d build/dp65_right -b nice_nano_v2 -- \
  -DSHIELD="dp65_right nice_view_adapter nice_view" \
  -DZMK_CONFIG="$PWD/config"
```

## Flashing

After a successful build, copy the generated UF2 to the nice!nano bootloader drive:

```text
build/dp65_left/zephyr/zmk.uf2
build/dp65_right/zephyr/zmk.uf2
```

Flash one half at a time. Use the `settings_reset` artifact if a controller needs its ZMK settings cleared before pairing.

## Open TODOs before hardware bring-up

- Confirm the row/column GPIO assignments against the finalized KiCad netlist.
- Confirm whether the physical key count is 64 switches plus AS5600 scrubber, or a literal 65 switch positions plus scrubber.
- Add the AS5600 ZMK module/driver and enable the `as5600@36` node.
- Add sensor bindings for scrubber actions once the driver exposes rotation events.
