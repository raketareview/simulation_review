https://github.com/Mihail233/Simulation  
[михаил]

[Ревью сделано в рамках учебной подписки](https://zhukovsd.it/services/student-subscription/).

Есть много над чем поработать.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализованы пауза/пуск во время работы
+ 👍 Возможность конфигурирования карты через меню
+ 🚀 Прикольная интерпритация проекта: кладоискатели ищут монеты, а за ними гоняются призраки 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Если в проекте есть класс `WorldMap`, то все переменные с именем, включающим это название, должны быть экземплярами этого класса.
Если разные концепции называются одним и тем же именем, это приводит к путанице
```java
public class WorldMap {
  private final Cell originWorldMap = new Cell(0, 0);  <-- Это не объект типа WorldMap
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private final Cell какаятоCell = new Cell(0, 0); 
  //...
}
```

- Старайся использовать стандартные имена.  
Для обозначения геометрических размеров обычно используется пара width/height
```java
public class WorldMap {
  private int maxWidthField;
  private int maxLengthField;
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  private int width;
  private int height;
  //...
}
```

- Формулируй точнее
```java
abstract public class Creature extends Entity {
  public void initDeath(Cell location, WorldMap worldMap) {
    //убивает существо
  }
}

//ПРАВИЛЬНО:
abstract public class Creature extends Entity {
  public void kill(Cell location, WorldMap worldMap) {...}
}
```

- "Рендер" это процесс, а "рендерер" это тот, кто выполняет этот процесс.  
Это не рендерер поля(тут нет класса Field), это рендерер игровой карты
```java
class RenderField

//ПРАВИЛЬНО:
class GameMapRenderer
```
Порядок названий: КонтекстРоль.  
Пример из стандартных библиотек: InputStreamReader, DefaultTreeCellRenderer.

- Рендерер должен рендерить
```java
public class RenderField {
  public void displayWorldMap(WorldMap worldMap) {...}
}

//ПРАВИЛЬНО:
public class GameMapRenderer {
  public void render(WorldMap worldMap) {...}
}
```

- Выбери что-то одно- или "has" или "contain", эти слова означают одно и то же
```java
public boolean hasContainInMainMap(WorldMap worldMap, Cell location)
```

- Почему "MainMap", почему не "GameMap"? Что означет здесь "Main"?
```java
public boolean hasContainInMainMap(...)
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. Используй классы через их интерфейсы**
```java
HashMap<Cell, Entity> entities = new HashMap<>();
ArrayList<Cell> cells = new ArrayList<>();
ArrayList<Cell> getCellsOfCertainType(Class<? extends Entity> clazz) {...}

//ПРАВИЛЬНО:
Map<Cell, Entity> entities = new HashMap<>();
List<Cell> cells = new ArrayList<>();
List<Cell> getCellsOfCertainType(Class<? extends Entity> clazz) {...}
```
Общее правило: ArrayList нужно использовать через List, HashMap- через Map и т.д.  
Это позволяет пользоваться преимуществами полиморфизма.

Да, бывают ситуации, когда, например, с LinkedList нужно работать именно как с LinkedList, а не с List. 
Но это уже нюансы.  
*"Java. Эффективное программирование", изд.3, гл.9.8*
```java
"Если вы выработаете привычку использовать в качестве типов интерфейсы, ваша программа будет гораздо более гибкой" - Блох.
```

**3. В проекте системно нарушается инкапсуляция**

Во всех классах все методы публичные. 
Это делает классы уязвимыми для повреждения их внутреннего состояния через неправильное использование клиентами тех методов,
которые не прредназначены для пользования клиентами.  

Публичными должны быть только те методы, которые предназначены для пользования клиентами.  
Доступ protected должен быть у тех методов, которые предназначены только для пользования классами-потомками.  
Все остальные методы должны быть private. 

**4. Форматированный вывод**

- Не используй форматированный вывод там, где нужно просто поставить текст спереди или сзади
```java
System.out.printf("Существо умерло %s", location);

//ПРАВИЛЬНО:
System.out.print("Существо умерло " + location);
```
+ 👍 Вот тут норм- текст вставляется не сбоку, а внутрь шаблона
```java
System.out.printf("Ghost %s ", location);
```

**5. Вводи поясняющие переменные или вспомогательные методы, делай программу более простой и понятной**
```java
if (worldMap.getCellsOfCertainType(Coin.class).size() < remainingEntities) {...}

//ПРАВИЛЬНО:
if (isНазваниеВсеОбъясняет(worldMap, remainingEntities)) {...}

private boolean isНазваниеВсеОбъясняет(WorldMap worldMap, int remainingEntities) {
  return worldMap.getCellsOfCertainType(Coin.class).size() < remainingEntities;
}
```

**6. Распределение по слоям**

Сейчас в проекте есть три пакета-слоя:  
🔹 actions  
🔹 models  
🔹 service

- Пакет actions.  
Классы поиска пути(или выполняющие роль таковых) не являются экшенами и не должны тут находиться.

- Пакет models.

От пакета с таким названием мы ожидаем, что в нем будут находиться *все* модели проекта.  
Но в этом пакете не все модели, а только entity, а это не одно и то же.  

С точки зрения ООП, моделями так же являются Cell, Node etc.  
И WorldMap, когда он будет переделан и избавлен от представления.

- Пакет services.

С точки зрения ООП, здесь ни один класс не является сервисом:
```java
Cell - модель  
RenderField - представление
Simulation - контроллер
WorldMap - гибрид модели и представления
Main - точка входа
```

В ООП сервисы это классы без состояния, но с поведением, которое выполняет бизнес логику и не использует представление.  
Короче, сервисы что-то делают с моделями, но ничего никуда не печатают. При этом сами никаких полей не хранят.

Если не используешь архитектуру, которая содержит такие понятия, как "service", "model" и "controller", 
то лучше не используй в своей программе эти термины.  
Такие термины используются в архитектуре MVC(S). Например, в Spring.

**7. class Cell implements Cloneable**

+ 👍 Нет ничего лишнего, это хорошо. Если тут нужен метод clone(), то пусть будет.

- Неясные намерения.

Непонятно, почему одно поле final, а другое нет.  
Почему у одного поля есть сеттер, а у другого нет.  

Самый лучший выбор для такого простого класса- сделать его immutable
```java
public class Cell implements Cloneable {
  private int x;  <-- Почему не final?
  private final int y;  <-- Почему final?

  public void setX(int x) {  <-- Сеттер только у поля "x"
    this.x = x;
  }
  //...
}

//ЛУЧШЕ:
public class Cell implements Cloneable {
  private final int x;
  private final int y;
  //...
}
```

Еще лучше- сделать этот класс Record.  
Record'ы по умолчанию умеют правильно делать `hashCode()`, `equals()` и `toString()`.  
Про возможности рекордов почитай [тут](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/lang/Record.html).

**8. class WorldMap**

- Нарушение SRP. Божественный класс, который делает разное.

Карта должна только хранить существа и обеспечить базовые операции с ними:  
Вставить, выдать одно существо и список всех хранимых существ, удалить.  
И методы, которые напрямую не управляют размещением существ, но необходимы для этого функционала:  
Сказать ширину/высоту карты и т.д.

Если какой-то метод не нужен для обеспечения хранения существ в карте, 
значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.  

Здесь части чужих ответственностей:  
🔸 Хранит координаты смещения  
🔸 Хранит тексты  
🔸 Ведет диалоги с юзером, принимает от него данные, конфигурирует сам себя через диалог  
🔸 Вернуть список клеток определенного вида  
🔸 Вернуть список соседних клеток  
🔸 Вернуть рейтинг спавна

Для проекта в целом полезно иметь метод, который находит соседние клетки.  
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно искать соседние клетки.

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.  
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- Нарушение инкапсуляции. 

Публичными должны быть только те методы, которые предназначены для использования клиентом.  
Вспомогательные методы, которые используются только в самом классе или его потомках, должны быть private/protect
```java
public WorldMap() {
  initWorldMap();
}

public void initWorldMap() {...}

//ПРАВИЛЬНО:
public WorldMap() {
  initWorldMap();
}

private void initWorldMap() {...}
```
Если клиент будет иметь доступ к тем методам, которые не предназначены для использования клиентами, 
то клиент сможет повредить внутреннее состояние объекта.  
*Вайсфельд "Объектно-ориентированный подход", гл.5, "Минимальный открытый интерфейс"*

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель(а это модель) не должна ничего печатать в консоль.  
```java
public void initWorldMap() {
  System.out.println("Введите начальные параметры");
  //...
}
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах нужно будет менять код в этом классе, чтобы вывод осуществлялся по правилам этой среды, то есть, это лишняя причина для изменения класса.
В других средах(напр. Андроид) эту модель нельзя будет использовать- она там просто не скомпилируется. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Подробнее необходимость отделения модели от представления обоснована в архитектуре MVC(model-view-controller).  
Но необходимость отделения модели от представления появляется не на уровне MVC. Оно появляется уже на уровне SOLID.  
Просто в MVC это требование более явно формализировано.

Модель, которая зависит от представления, перестает быть моделью.

- Карта не должна сама себя конструировать.

Сейчас карта сама себя конструирует через диалог с юзером "Введите начальные параметры"
```java
public WorldMap() {  
  initWorldMap();
}

public void initWorldMap() {
  System.out.println("Введите начальные параметры");
  System.out.println("Введите длину поля");
  setMaxLengthField();
  //...
}
```
Из-за этого карта становится неуниверсальной.  
И не получится, например, сделать карту с заранее заданными размерами- в любом случае их придется устванавливать вручную через диалог.  
Объект не должен сам себя конструировать.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

- Карта должна принимать свои размеры в конструктор. Вот так:
```java
class WorldMap {
  //...
  public WorldMap(int width, int height) {...}
}
```

Должна быть возможность создавать карту без диалога с юзером, просто вызвав ее конструктор:
```java
gameMap gameMap = new gameMap(13, 17);
```

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```java
public HashMap<Cell, Entity> getEntities() {
  return entities;
}
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```java
Карта карта = new Карта();
<заселить карту существами>
карта.getEntities().clear(); //геноцид- удаление из карты всех существ, минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6*

Тут есть два варианта, как сделать правильно.  
Либо вернуть не оригинал мапы, а ее копию: 
```java
public Map<Coordinates, Entity> getEntities() {
  return new HashMap(entities);
}
```

Либо возвращать существ персонально:
```java
public Entity get(Coordinates coordinates)
public Coordinates getCoordinates(Entity entity)
```
Лично мне больше нравится второй вариант, когда существ возвращают персонально. 

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность.  
```java
public void setEntity(Cell cell, Entity entity) {
  this.getEntities().put(cell, entity);
}
```

Сейчас в карту можно вставить существо на координату, выходящую за размер карты
```java
Coordinate coordinates = new Coordinates(+100500, -100500);
Карта карта = new Карта(10, 10);
карта.setEntity(coordinates, new Заяц());
```

Если координата некорректна(находится вне пределов карты), нужно бросать исключение:
```java
public void setEntity(Cell cell, Entity entity) {
  validate(cell);  <-- Если кордината вне пределов карты, бросает исключение
  this.getEntities().put(cell, entity);
}
```

Ближайшая аналогия- стандартные хранилища типа List и массива.  
При попытке обратиться к ним по несуществующему индексу, бросается исключение.

- Если класс Cell сделать immutable, как я предлагал выше, то его не придется клонировать при глубоком клонировании мапы
```java
public Map<Cell, Entity> getCopyEntities()  {<-- Глубокое клонирование мапы
  //...
  cell = (Cell) pieceOfWorldMap.getKey().clone();
  //...
}
```
Потому что состояние иммутабельного объекта невозможно изменить.

- Положительные условия читаются лучше отрицательных.  
Обычно одно можно легко развернуть в другое
```java
public boolean isOffTheMap(Cell cell) <-- Определяет, что координата находится за пределами карты

//ПРАВИЛЬНО:
public boolean isInside(Cell cell) <-- Определяет, что координата находится ВНУТРИ карты
```
*"ЧК", гл.17, G29 "Избегайте отрицательных условий"*

- Магические цифры выноси в константы
```java
spawnRate = (0.125f * scanner.nextFloat() + 0.875f);
```

- Такой сложный дженерик не дает ничего дополнительно хорошего по сравнию с обычным дженериком.  
Он только делает код более громоздким
```java
public Optional<? extends Entity> getEntity(Cell cell)

//ПРАВИЛЬНО:
public Optional<Entity> getEntity(Cell cell)
```

В остальных местах у тебя же нормальные дженерики:
```java
private final HashMap<Cell, Entity> entities
```

**9. Пакет path**

В этом пакете 4 класса, но я так и не смог понять, что 3 из них делают.

Единственный тут понятный мне класс- Node.  
Это полезный вспомогательный класс для процесса прокладывания пути.

Остальные классы мне непонятны.  
В ООП, любой класс должен представлять собой какую-то внятную сущность.  
Здесь я не могу сказать, какую сущность из себя представляет класс Path.

Класс Path соответствует своему названию и содержит путь? Нет.  
Путь это последовательность координат от точки старта до точки финиша.  
Этот класс не содержит путь, он содержит много какихто-то публичных методов.  

Может быть имеется ввиду, что этот класс не путь(Path) а поиск пути(PathFinder)?  
Но я не вижу здесь метода, который ищет путь.  
То есть, возвращает последовательность координат от точки старта до финиша.

В этом классе много, ОЧЕНЬ много разных публичных методов.  
Но для чего они существуют и как ими пользоваться, клиент не поймет.  

Возможно, стало бы немного понятнее, если бы в этом классе публичными были не все методы.  
А только те, которые по задумке автора предназначены для использования клиентами.

Я думаю, этот класс называется "Путь" не потому, что являет собой сущность Путь или сущность ПоискПути.  
Это контейнер с методами, которые как-то относятся к проблематике пути.  
То есть, это не объект в понимании ООП, это модуль в стиле процедурного программирования. 

**10. Поиск пути, которого нет**

В ТЗ указано, что должен быть класс поиска пути
```java
Поиск пути
Следует вынести это в отдельный класс.
```
Сейчас такого класса в проекте нет.

Путь это последовательность точек от точки старта до точки финиша.  
Поиск пути должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритма BFS.  
В данном случае- до точки, в которой находится существо нужного класса
```java
public class BfsPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
  //ищет путь на карте от точки start
  //до точки, где находится существо нужного класса(напр. target == Grass.class)
  }
}
```

Или поиск должен искать путь между точками старта и финиша, согласно алгоритма A*
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates from, Coordinates to) {...}
}
```

Так же, поиск пути по алгоритму A* может использовать ту же сигнатуру, что и BFS.  
Тогда поиск пути в нем интерпритируется не как поиск пути между двумя точками, а как поиск цели и прокладывание пути к ней.  
Например: 
```java
public class AstarPathFinder {
  public List<Coordinates> find(Карта карта, Coordinates start, Class<? extends Entity> target) {
    //сначала находит координату цели
    //потом прокладывает путь по алгоритму A* между точкой стартой и точкой цели
  }
}
```
Плюсы- унификация сигнатуры разных алгоритмов поиска для полиморфизма.  
То есть, можно сделать семейство родственных классов поиска, объединенных общим интерфейсом.

**11. abstract public class Entity**

- Неправильноая последовательность в сигнатуре
```java
abstract public class Entity

//ПРАВИЛЬНО:
public abstract class Entity
```

- Эти поля приватные и к ним нет доступа, их невозможно использовать.  
Поэтому их и не используют
```java
abstract public class Entity {
  private int x;  <- Нет доступа к этому полю
  private int y;  <- Нет доступа к этому полю
}
```

В любом случае, если хранить координату в существах, то делать это нужно начиная с уровня Creature.  
Потому что координата нужна только тому существу, которое ходит.

Вот идеальные для этого проекта базовый энтити и его неходячие потомки:
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**12. public class Creature extends Entity**

- Последовательность полей в классе должна соответствовать последовательности аргументов этих полей в конструкторе
```java
abstract public class Creature extends Entity {
  private final int speed;
  private final int interactionDistance;
  private int healthPoints;

  public Creature(int interactionDistance, int speed, int healthPoints) {...}
  //...
}

//ПРАВИЛЬНО:
abstract public class Creature extends Entity {
  private final int speed;
  private final int interactionDistance;
  private int healthPoints;

  public Creature(int speed, int interactionDistance, int healthPoints) {...}
  //...
}
```

- Два публичных метода, которые для клиента выглядят совершенно одинаковыми.  
Непонятно, в каком случае клиент должен использовать один из них, а в каких случаях- другой
```java
public abstract void makeTurn(Cell location, WorldMap worldMap);
public void makeMove(Cell location, Cell nextLocation, WorldMap worldMap) {...}
```
Названия методов не должны быть настолько похожими по смыслу.

- Нарушение SRP, чужая ответственность, зависимость модели от представления.

Модель(а это модель) не должна ничего печатать в консоль
```java
public void initDeath(Cell location, WorldMap worldMap) {
  //...
  System.out.printf("Существо умерло %s", location);
}
```
Вот к чему приводит зависимость модели от представления, карта рисуется в виндовый интерфейс, 
а эгтити [пишет в консоль](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim064-davidtagirov-DavidTagirov.md). 

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack.  
[CallBack луноход](https://t.me/zhukovsd_it_chat/53243/139594)  
[CallBack разрушение](https://t.me/zhukovsd_it_chat/53243/184749)

- Метод убивания должен для убийства получать убиваемое существо.  
А не координату с убиваемым существом
```java
public void initDeath(Cell location, WorldMap worldMap) {
  worldMap.removeEntity(location);
  System.out.printf("Существо умерло %s", location);
}

//ПРАВИЛЬНО ТАК:
public void kill(WorldMap worldMap, Entity entity) {
  worldMap.removeEntity(entity);
  //опционально- дергать иниерфейс callback
}

//ТАК ТОЖЕ ПРАВИЛЬНО:
public void kill(WorldMap worldMap, Entity entity) {
  Cell = worldMap.getCell(entity);
  worldMap.removeEntity(cell);
  //опционально- дергать иниерфейс callback
}
```

- Отсутствие инкапсуляции приводит к тому, что клиент может юзать внутренности классов как хочет.  
Например, Охотника за монетами можно использовать как универсальный убиватор чего угодно, хоть камня:
```java
public static void main(String[] args) {
  WorldMap worldMap = new WorldMap();

  CoinHunter coinHunter = new CoinHunter(null);
  worldMap.setEntity(new Cell(0, 0), coinHunter);

  Cell rockCell = new Cell(2, 2);
  worldMap.setEntity(rockCell, new Rock());

  RenderField renderField = new RenderField();
  renderField.displayWorldMap(worldMap);

  coinHunter.initDeath(rockCell, worldMap);  //дистанционно убить камень используя охотника за монетами в качестве транглюкатора
  System.out.println();

  renderField.displayWorldMap(worldMap);
}
```
Результат:
```java
⬛ ⬛ ⬛ ⬛ ⬛  
⬛ 👤 ◻️  ◻️  ⬛ 
⬛ ◻️  ◻️  ◻️  ⬛ 
⬛ ◻️  ◻️  ️⛰️ ⬛ 
⬛ ⬛ ⬛ ⬛ ⬛  
Собранные монеты 0
Существо умерло x = 2, y = 2
⬛ ⬛ ⬛ ⬛ ⬛  
⬛ 👤 ◻️  ◻️  ⬛ 
⬛ ◻️  ◻️  ◻️  ⬛ 
⬛ ◻️  ◻️  ◻️  ⬛ 
⬛ ⬛ ⬛ ⬛ ⬛  
Собранные монеты 0
```

**13. class CoinHunter extends Creature**

- Не используй статические ИЗМЕНЯЕМЫЕ поля в классах. Это нарушение дизайна ООП
```java
private static int numberOfCoinsCollected = 0;
```
Если нужно решить задачу учёта каких-то действий, то учет должен вести кто-то другой.  

Сейчас это статическое поле ведет учет собранных монет всеми действующими на карте охотниками за монетами
```java
public void incrementNumberOfCoinsCollected() {
  numberOfCoinsCollected++;
}
```
Ту же самую задачу можно решить нормально с архитектурной точки зрения, используя паттерн callback-
каждый раз, поднимая монету, охотник должен дёрнуть за этот интерфейс.

- В таких случаях используй константы, а не поясняющие переменные
```java
public class CoinHunter extends Creature {
  //...
  public CoinHunter(TurnStrategy turnStrategy) {
    int interactionDistance = 1;
    int speed = 2;
    int healthPoints = 1;
    super(interactionDistance, speed, healthPoints);
    this.turnStrategy = turnStrategy;
  }
  //...
}

//ПРАВИЛЬНО:
public class CoinHunter extends Creature {
  private final static int INTERRACTION_DISTANCE = 1;
  //...

  public CoinHunter(TurnStrategy turnStrategy) {
    super(INTERRACTION_DISTANCE, SPEED, HEALTH_POINTS);
    this.turnStrategy = turnStrategy;
  }
  //...
}
```
То же замечание к классу Ghost.  

**14. class Ghost extends Creature**

- Нарушение DRY. Дублирование полей у родственников.

И Призрак и Охотник имеют одинаковое поле
```java 
public class CoinHunter extends Creature {
  private final TurnStrategy turnStrategy;
  //...
}

public class Ghost extends Creature {
  private final TurnStrategy turnStrategy;
  //...
}
```
Нужно избавиться от дублирования и вынести это поле в общего предка.

**15. abstract class Action**

👍 Идеально
```java
public abstract class Action {
  public abstract void execute(WorldMap worldMap);
}
```

**16. class CreaturesMakeTurn extends Action**

- Можно проще
```java
if (Creature.class.isAssignableFrom(entity.getClass())) {
  Creature creature = (Creature) entity;
  creature.makeTurn(location, worldMap);
}

//ПРАВИЛЬНО:
if (entity instanceof Creature creature) {
  creature.makeTurn(location, worldMap);
}
```

- А лучше сделай код в классе более читаемым. Например так:
```java
class CreaturesMakeTurn extends Action {
  //....
  @Override
  public void execute(Карта карта) {
    List<Creature> creatures = getCreatures(карта);
    for(Creature creature: creatures) {
      creature.makeTurn(...);  //даёт пинка
    }
  }

  private static List<Creature> getCreatures(Карта карта) {
    //найти и вернуть все креатуры из карты
  }
}
```

**17. abstract class Spawn extends Action**

- Нарушение концепции экшенов.  

В экшенах должен быть только один публичный метод.  
Чтобы можно было все экшены использовать одинаково через полиморфизм.  
При этом protected и private методов может быть сколько угодно.  
Экшены должны соответствовать паттерну "Command".

Если в экшене кроме `execute(...)` есть другие публичные поля или методы, то это не экшен.
```java
abstract public class Spawn extends Action {
  public abstract void execute(WorldMap worldMap);
  public float getRandomNumber() {...}
  public boolean isPlaceEntity(float spawnRate, float spawnProbability) {...}
}
```
Возможно, кроме `execute(...)`, все остальные методы в этом классе можно сделать protectrd/private и ничего не поломается.  
Но я даже не хочу в этом разбираться- тотальное игнорирование инкапсуляции в проекте делает это утомительным.

Чтобы не повторяться, это замечание относится ко всем остальным экшенам.

**18. SpawnObstacle extends Spawn**

Слишком много всего, разобраться что тут происходит сложно.  
Как-то упрощай код
```java
for (int indexRow = worldMap.getOriginWorldMap().getY(); indexRow < worldMap.getMaxWidthField(); indexRow++) {
  for (int indexColumn = worldMap.getOriginWorldMap().getX(); indexColumn < worldMap.getMaxLengthField(); indexColumn++) {
    if (super.isPlaceEntity(worldMap.getSpawnRate(), SpawnProbability.OBSTACLE.getProbability())) {
      //что-то делается
    }
  }
}

//ПРАВИЛЬНО:
int startRow = worldMap.getOriginWorldMap().getY();
int width = worldMap.getMaxWidthField();

int startColumn = worldMap.getOriginWorldMap().getX();
int height = worldMap.getMaxLengthField();

for (int row = startRow; row < width; row++) {
  for (int column = startColumn; column < height; column++) {
    boolean isНазваниеКотороеВсеОбъясняет = super.isPlaceEntity(worldMap.getSpawnRate(), SpawnProbability.OBSTACLE.getProbability());
    if (isНазваниеКотороеВсеОбъясняет) {
      //что-то делается
    }
  }
}
```

И тогда сразу станет понятно, что здесь что-то не так
```java
for (int indexRow = worldMap.getOriginWorldMap().getY(); indexRow < worldMap.getMaxWidthField(); indexRow++) {...}
```
Потому что "Row's" это строки, а "WidthField" это ширина поля.  
Но строкам соответствует высота(height), а не ширина(width)

**19. Группа экшенов-спавнеров**

Много классов, которые делают одно и то же- помещают в карту существ определенного типа в определенном количестве.  
Нарушение чеклиста ТЗ:
```java
Проблемы и ошибки в коде:
Иерархия классов Action’s: 
Дублирование кода
```

Вместо отдельного класса на каждый вид существ, нужно сделать универсальный класс. 
Он будет принимать в конструктор количество создаваемых существ и способ их создания.  

Способ создания можно передавать через стандартный интерфейс Supplier
```java
public class SpawnAction extends Action {
  private final Supplier<Entity> entitySupplier;
  private final int count;

  public SpawnAction(Supplier<Entity> entitySupplier, int count) {...}

  @Override
  public void execute(Карта карта) {
    for (int i = 0; i < count; i++) {
      //получить из саплаера существо 
      //и поместить его в карту на случайную координату
    }
  }
}
```
Использование:
```java
List<Action> turnActions = List.of(new SpawnAction(() -> new Заяц(), КОЛИЧЕСТВО_ЗАЙЦЕВ),
                                   new SpawnAction(() -> new Волк(), КОЛИЧЕСТВО_ВОЛКОВ),
                                   //...
                                  );
```

**20. Пакет strategy**

Внезапно, в пакете "actions" нашелся пакет "strategy" с классами, которые не являются экшенами.  

Почему эти классы названы стратегией, почему они в пакете с экшенами, 
почему они внутри себя реализуют алгоритм поиска целей вместо реализации этого поиска в классе поиска пути, 
что тут вообще происходит среди ада методов, зависимостей и стримов?  
Код абсолютно нечитаем, к объектно-ориентированному программированию все это не имеет никакого отношения.

Я думаю, это последствия того, что не сделан нормальный класс поиска пути.  
Сделай нормальный класс поиска пути и перераспредели код вокруг использования процесса поиска, тогда что-то прояснится.

**21. class RenderField**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ. Это хорошо.

- Нарушение SRP.

Этот Рендерер должен только печатать карту.  
Печатать информацию про количество собранных монет и прочее должен кто-то другой.

- Много букв, упрощай 
```java
for (int indexRow = worldMap.getOriginWorldMap().getY(); indexRow < worldMap.getMaxWidthField(); indexRow++) {
  printPartOfVerticalBorder();
  for (int indexColumn = worldMap.getOriginWorldMap().getX(); indexColumn < worldMap.getMaxLengthField(); indexColumn++) {
     Optional<? extends Entity> entity = worldMap.getEntity(new Cell(indexRow, indexColumn));
```

- Не соединяй много инструкцй в одну просто потому, что можешь.

Старайся делать код простым.  
Заумный код это не круто. Простой и понятный код, который легко читается- вот, что по-настоящему круто
```java
Optional<? extends Entity> entity = worldMap.getEntity(new Cell(indexRow, indexColumn));
entity.ifPresentOrElse((presentEntity) -> {
  switch (presentEntity) {
    case Ghost ghost -> System.out.print(ghostPicture);
    case CoinHunter CoinHunter -> System.out.print(coinHunterPicture);
    //... 
    default -> throw new IllegalArgumentException("Unexpected value");
  }
  },
  () -> System.out.print(graniteBlockPicture)
);

//ПРАВИЛЬНО:
Cell cell = new Cell(row, column);
Optional<Entity> optional = worldMap.getEntity(cell);

if(optional.ifEmpty()) {
  System.out.print(СПРАЙТ_ЗЕМЛИ);
} else {
  Entity entity = optional.get();
  String sprite = toSprite(entity);
  System.out.print(sprite);
}
//....

private String toSprite(Entity entity) {
  //возвращает спрайт ентити
}
```

**22. class Simulation**

+ 👍 Принимает достаточное количество зависимостей в конструктор.  
В данном случае- карту, в текущей реализации, этого достаточно 
```java
public Simulation(WorldMap worldMap) {
  this.worldMap = worldMap;
}
```

- Не используй базовый класс `Object` непосредственно в виде `Object`.

Если нужен какой-то объект, то подумай какую сущность он из себя представляет и что от него требуется.  
Если от него требуется только "быть самим собой" и что-то манифестировать своим названием, то просто сделай объект без поведения и состояния.  
Все равно он будет наследоваться от Object
```java
private final Object monitor = new Object();

//ПРАВИЛЬНО:
public class Simulation {
  private final Monitor monitor = new Monitor();
  //...

  private static class Monitor {
  }
}
```

- Неправильное использование потоков в проекте.

Внутри класса Simulation не должно быть никакой работы с потоками
```java
public void changeMonitor() {
  try {
    monitor.notify();
    monitor.wait();
  } catch (InterruptedException e) {
    throw new RuntimeException("Вмешательство в работу потока");
  }
}

void initStreamInput() {
  //тут тоже работа с потоками
}

//в других методах тоже работа с потоками
```

Этот класс не должен управлять потоками.  
Наоборот, какие-то потоки должны управлять этим классом и дергать за его публичные методы, которые указаны в ТЗ:
```java
Simulation #
Главный класс приложения, включает в себя:
...
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Это должен быть такой класс, который можно будет не только запустить из потоков.  
Но и запустить без потоков просто в бесконечном цикле и он должен(бесконечно) работать:
```java
public class Main {
  public static void main(String[] args) {
    GameMap gameMap = new GameMap(10, 10);
    //...
    Simulation simulation = new Simulation(gameMap, ...);
    simulation.startSimulation(); //бесконечная симуляция без потоков 
  }
}
```

Для этого класс должен выглядеть примерно так:
```java
public class Simulation {
  //...

  public void nextTurn() {
    //выполняет все turnActions
    //печатает карту
    //печатает сопроводительную информацию: n.хода и прочее
  }

  public void startSimulation() {
    isRunning = true;
    while(isRunning) {
      nextTurn();
      sleep();
    }
  }

  private void sleep() {
    try {
      Thread.sleep(SLEEP_TIME);
    } catch (InterruptedException e) {...}
  }

  public void pauseSimulation() {
    //останавливает бесконечный цикл
  }
}
```

Когда мы добавляем в программу потоки, то поток должен принимать команды от юзера и дергать `Simulation` за ее методы:   
`nextTurn()`, `startSimulation()` и `pauseSimulation()`.


**23. class Main**, содержит точку входа main

+ 👍 Только создает и запускает Симуляцию, это хорошо.

- Во всем проекте ты полностью игнорируешь инкапсуляцию.  
Но здесь решил подрезать публичный доступ там, где он безусловно нужен.  
Мотивация неясна
```java
public class Main {
  static void main(String[] args) {
    //...
  }
}

//ПРАВИЛЬНО:
public class Main {
  public static void main(String[] args) {
    //...
  }
}
```

**Совет:**  
Для тренировки и понимания некоторых ООП концепций, создай несколько майн-классов.  
Один пусть запускает игру в потоках так, как это сейчас- с конфигурированием карты через диалог с юзером.  
Второй майн пусть запускает игру в потоках с картой размерами 10x10 и т.д.
Третий майн пусть запускает игру без потоков в бесконечном цикле с картой размерами 10x10 и т.д.

## ВЫВОД

Видно непонимание того, что такое инкапсуляция и зачем она нужна.  
Проработай этот вопрос. Инкапсуляция это не прихоть, это действительно важно.

ООП сильно поплыло на этапе реализации поиска пути. Проработай тему SOLID, прежде всего SRP.
Почитай про отделение модели от представления.

Я знаю, что в поиске пути у тебя была заложена оригинальная концепция- призраки охотятся за "своими" кладоискателями.  
Но с архитектурной точки зрения нормально воплотить эту идею не получилось.  
Поэтому советую идти от простого к сложному- сделай поиск по ТЗ, но сделай его хорошо.

Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы.  
Посмотреть ролики Немчинского про SOLID- по одному ролику на каждую букву.

n.145(307)  
#ревью #симуляция 