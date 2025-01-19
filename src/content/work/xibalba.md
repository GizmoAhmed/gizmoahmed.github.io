---
title: Temple of Xibalba
publishDate: 2020-03-02 00:00:00
img:  \assets\xibalba-images\xibalba_hallway.jpg
img_alt: Xibalba img
description: |
  Dungeon Crawler
tags:
  - Dungeon Crawler
  - Cooperative Development
  - Unity & C#
---

Temple of Xibalba is a dungeon-crawling, survival horror game created by the prototyping team at <a href="https://gamedev.msu.edu/spartasoft/" target="_blank" rel="noopener noreferrer">Spartasoft Studio @ Michigan State</a>. In this game, you raid a Mayan temple to steal treasure, but traps and monsters lurk around every corner. This game takes inspiration from Lethal Company by <a href="https://x.com/zeekerssrblx?lang=en" target="_blank" rel="noopener noreferrer">Zeekerss</a>. I contributed as a programmer, collaborating with audio engineers, designers, and artists.

<video controls width="600">
  <source src="\assets\xibalba-images\some_gameplay.mp4" type="video/mp4" alt="some gameplay" class="stack gap-10 content"> 
</video>

Although this game is not publicly playable, it represents a significant development effort. Here are some things I worked on for this game.

- Player Controller and movement
- Raycasts
- Inventory
- Monster AI

> Player Controller and movement

I was tasked with handling the player controller, which includes the crouch, camera, sprint, jump, and inventory management. 

Movement was handled via Unity's <a href="https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html" target="_blank" rel="noopener noreferrer">new input system</a>. This system allows developers to map inputs to keywords that can be referenced in scripts. This means that when I map an action, like a jump, to the action map, I can easily manipulate its effect in the player controller script. Here are some of the actions for this game:

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\xibalba-images\input_map.png
        width="300px"
        height="300px"
        alt= "input map";
    />
</a>

This conveniently works for all forms of input, meaning this game can be played using a keyboard or a controller. 

While the crouch and sprint are straightforward in terms of programming, jumping requires a lot more consideration. During the ideation phase, our team decided that platforming would be a key feature as it is a representation of the player's skill in a challenging environment. The temple is the setting, where traps and monsters are the challenge. The jump needs to feel good, which means it needs to be responsive and predictable. I added the <a href="https://www.youtube.com/watch?v=RFix_Kg2Di0" target="_blank" rel="noopener noreferrer">jump buffer and coyote time</a> to the jump to give it these qualities.

<video controls width="600">
  <source src="\assets\xibalba-images\jumping.mp4" type="video/mp4" alt="jumping" class="stack gap-10 content"> 
</video>

> Raycasts

A neat trick I learned in Unity is the raycast. The raycast is useful for visualizing things like camera vision and checking when the player is touching a surface. Instead of using colliders for the player's feet, I used a raycast to check if the player is actively grounded:

<code>

    bool IsGrounded()
    {
      Debug.DrawRay(capsuleCollider.bounds.center, Vector3.down * (capsuleCollider.bounds.extents.y + 0.1f), Color.cyan);

      return Physics.Raycast
      (
        capsuleCollider.bounds.center,
        Vector3.down,
        capsuleCollider.bounds.extents.y + 0.1f
      );
    }
</code>

The debug line draws a raycast pointing downwards within the bounds of the player and colors it cyan. The function will return true if it is touching the ground. I run the <code>IsGrounded</code> function to cast this ray.  

This is a functional use case for the raycast, there is also a use case for when I want to debug, like when visualizing a vector. If I wanted to know exactly where the camera is looking and how far a player can interact from, I can visualize it with a raycast. You'll see in the video below how the raycast (the red line for the camera and blue line for the ground check) appears and why it is useful to have it for things like the camera:  

<video controls width="600">
  <source src="\assets\xibalba-images\raycast_camera.mp4" type="video/mp4" alt="raycast camera" class="stack gap-10 content"> 
</video>

> Inventory

A green light feature we wanted for this game was to have items that the player could store and hold, items being things like podiums and relics. We elected to use a similar system to <a href="https://www.minecraft.net/en-us" target="_blank" rel="noopener noreferrer">Minecraft</a>, where items are added to slots in the HUD when picked up.

To go about this, I decided to have each slot in the HUD have its own <code>ItemSlot</code> script. There would also be an inventory object that keeps track of each <code>ItemSlot</code>. An <code>Inventory</code> script reads a switch input from the player script that changes the active slot in the inventory hotbar.

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\xibalba-images\inv.png
        width="300px"
        height="300px"
        alt="inventory component"
    />
</a>

 Because each <code>ItemSlot</code> is indexed with a number like a list, I can simply increment and decrement the count. When the index reaches a slot, the item is shown in the player's hand. Here is how it looks picking up, switching slots, and dropping items:

<video controls width="600">
  <source src="\assets\xibalba-images\inv_and_items.mp4" type="video/mp4" alt="inventory at work" class="stack gap-10 content"> 
</video>

> Monster AI

This was my first time working with enemy AI in game development. We wanted enemies to be able to patrol and chase the player when in proximity. Our artist presented the model and we moved them using a nav mesh.

The idea is to use nav points to guide enemies around the temple. When near the player, the monster will chase them down, ignoring the nav points. Below, you can see the nav points as blue spheres:


<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\xibalba-images\nav_points.png
        width="300px"
        height="300px"
        alt="nav points"
    />
</a>

A nav mesh surface is created for the temple and each enemy with the <code>patrol_enemy</code> script is linked to it. All the enemies become <code>agents</code>, whose movement is dictated by the rules of the nav mesh and their <code>patrol_enemy</code> script. 

The <code>nav mesh surface</code> object is pictured below, as well as a nav link. Nav links are areas that the enemies should behave differently around, like pits and bridges. These can be set to things like 'jump' or 'stop'.

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\xibalba-images\navnav.png
        width="300px"
        height="300px"
        alt="nav mesh and link"
    />
</a>

> A little fun fact

This game takes inspiration from Mayan culture, which is where the name 'Xibalba' comes from. Xibalba is the name of the underworld in the Mayan mythos and it translates to 'place of fright', which is an apt name since that was a design goal of ours. The monster that chases when you steal the correct relic, 'Ah Puch' embodies that (Well done <a href="https://automatoncreations.com/" target="_blank" rel="noopener noreferrer">Quinten</a>).

<video controls width="600">
  <source src="\assets\xibalba-images\ah_puch.mp4" type="video/mp4" alt="ah puch" class="stack gap-10 content"> 
</video>
