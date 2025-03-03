https://github.com/Ar4ik4ik/SimulationJava  
[Arturo]

Есть продвинутые механики.

## ХОРОШО

+ 👍 Есть механика голода
+ 🚀 Разные варианты совершения ходов: движение к цели и случайное движение
+ 👍 Настройки игры хранятся в конфигурационном файле, значит настройки приложения можно менять без перекомпиляции проекта

## ЗАМЕЧАНИЯ

**1. Нейминг**

- UPPER_SNAKE только для констант, а это не константа
```
public final int WIDTH;

//ПРАВИЛЬНО:
public final int width;
```

- Избыточный контекст
```
WorldMap.deleteFromMap(Entity entity)
WorldMap.placeOnMap(Entity entity)

//ПРАВИЛЬНО:
WorldMap.delete(Entity entity)
WorldMap.place(Entity entity)
```

- Если есть `int hungry`, который означает уровень голода, и есть `boolean isHungry()`, который означает бинарное состояние голоден/сыт, то нужно в названии как-то разделить эти концепции
```
int hungry

//ПРАВИЛЬНО:
int hungryPoint
```

- Названия пакетов пишутся стилем lower_case
```
Utils

//ПРАВИЛЬНО:
utils
```

- Почему экземпляр WorldMap во всех классах называется `WorldMap mapInstance`, а например экземпляр рендерера называется `Renderer renderer` а не `Renderer rendererInstance`?
Просто каждый раз, видя `mapInstance` думаешь, какой особый смысл несет его постфикс.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Комментарии**  

- Некоторые коментарии не несут полезной нагрузки, а констатируют очевидное
```
public final int WIDTH; // x coordinate
public final int HEIGHT; // y coordinate
```
Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.
В идеале, комментариев вообще не должно быть, код должен объяснять сам себя через правильный нейминг и лаконичный код.  
*Мартин, "ЧК", гл.4*  

**3. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками. Исключение- метод equals()
```
if (entityClass == Herbivore.class) return config.entitiesSettings.herbivore;
if (entityClass == Predator.class) return config.entitiesSettings.predator;

//ПРАВИЛЬНО:
if (entityClass == Herbivore.class) {
  return config.entitiesSettings.herbivore;
}    
if (entityClass == Predator.class) { 
  return config.entitiesSettings.predator;
}  
```

**4. Создавай вспомогательные методы**, делай программу более простой и понятной
```
while (running && !worldMap.collectTargetEntity(Creature.class).isEmpty()) {...}

//ПРАВИЛЬНО:
while (isНазваниеКотороеВсеОбъясняет() {...}

private boolean isНазваниеКотороеВсеОбъясняет() {
  return running && !worldMap.collectTargetEntity(Creature.class).isEmpty();
}
```

**5. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
System.out.println("Введите команду (pause/resume/stop): ");
switch (command) {
  case "pause": //...
  case "resume":  //...
  //...
}

//ПРАВИЛЬНО:
private final static String PAUSE = "pause";
private final static String RESUME = "resume";
private final static String STOP = "stop";

System.out.printf("Введите команду (%s/%s/%s): %n", PAUSE, RESUME, STOP);
switch (command) {
  case PAUSE: //...
  case RESUME:  //...
  //...
}
```

**6. record Coordinates**

+ 👍 Рекорд для координаты- идеально

- Координата должна уметь хранить любые координаты, в том числе отрицательные
```
public Coordinates {
  if (x < 0 || y < 0) {  <-- нельзя ввести отрицат. коорд.
    throw new IllegalArgumentException("Attempted to set negative value for coords: \n" +
        ("X: %s\nY: %s").formatted(x, y));
  }
}
```
Координата не должна мешать использовать себя клиентским кодом так, как ему будет нужно. 

**7. class WorldMap**, карта симуляции

- Не возвращай статус выполнения операции или код ошибки. Этот метод должен быть void и просто вставлять существо в карту. 
А если существо вставить не получается когда клетка занята- нужно бросать исключение
```
board.placeOnMap(Entity entity);

//...
public boolean placeOnMap(Entity entity) {
  if (isEmpty(entity.getCoordinates())) {
    entitiesByCoords.put(entity.getCoordinates(), entity);
    return true;
  }
  return false; 
}

//ПРАВИЛЬНО:
if(board.isEmpty(coordinates)) {
  board.placeOnMap(Entity entity);
}

//...
public void placeOnMap(Entity entity) {
  if (!isEmpty(entity.getCoordinates())) {
    //кинуть исключение
  }  

  entitiesByCoords.put(entity.getCoordinates(), entity);
}
```
*Мартин, "Чистый код", гл.3, п."Разделение команд и запросов", "...вместо возвращения кодов ошибок"*  
 
+ 👍 Молодец, что возвращаешь не оригинал мапы, а ее копию. В этом случае клиент не получает доступ к внутренностям Карты- оригинальной мапе
```
public Map<Coordinates, Entity> getEntitiesByCoords() {
  return new HashMap<>(entitiesByCoords);
}
```

- Но название `getEntitiesByCoords()` неправильное- если есть поле с таким именем, то геттер подразумевает, что возвращается оригинал. Этот метод правильно назвать `toMap()`.

- Никогда не возвращай null
```
private final Map<Coordinates, Entity> entitiesByCoords = new HashMap<>();

public Entity getEntityByCoords(Coordinates coords) {
  return entitiesByCoords.get(coords);  <-- может вернуть null
}

public <T extends Entity> T getEntityByCoords(Coordinates coords, Class<T> type) {
  //...
  return null;  <-- возвращает null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
public void moveEntity(Entity entity, Coordinates newEntityCoords) {
  Coordinates oldEntityCoords = entity.getCoordinates();
  if (replaceCoords(entity, newEntityCoords)) {
    entitiesByCoords.remove(oldEntityCoords);
    placeOnMap(entity);
  }
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Вот здесь должна быть та проверка на корректность, которая была в конструкторе координаты- нужно проверять, что координата больше нуля и меньше границ карты
```
public boolean isWithinBorders(Coordinates coords) {
  return coords.x() < WIDTH && coords.y() < HEIGHT;
}
```

+ 👍 В целом, замечаний к карте немного.

**9. class Hungry**, механизм голода.

- Магическоие цифры: -5, 10.

**10. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.
- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```
abstract public String toString();
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами.
Спрайты всех существ должны храниться в классе, который распечатывает карту.

**11. class Tree/Grass/... extends Entity**

- Модель не должна зависеть от представления. Но раз уж зависит, то пусть хранит это представление в себе, а не берет это из третьего класса. В данном случае- свой спрайт.
Сейчас это нарушение Low Coupling
```
@Override
public String toString() {
  return EntitiesRepresentation.TREE.getEntityRepresentation();
}

//ЛУЧШЕ(но это все равно плохо):
private static final String SPRITE = "\uD83D\uDFEB";

@Override
public String toString() {
  return SPRITE;
}
```

**12. class Herbivore/Predator extends Creature**

- Магическоие цифры: -1, 25, 100.

**13. class PathFinderService**, поиск пути

+ 👍 Использует связный однонаправленный список- кастомный class Node, это упрощает код.
+ 👍 Продвинутый поиск, который учитывает вес ноды.

- Смещение нужно вынести в константы, чтобы каждый раз не пересоздавать их при вызове метода
```
public static Map<Coordinates, Integer> getNeighbors(...) {
  int[][] DIRECTIONS = new int[][]{{x - 1, y}, {x, y - 1}, {x + 1, y}, {x, y + 1},
    {x - 1, y - 1}, {x + 1, y - 1}, {x - 1, y + 1}, {x + 1, y + 1}};
  //...
}

//ПРАВИЛЬНО:
private final static SHIFTS int[][] DIRECTIONS = new int[][]{{x - 1, y}, {x, y - 1}, {x + 1, y}, {x, y + 1},
    {x - 1, y - 1}, {x + 1, y - 1}, {x - 1, y + 1}, {x + 1, y + 1}};
```

**14. Generator.java**

- Куча классов в одном файле- плохая практика. Создай пакет `generator` и помести в него файлы с классами генераторов. Один файл- один класс.
- Путь к конфигурационному файлу класс должен принимать в конструктор, а не хранить в себе
```
config = ConfigLoader.loadConfig("src/main/java/Game/config.json");
```

**15. class Config**

+ 👍 Возможность использования разных конфигураций это хорошо.
- Про применение конфигурационных класов я писал тут: https://t.me/zhukovsd_it_chat/53243/176984

**16. class PredatorGenerator/GrassGenerator/... extends Generator**

- Нарушение DRY, тотальное дублирование кода, куча классов которые делают одно и то же- заселяют карту существами
```
class TreeGenerator extends Generator<Tree> {

  public TreeGenerator(WorldMap mapInstance) {
    super(mapInstance);
  }

  @Override
  public Tree generateEntity(Coordinates coordinates) {
    return new Tree(coordinates);
  }
}

class RockGenerator extends Generator<Rock> {

  public RockGenerator(WorldMap mapInstance) {
    super(mapInstance);
  }

  @Override
  public Rock generateEntity(Coordinates coordinates) {
    return new Rock(coordinates);
  }
}
```

**17. class MoveCreaturesAction implements Action**

+ 👍 Прям идеальный СовершательХодов - только находит всех креатур на карте и инициирует их движение.

**18. class MovementStrategyChooseAction implements Action**

+ 👍 Интересная акция, я такой механики кажется еще не встречал- если креатура голодная, она ищет цель, если сытая- просто гуляет. Акция сетит в креатуру эту стратегию. 

**19. class PlaceEntitiesAction implements Action**

- Путь к конфигурационному файлу класс должен принимать в конструктор, а не хранить в себе
```
ConfigLoader.getEntitiesBalanceFromConfig("src/main/java/Game/config.json").entrySet()
```

- Слишком длинные конструкции, которые трудно понять, тем более в for'е
```
for (Map.Entry<Class<? extends Entity>, Double> entry:
  ConfigLoader.getEntitiesBalanceFromConfig("src/main/java/Game/config.json").entrySet()) {...}

//ПРАВИЛЬНО:
Map<Class<? extends Entity>, Double> map = ConfigLoader.getEntitiesBalanceFromConfig("src/main/java/Game/config.json").entrySet();
for (Map.Entry<Class<? extends Entity>, Double> entry: map) {...}
```
Вводи поясняющие переменные.
*Фаулер, "Рефакторинг", гл.6,п."Введение поясняющей переменной"*  

**20. class TargetMoveStrategy**

- Не забывай про инкапсуляцию. Поля классов должны быть `private final`, если нет причин для иного. Тут причин нет
```
public class TargetMoveStrategy бла-бла {
  Class<T> target;
  int moveSpeed;
  //...
}  
```

**21. class Simulation**

+ 👍 Правильное использование Action'ов- они сгрупированы в два листа, один обрабатывается на старте, другой- каждыый ход
```
List<? extends Action> initActionsList = new ArrayList<>(List.of(new PlaceEntitiesAction(), new MovementStrategyChooseAction()));
List<? extends Action> turnActionsList = new ArrayList<>(List.of(new MoveCreaturesAction(), new CheckAliveEntityAction(), new MovementStrategyChooseAction()));
```

- Только я не пойму, что здесь дает `List<? extends Action>` по сравнению с просто `List<Action>`? Кажется, ничего не дает. При прочих равных всегда делай проще.

- Думается, что симуляция должна принимать экземпляр `Config` в конструктор, а не самостоятельно его создавать. 
Потому что, в теории, может быть несколько разных конфигов, а сейчас симуляция заточена под один конкретный и она даже знает где именно он живет
```
Config config = ConfigLoader.loadConfig("src/main/java/Game/config.json");
```

**22. class Renderer**

- Должен содержать в себе спрайты существ. Или брать их из `num EntitiesRepresentation`, но не из самих существ.

**23. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## АРХИТЕКТУРА

- Местами архитектура показалась мне более сложной, чем могла бы быть. Например, за движение/поиск путей отвечает целая группа классов: PathFinderService, RandomMoveStrategy, TargetMoveStrategy, MovementStrategy. Можно ли было сделать проще, не знаю.
- Экземпляр конфига нужно создавать в майне и дальше инжектить в Симуляцию. Симуляция, получив в себя конфиг, должна брать из него данные и инжектить в остальные классы. 
При этом желательно, если есть возможность, нужно инжектить в эти классы только нужные им данные, а не Config целиком- иначе классы будут знать не только про нужные им данные, но и про класс Config и всю лишнюю для них информацию, которая в нем хранится.

- Чрезвычайно сложная логика заселения существ. Много дублирующих классов. 
И если невозможно откзазаться от несколькиз заселяющих классов потому что существа конфигуируются информацией из конфига, то саму структуру заселения можно все же немного упростить.

Делаем универсальный Заселятор, его уже достаточно для заселения карты простыми статичными существами Rock/Tree etc.
```
public class Spawner {

  private final WorldMap worldMap;
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public Spawner(WorldMap worldMap, Supplier<Entity> entitySupplier, int amount) {
    this.worldMap = worldMap;
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  public void spawn() {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      Coordinates coordinates = getRandomCoordinates(worldMap);
      entity.updateCoordinates(coordinates);  //сделать updateCoordinates public
      worldMap.placeOnMap(entity);
    }
  }

  private Coordinates getRandomCoordinates(WorldMap worldMap) {
    //возвращает случайную пустую координату из карты
  }
}
```

Для более сложных нужно делать персонального потомка 
```
public class HerbivoreSpawner extends Spawner {

  public HerbivoreSpawner(WorldMap worldMap, int amount, int moveSpeed, int maxHealthPoints, int maxHungry) {
    super(
        worldMap,
         () -> new Herbivore(
              new Coordinates(1, 1),  //костыль
              worldMap,
              moveSpeed,
              maxHealthPoints,
              maxHungry
          ),
        amount
    );
  }
}
```

Теперь делаем Action для первоначального заселения
```
public class SpawnAction implements Action {
  private final Config config;

  public SpawnAction(Config config) {
    this.config = config;
  }

  @Override
  public void execute(WorldMap worldMap) {
    Coordinates defaultCoordinates = new Coordinates(1, 1); //костыль

    List<Spawner> spawners = List.of(
        new Spawner(worldMap, ()-> new Tree(defaultCoordinates), config.balance.treeAmount),
        new Spawner(worldMap, ()-> new Rock(defaultCoordinates), config.balance.rockAmount),
        new HerbivoreSpawner(
            worldMap,
            config.balance.herbivoreAmount,
            config.entitiesSettings.herbivore.moveSpeed,
            config.entitiesSettings.herbivore.maxHealthPoints,
            config.entitiesSettings.herbivore.maxHungry
        )
    );
    spawners.forEach( s -> s.spawn());
  }
}
```

## ВЫВОД

В целом, учитывая дополнительные механики и конфигурирование, неплохо.

n.60(136)  
#ревью #симуляция