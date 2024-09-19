---
layout:     page
title:      GUI-Agnostic 3D Rendering Library in Java, TurquoiseGraphics
summary:    A GUI-Agnostic renderer built in Java, can display move, scale and rotate objects. Implemented in Swing.
categories: jekyll pixyll
---

Realizing that I was able to use GUIs like Swing to draw triangles, I became inspired. If you can draw triangles, you can draw **3D Shapes**! This idea led me to create a 3D renderer in Java. The object structure of the program was intricate and took a lot of refining, but I’m happy with the result.

**TurquoiseGraphics** is a **GUI-agnostic** renderer written in Java. This specific demo is implemented in Swing, but it can load any **OBJ file** and allow you to **move**, **scale**, and **rotate** it while the player moves around. One of its key strengths is its ability to be reused across any particular visual implementation, even in a terminal if necessary!

_![Showing the 3D engine](/images/main_show_3d.png){:width="1000px"}_

The program renders a **Scene** that contains many **RenderObjects**. Here’s an example of the process of creating an object and adding it to the scene:

```java
// Create scene object, which holds many RenderObjects
scene = new Scene(new ArrayList<RenderObject>(Arrays.asList()));

// Create a teapot object, from the "teapot.obj" file, with a red shadowed shader
teapot = RenderObject.loadObject("data/teapot.obj", "teapot", new InverseSqrShadow(new Color(255,0,0), scene), new Vertex(0,0f,0));

scene.addObject(teapot);

teapot.setScale(new Vertex(3,2,1));
```

In this example, the **InverseSqrShadow** is a subclass of the **ColourShader** class. It works similarly to a pixel shader, but instead of shading individual pixels, it shades whole triangles. Here are some examples of how different shaders look:

**InverseSqrShadow** Teapot and **InverseSqrShadow** Floor:

![InverseSqrShadow Shader](/images/sqrshadow.png){:width="600px"}

**NonShadow** Teapot and **InverseSqrShadow** Floor:

![InverseSqrShadow Shader](/images/noshadowteapot.png){:width="600px"}

**HorizontalShader** Teapot and **InverseSqrShadow** Floor:

![InverseSqrShadow Shader](/images/horizontal.png){:width="600px"}

Here’s some of the code behind the **InverseSqrShadow** color shader:

```java
/**
 * Returns the color this triangle should be based on its position in the scene.
 * 
 * In particular, this function reduces the brightness of triangles
 * based on their proximity to the camera.
 * 
 * @param triangle the triangle to be shaded
 * @return finalColour the color the triangle should be
 */
public Color shadeBasedOnTriangle(Triangle triangle) {
    // Average out the triangle's vertices so it can be treated as one point
    Vertex tV = averageTriangleAsVertex(triangle);

    // Find the distance to the camera
    Vertex diff = Vertex.difference(tV, scene.getCamPos());
    
    // Scale the RGB values by the inverse square of the distance to the camera and the shaderFactor
    int red = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor) * colour.getRed());
    int green = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor) * colour.getGreen());
    int blue = (int) Math.round(inverseSquare(diff.magnitude() * shaderFactor) * colour.getBlue());

    Color finalColour = new Color(red, green, blue);
    return finalColour;
}

/**
 * Calculates the inverse square of a given value.
 * In physics, the greater the distance from a light source, 
 * the exponentially less light you see.
 * 
 * @param x the value to be inverse squared
 * @return 1/((absX+1)*(absX+1)) the result of the calculation
 */
public static float inverseSquare(float x) {
    float absX = Math.abs(x);
    return 1 / ((absX + 1) * (absX + 1));
}
```
Below is the main section of code required to implement the 3D renderer into any project:

```java
/**
 * Performs all projection calculations and draws the 
 * scene to the screen.
 */
private void drawSceneToScreen() {
    
    // Perform all projection calculations for every triangle
    scene.renderScene();
    
    // Create parallel arrays for the rendered data
    Triangle2D[] trianglesToDisplay = scene.getRenderedTriangles();
    Color[] colours = scene.getColours();
    String[] names = scene.getNames();

    // For each triangle in the scene, display it
    for (int index = 0; index < scene.getCount(); index++) {
        if (trianglesToDisplay[index] != null) {
            
            // Take the triangle coordinates and color and draw it
        }
    }
}
```

As this project is open source, you can [find the code here on GitHub](https://github.com/jc10101010/TurquoiseGraphics).

### Future Plans:
- Implement shaders that cast shadows from objects
- Improve efficiency
