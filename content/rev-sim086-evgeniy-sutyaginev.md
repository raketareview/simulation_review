https://github.com/sutyaginev/simulation  
[Евгений]

В целом, неплохо.

## ХОРОШО

👍 Спрайты существ не хранятся в самих существах

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название должно как можно лучше объяснять суть явления. Из названия "Генератор" мы ничего понять не сможем. На самом деле, класс помещает существ на карту- этот функционал нужно отразить в названии
```
class Generator
```
*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. class Coordinate**

+ 👍 Нет ничего лишнего, это хорошо.

- Всегда явно указывай область видимости. В данном случае, она должна быть private. 
```
final Integer x;
final Integer y;
```

- При прочих равных используй примитивные типы, а не классы обертки
```
final Integer x;
final Integer y;

//ПРАВИЛЬНО:
private final int x;
private final int y;
```
Для использования обертки должна быть причина и выгода, которые дает применение этой обертки.
Тут выгоды никакой нет.

- Класс может быть преобразован в record без потери функционала.

**3. class WorldMap**

- Мапа координата-существо является подробностью внутреннего устройства класса Карта и не должна передаваться карте в конструктор. 
В данном случае актуален паттерн GRASP "Creator"- создает экземпляры класса тот, кто его использует. `WorldMap` должен сам создавать мапу
```
public WorldMap(int width, int height, Map<Coordinate, Entity> entities) {
  this.width = width;
  this.height = height;
  this.entities = entities;
}

//ПРАВИЛЬНО:
public WorldMap(int width, int height) {
  //...
  this.entities = new HashMap();
}
```

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: подсчитать существ определенного класса, вернуть список креатур.

Наверное, для проекта в целом полезно иметь метод, который считает траву. 
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, самой карте не нужно использовать метод, который считает траву, зайцев и т.д.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение SRP. 
Карта должна работать со всеми хранимыми существами одинаково и не работать как-то по-особому с конкретным классом представителей и не должна даже знать по именам наследников Entity
```
List<Creature> getCreatures() 

//ПРАВИЛЬНО ТАК:
public List<Entity> getAll() //и тогда пусть клиент отбирает отсюда креатур

//ИЛИ ТАК:
public List<? extends Entity> getAllBy(Class<? extends Entity> clazz)  //вернуть существ определенного класса 
```

- Никогда не возвращай null
```
private final Map<Coordinate, Entity> entities;

public Entity getEntity(Coordinate coordinate) {
  return entities.get(coordinate);  //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
private final Map<Coordinate, Entity> entities;

public Map<Coordinate, Entity> getEntities() {
  return entities;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например, так
```
Карта карта = new Карта(100, 100);
<заселить карту существами>
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public void addEntity(Entity entity) {
  entities.put(entity.getCoordinate(), entity);
}
```

**4. interface PathFinder**

+ 👍 Интерфейс поиска пути это хорошо. Теперь можно делать разные реализации поиска: BFS, AStar и т.д.

+ 👍 Сигнатура поиска- лучше обычного. Благодаря предикату, существо потенциально может охотиться не на один класс существ, а на несколько.
```
List<Coordinate> findPathToNearest(Coordinate start, Predicate<Entity> targetCondition);
```

**5. BreadthFirstSearch implements PathFinder**

Эти неизменяемые данные нужно вынести из метода и сделать константами. 
Потому что сейчас объект с этими данными(а массив это объект) пересоздается каждый раз при вызове метода
```
public List<Coordinate> findPathToNearest(Coordinate start, Predicate<Entity> targetCondition) {
  int[][] directions = {{1, 0}, {0, 1}, {-1, 0}, {0, -1}};
  //...
}

//ПРАВИЛЬНО:
private final static List<Coordinate> SHIFT_COORDINATES = List.of(new Coordinate(1, 0), new Coordinate(0, 1), ...);

public List<Coordinate> findPathToNearest(Coordinate start, Predicate<Entity> targetCondition) {
  //...
}
```

**6. class Entity**

Содержит координату. Но координата нужна только тому существу, которое ходит. Поэтому entities должны хранить координату только начиная с уровня `Creature`.

**7. abstract class Creature extends Entity**

- Методы не должны возвращать null, а клиенты не должны проверять возвращаемые значения на null
```
Entity entity = worldMap.getEntity(nextStep);

if (entity == null) {
  move(worldMap, nextStep);
} else {
  attack(worldMap, nextStep);
  return;
}

//ПРАВИЛЬНО:
if(!worldMap.isEmptyCell(nextStep)) {
  attack(worldMap, nextStep);
  return;    
}
move(worldMap, nextStep);
```

+ 👍 Определение еды через предикат это хорошо- каждое существо теперь может питаться существами нескольких классов, а не одного
```
protected abstract Predicate<Entity> getTargetPredicate();
```

- Но делать метод, возвращающий предикат вовсе не обязательно. Намного понятнее будет смотреться обычный метод-определитель еды, например, такой
```
protected boolean isFood(Entity entity) {
  return entity instanceof Herbivore || entity instanceof Kolobok;
}
```
И тогда можно будет этоот метод передавать в аргументы поиска пути качестве предиката
```
//ВМЕСТО:
List<Coordinate> pathToTarget = pathFinder.findPathToNearest(getCoordinate(), getTargetPredicate());

//БУДЕТ:
List<Coordinate> pathToTarget = pathFinder.findPathToNearest(getCoordinate(), this::isFood);
```

**8. Наследники Creature**

👍 Все ок.

**9. пакет utility**

Не содержит ни одной утилиты. Гугли, что в java понимается под утилитным классом.

**10. interface Action и его реализации**

👍 Все ок.

**11. class Generator**

Магические цифры: 2, 3, 5, 10, 20, 40. Вводи константы.

**12. class Renderer**

- В switch-case в default нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт.

- Нарушение DRY
```
private String getEntitySprite(Entity entity) {
  switch (entity.getClass().getSimpleName()) {
    case "Tree":
      return "\u2005" + "🌳" + "\u2005"; // 🌵
    case "Grass":
      return "\u2005" + "🌾" + "\u2005"; // 🌿
    //...
  }
}

//ПРАВИЛЬНО:
private String getEntitySprite(Entity entity) {
  String sprite = switch (entity.getClass().getSimpleName()) {
    case "Tree" -> "🌳";
    case "Grass" -> "🌾";
    //...
  }
  return "\u2005" + sprite + "\u2005";
}
```

- Спрайты нужно сделать константами
```
String sprite = switch (entity.getClass().getSimpleName()) {
  case "Tree" -> "🌳";
  //...
}

//ПРАВИЛЬНО:
String sprite = switch (entity.getClass().getSimpleName()) {
  case "Tree" -> TREE;
  //...
}
```

- Кстати, что это за магическая штука- "\u2005"?

**13. class Simulation**

- Класс должен принимать в конструктор не ширину и высоту карты, а сам экземпляр карты
```
public Simulation(int width, int height)

//ПРАВИЛЬНО:
public Simulation(WorldMap worldMap)
```
Хочешь спросить, почему в этом случае не актуален паттерн "Creator"? 
Потому что, если симуляция будет принимать размеры и потом по этим размерам создавать карту, то значит, что процесс создания экземпляра карты размазывается на два класса. 
Один принимает решение, какого размера будет карта, а второй создает карту.

- Старайся не использовать преинкремент. Иначе ты заставляешь вспоминать различия между пре и постинкрементом и можно легко допустить ошибку, перепутав их
```
System.out.println("Turn: " + (++turnCounter));

//ЛУЧШЕ:
turnCounter++;
System.out.println("Turn: " + turnCounter);
```

**14. class Main**, содержит точку входа main

👍 Только создает и запускает Симуляцию, это хорошо.

## ВЫВОД

В целом, неплохо 👍

n.86(194)  
#ревью #симуляция #predicate 