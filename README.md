# VirtualCPU
Virtual psuedo-pipeline CPU created in logisim

Created for a group project in COMP 273: Intro to Computer Systems (Fall 2014) at McGill University


![Overview shot](/overview.png?raw=true "Overview")


### *Functionality and Flow*
Instructions are stored in the instruction cache, which then enter the instruction register. The instructions flow into the Control Unit, which takes as input all but the left-most function bit. The register input of the instruction are also sent to the register block to write to our output from the register block, depending on the instruction. Part of the instruction, the three rightmost bits of the function code, is also sent into the ALU control block, which outputs an add/sub and a multiplication bit into the ALU. Along with the data from the register block, these two outputs are fed into the ALU. A series of multiplexers and AND gates (with further input from the control unit) determines whether the ALU output is written into the data cache, written back into the register block, or fed into the branch equals / branch not equals controls, along with further input from the control unit and instruction byte, to select branching or PC + 1, which is further fed into a set of registers before being input to the instruction cache. If PC + 1, a counter has been put in place to increment the PC. Data may also be put input straight from the keypad input buffer into the register block, or output straight from the register block to the hexadecimal digit display.


### *Logisim Files*

#### Input Buffer/Keypad
Keypad consists of 11 input pins, 10 of which represent an integer value from 0 to 9 which are decoded into their binary representations and an enter key to load into input buffer. The sum of all of the keypad digits clicked is sent to the input buffer on enter press. 

#### Instruction Register
	Contains 8 D flip-flops, one to store and output each bit of the instruction address such that the value of the bit can only change on a rising clock edge; otherwise, the flip-flop holds the current value of its respective bit. Outputs the instruction byte delivering the appropriate bits to the CU and the register block.

#### Program Counter
	Contains 8 D flip-flops that update upon clock input by taking a byte from the input pins. The byte stored in the PC is then sent to an address that adds 1 to the PC register which is fed into a multiplexer along with the branch target address which selects the branch address if the input from the CU is 0 and the regular PC+1 address if the input from the CU is 1.

#### Control Unit
	The Control Unit takes in the instruction from the IR at address PC and handles the instruction byte by dividing it into its constituent bits. Each set of bits that together form the opcode, function code (only lower 3 bits; left-most bit is extraneous), and operand registers is decoded into output bits that activate the instruction specified components to perform their functions based on the input bits of the instruction byte. 	

#### Register Block
	Has as input 2 register selector bits, a write destination bit, data input, a mode input, and a clock input. Takes in a mode bit that determines whether it will read or write to the register(s) provided. If the register block is set to read, output will be fed into the ALU. If set to write, will store the data input into the destination register.

#### ALU Control Block
	Takes as input the ALU Op-code output from the CU, designating whether ALU control should be generated from the function field (10), whether ALU should test for equality (01), or nothing happens (00,11), and the three right-most bits of the function code in the instruction. Each input section enters a series of AND gates to determine the add/sub and mult bits, which are input into the ALU to determine its operation.

#### 8-bit ALU
	Takes in as input two 8-bit registers as data, an input bit from the CU which determines whether the operation will be addition or subtraction, and another input bit which determines whether the operation performed will be multiplication. Multiplication bit trumps the add/sub bit such that when a 1 is fed into the multiplication input bit, the ALU performs a multiplication regardless of the add/sub bit. Subtraction is performed by taking the two’s complement of the B input (which we get using a multiplexer which selects between the original B input and the NOT B input, and passing a 1 to the carry-in at bit 0), and adding. An XOR gate is connected to the last add/sub ALU bit which outputs the overflow only when: 1) overflow exists, 2) operation performed was an addition. Output register is fed into an OR gate which outputs a 1 in the zero output bit only when all the bits in the output are 0.

#### Multiplier
The 8 bit multiplier is built up from 2 bit multipliers arranged to make 4 bit multipliers which are in turn arranged in our ALU to make the 8 bit multiplier. The method by which the multipliers are arranged is as follows: Since the multiplication of two 4 bit binary numbers, 1111 and 1111 can be expressed as (1100 +0011) x (1100+0011)  which we can expand to (1100x1100)+(1100x0011)+(0011x1100)+(0011x0011) is essentially the addition (along with some shifting of bits) of 2 bit muliplications. Since we have a 2 bit multiplier we can therefore create a 4 bit multiplier following the logic above. We can repeat the process with 4 bit multipliers as our base to create our 8 bit multiplier.

#### RAM
The 8-byte RAM has a single-bit mode input; when mode is set to 0, RAM reads/outputs data; when 1, RAM writes/accepts data into its registers. There is also a clock input put to synchronize the reading/writing of registers in RAM. 4 address input bits are fed into AND gates feeding into the open-control of each 8-bit register to allow, or not, the reading/writing of a particular register; each bit of the 8-bit input is input directly into the register, and can only be input given the open and mode bits are set to 1 for the given register. If write mode is set, the output bits through each register go through a set of OR gates to feed output data from the RAM. This is implemented for both the Instruction and the Data caches. 
	
#### 2-Hex Display
 Takes as input 8 wires, splits into 16 bits for each display digit. Each 7-segment display is capable of displaying hexadecimal digits 0-F. Each bit goes through a logic array of AND and NOT gates to create the necessary digit; those outputs are then passed through respective OR gates to light up the appropriate segments of display for the particular digit.


### *Assembly Language*

#### Assembly Instructions
| *Instruction*     | *Assembly Format* | *Description*                                        |
|-------------------|-------------------|------------------------------------------------------|
| Add               | ADD Rx, Ry        | Add numbers stored in Rx, Ry, store result in Rx     |
| Subtract          | SUB Rx, Ry        | Subtract numbers stored in Rx, Ry, store result in Rx|
| Multiply          | MULT Rx, Ry       | Multiply numbers store din Rx, Ry, store result in Rx|
| Store             | STR Rx, Address   | Store contents of Rx into data cache at Address      |
| Load              | LD Rx, Address    | Load contents at Address from data cache to Rx       |
| Branch if zero    | BZ Rx, Address    | If Rx is zero, go to instruction cache address       |
| Branch if not zero| BNZ Rx, Address   | If Rx is not zero, go to instruction cache address   |
| Print             | PRT Rx            | Output Rx contents to 1-digit digital display buffer |
| Read              | INP Rx            | Store data to Rx from keypad input buffer            |
| Stop              | RETURN            | Program stops running
 *Rx and Ry can be general purpose registers R0 or R1*

#### Instruction Bit Formatting
| *Instruction*  | *OpCode* | *Function/Addr* | *Reg/EndFn* | *Full Binary Definition* |
|----------------|:--------:|:---------------:|:-----------:|:------------------------:|
| ADD Rx, Ry     | 00       | 0000            | xy          | 000000xy                 |
| SUB Rx, Ry     | 00       | 0010            | xy          | 000010xy                 |
| MULT Rx, Ry    | 00       | 0001            | xy          | 000001xy                 |
| PRT Rx         | 00       | 1100            | x-          | 001100xx                 |
| INP Rx         | 00       | 1101            | x-          | 001101xx                 |
| RETURN         | 00       | 1111            | --          | 00111111                 |
| BZ Rx, Address | 01       | addr            | x0          | 01addrx0                 |
| BNZ Rx, Address| 01       | addr            | x1          | 01addrx1                 |
| LD Rx, Address | 10       | addr            | x-          | 10addrxx                 |
| STR Rx, Address| 11       | addr            | x-          | 11addrxx                 |



### *Program to multiply two numbers, one from data cache, other from input buffer; solution is displayed on 1-digit digital display*

#### Using MULT:

| *Instruction* | *Binary* | *RAM Address* |
|---------------|----------|---------------|
| INP R0        | 00110100 | 0001          |
| STR R0, 1001  | 11100100 | 0010          |
| LD R0, 1001   | 10100100 | 0011          |
| INP R1        | 00110111 | 0100          |
| MULT R0, R1   | 00000101 | 0101          |
| PRT R0        | 00110000 | 0110          |
| RETURN        | 00111100 | 0111          |

#### Without MULT:

Case 0001 of RAM must be 1
Case 0100 of RAM must be 0

| *Instruction* | *Binary* | *RAM Address* |
|---------------|----------|---------------|
| INP R0        | 00110100 | 0000          |
| STR R0, 0010  | 11001000 | 0001          |
| INP R1        | 00110110 | 0010          |
| STR R1, 0011  | 00110110 | 0011          |
| LD R1, 0001   | 10000110 | 0100          |
| LD R0, 0010   | 10001000 | 0101          |
| SUB R0, R1    | 00001001 | 0110          |
| STR R0, 0010  | 11001000 | 0111          |
| LD R1, 0011   | 10001110 | 1000          |
| LD R0, 0100   | 10010000 | 1001          |
| ADD R1, R0    | 00000010 | 1010          |
| STR R1, 0100  | 11010010 | 1011          |
| LD R0, 0010   | 10001000 | 1100          |
| BNZ R0, 0100  | 01010001 | 1101          |
| PRT R1        | 00110010 | 1110          |
| RETURN        | 00111100 | 1111          |    
