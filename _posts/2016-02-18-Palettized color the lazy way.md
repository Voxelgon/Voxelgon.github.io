---
title: Palettized color the lazy way
post: 2016-02-18-Palette
thumb: palette.png
layout: post
---

Early on when I started developing the game, I decided on using vertex colors for my art style. I had already decided on using .obj files for importing models, however, which dont support vertex color, but do support UVs. In addition, I wanted to make sure these vertex colors came from a limited palette, something I called a "colormap". This isnt a color palette in the traditional sense. The colors are all still full 32-bit when displayed on the screen, and can be shaded and added as needed, they just needed to start from a limited set.

To solve this, and to create a more efficient workflow, I decided to put all my colors into a PNG file and import it as a texture into both Blender and Unity. This allowed me to assign colors using the UV channel of the mesh, preserving them in the .obj files, as well as giving me a kind of "color picker" to choose them in Blender. It also allowed me to quickly view the colored mesh in unity.

{% include image.html post=page.post img="blender.png" description="using the color map in Blender" %}

 The problem, however, was that if the UV vertices werent centered perfectly on the pixels, other colors could "bleed" into the model. The fix to this was to use actual vertex colors when inside unity. I wrote a quick script that took a mesh and a texture and assigned vertex colors based on its UVs.

```C#
public static Mesh ColorMesh(Mesh colorMesh, Texture2D colormap) {
    //colors a meshe's vector-colors to match its UV mapping over a colormap
    Vector2[] uvs = colorMesh.uv;
    Color32[] colors = new Color32[uvs.Length];
    int textureW = colormap.width;
    int textureH = colormap.height;

    for(int i = 0; i < uvs.Length; i++) {
        int pixelX = (int) ((uvs[i].x % 1) * textureW);
        int pixelY = (int) ((uvs[i].y % 1) * textureW);
        Color32 color = colormap.GetPixel(pixelX, pixelY);
        colors[i] = color;
    }
    colorMesh.colors32 = colors;

    return colorMesh;
}
```

After creating a simple vertex-color shader (a task far easier said than done!) I had the style I wanted. As a bonus, I now had a custom shader I could tweak.

The limited color set works in my favor in terms of gameplay. I decided to associate all the colors with specific resources and systems, for example energy is yellow, food is green, water is blue, etc. (Color blind players: Dont worry! color will never be the _only_ way to distinguish gameplay elements).

{% include image.html post=page.post img="labeled.png" description="gameplay elements assigned to colors" %}

(If you are wondering why I chose magenta for radiation, check out [Cherenkov Radiation](https://en.wikipedia.org/wiki/Cherenkov_radiation).)

This all happened before Unity 5, so this was a normal non-PBR shader. Based on the style im using, I probably will not use PBR at all.