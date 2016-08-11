Title: Виправляємо Segfault при запуску Emacs в режимі демона
Date: 12/08/2016
Slug: emacs-daemon-fonts
Summary: Виправляємо причину, та налаштовуємо нормальні шрифти.
Category: Emacs
Lang: uk
Image: /images/segfault.png
Tags: emacs, bug, lisp, segfault

![Segfault](/images/segfault.png)
: Авторство: XKCD.com

Я давно перестав користуватись такою надзвичайно зручною штукою, як запуск Emacs в режимі демона, по
надзвичайно дивній для мене причині --- лінь було розбиратись у причинах segfault'у при запуску
демона. А причина 100% знаходилась в конфігах і нормального пояснення того, що може привести до
"випадання в осад" ніде якось не трапилось. Тому нічого розумнішого окрім як по черзі коментувати
рядки в `init.el`, щоб знайти можливу причину мені на думку не спало.

Розв'язок проблеми виявився простим, все злітало при задаванні шрифту:

```
:::emacs-lisp

(add-to-list 'default-frame-alist '(font . "Roboto Mono-9" ))
(set-face-attribute 'default t :font "Roboto Mono-9" )
(set-frame-font "Roboto Mono-9" nil t)

```

Не така велика біда подумав я, зараз виправимо:

```
:::emacs-lisp
(if window-system ;; Перевіряємо чи emacs запускається не в консольному режимі
    (progn
      (add-to-list 'default-frame-alist '(font . "Roboto Mono-9" ))
      (set-face-attribute 'default t :font "Roboto Mono-9" )
      (set-frame-font "Roboto Mono-9" nil t)))
```

Але тут виявилось, що при запуску `emacsclient -c` цей код виконуватись не буде, тобто треба щось
щоб при створенні нового фрейму задавало шрифт.

Великий і всемогутній StackOverFlow підказав мені
[відповідь](http://stackoverflow.com/questions/3984730/emacs-gui-with-emacs-daemon-not-loading-fonts-correctly),
і тепер повноцінне рішення виглядає так:

```
:::emacs-lisp

(if window-system ;; Перевіряємо чи emacs запускається не в консольному режимі
    (progn
      (add-to-list 'default-frame-alist '(font . "Roboto Mono-9" ))
      (set-face-attribute 'default t :font "Roboto Mono-9" )
      (set-frame-font "Roboto Mono-9" nil t))
  ;; Інакше задаємо шрифт іншим способом
  (setq default-frame-alist '((font . "Roboto Mono-9"))))

```

## P.S

Раніше не пам'ятаю вже навіщо я викладав відео яке демонструє, як працює Emacs в режимі демона:

<div class="embed-responsive embed-responsive-16by9">
<iframe class="embed-responsive-item" src="https://www.youtube.com/watch?v=01AvPRexOO0">
</iframe>
</div>

