# Blender custom compositor nodes

Support for three new compositor nodes:

* [G'MIC](http://gmic.eu/) node.
* OpenGL GLSL fragment shader node. Mostly [Shadertoy](https://www.shadertoy.com/) (Warning: heavy site) compatible.
* Python script node.

All nodes are in the compositor under the "Add -> Filter". Watch the [demo video](https://www.youtube.com/watch?v=vGNB13ovwk4) to see the node in action.

## [Get the latest binary release (Win64)](/../../releases/latest)

If you encounter missing .dll errors when starting Blender, try installing [Microsoft Visual C++ Redistributable for Visual Studio 2017](https://support.microsoft.com/en-us/help/2977003/the-latest-supported-visual-c-downloads)

## What's new?

**0.3.0**

* Added GLSL node
* Added Python script node

## G'MIC Node

Some operations can be pretty slow, especially with large images. You can adjust the scaling quality if you are not rendering the final image and you are just tweaking other nodes.

The release ships with a version of the GIMP-plugin and most filters seem to work out of the box. This is still work in progess, so some of them don't work yet. Many GIMP filters use images that are placed in your "%APPDATA%/gmic"-directory. G'MIC bundled with the Blender release will find them from there if they exist. You can download the whole package from [this link](http://gmic.eu/gmic_all_data.zip) and extract the contents into the target directory.

* Quality setting can be used to scale down the image for faster processing. This is useful if you are working with a large image with a complex node tree. Just remember to set quality to high once you are ready to save the final output.

* Normalize can be used to normalize RGBA values to 0-1.0 range, perform color space conversion and image orientation fixes. You usually want to keep this on, but you can also do these manually if you want more control.

* Argument slots can be used to pass arguments to your commands. You can reference variables like this: "$arg1".

## GLSL Shader Node

The shader node uses (mostly) Shadertoy compatible functions and variables. See the [Shadertoy documentation](https://www.shadertoy.com/howto) for more information. You can use this very simple script to test node functionality:

```glsl
void mainImage(out vec4 fragColor, in vec2 fragCoord)
{	
    fragColor = vec4(0.0, 1.0, 0.0, 1.0); //Output a completely green image (RGBA)
}
```

Another simple example is a mix node that mixes input image channels 0 and 1 together. If the images are read from a file/render layer and not generated in shader, disable node Gamma Correction. Control the mix factor by adjusting the node Input 0 (should be from 0.0 to 1.0).

```glsl
void mainImage(out vec4 fragColor, in vec2 fragCoord)
{	
    vec2 uv = fragCoord.xy / iResolution.xy;
    fragColor = mix(texture(iChannel0, uv), texture(iChannel1, uv), input0);
}
```

## Python Script Node

The purpose of this node is to enable complex new functionality to the compositor without recompiling Blender. The node can be used to make simple modifications to the input image, draw shapes or text, generate procedural images or even use complex third-party libraries. Pillow image processing library is included in the distribution. Thread safety of Pillow is not explicitly documented, so it might be safest to only use it during script import and in on_main().

A **pycompositor** helper module is can be imported by the scripts. It contains some helper functions and it can be used to interface with the [Pillow](https://pillow.readthedocs.io/en/4.3.x/) image processing library. The module is located in ***2.79/scripts/modules*** directory and can be useful as an example.

Here is a simple example script. Images are [numpy arrays](http://www.numpy.org/) which makes it possible to do image processing in an efficient manner. The images are 3D-arrays with a shape of (height, width, depth). Depth is always 4 (red, green, blue and alpha). Image data is also "upside down" as the y-axis starts from the bottom.

```python
import bpy # If you want to read any Blender data
import numpy as np
import pycompositor # Helper script
from PIL import Image, ImageDraw # Pillow

def on_main(context):
    """on_main() is called in the main thread and can be used to access Blender and other libraries in a 
        thread-safe manner. It is possible to add extra data to the context object and use it later
        in the on_async()."""
    images = context["images"] # A list of 4 node input images
    inputs = context["inputs"] # A list of 8 node input values
    outputs = context["outputs"] # A list of 1 node output image. Might have more in the future
    out = outputs[0] # Final output image
    
    # Copy Image 0 to output without any modifications
    #out[:] = images[0]
    
    # Set output to green
    #out[:] = (0, 1, 0, 1)
    
    # Draw a 10 pixel wide horizontal line at the middle of the image
    #y = int(out.shape[0] / 2)
    #out[y:y+10] = (1, 0, 0, 1) #Overwrite image alpha with 1.0
    #out[y:y+10,:,0:3] = (1, 0, 0) #Same as above, but don't touch image alpha, only RGB
    
    # Convert input Image 0 to Pillow image and metadata
    im, meta = pycompositor.array_to_pil(images[0])
    
    # Draw a red line
    draw = ImageDraw.Draw(im)
    draw.line((0, 0) + im.size, fill=(255, 0, 0, 255), width=5)
    
    # Write Pillow image back to output image
    out[:] = pycompositor.pil_to_array(im, meta)
    
def on_async(context):
    """on_async() is called in background thread after on_main() and should be used for processing 
        that takes a long time to avoid blocking Blender UI. Do not touch anything that is not
        thread safe from this function."""
    pass
```

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
