# PCM Planning 
  
## Necessary Functionlity
- Ready to Drive (RTD)
- Acceleration Pedal Plausability System (APPS)
- Pulse Width Modulation (PWM)
    - Liquid Cooling 
    - (Radiator) Fans
- Lamp Check 
- Power Limiting Algorithm
- Brake Light
- CAN Communication


### Ready to Drive 
Sounds the horn and enables the acceleration pedals
- Must check if brake pedal is pressed (1)
    - via brake pressure sensors
        - via GPIO (PA1)
    - set "pressed" threshold at around ~200psi 
- Must check if RTD switch is up (2)
    - via GPIO (PA0)
- If condition (1) and (2) are satisfied:
    - enable acceleration pedals (GPIO)
    - sound horn for 2 seconds
        - via GPIO (PB0)
    - enable the TSSI Flasher 
        - via GPIO (PB3)
- Must only run once

### Acceleration Pedal Plausability System
State machine to enable/disable the inverter based on throttle and brake pedals 
- Must check that both rotary pots are outputing a voltage between 0.5V and 4.5V (1)
    - via external SPI ADC
- Must check that both rotary pots are within 10% of each other (2) 
- Must check that both rotary pots have traveled over 25% (3)
- Must check if the brakes are pressed (4)
    - if the brake pressure cannot be read for a duration greater than 100ms, we assume this condition to be met
        - in this scenario condition (5) is TRUE
- Must check if there is implausability in the system (5)
- If conditions (1), (2), (3), and NOT (4) are satisfied:
    - the inverter should be enabled
        - via GPIO (PB1)
    - condition (5) is then FALSE
- If any of conditions (1), (2), or (3) is NOT satisfied while the other three are for a duration greater than 100ms, the inverter must be diabled 
    - via GPIO (PB1)
    - condition (5) is then TRUE
- If condition (4) is ever satisfied the inverter must be disabled not matter the state of any other condition 
    - via GPIO (PB1)
- If condition (5) is ever satisfied the inverter should not be enabled until pedal travel is less than 5%
    - this is independent of condition (4)

### Pulse Width Modulation 
Controls the speed at which the fans spin and the pump pumps

> [!NOTE]
> This is based on RFR26, and is subject to change for RFR27 based on emperical testing/simulation

- Must set the pump duty to 70% at all times (0)
    - via GPIO/PWM through timers (PA9/TIM1_CH2)
- Must check the temperature of the coolant in the liquid cooling system (1)
    - via GPIO (PA4/PA5)
- If condition (1) is < 30 C, fan duty should be set to 0%
    - via GPIO/PWM through timers  (PA8/TIM1_CH1)
- If condition (1) is between 30C and 45 C, fan duty should be set to 60%
- If condition (1) is > 45 C, fan duty should be set to 100%

### Lamp Check 
Flashes all LEDs on statup for 2 seconds 
- Must turn on all LEDs (connected to PCM) on start up for 2 seconds: BMS Fault, IMD Fault, TSS Green, TSS Red
    - via GPIO (PB6)

### Power Limiting Algorithm
Limits the output power of the motor to a max of 80kW

> [!CAUTION]
> This algorithm is in progress and still being developed

### Brake Light 
Turns on the brake light
- Must check if the brakes are pressed (1)
- If condition (1) is met the brake light must be enabled
    - via GPIO (PB10)

### CAN Communication
Recieve and transmit CAN messages
- Must operate at a speed of 500kbps
    - 12mHz external crystal 
- When transmitting must check for acknowledgement 
- When transmitting must check that recpient has at least one open mailbox
