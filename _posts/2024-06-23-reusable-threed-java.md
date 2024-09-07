---
layout:     page
title:      Reusable 3D in Java, TurquoiseGraphics
summary:    A reusable renderer built in Java, can display move, scale and rotate objects.
categories: jekyll pixyll
---

As some of you may know, Java Swing provides GUI tools for Java, which means you 
can draw triangles. And if you can draw triangles then you can draw __3D Shapes__! 
This inspired me to create a 3D Renderer in Java. Getting the object structure of 
the program to work was the biggest challenge, but was very rewarding when a solution
works well.

In the end I had created a reusable 3D Graphics library in Java, 
I named it __TurquoiseGraphics__. It could load in any __obj file__ then __move it__, __scale it__ and __rotate it__ while
the player walks around. It can also be reused across any particular visual implementation
(not just Swing, even in the terminal if neccesary!). 

_![Showing the 3D engine](/images/main_show_3d.png){:width="1000px"}_

The program renders a __Scene__ which has many __RenderObjects__ in it. Here is what the process of creating an 
object to add to the scene looks like:



{% highlight java lineanchors %}

//Create scene object, which holds many RenderObjects
scene = new Scene(new ArrayList<RenderObject> (Arrays.asList()));

//Create a teapot object, from the "teapot.obj" file, with a red shadowed shader
teapot = RenderObject.loadObject("data/teapot.obj", "teapot", new InverseSqrShadow(new Color(255,0,0), scene), new Vertex(0,0f,0));

scene.addObject(teapot)

teapot.setScale(new Vertex(3,2,1))

      . . . 
{% endhighlight %}

Here the __InverseSqrShadow__ is a subclass of the __ColourShader__ class which works almost like a pixel shader,
except instead of shading individual pixels it shades whole triangles. Here are some examples of how shaders look:

__InverseSqrShadow__ Teapot and __InverseSqrShadow__  Floor:

![InverseSqrShadow Shader](/images/sqrshadow.png){:width="600px"}

__NonShadow__ Teapot and __InverseSqrShadow__ Floor:

![InverseSqrShadow Shader](/images/noshadowteapot.png){:width="600px"}

__HorizontalShader__ Teapot and __InverseSqrShadow__ Floor:

![InverseSqrShadow Shader](/images/horizontal.png){:width="600px"}

Here is some of the code behind the 
__InverseSqrShadow__ colour shader:

{% highlight java lineanchors %}

/**
* Returns the colour this triangle should be. 
* Based on the position of the triangle, the rest of the scene.
* 
* In particular this function reduces the brightness of triangles
* based on their proximity to the camera
* 
* @param  triangle  the triangle to be shaded
* @return  finalColour the colour the triangle should be
*/
public Color shadeBasedOnTriangle(Triangle triangle) {
    //Averages out triangle so it can be treated as one point
    Vertex tV = averageTriangleAsVertex(triangle);

    //Find distance to camera
    Vertex diff = Vertex.difference(tV, scene.getCamPos());
    
    //Scale the rgb values by the inverse square distance to the camera and the shaderFactor
    int red = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor) * colour.getRed());
    int green = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor)* colour.getGreen());
    int blue = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor)* colour.getBlue());

    Color finalColour = new Color(red, green, blue);
    return finalColour;
}

/**
* Calculates the inverse square of some value.
* In physics, the greater the distance from a light 
* The exponentially less light you see.
* 
* @param  x  the value to be inverse squared
* @return  1/((absX+1)*(absX+1)) the calculation result
*/
public static float inverseSquare(float x) {
    float absX = Math.abs(x);
    return 1/((absX+1)*(absX+1));
}

{% endhighlight %}

As this project is open source, [Find the code here on github](https://github.com/jc10101010/TurquoiseGraphics). This is the main bit of code required to 
implement the 3D renderer into any project:

{% highlight java lineanchors %}

/**
* Performs all projection calculations and draws the 
* scene to the screen
*/
private void drawSceneToScreen() {
    
    //Does all of the projection calculations for every triangle
    scene.renderScene();
    
    //Create parralel arrays for the rendered data
    Triangle2D[] trianglesToDisplay = scene.getRenderedTriangles();
    Color[] colours = scene.getColours();
    String[] names = scene.getNames();

    //For each triangle in the scene, display it
    for (int index = 0; index < scene.getCount(); index++) {
        if (trianglesToDisplay[index] != null) {
            
            //TAKE THE TRIANGLE COORDINATES AND COLOR AND DRAW IT

        }
    }
}

{% endhighlight %}

### Future Plans: 
  * Shaders that cast shadows from objects
  * Improve efficiency
