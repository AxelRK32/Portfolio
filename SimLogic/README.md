# Internship at Simlogic

## _Brief description of my internship at Simlogic_
My internship at Simlogic lasted for 6 months, and my role was as a programmer/developer for their VR platform. The application was developed for the purpose of training medical staff.

## _Tools used_
The VR application was developed using Unity, and tested using Meta Quest 2 and 3 headsets. The application was designed to only use handtracking. 

## _My assignments_
During my internship I was given several different assignments. The first mainly involved systematically testing the application to find any bugs, and to fix any bugs that our supervisor assigned to me.

The next assignment was to update the UI with a new UI that another intern had designed. The new UI also included some changes to functionality that I worked on implementing. 

The final assignment before my internship ended, was to develop a new training scenario using the systems that had already been created. I received an AI generated design document to follow, and the clients would periodically test the scenario and give feedback. 

## _Bugfixing_
<details>
  <summary>Details on assignments and workflow</summary>
  We used a speardsheet to keep track of what things we should test and what bugs we found when testing. We also had a section to show the current status of the bug.   After fixing a bug you update the status to say that the bug should be tested. After testing the bug we either set its status as fixed, or if we find it's not fixed, we update the status with a description explaining why the bug needs additional work. 
</details>


## _Updating UI_
<details>
  <summary>Details on assignments and workflow</summary>
  The company had taken on another intern who focused on UX and UI design, and after redesigning the existing UI I got a prrof of concept image showing how the new UI should look and function. 

  Implementing the new UI mostly involved replacing certain icons and text that already existed in the old UI, but some aspects of the new UI involved functionality that did not exist in the old UI. Because of that I also created and implemented these new functions to the UI. 
</details>


## _Developing a new scenario_
<details>
  <summary>Details on assignment and workflow</summary>
  The final assignment of the internship was the largest one. After our supervisor had given us the design document to look over, we also got a premade scene that only contained the essentials for running the application. 

  I spent some time at the start to read through the design document and asked several questions and clarifications. I also started coming up with what systems will be needed and how I should design them.
</details>


## _Systems I developed_
Most of the systems I developed from scratch was during the final assignment to support some features in the new scenario.  

<ins>Substep system:</ins>  
Some of the steps included in the new scenario could be considered sub-steps of a previous main step, so I developed a system that could interact with the pre-existing step system, and skip to specifc steps and check if all substeps had been completed. 

Here is the serialized object I used to represent a main step with multiple sub-steps:  
<details>
  <summary>Sub and Main step</summary>
  
  ```csharp
    [Serializable]
    public class ProcedureStepData
    {
        public ProcedureObject procedureStep;
        public int stepIndex;
        public bool completed;
    }
    
    [Serializable]
    public class ProcedureSubAndMainStep
    {
        public ProcedureStepData MainStep;
        [Tooltip("If the main step also has an action tied to it, along with the substeps")] 
        public bool mainStepRequiresActions;
        public List<ProcedureStepData> SubSteps;
        public int stepIndexAfterCompletingMainStep;
    }
  ```
</details>

Here is some of the code that handled the sub-steps:
<details>
  <summary>Sub step manager</summary>

  ```csharp
    public void SkipToSubStep(ProcedureObject procedure)
    {
        ProcedureStepData procedureStep = GetStepData(procedure);
        if (procedureStep != null)
            if (procedureStep.completed)
                return;
            else
                TutorialSystem.instance.SkipToSubStep(procedureStep.stepIndex);
        else
            Debug.LogWarning("No substep for in SubStepSkipManager for: " + procedure);
    }
    public void SkipToSubStep(int procedureIndex)
    {
        ProcedureStepData procedureStep = GetStepData(procedureIndex);
        if (procedureStep != null)
        {
            if (procedureStep.completed)
                return;
            else
                TutorialSystem.instance.SkipToSubStep(procedureStep.stepIndex);
        }
        else
            Debug.LogWarning("No substep for in SubStepSkipManager for step: " + procedureIndex);
    }

    public void CompleteSubStep(int index)
    {
        ProcedureStepData subStep = GetStepData(index);
        ProcedureSubAndMainStep mainStep = GetMainStep(index);
        if (index == mainStep.MainStep.stepIndex) //Completed main step
        {
            Debug.Log("Completed main step." + index + " Skipping to step: " + mainStep.stepIndexAfterCompletingMainStep);
            TutorialSystem.instance.SkipToSubStep(mainStep.stepIndexAfterCompletingMainStep);
        }
        else if (subStep != null)
        {
           subStep.completed = true;
            //if main step requires action, all three conditionals need to be true. if it doesn't require action, only CheckSubStepsCompleted need to be true
            if (CheckSubStepsComleted(mainStep) && (!mainStep.mainStepRequiresActions || mainStep.MainStep.completed))
            {
                Debug.Log("All substeps completed");
                TutorialSystem.instance.SkipToSubStep(mainStep.stepIndexAfterCompletingMainStep);
            }
            else
            {
                TutorialSystem.instance.SkipToSubStep(mainStep.MainStep.stepIndex);
            }
        }
        
        else
            Debug.LogWarning("No substep for in SubStepSkipManager for index: " + index);
    }

    bool CheckSubStepsComleted(ProcedureSubAndMainStep subAndMain)
    {
        bool allCompleted = true;
        foreach (ProcedureStepData step in subAndMain.SubSteps)
        {
            if (!step.completed)
            {
                allCompleted = false;
                break;
            }
        }
        return allCompleted;
    }

    ...

    ProcedureSubAndMainStep GetMainStep(int anyIndex) //Gets the main step from eiter the main index or any substep index 
    {
        ProcedureSubAndMainStep mainStep;

        mainStep = allSubAndMainStepGroups
                .FirstOrDefault(s => s.SubSteps
                .Any(sub => sub.stepIndex == anyIndex));
        if (mainStep == null)
        {
            mainStep = allSubAndMainStepGroups
                    .FirstOrDefault(s => s.MainStep.stepIndex == anyIndex);
        }
        if (mainStep != null)
        {
            return mainStep;
        }
        else
        {
            Debug.LogError("Could not find any ProcedureSub/Main step object that contained index: " + anyIndex);
            return null;
        }
    }
  ```
</details>


<ins>Waste disposal system</ins>  
The new scenario also included a procedure to dispose of some items after they were used, and some of those items should also be replenishable.  
I decided that the best way to replenish those items, were to just disable them when they were disposed, and then enable them when they get replenished.  
Because of that I decided to create an interface to have better control over what actually happens when something is disposed.

<details>
  <summary>Waste disposal system</summary>

  ```csharp
public interface IWasteDisposable
{
    void DisposeItem();
}

[Serializable]
public class WasteObject
{
    public string objectName;
    public UnityEvent OnObjectDisposed;
}

public class WasteDisposal : MonoBehaviour
{
    public static WasteDisposal instance;
    [SerializeField] WasteObject[] objectsToDispose;

    void Awake()
    {
        if (instance == null)
        {
            instance = this;
        }
        else
        {
            Destroy(this);
        }
    }

    void OnTriggerEnter(Collider other)
    {
        if (DisposeGameObject(other.gameObject))
        {
            Debug.Log("Disposed object: " + other.name);
        }
        else if (objectsToDispose.FirstOrDefault(obj => obj.objectName == other.name) != null) 
        {
            //If the object is in the list, but doesnt implement IWasteDisposable, it Destroys the object by default
            Debug.LogWarning(other.name + " does not implement IWasteDisposable. Destroying object...");
            objectsToDispose.FirstOrDefault(wo => wo.objectName == other.name).OnObjectDisposed?.Invoke();
            Destroy(other.gameObject);
        }
        
    }

    public bool DisposeGameObject(GameObject obj) //Returns false if object does not have IWasteDisposable
    {
        if (obj.TryGetComponent(out IWasteDisposable dispose))
        {
            WasteObject wasteObject = objectsToDispose.FirstOrDefault(wo => wo.objectName == obj.name);
            if (wasteObject != null)
            {
                Debug.LogWarning("Disposing item: " + wasteObject.objectName);
                dispose.DisposeItem();
                wasteObject.OnObjectDisposed?.Invoke();
            }
            else //If the disposed object is not in the list, but still implements IWasteDisposable, it still calls DisposeItem
            {
                Debug.LogError("Waste Disposal list does not contain object: " + obj.name); 
                dispose.DisposeItem();
            }
            return true;
        }
        else 
            return false;
    }

}

  ```
</details>


<ins>Multi trigger procedure</ins>  
Another system that was needed, was the ability to complete a procedure by moving a certain object, following a strict path. The way I made this system was to use a series of white balls that appeared in the order they should be entered. When a ball was entered correctly, it would change color to green, and the next white ball would appear. If you went the wrong way and entered a ball that had already been turned green, it would instead turn red and trigger a error prompt to the user.  
Some of these movements in the design documents, also needed multiple movements and an option for something to happen when a single movement is completed. So I designed the system to support that.

<details>
  <summary>Single trigger object</summary>

  ```csharp
[Serializable]
public class MultiTriggerObject
{
    [Tooltip("Index determines the order the colliders need to be entered")]
    public List<MultiTriggerCollider> TriggerColliders;
    public UnityEvent OnCompletion;
    [HideInInspector] public bool Finished;
}

public class MultiTriggerCollider : MonoBehaviour
{
    [HideInInspector] public string requiredObjectName;
    [HideInInspector] public List<string> incorrectObjectNames;
    public bool triggerEntered {get; set;}
    public Action<MultiTriggerCollider> OnTriggerEntered;
    public Action<string> OnIncorrectTriggerEntered;
    [SerializeField] MeshRenderer triggerVisual;
    [SerializeField] Material defaultMaterial, clearTriggerMaterial, reEnteredTriggerMaterial;

    void OnTriggerEnter(Collider other)
    {
        if (other.gameObject.name == requiredObjectName)
        {
            OnTriggerEntered?.Invoke(this);
            CheckIfAlreadyEntered();
        }
        else if (incorrectObjectNames.Contains(other.gameObject.name))
        {
            OnIncorrectTriggerEntered?.Invoke(other.gameObject.name);
        }
    }

    void CheckIfAlreadyEntered()
    {
        if (triggerEntered)
        {
            if (reEnteredTriggerMaterial)
                triggerVisual.material = reEnteredTriggerMaterial;
        }
        else
        {
            triggerEntered = true;
            triggerVisual.material = clearTriggerMaterial;
        }
    }
    public void ForceCheckEntered()
    {
        CheckIfAlreadyEntered();
    }

    public void ResetTriggerCollider()
    {
        triggerEntered = false;
        triggerVisual.material = defaultMaterial;
    }
}

  ```
</details>

<details>
  <summary>Procedure with multiple triggers</summary>

  ```csharp

[Serializable]
public class IncorrectObjectEvent
{
    public string incorrectObjectName;
    public UnityEvent OnIncorrectObject;
}

public class MultiTriggerProcedure : MonoBehaviour
{
    [SerializeField][Tooltip("Index determines in what order the trigger objects are completed")]
    private List<MultiTriggerObject> triggerObjects;
    [SerializeField] float delayBetweenTriggerObjects = 1f;
    [SerializeField] string correctObjectName;
    [SerializeField] List<IncorrectObjectEvent> incorrectObjects;
    [SerializeField]
    UnityEvent OnReEnteringCollider;
    [SerializeField]
    ProcedureObject procedure;
    [SerializeField]
    UnityEvent OnFullProcedureCompleted;

    void Start()
    {
        List<string> incorrectNames = incorrectObjects.Select(i => i.incorrectObjectName).ToList();
        foreach (MultiTriggerObject o in triggerObjects)
        {
            foreach (MultiTriggerCollider c in o.TriggerColliders)
            {
                c.requiredObjectName = correctObjectName;
                c.incorrectObjectNames = incorrectNames;
                c.gameObject.SetActive(false);
            }
        }
    }

    void OnEnable()
    {
        foreach (MultiTriggerObject mto in triggerObjects)
        {
            foreach (MultiTriggerCollider mtc in mto.TriggerColliders)
            {
                if (mtc.OnTriggerEntered == null || 
                    !mtc.OnTriggerEntered.GetInvocationList()
                    .Any(d => d.Method.Name == nameof(TriggerEntered))) //Prevent a single TriggerCollider from getting duplicate subscribers
                    {
                        mtc.OnTriggerEntered += TriggerEntered;
                        mtc.OnIncorrectTriggerEntered += IncorrectTriggerEntered;
                    }
            }
        }
    }
    void OnDisable()
    {
        foreach (MultiTriggerObject mto in triggerObjects)
        {
            foreach (MultiTriggerCollider mtc in mto.TriggerColliders)
            {
                if (mtc.OnTriggerEntered.Method.Name.Equals(nameof(TriggerEntered)))
                    mtc.OnTriggerEntered -= TriggerEntered;
                if (mtc.OnIncorrectTriggerEntered.Method.Name.Equals(nameof(IncorrectTriggerEntered)))
                    mtc.OnIncorrectTriggerEntered -= IncorrectTriggerEntered;
            }
        }
    }

    public void StartMultiTrigger(int index = 0)
    {
        if (!triggerObjects[index].TriggerColliders[0].transform.parent.gameObject.activeSelf)
            triggerObjects[index].TriggerColliders[0].transform.parent.gameObject.SetActive(true);

        triggerObjects[index].TriggerColliders[0].gameObject.SetActive(true);
        foreach (MultiTriggerCollider col in triggerObjects[index].TriggerColliders)
        {
            col.ResetTriggerCollider();
        }
    }

    public void RestartAllMultiTriggers()
    {
        foreach (MultiTriggerObject mto in triggerObjects)
        {
            mto.Finished = false;
            ResetMultiTriggerColliders(mto);
        }
        StartCoroutine(WaitToStartMultiTrigger(0));
    }
    public void RestartCurrentlyActiveMultiTrigger()
    {
        MultiTriggerObject currentTriggerObject = triggerObjects.FirstOrDefault(t => !t.Finished);
        if (currentTriggerObject == null)
        {
            Debug.LogError("Could not find the currently active MultiTriggerObject!");
            return;
        }
        ResetMultiTriggerColliders(currentTriggerObject);
        StartCoroutine(WaitToStartMultiTrigger(triggerObjects.IndexOf(currentTriggerObject)));
    }
    void ResetMultiTriggerColliders(MultiTriggerObject triggerObject)
    {
        foreach (MultiTriggerCollider c in triggerObject.TriggerColliders)
        {
            c.ResetTriggerCollider();
            c.gameObject.SetActive(false);
        }
    }

    IEnumerator WaitToStartMultiTrigger(int index)
    {
        yield return new WaitForSeconds(delayBetweenTriggerObjects);
        StartMultiTrigger(index);
    }

    void TriggerEntered(MultiTriggerCollider enteredCollider)
    {
        if (enteredCollider.triggerEntered) //Going backwards
        {
            Debug.LogWarning("Reentered an already entered trigger, Going backwards");
            OnReEnteringCollider?.Invoke();
            return;
        }
        MultiTriggerObject currentTriggerObject = triggerObjects.FirstOrDefault(t => t.Finished == false);
        if (!currentTriggerObject.TriggerColliders.Contains(enteredCollider))
        {
            Debug.LogError(currentTriggerObject + " does not contain the entered collider! " + enteredCollider);
            return;
        }

        int colliderIndex = currentTriggerObject.TriggerColliders.IndexOf(enteredCollider);

        if (colliderIndex + 1 == currentTriggerObject.TriggerColliders.Count) // Final collider
        {
            CompleteSingleMultiTriggerObject(currentTriggerObject);
        }
        else
        {
            //Activate next collider
            currentTriggerObject.TriggerColliders[colliderIndex + 1].gameObject.SetActive(true);
        }
    }
    void IncorrectTriggerEntered(string nameThatEntered)
    {
        IncorrectObjectEvent incorrect = incorrectObjects.FirstOrDefault(i => i.incorrectObjectName == nameThatEntered);
        if (incorrect != null)
        {
            incorrect.OnIncorrectObject?.Invoke();
        }
        else
        {
            Debug.LogError(nameThatEntered + " is not included in the incorrect objects");
        }
    }

    void ReEnterTrigger()
    {
        OnReEnteringCollider?.Invoke();
    }

    void CompleteSingleMultiTriggerObject(MultiTriggerObject completedObject)
    {
        completedObject.Finished = true;
        completedObject.OnCompletion?.Invoke();
        foreach (MultiTriggerCollider c in completedObject.TriggerColliders)
        {
            c.gameObject.SetActive(false);
        }

        int objectIndex = triggerObjects.IndexOf(completedObject);
        if (objectIndex + 1 == triggerObjects.Count)//Final trigger object
        {
            CompleteProcedure();
        }
        else
        {
            //Start next trigger object
            StartCoroutine(WaitToStartMultiTrigger(objectIndex + 1));
        }
    }

    void CompleteProcedure()
    {
        if (procedure)
        {
            procedure.TriggerProcedureClear(true);
        }
        else
        {
            Debug.LogWarning("Could not find any Procedure for: " + name);
        }
        OnFullProcedureCompleted?.Invoke();
    }

  ```
</details>
