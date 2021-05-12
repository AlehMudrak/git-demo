Информация по [редактору Houdini](http://bit.ly/2PxCN7N). [Официальный сайт](https://www.sidefx.com/docs/houdini/). [книга HOUDINI ON THE SPOT на русском](http://houdinibook.ru/hou_on_spot/15-spot)

# Процесс создания анимации в гудини
1. Моделирование
	* Аттрибуты
	* Использование аттрибутов
	* VEX
	* Ноды геометрии
	* Ноды искажения
	* Математические действия
	* Циклы
	* Доп. расчеты
2. UV развертка на объекты
3. Материалы mat накладываем
4. Ставим камеру
5. Рендерим
6. Help
	* Управление
	* Термины
7. Примеры
	* python
	* Уроки Houdini

---
# Моделирование

## Attributes - стандартные аттрибуты. [HELP - point attributes](http://127.0.0.1:48626/copy/instanceattrs.html)
* pscale - размер объекта - коэффициент
* id - уникальный номер = есть у каждой точки/частицы
* age - время жизни в секундах на данный момент
* life - время жизни объекта
* P - Position = позиция точки в пространстве
* v (vertex) - вектор скорости
* force - сила (popforce = может влиять на v для изменения нелинейного изменения)
* Time - время
* N - вектор нормали
* Cd - цвет
* Alpha - видимость от 0 до 1
* $PR - значения по примитивам
* $F - Frame = кадр анимации
* $FSTART / $FEND = первый / последний кадр
* @ptnum - текущая точка

## Использование аттрибутов / Работа с аттрибутами
* attribrandomize = Attribete Randomize - слцчайным образом изменять какой-то аттрибут
	* Distribution
		* Attribute Name = pscale (например)
		* Dimensions - размерность (количество колонок, обычно достаточно одной)
	* Options
		* Seed Attribute - по какому параметру, к чему привязать
* attribcreate = Attribute create - создание аттрибутов
	* Name - имя аттрибута = PatchID
	* Class = Primitive - привяз.
	* Value - значение
* attribdelete - удаление аттрибута, например убрать цвет Cd
* attribwrangle - создание/подключение переменных и мат. вычисления над ними в VEX
	* нажать справа от VEXpression = create spare 
	* Run Over = Detail (only once) - для i[]@target=target; (смотри Geometry Spreadsheet => Detail)
* attribfrommap - достаем аттрибуты с карты/объекта
* polyframe - создает атрибуты координат для точек и вершин
	* Style - стиль
		* Two Edges - вычисление нормалей
		* Attribute Gradient - вычисление градиента
* **Attribute VOP = attribvop** - среда программирования, которая позволяет выполнять с объектами 
					разные операции = нужно заходить внутрь (тоже можно записать на VEX - языке программирования)
* attribnoise - работа с шумом, например для искажения Cd или P
* attribtransfer - передает аттрибуты
	* Attributes - выбираем какие аттрибуты
	* Conditions - настройки
		* Distance Threshold - расстояние
		* Blend Width - градиент / размытие
* attribdelete - удаление аттрибутов 

## [VEXpression](http://127.0.0.1:48626/expressions/) переменные / операторы / функции. [VEX Tutorial](https://github.com/jtomori/vex_tutorial)
*  // - комментируем строки так **//**
* Типы переменных
	* float - переменная с плавающей запятой (32 бит)
	* int - integer = int i - переменная i типа integer (+/-32000, целое 16 бит)
	* int text[] - массив text типа int
	* vector uv - переменная вектор с именем uv
* for (int i=0;i<npoints(0);i++)	 - вводим переменную i типа integer, цикл по входу 0 attribwrangle с шагом +1
	* { работа в цикле }
	* break; - выход из цикла
* xyzdist(1,@P,prim,uv) - функция = со входа 1 берем позицию P и передаем переменной prim [integer] и uv [vector]
* len(text()) - функция = длина массива text[]
* invert(orient) - функция = обратное/инвертированное значение переменной orient
* if (условие) { работа при выполнении условия} - если () - то { }
	* else { } - иначе {...}
	* == - равно
	* > / < - больше / меньше
* detail (0,"target") - инфа по точке. 0 - нулевой вход attribwrangle, "target" - из переменной target
* ch("anim") - параметр anim = появится при нажатии param справа сверху от кода, появится снизу под кодом
	* @P=fit01(anim,@P,point(1,"P",tg[@ptnum])) - сдвигание из одной позиции точки в другую
* fit01(a,b,c) - перемещение из точки b в точку c
* point(1,p,n) - позиция точки. На 1-первом входе attribwrangle, p - позиция, n - номер точки
* npoints(0) - количество точек на 0-нулевом входе attribwrangle
* prim(0,"textsymbol",i) - примитив со входа 0, textsymbol - его код, i - номер по порядку
* removepoint(0,@ptnum) - удалить точку @ptnum с 0-го входа 
* rand() - случайное число
* floor() - округление до целого числа вниз

---
# Ноды / Значения нетворков

## Работа с геометрией
* Null - пустой объект или контейнер, который обычно завершает геометрию из нескольких объектов
* Add - точка
* font - работа с текстом 
	* AddTextAttributes - ставим галку, чтобы управлять каждой буквой по textindex (номер символа) и textsymbol (код символа) = в примитивах
* Grid - земля
	* Size - размер
	* Rows / Columns - полигоны
* box / sphere / grid - примитивы коробка / сфера / тонкая поверхность
* sphere - сфера
	* Uniform Scale - диаметр сферы
* curve - кривая по точкам, которые ставишь сам
	* NURBS - плавное искревление
* L-System - деревья алгоритмическим способом
	* Шестеренка - пресеты
		* Dandelion - раскрывающийся парашют одуванчика
	* Values
		* Angle - угол раскрытия уже проанимирован
	* Geometry
		* Generations - выращивание дерева, например, если менять
		* Type
			* Skeleton - скелет
			* Tube - объемная фигура
			* Tube закладка
				* Rows - прорисовка веток
				* Thickness Scale - толщина веток
* Mountain - гора = рельеф искажает
	* Height - высота
	* Element Size - количество выпуклостей

## Ноды искажения, деформации, эффектов
* switch - переключатель = можно включать нужный вход
	* Select input = ch(opinputpath(".",1)+"/anim")>0 - проверяем что с переменной anim, если >0, то переключаем с 0го на 1й вход
* copytopoints - преобразование точки в фигуру *(на рпавый вход точку, слева - фигура)*
* Copy Stamp - соединение анимации точек с примитивами. Например делаем движение точек (справа),
				присоединяем сферу (слева) - получаем движение сфер по траектории точек
	* Stamp
		* Stamp Inputs = enable - включаем
		* Attribute Stamps = anim_in - наш аттрибут для увеличения/роста дерева
* Resemple - приведение отрезков кривых к одной длине / равномерное распределение точек
* Subdevide - дробление поверхности для сглаживания, например
* Polyextrude - полигональное выдавливание
* Fuse - склеивание точек
	* distance - расстояние до точки
* Transform - задание размера объекта = scale
	* uniform scale - соотношение/коэффициент
* Bend - искривление поверхности
* Merge - объединение
* Normal - этот узел вычисляет нормали (перпендикуляр) точки, вершины, примитива или детализации
* Smooth - сглаживание
* convert - конвертация, например в Polygon 
* minpos - находит ближайшую точку к поверхности (P = pos; OpInput **first context geometry** = input) = точка будет лежать на поверхности входящей геометрии
* ray - луч искажение
* measure - измерения или подсчет, например площади примитива
* scatter - точки
* color - установка цвета
* assemble - заполнение незаполненных островков (font) и запаковать в Create Packed Geometry
* pack - упаковка объектов как одна точка в центре эбъекта, например для букв в font через for
	* Transfer Attributes - перенос аттрибутов, например textsymbol из font, чтобы знать что код символа
* material - нода присвоения материала = удобно, когда в одной геометрии несколько объектов с разными материалами (см. материалы)
* carve - движение/искажение по точкам на входе / плавное движение по кривой curve

## Циклы
* For - EACH Named Primitive - цикл по примитивам
	* Method (Fetch Piece of points) - обработка каждого треугольник геометрии
	* For EACH END - ставим тот же метод в конце
		галка *Piece Attrib = PatchID* - аттриб

## Дополнительные рассчеты
* popnet (dop network) - система частиц
	* source_first_input - значения на входе ДопНетворка
		* Source
			* Emission Type = Points - рождаем новые частицы на точках
			* Gide - disable
		* Birth
			* Impulse Type - счет частиц в один фрэйм
			* Impulse Count - рождение количества частиц в один фрейм
			* Const.Activation - кол-во частиц в секунду
			* Const.Birth Rate - рождение количества частиц в ctreyle
			* Life Expectancy - время жизни частицы в секундах
			* Life Variance - предел randomize времени жизни
		* Attibutes
			* Initial Velocity = Set initial Velocity - скорость рождения
				* Velocity - скорость рождения
				* Variance - random до 1, например
* popvop - тот же attribvop, только внутри popnet
* dopimport - 

## Математические действия (применяются в attrvop/popvop = Attribute VOP)
* Add - сложить input1 + unput_n
* Divide - деление (на input подключаем переменные, div - результат)
* Multiply - умножение (на input подключаем переменные, product - результат)
* Multiply Constant - scale val * multiplier = например для уменьшения
* Length - подсчет длины, например вектора P (position)
* Ramp Parameter = ramp - ! после выхода из Attribute VOP настроить Ramp !
	* Ramp Type 
		* RGB Color Ramp - по цветам
		* Spline Ramp - по размеру
* Bind - подключение аттрибута
	* Name - имя аттрибута, например = pscale 
	* Type - должен совпадать тип переменной (float)
* Bind Export - возвращение аттрибута или присвоение новому аттрибуту (*anim_in*, например)
				(или вот: *умножили и после присвоили новое значение pscale*) 
* Fit Range = fit - пределы значений = мин/макс
	* val - значение на вход
	* srcmin - минимальное значение (начало анимации в секундах)
	* srcmax - максимальное значение (длина анимации в секундах)
			средней кнопкой на srcmax-вход = Promote Parameter - появится параметр в созданном Attribute VOP
		* Label - имя нового параметра видимое
		* Name - переменная нового параметра
* aanoise - шум
	* Signature = 4D Input, 3D Noise
	* pos - позиция
	* amp - амплитуда
* turbnoise - турбулентный шум
* Power = pow - возведение val в степень exp / или exp-нента от val
* popforce - добавление шума или искажение линейного движения =ставим в popnet после source_first_input, в Attributes = Initial Velocity = Set initional velocity - теперь ручками ставим силу  0 2 0, например
* pcopen - поиск точек в определенной области = на выходе handle
	* file (input) - сюда геометрию (Opinput1 = popvop/Inputs/Input1=FirstContextGeometry - геометрию, что прила на первый вход popnet)
	* Number of points - число точек, которые ищем
	* Search radius - радиус, в котором ищем точки
* pcfilter - усреднение аттрибутов (на входе handle)
* cross (cross Product) - изменение направления векторов

### Кватернионы (orient) - [для вращения объектов геометрии + работа с камерой](https://www.youtube.com/watch?v=IVZ2qxmUMMk)
* 

## Chopnet
* lag - плавное покачивание
	* Overshoot - сдвиг 0.2, например
* shift - сдвиг во времени
	* Scroll Offset - сдвиг
		* $C - номер канала = каждый канал сдвигается на 1 сек вперед
* reorder (Numeric Suffix Sort) - переопределение по каналам
	* до = tx0 tx1 tx2
	* после = tx0 ty0 tz0

## UV (UV координаты + их обработка) - текстура на поверхности
> Persp (вид) => Set View => UV viewport (**пробел + 5**) - просмотр UV
* uvquickshade - быстрый просмотр материала на поверхности, например для теста
* uvtransform - трансформация UV по x, y, z
* UVProject - на плоскость
* uvflatten - растягивание UV на поверхности
* uvlayout - автоматическое расположение кусков геометрии на UV

# Материалы mat
* используем ноду material в geo для присвоения материалов
* /mat
	* Material Pallet (закладка) - палитра материалов
		> В гудини один материал = Principal Shader
		> а ниже готовые пресеты
	* Перетягиваем Principal Shader направо, входим в него двойным кликом, после можно переименовать для удобства. Далее настраиваем по закладкам и параметрам
		* После настройки материала - перетягиваем его на нужную фигуру/текст
		* Увидеть какой материал применен к геометрии можно в = geo / render / Material
	* Закладки Principal Shader
		* Surface - поверхность
		* Opacity - прозрачность
		* Textures - текстуры
		* Bump & Normals - карта рельефов или карта нормалей
		* Displacement - работает с displacement

## Рендеринг = out
* mantra_ipr - просчет через мантру по умолчанию
	* Images / Output - Pixel Filter = denoise optix -a basecolor - для просчета через nVidia видеокарту
* obj / cam / View 
	* Resolution - разрешение картинки на выходе
	* 

---
# Help

## Управление
* горячие клавиши только в [EN] раскладке
* F1 - help  в программе
* пробел + G - все детали в кадр
* P - показать диалог параметров = удобно при широком экране в Geometry
* Правой клавишей по переменной анимированной + Delete channels - удаление анимации
* В окне View нажать d - display options
	* Guides закладка
		* Scale vectors - множитель векторов
* Alt + координата/Translate/Rotate/Radius... - начало анимации и на нужном кадре повторить = создание анимации
* Shift + click на параметр, например Rotate - открытие Animation Editor
* Средней кнопкой на вход в VOP (см. **Математические действия**) = Promote Parameter - этот параметр появится в настройках popVOP/popNet

### При наведении на ноду/геометрию
* Bypass - не просчитывать ноду/геометрию
* Display/Render - показать ноду/геометрию
* Node info - информация по ноде, ошибки...
* Lock - заблокировать ноду
* Template - 

### Анимация
* Animate editor - редактор анимации
	слева возле основного вида (*выбрать часы для реального времени*)
* Alt + left mouse - приминить анимацию (ключ на анимацию)

## Термины
* UV / [UV-преобразование](https://ru.wikipedia.org/wiki/UV-%D0%BF%D1%80%D0%B5%D0%BE%D0%B1%D1%80%D0%B0%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5) или развёртка в трёхмерной графике (англ. UV map) — соответствие между координатами на поверхности трёхмерного объекта (X, Y, Z) и координатами на текстуре (U, V).

---

# Примеры
* Добавление модели с материалами...

```
#!python

def wiki_rocks(text):
* [5*stamp("../copy2",animin,0)] - взятие переменной animin из copy2 = Copy Stamp
* normalize(cross(cross(@N,self),@N))
```
* Спираль (VEX)
>     float freq = ch('freq');
>     float rad = ch('rad');
>     float height = ch('height');
>     @P.x = sin(@P.y * freq)*rad*@P.y;
>     @P.z = cos(@P.y * freq)*rad*@P.y;
>     @P.y *= height;

* Центр примитивов (VEX)
>     int prim;
>     vector uv;
>     float dist = xyzdist(1,@P,prim,uv);
>     vector center = primuv(1,'P',prim,uv);
>     @P  =center;

* Движение по окружности
> x = x0 + R*cos(t+a)
> y = y0 + R*sin(t+a)
> где x, y - координаты
> x0, y0 - начальное положение на окружности
> t - время
> a - угловая скорость (угол)

## Уроки Houdini 
* [SideFX Digital Learning Material](https://docs.google.com/spreadsheets/d/11FbYBV_OV2INv3LCk38fmcgZbuVrgxYaZK-1KifCpyc/htmlview#)
* [Создание и анимация футбольного мяча](https://www.youtube.com/watch?v=EN4EUQo4CGE) Houdini 17 - Первый проект в Houdini (Часть 2 - Риг и Анимация) - CGScope	| [часть 3](https://www.youtube.com/watch?v=j-K2N4hCP78) UV (в начале) + [материалы + шейдеры](https://youtu.be/j-K2N4hCP78?t=920) + рендеринг (ближе к концу)
* [Урок Houdini "Морфинг текста"](https://www.youtube.com/watch?v=ASUYYiRd5dA) Используем VEX и Chop для реализации эффекта перетекания букв из одной фразы в другую.
* [Морфинг частиц](https://www.youtube.com/watch?v=2BVoXPq0cVA) рассматриваем основные концепции Гудини, работу в контекстах SOP и VOP.
* [Векторные поля](https://www.youtube.com/watch?v=IvKRl6ZpBtM) Урок об использовании кастомных полей велосити в системе частиц для создания абстрактных эффектов. + [3D векторные поля](https://www.youtube.com/watch?v=jjrGV511SNU) 
* [Морфинг текста](https://www.youtube.com/watch?v=ASUYYiRd5dA&t=47s) Используем VEX и Chop для реализации эффекта перетекания букв из одной фразы в другую.
* [Создание волн в море/океане с борашками](https://www.youtube.com/watch?v=gZoYCLUoJpM&list=PLs6LnJQNBzrF2F-tsxfKoiz9rnsPIeWEp) n this tutorial, you will learn how to use the Ocean Waves shelf tool, how the resulting network is designed to streamline your workflow and how to work with the new Ocean Surface Shader. 
* [Грязь разлетается от шин](https://www.youtube.com/watch?v=mvKY4pBdSGs) In this video tutorial, we go over the Grain Solver and show you how to make a sand simulation inside of Houdini 17 from SideFX.
* [Дождь в гудини](https://www.youtube.com/watch?v=A8txoPVNuYA) How to create Rain Systems and Wet Maps
* [Волосы / шерсть](https://www.youtube.com/watch?v=1bpVWg7CiTc) Hair Grooming and rendering. Houdini 17 0 1 + [Guide Groom](https://www.youtube.com/watch?v=jzfdsiQgJz8)
* [Деление клетки](https://www.youtube.com/watch?v=qzj9Sr_x0Lk) Vellum Cell Replication Tutorial Houdini 17 Part 1/2/3
* [Создаем Солнце](https://www.youtube.com/watch?v=ExUbAuK0B-s) How to create a total Sun in Houdini
* [Взрыв](https://www.youtube.com/watch?v=_eN_tn-h0oM) Houdini 17 или куда исчез Fluid Source. CG-SCHOOL.ORG
* [Работа с камерой](https://www.youtube.com/watch?v=IVZ2qxmUMMk) Что такое кватернионы, зачем они нужны и как ими пользоваться
* [Создание облаков вокруг планеты](https://www.youtube.com/watch?v=5rLWbDvba_Q) Houdini Volume Twist Tool (Hip File)
* [Город из карты с окнами / Ночной город](https://www.youtube.com/watch?v=QRcBCoZ7Y94) 1. Генерация процедурного города из карт OSM 2. Создание окон 3. Движение частиц 4. Рендер в Redshift3D 5. Компоузинг в AE
