https://github.com/nikita-webdev/simulation  
[Никита]

Удручающе.

## ХОРОШО

+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализовано пауза/пуск во время работы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```
package simulation.map;

//ПРАВИЛЬНО:
package simulation.simulation_map;
```

- Название должно как можно лучше объяснять суть явления. Это размеры карты- ее ширина и высота
```
public static final int MAP_SIZE_ROW = 20;
public static final int MAP_SIZE_COLUMN = 15;

//ЛУЧШЕ:
public static final int HEIGHT = 20;
public static final int WIDTH = 15;
```

- Названия пакетов нужно писать стилем lower_snake
```
package simulation.actions.initActions;

//ПРАВИЛЬНО:
package simulation.actions.init_actions;
```

- Этот метод не создает `Map`, он заполняет массив данными
```
private void createMap() {
  for (int i = 0; i < field.length; i++) {
    Arrays.fill(field[i], EMPTY_ICON);
  }
}
```
Если под "Map" подразумевается "Карта симуляции", то используй в названии вместо "Map" какие-то синонимы, 
чтобы не путать с названием стандартного интерфейса "Map".  
Например "Field"- раз уж ты заполняешь данными одноименный массив.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Default в свич-кейсе должен быть внятным**

Default в свич-кейсе всегда должен делать что-то полезное.  
Если не знаешь, что написать в дефолте, пиши выброс исключения- не ошибешься
```
switch (record.getLevel().toString()) {
  case "INFO", "SEVERE", "WARNING":
    break;
  default:
}

//ПРАВИЛЬНО:
default: throw new <Исключение>;
```

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы.

Если к одной и той же магической штуке обращаются два разных класса, то делай ее константой в одном из этих двух классов. 
Либо перенеси ее в третий класс- эти данные должны быть синхронизированы между собой
```
public class MenuOptionsPrinter {
  public void printStartOptions() {
    System.out.println("1 - Start");
    System.out.println("2 - Pause");
    //...
  }
}

public class Simulation {
  private static final String START_RESUME = "1";
  private static final String PAUSE = "2";
  //...
}
```
*Фаулер, "Рефакторинг", гл.8, "Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*

**4. Утилитные и константные классы** должны быть `final` и иметь приватный конструктор- не должно быть возможности унаследоваться от утилиты или сделать ее экземпляр
```
public class Icons {
  public final static String HERBIVORE_ICON = "\uD83D\uDC11";
  //...
}

//ПРАВИЛЬНО:
public final class Icons {
  public final static String HERBIVORE_ICON = "\uD83D\uDC11";
  //...

  private Icons() {}
}
```

**5. record Coordinate(int row, int column)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

**6. class SimulationMap**

- Нарушение SRP, божественный класс, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: подсчитать количество травы, нарисовать карту(еще и через паузу) и многое, многое, МНОГОЕ другое.

Проекту очень нужно через заданные промежутки времени печатать в консоль карту.  
Но делать это явно должна не сама карта, а кто-то другой.
Этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно саму себя печатать.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Карта содержит в себе фиксированные размеры, поэтому становится не универсальной
```
public static final int MAP_SIZE_ROW = 20;
public static final int MAP_SIZE_COLUMN = 15;
```
Фиксированные размеры могут быть у шахматной доски(8x8) или игры крестики-нолики(3x3).  
Здесь размеры должны приниматься в конструктор, чтобы можно было создавать карту произвольных размеров.

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна саму себя печатать в консоль
```
public void updateMap() {
  renderer.renderMap();
  //...
}
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

+ 👍 Хорошо, что возвращаешь копию мапы существ, а не ее оригинал- тем самым соблюдается инкапсуляция и защищается внутреннее строение класса `SimulationMap`
```
public Map<Coordinate, Entity> getEntities() {
  return Collections.unmodifiableMap(entities);
}
```

- Карта не должна заниматься подсчетом существ и тем более, содержать персональные методы для подсчета каждого конкретного вида существ
```
  public int getCountOfEntity() {...}
  public int getCountOfHerbivores() {...}
```
Самой карте не нужно уметь это делать для обеспечения своей единой ответственности по хранению существ. 
Считать траву нужно в интересах класса `RespawnGrassAction`- вот пусть он внутри себя это и делает.

- Ну какое дело может быть Карте до того, какую еду едят животные?
```
public boolean isFood(Creature creature, Coordinate coordinate) {...}
```

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean isCoordinatesOccupied(Coordinate targetCoordinates) {
  return getEntities().containsKey(targetCoordinates);
}
```
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что свободна. 
А правильный ответ- такой координаты в карте вообще нет.

❌ Класс собрал в себе все худшие практики для класса Карта.

**7. class PathFinder**

- Что такое пара х/у? Правильно, координата
```
private static final int[][] OFFSETS = {
  {0, -1},
  //...
}

//ПРАВИЛЬНО:
private static final List<Coordinate> OFFSETS = List.of(
  new Coordinate(0, -1), 
  //....
);
```

- Нарушение инкапсуляции. Публичными должны быть только те методы, которые предназначены для использования сторонними классами. 
Этот метод не из их числа
```
public List<Coordinate> reconstructPath(Map<Coordinate, Coordinate> cameFrom, Coordinate food)
```

- Нарушение SRP. 

Классу поиска пути должно быть без разницы, ищет он путь для Креатуры или еще кого-то
```
public List<Coordinate> searchPath(SimulationMap simulationMap, Creature creature, Coordinate from)
```
Он должен искать путь не для *кого-то конкретного*. 

Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям. 
В данном случае- до точки, в которой находится существо нужного класса.
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно.
Примерноо, так:
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте world от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
}
```

- Мапа координата-координата обычно при реализации поиска используется как эрзац-заменитель связного списка
```
Map<Coordinate, Coordinate> cameFrom = new HashMap<>();
```
Вместо мапы координата-координата сделай класс связного списка, алгоритм тогда будет выглядеть понятнее.
Например, так:
```
class Node {
  private Coordinate coordinate; 
  private Node parent;
  //...
  public Coordinate getCoordinate() {...}
  public Node getParent() {...}
}
```

- Что это за ацкий ад? Это примерно АБСОЛЮТНО нечитаемый код
```
for (Coordinate i = new Coordinate(food.row(), food.column()); i != null; i = cameFrom.get(i))  {...}
```
Не делай так больше, циклы должны быть простыми и понятными.  
Если в условии цикла приходится вставлять такой закрученный код, значит окружающий этот цикл алгоритм нужно переделать так, 
чтобы цикл снова стал простым и понятным.

**8. class Entity**

- Нарушение инкапсуляции- публичный доступ к полям класса
```
public String name;
public String icon;
```
Доступ к полям класса извне должен осуществляться только через методы. 
Например, геттеры и сеттеры.

- Почему `name` устанавливается через конструктор, а `icon`- нет?
```
public Entity(String name) {
  this.name = name;
}
```

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```
public String icon;
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.
Потому что в разных средах(консоль, Swing, Android) одна и та же модель может быть показана разными способами.
Спрайты всех существ должны храниться в классе, который распечатывает карту.

**9. class Tree/Grass/Rock extends Entity**

- Не делай статических импортов, тем самым ты вводишь в заблужденние. 
Становится неочевидным, что какое-то поле не принадлежит данному классу
```
import static simulation.config.Icons.TREE_ICON;

public class Tree extends Entity {
  public Tree(String name) {
    //...
    icon = TREE_ICON;
  }
}

//ПРАВИЛЬНО:
icon = Icons.TREE_ICON;
```

- Нарушение ООП дизайна в наследовании. Все поля должны инжектиться в предка, а не частично сетиться 
```
public class Tree extends Entity {
  public Tree(String name) {
    super(name);

    icon = TREE_ICON;
  }
}

//ПРАВИЛЬНО:
public Tree(String name) {
  super(name, TREE_ICON);
}
```

- Нарушение Low Coupling. 

Абсолютно ненужная зависимость на сторонний класс `Icons`. 
Если хочешь здесь использовать константу со спрайтом, то объяви ее прямо в этом классе.

**10. abstract class Creature extends Entity**

Снова неправильное использование наследования
```
private int speed;
private int hp;

public Creature(String name) {
  super(name);
}

//ПРАВИЛЬНО:
public Creature(String name, int speed, int hp) {
  super(name);
  this.speed = speed;
  this.hp = hp;
}
```

- Организация движения.

Основная логика передвижения Креатуры должна быть или в самой Креатуре, или в стороннем классе, например, Мувере.
Сейчас эта логика разделена на несколько классов. 

Если основная логика движения должна находиться в самой Креатуре, то метод `makeMove(...)` внутри себя должен сам искать путь движения, а не получать его извне
```
public void makeMove(SimulationMap simulationMap, Coordinate from, List<Coordinate> path) {...}

//ПРАВИЛЬНО:
private final Class<? extend Entity> food;
private final ПоискПути поискПути = ...;

public Creature(..., Class<? extend Entity> food) {
  super(...);
  this.food = food;
}

public void makeMove(SimulationMap simulationMap) {
  Coordinate моиКоординаты = simulationMap.getCoordinate(this);
  List<Coordinate> patch = поискПути(simulationMap, моиКоординаты, food);
  //...
}
```

**11. class Herbivore/Predator extends Creature**

Нарушение инкапсуляции, `eat(...)` должен быть protected.

**12. interface Action**

👍 Норм
```
public interface Action {
  void execute(SimulationMap simulationMap);
}
```

**13. Пакет initActions**

Пакет должен содержать только классы-наследники интерфейса `Action`, либо классы, которые существуют в интересах этих наследников.  
Этот класс к ним не относится:
```
public class InitObjects {...}
```

- Нарушение DRY.

Множество классов, которые делают одно то же- заселяют карту определенным видом существ.
Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс, который будет принимать в конструктор количество создаваемых существ и способ их создания.  
Способ создания можно передавать через стандартный интерфейс Supplier
```
SpawnAction заяцSpawnAction = new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ);
SpawnAction волкSpawnAction = new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ);
заяцSpawnAction.execute(карта); 
волкSpawnAction.execute(карта); 
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
      Coordinates coordinates = getRandomEmptyCoordinates(карта);
      карта.put(entity, coordinates);
    }
  }

  private Coordinates getRandomEmptyCoordinates(Карта карта) {
    //находит и возвращает случайную пустую координату
  }
}
```

**14. Пакет turnActions**

- Этот класс не наследник `Action` и не должен быть в этом пакете
```
class MoveAllCreatures {...}
```

- Нарушение DRY. Два класса, которые делают одно и то же: возобновляют на карте количество существ определенного вида.  

**15. Разъяснение идеи Action'ов**

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники.  
Это вариация паттерна Command- экшены должны быть родственны и одинаково использоваться через полиморфизм.  
Примерно так
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

Суть:  
+ Все классы-экшены должны реализовывать интерфейс Action
+ Все классы-экшены должны иметь только один публичный метод- тот, что заявлен в интерфейсе Action

**16. class Renderer**

Избыточно сложная система распечатки карты. 
Каждый раз перед распечаткой карты создается массив, и только потом он распечатывается
```
private final SimulationMap simulationMap;
private final String[][] field;

private void createMap() {
  for (int i = 0; i < field.length; i++) {
    Arrays.fill(field[i], EMPTY_ICON);
  }
}

private void printMap() {
  //тут распечатывается массив field
}  
```

Это лишняя работа, печатай карту сразу, без промежуточных массивов.  
Примерно так:
```
public void render(Board board) {
   for(int row = 0; row < board.getHeight(); row++) {
     for(int column = 0; column < board.getWidth(); column++) {
       Coordinate coordinate = new Coordinate(row, column); 
       if(board.isПусто(coordinate)) {
         //напечатать пустую землю
       } else {
         Entity entity = board.get(coordinate);
         //напечатать спрайт entity
       }
     }
   } 
   System.out.println();
}
```

**17. class Simulation**

Action'ы объявляются индивидуально
```
private final RespawnGrassAction respawnGrassAction = new RespawnGrassAction();
private final RespawnHerbivoreAction respawnHerbivoreAction = new RespawnHerbivoreAction();
```

Это противоречит самой идее экшенов.  
Если их объявлять индивидуально, то и использовать их приходится индивидуально:
```
respawnGrassAction.execute(simulationMap);
respawnHerbivoreAction.execute(simulationMap);
```

На самом деле экшены нужно использовать не индивидуально, а через полиморфизм.
Примерно так:
```
private final List<Action> initActions = List.of(new UnoAction());
private final List<Action> turnActions = List.of(new DosAction(), new TresAction(), new MoveAction());

public Simulation(...) {
  //...
  executeActions(initActions);  //выполнение экшенов на старте
}

private void nextTurn() {
  //...
  executeActions(turnActions);  
}

private void executeActions(List<Action> actions) {
  for(Action a : actions) {
    a.execute(board);
  }
}
```

**18. class Main**, содержит точку входа main

👍 Только создает и запускает `Simulation`, это хорошо.

## ВЫВОД

Не видно понимания ООП даже на самом базовом уровне: зачем нужны геттеры, сеттеры, конструкторы. 
Что такое инкапсуляция, полиморфизм, наследование и как их использовать в программе.

Почитай статьи, посмотри видосики на ютубе. 
Возможно, сделай пару простых упражнений на наследование и полиморфизм.

Подробнее разберись с Action'ами, посмотри ролики на ютубе про паттерн "Command", чтобы получить о нем общее представление.

Не видно и понимания декомпозиции- как делить программу на классы.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. 
Посмотреть ролики Немчинского про SOLID.

n.101(229)  
#ревью #симуляция 