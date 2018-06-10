# r_ajax_scraping
*Примеры работы с динамическими веб-страницами при скачивании данных из Интернета в `R`*

## Мотивация
Процесс сбора данных в ряду других стадий проведения статистического анализа, наверное, можно считать самым утомительным, особенно если речь идёт о работе с большими базами данных. Поэтому даже если от этого процесса нельзя отвертеться (вдруг кто-то уже собирал подобные данные), хочется свести затраты ручного труда к минимуму. Так что, как только передо мною встаёт факт необходимости сбора данных - в последнее время довольно большого размера - я начинаю обдумывать способы реализации [закона Парето](https://ru.wikipedia.org/wiki/Закон_Парето). Для конкретной ситуации я бы сформулировала его как выполнение 80% работы в 20% времени, чтобы в остальное - есть мороженное и смотреть британские детективные сериалы, например :smile:

Конечно, поиск подходящего решения может занять очень много времени. Это особенно актуально для вопросов, которые не имеют хорошего описания, документации или поддержки сообщества пользователей программного обеспечения. К сожалению, работа с динамичными веб-страницами при скачивании данных из Интернета (или просто веб скрапинге) относится к ним. С этим нужно что-то делать.

## Цель
Я не знаю ситуацию с Питоном, но за последнее время об основах веб скрапинга и его примерах в статистическом пакете R на английском была опубликована только одна книга: [Munzert, S., Rubba, C., Meißner, P., & Nyhuis, D. (2014). Automated data collection with R: A practical guide to web scraping and text mining. John Wiley & Sons](https://onlinelibrary.wiley.com/doi/book/10.1002/9781118834732). Когда я только начинала пробовать свои силы в веб скрапинге, будучи студенткой 4 курса бакалавриата политологии, эта книга была по-настоящему настольной для меня, но даже она не идеальна. Хотя она содержит раздел про динамические веб-страницы (AJAX), он был что называется теоретическим: в нём не было примеров. Обращение к материалам сайта [Stackoverflow](https://stackoverflow.com/) дало основу для модифицируемых в дальнейшем решений. Ими я и хочу поделиться в надежде, что кому-либо из коллег по цеху они пригодятся в их стремлении оптимизировать свою работу.

## Оговорка
Если кто-то ещё не понял, то речь идёт о веб скрапинге (web scraping), который я люблю называть сбором данных из Интернета.

## Собственно, содержимое

* [Нетехническое объяснение особенностей AJAX](#В-чём-отличие-ajax-от-html)

* [Скрипт с примерами и комментариями](ajax_examples.Rmd)

* [Ещё несколько полезных источников для ликбеза](#Больше-полезных-ресурсов-про-ajax)

## В чём отличие AJAX от HTML
Без ухода в историю понятий и чрезмерно технические детали для пользователей можно объяснить разницу между AJAX и HTML с помощью следующего примера. Допустим, кто-то открыл веб-страницу в браузере (назовём этого человека пользователем), на которой отображается информация о контактах и режиме работы некоего музея. Но последняя информация содержится в ссылке. Задача пользователя - узнать, сможет ли он посетить музей в воскресенье.

Если бы исходная страница (с контактами и ссылкой на режим работы) была создана с помощью HTML, или *языка* гипертекстовой разметки, то пользователь, нажав на ссылку режима работы музея, увидел бы, что в его браузере обновляется страница. Затем бы он увидел, что исходная страница сменилась новой, на которой он видит расписание работы музея на каждый день недели, но не видит информации с исходной страницы.

С другой стороны, будь исходная страница сделана с помощью AJAX, или *технологий* программирования, то пользователь бы видел новую информацию информацию на странице (расписание работы) и старую тоже (контакты музея) без того, чтобы веб-страница обновлялась в браузере. Таким образом, пользователь видит и то, и другое. Сам термин AJAX расшифровывается как Асинхронный JavaScript и XML. Много непонятных терминов! JavaScript и XML в свою очередь языки программирования (в основном или помимо прочего; например, XML в основном используется как язык маркировки, однако, есть примеры использования и для программирования). 

Если смотреть на разницу между HTML и AJAX с более технической стороны, то в первом случае для отображения новой или другой информации по ссылке заново отсылается информация о странице с сервера, тогда во втором - информация с сервера требуется только для обновления части страницы. С точки зрения опыта пользователя, AJAX отличная вещь, поскольку делает работу быстрее и удобнее. Но при попытке веб скрапинга это создаёт технические проблемы. С ними, тем не менее, можно справиться. Как - [по этой ссылке](ajax_examples.Rmd).

## Больше полезных ресурсов про AJAX

1. Ullman, C., & Dykes, L. (2007). Beginning Ajax. John Wiley & Sons.

2. Thomas, A. P. (2008). Ajax: The Complete Reference.

3. Одно из многочисленных и, кстати, дублирующих другие [пособий по базовой работе в {RSelenium}](http://rpubs.com/johndharrison/12843)

4. [Юникод и текстовые обозначения для команд клавиатуры и мышки при работе с {RSelenium}](https://github.com/ropensci/RSelenium/blob/master/R/selKeys-data.R#L24)
