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
		"_type": "Shape",
		"appearance": {
			"_type": "Appearance",
			"material": {
				"_type": "Material",
				"diffuseColor": [0.8, 0.8, 1]
			}
		},
		"geometry": {
			"_type": "Box",
			"size": [2, 2, 2]
		}
	},
	{
		"_type": "Shape",
		"appearance": {
			"_type": "Appearance",
			"material": {
				"_type": "Material",
				"transparency": 0.5
			}
		},
		"geometry": {
			"_type": "Sphere",
			"radius": 5
		}
	}
]
```

Re-using a node (DEF/USE):

```javascript
[
	{
		"_def": "myshape",
		"_type": "Shape",
		"appearance": {
			"_type": "Appearance",
			"material": {
				"_type": "Material",
				"diffuseColor": [0.8, 0.8, 1]
			}
		},
		"geometry": {
			"_type": "Box",
			"size": [2, 2, 2]
		}
	},
	{
		"_type": "Transform",
		"translation": [1, 0, 0],
		"children": [
			{
				"_use": "myshape"
			}
		]
	}
]
```

Transforms in a Group:

```javascript
[
	{
		"_type": "Group",
		"children": [
			{
				"_type": "Transform",
				"translation": [-2, 0, 0],
				"children": [
					{
						"_type": "Shape",
						"appearance": {
							"material": {
								"_type": "Material",
								"diffuseColor": [0, 1, 0.5]
							}
						},
						"geometry": {
							"_type": "Box"
						}
					},
				]
			},
			{
				"_type": "Transform",
				"translation": [2, 0, 0],
				"children": [
					{
						"_type": "Shape",
						"appearance": {
							"material": {
								"_type": "Material",
								"diffuseColor": [0, 0.5, 1]
							}
						},
						"geometry": {
							"_type": "Sphere"
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


### 1. `"_type": "Shape"` vs `"Shape: [`

**Short version**: Each object is represented by the same set of fields regardless of it's position in the tree, and MFNode that contain multiple types of node don't lose the order of the nodes.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/1)


### 2. Properties defined multiple times

**Short version**: Most JSON parsers discard or refuse duplicated properties.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/2)


### 3. Repeating `_type` vs repeating `Shape`

**Short version**: Using `_type` result in less deeply nested structures.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/3)


### 4. `"_children": "Shape"`+ no `_type` vs Array of `{"_type":"Shape"}`

**Short version**: the Array allows MFNode of multiple types of nodes, not just one type. Also it's more explicit when a type has multiple MFNode properties.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/4)


### 5. `_type` vs `type`

**Short version**: `_type` because some VRML nodes already have a property `type`, plus underscore is a common naming convention in Javascript for special/private fields.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/5)


### 6. MFVec3f: `[1,2,3,4,5,6]` vs `[[1,2,3],[4,5,6]]`

**Short version**: `[1,2,3,4,5,6]` fits better because it's [like VRML & X3D](http://www.web3d.org/documents/specifications/19775-1/V3.2/Part01/fieldsDef.html#SFVec2fAndMFVec2f), less deeply nested, and easier for WebGL [Typed Arrays](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays).
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/6)


### 7. Explicit vs implicit properties

**Short version**: **explicit** because it avoids guessing and doesn't break the "always write a node the same way" rule.
[Long version and comments](https://github.com/wildpeaks/json-scenegraph/issues/7)


-------------------------------------------------------------------------------

## Custom nodes (protos)

You could use non-standard `_type` value to define custom nodes.

```javascript

{
	"_type": "Teapot",
	"bottom": false,
	"size": 5
}
```

The implementation would depend on the 3D engine, however I'd advise against putting the JS code inside the scenegraph because [it would run with eval(), which is not recommended](http://jslinterrors.com/eval-is-evil).

Instead, the 3D engine should have a way to register custom types that generates the matching scenegraph for the custom type (e.g. [X3DOM components](http://x3dom.readthedocs.org/en/1.4.0/components.html?highlight=components)).


-------------------------------------------------------------------------------

