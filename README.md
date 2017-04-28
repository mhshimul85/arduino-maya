# Arduino-Maya communication suite
Software to make an Arduino board and a Maya session communicate. This can be particularly useful to create custom input devices.  
It is programmed in Python and Arduino. It uses pySerial ([https://github.com/pyserial/pyserial](https://github.com/pyserial/pyserial)).

## Directory structure
This repository contains:
* A Maya Python API/PyMEL plugin
    ([src/maya-plugin](https://github.com/giuliom95/arduino-maya/tree/master/src/maya-plugin))
* A Python script to read the serial stream incoming from Arduino and feed it to Maya via socket connection
    ([src/driver](https://github.com/giuliom95/arduino-maya/tree/master/src/driver))
* An Arduino test sketch
    ([src/arduino](https://github.com/giuliom95/arduino-maya/tree/master/src/arduino))

## How to use this
### What do you need
* An Arduino board. _I've used an Arduino UNO v3. You can use every other Arduino board. Just watch out for interrupt mappings_
* A three pin rotary encoder
* Jumper wires
* pySerial ([https://github.com/pyserial/pyserial](https://github.com/pyserial/pyserial))
* Maya. _I've used 2017 v3 Student Edition. I think every other version is fine_
### Quick Start
1. Connect the rotary encoder to the Arduino. Pin mapping:  
Rotary encoder pin | Arduino pin
--- | ---
`A` | `D2`
`B` | `D8`
`REF` | `GND`  
2. Flash the Arduino sketch to you board
3. Put the `src/maya-plugin/arduinomaya.py` plugin into the Maya `plug-ins` folder
4. Load the `arduinomaya.py` plugin within a Maya session
5. Create an `arduinoNode` and connect its `Output` attribute to whichever node you may have in you scene
6. Run the `src/driver/serial2maya.py` Python script
7. Enjoy

## How does this works
We got three entities: the Arduino board, the OS and the Maya session. We need to make them exchange messages:  
![System diagram](https://github.com/giuliom95/arduino-maya/tree/master/docs/images/system_diagram.png)  
The Arduino board can communicate with the computer through a serial data stream via the USB cable and the OS can send commands to a Maya session through the `commandPort` socket built-in in Maya. So we need three software pieces: (1) the firmware on the Arduino board, (2) a program that reads and feeds the serial stream to the `commandPort` socket and (3) a Maya plugin to provide an interface for external communication and to make the data available to its internal data structures.
### 1: The firmware
The Arduino firmware provided simply reads the pulses of a rotary encoder and outputs periodically the number of ticks that the encoder has done in that period. It uses the interrupts mapping of the Arduino UNO board.
### 2: The "bridge" program
It is a simple Python script that uses the `pySerial` lib for Arduino serial communication and the `socket` module to send messages to Maya. It consists in an endless loop that runs the MEL command defined in the Maya plugin passing as arguments the data coming through the serial port.
### 3: The Maya plugin
It adds to Maya a new DG node and a new MEL command.  
The DG node called `arduinoNode` has two visible float attributes: `Output` and `Multiplier`. The `Output` attribute must be connected to the node to control via Arduino (typically a transform or a rig component) while `Multiplier` manages the sensibility of the controls.  
The MEL command called `arduinoUpdateChannel` takes an integer as argument and modifies the `Output` attribute of the `arduinoNode` by adding to it the argument multiplied by the current value of the `Multiplier` attribute.