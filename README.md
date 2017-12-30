# Introduction
I'm running a modified Prusa i3 using two e3D v6s on a Bowden setup, originally this printer was built from a reprapguru kit found [here](https://reprapguru.com/). Any and all gcode is typically made using Slicer or Cura, and modified using a text editor so the fan runs constantly.

# Bowden Installation
## 3D-Printed Parts
* 2 x [extruder mounts](https://www.thingiverse.com/thing:2078038)
* [x-carriage](https://www.thingiverse.com/thing:1065471)

## Materials
* Additional Extruder motor (I repurposed the ones from [here](https://www.amazon.com/Geeetech-Dual-Extruder-hotend-nozzle/dp/B075TGY28F/ref=sr_1_1?s=industrial&ie=UTF8&qid=1514663493&sr=1-1&keywords=geeetech+extruder), and you can use the direct drive system if you'd like, although I had a number of issues with it)
* [RRD fan extender](https://www.amazon.com/AuBreey-Extender-DUAL-EXTRUSION-Printer-Reprap/dp/B07115SX39/ref=sr_1_fkmr0_3?dd=8meGbv0H0ZUQmLQLSJfdzQ%2C%2C&ddc_refnmnt=pfod&ie=UTF8&qid=1514663427&sr=8-3-fkmr0&keywords=ramps+fan+extender&refinements=p_97%3A11292772011)
* 2 x [PTFE fittings](https://www.amazon.com/dp/B01F1XTRGI/ref=sxr_rr_xsim_1?pf_rd_m=ATVPDKIKX0DER&pf_rd_p=3008523062&pd_rd_wg=ZMOeN&pf_rd_r=46MJAA6X7F9JKQ17D5ET&pf_rd_s=desktop-rhs-carousels&pf_rd_t=301&pd_rd_i=B01F1XTRGI&pd_rd_w=6iyha&pf_rd_i=stepper+motor+cable&pd_rd_r=e7ea052b-b13e-4d06-9a6b-0f035aa68d9e&ie=UTF8&qid=1514663624&sr=1)
* 2 x [e3d v6s](https://www.amazon.com/E3D-All-metal-HotEnd-Full-Approximately/dp/B00NAK9JFO/ref=sr_1_2?rps=1&ie=UTF8&qid=1514664069&sr=8-2&keywords=e3d&refinements=p_85%3A2470955011%2Cp_89%3AE3D)

Please note that while these parts satisfy the minimum requirement, redundancies are recommended. The e3D thermistors are sensitive, and can crack very easily. Also note that you'll want to heat the hot ends up before removing the heat breaks (which you'll have to do as you're troubleshooting).

## Electronic/Mechanical Configuration
In-Progress

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

Using Slicer or Cura, adjust your retraction distance to be as short as possible (1mm or so). Also make sure to run the fan at all times. You can do this by changing any M107 commands in your gcode to M106.

Before changing your unit steps in configuration.h, you should test to see if your extruders are working, and try a test using only one extruder.

If underextruding, or the motor skips, or any problems with feeding; go to your ARMS Board and adjust the stepper drivers for the extruders. By turning the screws you increase the total current for the extruder motors, this is necessary when upgrading from direct drive as the filament has a longer distance to travel.

If this fails, attempt changing heat settings (I've only tested PLA, and I'm running around 190-195C). If both of these steps fail, move on to the next step.

It's also very likely you'll have to change your extruder steps per unit (no need to define for both extruders, defaults to both).

``` arduino
#define DEFAULT_AXIS_STEPS_PER_UNIT   {80,80,4000,130}  // steps per unit (x,y,z,e)
```

# Headset Designs
As part of a research project, several students and I have been working with the OpenBCI Ultracortex Mark IV EEG headset. The headset frame is entirely 3D printed, but can be problematic due to the shape. I've used Blender to cut the medium headset into parts, which should aid printing on smaller machines. The use of supports is necessary with any of these headset files, and the overall headset will need to be glued together. For more information on this project, our project code can be found in this [repository](https://github.com/mahn00b/openbci_testing_e).
