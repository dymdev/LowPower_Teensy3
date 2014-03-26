#LowPower_Teensy3 Library - Edge

>This is the latest code that should get branched if you want too contribute. Only the latest Teensyduino release will be used. Older versions of Teensyduino may or may not compile and work.

<h4>Currently used Teensyduino Version: 1.18</h4> 

<h3>ChangeLog beta v1.3:</h3>
1.  Added Idle function for lowering current during user waiting loops<br>
2.  Now Low Power "delay" and "delayMicroseconds" sleeps with minimal impact on precision.<br>

<h3>ChangeLog beta v1.2:</h3>
1.  Using Bitband to set peripheral clocks<br>
2.  Added support for IntervalTimer, delay, delayMicroseconds for dynamic CPU scaling<br>
3.  Added support for HardwareSerial at 16,8,4,2 MHz<br>
4.  Changed "Run" function to "CPU", now you can choose the frequency to run the CPU<br>

<h3>ChangeLog beta v1.1:</h3>
1.  Added all digital wakeup capable pins for DeepSleep and Hibernate function<br>
2.  User can now use a callback function for DeepSleep and Hibernate<br>
3.  Added Sleep function, now any interrupt can wake the processor<br>
4.  Improved code performance<br>
5.  New examples for Sleep function<br>

<h3>ChangeLog Stable v1.0:</h3>
1.  Fixed where VLPR was not being retained because of LPWUI bit not being set right<br>
2.  Added feature to enable LPWUI bit to exit VLPR with any interrupt<br>
3.  Fixed issue with VLPR locking up system if being called before exiting VLPR<br>
4.  Disabled USB Regulator Standby mode during Low Power<br>
5.  Cleaned up library code.<br>

<h3>ChangeLog Beta2:</h3>
1.  Added struct to store sleep configurations<br>
2.  Added RTC Alarm wake<br>
3.  Added TSI wake<br>
4.  Put usb regulator in standby for VLPR<br>
5.  Got rid of stand alone LPTMR timer functions<br>
6.  Cleaned up the library and example codes<br>
7.  New examples added<br>
8.  Defined GPIO wake pin names<br>

<h1>General Usage</h1>
<h3>Functions:</h3>
```c 
void CPU(uint32_t freq);

# Allows for dynamic changing of the CPU, BUS and MEM speed to lower avarege 
# power consumption. There are 5 speeds that the user can choose from: 2 MHz, 
# 4 MHz, 8 MHz, 16 MHz, F_CPU MHz. 

# Parameter "uint32_t  freq" can be any of the 5 predifined values above.

# These #defines have been added for the user convenience for "freq" parm, else 
# use the complete frequency.
1.  TWO_MHZ
2.  FOUR_MHZ
3.  EIGHT_MHZ
4.  SIXTEEN_MHZ
5.  F_CPU - defined in core
``` 
```c
void Idle();

# Use this function in any waiting loops such as waiting for Serial data, this 
# will lower the current consumption slightly while having a little impact on 
# performance. This can be used at any CPU speed.
```
```c
void Sleep();

# Most versitle low power mode, any system interrupt will exit this sleep mode. 
# Interrupts must be setup before hand such as attachInterrupt(). Since this 
# sleep mode disables the systick and the cpu is running at 2MHz anything that 
# uses timing must be recalibrated for the lower frequency.
```
```c
uint32_t DeepSleep(uint32_t wakeType, uint32_t time_pin, uint16_t threshold = 0, ISR myCallback = defaultCallback);

uint32_t DeepSleep(uint32_t wakeType, uint32_t time_pin, ISR myCallback);

void DeepSleep(sleep_block_t* configuration);

# Lowest current consumption sleep mode without a reset. Only certian digital 
# Pins or Periphereals can wake the cpu from this sleep mode. You will notice 
# that there are three functions, the two top functions are basic usage where 
# the second one is just an overloaded function. The last one uses a configuration 
# structure so many wake sources can be configured along with many individual 
# configurations.

# Parameter "uint32_t wakeType" - Pin or Peripheal that will wake the mcu
# Parameter "uint32_t time_pin" - Time or Pin number for "wakeType"
# Parameter "uint16_t threshold" - TSI wakeup threshold
# Parameter "ISR myCallback" - optional user callback function

# Paramter "sleep_block_t* configuration" - see below
typedef struct sleep_block_struct {
    /* Structure wake source */
    volatile uint32_t wake_source;      # stores what module or pin wakeup source
    /* Structure RTC Config */          
    unsigned long rtc_alarm;            # RTC wakeup in seconds
    /* Module Wake Type */
    uint32_t modules;                   # can be RTC, LPTMR, GPIO or TSI  
    /* Structure GPIO Config */
    uint16_t gpio_pin;                  # pin that will be used to wake
    uint16_t gpio_mode;                 # not used yet
    /* Structure LPTMR Config */
    uint16_t lptmr_timeout;             # low power timer in msec
    /* Structure TSI Config */
    uint16_t tsi_threshold;             # touch sense threshold value
    uint8_t tsi_pin;                    # touch sense pin
    /* pointer to callback function */
    void (*callback)();                 # user callback function
    sleep_block_struct() : wake_source(0), rtc_alarm(0), modules(0), gpio_pin(0), gpio_mode(0), lptmr_timeout(0), tsi_threshold(0), tsi_pin(0), callback(NULL) {};
} sleep_block_t;

# These #defines have been added for the user convenience for "wakeType":
1.  GPIO_WAKE - wakeup through selected pin
2.  LPTMR_WAKE - wakeup through Low Power Timer
3.  RTCA_WAKE - wakeup through Real Time Clock in seconds
4.  TSI_WAKE - wakeup through selected Touch Sense pin

# These #defines have been added for the user convenience for GPIO pins:
1.  PIN_2
2.  PIN_4
3.  PIN_6
4.  PIN_7
5.  PIN_9
6.  PIN_10
7.  PIN_11
8.  PIN_13
9.  PIN_16
10. PIN_21
11. PIN_22
12. PIN_26
13. PIN_30
14. PIN_33
```
```c
void Hibernate(uint32_t wakeType, uint32_t time_pin, uint16_t threshold = 0, ISR myCallback = defaultCallback);

void Hibernate(uint32_t wakeType, uint32_t time_pin, ISR myCallback);

void Hibernate(sleep_block_t* configuration);

# Lowest current consumption sleep mode with a reset. Only certian digital 
# Pins or Periphereals can wake the cpu from this sleep mode. You will notice 
# that there are three functions, the two top functions are basic usage where 
# the second one is just an overloaded function. The last one uses a configuration 
# structure so many wake sources can be configured along with many individual 
# configurations. 

# Parameter "wakeType" - Pin or peripheal that will wake the mcu
# Parameter "time_pin" - Time or Pin number for "wakeType"
# Parameter "threshold" - TSI wakeup threshold
# Parameter "myCallback" - optional user callback function

# Paramter "sleep_block_t* configuration" - see below
typedef struct sleep_block_struct {
    /* Structure wake source */
    volatile uint32_t wake_source;      # stores what module or pin wakeup source
    /* Structure RTC Config */          
    unsigned long rtc_alarm;            # RTC wakeup in seconds
    /* Module Wake Type */
    uint32_t modules;                   # can be RTC, LPTMR, GPIO or TSI  
    /* Structure GPIO Config */
    uint16_t gpio_pin;                  # pin that will be used to wake
    uint16_t gpio_mode;                 # not used yet
    /* Structure LPTMR Config */
    uint16_t lptmr_timeout;             # low power timer in msec
    /* Structure TSI Config */
    uint16_t tsi_threshold;             # touch sense threshold value
    uint8_t tsi_pin;                    # touch sense pin
    /* pointer to callback function */
    void (*callback)();                 # user callback function
    sleep_block_struct() : wake_source(0), rtc_alarm(0), modules(0), gpio_pin(0), gpio_mode(0), lptmr_timeout(0), tsi_threshold(0), tsi_pin(0), callback(NULL) {};
} sleep_block_t;

# These #defines have been added for the user convenience for "wakeType":
1.  GPIO_WAKE - wakeup through selected pin
2.  LPTMR_WAKE - wakeup through Low Power Timer
3.  RTCA_WAKE - wakeup through Real Time Clock in seconds
4.  TSI_WAKE - wakeup through selected Touch Sense pin

# These #defines have been added for the user convenience for GPIO pins:
1.  PIN_2
2.  PIN_4
3.  PIN_6
4.  PIN_7
5.  PIN_9
6.  PIN_10
7.  PIN_11
8.  PIN_13
9.  PIN_16
10. PIN_21
11. PIN_22
12. PIN_26
13. PIN_30
14. PIN_33
```
```c
void PrintSRS(Stream *port);

# Use this to print what caused the reset of the Teensy. Useful in debugging.

# Parameter "port" can be any Hardware Serial or USB serial.
```
```c
static uint32_t micros();

# Port of the Teensy Core 'micros()' function for use at CPU speeds less than 24MHz.
```
```c
static inline void delay(uint32_t msec);

# Port of the Teensy Core delay('timeout') function for lowering the current
# consumption by sleeping for small bit of time while waiting for the delay to 
# timeout. Also use this if you use the CPU('freq') function since it recalibrates 
# the delay for whatever cpu speed you are at.

# Parameter "uint32_t msec" is delay in milliseconds
```
```c
static void delayMicroseconds(uint32_t usec);

# Port of the Teensy Core delayMicroseconds('timeout') function for lowering the 
# current consumption by sleeping for small bit of time while waiting for the delay 
# to timeout. Also use this if you use the CPU('freq') function since it recalibrates 
# the delay for whatever cpu speed you are at.

# Parameter "uint32_t usec" is delay in microseconds
```
<h3>Examples:</h3>
Here is a basic walk through to use this library in a Arduino sketch. 

First you must include the the library: `#include <LowPower_Teensy3.h>`

Then create a instance of this class: `TEENSY3_LP LP = TEENSY3_LP();`

Now that we have defined the constructor as `LP` we will use this in front of any low power 
function: `LP.DeepSleep(RTCA_WAKE, 5);`

**example sketch: using the "CPU" function**<br>
This just shows the basic usage of the library's `CPU` function, it just sets the Teensy to run
at 2MHz for 5000 milliseconds and then run the cpu at F_CPU for 5000 miiliseconds. Notice the
use of the `LP.delay(5000);`. Use this port of the core `delay` function since it will reconfigure
the `delay` function to use the new dynamically entered cpu speed, in this case 2MHz. Also this delay
will sleep for very small amount of time waiting for the delay to timeout. You can choose from cpu 
speeds listed obove in the function description.
```c
#include <LowPower_Teensy3.h>

TEENSY3_LP LP = TEENSY3_LP();

void setup() {

}

void loop() {
    LP.CPU(TWO_MHZ);
    LP.delay(5000);
    LP.CPU(F_CPU);
    LP.delay(5000); 
}
```

**example sketch: using the "Idle" function:**<br>
The `Idle` function will sleep for a few clock cycles so this makes it great canidate for use in 
"waiting" loops. Even though the it will only sleep for a very short period of time it can save
some power by a cumulative effect. This example uses the `Idle` function when no USB Serial data
is availible.
```c
#include <LowPower_Teensy3.h>

TEENSY3_LP LP = TEENSY3_LP();

void setup() {
  Serial.begin(9600);
}

void loop() {
  char incomingByte;

  if (Serial.available() > 0) {
    incomingByte = Serial.read();
    Serial.print(incomingByte);
  }
  LP.Idle();
}
```

**example sketch: using "Sleep" function:**<br>
The `Sleep` function will sleep the mcu until an interrupt is generated. The interrupt can come from
any interrupt that the user wants which makes this very versitle. This sketch uses the `attachInterrupt'
routine to setup the interrupt that will wake the processor. This sleep mode will disable the systick
inerrupt and put the processor cpu speed at 2MHz so any Timer type interrupts will have to be reconfigured 
for this slower processor speed.
```c
#include <LowPower_Teensy3.h>

TEENSY3_LP LP = TEENSY3_LP();

void callbackhandler() {
  digitalWrite(LED_BUILTIN, HIGH);
  delay(100);
  digitalWrite(LED_BUILTIN, LOW);
}

void setup() {
  Serial.begin(9600);
  pinMode(LED_BUILTIN, OUTPUT);
  pinMode(0, INPUT_PULLUP);
  attachInterrupt(0, callbackhandler, RISING);
}

void loop() {
  LP.Sleep();
}
```

**example sketch: using "DeepSleep" function:**<br>
The `DeepSleep` function puts the processor into lowest current consumtion sleep mode. Only certian
wakeup events are used that are defined above in the function description. This sketch sets up the 
Teensy to sleep for 5 seconds then wake using the built-in RTC. An optional callback handler can be 
added by the user to restore things after waking up is desired -> `LP.DeepSleep(RTCA_WAKE, 5, callback);`.
```c
#include <LowPower_Teensy3.h>

TEENSY3_LP LP = TEENSY3_LP();

void setup() {

}

void loop() {
  LP.DeepSleep(RTCA_WAKE, 5);
  blink();
}

void blink() {
  digitalWrite(LED_BUILTIN, HIGH);
  LP.delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  LP.delay(100)
}
```

**example sketch: using "Hibernate" function:**<br>
The `Hibernate` function puts the processor into lowest current consumtion sleep mode but differs
from `DeepSleep` by it wakes up through a reset event. This means that code below the `Hibernate` 
function will not be called. Only certian wakeup events are used that are defined above in the 
function description. This sketch sets up the Teensy to sleep only to be awaken by the selected 
digital pin. An optional callback handler can be added by the user to restore things after wakup 
is called -> `LP.Hibernate(GPIO_WAKE, PIN_22, callback);`.
```c
#include <LowPower_Teensy3.h>

TEENSY3_LP LP = TEENSY3_LP();

void setup() {
  pinMode(22, INPUT_PULLUP);
  blink();
}

void loop() {
  LP.Hibernate(GPIO_WAKE, PIN_22);
}

void blink() {
  digitalWrite(LED_BUILTIN, HIGH);
  LP.delay(100);
  digitalWrite(LED_BUILTIN, LOW);
  LP.delay(100)
}
```
<h3>Pitfalls and Problems:</h3>
TODO...
