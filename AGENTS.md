## Learned User Preferences

- Include nice!view display support in left-half ZMK firmware (user opted in when asked).
- Commit and push firmware changes to the `dp65-zmk-config` repo at `firmware/dp65-zmk-config`, not the parent DP65 repo.
- Derive ZMK GPIO assignments from `DP65_PCB` schematic nets, not KiCad footprint physical pin numbers.

## Learned Workspace Facts

- DP65 is a split wireless mechanical keyboard; the left half has 30 keys in a 5×7 matrix on a nice!nano (nRF52840).
- ZMK user config lives in `firmware/dp65-zmk-config` (remote: `https://github.com/Umi4Life/dp65-zmk-config`).
- PCB design files are in `DP65_PCB/` (KiCad schematic defines L_ROW*, L_COL*, I2C, and SPI nets).
- For ZMK devicetree, use `&gpioX Y` labels from schematic nRF pins; KiCad physical pin numbers are not `&pro_micro` Arduino D-pin indices on nice!nano.
- nice!nano `&pro_micro` gpio-map only covers pins 0–21; higher physical pins need direct `&gpio` labels (e.g. `&gpio1 11` for P1.11).
- Build target: `nice_nano@2.0.0//zmk` with shields `dp65_left nice_view`; `dp65_left` must be listed first because it defines `nice_view_spi`.
- `build.yaml` needs explicit `artifact-name` entries because the board ID `//` breaks CI artifact rename paths.
- nice!nano external 3.3V (P0.13 MOSFET) powers peripherals like nice!view; ZMK may disable it at boot — use EXT_POWER init-delay and `settings_reset` if the rail stays off.
- AS5600 scroll wheel uses the community driver `zmk-driver-ams-as5600` declared in `west.yml`.
- Matrix diode direction is col2row; row 4 has only two physical keys (cols 2–6 are unpopulated matrix slots).
- Right-half firmware (`dp65_right`) is placeholder/deferred; left half runs as BLE central standalone.
