---
title: JuJu
publishDate: 2020-03-02 00:00:00
img:  /assets/juju-images/juju.jpg
img_alt: JuJu img
description: |
  Puzzle Platformer
tags:
  - Puzzle Platformer
  - Solo Development
  - Unity & C#
---

> Play <a href="https://gizmo435.itch.io/juju" target="_blank" rel="noopener noreferrer">JuJu</a>

After recreating Celeste Classic (you can play that <a href="https://gizmo435.itch.io/mos-celeste" target="_blank" rel="noopener noreferrer">here</a>), I pondered where I can go with the concept. How could I expand on  Celeste's platforming? I did the first thing that came to my mind: Give Celeste a gun. 

<a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/juju-images/juju.jpg
        width="300px"
        height="300px"
        alt= "JuJu Cover";
    />
</a>

> Art and Credit

I went to YouTube for help with art and physics. Thanks to <a href="https://www.youtube.com/watch?v=STyY26a_dPY" target="_blank" rel="noopener noreferrer">Mix and Jam</a> for all of that. 

I used this tutorial to fine-tune my physics as well as acquire a tile palette. I also discovered <a href="https://dotween.demigiant.com/index.php" target="_blank" rel="noopener noreferrer">DoTween</a> through this video, which handles things like screen shake. 

> Motivation and Design

From a design perspective, I aimed for a game similar to Celeste, but more involved in its platforming. I chose the popular and effective route: I added a puzzle element. Players must consider both the limited bullets and their order. The gun in JuJu is not for combat, but for movement, effectively replacing the dash mechanic that Celeste Classic uses. This short puzzle-platformer was a class project I took on early in my game development journey. It is very simple and has a lot of rough edges (Take a look at JuJu's misaligned left eye, Unity made an attempt to 'perfect pixel' my art), but it nonetheless challenged me in several aspects of solo development. Here are key areas I’d like to highlight:

• Game Feel and Gravity

• UI display using a Unity List

• Bullet Types

• Lessons Learned

> Game Feel and Gravity  

Game feel makes Celeste what it is, and I sought to replicate that. JuJu needed responsive, dynamic jumps with controlled gravity depending on vertical movement. I applied forces to the jump to simulate this:

<code>      
   

    if (rigid.velocity.y < 0)
    {
      //going down
      rigid.velocity += Vector2.up * Physics2D.gravity.y * (fallSpeed - 1) * Time.deltaTime;
    }
    else if (rigid.velocity.y > 0)
    {
      //going up
      rigid.velocity += Vector2.up * Physics2D.gravity.y * (lowMult - 1) * Time.deltaTime;
    }
</code>

This code adjusts vertical velocity: as JuJu ascends, upward force is reduced for a more floaty peak; as JuJu descends, gravity increases fall speed. Although both conditions are similar, the logic successfully emulates nuanced gravity control.

I admit that this can be written better, both <code>if</code> blocks look similar, but the idea is sound and readily applies the idea of gravity and velocity in Unity.

> UI display using a Unity List

If you have been playing the game (thanks for that), you may have noticed how ammo is displayed. As you shoot bullets, your ammo is used up in order from left to right (this picture's order is: white, green, red). I used a Unity List to keep track of ammo and the UI Canvas to display the bullets on your screen. 

<a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/juju-images/bullets.jpg
        width="300px"
        height="300px"
        alt= "some bullets";
    />
</a>

Two scripts manage this: <code>Ammo</code> tracks bullets, while <code>AmmoDisplay</code> updates the UI. Something I learned from this project was to avoid having a small amount of dense scripts. It is better to have smaller scripts with simpler responsibilities. That way, you can achieve modularity and readability. Here’s the <code>GetNextBullet</code> method from Ammo:

<code>      
   
    // Some code is removed for readability {...} 
    public class Ammo : MonoBehaviour
    {
      {...}

      public GameObject GetNextBullet()
      {
        if (AmmoList.Count > 0)
        {
          GameObject nextBullet = AmmoList[0];
          AmmoList.RemoveAt(0);

          ammo_display = GameObject.FindGameObjectWithTag("Ammo Display").GetComponent<AmmoDisplay>();
          ammo_display.DropIcon();

          return nextBullet;
        }
        else
        {
          empty = true;
          return null; 
        }
      }
    }
</code>

Each time a bullet is shot, <code>GetNextBullet</code> retrieves the next available one. <code>AmmoList</code> then removes that bullet from the list and <code>AmmoDisplay</code> on the canvas removes it from the display. This bullet is then shot. Unity supports this kind of "order display" with its <a href="https://docs.unity3d.com/6000.0/Documentation/ScriptReference/Grid.html" target="_blank" rel="noopener noreferrer">Grid Layout Interface</a>. It is very convenient to have the bullets move over as they're being used without having to write the code for it. 

The code below shows the <code>DropIcon</code> function used in <code>AmmoDisplay</code>. It counts the bullets and removes the icon at the front of the list (the bullet being fired):
<code>

    public class AmmoDisplay : MonoBehaviour
    {
        {...}
        public void DropIcon()
          {
            if (bulletCount == Icons.Count) // if the first bullet
            {
              //Debug.Log("Skipping first icon");
            }
            else 
            {
              GameObject nextIcon = Icons[0];
              nextIcon.SetActive(false);
              Icons.RemoveAt(0);
            }
            bulletCount -= 1;
          }
    }
</code>

> Bullets

JuJu features four bullet types: White, Green, Blue, and Red.

White propels JuJu using a force opposite to the shot direction:

<code>

    Vector2 forceDirection = (BulletDir * -1).normalized; // direction to move in, opposite of bullet
    JuJusBody.AddForce(forceDirection * ShotGunForce, ForceMode2D.Impulse);
</code>

Green teleports JuJu to its position after a duration based on button-hold time. Green walls signify areas where teleporting is necessary:

<code>


    public void SetLifetimeBasedOnHoldDuration(float holdDuration)
      {
        lifeTime = baseLifetime + holdDuration;
        if (lifeTime > maxWaitTime) 
        {
          lifeTime = maxWaitTime;
        }

        Debug.Log(lifeTime);
        Invoke("Teleport", lifeTime);
      }
</code>

This was tricky to implement. I measured button-press duration in an <code>Update</code> method instead of using <a href="https://docs.unity3d.com/Packages/com.unity.inputsystem@1.11/manual/index.html" target="_blank" rel="noopener noreferrer">Unity's Input system</a>, which is a little inefficient in hindsight. Here’s a video showing how to clear the Green02 level:

<video controls width="600">
  <source src="/assets/juju-images/green02.mp4" type="video/mp4" alt="green02 tutorial" class="stack gap-10 content"> 
</video>

The first green bullet shot was preceded by a long button press. That's why it took longer to teleport to it. This function uses <code>Invoke</code> to call teleport after the <code>lifeTime</code> duration is .  

> Lessons Learned

As a programmer, I'm also looking for ways to make my code more efficient, since code quality is as important as functionality. Efficient coding practices mean less work for me in the long run and an easier time for any prospective teammates. My bullet scripts were redundant and could have benefited from <a href="https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)" target="_blank" rel="noopener noreferrer">inheritance</a>:

<code>

    public class Bullet : MonoBehaviour
    {
      public virtual Vector2 ShootBullet
      {
        // get some mouse position
        // apply velociy too bullet
      }
    }

    public class WhiteBullet : Bullet
    {
      public override Vector2 ShootBullet
      {
        // apply force to JuJu
      }
    }
</code>

Using inheritance like this would have been the way to go. Making a bullet super class with each color as a subclass would mean I could write less code and make it easier to create new functionality for different bullets. 

Overall the project is a stepping stone, as it was one of the first times I truly challenged myself as a game developer. Reflecting on this project reminds me of the importance of foundational practices and that sharing early work is valuable as it shows growth.

Thank you for enjoying JuJu. Who knows, I might revisit it.