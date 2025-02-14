WebGL Forward+ and Clustered Deferred Shading - Instructions
==========================================================

**This is due Sunday, October 29th at 11:59 PM.**

**Summary:**

In this project, you will implement the Forward+ and Clustered Deferred shading methods as discussed in class. You are given a scene with the Sponza Atrium model and a large number of point lights, as well as a GUI to toggle between the different rendering modes.

## Contents
- `src/`contains all the Javascript and GLSL code for this project. This contains several subdirectories:
  - `renderers/` defines the different renderers in which you will implement Forward+ and Clustered Deferred shading.
  - `shaders/` contains the GLSL files that are interpreted as shader programs at runtime.
- `models/` contains the Sponza Atrium model used in the test scene.
- `lib/` includes a minimal GLTF loader to load in the model.

## Running the code

Follow these steps to install and view the project.

- Clone this repository
- Download and install [Node.js](https://nodejs.org/en/)
- Run `npm install` in the root directory of this project. This will download and install dependences
- Run `npm start` and navigate to [http://localhost:5650](http://localhost:5650)

This project requires a WebGL-capable browser with support for several extensions. You can check that your browser supports them on [WebGL Report](http://webglreport.com/):
- OES_texture_float
- OES_texture_float_linear
- OES_element_index_uint
- EXT_frag_depth
- WEBGL_depth_texture
- WEBGL_draw_buffer

Google Chrome seems to work best on all platforms. If you have problems running the starter code, use Chrome or Chromium, and make sure you have updated your browser and video drivers.

`npm start` launches a script that continuously rebuilds your code when you make changes and serves it for access via your web browser.
On some Linux systems, you may see warnings like `Error: ENOSPC: System limit for number of file watchers reached`, which may prevent automatic rebuilding from working properly.
You may be able to resolve this by following the instructions here: https://code.visualstudio.com/docs/setup/linux#_visual-studio-code-is-unable-to-watch-for-file-changes-in-this-large-workspace-error-enospc

## Requirements
**Ask on Ed Discussion for any clarifications.**

In this project, you are given code for:
- Loading glTF models
- Camera control
- Simple forward renderer
- Partial implementation and setup for Forward+ and Deferred shading using 3D light clusters.
- Many helpful helpers

### Part 1: Implement the different rendering methods

Based on the discussions in lecture and recitation, you are expected to implement the Forward+ and Clustered Deferred rendering methods and analyze their results. Here is a summary of both methods:

**Forward+**
  - Build a data structure to keep track of how many lights are in each cluster and what their indices are
  - Render the scene using only the lights that overlap a given cluster

**Clustered Deferred**
  - Reuse clustering logic from Forward+
  - Store vertex attributes in g-buffer
  - Read g-buffer in a shader to produce final output

### Part 2: Effects and Optimizations

**Effects**: Choose one of the following effects to implement. (Or do multiple for extra credit!) 

- Implement deferred Blinn-Phong shading (diffuse + specular) for point lights
- OR
- Implement one of the following post-processing effects:
  - Bloom using post-process blur (box or Gaussian)
  - Toon shading (with ramp shading + simple depth-edge detection for outlines)

**Optimizations**: We ask that you optimize the g-buffers used for this renderer. In particular, aim to reduce the number of them used and/or their size. Some ideas to get you started:
- Pack values together into vec4s
- Use 2-component normals
- Quantize values by using smaller texture types instead of gl.FLOAT
- Reduce number of properties passed via g-buffer, e.g. by: reconstructing world space position using camera matrices and X/Y/depth

 For credit, you must show a good optimization effort and record the performance of each version you test, in a simple table. It is expected that you won't need all 4 provided g-buffers for a basic pipeline. Make sure you disable the unused ones.

## Performance & Analysis

**Before doing performance analysis**, you must disable debug mode by changing `DEBUG` to false in `src/init.js`. Keep it enabled when developing - it helps find WebGL errors *much* more easily.

Compare your implementations of Forward+ and Clustered Deferred shading and analyze their differences.
  - Is one of them faster?npm
  - Is one of them better at certain types of workloads?
  - What are the benefits and tradeoffs of using one over the other?
  - For any differences in performance, briefly explain what may be causing the difference.

Optimize your JavaScript and/or GLSL code. Chrome/Firefox's profiling tools (see Resources section) will be useful for this. For each change that improves performance, show the before and after render times.

For each new effect feature (required or extra), please provide the following analysis:
  - Concise overview write-up of the feature.
  - Performance change due to adding the feature.
  - If applicable, how do parameters (such as number of lights, etc.) affect performance? Show data with simple graphs.
    - Show timing in milliseconds, not FPS.
  - If you did something to accelerate the feature, what did you do and why?
  - How might this feature be optimized beyond your current implementation?

For each performance feature (required or extra), please provide:
  - Concise overview write-up of the feature.
  - Detailed performance improvement analysis of adding the feature
    - What is the best case scenario for your performance improvement? What is the worst? Explain briefly.
    - Are there tradeoffs to this performance feature? Explain briefly.
    - How do parameters (such as number of lights, tile size, etc.) affect performance? Show data with graphs.
      - Show timing in milliseconds, not FPS.
    - Show debug views when possible.
      - If the debug view correlates with performance, explain how.

## Starter Code Tour

Initialization happens in `src/init.js`. You don't need to worry about this; it is mostly initializing the gl context, debug modes, extensions, etc.

`src/main.js` is configuration for the renderers. It sets up the gui for switching renderers and initializes the scene and render loop. The most important things here are the arguments for `ForwardPlusRenderer` and `ClusteredRenderer`. These constructors take the number of x, y, and z slices to split the frustum into.

We have also provided a `Wireframe` helper class for debugging. It lets you draw arbitrary colored line segments in the scene, which may be helpful for visualizing things like frustum clusters. For example, on app startup you could populate the `Wireframe` with lines for the clusters based on the initial camera view, and then you can pan around the scene to inspect the clusters.

We've provided an example in `src/main.js` for drawing lines on top of everything in the scene, but you can also draw them so that they depth test properly with the rest of the scene. Feel free to modify this class as you see fit. **Before doing performance analysis**, be sure to disable wireframe drawing, especially if you have made significant modifications that may impact performance.

`src/scene.js` handles loading a .gltf scene and initializes the lights. Here, you can modify the number of lights, their positions, and how they move around. Also, take a look at the `draw` function. This handles binding the vertex attributes, which are hardcoded to `a_position`, `a_normal`, and `a_uv`, as well as the color and normal maps to targets `gl.TEXTURE0` and `gl.TEXTURE1`.

**Simple Forward Shading Pipeline**

There is a simple forward shading pipeline as an example for how everything works. Check out `src/forward.js`.

The constructor for the renderer initializes a `TextureBuffer` to store the lights. This isn't totally necessary for a forward renderer, but you'll need this to implement light clusters. What we're trying to do here is upload to a shader all the positions of our lights. However, we unfortunately can't upload arbitrary data to the GPU with WebGL so we have to pack it as a texture. This is set up for you.

The constructor for `TextureBuffer` takes two arguments, the number of elements, and the size of each element (in floats). It will allocate a floating point texture of dimension `numElements x ceil(elementSize / 4)`. This is because we pack every 4 adjacent values into a single pixel.

Go to the `render` function to see how this is used in practice. Here, the buffer for the texture storing the lights is populated with the light positions. Notice that the first four values get stored at locations: `this._lightTexture.bufferIndex(i, 0) + 0` to `this._lightTexture.bufferIndex(i, 0) + 3` and then the next three are at `this._lightTexture.bufferIndex(i, 1) + 0` to `this._lightTexture.bufferIndex(i, 0) + 2`. Keep in mind that the data is stored as a texture, so the 5th element is actually the 1st element of the pixel in the second row.

Look again at the constructor of `ForwardRenderer`. Also initialized here is the shader program. The shader program takes in a vertex source, a fragment source, and then a map of what uniform and vertex attributes should be extracted from the shader. In this code, the shader location for `u_viewProjectionMatrix` gets stored as `this._shaderProgram.u_viewProjectionMatrix`. If you look at `fsSource`, there's a strange thing happening there. `fsSource` is actually a function and it's being called with a configuration object containing the number of lights. What this is doing is creating a shader source string that is parameterized. We can't have dynamic loops in WebGL, but we can dynamically generate static shaders. If you take a look at `src/shaders/forward.frag.glsl.js`, you'll see that `${numLights}` is used throughout.

Now go look inside `src/shaders/forward.frag.glsl.js`. Here, there is a simple loop which loops over the lights and applies shading for each one. There is a helper `UnpackLight(index)` which unpacks the `index`th light from the texture into a struct. Make sure you fully understand how this is working because you will need to implement something similar for clusters. Inside `UnpackLight` there is another helper called `ExtractFloat(texture, textureWidth, textureHeight, index, component)`. This pulls out the `component`th component from the `index`th value packed inside a `textureWidth x textureHeight` texture. Again, this is meant to be an example implementation. Using this function to pull out four values into a `vec4` will be unecessarily slow.

**Getting Started**

Here's a few tips to get you started.

1. Complete `updateClusters` in `src/renderers/base.js`. This should update the cluster `TextureBuffer` with a mapping from cluster index to light count and light list (indices).

2. Update `src/shaders/forwardPlus.frag.glsl.js` to
  - Determine the cluster for a fragment
  - Read in the lights in that cluster from the populated data
  - Do shading for just those lights
  - You may find it necessary to bind additional uniforms in `src/renderers/forwardPlus.js`

3. Update `src/shaders/deferredToTexture.frag.glsl` to write desired data to the g-buffer
4. Update `src/deferred.frag.glsl` to read values from the g-buffer and perform simple forward rendering. (Right now it just outputs the screen xy coordinate)
5. Update it to use clustered shading. You should be able to reuse lots of stuff from Forward+ for this. You will also likely need to update shader inputs in `src/renderers/clusteredDeferred.js`

## README

Replace the contents of the README.md in a clear manner with the following:
- A brief description of the project and the specific features you implemented.
- At least one screenshot of your project running.
- A 30+ second video/gif of your project running showing all features. (Even though your demo can be seen online, using multiple render targets means it won't run on many computers. A video will work everywhere.)
- Performance analysis (described above)

**GitHub Pages**

Since this assignment is in WebGL, you can (and should) make your project easily viewable by taking advantage of GitHub's project pages feature.

Once you are done with the assignment, create a new branch:

```
git branch gh-pages
git checkout gh-pages
```

Run `npm run build` and commit the compiled files in `build/` on the `gh-pages` branch.

Push the branch to GitHub:

`git push origin gh-pages`

Now, you can go to `<user_name>.github.io/<project_name>` to see your renderer online from anywhere. Add this link to your README.

## Submit

Open a GitHub pull request so that we can see that you have finished. The title should be "Project 5B: YOUR NAME". The template of the comment section of your pull request is attached below, you can do some copy and paste:

- Repo Link
- (Briefly) Mentions features that you've completed. Especially those bells and whistles you want to highlight
  - Feature 0
  - Feature 1
  - ...
- Feedback on the project itself, if any.

### Third-Party Code Policy

- Use of any third-party code must be approved by asking on our mailing list.
- If it is approved, all students are welcome to use it. Generally, we approve use of third-party code that is not a core part of the project. For example, for the path tracer, we would approve using a third-party library for loading models, but would not approve copying and pasting a CUDA function for doing refraction.
- Third-party code **MUST** be credited in README.md.
- Using third-party code without its approval, including using another student's code, is an academic integrity violation, and will, at minimum, result in you receiving an F for the semester.
