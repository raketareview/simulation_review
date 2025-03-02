https://github.com/zmt982/simulation  
[Q]

Есть много над чем поработать.

## ХОРОШО

1. 👍 Симпатичная розовенькая карта при распечатке
2. 👍 Спрайты существ не хранятся в самих существах

## ЗАМЕЧАНИЯ

**1. Нейминг**

- UPPER_SNAKE только для констант, а это не константы
```
public final int X;
public final int Y;
public final int X_SHIFT;
public final int Y_SHIFT;

//ПРАВИЛЬНО ТАК:
public final int x;
public final int y;
public final int shiftX;
public final int shiftY;
```

- Избыточный контекст, который загружает название метода ненужной информацией
```
int getRandomIntInRange(int min, int max)

//ПРАВИЛЬНО:
int getRandom(int min, int max)
```

- Класс с таким названием может делать что угодно: `class Utils`. Название должно объяснять, что делает класс. 

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Если нужно печатать текст** с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.println(this.coordinates + " атаковал " + herbivore.coordinates.toString());

//ПРАВИЛЬНО:
System.out.printf("%s атаковал %s %n", this.coordinates, herbivore.coordinates);
```

**3 Комментарии**  

- Коментарии тут  в основном не несут полезной нагрузки, а констатируют очевидное
```
private int turnCounter = 0; // счетчик ходов
```
Когда в проекте много каментов, это плохо- пользы от них практически нет, они только забивают пространство и мешают читать код.
В идеале, комментариев вообще не должно быть, код должен объяснять сам себя через правильный нейминг и лаконичный код.
*Мартин, "ЧК", гл.4*

**4. Используй List.of**
```
List<Action> initActions = Arrays.asList(
  new InitEntity()
);

//ПРАВИЛЬНО:
List<Action> initActions = List.of(
  new InitEntity()
);
```

**5. class Coordinates**

- Складывать координаты нужно с экземпляром такого же класса
```
public Coordinates shift(CoordinatesShift shift) {
  return new Coordinates(this.X + shift.X_SHIFT, this.Y + shift.Y_SHIFT);
}

//ПРАВИЛЬНО:
public Coordinates shift(Coordinates coordinates) {...}
```

- Нарушение SRP, метод определяет, будет ли координата находиться в геомерических рамках Карты после сдвига координат 
```
public boolean canShift(Coordinates coordinates, CoordinatesShift coordinatesShift)
```

Координата должна только хранить данные для идентификации точки в пространстве. Для координаты не должно быть важно, как ее будут использовать и в каких целях.
Тем более координата не должна зависеть от Рендерера(нарушение Low Coupling) и брать из него данные
```
public boolean canShift(CoordinatesShift shift) {
  //...
  return x >= 0 && x <= Renderer.MAP_X_SIZE && y >= 0 && y <= Renderer.MAP_Y_SIZE;
}
```
Принцип единой ответственности из SOLID гласит, что в классе должна быть одна и только одна причина для изменений.  
Здесь эта единственная причина- хранение данных для идентификации точки в пространстве.  
Вторая, лишняя причина для изменения- размеры карты. Если(когда) изменится расположение данных о размерах карты, из Рендерер.MAP_X_SIZE в Карта.getWidth(), то придется изменять код и в Координате.   
Этот метод нужно убрать из координаты.

- Вообще, идеальная координата это координата, представленная в виде record'а.

**5. class CoordinatesShift**

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**6. class Map**, карта игры

- Следовать ТЗ это хорошо. Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.
Второе приводит к постоянной путанице в коде. `Мар` это название стандартного интерфейса, поэтому меньшим злом является дать другое название прикладному классу: Board, GameMap или что-то подобное.

- В следствие неправильного нейминга приходится прописывать полный путь к интерфейсу Map
```
java.util.Map<Coordinates, Entity> entities = new HashMap<>();
```

- Нарушение SRP, божественный класс, в котором полностью или частично находится много чужих ответственностей.  
Карта должна только хранить существа и обеспечить базовые операции с ними- вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  
Здесь методы чужих ответственностей: 
  - Проверка доступности для перемещения в клетку
  - Самостоятельное создание Существа
  - Подсчет существ
  - Нахождение пути(?!) 
  - Начальная расстановка существ на фиксированных позициях и т.д.

Нужно разделить этот класс на несколько. Как минимум, вынести в отдельный класс поиск пути, начальную расстановку существ на карте вынести в InitAction.

- Карта должна принимать размер в конструктор. Тут сейчас конструктора нет вообще и карта не знает своих границ.

- Никогда не возвращай null
```
java.util.Map<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  return entities.get(coordinates);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- Один раз сделай сдвиговые константы, а не создавай их каждый раз при вызове метода- это неоправданное расходование ресурсов
```
private List<Coordinates> getNeighbors(Coordinates coordinates) {
  List<Coordinates> neighbors = new ArrayList<>();
  int[][] shifts = {
    {0, 1}, {1, 0}, {0, -1}, {-1, 0}, // Вверх, вправо, вниз, влево
    {1, 1}, {1, -1}, {-1, -1}, {-1, 1} // Диагонали
  };

  for (int[] shift : shifts) {
    CoordinatesShift coordShift = new CoordinatesShift(shift[0], shift[1]);
    if (coordinates.canShift(coordShift)) {
      neighbors.add(coordinates.shift(coordShift));
    }
  }
  return neighbors;
}

//ПРАВИЛЬНО:
private static final Coordinates[] SHIFT_COORDINATES = {  
  new Coordinates(-1, 0), //вниз
  new Coordinates(-1, -1), //вниз-влево
  new Coordinates(0, -1),  //влево
  new Coordinates(1, -1),  //вверх-влево
  //oths's
};

private List<Coordinates> getNeighbors(Coordinates coordinates) {
  List<Coordinates> neighbors = new ArrayList<>();

  for (Coordinates shiftCoordinates : SHIFT_COORDINATES) {
    if (coordinates.canShift(shiftCoordinates)) {
      neighbors.add(coordinates.shift(shiftCoordinates));
    }
  }
  return neighbors;
}
```

- Карта не должна сама себя заселять существами, иначе она становится неуниверсальной и нельзя будет создать несколько конфигураций игры с разными комбинациями существ на карте
```
public void setupEntitiesStartPositions() {
  setEntity(new Coordinates(0, 1), new Grass(new Coordinates(0, 1)));
  //еще миллион строк
}
```
Объект не должен сам себя инициализировать.
*Мартин, "ЧК", гл.11,п."Отделение конструирования системы от ее использования"*

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public Entity getEntity(Coordinates coordinates) {
  return entities.get(coordinates);
}

public void removeEntity(Coordinates coordinates) {
  entities.remove(coordinates);
}
```

- Нарушение SRP, Low Coupling. Карта не должна знать логику создания существ и заселять саму себя существами. Карта не должна иметь зависимость на фабрику существ
```
public Entity addNewEntity(String type) {
  while (true) {
    int X = Utils.getRandomIntInRange(0, Renderer.MAP_X_SIZE);
    int Y = Utils.getRandomIntInRange(0, Renderer.MAP_Y_SIZE);
    Coordinates coordinates = new Coordinates(X, Y);
    if (isEmptyCell(coordinates)) {
      Entity entity = EntityFactory.createEntity(type, coordinates); // <-- зависимость на фабрику существ
      setEntity(coordinates, entity);
      return entity;
    }
  }
}
```

- К тому же этот метод нарушает принцип разделения команд и запросов
```
public Entity addNewEntity(String type) {
  //...  
  setEntity(coordinates, entity); // ввод существа в карту- выполнение команды
  return entity;  // возврат данных- выполнение запроса
}
```
Этот метод должен или выполнить действие или выполнить запрос. Но не то и другое вместе.  
*Мартин, "Чистый код", гл.3, п."Разделение команд и запросов"*

**7. class Utils**

- Название класса не соответствует тому, что он делает. Это рандомайзер
```
public class Utils {
  public static int getRandomIntInRange(int min, int max) {
    Random random = new Random();
    return random.nextInt((max - min) + 1) + min;
  }
}

//ПРАВИЛЬНО:
public class MyRandom {
  public static int get(int min, int max) {...}
}
```

**8. class Entity**

- Класс должен быть абстрактным. Не должно быть возможности создать "просто Entity".
- Нарушение инкапсуляции: `public Coordinates coordinates`. Используй поля через геттеры и сеттеры.
- Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entity должен хранить координату только начиная с уровня Creature.

**9. class Creature extends Entity**

- Нарушение SRP. Кто-то другой подготавливает для существа путь, по которому он должен перемещатся. То есть, берет на себя часть ответственности Creature. 
Существо само должно находить этот путь через обращение к классу поиска
```
abstract public class Creature бла-бла {
  //...  
  public void makeMove(List<Coordinates> path, Map map) {...}
}

//ПРАВИЛЬНО:
abstract public class Creature бла-бла {
  private final PatchSearch  patchSearch; 
  //...  
  public void makeMove(Map map) {
    List<Coordinates> path = patchSearch.find();
    //...
  }
}
```

- Нарушение SRP. Существо не должно самостоятельно реализовать поиск путей. Существо должно получить путь из класса поиска пути
```
// получить перечень доступных ходов
public Set<Coordinates> getAvailableMoveCells(Map map) {...}
```

**10. class Herbivore/Predator extends Creature**

+ 👍 Отдельный метод идентификации еды это хорошо. Теперь существо потенциально может есть несколько видов существ
```
@Override
public boolean isEatable(Entity entity) {
  return entity instanceof Herbivore;
}
```

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println(this.coordinates + " атаковал " + herbivore.coordinates.toString());
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack: https://t.me/zhukovsd_it_chat/53243/139594

**11. public class Action**

- Нарушение LSP и вообще дизайна ООП наследования. Метод `addEntities()` используется не всеми потомками Action, а только некоторыми.

- При заполнении карты существами, класс должен сам создавать существо, а не просить Карту это сделать самостоятельно. 
Создавать существ  это не ответственность карты, ее ответственность- просто хранить существ в себе
```
protected void addEntities(Map map, String entityType, int initCount) {
  for (int i = 0; i < initCount; i++) {
    map.addNewEntity(entityType);  //сейчас карта сама создает существо (прим. ред.)
  }
}
```

**12. AddGrassAction, AddHerbivoreAction и т.д.**

- Дублирование кода в классах. Нет необходимости в отдельных спавнерах для разных видов существ. Достаточно сделать один универсальный, типа такого
```
public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int amount;

  public SpawnAction(Supplier<Entity> entitySupplier, int amount) {
    this.entitySupplier = entitySupplier;
    this.amount = amount;
  }

  @Override
  public void execute(Map map) {
    for (int i = 0; i < amount; i++) {
      Entity entity = entitySupplier.get();
      //добавить существо в карту
    }
  }
}
```

Тогда использование универсального спавнера будет выглядеть так
```
SpawnAction grassSpawnAction = new SpawnAction(()-> new Grass(getRandomCoordinates()), GRASS_AMOUNT);
grassSpawnAction.execute(gameMap);
    
SpawnAction rockSpawnAction = new SpawnAction(()-> new Rock(getRandomCoordinates()), GRASS_AMOUNT);
rockSpawnAction.execute(gameMap);
```

- Нарушение Low Coupling. Какие бы данные не понадобились классу для заселения существ в карту, он не должен брать эти данные из Рендерера. 
Потому что рендерер должен только распечатывать карту, а не хранить часть логики по заселению карты существами
```
addEntities(
  map, 
  Renderer.HERBIVORE, <-- тута
  InitEntity.INIT_HERBIVORE_COUNT
);
```

**13. class PredatorTurnAction/HerbivoreTurnAction extends Action**

- Не должно быть отдельных екшенов для запуска движения наследников Creature. 
TurnAction должен просто обойти всю карту, найти каждую креатуру, вне зависимости от ее конкретного вида- Хищник или Травоядное, и дать ей пинка, чтобы она побежала.
При этом как будет бежать креатура и что при этом делать, не должно волновать TurnAction. 
Выглядеть должно примерно так
```
class ХодитьAction реализует Action {

  @Переопределить
  public void execute(Карта карта) {
    List<Creature> creatures = getКреатурыНаКарте(карта);
    for(Creature c: creatures) {
      c.бежатьЖивотное(/* args */);  //даёт пинка
    }
  }

  private static List<Creature> getКреатурыНаКарте(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**14. class Renderer**, распечатка карты

+ 👍 Спрайты существ хранит в константах, это хорошо.
+ 👍 При попытке получить спрайт неизвестного существа бросает исключение, это хорошо.

- Нарушение SRP. Хранит в себе много информации, которую не должен. Например, размер карты. 
Размер карты должна хранить сама карта, а рендерер и все остальные заинтересованные классы должны брать эти данные из карты.

- Метод, который я вообще не могу понять
```
private String getChoice(Entity entity, String chooseGrass, String chooseHerbivore, String choosePredator, String chooseRock, String chooseTree) {...}
```

- Вообще, много запутанной логики в методах. 
Если нужно получать не только изображение, но и цвет в зависимости от существа, можно внутри Рендерера сделать енам и привязать эти данные к имени класса существа
```
class Renderer {
  //...  
  private enum GraphicResources {
   GRASS("🌿", "\u001B[32m");

   private final String sprite;
   private final String color;

   GraphicResources(String sprite, String color) {
     this.sprite = sprite;
     this.color = color;
   }
  }
}  
```

Тогда его использование  
```
String name = Grass.class.getSimpleName().toUpperCase();
GraphicResources grassResources = GraphicResources.valueOf(name);
System.out.println(grassResources.sprite);

//РЕЗУЛЬТАТ:
🌿
```

**15. class Simulation**

+ 👍 Класс принимает достаточное количество зависимостей в конструктор, тем самым можно делать разные игровые конфигурации
```
public Simulation(Map map, Renderer renderer, List<Action> initActions, List<Action> turnActions) {...}
```

+ 👍 Отдельные списки для стартовых и ходовых екшенов это хорошо
```  
List<Action> initActions; // список действий перед стартом симуляции
List<Action> turnActions; // список действий на каждый ход
```

**16. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

- Рендерер не конфигурируется и в проекте нет намеков на то, что могут быть разные рендереры. 
То есть нет интерфейса рендерера, что подразумевало бы возможность делать разные рендереры. 
Поэтому Симуляция должна рендерер в этом виде создавать самостоятельно, а не принимать в конструктор. 
Учитывай паттерн "Creator" - создает класс тот, кто его использует
```
public static void main(String[] args) {
  Renderer renderer = new Renderer();
  //...
  Simulation simulation = new Simulation(map, renderer, initActions, turnActions);
  simulation.startSimulation();
 }
```

- Какой-то мусор
```
int f = 123;
```

## АРХИТЕКТУРА

- В проекте много сложных зависимостей между классами, что приводит к спагетти-коду.
- Слабое понимание принципа единой отвественности, особенно пострадал класс Карты.

Пока еще нет достаточного понимания SRP и принципов правильноого распределения ответственностей, сделай и используй доску только с такими методами
```
public class Board {
  private final Map<Coordinates, Entity> entities = new HashMap<>();
  //...

  public Board(int rows, int columns) {
    //...
  }

  public int getRows();    //кол-во строк (y)
  public int getColumns(); // кол-во столбцов (x)

  public void put(Coordinate coordinate, Entity entity);
  public Entity get(Coordinate coordinate);
  public Entity remove(Coordinate coordinate);

  public Coordinate getCoordinate(Entity entity);
  public List<Entity> getAll();

  public boolean isEmpty(Coordinate coordinate);
  public boolean isValid(Coordinate coordinate); //проверка на то что координата находится в пределах доски
}
```

**Еще раз про паттерн Callback**

Если модель хочет что-то сообщить миру, например о том, что существо переместилось из точки А в точку Б, то модель не должна эту информацию печатать в консоль самостоятельно.
Иначе модель должна будет знать много лишего- в какую среду печатается информация(консоль, windows), на каком языке и т.д.

Поэтому модель может только просигнализировать о том, что с ней произошло действие, но не распечатывать информацию. 
Это называется "активная модель". Для этого используется паттерн "Callback".

Допустим, мы пишем военную стратегию, в которой мы строим разные здания, атакуем здания врага и т.д. В игре происходит много разных действий и нужно сообщать о них
```
Нам объявила войну Португалия
Родился принц
Разрушено: башня
Разрушено: барак
```

Делаем класс, который будет печатать сообщения
```
public class GameLogger {
  public void show(String message) {
    System.out.println(message);
  }
}
```

Делаем класс Здание. Здание может быть разрушено противником
```
public class Building {
  private final String name;
  private int hp;

  public Building(String name, int hp) {
    this.name = name;
    this.hp = hp;
  }

  //разрушение
  public void decreaseHp() {
    if(isDestroyed()) {
      return;
    }
    hp--;
  }

  //здание разрушено
  public boolean isDestroyed() {
    return hp == 0;
  }

  public String getName() {
    return name;
  }

  public int getHp() {
    return hp;
  }
}
```

Теперь нужно сообщить, когда здание будет разрушено.  
Напрямую это делать нельзя, поэтому вводим в проект интерфейс обратной связи Callback
```
public interface Callback {
  void execute(Building building);
}
```

И подключаем его к Зданию
```
public class Building {
  private final String name;
  private int hp;
  private Callback onDestroyCallback;

  public Building(String name, int hp) {
    this.name = name;
    this.hp = hp;
  }

  //...
  public void setOnDestroyCallback(Callback onDestroyCallback) {
    this.onDestroyCallback = onDestroyCallback;
  }

}
```

В момент разрушения здания будем дергать интерфейс
```
//разрушение
public void decreaseHp() {
  if(isDestroyed()) {
    return;
  }
    
  hp--;
  if(isDestroyed() && onDestroyCallback != null) {
    onDestroyCallback.execute(this);
  }
}
```

В игре создадим здания и настроим автоматическую распечатку информации при разрушении здания врагами
```
public class Main {
  public static void main(String[] args) {
    GameLogger gameLogger = new GameLogger();

    List<Building> buildings = List.of(
        new Building("барак", 10),
        new Building("башня", 15),
        new Building("стрельбище", 7)
    );

    for (Building building : buildings) {
      building.setOnDestroyCallback(b -> gameLogger.show("Разрушено: " + b.getName()));
    }

    boolean repeat = true;
    while (repeat) {
      repeat = false;
      for (Building building : buildings) {
        if (!building.isDestroyed()) {
          repeat = true;
        }
        building.decreaseHp();  //один раз в момент разрушения сделает обратный звонок
      }
    }

  }
}
```

Результат:
```
Разрушено: стрельбище
Разрушено: барак
Разрушено: башня
```

## ВЫВОД

Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про SOLID и "Чистый код".

n.58(133)  
#ревью #симуляция #паттерн #callback