# 3D Asteroids
![](https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/asteroidsGameplay.gif)

## Brief Game Description
This was a project that I made to learn the basics of Unreal Engine. It is my attempt at recreating the classic asteroids arcade game, in a 3D environment. After I made a basic version I realised that it was too easy, so for an extra challenge I added in the Earth that can also be hit by the falling asteroids, so the player has to protect the Earth from getting hit, while also avoid getting hit themselves. 

---

## _Notable Features_
Some of the more notable features that I made for this is the player, asteroid behaviour, the looping of the arena and the remaining asteroids counter. 

## -Asteroids
Since the asteroids are the central focus of the game, I spent a lot of time tweeking and reworking their behaviour. In order to easily seperate each asteroid type, I made them into three different blueprints for each size. When an asteroid is first spawned in, it first gets a random asteroid model from a list. After that they get a downward force applied along with some random force in the other directions. Since each asteroid has gravity and linear/angular damping turned off, the asteroids will continue falling as the same pace endlessly. The final thing that the asteroid does when spawning, is add itself to the asteroid counters list.  

<details>
  <summary>Starting nodes snippet for an asteroid</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/FFffdd4.png?raw=true" width=800/>
</details>

The big and medium sized asteroids have custom events for splitting that get called when shot by the player. The asteroids will spawn two of the next asteroid after destroying itself. The small asteroids only destroys itself. Each asteroid also removes itself from the asteroid counter list, when the Destroyed event gets called. 

<details>
  <summary>Shot and Split node snippet</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/UjhjHJ45.png?raw=true" width=800/>
</details>

## -Looping Arena
Since the game revolves around shooting the asteroids until they are all destroyed, I needed some way to keep the asteroids and the player inside a small area. I took inspiration from the original 2D version where the player and asteroids would loop around to the other side whenever they hit the edge of the gamescreen.  
I created a blueprint with a large box collider. Whenever an actor overlaps the box collider, It simply teleports the actor a set length away from the looping blueprint.
I created a public vector variable to represent the direction and distance of the teleport, so that I can use the same blueprint for every border and only change one value of the vector variable for each wall, floor and ceiling. 

<details>
  <summary>Looping area nodes</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/S43FDgfs.png?raw=true" width=800/>
</details>

After implementing this feature, it became clear that this feature works much better in a 2D game. Seeing asteroids just disappearing and appearing behind you in a 3D environment was confusing for the player. To help with this, I created a blueprint that could spawn a portal/black hole where the asteroid would teleport. It's designed similarly to the looping area with the large box collider. The difference is that it calculates the asteroids direction and spawns in a portal/black hole a bit ahead of their flight path. I then simply place these portal spawner blueprint just before the looping blueprints. 

![](https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/asteroid%20portal.gif)
<details>
  <summary>Portal/Black hole nodes</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/gaGA32f.png?raw=true" width=800/>
</details>
