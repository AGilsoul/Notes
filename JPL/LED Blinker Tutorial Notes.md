### Project Setup
- Create project:
	- ```fprime-bootstrap project```
- In project folder, activate venv:
	- ```. fprime-venv/bin/activate```
- Generate a build cache
	- ```fprime-util generate```
- To build the whole project:
	- ```fprime-util build```
	- With name led-blinker
### Requirements
- What is the project?
	- Blink an LED using a Raspberry Pi
- Start with the requirements!
- System Requirements:
	- ![[Pasted image 20241021134919.png]]
- High-Level Software Requirements
	- ![[Pasted image 20241021134937.png]]
	- The software shall stop LED blinking in response to a command
	- The software shall accept a command to change the blink rate
- Component Level Requirements
	- ![[Pasted image 20241021134954.png]]
### Design Patterns
- Driver design pattern (LED-BLINKER-006, GPIO pin!)
	- Need GPIO driver
- Rate group (we need to blink an LED at a repetitive interval)
	- Need rate group
	- Need LED manager
### Component Design
- ![[Pasted image 20241022084153.png]]
- Input ports on the left, output ports on the right
- Ports:
	- run: invoked at a set rate from the rate group, used to control LED blinking
	- gpioSet: invoked by the `Led` component to control the GPIO driver
	- the rest are standard ports
- Commands:
	- BLINKING_ON_OFF: turn the LED blinking on/off
- Events:
	- SetBlinkingState: emitted when the component sets the blink state
	- BlinkIntervalSet: emitted when the component blink interval parameter is set
	- LedState: emitted when the LED is driven to a new state
- Telemetry Channels:
	- BlinkingState: state of the LED blinking
	- LedTransitions: count of the LED transitions
- Parameters:
	- BLINK_INTERVAL: LED blink period in number of rate group calls
### Create the Component
- In led-blinker, go to Components
	- ```cd Components```
- Create a new componet
	- `fprime-util new --component`
	- Name: Led
	- Namespace: Components
	- Active component
	- Commands enabled
	- Telemetry enabled
	- Events enabled
	- Parameters enabled
	- Add to CMakeLists.txt
	- Generate implementation files
### Commands and Events
- Replace the following in Led.fpp
```FPP
# One async command/port is required for active components
        # This should be overridden by the developers with a useful command/port
        @ TODO
        async command TODO opcode 0
```
- With
```FPP
# One async command/port is required for active components
# This should be overridden by the developers with a useful command/port

	@ Command to turn on or off the blinking LED
	async command BLINKING_ON_OFF (
		onOff: Fw.On @< Indicates whether the blinking should be on or off
	)

	# $ in front of state to use it as a variable name, since it is a keyword
	@ Reports the state we set to blinking
	event SetBlinkingState($state: Fw.On) \
		severity activity high \
		format "Set blinking state to {}."

	@ Reports the interval we set the LED to blink at
	event BlinkIntervalState(interval: U32) \
		severity activity high \
		format "Set blinking interval to {}."

	@ Reports that the LED state has changed
	event LedState(onOff: Fw.On) \
		severity activity low \
		format "Set LED state to {}."
```
### Component Implementation
- In led-blinker/Components/Led
	- `fprime-util impl`
	- `mv Led.template.hpp Led.hpp`
	- `mv Led.template.cpp Led.cpp`
	- `fprime-util build`
### Component State
- In Led.hpp private variables
```C++
Fw::On m_state = Fw::On::OFF; //! Keeps track if LED is on or off
U64 m_transitions = 0; //! The number of on/off transitions that have occurred from FSW boot up
U32 m_toggleCounter = 0; //! Keeps track of how many ticks the LED has been on for
bool m_blinking = false; //! Flag: if true then LED blinking will occur else no blinking will happen
```
- `fprime-util build` (to ensure it builds correctly)
### Command and Events
- In Led.cpp
```C++
void Led::BLINKING_ON_OFF_cmdHandler(FwOpcodeType opCode, U32 cmdSeq, Fw::On onOff) {
    this->m_toggleCounter = 0;               // Reset count on any successful command
    this->m_blinking = Fw::On::ON == onOff;  // Update blinking state

	this->log_ACTIVITY_HI_SetBlinkingState(onOff);  // Report blinking state event

    // TODO: Report the blinking state (onOff) on channel BlinkingState.
    // NOTE: This telemetry channel will be added during the "Telemetry" exercise.

    // Provide command response
    this->cmdResponse_out(opCode, cmdSeq, Fw::CmdResponse::OK);
}
```
- `fprime-util build`
### Initial Component Integration
 - In led-blinker
	 - `fprime-util new --deployment`
	 - Deployment name: LedBlinker
	 - TcpServer
	 - Add component to project.cmake
	 - `fprime-util build`
### Adding Led Component to the Deployment
- In led-blinker/LedBlinker/Top/instances.fpp active component instances
	- Defines Led instance called led
```FPP
instance led: Components.Led base id 0x0E00 \
	queue size Default.QUEUE_SIZE \
	stack size Default.STACK_SIZE \
	priority 95
```
- In led-blinker/LedBlinker/Top/topology, add led to list of instances
```FPP
instance led
```
- in led-blinker
	- `frpime-util build`
### Testing the Topology
- `fprime-util build`
- `fprime-gds --ip-client`
- Test your implemented commands and check they initiate events, etc.
## LED Blinker: Component Design and Implementation, Continued
### Telemetry
- In led-blinker/Components/Led/Led.fpp, after added events, add BlinkingState telemetry channel of type Fw.On, and LedTransitions telemetry channel of type U64
```FPP
@ Telemetry channel to report blinking state.
telemetry BlinkingState: Fw.On

@ Telemetry channel to report number of LED transitions
telemetry LedTransitions: U64
```
### Parameters
- In led-blinker/Components/Led/Led.fpp, add a parameter for blinking interval after tlm channels
```FPP
@ Blinking interval in rate group ticks
param BLINK_INTERVAL: U32 default 1
```
### Additional Ports
- In led-blinker/Components/Led/Led.fpp, add two ports for run and gpioSet after params
```FPP
@ Port receiving calls from the rate group
async input port run: Svc.Sched

@ Port sending calls to the GPIO driver
output port gpioSet: Drv.GpioWrite
```
### Input Port Implementation
- In led-blinker/Components/Led, autogenerate stub functions for run input port
	- `fprime-util impl'
- Copy the following from Led.template.hpp -> Led.hpp
```FPP
PRIVATE:
    // ----------------------------------------------------------------------
    // Handler implementations for user-defined typed input ports
    // ----------------------------------------------------------------------

    //! Handler implementation for run
    //!
    //! Port receiving calls from the rate group
    void run_handler(NATIVE_INT_TYPE portNum,  //!< The port number
                     NATIVE_UINT_TYPE context  //!< The call order
                     ) override;
```
- Copy the following from Led.template.cpp -> Led.cpp
```FPP
// ----------------------------------------------------------------------
// Handler implementations for user-defined typed input ports
// ----------------------------------------------------------------------

void Led ::run_handler(NATIVE_INT_TYPE portNum, NATIVE_UINT_TYPE context) {
    // TODO
}
```
- Implement the run handler in led-builder/Components/Led/Led.cpp
```C++
void Led ::run_handler(NATIVE_INT_TYPE portNum, NATIVE_UINT_TYPE context) {
    // Read back the parameter value
    Fw::ParamValid isValid = Fw::ParamValid::INVALID;
    U32 interval = this->paramGet_BLINK_INTERVAL(isValid);
    FW_ASSERT((isValid != Fw::ParamValid::INVALID) && (isValid != Fw::ParamValid::UNINIT),
              static_cast<FwAssertArgType>(isValid));

    // Only perform actions when set to blinking
    if (this->m_blinking && (interval != 0)) {
        // If toggling state
        if (this->m_toggleCounter == 0) {
            // Toggle state
            this->m_state = (this->m_state == Fw::On::ON) ? Fw::On::OFF : Fw::On::ON;
            this->m_transitions++;
            // TODO: Report the number of LED transitions (this->m_transitions) on channel LedTransitions

            // Port may not be connected, so check before sending output
            if (this->isConnected_gpioSet_OutputPort(0)) {
                this->gpioSet_out(0, (Fw::On::ON == this->m_state) ? Fw::Logic::HIGH : Fw::Logic::LOW);
            }

            // TODO: Emit an event LedState to report the LED state (this->m_state).
        }

        this->m_toggleCounter = (this->m_toggleCounter + 1) % interval;
    }
    // We are not blinking
    else {
        if (this->m_state == Fw::On::ON) {
            // Port may not be connected, so check before sending output
            if (this->isConnected_gpioSet_OutputPort(0)) {
                this->gpioSet_out(0, Fw::Logic::LOW);
            }

            this->m_state = Fw::On::OFF;
            // TODO: Emit an event LedState to report the LED state (this->m_state).
        }
    }
}
```
- `fprime-util build`
### Command Implementation Continued
- In led-blinker/Components/Led/Led.cpp, report blinking state in BLINKING_ON_OFF
```C++
this->tlmWrite_BlinkingState(onOff);
```
- `fprime-util build`
### Parameter Implementation
- In led-blinker/Components/Led/Led.hpp, add signature of method to react to parameter update
```C++
//! Emit parameter updated EVR
//!
void parameterUpdated(FwPrmIdType id  //!< The parameter ID
					  ) override;
```
- F Prime provides this function, so it is called when a parameter is updated automatically
- Implement this function in led-blinker/Components/Led/Led.cpp
```C++
void Led ::parameterUpdated(FwPrmIdType id) {
    Fw::ParamValid isValid = Fw::ParamValid::INVALID;
    switch (id) {
        case PARAMID_BLINK_INTERVAL: {
            // Read back the parameter value
            const U32 interval = this->paramGet_BLINK_INTERVAL(isValid);
            // NOTE: isValid is always VALID in parameterUpdated as it was just properly set
            FW_ASSERT(isValid == Fw::ParamValid::VALID, static_cast<FwAssertArgType>(isValid));

            // Emit the blink interval set event
            // TODO: Emit an event with, severity activity high, named BlinkIntervalSet that takes in an argument of
            // type U32 to report the blink interval.
            break;
        }
        default:
            FW_ASSERT(0, static_cast<FwAssertArgType>(id));
            break;
    }
}
```
- `fprime-util build`
### Do it Yourself
- Fill out all of the TODOs in the Led.cpp file in the run_handler and parameterUpdated functions
- parameterUpdated:
```C++
void Led::parameterUpdated(FwPrmIdType id) {
    Fw::ParamValid isValid = Fw::ParamValid::INVALID;
    switch (id) {
      case PARAMID_BLINK_INTERVAL: {
        // Read back the parameter value
        const U32 interval = this->paramGet_BLINK_INTERVAL(isValid);

        // NOTE: isValid is always VALID in parameterUpdated as it was just properly set
        FW_ASSERT(isValid == Fw::ParamValid::VALID, static_cast<FwAssertArgType>(isValid));
        
        // Emit the blink interval set event
        // Log blinking interval
        this->log_ACTIVITY_HI_BlinkIntervalSet(interval);
        break;
      }
      default:
        FW_ASSERT(0, static_cast<FwAssertArgType>(id));
        break;
    }
  }
```
- run_handler
```C++
void Led ::
    run_handler(
        FwIndexType portNum,
        U32 context
    )
{
	// Read back the parameter value
	Fw::ParamValid isValid = Fw::ParamValid::INVALID;
	U32 interval = this->paramGet_BLINK_INTERVAL(isValid);
	FW_ASSERT((isValid != Fw::ParamValid::INVALID) && (isValid != Fw::ParamValid::UNINIT),
			  static_cast<FwAssertArgType>(isValid));
			  
	// Only perform actions when set to blinking
	if (this->m_blinking && (interval != 0)) {
	  // If toggling state
	  if (this->m_toggleCounter == 0) {
		// Toggle state
		this->m_state = (this->m_state == Fw::On::ON) ? Fw::On::OFF : Fw::On::ON;
		this->m_transitions++;
	
		// Report number of LED transitions on channel LedTransitions
		this->tlmWrite_LedTransitions(this->m_transitions);
	
		// Port may not be connected, so check before sending output
		if (this->isConnected_gpioSet_OutputPort(0)) {
		  this->gpioSet_out(0, (Fw::On::ON == this->m_state) ? Fw::Logic::HIGH : Fw::Logic::LOW);
		}
	
		// Emit an event LedState to report the LED state
		this->log_ACTIVITY_LO_LedState(this->m_state);
	  }
	  this->m_toggleCounter = (this->m_toggleCounter) % interval;
	}
	// We are not blinking
	else {
	  if (this->m_state == Fw::On::ON) {
		// Port may be connected, so check before sending output
		if (this->isConnected_gpioSet_OutputPort(0)) {
		  this->gpioSet_out(0, Fw::Logic::LOW);
		}
		this->m_state = Fw::On::OFF;
	
		// Emit an event LedState to report the LED state
		this->log_ACTIVITY_LO_LedState(this->m_state);      
	}
}
```
## LED Blinker: Full System Integration
### Adding GPIO Driver to Topology
- In led-blinker/LedBlinker/Top/instances.fpp, in the Passive Component section
	- `instance gpioDriver: Drv.LinuxGpioDriver base id 0x4C00`
- Add the instance in led-blinker/LedBlinker/Top/topology.fpp
	- `instance gpioDriver`
### Wiring the led Component Instance to the gpioComponent Component Instance and Rate Group
- In led-blinker/LedBlinker/Top/topology.fpp, add the connections block to connect the ports in our Led component
```FPP
# Named connection group
connections LedConnections {
  # Rate Group 1 (1Hz cycle) ouput is connected to led's run input
  rateGroup1.RateGroupMemberOut[3] -> led.run
  # led's gpioSet output is connected to gpioDriver's gpioWrite input
  led.gpioSet -> gpioDriver.gpioWrite
}
```
### Configuring the GPIO Driver
- In led-blinker/LedBlinker/Top/LedBlinkerTopology.cpp method configureTopology, configure GPIO pin 13 for our gpioDrive
```FPP
Os::File::Status status =
        gpioDriver.open("/dev/gpiochip4", 13, Drv::LinuxGpioDriver::GpioConfiguration::GPIO_OUTPUT);
if (status != Os::File::Status::OP_OK) {
	Fw::Logger::log("[ERROR] Failed to open GPIO pin\n");
}
```
- Also `#include <Fw/Logger/Logger.hpp>` at the top
- `fprime-util build` in led-blinker/LedBlinker
## Unit Testing
### Unit Test Generation
- use `fprime-util` to generate a unit test outline, in led-blinker/Components/Led/CMakeLists.txt, after `register_fprime_module()`:
```CMake
set(UT_SOURCE_FILES
  "${CMAKE_CURRENT_LIST_DIR}/Led.fpp"
)
set(UT_AUTO_HELPERS) ON # Additional Unit-Test autocoding
register_fprime_ut()
```
- In led-blinker/Components/Led
	- `fprime-util generate --ut`
	- `fprime-util impl --ut`
	- `mkdir -p test/ut`
	- `mv LedTest* test/ut/`
- Update led-blinker/Components/Led/CMakeLists.txt to add the testing files to `UT_SOURCE_FILES`
```CMake
set(UT_SOURCE_FILES
    "${CMAKE_CURRENT_LIST_DIR}/Led.fpp"
    "${CMAKE_CURRENT_LIST_DIR}/test/ut/LedTestMain.cpp"
    "${CMAKE_CURRENT_LIST_DIR}/test/ut/LedTester.cpp"
)
set(UT_AUTO_HELPERS ON) # Additional Unit-Test autocoding
register_fprime_ut()
```
- Test the skeleton unit tests: `fprime-util check`
### Add a New Test Case
- In led-blinker/Components/Led/test/ut/LedTester.hpp replace `toDo`
```C++
void testBlinking();
```
- In led-blinker/Components/Led/test/ut/LedTester.cpp rename `toDo` definition
```C++
void LedTester::testBlinking() {

}
```
- In led-blinker/Components/Led/test/ut/LedTestMain.cpp
```
TEST(Nominal, TestBlinking) {
	Components::LedTester tester;
	tester.testBlinking();
}
```
- Run `fprime-util check`
### Write a Test Case
- Add the following to the `testBlinking` method
```C++
// This test will make use of parameters. So need to load them.
    this->component.loadParameters();

    // Ensure LED stays off when blinking is disabled
    // The Led component defaults to blinking off
    this->invoke_to_run(0, 0);     // invoke the 'run' port to simulate running one cycle
    this->component.doDispatch();  // Trigger execution of async port

    ASSERT_EVENTS_LedState_SIZE(0);  // ensure no LedState change events we emitted

    ASSERT_from_gpioSet_SIZE(0);  // ensure gpio LED wasn't set

    ASSERT_TLM_LedTransitions_SIZE(0);  // ensure no LedTransitions were recorded
```
- Run `fprime-util check`
- Now enable blinking
```C++
// Send command to enable blinking
this->sendCmd_BLINKING_ON_OFF(0, 0, Fw::On::ON);
this->component.doDispatch(); // Trigger exeuction of async command
ASSERT_CMD_RESPONSE_SIZE(1); // Ensure a command response was emitted
ASSERT_CMD_RESPONSE(0, Led::OPCODE_BLINKING_ON_OFF, 0,
				    Fw::CmdResponse::OK); // Ensure the expected command response was emitted
```
- Check component state after three cycles
```C++
// Possible LED states
    Fw::On LedStates[2] = { Fw::On::ON, Fw::On::OFF };
    // Possible GPIO states
    Fw::Logic gpioStates[2] = { Fw::Logic::HIGH, Fw::Logic::LOW };
    // Cyclce LED On/Off
    for (uint16_t i = 0; i < 3; i++) {
      this->invoke_to_run(0, 0);
      this->component.doDispatch(); // Trigger execution of async port
      ASSERT_EVENTS_LedState_SIZE(i+1); // check number of LedState events
      ASSERT_EVENTS_LedState(i, LedStates[i % 2]); // check LedState event value
      ASSERT_from_gpioSet_SIZE(i+1); // check number of entries in GPIO from port
      ASSERT_from_gpioSet(i, gpioStates[i % 2]); // check value of entry in GPIO from port
      ASSERT_TLM_LedTransitions_SIZE(i+1); // check number of entries in LedTransitions channel
      ASSERT_TLM_LedTransitions(i, i+1); // check value of entry in LedTransitions channel
    }
```
### Adding a Parameter Test Case
- The next test will test the adjustment of `BLINK_INTERVAL`
- Add a new test case called `testBlinkInterval`, in LedTester.cpp, LedTester.hpp, LedTestMain.cpp
```C++
void LedTester::testBlinkInterval() {
    // Enable LED Blinking
    this->sendCmd_BLINKING_ON_OFF(0, 0, Fw::On::ON);
    this->component.doDispatch(); // Trigger execution of async command
    // Adjust blink interval to 4 cycles
    U32 blinkInterval = 4;
    this->paramSet_BLINK_INTERVAL(blinkInterval, Fw::ParamValid::VALID);
    this->paramSend_BLINK_INTERVAL(0, 0);
    ASSERT_EVENTS_BlinkIntervalSet_SIZE(1);
  
    // TODO: Add logic to test adjust blink interval
  }
```
### Checking Coverage
- In led-blinker/Components/Led
	- `fprime-util check --coverage`
	- And find the `coverage/coverage.html` file to view the report
		- Open with `explorer.exe coverage.html`
## System Testing
### Intro to F Prime System Testing
- In Components/Led/test/int/led_integration_tests.py, add this:
```Python
def test_cmd_no_op(fprime_test_api):
	"""Test command CMD_NO_OP

	Test that CMD_NO_OP can be sent and return without errors
	"""
	fprime_test_api.send_and_assert_command("LedBlinker.cmdDisp.CMD_NO_OP")
```
- If running locally
	- `fprime-util generate -DFPRIME_USE_STUBBED_DRIVERS=ON`
- In led-blinker/LedBlinker
	- `pytest ../Components/Led/test/int/led_integration_tests.py`
- Do this while running F Prime GDS in another terminal!
### Testing SetBlinking On/Off
- In Components/Led/test/int/led_integration_tests.py
```Python
import time

from fprime_gds.common.testing_fw import predicates

def test_blinking(fprime_test_api):
	"""Test that LED component can respond to ground commands"""
	# Send command to enable blinking, then assert expected events are emitted
    blink_start_evr = fprime_test_api.get_event_pred("LedBlinker.led.SetBlinkingState", ["ON"])
    led_on_evr = fprime_test_api.get_event_pred("LedBlinker.led.LedState", ["ON"])
    led_off_evr = fprime_test_api.get_event_pred("LedBlinker.led.LedState", ["OFF"])

    fprime_test_api.send_and_assert_event(
        "LedBlinker.led.BLINKING_ON_OFF",
        args=["ON"],
        events=[blink_start_evr, led_on_evr, led_off_evr, led_on_evr],
    )

    # Send command to stop blinking, then assert blinking stops
    #TODO: Define blink_stop_evr
    fprime_test_api.send_and_assert_event(
        "LedBlinker.led.BLINKING_ON_OFF", args=["OFF"], events=[blink_stop_evr]
    )
```
### Test BlinkingState Telemetry
- In `test_blinking()`, add the following
```Python
# Assert that blink command sets blinking state on
blink_state_on_tlm = fprime_test_api.get_telemetry_pred("LedBlinker.led.BlinkingState", "ON")
fprime_test_api.assert_telemetry(blink_state_on_tlm, timeout=2)
# Assert that blink command sets blinking state off
# Timeouts added to ensure we wait to get the expected telemetry
blink_state_off_tlm = fprime_test_api.get_telemetry_pred("LedBlinker.led.BlinkingState", "OFF")
fprime_test_api.assert_telemetry(blink_state_off_tlm, timeout=2)
```
### Advanced: Test LedTransitions Telemetry
- Check that blinking stops after turning blinking off (check that LedTransitions channel is no longer emitted). In `test_blinking()`
```Python
time.sleep(1) # Wait 1 second to allow in-progress telemetry to be sent
# Save reference to current tlm history so we can search against future tlm
telem_after_blink_off = fprime_test_api.telemetry_history.size()
time.sleep(2) # Wait to receive telemetry after stopping blinking
# Assert that blinking has stopped and that LedTransitions is no longer updated
fprime_test_api.assert_telemetry_count(
	0, "LedBlinker.led.LedTransitions", start=telem_after_blink_off
)
```
- Verify that LedTransitions increments over time
```Python
# Assert that the LedTransitions channel increments
results = fprime_test_api.assert_telemetry_count(
	predicates.greater_than(2), "LedBlinker.led.LedTransitions", timeout=4						 
)
ascending = True
prev = None
for res in results:
	if prev is not None:
		if not res.get_val() > prev.get_val():
			ascending = False
			fprime_test_api.log(
				f"LedBlinker.led.LedTransitions not in ascending order: First ({prev.get_val()}) Second ({res.get_val()})"
			)
	prev = res
assert = fprime_test_api.test_assert(
	ascending, "Expected all LedBlinker.led.LedTransitions updates to ascend.", True								 
)

```
