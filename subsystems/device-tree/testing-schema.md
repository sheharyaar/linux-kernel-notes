# Testing yaml dtschemas

1. First test the binding : `make clean && make -j8 dt_binding_check DT_SCHEMA_FILES=Documentation/devicetree/bindings/<path to yaml>`

2. Then check the `dtsi/dts` files that are compatible with this schema. Example : `rg "generic-uhci"` , here **rg** is recursive grep program

```bash
arch/arm/boot/dts/aspeed/aspeed-g4.dtsi
158:			compatible = "aspeed,ast2400-uhci", "generic-uhci";

arch/arm/boot/dts/aspeed/aspeed-g5.dtsi
186:			compatible = "aspeed,ast2500-uhci", "generic-uhci";

```

3. Now set options to cross-compile with the architecture that has the dts files (here-arm) : `export CROSS_COMPILE=aarch64-linux-gnu- ARCH=arm KBUILD_OUTPUT=out/`

4. Now make config file :
   1. First clean your setup (and you have to build your bindings again in step 4)
   2. Then you may have to run `make mrproper` to switch to the new architecture
   3. Then run `make defconfig` for default config
   4. Build the bindings again : `make -j8 dt_binding_check DT_SCHEMA_FILES=Documentation/devicetree/bindings/<path to yaml>`

5. Now test the dts files : `make -j8 dtbs_check DT_SCHEMA_FILES=<path to yaml>`
