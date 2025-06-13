# Part Number Nomenclature

- [Part Number Nomenclature](#part-number-nomenclature)
  - [Default Field](#default-field)
  - [Family Name](#family-name)
  - [Application](#application)
  - [Code Flash / Work Flash / SRAM / VRAM Quantity](#code-flash--work-flash--sram--vram-quantity)
  - [Pin count](#pin-count)
  - [Hardware Option](#hardware-option)
    - [Body Part](#body-part)
    - [Cluster Part](#cluster-part)
  - [References](#references)

## Default Field

| Field | Description    |
| ----- | -------------- |
| CY    | Cypress Prefix |
| T     | TRAVEO         |

## Family Name

| Field | Description      |
| ----- | ---------------- |
| 2     | Cortex-M4        |
| 3     | Cortex-M7 Single |
| 4     | Cortex-M7 dual   |
| 6     | Cortex-M7 quad   |

## Application

| Field | Description                                |
| ----- | ------------------------------------------ |
| B     | Body                                       |
| C     | Cluster Entry                              |
| D     | Cluster with 2D Graphics                   |
| E     | Cluster with 2D Graphics + LPDDR Interface |

## Code Flash / Work Flash / SRAM / VRAM Quantity

| Field | Description                                      |
| ----- | ------------------------------------------------ |
| 6     | 576 KB / 64 KB / 64 KB                           |
| 7     | 1088 KB / 96 KB / 128 KB                         |
| 9     | 2112 KB / 128 KB / 256 KB                        |
| B     | 4160 KB / 256 KB / 768 KB                        |
| F     | 8384 KB / 256 KB / 1024 KB                       |
| J     | 16768 KB / 512 KB / 2048 KB                      |
| L     | 4160 KB / 128 KB / 512 KB (384KB in 3dl) /2048KB |
| N     | 6336 KB / 128 KB / 640 KB / 4096 KB              |

## Pin count

| Field | Description  |
| ----- | ------------ |
| 3     | 64  |
| 4     | 80  |
| 5     | 100 |
| 7     | 144 |
| 8     | 176 |
| A     | 216 |
| B     | 272 |
| C     | 320 |
| D     | 500 |
| H     | 144 |
| J     | 327 |

## Hardware Option

### Body Part

| Field | Description                     |
| ----- | ------------------------------- |
| B     | eSHE – on, HSM – off, RSA-2048  |
| C     | eSHE – on, HSM – on, RSA-2048   |
| D     | eSHE – ON, HSM – ON, RSA - 4096 |

### Cluster Part

| Field | Description                    |
| ----- | ------------------------------ |
| B     | Security on (HSM), RSA - 3K    |

## References

- [Infineon_Datasheets](https://visteon.sharepoint.com/sites/InfraCore-All/_layouts/15/guestaccess.aspx?share=EmQJeZPm1WJFrUk4R5Lw300BGMSLmjyzUaiX2SVb7lm4mA&e=OYdCDU)
