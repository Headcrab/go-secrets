**Смешивание типов получателей**

Можно ли смешивать типы получателей, например, в структуре, содержащей не- сколько методов, где некоторые содержат получатели указателя, а другие — по- лучатели значения? Общее мнение таково, что это следует запретить. Но в стан- дартной библиотеке есть контрпримеры, например time.Time.

Разработчики хотели обеспечить неизменяемость структуры time.Time. Сле- довательно, большинство методов, таких как After, IsZero и UTC, имеют полу- чатели значения. Но для совместимости с существующими интерфейсами — encoding.TextUnmarshaler — структура time.Time должна реализовать метод UnmarshalBinary([]byte)error, который изменяет получатель, задаваемый байтовым срезом. Этот метод имеет получатель указателя.

В целом следует избегать смешивания типов получателей, но это не запрещено в 100 % случаев.

```go
package main

type IA interface {
	B()
	C()
}

type A struct{}

func (A) B()  {}
func (*A) C() {}

func main() {
	var a IA
	a = A{} // cannot use A{} (value of type A) as IA value
	_ = a
}
```

Если будешь смешивать приёмники, то так не сделаешь:

```go
package main

type IA interface {
	B()
	C()
}

type A struct{}

func (A) B()  {}
func (A) C() {}

type PA struct{}

func (*PA) B() {}
func (*PA) C() {}

func main() {
	var a, pa IA
	a = A{}
	pa = &PA{}
	_ = a
	_ = pa
}
```

Хотя можно обмануть судьбу разделением интерфейсов 🙃

```go
package main

type IA interface {
	C()
}

type IB interface {
	D()
}

type A struct{}

func (A) C()  {}
func (*A) D() {}

func main() {
	var a IA
	a = A{}
	var pb IB
	pb = &A{}
	_ = a
	_ = pb
}
```
