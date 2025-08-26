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

## - Inventory/Shop system
![]()

The way I made the inventory system was to make it the inventory into singleton so that I could reference it directly when I needed to add or remove an item. I also made a scriptable object to represent each item.

<details>
  <summary>Item object</summary>

  ```cs
using UnityEngine;

[CreateAssetMenu(fileName = "Item", menuName = "ScriptableObjects/Item", order = 1)]
public class Item : ScriptableObject
{
    public string itemName;
    public string itemDescription;
    public Sprite itemIcon;
    public int price;
    public bool isBettable = true;
    public bool isUseable = true;
    public string onPurchaseCommand;
}
  ```
</details>
<details>
  <summary>Player Inventory</summary>

  ```cs
using AEssentials;
using System.Collections.Generic;
using UnityEngine.SceneManagement;

public class PlayerInventory : Singleton<PlayerInventory>
{
    public List<Item> Inventory
    {
        get { return inventory; }
    }

    private List<Item> inventory = new();
    int inventorySlotCount;

    public int InventorySlotCount
    {
        get { return inventorySlotCount; }
        set { inventorySlotCount = value; }
    }

    void Start()
    {
        InitializeInventory();
    }

    public void AddItem(Item itemToAdd)
    {
        inventory.Add(itemToAdd);
    }

    public void RemoveItem(Item itemToRemove)
    {
        inventory.Remove(itemToRemove);
    }

    public void ClearInventory()
    {
        inventory.Clear();
    }

    void InitializeInventory()
    {
        if (SceneManager.GetActiveScene().name != "Menu" && PersistentPlayerData.Instance.HeldItems != null)
        {
            foreach (Item item in PersistentPlayerData.Instance.HeldItems)
            {
                AddItem(item);
            }
        }
    }
}
  ```
</details>

In order to keep track of the players inventory accross each scene, I also made a persistent singleton class for the players inventory along with other data. That class is also tied to the saving system. Whenever you load a game, it starts by getting the existing data from the playerpref. That data is then used to set the data in PersistentPlayerData. After that, every other script just reference the persistent player data, instead of constantly loading data from playerprefs. 

<details>
  <summary>Persistent Player Data</summary>

  ```cs
using System.Collections.Generic;
using System.Linq;
using AEssentials;
using Managers;
using UnityEngine;

[DefaultExecutionOrder(-1)]
public class PersistentPlayerData : SingletonPersistent<PersistentPlayerData>
{
    public int Cash { get; private set; }
    public int Debt { get; private set; }
    public int ProgressIndex { get; private set; }
    public Transform DoorExit { get; set; }
    public bool CanCheat { get; set; }
    public bool HasDisplayedCheatHint { get; set; }
    public bool HasDisplayedItemHint { get; set; }
    public List<Item> HeldItems { get; set; }

    public void SetPlayerData(PlayerSaveData playerSaveData)
    {
        Cash = playerSaveData.money;
        Debt = playerSaveData.debt;
        ProgressIndex = playerSaveData.progress;
        CanCheat = playerSaveData.canCheat;
        HasDisplayedCheatHint = playerSaveData.hasDisplayedCheatHint;
        HasDisplayedItemHint = playerSaveData.hasDisplayedCheatHint;
        HeldItems = playerSaveData.items?.Select(ItemDatabase.GetItem).Where(x => x != null).ToList() ?? new List<Item>();
    }
}
  ```
</details>
