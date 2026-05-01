# Light Fusion Rev-A design notes

## Frozen operating points
- LED850 stimulation target: 0.75 A nominal, 10% duty cycle
- Implemented with STCS1A set resistor RSET850 = 0.13 ohm
- STCS1A set equation: I_LED = 0.1 V / RSET
- Therefore U5 nominal current is about 0.769 A with 0.13 ohm
- LED730 starting current: 0.35 A nominal
- Implemented with RSET730 = 0.287 ohm
- Therefore U4 nominal current is about 0.348 A

## Power rail structure
- VBAT: 1-cell LiPo / battery input, expected about 3.0 V to 4.2 V
- VLED: tie directly to VBAT
- 3V3_DIG: generated from VBAT by TPS7A2033PDBVR
- 3V3_A: generated from 3V3_DIG through FB1 ferrite bead, then local analog bulk and decoupling
- AGND: analog ground for OPA381, MCP3564R analog side, VEMD8081 return
- DGND: digital ground for MCU and ADC digital side
- Stitch AGND and DGND together at one quiet point under/near ADC, not in LED current return path

## MCP3564R hookup
- Use internal 2.4 V reference
- Tie VREF- to AGND
- Decouple REFIN+/OUT with local capacitor close to U2
- Keep TIA output to ADC input short and quiet

## TIA starting values
- RF_TIA = 330 kOhm
- CF_TIA = 10 pF
- RADC = 100 ohm
- CADC = 1 nF

## Temperature sensor
- U7 = TMP117AIDRVR
- Place close to LED850 thermal copper, but not under the LED pad
- Use I2C to MCU

## nRF52832 clocks
- Y1 32 MHz crystal with two 12 pF load capacitors
- Y2 32.768 kHz crystal with two 15 pF load capacitors

## Recommended net names
- VBAT
- VLED
- 3V3_DIG
- 3V3_A
- AGND
- DGND
- LED730_A
- LED730_K
- LED850_A
- LED850_K
- PD_ANODE
- PD_CATHODE
- TIA_IN
- TIA_OUT
- ADC_AINP
- MCU_LED730_PWM
- MCU_LED850_PWM
- MCU_SPI_SCK
- MCU_SPI_MOSI
- MCU_SPI_MISO
- MCU_ADC_CS
- MCU_ADC_DRDY
- I2C_TEMP_SCL
- I2C_TEMP_SDA

## Placement priorities
1. PD1 and U3 as close as possible
2. U3 and U2 close together
3. U5 and LED2 on a thermally generous copper area
4. Keep LED current loops and PWM traces away from PD1 / U3 / ADC_AINP
