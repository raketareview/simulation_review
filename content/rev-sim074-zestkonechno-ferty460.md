https://github.com/ferty460/simulation  
[жесть, конечно]

В целом ок. Местами трудно разобраться в коде, но это связано с большим количеством точек расширения, которые предусмотрены архитектурой проекта. 
Отсюда- разросшееся количество классов и повысившаяся сложность.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Хранение настроек игры во внешнем файле, позволяет менять игровую конфигурацию без перекомпилирования программы

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Избыточный контекст
```
Optional<Coordinates> getCoordinatesOfEntity(Entity entity) 

//ЛУЧШЕ:
Optional<Coordinates> getCoordinates(Entity entity) 
```

- Не называй свои классы и интерфейсы так, как называются системные классы и интерфейсы, это приводит к путанице
```
public class Logger <-- Есть системный класс с таким имененм: java.util.logging.Logger
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Для получения случайного числа** объективно удобнее использовать класс Random
```
health = (int) Math.min(maxHealth, this.health + amount);
```

**3. Если в блоке if есть return**(break, continue, throw, exit и т.д.), то else не пишется - в этом случае неважно, будет else или нет, так как программа будет работать одинаково и код выглядит читабельней
```
if (entity instanceof Creature creature) {
  return String.format(" %s ", getSpriteForCreature(creature));
} else {
  throw new IllegalArgumentException("Unknown Entity: " + entity.getClass().getSimpleName());
}

//ПРАВИЛЬНО:
if (entity instanceof Creature creature) {
  return String.format(" %s ", getSpriteForCreature(creature));
}
throw new IllegalArgumentException("Unknown Entity: " + entity.getClass().getSimpleName());
```

**4. record Coordinates(int row, int column)**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

+ 👍 Хорошо, что координаты называются row и column а не x и y. Мне так больше нравится.  
В этом случае нет, как часто бывает, путаницы с порядком расположения в массивах: координата(x,y), а в массивах наоборот- массив[y, x].

**5. class WorldMap**

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void putEntityAt(Coordinates coordinates, Entity entity) {
  entities.put(coordinates, entity);
}
public Optional<Entity> getEntityAt(Coordinates coordinates) {
  return Optional.ofNullable(entities.get(coordinates));
}
```

+ 👍  В целом, притензий к этому классу почти нет.

**6. class WorldMapUtils**

Этот метод необходим карте для реализации ее единой ответственности- при всех операциях в карте с участием координат, координату нужно проверять на вхождение в карту.
Поэтому этот метод должен находиться в Карте
```
boolean isWithinBounds(Coordinates coordinates, WorldMap worldMap)
```

**7. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.

+ 👍 Предикат для определения цели поиска это хорошо
```
List<Coordinates> find(WorldMap map, Coordinates start, Predicate<Entity> targetCondition);
```
Теперь существа в качестве еды могут через класс поиска искать объекты более чем одного класса
```
Predicate<Entity> food = en -> en instanceof Заяц || en -> en instanceof КраснаяШапочка;
List<Coordinates> path = pathFinder.find(map, from, food);
```

**8. class Config**, класс с константами

- Конструктор должен быть приватный
```
public Config() {
  throw new UnsupportedOperationException("Utility class");
}
```

- Константный класс здесь используется неправильно- другие классы напрямую лезут в константы вместо того, чтобы получать необходимые этим классам данные в свой конструктор.  
Это делает классы неуниверсальными и зависимыми от класса констант.  

Условный пример
```
public final class Settings {
  //приватный конструктор
  public static final int HOUSE_ROOMS = 5;
}

/ПЛОХО:
public static void main(String[] args) {
  House house = new House();
  //oth code
}

public class House {
  private final int rooms;

  public House() {
    this.rooms = Settings.HOUSE_ROOMS;
  }

  public int getRooms() {
    return rooms;
  }
}

//ХОРОШО:
public static void main(String[] args) {
  House house = new House(Settings.HOUSE_ROOMS);
  //oth code
}

public class House {
  private final int rooms;

  public House(int rooms) {
    this.rooms = rooms;
  }

  public int getRooms() {
    return rooms;
  }
}
```
Про классы констант, конфигурации и их использование я писал тут: https://t.me/zhukovsd_it_chat/53243/176984

**9. пакет behavior**, поведение креатур

В этом пакете находится `interface Behavior`, который декларирует поведение креатур, а также классы, реализующие это поведение для разных креатур.
Таким образом, единая сущность потомка креатуры делится на две: собственно потомок креатуры и класс с его поведением.

Решение дискуссионно.
- Пюсы: 
    - в разные экземпляры креатур можно внедрять разное поведение.  
- Минусы: 
    - сейчас для одного вида существ декларировано одно поведение (класс Заяц - класс ПоведениеЗайца)
    - в Зайца можно внедрить поведение Волка и наоборот 
    - структура проекта заметно усложняется

**10. class Herbivore/Predator extends Creature** 

Неправильная инициализация- через константный класс. Инициализационные параметры класс должен получать в конструктор
```
public Herbivore() {
  this(
    Config.getInt("herbivore.speed", MIN_SPEED, MAX_SPEED),
    Config.getInt("herbivore.health", MIN_HEALTH, MAX_HEALTH),
    Config.getInt("herbivore.energyConsumption", MIN_ENERGY_CONSUMPTION, MAX_ENERGY_CONSUMPTION),
    Config.getInt("herbivore.nutritionalValue", MIN_NUTRITIONAL_VALUE, MAX_NUTRITIONAL_VALUE)
  );
}

//ПРАВИЛЬНО:
public Herbivore(int speed, int health, int energyConsumption, int nutritionalValue) {...}
```

**11. abstract class RegrowEntityAction<T extends Entity> implements Action** и его потомки RegrowGrassAction, RegrowHerbivoreAction, RegrowPredatorAction

Куча классов, которые делают одно и то же- заселяют карту существами определенного вида. Лучше сделать один универсальный класс, который будет заселять любым существом.  

**12. пакет menu**

👍 Меню в ООП стиле, ок.

**13. interface Renderer**

👍 Интерфейс рендерера это хорошо. Теперь можно делать разные рендереры для разных визульных сред(консоль, интерфейс виндовс, http, матричный принтер etc)
и разного отображения информации(цветной, черно-белый etc)

**14. class ConsoleRenderer implements Renderer**

- Дублирование кода, магические строки. Если придется менять шаблон, например делать его шире или уже, то нужно будет это делать во многих строках
```
if (entity instanceof Rock) {
  return String.format(" %s ", ROCK_SPRITE);
}

if (entity instanceof Tree) {
  return String.format(" %s ", TREE_SPRITE);
}

//ПРАВИЛЬНО:
if (entity instanceof Rock) {
  return String.format(SPRITE_TEMPLATE, ROCK_SPRITE);
}

if (entity instanceof Tree) {
  return String.format(SPRITE_TEMPLATE, TREE_SPRITE);
}
```

- Непонятно, почему преобразование существ в спрайты разделено на два метода. Мне кажется, достаточно было бы одного
```
private String getSpriteForEntity(Entity entity) {...}
private String getSpriteForCreature(Creature creature) {...}
```

**15. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
public Simulation(WorldMap worldMap, Renderer renderer, List<Action> initActions, List<Action> turnActions) 
```

- Если есть специальный интерфейс рендерера, который печатает не только карту, но и текстовую информацию, то вообще всю информацию нужно печатать через этот рендерер
```
System.out.println("Press Enter to stop...");
renderer.printCurrentTurn(currentTurn++);

//ПРАВИЛЬНО:
renderer.printСообщениеПроТоКакСделатьСтоп();
renderer.printCurrentTurn(currentTurn++);
```
Иначе теряется смысл существования интерфейса рендерера в том виде, в котором он сейчас явлен.  
Например, если игра будет использовать HttpRenderer, то часть информации будет отображаться по сети, а часть- печататься в консоль на локальном ПК.

**16. class Main**, содержит точку входа main

👍 Только создает и запускает игру, в данном случае через класс `Application`, это хорошо.

## ВЫВОД

В целом ок.

n.74(163)  
#ревью #симуляция 