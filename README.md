# JSON scenegraph

The goals are:

 - directly usable in *Javascript* using `JSON.parse()`
 - same node types and properties as **VRML / X3D**
 - don't use *strings* for boolean or numbers
 - write nodes *always the same way*, regardless of the container
 - avoid *deep nesting*
 - avoid *duplicated* data


-------------------------------------------------------------------------------

## Examples

Two shapes:

```javascript
[
	{
		"$": "Shape",
		"appearance": {
			"$": "Appearance",
			"material": {
				"$": "Material",
				"diffuseColor": [0.8, 0.8, 1]
			}
		},
		"geometry": {
			"$": "Box",
			"size": [2, 2, 2]
		}
	},
	{
		"$": "Shape",
		"appearance": {
			"$": "Appearance",
			"material": {
				"$": "Material",
				"transparency": 0.5
			}
		},
		"geometry": {
			"$": "Sphere",
			"radius": 5
		}
	}
]
```

Re-using a node (DEF/USE):

```javascript
[
	{
		"$DEF": "myshape",
		"$": "Shape",
		"appearance": {
			"$": "Appearance",
			"material": {
				"$": "Material",
				"diffuseColor": [0.8, 0.8, 1]
			}
		},
		"geometry": {
			"$": "Box",
			"size": [2, 2, 2]
		}
	},
	{
		"$": "Transform",
		"translation": [1, 0, 0],
		"children": [
			{
				"$USE": "myshape"
			}
		]
	}
]
```

Transforms in a Group:

```javascript
[
	{
		"$": "Group",
		"children": [
			{
				"$": "Transform",
				"translation": [-2, 0, 0],
				"children": [
					{
						"$": "Shape",
						"appearance": {
							"material": {
								"$": "Material",
								"diffuseColor": [0, 1, 0.5]
							}
						},
						"geometry": {
							"$": "Box"
						}
					},
				]
			},
			{
				"$": "Transform",
				"translation": [2, 0, 0],
				"children": [
					{
						"$": "Shape",
						"appearance": {
							"material": {
								"$": "Material",
								"diffuseColor": [0, 0.5, 1]
							}
						},
						"geometry": {
							"$": "Sphere"
						}
					},
				]
			}
		]
	}
]
```

-------------------------------------------------------------------------------

## Design choices / FAQ


### `"$": "Shape"` vs `"Shape: [`

> **$** because each object is represented by the same set of fields regardless of it's position in the tree, and MFNode that contain multiple types of node don't lose the order of the nodes.
> 
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/1)


### Properties defined multiple times

> **Don't repeat properties**: most JSON parsers discard or refuse duplicated properties.
> 
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/2)


### Repeating `$` vs repeating `Shape`

> **$** because it produces less deep structures.
> 
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/3)


### `"_children": "Shape"` vs Array of `{"$":"Shape"}`

> **Array** because it allows MFNode of multiple types of nodes, not just one type. Also it's more explicit when a type has multiple MFNode properties.
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/4)


### MFVec3f: `[1,2,3,4,5,6]` vs `[[1,2,3],[4,5,6]]`

> **[1,2,3,4,5,6]** because it's [like VRML & X3D](http://www.web3d.org/documents/specifications/19775-1/V3.2/Part01/fieldsDef.html#SFVec2fAndMFVec2f), less deeply nested, and easier for WebGL [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays).
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/6)


### Explicit vs implicit properties

> **Explicit** because it *avoids guessing*, doesn't break the *"always write a node the same way"* rule, has *shorter paths* to access nested values and uses the same paths as *VRMLscript*.
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/7)


### Routes

> Use `$ ROUTE` objects for expressing routes.
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/8)


### CDATA

> Like VRML, use simple String values for SFString values that the XML encoding expresses as CDATA.
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/9)


### Comments

> Use Metadata nodes when comments must be part of the scenegraph,
> or regular Javascript-style comments with a JSON5 parser.
>
> [More details and comments](https://github.com/wildpeaks/json-scenegraph/issues/10)


-------------------------------------------------------------------------------

## Custom nodes (protos)

You can use non-standard `$` types to make custom nodes.

```javascript

{
	"$": "Teapot",
	"bottom": false,
	"size": 5
}
```

The implementation would depend on the 3D engine, however I'd advise against putting the JS code inside the scenegraph itself because [it would run with eval(), which is not recommended](http://jslinterrors.com/eval-is-evil).

Better structure your code using modules (CommonJS, ADM or ES6 classes) that generate JSON on demand, or even Web Components like [A-Frame](https://aframe.io).


-------------------------------------------------------------------------------

