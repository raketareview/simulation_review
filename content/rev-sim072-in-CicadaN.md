https://github.com/CicadaN/simulation  
[И N]

Хорошая реализация.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Цветовая индикация здоровья существ
+ 👍 Алгоритм поиска AStar
+ 🚀 Есть редактор карты
+ 👍 Два варианта запуска программы: с настройками по умолчанию и через редактор карты

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Метод не делает ход, он складывает две координаты
```
public Position move(int dx, int dy) {
  return new Position(x + dx, y + dy);
}
```

- Метод возвращает копию не существа, а существ
```
public Map<Position, Entity> getCopyEntity() {
  return Map.copyOf(entities);
}
```

- Метод с названием "Показать меню", должен только показывать меню, а этот метод показывает меню и возвращает конфиг
```
SimulationConfig config = menu.showMenu();
```

- UPPER_SNAKE только для констант, а это не константа
```
private final int DEFAULT_TREES = 10;
```

- Эту переменную будут постоянно путать с экземпляром `Map`, а не `GameMap`
```
WorldMap map;

//ПРАВИЛЬНО:
WorldMap worldMap;
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (dist < minDist) minDist = dist;
if (closedSet.contains(current)) continue;

//ПРАВИЛЬНО:
if (dist < minDist) {
  minDist = dist;
}
if (closedSet.contains(current)) {
  continue;
}
```

**3. Нарушение DRY**, магические буквы, числа, слова. Вводи константы 
```
int width = readInt("Введите ширину карты (5-20): ", 5, 20);
int height = readInt("Введите высоту карты (5-20): ", 5, 20);

//ПРАВИЛЬНО:
private final static int GAME_MAP_SIZE_MIN = 5;
private final static int GAME_MAP_SIZE_MAX = 20;

String widthMessage = "Введите ширину карты (%d-%d): ".formatted(GAME_MAP_SIZE_MIN, GAME_MAP_SIZE_MAX);
int width = readInt(widthMessage, GAME_MAP_SIZE_MIN, GAME_MAP_SIZE_MAX);
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*   
*refactoring.guru "Замена магического числа символьной константой"*  

**4. class WorldMap**

- Нарушение SRP, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: создать список всех координат. Это возможно полезный метод для проекта, но он не нужен Карте для реализации ее ответственности по хранению/выдаче существ.

Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Почему `target` валидируется, а `pos` нет?
```
public void moveEntity(Position pos, Position target) {
  //...
  if (!isWithinBounds(target)) {
    throw new IllegalArgumentException("Invalid target" + target);
  }
  //...
}
```

- Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
карта.addEntity(new Position(0, 0), new Заяц());
карта.moveEntity(new Position(0, 0), new Position(99, 99));

/* class WorldMap */
public void moveEntity(Position pos, Position target) 
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void removeEntity(Position pos) {
  entities.remove(pos);
}

public boolean isOccupied(Position pos) {
  return entities.containsKey(pos);
}
```

+ 👍 В целом, замечаний к классу намного меньше, чем обычно.

**5. class AStarPathFinder**

Сигнатура метода поиска ок, но структура предиката мне не нравится
```
public List<Position> findPath(WorldMap map, Position start, Predicate<Position> isGoal)
```
Короче, работает это так: при вызове `findPath(...)` в него в качестве аргумента передается `WorldMap map` и в этй карте ищется путь.  
При этом также передается предикат для поиска цели`Predicate<Position> isGoal`- в предикат отправляется координата, далее этот предикат проверяет *по своему экземпляру WorldMap*, 
находится ли на этой координате еда или нет. Таким образом, поиск пути происходит в рамках одного экземпляра карты, а поиск еды- в рамках другого экземпляра. 
Да, здесь это один и тот же экземпляр, но сам архитектурный подход мне не нравится.  

Мой альтернативный вариант:
```
public List<Position> findPath(WorldMap map, Position start, Predicate<Entity> isGoal)
```
Ниже, в креатурах, рассмотрим, как такой вариант мог бы работать.

**6. abstract class Creature extends Entity**

- Метод хода с креатуре не должен принимать в себя кроме карты еще и координату этой креатуры. 
Потому что в этом случае мы делаем допущение, что вызывающий код передаст корректную координату. То есть координату, на которой находится именно это существо, а это далеко не гарантировано  
```
public abstract Position makeMove(WorldMap map, Position position);
```

Свою координату креатура должна получать одним из двух способов: либо дублировать свою координату в самой себе, либо спрашивать из карты: `Position position = wordMap.getPosition(this);`

- При подсчете повреждений нужно не выходить за граничные значения здоровья. 
Что, если `health` после повреждения станет меньше нуля? 
Что, если в `takeDamage(...)` передадут отрицательную величину?
```
public void takeDamage(int damage) {
  health -= damage;
}
```

**7. class Herbivore/Predator extends Creature**

Среди прочего, здесь формируется тот самый предикат
```
Predicate<Position> isGoal = pos ->
  map.getEntity(pos).filter(e -> e instanceof Grass).isPresent();
```

Мое предложение: сделать в креатуре определитель еды. Это будет абстрактный метод, который будет переопределяться у потомков. Например, так
```
@Override
protected boolean isFood(Entity entity) {
  return entity.getClass() == Заяц.class || entity.getClass() == КраснаяШапочка.class;
}
```

И тогда, после переделки поиска, в него креатура будет передавать этот определитель еды
```
List<Position> path = pathFinder.findPath(map, currentPosition, this::isFood);
```

**8. class SimulationConfig**

Объект конфигурации- используется для хранения настроек
```
public class SimulationConfig {
  private final int width;
  private final int height;
  //еще миллион полей

  public SimulationConfig(int width, int height, int herbivoresCount, int predatorsCount, int grassCount, int grassPerTurn) {
    this.width = width;
    this.height = height;
    //...
  }

  public int getWidth() {
    return width;
  }

  public int getHeight() {
    return height;
  }
  //еще миллион геттеров
}
```
Класс хранит только интовые поля, поэтому при инициализации объекта через конструктор нужно быть предельно осторожным- можно легко перепутать последовательность переменных.  
Чтобы не перепутать, используются комментарии
```
new SimulationConfig(
  10,  // width
  10,  // height
  5,   // herbivores
  2,   // predators
  8,   // grass
  1    // grass per turn
);
```
Для более безопасного использования можно пойти двумя путями. Либо заполнять объект напрямую через публичные поля. Эдесь это будет норм, потому что эта конфигурация может быть структурой: *"ЧК", гл.6*  
Но тогда есть риск, что не все поля будут инициализированы
```
SimulationConfig config = new SimulationConfig();
config.width = 10;
config.height = 10;
```
Либо создавать объект через паттерн "Билдер"
```
SimulationConfig config = new SimulationConfig.Builder()
  .addWidth(10)
  .addHeight(10)
  //oth's
  .build();  <-- тут среди прочего можно провалидировать введенные данные
```
Ну, или можно не загоняться и оставить как есть.

**9. class InitAction implements Action**

Дублирование кода
```
for (int i = 0; i < numGrass && index < positionList.size(); i++) {
  map.addEntity(positionList.get(index++), new Grass());
}
for (int i = 0; i < DEFAULT_TREES && index < positionList.size(); i++) {
  map.addEntity(positionList.get(index++), new Tree());
}
//еще миллион дублей

//ПРАВИЛЬНО:

spawn(map, Tree::new, treeAmount);
spawn(map, Grass::new, grassAmount);
//...

private void spawn(WorldMap worldMap, Supplier<Entity> entitySupplier, int amount) {
  for (int i = 0; i < amount; i++) {
    worldMap.addEntity(getRandomPosition(), entitySupplier.get());
  }
}
```

**10. class MoveCreaturesAction implements Action**

- Нарушение SRP. class MoveCreaturesAction должен просто обойти всю карту, найти каждую креатуру и дать ей пинка, чтобы она побежала.
При этом как будет бежать креатура и что при этом делать, не должно волновать MoveCreaturesAction. Просто переместись по карте или атаковать, это должна решать сама креатура.  
Здесь же класс управляет действиями креатуры и дергает ее за ниточки: в этом случае просто иди, а в этом случае атакуй.

- При правильной архитектуре- нет
```
if (creature.isDead()) continue; // Доп проверка, нужна ли?
```
Сначала нужно решить проблему на уровне Creature: что делать, если существо умерло, но ему дают команду двигаться? Может ли мёртвый негр играть в баскетбол? Возможно, в этом случае стоит кинуть исключение.  

Далее, при вызове экшена ходьбы, на карте не должно быть мертвых креатур.  
Мертвые креатуры должны быть удалены ранее тем или иным способом- либо самими хищниками, либо специальным экшеном, который запустится перед экшеном совершения хода и уберет с карты всю падаль.  

И вот тогда проверка "на умер" будет не нужна и даже вредна- если в алгоритме будет баг и игра попытается двинуть мертвое существо, вылетит исключение и сразу станет понятно, что есть баг и его нужно исправить. 

**11. class AnsiColors**, константный класс с кодами цветности

- Константные классы делай final, их конструктор должен быть приватный. Чтобы нельзя было унаследоваться или сделать экземпляр класса.

- Возможно, стоит сделать не константный класс для хранения цветности, а енам.

**12. class ConsoleRenderer**

- Если вместо существа пытаются распечатать null, это баг и его нельзя заметать под ковер. Нужно бросать исключение 
```
if (entity == null) return UNKNOWN_SPRITE;
```

- Если в распечатку попадает неизвестное существо, нужно кидать исключение, а не распечатывать символ неизвестного существа.  
В первом случае произойдет быстрое падение и баг будет обнаружен и исправлен сразу.  
Во втором случае глючная программа будет работать долго, вызывая недоумение у пользователя.

В распечатку в принципе не должно попадать неизвестное существо. Но если попадает- это баг алгритма, когда при добавлении нового существа в проект, его спрайт просто забыли добавить в рендерер.

- Избавься от промежуточных методов для определения цвета фона через енам HealthLevel. Сделай метод, который будет получать существо, а возвращать цвет
```
private static final double LOW_HEALTH = 0.3;

private String toColor(Entity entity) {
  if(!(entity instanceof Creature)) {
    return AnsiColors.RESET;
  }
  
  Creature creature = (Creature) entity; 
  double healthRate = (double) creature.getHealth() / creature.getMaxHealth();

  if(healthRate < LOW_HEALTH) {
    return AnsiColors.BACKGROUND_RED;
  }
  //...
}
```

**13. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе симуляции
```
public Simulation(???)
```
```
public Simulation(WorldMap map,
  List<Action> initActions,
  List<Action> turnActions,
  ConsoleRenderer renderer)
```

- А вот ConsoleRenderer он в конструктор принимать не должен. Класс должен создавать тот, кто его использует: паттерн GRASP "Creator". 
Другое дело, если бы был интерфейс `GameMapRenderer` и отдельные его реализации, например консольная.
Тогда Симуляция должна была бы принимать в себя `GameMapRenderer gameMapRenderer`.

**14. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

В целом, ок.

n.72(160)  
#ревью #симуляция 