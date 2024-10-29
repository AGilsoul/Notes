### Single processor simulations
- sus
- ```sus input.ups```
- ```input.ups```is an XML input file
- example input files in ```src/StandAlone/inputs/UintahRelease```
### Multi processor simulations
- ```mpirun -np num_processors sus -mpi input.ups```
### Uintah problem Specification (UPS)
- XML input files
- Specification found in ```src/StandAlone/inputs/UPS_SPEC/ups_spex.xml```
- Essential tags:
```XML
<Uintah_specification>

<SimulationComponent>

<Time>

<DataArchive>

<Grid>
```
### Simulation Components
- SimulationComponent has type attribute that must be specified as mpm, mpmice, ice, arches, or mpmarches:
```<SimulationComponent type = "mpm" />``` 
### Time Related Variables
- Example
```XML
<Time>
	<maxTime> 1.0 </maxTime> <!-- How long to run simulation for REQUIRED --> 
	<initTime> 0.0 </initTime> <!-- Simulation start time REQUIRED -->
	<delt\_min> 0.0 </delt\_min> <!-- Smallest time step in simulation REQUIRED -->
	<delt\_max> 1.0 </delt\_max> <!-- Largest time step in simulation REQUIRED -->
	<delt\_init> 1.0e-9 </delt\_init> <!-- Initial time step -->
	<max\_delt\_increase> 2.0 </max\_delt\_increase> <!-- Maximum amount to multiply previous delt by -->
	<timestep\_multiplier> 1.0 </timestep\_multiplier> <!-- Multiply time step by this number REQUIRED -->
	<max\_Timestep> 100 <max\_Timestep> <!-- Max number of time steps to use (not usually used with max_iterations) -->
	<end\_on\_max\_time\_exactly> true </end\_on\_max\_time\_exactly> <!--clamp delt s.t. last timesteps end on the specified maxTime (default is false)  -->
</Time>
```
### Data Archiver
- ```<filebase>``` tag specifies directory name
- Data saving frequency
- Time intervals:
```<outputTimestepInterval> interger_number_of_steps </outputTimeStepInterval>```
- Timestep intervals
```<outputInterval> floating_point_time_increment </outputInterval>```
- Particle data denoted by p, node data by g
```XML
<save label = "p.mass" />
<save label = "g.mass" />
```
- To list variables for saving in a given component, use the following command in the StandAlone directory
```inputs/labelNames component```
- Replace component with mpmice, etc.
- To checkpoint with two timesteps worth of data every 0.1 seconds of simulation time: 
```<checkpoint cycle = "2" interval = "0.01"/>```
- To restart from checkpointed archive:
	- -t specifies timestep to start from, last checkpoint timestep is default
```./sus -restart disks.uda 000 -nocopy```
```./sus -restart disks.uda 000 -t 29```
### Geometry Objects
- Intersection of box and two spheres shape example:
```XML
<geom_object>
	<intersection>
		<box label = "Domain">
			<min>[0.0, 0.0, 0.0]</min>
			<max>[0.1, 0.1, 0.1]</max>
		</box>
		<union>
			<sphere label = "First node">
				<origin>[0.022, 0.028, 0.1]</origin>
				<radius> 0.01 </radius>
			</sphere>
			<sphere label = "2nd node">
				<origin>[0.030, 0.075, 0.1]</origin>
				<radius> 0.1 </radius>
			</sphere>
		</union>
	</intersection>
	<res>[2, 2, 2]</res>
	<velocity>[0., 0., 0.]</velocity>
	<temperature> 0 </temperature>
</geom_object>
```
- Other objects
	- box
		- min and max vector quantities [a, b, c] format
	- sphere
		- origin vector and radius float
	- cone
		- top and bottom origin vectors and top and bottom radius float
	- cylinder
		- top and bottom origin vectors and radius float
	- ellipsoid
		- origin vector and
			- If axes aligned with cartesian grid, floating point values rx, ry, rz
			- If any other orientation, three vector quantities in [a, b, c] format, v1, v2, v3. Must be orthogonal within 1e-12 after dot product
	- parallelpiped
		- four points specified
```
//********************************************
//                                          //
//                                          //
//        *------------------*              //
//       / \                / \             // 
//     P3...\..............*   \            //
//       \   \             .    \           //
//       (z)  P2-----------------*          //
//         \  /             .   /           //
//          \/               . /            //
//          P1-------(x)-----P4             //                                     //                                          //                  
//
// Returns true if the point is inside (or on) the parallelepiped.
```
- tri is a tag for triangulated surface
	- contains list of integers describing connectivity of points in file_name.pts
	```
	Triangulated file
	1 39 41
	1 41 38
	38 41 41
	.
	Points file
	0 0.03863 -0.005
	0.35227 0.13023 -0.005
	0.00403479 0.0296797 -0.005
	...
	```
	- look at Mach 2 wedge example in section 11.5 for use of this
### Boundary Conditions
- Specified within ```<Grid>```
- Can be Dirichlet (value) or Neumann (derivative)
```XML
<Grid>
	<BoundaryConditions>
		<Face side = "x-"> <!-- BC for entire x- side -->
			<!-- all means all materials will have this value -->
			<!-- To specify BC for specific material, use integer number instead of all -->
			<!-- label specifies variable being assigned -->
			<BCType id = "all" var = "Dirichlet" label = "Velocity">
				<value> [0.0, 0.0, 0.0] </value>
			</BCType>
		</Face>
		<Face side = "x+">
			<BCType id = "all" var = "Dirichlet" label = "Veloicty">
				<value> [0.0, 0.0, 0.0] </value>
			</BCType>
		</Face>
		...
	</BoundaryConditions>
</Grid>
```
- Complicated BC example:
	- Hot jet of fluid into domain, described by circle on one side of the domain with BC that are different in the circular jet compared to the rest of the side
```XML
<Face circle = "y-" origin = "0.0 0.0 0.0" radius = ".5">
	<BCType id = "0" label = "Pressure" var = "Neumann">
		<value> 0.0 </value>
	</BCType>
	<BCType id = "0" label = "Velocity" var = "Dirchlet">
		<value> [0., 1., 0.] </value>
	</BCType>
	<BCType id = "0" label = "Temperature" var = "Dirichlet">
		<value> 1000.0 </value>
	</BCType>
	<BCType id = "0" label = "Density" var = "Dirichlet">
		<value> .35379 </value>
	</BCType>
	<BCType id = "0" label = "SpecificVol" var = "computeFromDensity">
		<value> 0.0> </value>
	</BCType>
</Face>
<Face side = "y-">
	<BCType id = "0" label = "Pressure" var = "Neumann">
		<value> 0.0 </value>
	</BCType>
	<BCType id = "0" label = "Velocity" var = "Dirchlet">
		<value> [0., 0., 0.] </value>
	</BCType>
	<BCType id = "0" label = "Temperature" var = "Neumann">
		<value> 0.0 </value>
	</BCType>
	<BCType id = "0" label = "Density" var = "Neumann">
		<value> 0.0 </value>
	</BCType>
	<BCType id = "0" label = "SpecificVol" var = "computeFromDensity">
		<value> 0.0> </value>
	</BCType>
```
### Grid Specification
- Split into multiple patches
- Specified along x,y,z directions of grid with 
	- ```<patches> [2, 5, 3] </patches>```
- Single processor problems use one patch for whole domain
- Can't use more processors than patches
- Can also specify grid spacing with number of grid cells in x,y,z direction with 
	- ```<resolution> [20, 20, 3] </resolution>```
- To specify grid cell size
	- ```<spacing> [0.5, 0.5, 0.3] </spacing>```
- Example
```XML
<Level>
	<Box label="1">
		<lower> [0, 0, 0] </lower>
		<upper> [5, 5, 5] </upper>
		<extraCells> [1, 1, 1] </extraCells>
		<patches> [1, 1, 1] </patches>
	</Box>
	<spacing> [0.5, 0.5, 0.5] </spacing>
	<Box label="1">
		<lower> [0, 0, 0] </lower>
		<upper> [5, 5, 5] </upper>
		<resolution> [10, 10, 10] </resolution>
		<extraCells> [1, 1, 1] </extraCells>
		<patches> [1, 1, 1] </patches>
	</Box>
</Level>
```
### Schedulers
- Determines when tasks are executed
- MPI Scheduler: Static task ordering, deterministic execution with MPI. One MPI rank per CPU core
- Dynamic MPI Scheduler: Dynamic scheduling, non-deterministic, out-of-order execution of tasks at runtime. One MPI rank per CPU core
- Unified Scheduler: Multi-threaded scheduler, MPI + Pthreads, support for GPU tasks (MPI + Pthreads + NVIDIA CUDA), non-deterministic, out-of-order execution of tasks at runtime.
- Example
```XML
<Scheduler type="DynamicMPI">
	<small_messages> true </small_messages>
	<taskReadyQueueAlg> MostMessages </taskReadyQueueAlg>
	<VarTracker>
		<start_time> 0.0 </start_time>
		<end_time> 0.2 </end_time>
		<start_index> [0, 0, 0] </start_index>
		<end_index> [12, 12, 12] </end_index>
		<var label="divQ" dw="OldDW"/>
		<locations before_comm="false" before_exec="true" after_exec="true"/>
		<task name="Ray::rayTrace"/>
	</VarTracker>
</Scheduler>
```



