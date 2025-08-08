# Head On

![](https://github.com/AxelRK32/Portfolio/blob/main/HeadOn/Images/2rH3l%2B.png)

[itch.io page](https://yrgo-game-creator.itch.io/head-on)  
[Trailer](https://www.youtube.com/watch?v=OZKcdXMteiw)

## _A brief game descriptions_

**Head On** is a 2D puzzle platformer game that centers around a robot that no longer has it's head attached and has to carry it in order to see. Traverse long abandoned factories while solving puzzles to reach another robot that still functions. 

## _My contributions to the project_

I worked alongside the other programmers on several of the game's systems, constantly iterating on eachothers code. The parts that I contributed most to was a checkpoint system, debug functions, pause menu, changing controls in settings, code cleaning and adding jucie to the game. 

## - Checkpoint system
![](https://github.com/AxelRK32/Portfolio/blob/main/HeadOn/Images/4fdfgsD.png)

The game is split up into different puzzles that contain a puzzle that the player solves in order to progress. The checkpoints are placed at the start of every factory and they place the player and their head there when the player either dies or resets. Reseting also resets the puzzle it's attached to so you're never stuck from solving the puzzle. 

The way I made the checkpoint system was to teleport the players body and head whenever they choose to reset. Each checkpoint also have a public gameobject where you can insert the parent object for the associated factory/puzzle. When the player chooses to reset, the checkpoint deletes the puzzle in it's current state and spawns in a copy of the puzzle as it was before to replace it, and effectivaly reseting the puzzle fromm the player point of view. 

## - Debug functions
![]()

At the start of development I made several debug functions for the sake of testing the game throughout development. These included things such as NoClip, Teleporting the head to the body, Teleporting the body to the head, Teleporting to the next checkpoint and reseting the entire level. 
