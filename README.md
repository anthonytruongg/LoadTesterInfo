# Load Tester Source Code
# Intro
The following program is written in C#. This program was written by reverse engineering the already pre-existing hardware. The program was written due to the previous load tester PC motherboard having been damaged. Since the motherboard was damaged, the source program was lost. 

# Hardware Used
- PCIe - DIO24
- SSR-RACK 08
- Keithley 2000 Multimeter
- TDK-Lambda 600-1.3 Power Supply
- USB R232 Converter

# Software Used
## Testing Software
- InstaCal
- NI MAX (LabView)
- Measurement Computing Universal Library

## Programming Software
- C# .NET Framework (WPF)

# How stuff works
## Hardware
The program utilizes the Universal Library from Measurement Computing to make connection to the PCIe DIO24. This is connected to the SSR-RACK08. The universal library allows the communication to the on board relays, specifically PORT C. The DMM is wired into this relay, along with the Power Supply. 

## Load Test
1. Run program.
2. Turn on Power Supply and Multimeter.
3. Insert part to be subjected to voltage load into the test box.
4. Select part model, RT value, and RT tolerance.
5. Press start test.
   
Some notes:
- If test passes, the unit will eject.
- If test does not pass, operator must manually eject part

# Thought Process

## Reverse Engineering
This project required me to reverse engineer the hardware that was already configured. Since the motherboard on the old tester went out, the program was lost. This led me to develop a new program with the pre existing hardware.
I broke this project up into 3 portions to allow me to develop the program successfully. The portions were divided like so:
1. Load Tester Box
2. Relay Box
3. Keithley 2000 Multimeter (COM PORT 4)
4. TDK Lambda Power supply (COM PORT 3)

I saved the power supply for last, since the project required a high voltage load. I started off looking through the load tester box and figuring out what made it work. Luckily, there were still some files left from the previous program that pointed me in the
right direction. I downloaded the instacal program and from there, was able to control the relay box by lighting up the LEDS. D8, D7, D6, and D5 are configured for output signals. This means that the computer can control these inputs manually by sending bits to
the ports.

D1-D4 ports were used for mechanical inputs. The lights would be triggered upon pressing the buttons on the load tester box. After figuring out how the box worked, I began working on the relay box and controlling it. This was done through Measurement Computing Universal Library. They have functions that allow for the programmer to interact with the ports and send specific bits to them. Reading and figuring this out took approximately two days.

After being able to access the relay via programming, I moved onto the DMM. From here, I pulled documentation from Keithley 2000 website and looked at their command/programming section. This section included all the various commands that I would need to successfully communicate with the DMM. I sent a "*IDN?\n" command to verify the connection. This returned a byte string with the devices' information.

After the DMM configured, I made connection with the power supply via TDK Lambda 750W documentation. The power supply also had preset commands that the computer can communciate with via serial ports. This was found through their documentation. Once I got the power supply communicating (via "IDN?\r"), I successfully made connections to all the hardware and started building the program in a fast and efficient manner.

Utilizing my knowledge in object oriented programming, I was able to create different classes for the hardware components. Creating these classes allows for the communication to flow seamlessly. It also allows for future programmers to look through my code and easily piece together how my code works. I think this was the most important aspect, as having all the code work without separate classes causes the syntax of the program to be verbose and not readable. From here, I also added failure handling as well as adding error codes that engineers can read while using the program. I also added specific events required to invoke the high voltage load before the test runs.

Once the program and hardware was successfully interacting with each other, I worked on building out the classes further to meet ES 2329 testing procedures. I will be updating the ES 2329 in the future to follow the newly developed program.

As of 9/1/2023, I am cleaning up and making sure all the code flows smoothly. On top of that, I will need to remove/edit some comments so that the code can be pushed to a production branch. During this phase, the load testing box lid is currently being fixed, so there is the potential to adding functionality with the lid closing. As of current, the test is being ran with the lid open.

After these changes, the load tester will go through qualification to ensure that parts are working and the program does not break down. Changes will be made if this is the case.

# Qualification
## Todo's
- Make changes to ES2329. Specifically, make changes to data tables to include voltages and omit "burn in power select switch" and "RT switch range".
- Create a voltage / current per model excel sheet (V=IR) for extra documentation.
- Create online flow chart.
- Create error code sheet.

## Program Tests
- Unit Tests
- Ensure program works on extra PC with same hardware attached.

## Physical Tests (FAIL)
- Launch program power supply off -> return error code 044.
- Launch program with DMM off -> return error code 043.
- Launch program with relay box off -> return error code 046.
- Remove power wire and start test -> return error code 041.
- Place 534 10K pot but select 1K RT. Start test -> return error code 045.
- Run test (close lid) before voltage level has been reached -> return error code 042.
- Run test via engineering access, and input incorrect formatting -> return error code 047.

## Physical Tests (PASS)
- Test all 533, 534, 534 parts and document into data sheet with respective RT, TOL, and voltage values.
