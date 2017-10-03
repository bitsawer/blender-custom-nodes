# blender-gmic

This repository contains files for adding a G'MIC compositor node for Blender. Currently Windows only, but it should not be hard to add support for Linux and Mac at some point.

## How to use

Download or build your own Blender with the node. The G'MIC node if found in the compositor under the "Add -> Filter -> G'MIC". It works like a norma node, just put G'MIC commands into the node text field.

Some operations can be pretty slow, especially with large images. You can adjust the scaling quality if you are not rendering the final image and you are just tweaking other nodes.

## Node settings

* Quality setting can be used to scale down the image for faster processing. This is useful if you are working with a large image with a complex node tree. Just remember to set quality to high once you are ready to save the final output.

* Normalize can be used to normalize RGBA values to 0-1.0 range. This just inserts command "-n 0,1" after user command string.

* Argument slots can be used to pass arguments to your commands. You can reference variables like this: "$arg1".

## Troubleshooting and tips

* You can accidentally put commands (like "-display" or just a single, lone "-"-character) into the node which will prompt GMIC to do something interactively. Don't do that. You can usually notice this if the progress bar in the compositing view is not going away (or the filter can just be slow). In some cases you can open the console and type some text and press enter to recover. Sometimes the safest bet is to kill the Blender-process and start it again.

* If the image is red, there was an error with the GMIC. Usually a misspelled command or wrong arguments. Check the console (Window -> Toggle System Console) for more information about what went wrong.

* Try to fiddle with the "Use Alpha"-flag in the final "Composite"-node. Or connect the image source node "Alpha" directly into the final "Composite"-node alpha to ignore any (possibly unwanted) alpha changes done by the G'MIC filter.

* Some filters can output very large RGB-values (way above the common range of 0.0 to 1.0). Useful nodes for debugging and working with black and white filters: "Alpha Over", "Set Alpha", "RGB to BW" and "Normalize".

* When you see "-normalize 0,255" or "-n 0,255" in G'MIC examples, try to use "-n 0,1" instead. Same goes for "-equalize 255" etc. The Blender compositor uses 32-bit floating point values (where values are usually between 0.0 and 1.0) instead of normal 8-bit (0 to 255) values used in many image formats.

* Hover your cursor over the final compositor image and press mouse button (right or left depending on your configuration) so you can see the individual RGBA values of the pixel under the cursor. Sometimes the image is there, but alpha is 0 or RGB-values are massively overblown.

## Links

* [G'MIC Reference](http://gmic.eu/reference.shtml)
* [G'MIC gallery and examples](http://gmic.eu/gallery.shtml)
