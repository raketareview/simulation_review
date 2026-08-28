https://github.com/veprintsev-ivan/Simulation  
[Ivan Veprintsev]

Пример хорошей реализации проекта "Симуляция".

## ХОРОШО

+ 👍 Есть пауза/пуск во время работы
+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Несколько алгоритмов поиска: BFS, A*
+ 👍 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

+ 👍 Хорошо, что аббревиатура написана по обычным правилам ("Bfs", а не "BFS") - так приятнее читается 
```java
class BfsPathFinder 
```

- Не используй двусмысленных названий.

"Закрытое травоядное"? "Ближайшее травоядное"?  
Непонятно, что конкретно имелось в виду. Звучит, как команда "закрыть травоядное"
```java
Herbivore closeHerbivore
```
Не используй слова в их редких значениях.  
Не все коллеги будут знать, что у слова "close" (закрыть) есть второе значение: "близко".  
Если нужно передать смысл "близко, рядом", используй более однозначное слово, напр. "near". 

- Избыточно
```java
public class WorldMapCreator {
  public WorldMap createWorldMap() {...}
}

//ПРАВИЛЬНО:
public class WorldMapCreator {
  public WorldMap create() {...}
}
```

- Точнее передавай намерения.

Этот метод получает от юзера команды старт/стоп/etc
```java
public class SimulationManager {
  //...
  private char getUserAnswer(String title, String error, List<Character> buttons) {...}
}

//ЛУЧШЕ:
public class SimulationManager {
  //...
  private char inputCommand(String title, String error, List<Character> commands) {...}
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*


**2. record Position**

+ 👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально
```java
public record Position(int row, int column) {
}
```

**3. class WorldMap**

+ 👍 При БОЛЬШИНСТВЕ операций с участием координаты, предварительно выполняется валидация, это хорошо.  
Если координата находится вне пределов карты, бросается исключение- это хорошо 
```java
public Entity getEntity(Position position) {
  validate(position);  <-- бросает исключение, если координата за пределами карты 
  return entities.get(position);
}
```

- Тут тоже нужна валидация координаты
```java
public boolean isEmptyPosition(Position position) {
  return !entities.containsKey(position);
}
```
Иначе на вопрос "Свободна ли координата (+100500, -100500)" метод ответит что да, свободна.  
Тогда как правильный ответ- такой координаты в карте нет вообще.

- Никогда не возвращай null
```java
public Entity getEntity(Position position) {
  validate(position);
  return entities.get(position);  <-- вернет null если position нет в entities
}

```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

+ 👍 С точки зрения SOLID, это превосходный класс. В нём нет ничего лишнего и есть всё, что надо.

**4. abstract class PathFinder**

+ 👍 Абстрактный класс поиска пути- это хорошо. Теперь можно делать разные реализации поиска: BFS, A* и т.д.

+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться поиском
```java
public abstract List<Position> find(WorldMap worldMap, Position start, Class<? extends Entity> target);
```

**5. class Node**

+ 👍 Хороший класс. Его использование делает алгоритм поиска проще и понятнее.

- Всегда явно указывай область видимости полей. 

Здесь можно всем полям поставить просто `public`- нода выполняет функцию простого контейнера данных. 

- Вместо одной универсальной ноды нужно несколько специализированных.

Эта нода содержит стоимость хода:
```java
public class Node {
  Position position;  <-- Общее для BFS и A*
  Node previous;      <-- Общее для BFS и A*
  int costFromStart;  <-- Только для  A*
  int heuristicCost;  <-- Только для  A*
  int totalCost;      <-- Только для  A*
  //...
}
```

Сейчас эту универсальную ноду используют два класса поиска: BFS и A*.  

Для A* такая нода в самый раз.  
Но для BFS эта нода избыточна- она содержит поля, которые не нужны алгоритму BFS.

Для алгоритма BFS нужно использовать более простую ноду, которая содержит только те поля, которые нужны алгоритму BFS.  
Поэтому правильнее сделать отдельные ноды, например так:
```java
public class Node {
  //Общие поля для BFS и A*
}

public class CostNode extends Node{
  //Дополнительные поля для алгоритмов со стоимостью хода, напр. для A*
}
```

**6. Отдельные классы-наследники PathFinder для алгоритмов BFS и AStar**

+ 👍 В проекте есть классы с двумя видами поиска: BFS и A*, это хорошо.

Оба класса являются наследниками АК `PathFinder`, имеют одинаковую сигнатуру метода поиска и поэтому оба класса взаимозаменяемы для использования через полиморфизм
```java
public class BfsPathFinder extends PathFinder {
  @Override
  public List<Position> find(WorldMap worldMap, Position start, Class<? extends Entity> target) {...}
}

public class AStarPathFinder extends PathFinder {
  @Override
  public List<Position> find(WorldMap worldMap, Position start, Class<? extends Entity> target) {...}
}
```

Особых замечаний к этим классам нет.  
За исключением того, что  `BfsPathFinder` не должен юзать избыточную ноду, которая содержит стоимость.  
Он должен юзать более простую ноду без лишних полей, которые не нужны алгоритму BFS.

**7. abstract class Entity и его простые наследники Tree/Rock/Grass**

+ 👍 Идеально
```java
public abstract class Entity {
}

public class Tree extends Entity {
}
```

**8. abstract class Creature extends Entity и его классы-наследники**

+ 👍 Особых замечаний нет.

**9. interface Action**

+ 👍 Идеально
```java
public interface Action {
  void execute(WorldMap worldMap);
}
```

**10. class MoveAction implements Action**

- Нарушение SRP.

Мувер не должен самостоятельно решать, каким образом будет выбираться алгоритм Поиска пути.  
Он должен принять это в конструктор
```java
public class MoveAction implements Action {
  @Override
  public void execute(WorldMap worldMap) {
    PathFinder pathFinder = PathFinderFactory.create(worldMap);
    //...
  }
}

//ПРАВИЛЬНО:
public class MoveAction implements Action {
  private final PathFinder pathFinder;

  public MoveAction(PathFinder pathFinder) {...}

  //...
}
```

Эта зависимость должна инжектиться с самого верха, в данном случае примерно так:
```java
public class SimulationApp {
  public static void main(String[] args) {
    WorldMapCreator creator = new WorldMapCreator();
    WorldMap worldMap = creator.createWorldMap();
    PathFinder pathFinder = PathFinderFactory.create(worldMap);

    Simulation simulation = new Simulation(worldMap, pathFinder);
    //...
  }
}
```

**11. Остальные экшены-наследники Action**

+ 👍 Всё ок.

**12. Диалоги**

😎🍻😎

**13. Фабрики: PathFinderFactory и EntityFactory**

+ 👍 Всё ок.

**14. class WorldMapConsoleRenderer**

+ 👍 Спрайты существ хранятся здесь, а не берутся из самих существ, это хорошо.

- Рендерер должен просто распечатать карту.

Рендерер не должен учитывать в своей работе проблемы синхранизации с потоками 
```java
public void render(WorldMap worldMap) {
  synchronized (...) {...}
}
```
Алгоритм работы рендерера должен быть одинаковым как для однопоточной программы, так и для многопоточной.

Если в проекте есть проблема взаимодействия потоков и рендерера, то эта проблема должна решаться не в самом рендерере.  
А в тех классах, которые используют рендерер в потоках.

+ 👍 В остальном всё ок.

**15. ConsoleLock.java**

- В одном файле `*.java` должен быть только один Java class. 

В этом файле их два:
```java
public class ConsoleLock {
  public static final Lock LOCK = new Lock();
}

class Lock {
}
```

- Не используй глобальных переменных в ООП языках.

Если в программе разные классы в своей работе должны обращаться к одному и тому же экземпляру класса `Lock`,
то этот экземпляр нужно передавать через использование `Dependency Injection`.  
Например, путем передачи его в конструктор всех заинтересованных классов.

Сейчас вместо этого заинтересованные классы `Simulation` и `WorldMapConsoleRenderer` фактически обращаются к `ConsoleLock.LOCK` как к ГЛОБАЛЬНОЙ ПЕРЕМЕННОЙ.  
И это похоже на костыль.

Концепция ООП языка Java среди прочего состоит в отказе от использования глобальных переменных как системной практики.  
Использование глобальных переменных это практика процедурных языков программирования, например, C и Pascal.  
И хотя разными уловками можно ввести в свою Java программу глобальные переменные, лучше этого не делать.

Но в этом случае, проблема решается легко. 

Достаточно убрать в классе `WorldMapConsoleRenderer` использование `ConsoleLock.LOCK`.  
Тогда в `Simulation` можно будет напрямую через `new` создать объект `Lock` и использовать его внутри себя как свою внутреннюю переменную. 

**16. class Simulation**

- Так же, как в `WorldMapConsoleRenderer`, в этом классе не должно быть никаких штук, связанных с проблемой синхронизации потоков
```java
public void nextTurn() {
  synchronized (...) {...}
}
```

- Лишний метод, в котором нет необходимости
```java
public void resetPauseSimulation()
```

Для необходимого алгоритма работы этого класса достаточно сделать только указанные в ТЗ публичные методы:
```java
nextTurn() - просимулировать и отрендерить один ход
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
pauseSimulation() - приостановить бесконечный цикл симуляции и рендеринга
```

Сбрасывать флаг паузы можно тут
```java
public void startSimulation() {
  isPaused = false;
  //...
}
```

Потому что согласно ТЗ, этот метод и должен запускать бесконечную симуляцию:
```java
startSimulation() - запустить бесконечный цикл симуляции и рендеринга
```

+ 👍 В остальном всё ок.

**17. class SimulationManager**

+ 👍 Класс, который обеспечивает многопоточность для реализации старт/стоп во время работы программы.

- Текст в Exception всегда должен быть на английском языке.
```java
throw new IllegalArgumentException("Данная команда не поддерживается.");  <-- Неправильно: сообщение в исключении не на английском языке
```
Исключение это не просто телеграмма, которая летит сквозь слои.

У exception особое назначение- если исключение вылетит и не будет перехвачено внутри программы, то аварийно прекратит выполнение программы.  
Тогда на экране будет распечатано сообщение эксепшена, и это сообщение должно быть понятно сисадмину в любой точке планеты.  
А значит, сообщение должно быть на английском.

Интерпретация исключения и перевод его на локальный язык должны происходить там, где это соответствует архитектуре программы.  
Или не происходить вовсе, если исключение не планируется перехватывать.

+ 👍 В остальном всё ок.

**18. class SimulationApp**

Содержит точку входа main.

+ 👍 Только создает и запускает ????, это хорошо.  
*Мартин, "ЧК", гл.11, "Отделение конструирования системы от ее использования"*

## ВЫВОД

Получилось хорошо, молодец!  
👏😎 👏😎 👏😎

n.166(352)  
#ревью #симуляция 