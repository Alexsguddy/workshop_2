# workshop_2
Мастерская 2  Мэтчинг товаров

Задача заключается в разработке модели, позволяющей определить пять товаров, наиболее близких к запрашиваемому.
Качество модели определяется метрикой accuracy@5 (также top5 accuracy). Модель для каждого запроса выдает 5 наиболее близких результатов. Если запрашиваемый результат попал в список из пяти ответов, то метрика положительна. Необходимо добиться наиболее высокого значения метрики.
Описание данных
1) base.csv - анонимизированный набор товаров. Каждый товар представлен как уникальный id (0-base, 1-base, 2-base) и вектор признаков размерностью 72.
2) train.csv - обучающий датасет. Каждая строчка - один товар, для которого известен уникальный id (0-query, 1-query, …) , вектор признаков И id товара из base.csv, который максимально похож на него (по мнению экспертов).
3) validation.csv - датасет с товарами (уникальный id и вектор признаков), для которых надо найти наиболее близкие товары из base.csv
4) validation_answer.csv - правильные ответы к предыдущему файлу
Объекты в train.csv и validation.csv это запросы, индексы объектов соответственно обозначены как запросы. В train.csv объекты размечены (то есть им найдены соответствия товаров из base). В validation.csv такой разметки нет. Но это разметка хранится в validation_answer.csv
Base.csv - список товаров, индексы которого обозначают сам товар.
По большому счету нужно разработать алгоритм, который будет наиболее точно подбирать 5 товаров из base, соответствующих запросу.

Вывод по работе

Задача мэтчинга была решениа с использованием библиотеки FAISS

Были проанализированы данные. Установлено, что 8 признаков из 72 имеют отличное от нормального распределение. Мы их в итоге выбросили из модели

Были опробованы разные скейлеры (standard, minmax, robust (c PowerTransformer с ходу не вышло)). Был выбран standard scaler.

Далее на уменьшенных выборках (500000 base, и по 1000 train и valid) было определено влияние количества кластеров и nprobe на метрику.

Установлено, что начиная с количества кластеров (n_cell) более 150 на малой выборке влияние на метрику практически отсутствует (тем не менее есть заметные колебания значения метрики).

Увеличение nprobe (или количество соседних опрашиваемых кластеров) дает почти непрерывный прирост к метрике. Однако негативно сказывается на времени вычисления, что в целом логично. На графике влияния nprobe на метрику видно, что скорость роста метрики меняется при nprobe = 0.05 % и 0.15% от количества кластеров (n_cells)

Оценка совокупного влияния n_cell и nprobe показала, что на уменьшенной выборке метрика падает в диапазоне от 200 до 300 кластеров вне зависимости от nprobe. А при большем количестве кластеров (600-800) метрика в меньшей степени зависит от nprobe.

В связи с этим для оценки метрики на полной выборке были выбраны n_cell = 600 и nprobe = 0.05 * n_cell = 30 Метрика accuracy@5 показала резльтат 70.51% и 70.43% на тренировочной и валидационной выборках соответственно
Уменьшение n_cell, ровно как и увеличение nprobe может привести к улучшению результата, но длительность расчетов может увеличиться в разы, а метрика при этом может вырасти на 0.5%, что в рамках данной задачи мне видится нецелесообразным. (отдельно была проверена модель с nprobe = 15 и n_cell = 100, расчет занял в 6 раз больше времени, а метрика выросла на с 70.5% до 70.8%).
