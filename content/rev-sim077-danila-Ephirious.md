https://github.com/Ephirious/newSimulation  
[Данила]

После первого рефакторинга. Стало значительно лучше.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Если нет еды, существа делают случайные ходы
+ 👍 Механика фотосинтеза у травы

## ЗАМЕЧАНИЯ

**1. Нейминг**

Особых замечаний нет.  

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. record Coordinates(int row, int column)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

**3. class SimulationMap**

- Даже если создаешь карту через фабрику, стоит сделать конструктор карты публичным. Не вижу оснований, чтобы ограничивать возможность создания экземпляра карты другими классами
```
protected SimulationMap(int width, int height)
```

- При всех операциях с участием координаты(проверить наличие и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public boolean hasEntity(Coordinates coordinates) {
  //тут нужно провалидировать координату
  return entities.containsKey(coordinates);
}
```
Например, если карта имеет размер 10х10 и мы спросим есть ли существо на координате 200,200 то карта сейчас ответит, что его там нет. А на самом деле правильный ответ: "такой координаты не существает".  

Это как с вопросом *"Ты бросил употреблять наркотики?"*.  
Корректно ответить на него только с помощью "да/нет"(true/false) возможно лишь в том случае, если респонтент их действительно употреблял.  
Если не употреблял, то правильный по букве ответ "нет"(не делал этого- а значит и не бросал делать это) будет неверным по его, вопроса, сути.

- Придерживайся единообразия в нейминге. Оба этих метода работают одинаково- проверяют данные и бросают исключение в случае из некорректности, оба должны начинаться с `validate`
```
void checkAbsence(Coordinates coordinates)
void validateCoordinates(Coordinates coordinates) 
```

+ 👍 Метод норм. Если не хочешь, чтобы idea в данном методе не подчеркивала предупреждение при касте, можно поставить соответствующую аннотацию
```
@SuppressWarnings("unchecked") <-- отключает предупреждение для каста
public <T extends Entity> List<T> getEntitiesByType(Class<T> clazz) {
  //...
  result.add(((T) currentEntity));  <-- каст
  //...
}
```

+ 👍 В целом, класс сделан хорошо: соответствует SRP. 

**4. Классы фабрик SimulationMap**

Есть семейство классов-фабрик для создания карты 
```
abstract class AbstractMapFactory
class SmallMapFactory extends AbstractMapFactory
class MediumMapFactory extends AbstractMapFactory 
class LargeMapFactory extends AbstractMapFactory
```

Фабрики это хорошо. Каждая фабрика здесь предельно простая
```
public class SmallMapFactory extends AbstractMapFactory {
  private static final int WIDTH = 10;
  private static final int HEIGHT = 10;

  @Override
  public SimulationMap createMap() {
    return new SimulationMap(WIDTH, HEIGHT);
  }
}
```
Можно оставить, как есть, можно вместо группы классов сделать простую параметризованную фабрику. Тогда вместо четырех классов останется один, примерно такой
```
public final class SimulationMapFactory {
  private SimulationMapFactory() {}

  public static SimulationMap get(Size size) {
    return new SimulationMap(size.width, size.height);
  }

  public enum Size {
    SMALL(10, 10),
    MEDIUM(20, 20),
    LARGE(30, 30);

    public final int width;
    public final int height;

    Size(int width, int height) {
      this.width = width;
      this.height = height;
    }
  }
}
```
Но это уже на твой вкус, правильно и так и так.

**5. class CoordinatesUtils**

Забыл поставить `final`- утилитные классы должны быть финальными.

**6. class SimulationMapUtils**

При подходе, явленном в реализации проекта, этот метод- часть ответстсвенности класса `Creature` и должен находиться в нем
```
void moveEntity(SimulationMap worldMap, Coordinates from, Coordinates to, boolean isTargetRemove)
```

**7. Поиск пути** 

+ 👍 Абстрактный класс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS и AStar, что здесь и сделано.

+ 👍 Сигнатура поиска ок
```
public abstract List<Coordinates> find(Coordinates source, Class<? extends Entity> target)
```

**8. abstract class Entity и его "статичные" потомки Tree, Rock**

Нет никаких оснований делать конструктор `protected`. Сделай его публичным.

**9. class Grass extends Entity**

Класс не должен инициализировать сам себя значениями. Сейчас клиент может создать экземпляр травы только со случайными параметрами. А с заранее заданными- не сможет
```
private int nutritionValue;
private final int decreaseValue;

protected Grass() {
  this.nutritionValue = randomizer.nextInt(NUTRITION_LOWER_BOUND, NUTRITION_UPPER_BOUND);
  this.decreaseValue = randomizer.nextInt(DECREASE_LOWER_BOUND, DECREASE_UPPER_BOUND);
}

//ПРАВИЛЬНО:
protected Grass(int nutritionValue, int decreaseValue) {...}
```
*Мартин, "ЧК", гл.11.2* 

Тем более, траву ты, среди прочего, создаешь через фабрику. Пусть тогда фабрика и высчитывает рандомные значения травы
```
public class UnmovedEntitiesFactory implements EntityFactory {
  //...
  public Entity create() {
    //...
    case GRASS -> returningEntity = new Grass();
  }
}

//ПРАВИЛЬНО:
case GRASS -> returningEntity = new Grass(getRandomGrassNutrition(), getRandomGrassDecrease());
//getRandomGrassNutrition(), getRandomGrassDecrease() - методы в фабрике
```

**10. abstract class Creature extends Entity**

- Этот метод и его реализации в наследниках должны быть protected, потому что он вспомогательный- используется только внутри этого класса 
```
public abstract void eat(SimulationMap worldMap, Coordinates target);
```

- Какова концепция движения, мертвый негр может играть в баскетбол, а мертвое существо может ходить? 
Если нет, то в начале метода движения нужно бросать исключение, если пытаются двинуть мертвого
```
void move(SimulationMap worldMap)
```

**11. class Predator/Herbivore extends Creature**

+ 👍 Общая логика по максимуму вынесена в предка- `Creature`

- Нет никаких причин делать конструктор `protected`, оставь его публичным.

**12. Фабрики существ**

- Для создания существ используются фабрики. Причем, не одна фабрика на всех и не пять отдельных фабрик под каждый вид существ.  
А одна фабрика для Зайца, одна для Волка и одна общая фабрика для "статичных" существ: Трава, Камень, Дерево.  
Возможно, стоит сделать одну общую фабрику существ по аналогии с моим предложением сделать одну фабрику карты.

- В данном виде фабрика не делает ничего больше, чем обычный конструктор `Herbivore`.  
Фактически, это антипаттерн "Полтергейст"- класс получает в себя данные и тразитом передает в конструктор другого класса, на этом всё
```
public class HerbivoreFactory implements EntityFactory {
  private final int healthPoints;
  private final Speed speed;
  private final PathFinder finder;

  public HerbivoreFactory(int healthPoints, Speed speed, PathFinder finder) {...}

  @Override
  public Entity create() {
    return new Herbivore(healthPoints, speed, finder);
  }
}
```

- Избыточно
```
public Entity create() {
  Entity returningEntity = null;
  switch (type) {
    case ROCK -> returningEntity = new Rock();
    case TREE -> returningEntity = new Tree();
  }
  return returningEntity;
}

//ПРАВИЛЬНО:
public Entity create() {
  return switch (type) {
    case ROCK ->  new Rock();
    case TREE ->  new Tree();
    default -> ... //бросить исключение
  }
}
```

**13. interface Command**, то, что в ТЗ называется `Action`

👍 Action/Command в виде интерфейса, а не абстрактного класса- ок
```
public interface Command {
  void execute();
}
```

**14. class InitCommandsFactory extends AbstractCommandsFactory**

- Эта переменная означает не проценты(PERCENT), а коэффициент. Например, при карте 10х10, в ней будет создано 10 камней(я проверял): 10 * 10 * 0.1 = 10.
Поэтому или переименуй на коэффициент, или проценты сделай процентами
```
//КОЭФФИЦИЕНТ:
private static final double ROCK_PERCENT = 0.1f;

//ПРОЦЕНТЫ:
private static final int ROCK_PERCENT = 10;
```

- Избыточно много кода, дублирование. Я бы сократил до такого
```
  public List<Command> getCommands() {
    List<Command> initCommands = new LinkedList<>();

    int mapSize = worldMap.getWidth() * worldMap.getHeight();

    Map<EntityFactory, Double> map = Map.of(
        new UnmovedEntitiesFactory(UnmovedEntityType.ROCK), ROCK_PERCENT,
        new UnmovedEntitiesFactory(UnmovedEntityType.TREE), TREE_PERCENT,
        new UnmovedEntitiesFactory(UnmovedEntityType.GRASS), GRASS_PERCENT,
        new HerbivoreFactory(Creature.MAX_HP, Speed.getRandomSpeed(), finder), HERBIVORE_PERCENT,
        new PredatorFactory(Creature.MAX_HP, Speed.getRandomSpeed(), finder, Damage.getRandomDamage()), PREDATOR_PERCENT
    );

    for (Map.Entry<EntityFactory, Double> entry : map.entrySet()) {
     int amount = ((int) (mapSize * entry.getValue()));

      initCommands.add(new PlaceEntityCommand(
          worldMap,
          () -> SimulationMapUtils.getFreeCoordinates(worldMap),
          amount,
          entry.getKey()
      ));
    }
    return initCommands;
  }
```

**15. class MoveCreaturesCommand extends AbstractCommandMap**

- Лучше сделать в `Creature` соответствующий метод. У травы есть `boolean isAlive()`, чем `Creature` хуже? 
```
if (current.getHealthPoints() > Creature.MIN_HP) {
  current.move(worldMap);  //
} else {
  //действия если мертвый  
}

//ЛУЧШЕ:
if (current.isAlive()) {
  current.move(worldMap);  //
} else {
  //действия если мертвый  
}
```

- Дискуссионно: с точки зрения SRP, не имеет ли сейчас класс две ответственности: уборка мертвых существ с карты и движение живых креатур?

Возможно, стоит разделить класс на два: один должен убирать мертвые существа с карты(и креатуры и траву), второй- двигать живых.  
Трава может быть жива, креатуры могут быть живы- возможно, эти существа должны имплементировать какой-то общий интерфейс, который будет характеризовать их "живое" либо "неживое" состояние.

**16. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## АРХИТЕКТУРА

Так как с ООП а базовом уровне у тебя все ок, рассмотрим более тонкие моменты.

При написании программ нужно балансировать между двумя крайностями.  
Первая крайность это антипаттерн **hard кодирование** , когда в программе минимум абстракций и настраиваимых значений. Типичный пример этого антипаттерна- когда карта симуляции может быть только одного фиксированного размера
```
class GameMap {
  //...
  public GameMap() { <-- ЕДИНСТВЕННЫЙ КОНСТРУКТОР
    this.width = 15; <-- ПРИМЕР АНТИПАТТЕРНА "ЖЕСТКОЕ КОДИРОВАНИЕ"
    this.height = 20;
  }
  //...
}
```
Вторая крайность- **soft кодирование**, когда в программе максимум абстракций и настраиваемых значений. Это делает программу супергибкой но такой же супернепоянтной. 
Нужно найти разумный баланс между этими крайностями.

**Рассмотрим на примере**

Делаем класс Хищник. Хищник имеет очки жизни, скорость, уровень наносимого повреждения и т.д.  
Если эти параметры инжектить в конструктор, ***это шаг*** в сторону софт.   
Если эти данные устанавливать из констант, хранимых в самом классе- ***это шаг*** в сторону хард.
```
//to hard
public Predator() {
  this.hp = HP;  
  this.speed = SPEED;
  this.damage = DAMAGE;
  //oth's
}

//to soft
public Predator(int healthPoints, Speed speed, PathFinder finder, Damage damage)  <-- ТАК СЕЙЧАС
```
Как лучше сделать? В рамках этого проекта- можно и так и так.  
to hard подход соответствует KISS.  
to soft подход позволяет делать разные игровые конфигурации и более сложные алгоритмы, где разные экземпляры одной сущности будут иметь персонифицированные характеристики типа "быстрый волк", "медленный волк".

Выбор софт подхода при создании Хищника и его потенциальные возможности здесь мне нравится больше.

Но обратная сторона состоит в том, что теперь создавать хищника становится сложнее, ведь кто-то должен в себе хранить инициализационные значения его характеристик.  
А значит, нужно делать фабрику/фабрики существ, то есть, усложняется архитектура.  

Далее при создании фабрик снова появляется развилка.  
Можно сделать простую параметризованную фабрику для создания всех существ, либо семейство фабрик с отдельной фабрикой под каждое существо
```
//(1)
public interface EntityFactory {
   Entity get(Class<? extends Entity> clazz);
}
class DefaultEntitiesFactory implements EntityFactory 

//(2), СЕЙЧАС ТАК
public interface EntityFactory {
  Entity create();
}
class UnmovedEntitiesFactory implements EntityFactory 
class HerbivoreFactory implements EntityFactory 
class PredatorFactory implements EntityFactory 
```
Сейчас сделан выбор в пользу второго варианта и здесь кажется идет скатывание чисто в крайность софт-программирования. 
Потому что количество классов стремительно увеличивается и разобраться в их связях становится все труднее.  
И, как я писал выше, некоторые классы становятся полтергейстами, например `PredatorFactory`. Это просто посредник между конструктором Хищника: фабрика принимает в себя данные для хищника и вызывает конструктор хищника, передавая в него эти данные.  

В данном случае, создание экземпляра фабрики хищника происходит в классе `InitCommandsFactory`, он же хранит в себе константы для инициализации фабрики. Это те самые данные, которые транзитом идут в конструктор хищника.  
И вот на этом этапе софт-уклон теряет всякий смысл, потому что хотя фабрики сделаны супергибкими, все равно данные с характеристиками существ(жизни, скорость), все равно хрянятся в виде констант. Но уже в `InitCommandsFactory`.

Поэтому лично я бы сделал интерфейс фабрики, которая создает любое существо
```
public interface EntityFactory {
   Entity get(Class<? extends Entity> clazz);
}
```
Далее сделал бы реализацию фабрики, которая бы хранила в себе те константы, которые сейчас в себе хранит `InitCommandsFactory`
```
public class DefaultEntityFactory implements EntityFactory {
  private static final int HP = 100;
  private final PathFinder finder;
  //oth's

  private DefaultEntityFactory(SimulationMap simulationMap) {
    this.finder = new AStar(simulationMap);
  }

  @Override
  public Entity get(Class<? extends Entity> clazz) {
    return switch (clazz.getSimpleName()) {
      case "Tree" -> new Tree();
      case "Herbivore" -> new Herbivore(HP,  Speed.getRandomSpeed(), finder);
      //...
      default -> throw new IllegalStateException(/* message */);
    };
  }
}
```
И далее инжектил бы эту фабрику в Комманду
```
EntityFactory entityFactory = new DefaultEntityFactory(simulationMap);
List<Command> initCommands = List.of(
  new SpawnCommand(simulationMap, entityFactory), <-- ЗАСЕЛЯЕТ СУЩЕСТВ НА КАРТУ В НУЖНОМ КОЛИЧЕСТВЕ(РАССЧИТЫВАЕТ КОЛИЧЕСТВО ИЗ ПЛОЩАДИ)
  //
);
```
Тогда, вероятно, алгоритм стал бы более простым и понятным. Плюсы: можно делать разные реализации интерфейса фабрики существ, которые будут делать существ с разными индивидуальными характеристиками.

Но это так, информация к размышлению, переделывать не обязательно.  

## ВЫВОД

После рефакторинга стало значительно лучше. Оставшиеся замечания не особо существенные, грубых ошибок нет.  
На мой взгляд, имеется излишнее увлечение фабриками- количество их классов можно немного сократить, объединив несколько классов в один. 
Впрочем, это тоже скорее дело личного вкуса, а не принципиальное замечание.  
В целом, ок.

n.77(166)  
#ревью #симуляция