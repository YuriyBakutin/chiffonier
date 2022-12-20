# Вариант схемы построения форм на Vue

_chiffonier - всё по полочкам_

Предлагаемая схема построения форм решает три проблемы.

1. Проблемы в шаблоне формы: Компонент формы является контейнером для компонентов-полей. С каждым полем связана своя функциональность и данные. Обычно в компоненте формы задаётся объект form, поля которого хранят значения соответствующих компонентов полей формы и связываются с ними конструкцией v-model. Кроме этого, компоненту поля могут требоваться дополнительные данные. Например, строка заголовок поля. Если компонент должен обеспечивать выбор из заданных вариантов, то требуется и список этих вариантов. Для целей последующего тестирования каждому элементу управления компонента должен назначаться уникальный id. Для этого на форме каждому компоненту назначается уникальный базовый id, который компонент использует, как префикс при назначении id своим управляющим элементам в шаблоне или рендер-функции. Также необходимо обеспечить валидацию ввода. Чаще всего для этого используется модуль Vuelidate, который в коде представлен объектом $v. Для него в скриптовой части компонента формы задаются правила валидации каждого компонента поля ввода. Значение поля ввода в этом случае передаётся не напрямую, а посредством обёртки специальным полем объекта $v. Объект, обеспечивающий поддержку этих правил, в частности, список строк с актуальными сообщениями о нарушении правил, передаётся компоненту через отдельное свойство. Также компоненту передаётся параметр, определяющий, пора ли показывать пользователю ошибки, которые он допустил при вводе.

То есть, на один компонент приходится как минимум пять свойств: значение поля, его отображаемое название, базовый id, и два свойства, связанные с валидацией. В коде шаблоне это семь строк, если учесть структуру тэга компонента. Из-за этого файл формы, имеющей несколько десятков полей, разрастается до огромного размера. Он теряет наглядность, и в нём становится труднее ориентироваться. Это составляет первую проблему.

2. Проблемы в скриптовой части: у всех форм есть обработчик события 'submit'. В нём выполняются однотипные действия: устанавливается признак того, что пользователю следует показать ошибки валидации, если они есть, выполняется валидация формы в целом, и если она не проходит, то обработчик завершает работу. Если валидация прошла успешно, выполняется сохранение данных, чаще по сети, но не всегда. На этом этапе могут возникать ошибки. Например, ошибки API. Их нужно обрабатывать. Поэтому блок кода, который сохраняет данные, обёрнут в конструкцию try — catch. При появлении ошибок на этом этапе, они перехватываются, их текст обрабатывается и выводятся пользователю. Чаще всего их текст выводятся в виде строки на форме, но может выводиться и как-то иначе, например, во всплывающем сообщении. Эта функциональность повторяется от формы к форме и различается только блоком сохранения данных и иногда блоком вывода текста ошибки. Перечисленные повторяющиеся действия наушают принцип DRY (Don't repeat yourself) и неоправданно увеличивают размер скриптовой части кода.

3. Проблема с одинаковыми полями (т.е. связанными с одним и тем же полем в модели приложения) на разных формах. Поскольку такие поля в коде никак не связаны, от формы к форме могут быть разночтения в написании названия и в установленных для этих полей правилах валидации. Если, например, нужно внести исправления в правила валидации такого поля, то нужно это делать на разных формах независимо, при этом легко можно допустить ошибки или разночтения. Здесь также нарушается принцип DRY.

Для решения этих проблем предлагается схема построения формы, основанная на трёх принципах: инкапсуляции ответственности, инжекции общих методов и декомпозиции.

Инкапсуляция предполагает создание отдельных уникальных компонентов для каждого поля, которое может встречаться на разных формах, но соотсетствует одному и тому же полю модели приложения, а также передачу всего того, что относится к этому полю — валидации, определения базового id, специфичных для этого поля статических данных и т.п. в сам этот компонент.

Инжекция призвана решить проблему нарушения принципа DRY. Несмотря на свою эффективность в деле избегания повторов, инжекция делает код запутанным и менее наглядным. В меньшей степени это относится к инжекции через наследование. Тем не менее, наследование тоже запутывает код. Чтобы уменьшить это зло, в предлагаемой схеме устанавливается чёткая схема наследования, как для компонента поля, так и для компонента формы. Для компонента поля есть три уровня наследования со строгим разграничением функций по этим уровням. Если ориентироваться на эту схему, то код становится понятным. Для компонента формы аналогично, только обязательных уровней наследования всего два. Кроме этих двух, есть третий необязательный, для случая, когда в разных местах используются близкие по составу полей формы. Хотя этот случай добавляет третий уровень наследования, сам по себе он с предлагаемой схемой не связан.

Для декомпозиции схема предлагает использовать компонент-контейнер субформу, которая позволяет какой-то блок формы вынести в отдельный компонент. Субформы размещаются на форме и аналогично простым полям обмениваются с ней данными с помощью конструкции v-model. Компоненты полей, размещённые в субформе также обмениваются с ней данными с помощью конструкции v-model. Субформы достаточно просто устроены и всегда имеют два уровня наследования. Основная функциональность корневого компонента SubForm является поддержка обмена данными по v-model, как с полями внутри, так и с внешней формой или субформой.

## Три уровня наследования для компонента поля формы

### Первый уровень — уровень корня

FormField — корневой компонент в схеме наследования компонентов полей форм. Он выполняет следующие функции:

1. Обрабатывает приходящие от компонента родительской формы события 'form-is-ready' и 'submitted'. В этом месте используется механизм пользовательских событий. Использование пользовательских событий в целом нежелательно, так как оно ненаглядно и запутывает код. Но это зло в данном случае устраняется тем, что такой канал строится лишь между двумя конкретными компонентами схемы. Он находится глубоко под капотом, не предполагает расширения и хорошо задокументирован.
2. В обработчике хука mounted (или по событию готовности формы 'form-is-ready', если попытка в хуке mounted прошла неудачно), Поиском по дереву DOM в сторону корня определяет элемент родительской формы, вешает на него обработчик события 'submitted'.
3. Определяет поле formId — id родительской формы.
4. Прослушивает и обрабатывает событие 'submitted' формы. По этому событию состояние поля showError_ изменяется с false на true, что используется в компонентах-наследниках от FormField для показа ошибки при ошибочном значении поля. Этот механизм является заменой для механизма задания этого значения через свойства, что уменьшает размер кода, освобождая его от очевидных однотипных стандартных конструкций.
5. Формирует базовый id компонента на основе formId и filedName. Базовый id компонента служит префиксом для id, расставляемых на управляющих элементах DOM компонента. Расстановка id необходима для обеспечения работы средств тестирования приложения через взаимодействие с DOM (Selenium)
6. Записывает пару ключ/значение filedName/label в структуру titlesByFiledNames (components/form/form-fields/common/Data/titlesByFiledNames.ts) Структура titlesByFiledNames используется в функции getErrorMessage (helpers/error.ts).	Она служит словарём для расшифровки сообщений об ошибках, возвращаемых API. Некоторые сообщения имеют вид <filedName>: <сообщение об ошибке значения данного поля>. В таком виде показывать пользователю это сообщение некорректно. filedName является служебным названием поля. Логичнее показывать пользователю соответствующий lable, но label не хранится на сервере, поэтому на клиенте нужен словарь. Можно создать на клиенте статический словарь, но тогда его придётся поддерживать и руками синхронизировать со всеми вносимыми изменениями, что является трудозатратным и чревато ошибками. Словарь titlesByFiledNames создаётся динамически во время монтирования формы в DOM, и не требует дополнительного внимания при изменениях в коде
7. Предоставляет метод updateValidation() для передачи информации о валидности ввода в поля формы на компонент формы. Механизм передачи использует структуру invalidIds (components/form/data/invalidIds.ts). Она имеет тип { [key: string]: string[] }, где в качестве key используются id формы, а в массиве хранятся id компонентов полей форм значения которых не проходит проверку на валидность. Методу updateValidation() передаётся логический параметр isValid. Если он истинный, id компонента с помощью метода removeItemToList() исключается из массива invalidIds, соответствующего форме, на которой размещено это поле. Если id данного компонента в массиве не было, то в нём ничего и не меняется. Если параметр isValid ложный, то id компонента с помощью метода addItemToList() добавляется в массив invalidIds, соответствующий форме. Если id данного компонента в массиве уже есть, то в нём ничего и не меняется. Во время выполнения метода onSubmit() формы, проверяется массив из invalidIds, соответствующий formId, и если он не пустой, форма не отправляется
8. В компоненте FormField размещён вотчер, отслеживающий изменения в объекте $v компонента. Как только изменение произошло, вызывается метод updateValidation() (см. предыдущий пункт). Это позволяет в компонентах-наследниках не беспокоиться о передаче данных о валидации в форму в случае использования Vuelidate. В компоненте FormField также определено поле noVuelidate, имеющее по умолчанию значение false. Переопределение этого поля в наследниках в true отключает вотчер на $v. Переопределение noVuelidate следует сделать, если в компоненте Vuelidate не используется.

9. При использовании Vuelidate в этот плагин необходимо передать объект применяемых правил. Это делается с помощью метода validations(), который размещён в параметре декоратора @Component модуля nuxt-property-decorator компонента FormField. Список правил берётся из поля validationRules, которое в FormField оставлено пустым и должно быть переопределено в наследниках. При передаче списка учитываются две особенности.

9.1. Первая — особая роль правила required. Один и тот же компонент поля формы может размещаться на разных формах. При этом, на одной из них он может быть обязательным, а на другой — нет. Поэтому для правила required в компоненте FormField есть специальное необязательное свойство isRequired. Значение этого свойства проверяется в методе validations(), и если оно истинно, то в список правил добавляется правлило required. Если оно ложно, то required не добавляется. По умолчанию isRequired истинно. Если нужно, чтобы поле не было обязательным, нужно передать ему свойство

```html
:is-required="false".
```

Остальные правила касаются корректности введённого значения. Они должны выполняться независимо от того, является ли поле обязательным, или нет. Список правил хранится в поле validationRules, которое в компоненте FormField определено пустым объектом, и может быть переопределён в дочерних компонентах, если требуются какие-либо дополнительные правила, кроме required. Само правило required в объект validationRules входить не должно.

9.2. Вторая особенность в том, что в объект применяемых правил нужно передавать имя переменной, которая должна проходить валидацию по указанным правилам. Имя переменной хранится в поле validatedVariableName, которое в компоненте FormField получает значение 'value', так как чаще всего в шаблоне нужно устанавливать валидацию на поле value. Если в компоненте-наследнике получается, что валидация требуется для другого поля, нужно переопределить validatedVariableName и инициализировать его строкой с нужным именем.

10. В обработчике хука mounted генерирует событие 'input', в payload которого посылается значение поля emptyValue. Это поле определяет значение, которым соответствующее поле формы должно быть инициализировано. В компоненте FormField значение этого поля установлено в null. Оно может быть переопределено в потомках. Такое поведение позволяет на лету формировать объект инициализации формы.

### Второй уровень — уровень шаблона (RenderField-компоненты)

Компоненты-наследники от FormField должны содержать шаблон или рендер-функцию, реализующую рендеринг компонента. Названия таких компонентов оканчиваются на 'RenderField'.

1. Также они должны содержать все необходимые свойства (Prop), и должны обеспечивать обмен данными с формой или субформой, с помощью v-model. Т.е., содержать свойство value и при изменениях значения поля генерировать событие 'input', содержащее в payload новое значение поля.

2. Также в них обеспечивается расстановка атрибутов id на всех элементах управления. Это может быть сделано явно в шаблоне. Но может оказаться так, что в шаблоне непосредственного доступа к элементу управления не будет. Так может быть, если используется компонент из сторонней библиотеки. В этом случае элементы управления могут быть найдены методом поиска по DOM. Конкретный id элемента строится по схеме: <Имя формы><Название поля><Суффикс>. Первая часть — <Имя формы><Название поля> хранится в поле id, наследуемом от FormField. Суффикс формируется путём добавления либо названия тэга элемента, либо как-то иначе, в зависимости от функциональности. Например, фукнцию кнопки может выполнять элемент div, на который пославлен @click. В этом случае логично этому div дать id с суффиксом 'Button'. Для элементов списков, получаемых по v-for, в начало суффикса добавляется 'Item' + index. Получается <Имя формы><Название поля>{'Item' + index}<Суффикс внутри элемента списка>.

3. В RenderField-компоненте должна быть реализована валидация. Валидация сводится к анализу допустимости значения поля. Если оно недопустимо, и признак showError имеет значение true, то компонент выделяется с помощью стилей и в нём появляется сообщение об ошибке. Есть три момента, которые требуют отдельного рассмотрения

3.1 Информация от том, что данные в поле формы не являются валидными должна передаваться в компонент формы, чтобы при submit данные не сохранялись, а выполнялся бы скроллинг к компоненту с ошибкой валидации. Для этого удобно использовать пакет Vuelidate, который подключён в качестве плагина. В компоненте FormField определен вотчер, который отслеживает изменения в объекте $v и вносит соответствующие изменения в структуры, обеспечивающие передачу информации о валидности полей компоненту формы (см п. 7 раздела FormField). Если для валидации в RenderField-компоненте используется Vuelidate, каких то дополнительных действий для передачи информации о валидности данных в компонент формы не требуется, т.к. эта функциональность реализована в FormField (см п. 7 раздела FormField). Но в каких-то сложных случаях использование Vuelidate может оказаться неудобным или невозможным. В этой ситуации валидация может быть реализована иначе. При этом, нужно позаботиться о том, чтобы данные о валидации также передавались в компонент формы, поскольку вотчер на $v в FormField срабатывать не будет. Для этого можно явно использовать унаследованный от FormField метод updateValidation() (см п. 6 раздела FormField).

3.2 При использовании Vuelidate, если требуются какие-то дополнительные правила валидации, кроме required, нужно переопределить поле validationRules и инициализировать его объектом, содержащим все требуемые правила, кроме required, которое в этот объект никогда не пишется (см п. 8.1 раздела FormField)

3.3 Если Vuelidate не используется, то необходимо специально позаботиться об учёте свойства isRequired, если оно актуально (см п. 8.1 раздела FormField). Также необходимо переопределить поле noVuelidate так чтобы оно инициализировалось значением true (см. п. 7 раздела FormField)

4. В RenderField-компоненте должны быть определены все поля, через которые компонент-наследник (Field-компонент) передаст в него какие-то данные. Например, если RenderField-компонент является списком (радиокнопок, чекбоксов и т.п.), и опции этого списка для конкретного поля заранее известны, т.е. являются статическими, то они размещаются в Field-компоненте в поле innerOptions. Если опции списка могут подгружаться динамически, то придётся использовать соответствующее свойство options, через которое данные будут получены от формы. На уровне RenderField-компонента нет однозначности, как будут получаться options, так как они могут поступать от разных компонентов полей. В этом случае можно сделать необязательное свойство options, пустое поле innerOptions, которое может быть переопределено в наследнике, а в шаблон выводить значение геттера mixOptions, полученного объединением options и innerOptions.

```javascript
@Prop({ type: Array, default: () => []}) readonly options?: any[]

innerOptions: any[] = []

get mixOption() {
	return [...option, ...innerOptions]
}
````

Если для Field-компонента структура списка будет динамической, то innerOptions в нём не переопределяется, а в шаблоне формы используется свойство options. При этом, если для другого Field-компонента структура списка статическая, то она задаётся в нём в innerOptions, а свойство options в шаблоне формы не используется.

### Третий уровень — уровень представительства.

Компоненты-представители (RepresentField-компоненты или просто Field-компоненты)

Каждое поле, которое может встречаться на разных формах, но связанное с одним и тем же полем модели данных приложения должно иметь свой уникальный компонент. Такие компоненты непосредственно размещается в шаблоне формы. Они, являются наследниками RenderField-компонентов. Названия таких компонентов оканчиваются на 'Field'. Они не содержат шаблона или рендер-функции и по сути являются компонентами-прослойками, хранящими какие-то специфичные для этого поля данные, например, строку подписи или опции выбора для компонентов выбора, если эти опции статичны. Компоненты-представители должны:

1. Переопределить поле fieldName, которое должно содержать строку с именем, совпадающим с именем в модели приложения (с учётом преобразования snake_case/camelCase). Оно используется для формирования id на элементах управления. Также оно используется для расшифровки текста ошибки, передаваемого по API (см п. 5 раздела FormField).

2. Переопределить поле label. Его значение используется в шаблоне RenderField-компонента. Также оно используется для расшифровки текста ошибки, передаваемого по API (см п. 5 раздела FormField).

3. Переопределять, когда это требуется, поля, заданные на уровне шаблона, таким способом передавая туда актуальные данные.


Таким образом, компонент поля в предлагаемой схеме представляет собой трёхуровневую структуру, FormField —> RenderField —> RepresentField. Если ориентироваться на эту структуру, то поддержка компонента не составит проблемы. Если нужно что-то изменить в данных, связанных с уникальным полем-представителем поля модели, например, название (fieldName), строку подписи, что-то в списке для выбора, если это select или checkboxBlock со статическим и т.п., заранее известным для данного поля списком выбора, то нужно редактировать компонент на RepresentField уровне. Если нужно изменить что-то в шаблоне, то нужно редактировать компонент на RenderField уровне, определив его по ссылке на родителя RepresentField компонента поля формы. Если нужно что-то из пунктов 1..9, описанных в разделе о FormField компоненте, то нужно редактировать FormField компонент. Предполагается, что его поведение отлажено настолько, что потребность во вмешательстве в его работу крайне маловероятна. Дополнительным удобством в поддержании этой схемы может служить использование в названии постфикса RenderField для имён компонентов уровня шаблона и просто Field для имён компонентов уровня представительства.

Для компонентов уровня представительства можно использовать средства, позволяющие Vue рассматривать их, как глобальные. Для Nuxt можно в файле nuxtConfig в объекте NuxtConfig в поле components указать список папок в которых будут размещаться эти компоненты. Для других систем существуют аналогичные средства. В этом случае эти компоненты можно будет размещать в шаблоне, не заботясь об их импорте и регистрации, что уменьшает размер кода формы, избавляя его от строк с рутинной информацией.


## Два уровня наследования для компонента формы, а также третий — необязательный

### Корневой уровень представленный компонентом BaseForm

BaseForm не содержит ни шаблона, ни рендер-функции. Выполняет следующие функции:

1. В обработчике хука created определяется имя крайнего по цепочке наследования компонента-потомка, это имя присваивается в качестве значения полю formId. Например, в цепочке наследования <BaseForm> —> <AircraftEditRenderForm> —> <AircraftInfo> formId получит значение 'AircraftInfo'. В непосредственном наследнике значение поля formId передаётся через Prop в компонент UIForm. В компоненте UIForm это свойство используется для постановки id на DOM-элементе form самой формы, а также на кнопках Submit и Cancel с добавлением соответствующих окончаний. id, установленный на DOM-элементе form, используется компонентами полей формы для определения своего поля formId. Эта функциональность реализована в компоненте FormField, от которого они все наследуются (см п. 2 раздела FormField).

2. В обработчике хука mounted инициализируется массив, соответствующий formId, в структуре invalidIds (см. п. 6 раздела FormField)

3. В некоторых случаях, при отрисовки формы на стороне клиента, данные необходимые для отрисовки, оказываются получены не сразу. В этом случае момент готовности формы оказывается не связан с хуком mounted. При этом могут возникать сбои при определении id формы компонентами полей. Поэтому в случае задержки своей отрисовки по её окончании форма генерирует событие 'form-is-ready', которое обрабатывается в компоненте FormField (см. п. 2 раздела FormField).

4. В BaseForm реализован метод onSubmit() в котором:

4.1 Генерируется событие 'submitted', обработчик которого реализован в компоненте FormField (см. п. 1 раздела FormField). 

4.2 Проводится проверка валидации путём анализа содержимого соответствующего массива структуры invalidIds. Если массив пустой, значит валидация прошла успешно. В противном сллучае из всех полей, не прошедших валидацию, выбирается то, которое расположено ближе всего к верхнему краю страницы, после чего к нему осуществляется скроллинг.

4.3. При успешной валидации внутри tray-блока перехвата ошибки вызывается метод saveData(), который в BaseForm() оставлен пустым, и должен быть переопределён в компоненте-наследнике. Если при сохранении данных произошла ошибка, она перехватывается в catch. Из полученного объекта ошибки извлекается строка с сообщением об ошибке, при необходимости корректируется, как описано в п. 5 раздела FormField, и передаётся в метод showError() в качестве параметра. В BaseForm этот метод не пустой. В нём строка с сообщением об ошибке присваивается полю errorMessage, которое в компоненте-наследнике может передаваться в одноимённое свойство компонента UIForm для непосредственного отображения на форме. Если в компоненте-наследнике нужно реализовать другое поведение, например, вывести сообщение об ошибке, как всплывающее, то метод showError() должен быть в нём переопределён.

### Уровень шаблона

Компонент формы уровня шаблона, прямой наследник от BaseForm. Содержит шаблон, включающий компоненты полей формы. Названия таких компонентов оканчиваются на 'RenderForm'. Шаблон (или рендер-функция) включает основной компонент UIForm. Внутри него размещаются компоненты полей ввода, у которых корневым предком в цепочке наследования служит FormField. Кроме Field-компонентов шаблон может включать компоненты-контейнеры, наследуемые
от BaseSubForm, позволяющие разбивать большие формы на отдельные компоненты в целях упрощения кода путём декомпозиции. Имена таких компонентов должны оканчиваться на 'SubForm'.

1. Field-компоненты и SubForm-компоненты обмениваются данными с формой с помощью конструкции v-model.

2. RenderForm-компонент не должен переопределять функцию onSubmit(), определённую в BaseForm. Смысл использования наследования от BaseForm при таком построении кода совершенно теряется. Кроме того, вряд ли это для чего-то может потребоваться.

3. RenderForm-компонент должен переопределить изначально пустую функцию-заглушку saveData(), наследуемую из BaseForm. Эта функция должна быть асинхронной для совместимости типов, даже когда асинхронность не требуется (например, когда данные сохраняются на клиенте).

Компонент уровня шаблона должен обеспечить вывод текста ошибки, возникшей при выполнении saveData(). Текст ошибки содержится в поле errorMessage, определённом в компоненте BaseForm. Достаточно передать его в качестве соответствующего параметра в компонент UIForm, чтобы обеспечить в случае ошибки простой вывод текста ошибки внизу формы. Если нужно реализовать какое-то иное поведение, следует переопределить наследуемый от BaseForm метод showError(), который получает обработанную строку ошибки в качестве параметра. Также в компоненте формы уровня шаблона должна быть обеспечена инициализация данных формы.

### Третий (необязательный) уровень наследования от BaseForm


Наследник компонента формы уровня шаблона. В приложении могут встретиться формы, близкие по содержанию, незначительно отличающиеся набором компонентов и получением / сохранением данных. Чтобы избежать повторения кода, такие формы можно наследовать от общего RenderForm-компонента. Для уточнения состава полей можно использовать конструкции вида:

```javascript
v-if="formId === <formId формы, для которой требуется данное поле>"
```

Для наследников RenderForm-компонента способы сохранения данных различны, в каждой них должны быть переопределен метод saveData(), а в самом RenderForm-компоненте, от которого они наследуются, метод saveData() определять не следует.


BaseSubForm — корневой компонент в схеме наследования субформ. Не содержит ни шаблона, ни рендер-функции. Субформы являются контейнерами для полей формы. Субформа взаимодействует с родительской по DOM формой или субформой через конструкцию v-model. Следовательно она должна иметь свойство value, представляющее из себя объект с полями, соответствующими значениям содержащихся в ней полей формы, а также если субформа содержит в себе другие субформы, то и поля-объекты, содержащие данные этих субформ. Также субформа при изменении данных вложенных полей должна генерировать событие 'input', в payload которого должен помещаться объект, содержащий текущие данные вложенных полей. В плане передачи данных от полей к форме и от формы к полям, субформа представляет из себя простую прокси-конструкцию. Изменение walue должно приводить передаче вложенным полям данных с формы, а изменение данных полей должно приводить к генерации события 'input'. С вложенными полями субформа взаимодействует с помощью конструкции v-model. Передавать в неё поля value нельзя, т.к при этом будут изменяться данные входящего свойства, что противоречит идеологии Vue, и будет вызывать ошибки. Для взаимодействия с вложенными полями в компоненте BaseSubForm есть поле form, в которое клонируется свойство value, когда в нём происходят изменения. Это обеспечивается вотчером, поставленым на value. При изменении данных вложенных полей сработает вотчер, установленный на form, и будет сгенерировано событие 'input'. После этого произойдёт изменение данных в родительской по DOM форме, изменится value и сработает соответствующий вотчер. Если не принять никаких дополнительных мер в коде, value будет клонирована в form, данные полей обновятся, и, несмотря на то, что их значения не изменились, вновь сработает вотчер, установленный на form, и процесс зациклится. Чтобы этого не произошло, в вотчер, поставленный на value, добавлена проверка на совпадение данных value и form. Проверка выполняется путём сравнением строк JSON. Такой проверки оказывается достаточно, поскольку form получается путём клонирования value, и беспокоиться о том, что у них может отличаться порядок расположения полей, не следует. Если оказывается, что данные в form и value одинаковые, то клонирование value в form не выполняется, и цикл разрывается. Таким образом, механизм проксирования данных от формы к полям и от полей к форме полностью реализован в BaseSubForm — как бы убран под капот.

## Субформы

Компонент-наследник от BaseSubForm (субформа) содержит шаблон, в котором помещены поля формы и/или другие субформы. В шаблоне для обмена данными с полями форм используются конструкции v-model="form.<имя параметра поля>", а для вложенных субформ v-model="form.<имя параметра вложенной субформы>". Кроме шаблона и импортов компонентов полей ввода и субформ, код субформы в простом случае может больше ничего не содержать. Поле form в субформе определять не нужно, т.к. оно наследуется из BaseSubForm. Если требуется реализовать какое-то взаимодействие компонентов полей формы, входящих в субформу, например, показ каких-то полей в зависимости от значения других полей, то код, обеспечивающий эту функциональность, следует размещать в субформе, а не выносить в форму.
