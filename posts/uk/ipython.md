Title: Розбираємось з IPython
Date: 21/06/2016 
Slug: ipython
Category: Python
Summary: IPython -- штука яка дозволяє послати Matlab лісом.
Tags: ipython, python
Image: /images/octopus.jpg

## Зміст

[TOC]

## Що таке IPython?

IPython [ref]IPython Documentation --- [Overview]( https://goo.gl/whnH8F){:rel=nofollow}[/ref] ---
середовище для інтерактивних дослідницьких обчислень яке включає в себе:

- Покращеного Python-shell
- Можливості інтерактивних паралельних обчислень

![Приклад побудови спектрограми в Jupyter Notebook](/images/ipy-notebook.png)
: Приклад побудови спектрограми в Jupyter Notebook. [ref]Microsoft Azure ---
[Jupyter Notebook on Azure](https://goo.gl/2YaJVP){:rel=nofollow}[/ref]

## Встановлення

### Приручаємо Anaconda

Особисто я **рекомендую** встановлення через Anaconda [ref] Continium Analytics --- Why Anaconda?
--- <https://www.continuum.io/why-anaconda>{:rel=noindex} [/ref] для того, щоб не було потреби в
докачуванні цілої купи необхідних для кожного чиху пакетів з pip.

Для початку качаємо дистрибутив з
[офіційного сайту](https://www.continuum.io/downloads#_unix){:rel=noindex} і запускаємо його:
```
:::bash
# Викликати BASH напряму ОБОВ'ЯЗКОВО

bash ./Anaconda3-*.sh

```

Встановлювач видасть умови ліцензії з якими треба погодитись, і запропонує шлях для встановлення,
для себе я встановив: `~/.anaconda`, оскільки мені не потрібно постійно бачити папку з анакондою.

Також встановлювач запропонує додати в `~/.bashrc` шлях до інсталяції Anaconda (PATH), але так як я
користуюсь ZSH замість Bash, тому я додавав в `~/.zshrc`:

```
:::bash

export PATH="/home/linevich/.anaconda/bin:$PATH"
```
Після цього треба відкрити нову сесію терміналу.

Тепер встановимо Jupyter --- веб додаток для роботи з IPython3, з ним автоматично підтягнуться всі
необхідні залежності:

``` 
:::bash
conda install jupiter
```

### Встановлення через pip [Альтернатива]

У випадку з Debian встановлення доволі просте, для початку треба встановити необхідні пакети:
```
:::bash
sudo apt-get update
sudo apt-get install -y ipython3 \
ipython3-notebook \
build-essential \
python3 \
python3-dev \
virtualenvwrapper
```

Тепер треба створити середовище де ми будемо гратись за нашим IPython'ом:

```
:::bash

mkdir python-demo && cd python-demo
mkvirtualenv ipython-demo --python=/usr/bin/python3
```

Далі встановимо IPykernel [ref][/ref] і Jupiter.

```
:::bash

pip install ipykernel jupiter
```


## Робота з IPython та Jupiter

Давайте запустимо jupiter:
```
:::bash

jupiter notebook

```
## Куди копати далі?

1. [Документація IPython [EN]](http://ipython.readthedocs.org/en/stable){:rel=noindex}
2. [Документація Jupyter [EN]](http://jupyter.readthedocs.org/en/latest){:rel=noindex}
3. Yandex Events ---
[Використання Python для інтерактивного аналізу даних [RU]](https://goo.gl/2T96u3){:rel=noindex}
4. [Шпаргалка по Anaconda[EN]](/pdf/conda-cheatsheet.pdf)

## Посилання
