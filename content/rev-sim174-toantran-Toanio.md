https://github.com/Toanio/simulation_by_Toanio  
[Toan Tran]


Есть над чем поработать. Программа сделана частично.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Карта распечатывается неровно, нужно подобрать спрайты одинаковой ширины.

2. Нет паузы/пуск во время работы. Хотя это требование есть в ТЗ.

## ХОРОШО

+ 👍 Замечаний к коду меньше, чем бывает обычно

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Придерживайся единообразия
```java
Entity getEntityByCoordinate(Coordinate coordinate)
void deleteEntity(Coordinate coordinate) 

//ПРАВИЛЬНО:
Entity getEntity(Coordinate coordinate)
void deleteEntity(Coordinate coordinate) 
```

- Венгерская нотация.

В названии переменных не пиши тип данных, к которым они относятся.  
И вообще не употребляй венгерскую нотацию. 
Название переменной должно отвечать на вопрос что хранит переменная, а не как хранит
```java
List<Creature> creaturesList = new ArrayList<>();

//ПРАВИЛЬНО:
List<Creature> creatures = new ArrayList<>();
```

- Метод, возвращающий список, должен называться во множественном числе
```java
List<Creature> getCreature() 

//ПРАВИЛЬНО:
List<Creature> getCreatures()
```

- Согласовывай названия 
```java
public List<Creature> getCreature() {
  List<Creature> creaturesList = new ArrayList<>();
  //...
  return creaturesList;
}

//ПРАВИЛЬНО:
public List<Creature> getCreatures() {
  List<Creature> creatures = new ArrayList<>();
  //...
  return creatures;
}
```

- Избыточный контекст. 

Мы и так понимаем, что метод поиска в классе поиска пути ищет именно путь, а не что-то иное
```java
public class PathFinder {
  public List<Coordinate> findPath(GameMap map, Coordinate start, Class<?> targetType) {...}
}

//ПРАВИЛЬНО:
public class PathFinder {
  public List<Coordinate> find(GameMap map, Coordinate start, Class<?> targetType) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Если в блоке if есть return(break, continue, throw, exit и т.д.), то else не пишется**

В этом случае неважно, будет else или нет, так как программа будет работать одинаково, а код без else будет выглядеть читабельней
```java
if (isCoordinateValid(coordinate)) {
  return !grid.containsKey(coordinate);
} else {
  return false;
}

//ПРАВИЛЬНО:
if (isCoordinateValid(coordinate)) {
  return !grid.containsKey(coordinate);
} 
return false;
```

**3. Exceptions**

- Текст в Exception всегда должен быть на английском языке.
```java
throw new IllegalArgumentException("Файл config.json не найден в resources!");  <-- Неправильно: сообщение в исключении не на английском языке
```
Исключение это не просто телеграмма, которая летит сквозь слои.

У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.  
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты.  
А значит, сообщение должно быть на английском.

Интерпретация исключения и перевод его на локальный язык должны происходить там, где это соответствует архитектуре программы.  
Или не происходить вовсе, если исключение не планируется перехватывать.

- Не бросай базовый exception. Конкретизируй ситуацию- бросай то исключение, которое подходит под этот конкретный случай
```java
throw new RuntimeException("Не удалось прочитать конфигурационный файл", e);
```
*Хорстманн "Java. Библиотека профессионала", т.1, гл.11*
```java
"Не ограничивайтесь генерацией RuntimeException. 
Найдите подходящий подкласс или создайте собственный." - Хорстманн
```

**4. record Coordinate(int x, int y)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Coordinate(int x, int y) {
}
```

**5. class GameMap**

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, вернуть существо по координате, вернуть координату по существу, удалить существо, вернуть список всех хранимых существ.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь методы чужих ответственностей:  
🔸 Сделать ход существом    
🔸 Найти случайную пустую координату   
🔸 Найти всех креатур

Наверное, для проекта в целом полезно иметь метод, который находит всех креатур на карте.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать всех креатур.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, "BoardUtils".

Конкретно здесь нужно определить тот класс, который не может выполнять свою работу без нахождения списка всех креатур на карте.  
Это класс `SpawnAction`, именно там должен находиться поиск случайной пустой координаты в карте.  

- Нарушение SRP.

Карта должна вставлять существо на заданную координату.  
Получать эту координату карта должна извне.  
Карта не должна сама опрашивать существо, чтобы выяснить, куда нужно его поставить
```java
public void addEntity(Entity entity) {
  grid.put(entity.getCoordinate(), entity);
}

//ПРАВИЛЬНО:
public void addEntity(Entity entity, Coordinate coordinate) {
  grid.put(coordinate, entity);
  entity.setCoordinate(coordinate);
}
```

- Никогда не возвращай null
```java
private final Map<Coordinate, Entity> grid = new HashMap<>();

public Entity getEntityByCoordinate(Coordinate coordinate) {
  return grid.get(coordinate);  <-- вернет null если в grid нет coordinate
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- При всех операциях с участием координаты (добавить, выдать, удалить и т.д.) нужно проверять координату на корректность.

Сейчас в карту можно вставить существо на координату, выходящую за размер карты.  
Если координата некорректна (находится вне пределов карты), нужно бросать исключение:
```java
public void addEntity(Entity entity) {
  grid.put(entity.getCoordinate(), entity);
}

//ПРАВИЛЬНО:
public void addEntity(Entity entity) {
  validate(coordinates);  <-- Если координата не в пределах карты, то бросает исключение 
  grid.put(entity.getCoordinate(), entity);
}
```
Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Метод совершения хода в карте- нарушение SRP.

Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```java
boolean moveEntity(Entity entity, Coordinate coordinate) {
  //телепортирует entity из её текущего места на coordinate  
}
```
Должна ли карта учитывать правила логистики зайцев? Если да, то как?

Например, имеет ли право заяц телепортироваться на координату, которая уже занята?  
Может ли телепортироваться камень?  
Может ли телепортироваться пустое место на голову волка?

Метод частично отвечает на эти вопросы- вместе со вспомогательным методом, за это отвечает 9 строк кода.  
То есть карта не только хранит существ, но и хранит правила того, как они должны ходить.  
Тем самым карта забирает себе часть чужой ответственности.

Единственно правильный ответ- карта вообще не должна иметь в себе метод телепортации.

- Нарушение OCP. 

Карта должна работать со всеми хранимыми существами одинаково.  
И не должна работать как-то по-особому с конкретными классами-наследниками Entity
```java
public List<Creature> getCreature()

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент сам отбирает отсюда креатуры

//ИЛИ ТАК:
public <T extends Entity> List<T> getEntitiesBy(Class<T> type)  //вернуть существ класса, указанного клиентом 
```
Если карта будет знать по именам наследников Entity и иметь для них персональные методы, то класс будет открыт для изменений.  
Например, при добавлении в проект класса Птица, понадобится изменить класс Карта и добавить в него метод `getBirds()`.

- Неправильная работа метода.

Сейчас если спросить у карты, свободна ли ячейка с координатой (+100500, -100500), то скажет, что занята
```java
static void main() {
  GameMap gameMap = new GameMap(10, 10);
  Coordinate coordinate = new Coordinate(+100500, -100500);
  System.out.printf("%s its free? - %b", coordinate, gameMap.isCoordinateEmpty(coordinate));
}

//РЕЗУЛЬТАТ:
Coordinate[x=100500, y=-100500] its free? - false
```
Правильный ответ: кинуть исключение, если координаты нет в карте.

- Особые случаи обрабатывай первыми
```java
if (isCoordinateEmpty(coordinate)) {
  deleteEntity(entity.getCoordinate());
  entity.setCoordinate(coordinate);
  addEntity(entity);
  return true;
} else {
  return false;
}

//ПРАВИЛЬНО:
if (!isCoordinateEmpty(coordinate)) {  //что-то в аргументе не устраивает- просто выходим
  return false;
}

deleteEntity(entity.getCoordinate());
entity.setCoordinate(coordinate);
addEntity(entity);
return true;
```

**6. class PathFinder**

- Пустой конструктор не нужно писать.

Если в классе используется только один пустой *публичный* конструктор, то его не нужно писать.  
Этот конструктор уже есть по умолчанию
```java
public class PathFinder {

  public PathFinder() {  <-- Убрать
  }

  //...
}
```

+ 👍 Идеальная сигнатура поиска.

Сигнатура поиска соответствует алгоритму BFS и сразу понятно, как пользоваться поиском
```java
public List<Coordinate> findPath(GameMap map, Coordinate start, Class<?> targetType) {...}
```

- Антипаттерн "Стрела".

Не должно быть больше 2-3 уровней вложенности.  
Если больше, это антипаттерн "Стрела", такой код очень труден для понимания
```java
while (!queue.isEmpty()) {
  if (targetType.isInstance(entity)) {
    //...
    return path.reversed();
  } else {
    for (Coordinate neighbor : neighbors) {
      if (map.isCoordinateValid(neighbor) && !visited.contains(neighbor) ... ) {
        //наконечник стрелы
      }
    }
  }
}
```
"Стрела" значит, что метод делает несколько дел сразу и его нужно разделить на несколько вспомогательных.
```java
"Если вам нужно более трех уровней вложенности, вы все равно запутались, так что исправьте программу" - Линус Торвальдс
```

Конкретно здесь стрела лечится просто- если в блоке `if` есть return, то `else` не пишется:
```java
while (!queue.isEmpty()) {
  if (targetType.isInstance(entity)) {
    //...
    return path.reversed();
  } 
  
  for (Coordinate neighbor : neighbors) {
    if (map.isCoordinateValid(neighbor) && !visited.contains(neighbor) ... ) {...}
  }
}
```

- Не делай названия в блоках `if` сложными и нечитаемыми. 

Делай их простыми и читаемыми с помощью вспомогательного метода или поясняющей переменной
```java
if (map.isCoordinateValid(neighbor) && !visited.contains(neighbor) && (map.isCoordinateEmpty(neighbor) || targetType.isInstance(map.getEntityByCoordinate(neighbor)))) {...}

//ПРАВИЛЬНО (ВСПОМОГАТЕЛЬНЫЙ МЕТОД):
if (isНазваниеКотороеВсёОбъясняет(...)) {...}

//ПРАВИЛЬНО (ПОЯСНЯЮЩАЯ ПЕРЕМЕННАЯ):
boolean isНазваниеКотороеВсёОбъясняет = map.isCoordinateValid(neighbor) && !visited.contains(neighbor) && (map.isCoordinateEmpty(neighbor) || targetType.isInstance(map.getEntityByCoordinate(neighbor)));

if (isНазваниеКотороеВсёОбъясняет) {...}
```

- Повторяющиеся действия делай через циклы
```java
public List<Coordinate> findPath(GameMap map, Coordinate start, Class<?> targetType) {
  //...    
  List<Coordinate> neighbors = new ArrayList<>();

  Coordinate leftNeighbor = new Coordinate(current.x() - 1, current.y());
  Coordinate rightNeighbor = new Coordinate(current.x() + 1, current.y());
  //...
        
  neighbors.add(leftNeighbor);
  neighbors.add(rightNeighbor);
  //...
}

//ПРАВИЛЬНО:
private static final List<Coordinate> SHIFT_COORDINATES = List.of(
    new Coordinate(-1, 0),
    new Coordinate(1, 0),
    //oth's coord's
);

public List<Coordinate> findPath(GameMap map, Coordinate start, Class<?> targetType) {
  //...    
  List<Coordinate> neighbors = getNeighbors(start);
  //...
}

private List<Coordinate> getNeighbors(Coordinate coordinate) {
  List<Coordinate> neighbors = new ArrayList<>();
  
  for(Coordinate shift : SHIFT_COORDINATES) {
    Coordinate neighbor = new Coordinate(coordinate.x() + shift.x(), coordinate.y() + shift.y());
    neighbors.add(neighbor);
  }

  return neighbors;
}
```

**7. abstract class Entity**

- Содержит координату
```java
public abstract class Entity {
  private Coordinate coordinate;
  //...
}
```

Но координата нужна только тому существу, которое ходит.  
Поэтому entities должны хранить координату только начиная с уровня `Creature`.  
В иерархии классов все состояния и поведения должны появляться только на тех уровнях наследования, где они начинают использоваться.

- Нарушение SRP, зависимость модели от представления- существо хранит спрайт с собственным изображением
```java
public abstract class Entity {
  private final String image;
    //...
}
```
Модель(а это модель) не должна зависеть от представления и знать, как ее будут показывать юзеру.  
Потому что в разных средах (консоль, Swing, Android) одна и та же модель может быть показана разными способами- пиксельной картинкой, анимацией etc.  
Спрайты всех существ должны храниться в классе, который распечатывает карту.

- Существа могут иметь имена, но только имена личные.

Существа могут иметь личные имена и хранить их в себе: "заяц Степан", "волк Павел Иванович"  
```java
public abstract class Entity {
  private final String name; <-- Норм, если имя личное. Плохо, если это имя всего типа
  //...
}
```

Но если в этом имени хранится просто "заяц" или "волк", то есть имя типа существа, то это ошибка.  
Определять принадлежность объектов к типам нужно стандартными способами: `instanceof`, `getClass()`.

**8. class Rock extends Entity**

Вместо персональных имён существам даётся имя всего типа 
```java
public class Rock extends Entity {
  public Rock(Coordinate coordinate) {
    super("Камень", "\uD83E\uDEA8", coordinate);
  }
}
```
Среди прочего, это зависимость модели от представления- русские тексты захардкодены в коде класса.  
Поэтому этот класс-модель можно использовать только в русскоязычной версии программы.  

**9. abstract class Creature extends Entity**

Если значение поля не меняется во время жизни объекта, делай его `final`
```java
private Class<?> target;

//ПРАВИЛЬНО:
private final Class<?> target;
```

**10. abstract class Creature extends Entity и его наследники**

👍 За исключением унаследованных проблем, выглядит хорошо.

**11. interface Action**

👍 Идеально
```java
public interface Action {
  void execute(GameMap map);
}
```

**12. class MakeMoveAction implements Action**

👍 Почти хорошо
```java
public void execute(GameMap map) {
  List<Creature> creatures = map.getCreature();
  for (Creature creature : creatures) {
    creature.makeMove(map);
  }
}
```
В карте не должно быть метода `map.getCreature()`.

**13. abstract class SpawnAction implements Action и его наследники**

Избыточно
```java
public abstract class SpawnAction implements Action {
  private final int countEntity;

  public SpawnAction(int countEntity) {
    this.countEntity = countEntity;
  }

  abstract Entity createEntity(Coordinate coordinate);

  @Override
  public void execute(GameMap map) {
    for (int i = 0; i < countEntity; i++) {
      map.addEntity(createEntity(map.getRandomEmptyCoordinate()));
    }
  }
}

public class SpawnGrassAction extends SpawnAction {

  public SpawnGrassAction(int countEntity) {
    super(countEntity);
  }

  @Override
  Entity createEntity(Coordinate coordinate) {
    return new Grass(coordinate);
  }
}
```

Используй стандартные функциональные интерфейсы Java:
```java
public abstract class SpawnAction implements Action {
  private final int countEntity;
  private final Function<Coordinate, Entity> entityCreator;

  public SpawnAction(Function<Coordinate, Entity> entityCreator, int countEntity) {...}

  @Override
  public void execute(GameMap map) {
    for (int i = 0; i < countEntity; i++) {
      Coordinate coordinate = ...
      Entity entity = entityCreator.apply(coordinate);  
      //Вставить существо в карту на координату
    }
  }
}

public class SpawnGrassAction extends SpawnAction {
  private static final Function<Coordinate, Entity> ENTITY_CREATOR = (coordinate) -> new Grass(coordinate);

  public SpawnGrassAction(int countEntity) {
    super(ENTITY_CREATOR, countEntity);
  }
}
```

**14. class MapRenderer**

- Спрайты существ должны храниться здесь, а не браться из самих существ.

- Названия индексов.

Стандартные имена индексов в цикле- i,j и это нормально.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < map.getHeight(); i++) {
  for (int j = 0; j < map.getWidth(); j++) {
    Coordinate coordinate = new Coordinate(j, i);
    //...
  }
}

//ЛУЧШЕ:
for (int y = 0; y < map.getHeight(); y++) {
  for (int x = 0; x < map.getWidth(); x++) {
    Coordinate coordinate = new Coordinate(x, y);
    //...
  }
}
```

**15. class Simulation**

- Нарушение SRP.

Если класс использует данные из конфига, то он не должен сам его генерировать, а должен принимать его в конструктор  
```java
public class Simulation {
  SimulationConfig config = ConfigLoader.loadConfig();
  //...
}
```

- Нарушение DI. 

Класс не должен сам себя конструировать.  
В данном случае класс конструирует сам себя тем, что сам создает карту, а не принимает ее в конструктор
Таким образом нельзя без изменения кода в классе `Simulation` создать несколько игровых конфигураций с разными картами.

*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Доделать паузу/пуск во время работы программы.

К коду замечаний меньше, чем обычно.

Эталонная версия Симуляции с объяснениями есть у Сергея в расширенных материалах.

n.174(375)  
#ревью #симуляция 