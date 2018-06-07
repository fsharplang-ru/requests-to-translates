# Использование последовательности синтаксических анализаторов

Всякий раз, когда вам нужно применять несколько синтаксических анализаторов в последовательности, а нужен только результат одного из них, подходящая комбинация [`>>.`](http://www.quanttec.com/fparsec/reference/primitives.html#members.:62::62:..) и [`.>>`](http://www.quanttec.com/fparsec/reference/primitives.html#members...:62::62:) операторов поможет выполнить эту работу. Однако этих комбинаторов не хватит, если вам нужен результат более чем одного из задействованных синтаксических анализаторов. В этом случае вы можете использовать [`pipe2`](http://www.quanttec.com/fparsec/reference/primitives.html#members.pipe2), ..., [`pipe5`](http://www.quanttec.com/fparsec/reference/primitives.html#members.pipe5), которые последовательно применяют несколько синтаксических анализаторов и передают все отдельные результаты функции, которая вычисляет итоговый результат.

Например, с комбинатором [`pipe2`](http://www.quanttec.com/fparsec/reference/primitives.html#members.pipe2)
```fsharp
val pipe2: Parser<'a,'u> -> Parser<'b,'u> -> ('a -> b -> 'c) -> Parser<'c,'u>
```

Вы можете построить синтаксический анализатор `pipe2 p1 p2 f` который последовательно применяет два синтаксических анализатора `p1` и `p2`, а затем возвращает результат применения функции `f x1 x2`, где `x1` и `x2` - результаты, возвращаемые `p1` и `p2`.

В следующем примере мы используем [`pipe2`](http://www.quanttec.com/fparsec/reference/primitives.html#members.pipe2) для синтаксического анализа произведения из двух чисел:

```fsharp
let product = 
  pipe2 float_ws (str_ws "*" >>. float_ws) (fun x y -> x * y)
```

```fsharp
> test product "3 * 5";;
Success: 15.0
```

`pipe2-5` особенно полезны для построения объектов Абстрактного синтаксического дерева (далее АСТ). В следующем примере мы используем [`pipe3`](http://www.quanttec.com/fparsec/reference/primitives.html#members.pipe3) для анализа определения строковой константы в объекте `StringConstant`:

```fsharp
type StringConstant = StringConstant of string * string

let stringConstant = 
  pipe3 identifier (str_ws "=") stringLiteral (fun id _ str -> StringConstant(id, str))
```

```fsharp
> test stringConstant "myString = \"stringValue\"";;
Success: StringConstant ("myString","stringValue")
```

Если вы просто хотите вернуть проанализированные значения в виде кортежа, вы можете использовать предопределенные функции [`tuple2`](http://www.quanttec.com/fparsec/reference/primitives.html#members.tuple2),...,[`tuple5`](http://www.quanttec.com/fparsec/reference/primitives.html#members.tuple2). Например, `tuple2 p1 p2` эквивалентен  `pipe2 p1 p2 ( fun x1 x2 -> ( x1 , x2 ) )`.

Синтаксический анализатор [`tuple2`](http://www.quanttec.com/fparsec/reference/primitives.html#members.tuple2) также доступен в виде комбинатора [`.>>.`](http://www.quanttec.com/fparsec/reference/primitives.html#members...:62::62:..). Так что вы можете написать  `p1 .>>. p2` вместо `tuple2 p1 p2`. В следующем примере мы проанализируем пару разделенных запятыми чисел с этим оператором:

```fsharp
> test (float_ws .>>. (str_ws "," >>. float_ws)) "123, 456";;
Success: (123.0, 456.0)
```

Надеемся, что вы уже интуитивно понимаете шаблон записи `>>`*одна-или-две-точки*.

Если вам нужен синтаксический анализатор `pipe` или `tuple` более чем с пятью аргументами, вы можете легко построить его с помощью существующих. Например, вы хотите определить синтаксический анализатор `pipe7`.

```fsharp
let pipe7 p1 p2 p3 p4 p5 p6 p7 f =
    pipe4 p1 p2 p3 (tuple4 p4 p5 p6 p7)
          (fun x1 x2 x3 (x4, x5, x6, x7) -> f x1 x2 x3 x4 x5 x6 x7)
```