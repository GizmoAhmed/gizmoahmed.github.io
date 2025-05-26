---
title: The Sprite Suite
publishDate: 2025-05-19 00:00:00
img:  \assets\sprite-images\04.png
img_alt: sprite img
description: |
  Experimental Unity Projects  
tags:
  - Unity & C#
  - Visual Scripting 
  - Procedural Generation
  - Unity Editor Tooling
---

> Motivation

One of my favorite undergrad classes at Michigan State University was Advanced Game Development (MI 431), taught by <a href="https://comartsci.msu.edu/our-people/jeremy-gibson-bond" target="_blank" rel="noopener noreferrer">Jeremy Gibson Bond</a>. It stood out because it pushed us to explore game programming concepts we were curious about but hadn’t yet mastered. Prof. Bond emphasized that the best learning happens when you're diving into unfamiliar but interesting territory. It was an open-ended coruse, where we were given topics to research, but once we got the topic, we could research anything we were curious about within it.

We explored three advanced topics: **visual scripting**, **procedural generation**, and **editor tooling**. For each, we followed a tutorial, researched the concept, and built a practical implementation. I’ve collected these projects into what I will call, for the sake of this portfolio, ***The Sprite Suite***. Each one uses sprites either as a central feature or a stylistic choice, thanks to assets I had on hand from each subsequent project (Forgive me for reusing sprites, it saved me a lot of time. It’s still school, I need to space out my work load.).

> Pixel Particle Shader Graph - Visual Scripting

Each of these projects has a corresponding video, here is the <a href="https://youtu.be/K7LluQnPB1U" target="_blank" rel="noopener noreferrer">video</a> for this topic.

Visual scripting is essentially coding without writing traditional code—you build logic visually using nodes. It’s often used to create effects or shaders without diving deep into raw scripts. Since I’m a fan of pixel art (something about that nostalgic feel), I started thinking about the kinds of visual effects I’d want in a pixel-style game. Pixelized fire was an easy choice for an affect. This was the result:

<video controls width="600">
  <source src="\assets\sprite-images\fire.mp4" type="video/mp4" alt="fire.mp4" class="stack gap-10 content"> 
</video>

To make this, I used <a href="https://docs.unity3d.com/Packages/com.unity.shadergraph@17.3/manual/index.html" target="_blank" rel="noopener noreferrer">Unity's Shader Graph</a>, which is a powerful tool that replaces low-level shader coding with a node-based editor. I followed this <a href="https://www.youtube.com/watch?v=JZDlCuIpq9I" target="_blank" rel="noopener noreferrer">tutorial</a> as a starting point, then customized it to get the effect I wanted.

In Shader Graph, each node acts like a tiny operation—things like tiling, division, multiplication, and so on. To pixelate individual particles, I had to think about how pixelation works mathematically. Here’s a quick breakdown of the logic behind my shader:

First, you take the UV coordinates of the particle (basically its layout on the texture). Then you multiply them by a number—which we’ll call bits. Increasing this number gives the appearance of more “blocks” or “pixels.” (Technically, it's not pixel count—it just controls how UVs get divided and floored. The name is just a helpful way to think about it.)

After that, you floor divide and offset the UVs. This step chops the space into evenly sized, grid-like chunks. The math looks something like this:

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\sprite-images\UVpng.png
        width="300px"
        height="300px"
        alt="UVpng"
    />
</a>

To color the particle, I multiplied the output by the vertex color, which acts as a customizable input. That result goes into the fragment shader, and just like that—you get your particle.

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\sprite-images\result_texture.png
        width="300px"
        height="300px"
        alt="UVpng"
    />
</a>

This setup is flexible—it works across different colors, bit values, and even textures (you can see smoke in the lower left corner). I also added bloom effects by adjusting Unity’s global volume settings. You can see how the fire glows more intensely in the top row of the scene:

<video controls width="600">
  <source src="\assets\sprite-images\fire.mp4" type="video/mp4" alt="fire.mp4" class="stack gap-10 content"> 
</video>

When it comes down to it, it’s all just math. Shader Graph is an entire skill in itself, and a super useful one. To test the effect, I built a small demo scene to see how it might feel in an actual game environment.

<video controls width="600">
  <source src="\assets\sprite-images\shader_showoff.mp4" type="video/mp4" alt="shader_showoff" class="stack gap-10 content"> 
</video>

*(And yep—notice the weapons in the scene. It’s a perfect segue into the next project...)*

> Procedural Generation - Randomly Generated Weapons

Here is my <a href="https://www.youtube.com/watch?v=qGzb43dUKaY" target="_blank" rel="noopener noreferrer">video</a> for this one.
 
Procedural generation is about using algorithms to create game content (usually randomly), like environments, enemies, weapons, etc. I’ve always loved games like Terraria, Destiny, and Borderlands, where gear is randomly generated. So, I tried building a system that generates pixel-art weapons.

Each weapon consists of three parts: blade, hilt, and handle. I used an <a href="https://assetstore.unity.com/packages/2d/textures-materials/abstract/pixel-art-swords-and-staffs-200484" target="_blank" rel="noopener noreferrer">pixel art asset pack by SnoopethDuckDuck</a> that has all of the weapon sprites you will be seeing. On generation, a random combination of those parts is selected and enhanced with effects like color, particle glow, rarity, and a unique name.

<video controls width="600">
  <source src="\assets\sprite-images\three.mp4" type="video/mp4" alt="three" class="stack gap-10 content"> 
</video>

*The three weapons above are randomly generated.*

I can also do an entire grid of them so you can see the different kinds of wepaons that can be made:

<video controls width="600">
  <source src="\assets\sprite-images\grid.mp4" type="video/mp4" alt="grid" class="stack gap-10 content"> 
</video>

The implementation is actually rather simple. Each piece (handle, hilt, and blade) are all categories for a type of sprite. These sprites, when spawned per sword spawn, are stacked at an anchor point at 0,0. The blade’s anchor is at the base, the hilt is in the middle, and the handle is at the top. That way, when they all have their positions reset to 0, they align logically:

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\sprite-images\align.png
        width="300px"
        height="300px"
        alt="align"
    />
</a>

Once these three pieces are chosen randomly, the other factors can be applied. 

Rarity is determined by the internal tags on each sprite. If all three parts (blade, hilt, handle) come from different sets, the weapon is **common**. If two match, it’s **uncommon**, and if all three are from the same set, it’s **rare**. 

So you can see where the rarity comes in. Statistically speaking, here are your chances of getting a sword of following rarities:

- common has a **61.2%** chance
- uncommon is **36.7%**
- rare is **2%**

Based on the rarity, glow effects are applied. I use a <code>Light2D</code> component animated with <a href="https://dotween.demigiant.com/documentation.php" target="_blank" rel="noopener noreferrer">DoTween</a> for a pulsing glow. The rarer the item, the brighter and more dramatic the glow:

<code>
  

  
    public class Weapon : MonoBehaviour
    public enum Rarity
    {
        Unset,
        Common,
        Uncommon,
        Rare
    }
    {...}
    switch (currentRarity)
    {
        case Rarity.Common:
            FadeLight(0.75f);
            break;
        case Rarity.Uncommon:
            FadeLight(3f);
            break;
        case Rarity.Rare:
            FadeLight(4f);
            break;
    }

    public void FadeLight(float amount)
    {
        if (personalSpriteLight == null)
        {
            Debug.LogError("personal light not on this object");
            return;
        }

        var light2D = personalSpriteLight.GetComponent<Light2D>();

        if (light2D == null)
        {
            Debug.LogError("light 2D is not on this weapons personal light");
            return;
        }

        var variation = amount * 0.2f; // ±20% oscillation range

        // DOTween DOFloat to animate Light2D intensity
        DOTween.To(() => light2D.intensity, x => light2D.intensity = x, amount + variation, 0.5f)
            .SetLoops(-1, LoopType.Yoyo) // Infinite back-and-forth oscillation
            .SetEase(Ease.InOutSine); // Smooth transition
    }
</code>
 
Weapon names are composed based on the selected hilt and rarity. Each hilt has an associated name bank, and rarity determines which bank is pulled from. Names are then stitched together using some simple string logic. (Right now, names are only visible in the editor — that’s fine for showcasing.)

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\sprite-images\namegrid.png
        width="500px"
        height="500px"
        alt="name grid"
    />
</a>

Color of the particles is the average colors of all of the pixels that compose the blade. This one is a little more complicated. The idea is to find the most common color in the blade texture, ignoring the alpha particles (as they contribute nothing). Here is how those colors are collected:

<code>


      private List<Color> GetMostCommonColors(Texture2D texture, int topN)
      {
        texture.filterMode = FilterMode.Point;
        texture.Apply();

        var pixels = texture.GetPixels();
        var colorCounts = new Dictionary<Color, int>();

        var totalColors = 0;

        foreach (var color in pixels)
        {
            if (color.a < 0.1f) continue; // Ignore near-transparent pixels

            var roundedColor = new Color(
                Mathf.Round(color.r * 255) / 255f,
                Mathf.Round(color.g * 255) / 255f,
                Mathf.Round(color.b * 255) / 255f,
                1f
            );

            if (colorCounts.ContainsKey(roundedColor))
            {
                colorCounts[roundedColor]++;
            }
            else
            {
                colorCounts[roundedColor] = 1;
                if (WeaponFactory.Instance().ShowColorDebug) Debug.Log($"New Color: {color.ToString()}");
            }

            totalColors++;
        }

        if (WeaponFactory.Instance().ShowColorDebug) Debug.Log($"total colors: {totalColors}");

        return colorCounts.OrderByDescending(kv => kv.Value)
            .Select(kv => kv.Key)
            .Take(topN)
            .ToList();
      }
</code>

This list is passed into the particle system's color gradient. Combined with randomized particle shapes, this gives each weapon particle colors that make sense to it.

<video controls width="600">
  <source src="\assets\sprite-images\colors.mp4" type="video/mp4" alt="colors" class="stack gap-10 content"> 
</video>

Of course, all of this is cosmetic. But in an actual game, this same system could be used to generate meaningful stats — attack power, durability, special effects, and so on. As you add more sword peices, the pool possible combinations gets exponentially bigger. You can have as much variation as you like.

That said, making new sprites manually was a chore. I'd draw them in my browser, export them, bring them into Unity, and manually adjust anchor points every time. How could I keep it in the editor?...

> Editor Tooling - Sprite Editor

…I would keep it in editor by making my own sprite editor <a href="https://www.youtube.com/watch?v=Q93si4NkQ7E" target="_blank" rel="noopener noreferrer">(video)</a>. I’m pretty proud of this one, as I see myself (and others) using it a lot for other projects in the future.

I built a Unity Editor Tool that lets you draw and edit pixel art directly in the editor. I started with this <a href="https://www.youtube.com/watch?app=desktop&v=34736DHWzaI&t=0s" target="_blank" rel="noopener noreferrer">tutorial</a> as a foundation, then expanded it with features like:

- Undo-ing

- Adjustable pixels per unit (PPU)

- Color swapping and clearing

- Saving/exporting drawings as textures

You can create completely original sprites or modify ones already in your project:

<video controls width="600">
  <source src="\assets\sprite-images\apple.mp4" type="video/mp4" alt="apple" class="stack gap-10 content"> 
</video>

**GUILayout**

To get this working, I made a custom editor window by putting a script inside an <code>Editor</code> folder and creating a class like this:

<code>


    public class SpriteEditor : EditorWindow
    {
      protected void OnGUI()
      {
          // Called every time the window redraws
      }
    }
</code>

The <code>OnGUI()</code> method gets every refresh of the window, so any clicks or interactions inside the window trigger it. This gives you control to draw sprites, toggle the grid, or show tool options like this:

<code>


    if (spriteToEdit != null)
    {
        HandleMouseInput(drawArea);
        DrawSprite(drawArea);
        if (GridOn) DrawGrid(drawArea);
    }
    else
    {
        EditorGUI.LabelField(drawArea, "No Sprite Selected",
            new GUIStyle { alignment = TextAnchor.MiddleCenter, normal = { textColor = Color.gray } });
    }
</code>

You can also layout the window UI with <code>GUILayout</code>, which works sort of like HTML/CSS:

<code>



    GUILayout.Label("Edit a Sprite", EditorStyles.boldLabel);
    pickedColor = EditorGUILayout.ColorField("Selected Color: ", pickedColor);
    switchedToColor = EditorGUILayout.ColorField("Switch to this Color: ", switchedToColor);
    pixelsPerUnit = EditorGUILayout.IntField("Pixels Per Unit", pixelsPerUnit);

    // Grid size update
    EditorGUI.BeginChangeCheck();
    var newGridSize = EditorGUILayout.IntField("Grid Size: ", GridSize);
    if (EditorGUI.EndChangeCheck())
    {
        GridSize = Mathf.Max(1, newGridSize);
        spriteToEdit = null;
        InitEditableTexture();
        Repaint();
    }

    // Sprite picker
    spriteToEdit = EditorGUILayout.ObjectField("Sprite To Edit: ", spriteToEdit, typeof(Texture2D), false) as Texture2D;
</code>

It ends up looking like this:

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\sprite-images\layout.png
        width="500px"
        height="500px"
        alt="layout"
    />
</a>

**Drawing**

When you click inside the editor window, the tool tracks mouse position, converts it to pixel coordinates, and colors in the pixels:

<code>



      protected override void HandleMouseInput(Rect area)
      {
          var evt = Event.current;
          var localMousePos = evt.mousePosition - new Vector2(area.x, area.y);
          var pixelX = Mathf.FloorToInt(localMousePos.x / 10);
          var pixelY = Mathf.FloorToInt(localMousePos.y / 10);

          if (evt.type == EventType.MouseDown && evt.button == 0)
          {
              isDrawing = true;
              strokeSnapshot.Clear();
              SetPixelColor(pixelX, pixelY);
              evt.Use();
          }
          else if (evt.type == EventType.MouseDrag && isDrawing)
          {
              SetPixelColor(pixelX, pixelY);
              evt.Use();
          }
          else if (evt.type == EventType.MouseUp && isDrawing)
          {
              isDrawing = false;
              if (strokeSnapshot.Count > 0)
                  undoActionStack.Push(new Dictionary<Vector2Int, Color>(strokeSnapshot));
              evt.Use();
          }
      }
</code>

It’s pretty intuitive—click, drag, draw.

**Saving**

Of course, you’ll want to save your masterpiece. By default, Unity textures are blurry and compressed, so I had to tweak some settings through the AssetDatabase to make things look crisp:

<code>


      AssetDatabase.ImportAsset(relativePath);
      var importer = AssetImporter.GetAtPath(relativePath) as TextureImporter;
      if (importer != null)
      {
          importer.textureType = TextureImporterType.Sprite;
          importer.isReadable = true;
          importer.filterMode = FilterMode.Point;
          importer.textureCompression = TextureImporterCompression.Uncompressed;
          importer.maxTextureSize = 16384;

          importer.spritePixelsPerUnit = Mathf.Max(1, pixelsPerUnit);

          AssetDatabase.WriteImportSettingsIfDirty(relativePath);
          AssetDatabase.Refresh()
          Debug.Log("Texture settings applied!");
      }
</code>

This trick using AssetDatabase lets you control import settings that aren’t usually exposed.

**Clearing and Switching Colors**

In the demo from earlier, you’ll notice I can clear or swap colors in the sprite. I remembered from a procedural generation project that you can loop through a texture’s pixels, so I used that to implement the feature:

- Swap Colors

<code>



      for (var i = 0; i < allPixels.Length; i++)
          if (allPixels[i] == pickedColor)
              allPixels[i] = switchedToColor;
</code>

- Clear Color

<code>



      for (var i = 0; i < allPixels.Length; i++)
          if (allPixels[i] == pickedColor)
              allPixels[i] = Color.clear;
</code>

It’s a simple way to update whole regions of a sprite. Here's a quick look:

<video controls width="600">
  <source src="\assets\sprite-images\clear_switch.mp4" type="video/mp4" alt="clear_switch" class="stack gap-10 content"> 
</video>

**Undo**

An editor like this really needs undo. I realized that quickly while testing and messing up, then having to start the whole sprite drawing over. I implemented an undo system using a stack—push pen strokes onto it and pop them off when undo is clicked, which is really the standard for undo functions like this.

<code>



      private void AddToActionStack()
      {
          if (editableTexture == null) return;
          undoActionStack.Push(editableTexture.GetPixels());
      }

      private void Undo()
      {
          if (undoActionStack.Count == 0) return;

          var lastAction = undoActionStack.Pop();

          if (lastAction is Dictionary<Vector2Int, Color> stroke)
          {
              foreach (var pixel in stroke)
                  editableTexture.SetPixel(pixel.Key.x, editableTexture.height - pixel.Key.y - 1, pixel.Value);
          }
          else if (lastAction is Color[] fullTextureState)
          {
              editableTexture.SetPixels(fullTextureState);
          }

          editableTexture.Apply();
          Repaint();
      }
</code>

Here it is in action:

<video controls width="600">
  <source src="\assets\sprite-images\undo.mp4" type="video/mp4" alt="undo" class="stack gap-10 content"> 
</video>

**Asset Store?**

I’ve been thinking of putting this on the Unity Asset Store. Yes, I know there are already really <a href="https://assetstore.unity.com/search#q=sprite%20editor%20window" target="_blank" rel="noopener noreferrer">good ones</a> on there, but I’m really happy with how mine turned out, and I think others might find it useful.

**Conclusion**

I might just release all three of my tools for free on the Asset Store eventually. They’re not quite ready for that yet, but it’s on my radar. Thanks for reading—I really appreciate it!