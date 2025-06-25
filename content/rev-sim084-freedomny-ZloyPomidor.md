https://github.com/ZloyPomidor/simulationJP/tree/master  
[FreedomНый]

После первого рефакторинга. Стало лучше, но есть над чем поработать.  
[Ревью на предыдущую версию](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim070-z-ZloyPomidor.md)

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Алгоритм поиска AStar

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Для избежания путаницы не называй пакеты, классы, интерфейсы так же, как называются классы и интерфейсы стандартной библиотеки.
Есть стандартный интерфейс `Map`, не должно быть кастомного пакета с таким же названием
```
map

//ПРАВИЛЬНО:
world_map
```

- Избыточный контекст
```
SimulationCreator.createSimulation()
Simulation.startSimulation()

//ПРАВИЛЬНО:
SimulationCreator.create()
Simulation.start()
```

- В методах, возвращающих булево значение, `is` пишется спереди
```
boolean coordinatesIsEmpty(Coordinates key)

//ПРАВИЛЬНО:
boolean isEmptyCoordinates(Coordinates coordinates)
```

- Название должно как можно лучше объяснять суть явления
```
userPressed = scanner.nextLine();
switch (userPressed) {
  case START_STOP -> running();
  //...
}

//ЛУЧШЕ:
command = scanner.nextLine();
switch (command) {
  case START_STOP -> running();
  //...
}
```

- Название метода должно быть глаголом в повелительном наклонении
```
void startingInteractionWithUser()

//ПРАВИЛЬНО:
void startInteractionWithUser()
```

- Это не просто текстовое сообщение, это шаблон текстового сообщения- он используется для форматированного вывода. 
И по поводу "STRING"- может быть, ты имел ввиду "STARTING"?
```
String STRING_MESSAGE = "Please select an action:  - %s (START) or %s - (EXIT)";

//ПРАВИЛЬНО:
String STARTING_MESSAGE_TEMPLATE = "Please select an action:  - %s (START) or %s - (EXIT)";
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Нарушение конвенции кода.** В любой ситуации выделяй тело блока скобочками, даже если тело состоит из одной строки. 
Исключение- метод equals(), там можно после if не выделять блоки скобочками
```
if (userPressed.equalsIgnoreCase(START_STOP)) {
  //...  
} else
  WorldConsoleRenderer.output(INCORRECT_INPUT_MESSAGE);

//ПРАВИЛЬНО:
if (userPressed.equalsIgnoreCase(START_STOP)) {
  //...  
} else {
  WorldConsoleRenderer.output(INCORRECT_INPUT_MESSAGE);
}
```

**3. class Coordinates**

- Эти поля могут быть публичными только в том случае, если бы этот класс являлся структурой. Но он *сейчас* не структура- в нем есть развитое поведение.
Эти поля должны быть приватными, а доступ к ним- через геттеры
```
public final int row;
public final int column;
```
Про отличия объектов от структур данных читай тут: *Мартин "ЧК", гл.6*

- Должен быть какой-то один вариант написания класса и его экземпляров- либо Coordinate, либо Coordinates
```
randomCoordinate = getRandomCoordinates(worldMap);
```

- Это должен быть простейший класс, который должен содержать только два числа, обозначающие положение точки на плоскости: x,y или row,column.
Лучше даже, если этот класс будет record'ом.

Здесь же класс отвечает не только за хранение координаты, но выполняет другие действия, тем самым нарушая SRP: возвращает случайную координату и случайную соседнюю координату в карте.
Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс.  
Например, в данном случае- `WorldMapUtils`.

- Нарушение Low Coupling. Координата знает про класс Карта, хотя знать этого координата не должна. 
Координата вообще не должна знать ни про какие пользовательские классы, потому что это знание ей не нужно для идентификации точки в пространстве.

- Для получения случайных чисел пользуйся классом `Random`, а не `Math.random()`- это проще
```
return WorldMap.MIN_COORDINATES_OF_THE_WORLD + (int) (Math.random() * (boundsValue - WorldMap.MIN_COORDINATES_OF_THE_WORLD + 1));
```

**4. class WorldMap**

- Не нужно специальной константы, которая будет обозначать точку отсчета координат, убери ее
```
public static final int MIN_COORDINATES_OF_THE_WORLD = 0;
```
Должно быть само собой разумеющимся, что координаты в карте начинаются с точки (0,0).
Википедия по этому поводу пишет:
```
В декартовой системе координат, начало координат — это точка, в которой пересекаются все оси координат. 
Это означает, что все координаты этой точки равны нулю. 
Например, на плоскости она имеет координаты (0,0)
```

- Нарушение инкапсуляции, все поля должны быть приватными, их чтение должно происходить через геттеры.
```
public final int column;
public final int row;
public final int mapSize;
```

- Названия переменных должны как можно лучше объяснять суть явлений. Это- ширина и высота карты
```
public final int column;
public final int row;

//ПРАВИЛЬНО:
public final int width;
public final int height;
```
В данном случае, встречаясь в коде, переменные карты column и row взаимодействуют с переменными таких же названий класса Координата. 
Но в координате эти названия означают другое явление- положение точки в пространстве. Взаимодествуя друг с другом, это вызывает путаницу
```
return current.row <= row && current.row >= MIN_COORDINATES_OF_THE_WORLD && current.column <= column && current.column >= MIN_COORDINATES_OF_THE_WORLD;

//ПРАВИЛЬНО:
return current.row <= height && current.row >= 0 && current.column <= width && current.column >= 0;
```

- Название `set` применяется, когда какое-то значение устанавливается. Например, `setName(String name)`, или `setWidth(int width)`. 
То есть изменяется состояние какой-то *уже существующей* перменной.
В данном случае, данные *добавляются*, а не устанавливаются. Например, в карте было 12 существ, а мы добавляем 13-е 
```
public void set(Coordinates coordinates, Entity entity)

//ПРАВИЛЬНО:
public void add(Coordinates coordinates, Entity entity)
```

- Нарушение SRP. В карте есть публичная переменная "площадь карты"
```
public final int mapSize;
this.mapSize = row * column;
```
Ок, а почему нет переменной периметр карты, диагональ карты, диаметр вписанной окружности? 
Эти значения точно так же не имеют отношение к единой ответственности карты по хранению существ, как и площадь карты.
Внутри самой Карты никак не используется значение площади для хранения/выдачи существ.

Здесь площадь карты нужно знать в интересах классов, которые помещают существа в карту. 
Вот пусть в каждом из этих классов будет происходить расчет площади карты. Либо метод расчета площади будет находиться в утилитном классе, например, `WorldMapUtils`.

+ 👍 При всех операциях с координатой, она предварительно валидируется.

+ 👍 Геттеры не возвращают null.

- Если метод что-то возвращает, то это возвращаемое значение нужно принимать в какую-то перменную
```
public void removeEntity(Coordinates coordinates) {
  isCoordinatesValid(coordinates); <-- ВОЗВРАЩАЕТ ЗНАЧЕНИЕ В ПУСТОТУ
  //...
}

//ПРАВИЛЬНО:
public void removeEntity(Coordinates coordinates) {
  boolean result = isCoordinatesValid(coordinates);
  //...
}
```
Другое дело, что с методом `isCoordinatesValid(...)` есть другие проблемы, о чем ниже.

- Если метод проверяет валидность координаты, как заявлено контрактом(именем) метода, то метод должен получить любую координату и сказать, валидна она или нет. 
То есть вернуть `true` или `false`. В данном случае, метод либо вернет `true`, либо бросит исключение
```
private boolean isCoordinatesValid(Coordinates coordinates) {
  if (!isCoordinatesAvailable(coordinates) && getEntity(coordinates) != null) {
    throw new IllegalArgumentException();
  }
  return true;
}
```

У тебя уже есть метод, который проверяет вхождение координаты в пределы карты, это `isCoordinatesAvailable(...)`. 
Поэтому метод `isCoordinatesValid(...)` нужно переделать вот так
```
private void validate(Coordinates coordinates) {
  //если координата НЕ находится в пределах карты,
  //бросить исключение
}
```
И тогда использование его будет более логичным и понятным
```
//БЫЛО:
public void removeEntity(Coordinates coordinates) {
  isCoordinatesValid(coordinates);
  entities.remove(coordinates);
}

//СТАНЕТ:
public void removeEntity(Coordinates coordinates) {
  validate(coordinates);
  entities.remove(coordinates);
}
```

+ 👍 Метод норм
```
public List<Coordinates> getCoordinatesByEntityType(Class<? extends Entity> searchingType)
```

+ 👍 В целом, с точки зрения SRP, класс хороший. Замечание про площадь- несущественное.

**5. Пакет utils**

Пакет содержит только один утилитный класс- MoveType. Остальные классы не утилитные. Гугли, что в java понимается под утилитным классом.

**6. class PathFinder**

Класс должен искать путь, что следует из его названия. На самом деле, класс ищет не путь, а первую точку пути
```
Coordinates getNextStep(Coordinates currentCoordinates )
```
Да, есть класс `AStar`, который ищет именно путь. А этот класс- обертка над AStar и выдает по одному шагу из пути, найденного классом AStar. 
Нужно хорошенько подумать над концепцией данного класса, потому что его суть не соответствует его названию.

**7. class AStar**

- Нарушение конвенции кода. Константы должны находиться в самом верху.

- Нарушение инкапсуляции. Публичным должен быть только метод `Deque<Coordinates> getPath(...)`.

+ 👍 Сигнатура метода поиска ок
````
public Deque<Coordinates> getPath(Coordinates start, Class<? extends Entity> targetType)
````

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar.  
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем получения и анализа Creature
```
public Deque<Coordinates> getPath(Coordinates start, Class<? extends Entity> targetType) throws GoalHasBeenNotFoundedException {
  Creature currentCreature = (Creature) worldMap.getEntity(start);
  //...
  if (!isGoalNotFunded(currentCreature.getTarget())) {...}
}
```
Карта не должна сама лезть в креатуру и получать из креатуры какие-то данные.

В данном случае это вдвойне непонятно, потому что метод уже принимает цель во входящие аргументы метода в виде 
```
Class<? extends Entity> targetType
```
Но потом снова получает эту же цель, но через currentCreature:
```
if (!isGoalNotFunded(currentCreature.getTarget())) {...}
```

- Большой, а потому трудно читаемый метод в 60 строк, а по ощущениям- в миллион строк
```
public Deque<Coordinates> getPath(Coordinates start,Class<? extends Entity> targetType)
```
Метод нуно разделить на несколько, каждый из которых будет делать только своё.  
*Фаулер "Рефакторинг", гл.6, "Извлечение метода"*  

- В дополнение к предыдушему пункту. Не должно быть больше 2-3 уровней вложенности- антипаттерн "Стрела"
```
while (...) {
  while (...) {
    for (...) {
      if (...) {
        if (...) {
          //тут наконечник стрелы
        }
      }
    }
  }
}
```

**8. abstract class Entity и его статичные наследники Grass/Rock/Tree**

👍 Всё ок.

**9. abstract class Creature extends Entity**

Нарушение инкапсуляции. Публичными должны быть только те методы, которыми пользуется клиентский код
```
public void moveEntity(WorldMap worldMap,Coordinates from, Coordinates to) {
  //используется только в классе Creature
}

public void eit() {
  //используется только в потомках  
}

//ПРАВИЛЬНО:
private void moveEntity(WorldMap worldMap,Coordinates from, Coordinates to) {
  //используется только в классе Creature
}

protected void eit() {
  //используется только в потомках  
}
```

**10. class Herbivore/Predator extends Creature**

+ 👍 Принципиально всё ок.

- В этом переопределении нет смысла- все равно в этом методе ты вызываешь метод предка. То есть, метод `hashCode()` из класса `Creature`.  
Если ты здесь уберешь метод hashCode(), он будет работать точно так же- по наследству от Creature
```
@Override
public int hashCode() {
  return super.hashCode();
}
```

**11. Пакет action.utils**

Не содержит ни одной утилиты.

**12. Action'ы**

👍 С ними всё ок. Один из экшенов вызывает отрисовку карты- почему бы и нет.

**13. class WorldConsoleRenderer**

- Нарушение инкапсуляции. Все константы и методы, кроме `render(...)`, должны быть приватными.

+ 👍 Хранит в себе спрайты существ, а не берет их из самих существ, это хорошо.

+ 👍 В свич-кейс в дефолте бросает исключение, это хорошо.

**14. Фабрика симуляции**

+ 👍 Фабрика это хорошо. 

+ 👍 Интерфейс фабрики тоже хорошо- можно делать разные реализации, которые бы по-разному создавали Симуляцию. 
То есть, Симуляцию с разной конфигурацией экшенов и карты.

- Нейминг немного путает
```
interface SimulationCreator 
class SimulationFactory implements SimulationCreator

//ЛУЧШЕ:
interface SimulationFactory 
class BasicSimulationFactory implements SimulationFactory
```

- Нарушение DRY. Ацкое дублирование кода в перегруженных конструкторах
```
public SimulationFactory() {
  this.world = new WorldMap(DEFAULT_WORLD_MAP_ROW, DEFAULT_WORLD_MAP_COLUMN);
  this.spawnData = new EntitiesFactory();
  this.initActions = List.of(new SpawnEntityAction(spawnData), new SimulationRenderAction(days));
  this.turnActions = List.of(new AddEntityAction(spawnData), new MoveCreatureAction(days), new SimulationRenderAction(days));
}

public SimulationFactory(int row, int column) {
  if (!isRowAndColumnValid(row, column)) {
    throw new IllegalArgumentException();
  }
  this.world = new WorldMap(row, column);
  this.spawnData = new EntitiesFactory();
  this.initActions = List.of(new SpawnEntityAction(spawnData), new SimulationRenderAction(days));
  this.turnActions = List.of(new AddEntityAction(spawnData), new MoveCreatureAction(days), new SimulationRenderAction(days));
}
//еще миллион конструкторов

//ПРАВИЛЬНО:
public SimulationFactory() {
  this(DEFAULT_WORLD_MAP_ROW, DEFAULT_WORLD_MAP_COLUMN);
}

public SimulationFactory(int row, int column) {
  if (!isRowAndColumnValid(row, column)) {
    throw new IllegalArgumentException();
  }
  this.world = new WorldMap(row, column);
  this.spawnData = new EntitiesFactory();
  this.initActions = List.of(new SpawnEntityAction(spawnData), new SimulationRenderAction(days));
  this.turnActions = List.of(new AddEntityAction(spawnData), new MoveCreatureAction(days), new SimulationRenderAction(days));
}
//еще миллион конструкторов
```

**15. class EntitiesFactory**, фабрика карты(?)

- Что это за ацкие комментарии? Задача комментариев- сделать код понятнее в тех случаях, когда этого нельзя достичь хорошим неймингом переменных и методов. 
Эти же комментарии только делают код загадочнее
```
public final double predator; //<|
public final double herbivore;// |
public final double grass;//     |
public final double tree;//      |
public final double rock;//    < \___ <=1.00
```
Правильное использование комментариев- *"Чистый код", гл.4*

- Название класса не соответствует тому, что он делает. 
Судя по названию, класс должен создавать существа. Но он делает все что угодно, только не это. Например, считает... что-то считает, короче
```
public int getTargetSpawnValue(Entity entity, WorldMap worldMap) {
  //...
  double spawnCounter = 0.0;
  for (String s : getSpawnValues()) {
    String entitySimpleName = entity.getClass().getSimpleName();
    if (isTargetSpawnValue(entitySimpleName, s)) {
      spawnCounter = getSpawnCounter(s);
    }
  }
  return (int) (spawnCounter * worldMap.mapSize);
}
```
Нужно разобраться с концепцией класса- понять, что конкретно он делает, зачем он нужен и назвать его соответственно этому.
После анализа того, что конкретно делает класс, может оказаться(а может и нет), что он выполняет разные ответственности. 
Тогда методы нужно будет разнести по их ответственностям в разные классы. 

- Нарушение инкапсуляции. Все поля должны быть приватными и читаться только через геттеры.

**16. class Simulation**

+ 👍 Массово применяются константы.

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий. 
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе `Simulation`
```
public Simulation(WorldMap worldMap, List<Action> initActions, List<Action> turnActions)
```

- В любом switch-case должен быть default
```
if (inputIsValid(userPressed)) {
  userPressed = userPressed.toUpperCase(Locale.ROOT);
  switch (userPressed) {
    case START_STOP -> running();
    case NEXT -> executeActions(turnActions);
    case EXIT -> exit();
  }
} else
  WorldConsoleRenderer.output(INCORRECT_INPUT_MESSAGE);

//ПРАВИЛЬНО:
̶i̶f̶ ̶(̶i̶n̶p̶u̶t̶I̶s̶V̶a̶l̶i̶d̶(̶u̶s̶e̶r̶P̶r̶e̶s̶s̶e̶d̶)̶)̶
userPressed = userPressed.toUpperCase(Locale.ROOT);
switch (userPressed) {
  case START_STOP -> running();
  case NEXT -> executeActions(turnActions);
  case EXIT -> exit();
  default -> WorldConsoleRenderer.output(INCORRECT_INPUT_MESSAGE);
} 
```
Метод `inputIsValid(...)` в классе явно лишний.

- Здесь можно сделать проще
```
private static final String STRING_MESSAGE = "Please select an action:  - %s (START) or %s - (EXIT)";
private static final String EXIT = "E";
private static final String START_STOP = "S";

private void startingInteractionWithUser() {
  //...  
  WorldConsoleRenderer.output(String.format(STRING_MESSAGE, START_STOP, EXIT));
}

//ПРАВИЛЬНО:
private static final String EXIT = "E";
private static final String START_STOP = "S";
private static final String STRING_MESSAGE = "Please select an action:  - %s (START) or %s - (EXIT)".formatted(START_STOP, EXIT);

private void startingInteractionWithUser() {
  //...  
  WorldConsoleRenderer.output(STRING_MESSAGE);
}
```

- Избыточно. В java при объявлении полей класса, если им не заданы значения, то они устанавливаются значениями по умолчанию. Для boolean это будет `false`
```
private boolean running = false;
private boolean finished = false;

//ПРАВИЛЬНО:
private boolean running;   // = false;
private boolean finished;  // = false;
```
А вот в `Си++` действительно при объявлении переменной всегда нужно устанавливать ей явное значение.

**17. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

+ 👍 Использует фабрику через её интерфейс, это хорошо
```
SimulationCreator simulationCreator = new SimulationFactory();
```

## ВЫВОД

Всегда используй поля классов не напрямую, а через геттеры и сеттеры.  
Константы могут быть публичными, но каждый раз нужно думать- а должны ли эти конкретные контанты быть доступными для сторонних классов или они нужны только для того класса, внутри которого эти константы находятся.

В классе `EntitiesFactory`, кажется, немного перемудрил.

Судя по классу `Coordinates`, понимание принципа единой ответственности еще недостаточное.  
Для лучшего понимания декомпозиции ООП посмотреть ролики Сергея про шахматы. 
Посмотреть ролики Немчинского про SOLID.

n.84(189)  
#ревью #симуляция 