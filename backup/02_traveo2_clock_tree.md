# Clock Tree for APP, BM and HSM

## Resources

- All possible values of all PLL(s) for all variants of TV2 are present as part of [clock_tree_v3.xlsx](/cybersecurity/clock_tree_v3.xlsx)

- Python script used for generating all possible values is [clock_pll_value_generator_v2.py](/cybersecurity/clock_pll_value_generator_v2.py)

Objective:

- Maximum `Fpfd`
- Minimum `Fvco`

Please refer excel for valid:

- `reference_div`
- `output_div`
- `feedback_div`
- `Fpfd`
- `Fvco`

## Generic Terms

- Desired reference clock frequency (`Fref`)
- Desired output frequency (PLL_OUT)(`Fout`)
- Reference Divider (`Q`)
- Feedback Divider (`P`)
- Output Divider (`OUTPUT_DIV`)
- PFD frequency (phase detector frequency)(`Fpfd`)

## Generic Formulas

### Integer

- `Fpfd = Fref/Q`
- `VCO = Fpfd × P`
- `PLL_OUT = VCO / OUTPUT_DIV`

### Fractional

- `PLL_OUT = Fpfd*(P+FRAC_DIV)/OUTPUT_DIV`

## Wait states

### ROM / RAM

- `CPUSS_ROM_CTL.SLOW_WS = ‘0’ for (CLK_MEM ≤ 100 MHz)`. Please note that CLK_MEM might be HF clock in body entry controllers.
- `CPUSS_ROM_CTL.SLOW_WS = ‘1’ for (CLK_MEM > 100 MHz) and (CLK_MEM ≤ CLK_MEM Max)`
- `CPUSS_ROM_CTL.FAST_WS = ‘0’ up to CLK_MEM Max`

## Important notes

1. CLK_PATH0 (with FLL) shall be reserved in all variants for CLK_HF0 in case of fuse blowing like transitioning lifecycles.
2. Clock shall be same across HSM, BM, BL and APP.
3. Recommended SSCG DS is 3% which is maximum but modulation rate shall be 512 as we have audio applications.

## Body Entry

### CYT2B6

#### Recommended configurations for CYT2B6

| Clock            | Source    | Divider | Frequency  | Max Allowed(ECO) |
| ---------------- | --------- | ------- | ---------- | ---------------- |
| CLK_PATH1(PLL80) | ECO       | x       | 80 MHz     | -                |
| CLK_PATH2        | ILO       | x       | 32.768 KHz | -                |
| CLK_HF0          | CLK_PATH1 | 1       | 80 MHz     | 80 MHz           |
| CLK_HF1          | CLK_PATH1 | 1       | 80 MHz     | 80 MHz           |
| CLK_HF2          | CLK_PATH2 | 1       | 32.768 KHz | 2 MHz            |
| CLK_FAST         | CLK_HF0   | 1       | 80 MHz     | 80 MHz           |
| CLK_PERI         | CLK_HF0   | 1       | 80 MHz     | 80 MHz           |
| CLK_SLOW         | CLK_PERI  | 1       | 80 MHz     | 80 MHz           |

##### Sample Code for 2B6

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 0u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 0u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4MAIN_WS = 0u;

    // Dividers
    /***  Set CPUSS dividers as required        ***/
    // FAST = 80,000,000; PERI and SLOW = FAST;
    CPUSS->unCM4_CLOCK_CTL.stcField.u8FAST_INT_DIV = 0u; // no division
    CPUSS->unCM0_CLOCK_CTL.stcField.u8PERI_INT_DIV = 0u; // no division
    CPUSS->unCM0_CLOCK_CTL.stcField.u8SLOW_INT_DIV = 0u; // no division

    // Reference divider value (Q)         =  2 | 3 | 4
    // Feed back divider value (P)         = 30 | 40 | 50 | 45 | 60 | 75 | 60 | 80 | 100
    // Out put divider value   (OUTPUT_DEV)=  3 | 4 | 5
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 2(Q) * 30(P) / 3(OUTPUT_DIV) = 80,000,000Hz
    // Restriction: 170,000,000 < Fvco < 400,000,000
    // This time, Fvco = 16,000,000 *30 /2(Q) = 240,000,000Hz.
    #define CY_SYSTEM_PLL_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL_CONFIG_FEEDBACKDIV (30UL)
    #define CY_SYSTEM_PLL_CONFIG_OUTDIV      (3UL)
```

### CYT2B7 / CYT2B9 / CYT2BL

#### Recommended configurations

| Clock             | Source    | Divider | Frequency  | Max Allowed(ECO) |
| ----------------- | --------- | ------- | ---------- | ---------------- |
| CLK_PATH1(PLL160) | ECO       | x       | 160 MHz    | -                |
| CLK_PATH2         | ILO       | x       | 32.768 KHz | -                |
| CLK_HF0           | CLK_PATH1 | 1       | 160 MHz    | 160 MHz          |
| CLK_HF1           | CLK_PATH1 | 2       | 80 MHz     | 100 MHz          |
| CLK_HF2           | CLK_PATH2 | 1       | 32.768 KHz | 2 MHz            |
| CLK_FAST          | CLK_HF0   | 1       | 160 MHz    | 160 MHz          |
| CLK_PERI          | CLK_HF0   | 2       | 80 MHz     | 100 MHz          |
| CLK_SLOW          | CLK_PERI  | 1       | 80 MHz     | 100 MHz          |

##### Sample Code for 2B7

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4MAIN_WS = 1u;

    // Dividers
    /***  Set CPUSS dividrs as required        ***/
    // FAST = 160,000,000; PERI and SLOW = FAST / 2;
    CPUSS->unCM4_CLOCK_CTL.stcField.u8FAST_INT_DIV = 0u; // no division
    CPUSS->unCM0_CLOCK_CTL.stcField.u8PERI_INT_DIV = 1u; // divided by 2
    CPUSS->unCM0_CLOCK_CTL.stcField.u8SLOW_INT_DIV = 0u; // no division

    // Reference divider value (Q)         =  2 | 3 | 4
    // Feed back divider value (P)         = 40 | 60 | 80
    // Out put divider value   (OUTPUT_DEV)=  2
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 2(Q) * 40(P) / 2(OUTPUT_DIV) = 160,000,000 Hz
    // Restriction: 170,000,000 < Fvco < 400,000,000
    // This time, Fvco = 16,000,000 *40 /2(Q) = 320,000,000.
    #define CY_SYSTEM_PLL_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL_CONFIG_FEEDBACKDIV (40UL)
    #define CY_SYSTEM_PLL_CONFIG_OUTDIV      (2UL)
```

## Body High

### CYT 3BB/4BB

#### Recommended Configuration for 3BB 4BB

| Clock               | Source   | Divider | Frequency  | Max Allowed       |
| ------------------- | -------- | ------- | ---------- | ----------------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 240 MHz    | -                 |
| CLK_PATH2(PLL400_1) | ECO      | x       | 193 MHz    | -                 |
| CLK_PATH3(PLL200_0) | ECO      | x       | 160 MHz    | -                 |
| CLK_PATH4(PLL200_1) | ECO      | x       | 200 MHz    | -                 |
| CLK_PATH5(ILO)      | ILO      | x       | 32.768 KHz | 100 MHz           |
| CLK_HF0             | PLL200_0 | 1       | 160 MHz    | 160 MHz           |
| CLK_HF1             | PLL400_0 | 1       | 240 MHz    | 250 MHz(SSCG)     |
| CLK_HF2             | PLL200_1 | 2       | 100 MHz    | 100 MHz           |
| CLK_HF3             | PLL200_1 | 2       | 100 MHz    | 100 MHz           |
| CLK_HF4             | PLL200_1 | 4       | 50 MHz     | 50 MHz            |
| CLK_HF5             | PLL400_1 | 1       | 193 MHz    | 196.608 MHz(SSCG) |
| CLK_HF6             | PLL200_1 | 1       | 200 MHz    | 200 MHz           |
| CLK_HF7             | ILO      | 1       | 32.768 KHz | 8 MHz             |
| CLK_FAST_0          | CLK_HF1  | 1       | 240 MHz    | 250 MHz (SSCG)    |
| CLK_FAST_1          | CLK_HF1  | 1       | 240 MHz    | 250 MHz (SSCG)    |
| CLK_MEM             | CLK_HF0  | 1       | 160 MHz    | 160 MHz           |
| CLK_SLOW            | CLK_MEM  | 2       | 80 MHz     | 100 MHz           |
| CLK_PERI            | CLK_HF0  | 2       | 80 MHz     | 100 MHz           |

##### Sample Code for 3BB / 4BB

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    /* CLK_FAST_1 */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)
    // Reference divider value (Q)         =  1| 2 | 3 | 4
    // Feed back divider value (P)         = 30| 45 | 60 | 90 | 120 | 135 | 180
    // Out put divider value   (OUTPUT_DEV)=  2 | 3
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 1(Q) * 30(P) / 2(OUTPUT_DIV) = 240,000,000 Hz
    // Restriction: 400,000,000 < Fvco < 800,000,000
    // This time, Fvco = 16,000,000 *30 /1(Q) = 480,000,000.
    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (30UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)
    // Reference divider value (Q)         =  4
    // Feed back divider value (P)         = 193
    // Out put divider value   (OUTPUT_DEV)=  4
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 4(Q) * 193(P) / 4(OUTPUT_DIV) = 193,000,000 Hz
    // Restriction: 400,000,000 < Fvco < 800,000,000
    // This time, Fvco = 16,000,000 *193 /4(Q) = 772,000,000.
    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (193UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (4UL)
    // Reference divider value (Q)         =  2 | 3 | 4
    // Feed back divider value (P)         = 40 | 60 | 80
    // Out put divider value   (OUTPUT_DEV)=  2
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 2(Q) * 40(P) / 2(OUTPUT_DIV) = 160,000,000 Hz
    // Restriction: 170,000,000 < Fvco < 400,000,000
    // This time, Fvco = 16,000,000 *40 /2(Q) = 320,000,000.
    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (40UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)
    // Reference divider value (Q)         =  2 | 3 | 4
    // Feed back divider value (P)         = 25 | 50 | 75 | 100
    // Out put divider value   (OUTPUT_DEV)=  1 | 2
    // Fref                                = 16
    // PLL_OUT = 16,000,000(Feco) / 2(Q) * 50(P) / 2(OUTPUT_DIV) = 200,000,000 Hz
    // Restriction: 170,000,000 < Fvco < 400,000,000
    // This time, Fvco = 16,000,000 *25 /2(Q) = 200,000,000.
    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)
```

### CYT 4BF

#### Recommended Configuration for 4BF

| Clock               | Source   | Divider | Frequency   | Max Allowed       |
| ------------------- | -------- | ------- | ----------- | ----------------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 340 MHz     | -                 |
| CLK_PATH2(PLL400_1) | ECO      | x       | 192 MHz     | -                 |
| CLK_PATH3(PLL200_0) | ECO      | x       | 200 MHz     | -                 |
| CLK_PATH4(PLL200_1) | ECO      | x       | 100 MHz     | -                 |
| CLK_PATH5           | ILO      | x       | 32.768 KHz  | 32.768 KHz        |
| CLK_HF0             | PLL200_0 | 1       | 200 MHz     | 200 MHz           |
| CLK_HF1             | PLL400_0 | 1       | 340 MHz     | 344 MHz(SSCG)     |
| CLK_HF2             | PLL200_1 | 1       | 100 MHz     | 100 MHz           |
| CLK_HF3             | PLL200_0 | 2       | 100 MHz     | 100 MHz           |
| CLK_HF4             | PLL400_1 | 2       | 96 MHz      | 122 MHz(SSCG)     |
| CLK_HF5             | PLL400_1 | 1       | 192 MHz     | 196.608 MHz(SSCG) |
| CLK_HF6             | PLL200_0 | 1       | 200 MHz     | 200 MHz           |
| CLK_HF7             | ILO      | 1       | 32.768 KHz  | 8 MHz             |
| CLK_FAST_0          | CLK_HF1  | 1       | 340 MHz     | 344 MHz (SSCG)    |
| CLK_FAST_1          | CLK_HF1  | 1       | 340 MHz     | 344 MHz (SSCG)    |
| CLK_MEM             | CLK_HF0  | 1       | 200 MHz     | 200 MHz           |
| CLK_SLOW            | CLK_MEM  | 2       | 100 MHz     | 100 MHz           |
| CLK_PERI            | CLK_HF0  | 2       | 100 MHz     | 100 MHz           |

##### Sample Code for 4BF

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM2_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM2_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    /* CLK_FAST_1 */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (85UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (36UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (25UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)
```

### CYT 6BJ

#### Recommended Configuration for 6BJ

| Clock               | Source   | Divider | Frequency   | Max Allowed       |
| ------------------- | -------- | ------- | ----------- | ----------------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 310 MHz     | -                 |
| CLK_PATH2(PLL400_1) | ECO      | x       | 192 MHz     | -                 |
| CLK_PATH3(PLL200_0) | ECO      | x       | 200 MHz     | -                 |
| CLK_PATH4(PLL200_1) | ECO      | x       | 100 MHz     | -                 |
| CLK_PATH5           | ILO      | x       | 32.768 KHz  | 32.768 KHz        |
| CLK_HF0             | PLL200_0 | 1       | 200 MHz     | 200 MHz           |
| CLK_HF1             | PLL400_0 | 1       | 310 MHz     | 344 MHz(SSCG)     |
| CLK_HF2             | PLL200_1 | 1       | 100 MHz     | 100 MHz           |
| CLK_HF3             | PLL200_0 | 2       | 100 MHz     | 100 MHz           |
| CLK_HF4             | PLL400_1 | 2       | 96 MHz      | 122 MHz(SSCG)     |
| CLK_HF5             | PLL400_1 | 1       | 192 MHz     | 196.608 MHz(SSCG) |
| CLK_HF6             | PLL200_0 | 1       | 200 MHz     | 200 MHz           |
| CLK_HF7             | ILO      | 1       | 32.768 KHz  | 8 MHz             |
| CLK_FAST_0          | CLK_HF1  | 1       | 310 MHz     | 344 MHz (SSCG)    |
| CLK_FAST_1          | CLK_HF1  | 1       | 310 MHz     | 344 MHz (SSCG)    |
| CLK_MEM             | CLK_HF0  | 1       | 200 MHz     | 200 MHz           |
| CLK_SLOW            | CLK_MEM  | 2       | 100 MHz     | 100 MHz           |
| CLK_PERI            | CLK_HF0  | 2       | 100 MHz     | 100 MHz           |

##### Sample Code for 6BJ

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM2_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM2_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    /* CLK_FAST_1 */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (155UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (36UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (25UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)
```

## Cluster 2D

### Note

1. FPD-Link shall be used only with PLL400#4 in integer mode and SSCG disabled.
   1. Disable SSCG and fractional divider on PLL400#4 when FPD-Link is used as display output on Display#0 root clock (CLK_HF11) and/or Display#1 root clock (CLK_HF12). Note: Other PLL400 instances cannot be used for Display root clocks.

### CYT 2CL

#### Recommended Configuration for 2CL

| Clock               | Source   | Divider | Frequency | Max Allowed |
| ------------------- | -------- | ------- | --------- | ----------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 196 MHz   | -           |
| CLK_PATH2(PLL200_1) | ECO      | x       | 160 MHz   | -           |
| CLK_PATH3(PLL200_2) | ECO      | x       | 200 MHz   | -           |
| CLK_PATH4           | ILO      | x       | 32.768KHz | -           |
| CLK_HF0             | PLL200_1 | 1       | 160 MHz   | 160 MHz     |
| CLK_HF1             | PLL200_1 | 2       | 80 MHz    | 100 MHz     |
| CLK_HF2             | PLL400_0 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF3             | PLL400_0 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF4             | PLL400_0 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF5             | PLL200_2 | 1       | 200 MHz   | 200 MHz     |
| CLK_HF6             | ILO      | 1       | 32.768KHz | 32.768KHz   |
| CLK_FAST            | CLK_HF0  | 1       | 160 MHz   | 160 MHz     |
| CLK_SLOW            | CLK_PERI | 1       | 80 MHz    | 100 MHz     |
| CLK_PERI            | CLK_HF0  | 2       | 80 MHz    | 100 MHz     |

#### Sample Code for 2CL

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;

    CPUSS->unCM4_CLOCK_CTL.stcField.u8FAST_INT_DIV = 0u; // no division
    CPUSS->unCM0_CLOCK_CTL.stcField.u8PERI_INT_DIV = 1u; // divided by 2
    CPUSS->unCM0_CLOCK_CTL.stcField.u8SLOW_INT_DIV = 0u; // no division

    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (147UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (40UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)
```

### CYT 3DL

#### Recommended Configuration for 3DL

| Clock               | Source   | Divider | Frequency | Max Allowed |
| ------------------- | -------- | ------- | --------- | ----------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 234 MHz   | -           |
| CLK_PATH2(PLL400_1) | ECO      | x       | 196 MHz   | -           |
| CLK_PATH3(PLL400_2) | ECO      | x       | 266 MHz   | -           |
| CLK_PATH4(PLL400_3) | ECO      | x       | 200 MHz   | -           |
| CLK_PATH5(PLL400_4) | ECO      | x       | 224 MHz   | -           |
| CLK_PATH6(PLL200_0) | ECO      | x       | 160 MHz   | -           |
| CLK_PATH7(PLL200_1) | ECO      | x       | 100 MHz   | -           |
| CLK_PATH8(PLL200_2) | ECO      | x       | 160 MHz   | -           |
| CLK_PATH9           | ILO      | x       | 32.768KHz | -           |
| CLK_HF0             | PLL200_0 | 1       | 160 MHz   | 160 MHz     |
| CLK_HF1             | PLL400_0 | 1       | 234 MHz   | 240 MHz     |
| CLK_HF2             | PLL200_1 | 1       | 100 MHz   | 100 MHz     |
| CLK_HF3             | PLL200_0 | 2       | 80 MHz    | 100 MHz     |
| CLK_HF4             | PLL200_1 | 2       | 50 MHz    | 50 MHz      |
| CLK_HF5             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF6             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF7             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF8             | PLL400_2 | 1       | 266 MHz   | 266 MHz     |
| CLK_HF9             | PLL400_2 | 1       | 266 MHz   | 266 MHz     |
| CLK_HF10            | PLL400_3 | 1       | 200 MHz   | 203 MHz     |
| CLK_HF11            | PLL400_4 | 1       | 220 MHz   | 224 MHz     |
| CLK_HF12            | PLL200_2 | 2       | 160 MHz   | 161 MHz     |
| CLK_HF13            | ILO      | 1       | 32.768KHz | 32.768KHz   |
| CLK_FAST_0          | CLK_HF1  | 1       | 234 MHz   | 240 MHz     |
| CLK_MEM             | CLK_HF0  | 1       | 160 MHz   | 160 MHz     |
| CLK_SLOW            | CLK_MEM  | 2       | 80 MHz    | 100 MHz     |
| CLK_PERI            | CLK_HF0  | 2       | 80 MHz    | 100 MHz     |

#### Sample Code for 3DL

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM2_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM2_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (117UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (147UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (133UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (25UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL4_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL4_CONFIG_FEEDBACKDIV (28UL)
    #define CY_SYSTEM_PLL4_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL5_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL5_CONFIG_FEEDBACKDIV (40UL)
    #define CY_SYSTEM_PLL5_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL6_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL6_CONFIG_FEEDBACKDIV (25UL)
    #define CY_SYSTEM_PLL6_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL7_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL7_CONFIG_FEEDBACKDIV (40UL)
    #define CY_SYSTEM_PLL7_CONFIG_OUTDIV      (2UL)
```

### CYT 4DN

#### Recommended Configuration for 4DN

| Clock               | Source   | Divider | Frequency | Max Allowed |
| ------------------- | -------- | ------- | --------- | ----------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 312 MHz   | -           |
| CLK_PATH2(PLL400_1) | ECO      | x       | 196 MHz   | -           |
| CLK_PATH3(PLL400_2) | ECO      | x       | 332 MHz   | -           |
| CLK_PATH4(PLL400_3) | ECO      | x       | 250 MHz   | -           |
| CLK_PATH5(PLL400_4) | ECO      | x       | 220 MHz   | -           |
| CLK_PATH6(PLL200_0) | ECO      | x       | 200 MHz   | -           |
| CLK_PATH7(PLL200_1) | ECO      | x       | 100 MHz   | -           |
| CLK_PATH8           | ILO      | x       | 32.768KHz | -           |
| CLK_HF0             | PLL200_0 | 1       | 200 MHz   | 200 MHz     |
| CLK_HF1             | PLL400_0 | 1       | 312 MHz   | 313 MHz     |
| CLK_HF2             | PLL200_1 | 1       | 100 MHz   | 100 MHz     |
| CLK_HF3             | PLL200_0 | 2       | 100 MHz   | 100 MHz     |
| CLK_HF4             | PLL200_0 | 4       | 50 MHz    | 50 MHz      |
| CLK_HF5             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF6             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF7             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF8             | PLL400_2 | 1       | 332 MHz   | 332 MHz     |
| CLK_HF9             | PLL400_2 | 1       | 332 MHz   | 332 MHz     |
| CLK_HF10            | PLL400_3 | 1       | 250 MHz   | 250 MHz     |
| CLK_HF11            | PLL400_4 | 1       | 220 MHz   | 220 MHz     |
| CLK_HF12            | PLL400_4 | 2       | 220 MHz   | 220 MHz     |
| CLK_HF13            | ILO      | 1       | 32.768KHz | 32.768KHz   |
| CLK_FAST_0          | CLK_HF1  | 1       | 312 MHz   | 313 MHz     |
| CLK_FAST_1          | CLK_HF1  | 1       | 312 MHz   | 313 MHz     |
| CLK_MEM             | CLK_HF0  | 1       | 200 MHz   | 200 MHz     |
| CLK_SLOW            | CLK_MEM  | 2       | 100 MHz   | 100 MHz     |
| CLK_PERI            | CLK_HF0  | 2       | 100 MHz   | 100 MHz     |

#### Sample Code for 4DN

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM2_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM2_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    /* CLK_FAST_1 */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */
    
    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (39UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (147UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (83UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (125UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL4_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL4_CONFIG_FEEDBACKDIV (55UL)
    #define CY_SYSTEM_PLL4_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL5_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL5_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL5_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL6_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL6_CONFIG_FEEDBACKDIV (25UL)
    #define CY_SYSTEM_PLL6_CONFIG_OUTDIV      (2UL)
```

### CYT 4EN

#### Recommended Configuration for 4EN

| Clock               | Source   | Divider | Frequency | Max Allowed |
| ------------------- | -------- | ------- | --------- | ----------- |
| CLK_PATH1(PLL400_0) | ECO      | x       | 312 MHz   | -           |
| CLK_PATH2(PLL400_1) | ECO      | x       | 196 MHz   | -           |
| CLK_PATH3(PLL400_2) | ECO      | x       | 332 MHz   | -           |
| CLK_PATH4(PLL400_3) | ECO      | x       | 266 MHz   | -           |
| CLK_PATH5(PLL400_4) | ECO      | x       | 220 MHz   | -           |
| CLK_PATH6(PLL200_0) | ECO      | x       | 200 MHz   | -           |
| CLK_PATH7(PLL200_1) | ECO      | x       | 104 MHz   | -           |
| CLK_PATH8           | ILO      | x       | 32.768KHz | -           |
| CLK_HF0             | PLL200_0 | 1       | 200 MHz   | 200 MHz     |
| CLK_HF1             | PLL400_0 | 1       | 312 MHz   | 313 MHz     |
| CLK_HF2             | PLL200_1 | 1       | 104 MHz   | 104 MHz     |
| CLK_HF3             | PLL200_0 | 2       | 100 MHz   | 104 MHz     |
| CLK_HF4             | PLL200_0 | 4       | 50 MHz    | 50 MHz      |
| CLK_HF5             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF6             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF7             | PLL400_1 | 1       | 196 MHz   | 200 MHz     |
| CLK_HF8             | PLL400_2 | 1       | 332 MHz   | 332 MHz     |
| CLK_HF9             | PLL400_2 | 1       | 332 MHz   | 332 MHz     |
| CLK_HF10            | PLL400_3 | 1       | 266 MHz   | 266 MHz     |
| CLK_HF11            | PLL400_4 | 1       | 220 MHz   | 220 MHz     |
| CLK_HF12            | PLL400_4 | 2       | 220 MHz   | 220 MHz     |
| CLK_HF13            | PLL200_1 | 1       | 104 MHz   | 104 MHz     |
| CLK_HF14            | ECO      | 1       | 16 MHz    | 32 MHz      |
| CLK_HF15            | ILO      | 1       | 32.768KHz | 32.768KHz   |
| CLK_FAST_0          | CLK_HF1  | 1       | 312 MHz   | 313 MHz     |
| CLK_FAST_1          | CLK_HF1  | 1       | 312 MHz   | 313 MHz     |
| CLK_MEM             | CLK_HF0  | 1       | 200 MHz   | 200 MHz     |
| CLK_SLOW            | CLK_MEM  | 2       | 100 MHz   | 100 MHz     |
| CLK_PERI            | CLK_HF0  | 2       | 100 MHz   | 100 MHz     |

#### Sample Code for 4EN

```c
    // Assumption is eco is 16 - if its other value please change accordingly

    // Wait Cycles
    CPUSS->unROM_CTL.stcField.u2SLOW_WS = 1u;
    CPUSS->unROM_CTL.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM0_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM0_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM1_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM1_CTL0.stcField.u2FAST_WS = 0u;

    CPUSS->unRAM2_CTL0.stcField.u2SLOW_WS = 1u;
    CPUSS->unRAM2_CTL0.stcField.u2FAST_WS = 0u;

    FLASHC->unFLASH_CTL.stcField.u4WS = 1u;


    /* CLK_MEM */
    CPUSS->unMEM_CLOCK_CTL.stcField.u8INT_DIV     = 0u; /* no division */

    /* CLK_SLOW */
    CPUSS->unSLOW_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_PERI */
    CPUSS->unPERI_CLOCK_CTL.stcField.u8INT_DIV    = 1u; /* divided by 2 */

    /* CLK_FAST_0 */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_0_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */

    /* CLK_FAST_1 */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u8INT_DIV  = 0u; /* no division */
    CPUSS->unFAST_1_CLOCK_CTL.stcField.u5FRAC_DIV = 0u; /* no division */
    
    // Common
    #define CY_SYSTEM_PLL_CONFIG_SSCG_DEPTH_3 (0x0f6UL)
    #define CY_SYSTEM_PLL_CONFIG_SSCG_MOD_RATE_512 (3UL)

    #define CY_SYSTEM_PLL0_CONFIG_REFDIV      (1UL)
    #define CY_SYSTEM_PLL0_CONFIG_FEEDBACKDIV (39UL)
    #define CY_SYSTEM_PLL0_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL1_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL1_CONFIG_FEEDBACKDIV (147UL)
    #define CY_SYSTEM_PLL1_CONFIG_OUTDIV      (3UL)

    #define CY_SYSTEM_PLL2_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL2_CONFIG_FEEDBACKDIV (83UL)
    #define CY_SYSTEM_PLL2_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL3_CONFIG_REFDIV      (4UL)
    #define CY_SYSTEM_PLL3_CONFIG_FEEDBACKDIV (133UL)
    #define CY_SYSTEM_PLL3_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL4_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL4_CONFIG_FEEDBACKDIV (55UL)
    #define CY_SYSTEM_PLL4_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL5_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL5_CONFIG_FEEDBACKDIV (50UL)
    #define CY_SYSTEM_PLL5_CONFIG_OUTDIV      (2UL)

    #define CY_SYSTEM_PLL6_CONFIG_REFDIV      (2UL)
    #define CY_SYSTEM_PLL6_CONFIG_FEEDBACKDIV (26UL)
    #define CY_SYSTEM_PLL6_CONFIG_OUTDIV      (2UL)
```
