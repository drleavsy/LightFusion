
# Light Fusion Rev-A schematic connectivity sheet

## 1) Power rails
- `VBAT`: 1-cell Li-ion input
- `VLED`: tie directly to `VBAT`
- `3V3_DIG`: output of `U6 TPS7A2033PDBVR`
- `3V3_A`: `3V3_DIG` through `FB1 BLM18AG121SN1D`
- `AGND`: analog ground
- `DGND`: digital ground
- Stitch `AGND` and `DGND` at one quiet point near `U2`

---

## 2) U6 TPS7A2033PDBVR (3.3 V LDO)
- `IN` -> `VBAT`
- `GND` -> `DGND`
- `EN` -> `VBAT` (always on) or MCU-controlled enable net if desired
- `OUT` -> `3V3_DIG`
- `CIN_U6 1uF` from `IN` to `DGND`
- `C3V3_U6 1uF` from `OUT` to `DGND`

---

## 3) FB1 analog rail filter
- `FB1` between `3V3_DIG` and `3V3_A`
- `C3V3_A 4.7uF` from `3V3_A` to `AGND`

---

## 4) U4 STCS1APHR (730 nm driver)
- `VIN` -> `VLED`
- `GND` -> `DGND`
- `OUT` -> `LED1_K`
- `LED1_A` -> `VLED`
- `FB` -> top of `RSET730`
- bottom of `RSET730` -> `DGND`
- `RSET730 = 0.287 ohm`
- `PWM` -> `MCU_LED730_PWM`
- `RPD_U4 100k` from `PWM` to `DGND`
- `CIN_U4 4.7uF` from `VIN` to `DGND`
- `100nF` from `VIN` to `DGND`, place close
- `SLOPE`/soft-start cap footprint optional, DNI initially
- `FAULT`/diagnostic -> optional MCU GPIO if used, otherwise leave per datasheet recommendation

Target current:
- `I_LED730 ≈ 0.1 / 0.287 = 0.348 A`

---

## 5) U5 STCS1APHR (850 nm driver)
- `VIN` -> `VLED`
- `GND` -> `DGND`
- `OUT` -> `LED2_K`
- `LED2_A` -> `VLED`
- `FB` -> top of `RSET850`
- bottom of `RSET850` -> `DGND`
- `RSET850 = 0.13 ohm`
- `PWM` -> `MCU_LED850_PWM`
- `RPD_U5 100k` from `PWM` to `DGND`
- `CIN_U5 4.7uF` from `VIN` to `DGND`
- `100nF` from `VIN` to `DGND`, place close
- `SLOPE`/soft-start cap footprint optional, DNI initially
- `FAULT`/diagnostic -> optional MCU GPIO if used, otherwise leave per datasheet recommendation

Target current:
- `I_LED850 ≈ 0.1 / 0.13 = 0.769 A`

Stim operating point:
- use PWM for `10%` duty target from MCU on `MCU_LED850_PWM`

---

## 6) PD1 VEMD8081 + U3 OPA381AIDGKT
Use classic single-supply transimpedance stage.

Photodiode:
- `PD1_ANODE` -> `AGND`
- `PD1_CATHODE` -> `TIA_IN`

OPA381:
- `V+` -> `3V3_A`
- `V-` -> `AGND`
- `IN-` -> `TIA_IN`
- `IN+` -> `AGND`
- `OUT` -> `TIA_OUT`
- `CDEC_U3 100nF` from `3V3_A` to `AGND`, very close

Feedback network:
- `RF_TIA 330k` between `TIA_OUT` and `TIA_IN`
- `CF_TIA 10pF` in parallel with `RF_TIA`

ADC input filter:
- `RADC 100 ohm` between `TIA_OUT` and `ADC_AINP`
- `CADC 1nF` from `ADC_AINP` to `AGND`

Note:
- Keep `PD1` and `U3` extremely close
- No digital or PWM routing near `TIA_IN`

---

## 7) U2 MCP3564R-E/ST
Recommended single-ended hookup for Rev-A:

Power:
- `AVDD` -> `3V3_A`
- `DVDD` -> `3V3_DIG`
- analog ground pins -> `AGND`
- digital ground pins -> `DGND`
- `CDEC_U2A 100nF` from `DVDD` to `DGND`
- `CDEC_U2B 1uF` from `AVDD` to `AGND`
- `CREF_U2 1uF` from `VREF+/REFOUT` to `AGND`

Analog:
- `AIN+` -> `ADC_AINP`
- `AIN-` -> `AGND`
- use internal 2.4V reference

Digital SPI:
- `SCK` -> `MCU_SPI_SCK`
- `SDI` -> `MCU_SPI_MOSI`
- `SDO` -> `MCU_SPI_MISO`
- `CS` -> `MCU_ADC_CS`
- `IRQ/DRDY` -> `MCU_ADC_DRDY`
- `RESET` -> `MCU_ADC_RST` or pull-up and control in firmware if desired

---

## 8) U7 TMP117AIDRVR
- `V+` -> `3V3_DIG`
- `GND` -> `DGND`
- `SCL` -> `I2C_TEMP_SCL`
- `SDA` -> `I2C_TEMP_SDA`
- `ALERT` -> optional MCU GPIO, otherwise leave open if not used
- `100nF` local decoupling from `V+` to `DGND`
- Add `4.7k` pull-up resistors on `SCL` and `SDA` to `3V3_DIG`
- Place near LED850 thermal copper, not under the LED

---

## 9) U1 NRF52832-QFAA-R
Power:
- all `VDD` pins -> `3V3_DIG`
- all ground pins + exposed pad -> `DGND`
- `CDEC_U1A 100nF` at each supply group
- `CDEC_U1B 4.7uF` local bulk

Clocks:
- `Y1 32MHz` between XL1/XL2 with `2 x 12pF` to `DGND`
- `Y2 32.768kHz` between XC1/XC2 with `2 x 15pF` to `DGND`

Programming:
- `SWDIO`
- `SWCLK`
- `RESET`
- optional `SWO`

Main control nets:
- `MCU_LED730_PWM` -> U4 PWM
- `MCU_LED850_PWM` -> U5 PWM
- `MCU_SPI_SCK` -> U2 SCK
- `MCU_SPI_MOSI` -> U2 SDI
- `MCU_SPI_MISO` -> U2 SDO
- `MCU_ADC_CS` -> U2 CS
- `MCU_ADC_DRDY` -> U2 IRQ/DRDY
- `I2C_TEMP_SCL` -> U7 SCL
- `I2C_TEMP_SDA` -> U7 SDA

RF:
- follow Nordic reference layout for antenna / matching network

---

## 10) Global bulk caps
- `C_BAT 22uF` from `VBAT` to `DGND`
- keep close to power entry / LED driver source area

---

## 11) Frozen operating values
- 730 nm driver current: `~0.348 A`
- 850 nm driver current: `~0.769 A`
- 850 nm stimulation duty: `10%`
- 850 nm average electrical power at stim: approximately `2.9V x 0.769A x 0.10 ≈ 0.22 W`

---

## 12) Suggested net labels
- `VBAT`
- `VLED`
- `3V3_DIG`
- `3V3_A`
- `AGND`
- `DGND`
- `LED730_A`
- `LED730_K`
- `LED850_A`
- `LED850_K`
- `TIA_IN`
- `TIA_OUT`
- `ADC_AINP`
- `MCU_LED730_PWM`
- `MCU_LED850_PWM`
- `MCU_SPI_SCK`
- `MCU_SPI_MOSI`
- `MCU_SPI_MISO`
- `MCU_ADC_CS`
- `MCU_ADC_DRDY`
- `I2C_TEMP_SCL`
- `I2C_TEMP_SDA`
