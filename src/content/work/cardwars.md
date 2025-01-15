---
title: Card Wars II
publishDate: 2020-03-02 00:00:00
img:  /assets/cardwars-images/Screenshot 2024-10-14 225429.png
img_alt: Card Wars img
description: |
  Networked Card Battler
tags:
  - Networked Card Battler
  - Solo Development
  - Unity & C#
---

This is a passion project I started in the summer of 2025. As a kid, one of my hobbies was making card games and board games. One of the games I made, actually remade, was <a href="https://adventuretime.fandom.com/wiki/Card_Wars_(game)" target="_blank" rel="noopener noreferrer">Card Wars from Adventure Time</a>. I loved that episode so much that I made my own, slightly tweaked version of the game with index cards. I played this game a lot with my brother, changed the rules, and remade the game from scratch many times. 

It was really annoying keeping track of numbers, putting the right chips, keeping the board organized, and acutally making cards. I wished I had something that could solve all of these problems. Now that I'm pretty good at Unity, I can solve all of these problems and make the game even better. 

Of course, this is just a passion project. It's hardly like the orignal at all, as I only started making my own because I thought the orignal game was very flawed in its gameplay. I only intend to play this with firends and family and don't intend to monetize it in anyway. The IP is not mine, it belongs to Warner Bros. Entertainment. 

How I change the game is still being worked on. This game has had a couple different versions and I tend to brainstorm it during development. If you are curious here is a design document describing my thought processes, I will also put the GitHub repo here if you want to look at any of the assets and/or scripts:

- <a href="https://github.com/GizmoAhmed/Mos-CardWars2" target="_blank" rel="noopener noreferrer">Mos-CardWars2 | GitHub Repo</a>.

- <a href="https://docs.google.com/document/d/1J2F-fg1T6fxYczbac2s4jPrRyimjqx_EszhiFlSmRzo/edit?usp=sharing" target="_blank" rel="noopener noreferrer">Card Wars II: DevLog</a>.

Here are some the areas I would like to disucss. Keep in mind I'm still actively making this game, so expect changes in the future.

- Mirror & Networking
- Card Class Heirarchy (Inheritance)

### Mirror & Networking

I want the game to be a two player game that can be played on two seperate machines. Networking is a diificult thing to do and there are several different tools to do it with. I decided to use <a href="https://mirror-networking.com/" target="_blank" rel="noopener noreferrer">Mirror Networking</a>, for this project. Among other things Mirror allows you to clone your editor and run to seperate clients on a single machine, which is invaluable

<video controls width="600">
  <source src="/assets/undercooked-images/trash_attack.mp4" type="video/mp4" alt="trash attack" class="stack gap-10 content"> 
</video>

Essentially how it works is that every object in a Unity scene is a component of the server. THis means it is no longer monobehaviour but a sub class of <code>NetworkBehaviour</code>. 

<code>

    public class GameManager : NetworkBehaviour

</code>

If there are two players/clients, how can certain objects and rules be applied to both or one at a time. There is a network manager for that prupose, it keeps track of all of the "spawnable" objects in a scene. 

// picture of these spawnable objects

<a>
    <img
        class="wrapper stack gap-10 content"
        src= /assets/cardwars-images/interview3.mov
        width="300px"
        height="300px"
    />
</a>

While things like the player area, enemy area, decks, and lands are objects that server provides to clients, indiviudal clients spawn cards from their own deck. In order to handle how cards are spawned I have a player script for each client. This script holds things like handling card draws and placements, stats like health and money, as well as deck sizes. 

As you can see below drawing a card and placing it is something both clients will see, but only one client controls it:

// video of just that

<video controls width="600">
  <source src="/assets/undercooked-images/trash_attack.mp4" type="video/mp4" alt="trash attack" class="stack gap-10 content"> 
</video>

I'm able to distinguish between the behabiours of the actor and observer by using Mirror's Command and Remote Proceudre Calls.

> Command vs Remote Procedure Calls

Commands and Remote Procedure Calls (RPC's) are function headers that describe how a function is run: Commands are run on the server and RPC's are on each individual client on the server. Look at the code for dropping a card and then showing the card: 

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
        // place card hand area THIS players hand
      }
      else
      {
        // find opposing hand area and place card over there
      }
    }
</code>

As a card is being dropped, the deck script calls the Command <code>CmdDropCard</code>, this code runs on the server. And since the server can tell clients what to do, it can make an RPC whcih runs function code simultanesoly on both clients. 

> How <code>IsOwned</Code> works

IsOwned is a boolean that asks whther a specific client is the one running function code. So when player 1 drops a card, the RPC call has isOwned become true and Player 2 has isOwned become false. This is a very easy way for me to essentially mirror the behaviour of certain functions. 

So when Player 1 runs the and IsOwned is true, I know that it is the active client, then I can put cards on the Player 1 side. Player 2 returns false, so when Player 1 places a card, it places it on Player 2's, Player 1 hand. It sounds confusing but it is essentially a mirror.

This can also be used with coroutines, which is how I handle cards attacking each other during the attack phase of play:

<video controls width="600">
  <source src="/assets/undercooked-images/trash_attack.mp4" type="video/mp4" alt="trash attack" class="stack gap-10 content"> 
</video>

> Game Manager

I can delegate game state and clients in a game manager script. Unity has a another function header called <code>[Server]</code>. This is good for a game manager as clients can't touch any of the code run in this function and this function can run code on one or both clients. For example, I can have the game manager keep track of when players hit the ready button:

<video controls width="600">
  <source src="/assets/undercooked-images/trash_attack.mp4" type="video/mp4" alt="trash attack" class="stack gap-10 content"> 
</video>

Here is the code for that:

<code>




    public class GameManager : NetworkBehaviour {
      [Server]
	  public void PlayerReady(NetworkConnectionToClient conn)
	  {
		if (currentPhase == GamePhase.ChooseLand)
		{
			if (!readyPlayers.Contains(conn))
			{
				readyPlayers.Add(conn);

				Player thisPlayer = conn.identity.GetComponent<Player>();

				thisPlayer.RpcEnablePlayer(false);

				CheckAllPlayersReady();
			}
		}
      
      [Server]
      private void CheckAllPlayersReady()
      {
      if (readyPlayers.Count >= 2)
      {
        readyPlayers.Clear();
        ChangePhase(GamePhase.ChooseLand, GamePhase.SetUp);
      }
      }
    }
</code>

Each time a client hits ready, the <code>PlayerReady</code> function is run and makes sure the player is in the GameManager players list. It tehn checks if there are exactly two players on the server. The spawnable objects only appear if there are two clients anyway.

