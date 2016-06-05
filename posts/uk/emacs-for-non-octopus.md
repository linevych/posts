Title: Emacs для не-восьминогів. Частина 1
Category: Emacs
Tags: emacs, ergoemacs, lisp, gnu, tutorial
Slug: emacs-for-non-octopus
Image: /images/octopus.jpg
Date: 21/04/2016
Summary: Чи справді Emacs був придуманий восьминогами?
          Як бути якщо в тебе немає щупалець?

## Зміст

[TOC]

## Короткий вступ

Мені давно хотілось якось об’єднати всі набуті знання про
[Emacs](https://uk.wikipedia.org/wiki/Emacs) в якусь статтю чи збірник заміток, тому я вирішив
зробити це в своєму блозі, думаю з часом я зможу доповнити і розширити деякі параграфи.

Зразу хотілось би сказати, що я не є спеціалістом, радше аматором, проте, на даний момент, в
інтернеті не так багато інформації на дану тематику, а тим паче українською. 


## Навіщо потрібен Emacs?

### 1. Прикладний редактор

Я використовую Emacs як прикладний редактор. Наприклад потрібно відредагувати конфіг NGINX на
[VDS](https://billing.ua-hosting.company/aff.php?aff=90){:rel=nofollow} через SSH, я просто ввожу
`/:ssh:user@server:/path/to/file` і натискаю `M-x nginx-mode` --- вуаля, в мене є підсвітка,
авто-доповнення і перевірка синтаксису. А взагалі цей файл можна додати в закладки для швидкого
доступу і відкривати в декілька натискань клавіш.

### 2. IDE для нових/рідкісних мов програмування чи розмітки

Редактор підтримує просто широченний спектр синтаксисів і їх діалектів (127
[ref][EmacsWiki --- Programming Modes](https://goo.gl/bgQtQb){rel:nofollow}[/ref] штук не рахуючи
сторонніх плагінів), тим паче ви можете створити підтримку власного або комбінувати декілька мов.

Власне я користуюсь ним для написання та редагування конспектів в Markdown.

**Не варто порівнювати Emacs з IDE для популярних мов програмування**, а особливо з продуктами від
JetBrains, це два різних класи інструментів.

## Чому я саме Emacs?

### 1. Гнучкість в налаштуванні

### 2. Тонна плагінів

Emacs має просто велетенську купу плагінів за допомогою яких можна вирішити широкий спектр завдань.
Існує навіть окремий ресурс, на якому автор публікує огляд роботи з плагінами, що допомагають робити
неймовірне --- [Emacsrocks](http://emacsrocks.com/){:target="_blank"}.

![Меню менеджера пакетів Emacs](/images/emacs-packages.png)
: Меню менеджера пакетів Emacs.

### 3. Open Source

На відміну від того ж Sublime Text, вам не доведеться платити за Emacs та і вихідний код, <s>в якому
ви все-одно ніфіга не зрозумієте</s>, можна правити і приймати безпосередню участь в
розробці.

Та і віддати $70 [ref] [Sublime Text --- Buy](https://www.sublimetext.com/buy){:rel=noindex}[/ref]
за ST (ми ж всі з вами чесні люди, чи не так?  :)) і користуватись ним як прикладним редактором ІМХО
якось не дуже.

### 4. PROFIT!

Один з прикладів демонстрації можливостей:

<div class="embed-responsive embed-responsive-16by9">
<iframe class="embed-responsive-item" src="https://www.youtube.com/embed/jNa3axo40qM">
</iframe>
</div>

## Де взяти Emacs?

TODO: Розказати де 

## Чому Emacs не надто популярний?

Кажуть, що у Emacs дуже високий поріг входження, це справді так, проте більшість складнощів
викликані тим, що комбінації клавіш є не звичними і ІМХО катастрофічно незручними. Тому якщо у вас
немає педалів до комп’ютера або щупалець як у восьминога користуватись цим редактором те ще
задоволення.

Справа в тому, що в 1976 році
[ref][Emacs --- Вікіпедія.](https://uk.wikipedia.org/wiki/Emacs){:rel=nofollow}[/ref] коли був
написаний Emacs клавіатура виглядала зовсім по іншому:

![Клавіатура Space-cadet](/images/emacs-keyboard.jpg)
: Клавіатура Space-cadet, одна з перших клавіатур для
[Lisp машин](https://wikipedia.org/){:rel=nofollow}.

## Конфігурація

За замовчуванням головний файл конфігурації знаходиться за адресою `~/.emacs`, але буде набагато
зручніше перенести його за адресою `~/.emacs.d/init.el`, оскільки всі інші файли (кеш, завантажені
плагіни, тощо) зберігаються саме тут.

### Пакети

Так як і більшість Linux-дистрибутивів Emacs має власну систему репозиторїв.

```
:::lisp
;; Підключаємо модуль керування пакетами
(require 'package)

;; Формуємо список джерел звідки будемо тягнути пакети
(setq package-archives '(("gnu" . "https://elpa.gnu.org/packages/")
                         ("marmalade" . "https://marmalade-repo.org/packages/")
                         ("melpa" . "https://melpa.org/packages/")
						 ("org" . "http://orgmode.org/elpa/")))
;; Завантажуємо і ініціалізуємо пакети
(package-initialize)
```

### Use-package --- грамотне керування пакетами


```
:::lisp
(unless (package-installed-p 'use-package)
  (package-install 'use-package))
  
(setq use-package-verbose t)
(require 'use-package)
```

### Ergoemacs --- збереже Ваші пальці

```
:::lisp
(use-package ergoemacs
  :pin gnu
  :init (ergoemacs-mode 1)
  :config (progn 
	    (setq ergoemacs-theme nil)
	    (setq ergoemacs-keyboard-layout "us")
	    (setq ergoemacs-message-level nil) ;; Disabling all debug messages.
	    (ergoemacs-mode 1)))
```

## Використані джерела
2. [Why Emacs's Keyboard Shortcuts are Painful](https://goo.gl/LDjHrl){:rel=nofollow}
3. [Почему Emacs?](http://goo.gl/c3NYy8){:rel=nofollow}

## Посилання
