# Introduction
A controller area network bus (CAN bus) is a vehicle bus standard designed to enable efficient communication primarily between electronic control units (ECUs). Originally developed to reduce the complexity and cost of electrical wiring in automobiles through multiplexing,[1] the CAN bus protocol has since been adopted in various other contexts. This broadcast-based, message-oriented protocol ensures data integrity and prioritization through a process called arbitration, allowing the highest priority device to continue transmitting if multiple devices attempt to send data simultaneously, while others back off. Its reliability is enhanced by differential signaling, which mitigates electrical noise. Common versions of the CAN protocol include CAN 2.0, CAN FD, and CAN XL which vary in their data rate capabilities and maximum data payload sizes.

# Character


# Interface

# CAN Bit Timing
The Nominal Bit Rate (NBR) is the number of bits per second transmitted on the CAN bus
### Equation 1: `Nominal Bit Rate/Time`
$$ NBR = \dfrac{1}{NBT} $$
- NBR = Nominal Bit Rate
- NBT = Nominal Bit Time

The Nominal Bit Time (NBT) is made up of four non-overlapping segments. Each of these segments is made up of an integer number of so called Time Quanta (TQ).

### Equation 2: `Time Quanta`
$$ T_Q = 2*(BRP<5:0>+1)*T_{OSC} = \dfrac{2*(BRP<5:0>1+1)}{F_{OSC}} $$  
#### Can derived
$$ T_Q = (BRP+1)*F_{OSC} $$

### Equation 3: `TQ per NBT`
$$ \dfrac{NBT}{T_Q} = SYNC + PRSEG + PRSEG1 + PRSEG2 $$

### Equation 4: `Sample Point`
$$ SP = \dfrac{PRSEG+PHSEG1}{\dfrac{NBT}{T_Q}}*100 $$

### Equation 5: `Oscillator Tolerance`
$$ (1-df)*(f_{nom} \leq F_{OSC} \leq (1+df)*f_{nom}) $$

### Equation 6: `Condition 1`
$$ df \leq \dfrac{SJW}{2*10*\dfrac{NBT}{T_Q}} $$

### Equation 7: `Condition 2`
$$ df \leq \dfrac{min(PHSEG1,PHSEG2)}{2*(13*\dfrac{NBT}{T_Q}-PHSEG2)} $$


### Equation 8: `Maximum Propagation Delay`
$$ T_{PROP} = 2*(t_{TXD-RXD}+T_{BUS}) $$

![Propagation delay](/src/CAN%20bus/Propagation%20delay.png)

![Elements of nominal bit time](/src/CAN%20bus/Elements_of_nominal_bit_time.png)

# CAN Bit Time Configuration Example
The following example illustrates the configuration of the CAN Bit Time registers. Assuming we want to set up a CAN network in an automobile with the following parameters:  
 • 500kbps Nominal Bit Rate (NBR)  
 • Sample point between 60 and 80% of the Nominal Bit Time (NBT)  
 • 40 meters minimum bus length

Table 1: Step-by-Step Register Configuration Example
Parameter|Register|Constraint|Value|Unit|Equations and Comments
:--------|:------:|:--------:|:---:|:--:|:---------------------
NBT|-|$NBT \geq 1 \mu s$|2|$\mu$s|`Equation 1`  
$F_{OSC}$|-|$F_{OSC} \leq 25$ MHz|16|MHz|Select crystal or resonator frequencyusually 16 or 20MHz work
$T_Q/$Bit|-|5 to 25|16||The sum of the $T_Q$ of all four segments must be between 5 and 25;$\\$selecting 16 $T_Q$ per bit is a good starting point
$T_Q$|-|NBT, $F_{OSC}$|125|ns|`Equation 3`
BRP [5:0]|CNF1|0 to 63|0||`Equation 2`
SYNC|-|Fixed|1|$T_Q$|Defined in ISO 11898-1
PRSEG|CNF2|1 to 8 $T_Q$; $\\$ PRSEG > $T_{PROP}$|7|$T_Q$|`Equation 8` $T_{PROP}$ = 870 ns,$\\$minimum $PRSEG = T_{PROP}/T_Q = 6.96 \space T_Q; \\$ selecting 7 will allow 40 m bus length
PHSEG1|CNF2|1 to 8 $T_Q; \\$ PHSEG1 $\geq$ SJW [1:0]|4|$T_Q$|There are 8 $T_Q$ remaining for PHSEG1+PHSEG2; divide the remaining $T_Q$ in half to maximize SJW [1:0]
PHSEG2|CNF3|2 to 8 $T_Q; \\$ PHSEG2 $\geq$ SJW [1:0]|4|$T_Q$|There are 4 $T_Q$ remaining
SJW [1:0]|CNF1|1 to 4 $T_Q; \\$ SJW [1:0] $\leq$ min(PHSEG1, PHSEG2)|4|$T_Q$|MaximumSJW [1:0] lessens the requirement for the oscillator tolerance
Sample Point|-|Usually between 60 and 80%|69|%|use `Equation 4` to double check the sample point
Oscillator Tolerance Condition 1|-|Double Check|1.25|%|`Equation 6`
Oscillator Tolerance Condition 2|-|Double Check|0.98|%|`Equation 7`; better than 1% crystal oscillator required


# Reference
[1] [CAN Bit Time Calculation](http://www.bittiming.can-wiki.info)  
[2] [CAN Bus wiki](https://en.wikipedia.org/wiki/CAN_bus)  
[3] [MCP25625 CAN Controller with Integrated Transceiver](https://ww1.microchip.com/downloads/aemDocuments/documents/OTH/ProductDocuments/DataSheets/MCP25625-CAN-Controller-Data-Sheet-20005282C.pdf)