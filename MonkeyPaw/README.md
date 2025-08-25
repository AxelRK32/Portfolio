# Monkey Paw
![](https://github.com/AxelRK32/Portfolio/blob/main/MonkeyPaw/Images/fyVF887n.png)

[itch.io page](https://yrgo-game-creator.itch.io/monkey-paw)  
[Trailer](https://www.youtube.com/watch?v=RkNo9P-4Dn4)

## _A brief game description_

**Monkey paw** is a 3D first person story game that is about a monkey's downward spiral involving debt and gambling. Talk with the other monkeys during the day to try and scrape together some money. At night, head down to the local gambling den to try and win enough money to repay your debt. Over the course of a week, you get more and more desperate and resort to even more reprehensible acts. 

## _My contributions to the project_
I worked on this game alongside three other programmers. We each got some functions and areas to focus on, but would on occasion iterate on each others code to ensure that all the systems worked together. I was assigned to work on a save system, multiple UI elemants like inventory, shop systems and a pause menu, a day/night transition, implementing animations and sound effects, along with some more general juice in different parts of the game. 

## - Save System/Day Night transition
The save system I made is tied to the day/night transition that I also made. The way we defined the progress of the game was to assign each day and night it's own scene so that each of them could be changed without affecting the other days. The day/night transition that I made fades the screen and then loads the next scene in the build index.  
After discussing saving with the rest of the group, we decided it would make the most sense to save after each day/night. In order to not fill the save file with unnecessary data, we always kept the players starting position at his home door, so I didn't have to save the players position and rotation.  
The only data being saved is: current money and debt, every item in the players inventory, some bool variables that check if the player has unlocked certain things, and a progress variable as an integer that keeps track of the scene build index. This data is converted into json which is then saved using PlayerPrefs.

<details>
  <summary>Save Data</summary>
  
  ```csharp
[Serializable]
    public class PlayerSaveData
    {
        public PlayerSaveData(int progress, int debt, int money, List<string> items, bool canCheat,
            bool hasDisplayedCheatHint, bool hasDisplayedItemHint)
        {
            this.progress = progress;
            this.debt = debt;
            this.money = money;
            this.items = items;
            this.canCheat = canCheat;
            this.hasDisplayedCheatHint = hasDisplayedCheatHint;
            this.hasDisplayedItemHint = hasDisplayedItemHint;
        }

        public int progress;
        public int debt;
        public int money;
        public List<string> items;
        public bool canCheat;
        public bool hasDisplayedCheatHint;
        public bool hasDisplayedItemHint;
    }
  ```
</details>
