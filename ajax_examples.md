# Веб скрапинг страниц с AJAX с помощью R

Ниже я надеюсь разъяснить, как использовать возможности R для того, чтобы получать данные с динамических веб страниц, созданных с помощью технологий AJAX. Такие страницы не всегда имеют расширение `.html`, к слову.

Сложность нижеследующих примеров, на мой взгляд, не очень большая. В каких-то случаях она больше, чем показано в пособиях для новичков, но но ни один из случаев не требовал слишком обильного масштабирования или ещё чего-нибудь затрудняющего, типа огромного скрипта.

Обобщить опыт можно тремя способами:

1. [Доступ по внешней ссылке: база адресов ЦИК](#Принадлежность-домов-к-уик)

2. [Работа с внешними программами: скачивание отчётов](#Скачивание-множества-отчётов-групп-интересов-ес)

3. [Удалённый доступ из R через {RSelenium}: навигация по главной странице сайта ЦИК](#Получение-множества-ссылок-на-различные-выборы-с-главной-страницы-цик)

## Пакеты для работы

Для своего удобства не буду дублировать их для каждого из типов решений. Интересующимся лучше искать их по функциям в Интернете.

```r
# install.packages("RCurl")
library(RCurl)
# install.packages("jsonlite")
library(jsonlite)
# install.packages("rvest")
library(rvest)
# install.packages("stringr")
library(stringr)
# install.packages("magrittr")
library(magrittr)
# install.packages("RSelenium")
library(RSelenium)
```

## Принадлежность домов к УИК
На сайте ЦИК можно осуществлять поиск своего УИК по [адресу](http://www.cikrf.ru/services/lk_address/?do=address). Рассмотрим задачу, когда пользователю необходимо собрать информацию о принадлежности квартир в одном доме к УИК. Этот дом находится по адресу г. Москва, проспект 60-летия Октября, д. 10/1 в Академическом районе. Это может понадобиться, чтобы исследовать, как меняются границы избирательных участков. Полезно для достижения точности статистического анализа результатов выборов на дизагрегированном уровне.

Просматривая страницу Сеть в инструментах для веб-разработчиков в браузере (Опции>>Веб-разработка>>Сеть в Firefox), мы видим, что, пока не кликнем  на иконку с папкой для района, улицы или дома, информация о подгружаемом файле в формате JSON не будет отображаться. Конечную информацию о принадлежности дома к УИКу придётся агреггировать из данных о принадлежности квартир. Для доступа к этой информации к базовой ссылке [http://www.cikrf.ru/services/lk_address/do=result?do=address](http://www.cikrf.ru/services/lk_address/do=result?do=address) необходимо добавить ряд цифр в середину адреса. Конечная ссылка такая: [http://www.cikrf.ru/services/lk_address/138452622832576000000353023?do=result](http://www.cikrf.ru/services/lk_address/138452622832576000000353023?do=result). 

Кликнув на иконку нужного дома, получаем в окне браузера Опции>>Веб--разработка>>Сеть адрес JSON файла с нужными наборами цифр для квартир: [http://www.cikrf.ru/services/lk_tree/?id=8458211578](http://www.cikrf.ru/services/lk_tree/?id=8458211578).

```r
house_json = getURL("http://www.cikrf.ru/services/lk_tree/?id=8458211578",.encoding = "CE_UTF8") %>% 
  fromJSON()
json_to_dt <- function(init_file) {
  l = 1:length(init_file)
  text = lapply(l,function(x)init_file[[x]][["text"]]) %>% unlist()
  id = lapply(l,function(x)init_file[[x]][["id"]]) %>% unlist()
  intid = lapply(l,function(x)init_file[[x]][["a_attr"]][["intid"]]) %>% unlist
  init_file = cbind(text,id,intid) %>% data.frame()
}
json_dt = json_to_dt(house_json)
url_addr = paste0("http://www.cikrf.ru/services/lk_address/",json_dt$intid,"?do=result")
list_uik = lapply(url_addr,function(x)getURL(x,.encoding = "CE_UTF8") %>% str_match(.,"№\\d{1,4}")) %>% unlist() %>% print()
```

Вуаля. Сначала скачиваем файл JSON и переводим его в формат `data.frame`, потом полученные значения id квартир подставляем к конечной ссылке, скрэпим, чистим немного и смотрим, что получилось. Получилось, что все квартиры в доме относятся к одному УИК. Расширение задачи к сбору данных обо всех домах в Москве или России просто ведёт к увеличению времени сбора и необходимости гораздо больших мощностей. Одним из решений может быть подключение к внешними базам данных.

## Скачивание множества отчётов групп интересов ЕС

С этого случая всё и началось! Представим, что нужно получить информацию о ряде элементов на веб--странице, но часть из них скрыта под условным катом. Например, получить ссылки для скачивания отчётов одной из групп интересов в .pdf.

Для этого я оптимизировала простой скрипт для открытия динамических страниц с помощью PhantomJS, браузера, который работает не с кликами мышки, а с кодом. Скрипт гуляет по сети и, например, есть на [DataCamp](https://www.datacamp.com/community/tutorials/scraping-javascript-generated-data-with-r). Но мне нужно было пролистнуть страницу:

```phantomjs
var webpage = require('webpage').create();

var url ='http://ldac.chil.me/publications';
var page = new WebPage()
var fs = require('fs');

webpage.viewportSize = { width: 1280, height: 800 };
webpage.scrollPosition = { top: 0, left: 0 };

webpage.open(url, function(status) {
	if (status === 'fail') {
	console.error('webpage did not open successfully');
	phantom.exit(1);
							}
var i = 0,
top,
queryFn = function() {
	return document.body.scrollHeight;
					};
setInterval(function() {
		var filename = 'ldac-' + (++i) + '.png';
		console.log('Writing ' + filename + '...');
		webpage.render(filename);
		top = webpage.evaluate(queryFn);
		console.log('[' + i + '] top = ' + top);
		webpage.scrollPosition = { top: top + 1, left: 0 };
	if (i >= 5) {
	fs.write('ldac_page_save.html',webpage.content,'w')
		phantom.exit()
				}
	}, 3000);
});
```
Работа по скачиванию, тем не менее, ведётся в R.

```r
system("phantomjs open.js") # выполняем чтение скрипта выше
ldac_page = read_html("ldac_page_sav.html") # вчитываем в память R выданную в итоге страницу
ldac_dwn_url = html_nodes(ldac_page,xpath = ".//h4/a/@href") %>% html_text() %>% 
  paste0("http://ldac.chil.me",.) %>% str_replace_all(.,",","") # получаем ссылки на файлы, делаем название документов
ldac_dwn_names = html_nodes(ldac_page,xpath = ".//h4/a/span") %>% html_text() %>% 
  str_replace_all(.,"/","-") %>% str_replace_all(.,"\\.","-") %>% tolower(.) %>% 
  str_replace_all(.," ","_") %>% str_replace_all(.,"º","") %>% 
  str_replace_all(.,"´","") # исправляем несовершенные названия, чтобы при скачивании не было ошибок

ldac_dwnl_dir = paste0(ldac_dwn_names,".pdf") # создаём название файлов
for (i in 1:length(ldac_dwn_url)) {
  download.file(url = ldac_dwn_url[i],ldac_dwnl_dir[i],
                method = "libcurl",quiet = F,mode = "wb",cacheOK = T,
                extra = getOption("download.file.extra"))
} # обычно я использую масштабирование apply-семьи, но в данном случае это работает лучше
```
Около сотни скачанных файлов в `.pdf` теперь можно читать лично или начать процедуру распознавания документов и доверить текстуальный анализ машине.

## Получение множества ссылок на различные выборы с главной страницы ЦИК

До того, как научиться работать с данными ЦИК РФ по нескольким выборам с помощью `{RSelenium}`, я находила нужные мне выборы, собирала в файл `.xlsx` ссылки вручную и затем доставала эти данные с помощью масштабированной функции `GET()`. Ниже я представлю, как адаптировать выдачу количества и типов выборов в зависимости от манипуляций удалённо, а не "по старинке", то есть прямо в память R. При определённом опыте это позволит сэкономить кучу времени.

По сути, вариант с `{RSelenium}` мало отличается от предыдущего. В чём-то он даже удобнее, но в каждом отдельном случае сложность строения страницы определяет нужный выход. С другой стороны, `{RSelenium}` в чём-то проблемный, потому что часто возникают проблемы с портами доступа. Например, у меня не срабатывает базовый порт 4444, но сработал 4567.  Технически я не понимаю всю разницу между ними, но интересующиеся могут поискать больше информации про порты и какую-нибудь уязвимость компьютера (ну а как не безопасностью компьютера все озадачиваются?). У многих также работает доступ только к браузерам Firefox или PhantomJS.

При работе в `{RSelenium}` происходит следующее: пользователь R подключается к стороннему серверу, на котором происходят операции с веб страницей. Эти операции включают в себя введение информации в текстовые поля, кликание на кнопки, ссылки и проставление флажков. Всё происходит условно говоря в "режиме настоящего времени", поэтому модификации с множеством полей очень удобно.

```r
rsDriver() # подключаемся к удалённому серверу
remDr = remoteDriver(remoteServerAddr="localhost",
                 port=4567L, # L в данном случае показывает, что это не знак, а число
                 browserName="firefox") # по документации можно использовать Chrome, Internet Explorer, Firefox и PhantomJS
remDr$open() # открываем браузер; его можно открыть в фоновом режиме или воспользоваться PhantomJS
remDr$getStatus() # проверяем, получилось ли открыть
remDr$navigate("http://www.izbirkom.ru/region/izbirkom") # переходим на главную страницу сайта ЦИК
# для выбора выборов можно искать в определённом временном промежутке, по типам и уровню выборов
# попробуем собрать ссылки для всех президентских выборов
start_date = remDr$findElement(using="xpath",value = ".//input[@value='01.04.2018']") # дата меняется во времени
start_date$sendKeysToElement(list("\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003")) # знаки юникода заменяемы на key="backspace" в каждом случае; задача - стереть нынешнее значение
remDr$sendKeysToElement(list("01.03.2003"))
end_date = remDr$findElement(using = "xpath",value=".//input[@value='30.09.2018']") # опять же, дата изменяема
end_date$sendKeysToElement(list("\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003","\uE003"))
end_date$sendKeysToElement(list("01.04.2018"))
level_el$findElements(using = "xpath",value=".//div[@tabindex='0']")[[1]]#здесь нужно немного покапаться в коде страницы снача; выбираем Федеральные
level_el$clickElement() # отмечаем элемент
level_el$sendKeysToElement(list(key="down_arrow",key="down_arrow",key="enter")) # возможно, не самый варварский способ навигации в списке
type_el = remDr$findElements(using = "xpath",value = ".//div[@tabindex='0']")[[2]] # выбираем Выборы на должность
type_el$clickElement() # отмечаем элемент
type_el$sendKeysToElement(list(key="down_arrow",key="down_arrow",key="down_arrow",key="enter"))
search_pg = remDr$findElement(using = "xpath",value=".//input[@type='submit']")#теперь надо нажать на кнопку "Искать"
search_pg$clickElement() # кликаем на кнопку - получаем результат поиска
urls = remDr$findElements(using = "css",value = "[href]") # собираем ссылки на выпавший список выборов
all_presid_el_url = unlist(sapply(urls,function(x){x$getElementAttribute("href")}))
all_presid_el_url = all_presid_el_url[7:10]
```
ОК, теперь есть ссылки на страницы с каждыми президентскими выборами Президента РФ, которые есть на сайте ЦИК. Если цель не просто ссылки, а результаты выборов, то процесс сбора данных гораздо больше.
