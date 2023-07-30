

# Хакатон "ЛЦТ 2023". Задача 5. Поиск одинаковых товаров на маркетплейсе для компании OZON

## Стек:
`Python` `Pandas` `CatBoost`  `sklearn` `BertModel` `transformers`


## ПОСТАНОВКА ЗАДАЧИ

**Разработать ML–модель, которая по текстовым описаниям, атрибутам и картинкам - сможет ответить на вопрос, являются ли два товара одинаковыми.**

## КЛЮЧЕВЫЕ ЭТАПЫ РАБОТЫ
- Из текстовых описаний, с помощью предобученной модели bert768 были получены эмбеддинги размерностью 768
- Колонки "Цвет" и "Атрибуты" были также переведены в числовые (из формата словарей), с помощью специально, разработанных функций
- Из полученных колонок, а также из эмбеддингов картинок, предоставленных заказчиком, были сгенерированы дополнительные признаки. 
Основная идея при генерации новых фичей - считать расстояния (евклидово, косинусное, Левенштейна и др.) между признаками, чтобы выявить похожесть товаров.
- Выбор и настройка модели (был выбран CatBoostClassifier)

## РЕЗУЛЬТАТЫ

Наше решение вошло в топ-20 лучших.
---
---



## ДАННЫЕ

**OZONtech предоставил наборы данных:**
- train_pairs.parquet - тренировочная выборка с таргетом
- train_data.parquet - дополнительные данные для тренировочной выборки
- test_pairs_wo_target.parquet - тестовая выборка для участников без таргета
- test_data.parquet - дополнительные данные для тестовой выборки
- submission_example.csv - сабмит на бейзлайн



*Тренировочная выборка: пары одинаковых и различных товаров*

*Тестовая выборка: пары товаров без разметки (выборка для формирования лидерборда)*

*Дополнительные данные: названия, атрибуты, векторные представления картинок (эмбединги) товаров*

---

**Описание колонок из таблиц:**
- variantid - уникальный id товара
- name - название товара
- categories - категории товара
- color_parsed - цвета, которые удалось получить из атрибутов товара и названия
- embeddings - векторные представления товаров, получены с помощью resnet50 и Bert
- characteristic_attributes_mapping - атрибуты товаров



---

## РАБОТА В JUPYTER NOTEBOOK
---
**Раздел №1, введение, загрузка и знакомство с данными**

**В этом разделе производится:**

- загрузка нужных библиотек для работы
- загрузка исходных датасетов
- знакомство с данными

---
---

**Раздел №2, подготовка данных**

**В этом разделе производится:**

- 2.1 Создаем дополнительные колонки в тренировочной выборке с названиями категорий 3-го и 4-го уровня, также преобразование двумерных эмбеддингов в одномерные.
- 2.2 Сгруппируем категории, в которых меньше чем 1000 товаров и сменим у них название на "rest", затем закодируем их с помощью OrdinalEncoder()
- 2.3 Преобразование колонки с информацией о цвете товара, объединение похожих цветов одним названием и преобразование значений из строкового в цифровой вид.
- 2.4 Объединение датасетов в один, применим метод .merge() 2 раза по id каждого из пар товаров с соответствующими суффиксами. В итоге получаем датасет features с набором признаков по 2 товарам.
- 2.5 Создаем дополнительные признаки, чтобы выявить похожесть товаров: 
  - статистические характеристиками разности товаров, такие как разность между двумя эмбеддингами, среднее значение этой разности, медиану разности, стандартное отклонение разности. Применяем эту созданную функцию к эмбеддингам названий товаров, к эмбеддингам картинок товаров. 
  - Вычисляем коэффициент Жаккара для колонок с названиями и атрибутами товаров, и добавляем их в качестве новых признаков.
  - Вычисляем количество одинаковых значений в общих ключах, количество общих ключей, количество ключей с одинаковыми значениями, среднее значение числовых характеристик для первого и второго товара и добавим как новые признаки.
  - Вычисляем манхэттенское расстояние между эмбеддингами названий товаров и главными картинками.
  - Вычисляем среднее, медиану и стандартное отклонение для вычисленного манхеттонского расстояния, добавим как новые признаки.
  - Вычисляем расстояние Левенштейна для колонок с названиями товаров, добавляем его как новые признаки.
- 2.6 Вычислим косинусное и Евклидово расстояние для имеющихся эмбеддингов названий и главных картинок пар товаров, добавляем как новые признаки.
- 2.7 Удаляем колонки с данными, нужными для промежуточный вычислений.
- 2.8 Делим датасет с признаками на выборки, X_train, y_train, X_my_test, y_my_test. Отделяем 10% выборки чтобы получить промежуточный результат метрики сразу, по необходимости внести коррективы, дообучить в итоге на всей выборке.

---
---

**Раздел №3, работа с моделью**

**В этом разделе производится:**
- 3.1 Создание объектов Pool для работы модели CatBoostClassifier
- 3.2 Обучение модели на ранее подготовленной небольшой выборке и созданном объекте Pool, а также подбор гиперпараметров с кроссвалидацией с помощью RandomizedSearchCV.
- 3.3 Промежуточное сохранение модели.

---
---

**Раздел №4, Описание метрики PROC-AUC, метрику предоставил OZONtech**

Реализуем метрику в виде функции для получения промежуточных результатов (валидации).

---
---

**Раздел №5, промежуточный расчет PROC-AUC на отделенной ранее небольшой части выборки**

**В этом разделе производится:**
- расчет промежуточных значений таких метрик как:
  - PROC
  - AUC
  - Accuracy
  - F1
  - Precision
- построение таблицы важности признаков

---
---

**Раздел №6, дообучение модели**

**В этом разделе производится:**
- Создание объектов Pool для всей выборки.
- Дообучение модели на созданном объекте Pool

---
---

**Раздел №7, предобработка тестовой выборки, предсказание, подготовка сабмита**

**В этом разделе производится:**
- Подготовка тестовой выборки по аналогии с трейн выборкой в пункте №2:
  - Создаем дополнительные колонки в тренировочной выборке с названиями категорий 3-го и 4-го уровня, также преобразование двумерных эмбеддингов в одномерные.
  - Сгруппируем категории, в которых меньше чем 1000 товаров и сменим у них название на "rest", затем закодируем их с помощью OrdinalEncoder()
  - Объединение датасетов в один, применим метод .merge() 2 раза по id каждого из пар товаров с соответствующими суффиксами. В итоге получаем датасет features с набором признаков по 2 товарам.
  - Создаем дополнительные признаки, чтобы выявить похожесть товаров: 
    - статистические характеристиками разности товаров, такие как разность между двумя эмбеддингами, среднее значение этой разности, медиану разности, стандартное отклонение разности. Применяем эту созданную функцию к эмбеддингам названий товаров, к эмбеддингам картинок товаров. 
    - Вычисляем коэффициент Жаккара для колонок с названиями и атрибутами товаров, и добавляем их в качестве новых признаков.
    - Вычисляем манхэттенское расстояние между эмбеддингами названий товаров и главными картинками.
    - Вычисляем среднее, медиану и стандартное отклонение для вычисленного манхеттонского расстояния, добавим как новые признаки.
    - Вычисляем расстояние Левенштейна для колонок с названиями товаров, добавляем его как новые признаки.
  -  Вычислим косинусное и Евклидово расстояние для имеющихся эмбеддингов названий и главных картинок пар товаров, добавим как новые признаки.
  -  Удаляем колонки с данными, нужными для промежуточный вычислений.
- Создаем сабмит необходимого формата.

---
---


- Ссылка на скачивание весов модели: https://drive.google.com/file/d/1_473wLb1boSTLy3i_ww7Re-ZQUjjsYCt/view?usp=sharing
- Распаковываем в папку с проектом.
- Ноутбук для создания эмбеддингов для тестовой выборки: make-name-bert768-test.ipynb
- Ноутбук обработка тестовой выборки и сабмит: final_end_function_29038.ipynb
- Ссылка на скачивание готовых эмбедингов для тестовой выборки: https://drive.google.com/file/d/1a4eDk6Xo2KsGECOTkmBbICW_9Oor21Jf/view?usp=sharing


`
# hackathon_ozon
