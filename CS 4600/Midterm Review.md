- What is gamma correction?
	- It is easier to tell the difference between darker colors than brighter colors
	- $c_{out} = c_{in}^\gamma, \gamma=2.2$ to nonlinear correct for this, and give more resolution to the darker colors
- Use alpha blending to composite the colors A and B below, assuming A is place on top of B. Show your work: A=(160, 80, 40, 128), B=(40, 80, 160, 128) in RGBA format
	- Hint: 128/255 approx 0.5
	- $A_{rgb}^{pre}=A_{rgb}A_{\alpha}=(80,40,20)$
	- $B_{rgb}^{pre}=B_{rgb}B_{\alpha}=$
	- $C_{rgb}^{pre}=A_{rgb}A_{\alpha}+B^{pre}_{rgb}(1-A_{\alpha}=(90,60,60)$
	- $C_\alpha=A_\alpha+B_\alpha(1-A_\alpha) \approx 3/4$
	- Premultiplied alpha: $C^{pre}=(90,60,60,192)$
	- Straight alpha: $C=(120,80,80,192)$
- Using homogeneous coordinates in 2D, construct rotation matrix of 60 degrees (counterclockwise), a 2x uniform scale matrix, and a translation matrix by $\begin{bmatrix}4&3\end{bmatrix}^T$. Combine them into a single matrix (scale, then rotation, then translation)
	- Rotation Matrix: R=$\begin{bmatrix}1/2&-\sqrt3/2&0\\\sqrt3/2&1/2&0\\0&0&1\end{bmatrix}, \begin{bmatrix}\cos(\theta)&-\sin(\theta)&0\\\sin(\theta)&\cos(\theta)&0\\0&0&1\end{bmatrix}$
	- Scale Matrix: S=$\begin{bmatrix}2&0&0\\0&2&0\\0&0&1\end{bmatrix}$
	- Translation Matrix: T=$\begin{bmatrix}1&0&4\\0&1&3\\0&0&1\end{bmatrix}$
	- A=TRS = 
- What does the subdivision operation do to a polygonal mesh?
	- Increase mesh resolution
	- By splitting elements into smaller elements (tessellation)
	- Smoothing as well
	- Most common is Catmull-Clark
- True/False
	- Any given series of 3D rotations around different axes can be represented as a single rotation around a single axis
		- True
	- Perspective transformation does not change the camera space z-coordinate values
		- False
	- A piecewise Catmull-Rom curve interpolates all of its control points (first and last points repeated)
		- True
	- Any given cubic B-Spline curve can be represented using a cubic Bezier curve
		- True
	- NURBS surfaces interpolate their control points
		- False
- Stages of the GPU rendering pipeline
	- Vertex shader, Rasterizer, Fragment shader
- What is the language used?
	- GLSL
- Where can it be used?
	- Vertex shader (this example specifically) and fragment shader
- What does it do?
	- Transforms a vertex
- The code segment below for drawing a single triangle has a bug. It draws a triangle, but the result does not match the intended 3D positions for its vertices. What is the bug and how can you fix it?
	- Wrong dimension of vector
- Find the position of the Bezier curve at t=1/2 using de Casteljau's algorithm
- Answer the following about a point on the same plane as a triangle with barycentric coordinates (0.2, 0.5, c).
	- What is the value of c?
		- c = 0.3, they must add to 1, because it must be a linear combination
	- Is the point inside the triangle? Why or why not?
		- Inside, all values between 0 and 1
	- If the triangle has vertex positions $\begin{bmatrix}1&2&1\end{bmatrix}^T,\begin{bmatrix}2&1&0\end{bmatrix}^T,\begin{bmatrix}0&0&0\end{bmatrix}^T$, compute the point
		- $0.2*\begin{bmatrix}1&2&1\end{bmatrix}^T+0.5*\begin{bmatrix}2&1&0\end{bmatrix}^T+0.3*\begin{bmatrix}0&0&0\end{bmatrix}^T=$
### REVIEW 2
- Explain the difference between HDR and LDR images
	- LDR must be between 0-1
	- HDR can be greater than 1
- Use alpha blending to composite! (See pictures)
- Matrix question! (See pictures)
- 3 Stages of GPU pipeline in their order. Which ones are programmable?
	- Vertex Shader (programmable), Rasterizer, Fragment Shader (programmable)
- TRUE/FALSE
	- The PNG image format offers lossless compression
		- True
	- The order in which a set of rotations is applied does NOT matter
		- False
	- The order in which a set of translations is applied does NOT matter
		- True
	- A cubic Bezier curve interpolates just the first and last control points
		- True
	- A B-spline interpolates all of its control points
		- False
- Consider a parametric curve formulation that computes the position of the curve C(t) at any parameter value t as a weighted sum of a set of control points p_i, such that $\sum^N_iw_ip_i$, where w_i is the weight of the control point p_i. Should the weights form a partition of unity, meaning the sum of all weights add to 1? Explain why?
	- Yes, if they don't then the curve is not affine invariant
- Convert code to draw a triangle strip:
```JS
var positions = [-1, -1, 0,
				  1, -1, 0,
				  -1, 1, 0,
				  1,  1, 0,
				  -1, 1, 0,
				  1, -1, 0];

var position_buffer =- gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, position_buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
var p = gl.getAttribLocation(prog, "pos");
gl.vertexAttribPointer(p, 3, gl.FLOAT, flase, 0, 0);
gl.enableVertexAttribArray(p);
gl.drawArrays(gl.TRIANGLES, 0, 6);
```
	- into
```JS
var positions = [-1, -1, 0,
				  1, -1, 0,
				  -1, 1, 0,
				  1,  1, 0;

var position_buffer =- gl.createBuffer();
gl.bindBuffer(gl.ARRAY_BUFFER, position_buffer);
gl.bufferData(gl.ARRAY_BUFFER, new Float32Array(positions), gl.STATIC_DRAW);
var p = gl.getAttribLocation(prog, "pos");
gl.vertexAttribPointer(p, 3, gl.FLOAT, flase, 0, 0);
gl.enableVertexAttribArray(p);
gl.drawArrays(gl.TRIANGLE_STRIP, 0, 4);
```