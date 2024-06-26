# react-three-shader-passes (DRAFT)

Note: `react-three-shader-passes` (`R3SP`) is currently still in pre-release stages.

This library is designed for Three.js/React apps built on [react-three-fiber](https://docs.pmnd.rs/react-three-fiber/getting-started/introduction) (R3F). Check out and support the ecosystem and [@pmndrs](https://docs.pmnd.rs).

&rarr; Easily chain shader passes, using the rendered texture from one scene as a `uniform` type input to any other shader pass

&rarr; Ideal for raw physics simulations and for effects, post processing, optimization, and complex shader-based logic

&rarr; Use output textures of any scene or shader as an FBO input to another without cumbersome, hierarchical JSX code or complex `ref`-based code design

&rarr; Use chaining capabilities as a GPGPU solution that's fully hooked into your React app, or as a simple framework for modular shader logic

&rarr; Fully declarative API and implementation for a turnkey R3F integration

&rarr; Exports additional utilities and helpers for even more R3F syntactic sugar

&rarr; Fully managed objects/components implemented on top of R3F ecosystem; you have full control of all objects but do not have to worry about memory if you don't need to

## Usage

Use with `react-three-fiber` to create chained, modular shader passes.

```bash
npm i [TODO]
```

`ShaderPass` is just a wrapper of [`RenderTexture`](https://github.com/pmndrs/drei/?tab=readme-ov-file#rendertexture) under the hood. You place instances of it in a `ShaderPassesTexture` to easily use the output texture of any other `ShaderPass` render.

This is best illustrated with an example. The code below renders two scenes in a portal using `RenderTexture` and accepts all props that `RenderTexture` does.

In addition to accepting the same `children` as `RenderTexture`, `ShaderPass` optionally takes as `children` a function, `(fbos) => children`. `fbos` contains `ref`s to each scene's FBO texture output, keyed by the `name` prop of the corresponding `ShaderPass`.

The code snippet below shows an invisible scene being rendered and its output being used as an input `uniform` FBO texture to the fragment shader of a `THREE.RawShaderMaterial` used in a second invisible scene. The output of the second scene is then attached as the `map` to a material in a third, visible scene.

```jsx
// Example
<mesh>
  <Geometry />
  <meshStandardMaterial>
    <ShaderPassesTexture>
      <ShaderPass name="firstPass">
        <Camera />
        <mesh>
          <Geometry/>
          <ShaderMaterial {...uniforms} />
        </mesh>
      </ShaderPass>
      <ShaderPass name="secondPass" attach="map">
        {(fbos) => (
          <>
            <Camera2 />
            <mesh>
              <Geometry/>
              <ShaderMaterial2
                {
                ...uniforms2,
                inputFbo: fbos?.firstPass.current
                }
              />
            </mesh>
          </>
        )}
      </ShaderPass>
    </ShaderPassesTexture>
  </meshStandardMaterial>
</mesh>
```

## Exports

1. [`ShaderPassesTexture`](#ShaderPassesTexture)
2. [`ShaderPass`](#ShaderPass)
3. [`GetParent` & `withParent`](#getParent)

<hr/>

### <a name="ShaderPassesTexture">**`ShaderPassesTexture`**</a>

```js
import { ShaderPassesTexture } from 'react-three-shader-passes'
```

Wraps any number of `ShaderPass` components among its `props.children` and registers their output textures to pass through to `ShaderPass` instances that need to use them (e.g., as a shader `uniform` input for further calculations).

<table>
  <tr>
    <th>prop</th>
    <th>type</th>
    <th>description</th>
  </tr>
  <tr>
    <td><code>children</code> </td>
    <td><code>React.ReactNode | React.ReactNode[]</code></td>
    <td><code>ShaderPass</code> instances can be rendered at any level in the <code>children</code>  tree.<br/><br/>
    To benefit from this API, at least one of these <code>ShaderPass</code> instances should utilize the function-as-<code>children</code> pattern to access another <code>ShaderPass</code>'s output. <br/><br/>
    Otherwise, use <code>RenderTexture</code> instead of <code>ShaderPass</code> and omit this component. 
    </td>
  </tr>
</table>

### <a name="ShaderPass">**`ShaderPass`**</a>

Behaves exactly as [`RenderTexture`](https://github.com/pmndrs/drei/?tab=readme-ov-file#rendertexture), which it renders at the top level.

The only difference is that `ShaderPass` must be rendered as a descendent of `ShaderPassesTexture` and can, in addition to a `React.ReactNode`, optionally take a single function `(fbos) => React.ReactNode` as children.

```js
import { ShaderPass } from 'react-three-shader-passes'
```

<table>
  <tr>
    <th>prop</th>
    <th>type</th>
    <th>description</th>
  </tr>
  <tr>
    <td><code>children</code> </td>
    <td><code>React.ReactNode | (fbos) => React.ReactNode</code></td>
    <td>As plain JSX, identical to <code>@pmndrs/drei/RenderTexture</code>'s <code>props.children</code>. <br/><br/> Alternatively, a function that returns the above. It will be called with one argument: an object containing the textures from any other <code>ShaderPass </code> in the same <code>ShaderPassesTexture</code>, keyed by <code>props.name</code>.
      <pre>
&lt;ShaderPass name="somePass">
  ...
&lt;/ShaderPass>
&lt;ShaderPass name="someOtherPass" />
  {
    ({ somePass }) => 
      &lt;>
        ...
        &lt;ShaderMaterial someUniform={somePass.current}/>
        ...
      &lt;/>
  }
&lt;/ShaderPass>
      </pre>
    
  </tr>
  <tr>
    <td><code>name</code></td>
    <td><code>string</code></td>
    <td><strong>Required.</strong> In the object passed as argument to any <code>props.children</code> in function form, this will be used as a property name referencing the texture returned from the underlying <code>RenderTexture</code>
    </td>
  </tr>
  <tr>
    <td><code>...RenderTextureProps</code></td>
    <td><code>Object</code></td>
    <td>
    Props that go into a <code>RenderTexture</code>. E.g., setting <code>props.renderPriority</code> allows you to control the rendering order of the shader passes.
    </td>
  </tr>
</table>

### <a name="getParent">**`GetParent` & `withParent`**</a>

```js
import { getParent, withParent } from 'react-three-shader-passes'
```

An alternative to using `instance.__r3f.parent` (see `useInstanceHandle` [here](https://docs.pmnd.rs/react-three-fiber/api/additional-exports)).

Provides a React `ref` to the parent of non-`Object3D` primitives (e.g. materials). Unlike `Object3D` instances, these do not have a built-in (`.parent`) reference to the containing object in the scene graph.

Using inside a component:

```jsx
function MaterialWithParentInternal(props) {
  const parent = useRef()
  useEffect(() => {
    console.log(parent.current), [parent]
  })
  return (
    <>
      <meshStandardMaterial {...props} />
      <GetParent ref={parent} />
    </>
  )
}
```

Using inside the component and exposing to parent:

```jsx
forwardRef(function MaterialWithParentInternalAndExternal(props, ref) {
  const internal = useRef()
  const parent = useRef()
  useImperativeHandle(ref, () => ({ ...internal.current, parent: parent.current }))
  return (
    <>
      <meshStandardMaterial ref={internal} {...props} />
      <GetParent ref={parent} />
    </>
  )
})
```

Only exposing to parent (`ref` will include added `.parent` property):

```jsx
const MyMaterialWithParentExternal = withParent(MyMaterial)
```

```jsx
const Anscestor = () => {
  const descendentParent = useRef()
  useEffect(() => {
    console.log(descendentParent.current.parent)
  }, [])
  return <MyMaterialWithParentExternal ref={descendentParent} />
}
```

Optional custom parent key:

```jsx
const MyMaterialWithParentExternal = withParent(MyMaterial, 'parentKey')
```

```jsx
const Anscestor = () => {
  const descendentParent = useRef()
  useEffect(() => {
    console.log(descendentParent.current.parentKey)
  }, [])
  return <MyMaterialWithParentExternal ref={descendantParent} />
}
```
