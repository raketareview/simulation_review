https://github.com/Eug4n4/simulation  
[Евгений]

В целом, норм.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

1. Слишком большая скорость совершения ходов. За полторы-две секунды произошло 559 распечаток ходов.  
Ход нужно совершать раз в 0.5-1сек.

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 👍 Реализовано пауза/пуск во время работы
+ 👍 Меню для создания карт разного размера

## ЗАМЕЧАНИЯ

**1. Нейминг**

+ 👍 В основном, всё ок.

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*

**2. record Coordinate(int row, int column)**

👍 Нет ничего лишнего, это хорошо. Record для координаты- идеально.

**3. class WorldMap**

+ 👍 Геттеры не возвращают null, они возвращают Optional, это хорошо.

- Нарушение SRP, методы чужих ответственностей. 

Карта должна только хранить существа и обеспечить базовые операции с ними: вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей: вернуть случайную пустую координату, вернуть список соседних координат, посчитать количество существ нужного типа.

Наверное, для проекта в целом полезно иметь метод, который считает существ нужного типа. 
Но этот процесс не имеет никакого отношения к единой ответственности карты- хранению существ в себе.  
Для организации хранения/удаления/выдачи существ, карте не нужно считать траву и зайцев.  

Методы чужих ответственностей должны находиться в тех классах, в интересах которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 

Если координата некорректна(находится вне пределов карты), нужно бросать исключение.
Сейчас, если у карты спросить, свободна ли ячейка с координатой (+100500, -100500), то карта скажет, что свободна
```java
public boolean isEmptyCell(Coordinate coordinate) {
  return !occupiedCells.containsKey(coordinate);
}
```
А правильный ответ- ячейки с такой координатой в карте нет вообще.

Ближайшая аналогия- стандартные хранилища типа List и массива. 
При попытке обратитья к ним по несуществующему индексу, бросается исключение.

+ 👍 Хороший метод
```java
public List<Entity> getOfType(Predicate<Entity> entityType)
```

+ 👍 В целом, этот класс нарушает SRP в минимальной степени.

**4. class Pathfinder**


+ 👍 Сигнатура метода поиска ок. Сразу понятно, как пользоваться классом
```java
public List<Coordinate> findFood(Coordinate start, Class<? extends Eatable> foodType)
```

+ 👍 Хороший внутренний класс. Его использование облегчает чтение алгоритма
```java
private static class TreeNode
```

**5. Диалоги**

👍 Для ввода данных от юзера, применяются диалоги
```java
public interface Dialog<T>
public class ScannerIntegerConsoleDialog implements Dialog<Integer>
```

Подробнее про них тут:  
[Диалоги](https://github.com/raketareview/simulation_review/blob/master/content/rev-sim076-ivan-LlqWst.md)  п."Архитектура"  
[Стрим](https://t.me/zhukovsd_it_chat/53243/176204) начиная с 6:28 

**6. abstract class Entity и его простые(неходячие) наследники**

👍 Идеально
```java
public abstract class Entity {
}

public class Palm extends Entity {
}
```

**7. public class Creature extends Entity и его наследники**

👍 Норм.

**8. interface Action**

👍 Идеально

**9. class MoveEntityAction implements Action**

👍 Идеально

**10. class SpawnEntityAction implements Action**

👍 Норм, но идея сетить количество создаваемых существ мне кажется дискуссионной.  
Я не вижу для этого необходимости, достаточно количество существ передавать в конструктор 
```java
public void setCounter(EntityCounter counter)
```

**11. enum Sprite**

Неправильное использование енама.  
Енам задуман для хранения констант и так должен использоваться в программе.  
Енам не должен содержать сложную логику.  

Здесь этот енам не используют как хранилище констант- клиент не берет из него константы напрямую, а запрашивает через метод со сложным поведением
```java
public enum Sprite {
  CELL(11035),
  //...
  PALM(0x1F334);
  private final int codePoint;

  Sprite(int codePoint) {...}

  public static Sprite getSpriteFromEntity(Entity entity) {
    return switch (entity) {
      case Herbivore herbivore -> HERBIVORE;
      //... 
      case null, default -> CELL;
    };
  }

  public int getCodePoint() {
    return codePoint;
  }
}
```

Фактически, клиенту не нужны константы этого енама как таковые.  
Ему нужно получить спрайт существа.  
По функционалу, это фабрика спрайтов. Лучше будет примерно так:
```java
public class SpriteFactory{
  //...
  public String get(Entity entity); 
}
```

Или так:
```java
public class SpriteFactory{
  //...
  public String getEntitySprite(Entity entity); 
  public String getGroundSprite(); 
}
```

**12. class WorldMapRenderer**

- Это очень трудно читается
```java
entityToPrint.ifPresentOrElse(entity ->
  System.out.print(Character.toString(Sprite.getSpriteFromEntity(entity).getCodePoint())),
 () -> System.out.print(Character.toString(Sprite.CELL.getCodePoint()))
);

//ПРАВИЛЬНО:
if(entityToPrint.isEmpty()) {
  int codePoint = Sprite.CELL.getCodePoint();
  String sprite = Character.toString(codePoint);
  System.out.print(Sprite.CELL.getCodePoint());
} else {
  Entity entity = entityToPrint.get();
  int codePoint = Sprite.getSpriteFromEntity(entity).getCodePoint();
  String sprite = Character.toString(codePoint);
  System.out.print(sprite);
}
```
Да, так длиннее. Зато понятно, что происходит.

При использовании фабрики, будет еще понятнее:
```java
if(entityToPrint.isEmpty()) {
  System.out.print(GROUND_SPRITE);
} else {
  Entity entity = entityToPrint.get();
  String sprite = spriteFactory.get(entity)
  System.out.print(sprite);
}
```

- Стандартные имена индексов в цикле- i,j и это правильно.  
Но иногда лучше использовать более подходящие к случаю имена
```java
for (int i = 0; i < worldMap.getHeight(); i++) {
  for (int j = 0; j < worldMap.getWidth(); j++) {
     Optional<Entity> entityToPrint = worldMap.getEntityFromCell(new Coordinate(i, j));
  }
}

//ЛУЧШЕ:
for (int row = 0; row < worldMap.getHeight(); row++) {
  for (int column = 0; column < worldMap.getWidth(); column++) {
     Coordinate coordinate = new Coordinate(row, column); 
     Optional<Entity> entityToPrint = worldMap.getEntityFromCell(coordinate);
  }
}
```

**13. class Simulation**

+ 👍 Принимает в конструктор достаточное количество зависимостей, в том числе списки действий.  
Это позволяет делать майны с разными игровыми конфигурациями, не меняя код в классе `Simulation`
```java
public Simulation(WorldMapRenderer renderer, List<Action> initActions, List<Action> turnActions) 
```


- Нарушение паттерна GRASP "Creator"(Создатель)

Здесь класс принимает в конструктор зависимость, которую должен создавать самостоятельно:
```java
public Simulation(List<Action> initialActions, ... , SimulationEndCondition endCondition, ...) {...}

//ПРАВИЛЬНО:
public Simulation(List<Action> initialActions, ...) {
  //...
  this.endCondition = new SimulationEndCondition();
}
```
Creator гласит, что создавать объект должен тот, кто его использует. 

Рассмотрим, что это значит.

В данном случае `Simulation` принимает в конструктор `initActions`- это правильно.  
Потому что список экшенов может быть разным. 

Но принимать в конструктор `SimulationEndCondition`- уже неправильно.  
Потому что `SimulationEndCondition` это конкретный класс, а не интерфейс и этот класс не параметризируется. 

Экземпляр `SimulationEndCondition` всегда одинаковый.  
Поэтому, согласно паттерна Creator, объект `SimulationEndCondition` *здесь* должен создавать сам класс Simulation.

Технически, можно создать наследника `SimulationEndCondition`, переопределить в нем методы и передавать в конструктор Simulation.  
Но если программист хочет использовать в своем классе разные рендереры через полиморфизм, то он должен обозначить свои намерения более явно.  
Например, передавать в конструктор интерфейс или абстрактный класс.


- Слишком большие условия выноси во вспомогательные методы, которые своим названием будут объяснять, что проиходит
```java
if ((!isRunning && (option == OPTION_START_ENDLESS_SIMULATION || option == OPTION_SIMULATE_SINGLE_TURN)) || option == OPTION_QUIT) {...}

//ПРАВИЛЬНО:
if(isНазваниеКотороеВсеОбъясняет(option)) {...}
```

**14. class Main**, содержит точку входа main

👍 Только создает и запускает Simulation, это хорошо.

## ВЫВОД

Существенных ошибок нет. В целом, норм.

n.131(286)  
#ревью #симуляция 