# Case-study оптимизации

## Актуальная проблема
В нашем проекте возникла серьёзная проблема.

Необходимо было обработать файл с данными, чуть больше ста мегабайт.

У нас уже была программа на `ruby`, которая умела делать нужную обработку.

Она успешно работала на файлах размером пару мегабайт, но для большого файла она работала слишком долго, и не было понятно, закончит ли она вообще работу за какое-то разумное время.

Я решил исправить эту проблему, оптимизировав эту программу.

## Формирование метрики
Для того, чтобы понимать, дают ли мои изменения положительный эффект на быстродействие программы я решил 
использовать следующие метрики: ips(iteration per seconds) и realtime(скорость выполнения программы в ms) 

## Гарантия корректности работы оптимизированной программы
Программа поставлялась с тестом. Выполнение этого теста в фидбек-лупе позволяет не допустить изменения логики программы при оптимизации.

## Feedback-Loop
Для того, чтобы иметь возможность быстро проверять гипотезы я выстроил эффективный `feedback-loop`, который позволил мне получать обратную связь по эффективности сделанных изменений за 1 минуту. 

Вот как я построил `feedback_loop`: Бенчмарк на время выполнения, профайлинг с помощью *ruby-prof*.

## Вникаем в детали системы, чтобы найти главные точки роста
Для того, чтобы найти "точки роста" для оптимизации я пользовался, в основном, профайлером *ruby-prof* и *Benchmark realtime*

Вот какие проблемы удалось найти и решить

### Ваша находка №1
- rbprof/callgrind => collect_stats_from_users.
- Устранить квадратичную ассимптотику в блоках, использовать быстрые идиомы
- 90% времени - начал занимать Array#each
- Исправленная проблема перестала быть главной точкой роста

### Ваша находка №2
- rbrpof/callgrind => Array#each, слишком много вложенных циклов
- Сделать алгоритм линейным, использовать SortedSet для хранения уникальных браузеров. 
- как изменилась метрика
- Значительно уменьшилось количество вызовов Array#each

## Результаты
В результате проделанной оптимизации наконец удалось обработать файл с данными.
Удалось улучшить метрику системы с *невозможно дождаться завершения* до *19.5 секунд*

## Защита от регрессии производительности
Для защиты от потери достигнутого прогресса при дальнейших изменениях программы был написан тест на проверку линейной ассимпотики
и тест на скорость выполнения.
