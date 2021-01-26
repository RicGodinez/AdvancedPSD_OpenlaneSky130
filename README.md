# AdvancedPSD_OpenlaneSky130
Documentation of the VSD Advanced Physiscal Design using OpenLANE/Sky130 5 day workshop
# Advanced Physical Design Using OpenLANE and Skywater130

![image](https://user-images.githubusercontent.com/16291730/105903884-138c2300-5fe6-11eb-8c2a-9fbd4fd2b79a.png)

## DAY 1

### Openlane flow

![image](https://user-images.githubusercontent.com/16291730/105903892-171faa00-5fe6-11eb-85f0-6bce8b05f97a.png)

### Running Open lane:

First, move to the work dir:
```
$ cd ~/Desktop/work/tools/open\_lane\_working\_dir/openLane\_flow
```
In this dir there is a file called flow.tcl, we need to source it with the -design switch to specify the name of the design to run and optionally if you want to run step by step place the switch -interactive:

$ ./flow.tcl -design \&lt;design\_name\&gt; -interactive

![image](https://user-images.githubusercontent.com/16291730/105903902-1a1a9a80-5fe6-11eb-896d-ca3e66659bee.png)

Then import the open lane commands package when Open Lane opens

% package require openlane 0.9

Then create the run directory of this specific run. Recommended to give the run a name so you can rework on this run under the same name using the switch -tag

% prep -design picorv32a -tag \&lt;the name you want to give the run\&gt;

After running the prep command, the tool configured the run using the setting that appear con the config.tcl found in the design folder you chose to run, in the example avobe, the picorv32a design config file.

Now we are ready to run synthesis with yosis on the loading the design with the command:

% run\_synthesis

A synthesized netlist, if everything ran ok, can be found in the project&#39;s run directory:

$ ls ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/synthesis

And a set of reports regarding the resulting netlist can be found at:

$ ls ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/reports /synthesis

For example there is a report that shows the utilization of the cells in the designs and other useful information:

$less ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/reports /synthesis/yosys\_2.stat.rpt

![image](https://user-images.githubusercontent.com/16291730/105903911-1d158b00-5fe6-11eb-854d-ddf8698c4714.png)

## Day 2

### Floor Planning

For floor planning confugaration you can refer to the variables needed for this step inside the open lane configuration folder&#39;s README file:

$ less ~/Desktop/work/tools/open\_lane\_working\_dir/openLane\_flow/configuration/README.md

This file contains a description of the basic variables need to set on the whole flow and how to set them.

The file in this directory called floorplan.tcl contains all these variables set with a default value.

These variables can be customized by the designer depending on the needs of the design, but this needs to be done on a separate tcl file which will override this default settings, this file is located here:

$ ls ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/config.tcl

Once configured, you can run the floorplan process on an openlane terminal that has the synthesis step already completed with the command:

% run\_floorplan

Once it completes, you can see the resulting def (Design Exchange Format) file in:

$ ls ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/\&lt;design name\&gt;\_floorplan.def

To view the file on a GUI, you need to invoke the Magic tool, to do so you need the following command:

$ magic -T \&lt;PDK tech file\&gt; lef read \&lt;Merged lef\&gt; read \&lt;Floorplan def\&gt;

For the command above the location of the files is the following:

- PDK tech file:
  - ~/Desktop/work/tools/open\_lane\_working\_dir/pdks/sky130A/libs.tech/magic/sky130A.tech
- Merged lef:
  - ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/merged.lef
- Floorplan def:
  - ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/floorplan/\&lt;design name\&gt;\_floorplan.def

This will open the Magic GUI:

![image](https://user-images.githubusercontent.com/16291730/105903928-20a91200-5fe6-11eb-9dc0-c879da04cb06.png)

Since this is just the floor plan stage, only the pins were placed according to the floorplan configuration, all the cells inside the design are not placed still, they are just appearing all in the bottom left corner of the die.

## Placement

#### Global Placement

This placement is done roughly, just for trying to approximate the location of the cells to be effective for signal integrity, this is done by constraining the wire length. This will not align the cells properly or avoid overlapping them.

The command to run placement in the open lame command line is:

%run\_placement

The resulting file is a def file located in:

~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/placement/\&lt;design name\&gt;.placement.def

To see the placement in a GUI:

$ magic -T \&lt;PDK tech file\&gt; lef read \&lt;Merged lef\&gt; read \&lt;Floorplan def\&gt;

For the command above the location of the files is the following:

- PDK tech file:
  - ~/Desktop/work/tools/open\_lane\_working\_dir/pdks/sky130A/libs.tech/magic/sky130A.tech
- Merged lef:
  - ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/merged.lef
- Placement def:
  - ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/placement/\&lt;design name\&gt;.placement.def

![image](https://user-images.githubusercontent.com/16291730/105903939-243c9900-5fe6-11eb-84c4-95a68b54a430.png)

## Day 3

### Custom CMOS inverter cell design- Download

To understand the flow to create a standard cell and add it to be available on the std library, we will need to download the layout files of an already created custom inverter cell and analyze it to be able to go through the cell library characterization process.

The custom layout of the inverter we are going to use is stored on a github repository (https://github.com/nickson-jose/vsdstdcelldesign.git). To add it to our work area, we need to move to the directory we want the files to be added, we will use this directory as an example:

$ cd ~/Desktop/

Then to clone (or download) all the contents from the github directory to this location we will execute:

$ git clone [https://github.com/nickson-jose/vsdstdcelldesign.git](https://github.com/nickson-jose/vsdstdcelldesign.git) .

This will create a directory called vsdstdcelldesign, where all the files will be added.

### Custom CMOS inverter cell design, layout exploration

Once we have the inverter cell files downloaded, we can open them with the Magic tool to look at the layout design.

For this, we need the following command:

$ magic -T \&lt;PDK tech file\&gt; \&lt;cell&#39;s magic format file\&gt;

Where in this example these files are:

- PDK tech file:
  - ~/Desktop/work/tools/open\_lane\_working\_dir/pdks/sky130A/libs.tech/magic/sky130A.tech
- Cell&#39;s magic format file
  - ~/Desktop/vsdstdcelldesign/sky130\_inv.mag

This will open the magic&#39;s GUI to see the layout of this inverter:

![image](https://user-images.githubusercontent.com/16291730/105903950-2868b680-5fe6-11eb-92c0-ca9a81f2e952.png)

###

###

### Custom CMOS inverter cell design- Parasitic extraction

While the Magic tool is open with the inverter design loaded, we will want to extract the parasitic resistances and the parasitic capacitances in order to run some SPICE simulations on this cell.

To do this, we need to go to the Magic tool console, that was opened along the GUI of the tool, this console window is called &quot;tkcon\_2.3 Main&quot;.

In this window you are able to type commands to get more details on the selected objects on the layout GUI page and some other commands. To extract the parasitic data, you need to type the following on the tkcon console:

% extract all

This will create a &quot;sky\_130\_inv.ext&quot; file on the &quot;vsdstdcelldesign&quot; directory. Which contains the extraction for all the elements on this design.

Next, we will type on the console:

% ext2spice cthresh 0 rthresh 0

Which will prepare the tool to extract a spice compatible file with the RC parasitics. To save this spice file you just need to type:

% ext2spice

This will generate the file &quot;sky\_130\_inv.spice&quot; file on the &quot;vsdstdcelldesign&quot; directory. With this file we now have the template to run a spice simulation on this inverter.

### Custom CMOS inverter cell design- Prepare SPICE simulations

To launch a simulation, we first need to do some edits on the sky130\_inv.spice file we got from the magic tool, which looks like this:

![image](https://user-images.githubusercontent.com/16291730/105903960-2b63a700-5fe6-11eb-8906-c4718e9118ff.png)

The changes needed are:

1. On line 3, the scale should match the grid scale of the layout, which is 0.01u not 10000u
2. Point to the library of the nmos transistor
3. Point to the library of the pmos transistor
4. Correct the name of the models for the PMOS and NMOS on lines 6 and 8 (These names can be referenced in the lib files referenced on bullets 2 and 3)
5. Comment out the lines that are not needed, lines 5 and 15
6. A set of variables to complete the simulated circuit:
  1. VDD to specify the 3.3V power supply
  2. VSS to specify the ground, which is 0V
  3. Va to specify the simulated input pulse. Since we are doing transient sim, we ned to specify for this pulse, the starting and end voltage, the start sim time, the time it takes for rise, time it takes for fall, the &quot;ON&quot; pulse width, and the complete time period of the pulse.
7. A set of commands for the sim tool to run:
  1. .tran which will tell it to run a transient sim and for how long, the first parameter will tell the tool the length of the steps to simulate, and how long.
  2. And the run command

The resulting file looks like this:

![image](https://user-images.githubusercontent.com/16291730/105903967-2e5e9780-5fe6-11eb-91c4-0537ee1fb89c.png)

### Custom CMOS inverter cell design- Run SPICE simulations

To start the simulation, we need to run the ngspice tool providing the configuration file we fixed on the previous step. Be sure to be on the directory of this file to run this command:

% ngspice sky130\_inv.spice

Once this runs we can plot the results on different views, the first plot we will try is the voltage on the Y axis, the simulation time on the X axis and we will be sweeping the output voltage(y) and the input voltage (a).

This is done by typing this command on the ngspice console:

-\&gt; plot y vs time a

The resulting plot will look like this:

![image](https://user-images.githubusercontent.com/16291730/105903992-34547880-5fe6-11eb-94e9-7501baa33e82.png)

### Custom CMOS inverter cell design- Cell characterization

Using the plot from the previous exercise, we can characterize the cell by obtaining the rise transition time, fall transition time, rise propagation delay, fall propagation delay.

To zoom in on a specific region of the plot you need to use the right click. Click and hold the click and drag the mouse until the box shown wraps around the area you want to zoom. This will open a new plot window showing the area selected.

Then you can use the left click, to click on any region on the plot to get the exact coordinates of the area you clicked. These coordinates will show on the ngspice terminal.

By doing this exercise we can retrieve the characterization values of this cell, which are shown below:

|
 | MAX | 20% | 80% | 50% |
| --- | --- | --- | --- | --- |
| Voltage | 3.3 | 0.66 | 2.64 | 1.65 |

|
 | 20% output V time | 80% output V time | Delay |
| --- | --- | --- | --- |
| rise transition delay | 2.1819E-09 | 2.2469E-09 | 6.5E-11 |
| fall transition delay | 4.095E-09 | 4.0529E-09 | 4.21E-11 |
| --- | --- | --- | --- |
|
 |
 |
 |
 |
| --- | --- | --- | --- |
|
 | 50% V input time | 50% V output time | Delay |
| rise propagation delay | 2.1507E-09 | 2.21197E-09 | 6.127E-11 |
| fall propagation delay | 4.0498E-09 | 4.07746E-09 | 2.77E-11 |
| --- | --- | --- | --- |

## Day 4

### Extract LEF for Inverter custom cell layout

Make sure the cells pins aling with the metal grid they will connect to. We can reference the grid specification for all the PDK&#39;s layer on the following file:

~/Desktop/work/tools/open\_lane\_working\_dir/pdks/sky130A/libs.tech/openlane/sky130\_fd\_sc\_hd/tracks.info

The file looks like this:

![](RackMultipart20210126-4-zlf5u9_html_340b6cf1e42ab531.png)

In each layer, you will see a row for horizontal grid definition (X) and another for the vertical (Y). The first value is the origin value of the grid and the second one the spacing.

So, we can open the custom layout in magic and set a visual grid with those values to verify the connection ports in the layout intersect with both the horizontal and vertical grids of the layer they are connected to. In this case the ports are in the li1 layer.

We open the design in magic and we activate the grid by pressing &quot;g&quot; on the keyboard. Then we go to the tkcon window to see how to set a custom grid with the following command:

% help grid

Which will print the following instructions:

![image](https://user-images.githubusercontent.com/16291730/105904013-3a4a5980-5fe6-11eb-8cb6-079228363ffb.png)

So based on what the tracks.info file tells us for the li1 layer grid, we can set up the visual grid to match that grid with the command:

% grid 0.46um 0.34um 0.23um 0.17um

Then you can go back ti layout window and verify the A and Y contacts indeed intersect with the li1 layer grid:

![image](https://user-images.githubusercontent.com/16291730/105904031-3e767700-5fe6-11eb-9621-a0f57da875d2.png)

We can also verify that the width and the height of the full cell matches the requirements.

Next, we need to create labels for every contact/port on the layout so the tool can create the lef correctly.

You need to select the area of the port with the mouse and create a label on it by going to Edit -\&gt; Text on the upper menu.

Then you need to fill the form that just appeared with the details of the port:

1. On the text string you set the port name
2. You change the size of the font to something smaller (0.15)
3. Then untick the default box on &quot;Attach to layer&quot; and tick the sticky box
4. On the &quot;Attach to layer&quot; textbox type the name of the layer on which this port is connected to
5. Tick the enable box under &quot;Port&quot; and assign it a number
  1. You need to assign each port you define a different number incrementally starting with 0.

See example below:

![image](https://user-images.githubusercontent.com/16291730/105904038-41716780-5fe6-11eb-810c-de6c5952f2d8.png)

Then these labels we created as ports, we need to classify them so the tool knows how to treat it. You select the label by pressing &quot;s&quot; on the keyboard until just the label is selected, then go to the tkcon window and type &quot;what&quot; to make sure you selected the correct label and see it is attached to the correct label.

Then on the tkcon window you will specify if the port is an input, output or inout with the command &quot;port class \&lt;input output inout\&gt;&quot;.

Then you will specify the port usage with &quot;port use \&lt;signal power\&gt;&quot;.

You can see the example below:

![image](https://user-images.githubusercontent.com/16291730/105904046-459d8500-5fe6-11eb-9e6e-44c10571849d.png)

Then we save a copy of the cell with the command:

% save \&lt;the name you want to give\&gt;.mag

Lastly, to extract the lef type:

% lef write

This creates the lef file with the same name of your cell but with the extension .lef

If you read it you will see something like this:

![image](https://user-images.githubusercontent.com/16291730/105904057-49c9a280-5fe6-11eb-8e1c-e73cbcb8a1a1.png)

## Adding the custom cell to the design library

To add the newly generated lef, we need to copy the file to:

$ cp ~/Desktop/vsdstdcelldesign/\&lt;name of your cell\&gt;.lef ~/designs/\&lt;your design\&gt;/src

Also, the cloned git repo provided the characterization for this cell for all the corners (we already saw the basic way to characterize a cell in spice but this files have the complete characterization and it is placed in the correct format)

We need to copy this library characterization files in the same src folder

$ cp ~/Desktop/vsdstdcelldesign/libs/sky130\_fd\_sc\_hd\_\_\* ~/designs/\&lt;your design\&gt;/src

Then we need to edit the config.tcl in the design directory:

~/designs/\&lt;your design\&gt;/config.tcl

We need to add the lines 15 through 20 shown in the picture:

![image](https://user-images.githubusercontent.com/16291730/105904074-4df5c000-5fe6-11eb-93ff-839425418186.png)

This will let open lane know where to grab the characterization files and will add our lef to the list of lefs in the synthesys run.

Once the edit is complete, we need to run openlane as explained in the Day 1 chapter

$ ./flow.tcl -interactive

% package require openlane 0.9

%prep -design \&lt;your design name\&gt; -tag \&lt;tag you assigned to your runs\&gt; -overwrite

When running prep, it will delete the previous synthesis run since we gave it the -overwrite switch

Next wee need to point to the lef file on the fly in the console so it takes it in the synthesis run:

% set lefs [glob $::env(DESIGN\_DIR)/src/\*.lef]

% add\_lefs -src $lefs

Now, we just need to run synthesis and wait to see if the new library was used:

% run\_synthesis

You can check the list of standard cells used on the synthesis on the prompt:

![image](https://user-images.githubusercontent.com/16291730/105904080-50f0b080-5fe6-11eb-9b58-996b8002db30.png)

And we can see the custom cell was used 2201 times in the synthesis.

##

## Fixing the slack caused by the custom inverter

Unfortunately, after running synthesis, the tool reported a slack problem with the new cell as shown on the image below:

![image](https://user-images.githubusercontent.com/16291730/105904091-54843780-5fe6-11eb-849b-b50c7715185f.png)

To fix it, we will change the synthesis strategy used. By changing this, you will sacrifice area for speed by a selecting different set of standard cells that will be faster, but bigger in size.

If you open the README.md inside the configuration folder:

$ less ~/Desktop/work/tools/open\_lane\_working\_dir/openLane\_flow/configuration/README.md

You will notice there is an environmental variable used to change the strategy called &quot;SYNTH\_STRATEGY&quot;, which description is as follows:

![image](https://user-images.githubusercontent.com/16291730/105904103-577f2800-5fe6-11eb-8d26-f7b56987bb75.png)

As you can see the current SYNTH\_STRATEGY is set to 2, so we will change it to 2:

![image](https://user-images.githubusercontent.com/16291730/105904123-5d750900-5fe6-11eb-95b9-77ffb0273edb.png)

We will also modify the SYNTH\_SIZING variable:

![image](https://user-images.githubusercontent.com/16291730/105904136-61089000-5fe6-11eb-8f2f-ffa3c72756e0.png)

From 0 to 1

Next we will run synthesis again:

% run\_synthesis

![image](https://user-images.githubusercontent.com/16291730/105904144-64038080-5fe6-11eb-87eb-728e39dfcc43.png)

We can see the slack still being violated, but we made a huge improvement, which we can fix later with a more focused timing analysis.

## Running floorplan and placement with the custom inverter

Next, to set up the floorplan, use the command:

% run\_floorplan

Once it finishes, run placement with:

% run\_placement

The placement will create a def file you can see using magic:

$ magic -T \&lt;PDK tech file\&gt; lef read \&lt;Merged lef\&gt; read \&lt;Floorplan def\&gt;

For the command above the location of the files is the following:

- PDK tech file:
  - ~/Desktop/work/tools/open\_lane\_working\_dir/pdks/sky130A/libs.tech/magic/sky130A.tech
- Merged lef:
  - ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/merged.lef
- Placement def:
  - ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/placement/\&lt;design name\&gt;.placement.def

![image](https://user-images.githubusercontent.com/16291730/105904150-67970780-5fe6-11eb-9db4-f7328872110a.png)

## Running STA

To run STA, we need to create a short script on the openlane work directory:

~/Desktop/work/tools/open\_lane\_working\_dir/openLane\_flow/pre\_sta.conf

Which looks like this:

![image](https://user-images.githubusercontent.com/16291730/105904166-6c5bbb80-5fe6-11eb-89ff-c7eeeaad69fe.png)

To tell the sta tool where to find the models for setup time, hold time, the sdc file and the synthesized netlist.

To run sta, type the command on the terminal:

$ sta \&lt;the sta config file\&gt;

## Optimizing slew and load values by optimizing fanout

After doing our synthesis, we noticed a slack problem. After looking at the cells use, this slew values and load values are too high due to the fact they have a lot of fanout:

![image](https://user-images.githubusercontent.com/16291730/105904183-71206f80-5fe6-11eb-82a0-8783e7afd13b.png)

You can see the fanout of some cells is driving 5 to 6 cells. To get a detail report on this on the sta tool, you can execute the command:

% report\_net -connections \&lt;net to report\&gt;

![image](https://user-images.githubusercontent.com/16291730/105904192-741b6000-5fe6-11eb-9612-e6762c23df39.png)

We can change the strategy for fanout to improve it:

% set ::env(SYNTH\_MAX\_FANOUT) 4

After doing some fixes on the netlist using the sta tool, we can write out the fixed netlist in the sta command line with:

% write\_verilog \&lt;filepath/filename\&gt;

The idea is to replace the netlist on:

~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/synthesis

Once we replace it, we have to meake sure not to run synthesis again on openlane, so it takes this new gate level netlist.

So just run floorplan and placement on open lane to continue with the flow.

### CTS

To insert the clock tree in our netlist, just type:

% run\_cts

This will run the Triton CTS tool and route the clocks in the design.

## Timing Analysis with real clocks

To perform the sta with the cts integrated, open the tool openroad by typing:

% openroad

Now we need to create a data base for the run. First step is to read the merged lef:

% read\_lef ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/merged.lef

The next file we need to load is the post cts def file:

% read\_def ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/cts/\&lt;design name\&gt;.cts.def

After reading the input files now we can write the data base:

% write\_db \&lt;data base name you want\&gt;.db

After writing out the DB, now we need to read it:

% read\_db \&lt;data base name you want\&gt;.db

Next we load the post cts netlist Verilog file

% read\_verilog ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/synthesis/\&lt;design\&gt;.synthesis\_cts.v

Now read the liberty files:

% read\_liberty -max $::env(LIB\_SYNTH\_COMPLETE)

THIS functionality is not ready in open lane since the cts process is not multi corner enabled yet

% read\_liberty -max $::env(LIB\_MAX)

% read\_liberty -max $::env(LIB\_MIN)

The next file is the SDC file:

% read\_sdc

~/designs/\&lt;your design\&gt;/src/my\_base.sdc

Then set the tool to use the propagated clocks model:

% set\_propagated\_clock [all\_clocks]

Next you can run the report generation with:

% report\_checks -path\_delay min\_max -format full\_clock\_expanded -digits 4

You should see the timing report like this:

![image](https://user-images.githubusercontent.com/16291730/105904202-77165080-5fe6-11eb-9e59-a1844c2ecbb4.png)

You can also create a hold clock skew report with:

![image](https://user-images.githubusercontent.com/16291730/105904221-7b426e00-5fe6-11eb-8ef8-219fc50f178e.png)

That looks like this:

![image](https://user-images.githubusercontent.com/16291730/105904235-7ed5f500-5fe6-11eb-94f5-97acdb508d1d.png)

And a skew report for setup:

% report\_clock\_skew -setup

![image](https://user-images.githubusercontent.com/16291730/105904244-809fb880-5fe6-11eb-84ed-2b29bb2c92e8.png)

## Day 5

###

### Build power distribution network

On a standard flow, the power distribution network needs to be created at the floorplaning stage, but openlane has some issues doing it on that stage, so it is done as a pre routing step.

To to the pwer distribution network type:

% gen\_pdn

This step will insert the power rails VDD and GND for the std cells to be connected. This is done in metal1.

This will generate a def file containing the power rails on:

~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/floorplan/pnd.def

If you open the def file it will look like this in magic:

![image](https://user-images.githubusercontent.com/16291730/105904249-84333f80-5fe6-11eb-8835-80d16b9f9fb9.png)

## Routing

To start the routing process, just type:

% run\_routing

Once the routing finishes using FastRoute for route preprocessing and Triton route for detailed routing, we need to extract the routing parasitics to be able to run a final timing analysis on the design.

Since the Openlane tool is still missing this functionality, this needs to be done externally on the SPEF\_EXTRACTOR tool. You can fin this tool in the following directory:

$ cd ~/Desktop/work/tools/SPEF\_EXTRACTOR

Inside the directory you will find the python script that need to be run. To make it work, you need to provide it the merged lef file and the post-routing def file. This is the command

$ python3 main.py \&lt;path to the merged lef\&gt;/.merged.lef \&lt;path to the routing def\&gt;/\&lt;design name\&gt;.def

This are the usual paths where this files can be found:

- Path to the merged lef:
  - ~/designs/\&lt;your design\&gt;/\&lt;run name you placed on -tag\&gt;/tmp/merged.lef
- Path to the routing def:
  - ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/routing/\&lt;design name\&gt;.def

This will create a SPEF file in the same directory as the def:

~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/routing/picorv32a.spef

Now that you have a spef file, you can run openroute to do sta analysis. Just follow the complete steps to run the tool.

The only 2 changes you need to give it before running the report is the spef to take the parasitics of the routing into consideration:

% read\_spef ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/routing/picorv32a.spef

And the Verilog netlist you need to provide is different:

% read\_verilog ~/designs/picorv32a/runs/\&lt;the tag you used\&gt;/results/synthesis/\&lt;design\&gt;.synthesis\_preroute.v

After running the report, you can see if you meet the slack for hold:

![image](https://user-images.githubusercontent.com/16291730/105904262-885f5d00-5fe6-11eb-8275-751bfaabe6a8.png)

Slack for setup:

![image](https://user-images.githubusercontent.com/16291730/105904269-8bf2e400-5fe6-11eb-94a4-71b2287c7fff.png)

Clock skew for hold

![image](https://user-images.githubusercontent.com/16291730/105904290-91502e80-5fe6-11eb-8875-a6c3b01461a1.png)

And clock skew for setup

![image](https://user-images.githubusercontent.com/16291730/105904299-944b1f00-5fe6-11eb-8959-97355fa9d986.png)
