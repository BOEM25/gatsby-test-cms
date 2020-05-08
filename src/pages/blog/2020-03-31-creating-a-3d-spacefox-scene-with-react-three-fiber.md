---
templateKey: blog-post
title: Test
date: 2020-03-31T01:02:20.866Z
featuredpost: false
published: true
author: Stephen Castle
authorimage: /img/contributors/stephen.jpg
featuredimage: /img/blog-images/reactfox.jpg
description: >-
  Build an interactive flight sim scene using react three fiber and import a 3d model in obj format.
tags:
  - programming
  - javascript
  - intermediate
  - gamedev
  - react
  - react-three-fiber
  - Three.js
  - libraries
  - tutorials
---

In this series we will build a fully interactive "Space Fox" game with a ship that can fire lasers, do barrell rolls, and fly through space. In doing so we will touch on many parts of using `react-three-fiber`. We will cover animation, camera controls, collision detection and more, but in part one let's start small and just get our ship geometry loaded into React. In this part we will build the below scene which will serve as a starting place for the rest of the series.

<iframe
     src="https://codesandbox.io/embed/stupefied-mestorf-5fvlo?fontsize=14&hidenavigation=1&theme=dark"
     style="width:100%; height:500px; border:0; border-radius: 4px; overflow:hidden;"
     title="React Fox : A React Three Fiber Scene Part One - Loading Geometry"
     allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media; usb"
     sandbox="allow-modals allow-forms allow-popups allow-scripts allow-same-origin"
   ></iframe>

## TEST

1. Create a new react project and install react-three-fiber and three.

```bash
npx create-react-app react-fox
npm i -S react-three-fiber three
```

2. Create a blank app with an empty Canvas component.

```js
import React from "react";
import { Canvas } from "react-three-fiber";
import "./styles.css";

export default function App() {
  return <Canvas style={{ background: "#171717" }}></Canvas>;
}
```

3. Replace styles.css contents to maximize the viewport container for our threeJS canvas.

```css
/*styles.css*/
* {
  box-sizing: border-box;
}

html,
body,
#root {
  width: 100%;
  height: 100%;
  margin: 0;
  padding: 0;
}
```

## Getting our Object File Ready for Loading

ThreeJS prefers to load geometry in the glTF format. Since my original ship model was in obj, I used this tool to convert it. https://blackthread.io/gltf-converter/ To save time here is a link to the model file in gltf. If you want to try using your own model though feel free to convert it using the tool and just use it instead of the one I am using.
[Model File](/models/arwing.glb)

## Loading an Object With the ThreeJS Loader

Once you have a model file ready let's load it into threeJS to use in a mesh. We will use a hook from react-three-fiber called `useLoader` to load the mesh from our file generated in the previous step. We can create a new component to do this in called ArWing. This will be the component that represents our ship.

This hook will return a promise while it resolves which means we need to use the Suspense component from React to provide fallback UI. We can just make a placeholder sphere for now to display while loading.

```js
import Reactfrom {Suspense} "react";
import { Canvas, useLoader } from "react-three-fiber";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import "./styles.css";

function Loading() {
  return (
    <mesh visible position={[0, 0, 0]} rotation={[0, 0, 0]}>
      <sphereGeometry attach="geometry" args={[1, 16, 16]} />
      <meshStandardMaterial
        attach="material"
        color="white"
        transparent
        opacity={0.6}
        roughness={1}
        metalness={0}
      />
    </mesh>
  );
}

function ArWing() {
  const { nodes } = useLoader(GLTFLoader, "models/arwing.glb");
  return (
    <group>
      <mesh visible geometry={nodes.Default.geometry}>
        <meshStandardMaterial
          attach="material"
          color="white"
          roughness={0.3}
          metalness={0.3}
        />
      </mesh>
    </group>
  );
}

export default function App() {
  return (
    <Canvas style={{ background: "#171717" }}>
      <directionalLight intensity={0.5} />
      <Suspense fallback={<Loading />}>
        <ArWing />
      </Suspense>
    </Canvas>
  );
}

```

If you are wondering where the `nodes` value being destructured from useLoader come from, try console logging the entire return value of useLoader without destructuring.

```
  const scene = useLoader(GLTFLoader, "models/arwing.glb");
  console.log(scene);
```

The object returned actually contains a lot of information. This is because the glb file can contain more than just geometry, it can also contain animation, textures, and all kinds of other advanced information.
In our case for now we are just using the geometry of the Default object in the scene. Depending on our original glb file though there could have been multiple objects and the structure could have been different.

## Adding Animation

We can add animation to our model with another hook provided by react-three-fiber called useFrame. This will get called on every frameUpdate and run the callback function provided. We will need a ref to our geometry so we can update it directly from within the callback. We can also add a group around our mesh in case we end up wanting to add more objects to the animation.

```js
import React, { Suspense, useRef } from "react"; // highlight-line
import { Canvas, useLoader, useFrame } from "react-three-fiber";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import "./styles.css";

function Loading() {
  return (
    <mesh visible position={[0, 0, 0]} rotation={[0, 0, 0]}>
      <sphereGeometry attach="geometry" args={[1, 16, 16]} />
      <meshStandardMaterial
        attach="material"
        color="white"
        transparent
        opacity={0.6}
        roughness={1}
        metalness={0}
      />
    </mesh>
  );
}

function ArWing() {
  const group = useRef();
  const { nodes } = useLoader(GLTFLoader, "models/arwing.glb");
  // highlight-start
  // useFrame will run outside of react in animation frames to optimize updates.
  useFrame(() => {
    group.current.rotation.y += 0.004;
  });
  // highlight-end
  return (
    // Add a ref to the group. This gives us a hook to manipulate the properties of this geometry in the useFrame callback.
    <group ref={group}>
      <mesh visible geometry={nodes.Default.geometry}>
        <meshStandardMaterial
          attach="material"
          color="white"
          roughness={0.3}
          metalness={0.3}
        />
      </mesh>
    </group>
  );
}

export default function App() {
  return (
    <Canvas style={{ background: "#171717" }}>
      <directionalLight intensity={0.5} />
      <Suspense fallback={<Loading />}>
        <ArWing />
      </Suspense>
    </Canvas>
  );
}
```

For now we are just slowly increasing the y rotation. The rotation units are in Radians so if you remember your HS geometry it will rotate forever just by continually increasing the value. Try manipulating other properties inside of the useFrame animation. Try rotating on other axis by changing between x,y and z. Or try other translations besides rotation. There are also `scale` and `position`, just replace `rotation` to try those out. They also work on all 3 axis in the same way. Though you will get some pretty bizarre results if you just increase the y scale forever and your ship may leave you behind if you change the position for too long.

## Next Part

In part two we will add camera and movement controls for our ship to replace the rotation animation we have now.. Subscribe to the mailing list for notifications when new articles are published.
