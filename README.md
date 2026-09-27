## Welcome to my repository featuring a demo version of my game 👋
[🎬 Gameplay samples video](https://drive.google.com/file/d/1PNKDttSdC9amDdgD7SlUfcFkFKjC12S5/view?usp=drive_link)
## Link to the main development repository with the source code
https://github.com/Oleg0501/moonsteel-saga
If you need access to the source code for technical review, I can temporarily add you as a collaborator upon request
## What has already been implemented
**Architecture**<br>
The main goal of the architecture is to ensure low coupling between components and provide a foundation that can be reused and adapted across different projects.<br>

The architecture is designed around the following requirements:<br>
• reusability in future projects;<br>
• easy extensibility;<br>
• minimizing the cost of changes and maintenance;<br>
• independent development of individual parts.<br>

The implementation is based on the following principles:<br>
• **MVP** — the project is divided into two main independent layers: game logic and user interface. The layers do not depend on each other directly and interact through a separate integration layer.<br>
• **Dependency Injection** — dependencies are managed using Zenject. Components are registered through Scene Contexts and Installers, while the startup and initialization of individual subsystems are handled by custom bootstrap components.<br>

## Sample of MVP realization
For example, there is a logical ECS entity representing the player, which is processed by an ECS system responsible for health changes:
```csharp
namespace Code.Client.Logic.ECS.Health.Systems
{
    public class HealthChangeSystem : IEcsRunSystem
    {
        private readonly EcsFilter _ecsFilter;
        private readonly EcsPool<CHealth> _healthEcsPool;
        private readonly EcsPool<CHealthChange> _healthChangeEcsPool;
        private readonly EcsPool<CStatistics> _statisticsEcsPool;
        private readonly EcsPool<CDeath> _deathEcsPool;
        
        [Inject]
        public HealthChangeSystem(EcsWorld ecsWorld)
        {
            _ecsFilter = ecsWorld.Filter<CHealthChange>().Inc<CHealth>().End();
            _healthEcsPool = ecsWorld.GetPool<CHealth>();
            _healthChangeEcsPool = ecsWorld.GetPool<CHealthChange>();
            _statisticsEcsPool = ecsWorld.GetPool<CStatistics>();
            _deathEcsPool = ecsWorld.GetPool<CDeath>();
        }
        
        public void Run(IEcsSystems systems)
        {
            foreach (var entity in _ecsFilter)
            {
                ref var healthComponent = ref _healthEcsPool.Get(entity);
                var healthChange = _healthChangeEcsPool.Get(entity).HealthChange;
                var healthMax = _statisticsEcsPool.Get(entity).StatisticsArgs.HealthMax;
                
                if (healthComponent.Health + healthChange > healthMax)
                {
                    healthComponent.Health = healthMax;
                }
                else
                {
                    healthComponent.Health += healthChange;
                }
                
                if (healthComponent.Health <= 0f)
                {
                    _deathEcsPool.Add(entity);
                }
            }
        }
    }
}
```
Next, there is an adapter layer that tracks changes in the ECS and notifies all registered listeners, passing them a prepared data structure:
```csharp
namespace Code.Client.Logic.ECS.Player.Systems.Trackers
{
    public class PlayerHealthChangeTracker : EcsEventTracker<CHealthChange, HealthChangedEvent>
    {
        private readonly EcsPool<CHealth> _healthPool;
        private readonly EcsPool<CStatistics> _statisticsPool;

        protected override EcsFilter EcsFilter => EcsWorld.Filter<CHealth>().Inc<CHealthChange>().Inc<CPlayer>().End();
        protected override EcsPool<CHealthChange> EcsPool => EcsWorld.GetPool<CHealthChange>();
        
        [Inject]
        public PlayerHealthChangeTracker(EcsWorld ecsWorld) : base(ecsWorld)
        {
            _healthPool = ecsWorld.GetPool<CHealth>();
            _statisticsPool = ecsWorld.GetPool<CStatistics>();
        }

        protected override HealthChangedEvent CreateData(int entity, CHealthChange eventComponent)
        {
            var health = _healthPool.Get(entity);
            var statistics = _statisticsPool.Get(entity);
            
            return new HealthChangedEvent(health.Health, statistics.StatisticsArgs.HealthMax);
        }
    }
}
```
Then, there is the presenter layer, which acts as one of the listeners and controls the visual interface as a controller:
```csharp
namespace Code.Client.Presenter.PlayerHealth
{
    public class PlayerHealthPresenter : Presenter<PlayerHealthView>
    {
        private readonly PlayerHealthChangeTracker _playerHealthChangeTracker;
        
        [Inject]
        public PlayerHealthPresenter(EcsWorld ecsWorld, PlayerHealthChangeTracker playerHealthChangeTracker)
        {
            _playerHealthChangeTracker = playerHealthChangeTracker;
        }

        protected override void InitializeView()
        {
            base.InitializeView();
            
            _playerHealthChangeTracker.OnChanged += OnTracked;
        }
        
        public override void UnloadView()
        {
            _playerHealthChangeTracker.OnChanged -= OnTracked;
            
            base.UnloadView();
        }

        private void OnTracked(HealthChangedEvent healthChangedEvent)
        {
            var ratio = healthChangedEvent.Current / healthChangedEvent.Max;
            
             View.SetFillAmount(ratio);
        }
    }
}
```
Finally, there is the visual interface implemented as a MonoBehaviour, which contains only references to the object's components and methods for updating them:
```csharp
public class PlayerHealthView : UIView
{
    [SerializeField] private Image _healthImage;

    public void SetFillAmount(float fillAmount)
    {
        _healthImage.fillAmount = fillAmount;
    }
}
```
