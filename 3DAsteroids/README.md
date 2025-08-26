# 3D Asteroids
![](https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/asteroidsGameplay.gif)

## Brief Game Description
This was a project that I made to learn the basics of Unreal Engine. It is my attempt at recreating the classic asteroids arcade game, in a 3D environment. After I made a basic version I realised that it was too easy, so for an extra challenge I added in the Earth that can also be hit by the falling asteroids, so the player has to protect the Earth from getting hit, while also avoid getting hit themselves. 

---

## _Notable Features_
Some of the more notable features that I made for this is the player, asteroid behaviour, the looping of the arena and the remaining asteroids counter. 

## -Asteroids
Since the asteroids are the central focus of the game, I spent a lot of time tweeking and reworking it. In order to easily seperate each asteroid type, I made them into three different blueprints for each size. When an asteroid is first spawned in, it first gets a random asteroid model from a list. After that they get a downward force applied along with some random force in the other directions. Since each asteroid has gravity and linear/angular damping turned off, the asteroids will continue falling as the same pace endlessly. The final thing that the asteroid does when spawning, is add itself to the asteroid counters list.  

<details>
  <summary>Starting nodes snippet for an asteroid</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/FFffdd4.png?raw=true" width=800/>
</details>

The big and medium sized asteroids have custom events for splitting that get called when shot by the player. The asteroids will spawn two of the next asteroid after destroying itself. The small asteroids only destroys itself. Each asteroid also removes itself from the asteroid counter list, when the Destroyed event gets called. 

<details>
  <summary>Shot and Split node snippet</summary>
  <img src="https://github.com/AxelRK32/Portfolio/blob/main/3DAsteroids/Images/UjhjHJ45.png?raw=true" width=800/>
</details>
