https://github.com/Andr0id0/Simulation  
[Андрей]

Довольно удручающе.

## ХОРОШО

1. Спрайты существ хранятся не в самих существах
2. Координаты существ хранятся не в самих существах(мне так больше нравится)

## ЗАМЕЧАНИЯ

**1. Нейминг**

-Названия пакетов пишутся стилем lower_case
```
Actions

//ПРАВИЛЬНО:
actions
```

-Если в проекте есть класс Map, то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
public class Map {
  public HashMap<Coordinates, Entity> map;
  //...
}

//ПРАВИЛЬНО:
public class Map {
  public HashMap<Coordinates, Entity> entities;
  //...
}
```

-(А)В названии переменных не пиши тип данных, к которым они относится. И вообще не употребляй венгерскую ноттацию. (Б)Название поля должно объяснять, что оно хранит
```
public HashMap<Coordinates, Entity> map;

//ПРАВИЛЬНО:
public HashMap<Coordinates, Entity> entities;
```

-"Выполнить действие", "выполнить действие два" - названия, которые не говорят ни о чем. Трудно придумать менее информативные и более бесполезные названия для методов
```
void performAction()
void performActionTwo() 
```

*Oracle Java code conventions, part."Naming conventions"*
*Мартин, "Чистый код", гл.2*

**2. Нарушение конвенции кода**  
Методы располагаются черт-те как. Например, конструкторы находится внизу класса, под миллионом других методов.

**3. Используй классы через их интерфейсы**
```
public HashMap<Coordinates, Entity> map;

//ПРАВИЛЬНО:
public Map<Coordinates, Entity> map;
```

**4. Создавай вспомогательные методы**, делай программу более простой и понятной
```
if (herbivore.getHealth() < parameters.getHerbivoreHp() - healAfterEat) {...}

//ПРАВИЛЬНО:
if (isНазваниеКотороеВсеОбъясняет(/*args or empty*/)) {...}

private boolean isНазваниеКотороеВсеОбъясняет(/*args or empty*/) {
  return herbivore.getHealth() < parameters.getHerbivoreHp() - healAfterEat;
}
```

-Для комментариев достаточно двух палочек
```
/// если creature может дойти до target за один ход или creature уже возле target
```

**5. Используй форматированный вывод**  
Если нужно печатать текст с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.println("Herbivore x:" + coordinates.x() + " y:" + coordinates.y() + " hp:" + newHp);

//ПРАВИЛЬНО:
System.out.printf("Herbivore x:%d y:%d hp:%d \n", coordinates.x(), coordinates.y(), newHp);
```

**6. При наследовании данные нужно инжектить в конструктор предка**, а не устанавливать вручную
```
public Herbivore(int speed, int health) {
  this.speed = speed;
  this.health = health;
  this.target = herbivoreTarget;
}

//ПРАВИЛЬНО:
public Herbivore(int speed, int health) {
  super(speed, health, herbivoreTarget);
}
```

**7. Избыточно**
```
boolean flag = false;
for (...) {
  if (...) {
    flag = true;
  }
}
return flag;

//ПРАВИЛЬНО:
for (...) {
  if (...) {
    return true;
  }
}
return false;
```

**8. record Coordinates(int x, int y)**

+Для координаты рекорд- идеально.

**9. class Map, карта игры**

-Следовать ТЗ это хорошо. Называть свои пользовательские классы так же, как стандартные классы и интерфейсы- плохо. 
Второе перевешивает, потому что это приводит к постоянной путанице в коде. Класс нужно переименовать.

-Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
public HashMap<Coordinates, Entity> map;
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта(100, 100);
//заселить карту существами
карта.map.clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: "Чистый код", гл.6 

-Размеры карты класс должен получать в конструктор, а не высасывать из класса настроек
```
SimulationParameters parameters = new SimulationParameters();
private final int xSize = parameters.getMapSizeX();
private final int ySize = parameters.getMapSizeY();
```

-Нарушение SRP, божественный класс, методы чужих ответственностей. 
Карта должна только хранить существа и обеспечить базовые операции с ними- вставить, выдать одно существо и список всех хранимых существ, удалить.
Если какой-то метод не нужен для обеспечения этого функционала, значит он не принадлежит к ответственности карты, а принадлежит к чужой ответственности.
Здесь методы чужих ответственностей(порой очень странных): считает количество травы, определяет что по кординате находится креатура, определяет что по координате находится трава или пусто, считывает ентити по координате и сразу его кастует в креатуру и т.д.
Методы чужих ответственностей должны находиться в тех классах, в интересх которых они работают.
Если один и тот же метод используют разные классы, то метод нужно вынести в отдельный класс, например, BoardUtils.

-Никогда не возвращай null
```
public HashMap<Coordinates, Entity> map;

public Entity getEntity(Coordinates coordinates) {
  return map.get(coordinates); //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе

*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*

-При добавлении существа в карту, не проверяет координату на корректность
```
public void putEntity(Coordinates coordinates, Entity entity) {
  map.put(coordinates, entity);
}
```
Сейчас в карту можно вставить существо на координату, выходящую за размер карты
```
Coordinates coordinates = new Coordinates(+100500, -100500);
Карта карта = new Карта(10, 10);
карта.putEntity(coordinates, new Заяц());
```
Перед добавлением существа в карту, нужно проверить координату на корректность.

-При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение.

-При попытке удалить существо по координате, где нет существа, должно бросать исключение.
```
public void deleteEntity(Coordinates coordinates) {
  map.remove(coordinates);
}
```

-Метод совершения хода в карте- нарушение SRP.
Теперь можно вызвать из карты метод хода и телепортировать зайца из одного края в другой минуя все правила игровой логистики
```
Карта карта = new Карта(100, 100);
Заяц заяц = new Заяц();
карта.putEntity(new Coordinates(0, 0), заяц);
карта.moveEntity(new Coordinates(0, 0), new Coordinates(99, 99));

/* class Map */
public void moveEntity(Coordinates from, Coordinates to, Entity entity, заяц) {
  map.remove(from);
  map.put(to, entity);
}
```
Должна ли карта учитывать логистику зайцев? Если да, то как? Правильный ответ- карта вообще не должна иметь в себе метод телепортации.

-Избыточно
```
public boolean isHaveFreePlaceOnMap() {
  int square = xSize * ySize;
  for (Coordinates c : map.keySet()) {
    square--;
  }
  return square != 0;
}

//ПРАВИЛЬНО:
public boolean isHaveFreePlaceOnMap() {
  return map.size() < xSize * ySize;
}
```

**10. abstract class Creature extends Entity**

-Нарушение инкапсуляции. Всегда явно указывай область видимости полей. В данном случае они должны быть private
```
public abstract class Creature extends Entity {
  int speed;
  int health
  //...
}
```

-Нет
```
///  метод который будет переопределен только у Herbivore это нормальная практика так делать????
```

это грубая ошибка при организации наследования. Наследники должны реализовывать все поведение предка, иначе это нарушение LSP
```
public class Predator extends Creature {
  @Override
  void performActionTwo(Coordinates coordinates, Map map) {
  }
  //...
}

public class Herbivore extends Creature {
  @Override
  void performActionTwo(Coordinates coordinates, Map map) {
    heal(coordinates, map);
  }
  //...
}
```

-Нарушение SRP. Поиск пути нужно вынести в отдельный класс- читай чеклист для самопроверки в ТЗ.

**11. class Herbivore extends Creature**

-Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println("Herbivore x:" + coordinates.x() + " y:" + coordinates.y() + " полечилась на: " + healAfterEat + " текущее hp " + newHp);
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack: https://t.me/zhukovsd_it_chat/53243/139594

**12. Пакет Actions**

-Идея Actions здесь не осмыслена. 
Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.
Actions, изложенный в ТЗ, это вариант реализации паттерна Command.
Сейчас Actions у тебя это просто разные классы, у которых между собой нет ничего общего, кроме названия.

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники. 
Т.е. акции должны быть родственны и одинаково использоваться через полиморфизм. Примерно так
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

**13. class Action и его наследники:** InitMapAction, MapConsoleRenderAction и т.д.

-Наследование здесь это чисто фикция. Родительский класс Action имеет метод putEntityCountTimes(...), который не используется в большинстве его потомков.

**14. class MapConsoleRenderer**

-В switch-case в default нужно кидать исключение
```
return switch (entity.getClass().getSimpleName()) {
  case "Tree" -> SYMBOL_TREE;
  //...
  default -> SYMBOL_EMPTY;
};
``` 
Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт

**15. public class Simulation**

-Обычное для проекта нарушение инкапсуляции
```
public class Simulation {
  Map map;
  boolean didEntityMove;
  //...
}
```

-Тут ты был за шаг до понимания сути Action'ов. Увы, эти поля не используются в программе
```
Action[] initActions = new Action[]{new InitMapAction(), new InitEntityAction(), new MapConsoleRenderAction()};
Action[] turnActions = new Action[]{new MoveEntityAction(), new AddGrassAction(), new MapConsoleRenderAction()};
```

Вместо этого "псевдо-Action'ы" используются как просто набор несвязанных между собой классов
```
InitMapAction initMapAction = new InitMapAction();
InitEntityAction initEntityAction = new InitEntityAction();
MapConsoleRenderAction mapConsoleRenderAction = new MapConsoleRenderAction();
MoveEntityAction moveEntityAction = new MoveEntityAction();
AddGrassAction addGrassAction = new AddGrassAction();
```

**16. class Main**

-Любой закоментированный код в проекте- по определению мусор
```
public static void main(String[] args) {
  Simulation simulation = new Simulation();
  simulation.initSimulation();

///  для бесконечной симуляции (в теории)
  simulation.startInfinitySimulation();

//        /// симуляция конкретного числа циклов
//        simulation.startCountSimulation(10);

//        /// для симуляции одного хода
//        simulation.nextTurn();
}
```

Видимо, тут предпалагается, что если нужно выполнить не бесконечную симуляцию а, например, конкретное число циклов, то соответствующий кусок кода нужно раскоментировать, а остальные куски- закоментировать.
А потом вернуть все взад.
Не делай так. В данном случае нужно сделать просто три класса с майнами, каждый из которых будет запускать симуляцию со своим режимом работы.

## АРХИТЕКТУРА

Класс Map собрал в себе все худшие практики для класса Карты.  
Наследование часто пременяется неправильно.  
Action'ы сделаны неправильно.

## ВЫВОД

Нужно начать с основ: почитать конвенцию кода, придумывать осмысленные названия. 

n.51(119)  
#ревью #симуляция