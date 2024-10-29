# Day 1
- Golden Binary: Failsafe version of software (typically what is launched) that can be reset to automatically if trying to update software does not work
## Flight Software System Engineering
- Definition - Designing the various functions of the fight software and their interactions with the spacecraft and each other
- Understanding the interfaces to the system
	- How it interacts with the environment, payload, etc.
	- What is the avionics platform?
	- What is the execution environment?
		- RTOS, Non-RTOS, bare metal
- Understanding the requirements
	- Performance
		- Deadlines
		- Computing load
		- Data throughput
	- Resources
		- Energy
		- Memory
		- Storage
	- Activities
		- Behaviors to complete mission
	- Testability
### Layering
- Hardware layers are designed (for the most part) to be modular and reusable
- Software should be layered for the same purpose
	- Modularity
	- Reusability
	- Testability
### Software System Architecture
- Decompose software into modules. Advantages:
	- Separation of function
	- Definition of interfaces
	- Behavioral characteristics
		- Rate groups
		- Concurrency
		- Data flow
	- Test at the unit level
	- Ownership
	- Reusability
### Software Stack
- Application (behavior of spacecraft in general)
- Manager (manages a peripheral, like an IMU)
- Driver (generic software piece that interacts with a set of wires of a known protocol, UART driver, etc.)
	- Example: IMU
	- Drone Guidance (application) -> (get IMU rates from ImuMgr)
	- ImuMgr (manager) -> (send I2C commands to I2CDriver)
	- I2CDriver (driver)
- You should not call up the stack! Application calls manager, manager calls driver
	- This still allows for callbacks, but behavior is still dictated by application
- Layering allows:
	- Separation of concerns
	- Reusability
	- Reliability
	- Replacement
	- Testing
	- Fault protection
## Architecture, Requirements, and Design
### Software Architecture
- Software Architecture is different from Software Design
- Software Architecture describes the skeleton and high level infrastructure of the software
	- Independent of the application domain
- Software Design describes the implementation of the domain within the software architecture
	- Breaks the software down into elements
	- Describes the purpose of each element
	- Describes the inner workings of each element
- Software Architecture is important
	- Antidote to software chaos
	- Glue and foundation that holds the software together
- Be vigilant against architectural erosion
	- Maintain the architectural integrity throughout development
- Examples:
	- Client-Server
	- Data-Base
- Component based architecture
	- A component is a unit of computation with a well-defined interface
	- No symbolic dependencies on other components
		- Compile, Load, and Execute independently of other components
	- Components only communicate with each other through ports
	- Component encapsulates threads and queues and states
### Software Architecture Attributes
- Modularity
	- Modularity imporves software development quality and maintainability
	- Decompose the software into a colleciton of modules (components or libraries)
- Module Coupling
	- The extent that modules (components) are related to each other
	- High coupling is bad
		- One component controlling the flow of another (control coupling)
		- Two components share a common data space (data coupling)
		- Components share common code/data structures (content coupling)
	- Module Cohesion
		- The extent that data and functions inside a module (component) belong to each other
		- High cohesion is good
			- All the functions and data for a device pertain to the operation of the device
			- A Telemetry Manager component only processes telemetry channesl and not commands
	- Strive for Low Coupling, High Cohesion
- Portability
- Reusability
### Other Architectural Principles
- No dynamic memory allocation after initialization
	- Deterministic behavior
- No multiple class inheritance
- Limit class hierarchy
- Integrity checks
	- Asserts
- Performance
	- The software should perform well in a resource constrained environment
- Keep it simple
	- If your code is complicated and ugly, it's probably wrong
### Software Architectural Views
- Software architecture is captured by different views or perspectives
- These perspectives encompass the software architectural model
- These perspectives are not mutually exclusive
### Architecture Views: Software Task View
- Tasks are execution threads
- Tasks communicate via event messages which are placed on the task input queue
- Tasks sleep until a message arrives and then process events off their input queue
- Tasks have execution priority
- Tasks can be:
	- Rate-group driven (1 Hz, 10 Hz etc.)
	- Data drive
	- Background (continuous low priority task)
### Architecture Views: Component View
- Different components for:
	- RS422 Driver
	- Encoder Manager
	- Estimator
	- Actuator Manager
	- Controller
	- Telemetry Manager
	- SpaceWire Driver
	- Command Dispatcher
### Software Component Encapsulation
- A component encapsulates
	- A task
	- A state-machine
	- An input queue
	- Input and Output Ports
### Software Requirements
- Requirements are typically layered
	- Mission requirements
	- Project requirements
	- System requirements
	- Subsystem requirements
	- Flight software requirements
	- Component requirements
- Requirements are traced up and down
	- Higher level requirements are satisfied by lower level requirements
	- Lower level requirements are traced back to upper level requirements
- Focus on Flight Software Requirements
	- FSW Requirements should be:
		- Concise
		- Unambiguous
		- Testable
		- A shall statement
	- Help you, the developer, know what you are implementing
	- A contract to the project on what you are implementing
		- Avoid nasty surprises when your software is finally delivered
	- Examples
		- The FSW shall produce spacecraft health information via telemetry
			- Context info can also be provided as an auxiliary
				- Spacecraft health information consists of the following...
- Before design and code:
	- First answer the question of "What are we building?", NOT "How do we implement it?"
- Thoroughly understand what we are building
	- Follow a process that will develop the FSW Requirements
- Structured Analysis is a process that generates a Data Flow Diagram:
	- Answers the question: What are we building
	- Naturally produces the FSW Requirements
	- Not meant to show design
	- Does not capture timing, sequence, and synchronization of processes
	- Breaks the system down into smaller manageable chunks
	- A graphical diagram that is relatively easy to understand for software/system engineers
	- A clear and detailed information about the system - processes and boundaries
	- Shows the logic of the data flow
### Data Flow Diagram Model
- Process (data transformation)
- Data flows
- Data stores
- Levels
	- Context diagram
		- Focus on system interfaces
	- Process decomposition
		- Overviews of system processing
	- Deeper dives
		- Detailed views
![[Pasted image 20241021100347.png]]
### Flight Software Context Diagram
- ![[Pasted image 20241021100416.png]]
### Top Data Flow Diagram
- ![[Pasted image 20241021100535.png]]
### GNC Data Flow Diagram
- ![[Pasted image 20241021100559.png]]
- The FSW shall convert the GNC Command to a 3 axis pointing attitude command with an earth reference frame
- The FSW shall send the GNC state as a telemetry channel
- The FSW shall compute a quaternion position (a, b, c, d) from the Linear Acceleration (x, y, z) and current GPS data and send the position out as a telemetry channel
- The FSW shall compute 3 axis torque commands (x, y, z) from a pointing attitude command and the angular velocity data
### From Requirement to Design
- Data Flow Diagrams (DFD's) specify:
	- Flight Software Interfaces
	- The flow of data to processes
	- The processing and transformation of the data
	- Data stores (state)
	- Used to derive flight software requirements
	- Natural transition to implementation
- Components specify:
	- The concrete implementation of the DFD
	- The periodic scheduling
	- The threads of execution (component type)
	- The component data interfaces
	- The component port types
		- Synchronous or Asynchronous
### Design Decisions
- GPSMgr component
- RadioMgr component
- IMUMgr component
- UARTDrv component
- FlashDrv component
- CameraMgr component
- ImageProcess component
- GNC component
- F' components ChanTlm and TlmPacketizer
- F' component CmdDispatcher
- F' components RateGroupDrv and ActiveRateGroup
### Component Topology Diagram
- ![[Pasted image 20241021101138.png]]
### Software Component State
- For each component create a crisp notion of state with a state-machine that has the following identified:
	- Discrete states
	- Signals that cause state transitions
	- Actions that get invoked on a Transition
	- State entry and exit behavior
- Encapsulate the state-machine logic within a single function or class
- Avoid a fuzzy notion of state with a collection of Boolean flags and scattered state logic
### UART Driver Component State-Machine Example
- UART Driver Requirements
	- The UART Driver shall handle commands from different clients
	- The UART Driver shall process one command at a time waiting for an Ack or Nack
	- The UART Driver shall retry the command upon receiving a Nack up to a specified limit
	- UART Driver shall time-out after a specified duration
	- ![[Pasted image 20241021101508.png]]
### Embedded State Machines in a Component
- ![[Pasted image 20241021101607.png]]
### LedBlinker Tutorial
- The LedBlinker tutorial can be implemented as a formal state machine using the latest F Prime state machine modeling capabilities
- In the Led component Start and Stop command handlers:
	- Send signals Blinking_On and Blinking_Off to the state machine
- In the Led component Run input port handler:
	- Send signal RTI to the state machine
- Developer implements:
	- turnOnLED()
		- Turn on the LED
	- turnOffLED()
		- Turn off the LED
	- checkTime()
		- incr counter
		- if counter > Threshold
			- Send TIMEOUT signal
			- Initialize counter
- LedBlinker state machine current implementation
- ![[Pasted image 20241021102855.png]]
- LedBlinker state machine in Plant UML
- ![[Pasted image 20241021102951.png]]
- LedBlinker state machine future implementation
- ![[Pasted image 20241021103009.png]]
### Take-Home Message
- Understand and maintain the software architecture throughout development
- Understand what you are building before making design choices
- Generate requirements from a well understood analysis model
	- i.e. Data Flow Diagram or other models
- Capture the design as a topology model
	- Component instances
	- Component types
		- Active (thread of execution)
		- Passive
		- Queued
	- Component interfaces
		- Data structure
		- Synchronous/Asynchronous
	- Maintain a crips notion of state for each component
		- Use https://github.com/JPLOpenSource/STARS to specify the state machine
		- Use FPP to specify the state machine (when released)
## F Prime Introduction
### F': A Component Architecture
- Definition: The F' Component Architecture is a design pattern based on an architectural concept combined with a software architectural framework
- Not just the concepts, but framework classes and tools are provided for the developer/adapter
- Implies patterns of usages as well as constraints on usage
- Centered around the concept of "components" and "ports"
- Uses code generation to produce code to implement common framework logic
	- Developer specifies in FPP (F Prime Prime) DSL (Domain Specific Language)
- Developer writes implementation classes to implement interfaces
- Boxes are components, lines are ports:![[Pasted image 20241021105945.png]]
### Characteristics of Components
- Encapsulates behavior
- Components are not aware of other components
- Localized to one compute context
- Interfaces are via strongly typed ports
	- Ports are formally specified interfaces
	- No direct calls to other components
- Context for threads
- Executes commands and produces telemetry
### Characteristics of Ports
- Encapsulates typed interfaces in the architecture
	- Think C++ class with one interface method
- Point of interconnection in the architecture
- Ports are directional; there are input and output ports
	- Direction is direction of invocation, not necessarily data flow. Ports can retrieve data
- Ports can connect to 3 things:
	- Another typed port
		- Call is made to method on attached port
	- A component
		- Incoming port calls call component provided callback
	- A serialized port
		- Port serializes call and passes as data buffer (more to come)
- All arguments in the interface are serializable, or convertible to a data buffer. There are built-in types supported by the framework; user can write custom types
- Ports can have return values, but that limits use
	- Only returns data when the component has synchronous interface
	- No serializable connections since serialization passes a data buffer but does not return one
- Pointers/references allowed for performance reason
- Multiple output ports can be connected to a single input port
### A Component Topology
- Components are instantiated at run time
- They are then connected via ports into a Topology, or a specific set of interconnected components
- There are no code dependencies between components, just dependencies on port interface types
- Alternate implementations can easily be swapped
	- E.g. simulation versions
### Deployment Variations
- F Prime allows components to be reconfigured in different configurations for testing. Ingenuity had 11 different deployments that converged on a flight deployment that shared components
- Allowed decoupled testing and support of different venues
- Simulation versions can substitute driver/bus components for sim versions
- Typically the farther down the hierarchy that this switching is done, the more work is required
- ![[Pasted image 20241021110849.png]]
### Component Type Hierarchy
- Hierarchy consist of:
	- core framework classes
	- generated classes that implemented architecture features
	- Developer written classes that implement interfaces and project-specific logic
### Component Types
- User specifies type of component in FPP. Types are:
- Passive Component
	- No thread or queue
	- Port interface calls are made directly to user derived class methods
	- Example: I2C driver under a manager (read and write one call, want data back immediately)
- Queued Component
	- No thread
	- A queue is instantiated, and asynchronous port calls are serialized and placed on queue
	- Implementation class makes call to base class to dispatch class to implementation class methods for asynchronous ports
		- Can be made from any implementation class function
		- Thread of execution provided by caller to a synchronous port
	- Don't have their own compute context, use whatever is being called by (I think), so you can have a rate group call them in a queue, to say do this, wait, do this, wait, do this (I think)
	- Good way to integrate a library into the F Prime framework
	- Example: Tricky, function we want to drive at a certain rate, but want to be able to hand commands, such as managers
- Active Component
	- Component has thread of execution as well as queue
	- Thread dispatches port calls from queue as it executes based on thread scheduler
	- Example: Command dispatcher (always responding, on its own thread)
- Calls to output port are on thread of implementation functions
	- Thread making call is dependent on port type
### Port Characteristics
- The way incoming port calls are handled is specified by the component FPP
- Input ports an have three attributes:
	- Synchronous - port calls directly invoke derived functions without passing through queue
	- Guarded - port calls directly invoke derived functions, but only after locking a mutex shared by all guarded ports in a component
	- Asynchronous - port calls are placed in a queue and dispatched on thread emptying the queue
- A passive component can have synchronous and guarded ports, but no asynchronous ports (no queue). Calls execute on thread of calling component
- A queued component can have all three port types
- Active can have all three port types
### Serialization
- Serialization is a key concept in the framework
- Definition: Taking a specific set of typed valued or function arguments and converting them in an architecture-independent way into a data buffer
- Use can define complex types in FPP and code generator will generate classes that are serializable for use internally and to and from ground software
### Serialization Ports
- A special optional port that handles serialized buffers
- Takes as input a serialized buffer when it is an input port, and outputs a serialized buffer when it is an output port
- Can be connected to *any* types port (almost)
	- For input port, calling port detects connection and serializes arguments
	- For output port, serialized port calls interface on types port that deserializes arguments
	- *Not* supported for ports with return types
- Useful for generic storage and communication components that don't need to know type
	- Allows design and implementation of C&DH (command and data handling) components that can be reused
- ![[Pasted image 20241021113354.png]]
### Commands, Telemetry, Events, and Parameters
- The code generator provides a method of implementing commands, telemetry, events, and parameters
- Component FPP specifies arguments and types
- Data for service is passed in serialized form
- The code generator parses arguments and types and generates code to convert arguments to serialized data for vice versa
- Calls implementation functions with deserialized arguments (commands) or provides base class functions to implement calls (telemetry, event, and parameters)
### Commands
- Commands are sent from the ground system to the spacecraft to initiate activities
- Component command FPP specifies:
	- Opcode, mnemonic, and arguments
	- Synchronization attribute
- Implementation class implements function for each command
	- Framework code deserializes arguments from argument data buffer
- Autocoder automatically adds ports for registering commands, receiving commands, and reporting an execution status
### Events
- Events are asynchronous notifications of spacecraft activity
- Component event FPP specifies:
	- ID, name, severity, and arguments
	- Format specifier string
- Code generated base class provides function to call for each event with typed arugments
- Code generator automatically adds ports for retrieving time tag and sending event
- ![[Pasted image 20241021113849.png]]
### Telemetry
- Telemetry is data periodically sampled and emitted
- Component telemetry FPP specifies channels that have:
	- ID, name, and data type
	- Format specifier string
- Code generated base class provides function to call for each channel with typed argument
	- Called by implementation class
- Code generator automatically adds ports for retrieving time tag and sending channelized data
- ![[Pasted image 20241021114033.png]]
### Parameters
- A means of storing non-volatile state
	- Allows modification of software behavior with no software update
- Component FPP specifies parameters that have:
	- ID, name, and data type
	- Optional default value
- Code generator automatically adds port for retrieving parameters
- During init, a public method in the class is called that gets the params and stores copies locally
	- Can be called again if param updated
- Code generated base class provides function to call for each parameter to retrieve stored copy
	- Implementation class can call whenever param value needed
- Framework provides code generation to manage, but projects must write specific storage component
	- Basic file-based storage provided by F Prime core component
### F Prime Core Components
- ![[Pasted image 20241021114310.png]]
## Introduction to FPP Modeling
### Software Modeling in F Prime
- Developers write models
	- Define components and ports
	- Specify connections in a topology
	- Define the flight-ground interface
- Tools generate code
	- C++ code for implementation and unit testing
	- JSON for command and telemetry dictionaries
- Developers fill in the mission-specific details in C++
### F Prime Modeling: Block Diagram
- ![[Pasted image 20241021130312.png]]
### FSW Modeling: Benefits
- Clear statement of design intent
- Auto-generation of architecture diagrams
- Automatic checking of correctness properties
- Auto-generation of "boilerplate" implementation code
- Auto-generation of ground dictionaries
- Potential for integration with system modeling
### FPP (F Prime Prime)
- A domain-specific modeling language for F Prime
	- Free and open source
	- Simple and easy to use
- Provides
	- A succinct and readable source representation
	- Robust error checking and reporting
	- Good integration between the model and the generated code
- Integrated with the F Prime build system
### Constants and Types
```FPP
@ A constant // annotation, sorta like doxygen comments
constant c = 5

@ An enum
enum E { X, Y }

@ An array type
array A = [3] U32

@ A struct type
struct S {
	x: U32
	y: string
} default { x = 1, y = "hello" }
```
### Ports and Components
```FPP
@ A port for carrying an F32 value 
port F32Value(value: F32)

@ A component for adding F32 values
active component F32Addder {
	@ Input: An array of two F32 values
	async input port f32ValueIn: [2] F32Value

	@ Output: A single F32 value
	output port f32ValueOut: F32Value
}
```
### Instances and Topologies
```FPP
@ Command dispatcher instance
instance cmdDisp: Svc.CommandDispatcher base id 0x0500 \
	queue size 20 \
	stack size Default.stackSize \
	priority 101

...

@ An example topology
topology Example {
	...

	@ Automatically insert all command connections
	command connections instance cmdDisp

	@ Command sequence connections
	connections Sequencer {
		cmdSeq.comCmdOut -> cmdDisp.seqCmdBuff
		cmdDisp.seqCmdStatus -> cmdSeq.cmdResponseIn
	}
}
```
### Ground Dictionaries
```FPP
active component Dictionaries {
	...

	@ An asynchrnonous command
	async command START(a: F32, b: U32) opcode 0x10

	@ An event report
	event Event(
		count: U32 @< The count
	) \
	  severity activity high \
	  id 0x10 \
	  format "The count is {}"

	@ A telemetry channel
	telemetry Channel: F64 id 0x10 update on change

	@ A parameter (ground-configurable constant)
	param Param: F64 default 2.0 id 0x10
}
```
### Topology Visualization
- Visualization tool uses simple layout algorithm
- Uses named connection groups to generate subgraphs
- ![[Pasted image 20241021131446.png]]
## Flight Software Design
### Software as a System
- Software is composed of more than one units ***components***
- Components are composed into a system or ***topology***
- Essential to understand ***topology*** before designing components
- Examples:
	- Python uses composition of modules
	- Java is organized through class compositions
	- F' uses component topologies
### Software as a System (Static)
- Not so great!
- ![[Pasted image 20241021141938.png]]
```C++
/**
* Table of contents approach
*/
int main(int argc, char** argv) {
	int output = step1();
	step2(output);
}
/**
* Interacting services
*/
int main(int argc, char** argv) {
	MyService service1;
	OtherService service2;
	service1.register(service2);
}
```
### Software as a System: Web Browser Example
- This is better than above!
- ![[Pasted image 20241021142132.png]]
### System Design
- Topologies are compositions of components:
	- Set of components
	- Connections between components
- Identify system's components
- Identify connections between components
- Sketch software topology and software component interaction
- Examples:
	- Motor controller -> software motor manager
	- Motor controller -> hardware drive (UART, SPI, I2C, ...)
	- Radio communication -> radio component manager
	- Radio communication -> radio hardware driver (SPI, etc.)
### Component Breakdown: Embedded Example
- ![[Pasted image 20241021142351.png]]
### Interface Design
- Interfaces specify interactions between components:
	- Expose certain functionality
	- Protocol used for communication (function calls, register writes, IPv4)
	- May exchange data
- Identify functionality to be exposed
- Identify communication protocol
- Establish data to be exchanged and ownership of that data
- Examples
	- Event manager sends F' packets to radio manager via port call
	- Radio manager sends byte data via a function call to SPI driver
	- SPI driver writes hardware registers to trigger telecom transmission
### Interfaces: Embedded Example
- ![[Pasted image 20241021143036.png]]
### Component Design
- Components implement functionality of interfaces:
	- Implements and uses set of interfaces
	- Executes within a particular context
	- Produces, consumes, or manages data
- Identify interfaces implemented and used by component
- Select execution context
	- Caller, thread, ISR, etc.
- Determine needed shared data
- Examples:
	- IMU manager has I2C read, and I2C write ports and executes on callers thread
	- TCP driver uses Berkley socket interrace to IP stack on a dedicated read thread
### Component Design: Embedded Example
- ![[Pasted image 20241021142759.png]]
### Data and Ownership
- Identify shared data between components
- Establish how shared data is allocated, exchanged, and deallocated
- Designate the owner of shared data items at all times
- Identify off-nominal handling conditions
- Example:
	- Framer component allocates memory via buffer manager, receiving ownership of shared buffer
	- Framer delegates ownership to IPv4 driver via port call
	- IPv4 component delegates ownership back to buffer manager deallocating the shared memory
### Data Ownership: Embedded Example
- ![[Pasted image 20241021143016.png]]
### Off-Nominal Conditions
- Identify places where non-standard conditions can occur
- Identify the severity of the condition
- Identify the appropriate response to the condition
- Examples:
	- Hardware failure -> go to safe mode
	- Malformed user input or data -> emit warning event
	- Memory inconsistency -> full system reset
### Off-Nominal Conditions: Embedded Example:
- ![[Pasted image 20241021143227.png]]
## Flight Software Design Considerations
### Startup: Initialization and Allocation
- Finite resources are allocated at initialization to reduce risk
- Typical resources include: RAM, Threads, Critical Files
	- Try not to use new or malloc unless at the beginning of the program, instead of sometime in the middle where it can return errors and mess things up
		- If there is a problem it will happen right away, and will likely be caught in the testbed
	- Also start threads in the beginning, even if you don't use them right away, to avoid a crash in the middle of a mission
	- For critical files, preallocate in the beginning to ensure you have enough space
- Dynamic resources draw from preallocated pool; failures handled
- Implications:
	- Static memory or initialization allocated heap
	- Preallocated worker threads
	- Preallocated critical files
	- No recursion (unbounded allocation of stack frames)
	- Buffer managers, file managers handle unpredictable requests
### Synchronous Execution, Concurrency, Threads
- Synchronous execution uses caller's execution context
- Parallel execution has multiple execution contexts via threads
- Sharing data across contexts requires locking and/or queues
- "Ships passing in the night" (disagreement of data between contexts)
- Implications:
	- Must plan concurrency model
	- Shared resources must be handled appropriately
	- Messaging and scheduling must be thought through
	- Care must be taken with work requiring specific timing
### Deadlines, Timeliness
- Some processes have strict timing or deadlines
- Identify tasks that require strict timing, active processing, and background processing
- Strict timing needs specific deadline handling
- Implications:
	- Non-time sensitive work should be placed in low-priority background tasks
	- Critical work without deadlines occupies medium priority tasks
	- Work with specific deadlines goes in the highest priority tasks
### Faults, FATALS, and Error Handling
- Flight software is expected to protect the spacecraft
- Off-nominal conditions will always occur; the universe desires this
- Spacecraft operators need to understand cause of behavior
- Implications:
	- Uncontrolled reboots and crashes should be avoided
	- Logging of off-nominal conditions should occur
	- Spacecraft should be made safe before loss-of-control responses
### Data Serialization and Deserialization
- Data in RAM may be padded, expanded, or mixed with other values
- Bytes in RAM may have different orders between different machines
- Data in transit should be an array of bytes in specified order
- Implications:
	- Data exchange format must be well-specified and obeyed
	- Data cannot always be directly copied into buffers
- ![[Pasted image 20241021144758.png]]
## Modeling Flight Software in F'
### Topologies
- Topologies represent a network of components
- Contain instantiations of each component
- List connections between the ports of all the components
- ![[Pasted image 20241021144947.png]]
### Ports
- Represent interface to components
- Form a point-to-point network for communication
- Have arguments with specific types
- May have return values
- ![[Pasted image 20241021145039.png]]
### Components
- Represents concrete function in the system
- Come in three variants: Active, Passive, and Queued relating to execution context
- Communicates with other components via ports
- ![[Pasted image 20241021145200.png]]
## F' Design Patterns
### Adapter Pattern
- Adapts "something else" to work within F'
- Typically done by writing an F' component bridging functionality
- Adapts or adds concurrency and timeliness considerations
- Adds commands, events, and telemetry
- ![[Pasted image 20241021150453.png]]
### Rate Group Pattern
- Drives components at a set rate
- Simple provider of timeliness, allowing work at a set time
- Care must be taken with other forms of concurrency
- Care must be taken to not slip
- ![[Pasted image 20241021150535.png]]
### Hub Pattern
- Routes multiple F' ports across some communication layer
- Unwraps on the far side of the communication layer
- Allows for inter-process communication
- ![[Pasted image 20241021150616.png]]
### Manager-Worker Pattern
- Decouples long-running tasks from need for quick interaction
- Manager sends work to worker, worker responds back afterwards on lower priority thread
- ***Only*** Manager communicates with worker
- Parallels "worker thread" pattern
- ![[Pasted image 20241021151116.png]]
# Day 2
## Reducing Software Risk
- Flight code must perform well and be reliable
- C and C++ are widely used
	- High performance
	- Direct interface to hardware
- But they come with risks
	- Code can have undefined or unexpected behavior
	- Compiler can catch some errors
	- But many it can't (integer overflow, dangling pointers, etc.)
### Check Numeric Bounds
- Integer and FP types in C and C++ are not numbers
- They are approximations of numbers with a finite representation
	- Integer types have a limited range
	- FP types have a limited range and a limited precision
- In FSW, you should avoid using platform-specific integer types
	- Instead of int and unsigned, use int32_t and uint32_t
	- This way the bounds are known and are the same across platforms
	- Use int if and only if it's required by a a library or system interface
		- E.g. the return value of main is int
		- May C library calls return int as their error codes
### Checking Integer Conversion
- Unchecked conversion
```C
#include <cinttypes>

int32_t i64_to_i32(int64_t x) {
	return static_cast<int32_t>(x);
}
```
- Checked conversion
```C
#include <cinttypes>
#include <limits>

enum class Status { FAILURE, SUCCESS };

Status i64_to_i32(int64_t x, int32_t& result) {
	Status status = Status::FAILURE;
	if ((x >= std::limis<int32_t>::min()) && (x <= std::limits<int32_t>::max())) {
		result = static_cast<int32_t>(x);
		status = Status::SUCCESS;
	}
	return status;
}
```
### Checking Integer Overflow
- Unchecked Addition
```C
int32_t add_i32(int32_t a, int32_t b) {
	return a + b;
}
```
- Checked Addition
```C
Status add_i32(
	int32_t a,
	int32_t b,
	int32_t& result
) {
	int64_t c = static_cast<int64_t>(a) + static_cast<int64_t>(b);
	return i64_to_i32(c, result);
}
```
### Beware of Floating Precision Loss
### Use Bounded Loops
- Unbounded Loop
```C
while (true) {
	Queue::Item *item = nullptr;
	const Status status = queue.pop(item);
	if (status != Status::SUCCESS) {
		break;
	}
	dispatch(item);
}
```
- Bounded Loop
```C
const size_t queueDepth = queue.getDepth();
for (size_t i = 0; i < queueDepth; i++) {
	Queue::item *item = nullptr;
	const Status status = queue.pop(item);
	if (status != Status::SUCCESS) {
		break;
	}
	dispatch(item);
}
```
- Unbounded loops can run forever and hang a thread, bad in flight code
### Avoid Recursion
- This is a form of potentially unbounded loop
- Can also cause stack overflow
### Use Assertions
- An assertion is a macro expression ASSERT(e)
	- e is a boolean expression that is expected to evaluate to true
	- If e does evaluate to true, macro does nothing
	- Otherwise an assertion failure occurs
- When an assertion failure occurs
	- In unit testing, usually halt the program
	- In system and flight
		- At JPL, usually restart unless it would cause mission failure
### When to Use Assertions
- Use assertions when failure "cannot happen" (e.g., it indicates a bug)
	- Pointer is null just before dereference
	- Array index is out of bounds just before array access
	- Computed value is outside expected range
- Otherwise check for and handle the error (e.g. bad user input)
- Common pattern:
	- Check for and reject invalid input
	- After passing the check, assert that the input is valid before it is used
	- Assertion failure indicates a bug in the code that did the checking
- **Never assert that ground or user input is valid**
### Avoid Dangerous Code
- C and C++ have many traps for the unwary
- It is easy to write and run programs that have undefined behavior
	- Straighforward:
		- Read of uninitialized memory
		- Object access through dangling pointer
	- More subtle
		- Left shift of negative signed integer (C, C++)
		- Array index starting at pointer to base-class object (C++)
- Some behavior is defined but obscure
	- Tricky operator precedence (use parentheses to disambiguate)
	- Complex C macros (avoid)
	- Forgetting to declare a base-class destructor *virtual* (C++) (don't do it)
### Avoid Side Effects in Macro Arguments
```C++
#define ABS(X) ((X) < 0) ? -(X): (X))

int 32_t a = get_a();
a++;
int32_t b = ABS(a); // as intended

int_t a = get_a():
int32_t b = ABS(a++); // probably not
```
### Avoid Side Effects in Index Expressions
```C++
return a[i] + a[i++]; // evaluation order is not guaranteed
```
### Guards Against Out-of-Bounds Array Access
```C++
class History {
...
public:
	Item& getItemAt(size_t index) {
		ASSERT(index < HISTORY_SIZE);
		return m_items[index];
	}
	...
private:
	Item m_items[HISTORY_SIZE];
};
```
- Use error checking or assertions to ensure that array accesses stay in bounds (unless performance is extremely critical)
### Avoid Dangling Pointers
- x will go out of scope and this will return a dangling pointer
```C++
int32_t* stackReturn() {
	int32_t x;
	return &x;
}
```
### Avoid Dynamic Memory Allocation
- Allocate all memory you need when the program starts
- Don't sue malloc/new or free/delete after initialization
- If you need dynamic behavior, use a memory pool
### Run Static Analysis
- Compile with --Wall --Werror
- Run static analysis tools
	- Free tools, e.g., as GitHub actions
	- Commercial tools such as Coverity and CodeSonar
- Run the tools as often as feasible
	- Compiler: On every build
	- GitHub actions: On every pull request to main
	- Other tools: On every code review
### Follow a Coding Standard
- A coding standard is an agreed-upon set of rules for developing code
- Enforcement is usually a combo of
	- Automated tools
	- Code review
- Examples
	- JPL C, C++
	- Google C++ Style Guide
### The Power of 10
1. Restrict to simple control flow constructs
2. Do not use recursion and give all loops a fixed upper-bound
3. Do not use dynamic memory allocation after initialization
4. Limit functions to no more than ~60 lines of text
5. Target an average assertion density of 2% per module
6. Declare data objects at the smallest possible level of scope
7. Check the return value of non-void functions; check the validity of parameters
8. Limit the use of preprocessor to file inclusion and simple macros
9. Limit the use of pointers. Use no more than 2 levels of dereferencing
10. Compile with all warnings enabled, and use source code analyzers
## Suggestions for Coding Style
### Encapsulate Operations on Data
- In C++, use an object-oriented style
- In C, write functions that accept pointers to structs as arguments
- Avoid inline operations on data structures
- Just okay:
```C++
Value value;
Status status = NOT_FOUND;
for (...) {
	if (...) {
		value = ...
		status = FOUND;
		break;
	}
}
```
- Better:
```C++
class Map {
	...
	Status find(const Key& key, Value& value) const {
		...
	}
	...
}

Value value;
const Status status = map.find(key, value);
```
### Use Helper Classes (C++ Only)
- In C++, use helper classes inside the component implementation
- You can make them inner classes of the component class
	- C++ inner classes can't refer to members of outer classes directly
	- To work around this, you can
		- Pass a reference to the outer class into the constructor for the inner class
		- Store the reference in a member of the inner class
- Alternatively, you can use non-inner classes
	- But give the class names appropriate prefixes
	- For example: MyComponentHelper instead of MyComponent::Helper
### Factor Functions into Layers
- Factor functions into
	- High-level structure (e.g., testing, branching, looping)
	- Detailed work
- Write a high-level function that has just the logical structure
- Have it call functions to do the detailed work
- Just okay
```C++
if (this->mode == ...) {
	...
}
...
```
- Better
```C++
if (this->mode == ...) {
	this->emitTelemetry();
}
this->handleCommands();
```
### Keep Functions Short
- Move large code blocks to separate functions
- Commented code blocks should become new functions
### Avoid Multiple Return Statements
- Avoid multiple return statements within the same function
	- They make it hard to follow the logic
	- They can lead to resource leaks
- Where possible, functions should follow this pattern (especially in C)
	- Claim resources
	- Do work
	- Release resources
	- Return
- Error handling can use the state propagation pattern
### Propagate State to a Single Return Point
- Consider using this pattern:
	- It avoids multiple returns
	- It makes state handling explicit
```C++
Status status = operation1();
if (status == SUCCESS) {
	status = operation2();
}
...
if (status == SUCCESS) {
	finish();
}
else {
	// Report error based on status
}
```
### Prefer Passing of State through Arguments
- Don't use shared variables to pass state between helper methods
	- Put mutable data in struct or class members
	- Update a member only when it is part of the object's persistent state
- If a function updates shared data, its name should announce that fact
- Misleading
```C++
U32 calculateSize(const Data& d) {
	U32 size = 0;
	for (...) {
		size += ...
		globalState += ...
	}
	return size;
}
```
- Better
- Move the state update out of the function or name the function `calculateSizeAndUpdateState`
### Avoid Boolean Flag Arguments
- Confusing
```C++
void doSomething(bool foobarMode) {
	...
}
// True means foobar mode
doSomething(true);
```
- Better
```C++
void doSomething(Mode mode) {
	...
}
...
doSomething(Mode::FOOBAR);
```
### Don't Use Boolean Values for Return Status
- Confusing
```C++
// true means success
const bool status = doSomething();
```
- Better
```C++
// Status::SUCCESS means success
const Status status = doSomething();
```
- Boolean values should represent truth or falsity, not success or failure
- Functions that return boolean values should be side-effect-free
### Use Fixed-Width Types as Much as Possible
- **Do not** assume that int is a 32-bit integer
	- It may not be on some platforms
	- If you need a 32-bit integer, use int32_t (from stdint.h (C) or cstdint(C++))
	- F Prime renames these standard types, e.g., I32
- Use int only when required by the environment
	- For example, to hold the return value of a C library function that returns int
	- In this case the size of the value is platform-dependent
### Avoid Using %d, %u, %x
- **Do not** bind fixed-width types to %d, %u, %x, etc. in printf formats
- Use fixed-width equivalents: PRId32, PRIu32, PRIx32, etc.
- Bind only int to %d, unsigned to %u, long int to %lu, etc.
	- And prefer not to use these types at all
- Violating this can introduce portability issues
### Initialize All Variables
- Don't leave memory uninitialized unless required
	- E.g., to preserve error state on restart
- Don't write `I32 x;` . Instead write `I32 x = 0;`
	- This includes
		- Local/temp variables
		- Members in zero-argument constructors
	- If there's no meaningful initial value, just pick one
		- This guards against nondeterministic behavior
		- E.g., on some platforms or runs, variables happen to have the right value
- Initialize all members of structs, arrays, and classes
	- In C you can use `memset(0)`
	- In C++ you can use `= {}`
### Use Pointers and References Wisely
- In C++, prefer references to pointers
	- References cannot be NULL
	- Use pointers only when the value is assigned after the variable is created
		- Initialize each pointer variable x = NULL
		- Then assign a non-null pointer value to x
		- Then assert x != NULL before each use of x
- Use references (in C++) or pointers (in C) for concision
	- For example, don't write `this->a.b.c[i]` over and over
	- Instead
		- Write `C& c = this->a.b.c` once
		- Write `c[i]` several times
### Use Flight-Like Memory Allocation
- Rigorously avoid the following
	- Using malloc or new after initialization
	- Allocating large objects on the stack
	- Allocating variable-size arrays on the stack
- Prefer simple memory allocation when possible
	- malloc or new is often not needed
	- Objects are statically sized, so you can statically allocate them
	- Some platforms may place limitations on the size of static allocations
- F Prime provides a MemAllocator interface that you can use
### Wrap Accesses to Buffer Pointers and Arrays
- Avoid bare array access `A[i]`
	- It's not safe, because it's not bounds checked
	- In C++, use classes
		- Make the bare array a private member
		- Provide access through overloaded [] operators with bounds checking
	- In C, use functions that access arrays with bounds checking
- Avoid accessing bare buffer pointers `*p`
	- Read or write buffer objects that store a pointer and a size
	- Make the pointer private
	- When accessing the buffer, provide bounds checking against the size
### Use const as Much as Possible
- C/C++ const syntax is weird but worth mastering
- C and C++
	- `const T x`: x is a const T
	- `const T *x`: x is a pointer to const T (T is const)
	- `T* const x`: x is a const pointer to T (x is const)
	- `const T *const x`: x is a const pointer to const T (T and x are const)
- C++ only
	- `void f() const`: f is a const member function
	- `T& x`: Like `T *const x`
	- `const T& x`: Like `const T *const x`
- const annotations are useful for
	- Keeping track of where state is changing
	- Keeping track of inputs and outputs
- Example: `void f(const Buffer& in, Buffer& out)`
	- in is a const input
	- out is a mutable output
- In C++>=11, constexpr is preferred to variable declarations
	- `constexpr F32 x = 3.0;`
	- Can use x in more places than with `const F32 x = 3.0;`
### Avoid Direct Use of C Lib String Functions
- **Do not use** strcpy, etc. They are not bounds-checked
- strncpy, etc. (note the n) are better, but still problematic
	- strncopy in particular does not guarantee null termination
	- This behavior invites memory overruns and obscures bugs
- In C, wrap problem functions in other functions with better behavior
- In F Prime, avoid direct use of C string functions altogether
	- In F Prime, use a subclass of StringBase
	- For example, operator= is a safer alternative to strncpy
## Data Structures
### Data Structures in FSW
- Many standard data structures allocate and free elements on demand
- In FSW we do not do that
- FSW data structures must
	- Use a fixed total size of memory to hold their data
	- Call malloc or new only at FSW startup
- We will focus on data structures that meet these requirements
### Data Structure Operations
- **Insert**: Add an element to a collection
- **Remove**: Take an element out of a collection
- **Find**: Search for and return an element
- **Iterate**: Visit each element
### Basic Data Structures
- **Array**: An index collection of fixed size
- **List**: An ordered, non-indexed collection
- **Set**: An unordered collection
- **Map**: An unordered mapping from keys to values
- **Queue**: A collection that supports insert and remove only
### Implementing Data Structures
- Building blocks
	- C/C++ arrays to hold data elements
	- Pointers or array indices for links between elements
- For FSW, prefer arrays to pointers
	- Often, you don't need a linked data structure at all: just use an array
	- If you do need links, then
		- All memory is pre-allocated at FSW startup
		- So all memory has an associated index into a pre-allocated array
		- Using the index is safer, because it can be bounds checked
- **Avoid C++ STL** (it uses hidden new and delete operations)
- Consider using ETL: https://www.etlcpp.com (stl-like data structures with embedded flight principles!)
### Array
- Data representations: A C/C++ array
- Operations
	- **Insert**: N/A
	- **Remove**: N/A
	- **Find**: Bounds-checked access to the underlying array
	- **Iterate**: Loop over elements
### Array: Recommended Use
- Use if
	- The number of elements is fixed; and
	- The elements map to indices; and
	- The index set is numeric and dense (most numbers in index set used)
- Don't use if
	- The number of elements grows or shrinks; or
	- There is no numeric index set; or
	- There is a sparse numeric index set (not many numbers in index set used)
### Array-Based LIFO Queue (Stack)
- LIFO means "last in, first out"
- Data representation
	- An array A of elements
	- A variable size that stores the data size. Initially it is zero
- Operations (check that size is in bounds)
	- **Insert (enqueue)**: Add the element at `A[size]` and increment size
	- **Remove (dequeue)**: Decrement size
	- **Find**: N/A
	- **Iterate**: N/A
- Recommended use: If you need LIFO behavior
### Array-Based FIFO Queue
- FIFO means "first in, first out"
- Implementation is similar to LIFO, but it
	- Uses a circular array (index mod array size)
	- Tracks starting position and size
	- Performs enqueue at one end of the array, dequeue at the other
		- Dequeue moves start forward
- Exercise:
	- Sketch the implementaation
	- Pay special attention to the cases where the queue is full and empty
- Example: Utils/Types/CircularBuffer.{hpp, cpp} in F Prime
### Array-Based Set/Map: Data Representation
- Data representation
	- An array A of
		- Elements (small elements); or
		- Indices into elements stored in a different array (large elements)
	- A variable size that stores the current size. Initially it is zero
- Operations (check that size is in bounds)
	- **Insert**: Add the element at `A[size]` and increment size
	- **Remove**:
		- Swap the element with the element at `A[size-1]`
		- Decrement Size
	- **Find**: Linear search of first size elements of A
	- **Iterate**: Loop over the first size elements of A
### Array-Based Set/Map: Recommended Use
- Use if
	- The find operation is rate; or
	- The number of elements is small
- Don't use if
	- find is common and number of elements is large
	- Use hash set/map instead
- Variant: Mark nodes as unused instead of swapping nodes
	- Avoids moving nodes around
	- Insert is slower
	- Example: Command Dispatcher component in F Prime
### Linked List: Data Representation
- An array A of nodes
	- Each node has a member next that stores a link to the next node or a special NONE value that is distinct from any index
- A variable head that stores the index of the first node in the list
	- Initially contains NONE
- A variable freeHead that stores the index of the first node in the free list
	- Initially all nodes in A are in the free list:
		- `freeHead = 0`
		- `for 0 <= i < n - 1 do A[i].next = i+1`
		- `A[n-1].next = NONE`
### Linked List: Operations
- **Insert**:
	- `tmp = A[freeHead.next]`
	- `A[freeHead].next = head`
	- `head = freeHead`
	- `freeHead = tmp`
- **Remove**: 
	- Let N be the node to remove. Search through the linked nodes for N, starting at head. Maintain a reference L to the previous link
	- Set `L.next = N.next` in general, `head = N.next` at the front
	- Insert N into the free list
- **Find**: Linear search through the linked nodes, starting at the head
- **Iterate**: Loop over the linked nodes, starting at head
### Linked List: Recommended Use
- Use this if you want several lists to share nodes from a common array
- Otherwise use an array-based set/map
	- Significantly less complex
### Hash Set/Map: Data Representation
- An array A of linked lists that share the same array of nodes
- Each list node stores a key only (set) or a key-value pair (map)
- Initially the lists are all empty
### Hash Set/Map: Operations
- **Insert**:
	- Use a hash function to convert the key into an index I into A
	- Insert the key into the linked list at `A[i]`
- **Remove**: 
	- Use the hash function to convert the key into an index I
	- Perform a remove on the list at `A[i]`
- **Find**: 
	- Use the hash function to convert the key into an index I
	- Perform a find on the list at `A[i]`
- **Iterate**: Iterate over the array and each list in the array
### Hash Set/Map: Recommended Use
- Use if
	- There is no numeric index set for the data; or
	- The index set is sparse; or
	- The data set is large, and you need fast find capability
- Otherwise use an array or array-based set/map
- Notes
	- Good hash function
	- Interleaving inserts and finds can cause nondeterministic performance
	- Example: TlmChan component in F Prime
# Day 3
## Unit Testing
### Intro
- Testing is an important part of FSW dev
- We usually divide testing into at least two phases
- **Unit Testing**: Test individual units (e.g. F Prime components)
- **Integration Testing**: Test the integrated system
### Goals of Unit Testing
- Cover all component-level requirements
- Achieve high code coverage
- Achieve a reasonable amount of state and path coverage
### Mapping Tests to Requirements
- If you have component-level requirements, then
	- The requirements should drive the tests
	- It's good to maintain a record of how the tests cover the requirements
- The mapping can be recorded
	- In a separate table or spreadsheet; or
	- In the tests themselves (comments, console output, etc.)
- Examples in F Prime Svc Components
### Writing Unit Tests
- A standard approach
	- Write a first complete test that covers a requirement
	- Write a second complete test, etc.
	- Where there is overlap, copy-paste (bad!) or refactor into functions
- A more disciplined approach
	- Write functions that test individual behaviors
	- Write tests by composing the functions
	- Developed support for this approach (rule-based testing)
- First approach is "easier" but can lead down a bad path
	- Code duplication can get out of hand
	- For large test sets, becomes a maintenance problem
### Writing Test Code
- Treat unit tests as a programming problem
	- Apply the same style guidelines as for flight code
		- Pay attention to code structure and clarity
		- Avoid code duplication
		- Avoid undefined behavior, dangerous C and C++ code
	- Apply (almost) the same level of rigor as for flight code
		- Some FSW rules don't apply to test code
		- E.g., malloc/new and free/delete are allowed
- Don't throw down messy code to get coverage
	- Requires a bit more up-front work
	- But will pay off with more readable, maintainable, and modifiable tests
### Picking Inputs
- Good tests require good inputs
	- "Good" means "exercising enough behavior"
	- "Enough" can be made precise on some dimensions
		- For example, code coverage
		- In practice it can be a qualitative judgment
- Techniques for picking inputs
	- Use arbitrary values, e.g., 42: "Easy" but not robost
	- Use significant values, e.g., boundary values
	- Use random values
	- Use an analysis tool to pick values
- In F Prime, the STest framework has a random value picker
### Modeling External Behavior
- A unit under test is part of a complete system
- When unit testing a component, you must model external behavior
	- For example, Command Dispatcher sends commands, expects responses
	- Test code must model behavior "receive command, send response"
- To do this, write a test harness or "mock system"
	- Think about relevant system behavior
	- Model it at the appropriate level of abstraction
- This approach fosters modularity of tests
	- Write one abstracted system model
	- Use it in many tests
### Testing Against the Interface
- When writing unit tests for a component, test against the interface
	- For example, send commands, send data on ports
	- Avoid directly updating the state (member variables) of the component
		- You can read internal component state to verify it is OK
		- But try to only modify state through interface
	- This approach leads to more structured, better tests
- Sometimes you want to test a function in a component implementation
	- E.g., if it implements a complex algorithm
	- In this case, write a test against the function interface
### Testing Components That Use Libraries
- Sometimes components call into external libraries
- You can link against the library in the test
- However, it's usually better to link against a mock or stub library
	- Avoids complex library behavior
	- Makes it easier to induce behaviors for testing (e.g., inject faults)
	- May be the only option on some platforms
### Checking Code Coverage
- Code coverage means
	- Of all lines of code in the source program, which lines were run at least once in some tests?
- Standard tools such as gcov can do this analysis
	- Compile tests with coverage flags
	- Run tests to generate .gcov files
	- Run gcov to analyze .gcov files and produce a report
- Report consists of source files with coverage annotations for each line
	- You can convert this into a percentage (covered / total)
	- Or use a wrapper tool such as gcovr to produce pretty output
### Achieving Code Coverage
- It's usually easy to get about 80% code coverage
	- Think about "ordinary" behavior of the code
	- Write tests that exercise it
- The cases that remain are usually rare or off-nominal behaviors
	- Covering these cases may require more thought
	- You may have to
		- Reason backwards from the desired behavior to synthesize inputs
		- Inject faults into library behaviors
- At some point you may hit diminishing returns
	- One common case is assert(0) in code that should never be reached
	- Use judgment
### Limitations of Code Coverage Analysis
- 100% code coverage is not a panacea!
- It says nothing about 
	- State coverage: Which system states were tested?
	- Path coverage: Which paths through the code were tested?
- In general, full checking of state and path coverage is not possible
## The F Prime Unit Test Framework
### Framework Overview
- ![[Pasted image 20241023090308.png]]
### Test Framework Classes
- *TesterBase* (auto-generated)
	- The base class for testing a component
	- Provides a harness for unit tests
- *GtestBase* (auto-generated)
	- Derived from TesterBase
	- Includes
		- Headers for the Google Test framework
		- F Prime-specific macros
- *Tester* (developer-written from generated template)
	- Class that contains tests as members
	- Contains the component under test as a member
### The TesterBase Class (Auto-Generated)
- Its interface is the "mirror image" of the component C under test
	- For each output port in C, an input port called a **from port**
	- For each input port in C, an output port called a **to port**
	- For each from port
		- A history H of data received
		- A virtual input handler that stores its arguments into H
- It provides utility methods for writing tests
	- Send commands to C
	- Send invocations of ports to C
	- Get and set parameters of C
	- Set the time
### The GTestBase Class (Auto-Generated)
- Derived from TesterBase
- Includes headers for Google Test framework
	- https://github.com/google/googletest
	- Supports test assertions such as ASSERT_EQ(x, 3)
- Adds F Prime-specific macros for checking
	- Telemetry received on telemetry from ports
	- Events received on event from ports
	- Data received on user-defined from ports
- Factored into a separate class so its use is optional
### The Tester Class
- Autocode provides a template
- You add tests as public methods
- You can also write tests in a derived class of Tester
### Writing Unit Tests
- Generate the test classes
	- In the component directory, run `fprime-util impl --ut`
	- Move the classes to the test/ut directory
- Add public test methods to *Tester*
- Write a main.cpp file that calls the test methods:
```C++
#include "ComponentTester.hpp"

TEST(TestCaseName, TestName) {
	Namespace::ComponentTester tester;
	tester.testName();
}
...
int main(int argc, char** argv) {
	::testing::InitGoogleTest(&argc, argv);
	return RUN_ALL_TESTS();
}
```
### Sending Commands
```C++
// Send command
this->sendCOMMAND_NAME(
	cmdSeq, // Command sequence number
	arg1, // Argument 1
	arg2 // Argument 2
);
// Manually dispatch off of queue
this->component.doDispatch();
// Assert command response
ASSERT_CMD_RESPONSE_SIZE(1);
ASSERT_CMD_RESPONSE(
	0, // Index in the histroy
	Component::OPCODE_COMMAND_NAME, // Expected command opcode
	cmdSeq, // Expected command sequence number
	Fw::COMMAND_OK // Expected command response
)
```
### Checking Events
```C++
// Send command and check response
...
// Assert total number of events in history
ASSERT_EVENTS_SIZE(1);
// Assert number of a particular event
ASSERT_EVENTS_EventName_SIZE(1);
// Assert arguments for a particular event
ASSERT_EVENTS_EventName(
	0, // Idnex in history
	arg1, // Expected value of argument 1
	arg2 // Expected value of argument 2
);
```
### Checking Telemetry
```C++
// Send commands and check response
...
// Assert total number of tlm entries in history
ASSERT_TLM_SIZE(1);
// Assert number of entries on a particular channel
ASSERT_TLM_ChannelName_SIZE(1);
// Assert value for a particular entry
ASSERT_TLM_ChannelName(
	0, // Index in history
	value // Expected value
);
```
### Checking User-Defined Output Ports
```C++
// Send commands and check response
...
// Assert total number of entries on from ports
ASSERT_FROM_PORT_HISTORY_SIZE(1);
// Assert number of entries on a particular from port
ASSERT_from_PortName_SIZE(1);
// Assert value for a particular entry
ASSERT_from_PortName(
	0, // Index in history
	arg1, // Expected value of argument 1
	arg2 // Expected value of argument 2
);
```
### Setting Parameters
- In a test of component C, you can write
```C++
this->paramSet_ParamName(
	value, // Parameter value
	Fw::PARAM_VALID // Parameter status
);
```
- This stores the arguments into member variables of *TesterBase*
- When C invokes its *ParamGet* port, it will receive the arguments
### Setting the Time
- In a test of component C, you can write
```C++
this->setTime(time);
```
- *time* is an Fw::Time object
- When C invokes its TimeGet port, it will receive the value *time*
### Building and Running Unit Tests
- To generate starter code for unit testing
	- Go to component directory (not the test/ut directory)
	- Run `fprime-util impl --ut`
- To build unit tests
	- Go to the component directory (not the test/ut directory)
	- Run `fprime-util build --ut`
- To run unit tests
	- Go to component directory (not the test/ut directory)
	- Run `fprime-util check
### Analyzing Code Coverage
- Generate the analysis
	- Go to the components directory (not the test/ut directory)
	- Run `fprime-util check --coverage`
- Review the results in the coverage directory
	- coverage.html: Summary
	- Coverage.[filename].cpp.[hash].html: Details
## Systems Testing
### Systems Testing Intro
- Testing software within the full running system
- Includes flight-like hardware and ground support
- Also called **integration testing** and similar to **regression testing**
- "Test as you fly, fly as you test"
### Why Systems Testing?
- A system is more than the sum of its parts
- Some requirements can only be verified at system level
- Hardware operations need testing on hardware
- "Eat our own dog food"
### Thundering Herd
- The more parts we have, the more problems arise
### System Testing Tools
- Ground Data Systems
	- Operate spacecraft from earth
	- May be used to conduct testing
- Testing Frameworks
	- Allow for automated testing
	- Reporting, verification, reproducibility
- Hardware Simulations
	- Allows system testing without physical hardware
- Command Sequences
	- How the flight spacecraft is operated
	- May be used to construct tests
### Types of System Testing
- Manual Systems Testing
	- Operating the spacecraft by-hand to verify requirements
	- Running sequences and commands and verifying results
	- Ground system is used to send commands
	- More complete tests often require full test team
- Automated System Testing
	- System testing performed by software
	- Uses automated testing support from ground station
	- Automatically logs data, results, and verification products
	- May not require dedicated test team
- Continuous Integration
	- Extension of automatic testing
	- Triggered by software change request or other criteria
	- Requires no human interaction
	- Implements full regression testing
### System Testing with F Prime
- F Prime GDS
	- Provides GDS capabilities out-of-the-box for F Prime projects
	- Designed for system testing early in project development
	- Enables developers to:
		- Send commands
		- Receive events and telemetry
		- Uplink and downlink files
		- Design and upload sequences
		- Compose custom dashboards
- F Prime Integration Test Framework
	- Framework designed for integration testing with F Prime GDS
	- Allows users to write automated tests in Python
	- Tests dispatch commands and assert on events and telemetry
	- Creates excel test logs
	- Can be run within continuous integration system
## FSW Development Process
### FSW Development Process Overview
- Why is it important to have a development process?
	- Enables a higher chance of success by providing backing for adjusting scope and schedule
	- Improves quality
- Waterfall model fits in line with deadline driven development
	- The right level of process is important
- ![[Pasted image 20241023135646.png]]
- Development Phases
	- Requirements
		- Provide measurable constraints and characteristics from concepts of operations
	- Design
		- Provides blue print for software implementation given a set of requirements
	- Implementation
		- Provides a testable product for verification
	- Verification
		- Ensures implementation functionality and correctness
	- Each phase has a review to ensure readiness for the next phase and address any issues
### FSW Subsytem Level Process
- Requirements Phase
	- Why is it important to gather requirements?
	- Map from concept of operations to specific capabilities that can be designed and implemented
	- Manages assumptions
	- Heads off disagreements/misunderstanding between designers/implementors and their stakeholders
	- Provides the structure for
		- Measuring progress
		- Verifying we do what is needed
- Requirements Derivation
	- Understand project level requirements and concept of operations (ConOps) i.e. what is needed for the project
		- Decompose into various software components at high level
		- Functional breakdown rather than design
	- Most sub-system requirements are expected to be derived from a parent requirement, but some may be self-derived
	- Artifact: Requirements specification document
	- Conduct requirements review
	- ![[Pasted image 20241023140454.png]]
- Proof of Concept and Prototyping
	- Target OS and hardware platform
	- Compile and execute software on target
	- Communicate over planned interfaces
	- Data bandwidth and performance analysis
	- ![[Pasted image 20241023140504.png]]
- Design Phase
	- Trade studies and prototyping
	- Develop list of components with functionality description
		- Services
		- Communication
		- Hardware managers and drivers
		- Guidance and control
		- Science
		- Fault protections and mode management
	- List of planned releases by components
	- ![[Pasted image 20241023140523.png]]
	- Design Artifacts
		- Context diagrams
		- Interconnect block diagrams (topologies)
		- Sequence diagrams
		- Data flow diagrams
		- Component block diagrams
	- Define ports and types to be used across components
	- Analyze resource utilization and performance
		- Memory, CPU, I/O
	- Address any concurrency issues
	- Artifacts:
		- Software architecture and design documentation
		- Trade study results
		- Resource utilization and performance analysis
		- Receivables and deliverables list
		- FSW release delivery schedule
		- Budget and staffing plan
	- Conduct design review
	- ![[Pasted image 20241023140605.png]]
- Implementation Phase
	- Conduct component level reviews - requirements, design, implementation and unit-test, integrated test results, closeout
	- With good design, shouldn't be too hard
	- May require some design updates, but most design expected to be completed in design phase
	- Deployment
		- Functional integration of software components
	- Development test venues
		- Simulation
		- Protoype/development hardware
		- Testbed
		- EM
	- FSW builds
		- Prototype
		- Internal releases
		- External releases to support sub-system proof of concept
	- Artifacts:
		- FSW release package
			- FSW binaries, non-volatile parameter or config files
			- Documentation, build environment, config, etc.
			- Dictionaries
		- Test reports
		- Prelim requirements verification and validation matrix
	- Conduct delivery review
- Version Control
	- ![[Pasted image 20241023140935.png]]
- Verification Phase
	- Critical to overall software functionality and mission successs
	- Catching bugs early is cheaper and easier to fix
	- Driven by requirements verification
		- Performed using test scripts executed against a release deployment
	- FSW builds
		- External releases
	- Development test venues
		- Simulation
		- Testbed
		- EM
		- FM
	- Artifacts
		- FSW release package
			- FSW binaries, non-volatile parameter or config files
			- Documentation, build-environment, config, etc.
			- Dictionaries
		- Test reports
		- Requirements verification and validation matrix
	- Conduct delivery review / SRCR
	- ![[Pasted image 20241023141149.png]]
- Delivery Review
	- Release description document (RDD)
		- Change log
		- Version Identification
		- Project Overview and Release Description
		- Controlling Documents
		- Test Reports
		- Requirements Verification Summary
		- Idiosyncrasies and Known Issues
		- Problem Disposition
		- Detailed Contents
	- Users guide
		- Operational constraints
		- Usage guidelines
	- Software design documents
- Change Requests and Maintenance
	- ![[Pasted image 20241023141349.png]]
### Component Level Process
- Components Requirements Development
	- List assumptions relevant to component design
	- Develop component requirements table
		- Requirement ID for traceability in unit testing
		- Requirement description and rationale
		- Indicate verification method: unit-test, inspection, or analysis
		- Link to parent FSW sub-system level requirement
	- Conduct component requirements review
	- ![[Pasted image 20241023141509.png]]
- Component Design
	- Develop component software design document (SDD)
	- Component overview
	- Assumptions
	- Component level requirements
	- Design
		- Component block diagram
		- Sequence, dataflow, state transition, class diagrams
		- Port List
		- Custom data types
		- State
		- Port Behaviors
	- Commands, telemetry, events, and parameters
	- Reference datasheets and other technical documents as applicable
	- Component and port models (FPP)
	- Conduct component design review
	- Design checklist walkthrough
- Component Block Diagram and Ports List
	- ![[Pasted image 20241023141721.png]]
- Component Implementation
	- Use auto-coded component implementation template files as starting point
	- Review JPL C-Coding Standard (JPL Rules DocID 78115) and code checklist being used by the project for coding guidelines
	- Reference other mature F Prime components for C++ coding style
	- Code development
		- Port handler behaviors
		- State management
		- Command handlers
		- Telemetry and events
		- Parameters
	- Component compilation for all targets
	- Static analysis (SCRUB tool - GCC, Coverity, Code Sonar)
	- Conduct code review
	- Code checklist walkthrough
- Component Unit Testing
	- Component unit tests are developed using F Prime unit test harness
	- Traceability of each test-case to component level requirements
	- Code coverage analysis
	- Unit test output and coverage results
	- Conduct unit test review
	- Unit test checklist walkthrough
- Integrated Testing
	- Test component functionality with the integrated FSW build
	- Test venues
		- Simulation
		- Hardware
	- Test scripts
		- Send commands and verify telemetry/event
	- Test reports
		- Test as-runs
		- Telemetry and event logs
	- Requirements V&V
		- FSW sub-system level requirements V&V
		- Traceability for each requirement to all test-cases verifying that requirement
		- Traceability for each test case to all requirements being teste din that test case
	- ![[Pasted image 20241023142205.png]]
- Component Closeout
	- Verify all component requirements have been verified
	- Verify any additional design and code updates have been reviewed with checklist updates
	- Verify all other component checklists have been completed
		- Design
		- Code
		- Unit Test
	- Generate component metrics
		- Source lines of code (SLOC)
		- Number of commands, telemetry, events, parameters
	- Conduct component closeout review
	- Closeout checklist walkthrough
- Checklists
	- Can be tailored per the project's risk posture
		- Class D / CubeSats
			- May use a single simple checklist for the entire component dev process
		- Type 1 Missions
			- Separate detailed checklist for each component phase including design, code, unit-test and close-out
	- ![[Pasted image 20241023142434.png]]
### Status Reports
- Weekly Progress Summary
	- Highlight accomplishments and progress
	- Indicate delays in receivables
	- Describe pending items
	- Report estimated upcoming release delivery date
	- Describe current progress against development plan schedule
	- Communicate problems to stakeholders early on to facilitate timely action
- Issue Tracking
	- Track current progress using percent complete metric for each task
		- Compute sum of all earned completion points
	- Estimate delivery dates and forecast delivery slips early on
	- ![[Pasted image 20241023142712.png]]
	- ![[Pasted image 20241023142735.png]]
- Earned Value Management
	- Measuring planned work against actual work completed
	- Schedule variance
	- Cost variance
	- ![[Pasted image 20241023142806.png]]