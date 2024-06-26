## Tune your TMC :)

### Installation
Install
```
cd ~ && git clone https://github.com/anton-vinogradov/tmc-chopper-tune && bash ./tmc-chopper-tune/install.sh
```
or reinstall with 
```
cd ~ && bash ./tmc-chopper-tune/uninstall.sh || true && git clone https://github.com/anton-vinogradov/tmc-chopper-tune && bash ./tmc-chopper-tune/install.sh
```
the calibration script. (the klipper will reboot!)
### Hardware configuration
[Connect the accelerometer](https://www.klipper3d.org/Measuring_Resonances.html) to the printing head by screwing it in.

### Measurement
Determine the current vibrations by calling `TMC_CHOPPER_CHECK`.

#
#
#
#
#
#
#

    2. After the macro is completed, the algorithm will automatically generate a table of data and graphics, place them in the `.../adxl_results/chopper_magnitude/` directory, download and open `interactive_plot_*.html`, and see the following picture -
       ![](/pictures/img_1.png)
       The graph will usually show 2 peaks, at a speed of about 50mm/s and 100mm/s - these are resonant speeds, we need the lowest of these speeds, for example, 55mm/s.
    3. Run the macro to iterate through all the chopper options at the previously selected speed, the command will look like this
       -`CHOPPER_TUNE MIN_SPEED=55 MAX_SPEED=55`. Check the availability of free space on the host, possible tmp folder limit on hosts with 1GB of RAM, about ~700mb is required for data.
       The data collection time will take approximately two hours (depending on kinematics), after completion we open the graph in the same way as the previous time, we get a graph of the form -
       ![](/pictures/img_2.png)
       In this example, the minimum vibrations are at TBL=0 and TOFF=8. Let's enlarge this area.
       ![](/pictures/img_3.png)
    4. Select the chopper option with the minimum magnitude value - these are the required parameters. It is also necessary to take into account that with large values of `TBL` and `TOFF` the motor frequency decreases, which leads to the appearance of nasty high frequency noise.
       If the vibration decreases with the occurrence of this phenomenon, move to a pleasant range of work between vibrations and noise, by using the program functionality (entering the registers ranges you need into the macro parameters), if this bothers you. If not, then it would be preferable to leave the high-frequency squeak.
       We enter them into the drivers section in printer.cfg, example -
   ```
   [tmc**** stepper_*]
   cs_pin: PC4
   ...
   driver_TBL: 0
   driver_TOFF: 8
   driver_HSTRT: 5
   driver_HEND: 5
   ```

    5. You can repeat the procedure with smaller variations of the chopper, for example, only `TBL=0` and `TOFF=8` and iterate over the full ranges of `HSTRT` and `HEND`, but with more repetitions of `ITERATIONS`. In this case, the graph will be based on average results to reduce the influence of mechanics on the readings.
    6. If you are the lucky owner of a TMC2240 or TMC5160, then after setting all of the above registers, you have the opportunity to configure another parameter called `TPFD`.
       It is responsible for damping the average resonances of the motor, and has a value range of `0-15`. Set its parameter value to `driver_TPFD: 0`, or calibrate it.
       The command with the data registers found above, two `ITERATIONS` - for greater accuracy, and resonant speed looks like this - `CHOPPER_TUNE TBL_MIN=0 TBL_MAX=0 TOFF_MIN=8 TOFF_MAX=8 HSTRT_MIN=5 HSTRT_MAX=5 HEND_MIN=5 HEND_MAX=5 TPFD_MIN=0 TPFD_MAX=15 MIN_SPEED=55 MAX_SPEED=55 ITERATIONS=2`


Description of the program functionality -

The values `'default'` in parameters mean that if there is no argument, this variable will assign the default parameters from printer.cfg, or calculate the minimum required ones.

1. `AXIS` - direction `(X/Y)` in which the measurement will be run.
2. `CURRENT_MIN_MA` and `CURRENT_MAX_MA` - are responsible for changes in the supplied current `(mA)` to stepper motors in 10mA steps. For example, if you have enough torque that the stepper motors produce, you can reduce their current to make the system quieter and reduce motor heating. This function partly allows you to analyze is it worth it, or just choose the current you need in measure.
3. `TBL_MIN-0` and `TBL_MAX-3`, `TOFF_MIN-1` and `TOFF_MAX-8`, `HSTRT_MIN-0` and `HSTRT_MAX-7`, `HEND_MIN-0` and `HEND_MAX-15`, `TPFD_MIN-0` and `TPFD_MAX-15` are actually also responsible for enumerating parameters, in this case, registers of driver pairs. Their range of work and search is indicated.
4. `HSTRT_HEND_MAX-16` - limit on the sum of `HSTRT and HEND`, change is undesirable. ([more](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC5160A_datasheet_rev1.17.pdf))
5. `MIN_SPEED` and `MAX_SPEED` - enumerate the speed range, with a step of `1mm/s`.
6. `ITERATIONS` - the number of repetitions of measurements, for more accurate data.
7. `TRAVEL_DISTANCE` - distance `(mm)` of the print head movement during which vibrations are read. By default, is calculated based on the printer's capabilities and measurement time.
8. `ACCELEROMETER` - an accelerometer that will be used to measure vibrations, auto will be detected if one is specified in the `resonance_tester` configuration, otherwise, without specifying will be applied `adxl345`.
9. `FIND_VIBRATIONS` - mode for measuring vibrations from speed, useful in order to remove resonant speeds from everyday printing, as for step 3.1 of this article, applies registers from printer configuration. Values - `(True / False), (1 / 0)`
10. `RUN_PLOTTER` - run the graph generation script. Values - `(True / False), (1 / 0)`

## Datasheets
- TMC 2130 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2130_datasheet_rev1.15.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc2130)
- TMC 2208 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2202_TMC2208_TMC2224_datasheet_rev1.14.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc2208)
- TMC 2209 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2209_datasheet_rev1.09.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc2209)
- TMC 2660 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC2660C_Datasheet_Rev1.01.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc2660)
- TMC 2240 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc2240_datasheet.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc2240)
- TMC 5160 [datasheet](https://www.analog.com/media/en/technical-documentation/data-sheets/TMC5160A_datasheet_rev1.17.pdf) Klipper [configuration](https://www.klipper3d.org/Config_Reference.html#tmc5160)