https://github.com/vasiliy-spb/The_Simulation  
[Василий]

Есть расширенный функционал.

## ХОРОШО

+ 👍 Спрайты существ хранятся не в самих существах
+ 👍 Есть обширное меню
+ 🚀 Есть редактор карты
+ 👍 Алгоритм поиска пути AStar
+ 👍 Альтернативный способ организации движения креатур

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Избыточный контекст
```
Entity getEntityByCoordinates(Coordinates coordinates)

//ПРАВИЛЬНО:
Entity get(Coordinates coordinates)
```

- Если придерживаешься Oracle code convetnions, то должно быть `DIRECTIONS`, а если соглашений Google, то `directions`
```
private static final Set<Direction> directions
```

- UPPER_SNAKE только для констант, а это не константа
```
private final Set<Class<? extends Entity>> NOT_OBSTACLES_TYPES_FOR_MOVE
```

- Название вводит в заблуждение: метод не проверяет здоровье. Он уменьшает здоровье, если голод
```
protected void checkHealth() {
  if (isHungry()) {
    this.healthPoint--;
  }
}
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"* 

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
Dialog<Character> characterDialog = new CharacterDialog(this.selectMessage, this.errorMessage, Set.of('1', '2', '\0'));
//...
case '\0' -> MenuItems.CONTINUE;
case '1' -> MenuItems.CHANGE_MOVES_DELAY;
case '2' -> MenuItems.EXIT;

//ПРАВИЛЬНО:
private final static CMD_CONTINUE = '\0';
private final static CMD_CHANGE_MOVES_DELAY = '1';
private final static CMD_EXIT = '2';

Set<Character> keys = Set.of(CMD_CONTINUE, CMD_CHANGE_MOVES_DELAY, CMD_EXIT);
Dialog<Character> characterDialog = new CharacterDialog(this.selectMessage, this.errorMessage, keys);
//...
case CMD_CONTINUE -> MenuItems.CONTINUE;
case CMD_CHANGE_MOVES_DELAY -> MenuItems.CHANGE_MOVES_DELAY;
case CMD_EXIT -> MenuItems.EXIT;
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**3. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (targetTypes.contains(entity.getClass())) {...}

//ПРАВИЛЬНО:
if (isTarget(entity)) {...}

private boolean isTarget(Entity entity) {
  return targetTypes.contains(entity.getClass());
}
```

**4. record Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**5. class WorldMap**

- В исключение всегда передавай сообщение на английском языке, которое будет объяснять причину, по которой было выброшено это исключение 
```
if (!entities.containsKey(coordinates)) {
  throw new IllegalArgumentException();
}
```

- Никогда не возвращай null
```
private final Map<Coordinates, Entity> entities;

public Entity getEntityByCoordinates(Coordinates coordinates) {
  //...  
  return entities.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void moveEntity(Coordinates fromCoordinates, Coordinates toCoordinates) {
  Entity entity = entities.remove(fromCoordinates);
  entities.put(toCoordinates, entity);
}

public boolean areBusy(Coordinates coordinates) {
  return entities.containsKey(coordinates);
}
```

+ 👍 В целом, класс понравился, к нему мало замечаний.

**6. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно создавать разные его реализации, например BFS и AStar.

- Судя по сигнатуре метода поиска пути, вызывающий код должен предварительно уже найти цель, к которой нужно проложить путь
```
List<Coordinates> find(Coordinates fromCoordinates, Coordinates toCoordinates, Set<Coordinates> obstacles);
```
Таким образом, единый процесс поиска пути разделяется на две части: поиск целей и поиск пути к этим целям. 
Я считаю, что это искусственное разделение одной ответственности на две части.  
Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям. 
В данном случае- до точки, в которой находится существо нужного класса. Например, Так
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {...}
```
Если для для того, чтобы алгоритм AStar рассчитал путь к цели ему нужно заранее знать координату цели, то класс поиска все равно должен самостоятельно найти эту координату. 
И только потом скормить эту координату алгоритму AStar.
Сейчас координаты целей ищат сами Predator и Herbivore, то есть берут на себя часть ответственности класса поиска пути.

**7. class AStarPathFinder implements PathFinder**

- Не передавай null в конструктор. Вместо этого используй перегруженный конструктор, который не предполагает того аргумента, в который передается null
```
PathNode startNode = new PathNode(start, heuristicDistance, heuristicDistance, null);

private static class PathNode {
  //...
  public PathNode(Coordinates coordinates, int heuristicDistance, int cost, PathNode parent) {...}
} 

//ПРАВИЛЬНО:
PathNode startNode = new PathNode(start, heuristicDistance, heuristicDistance);

private static class PathNode {
  //...
  public PathNode(Coordinates coordinates, int heuristicDistance, int cost, PathNode parent) {...}
  public PathNode(Coordinates coordinates, int heuristicDistance, int cost) {
    this(coordinates, heuristicDistance, cost, null);
  }
} 
```

**8. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.

**9. class Grass extends StaticEntity implements Prey<Herbivore>**

- Трава не должна знать, что ее ест именнно Herbivore. Трава должна просто получать повреждение
```
public class Grass extends StaticEntity implements Prey<Herbivore> {
  private int healthPoint;  
  //...
  @Override
  public void takeDamage(Herbivore hunter) {
    healthPoint -= hunter.getDamage();
  }
}

//ПРАВИЛЬНО:
public class Grass extends StaticEntity {
  private int healthPoint;  
  //...
  @Override
  public void subHealthPoint(int damage) {
    healthPoint -= damage;
  }
}
```

**10. abstract class Creature extends Entity**

- Интересная логика "должен ходить"
```
public boolean shouldMove(int currentTick) {
  return currentTick % turnFrequency == 0;
}
```
Как я понял идея в следующем: креатуру периодически дергают за хвост и один раз за каждые `turnFrequency` дерганий она говориит что готова к движению.
Таким образом волк готов к движениям каждые три дерганья за хвост, а заяц- каждые два. 

Идея понятная. Но логику дергания за хвост нужно убирать. Детальнее это рассмотрим в соответствующем Экшене.

**11. class Herbivore/Predator extends Creature**

+ 👍 Есть сет целей, таким образом существо может питаться несколькими видами пищи
```
Set<Class<? extends Entity>> TARGET_TYPES = Set.of(Herbivore.class);
```

- Дублирование кода в классах Herbivore и Predator в `makeMove()`. Общий код нужно вынести в предка.

**12. Пакет menu**

- Группа классов для меню в процедурном стиле. Если бы было одно меню, то ладно.  
Но здесь их несколько, а потому много избыточного кода.  
Думаю, здесь лучше бы подошел ООП стиль.    
Про меню в ООП стиле я писал тут: https://t.me/zhukovsd_it_chat/53243/114908

- Второй конец сложной логики поедания существа, которую мы видели у Травы
```
herbivore.takeDamage(this);

//ПРАВИЛЬНО:
herbivore.subHealthPoint(getDamage());
```

**13. class WorldMapFactory**

+ 👍 Фабрика карты это хорошо.

- Название `getInstance()` является стандартным названием для метода создающего синглтон. Здесь этот метод не создает синглтон
```
public static WorldMap getInstance(int height, int width) {
  return new WorldMap(height, width);
}

//ПРАВИЛЬНО:
public static WorldMap get(int height, int width) {...}
```

- Код класса читается тяжело. Много публичных методов, поэтому я не знаю, как нужно правильно пользоваться данным классом.  
Судя по всему, в этом классе сейчас две ответственности- сделать карту с дефолтными параметрами и сделать карту с параметрами, введенными вручную. То есть, через редактор карты.
Класс нужно разделить на два и ввести интерфейс фабрики
```
public interface WorldMapFactory {
  WorldMap get();
}

public class BasicWorldMapFactory implements WorldMapFactory{
  private final InitParams initParams;

  public BasicWorldMapFactory(InitParams initParams) {
    this.initParams = initParams;
  }

  @Override
  public WorldMapFactory get() {
    //карта с параметрами из initParams
  }
}

public class WorldMapEditor implements WorldMapFactory{

  public WorldMapEditor() {
  }

  @Override
  public WorldMapFactory get() {
    //редактор карты:
    //создает карту с параметрами
    //полученными в результате диалога
    //с пользователем
  }
}
```
Скорее всего в этих двух фабриках будет некий общий код. 
Например, логика помещения существ в карту на случайные координаты. 
Такой код нужно будет вынести в общего предка- AbstractWorldMapFactory.

Это в свою очередь скорее всего приведет к тому, что отпадет надобность в классах InitParamsFactory и InitParams. Часть кода из этих классов окажется ненужной, остальное переместится в AbstractWorldMapFactory.

Тогда при выборе пункта меню "Играть со случайными параметрами" карту будет создавать фабрика карты с параметрами по умолчанию. 
А при выборе "Играть со своими параметрами" карту будет создавать фабрика карты с параметрами, вводимыми вручну. То есть, редактор карты.

**14. class RockFactory/TreeFactory extends EntityFactory**, семейство фабрик существ

- Потребность в этих фабриках возникла потому что существа в себе хранят координаты. Поэтому при создании существа нужно найти на Карте свободную координату. 
Если бы существа не хранили в себе координату, их можно было бы создавать просто через `new`.

- Фабрик много, кода много, он сложный. Учитывая, как используются эти классы в коде, возможно стоит сделать одну фабрику на всех, а не по персональной фабрике на каждое существо, что сильно упростит код. 
Например, сейчас фабрики используются так
```
private static final Map<Class<? extends Entity>, EntityFactory<? extends Entity>> factories = Map.of(
  Grass.class, new GrassFactory(),
  Rock.class, new RockFactory(),
  Tree.class, new TreeFactory(),
  Herbivore.class, new HerbivoreFactory(),
  Predator.class, new PredatorFactory()
);

//type это Class
Entity entity = factories.get(type).createInstanceBy(coordinates);
```

- Вместо этого можно было бы сделать одну простую фабрику
```
public class EntityFactory {
  private final WorldMap worldMap;

  public EntityFactory(WorldMap worldMap) {
    this.worldMap = worldMap;
  }

  public Entity get(Class<? extends Entity> clazz) {
    if (clazz == Grass.class) {
      return new Grass(getFreeRandomCoordinates());
    }
    if (clazz == Tree.class) {
      return new Tree(getFreeRandomCoordinates());
    }
    //...
    throw new IllegalArgumentException(/* message */);
  }

  private Coordinates getFreeRandomCoordinates() {
    //рандомная пустая координата из worldMap
  }
}

//...
Entity entity = entityFactory.get(type)
```

**15. class MakeMoveAction implements Action**

- (±)Хранит в себе PathFinder, а не берет из существ. Почему бы и нет.

- Каждый ход дергает креатуру за хвост и как только креатура говорит, что готова, дает ей пинок и она бежит
```
for (Entity entity : entities) {
  if (entity instanceof Creature creature) {
    if (creature.shouldMove(currentTick)) {
      creature.makeMove(worldMap, pathFinder);
    }
  }
}
```
Дергать за хвост не нужно. Вместо этого MakeMoveAction должен сам учитывать периодичность активности креатур и руководствуясь этим вызывать их движение. Например, так
```
for (Entity entity : entities) {
  if (entity instanceof Creature creature) {
    if (shouldMove(creature, currentTick)) {
      creature.makeMove(worldMap, pathFinder);
    }
  }
}

//...
private boolean shouldMove(creature, currentTick) {
  return return currentTick % creature.getTurnFrequency() == 0;
}
```

**16. interface WorldMapRender**

+ 👍 Интерфейс рендерера это хорошо, можно делать разные распечатки карты. Например, в одной реализации предаторы будут волками, а в другой реализации- тиграми.

**17. class ConsoleEntityIcons**, константный класс со спрайтами существ

- Конструктор должен быть private, класс должен быть final.

**18. class Simulation**

- Магические числа
```
String title = """
    Выберите скорость симуляции:
    1 — медленная (1 ход/сек)
    2 — быстрее (2 хода/сек)
    //...
    5 — максимальная (20 ходов/сек)
""";

private int getMoveDelayForSpeed(int selectedSpeed) {
  return switch (selectedSpeed) {
    case 1 -> 1000;
    case 2 -> 500;
    //...
    default -> throw new IllegalStateException("Unexpected value: " + selectedSpeed);
    };
  }
```

Возможно, стоит ввести enum для хранения скорости
```
private enum Speed {
  VERY_SLOW(1),
  SLOW(2),
  //...
  ;
  private final int frequency;

  Speed(int frequency) {
    this.frequency = frequency;
  }

  public int getFrequency() {
    return frequency;
  }
}
```

## ВЫВОД

Местами код можно существенно сократить. В целом неплохо.

n.61(137)  
#ревью #симуляция