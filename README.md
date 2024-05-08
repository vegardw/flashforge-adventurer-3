# Miscellaneous information about the Flashforge Adventurer 3 3D printer.

## Prusa Slicer fork
My repo for trying to create a Prusa Slicer fork with built in support for the Flashforge Gcode flavor:

* [vegardw/PrusaSlicer_ffa3/](https://github.com/vegardw/PrusaSlicer_ffa3/)

## Gcode

### General
The Flashforge Adventurer 3 uses a custom Flashforge flavor of Gcode. There is some information about it regarding getting slicers like Prusa Slicer and Cura working with the printer. The Cura slicer comes with a community profile for the Adventurer 3 based on the documentation from flashforge

* [Prusa3D forum thread discussing getting Prusa Slicer to work with the FF A3, including a config bundle with custom profiles](https://forum.prusa3d.com/forum/prusaslicer/has-anybody-successfully-used-prusa-slicer-with-the-flashforge-adventurer-3/)
  * [config bundle direct link](https://forum.prusa3d.com/wp-content/uploads/2023/03/PrusaSlicer_config_bundle.txt)
* [Reddit thread discussing the M118 command in the Gcode generated by FlashPrint](https://www.reddit.com/r/FlashForge/comments/v81y3g/m118_line_in_the_g_files/)
* [Reddit thread discussing working modifications to Gcode from Prusa Slicer](https://www.reddit.com/r/FlashForge/comments/fxo9z3/prusaslicer_adv_3_settings/)
* [Official Flashforge document describing how to set up Cura for the Adventurer 3](https://en.fss.flashforge.com/10000/software/59578ec5bff6b5671194cde932f0e99f.pdf)

## Network protocol
The printer have a console listening at port 8899 accepting Gcode commands. The FlashPrint software uses this port to transfer gcode and start print, and to receive status during the print.
