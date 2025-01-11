---
title: Undercooked
publishDate: 2020-03-02 00:00:00
img:  /assets/undercooked-images/izzy_undercooked.jpg
img_alt: Undercooked img
description: |
  Mulitplayer Couch Choas 
tags:
  - Mulitplayer Couch Choas 
  - Cooperative Development
  - Unity & C#
---

> Play <a href="https://haydenhe.itch.io/undercooked" target="_blank" rel="noopener noreferrer">Undercooked</a>

Undercooked is a local multiplayer game created by the prototyping team at <a href="https://gamedev.msu.edu/spartasoft/" target="_blank" rel="noopener noreferrer">Spartasoft Studio @ Michigan State</a>, where I contributed as a programmer. This game takes inspiration from <a href="https://ghosttowngames.com/game/overcooked/" target="_blank" rel="noopener noreferrer">Overcooked by Ghost Town Games</a> in more than just its name. In this two-player game, one player takes on the role of the Server, tasked with cleaning the restaurant and serving guests, while the other player assumes the role of the Rival, whose goal is to create as much mess and havoc as possible. The Server wins by maintaining cleanliness until time runs out, while the Rival wins by causing enough chaos.

<a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/undercooked-images/izzy_undercooked.jpg
        width="300px"
        height="300px"
        alt= "Undercooked Cover";
    />
</a>

> Controls

The controls are viewable on the itch.io page but I will put them here anyway:

- Player 1 (Server) > WASD to move, press E to pick up & serve food, hold E to clean tables & trash
- Player 2 (Rival) > Arrow keys to move, press space to pick up & place trash, hold space to dirty tables

> Motivation

Undercooked was developed as a prototype to spark discussion about expanding on this genre within the studio. I worked alongside an audio engineer, artists, and fellow programmers, gaining valuable experience in communicating technical concepts to diverse team members. Developing a game within a limited time frame accelerated my learning across all facets of game development. 

Here is what I contributed to this project.

- Player Input 
- Interacting with items
- Switching Table states via Coroutines

> Player Input

Unity's <a href="https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html" target="_blank" rel="noopener noreferrer">new input system</a> aims to be modern and flexible. We leveraged it for this project as we believed it would work best for the two-player experience whether on keyboard or controller. 

This system allowed me to assign buttons to keywords that can be referenced in an input script. For example, I wanted each player to be able to interact with tables and trashcans, so I'll map the "<code>Interact</code>" keyword in the <code>Actions</code> table:

<a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/undercooked-images/input_actions.jpg
        width="300px"
        height="300px"
        alt= "input actions";
    />
</a>

Then to reference them in the script, I put the code below into the <code>Start</code> function of the <code>PlayerInputs</code> script. This allows <code>HandleInteract</code> to be called each time the interact button is pressed:

</code>


      var interactAction = playerActions.FindAction("Interact");
      interactAction.started += HandleInteractStarted;
      interactAction.performed += HandleInteractPerformed;
</code>

This system is really convenient as it removes the need for constant polling in Update loops. You can also write code once that works for all sources of input, sources being different controller types and keyboards. By using the name of the input in the action table, things like camera movement and directional input can be converted into vectors.

> Interacting with items

Several objects in Undercooked are interactable, including trash, food, tables, and trashcans. I designed an <code>Interactable</code> base class to handle common functionality, with specific behavior extended through <a href="https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)" target="_blank" rel="noopener noreferrer">inheritance</a>.

The most basic functionality of this <code>Interactable</code> class is to have an object be picked up and dropped. The trigger functions that would be called when a player comes in contact with the interactable, for example. This function is <code>virtual</code>, so sub classes can expand on it's functionality via <a href="https://en.wikipedia.org/wiki/Polymorphism_(computer_science)" target="_blank" rel="noopener noreferrer">polymorphism</a>. 

<code>


      {...}
      public virtual void OnTriggerStay(Collider other)
      {
        if (ContactDetected) return;

        detectedPlayer = other.transform.parent.gameObject;

        if (detectedPlayer != null && detectedPlayer.CompareTag("Player"))
        {
          ContactDetected = true;
        }
      }
</code>

An <code>InteractableManager</code> keeps track of all interactables in the scene. Each time input is detected, the manager determines which object the player is in contact with and triggers the corresponding action. 

 <a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/undercooked-images/interactable_manager.jpg
        width="300px"
        height="300px"
        alt= "interactable manager";
    />
</a>
 
 This may seem redundant and wasteful at first, but there aren't many interactables in a scene at once and items are dynamically added and removed from memory as they are spawned or destroyed:

<code>


      // This method is to be called when the interact action is held down
      public void HoldInteract(GameObject player)
      {
        // Ensure all interactables are added before processing
        // Process the queued interactables at the end of each frame (or when needed)
        ProcessAdditions();  

        foreach (Interactable interactable in interactablesList)
        {
          interactable.Hold(player);
        }

        // Safely remove items after iteration
        // Call this after processing all interactions
        ProcessRemovals();  

      }
</code>

This makes it so all interactables in a scene are accounted for. As items are spawned in, like trash coming out of a trashcan, it's added to the <code>InteractableManager</code>. When it is destroyed, it is removed. If the prototype were to be expanded on, items could be picked as usual and then a different input would use the held item, like a broom or laser gun for example. This system allowed me to add new items into the game very quickly as well as effectively handle possible bugs, like sticky trash:

<video controls width="600">
  <source src="/assets/undercooked-images/trash_attack.mp4" type="video/mp4" alt="trash attack" class="stack gap-10 content"> 
</video>

> Switching Table states via Coroutines

As mentioned before, tables are interactable. By holding down the interact button, Servers can clean a table and Rivals can dirty them up. The Server can also give color-coded food to tables of the same color (ie blue food goes to blue tables). I needed customers to request different colored food every few seconds. To achieve this, I used Unity <a href="https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Coroutine.html" target="_blank" rel="noopener noreferrer">Coroutines</a>. A coroutine is a method that allows for delays between function calls in scripts. Here is how I used them:

<code>


      void Start()
      {
        switchingCoroutine = StartCoroutine(SwitchFoodState());
      }

      IEnumerator SwitchFoodState()
      {
        while (true)
        {
          // Switch to a random food state
          foodWanted = GetRandomFoodWanted();

          // show floor color for this food
          if (floor != null) floor.ChangeFloor(foodWanted);

          // Wait for a random amount of time 
          float randomWaitTime = Random.Range(shortestWait, longestWait);

          yield return new WaitForSeconds(randomWaitTime);
        }
      }
</code>

The <code>Start</code> function starts the coroutine for the <code>SwitchFoodState</code> function. The return value is the <code>IEnumerator</code>. Since the while loop checks for the <code>true</code> condition, it will run indefinitely. The coroutine will yield the <code>IEnumerator</code> value at the random wait time BEFORE returning to the top of the while loop. I can apply the wait time I want here and have the coroutine run as long as the table runs <code>Start</code>. This is what happens when I set the wait time to one second instead of a random number between 20 and 30. We got some picky customers...

<video controls width="600">
  <source src="/assets/undercooked-images/switching_food.mp4" type="video/mp4" alt="picky customers" class="stack gap-10 content"> 
</video>