https://github.com/Ephirious/Simulation  
[Данила]

Есть над чем поработать

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Карта в консоли распечатывается кривовато
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim075/img0.png)

## ХОРОШО

+ 👍 Несколько алгоритмов поиска: BFS и AStar
+ 👍 Механика фотосинтеза у травы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Метод обещает верифицировать ключ, а верифицирует координату: `void verifyKey(Coordinates coordinates)`.  
Если вникнуть в код класса, то становится понятно, почему происходит "верификация ключа"- в классе Карты есть мапа, в которой ключ это координата, а значение- сущность. 
Но это всё подробности внутреннего устройства класса, в которое не должен вникать клиент.  
В данном случае с точки зрения клиента метод проводит не верификацию какого-то таинственного ключа.  
А верификацию координаты на предмет того, занята она или нет. Этот функционал и нужно отразить в названии метода.

- Название будет путать. Встречаясь в коде, переменную `map` будут считать реализацией стандартного интерфейса `Map`, а не экземпляром пользовательского класса `SimulationMap`
```
SimulationMap map

//ПРАВИЛЬНО:
SimulationMap simulationMap
```

- UPPER_SNAKE только для констант, а это не константа
```
final int CREATURES_HEALTHPOINTS = 100;
final int MAP_ROW = 20;
```

- Почему 'a' но не 's'? Так хотя бы логика намерений прослеживалась- `s` как первая буква `Simulation`
```
Simulation a = new Simulation();

//ПРАВИЛЬНО:
Simulation simulation = new Simulation();
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (!entities.containsKey(coordinates))
  throw new IllegalArgumentException(...);

if (current.equals(target))
  break;

//ПРАВИЛЬНО:
if (!entities.containsKey(coordinates)) {
  throw new IllegalArgumentException(...);
}

if (current.equals(target)) {
  break;
}
```

**3. Этот код совсем нечитаем**, не нужно писать код в одну строку в ущерб читаемости. Вводи поясняющие переменные
```
currentSprite = spritesForEntity.getSprite(renderingMap.get(currentCoordinates).getClass());

setCoordinates(shifts.get(coordinatesRandomizer.nextInt(shifts.size())));
```
*Фаулер, "Рефакторинг", гл.6,п."Введение поясняющей переменной"*  

**4. Что здесь вообще происходит?** 

- Невозможно понять этот ад с участием тернарника. 
Писать код в одну строку это прикольно. Но здесь лучше будет смотреться код из трех-четырех строк, из которых будет понятно, что здесь происходит
```
int countSteps = (pathToTarget.size() <= speed.getSpeed()) ? pathToTarget.size() - 1 : speed.getSpeed();
```
- Этот фор сам автор не сможет понять через неделю после того, как написал программу. Вводи поясняющие переменные
```
for (int damageIndex = values()[0].ordinal(); damageIndex < this.ordinal(); damageIndex++) {...}
```

**5. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (!path.isEmpty() && path.size() < minPathSize) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет()) {...}

private boolean isНазваниеКотороеВсеОбъясняет() {
  return !path.isEmpty() && path.size() < minPathSize;
}
```

**6. Нарушение DRY**, магические буквы, числа, слова. Вводи константы. 
Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов, либо перенеси ее в третий класс
```
public class SimulationRenderer {
  //... 
  private void renderControlRules() {
    System.out.println("Control keys: (p - pause; c - continue; r - restart; q - quit) + ENTER"); <-- ЭТИ МАГИЧЕСКИЕ БУКВЫ НУЖНО СИНХРОНИЗИРОВАТЬ С КОНСТАНТАМИ В Listener
  }
}

public class Listener implements Runnable {
  private static final char SYMBOL_FOR_PAUSE = 'p';
  private static final char SYMBOL_FOR_CONTINUE = 'c';
  private static final char SYMBOL_FOR_CLOSE = 'q';
  private static final char SYMBOL_FOR_RESTART = 'r';
  //...
}
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**7. Разделение класса по пакетам**

Классы проекта распределены по пакетам. Название многих пакетов не соответствует тому, что они в себе содержат.

- Пакет `util` содержит только один класс, который можно считать утилитным: `class SpritesConstructor`. Остальные- не утилиты. Гугли, что в java понимается под утилитным классом.
- Пакет `service` тоже содержит классы, которые трудно отнести к сервисам, например, `SimulationMap`. 
Здесь `SimulationMap` это модель, в данном случае- богатая модель. 

Сервис карты как класс, отдельный от модели Карты, в принципе может существовать. 
Но для этого класс Карта должен быть анемичной моделью. 
В `Spring`🌱 есть разделение в том числе на слои `service` и `entity` именно по причине того, что спринг построен по MVC с анемичной моделью.

Конечно, никто не может запретить тебе понимать под "сервисом" и "утилитой" что-то свое, но лучше пребывать в консенсусе с сообществом.

**8. class Coordinates**

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Класс может быть преобразован в record без потери функционала.

**9. class SimulationMap**

+ 👍 Норм метод
```
public <T extends Entity> List<T> getEntitiesByType(Class<T> clazz)
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
Здесь `verifyKey(coordinates)` это верифиукация того, что координата не занята, а не того, что координата находится в пределах карты
```
public Entity get(Coordinates coordinates) {
  verifyKey(coordinates);  
  return entities.get(coordinates);
}
```

- Нарушение DRY, не дублируй код, пользуйся методами, которые уже есть в классе
```
private void verifyKey(Coordinates coordinates) {
  if (!entities.containsKey(coordinates))
    throw new IllegalArgumentException("SimulationMap: hasn't entity by coordinates " + coordinates);
}

//ПРАВИЛЬНО:
private void verifyKey(Coordinates coordinates) {
  if (!hasEntity(coordinates))
    throw new IllegalArgumentException("SimulationMap: hasn't entity by coordinates " + coordinates);
}
```

+ 👍 В целом, к классу мало замечаний.

**10. abstract class PathFinder**

+ 👍 Абстрактный класс поиска, который позволяет делать разные варианты поиска, например, AStar и BFS, это хорошо.

- Из сигнатуры метода совсем неясно, как пользоваться поиском. 
Потому что в поиск передается не только координата старта, но и координата того существа, к которому нужно проложить путь.  
Следовательно, клиентский код должен каким-то образом предварительно уже провести часть работы по поиску пути- он должен самостоятельно найти координату цели
```
public List<Coordinates> find(Coordinates source, Coordinates target) 
```
Это нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям.  
Например, так
```
public List<Coordinates> getPatch(Карта карта, Coordinates start, Class<? extends Entity> target) {...}
```
Если поиску AStar нужно знать координаты конечных целей, то пусть этот класс сам их предварительно и найдет, а не отдает эту часть свой ответственности сторонним классам. 

**11. abstract class Entity**

Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня Creature.

**12. abstract class Creature extends Entity**

- Класс не должен инициализировать сам себя. В данном случае класс инициализирует сам себя тем, что сам себе устанавливает тип поиска  
Конкретную реализацию поиска класс должен принимать в конструктор
```
PathFinder finder = new AStar(map);

//ПРАВИЛЬНО:
public Creature(PathFinder finder, /* другие аргументы */)
```
*Мартин, "ЧК", гл.11.2* 

- Потомки креатуры едят какаую-то еду. Следовательно, они хранят у себя эту информацию, хоть и в довольно странном виде
```
findNearestPath(map, Grass.class);     <-- код в Зайце
findNearestPath(map, Herbivore.class); <-- код в Волке
```
Любой потомок креатуры ест еду. Следовательно, это общая информация для любой Креатуры, а значит это поле должно храниться в самой креатуре. Примерно так:
```
private final Class<Entity> food;

public Creature(Class<Entity> food, /* другие аргументы */) {
  this.food = food;
  //...  
}
```

**13. class Predator/Herbivore extends Creature**

- Если предок,то есть `Creature`, будет в себе хранить класс еды, как оно должно быть, то не придется переопределять метод `makeMove(...)` у его потомков. Вполне достаточно будет метода `void makeMove(...)` в Креатуре
```
@Override
public void makeMove(SimulationMap map) {
  findNearestPath(map, Grass.class);
  super.makeMove(map);
}
```

- Магическое число: `int hpForGrass = 10;`

**14. class PlaceHerbivore/Predator/....Command extends AbstractPlaceCommand implements Command**, группа классов по заселению карты существами

- Здесь в сигнатуре не нужно писать `implements Command`, потому класс уже расширяет `AbstractPlaceCommand`, а он в свою очередь имплементирует `Command`.

- Нарушение DRY, множество классов, которые делают одно то же- заселяют карту определенным видом существ.
Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс, который будет принимать в конструктор количество создаваемых существ и способ их создания.
Способ создания можно передавать через стандартный интерфейс Supplier.

Использоваться это будет например так
```
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(getRandomCoordinates()), КОЛИЧЕСТВО_ЗАЙЦЕВ);
заяцSpawnAction.perform(карта); 
//...

public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      //добавить существо в карту
    }
  }
}
```

**15. class MoveCreatureCommand extends AbstractTurnCommand**

Класс должен двигать всех креатур сразу. А не так, как сейчас- отдельно каждый вид креатур
```
manager.addCommand(new MoveCreatureCommand(map, Herbivore.class)); <-- двигаем Зайца
manager.addCommand(new MoveCreatureCommand(map, Predator.class));  <-- двигаем Волка
```
В текущем виде, при добавлении в проект новых наследников Креатуры, придется под каждый повый вид создавать отдельный экземпляр Мувера.

**16. class Listener implements Runnable**

Этот класс- не реализация паттерна "Слушатель". Не называй свои классы именами стандартных паттернов GoF/GRASP, если они не реализуют эти паттерны.

**17. class SpritesConstructor**

- Класс имеет только один статический метод. Тогда класс должен быть final и иметь приватный конструктор, чтобы нельзя было от него унаследоваться или создать экземпляр его класса.

- Стандартным явлениям давай стандартные названия. Фактически, этот класс это простая статическая фабрика спрайтов. Фабрики принято называть `Factory`. Я этот класс сделал бы примерно так
```
public final class SpritesFactory {
  //закрытый конструктор  
  public static Sprites get() {...}
}
```

**18. class SimulationRenderer**

- Этот метод распечатывает не доску, а сопроводительную информацию. Доску распечатывает `void renderMap()`
```
private void renderBoard() {
  System.out.println("Turn - " + board.getCountTurns());
  System.out.println("Number of grasses - " + board.getCountGrasses());
  //...
}
```
Да, если глубоко вникнуть, то становится понятно, что имеется ввиду "информационная доска", типа табло на вокзале. Но двояких толкований и непоняток не возникло, если бы метод назывался просто `renderInfo()`.

- Результат работы рендерера довольно странный. Для карты размером 20x20 ячеек, распечатанная в консоль картинка даже не стремится к квадратной форме
```
▫️▫️🪨🪨▫️🐑▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️🌳▫️🌱
▫️▫️🪨▫️▫️▫️▫️🌱▫️▫️▫️🌱▫️▫️▫️▫️🪨▫️▫️🌱
🪨🐑▫️▫️▫️▫️▫️🌱▫️▫️▫️🌱▫️▫️▫️▫️▫️▫️▫️🌱
🌱🌳▫️▫️▫️▫️🦁🐑▫️▫️▫️🌱🌳▫️▫️▫️▫️▫️▫️🌱
🌱▫️▫️🌳🐑▫️▫️▫️🦁▫️▫️🌱▫️▫️▫️▫️▫️▫️▫️▫️
🐑▫️▫️▫️▫️▫️🌳▫️🐑▫️🌳🌱▫️▫️🪨▫️▫️🌳🌳▫️
▫️▫️▫️🌱▫️🪨▫️▫️▫️🪨▫️▫️▫️▫️▫️🌱🌱🪨▫️🌱
▫️▫️▫️▫️▫️▫️▫️▫️🌳▫️▫️▫️🌱▫️▫️🌱🌱▫️🌱🌱
▫️▫️🪨▫️▫️🐑🦁▫️▫️▫️▫️▫️🌱▫️▫️🌱🌱▫️🌱🌱
▫️▫️▫️▫️▫️🪨▫️▫️▫️▫️▫️▫️🐑▫️🪨▫️🌱▫️▫️🪨
▫️▫️▫️▫️▫️▫️▫️▫️🌱▫️▫️▫️▫️🪨▫️▫️🌱🪨▫️▫️
▫️▫️🌳🌱▫️▫️▫️🌱🐑🦁▫️▫️🪨▫️▫️🌳▫️▫️▫️🌳
▫️▫️▫️▫️▫️▫️▫️🐑▫️▫️🪨▫️▫️▫️▫️▫️▫️🌳▫️▫️
▫️▫️▫️🌳▫️▫️▫️▫️▫️▫️▫️▫️🌳▫️▫️▫️▫️▫️🐑▫️
▫️▫️🐑▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️🌱▫️▫️▫️▫️
▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️🌱▫️▫️▫️🌱🌱🦁🐑▫️▫️
▫️▫️🌳▫️▫️🌳▫️▫️🌱▫️🌳▫️🌱🌳🌱▫️▫️▫️▫️▫️
▫️▫️▫️▫️▫️▫️🌱▫️▫️▫️▫️▫️🌱▫️🐑▫️▫️▫️▫️▫️
▫️🪨🌱▫️▫️▫️🌱▫️▫️🪨▫️▫️▫️▫️▫️▫️▫️▫️▫️🐑
▫️▫️🌱▫️▫️🌱🌳▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️▫️🪨▫️
```

**19. class Simulation**

- Нарушение DI. Класс не должен инициализировать сам себя. 
В данном случае класс инициализирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
```
this.map = new SimulationMap(MAP_ROW, MAP_COLUMN);
```
Таким образом нельзя создать несколько игровых конфигураций с разными картами.  
*Мартин, "ЧК", гл.11.2*

- Какая-то сложная система с использованием менеджера комманд. Мне неясно, какие преимущества это дает по сравнению с обычным списком экшенов/команд.  
Пока я вижу только недостатки: дополнительный класс `CommandManager` в проекте, пересоздание экземпляров комманд на каждом ходе
```
while (!isClosed && !isPaused) {
  manager.addCommand(new MoveCreatureCommand(map, Herbivore.class));
  manager.addCommand(new MoveCreatureCommand(map, Predator.class));
  manager.addCommand(new GrowGrassCommand(map));
  //...
  manager.executeAll();
  //...
}

//ЧЕМ ЭТО ЛУЧШЕ ОБЫЧНОГО?:
private final List<Action> turnActions = List.of{ 
    new ДатьЗайцамСигаретуAction(), 
    new MoveAction()
  };

while (!isClosed && !isPaused) {
  for(Action a : turnActions) {
    a.execute();  
  }
}
```

**20. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

- Плохо то, что майн не инжектит необходимые зависимости в Симуляцию.  
Сейчас именно в майне должен создаваться пустой экземпляр карты(без существ) и инжектиться в Симуляцию.  
Также именно в майне сейчас должен создаваться экземпляр конкретной реализации Поиска и инжектится в симуляцию.  
Примерно так
```
public static void main(String[] args) {
   Карта карта = new Карта(100, 100);
   Поиск поиск = new AStar(); 
   Simulation simulation = new Simulation(карта, поиск);
   simulation.startSimulation();
}
```

## ВЫВОД

Тренировать нейминг. Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.75(164)  
#ревью #симуляция