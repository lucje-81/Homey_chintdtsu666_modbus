
Chint DTSU666 (Modbus) – Homey App

Homey Pro app that reads a Chint DTSU666 three-phase kWh meter via an RS485-to-network converter.
Built for a meter in the power supply of an EV charger, featuring charging sessions and a daily counter. The app is read-only: nothing is ever written to the meter.

Connection

The DTSU666 communicates using Modbus RTU over RS485 (default 9600 baud, address 1). The converter exposes this on your network. Two operating modes are supported:

Converter mode	Setting in the app
Modbus TCP gateway (e.g., EW11 "Modbus TCP ⇔ RTU", Waveshare "Modbus TCP to RTU")	Modbus TCP
Transparent / "TCP server" without protocol conversion	Modbus RTU over TCP

If you leave the setting on Automatic, the app will first try Modbus TCP and then RTU over TCP, remembering whichever works.
When adding the device, the app tests the connection and displays the voltage on L1, so you immediately know if everything is working properly.
Note: Only one device is allowed to read the meter on an RS485 bus. Make sure no other program (such as a test tool on your PC) is polling it at the same time.

What the app displays


Value	Register (holding, FLOAT32)	Factor
Power total / L1 / L2 / L3	0x2012 / 0x2014 / 0x2016 / 0x2018	0.1 W
Voltage L1 / L2 / L3	0x2006 / 0x2008 / 0x200A	0.1 V
Current L1 / L2 / L3	0x200C / 0x200E / 0x2010	0.001 A
Power factor (total)	0x202A	0.001
Frequency	0x2044	0.01 Hz
Total energy imported	0x101E	kWh
Total energy exported	0x1028	kWh

In addition, the app calculates:
• Charging (yes/no), Charging session energy (kWh), and Charging session duration (min)
• Energy today (kWh, since midnight)
Total energy (imported) is the main meter for Homey Energy.

Charging sessions

• A session starts as soon as the power exceeds Charging starts above (default 1000 W).
• It stops when the power remains below Charging stops below (default 200 W) for longer than Stop delay (default 300 s). This prevents a brief pause by the car from counting as the end of the session.
• After the session ends, the energy and duration of the last session remain visible until the next start.
Is the meter wired in reverse (negative power while charging)? Enable Invert direction.

Flow cards

• When
	• Charging started (tag: power)
	• Charging stopped (tags: energy kWh, duration min, meter reading start/end)
	• Current on a phase rose above … A (tags: phase, current), useful for monitoring your grid connection
	• Value of [extra register] changed
• And
	• Is / is not charging
	• Charging session energy is / is not greater than … kWh
	• [extra register] is / is not greater than …
• All values are also available as tags and in Insights.
Example: When Charging stopped → Then notification "Car charged: [energy] kWh in [duration] min".

Extra registers

Device settings → Extra registers (read-only), one per line:
name ; input|holding ; address ; type ; factor ; unit
Reactive power ; holding ; 0x201A ; FLOAT32 ; 0.1 ; var
Voltage L1-L2 ; holding ; 0x2000 ; FLOAT32 ; 0.1 ; V
Power factor L1 ; holding ; 0x202C ; FLOAT32 ; 0.001 ;
Addresses can be decimal or hexadecimal (0x…). Types: FLOAT32, UINT16, INT16, UINT32, INT32.

Installation

bash
npm install -g homey
cd chint-dtsu666
npm install
homey login
homey app install
Wees voorzichtig met code.
Next, add a device: DTSU666 energy meter. Enter the IP address and port of the converter, along with the Modbus address of the meter, and click Test connection.

Files

• lib/modbus.ts: Modbus client (TCP and RTU over TCP, with CRC)
• lib/dtsu666.ts: register map, block reading, protocol detection
• lib/custom-registers.ts: extra registers
• drivers/dtsu666/device.ts: polling, charging sessions, daily counter, flow triggers
• drivers/dtsu666/pair/start.html: pairing with connection test
License: MIT.
