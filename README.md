# python_project_Berlizova_Manokhina
В нашем репозитории лежат 2 основных файла: с парсером и с основной частью анализа данных. 

**Парсинг и заполнение пропусков**

Мы парсили датасет с сайта https://ru.kinorium.com/collections/critics/131/ Там представлен ТОП-1000 фильмов, которые нужно посмотреть прежде, чем умереть по версии The Guardian. Парсинг проводился через selenium и BS4, для того, чтобы все работало, нужно скачать Chromedriver по ссылке: https://chromedriver.chromium.org/downloads Также там есть проверочный скрипт, по которому можно узнать, все ли работает. (иногда сайт во время парсинга выдает капчи, с этим остается только смириться и один раз ее пройти).

Парсинг проходит постранично, мы открываем каждый из 50 фильмов на странице и берем оттуда оригинальное название, русское название, режиссера, год выпуска, страну производителя, бюджет, сборы по миру и ссылку. В результате мы получили датасет из 8 переменных, часть из которых числовые, часть содержат в себе текстовые данные.
В данных содержатся пропуски, а именно в столбцах с бюджетом и сборами по миру. Они были заменены на средние значения по каждой переменной с той целью, чтобы не изменить медианные, модальные а также крайние значения.

**Визуализация**

Для перехода к анализу данных нужно открыть второй файл в репозитории. Начинается работа с проверки того, сколько уникальных значений содержится в каждом столбце и не осталось ли там пропусков. После того, как мы удостоверились, что данные в порядке, мы перешли к визуализации.

Представлены следующие графики: плотность распределения рейтинга фильмов, гистограммы с ТОП-20 фильмов по бюджету и сборам по миру, а также гистограммы, которая показывает бюджеты фильмов с самыми высокими сборами и рейтингом, интересная для того, чтобы показать, что не всегда дороже значит лучше. Далее мы построили хитмап, который показывает, что между бюджетом и сборами по миру все же присутствует положительная линейная связь, коэффициент корреляции достаточно высок. Для подтверждения этого также мы построили график зависимости сборов от бюджета фильмов, на котором можно заметить восходящий тренд. Следующим этапом анализа стало построение гистограммы по тому, в какой год выходило наибольшее количество фильмов из топа. Мы получили вывод о том, что The Guardian считают, что людям необходимо смотреть в основном фильмы 80-ых и 90-ых годов.

**Создание новых признаков**

Тут у нас все достаточно очевидно, мы вывели прибыли для каждого фильма, представляющую собой разность между бюджетами и сборами по миру, и добавили этот столбец к нашему датасету. Где-то прибыль была положительна, где-тот отрицательна, а какие-то фильмы ушли в 0. Именно с этой переменной мы далее работали в блоках, связанных с гипотезами и машинным обучением.

После этого мы сгруппировали все фильмы из нашего топа по странам, и увидели, что больше половины было снято в США. На основании данного наблюдения мы выдвинули гипотезу, представленную в следующем блоке.

**Проверка гипотез** 

Мы выдвинули гипотезу о том, что фильмы, снятые в США, собирают столько же денег, что и фильмы, снятые в других странах. Гипотеза была проверена на уровне значимости 5%.

Н0: Прибыль фильмов, снятых в США, такая же, как и прибыль фильмов, снятых в других странах.
H1: Прибыль фильмов, снятых в США, больше прибыли фильмов, снятых в других странах.

Для проверки гипотез был использован критерий Манна-Уитни. Его предпосылками является то, что в каждой из выборок наблюдения независимо идентично распределены и значения случайных величин могут быть упорядочены. Мы проверили нормальность данных с помощью теста функцией normaltest. Гипотеза о нормальности данных была отвергнута на всех уровнях значимости.

Для проверки нашей гипотезы с помощью критерия Манна-Уитни были созданы выборки usa_income - фильмы, в стране производства которых указано “США”, other_income - фильмы, в стране производства которых указаны другие страны и нет “США”. После использования встроенной функции для критерия Манна-Уитни с alternative = greater мы получили p-value = 5.634125251209793e-33. Это меньше 5%, поэтому нулевая гипотеза была отвергнута в пользу альтернативной.

**Машинное обучение**

В данном блоке была построена логистическая регрессия на основании имеющихся числовых признаков. Прибыль мы сделали бинарной, неотрицательная прибыль была заменена на 1, отрицательная на 0. Также мы убрали из таблицы столбцы с текстовыми данными и столбец со сборами по миру, ведь между бюджетом, сборами по миру и прибылью существует очевидная линейная связь, мы решили, что для предсказаний достаточно одной из 2 переменных. Мы также дополнительно преобразовали датафрейм, а именно убрали ту строку, где речь идет о сериале, у которого нет года выпуска, но есть информация о всех годах съемки и количестве сезонов, так как регрессия не работает с нечисловыми переменными.

Далее мы применили MinMaxScaler, чтобы отнормировать данные и не получать огромные значения для коэффициентов регрессии. Ведь мы работаем с шестизначными числами, регрессии такое не любят.

После построения модели мы перешли к подбору наилучших параметров с помощью GridSearchCV. Для этого мы:
1) использовали перебор по сетке;
2) в качестве метрики использовали ROC AUC;
3) для оценки параметров сделана кросс-валидация на 4 фолдах.

Мы использовали кросс-валидацию, так как именно она позволяет оценить устойчивость модели при изменении данных. В качестве метрики выбрана ROC AUC, так как она показывает, насколько хорош классификатор и насколько точны предсказания. Также она не зависит от порога. 
В результате были получены следующие параметры: Best solver - saga, лучшее значение - {'C': 9.50714306409916}, ROC AUC для лучшего параметра - 0.8568797953964196

Далее мы построили модель с наилучшими параметрами и оценили качество ее предсказаний с помощью precision, recall и accuracy score.
1) Precision отражает то, насколько мы можем доверять алгоритму, если он спрогнозировал единичку;
2) Recall показывает, как много объектов первого класса наш алгоритм находит, то есть метрика демонстрирует способность алгоритма обнаруживать данный класс вообще.
3) Accuracy показывает долю верно классифицированных объектов.

На тестах были получены следующие значения: accuracy =  0.625, precision =  0.6052631578947368, recall =  1.0. Это говорит нам о том, что модель после подбора гиперпараметров предсказывает достаточно точно, особенно высоки показатели ROC AUC и recall. 

**Выводы**

В результате выполнения проекта были сделаны следующие выводы:
1) Между бюджетом фильма и сборами по миру присутствует положительная линейная связь, коэффициент корреляции достаточно высок. 
2) The Guardian считают, что людям необходимо смотреть в основном фильмы 80-ых и 90-ых годов.
3) Больше половины фильмов из топа было снято в США.
4) Прибыль фильмов, снятых в США, больше прибыли фильмов, снятых в других странах.
5) Использованная для машинного обучения модель после подбора гиперпараметров предсказывает достаточно точно, особенно высоки показатели ROC AUC и recall.
