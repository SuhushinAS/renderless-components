# Renderless Components в React. Как интегрировать неинтегрируемое?

## Вступление

###### intro
Привет DevPRO, представляю вам доклад "Renderless Components в React. Как интегрировать неинтегрируемое?"

###### about
Для начала - немного о себе: меня зовут Сухушин Александр. Я - Frontend-разработчик в компании **Userstory** (навести на ссылку).
Наша компания занимается проектированием и разработкой сайтов и комплексных информационных систем для автоматизации бизнеса,
а также решениями коммерческих и маркетинговых задач.

###### stack
Основоной наш frontend стек, это _React_, _Redux_, _ReactRouter_, _Babel_, _Flow_, _Webpack_, _ESLint_, _StyleLint_, _Prettier_, и ещё много другого.
3.1. В последнем проекте нам нужно было работать с картами, наиболее популярная библиотека - это _Leaflet_,
также понадобилось обновление данных через web-сокеты, для этого мы использовали библиотеку _Centrifuge_.
Основная проблема этих двух библиотек то, что у них нет реализации для _React_.

###### react−leaflet
На самом деле есть _React-Leaflet_, но нам нужно было использовать сторонние плагины и _React-Leaflet_ не давал нужной гибкости. Что же делать?

###### leaflet
Для начала давайте посмотрим, как работает сам _LeafLet_, заглянем в официальную документацию: Нужно создать `div`,
который будет выступать в качестве контейнера для карты, и вызываем метод `L.map()`, куда передать идентификатор контейнера или узел _DOM_.
Также, нужно спозиционировать карту и насторить Тайл-сервер (это сервер картинок для карты).

###### map
Мы можем реализовать это в React следующим образом: В методе render, мы срздаём `div` и сохраняем ссылку на него,
в `componentDidMount` мы создаём карту, а `componentWillUnmount` - удаляем. Так же через контекст мы передаём созданый экземпляр `leaflet`.

###### tile-layer
Тайл-сервер: `render` - `null`. На `componentDidMount`, мы добавляем слой тайл-сервера, на `componentWillUnmount` - удаляем.
На `componentDidUpdate` - удаляем старый и добавляем новый. добавление и удаление слоёв - это уже вызовы методов _Leaflet_,
которые можно посмотреть в документации.

###### geo-json
С позиционированием немного посложнее: Самый простой способ - это задать координаты центра и масштаб,
но по проекту требовалось позиционировать карту, чтобы у неё был максимальный масштаб, но при этом в экран попадали все объекты.
Объекты на картах обычно описываются в формате _GeoJSON_, это обычный _JSON_, с определённой структурой. Для примера, я описал несколько объектов:

###### map-view
В `componentDidMount` я вызываю метод `fly()`, который получает `view` - это объект или массив объектов в формате _GeoJSON_,
из `view` мы формируем слой для _Leaflet_ и получаем у него `bounds` - это минимальная прямоуголяная область, которая включает все объекты слоя.
B далее, если `bounds` - корректный - мы позиционируем карту на нём, иначе показываем весь мир.
В `componentDidUpdate` я проверяю, изменились ли `view` и `bounds` и вызываю метод `fly()`.
И также я добавил обработчик измнения позиционирования карты, который вызывает `onViewChange`, если он был передан в `props`.
С `onViewChange` у меня связана ещё одна проблема: если объектом будет прямоугольник,
то по логике его `bounds` должна равняться самому прямоугольнику, однако во вселенной _Leaflet_, к сожалению, - это не так.
И это вызывало глитчи карты в приложении. Поэтому я получаю `bounds` до тех пор пока следующие `bounds` не станут равны текущим.

###### map-usage
Давайте посмотрим пример, как это выглядит в использовании: Выводится компонент Map, а внутри него - `TileLayer`, `View`, и объекты на карте.

###### [map-example](https://suhushinas.github.io/2019-04-27_renderless-components-example/#/example-map)
И, наконец, посмотрим на результат: Здесь мы можем менять позиционирование карты, при этом фиксируется текущее положение.
Можем менять тайл сервер, например есть такой, очень прикольный. Также можем отображать на карте разные объекты: например, маркер,
где проходит **DevPRO**, маркер, где находится наш новый офис, и путь, между ними.

###### socket-centrifuge
Подобным образом, когда мы ничего не выводим, а просто вызываем действия на методах жизненного цикла, можно интегрирровать практически что угодно:
Например так я интегрировал сокеты, `componentDidMount` - я подключаюсь к серверу центрифуги,
`componentWillUnmount` - отключаюсь, и передаю экзепляр центрифуги через контекст дочерним элементам.

###### socket-subscribe
Аналогично - компонент для подписки `componentDidMount` - подписывемся, `componentWillUnmount` - отписываемся.
Во время подписки - регистрируем обработчик сообщений.

###### socket-usage
В использовании это выглядит вот так: Снаружи мы оборачиваем компонентов `Centrifuge`, куда передаём данные для подключения.
Внутри выводим компонент `Subscribe`, куда передаём название канала и обработчик сообщения.

###### [socket-example](https://suhushinas.github.io/2019-04-27_renderless-components-example/#/example-socket)
Я подготовил пример, как по сокетам мы будем получать координаты точки:

###### questions
На этом у меня всё, пожалуйста ваши вопросы.
