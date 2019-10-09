# Egg Incubator Fully Automated

Is an arduino controlled incubator for chicken's eggs. It's purpose is to keep temperature and humidity at defined values, so that the eggs are incubated and the chicks finally hatch after some days.

It might as well be used to incubate other things than chicken's egg like other kinds of eggs (ducks, turtles, alligators, ...), or cultures of bacteria or fungus i.e. to make yoghurt or to leaven yeast/sour dough or to make [Tempeh](https://en.wikipedia.org/wiki/Tempeh).

## Incubation Conditions

Chicken's eggs are incubated for 21 days under temperature and humidity conditions as shown in the table.

| Day   | Temperature [°C] | Humidity [%] |
|:-----:|:----------------:|:------------:|
|  1-17 |      37.8        |    55-50     |
| 18-19 |      37.5        |    55-60     |
| 20-21 |      37.5        |      70      |

Turn the eggs 3 times per day on days 3-17.

The eggs are checked by "x-raying" them with a flashlight on day 7 and 15 (german [schieren](https://de.wikipedia.org/wiki/Schieren_(Biologie))). Dead or unfertilized eggs are sorted out.

## How it works

The arduino constantly measures temperature and humidity. The raw measurements are smoothed using [Holt-Winters double exponential smoothing](https://en.wikipedia.org/wiki/Exponential_smoothing#Double_exponential_smoothing). The smoothed values are then fed into a [PID control loop](https://en.wikipedia.org/wiki/PID_controller).

### Temperature control

The temperature is maintained by dimming the heat source using a MOC3020 - OptoTriac in a 2 seconds cycle. The duty cycle of the heat source is determined by the temperature PID loop. 


### Humidity control

The humidity is maintained by opening and closing an air vent on the incubator using a servo. The servo angle is determined by the humidity PID loop. Inside the incubator is a water filled jar. The water warms up and evaporates, the humidity rises, the vent is opened and the humid air can escape letting dryer air in. The air vent is also needed also to allow fresh air and oxygen to get into the incubator. You have to experiment with the size (water surface) of the jar to reach the humidty setpoint and have the air vent (half) open.

### Fan monitoring

The fan does not need to be controlled, it is constantly running and distributes heat and humidity equally in the incubator. I used a 12cm 12V PC fan operated at 5V, so it runs slowly. The Arduino monitors the fan using it's rpm signal and sets of an alarm if it fails. The fan also cools the heating wire, the heating is turned off, if the fan fails.

## Usage

Using the select button, you can cycle through the different modes of the display.

The first line always displays currently measured values or an alarm message. The second line displays information depending on the selected display mode. To silent the alarm, press any button (select).

1. display current T/H values
2. temperature setpoint, change with up/down left/right
3. humidity setpoint, change with up/down, right changes humidity control mode, on = control air vent, off = humidity regulation and alarm turned off, auto = control air vent if humidity error greater 3% to avoid constant noise from the servo
4. average temperature and standard deviation
5. average humidity and standard deviation
6. average heater dutycycle and current heater power (0-1)
7. air vent state (0-1) and fan speed
8. uptime

Changed values are stored permanently in the EEPROM.

## Setting it up

### Parts

- [Arduino UNO](https://store.arduino.cc/arduino-uno-rev3)
- [LCD shield](https://www.dfrobot.com/wiki/index.php/Arduino_LCD_KeyPad_Shield_(SKU:_DFR0009)  
  used with [LiquidCrystal library](https://www.arduino.cc/en/Reference/LiquidCrystal)
- [DHT22](https://www.sparkfun.com/datasheets/Sensors/Temperature/DHT22.pdf) temperature and humidity probe used with DHT library
- 12cm PC fan, operated at 5V for low speed
- some Bulb as heating element
- MOC3020 - OptoTriac as heating dimmer switch
- 5V regulator like an 7805 to power the fan
- [SG90](http://akizukidenshi.com/download/ds/towerpro/SG90.pdf) Servo to open/close the air vent, [SoftwareServo library](https://playground.arduino.cc/ComponentLib/Servo)
- mirco buzzer
- 12V Powersupply (1.5A)
- styrofoam box

### Wiring

Have a look into the code for the pin numers, starting at line 10.

The LCD uses the pins 8, 9, 4, 5, 6, 7 and 10 for the background light and A0 for the input keys.

The fan tacho signal is connected to pin 2 and it is powered with 5V from the 7805 regulator.

The heater is powered with 220V AC, the MOC3020 - OptoTriac is used as a dimmer switch with it's gate on pin 13.

The DHT22 T/H probe has it's data pin connected on pin 12, a 10k pullup resistor might be neccessary. 

The air vent servo is connected to pin 11.

The buzzer is connected between ground and A2.

![schematic](incubator_schematic.png)

### PID tuning

The PID coefficients in the code are those I used in my setup. They are probably not appropiate for you setup. You have to tune them.

- [HC12 tuning algorithm](http://www.iaeng.org/publication/WCECS2011/WCECS2011_pp463-468.pdf)
- [Ziegler-Nichols](https://en.wikipedia.org/wiki/Ziegler%E2%80%93Nichols_method) 

 Ziegler-Nichols does very well for this machine.
 
 Set the the I and D coefficients to 0 and increase the P gain until the controlled values starts oscillating. You can monitor them using the [Adruino Plotter](https://learn.adafruit.com/experimenters-guide-for-metro/circ08-using%20the%20arduino%20serial%20plotter). 
To this ultimate gain apply [Ziegler-Nichols](https://en.wikipedia.org/wiki/Ziegler%E2%80%93Nichols_method) formulas to get the P, I and D coefficients.

### Calibration

There is a `T_OFFSET` in line 11. Set it to 0 and let the system get into a stable state (standard deviation of temperature < 0.5 and decreasing). Now drop a mercury fever thermometer into the incubator and let it settle for a couple of minutes. Compare the temperatures measured by the incubator and measured by the fever thermometer and calculate the offset.

Use a mercury fever thermometer because it cheap, more accurate than an electronic one and it comes calibrated because it's a medical device meant to be used on humans. Be aware the it is a maximum thermometer.

## How does it look like?

![control unit](arduino.jpg)

Arduino with LCD shield, below 5V regulator with heatsink and the MOSFET

![air vent](air-vent.jpg)

air vent made from clear plastic, the servo turns the disk of the vent to open/close it

![heater and sensor](fan-heater-probe.jpg)

fan with heating wire in the back, T/H on the left

![heater close up](heater.jpg)

close up of fan with heating wire

## Hatching Chicks

![hatching chicks](hatching1.jpg)

![hatching chicks](hatching2.jpg)



