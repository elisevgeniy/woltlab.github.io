---
title: Создание простого плагина
sidebar: sidebar
permalink: getting-started_quick-start.html
folder: getting-started
---

## Требования и настройки

Это руководство поможет вам создать простой плагин, создающий простую тестувую страницу.
В этом ничего сложного, но вы можете использовать его в качестве основы для своего
следующего проекта..

Есть некоторые требования, с которыми вам необходимо ознакомиться перед началом:

- Текстовый редактор с подсветкой синтаксиса для PHP, [Notepad++](https://notepad-plus-plus.org/) - это хороший выбор
 - `*.php` and `*.tpl` должны иметь кодировку ANSI/ASCII
 - `*.xml` всегда должны иметь кодировку UTF-8, но без BOM (byte-order-mark)
 - Используйте табы (tabs) вместо пробелов для отступов
 - Рекомендуется установить ширину таба (отступа) в «8» пробелов, т.к. эта ширина используется во всем программном обеспечении и облегчит чтение исходных файлов
- Установленный WoltLab Suite 3
- Приложение для создания архивов `*.tar`, например, [7-Zip](http://www.7-zip.org/) для Windows

## Файл package.xml

Мы хотим создать простую страницу, на которой будет отображаться предложение «Hello World»
в окне приложения. Создайте пустой каталог, в котором вы хотите начать работать.

Создайте новый файл с именем `package.xml` и вставьте следующий код:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<package name="com.example.test" xmlns="http://www.woltlab.com" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.woltlab.com http://www.woltlab.com/XSD/tornado/package.xsd">
	<packageinformation>
		<packagename>Simple Package</packagename>
		<packagedescription>A simple package to demonstrate the package system of WoltLab Suite Core</packagedescription>
		<version>1.0.0</version>
		<date>2016-11-27</date>
	</packageinformation>

	<authorinformation>
		<author>YOUR NAME</author>
		<authorurl>http://www.example.com</authorurl>
	</authorinformation>

	<requiredpackages>
		<requiredpackage minversion="3.0.0">com.woltlab.wcf</requiredpackage>
	</requiredpackages>

	<instructions type="install">
		<instruction type="file" />
		<instruction type="template" />

		<instruction type="page" />
	</instructions>
</package>
```

В документации есть [целая глава][package_package-xml] которая объясняет, что код выше
делает и как вы можете настроить его в соответствии с вашими потребностями. Но пока мы оставим его как есть.

## PHP класс

Следующий шаг - создать класс PHP, который будет обслуживать нашу страницу:

1. Создайте каталог `files` в том же каталоге, где находится` package.xml`
2. Откройте `files` и создайте каталог` lib`
3. Откройте `lib` и создайте каталог` page`
4. В директории `page`, создайте файл `TestPage.class.php`

Скопируйте и вставьте следующий код в `TestPage.class.php`:

```php
<?php
namespace wcf\page;
use wcf\system\WCF;

/**
 * A simple test page for demonstration purposes.
 *
 * @author	YOUR NAME
 * @license	GNU Lesser General Public License <http://opensource.org/licenses/lgpl-license.php>
 */
class TestPage extends AbstractPage {
	/**
	 * @var string
	 */
	protected $greet = '';

	/**
	 * @inheritDoc
	 */
	public function readParameters() {
		parent::readParameters();

		if (isset($_GET['greet'])) $this->greet = $_GET['greet'];
	}

	/**
	 * @inheritDoc
	 */
	public function readData() {
		parent::readData();

		if (empty($this->greet)) {
			$this->greet = 'World';
		}
	}

	/**
	 * @inheritDoc
	 */
	public function assignVariables() {
		parent::assignVariables();

		WCF::getTPL()->assign([
			'greet' => $this->greet
		]);
	}
}

```


Класс наследуется от [wcf\page\AbstractPage](https://github.com/WoltLab/WCF/blob/master/wcfsetup/install/files/lib/page/AbstractPage.class.php), обеспечивающий базовую реализацию страниц без элементов управления форм (without form controls: поля ввода, кнопки и т.д.). 
В нём определяется довольно много методов, которые будут автоматически вызываться в определенном порядке, например `readParameters()` перед `readData()` и, наконец, `assignVariables()` для передачи ваших значений в шаблон.

Свойство `$greet` определяется как `World`, но может быть дополнительно заполнено через переменную GET (`index.php?test/&greet=You` выводит `Hello You!`). Этот дополнительный код иллюстрирует разделение обработки данных
которое имеет место во всех видах страниц, где все предоставленные пользователем данные считываются одним методом (функцией). Это помогает организовать код, но более того - навязывает чистоту логики класса, которая не даёт 
считывать ввод данных в случайных местах, включая риск избежать ввода переменной `$_GET['foo']` 4 из 5 раз. (including the risk to only escape the input of variable `$_GET['foo']` 4 out of 5 times.)

Чтение и обработка данных являются только половиной истории, теперь нам нужен шаблон, чтобы отобразить фактическое содержимое для нашей страницы. 
Вам не нужно определять его самим, это будет сделано автоматически на основе вашего пространства имен и названия класса. Вы сможете [прочесть больше об этом позже] (#appendixTemplateGuessing).

И последнее, но не менее важное: вы не должны писать в конце закрывающий PHP-тег `?>`, так как это может привести к тому, что PHP будет разбиваться на области, а этого нам не нужно.

## Шаблон

Перейдите в корневой каталог вашего пакета, пока не увидите каталог `files` и `package.xml`. Теперь создайте каталог под названием `templates`, откройте его и создайте файл `test.tpl`.

```smarty
{include file='header'}

<div class="section">
	Hello {$greet}!
</div>

{include file='footer'}
```

Шаблоны представляют собой смесь HTML и Smarty-подобных шаблонов для преодоления статической природы необработанного HTML. В приведенном выше коде будет отображаться фраза «Hello World!» в окне приложения, или другая переданная фраза. Включенные (included) шаблоны `header` и` footer` отвечают за большинство функций общей страницы, но предлагают множество возможностей настройки, что позволяет влиять на их поведение и внешний вид.

## Определение страницы

Пакет теперь содержит класс PHP и соответствующий шаблон, но до сих пор отсутствует определение страницы. Создайте файл `page.xml` в корневом каталоге вашего проекта, таким образом, на том же уровне, что и` package.xml`.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<data xmlns="http://www.woltlab.com" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://www.woltlab.com http://www.woltlab.com/XSD/tornado/page.xsd">
	<import>
		<page identifier="com.example.test.Test">
			<controller>wcf\page\TestPage</controller>
			<name language="en">Test Page</name>
			<name language="ru">Тетовая страница</name>
			<pageType>system</pageType>
		</page>
	</import>
</data>
```

Вы можете использовать много возможностей при составлении шаблона страницы, включая логические вложенности и классы обработчиков для отображения в меню.

## Сборка плагина

Если вы внимательно следовали приведенным выше рекомендациям, каталог пакетов должен выглядеть следующим образом:

```
├── files
│   └── lib
│       ├── page
│       │   ├── TestPage.class.php
├── package.xml
├── page.xml
├── templates
│   └── test.tpl
```

Both files and templates are archive-based package components, that deploy their payload using tar archives rather than adding the raw files to the package file. Please create the archive `files.tar` and add the contents of the `files/*` directory, but not the directory `files/` itself. Repeat the same process for the `templates` directory, but this time with the file name `templates.tar`. Place both files in the root of your project.

Last but not least, create the package archive `com.example.test.tar` and add all the files listed below.

- `files.tar`
- `package.xml`
- `page.xml`
- `templates.tar`

The archive's filename can be anything you want, all though it is the general convention to use the package name itself for easier recognition.

## Installation

Open the Administration Control Panel and navigate to `Configuration > Packages > Install Package`, click on `Upload Package` and select the file `com.example.test.tar` from your disk. Follow the on-screen instructions until it has been successfully installed.

Open a new browser tab and navigate to your newly created page. If WoltLab Suite is installed at `https://example.com/wsc/`, then the URL should read `https://example.com/wsc/index.php?test/`.

Congratulations, you have just created your first package!

## Developer Tools

{% include callout.html content="This feature is available with WoltLab Suite 3.1 or newer only." type="warning" %}

The developer tools provide an interface to synchronize the data of an installed package with a bare repository on the local disk. You can re-import most PIPs at any time and have the changes applied without crafting a manual update. This process simulates a regular package update with a single PIP only, and resets the cache after the import has been completed.

### Registering a Project

Projects require the absolute path to the package directory, that is, the directory where it can find the `package.xml`. It is not required to install an package to register it as a project, but you have to install it in order to work with it. It does not install the package by itself!

There is a special button on the project list that allows for a mass-import of projects based on a search path. Each direct child directory of the provided path will be tested and projects created this way will use the identifier extracted from the `package.xml`.

### Synchronizing

The install instructions in the `package.xml` are ignored when offering the PIP imports, the detection works entirely based on the default filename for each PIP. On top of that, only PIPs that implement the interface `wcf\system\devtools\pip\IIdempotentPackageInstallationPlugin` are valid for import, as it indicates that importing the PIP multiple times will have no side-effects and that the result is deterministic regardless of the number of times it has been imported.

Some built-in PIPs, such as `sql` or `script`, do not qualify for this step and remain unavailable at all times. However, you can still craft and perform an actual package update to have these PIPs executed.

## Appendix

### Template Guessing {#appendixTemplateGuessing}

The class name including the namespace is used to automatically determine the path to the template and its name. The example above used the page class name `wcf\page\TestPage` that is then split into four distinct parts:

1. `wcf`, the internal abbreviation of WoltLab Suite Core (previously known as WoltLab Community Framework)
2. `\page\` (ignored)
3. `Test`, the actual name that is used for both the template and the URL
4. `Page` (page type, ignored)

The fragments `1.` and `3.` from above are used to construct the path to the template: `<installDirOfWSC>/templates/test.tpl` (the first letter of `Test` is being converted to lower-case).

{% include links.html %}
