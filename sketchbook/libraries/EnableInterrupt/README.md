# EnableInterrupt
New Arduino interrupt library, designed for all versions of the Arduino.
Functions:

```
enableInterrupt- Enables interrupt on a selected Arduino pin.
disableInterrupt - Disables interrupt on the selected Arduino pin.
```

The EnableInterrupt library is a new Arduino interrupt library, designed for
all versions of the Arduino- at this writing, the Uno (and other ATmega328p-based
boards, like the mini), Due, Leonardo (and other ATmega32u4-based boards, like the
Micro), and Mega2560 (and other ATmega2560-based boards, like the MegaADK).
It enables you to assign an interrupt to pins on your chip that support them,
and presents a common interface to all supported chips. This means that on
the Arduino Uno and Mega you don't give it an interrupt number, as per
http://arduino.cc/en/Reference/attachInterrupt. Rather, your
first argument is a pin number of a pin that's supported on that chip (see
https://github.com/GreyGnome/EnableInterrupt/wiki/Usage#pin--port-bestiary ).

See the Wiki at https://github.com/GreyGnome/EnableInterrupt/wiki/Home .
For detailed usage information see https://github.com/GreyGnome/EnableInterrupt/wiki/Usage .
See the https://github.com/GreyGnome/EnableInterrupt/wiki/Download page to
download the library.

See the examples subdirectory in the distribution or in this Git site for code examples.

For a tutorial on interrupts, see
http://www.engblaze.com/we-interrupt-this-program-to-bring-you-a-tutorial-on-arduino-interrupts/
The posting gets into low-level details on interrupts.

**NOTICE** Many of the interrupt pins on the ATmega processor used in the Uno,
Leonardo, and ATmega2560 are "Pin Change Interrupt pins". This means that under
the sheets, the pins *only* trigger on CHANGE, and a number of pins share a
single interrupt subroutine. The library enables these interrupt types to appear
normal, so that each pin can support RISING, FALLING, or CHANGE, and each pin
can support its own user-defined interrupt subroutine. But there is a *significant*
time between when the interrupt triggers and when the pins are read to determine
what actually happened (rising or falling) and which pin changed. Therefore, these
pins are *not* suitable for fast changing signals, and under the right conditions
such events as a bouncing switch may actually be missed. Caveat Programmer.

## ATmega Processor Interrupt Types
Note that the ATmega processor at the heart of the Arduino Uno/Mega2560/Leonardo
has two different kinds of interrupts: “external”, and “pin change”.
For the list of available interrupt pins and their interrupt types, see the
PORT / PIN BESTIARY, below.

### External Interrupts
There are a varying number of external interrupt pins on the different
processors. The Uno supports only 2 and they are mapped to Arduino pins 2 and 3.
The 2560 supports 6 usable, and the Leonardo supports 5. These interrupts can be
set to trigger on RISING, or FALLING, or both ("CHANGE") signal levels, or on
LOW level. The triggers are interpreted by hardware, so by the time your user
function is running, you know exactly which pin interrupted at the time of the
event, and how it changed. On the other hand, as mentioned there are a limited
number of these pins, especially on the Arduino Uno (and others like it which
use the ATmega328p processor).

### Pin Change Interrupts
On the Arduino Uno (and again, all 328p-based boards) *only*, the pin change
interrupts can be enabled on any or all of the pins. The two pins 2 and 3
support *either* pin change or external interrupts. On 2560-based Arduinos,
there are 18 pin change interrupt pins in addition to the 6 external interrupt
pins. On the Leonardo there are 7 pin change interrupt pins in addition to the
5 external interrupt pins. See PIN BESTIARY below for the pin numbers and other
details.

Pin Change interrupts trigger on all RISING and FALLING signal edges.
Furthermore, the processor's pins, and pin change interrupts, are grouped into
“port”s, so for example on the Arduino Uno there are three ports and therefore
only 3 interrupt vectors (subroutines) available for the entire body of 20 pin
change interrupt pins.

### The Library and Pin Change Interrupts
When an event triggers an interrupt on any interrupt-enabled pin on a port, a
subroutine attached to that pin's port is triggered. It is up to the interrupt
library to set the proper port to receive interrupts for a pin, to determine
what happened when an interrupt is triggered (which pin? ...did the signal rise,
or fall?), to handle it properly (Did we care if the signal fell? Did we care
if it rose?), then to call the programmer's chosen subroutine. This makes the
job of resolving the action on a single pin somewhat complicated. There is a
modest slowdown in the interrupt routine because of this complication. Perhaps
more importantly, there is some latency between the interrupt and the system
determining exactly which pin and what change caused it. So the signal could
have changed by the time the pin's status is read, returning a false reading
back to your sketch. For a review of this issue see
https://github.com/GreyGnome/EnableInterrupt/blob/master/Interrupt%20Timing.pdf

# USAGE:
*enableInterrupt*- Enables interrupt on a selected Arduino pin.
```C
enableInterrupt(uint8_t pinNumber, void (*userFunction)(void), uint8_t mode);
or
enableInterrupt(uint8_t interruptDesignator, void (*userFunction)(void), uint8_t mode);

The arguments are:
* pinNumber - The number of the Arduino pin, such as 3, or A0, or SCK. Note that
these are *not* strings, so when you use A0 for example, do not use quotes.
* interruptDesignator- very much like a pin. See below.
* userFunction - The name of the function you want the interrupt to run. Do not
use a pointer here, just give it the name of your function. See the example code
in the Examples directory.
* mode - What you want the interrupt to interrupt on. For Pin Change Interrupt
pins, the modes supported are RISING, FALLING, or CHANGE.
** RISING - The signal went from "0", or zero volts, to "1", or 5 volts.
** FALLING - The signal went from "1" to "0".
** CHANGE - The signal either rose or fell.

For External Interrupts, the same modes are supported plus the additional mode
of LOW signal level.
** LOW - The signal is at a low level for some time.

Each pin supports only 1 function and 1 mode at a time.
```

*disableInterrupt*- Disables interrupt on a selected Arduino pin.  

```C
disableInterrupt(uint8_t pinNumber);
or
disableInterrupt(uint8_t interruptDesignator);
```

* interruptDesignator: Essentially this is an Arduino pin, and if that's all you want to give
the function, it will work just fine. Why is it called an "interruptDesignator", then? Because
there's a twist: You can perform a bitwise "and" with the pin number and PINCHANGEINTERRUPT
to specify that you want to use a Pin Change Interrupt type of interrupt on those pins that
support both Pin Change and External Interrupts. Otherwise, the library will choose whatever
interrupt type (External, or Pin Change) normally applies to that pin,
with priority to External Interrupt. 

* Believe it or not, the complexity is all because of pins 2 and 3 on the ATmega328-based
Arduinos. Those are the only pins in the Arduino line that can share External or Pin Change
Interrupt types. Otherwise, each pin only supports a single type of interrupt and the
PINCHANGEINTERRUPT scheme changes nothing. This means you can ignore this whole discussion
for ATmega2560- or ATmega32U4-based Arduinos.

It is possible to change the user function after enabling the interrupt (if you
want), by disabling the interrupt and enabling it with a different function.


# PIN / PORT BESTIARY
Theoretically pins 0 and 1 (RX and TX) are supported but as these pins have
a special purpose on the Arduino, their use in this library has not been tested.

## Summary
### Arduino Uno/Duemilanove/etc.
Interrupt Type | Pins
-------------- | --------------
External       | 2 3
Pin Change     | 2-13 and A0-A5
### Arduino Mega2560
Interrupt Type | Pins
-------------- | --------------
External       | 2 3 and 18-21
Pin Change     | 10-15 and A8-A15 and SS, SCK, MOSI, MISO
### Arduino Leonardo
Interrupt Type | Pins
-------------- | --------------
External       | 0-3 and 7
Pin Change     | 8-11 and SCK, MOSI, MISO

## Details
### Arduino Uno

<pre>
Interrupt Pins:
Arduino	External                Arduino Pin Change      Arduino Pin Change
Pin     Interrupt               Pin     Interrupt       Pin     Interrupt
                Port                           Port                     Port
2       INT0    PD2             2       PCINT18 PD2     A0      PCINT8  PC0
3       INT1    PD3             3       PCINT19 PD3     A1      PCINT9  PC1
                                4       PCINT20 PD4     A2      PCINT10 PC2
                                5       PCINT21 PD5     A3      PCINT11 PC3
                                6       PCINT22 PD6     A4      PCINT12 PC4
                                7       PCINT23 PD7     A5      PCINT13 PC5
                                8       PCINT0  PB0
                                9       PCINT1  PB1
                                10      PCINT2  PB2
                                11      PCINT3  PB3
                                12      PCINT4  PB4
                                13      PCINT5  PB5
</pre>

### Leonardo Pins LEONARDO

<pre>
Interrupt pins:
Arduino                         Arduino
Pin     External                Pin     Pin Change
        Interrupt                       Interrupt
               Port                               Port
 3      INT0   PD0              8       PCINT4    PB4
 2      INT1   PD1              9       PCINT5    PB5
 0      INT2   PD2              10      PCINT6    PB6
 1      INT3   PD3              11      PCINT7    PB7
 7      INT6   PE6              SCK/15  PCINT1    PB1
                                MOSI/16 PCINT2    PB2
                                MISO/14 PCINT3    PB3

                                on ICSP:
                                SCK/15:  PCINT1 (PB1)
                                MOSI/16: PCINT2 (PB2)
                                MISO/14: PCINT3 (PB3)

// Map SPI port to 'new' pins D14..D17
static const uint8_t SS   = 17;
static const uint8_t MOSI = 16;
static const uint8_t MISO = 14;
static const uint8_t SCK  = 15;
// A0 starts at 18

</pre>

### ATmega2560 Support

<pre>
External Interrupts ------------------------------------------------------------
The following External Interrupts are available on the Arduino:
Arduino           
  Pin  PORT INT  ATmega2560 pin
  21     PD0  0     43
  20     PD1  1     44
  19     PD2  2     45
  18     PD3  3     46
   2     PE4  4      6
   3     PE5  5      7
 n/c     PE6  6      8  (fake pin 75) **
 n/c     PE7  7      9  (fake pin 76)


Pin Change Interrupts ----------------------------------------------------------

ATMEGA2560 Pin Change Interrupts
Arduino              Arduino              Arduino
  Pin  PORT PCINT     Pin   PORT PCINT     Pin   PORT PCINT
  A8     PK0  16       10     PB4   4       SS     PB0   0
  A9     PK1  17       11     PB5   5       SCK    PB1   1
 A10     PK2  18       12     PB6   6       MOSI   PB2   2
 A11     PK3  19       13     PB7   7       MISO   PB3   3
 A12     PK4  20       14     PJ1  10
 A13     PK5  21       15     PJ0   9
 A14     PK6  22        0     PE0   8 - this one is a little odd. *
 A15     PK7  23
</pre>

The library supports all interrupt pins, even though not all pins to the
ATmega-2560 processor are exposed on the Arduino board. These pins are
supported as "fake pins", and begin with pin 70 (there are 70 pins on the
ATmega 2560 board). The fake pins are as follows:

<pre>
pin: fake70 PJ2 this is Pin Change Interrupt PCINT11
pin: fake71 PJ3 this is Pin Change Interrupt PCINT12
pin: fake72 PJ4 this is Pin Change Interrupt PCINT13
pin: fake73 PJ5 this is Pin Change Interrupt PCINT14
pin: fake74 PJ6 this is Pin Change Interrupt PCINT15
pin: fake75 PE6 this is External Interrupt INT6
pin: fake76 PE7 this is External Interrupt INT7
</pre>

* Note: Arduino Pin 0 is PE0 (PCINT8), which is RX0. Also, it is the only other
pin on another port on PCI1. This would make it very costly to integrate with
the library's code and thus is not supported by this library.  It is the same
pin the Arduino uses to upload sketches, and it is connected to the FT232RL
USB-to-Serial chip (ATmega16U2 on the R3).
