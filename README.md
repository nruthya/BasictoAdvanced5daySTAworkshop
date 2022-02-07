

# STA inputs
## 1.std cell liberty format libs
## 2.Netlist
## 3.SDC
## 4.SPEF
# Lab1:
## case a: when lib is missing 
 ![image](https://user-images.githubusercontent.com/56647490/152730805-92c1ed24-548d-4044-bfc4-0cdc95b2021f.png) 
open timer .log shows only taskflow
 
![image](https://user-images.githubusercontent.com/56647490/152730831-8b7c94d6-7dcf-465c-a2e8-c13bdc9af87f.png) 
## case b: when verilog is missing 
open timer .log shows "no critical path found "
![image](https://user-images.githubusercontent.com/56647490/152730860-ee8d1e30-50e4-4c3a-a1db-64ecc5650a85.png) 
 ![image](https://user-images.githubusercontent.com/56647490/152730894-5b2bf0d7-c1a6-479e-909f-1329aa5d7ebc.png)
## case c: when sdc is missing 
open timer .log shows "no critical path found "
![image](https://user-images.githubusercontent.com/56647490/152732067-66cd9d66-c882-4072-a5aa-f0f9ff011f7c.png)
 
## case d:without CPPR
![image](https://user-images.githubusercontent.com/56647490/152732173-11fd7df2-9ef6-4bcc-bd4b-458f1125201d.png)

## case e:with CPPR
 
## case f:dump_taskflow enable and disable 
 
## case g: dump_graph disabled  and num_paths selections 
 
## 1.Timing path
 
## 2.Timing path 
 

## 3.Timings path 
 

so 3 timing paths were  reported without the  timing graph

#  Lab2:
## 1.difference between Early lib and late lib 
Early -used for min delay
Late-used for max delay.
SDC input delay value changes 10.000 reflected in the report.


#  Day3 Labs:
1. Consider the following picture
• How many paths do you see F1:CK →F2:D?
1. F1:CK→U3→U4→U6:A2→U7:A1→F2:D
2. F1:CK→U6→U4→U5:A1→U7:A2→F2:D
3. F1:CK→U6:A1→U7:A1→F2:D
4. F1:CK→U6→U5:A2→U7:A2→F2:D
• Type ‘leafpad out.txt’ the slack reported for the path is -212.323
• Which of the 4 paths above it corresponds to
 
report_timing -num_paths 100
Interesting same path both max and min violations 
Path --input port to D pin :

 

 
 

from the report it looks like  G0 is input port.Timing from input port to D pin.Would like to understand how it is facing setup and hold in the same path. 

1.there is no delay between G0 and D pin .Data arrival =0.
Early.lib findings for DFFR_X2 Pin CK 
 

Late.lib findings for DFFR_X2
 

1ps  is the clock period.(Please correct me if i am wrong i was unable to find out it is nano or pico.
1.Setup violation: Data is arriving fast without any delay so it is well before the data required time.So there shouldnt have been any setup violation.
But, Tsetup from late lib is 30.212 ps and clock period = 1ps 
Port + pin arrival time < = Tperiod -Tsetup
0< 1ps -30.212ps
0<-29.212ps which is not true .
Slack = Data required time - Data arrival time 
= -29.212 ps
2. Hold violation: Data in the arrival path should be stable for > Thold .
Data arrival time = 0ps
Thold time = 2.183
0> 2.183ps which is again not true.
Slack = Data arrival time -Data required time 
         0-2.183
= -2.183 
Day 4:
Crosstalk 
1.	When there are signal wires in close proximity to each other, if the signal is changing in one wire then it may impact the signal in the wire close by.
2.	Signal in the aggressor net help the signal to make transition faster(capacitor and input transition enforces signal in victim net) if the signals in aggressor net and victim net is in same direction.It will be helpful for setup time.
3.	Signal in the aggressor net help the signal to make transition slower if the signals in aggressor net and victim net is not  in same direction.It will be helpful for  hold  time.

OCV :
Inter die Variation related to process --> causing different delays --> gates transition time may be different.

 
Intra die variation: inside a single die there may be different delays due to the timing paths manufactured having process variations.
STA need to take into account the process variations,operating modes .(logical lib should be appropriately seleted according to max/min delay.)


 
Clock gating checks:
Clock gating is used when clock path needs to be controlled by signal.
 


 
Note:
1.AND gate to D flip flop pin is not considered clock downstream

Conditions for clock gating check
1. Clock gating cell should be connecting to clock pin of FF/latch,output port , generated clock.
2.why clock gating cell is required : so that it does not create unnecessary edge of the clock in fanout when there is transition at gating cell pin.
 

 
Setup period is checked just before the the clock makes high to low transition.
Hold period is checkded just after the clock makes low to high transition.
Setup: EN should be stable for Tsetup time before the clock makes high to low transition.
Hold: EN should be stable for Thold time after the clock makes transition from low to high.


 
Enable signal should change when clock is low because it will not impact the clock output
Enable is high it is sending the clock to the output.
Setup: EN should be stable for Tsetup time before the clock makes low to high transition.
Hold: EN should be stable for Thold time after the clock makes transition from high to low.


Truth Table for AND                                                        
 
Truth Table for OR
 
Lab4:
 
 
 

 
 
Clock groups :
Synchronous clocks have deterministic phase relationship
Asnynchronous clocks have no fixed phase relationship.For Async clocks CDC tool are used .STA cannot handle async clocks.


 
Logically exclusive clocks where clocks are chosen by selection.(MUX select). And are prone to crosstalk.
Physically exclusive clocks are those which cannot co-exist at the same time in a design.And not prone to crosstalk.Same net cannot have both clocks .

set_clock_groups asynchronous-group {clk1 clk2 clk3 } - group {clk4 clk5 clk6}
there is no relation implied within the group.
Clock in group1 is aync to clocks in group2
clk1 is async to clk 4 clk5 clk6 similarly clk2 clk3.
No relation cannot be assumed among clk1 clk2 clk3
 


 
When specify one group there is exception --> clk1 clk2 clk3 are async to all other clocks in the design except between the clk1,clk2 ,clk3.
clock properties:
 

uncertainity:skew of the same clock and inter-clock skew .
latency :source latency and netwrok latency
clock sense -> stopping at one point and  whether it is a + ve unate or -ve unate.
suppose XOR gate is used as inputs for clocks so we can tell the tool whether the clock output of XOR gate is + unate or - unate as it may not be able to sense.
set_ideal_network --> CTS 

Timing exception :
Path specification:
-from
-to
-through
 

F1/CK -->U1/A --> U1/Z --> F5/D
set_path  -from F1/CK -through U1/A  -through U1/Z -to F5/D
Through options should be according to the order.

Set_false_path: Dont time these paths.
set_false_path - from F1/CK -through U1/A  -through U1/Z -to F5/D to be more particular about the path otherwise it will take all the paths down the F1/CK.
set_false_path -from {F1/CK F2/CK }-to {F4/D F5/D}
tool will not time the paths F1/CK to F4/D (existing paths) and F2/CK to F5/D }
set_multicycle_path -setup 2 -from FF1 -to FF2

set_multicycle_path -hold 1  -from FF1 -to FF2

Muticycle path is used to alter the default setup relationship.In case of default 1 cycle it will be 2 cycle  for meet in timing.
 

multicycle setup relationship also modifies hold.To alter it back to default hold we can use this command.
setup-1 --> hold 
set_multicycle_path -hold 1  -from FF1 -to FF2

set_max_delay ,set_min _delay to include in timing path to do setup and hold check 
set_disable_timing and set_false_path difference:
set_disable_timing is essentially used for disabling the timing arc of the cell.
Set_false_path is more robust where we can choose the  paths that need not be timed.CK@ and CK4 can be set as false path depenging on the requirements where set_disable_timing dont have these options.

 

Mutliple modes
set_case_analysis -specify whether design need to be in test mode or functional mode. 

Lab5:
 
 
CPPR:
 
CPPR 69.1 -62.5 = 6.6 
ECO
 

 
Adding one more buffer U16 in the clock path increased the postitve skew which improved the setup time slack.
Before Timing ECOs slack = 353.185
CPPR:6.598
After timing ECOs
 

 

Clock Gating Checks:
 
 


