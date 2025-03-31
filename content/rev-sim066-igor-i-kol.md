https://github.com/i-kol/Simulation  
[Igor]

Есть много над чем поработать.

## НЕДОСТАТКИ РЕАЛИЗАЦИИ

Нет защиты от дурака
```
Enter world map size.
Number of rows: 
цуцца
Exception in thread "main" java.util.InputMismatchException
	at java.base/java.util.Scanner.throwFor(Scanner.java:964)
    at java.base/java.util.Scanner.next(Scanner.java:1619)
```

## ХОРОШО

+ 👍 Спрайты существ не хранятся в самих существах
+ 👍 Координаты существ не хранятся в самих существах(мне так больше нравится)
+ 🚀 Есть редактор карты 

## ЗАМЕЧАНИЯ

**1. Нейминг**

- Название вводит в заблуждение. В данном случае переменная ничего не считает
```
Coordinates.rowCount
Coordinates.columnCount

//ПРАВИЛЬНО:
Coordinates.row
Coordinates.column
```

- Излишний контекст
```
WorldMap.mapWidth
WorldMap.mapHeight

//ПРАВИЛЬНО:
WorldMap.width
WorldMap.height
```

- Если в проекте есть класс WorldMap то все переменные с именем, включающим это название, должны быть экземплярами этого класса
```
public class WorldMap {
  public static HashMap<Coordinates, Entity> worldMap = new HashMap<>();
  //...
}

//ПРАВИЛЬНО:
public class WorldMap {
  public static HashMap<Coordinates, Entity> entities = new HashMap<>();
  //...
}
```

- Енамы содержат в себе констаны, константы должны писаться стилем UPPER_SNAKE
```
public enum EntitiesOnWorldMap {
  Grass, Tree, Rock, Herbivore, Predator;
}

//ПРАВИЛЬНО:
public enum EntitiesOnWorldMap {
  GRASS, TREE и т.д.
}
```

*Oracle Java code conventions, part."Naming conventions"*  
*Мартин, "Чистый код", гл.2*  
*Ютуб, Немчинский "Как называть переменные, методы и классы?"*  

**2. Используй классы через их интерфейсы**
```
HashMap<Coordinates, Entity> worldMap = new HashMap<>();

//ПРАВИЛЬНО:
Map<Coordinates, Entity> worldMap = new HashMap<>();
```

**3. Нарушение конвенции кода**, константы пишутся всегда сверху
```
protected int grassHealthRecover;
public final static int GRASS_HEALTH_RECOVER = 5;

//ПРАВИЛЬНО:  
public final static int GRASS_HEALTH_RECOVER = 5;
protected int grassHealthRecover;
```

**4. Нарушение DRY**, магические буквы, числа, слова. Вводи константы
```
//...
case "1":  //...
case "2":  //...
case "3":  //...
//...

System.out.println("""      
    Press:
    "1" and "Enter" - to pause the simulation
    "2" and "Enter" - to continue the simulation
    "3" and "Enter" - to make 1 move
    "4" and "Enter" - to stop the simulation and exit
    """);
 
//ПРАВИЛЬНО:
private final static String PAUSE = "1";
private final static String CONTINUE = "2";

//...
case PAUSE:  //...
case CONTINUE:  //...
//...

System.out.println("Press:");
System.out.printf("'%s' and 'Enter' - to pause the simulation %n", PAUSE);      
System.out.printf("'%s' and 'Enter' - to continue the simulation %n", CONTINUE);      
//...
```
*Фаулер, "Рефакторинг", гл.8 п."Замена магического числа символической константой"*  
*refactoring.guru "Замена магического числа символьной константой"*  

**5. Если нужно печатать текст** с более, чем одним подстановочным значением или значение вставляется внутрь сообщения, используй форматированный вывод- тогда сразу будет виден весь шаблон
```
System.out.println(getClass().getSimpleName() + " ate the Grass at: [" + coordinates.getRowCount() + "," + coordinates.getColumnCount() + "]");

//ПРАВИЛЬНО:
System.out.printf("%s ate the Grass at: [%d, %d] %n", getClass().getSimpleName(), coordinates.getRowCount(), coordinates.getColumnCount());
```

**6. Не используй статические импорты**, это делает код вообще нечитаемым- становится неясно, что используемый в классе метод не его собственный, а чей-то чужой
```
import static com.example.worldMap.WorldMap.removeEntity;
public class Predator extends Creature {
  //...
  removeEntity(coordinates);
  //...
}

//ПРАВИЛЬНО:
public class Predator extends Creature {
  //...
  WorldMap.removeEntity(coordinates);
  //...
}
```

**7. class Coordinates**

+ 👍 Нет ничего лишнего, это хорошо. 
* (±) Класс может быть преобразован в record без потери функционала.

**8. class WorldMap**

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему
```
public static HashMap<Coordinates, Entity> worldMap = new HashMap<>();
```
В данный момент клиентский код может выполнять любые операции с HashMap Карты напрямую, игнорируя допустимые картой способы. 
Например так
```
Карта карта = new Карта();
//заселить карту существами
карта.worldMap.clear(); //геноцид минуя дозволенные картой механизмы
```
Из-за этого класс является гибридом со всеми вытекающими последствиями: *"Чистый код", гл.6* 

- Нарушение инкапсуляции, класс предоставляет клиентскому коду подробности внутреннего устройства и доступ к нему 
```
public static int mapWidth;
public static int mapHeight;
```
Из-за этого, в том числе, трудно понять, как устанавливается размер карты.  

Сейчас это клиентский код делает напрямую, меняя значения в статических полях размеров в Карте.  
На самом деле размеры нужно передавать в конструктор при создании экземпляра карты, например так
```
public class WorldMap {
  public int width;
  public int height;

  public WorldMap(int width, int height) {
    this.width = width;
    this.height = height;
  }
  //...
}
```

- Нарушение дизайна ООП. Эти поля не должны быть статическими
```
public class WorldMap {
  public static int mapWidth;
  public static int mapHeight;
  public static HashMap<Coordinates, Entity> worldMap = new HashMap<>();
  //...
}
```
Статические поля в ООП программе нужно применять очень осторожно. 
Потому что сама идея классов состоит в том, что у класса может быть несколько экземпляров одновременно. И при этом все они должны корректно работать.

В данном случае если в программе будет существовать несколько экземпляров карты одновременно, они будут работать некорректно, потому что будут одновременно вносить изменения в одно и то же статическое поле.
И тут вопрос не в том, должно ли в проекте быть два экземплера одного класса, или не должно. 
Если класс не синглтон, то по умолчанию мы признаем, что его экземпляров в проекте может быть более одного.
Пока нормально не разобрался с ООП, статические поля лучше вообще не вводи в свои классы, кроме констант. 

- Все методы в классе статические, тем самым здесь напрочь убито ООП как явление. Методы в этом классе не должны быть статическими
```
public static Entity getEntity(Coordinates coordinates) {...}
public static void removeEntity(Coordinates coordinates) {...}
```  
Пока нормально не разобрался с ООП, лучше вообще не делай статические методы в классах, за исключением таковых в утилитных классах.

- Никогда не возвращай null
```
public static HashMap<Coordinates, Entity> worldMap = new HashMap<>();

public static Entity getEntity(Coordinates coordinates) {
  return worldMap.get(coordinates); //может вернуть null
}
```
Возврат null повышает риск возникновения NullPointerException в программе.  
*Мартин, "Чистый код", гл.7.7-8*  
*Ютуб, Немчинский "Почему нельзя возвращать NULL?"*  

- При всех операциях с участием координаты(добавить, выдать, удалить и т.д.), нужно проверять координату на корректность. 
Если координата некорректна(находится вне пределов карты), нужно бросать исключение
```
public static void removeEntity(Coordinates coordinates) {
  worldMap.remove(coordinates);
}
```

**9. class Pathfinder**, поиск пути

- В этом классе тоже все методы статические.
В принципе, этот класс может иметь только статические методы, но тогда класс должен быть final и иметь приватный конструктор, чтобы нельзя было создать экземпляр этого класса.

- Куча публичных методов, непонятно как пользоваться поиском. У этого класса должен быть один публичный метод, который просто ищет путь от точки старта до точки, соответствующей условиям.  
Например, так:
```
public List<Coordinates> getPatch(World world, Coordinates start, Class<? extends Entity> target) {...}
```  

- Нарушение SRP. Класс должен просто искать путь от точки старта до точки, соответствующей заданным условиям согласно алгоритму BFS или AStar. 
Эти условия класс должен принимать в себя и не определять эти условия самостоятельно, например путем анализа принадлежности Creature тому или иному виду существ
```
if ((getEntity(neighborForTargetSearch) instanceof Grass && getEntity(startCell) instanceof Herbivore) 
  ||(getEntity(neighborForTargetSearch) instanceof Herbivore && getEntity(startCell) instanceof Predator)) {...}
```
Поиск вообще не должен знать про особенности тех или иных существ, поиск просто должен искать путь от точки старта и т.д.

**10. enum EntitiesOnWorldMap**

- Не нужно заводить специальный енам для идентификации существ. Для идентификации объектов достаточно использовать метод getClass().

**11. abstract class Creature extends Entity**

- Нарушение SRP, чужая ответственность, зависимость модели от представления.
Модель(а это модель) не должна ничего печатать в консоль
```
System.out.println("Unknown type of creature");
```
Иначе модель перестает быть универсальной и становится заточенной под конкретную среду и конкретное представление себя в этой среде- в данном случае, консоль.
В других средах(напр. Андроид) модель нельзя будет использовать. 
Другое представление для модели(напр. если одну и ту же модель нужно в программе показвать по-разному) нельзя будет сделать, или придется делать через костыль.  
Вот к чему приводит зависимость модели от представления, карта рисуется в виндовый интерфейс, а модель пишет напрямую в консоль:  
https://github.com/raketareview/simulation_review/blob/master/content/rev-sim064-davidtagirov-DavidTagirov.md

Если модель должна что-то сообщить миру, она может это сделать через паттерн CallBack.  
CallBack луноход: https://t.me/zhukovsd_it_chat/53243/139594  
CallBack разрушение: https://t.me/zhukovsd_it_chat/53243/184749  

- Нарушение инкапсуляции. Вспомогательные методы делай приватными. Публичными должны быть только те методы, которые могут быть использованы клиентским кодом
```
public <T> void senseTheTarget(Coordinates coordinates, Class<T> targetClass) <-- это вспомогательный метод
```

- Нарушение OCP. Класс не должен знать своих потомков и как-то учитывать их в логике своей работы
```
if (this instanceof Herbivore) {
  senseTheTarget(coordinates, Grass.class);
} else if (this instanceof Predator) {
  senseTheTarget(coordinates, Herbivore.class);
}
```
Потому что при добавлении новых потомков, придется переписывать предка и тем самым нарушать правило "класс должен быть закрыт для изменений".

- Это еще что такое, почему ты здесь используешь Object?
```
Object entity = getEntity(cell);

//ПРАВИЛЬНО:
Entity entity = getEntity(cell);
```

**12. Пакет actions**

- Классы, находящиеся в этом пакете, не имеют ничего общего с Action's, которые описаны в ТЗ. Здесь экшены это просто какие-то классы, никак не связанные между собой.  
И снова классы содержат статические меоды, которые не дают шанса ООП.

- Идея Actions здесь не осмыслена. Идея состоит в том, что нужно создать семейство родственных классов, объединенных общим интерфейсом.
Каждый из этих классов должен делать что-то свое с картой: одна акция должна заселять карту существами, другая делать ходы и т.д.
Сейчас Actions у тебя это просто сборник глобальных функций, сделанный в стиле процедурного программирования

Смысл Action'ов состоит в том, что должен быть общий класс/интерфейс Action и его наследники. 
Это своего рода вариация паттерна Command.
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

- Нарушение инкапсуляции. Вспомогательные методы делай приватными. Например, `double selectCreatureCountRatio(...)` в `class EntitiesRespawn`.

**13. class WorldRenderer**

- В switch-case в default нужно кидать исключение. Потому что при добавлении нового существа в проект, можно забыть добавить его спрайт в распечатку.
И тогда существо будет жить на карте своей жизнью, но мы этого не увидим- вместо этого существа будет распечатываться дефолтный спрайт
```
return switch (entity.getClass().getSimpleName()) {
  case "Herbivore" -> HERBIVORE;
  //...
  default -> DEFAULT_SPRITE;
};
``` 

**14. class Menu**

- Опять всё статика.

* (±) Меню в процедурном стиле, для простой программы- почему бы и нет. Про меню в ООП стиле я писал тут: https://t.me/zhukovsd_it_chat/53243/114908

**15. class Simulation**

- Опять всё статика.
- Симуляция должна принимать необходимые зависимости в конструктор. В данном случае- как минимум экземпляр карты. 

## ВЫВОД

Сделать Action'ы по ТЗ. Переосмыслить использование статических полей и методов в ООП. Перестать использовать статические импорты.  
Нет понимания ООП на базовом уровне: зачем вообще нужно(или не нужно) создавать объекты и как при этом надо использовать конструкторы.  
Для лучшего понимания декомпозиции посмотреть ролики Сергея про шахматы. Посмотреть ролики Немчинского про инкапсуляцию, наследование, полиморфизм, SOLID

n.66(148)  
#ревью #симуляция