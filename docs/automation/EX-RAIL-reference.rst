**************************
EX-RAIL Command Reference
**************************

This is a detailed reference. For a summary version, please see :doc:`EX-RAIL Command Summary <EX-RAIL-summary>`.

`CommandStation-EX <https://github.com/DCC-EX/CommandStation-EX>`_ Provides full automation and accessory control through the Extended Railroad Automation Instruction Language (EX-RAIL). First, make sure you have the latest release of the `CommandStation-EX Firmware <https://github.com/DCC-EX/CommandStation-EX>`_.

See Also:

- :doc:`Introduction to EX-RAIL <EX-RAIL-intro>` 
- :doc:`EX-RAIL Command Summary <EX-RAIL-summary>`

Notes
======


- *AUTOMATION*, *ROUTE*, and *SEQUENCE* use the same ID number space, so a ``FOLLOW(n)`` command can be used for any of them.
- Sensors and outputs used by AT/AFTER/SET/RESET/LATCH/UNLATCH/SERVO/IF/IFNOT refer directly to Arduino pins, and those handled by I2C expansion (as virtual pins or vpins).
- Signals also refer directly to pins, and the signal ID (for RED/AMBER/GREEN) is always the same as the RED signal pin.
- It's OK to use sensor IDs that have no physical item in the layout. These can only be LATCHed, tested (IF/IFNOT), or UNLATCHed in the scripts. If a sensor is latched by the script, it can only be unlatched by the script… so ``AT(35) LATCH(35)`` for example, effectively latches sensor 35 on when detected once.
- All IDs used in commands and functions will be numbers, or an ALIAS name if configured.
- Most IDs simply need to be unique, however RESERVE/FREE and LATCH/UNLATCH must be in the range 0 - 255.


DIAGNOSTICS AND CONTROL
========================

There are some diagnostic and control commands added to the <tag> language normally used to control the Command Station over USB, WiFi or Ethernet.

EXRAIL
_______

``<D EXRAIL ON|OFF>`` Turns diagnostic traces for EX-RAIL events

  .. code-block::

    When the CS is connected to a serial monitor, EX-RAIL script logging can be turned on or off (Enabled or Disabled)

    Example output:

**NEED EXAMPLE OUTPUT HERE**
  
PAUSE/RESUME
_____________

``</PAUSE>`` Pauses **ALL** EX-RAIL automation activities, including sending an E-STOP to all locos.

``</RESUME>`` Resumes **ALL** EX-RAIL automation activities, and resumes all locos at the same speed at which they were paused.

Task info
__________

``</>`` Displays EX-RAIL running task information

   Example output:

**NEED EXAMPLE OUTPUT HERE**

ROUTES
_______

``</ ROUTES>``	Returns the Routes & Automations control list in WiThrottle format. JMRI integration only! **Really?**

  Example output:

**NEED EXAMPLE OUTPUT HERE**

START/KILL
___________

``</ START [loco_addr] route_id>``	Starts a new task to send a loco onto a Route, or activate a non-loco Animation or Sequence

``</ KILL task_id>``	Kills a currently running script task by ID (use to list task IDs)

RESERVE/FREE
_____________

``</ RESERVE block_id>``	Manually reserves a virtual track Block, valid IDs are in the range 0 - 255.

``</ FREE block_id>``	Manually frees a virtual track Block, valid IDs are in the range 0 - 255.

LATCH/UNLATCH
______________

``</ LATCH sensor_id>``	Lock sensor ON, preventing external influence, valid IDs are in the range 0 - 255.

``</ UNLATCH sensor_id>``	Unlock sensor, returning to current external state, valid IDs are in the range 0 - 255.


ROUTES, AUTOMATIONS, & SEQUENCES
=================================

EX-RAIL provides many commands to allow you to create routes that locomotives to follow that may involve turnouts, signals, etc. that can be automatically set to react when the loco trips a sensor.

Script Definition Terms
________________________

``AUTOMATION( id, "description" )``	Define an automation sequence that is advertised to WiThrottles to send a train along. See :ref:`automation/ex-rail-intro:example 4: automating a train (simple loop)` for a simple example.

``ROUTE( id, "description" )``	Define a route that is advertised to WiThrottles. This can be used to initiate automation sequences such as setting turnouts and signals to allow a train to be driven through a specific route on the layout. See :ref:`automation/ex-rail-intro:example 1: creating routes for a throttle` for various examples.

``SEQUENCE( id )``	A general purpose automation sequence that is not advertised to WiThrottles. This may be triggered automatically on startup, or be called by other sequences or activites. See :ref:`automation/ex-rail-intro:example 3: automating various non-track items`, :ref:`automation/ex-rail-intro:example 6: single line shuttle`, and :ref:`automation/ex-rail-intro:example 7: running multiple inter-connected trains` for further examples.

``ENDTASK`` or ``DONE``	Completes a Sequence/Route/Animation/Event handler, and any other automation definition as shown in the previous examples.

Object Definitions
___________________

Aliases
^^^^^^^^

``ALIAS( name[, value] )``	Assign names to values. Can go anywhere in the script. If a value is not assigned, a unique ID will be assigned based on the alias text.

This is a simple substitution that lets you have readable names for things in your script. For example, instead of having to remember the VPin a turnout is connected to, give the pin number an alias and refer to it by that name. You can use this to name routes, values, pin numbers, or anything you need.

If you simply need a unique identifier for an object such as a turnout, route, automation, or sequence, you don't even need to provide an ID, and EX-RAIL will generate one automatically.

However, IDs for RESERVE/FREE, LATCH/UNLATCH, and pins must be explicitly defined.

Alias naming rules:

- **Should be** reasonably short but descriptive.
- **Must start** with letters A-Z or underscore _ .
- **May then** also contain numbers.
- **Must not** contain spaces or special characters.

Examples:

Defining a pin turnout without an alias:

.. code-block:: cpp

  PIN_TURNOUT(1, 25, "Coal Yard")

Defining a pin turnout with aliases:

.. code-block:: cpp
  
  ALIAS(COAL_YARD)
  ALIAS(COAL_YARD_PIN, 25)
  PIN_TURNOUT(COAL_YARD, COAL_YARD_PIN, "Coal Yard")

In this simple example, aliases seem like overkill, however consider the case where you need to have the "Coal Yard" turnout closed or thrown in various different automation sequences, and you will soon see why it's easier to understand you're throwing the COAL_YARD turnout rather than turnout ID 12345.

Signals
^^^^^^^^

``SIGNAL( red_pin, amber_pin, green_pin )``	Define a pin based signal, which requires three active low pins to be defined to correspond with red, amber, and green lights.

``SIGNALH( red_pin, amber_pin, green_pin )`` As above to define a pin based signal, but with active high pins instead.

For both the SIGNAL/SIGNALH commands, signal colour is set using the pin defined for the red pin. If the signal only has two colours (eg. RED/GREEN), set the unused colour's pin to 0.

``SERVO_SIGNAL( vpin, red_pos, amber_pos, green_pos )`` Define a servo based signal, such as semaphore signals. Each position is an angle to turn the servo to, similar to the SERVO/SERVO2 commands, and SERVO_TURNOUT.

Signal examples:

.. code-block:: cpp

  SIGNAL(25, 26, 27)                // Active low red/amber/green signal using pins 25/26/27 directly on the CommandStation.
  SIGNALH(164 ,0, 165)              // Active high red/green signal using the first two pins of an MCP23017 I/O expander module.
  SERVO_SIGNAL(101, 100, 250, 400)  // Servo based signal using the first PCA9685 servo module.

  GREEN(25)                         // Sets our active low signal to green.
  GREEN(164)                         // Sets our active high signal to green.
  GREEN(101)                        // Sets our servo based signal to green.

Turnouts
^^^^^^^^^

All the below turnout definitions will define turnouts that are advertised to WiThrottle apps, Engine Driver, and JMRI, unless the HIDDEN keyword is used.

"description" is an optional parameter, and must be enclosed in quotes "". If you don't wish this turnout to be advertised to throttles, then substitute the word HIDDEN (with no "") instead of the description.

``TURNOUT( id, addr, sub_addr [, "description"] )``	Define a DCC accessory turnout. Note that DCC linear addresses are not supported, and must be converted to address/subaddress in order to be defined. Refer to the :ref:`reference/downloads/documents:stationary decoder address table (xlsx spreadsheet)` for help on these conversions.

``PIN_TURNOUT( id, pin [, "description"] )``	Define a pin operated turnout. When sending a CLOSE command, the pin will be HIGH, and a THROW command will set the pin LOW.

``SERVO_TURNOUT( id, pin, active_angle, inactive_angle, profile [, "description"] )``	Define a servo turnout. "active_angle" is for THROW, "inactive_angle" is for CLOSE, and profile is one of Instant, Fast, Medium, Slow or Bounce (although clearly we don't recommend Bounce for turnouts!). Refer to :doc:`/reference/hardware/servo-module` for more information.

``VIRTUAL_TURNOUT( id [, "description"] )`` Define a virtual turnout, which is backed by another automation sequence. For a good example of this refer to :ref:`automation/ex-rail-intro:realistic turnout sequeunces`.

Examples:

.. code-block:: cpp

  TURNOUT(100, 26, 0, "Coal Yard")                  // DCC accessory turnout at linear address 101.
  PIN_TURNOUT(101, 164, "Switching Yard")           // Pin turnout on an MCP23017 I/O expander module.
  SERVO_TURNOUT(102, 102, 400, 100, Slow, HIDDEN)   // A servo turnout on a PCA9685 servo module that is hidden from throttles.
  VIRTUAL_TURNOUT(103, "Lumber Yard")               // A virtual turnout which will trigger an automation sequence when CLOSE or THROW is sent.

Flow Control Functions
_______________________

``CALL( route )``	Branch to a separate sequence expecting a RETURN

``RETURN``	Return to caller

A very simple example using CALL with RETURN:

.. code-block:: cpp

  #define SIMPLE_EXAMPLE_NEEDED
  

``FOLLOW( route )``	Branch or Follow a numbered sequence (think of "GOTO")

``DELAY( delay )``	Delay a number of milliseconds

``DELAYMINS( delay )``	Delay a number of minutes

Delay examples:

.. code-block:: cpp

  ONCLOSE(102)      // When turnout 102 closed, wait 2 seconds, then set signal 101 green.
    DELAY(2000)
    GREEN(101)
    DONE

  AT(123)           // When sensor 123 is activated, set signal 102 red, wait 1 minute, then set signal 102 green.
    RED(102)
    DELAYMINS(1)
    GREEN(102)
    DONE

``DELAYRANDOM( min_delay, max_delay )``	Delay a random time between min and max milliseconds, see :ref:`automation/ex-rail-intro:example 7: running multiple inter-connected trains` for good examples.

``IF( sensor_id )``	If sensor activated or latched, continue. Otherwise skip to ELSE or matching ENDIF

``IFNOT( sensor_id )``	If sensor NOT activated and NOT latched, continue. Otherwise skip to ELSE or matching ENDIF

``IFGTE( sensor_id, value )``	Test if analog pin reading is greater than or equal to value (>=)

``IFLT( sensor_id, value )``	Test if analog pin reading is less than value (<)

``IFRANDOM( percent )``	Runs commands in IF block a random percentage of the time

``IFCLOSED( turnout_id )``	Check if turnout is closed

``IFTHROWN( turnout_id )``	Test if turnout is thrown

``IFRESERVE( block )``	If block is NOT reserved, reserves it and run commands in IF block. Otherwise, skip to matching ENDIF

``IFTIMEOUT``	Tests if "timed out" flag has been set by an ATTIMEOUT sensor reading attempt

``IFRED( signal_id )`` Test if signal is red

``IFAMBER( signal_id )`` Test if signal is amber

``IFGREEN( signal_id )`` Test if signal is green

``ELSE``	Provides alternative logic to any IF related command returning False

``ENDIF``	Required to end an IF/IFNOT/etc (Used in all IF.. functions)

Command Station Functions
__________________________

``POWERON`` Power on track and UNJOIN

``POWEROFF``	Power off track

``JOIN``	Joins PROG and MAIN track outputs to send the same MAIN DCC signal

``UNJOIN``	Disconnect prog track from main

``READ_LOCO``	Read loco ID from prog track

``POM( cv, value )``	Program CV value on main

``LCD( row, msg )``	Write message on LCD/OLED if fitted

``BROADCAST( msg )`` Broadcast to all throttles/JMRI on serial and WiFi

``PRINT( msg )``	Print diagnostic message to Serial Monitor

``SERIAL( msg )``	Writes direct to Serial (Serial0/USB)

``SERIAL1( msg )``	Writes direct to Serial1

``SERIAL2( msg )``	Wri1tes direct to Seria2

``SERIAL3( msg )``	Writes direct to Serial3

EX-RAIL Functions
__________________

``PAUSE``	E-STOP all locos and PAUSE all other EX-RAIL tasks until RESUMEd

``RESUME``	Resume all paused tasks, including loco movement

``RESERVE( block_id )``	Reserve a block (0-255). If already reserved, current loco will STOP and script waits for block to become free

``FREE( block_id )``	Free previously reserved block

``START( sequence_id )``	Start a new task to execute a route or sequence

``SETLOCO( loco )``	Set the loco address for this task

``SENDLOCO( cab, route )``	Start a new task send a given loco along given route/sequence

``AUTOSTART``	A task is automatically started at this point during startup

``DRIVE( analog_pin )``	Not complete, DO NOT USE

``ROSTER( cab, name, func_map )``	Provide roster info for WiThrottle

Loco DCC Functions
___________________

``ESTOP``	Emergency stop loco

``FWD( speed )``	Drive loco forward at DCC speed 0-127 (1=ESTOP)

``REV( speed )``	Drive logo in reverse at DCC speed 0-127 (1=ESTOP)

``SPEED( speed )``	Drive loco in current direction at DCC speed (0-127)

``STOP``	Set loco speed to 0 (same as SPEED(0) )

``FON( func )``	Turn on loco function

``FOFF( func )``	Turn off loco function

``INVERT_DIRECTION``	Switches FWD/REV meaning for this loco

Sensor input and Event Handlers 
________________________________

``AT( sensor_id )``	Wait until sensor is active/triggered

``ATTIMEOUT( sensor_id, timeout_ms )``	Wait until sensor is active/triggered, or if the timer runs out, then continue and set a testable "timed out" flag

``ATGTE( analogpin, value )``  Waits for analog pin to reach value

``ATLT ( analogpin, value )`` Waits for analog pin to go below value

``AFTER( sensor_id )``	Waits for sensor to trigger and then go off for 0.5 seconds

``LATCH( sensor_id )``	Latches a sensor on (Sensors 0-255 only)

``UNLATCH( sensor_id )``	Remove LATCH on sensor

``ONCLOSE( turnout_id )``	Event handler for turnout close. Note that there can be only one defined ONCLOSE event for a specific turnout.

``ONTHROW( turnout_id )``	Event handler for turnout thrown. Note that there can be only one defined ONCLOSE event for a specific turnout.

``ONACTIVATE( addr, sub_addr )``	Event handler for 2 part DCC accessory packet value 1

``ONACTIVATEL( linear )``	Event handler for linear DCC accessory packet value 1

``ONDEACTIVATE( addr, sub_addr )``	Event handler for 2 part DCC accessory packet value 0

``ONDEACTIVATEL( linear )``	Event handler for linear DCC accessory packet value 0

``WAITFOR( pin )``	Wait for servo to complete movement

Action Output Functions
________________________

``SET( pin )``	Set an output pin HIGH

``RESET( pin )``	Reset output pin (set to LOW)

``CLOSE( turnout_id )``	Close a defined turnout

``THROW( id )``	Throw a defined turnout

``GREEN( signal_id )``	Set a defined signal to GREEN (see SIGNAL)

``AMBER( signal_id )``	Set a defined signal to Amber. (See SIGNAL)

``RED( signal_id )``	Set defined signal to Red (See SIGNAL)

``FADE( pin, value, ms )``	Fade an LED on a servo driver to given value taking given time

``LCN( msg )``	Send message to LCN Accessory Network

``SERVO( id, position, profile )``	Move an animation servo. Do NOT use for Turnouts. (profile is one of Instant, Fast, Medium, Slow or Bounce)

``SERVO2( id, position, duration )``	Move an animation servo taking duration in ms. Do NOT use for Turnouts

``XFON( cab, func )``	Send DCC function ON to specific cab (eg coach lights) Not for Loco use - use FON instead!

``XFOFF( cab, func )``	Send DCC function OFF to specific cab (eg coach lights) Not for Loco use - use FON instead!

``ACTIVATE( addr, sub_addr )``	Sends a DCC accessory packet with value 1

``ACTIVATEL( linear )``	Sends a DCC accessory packet with value 1 to a linear address

``DEACTIVATE( addr, sub_addr )``	Sends a DCC accessory packet with value 0

``DEACTIVATEL( addr )``	Sends a DCC accessory packet with value 0 to a linear address