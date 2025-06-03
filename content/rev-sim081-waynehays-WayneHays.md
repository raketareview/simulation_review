https://github.com/WayneHays/Simulation  
[Wayne Hays]

Много хороших классов, много плохих классов.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Распечатка информации о действиях, происходящих в экшенах, происходит через использование паттерна "Слушатель"

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Непонятно, почему тут два названия-концепции вместо одного. "Координата" и "позиция" как синонимы 
```
Coordinates position

//ЛУЧШЕ:
Coordinates coordinates
```

- Судя по названию, метод валидирует счетчик. Почему именно счетчик? В методе нет специфического кода, который бы отсылал именно к концепции счетчиков.  
Метод просто валидирует любое входящее значение
```
static int validatePositiveCount(int count)

//ПРАВИЛЬНО:
static int validatePositive(int value)
```

- Не называй пакеты так, как называются стандартные классы и интерфейсы. Например, пакет `map`
```
map 

//ПРАВИЛЬНО:
game_map
```

- Пакеты называй или во множественном числе, или в одиночном
```
actions
event

//ПРАВИЛЬНО ТАК:
action
event 

//ИЛИ ТАК:
actions
events 
```

- `enum ActionName` не имеет никакого отношения к классу `Action`, который есть в проекте. 
Это слово в названии енама нужно заменить на какое-то другое, чтобы своим названием не отсылало к посторонней концепции экшенов.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется.  
В этом случае неважно, будет else или нет, так как программа будет работать одинаково и код без `else` будет выглядеть читабельней
```
if (handler != null) {
  return handler.apply(event, event.getData());
} else {
  throw new IllegalArgumentException(NO_HANDLER_TEMPLATE.formatted(actionName));
}

//ПРАВИЛЬНО:
if (handler != null) {
  return handler.apply(event, event.getData());
} 
throw new IllegalArgumentException(NO_HANDLER_TEMPLATE.formatted(actionName));
```

**3. record Coordinates(int row, int column)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**4. class GameMap**

- Если в конструкторе есть методы, которые могут выкинуть исключение, то для создания объекта рекомендуется использовать фабричный метод
```
public GameMap(int height, int width) {
  validateSize(height, width);  <- МОЖЕТ ВЫКИНУТЬ IllegalArgumentException
  //...
}

//ЛУЧШЕ:
private GameMap(int height, int width) {
  //...
}

public static GameMap of(int height, int width) {
    validateSize(height, width);  <- МОЖЕТ ВЫКИНУТЬ IllegalArgumentException
    return new GameMap(height, width);
}
```
Выброс исключения в конструкторе в большенстве случаев не принесет проблем, но возможны специфические ситуации, когда проблемы все-таки будут.  
Замечание дискуссионное и зелёное по [классификации Фёдора](https://gist.github.com/losevskiyfz/69979db2bb6026ebc548366fbc573d5f). 

+ 👍 При всех операциях с координатой, происходит валидация координаты, это хорошо
```
public void remove(Coordinates position) {
  validateBounds(position, height, width);
  entityPositions.remove(position);
}
```

+ 👍 Геттер не возвращает null
```
public Optional<Entity> get(Coordinates position)
```

- Непонятно, почему метод принимает размеры карты во входящие аргументы, а не берет из окружения.  
Этот метод проверяет вхождение координаты в геометрию текущего экземпляра карты, а значит размеры карты должен брать из окружения
```
private boolean isInBounds(Coordinates position, int height, int width) {
  return position.row() >= 0 && position.row() < height &&
    position.column() >= 0 && position.column() < width;
}

//ПРАВИЛЬНО:
private boolean isInBounds(Coordinates position) {
  return position.row() >= 0 && position.row() < this.height &&
    position.column() >= 0 && position.column() < this.width;
}
```

+ 👍 Класс содержит только необходимые методы для выполнения единой ответственности.

**5. class EntityLocator**

Утилитный класс, который содержит вспомогательные методы для работы с картой. 
Например, поиск соседних ячеек вокруг координаты. Претензий к классу нет.  

👍 Утилитный класс, а поэтому правильно, что final и закрытный конструктор.

**6. interface PathFinder**

👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.
```
public interface PathFinder {
  List<Coordinates> findPath(GameMap gameMap, Coordinates start, Class<? extends Entity> target);
}
```

**7. class BFSPathFinder implements PathFinder**

👍 Поиск разбит на вспомогательные методы, алгоритм хорошо читаем.

**8. Пакет entities**

Содержит не только существа, но и фабрику существ. Фабрика это не существо, поэтому не должна лежать в папке "существа".

**8. abstract class Creature implements Entity**

- Непонятно, почему для совершения хода креатуре сообщают класс существа, до которого нужно идти. Креатура и так знает класс существ, которые она ест
```
protected Class<? extends Eatable> foodType; <-- КРЕАТУРА И ТАК ЗНАЕТ СВОЮ ЕДУ

public Coordinates makeMove(GameMap gameMap, Class<? extends Eatable> target) <-- ЗАЧЕМ КРЕАТУРЕ ГОВОРИТЬ, КАКУЮ ЕДУ ЕЙ ИСКАТЬ?

//ЛУЧШЕ:
public Coordinates makeMove(GameMap gameMap)
```
Существующий подход потенциально багоопасен, потому что креатуре можно в метод совершения хода отправить не тот класс существа, которое оно ест. 
Например, Волку отправить класс цветка в качестве цели совершения хода
```
public static void main(String[] args) {
  Wolf wolf = new Wolf(new BFSPathFinder());
  GameMap gameMap = new GameMap();
  gameMap.addEntity(wolf, new Coordinates(1, 1));
  gameMap.addEntity(new Flower(), new Coordinates(5, 5));
  wolf.makeMove(gameMap, Flower.class); <-- ВОЛК БУДЕТ ЕСТЬ ЦВЕТОЧЕК
}
```

- Метод не совершает ход, физически перестановку существа по карте производит кто-то другой(`class Movement`). Метод рассчитывает координату для совершения хода
```
public Coordinates makeMove(GameMap gameMap, Class<? extends Eatable> target)
```

**9. class EntityFactory**

+ 👍 Фабрики- это хорошо.

- Нет смысла константы с количеством создаваемых существ выносить в отдельный класс с константами.  
В этом виде выглядит как искуственное разделение единой сущности на две. Перенеси константы из `SimulationParameters` в саму фабрику
```
List<Flower> flowers = createEntities(Flower::new, SimulationParameters.GRASS_COUNT);
```
Про классы констант, конфигурации и их использование я писал [тут](https://t.me/zhukovsd_it_chat/53243/176984)

**10. class QuantityValidator**

Валидатор должен просто валидировать, то есть в данном случае просто бросать исключение если значение не выдерживает проверку.  
Сейчас это не валидатор, а валидатор и *что-то еще*
```
public static int validatePositiveCount(int count) {
  if (count > 0) {
    return count;
  }
  throw new IllegalArgumentException(AMOUNT_NEGATIVE_TEMPLATE.formatted(count));
}

//ПРАВИЛЬНО:
public static void validatePositiveCount(int count) {
  if (count <= 0) {
    throw new IllegalArgumentException(AMOUNT_NEGATIVE_TEMPLATE.formatted(count));
  }
}
``` 

**11. Пакет actions**

С Action'ами ты откровенно пошел в разнос. Множество классов, разобраться в них мне сложно  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim081/img0.png)

Что здесь происходит, точно логика Симуляции требует таких сложных решений? Я не уверен в этом
```
private <U extends T> void moveAllEntitiesOfClass(GameMap gameMap, Class<U> clazz) {...}
```
Логика реализации экшенов тут мне видится переусложненной.

**12. abstract class Action**

👍 На Action's можно подписаться, паттерн "Слушатель". 
```
public abstract class Action {
  //...
  public abstract void execute(GameMap gameMap);
  //Ниже- паттерн "Слушатель"
  public void addListener(ActionEventListener listener) {...}
  public void removeListener(ActionEventListener listener) {...}
  protected void notifyListeners(Object... data) {...}
}
```
Благодаря этому информация о совершенных действиях, например ходе существ, распечатывается не в самих экшенах, что было бы нарушением SRP, а в специально уполномоченных на то классах
```
[CreatureMovement] -> [Rabbit] moved from [4,10] to [4,12]
[CreatureMovement] -> [Rabbit] moved from [6,10] to [5,9]
[CreatureMovement] -> [Rabbit] moved from [8,1] to [9,2]
```

**13. Логика совершения хода существами**

Логика совершения хода существами разделена на несколько классов и я, если честно, запутался, пробуя ее проследить.

Часть логики совершения хода находится в `Creature`. Остальная часть- в многочисленных экшенах. 

Например, в  `abstract class Interaction<T extends Entity, U extends Entity> extends Action`.  
Из сигнатуры класса `Interaction` и из методов, которые в нем находятся, я совершенно не могу понять, что это такое и как его нужно использовать.  
Но вот я вижу метод, который судя по названию, ищет цель(еду) в соседних координатах
```
private Optional<U> findTargetInNeighborsCells(GameMap gameMap, Coordinates actorPosition) {
  //миллион строк кода
}
```
Сама физическая перестановка существ в карте совершается в классе `Movement`.

**14. Пакет event**

Концепция событий мною осталась не понята. Тут тоже все сложно, как и в `actions`  
![pic](https://github.com/raketareview/simulation_review/blob/master/content/resources/rev-sim081/img1.png)

**15. Меню и диалоги**

😎🤝😎

**16. interface Renderer**

👍 Интерфейс рендерера это хорошо. Теперь можно делать разные рендереры для разных визульных сред(консоль, интерфейс виндовс, http, матричный принтер etc)
и разного отображения информации(цветной, черно-белый и пр.)
```
public interface Renderer {
  void render(GameMap gameMap);
}
```

**17. class Simulation**

👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
public Simulation(GameMap gameMap, Renderer renderer, List<Action> initActions, List<Action> turnActions)
```

**18. Создание экземпляра через классы Main и SimulationConfigurator**

Что класс `Main` большой и сложный, что `SimulationConfigurator`, что их взаимоотношения- тоже сложные.  
Если создание экземпляра класса `Simulation` требуют таких серьезных усилий, сделай фабрику симуляции.  

Пусть эта фабрика хранит в себе все правила создания экземпляра симуляции для игры.  
Сейчас эта единая сложная логика создания экземпляра симуляции разделена между классами `Main` и `SimulationConfigurator`.

## АРХИТЕКТУРА

Сорян, но по моему мнению реализация экшенов и евентов тут никуда не годится.  
Возможности реализованого функционала не соответствуют такой сложности кода.

Что касается паттерна "Слушатель", то если цель была попрактиковаться с паттерном, то хорошо.  
Но лично мне не очень зашла здесь и реализация этого паттерна, и само применение слушателя в виде возможности подписать большое количество подписчиков и т.д.

👍👍👍 При этом очень хорошо, что информацию ты не распечатываешь в самих экшенах и тем самым отделаешь бизнес логику от представления. 

Мое альтернативное предложение: в данном конкретном случае **вместо "Слушателя" использовать "Обратный звонок"**.

**Общая идея такова.**

Action передвижения существ передвигает существ по карте, как игрок шахматные фигуры на доске- сейчас это твоя концепция, ок.   
Во время совершения хода нужно распечатывать сообщение о том, какое существо откуда и куда совершило ход.

Делаем соответствующий экшен и внедряем в него обратный звонок.
```
public interface Action {
  void execute(GameMap gameMap);
}

public class MoveAction implements Action{
  //...
  private CallBack callBack;

  @Override
  public void execute(GameMap gameMap) {
    //из карты получаем creatures - список креатур 
    for(Creature creature : creatures) {
      /*
       Здесь передвинули creature
       с позиции Coordinates from
       на позицию Coordinates to
       */

      //теперь делаем обратный звонок
      if(callBack != null) {
        callBack.perform(creature, from, to);
      }
    }
  }

  public void setCallBack(CallBack callBack) {
    this.callBack = callBack;
  }

  public interface CallBack {
    void perform(Creature creature, Coordinates from, Coordinates to);
  }
}
```

Для простоты представим, что экшены создаются в классе `Simulation`, а не инжектятся ему в конструктор.  
Инжектить в конструктор ненамного сложнее, но не будем заморачиваться, а выделим суть.

Так вот, при создании MoveAction в классе Simulation, к нему на телефонную трубку привяжем метод распечатки информации о совершенном ходе
```
public class Simulation {

  private final GameMap gameMap;
  private final List<Action> turnActions = new ArrayList<>();
  //другие поля

  public Simulation(GameMap gameMap) {
    this.gameMap = gameMap;

    MoveAction moveAction = new MoveAction();
    moveAction.setCallBack(this::printMove);
    turnActions.add(moveAction);
    //...
  }

  private void printMove(Creature creature, Coordinates from, Coordinates to) {
    System.out.printf("[%s] moved from %s to %s \n",
        creature.getClass().getSimpleName(),
        from.toString(),
        to.toString()
        );
  }

  //другие методы
}
```

И теперь при каждом совершении хода существом, в консоль будет печатать такую инфу
```
[Wolf] moved from [0:0] to [3:3] 
```

Про паттерн Callback я также писал тут:  
[Callback луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)  

## ВЫВОД

Тем не менее, я не предлагаю прямо сейчас рефакторить евенты и экшены в проекте.
Наоборот, переходи к третьему проекту.  
Во-первых, тебя явно утомил второй проект и нужно передохнуть от него.  

Во-вторых, через 2-3 недели посмотри на проект свежим взглядом.  
Если для тебя тогда все так же будет понятно, как работают евенты и экшены, значит с ними все ок и моё бухтение необосновано.  
А если ты потом сам не сможешь понять, как это все работает, то отрефактори и сделай проект более простым и понятным.

Другие, более простые замечания, при желании можешь отрефакторить сейчас, до перехода к новому проекту.  
В любом случае, практика с паттернами пошла тебе в плюс.

n.81(180)  
#ревью #симуляция #паттерн #слушатель #callback 