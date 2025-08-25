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

  <details>
  <summary>Click to view Checkpoint Code</summary>

  ```csharp
  using UnityEngine;

  public class Checkpoint : MonoBehaviour
  {
    bool activated = false;
    public bool Activated { get { return activated; } }
    [SerializeField] Transform spawnPoint;
    public Vector2 SpawnPoint { get { return spawnPoint.position; } }
    GameObject switchObject;
    [SerializeField] GameObject puzzleParent;
    public GameObject PuzzleParent { get { return puzzleParent; } }
    GameObject puzzleCopy;
    SpriteRenderer switchRenderer;
    [SerializeField] Sprite onSprite;
    [SerializeField] Sprite offSprite;
    CheckpointManager manager;

    void Start()
    {
        manager = CheckpointManager.Instance;
        if (transform.Find("Sprite") != null)
        {
            switchObject = transform.Find("Sprite").gameObject;
            switchRenderer = switchObject.GetComponent<SpriteRenderer>();
            switchRenderer.sprite = offSprite;
        }
        if (puzzleParent != null)
        {
            puzzleCopy = Instantiate(puzzleParent, puzzleParent.transform.parent);
            puzzleCopy.SetActive(false);
        }
    }

    public void EnableCheckpoint()
    {
        activated = true;
        manager.SwitchCurrentCheckpoint(this);
    }
    public void DisableCheckpoint()
    {
        activated = false;
        if (GetComponentInChildren<GrabRelay>() != null)
            GetComponentInChildren<GrabRelay>().enabled = true;
        if (switchRenderer != null)
            switchRenderer.sprite = offSprite;
    }

    public void CheckpointReset()
    {
        if (puzzleParent != null)
        {
            Destroy(puzzleParent);
            puzzleParent = Instantiate(puzzleCopy, puzzleCopy.transform.parent);
            puzzleParent.SetActive(true);
        }
        if (switchRenderer)
        {
            if (switchRenderer.sprite == offSprite)
            {
                EnableSprite();
                if (GetComponentInChildren<GrabRelay>() != null)
                    GetComponentInChildren<GrabRelay>().enabled = false;
            }
        }
    }
    public void EnableSprite()
    {
        if (switchRenderer != null)
            switchRenderer.sprite = onSprite;
        if (!activated)
            EnableCheckpoint();
    }

    private void OnTriggerEnter2D(Collider2D other)
    {
        if (other.CompareTag("Player"))
        {
            EnableCheckpoint();
        }
    }

    void OnDrawGizmosSelected()
    {
        Gizmos.color = Color.yellow;
        Gizmos.DrawWireSphere(spawnPoint.position, 0.25f);
    }
}
  ```
</details>

<details>
  <summary>Click to view CheckpointManager Code</summary>
  
  ```csharp
  using UnityEngine;

  public class CheckpointManager : MonoBehaviour
  {
    public static CheckpointManager Instance { get; private set; }

    [SerializeField] Rigidbody2D playerBody;
    [SerializeField] Throwable playerHead;

    Checkpoint currentCheckpoint;

    private void Awake()
    {
        Instance = this;
    }

    private void Update()
    {
        if (Input.GetKeyDown(KeyCode.R))
        {
            RespawnPlayer();
        }
    }

    public void SwitchCurrentCheckpoint(Checkpoint checkpoint)
    {
        if (currentCheckpoint != null && currentCheckpoint != checkpoint)
        {
            currentCheckpoint.DisableCheckpoint();
        }
        currentCheckpoint = checkpoint;
    }

    public void TeleportPlayerToCheckpoint()
    {
        if (currentCheckpoint != null)
        {
            playerBody.transform.position = currentCheckpoint.SpawnPoint;
            playerHead.transform.position = currentCheckpoint.SpawnPoint;
        }
        playerBody.velocity = Vector2.zero;
        playerHead.Rigidbody.velocity = Vector2.zero;
    }

    public Checkpoint GetCurrentCheckpoint()
    {
        return currentCheckpoint;
    }

    public void RespawnPlayer()
    {
        if (!currentCheckpoint) return;
        LedgeGrabAnimator.Instance.ForceStopAnimation();
        PlayerGrabbing.Instance.SetHeldItem(null);
        TeleportPlayerToCheckpoint();
        currentCheckpoint.CheckpointReset();
    }
}
  ```
</details>

## - Debug functions
![](https://github.com/AxelRK32/Portfolio/blob/main/HeadOn/Images/Rjr756Ju.png)

At the start of development I made several debug functions for the sake of testing the game throughout development. These included things such as NoClip, Teleporting the head to the body, Teleporting the body to the head, Teleporting between checkpoints and reseting the entire level. Each debug function is tied to a specific number or key combination on the keyboard for easy access, but you first need to input a code to enable debug mode, so the player doesn't have access to the debug functions.

<details>
  <summary>Debug Code</summary>

  ```csharp
using System;
using System.Collections;
using System.Collections.Generic;
using TMPro;
using UnityEngine;
using UnityEngine.SceneManagement;

public class DebugFunctions : MonoBehaviour
{
    public GameObject PlayerBody;
    public GameObject PlayerHead;
    [Header("NoClip Movement")]
    public float NoClipMoveSpeed = 5;
    public bool NoClipEnabled = false;
    Rigidbody2D playerRB;
    Rigidbody2D[] armsRB;
    PlayerMovement playerMovement;
    PlayerGrabbing playerGrabbing;
    CheckpointManager checkpointManager;
    Collider2D[] colliders;
    public List<Checkpoint> allCheckpoints = new();

    bool debugMode = false;
   

    void Start() 
    {
        playerRB = PlayerBody.GetComponent<Rigidbody2D>();
        armsRB = PlayerBody.GetComponentsInChildren<Rigidbody2D>();
        playerMovement = PlayerMovement.Instance;
        playerGrabbing = PlayerGrabbing.Instance;
        colliders = PlayerBody.GetComponents<Collider2D>();
        checkpointManager = GetComponent<CheckpointManager>();
        DebugListener.Instance.DebugModeChanged += EnableDebug;
        enabled = false;
    }

    public void EnableDebug(object sender, EventArgs e)
    {
        enabled = true;
        debugMode = true;
    }

    void Update()
    {
        if (!debugMode) { return; }

        if (Input.GetKeyDown(KeyCode.Alpha1) && !Input.GetKey(KeyCode.LeftShift))
        {
            TpHeadToBody();
        }
        if (Input.GetKeyDown(KeyCode.Alpha2))
        {
            TpBodyToHead();
        }
        if (Input.GetKeyDown(KeyCode.Alpha3))
        {
            ResetLevel();
        }
        if (Input.GetKeyDown(KeyCode.Alpha4))
        {
            ToggleNoClip();
        }
        if( Input.GetKeyDown(KeyCode.Alpha5))
        {
            FindObjectOfType<ArmCalibrationTutorial>().ArmCalibrationTutorial_Complete(null);
        }

        if (Input.GetKey(KeyCode.LeftShift) && Input.GetKeyDown(KeyCode.Alpha1))
        {
            SwitchToLevel("Starting Level");
        }

        if (Input.GetKey(KeyCode.LeftShift) && Input.GetKeyDown(KeyCode.E))
        {
            TpToNextCheckpoint(true);
        }
        if (Input.GetKey(KeyCode.LeftShift) && Input.GetKeyDown(KeyCode.Q))
        {
            TpToNextCheckpoint(false);
        }
    }
    private void FixedUpdate() 
    {
        if (NoClipEnabled)
        {
            NoClipMovement();
        }
    }

    private void TpHeadToBody()
    {
        PlayerHead.transform.position = PlayerBody.transform.position;
        PlayerHead.GetComponent<Rigidbody2D>().velocity = Vector2.zero;
    }
    private void TpBodyToHead()
    {
        PlayerBody.transform.position = PlayerHead.transform.position;
    }

    private void ResetLevel()
    {
        Time.timeScale = 1;
        SceneManager.LoadScene(SceneManager.GetActiveScene().name);
    }
    private void SwitchToLevel(string sceneName)
    {
        Time.timeScale = 1;
        SceneManager.LoadScene(sceneName);
    }

    private void ToggleNoClip()
    {
        NoClipEnabled = !NoClipEnabled;
        int gravity = 0;
        if (NoClipEnabled)
            gravity = 0;
        else
            gravity = 1;
        //Toggles playermovement and colliders
        playerMovement.enabled = !NoClipEnabled;
        playerRB.gravityScale = gravity;
        foreach (Rigidbody2D rigid in armsRB)
        {
            rigid.gravityScale = gravity;
        }
        playerRB.velocity = Vector2.zero;
        foreach (Collider2D collider in colliders)
        {
            collider.enabled = !NoClipEnabled;
        }
    }

    private void NoClipMovement()
    {
        float xInput = Input.GetAxis("Horizontal");
        float yInput = Input.GetAxis("Vertical");
        PlayerBody.transform.position += new Vector3(xInput * Time.deltaTime * NoClipMoveSpeed, yInput * Time.deltaTime * NoClipMoveSpeed);
    }

    private void TpToNextCheckpoint(bool goToNextCheckpoint)
    {
        int index = 0;
        foreach (Checkpoint item in allCheckpoints)
        {
            if (item == checkpointManager.GetCurrentCheckpoint())
            {
                if (goToNextCheckpoint)
                {
                    if (allCheckpoints.Count < index + 2)
                    {
                        //Debug.Log("At final checkpoint");
                        return;
                    }
                    checkpointManager.SwitchCurrentCheckpoint(allCheckpoints[index + 1]);
                    checkpointManager.GetCurrentCheckpoint().EnableCheckpoint();
                    checkpointManager.TeleportPlayerToCheckpoint();
                    return;
                }
                else
                {
                    if (index == 0)
                    {
                        //Debug.Log("At first checkpoint");
                        return;
                    }
                    checkpointManager.SwitchCurrentCheckpoint(allCheckpoints[index - 1]);
                    checkpointManager.GetCurrentCheckpoint().EnableCheckpoint();
                    checkpointManager.TeleportPlayerToCheckpoint();
                    return;
                }
            }
            index ++;
        }
    }

    public void LogMessage(string message)
    {
        Debug.Log(message);
    }

    public void DisablePlayerControls()
    {
        playerMovement.enabled = false;
        playerGrabbing.enabled = false;
    }
    public void EnablePlayerControls()
    {
        if (!NoClipEnabled)
            playerMovement.enabled = true;
        playerGrabbing.enabled = true;
    }
}
  ```
</details>

## - Pause Menu/Changing Controls
![](https://github.com/AxelRK32/Portfolio/blob/main/HeadOn/Images/yo8yhfr.png) ![](https://github.com/AxelRK32/Portfolio/blob/main/HeadOn/Images/h86k6uH.png)

I made a simple pause menu that disables player movement and sets the timescale to zero, along with toggling mouse controls and showing some buttons. From the main menu you can access the controls for grabbing and throwing objects. The way I handled the different controls was to utilize Unity's input system that defines inputs as strings. By using a string variable as the input instead of the string, I can easily change grabbing from Fire1 to Fire2 for example. 
