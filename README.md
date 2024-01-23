
# DLP3DController
This project includes the code and schematics used for the assembly of a Sigma type 3D bioprinter with 4 heads in dual X carriage mode, with valve actuation and temperature control (heating-cooling) with peltier cells. Marlin 2.0.9.7 firmware was used on a RUMBA board.

- The **E0, E1** and **E2** extruders are **low temperature** for printing hydrogels and are heated or cooled with **peltier cells**.
- The **E3** extruder is **high temperature** for printing with thermoplastics such as PCL and only allows heating with a **thermal cartridge**. 

The **T0** head includes the E3 and E0 extruders. 
The **T1** head includes the E1 and E2 extruders. 

A pushbutton panel with switches has been added to prevent the peltier from operating continuously when not in use. 

## RUMBA pins

The default pin layout of the board has been modified and the EXP3 pins have been used.

![Rumba pins](https://github.com/anuargimenez/marlinSigmaBP/blob/master/captures/rumba_schem.PNG)

## Peltier Control

Two **L298N modules** have been used to control 4 peltier cells simultaneously (3 for the extruders E0, E1 and E2 and 1 for the printing bed) and invert their polarity. A **7400 integrated circuit** is also added to invert the input signal to the module.

![Rumba pins](https://github.com/anuargimenez/marlinSigmaBP/blob/master/captures/Peltier%20control.png)

The temperature loop has been modified as follows (lines 1500-1535 in temperature.cpp): 
   
      const bool is_idling = TERN0(HEATER_IDLE_HANDLER, heater_idle[ee].timed_out);
      float pid_output;

      if (ee == 3) {
          // Comportamiento para el extrusor 3 (alta temperatura, solo resistencia térmica - linea original)
          pid_output = (!is_idling && temp_hotend[ee].is_below_target()) ? BANG_MAX : 0;
      } else {
          // Comportamiento para los extrusores 0, 1, 2 con peltier
          if (!temp_hotend[ee].is_below_target()) {
              // Si está por encima del objetivo, activar Peltier
              pid_output = 1;
              switch(ee) {
                  case 0: analogWrite(21, 0); break;
                  case 1: analogWrite(20, 0); break;
                  case 2: analogWrite(63, 0); break;
              }
          } else {
              // Si está por debajo del objetivo, desactivar Peltier
              pid_output = 0;
              switch(ee) {
                  case 0: analogWrite(21, 255); break;
                  case 1: analogWrite(20, 255); break;
                  case 2: analogWrite(63, 255); break;
              }
          }
      }

    #endif

    return pid_output;
      

## Valve Control

To control the valves, it was necessary to add **two power expander modules** controlled by the exp3 output pins. The other two are connected directly to the FAN0 and FAN1 outputs of the RUMBA. 

![Rumba pins](https://github.com/anuargimenez/marlinSigmaBP/blob/master/captures/Valve%20control.PNG)

## Useful Gcode
Here are some useful Gcode lines for using the printer:

#### - Dual X Carriage:
- **T0**: Select Tool 0 (E0 and E3)
- **T1**: Select Tool 1 (E1 and E2)

#### - Fans:
- **M106 P0 S255**: Turn on Fan0 (E0)
- **M106 P0 S0**: Turn off Fan0 (E0)
- **M106 P1 S255**: Turn on Fan1 (E1)
- **M106 P1 S0**: Turn off Fan1 (E1)
- **M106 P2 S255**: Turn on Fan2 (E2)
- **M106 P2 S0**: Turn off Fan2 (E2)

#### - Valves:
- **M42 P7 S255**: Turn on Valve 0 (E0)
- **M42 P7 S0**: Turn off Valve 0 (E0)
- **M42 P4 S255**: Turn on Valve 1 (E1)
- **M42 P4 S0**: Turn off Valve 1 (E1)
- **M42 P5 S255**: Turn on Valve 2 (E2)
- **M42 P5 S0**: Turn off Valve 2 (E2)
- **M42 P8 S255**: Turn on Valve 3 (E3)
- **M42 P8 S0**: Turn off Valve 3 (E3)

#### - Hotends:
- **M104 T0 S[temp]**: Set E0 hotend temperature to [temp].
- **M104 T1 S[temp]**: Set E1 hotend temperature to [temp].
- **M104 T2 S[temp]**: Set E2 hotend temperature to [temp].
- **M104 T3 S[temp]**: Set E3 hotend temperature to [temp].


## Authors
_Biomedical & Tissue Engineering Laboratory | BTELab_

-   [Anuar R. Giménez El Amrani](https://www.github.com/anuargimenez)
- [Miguel Fernández Kolb](https://www.github.com/miferkolb)
- Enrique Sodupe Ortega

-   [Andrés Sanz García](https://www.github.com/mugiro)
