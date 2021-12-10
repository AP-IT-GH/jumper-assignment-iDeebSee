# jumper-assignment-iDeebSee


## Groep

*Groepsnaam: WhiteBeard*

| Groepsleden                                       | S-nummer          |
| --------------------------------------------------| ----------------- |
| Amiri Abdelmajid                                  | 109434            |
| Azdad Mohamed                                     | 106961            |

## Introductie

In dit project gaan we werken met ML-agents. ML-agents zorgt er eigelijk voor dat we agents in een soort spel omgeving of simulatie kunnen laten trainen. In dit project hebben we ervoor gezorgd dat we een speler kunnen laten springen over verschillende obstakels. Dat doen we door de speler meerdere keren te laten trainen.

## Benodigdheden

-   Unity
-   Visual Studio (om scripts te maken)
-   Anaconda (om u agent te trainen)
-   Pytorch (anders kun je anaconda niet runnen)
-   Project (https://github.com/tomptrs/MLAgents-Klas-Voorbeeld/tree/master)

## Environment

Ons environment bestaat uit een plane, obstacle, wall, agent en text. Op de afbeelding zie je de scene waarmee we starten.

![](Afbeeldingen\Scene.png)

### Plane

De plane is het blauwe veld waar de obstacles en agent op spelen/trainen. Daarop komen ook de obstacles te voorschijn.

![](Afbeeldingen\Plane.png)

### Obstacle

-   Groene obstacle: De groene obstacle is een obstacle waar de agent wel door mag. Zo krijgt hij zijn rewards. Rewards zijn nodig zodat de agent weet dat hij goed bezig is. Hij krijgt dus pluspunten als hij door het groene obstacle gaat zonder te springen.

![](Afbeeldingen\GreenObstacle.png)

-   Rode Obstacle: De rode obstacle is het geen waar de agent over moet springen. Als hij deze raakt, krijgt hij minpunten zodat hij weet dat hij daar niet door mag gaan.

![](Afbeeldingen\RedObstacle.png)

### Wall

De wall staat achter de agent. De wall detecteerd welke obstacle de wall aanraakt. Als de agent een obstacle raakt, dan verdwijnt deze obstacle. Als de agent over de obstacle springt, dan raakt de obstacle de wall aan en kan dan zo berekenen hoeveel punten hij verdient of niet.

![](Afbeeldingen\Wall.png)

### Agent

De agent is de protagonist in dit verhaal. Hij moet ervoor zorgen door meerdere keren te trainen, dat hij over de juiste obstacle springt. Zoals je kan zien op de afbeelding zie je dat hij kan zien wat er voor hem staat.

![](Afbeeldingen\Agent.png)

### Text

De tekst is simpel weg gewoon om de punten van de agent te volgen. Zo kan je zelf zien hoe hij het doet.

![](Afbeeldingen\Text.png)

## Scripts

Er zijn verschillende scripts dat ervoor zorgen dat alles juist zijn taken afhandeld. Deze scripts ga ik stap voor stap uitleggen en laten zien.

### Environment

De environment heeft een belangrijke script. Deze script zorgt ervoor dat alles zowat samenhangt. In de inspector ziet de script er zo uit.

![](Afbeeldingen\EnvScriptInspector.png)

Daarin zie je verschillende prefabs. Je sleept de juiste prefab naar het juiste vakje zodat het spel herkent welke obstacle de groene is of de rode.

Het script Environment Jumper ziet er als volgt uit.

```
public class EnvironmentJumper : MonoBehaviour
{
    public GameObject ObstaclePrefab;
    public GameObject GoodObstaclePrefab;
    public GameObject Obstacles;
    public bool canSpawnObstacles = true;
    
    // Start is called before the first frame update
    void Start()
    {
        
    }

    private void OnEnable()
    {
        Obstacles = transform.Find("Obstacles").gameObject;
        Debug.Log("start spwn");
        
        StartCoroutine(SpawnObstacleContinuously());

    }
    // Update is called once per frame
    void Update()
    {
        
    }

    private IEnumerator SpawnObstacleContinuously()
    {
        while (true)
        {
            float result = Random.Range(0, 2);
            float r = Random.Range(2f, 5.0f);
            yield return new WaitForSeconds(r); 
            if(canSpawnObstacles)
                if (result == 0)
                {
                    SpawnObstacle();
                }
                else
                {
                    SpawnGoodObstacle();
                }
               
        }
    }

    //Spawn every X seconds

    public void SpawnObstacle()
    {
        GameObject newObstacle = Instantiate(ObstaclePrefab.gameObject);
       

        newObstacle.transform.SetParent(Obstacles.transform);
        
       // float rx = Random.Range(-4f, 4);
        //float rz = Random.Range(-4f, 2);
        newObstacle.transform.localPosition = new Vector3(-8, 0.5f, 0);
        
    }
    public  void SpawnGoodObstacle()
    {
        GameObject newGoodObstacle = Instantiate(GoodObstaclePrefab.gameObject);
        newGoodObstacle.transform.SetParent(Obstacles.transform);
        newGoodObstacle.transform.localPosition = new Vector3(-8, 0.5f, 0);

    }

        public void ClearEnvironment()
        {        
            foreach (Transform obstacle in Obstacles.transform)
            {
                GameObject.Destroy(obstacle.gameObject);
            }

        canSpawnObstacles = true;
    }
}
```
### Obstacle

De groene en rode obstacles hebben beide een script. De groene obstacle heeft een script genaamd Obstacle Move Good. De rode heeft een script genaamd Obstacle Move. De inspector ziet er bij beide hetzelfde uit. Je kan de speed daar aanpassen.

![](Afbeeldingen\GrObstInspector.png)

In de scripts zelf is er ook niet veel verschil. Enigste verschil is bij de OncollisionEnter functie. Bij de rode obstacle een krijgt de agent als reward ``` agent.AddReward(1f); ```  en bij de groene obstacle script staat er ``` agent.AddReward(-1f); ```. Onderaan zie je hoe de script eruit ziet voor de rode obstacle.
```
[SerializeField]
    private float speed = 4f;

    

    private JumperAgent agent;
    // Start is called before the first frame update
    void Start()
    {
        agent = GameObject.Find("Agent").GetComponent<JumperAgent>();
    }

    private void OnEnable()
    {
        //speed = Random.Range(4, 8);
    }

    // Update is called once per frame
    void Update()
    {
        
        transform.localPosition += new Vector3(speed * Time.deltaTime, 0, 0);
    }

    private void OnCollisionEnter(Collision collision)
    {      
            if (collision.transform.CompareTag("Wall"))
            {
               // Debug.Log("raak");
                agent.AddReward(1f);
                Destroy(this.gameObject);
               // agent.EndEpisode();
        }
            
       
    }

```
### Agent

De agent heeft verschillende scripts. Namelijk een Ray Perception Sensor 3D script om de ogen te spele van de agent.

![](Afbeeldingen\RayPercSens.png)

```
[AddComponentMenu("ML Agents/Ray Perception Sensor 3D", (int)MenuGroup.Sensors)]
    public class RayPerceptionSensorComponent3D : RayPerceptionSensorComponentBase
    {
        [HideInInspector, SerializeField, FormerlySerializedAs("startVerticalOffset")]
        [Range(-10f, 10f)]
        [Tooltip("Ray start is offset up or down by this amount.")]
        float m_StartVerticalOffset;

        /// <summary>
        /// Ray start is offset up or down by this amount.
        /// </summary>
        public float StartVerticalOffset
        {
            get => m_StartVerticalOffset;
            set { m_StartVerticalOffset = value; UpdateSensor(); }
        }

        [HideInInspector, SerializeField, FormerlySerializedAs("endVerticalOffset")]
        [Range(-10f, 10f)]
        [Tooltip("Ray end is offset up or down by this amount.")]
        float m_EndVerticalOffset;

        /// <summary>
        /// Ray end is offset up or down by this amount.
        /// </summary>
        public float EndVerticalOffset
        {
            get => m_EndVerticalOffset;
            set { m_EndVerticalOffset = value; UpdateSensor(); }
        }

        /// <inheritdoc/>
        public override RayPerceptionCastType GetCastType()
        {
            return RayPerceptionCastType.Cast3D;
        }

        /// <inheritdoc/>
        public override float GetStartVerticalOffset()
        {
            return StartVerticalOffset;
        }

        /// <inheritdoc/>
        public override float GetEndVerticalOffset()
        {
            return EndVerticalOffset;
        }
    }

```

Het volgende script dat de agent heeft is de Behavior Parameters. Daarin kan je het gedrag van de agent aanpassen. Deze script bevat ook het neural network bestand.

![](Afbeeldingen\BehPar.png)


```
public enum BehaviorType
    {
        /// <summary>
        /// The Agent will use the remote process for decision making.
        /// if unavailable, will use inference and if no model is provided, will use
        /// the heuristic.
        /// </summary>
        Default,

        /// <summary>
        /// The Agent will always use its heuristic
        /// </summary>
        HeuristicOnly,

        /// <summary>
        /// The Agent will always use inference with the provided
        /// neural network model.
        /// </summary>
        InferenceOnly
    }

    /// <summary>
    /// Options for controlling how the Agent class is searched for <see cref="ObservableAttribute"/>s.
    /// </summary>
    public enum ObservableAttributeOptions
    {
        /// <summary>
        /// All ObservableAttributes on the Agent will be ignored. This is the
        /// default behavior. If there are no  ObservableAttributes on the
        /// Agent, this will result in the fastest initialization time.
        /// </summary>
        Ignore,

        /// <summary>
        /// Only members on the declared class will be examined; members that are
        /// inherited are ignored. This is a reasonable tradeoff between
        /// performance and flexibility.
        /// </summary>
        /// <remarks>This corresponds to setting the
        /// [BindingFlags.DeclaredOnly](https://docs.microsoft.com/en-us/dotnet/api/system.reflection.bindingflags?view=netcore-3.1)
        /// when examining the fields and properties of the Agent class instance.
        /// </remarks>
        ExcludeInherited,

        /// <summary>
        /// All members on the class will be examined. This can lead to slower
        /// startup times.
        /// </summary>
        ExamineAll
    }

    /// <summary>
    /// A component for setting an <seealso cref="Agent"/> instance's behavior and
    /// brain properties.
    /// </summary>
    /// <remarks>At runtime, this component generates the agent's policy objects
    /// according to the settings you specified in the Editor.</remarks>
    [AddComponentMenu("ML Agents/Behavior Parameters", (int)MenuGroup.Default)]
    public class BehaviorParameters : MonoBehaviour
    {
        [HideInInspector, SerializeField]
        BrainParameters m_BrainParameters = new BrainParameters();

        /// <summary>
        /// Delegate for receiving events about Policy Updates.
        /// </summary>
        /// <param name="isInHeuristicMode">Whether or not the current policy is running in heuristic mode.</param>
        public delegate void PolicyUpdated(bool isInHeuristicMode);

        /// <summary>
        /// Event that fires when an Agent's policy is updated.
        /// </summary>
        internal event PolicyUpdated OnPolicyUpdated;

        /// <summary>
        /// The associated <see cref="Policies.BrainParameters"/> for this behavior.
        /// </summary>
        public BrainParameters BrainParameters
        {
            get { return m_BrainParameters; }
            internal set { m_BrainParameters = value; }
        }

        [HideInInspector, SerializeField]
        NNModel m_Model;

        /// <summary>
        /// The neural network model used when in inference mode.
        /// This should not be set at runtime; use <see cref="Agent.SetModel(string,NNModel,Policies.InferenceDevice)"/>
        /// to set it instead.
        /// </summary>
        public NNModel Model
        {
            get { return m_Model; }
            set { m_Model = value; UpdateAgentPolicy(); }
        }

        [HideInInspector, SerializeField]
        InferenceDevice m_InferenceDevice = InferenceDevice.Default;

        /// <summary>
        /// How inference is performed for this Agent's model.
        /// This should not be set at runtime; use <see cref="Agent.SetModel(string,NNModel,Policies.InferenceDevice)"/>
        /// to set it instead.
        /// </summary>
        public InferenceDevice InferenceDevice
        {
            get { return m_InferenceDevice; }
            set { m_InferenceDevice = value; UpdateAgentPolicy(); }
        }

        [HideInInspector, SerializeField]
        BehaviorType m_BehaviorType;

        /// <summary>
        /// The BehaviorType for the Agent.
        /// </summary>
        public BehaviorType BehaviorType
        {
            get { return m_BehaviorType; }
            set { m_BehaviorType = value; UpdateAgentPolicy(); }
        }

        [HideInInspector, SerializeField]
        string m_BehaviorName = "My Behavior";

        /// <summary>
        /// The name of this behavior, which is used as a base name. See
        /// <see cref="FullyQualifiedBehaviorName"/> for the full name.
        /// This should not be set at runtime; use <see cref="Agent.SetModel(string,NNModel,Policies.InferenceDevice)"/>
        /// to set it instead.
        /// </summary>
        public string BehaviorName
        {
            get { return m_BehaviorName; }
            set { m_BehaviorName = value; UpdateAgentPolicy(); }
        }

        /// <summary>
        /// The team ID for this behavior.
        /// </summary>
        [HideInInspector, SerializeField, FormerlySerializedAs("m_TeamID")]
        public int TeamId;
        // TODO properties here instead of Agent

        [FormerlySerializedAs("m_useChildSensors")]
        [HideInInspector]
        [SerializeField]
        [Tooltip("Use all Sensor components attached to child GameObjects of this Agent.")]
        bool m_UseChildSensors = true;

        [HideInInspector]
        [SerializeField]
        [Tooltip("Use all Actuator components attached to child GameObjects of this Agent.")]
        bool m_UseChildActuators = true;

        /// <summary>
        /// Whether or not to use all the sensor components attached to child GameObjects of the agent.
        /// Note that changing this after the Agent has been initialized will not have any effect.
        /// </summary>
        public bool UseChildSensors
        {
            get { return m_UseChildSensors; }
            set { m_UseChildSensors = value; }
        }

        /// <summary>
        /// Whether or not to use all the actuator components attached to child GameObjects of the agent.
        /// Note that changing this after the Agent has been initialized will not have any effect.
        /// </summary>
        public bool UseChildActuators
        {
            get { return m_UseChildActuators; }
            set { m_UseChildActuators = value; }
        }

        [HideInInspector, SerializeField]
        ObservableAttributeOptions m_ObservableAttributeHandling = ObservableAttributeOptions.Ignore;

        /// <summary>
        /// Determines how the Agent class is searched for <see cref="ObservableAttribute"/>s.
        /// </summary>
        public ObservableAttributeOptions ObservableAttributeHandling
        {
            get { return m_ObservableAttributeHandling; }
            set { m_ObservableAttributeHandling = value; }
        }

        /// <summary>
        /// Returns the behavior name, concatenated with any other metadata (i.e. team id).
        /// </summary>
        public string FullyQualifiedBehaviorName
        {
            get { return m_BehaviorName + "?team=" + TeamId; }
        }

        void Awake()
        {
            OnPolicyUpdated += mode => { };
        }

        internal IPolicy GeneratePolicy(ActionSpec actionSpec, ActuatorManager actuatorManager)
        {
            switch (m_BehaviorType)
            {
                case BehaviorType.HeuristicOnly:
                    return new HeuristicPolicy(actuatorManager, actionSpec);
                case BehaviorType.InferenceOnly:
                    {
                        if (m_Model == null)
                        {
                            var behaviorType = BehaviorType.InferenceOnly.ToString();
                            throw new UnityAgentsException(
                                $"Can't use Behavior Type {behaviorType} without a model. " +
                                "Either assign a model, or change to a different Behavior Type."
                            );
                        }
                        return new BarracudaPolicy(actionSpec, actuatorManager, m_Model, m_InferenceDevice, m_BehaviorName);
                    }
                case BehaviorType.Default:
                    if (Academy.Instance.IsCommunicatorOn)
                    {
                        return new RemotePolicy(actionSpec, actuatorManager, FullyQualifiedBehaviorName);
                    }
                    if (m_Model != null)
                    {
                        return new BarracudaPolicy(actionSpec, actuatorManager, m_Model, m_InferenceDevice, m_BehaviorName);
                    }
                    else
                    {
                        return new HeuristicPolicy(actuatorManager, actionSpec);
                    }
                default:
                    return new HeuristicPolicy(actuatorManager, actionSpec);
            }
        }

        /// <summary>
        /// Query the behavior parameters in order to see if the Agent is running in Heuristic Mode.
        /// </summary>
        /// <returns>true if the Agent is running in Heuristic mode.</returns>
        public bool IsInHeuristicMode()
        {
            if (BehaviorType == BehaviorType.HeuristicOnly)
            {
                return true;
            }

            return BehaviorType == BehaviorType.Default &&
                ReferenceEquals(Model, null) &&
                (!Academy.IsInitialized ||
                    Academy.IsInitialized &&
                    !Academy.Instance.IsCommunicatorOn);
        }

        internal void UpdateAgentPolicy()
        {
            var agent = GetComponent<Agent>();
            if (agent == null)
            {
                return;
            }
            agent.ReloadPolicy();
            OnPolicyUpdated?.Invoke(IsInHeuristicMode());
        }
    }

```

De agent kan springen door de Jumper Agent script. Daarin kan je verschillende dingen aanpassen in verband met de jump van de agent. Ook ga je daar je scoreboard in slepen.

![](Afbeeldingen\JumperAgent.png)

```
public class JumperAgent : Agent
{
    [SerializeField]
    private float jumpStrength = 5f;
    [SerializeField]
    private TextMeshPro scoreboard;

    private bool canJump = true;
    private Rigidbody rigidbody;
    private EnvironmentJumper environment;

    // Start is called before the first frame update
    void Start()
    {
        rigidbody = GetComponent<Rigidbody>();
        environment = GetComponentInParent<EnvironmentJumper>();
    }

    // Update is called once per frame
    void Update()
    {
        scoreboard.text = GetCumulativeReward().ToString("f4");
    }

    public override void OnEpisodeBegin()
    {
        environment.ClearEnvironment();
        transform.localPosition = new Vector3(7, 0.5f, 0);
        transform.localRotation = Quaternion.Euler(0, -90, 0f);
    }
    

    public override void OnActionReceived(ActionBuffers actions)
    {
        var vectorAction = actions.DiscreteActions;
        if (vectorAction[0] == 1)
        {
            //punish with small negative award to prevent jumping all the time
            AddReward(-0.01f);
            Jump();
        }
    }

    public override void Heuristic(in ActionBuffers actionsOut)
    {
        //map actions to movement
        var jump = 0;
        if (Input.GetKey(KeyCode.Space))
        {
            jump = 1;
        }
        var discreteActionsOut = actionsOut.DiscreteActions;
        discreteActionsOut[0] = jump;
    }



    private void Jump()
    {
        if (canJump)
        {
            rigidbody.AddForce(new Vector3(0, jumpStrength, 0), ForceMode.VelocityChange);
            canJump = false;
        }
    }

    private void OnCollisionEnter(Collision collision)
    {
        if (collision.transform.CompareTag("Plane"))
        {
            canJump = true;
        }

        if (collision.transform.CompareTag("Obstacle"))
        {
            Debug.Log("collide with obstacle");
            Destroy(collision.gameObject);
            AddReward(-1f);           
        }
        if (collision.transform.CompareTag("GoodObstacle"))
        {
            Debug.Log("collide with good obstacle");
            Destroy(collision.gameObject);
            AddReward(1f);
        }
        /*if (collision.transform.CompareTag("HiddenCollider"))
        {
            Debug.Log("collide with hidden collider");
            AddReward(1f);
        }*/

    }
}
```

De laatste script dat de agent bevat is de Decision Requester. Deze script zorgt ervoor dat de agent moet kiezen of hij nu moet springen of niet.

![](Afbeeldingen\Decision.png)

```
/// <summary>
    /// The DecisionRequester component automatically request decisions for an
    /// <see cref="Agent"/> instance at regular intervals.
    /// </summary>
    /// <remarks>
    /// Attach a DecisionRequester component to the same [GameObject] as the
    /// <see cref="Agent"/> component.
    ///
    /// The DecisionRequester component provides a convenient and flexible way to
    /// trigger the agent decision making process. Without a DecisionRequester,
    /// your <see cref="Agent"/> implementation must manually call its
    /// <seealso cref="Agent.RequestDecision"/> function.
    /// </remarks>
    [AddComponentMenu("ML Agents/Decision Requester", (int)MenuGroup.Default)]
    [RequireComponent(typeof(Agent))]
    public class DecisionRequester : MonoBehaviour
    {
        /// <summary>
        /// The frequency with which the agent requests a decision. A DecisionPeriod of 5 means
        /// that the Agent will request a decision every 5 Academy steps. /// </summary>
        [Range(1, 20)]
        [Tooltip("The frequency with which the agent requests a decision. A DecisionPeriod " +
            "of 5 means that the Agent will request a decision every 5 Academy steps.")]
        public int DecisionPeriod = 5;

        /// <summary>
        /// Indicates whether or not the agent will take an action during the Academy steps where
        /// it does not request a decision. Has no effect when DecisionPeriod is set to 1.
        /// </summary>
        [Tooltip("Indicates whether or not the agent will take an action during the Academy " +
            "steps where it does not request a decision. Has no effect when DecisionPeriod " +
            "is set to 1.")]
        [FormerlySerializedAs("RepeatAction")]
        public bool TakeActionsBetweenDecisions = true;

        [NonSerialized]
        Agent m_Agent;

        /// <summary>
        /// Get the Agent attached to the DecisionRequester.
        /// </summary>
        public Agent Agent
        {
            get => m_Agent;
        }

        internal void Awake()
        {
            m_Agent = gameObject.GetComponent<Agent>();
            Debug.Assert(m_Agent != null, "Agent component was not found on this gameObject and is required.");
            Academy.Instance.AgentPreStep += MakeRequests;
        }

        void OnDestroy()
        {
            if (Academy.IsInitialized)
            {
                Academy.Instance.AgentPreStep -= MakeRequests;
            }
        }

        /// <summary>
        /// Information about Academy step used to make decisions about whether to request a decision.
        /// </summary>
        public struct DecisionRequestContext
        {
            /// <summary>
            /// The current step count of the Academy, equivalent to Academy.StepCount.
            /// </summary>
            public int AcademyStepCount;
        }

        /// <summary>
        /// Method that hooks into the Academy in order inform the Agent on whether or not it should request a
        /// decision, and whether or not it should take actions between decisions.
        /// </summary>
        /// <param name="academyStepCount">The current step count of the academy.</param>
        void MakeRequests(int academyStepCount)
        {
            var context = new DecisionRequestContext
            {
                AcademyStepCount = academyStepCount
            };

            if (ShouldRequestDecision(context))
            {
                m_Agent?.RequestDecision();
            }

            if (ShouldRequestAction(context))
            {
                m_Agent?.RequestAction();
            }
        }

        /// <summary>
        /// Whether Agent.RequestDecision should be called on this update step.
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        protected virtual bool ShouldRequestDecision(DecisionRequestContext context)
        {
            return context.AcademyStepCount % DecisionPeriod == 0;
        }

        /// <summary>
        /// Whether Agent.RequestAction should be called on this update step.
        /// </summary>
        /// <param name="context"></param>
        /// <returns></returns>
        protected virtual bool ShouldRequestAction(DecisionRequestContext context)
        {
            return TakeActionsBetweenDecisions;
        }
    }

    
```