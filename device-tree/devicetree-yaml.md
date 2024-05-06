# Device Tree YAML Components

> Make sure to build and test your schemas before creating a patch. See [Testing yaml dtschemas](./testing-schema.md) for more details.

- compatible → `vendor,product`
![fields.png](./assets/fields.png)

- each **cell** is of **32-bit integers**

- `#address-cells` and`#size-cells` → number of cells used for address and size in the **subnode** 
![soc-3.png](./assets/soc-3.png)

- `#interrupt-cells` → number of cells used to encode interrupts specifiers for this interrupt controller
![soc-1.png](./assets/soc-1.png)

- `#clock-cells`, etc
![soc-2.png](./assets/soc-2.png)
