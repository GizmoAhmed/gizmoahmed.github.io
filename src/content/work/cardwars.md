---
title: Card Wars II
publishDate: 2020-03-02 00:00:00
img:  \assets\cardwars-images\cardback.png
img_alt: Card Wars img
description: |
  Networked Card Battler
tags:
  - Networked Card Battler
  - Solo Development
  - Unity & C#
---

> Disclaimer

This is a passion project about making a game I love better. I only intend to play this game with friends and family, with no commercial intent or plans for monetization. All intellectual property rights belong to Warner Bros. Entertainment. 

> Motivation

I began this project in the summer of 2024. As a kid, one of my hobbies was making card and board games. Some of these games were original ideas, while others were remakes of existing games. One of the games I remade was <a href="https://adventuretime.fandom.com/wiki/Card_Wars_(game)" target="_blank" rel="noopener noreferrer">Card Wars from Adventure Time</a>. I loved that episode so much that I made my own version of the game using index cards.

Changing numbers, counting chips, and making new features was tedious when using paper and markers. Unity can handle all of these challenges and more.  

My design process is ongoing, and the game has gone through several iterations. I have my development log and GitHub repository linked below if you are ever curious about how it's currently going.

- <a href="https://github.com/GizmoAhmed/Mos-CardWars2" target="_blank" rel="noopener noreferrer">Mos-CardWars2 (GitHub Repo)</a>.

- <a href="https://docs.google.com/document/d/1J2F-fg1T6fxYczbac2s4jPrRyimjqx_EszhiFlSmRzo/edit?usp=sharing" target="_blank" rel="noopener noreferrer">Card Wars II: DevLog</a>.

This project has been a learning experience so far, especially in networking concepts using Mirror. While I continue to develop the game, I will be sure to update the repository, development log, and this project page. Stay tuned.

> Why Mirror

This game is two-player and needs to be playable on separate machines. I decided to use <a href="https://mirror-networking.com/" target="_blank" rel="noopener noreferrer">Mirror</a> for this project, as it allows you to, among other things, clone your editor using its "ParallelSync" feature as well as run two clients simultaneously on a single machine. 

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\cardwars-images\clone.png
        width="300px"
        height="300px"
        alt= "cloned editors";
    />
</a>

### Networking

In Mirror, every spawnable object in a Unity scene is a server component, subclassing <code>NetworkBehaviour</code> instead of the typical <code>MonoBehaviour</code>:
<code>

    public class GameManager : NetworkBehaviour

</code>

To synchronize objects between clients and the server, a <code>NetworkManager</code> keeps track of all spawnable objects, each with a <code>NetworkIdentity</code> component. 

Here are some of these spawnable objects:
<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\cardwars-images\network_manager.png
        width="300px"
        height="300px"
        alt="network manager"
    />
</a>

<a>
    <img
        class="wrapper stack gap-10 content"
        src= \assets\cardwars-images\spawnables.png
        width="300px"
        height="300px"
        alt="spawnables"
    />
</a>

The server synchronizes game objects like player areas, decks, and lands. Each <code>Player</code> script handles actions like drawing and placing cards, health, money, and deck size. 

I can essentially make the same game on two different clients. Here's a video showing how card drawing and placement are synchronized:

<video controls width="600">
  <source src="\assets\cardwars-images\cards.mp4" type="video/mp4" alt="placing cards" class="stack gap-10 content"> 
</video>

Gameplay can be networked between clients using Mirror's function headers: <code>[Command]</code> and <code>[ClientRpc]</code>. 

> Command vs Remote Procedure Calls (RPCs)

<code>[Command]</code> is run on the server, while a <code>[ClientRpc]</code> is run on each client. 

For example, each client can draw and move their own cards. When writing a function for drawing and dropping cards, there should be a <code>[Command]</code> call within that function that will ask the server to handle the card. A <code>[ClientRpc]</code>, which can only be run on the server, can then decide how the card is displayed for both clients.

Here is this example at work: 

<code>



    [Command]
    public void CmdDropCard(GameObject card, CardState state, GameObject land)
    {
      RpcHandleCard(card, state, land);
    }

    [ClientRpc]
    public void RpcHandleCard(GameObject card, CardState state, GameObject land)
    {
      if (isOwned)
      {
        // place card hand area THIS player's hand
      }
      else
      {
        // find opposing hand area and place card over there
      }
    }
</code>

As a card is being dropped, the deck script calls <code>CmdDropCard</code>. This code runs on the server and since the server can tell clients what to do, it can make a <code>[ClientRpc]</code> which runs function code simultaneously on both clients. 

> How <code>isOwned</Code> works

You will notice in the above code the use of the boolean, <code>isOwned</code>. <code>isOwned</code> identifies whether the current client is running the code within a function or if an object with the <code>NetworkIdentity</code> was spawned by a certain client. For example, when Player 1 drops a card, <code>isOwned</code> is true for Player 1 and false for Player 2. If Player 1 spawned a card, IsOwned is true for that card for Player 1 and false for Player 2. This allows mirroring functionality for both clients, by allowing me to code two different actions in one function.

Since <code>RpcBattle</code> is run on both clients, I need to discern which client owns the <code>attackingCard</code>: 

<code>

    [ClientRpc]
    public void RpcBattle(CreatureCard attackingCard, CreatureCard defendingCard) 
    {
      {...}
      if (attackingCard.isOwned) // if true, the local player is attacking, if false, they're defending
      {
        Player attackingPlayer = NetworkClient.localPlayer.GetComponent<Player>();
        attackingPlayer.CmdChangeStats(attackingPlayer.Score + attackingCard.AttackStat, "score");
      }
    }
</code>

Only the player that owns the attacking card will have their stats changed. This makes two different functions out of one using a boolean.

> Language Integrated Query (LINQ)

LNQ is how C# organizes and manipulates data, similar to list comprehension in other languages. In this game, creatures attack from left to right across the game board. In my <code>Player</code> script, I want each card to be put into a list and ordered via land number. That is easy enough, simply make a key-value pair. But getting only <code>Creature</code> cards that are <code>Placed</code> and <code>isOwned</code> is a little harder. Here, I use LNQ to do just that:

<code>

    var battleReadyCards = FindObjectsOfType<CreatureCard>()
          .Where(creatureCard => creatureCard.isOwned && creatureCard.currentState == CardState.Placed)
          .Select(creatureCard => {
            string landName = creatureCard.MyLand.name;
            int landNumber = int.Parse(landName.Substring(landName.Length - 1));
            return new KeyValuePair<GameObject, int>(creatureCard.gameObject, landNumber);
          } ).ToList();
</code>

Using <code>Select</code>, I can add key-value pairs to the </code>battleReadyCards<code> list. I then use <code>Where</code> to pick creature cards that are <code>isOwned</code> and <code>Placed</code>. I can iterate over this list using coroutines to create the battle phase: 

<video controls width="600">
  <source src="\assets\cardwars-images\battle.mp4" type="video/mp4" alt="card battle phase" class="stack gap-10 content"> 
</video>

