# Tips & Tricks


 

## Where are Houdini's internal lop utils stored?
You can find Houdini's internal loputils under the following path:
~~~admonish tip title=""
$HFS/houdini/python3.9libs/loputils.py
~~~
It is not an official API module, so use it with care, it may be subject to change.

You can simply import via `import loputils`. It is a good point of reference for UI related functions, for example action buttons on parms use it at lot.

Here you can find the [loputils.py - Sphinx Docs](https://ikrima.github.io/houdini_additional_python_docs/loputils.html) online.


## How do I get the LOPs node that last edited a prim ?
When creating data on your layers, Houdini attaches some custom data to the `customData` prim metadata. Among the data is the `HoudiniPrimEditorNodes`. This stores the internal [hou.Node.sessionId](https://www.sidefx.com/docs/houdini/hom/hou/nodeBySessionId.html) and allows you to get the last node that edited a prim.

This value is not necessarily reliable, for example if you do custom Python node edits, this won't tag the prim (unless you do it yourself). Most Houdini nodes track it correctly though, so it can be useful for UI related node selections.

~~~admonish tip title=""
```Python
...
def Xform "pig" (
    customData = {
        int[] HoudiniPrimEditorNodes = [227]
    }
    kind = "component"
)
...
```
~~~
Here is how you retrieve it:
~~~admonish tip title=""
```Python
import hou
from pxr import Sdf
stage = node.stage()
prim = stage.GetPrimAtPath(Sdf.Path("/pig"))
houdini_prim_editor_nodes = prim.GetCustomDataByKey("HoudiniPrimEditorNodes")
edit_node = None
if houdini_prim_editor_nodes:
    edit_node = hou.nodeBySessionId(houdini_prim_editor_nodes[-1])
```
~~~

You can also set it:
~~~admonish tip title=""
```Python
import hou
from pxr import Sdf, Vt
node = hou.pwd()
stage = node.editableStage()
prim = stage.GetPrimAtPath(Sdf.Path("/pig"))
houdini_prim_editor_nodes = prim.GetCustomDataByKey("HoudiniPrimEditorNodes") or []
houdini_prim_editor_nodes = [i for i in houdini_prim_editor_nodes]
houdini_prim_editor_nodes.append(node.sessionId())
prim.SetCustomDataByKey("HoudiniPrimEditorNodes", Vt.IntArray(houdini_prim_editor_nodes))
```
~~~

The Houdini custom data gets stripped from the file, if you enable it on the USD rop (by default it gets removed).
![Alt text](houdiniNodeBySessionId.jpg)

## How do I store side car data from node to node?
To have a similar mechanism like SOPs detail attributes to track data through your network, we can write our data to the `/HoudiniLayerInfo` prim. This is a special prim that Houdini creates (and strips before USD file write) to track Houdini internal data. It is hidden by default, you have to enable "Show Layer Info Primitives" in your scene graph tree under the sunglasses icon to see it. We can't track data on the root or session layer customData as Houdini handles these different than with bare bone USD to enable efficient node based stage workflows.

You can either do it via Python:
~~~admonish tip title=""
```Python
import hou
import json
from pxr import Sdf
node = hou.pwd()
stage = node.editableStage()
prim = stage.GetPrimAtPath(Sdf.Path("/HoudiniLayerInfo"))
custom_data_key = "usdSurvivalGuide:coolDemo"
my_custom_data = json.loads(prim.GetCustomDataByKey(custom_data_key) or "{}")
print(my_custom_data)
prim.SetCustomDataByKey(custom_data_key, json.dumps(my_custom_data))
```
~~~

Or with Houdini's [Store Parameters Values](https://www.sidefx.com/docs/houdini/nodes/lop/storeparametervalues.html) node. See the docs for more info (It also uses the loputils module to pull the data).