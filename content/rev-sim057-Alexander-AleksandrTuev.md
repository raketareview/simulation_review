https://github.com/AleksandrTuev/Simulation  
[Alexander]

Нет потоков. Есть продвинутый функционал.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

После запуска симуляции, программа просто распечатывает в консоль несколько десятков кадров карты. 
Кадры нужно выводить не все за один раз, а печатать через равные промежутки времени, чтобы был эффект динамической анимации, а не статичного комикса.

## ХОРОШО

👍  Обширное меню  
🚀  Есть редактор карты  
👍  Спрайты существ не хранятся в самих существах  
👍  Координаты в существах хранятся начиная с Creature  
👍  Есть механизм умирания от голода  

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Из этого названия нельзя понять, что делает класс. Класс с таким названием может делать что угодно
```
class Manager
```

- Метод проверяет, содержит ли карта живые существа, а не является ли карта живым существом. "is" - является, "has" - содержит
```
private boolean isLivingCreatures(Map map)

//ПРАВИЛЬНО:
private boolean hasLivingCreatures(Map map)
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Coordinates> path = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Coordinates> path = new HashMap<>();
```

**3. Избыточно**
```
while (!(queue.isEmpty()))
if (!(visitedCells.contains(newCoordinates)))
if (!(AvailableCellsChecker.hasAvailableCells(map)))
if (!(map.getEntities().containsKey(coordinates)))

//ПРАВИЛЬНО:
while (!queue.isEmpty())
if (!visitedCells.contains(newCoordinates))
if (!AvailableCellsChecker.hasAvailableCells(map))
if (!map.getEntities().containsKey(coordinates))
```

**4. class Coordinates**

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

- Складывать координаты нужно с экземпляром такого же класса
```
public Coordinates add(CoordinatesShift coordinatesShift) {...}

//ПРАВИЛЬНО:
public Coordinates add(Coordinates coordinates) {...}
```

- Поля row и column должны быть final.

**5. class CoordinatesShift**

- Нет необходимости в существовании этого класса, он не дает ничего больше, чем может дать класс Coordinates. Для обозначения координат сдвига достаточно использовать обычные координаты, например так
```
Coordinates shifDownRightCoordinates = new Coordinates(-1, 1);
```

**6. class Map**

- Следовать ТЗ это хорошо. Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо.  
Второе приводит к постоянной путанице в коде. `Мар` это название стандартного интерфейса, поэтому меньшим злом является дать другое название данному прикладному классу: Board, GameMap или что-то подобное.

+ 👍 Тут ты молодец, что возвращаешь не оригинал entities, а его копию. В этом случае клиентский код не может воздействовать на внутреннее устройство class Map
```
public HashMap<Coordinates, Entity> getEntities() {
  return new HashMap<>(entities);
}
```

- Нарушение SRP, для обеспечения единой ответственности класса по хранению существ, карте не нужен метод который будет проверять, возможен ли сдвиг координаты.
То есть, проверка того, будет ли координата после сдвига находиться в рамках размеров карты. Этот метод используется в интересах поиска пути, вот там пусть и находится
```
public boolean canShift(Coordinates coordinates, CoordinatesShift coordinatesShift)
```

- Никогда не возвращай null
```
private final HashMap<Coordinates, Entity> entities = new HashMap<>();

public Entity getEntity(Coordinates coordinates) {
  validateCoordinates(coordinates);
  return entities.get(coordinates); //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

+ 👍 В целом в классе кроме `canShift()` нет ничего лишнего.


**7. interface PathSearchAlgorithm**, интерфейс поиска

+ 👍 Специальный интерфейс поиска это хорошо, можно делать разные его реализации. Например, BFS и AStar.

- Возвращение пути в виде LinkedHashSet выглядит очень экзотично. Не знаю, какие преимущества это дает по сравнению с простым List
```
LinkedHashSet<Coordinates> getPath(Coordinates from, Class<? extends Entity> food, Map map);
```

**8. class BreadthFirstSearch implements PathSearchAlgorithm** 

- Не формируй тут каждый раз объекты, это очень избыточно. Сделай константы для сдвиговых координат
```
for (int i = -1; i <= 1; i++) {
  for (int j = -1; j <= 1; j++) {
    if ((i == 0) && (j == 0)) {
      continue;
    }
    CoordinatesShift coordinatesShift = new CoordinatesShift(i, j);
    Coordinates newCoordinates = coordinates.add(coordinatesShift);
    //...
  }
}

//ПРАВИЛЬНО:
private static final Coordinates[] SHIFT_COORDINATES = {  
  new Coordinates(-1, 0), //вниз
  new Coordinates(-1, -1), //вниз-влево
  new Coordinates(0, -1),  //влево
  new Coordinates(1, -1),  //вверх-влево
  //oths's
};

//...
for(Coordinates shift: SHIFT_COORDINATES) {  
  Coordinates newCoordinates = coordinates.add(shift);
  //...
}
```

**9. class Manager**

- Название класса не говорит ни о чем. На самом деле это фабрика поиска
```
public class Manager {
  public static PathSearchAlgorithm getDefaultPathSearchAlgorithm() {
    return new BreadthFirstSearch();
  }
}

//ПРАВИЛЬНО:
public class PathSearchAlgorithmFactory {
  public static PathSearchAlgorithm get() {
    return new BreadthFirstSearch();
  }
}
```

**10. Пакет entities**

- Кроме Entity и его наследников, пакет внезапно содержит класс `CoordinatesShift`, который не является наследником Entity. 

**11. abstract class Entity**

- Неправильная организация иерархии классов. Предок не должен хранить методы, которые потомки могут наделять действиями, а могут не наделять. 
Например, метод `act()` в классе Creature выполняет какие-то действия, а в классе Rock этот же метод имеет пустое тело
```
public abstract void act(Map map);
```

- Неправильная организация иерархии классов, нарушение OCP 
```
protected abstract boolean canMove();
```
По этой логике, если у Entity появится потомок, который умеет летать, то в Entity придется добавить абстрактный метод `boolean canFly()` и реализовывать его у абсолютно всех потомков. 
То есть, вносить каскадные изменения.

Методы `act()` и `canMove()` нужно убрать из этого класса.  
О правильной иерархии классов напишу в конце.

**12. class Rock extends Entity**

- Неправильная организация иерархии классов. Наследник не должен делать заглушку на методе предка
```
@Override
public void act(Map map) {
}
```
Если приходится делать заглушку на методе предка, значит в предке не должно быть этого метода. Все потомки должны делать что-то полезное в переопределенных методах предка.

- Из-за неправильной иерархии классов, камень вынужден отвечать на вопрос, умеет ли он ходить. Конечно, не умеет
```
@Override
protected boolean canMove() {
  return false;
}
```

**13. abstract class Creature extends Entity**

- Нарушение DRY, перегружай конструкторы правильно
```
public Creature(Coordinates coordinates, int health, int speed, Class<? extends Entity> food) {
  this.coordinates = coordinates;
  this.health = health;
  this.speed = speed;
  this.food = food;
}

public Creature(int health, int speed, Class<? extends Entity> food) {
  this.health = health;
  this.speed = speed;
  this.food = food;
}

//ПРАВИЛЬНО:
public Creature(Coordinates coordinates, int health, int speed, Class<? extends Entity> food) {
  this.coordinates = coordinates;
  this.health = health;
  this.speed = speed;
  this.food = food;
}

public Creature(int health, int speed, Class<? extends Entity> food) {
  this(null, health, speed, food);
}
```

- Если в проекте есть интерфейс поиска, значит по умолчанию предполагается, что могут быть разные его реализации. Значит должен быть простой способ внедрять в креатур разные реализации поиска. 
Сейчас креатуры жестко завязаны на конкретную реализацию поиска, поменять которую можно только путем переписывания кода в соответствующей фабрике. Креатуры должны принимать поиск в конструктор
```
public abstract class Creature extends Entity {
  //...
  public Creature(Coordinates coordinates, int health, int speed, Class<? extends Entity> food) {...}
  
  public void makeMove(Map map) {
    LinkedHashSet<Coordinates> path = Manager.getDefaultPathSearchAlgorithm().getPath(coordinates, food, map);
    //...
  }
}    

//ПРАВИЛЬНО:
public abstract class Creature extends Entity {
  //...
  public Creature(Coordinates coordinates, int health, int speed, Class<? extends Entity> food, PathSearchAlgorithm pathSearchAlgorithm) {...}
  
  public void makeMove(Map map) {
    LinkedHashSet<Coordinates> path = pathSearchAlgorithm().getPath(coordinates, food, map);
    //...
  }
}  
```

**14. class Predator/Herbivore extends Creature**

+ 👍 Хорошо, что применяются константы, а не инжектятся магические числа в конструктор предка 
```
private static final int INIT_HEALTH = 10;
private static final int INIT_SPEED = 2;
private static final Class<? extends Entity> INIT_FOOD = Grass.class;

public Herbivore() {
  super(INIT_HEALTH, INIT_SPEED, INIT_FOOD);
}
```

- Следствие неправильного нейминга класса карты. При чтении сигнатуры  кажется, что метод принимает стандартный интерфейс Map 
```
protected void makeAttack(Coordinates target, Map map)
```

А на самом деле он принимает экземпляр пользовательского класса Map, то есть это Карта симуляции: `import com.example.simulation.Map`

**15. class ClearMapAction implements Action**

- Нарушение SRP. Класс не только очищает карту, но и печатает ее
```
public class ClearMapAction implements Action {
  @Override
  public void execute(Map map) {
    map.removeEntities();
    MapConsoleRenderer.render(map);
  }
}
```

- Все остальные Action'ы тоже печатают карту и таким образом тоже нарушают SRP. 

**16. class CreatureInteractionAction implements Action**

- Нарушение SRP. Как я понимаю, идея класса состоит в том, что он должен каждую наличиствующую в карте креатуру дернуть за метод `act()`, то есть инициировать некое действие
```
for (Entity entity : map.getEntities().values()) {
  entity.act(map);
}
``` 

Так пусть дергает. Рассуждать о том, заполнена карта или нет и сигнализировать об этом- не его забота. Этим должен заниматься кто-то другой
```
if (isEmptyMap(map)) {
  System.out.println("Карта не заполнена сущностями");
  return;
}
```

**17. class SpawnAction implements Action**

- Нарушение DRY, какая-то сложная логика заселения существами. 
Дублирование хранения координат одновременно в существе и карте добавляет сложности, но все равно можно немного упростить с помощью применения стандартного интерфейса java
```
  private void setupEntitiesPositions() {
    createEntity(new Rock(), AMOUNT_ROCK);
    createEntity(new Tree(), AMOUNT_TREE);
    createCreature(new Predator(), AMOUNT_PREDATOR);
    //...
  }

  private void createEntity(Entity entity, int count) {
    Coordinates coordinates;

    for (int i = 0; i < count; i++) {

      if (entity instanceof Rock) {
        coordinates = RandomAvailableCoordinateGenerator.generate(map);
        Rock rock = new Rock();
        map.setEntity(coordinates, rock);
      } else if (entity instanceof Predator) {
        coordinates = RandomAvailableCoordinateGenerator.generate(map);
        Predator predator = new Predator(coordinates);
        map.setEntity(coordinates, predator);
      }
      //еще миллион строк
    }
  }

//ЛУЧШЕ:
  private void setupEntitiesPositions() {
    createEntity(() -> new Rock(), AMOUNT_ROCK);
    createEntity(() -> new Tree(), AMOUNT_TREE);
    createEntity(() -> new Predator(), AMOUNT_PREDATOR);
    //...
  }

  private void createEntity(Supplier<Entity> supplier, int count) {
    for (int i = 0; i < count; i++) {
      Coordinates coordinates = RandomAvailableCoordinateGenerator.generate(map);
      Entity entity = supplier.get();
      
      if(entity instanceof Creature creature) {
        creature.setCoordinates(coordinates);
      }

      map.setEntity(coordinates, entity);
    }
  }
```

**18. Дублирование кода** в классах RespawnHerbivoreAction, RespawnPredatorAction и RespawnGrassAction. По аналогии с предыдущим примером избавься от дублирования.

**19. class Menu**

+ 👍 Меню в ООП стиле.

**20. class MapConsoleRenderer**

+ 👍 Спрайты находятся в константах.

- Если пришло неизвестное существо для распечатки, то нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт.

**21. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## АРХИТЕКТУРА

- Нужно ввести потоки и распечатывать кадры карты через равные промежутки времени, чтобы был эффект движущихся картинок
- Нужно переделать иерархию классов Entity

**ИЕРАРХИЯ КЛАССОВ НА ПРИМЕРЕ**

Допустим, есть транспортная компания, которая доставляет грузы двумя видами транспорта: кораблями по морю и самолетами по небу.
Самолеты умеют летать, но не умеют плавать, а корабли наоборот.

Нужно создать абстрактный класс Транспорт и два его наследника: Корабль и Самолет.

**I. Неправильная иерархия классов**

Транспорт содержит в себе оба метода: плыть и лететь. И два метода, которые сообщают об этих умениях
```
//НЕПРАВИЛЬНО:
public abstract class Транспорт {
  private final double cargoWeight;

  protected Транспорт(double cargoWeight) {
    this.cargoWeight = cargoWeight;
  }

  public double getCargoWeight() {
    return cargoWeight;
  }

  public abstract void ship();
  public abstract void fly();

  public abstract boolean canShip();
  public abstract boolean canFly();
}
```

Потомки реализуют свои умения. А на те методы, которые делать не умеют, ставят пустые заглушки
```
public class Корабль extends Транспорт{
  protected Корабль(double cargoWeight) {
    super(cargoWeight);
  }

  @Override
  public void ship() {
    //РЕАЛИЗАЦИЯ ПЛАВАНИЯ
  }

  @Override
  public void fly() { } //пустая заглушка

  @Override
  public boolean canShip() {
    return true;
  }

  @Override
  public boolean canFly() {
    return false;
  }
}
```
```
public class Самолет extends Транспорт{
  protected Самолет(double cargoWeight) {
    super(cargoWeight);
  }

  @Override
  public void ship() { } //пустая заглушка

  @Override
  public void fly() {
    //РЕАЛИЗАЦИЯ ПОЛЕТА
  }

  @Override
  public boolean canShip() {
    return false;
  }

  @Override
  public boolean canFly() {
    return true;
  }
}
```

В принципе это будет работать, но есть проблемы
- Транспорт становится божественным классом, содержит в себе все методы всех потомков
- Нарушение OCP, добавление потомка с новым поведением вызовет каскадные изменения во всех классах иерархии, начиная с Транспорта
- Нарушение SRP, классы хранят в себе много ненужных методов, которые не имеют к их деятельности никакого отношения

**II. Правильная иерархия классов**

Транспорт в данном случае знает только свою грузоподъемность. Способы перемещения он не знает, это проблема его наследников
```
public abstract class Transport {
  protected final double cargoWeight;

  protected Transport(double cargoWeight) {
    this.cargoWeight = cargoWeight;
  }

  public double getCargoWeight() {
    return cargoWeight;
  }
}
```

Корабль это наследник Транспорта и он умеет плавать- вводим в него метод "плыть"
```
public class Boat extends Transport {
  private final String name;

  protected Boat(double cargoWeight, String name) {
    super(cargoWeight);
    this.name = name;
  }

  public void ship() {
    //РЕАЛИЗАЦИЯ ПЛАВАНИЯ
  }
}
```

Самолет тоже Транспорт, но плавать он не умеет, а умеет летать. Вводим в него метод "лететь"
```
public class Airplane extends Transport {
  protected Airplane(double cargoWeight) {
    super(cargoWeight);
  }

  public void fly() {
    //РЕАЛИЗАЦИЯ ПОЛЕТА
  }
}
```

В случае, если появится третий вид транспорта, мы его легко добавим в нашу иерархию. При этом не придется вносить каскадных изменений ни в базовый класс Транспорт, ни в его другие наследники.
В прошлом, неправильном примере, нам в этом случае пришлось бы вносить каскадные изменения
```
public class Train extends Transport {
  protected Train(double cargoWeight) {
    super(cargoWeight);
  }

  public void ride() {
    //РЕАЛИЗАЦИЯ ЕЗДЫ ПО РЕЛЬСАМ
  }
}
```

Добавляем в классы соответствующие toString'и
```
  @Override
  public String toString() {
    return "Boat{" +
        "name='" + name + '\'' +
        ", cargoWeight=" + cargoWeight +
        '}';
  }
```

Имея коллекцию разного транспорта и получив заказ отправить груз по морю, мы легко сможем выбрать корабли
```
public class Main {
  public static void main(String[] args) {
    List<Transport> transports = List.of(
        new Train(100),
        new Train(100),
        new Boat(2500, "Механик Гаврилов"),
        new Airplane(5),
        new Boat(1500, "Крыжополь")
    );

    List<Boat> boats = new ArrayList<>();
    for(Transport t : transports) {
      if(t instanceof Boat boat) {
        boats.add(boat);
      }
    }
    System.out.println("Корабли:");
    boats.forEach(System.out::println);
  }
}

//РЕЗУЛЬТАТ:
Корабли:
Boat{name='Механик Гаврилов', cargoWeight=2500.0}
Boat{name='Крыжополь', cargoWeight=1500.0}
```

Можно возразить, что все три метода(плыть, лететь, ехать) можно заменить одним методом `move()`, который будет находиться в родительском классе и переопределяться в каждом из потомков, реализуя в них свою специфическую механику перемещения.
Но это зависит от контектста задачи. Например, эти три вида перемещения могут быть принципиальными при организации логистики.

## ВЫВОД

Изучить тему наследования в ООП. Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы и Немчинского про SOLID.

n.57(131)  
#ревью #симуляция #меню #иерархияклассов