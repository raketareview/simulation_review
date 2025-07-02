https://github.com/igarick/Simulation_OOP  
[Igor]

Есть над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается кривовато  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim087/img0.png)

Связано это с тем, что при распечатке используются и эмодзи и обычные символы, а они имеют разную ширину.  
Если хочешь чтобы было ровно, печатай все через эмодзи. Например, так  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim076/img0.png)

2. В программе рабоатает команда "Пуск/Стоп". 
Но пользователю не сообщается о существовании этих команд.
Я сам узнал об этих командах только из кода.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название должно как можно лучше объяснять суть явления. Это ширина и высота карты
```
public class SimulationMap {
  private final int rowCount;
  private final int columnCount;
  //...
}

//ЛУЧШЕ:
public class SimulationMap {
  private final int height;
  private final int width;
  //...
}
```

- Избыточный контекст
```
simulation.startSimulation();

//ЛУЧШЕ:
simulation.start();
```

- class Path
  - Если в проекте есть класс с определенным названием, то все переменные с именем, включающим это название, должны быть экземплярами этого класса. Иначе это приводит к путанице. 
  - Класс Путь здесь не содержит путь, а содержит алгоритм поиска пути
```
class Path {}

public abstract class Creature extends Entity implements AliveEntity {
  public void makeMove(..., List<Coordinates> path) {...}
  //...
}

//ПРАВИЛЬНО:
class PathFinder {}

public abstract class Creature extends Entity implements AliveEntity {
  public void makeMove(..., List<Coordinates> path) {...}
  //...
}
```

- Название обманывает. Метод возвращает не координаты, а ход. Координата(Coordinates) и Ход(Move) это не синонимы, это разные классы
```
Move getCoordinatesForMove(SimulationMap simulationMap, List<Coordinates> path)
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```
if (input.equals("p")) {...}
System.out.println("Неизвестная команда. Нажмите *p* и затем Enter для включения / выключения паузы");

//ПРАВИЛЬНО:
private final static String PAUSE = "p";

if (input.equals(PAUSE)) {...}
System.out.printf("Неизвестная команда. Нажмите '%s' и затем Enter для включения / выключения паузы \n", PAUSE);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется.  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково и код без `else` будет выглядеть читабельней
```
if (target.isInstance(entity)) {
  return constructPathToTarget(cameFrom, coordinates);
} else {
  queue.add(coordinates);
}

//ПРАВИЛЬНО:
if (target.isInstance(entity)) {
  return constructPathToTarget(cameFrom, coordinates);
}
queue.add(coordinates);
```

**4. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. Поля публичные- почему бы и нет, координату *в данном случае* можно позициионировать как структуру
```
public class Coordinates {
  public final int row;
  public final int column;
  //...
}
```
Про отличия структур от объектов тут: *Мартин "ЧК", гл.6*

- Класс может быть преобразован в record без потери функционала.

- Закомментированный код- антипаттерн "Лодочный якорь". 

Этот код никогда уже не понадобится, не превращай программу в мусорку. 
Если что-то хочешь сохранить на память, сделай коммит в гите с соответствующим комментарием.  
Правильное использование комментариев- *"Чистый код", гл.4*

**5. class CoordinatesShift**, координата для сдвига

Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. 
Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**6. class SimulationMap**

- Нарушение конвенции кода: 
  - Поля должны находиться вверху, перед методами
  - Первым из методов должен стоять конструктор

+ 👍 Перегруженые конструкторы сделаны правильно.

- Никогда не возвращай null
```
private final HashMap<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return entities.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean isEmpty(Coordinates coordinates) {
  return !entities.containsKey(coordinates);
}
```
Сейчас, если спросить у карты, пустая ли ячейка с координатами (+100500, -100500), то карта ответит, что пуста. А на самом деле такой ячейки вообще нет.

+ 👍 Метод норм, но не "получитьСущество", а "получитьСущества"
```
 public List<Entity> getEntityByType(Class<? extends Entity> targetType)
```

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
карта.setEntity(new Coordinates(0, 0), new Заяц());
карта.makeMove(new Coordinates(0, 0), new Coordinates(99, 99));

/* класс Карта */
  public void makeMove(Coordinates from, Coordinates to) {
    Entity entity = entities.get(from);
    removeEntity(from);
    setEntity(to, entity);
  }
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

+ 👍 В целом, карта соблюдает SRP. Нюанс с `makeMove()`- несущественен.

**7. class Path**, поиск пути

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям.

В данном случае- до точки, в которой находится существо нужного класса.
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно. 
Например, поиск не должен вместо точки старта принимать существо и анализируя положение этого существа в пространстве, самостоятельно находить точку старта
```
public static List<Coordinates> findPath(Creature creature, SimulationMap simulationMap, Class<? extends Entity> target)

//ПРАВИЛЬНО:
public static List<Coordinates> findPath(Coordinates start, SimulationMap simulationMap, Class<? extends Entity> target)
```

- Нарушение SRP. 

Поиск пути анализирует логистические возможности креатур и в зависимости от этого этого прокладывает маршрут
```
private static Set<Coordinates> getReachableNeighbors(...) {
  //...
  if (creature.canMoveThrough(entity)) {
    result.add(neighbor);
  }
  //...
}
```
То есть, поиск спрашивает например, у Волка, по каким поверхностям(существам) Волк может пройти и добавляет и если на координате стоит такое существо, то добавляет эту координату в список проходимых.
Волк может проходить по поверхностям, которые не являются препятствием, травой и другим Волком. Таким образом, Волк может ходить по Зайцу
```
public class Predator extends Creature {
  public boolean canMoveThrough(Entity entity) {
    return !(entity instanceof Obstacle || entity instanceof Grass || entity instanceof Predator);
  }
  //...
}
```
Тут кроется какая-то избыточно сложная логика. 

Я так понимаю, что это в какой-то мере дублирует поиск цели, которая передается в аргументы поиска `findPath(..., Class<? extends Entity> target)`. 
Потому что суть метода `canMoveThrough(...)` для Волка состоит в том, что он разрешит прокладывать путь, шагая по `null` только до Зайца. 
А такой же метод у Зайца- шагать по `null`, но только до Травы.

Я бы понял еще, если бы разные существа могли бы ходить по разным поверхностям и для этого был бы специальный метод определения проходимости этой поверхности. 
Например, Волк мог бы перескакивать траву, а Заяц- перепрыгивать другого Зайца. Тогда бы в этом был смысл.

Но даже в этом случае, напоминаю, что поиск пути должен все условия для поиска принимать в аргументы, а не определять их самостоятельно. 
Если бы были какие-то проходимые поверхности, метод их определения тоже нужно было бы принимать в аргументы метода.  
Например, так
```
public static List<Coordinates> findPath(Coordinates start, SimulationMap simulationMap, Class<? extends Entity> target, Predicate<Entity> определительПроходимости)
```
У Волка был бы такой метод опредедления проходимой поверхности
```
class Волк {
  @Override  
  public boolean isПроходимаяПоверхность(Entity entity) {
    return entity instanceof Grass; //волк ходит по траве
  } 
}
```
И тогда поиск пути для Волка выглядел бы так:
```
Волк волк = new Волк();
Заяц заяц = new Заяц();

SimulationMap simulationMap = new WorldMap(100, 100);
simulationMap.setEntity(волк, new Coordinates(0, 0));
simulationMap.setEntity(заяц, new Coordinates(99, 99));

Coordinates start = worldMap.get(волк);
Class<? extends Entity> target = волк.getTarget();

List<Coordinates> путьОтВолкаДоЗайца = findPath(start, simulationMap, target, волк::isПроходимаяПоверхность);
```

А сейчас, кажется, ты просто перемудрил с алгоритмом поиска пути. 
Для начала проверь, будет ли так же работать алгоритм, если поменять
```
//ЭТО:
if (creature.canMoveThrough(entity)) {
  result.add(neighbor);
}

//НА:
if (entity == null || target.isInstance(entity)) {
  result.add(neighbor);
}
```

**8. class PathUtils**

Если класс содержит только статические методы, то класс должен быть `final` и иметь приватный конструктор. 
Не должно быть возможности создать экземпляр такого класса или унаследоваться от него.

**9. abstract class Entity**

- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня `Creature`.

- Нарушение инкапсуляции. Этот класс уже нельзя считать структурой- от него тянется большая иерархия наследования. Доступ к полям должен быть не прямой, а через геттеры и сеттеры
```
public Coordinates coordinates;
```

**10. class Creature extends Entity implements AliveEntity**

- Нарушение инкапсуляции. Много публичных методов. 

Если метод используется только внутри класса, он должен быть private- `Move getCoordinatesForMove(...)`.
Если метод используется только внутри класса и в потомках, он должен быть protected- `void dropToMinHealth(int health)` etc.

* 🤷 Метод совершения хода принимает во входящие аргументы путь к цели, а не получает его внутри себя через класс поиска цели. В принципе, почему бы и нет
```
//МОЖНО И ТАК(НАВЕРНОЕ)
void makeMove(SimulationMap simulationMap, List<Coordinates> path) <-- сейчас так

//ОБЫЧНО ДЕЛАЕТСЯ ТАК:
void makeMove(SimulationMap simulationMap) {
  List<Coordinates> path = pathFinder.get(simulationMap, coordinates, getTarget())  <-- мне так больше нравится
  //...
}
```
О целесообразности такого подхода можно дискутировать. 
Лично мне больше нравится, когда существо само себе определяет цель через обращение к поиску, а не получает путь извне. 
В этом случае(как мне больше нравится) существо как бы руководствуется своей волей, а не выполняет чужие команды.

**11. Predator/Herbivore extends Creature**

Нарушение DRY. Идентичный метод, вынеси его в общего предка- в Creature
```
@Override
public boolean isPassable() {
  return false;
}
```

**12. interface Actions и кго реализации**

- Зря закоментировал, в интерфейсе interface Action должен быть один метод. 
Сейчас `Actions` это просто маркерный интерфейс и в этом качестве от него в контексте задачи толку немного
```
public interface Actions {
//  void execute(SimulationMap simulationMap);
}
```

Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.  
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.  
Action, изложенный в ТЗ, это вариант реализации паттерна Command.   

Должен быть общий класс/интерфейс Action и его наследники.  
Это своего рода вариация паттерна Command.  
Т.е. акции должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
```
interface Action{
 void execute(Карта карта);
}

class ХодитьAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти каждую креатуру
    //и дать ей пинка чтоб побежала
  }
}

class ДатьСигаретуВсемЗайцамAction реализует Action {
  void execute(Карта карта) {
    //обойти всю карту
    //найти всех зайцев и дать им по сигарете 
  }
}

List<Action> actions = List.of{new ДатьСигаретуВсемЗайцамAction(), new ХодитьAction()};
for(Action a: actions) {
  a.execute(карта);
}

/*
Результат: программа обойдет всю карту, найдет всех креатур и даст им пинка, чтобы они побежали.
А каждому зайцу предварительно даст сигарету.
*/
```

- В интерфейсе Action должен быть только один метод и в каждом классе-реализации этого интерфейса тоже должен быть только один публичный метод. 
А протектед и приватных методов- сколько угодно.

**13. class MaintainAction implements Actions**

Нарушение DRY
```
List<Entity> tree = simulationMap.getEntityByType(Tree.class);
if (mapArea / tree.size() > 10) {
  entitySpawnerAction.placeEntitiesRandomly(simulationMap, 1, Tree::new);
}

List<Entity> herbivore = simulationMap.getEntityByType(Herbivore.class);
if (mapArea / herbivore.size() > 10) {
  entitySpawnerAction.placeEntitiesRandomly(simulationMap, 1, c -> new Herbivore(c, 3, 100));
}
//еще миллион таких блоков

//ПРАВИЛЬНО:
spawn(simulationMap, Tree.class, mapArea, 10, Tree::new);
spawn(simulationMap, Herbivore.class, mapArea, 10, c -> new Herbivore(c, 3, 100));

private void spawn(SimulationMap simulationMap, Class<? extends Entity> clazz, int mapArea, int limit, Function<Coordinates, Entity> factory) {
  List<Entity> entities = simulationMap.getEntityByType(clazz);
  if (mapArea / entities.size() >= limit) {
    entitySpawnerAction.placeEntitiesRandomly(simulationMap, 1, factory);
  }
}
```

- Все магические числа, то есть лимит существ и их характеристики, нужно вынести в константы.

**14. class EntitySpawnerAction implements Actions**

Магические числа.

**15. class MoveAction implements Actions**

Не используй статические импорты- становится неясно, откуда берется используемый в классе метод
```
//СО СТАТИЧЕСКИМ ИМПОРТОМ:
public class MoveAction implements Actions {
  public void makeMove(SimulationMap simulationMap) {
    List<Coordinates> path = findPath(creature, simulationMap, creature.getTarget());
    //...
  }
}

//БЕЗ НЕГО:
public class MoveAction implements Actions {
  public void makeMove(SimulationMap simulationMap) {
    List<Coordinates> path = Path.findPath(creature, simulationMap, creature.getTarget());
    //...
  }
}
```

**16. class Sprites**, хранит спрайты существ

- В switch-case в default нужно кидать исключение. 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку. 
И эту ситуацию нужно выявить как можно раньше.

- Класс должен быть final и иметь приватный конструктор.

**17. class Renderer**

Если нужно печатать или создавать строку с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.print("[" + "." + Sprites.selectSprite(simulationMap.getEntity(coordinates)) + "."  + "]");

//ПРАВИЛЬНО:
Entity entity = simulationMap.getEntity(coordinates);
String sprite = Sprites.selectSprite(entity);
System.out.printf("[.%s.]", sprite);
```

**18. class Simulation**

Из-за того, что Action'ы в проекте сделаны неправильно, то и используются они здесь не так, как должны были бы
```
private final EntitySpawnerAction entitySpawnerAction = new EntitySpawnerAction();
private final MoveAction moveAction = new MoveAction();
private final MaintainAction maintainAction = new MaintainAction();

entitySpawnerAction.spawnEntities(simulationMap);
moveAction.makeMove(simulationMap);
maintainAction.checkAndAddEntities(simulationMap);

//ДОЛЖНО БЫТЬ ТАК:
private final List<Action> initActions = List.of(new EntitySpawnerAction());
private final List<Action> turnActions = List.of(new MoveAction(), new MaintainAction());

executeActions(initActions);
executeActions(turnActions);

private void executeActions(List<Action> actions) {
  for(Action a : actions) {
    a.execute(simulationMap);  
  }
}
```
Использование Action'ов через полиморфизм, позволяет делать более простую и гибку архитектуру.

- Большой метод  `startSimulation()` - 40 строк. 

Если метод больше 10-20 строк, значит он делает несколько разных дел. 
Каждый метод должен делать только одно дело.  
Например, смотрим на код большого метода и находим отдельные смысловые блоки
```
public void startSimulation() {
  while (true) {
   //миллион строк
   try {
     Thread.sleep(1000);
    } catch (InterruptedException e) {
      throw new RuntimeException(e);
    }
  }
}
```
Выносим эти блоки во вспомогательные методы
```
public void startSimulation() {
  while (true) {
   //миллион строк
   pause();
  }
}

private void pause() {
  try {
    Thread.sleep(1000);
  } catch (InterruptedException e) {
    throw new RuntimeException(e);
  }
}
```
*Мартин "ЧК, гл.3"*  
*Фаулер "Рефакторинг", гл.6, "Выделение метода"*

**19. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

Сделать Action'ы по ТЗ.

n.87(195)  
#ревью #симуляция #проходимость 