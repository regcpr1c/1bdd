# 1BDD для OneScript

`1bdd` - инструмент для выполнения автоматизированных требований/тестов, написанных на обычном, не программном языке.

Иными словами, это консольный фреймворк, реализующий `BDD` для проекта [OneScript](https://github.com/EvilBeaver/OneScript).
Для Windows и Linux.

Идеи черпаются из проекта [Cucumber](https://cucumber.io).

[![Join the chat at https://gitter.im/artbear/1bdd](https://badges.gitter.im/artbear/1bdd.svg)](https://gitter.im/artbear/1bdd?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)  Здесь вы можете задавать любые вопросы разработчикам и активным участникам.

# Командная строка запуска

```
oscript bdd.os <features-path> [ключи]
oscript bdd.os <команда> <параметры команды> [ключи]

Возможные команды:
	<features-path> [ключи]
	или
	exec <features-path> [ключи]
		Выполняет сценарии BDD для Gherkin-спецификаций
		Параметры:
			features-path - путь к файлам *.feature.
			Можно указывать как каталоги, так и конкретные файлы.
			
			-fail-fast - Немедленное завершение выполнения на первом же не пройденном сценарии

			-name <ЧастьИмениСценария> - Выполнение сценариев, в имени которого есть указанная часть
			-junit-out <путь-файла-отчета> - выводить отчет тестирования в формате JUnit.xml

	gen <features-path> [ключи]
		Создает заготовки шагов для указанных Gherkin-спецификаций
		Параметры:
			features-path - путь к файлам *.feature.
				Можно указывать как каталог, так и конкретный файл.

Возможные общие ключи:
	-require <путь каталога или путь файла> - путь к каталогу фича-файлов или к фича-файлу, содержащим библиотечные шаги.
		Если эта опция не задана, загружаются все os-файлы шагов из каталога исходной фичи и его подкаталогов.
		Если опция задана, загружаются только os-файлы шагов из каталога фича-файлов или к фича-файла, содержащих библиотечные шаги.

	-out <путь лог-файла>
	-debug <on|off> - включает режим отладки (полный лог + остаются временные файлы)
	-verbose <on|off> - включается полный лог
```

Для подсказки по конкретной команде наберите
`bdd help <команда>`.

# Формат файла шагов

Это обычный os-скрипт, который располагает в подкаталоге `step_definitions` относительно файла фичи.

В этом файле должна быть служебная функция `ПолучитьСписокШагов`, которая возвращает массив всех шагов, заданных в этом скрипте.

Также внутри файла шагов могут располагаться специальные методы-обработчики (хуки) событий `ПередЗапускомСценария`/`ПослеЗапускаСценария`

## Пример файла шагов

```
// Реализация шагов BDD-фич/сценариев c помощью фреймворка https://github.com/artbear/1bdd

Перем БДД; //контекст фреймворка 1bdd

// Метод выдает список шагов, реализованных в данном файле-шагов
Функция ПолучитьСписокШагов(КонтекстФреймворкаBDD) Экспорт
	БДД = КонтекстФреймворкаBDD;

	ВсеШаги = Новый Массив;

	ВсеШаги.Добавить("ЯСохранилКлючИЗначениеВПрограммномКонтексте");
	ВсеШаги.Добавить("ЯПолучаюКлючИЗначениеИзПрограммногоКонтекста");

	Возврат ВсеШаги;
КонецФункции

// Реализация шагов

// Процедура выполняется перед запуском каждого сценария
Процедура ПередЗапускомСценария(Знач Узел) Экспорт
	
КонецПроцедуры

// Процедура выполняется после завершения каждого сценария
Процедура ПослеЗапускаСценария(Знач Узел) Экспорт
	
КонецПроцедуры

//Я сохранил ключ "Ключ1" и значение 10 в программном контексте
Процедура ЯСохранилКлючИЗначениеВПрограммномКонтексте(Знач Ключ, Знач Значение) Экспорт
	БДД.СохранитьВКонтекст(Ключ, Значение);
КонецПроцедуры

//я получаю ключ "Ключ1" и значение 10 из программного контекста
Процедура ЯПолучаюКлючИЗначениеИзПрограммногоКонтекста(Знач Ключ, Знач ОжидаемоеЗначение) Экспорт
	НовоеЗначение = БДД.ПолучитьИзКонтекста(Ключ);
	Ожидаем.Что(НовоеЗначение).Равно(ОжидаемоеЗначение);
КонецПроцедуры
```

# API фреймворка

Есть несколько вариантов использования API фреймворка из кода реализации шагов.

## Программный контекст

Для обмена информацией внутри кода реализации шагов можно использовать API контекста, предоставляемый продуктом.

Описание:
+ `Процедура СохранитьВКонтекст(Ключ, Значение)` - сохранить значение по специальному ключу
+ `Функция ПолучитьИзКонтекста(Знач Ключ)` - возвращает значение по ключу 

Пример кода указан выше, в разделе `Пример файла шагов`

## Программный вызов любого шага сценария

Из кода скрипта-реализации шагов фичи можно вызывать любой известный и доступный шаг.
При этом учитываются параметры, переданные в тексте шага 

Сигнатура вызова: `БДД.ВыполнитьШаг(Знач НаименованиеШагаСценария)`

Пример кода:
```
НаименованиеШагаСценария = "я записываю """ШагСценария""" в файл журнала";
БДД.ВыполнитьШаг(НаименованиеШагаСценария);
```
