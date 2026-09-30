## MoTeC C185 Documentation

### Setting up CAN
1. Go to the **Communications** menu
2. Go to `CAN4`
3. Click `New...` (Each source corresponds to its own CAN Frame (so one source per id)
    - Device: Receive Message Block
    - Format: Fixed Binary
    - Alignment: Normal
    - Receive: 2200ms
    - Address Format: Standard
    - Base Address: `CAN-ID`
        - Allow Fast Receive should be checked
4. Go to recieved channels
5. Message type: Compound
6. Under identifiers double click number 1
    - Offset: 8
    - ID: `CAN-ID`
    - Identifier Mask: `FFFF`
7. Then, under channels click 'Add'
    - Channel: Click 'Select' -> 'New'
        - Channel Name: should be descriptive (e.g. 'Temps 1-7' or 'Inverter DC Current')
        - Abbreviation: skip
        - Data Type: choose the matching one
        - Decimal Places: choose what makes sense, if you don't know `2` is a good default
    - Offset: should automatically be set as you add more channels (the offset is from the `0`-th byte, so an offset of `0` reads the `0`-th byte, while an offset of `2` reads the `2`-nd byte)
    - Length: the length of data, usually either `1` or `2`
    - Signed: checked or unchecked based on datasheet/specifications
    - Multipler/Divisor/Adder: set based on datasheet/specifications

### Setting up logging
1. Find the 'Setup Logging' button on the top row (its right next to the three red stacked arrows, its a chart with a blue line going through it)
2. Click 'Add'
3. Select the desired channel
4. Choose the desired rate (check with a lead or Jeevan if you're unsure what the correct rate should be)
5. Click 'Ok'
