# Raspberry Pi GPIO Intro with Python

This lesson introduces basic GPIO programming on the Raspberry Pi using Python. Students will wire an LED and a pushbutton, then write Python scripts to control an output and read an input. GPIO pins on the Raspberry Pi use 3.3 V logic, so incorrect wiring can damage the board; for that reason, wiring safety is emphasized throughout this activity.  

## Learning goals

By the end of this lesson, students should be able to:

- Identify the difference between GPIO pins, 3.3 V pins, 5 V pins, and GND pins on a Raspberry Pi header.
- Wire a simple LED output circuit safely using a current-limiting resistor.
- Wire a pushbutton as a digital input and read its state in Python.
- Write Python scripts that use pin setup, output control, input reading, loops, and cleanup commands.
- Use GitHub and Git to save, commit, and push their work from the Raspberry Pi.
## Before you begin

Students should already be able to SSH into their Raspberry Pi using PuTTY or VS Code Remote-SSH, open a terminal on the Pi, and work with a GitHub repository from the command line. They should also have completed a basic Python warmup with printing, variables, input, and simple functions so the focus of this lesson can stay on physical computing rather than Python syntax alone.

## Parts needed

Each student or group needs:

- Raspberry Pi with Raspberry Pi OS
- Power supply and microSD card
- Breadboard
- Jumper wires
- 1 LED
- 2 resistors for the LED and the button, typically 220–330 ohms.
- 1 pushbutton
- SSH access through PuTTY, the Command Prompt or VS Code Remote-SSH.

## Safety first

Read this section completely before connecting anything.

- Raspberry Pi GPIO pins are **3.3 V only**. Do not connect 5 V directly to any GPIO pin.
- Never connect a GPIO pin straight to GND or straight to a power pin unless the circuit is specifically designed for that purpose.
- Always use a resistor with an LED so too much current does not flow through the LED or the Raspberry Pi pin.
- Make wiring changes only when the Pi is powered down or when your program is stopped and you are carefully checking connections.
- If you are unsure where a wire goes, stop and ask for help before powering the circuit.

### Quick safety checks

Before running any code, verify all of the following:

- The LED is connected in series with a resistor. 
- No wire goes from 5 V directly to a GPIO pin.  
- No wire goes from 5 V directly to GND. 
- The GPIO pin numbers in your code match the pins you actually wired.  
- You can point to the 3.3 V pin, one GND pin, and each GPIO pin you are using on the header. 

## Pin plan for this lesson

Use the same pins for the whole class to reduce confusion.

| Purpose | Pin name in code | BCM pin number | Notes |
|---|---|---:|---|
| LED output | `LED_PIN` | 17 | A common starter output pin.   |
| Button input | `BUTTON_PIN` | 27 | A common starter input pin. To be safe, wire a 220 Ohm resistor in the circuit with the button pin.   |
| Ground | GND | n/a | Used for the LED return path.  |
| Power for button | 3.3 V | n/a | Use 3.3 V, not 5 V, for this lesson.   |

## Wiring instructions

### Part A: LED output circuit

Wire the LED circuit first.

1. Place the LED on the breadboard.
2. Identify the long leg of the LED as the anode and the short leg as the cathode.
3. Connect GPIO 17 to the LED circuit.
4. Put a resistor in series with the LED.
5. Connect the other side of the LED circuit to GND.  

A safe way to think about the LED path is:

`GPIO 17 -> resistor -> LED -> GND`

Depending on your layout, the resistor may be placed on either side of the LED as long as it is still in series with the LED. 

### Part B: Button input circuit

After the LED is wired and checked, wire the pushbutton.

1. Place the pushbutton so it bridges the center gap of the breadboard if needed.
2. Connect one side of the button to 3.3 V.
3. Connect the other side of the button to GPIO 27 through the resistor:

   `3.3V -> button -> resistor -> GPIO 27`

5. In software, use an internal pull-down resistor so the pin reads LOW when the button is not pressed and HIGH when it is pressed.  

This lesson does **not** use 5 V for the button circuit. Using 3.3 V keeps the input level appropriate for Raspberry Pi GPIO pins.  

## Recommended workflow

1. SSH into your Raspberry Pi using PuTTY or VS Code Remote-SSH. Navigate to your "Pi Projects" directory.
2. Open the GitHub repository for this lesson or clone it to the current directory. 
3. Create a directory for this assignment, such as `gpio_intro`.
4. Build and check the circuit before running code.
5. Write, test, and revise each Python program one at a time.
6. Commit and push your work when each part functions correctly. 

## Program 1: Turn the LED on and off

Create a file named `led_on_off.py`.

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)

LED_PIN = 17

GPIO.setup(LED_PIN, GPIO.OUT)

print("LED on for 2 seconds...")
GPIO.output(LED_PIN, GPIO.HIGH)
time.sleep(2)

print("LED off.")
GPIO.output(LED_PIN, GPIO.LOW)

GPIO.cleanup()
```

Run the file with:

```bash
python3 led_on_off.py
```

The LED should turn on for about two seconds and then turn off. `GPIO.setmode(GPIO.BCM)` tells Python to use BCM numbering, and `GPIO.cleanup()` resets the pins at the end so later programs start from a clean state.  

## Program 2: Blink the LED in a loop

Create a file named `led_blink.py`.

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)

LED_PIN = 17

GPIO.setup(LED_PIN, GPIO.OUT)

try:
    print("Blinking LED. Press Ctrl+C to stop.")
    while True:
        GPIO.output(LED_PIN, GPIO.HIGH)
        time.sleep(0.5)
        GPIO.output(LED_PIN, GPIO.LOW)
        time.sleep(0.5)

except KeyboardInterrupt:
    print("Stopping program...")

finally:
    GPIO.cleanup()
```

This version adds an infinite loop so the LED blinks continuously until the user stops the program with `Ctrl+C`. The `try / except / finally` structure makes shutdown safer because cleanup still runs even when the user interrupts the script.  

## Program 3: Use a button to control the LED

Create a file named `button_led.py`.

```python
import RPi.GPIO as GPIO
import time

GPIO.setmode(GPIO.BCM)

LED_PIN = 17
BUTTON_PIN = 27

GPIO.setup(LED_PIN, GPIO.OUT)
GPIO.setup(BUTTON_PIN, GPIO.IN, pull_up_down=GPIO.PUD_DOWN)

try:
    print("Press the button to turn the LED on. Ctrl+C to stop.")
    while True:
        button_state = GPIO.input(BUTTON_PIN)

        if button_state == GPIO.HIGH:
            GPIO.output(LED_PIN, GPIO.HIGH)
        else:
            GPIO.output(LED_PIN, GPIO.LOW)

        time.sleep(0.05)

except KeyboardInterrupt:
    print("Stopping program...")

finally:
    GPIO.cleanup()
```

This program reads the button as a digital input and uses its state to decide whether the LED should be on or off. The pull-down setting keeps the button pin at a stable LOW value until the button is pressed, which helps prevent floating input behavior.  

## Troubleshooting

| Problem | Likely cause | What to check |
|---|---|---|
| LED does not turn on | Wiring error, wrong pin, missing resistor path, code not using the wired pin | Recheck LED polarity, resistor placement, GND connection, and `LED_PIN` value.   |
| LED stays on all the time | Logic issue or wiring issue | Confirm the code sets the pin LOW where expected and that the LED is actually connected to the intended GPIO pin.  |
| Button does nothing | Wrong breadboard placement or wrong pin number | Recheck button orientation, 3.3 V connection, and `BUTTON_PIN` value.  |
| Button acts randomly | Floating input | Make sure `GPIO.PUD_DOWN` is used and the wiring matches that setup.   |
| Error about GPIO busy or inconsistent behavior | Previous program did not exit cleanly | Stop the program and rerun after making sure `GPIO.cleanup()` is included.  |

## Reflection questions

Add answers to `NOTES.md` in your assignment folder.

1. Which part of the wiring felt easiest to understand, and which part required the most attention?
2. Why is 3.3 V the correct voltage for Raspberry Pi GPIO work in this lesson?  
3. Why is a resistor used with the LED? 
4. What would happen if the code used the wrong BCM pin number?
5. What is the purpose of `GPIO.cleanup()`? 

## What to submit

Your repository should include:

- `gpio_intro/led_on_off.py`
- `gpio_intro/led_blink.py`
- `gpio_intro/button_led.py`
- `gpio_intro/NOTES.md`

Your `NOTES.md` file should include:

- Your name and class period
- The BCM pins you used
- A short reflection on safety and troubleshooting

## Stretch ideas

- Change the blink timing by storing on-time and off-time in variables.
- Print status messages like `Button pressed` and `Button released` as the code runs.
- Modify the button program so one press toggles the LED instead of only turning it on while held. 
- Add comments to your code explaining the purpose of each major line.
