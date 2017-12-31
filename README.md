# Introduction
I'm running a modified Prusa i3 using two e3D v6s on a Bowden setup, originally this printer was built from a reprapguru kit found [here](https://reprapguru.com/). Any and all gcode is made using Slicer or Cura, and modified using a text editor so the fan runs constantly.

# Bowden Dual Extruder Installation
## 3D-Printed Parts
* 2 x [extruder mounts](https://www.thingiverse.com/thing:2078038)
* 2 x [x-carriage mount](https://www.thingiverse.com/thing:1065471)

The x-carriage mount fits the standard Prusa i3 x-carriage, like the one modeled [here](https://www.thingiverse.com/thing:399130) (So no need to install a new x-carriage). I'm also gonna suggest that you print two of the x-carriage mounts, in case one breaks.

## Materials
* Additional Extruder motor (I repurposed the ones from [here](https://www.amazon.com/Geeetech-Dual-Extruder-hotend-nozzle/dp/B075TGY28F/ref=sr_1_1?s=industrial&ie=UTF8&qid=1514663493&sr=1-1&keywords=geeetech+extruder), and you can use the direct drive system if you'd like, although I had a number of issues with it)
* 1 x [RRD fan extender](https://www.amazon.com/AuBreey-Extender-DUAL-EXTRUSION-Printer-Reprap/dp/B07115SX39/ref=sr_1_fkmr0_3?dd=8meGbv0H0ZUQmLQLSJfdzQ%2C%2C&ddc_refnmnt=pfod&ie=UTF8&qid=1514663427&sr=8-3-fkmr0&keywords=ramps+fan+extender&refinements=p_97%3A11292772011)
* 2 x [PTFE fittings](https://www.amazon.com/dp/B01F1XTRGI/ref=sxr_rr_xsim_1?pf_rd_m=ATVPDKIKX0DER&pf_rd_p=3008523062&pd_rd_wg=ZMOeN&pf_rd_r=46MJAA6X7F9JKQ17D5ET&pf_rd_s=desktop-rhs-carousels&pf_rd_t=301&pd_rd_i=B01F1XTRGI&pd_rd_w=6iyha&pf_rd_i=stepper+motor+cable&pd_rd_r=e7ea052b-b13e-4d06-9a6b-0f035aa68d9e&ie=UTF8&qid=1514663624&sr=1)
* 2 x [e3d v6s](https://www.amazon.com/E3D-All-metal-HotEnd-Full-Approximately/dp/B00NAK9JFO/ref=sr_1_2?rps=1&ie=UTF8&qid=1514664069&sr=8-2&keywords=e3d&refinements=p_85%3A2470955011%2Cp_89%3AE3D)

Please note that while these parts satisfy the minimum requirement, redundancies are recommended. The e3D thermistors are sensitive, and can crack very easily. Also note that you'll want to heat the hot ends up before removing the heat breaks (which you'll have to do as you're troubleshooting).

## Electronic/Mechanical Configuration
In-Progress

### Wiring
The RRD extender fits on the Aux 1 pins, where you can follow the below diagram for wiring.

![Alt Text](https://github.com/ajump2/3D_Printing/raw/master/Images/Ramps1_4.png)

This frees the D9 pin (where the fan was connected) for use by the second extruder heater cartridge. As with the first extruder, polarity is not important. The extruder motor connects to the E1 pins (above where the first extruder is connected). Then you'll want to connect the second extruders thermistor to space T2. This can all be seen in the below diagram.

![Alt Text](https://github.com/ajump2/3D_Printing/raw/master/Images/ramps_schematic.png)

Other than the wiring shown here, the ramps board should be wired in the exact same way as for the single extruder setup.

## Firmware/Mechanical Configuration
My marlin files are included in this repository, [here](https://github.com/ajump2/3D_Printing/tree/master/Marlin). Note that the second fan doesn't work (I believe it's a problem with my RRD board), which you can use [this](https://www.geeetech.com/wiki/index.php/Reprap_Ramps1.4_RRD_Fan_Extender) code to test.

``` arduino
void setup() {
 pinMode(11, OUTPUT);
 pinMode(6, OUTPUT);
}

void loop() {
 digitalWrite(11, HIGH);
 digitalWrite(6, LOW);
 delay(5000);
 digitalWrite(11, LOW);
 digitalWrite(6, HIGH);
 delay(5000);
}
```

You'll want to change your motherboard settings found in configuration.h

``` arduino
#ifndef MOTHERBOARD
#define MOTHERBOARD 34
#endif

#define EXTRUDERS 2

#define EXTRUDER_OFFSET_X {0.0, -8.5} // (in mm) for each extruder, offset of the hotend on the X axis
#define EXTRUDER_OFFSET_Y {0.0, 0.0} // (in mm) for each extruder, offset of the hotend on the Y axis
```

Likewise, you'll need to define the number of extruders in Configuration_adv.h

``` arduino
//===========================================================================
//=============================Mechanical Settings===========================
//===========================================================================

// This defines the number of extruders
#define EXTRUDERS 2
```

Since the fan is no longer defined on D9, in pins.h you'll need to change the section,

``` arduino
#if MOTHERBOARD == 33 || MOTHERBOARD == 34
...
#endif// MOTHERBOARD == 33 || MOTHERBOARD == 34
```
to the following,
``` arduino
#define X_STEP_PIN         54
#define X_DIR_PIN          55
#define X_ENABLE_PIN       38
#define X_MIN_PIN           3
#define X_MAX_PIN           2   //2 //Max endstops default to disabled "-1", set to commented value to enable.

#define Y_STEP_PIN         60
#define Y_DIR_PIN          61
#define Y_ENABLE_PIN       56
#define Y_MIN_PIN          14
#define Y_MAX_PIN          15   //15

#define Z_STEP_PIN         46
#define Z_DIR_PIN          48
#define Z_ENABLE_PIN       62
#define Z_MIN_PIN          18
#define Z_MAX_PIN          19

#define Z2_STEP_PIN        36
#define Z2_DIR_PIN         34
#define Z2_ENABLE_PIN      30

#define E0_STEP_PIN        26
#define E0_DIR_PIN         28
#define E0_ENABLE_PIN      24

#define E1_STEP_PIN        36
#define E1_DIR_PIN         34
#define E1_ENABLE_PIN      30

#define SDPOWER            -1
#define SDSS               53
#define LED_PIN            13

#if MOTHERBOARD == 33
#define FAN_PIN            9 // (Sprinter config)
#else
#define FAN_PIN            6 // IO pin. Buffer needed
#endif
#define PS_ON_PIN          12

#ifdef REPRAP_DISCOUNT_SMART_CONTROLLER
#define KILL_PIN           41
#else
#define KILL_PIN           -1
#endif

#define HEATER_0_PIN       10   // EXTRUDER 1
#if MOTHERBOARD == 33
#define HEATER_1_PIN       -1
#else
#define HEATER_1_PIN       9    // EXTRUDER 2 (FAN On Sprinter)
#endif
#define HEATER_2_PIN       -1
#define TEMP_0_PIN         13   // ANALOG NUMBERING
#define TEMP_1_PIN         15   // ANALOG NUMBERING
#define TEMP_2_PIN         -1   // ANALOG NUMBERING
#define HEATER_BED_PIN     8    // BED
#define TEMP_BED_PIN       14   // ANALOG NUMBERING

#ifdef ULTRA_LCD

  #ifdef NEWPANEL
     //encoder rotation values
    #define encrot0 0
    #define encrot1 2
    #define encrot2 3
    #define encrot3 1

    #define BLEN_A 0
    #define BLEN_B 1
    #define BLEN_C 2

    #define LCD_PINS_RS 16
    #define LCD_PINS_ENABLE 17
    #define LCD_PINS_D4 23
    #define LCD_PINS_D5 25
    #define LCD_PINS_D6 27
    #define LCD_PINS_D7 29

    #ifdef REPRAP_DISCOUNT_SMART_CONTROLLER
      #define BEEPER 37
      #define BTN_EN1 31
      #define BTN_EN2 33
      #define BTN_ENC 35
      #define SDCARDDETECT 49
    #else
      //arduino pin which triggers an piezzo beeper
      #define BEEPER 33	 // Beeper on AUX-4

      //buttons are directly attached using AUX-2
      #define BTN_EN1 37
      #define BTN_EN2 35
      #define BTN_ENC 31  //the click

      #define SDCARDDETECT -1  // Ramps does not use this port
    #endif

  #else //old style panel with shift register
    //arduino pin witch triggers an piezzo beeper
    #define BEEPER 33		No Beeper added

    //buttons are attached to a shift register
	// Not wired this yet
    //#define SHIFT_CLK 38
    //#define SHIFT_LD 42
    //#define SHIFT_OUT 40
    //#define SHIFT_EN 17

    #define LCD_PINS_RS 16
    #define LCD_PINS_ENABLE 17
    #define LCD_PINS_D4 23
    #define LCD_PINS_D5 25
    #define LCD_PINS_D6 27
    #define LCD_PINS_D7 29

    //encoder rotation values
    #define encrot0 0
    #define encrot1 2
    #define encrot2 3
    #define encrot3 1


    //bits in the shift register that carry the buttons for:
    // left up center down right red
    #define BL_LE 7
    #define BL_UP 6
    #define BL_MI 5
    #define BL_DW 4
    #define BL_RI 3
    #define BL_ST 2

    #define BLEN_B 1
    #define BLEN_A 0
  #endif
#endif //ULTRA_LCD

#else // RAMPS_V_1_1 or RAMPS_V_1_2 as default (MOTHERBOARD == 3)

#define X_STEP_PIN         26
#define X_DIR_PIN          28
#define X_ENABLE_PIN       24
#define X_MIN_PIN           3
#define X_MAX_PIN          -1    //2

#define Y_STEP_PIN         38
#define Y_DIR_PIN          40
#define Y_ENABLE_PIN       36
#define Y_MIN_PIN          16
#define Y_MAX_PIN          -1    //17

#define Z_STEP_PIN         44
#define Z_DIR_PIN          46
#define Z_ENABLE_PIN       42
#define Z_MIN_PIN          18
#define Z_MAX_PIN          -1    //19

#define E0_STEP_PIN         32
#define E0_DIR_PIN          34
#define E0_ENABLE_PIN       30

#define SDPOWER            48
#define SDSS               53
#define LED_PIN            13
#define PS_ON_PIN          -1
#define KILL_PIN           -1

#ifdef RAMPS_V_1_0 // RAMPS_V_1_0
  #define HEATER_0_PIN     12    // RAMPS 1.0
  #define HEATER_BED_PIN   -1    // RAMPS 1.0
  #define FAN_PIN          11    // RAMPS 1.0
#else // RAMPS_V_1_1 or RAMPS_V_1_2
  #define HEATER_0_PIN     10    // RAMPS 1.1
  #define HEATER_BED_PIN    8    // RAMPS 1.1
  #define FAN_PIN           9    // RAMPS 1.1
#endif
#define HEATER_1_PIN        -1
#define HEATER_2_PIN        -1
#define TEMP_0_PIN          2    // MUST USE ANALOG INPUT NUMBERING NOT DIGITAL OUTPUT NUMBERING!!!!!!!!!
#define TEMP_1_PIN          -1
#define TEMP_2_PIN          -1
#define TEMP_BED_PIN        1    // MUST USE ANALOG INPUT NUMBERING NOT DIGITAL OUTPUT NUMBERING!!!!!!!!!
#endif// MOTHERBOARD == 33 || MOTHERBOARD == 34
```

Using Slicer or Cura, adjust your retraction distance to be as short as possible (1mm or so). Also make sure to run the fan at all times. You can do this by changing any M107 commands in your gcode to M106.

Before changing your unit steps in configuration.h, you should test to see if your extruders are working, and try a test using only one extruder.

If underextruding, or the motor skips, or any problems with feeding; go to your RAMPS Board and adjust the stepper drivers for the extruders. By turning the screws you increase the total current for the extruder motors, this is necessary when upgrading from direct drive as the filament has a longer distance to travel.

If this fails, attempt changing heat settings (I've only tested PLA, and I'm running around 190-195C). If both of these steps fail, move on to the next step.

It's also very likely you'll have to change your extruder steps per unit (no need to define for both extruders, defaults to both).

``` arduino
#define DEFAULT_AXIS_STEPS_PER_UNIT   {80,80,4000,130}  // steps per unit (x,y,z,e)
```



# Headset Designs
As part of a research project, several students and I have been working with the [OpenBCI](http://openbci.com/) Ultracortex Mark IV EEG headset. The headset frame is entirely 3D printed, but can be problematic due to the shape. I've used Blender to cut the medium headset into parts, which should aid printing on smaller machines. The use of supports is necessary with any of these headset files, and the overall headset will need to be glued together. For more information on this project, our project code can be found in this [repository](https://github.com/mahn00b/openbci_testing_e).
