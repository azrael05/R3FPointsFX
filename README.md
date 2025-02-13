# R3f Points FX

[r3f-points-fx - npm ](https://www.npmjs.com/package/r3f-points-fx)

The `R3FPointsFX` component is a customizable particle system for 3D graphics built using the three.js library and integrated with React via the `@react-three/fiber` package. It allows you to create visually appealing particle effects in your 3D scenes.

Just pass the array of meshes (you can load meshes from` gltf/glb` files) and see the particles arrange in its shape. Transitions can be created from one model to another.

## Table of Contents

- [Features](#features)
- [Requirements](#requirements)
- [Installation](#installation)
- [Examples](#examples)
- [Usage](#usage)
- [Props](#props)
- [Customization](#customization)
- [References](#references)

## Features

- Create a particle system with customizable properties.
- Use shaders to control particle behavior and appearance.
- Incorporate data textures for particle positions.
- Exposes methods to manipulate particle animations dynamically.

## Requirements

- `@react-three/fiber`
- `@react-three/drei`
- `three`
- Your react three fiber scene 🤩

## Installation

To use the `R3FPointsFX` component in your React three fiber application, you need to follow these installation steps:

1. Make sure you have Node.js and npm (Node Package Manager) installed on your machine.

2. Install the packages:

```bash
npm i r3f-points-fx
```

## Examples

[R3FPointsFX-basic - CodeSandbox](https://codesandbox.io/p/sandbox/r3fpointsfx-basic-qsq3jl)

## Usage

Basic:

```jsx
import { useState, useRef } from 'react'
import { Canvas } from '@react-three/fiber'
import { R3FPointsFX } from 'r3f-points-fx'

function App() {
  const [modelA, setModelA] = useState(null)
  const [modelB, setModelB] = useState(null)
  const pointsRef = useRef()

  //change the modelA and modelB to the respective indexes in the
  //array which is being passed as modelsArray
  return (
    <Canvas>
      {/* Other 3D components */}
      <R3FPointsFX
        modelsArray={/* Provide an array of models */}
        ref={pointsRef}
        modelA={modelA}
        modelB={modelB}
        // Customize other props as needed

      />
    </Canvas>
  );
}

export default App

```

For typescript:

```tsx
import { useState, useRef } from "react"
import { Canvas } from "@react-three/fiber"
import { R3FPointsFX, R3FPointsFXRefType } from "r3f-points-fx"

function App() {
  const [modelA, setModelA] = useState<number | null>(null)
  const [modelB, setModelB] = useState<number | null>(null)
  const pointsRef = useRef<R3fPointsFXRef>(null)

  return (
    <Canvas>
      {/* Other 3D components */}
      <R3FPointsFX
        modelsArray={/* Provide an array of models */}
        // Customize other props as needed
        ref={pointsRef}
        modelA={modelA}
        modelB={modelB}
      />
    </Canvas>
  )
}

export default App
```

#### Updating progress:

Always update the progress using `pointsRef`.

> [!Warning]
> Do not use state variables for progress as it will cause serious performance issues.

The usage of `pointsRef.udateProgress` is demonstrated in the sandbox 😊

#### Ref methods

| Method              | Usage                                               | Return                                                                                                  |
| ------------------- | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------- |
| `getSimulationMesh` | `const mesh = pointsRef.curret.getSimulationMesh()` | the ref to the simulation mesh                                                                          |
| `getPointsMesh`     | `const points = pointsRef.current.getPointsMesh()`  | the ref to points mesh (this can be used to modify the position, rotation, scaling of the points mesh ) |
| `updateProgress`    | `pointsRef.current.updateProgress(progress:number)` | the ref to the simulation mesh                                                                          |

## Props

The `R3FPointsFX` component accepts the following props:

- `modelsArray`: An array of 3D models used for generating particles.
- `pointsCount`: The number of particles in the system **(default = 128)**.
- `pointsVertFunctions`: Custom vertex shader functions.
- `pointsFragFunctions`: Custom fragment shader functions.
- `modelA`: Index of the first model in `modelsArray`.
- `modelB`: Index of the second model in `modelsArray`.
- `uniforms`: Custom uniforms for shaders.
- `baseColor`: The base color for the particles.
- `pointSize`: The size of individual particles **(default = 1.0)**.
- `alpha`: The alpha (transparency) value for particles.
- `attributes`: Additional attributes for the particle system.
- `blending`: Blending mode for particles **(default = `THREE.AdditiveBlending`)**.
- Attributes of `<points>...</points>` (in react three fiber) such as position, rotation, scale can also be passed to control the respective parameters of the mesh.

## Customization

You can pass your own uniforms and provided shader functions to give a more customized look to your particles 🤩. Use the provided shader function templates to control the color, size, shape, position of the particles.

> [!WARNING]  
> Do not modify the names or input variables of the function, you are supposed to use the provided inputs to complete the functions as per your needs.

Finally pass these templates (with whatever changes you have done) as props to the component. If your are not using any function just let it remain same as the template, **changing the name or input params** of these function will cause shader to crash.

#### GLSL Function templates:

##### Fragment Functions

```c
//Uniforms which you have passed must be declared here
//For example:
//uniform float uColor1;


vec3 model_color(int model_index, vec3 position, float time){
  //model index is provided to color a specific mesh in the described way
  //color the model with index 0 with uColor1
  //For example:
  //if(model_index == 0){
  //   return uColor1;
  //}
  vec3 color = uColor;
  return color;
}

float alpha_shape(vec2 point_cord, float alpha){
  //particle center at (0.5, 0.5)
  //by default circular particles
  //for square particles remove the below code and just return alpha
  float dist = distance(point_cord, vec2(0.5));
  float final = dist > 0.5 ? 0.0 : alpha;
  return final;
}
```

##### Vertex Functions

```c
//Uniforms or attributes which you have passed must be declared here
//For example:
//attribute vec3 aRandom;

vec3 position_calc(vec3 pos, float time){
  return pos;
}

float point_size(float distance_from_camera, float time){
  float size = uPointSize;
  return size;
}
```

Example:

```tsx
function MyComponent() {
  //your other code
  const fragmentFunctions = `
	uniform vec3 uColor1;
	uniform vec3 uColor2;
	uniform vec3 uColor3;

	vec3 angleMix(vec3 position){
		float pi = asin(1.0) * 2.0;
		float angle = atan(position.z, position.x) + pi;
		float total = pi * 2.0;
		float scale = angle/total;
		float division = fract((scale + 1.0/6.0) * 3.0);
		division *= 6.0;
		division = clamp(division, 0.0, 1.0);
		vec3 color;
		if(angle <= total/6.0){
			color = mix(uColor3, uColor1, division);
		}else if(angle <= total/2.0){
			color = mix(uColor1, uColor2, division);
		}else if(angle <= (5.0*total)/6.0){
			color = mix(uColor2, uColor3, division);
		}else{
			color = mix(uColor3, uColor1, division);
		}
		return color;
	}

	vec3 model_color(int model_index, vec3 position, float time){
	  if(model_index == 1){
		  return angleMix(position);
	  }
	  vec3 color = uColor;
	  return color;
	}

	float alpha_shape(vec2 point_cord, float alpha){
	  //particle center at (0.5, 0.5)
	  //by default circular particles 
	  //for square particles remove the below code and just return alpha
	  float dist = distance(point_cord, vec2(0.5));
	  float final = dist > 0.5 ? 0.0 : alpha;
	  return final;
	}
  `
  const vertexFunctions = `
	attribute vec3 aRandom;
	vec3 position_calc(vec3 pos, float time){
	  //add some random motion to particles
	  pos.x += sin(time * aRandom.x) * 0.01;
      pos.y += cos(time * aRandom.y) * 0.01;
      pos.z += sin(time * aRandom.z) * 0.01;
	  return pos;
	}

	float point_size(float distance_from_camera, float time){
	  //Make particles closer to camera more bigger
      float size = uPointSize;
      float adjustedPointSize = 40.0 * pow(2.0, -distanceToCamera) * size*;
      return adjustedPointSize;
    }
  `
  const generateRandomnArray = (size: number) => {
    const length = size * size * 3
    const data = new Float32Array(length)

    for (let i = 0; i < length; i++) {
      const stride = i * 3

      data[stride] = Math.random() * 3 - 1
      data[stride + 1] = Math.random() * 3 - 1
      data[stride + 2] = Math.random() * 3 - 1
    }
    return data
  }

  const randomArray = useMemo(() => {
    return generateRandomnArray(128)
  }, [])

  return (
    <>
      <R3FPointsFX
        modelsArray={/* Provide an array of models */}
        // Customize other props as needed
        ref={pointsRef}
        modelA={modelA}
        modelB={modelB}
        uniforms={{
          uColor1: new THREE.Color("#D0BFFF"),
          uColor2: new THREE.Color("#DAFFFB"),
          uColor3: new THREE.Color("#FF6AC2"),
        }}
        attributes={[
          {
            name: "aRandom",
            array: randomArray,
            itemSize: 3,
          },
        ]}
        pointsVertFunctions={vertexFunctions}
        pointsFragFunctions={fragmentFunctions}
      />
    </>
  )
}
```

## References

[https://blog.maximeheckel.com/posts/the-magical-world-of-particles-with-react-three-fiber-and-shaders/](https://blog.maximeheckel.com/posts/the-magical-world-of-particles-with-react-three-fiber-and-shaders/)

[FBO particles – Youpi ! (barradeau.com)](https://barradeau.com/blog/?p=621)
