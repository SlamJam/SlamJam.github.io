Немножечко набросов и языкового радикализма =). Во многом может резковато, велкам в коменты.

<!-- TEASER_END -->

Всё чаще всплывает «а писать надо на... Go!». Давайте посмотрим, сколько можно.

От чего болит. Реально болит:

- Дженерики. Тут всё просто - их нет. Отсюда сразу недотипизация коллекций, тут и там вылезающий `interface{}`.
  Вспоминается Джо Армстронг, автор _Erlang_'а, который сказал, чтобы не получить из коллекции какую-то хрень, не надо её туда класть.
  Но если у нас столь тонкие проверки, нужны ли они вообще?  
  Отсюда же вытекает отсутствие обобщённых функций. Оооо эти милые `MinInt32`, `MinInt64`, `MinFloat`, вместо `Min<T where T is Comparable>`.

- Канал можно закрыть. Если пишешь в закрытый канал - это **fatal**.

- Идея, заложенная в менеджмент зависимостей, их пининг. Реализация через вендоринг. То, что это сделано из коробки - это хорошо.
  Но то, _КАК_ это сделано - это боль и страдания.
  Сейчас есть некое движение к стандартизации в тулах `dep` и `vgo`.
  Сейчас можно пользоваться тулом `glide`, но зачастую это выливается в боль, непонятные ошибки и периодические вынужденные полные сбросы его кэша.

- Можно шарить _мутабельный_ стейт между горутинами - как итог взрыв, кишки, падения. Можно передать ссылку на мутируемые данные через канал.
  Причём [Go Mem Model](https://golang.org/ref/mem#tmp_2) декларирует, что одновременный доступ к данным не специфицирован.
  Поэтому не стоит строить Lock-free и думать о барьерах памяти.

- Совершенно невозможно писать переиспользуемые абстракции. В языке просто нет выразительности.
  По большому счету это тапок в сторону системы типизации и дженериков.

- Структурная типизация. Она защищает от физических ошибок, в рантайме не упадёте.
  Но вот логические ошибки... Если у `Person` и `Dog` есть по полю `name`, то всунув пёсика вместо человека ничего не упадёт, но будет неприятненько.

- _[богатая]_ стандартная библиотека _[достаточно]_ дерьмового качества.
  Вылетели из головы примеры нерасширяемого дизайна и тупо захардкоженых вещей, но впечатления остались. Можно на веру не принимать, но и не отметить я не мог.

- `go` для запуска горутин, втащено на уровень языка и синтаксиса.
  Логически это сугубо библиотечный уровень. При профайлинге или отладке мелькают _goroutine id_, но почему-то нет возможности по такому _id_ узнать статус или послать некий сигнал.
  Отсюда нельзя узнать, что горутина померла, и также нельзя прибить её извне.
  Такой вот принцип из принципа.  
  Развитие претензий этого пункта - нельзя интераптнуть горутину. Как строить «Fault-Tolerant» систему, не имея возможности декларировать:
  сделай это действие за _N_ времени или умри?
  Предлагают использовать шаблон [Context](https://golang.org/pkg/context/). Первым параметров прокидавать специальный объект, который везде добавлять в вызовы `Select`.  
  Ну ок, ещё и таймауты будем руками пробрасывать.

- В сторону дизайна языка можно бросить странное наличие кортежей, их распаковки, но недоступность этого для свободного использование программистом. Вернуть результат и ошибку, то есть два значения, из функции вы можете, а свободно обращаться с кортежами нет.

- Ошибки. `errors.New/fmt.Errorf` и не возможность заматчить пришедшую в результате выполнения функции ошибку, только по подстроке (и именно этот подход используют сами разрабочики в тестах!).

Всё это вместе приводит к тому, что в достаточно большом (каждый вибирает границы по вкусу) проекте вы уже не можете гарантировать выполнение глобальных инвариантов, а одиночные постреливания всё чаще и чаще приводят к ночным отказам. Наверное, это тема отдельного поста, но я веду речь о развивающихся системах. Не о том софте, в котором овер 100 KLOC и всё хорошо, но по сути он на поддержке у большой команды и развитие новой функциональности идёт черепашьими темпами. А я рассматриваю Go, если не как серебряную пулю, то уже как претендента на звание достойного инструмента для строительства больших, активно развиваемых систем. К сожалению такого я в нём не нашёл =((.


Теперь о хорошем. А хорошего тут немало и многое из этого уникально:

- Горутины. Идея реализации канкаренси на файберах — стековых корутинах. Ггорутины снимаются с исполнения если молотят слиииишком долго, но обычно смена контекста происходит в точках прерывания ввода/вывода, или по специальным вызовам типа сна или сискола.

- Статический бинарь. Божественная идея, реализованная на 5 баллов. Нет зависимости от сторонних туллчейнов и даже от `libc`.

- Приличная скорость кода. Примерно в 2 раза медленнее Си, что оооооочень и очень хорошо.

- Отличнейшие _«изкоробочные»_ тулы (дедлок-детектор, форматтер кода, поиск гонок, профайлер).
  Здесь оттельно отметим систему сборки из коробки. То есть когда вы видите проект на _Go_, вы знаете как его собрать и скорее всего у вас это получится.

- Форсирование форматирования кода, отсутствия неиспользуемых переменных.
  При разработке скорее мешает, наверное, надо валиться только на этапе сборки в _release_.

- _Блейзинг-эмейзинг_ быстрая компиляция и сборка. Просто молния! Скорость итерации разработки фантастическая для компилируемого [частично]~статически~типизированного языка. Не шутка, что можно прописать в шибенге и использовать как шелл скрипты.

- [CMS](https://blog.golang.org/ismmkeynote) сборщик мусора. Параллельный, инкрементальный, с микропаузами в работе. Весьма шустро без гигантских stop-the-world'ов позволяет крутить большими хипами (несколько десятков GB).

Про статический бинарик. На мой взгляд основная причина неуспеха применения _Ruby/Python_ в разработке инфраструктуры во многом из-за их собственной языковой инфраструктуры. Трудно паковать, надо уметь чётенько это делать. 100500 файлов. 100500 способов. Про _Python_ явно будет отдельный пост. Как итог: видишь приличного размера проект на _Python_, сразу знаешь, что в _setup.py_ будет ад, и скорее всего собрать это будет не тривиально. Если не сам проект, то уж парочку из его зависимостей так точно =)).



Мой личный итог: «хороший инструмент для непритязательных поделок низкоквалифицированных (около)разработчиков» (c). Цитата честно скоммунисжена.
Чем и объясняется его дикая популярность для инфраструктуры. Классическое _Easy_ в выборе _Easy/Simple_, оно же — «удобненько».
Выбор «практичных» людей, которым просто писать код, просто решить задачу, просто принести сиюминутную пользу бизнесу, просто закрыть тикет.
Но слава Богу, больше микросервисов обычно никто не пишет и столь негативных эффектов мы не наблюдаем, и стоимость разработки не растёт как снежный ком.

---

Fix #1. Спасибо [@corpix](https://t.me/documentsjournal) за правки и дельные замечания.
Добавил пункт про ошибки.
