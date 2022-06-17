# React Three Fiber

# General Info

## Object constructor

```jsx
<geometry args={[2, 2, 2,]} />
```

## **Deeply-nested properties and methods**

To access deeply-nested properties we can use the JSX piercing:

```jsx
<object3D rotation-order="YZX" />
```

To access methods we use Ref:

```jsx
const objectRef = React.useRef()

React.useEffect(() => {
  objectRef.current.rotation.reorder(...)
}, [])

<object3D ref={objectRef} />
```

# 1. Transform Objects

## Position

```jsx
<object3D position={[x, y, z]} />
```

## Scaling

```jsx
<object3D scale={5} />

//scale={[x, y, z]}
<object3D scale={[1, 2, 3]} />

//alternatively we could use scale-y, scale-z
<object3D scale-x={5} />
```

## Rotation

### Rotating Object

Rotation values must be specified in radians.

```jsx
//rotation={[x, y, z]}    Rotate X by 90°, Y by 45°, Z by 0
<object3D rotation={[Math.PI * 0.5, Math.PI * 0.25, Math.PI * 0.25]} />

//We can also rotate a single axes
<mesh rotation-x={Math.PI * 0.25}>
```

### Rotation Reorder:

```jsx
<object3D rotation-order="YZX" />
```

## Group

We can adjust the properties of all the children by adjusting it on the group

```jsx
<group scale={2}> //Scales all the children
<object3D />
<object3D />
</group>
```

# 2. Animation

Rotating and moving a Box along the x-axis

This is not the correct way to do it because every monitor has a different framerate 

```jsx
function Box() {
  const ref = useRef();
  useFrame(() => { //This will rerun with every frame
    ref.current.rotation.x += Math.PI * 0.005; //Rotating around the x-axis
    ref.current.position.x += 0.005; //Moving along the x-axis
  });
  return (
//We return our 3D object and attach the ref to it
    <mesh ref={ref}>
      <boxBufferGeometry />
      <meshBasicMaterial color="purple" />
    </mesh>
  );
}
```

The correct way to do it

```jsx
function Box() {
  const ref = useRef();
  useFrame(() => { //This will rerun with every frame

	const currentTime = Date.now();
  const delta = currentTime - time; //How much time have passed since the last frame
  time = currentTime; //Update the time

//ref.current.rotation.x += Math.PI * 0.005;
//ref.current.position.x += 0.005;
  ref.current.rotation.x += Math.PI * 0.001 * delta; 
  ref.current.position.x += 0.001 * delta; 

 });
  
return (
//We return our 3D object and attach the ref to it
    <mesh ref={ref}>
      <boxBufferGeometry />
      <meshBasicMaterial color="purple" />
    </mesh>
  );
}
    
```

***Explanation**: If the time passed since the last frame is short we multiply by a small number, and if the time passed was long we multiply by a larger number this way, no matter how fast the frame changes the animation will have the same speed and adjust accordingly.*

**Example:** 

Two different monitors with the same animation for 2 seconds:

`**A monitor with a low framerate (frame/2Seconds):**`

*`timeAtPreviousframe* = 5
*CurrentTime* = 7`

`delta = 7-5 = 2`

`objectPosition = 5`

`objectPosition + (0.5 * delta)`

`= 6`

`**A monitor with a high framerate (frame/1Second):**`

![Untitled](React%20Three%20Fiber%20a0d46b7499044cfb9f2b399210e082d7/Untitled.png)

![Untitled](React%20Three%20Fiber%20a0d46b7499044cfb9f2b399210e082d7/Untitled%201.png)

Another solution:

```jsx
function Box() {
  const ref = useRef();

  useFrame(({ clock }) => {//Clock time start at 0 the moment we intialized it
    const elapsedTime = clock.elapsedTime;
		
//One rotation every second
    ref.current.rotation.x = 2 * Math.PI * elapsedTime; 

//Osiliating along the x-axis [-3, 3]
    ref.current.position.x = 3 * Math.sin(elapsedTime);  
//Osiliating along the y-axis [-3, 3]
		ref.current.position.y = 3 * Math.cos(elapsedTime);

//This will create a circular motion!
});

  return (
    <mesh ref={ref}>
      <boxBufferGeometry />
      <meshBasicMaterial color="purple" />
    </mesh>
  );
}
```

# 3. Camera

### Camera Position

```jsx
<Canvas camera={{ position: [3, 3, 3] }}></Canvas>
```

### Camera Look At

Note that Look at might be overridden by some elements

```jsx
//We create a component to append it to the Canvas component
function Look() {
  const camera = useThree((s) => s.camera);
  useEffect(() => {
    camera.lookAt(4, 0, 0); //We can use a ref and get an object position here
  }, []);

  return null;
}
function App() {
  return (
    <div className="App">
      <Canvas>
        <Look />
      </Canvas>
    </div>
  );
}

```

## Perspective Camera

We use the *makeDefault* property to make Perspective Camera our default camera 

```jsx
<Canvas>
//You should put the properties here and not in the Canvas like before
<PerspectiveCamera
          makeDefault
          position={[2, 2, 2]}
          fov={140}
          near={1}   //near render distance
          far={1000} //render distance
        />
</Canvas>
```

### Animating the camera

Call need to call *updateProjectionMatrix()* every time we update the Camera

```jsx
function Box() {
  const ref = useRef();
  const camera = useThree((s) => s.camera);

  useFrame(({ clock }) => {
    const elapsedTime = clock.elapsedTime;
    camera.lookAt(ref.current.position);

//Our camera will Oscillate between 1-140
    camera.fov = 140 * Math.abs(Math.cos(0.1 * elapsedTime));

//We need to call updateProjectionMatrix() everytime we update the camera
    camera.updateProjectionMatrix();
  });
  return (
    <mesh ref={ref}>
      <boxBufferGeometry args={[2, 2, 2]} />
      <meshBasicMaterial color="purple" />
    </mesh>
  );
}
```

## Controls

We can implement our controls by appending it to the Canvas

```jsx
<Canvas>
<OrbitControls/>
</Canvas>
```

## Window position vs Three.js position

In Three.js going upward is considered as the positive-y axis while on the window going upward is the negative axes.

This is important to note when doing events related to things outside the Canvas such as the cursor movements, div’s position etc..