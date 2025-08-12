https://github.com/cakeslayer00/nature-lifecycle-simulation  
[v]

Есть над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию.  
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```
HashMap<Coordinate, Entity> map;

//ПРАВИЛЬНО:
HashMap<Coordinate, Entity> entities;  //или coordinateWithEntities
```

- Для одной концепции используй одно название
```
Coordinate position

//ПРАВИЛЬНО:
Coordinate coordinate
```

- Не называй метод "run" если он не реализует `Runnable.run()`- название этого метода ассоциируется с этим стандартным интерфейсом и применением потоков. 
Здесь этот метод не имеет отношение к `Runnable.run()`
```
public class Simulation {
  //...
  public void run() {...}
}

//ПРАВИЛЬНО: start(), go(), execute(), perform(), поехали() 
```

- Название "map", встречаясь в коде, будет вводит в заблуждение- все будут думать, что это реализация стандартного Map
```
GameMapImpl map = new GameMapImpl();

//ПРАВИЛЬНО:
GameMapImpl gameMap = new GameMapImpl();
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinate, Entity> map;

//ПРАВИЛЬНО:
Map<Coordinate, Entity> entities;
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д. 
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. Но это уже нюансы.

**3. Нарушение конвенции кода**. В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (entity == null) continue;
if (path.isEmpty()) return new MoveIntent(Intent.STUCK, position);

//ПРАВИЛЬНО:
if (entity == null) {
  continue;
}
if (path.isEmpty()) {
  return new MoveIntent(Intent.STUCK, position);
}
```

**4. record Coordinate(int x, int y)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```
public record Coordinate(int x, int y) {
}
```

**5. interface GameMap**

+ 👍 Интерфейс для карты это хорошо. Можно делать ее разные реализации, например, на массиве и хешмапе.

- Уже по методам интерфейса видно, то карта неуниверсальна и будет нарушение инкапсуляции- не хватает методов, которые скажут размеры карты. 
А значит, карта свои фаберже хранит в чужом серванте и кто-то другой решает, какого размера она будет
```
public interface GameMap {
  Entity getEntity(Coordinate coordinate);
  void putEntity(Coordinate coordinate, Entity entity);
  void removeEntity(Coordinate coordinate);
  boolean contains(Coordinate coordinate);
}
```

**6. class GameMapImpl implements GameMap**

- Назвать класс по принципу ИмяИнтерфейсаImpl это самый ленивый вариант. 
В данном случае в названии класса лучше отразить его внутреннюю суть- хранение данных в хешмапе: HashGameMap. 
Потому что может быть другая реализация интерфейса- на основе не хешмапы, а массива.

- Нарушение инкапсуляции. Свои собственные размеры карта не хранит в себе, а берет из констант стороннего класса
```
for (int i = 0; i < Simulation.HORIZONTAL_BOUND; i++) {
  for (int j = 0; j < Simulation.VERTICAL_BOUND; j++) {...}
}
```
Свои размеры карта должна хранить сама. Размеры не должны быть жестко фиксированы в константах, а должны приходить в конструктр карты. 
Для того, чтобы ката была универсальной и можно было создавать разные игровые конфигурации с разным размером карт.

- Кстати, интересная мелочь: для циклов традиционно применяются индексы i, j. Но бывают ситуации, когда намного нагляднее будут смотреться другие индексы. Например:
```
for (int y = 0; y < Simulation.HORIZONTAL_BOUND; y++) {
  for (int x = 0; x < Simulation.VERTICAL_BOUND; x++) {
    map.put(new Coordinate(x, y), null);
  }
}
```

- Карта не должна быть заполнена "пустыми записями", которые хранят соотношение координата-null. 
Иначе теряется все преимущество, которое дает HashMap для хранения сущностей
```
map.put(new Coordinate(x, y), null);
```

Фактически, тут HashMap становится аналогом обычного массива, только хуже, бесполезно забивая память.
Например, есть игровая доска 100*100 и в ней единомоментно находится общим числом 150 разных существ: камней, травы, зайцев и т.д.
Тогда при нормальном хранении этих данных в мапе с помощью ключ-значение, потребуется 150 записей.
Если в этой же мапе кроме этих значимых данных хранить еще пустые сущности, то потребуется уже 10.000 записей.
Для определения того, что в точке карты никого нет, достаточно ввести метод 
```
private boolean isПустаяЯчейка(Координата координата) {...}
```

- Никогда не возвращай null
```
private final HashMap<Coordinate, Entity> map;

public Entity getEntity(Coordinate coordinate) {
  return map.get(coordinate);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean contains(Coordinate coordinate) {
  return map.get(coordinate) != null;
}
```
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что свободна. 
А правильный ответ- такой координаты в карте вообще нет.

- В классе есть нарушение SRP. 
Но оно проявляется не в том, что класс хранит что-то лишнее. 
Нарушение SRP проявляется в том, что класс часть своей ответственности,- хранение собственных размеров,- передал другому классу.

**7. interface SearchService**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.
+ 👍 Сигнатура метода поиска ок
```
public interface SearchService {
  List<Coordinate> search(Coordinate start, Class<? extends Entity> target);
}
```

**8. class BFSSearchService implements SearchService**

👍 Все ок.

**9. abstract class Entity**

Содержит координату. Но координата нужна только тому существу, которое ходит. 
Поэтому entities должны хранить координату только начиная с уровня `Creature`.

**10. abstract class Creature extends Entity**

- Непонятно, зачем классу нужно хранить путь
```
protected List<Coordinate> path;
```
Если класс сам совершает свои ходы, то ему не нужно хранить путь в виде поля класса- 
при ходьбе он должен этот путь получать из класса поиска пути и тут же использовать.

Если класс двигает посторонний класс, то держать путь в креатуре тем более не надо-
тот класс, который двигает креатуру, тоже должен этот путь получать из класса поиска пути.

- Этот метод не совершает ход. Он создает интент(намерение)
```
public MoveIntent makeMove(GameMap map)
```

**11. class Predator/Prey extends Creature**

- Нарушение DRY. Дублирование логики в `makeMove(GameMap map)` у Зайца и Волка. Общий код выноси в предка
```
  //Prey
  public MoveIntent makeMove(GameMap map) {
    //...
    if (entity instanceof Grass) {
      return new MoveIntent(Intent.CONSUME, coordinate);
    }
    //...  
  }

  //Predator
  public MoveIntent makeMove(GameMap map) {
    //...
    if (entity instanceof Prey) {
      return new MoveIntent(Intent.CONSUME, coordinate);
    }
  }

//ПРАВИЛЬНО:
public class Creature бла-бла {

  public MoveIntent makeMove(GameMap map) {
    //...
    if (entity instanceof getTrget()) {
      return new MoveIntent(Intent.CONSUME, coordinate);
    }
  }

  protected abstract Class<? extends Entity> getTarget();
}
```

**12. Пакет util**

+ 👍 Содержит только утилитные классы: имеют только статические методы.
- Утилитные классы должны быть `final` и иметь приватный конструктор- не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр.

**13. class ActionUtils**

Нарушение DRY. Дублирование логики во всех трех методах: `gatherPrey(GameMap gameMap)`, `gatherGrass(GameMap gameMap)` и `gatherCreatures(GameMap gameMap)`.
Сделай один универсальный метод типа такого:
```
public static List<? extends Entity> getEntitiesBy(GameMap gameMap, Class<? extends Entity> clazz) 
```

**14. class FindPathAction extends Action**

- Используй полиморфизм. И Волк и Заяц чем-то питаются, а значит у них можно это спросить 
```
List<Coordinate> path = switch (creature) {
  case Predator _ -> searchService.search(coordinate, Prey.class);
  case Prey _ -> searchService.search(coordinate, Grass.class);
  default -> throw new RuntimeException("Invalid creature type");
};

//ПРАВИЛЬНО:
List<Coordinate> path = searchService.search(coordinate, creature.getFood()); //или .getTarget();
```

- Этот Action не соответствует идее Action'ов, изложенных в ТЗ. 
Потому что этот Action не самодостаточен. Он подготавливает путь, но не выполняет движения. 
А значит, одно действие,- движение существ,- выполняют два экшена, которые функционально зависят друг от друга.
Эти два экшена нужно совместить в один.

**15. class MoveAction extends Action**

Выполняет перемещение креатур по карте. Причем, именно выполняет- двигает их, как шахматист двигает фигуры по доске.
В принципе, почему бы и нет. Но сейчас единый процесс перемещения существа размазан на три класса: `FindPathAction`, `MoveAction` и `makeMove(...)` в `Creature`.

Собери логику движения в одном месте: или в `MoveAction`, или в `Creature`. Потому что за одно конкретное действие должен отвечать кто-то один.
Если вся логика движения будет в креатуре, то `MoveAction` должен будет только инициировать движение.  
Примерно так:
```
class MoveAction реализует Action {

  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeMove(карта);  //даёт пинка
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**16. abstract class SpawnAction extends Action**

- Нарушение DRY. Все методы дублируют друг друга.

- Магические числа.

- Избыточно сложная логика заселения существ.  
Заселить существа очень легко примерно по такому принципу:
```
public class SpawnAction extends Action {
  private final static int КОЛИЧЕСТВО_ТРАВЫ = 10;
  //...  

  @Override
  public void execute(Карта карта) {
    spawn(карта, (coordinates) -> new Grass(coordinates), КОЛИЧЕСТВО_ТРАВЫ);
    spawn(карта, (coordinates) -> new Rock(coordinates), КОЛИЧЕСТВО_КАМНЕЙ);
    //...
  }

  private void spawn(Карта карта, Function<Coordinates, Entity> mapper, int amount) {
    for (int i = 0; i < amount; i++) {
      Coordinates coordinates = getRandomEmptyCoordinates(карта);
      Entity entity = mapper.apply(coordinates);
      карта.put(entity, coordinates);
    }
  }

  private Coordinates getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```

**17. class MaintenanceSpawnAction extends SpawnAction**

Магические числа: 3, 2, 4.

**18. interface Renderer**

👍 Интерфейс рендерера это хорошо. Теперь можно делать разные рендереры для разных визульных сред(консоль, интерфейс виндовс, http, матричный принтер etc)
и разного отображения информации(цветной, черно-белый и пр.)

**19. class ConsoleRenderer implements Renderer**

- Избыточно:
```
case Prey _ -> System.out.print("\uD83D\uDC07");

//ПРАВИЛЬНО:
case Prey -> System.out.print("\uD83D\uDC07");
```

+ 👍 В целом, ок.

**20. class Simulation**

- Нарушение DRY, магические буквы, числа, слова. Вводи константы
```
case "p":
case "h":
System.out.println("Unknown command: '" + input + "'. Type 'h' for help.");
System.out.println("Controls: [ENTER/p] = pause/resume, [q] = quit, [h] = help");

//ПРАВИЛЬНО:
private final static String PAUSE = "p";
private final static String HELP = "h";
//...

case PAUSE:
case HELP:
System.out.printf("Unknown command: '%s'. Type '%s' for help.", input, HELP);
System.out.printf("Controls: [ENTER/%s] = pause/resume, [%s] = quit, [%s] = help", PAUSE, QUIT, HELP);
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

- Симуляция должна принимать необходимые зависимости в конструктор. Например, экземпляр карты.
Сейчас карта создается с фиксированными размерами, которые хранятся в константах Simulation.
Значит, нельзя создать в виде отдельных Main'ов разные конфигурации игры с рзными разными размерами карт.

- Нарушение DI. Раз в проекте есть разные интерфейсы(которые вообще-то вводить было необязательно), типа `Renderer`, 
то Simulation должен получать их в конструктор, а не самостоятельно устанавливать, как это сделано сейчас:
```
GameMapImpl map = new GameMapImpl();
ConsoleRenderer consoleRenderer = new ConsoleRenderer(map);
```

## АРХИТЕКТУРА

ООП здесь часто является карго-культом: интерфейсы не несут полезной нагрузки, а существуют просто для красоты.

В программе есть полезные интерфейсы: `Renderer`, `GameMap` и другие.  
Эти интерфейсы *потенциально* полезны тем, что можно делать их разные реализации и подставлять в проект. 
Например, рендерер эмодзи и рендерер, где существа это буквы.

Но из-за того, что эти интерфейсы в `Simulation` даже не используются в качестве интерфейсов, а юзаются в виде уже готовых классов, польза от них равна нулю.
Например, Карта используется не как интерфейс, а как реализация:
```
GameMapImpl map = new GameMapImpl();
```
Если бы карта использовалась в качестве интерфейса, в этом появился бы минимальный смысл:
```
GameMap gameMap = new GameMapImpl();
```
Но вообще, нужно зависимости формировать в самом верху, в слое `main`, и далее инжектить в симуляцию согласно DI. 
Тогда можно было бы собирать разные игровые конфигурации, не миняя при этом класс `Simulation` и другие нижестоящие классы.  
Например:
```
public class MainFirst {
  public static void main(String[] args) {
    GameMap gameMap = new HashGameMap(10, 15);
    SearchService searchService = new AStarSearchService();
    Renderer renderer = new EmodjiConsoleRenderer();

    Game game = new Game(gameMap, searchService, renderer);
    game.start();
  }
}

public class MainSecond {
  public static void main(String[] args) {
    GameMap gameMap = new HashGameMap(20, 30);
    SearchService searchService = new BfsSearchService();
    Renderer renderer = new TextConsoleRenderer();

    Game game = new Game(gameMap, searchService, renderer);
    game.start();
  }
}
```

## ВЫВОД

Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID.

n.96(216)  
#ревью #симуляция 