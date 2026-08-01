---
layout: publication
title: "Struct տվյալների տիպը"
description: "Struct-երի ներածություն՝ ինչպես սահմանել տվյալների հստակ կառուցվածք, ստեղծել և թարմացնել արժեքները, կիրառել pattern matching-ը"
lang: hy
date: 2026-07-30 15:01:00 +0400
---

## Struct տվյալների տիպը

[Map-երի մասին հոդվածում](./maps.md) տեսանք, որ map-ը բանալի-արժեք զույգեր պահելու հիմնական կառուցվածքն է։ Այն ճկուն է․ կարելի է ավելացնել նոր բանալի, հեռացնել գոյություն ունեցողը կամ փոխել արժեքը։

```elixir
iex> product = %{name: "Keyboard", price: 25_000}
%{name: "Keyboard", price: 25000}

iex> product.name
"Keyboard"

iex> %{product | price: 23_000}
%{name: "Keyboard", price: 23000}
```

Սակայն ծրագրում հաճախ անհրաժեշտ է աշխատել նախապես հայտնի կառուցվածք ունեցող հավաքածուների հետ։ Օրինակ՝ ապրանքը պետք է ունենա անուն, գին և պահեստում առկա քանակ։ Եթե սխալմամբ գրենք `prcie`՝ `price`-ի փոխարեն, սովորական map-ը կարող է այդ սխալ բանալին ընդունել։

Struct-ը map-ի հենքի վրա կառուցված տվյալների տիպ է։ Այն նախապես սահմանում է թույլատրելի դաշտերը, կարող է ունենալ նախնական արժեքներ և օգնում է սխալ բանալիները հայտնաբերել struct-ը ստեղծելիս կամ թարմացնելիս։

### Struct-ի սահմանումը

Struct սահմանելու համար կիրառվում է `defstruct/1` կառուցվածքը։ Struct-ը սահմանվում է module-ի ներսում և ստանում է այդ module-ի անունը։

```elixir
defmodule Product do
  defstruct name: "Unknown", price: 0, stock: 0
end
```

`defstruct`-ին փոխանցված [keyword list-ը](./keyword_lists.md) սահմանում է struct-ի դաշտերն ու դրանց նախնական արժեքները։ Այստեղ ստեղծեցինք `Product` struct՝ `name`, `price` և `stock` դաշտերով։

Struct-ի արժեքը ստեղծվում է `%Module{}` գրելաձևով։

```elixir
iex> %Product{}
%Product{name: "Unknown", price: 0, stock: 0}

iex> %Product{name: "Mechanical Keyboard"}
%Product{name: "Mechanical Keyboard", price: 0, stock: 0}

iex> %Product{name: "Mechanical Keyboard", price: 25_000, stock: 12}
%Product{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

Չնշված դաշտերը ստանում են `defstruct`-ում սահմանված նախնական արժեքները։ Այդ պատճառով առաջին օրինակում բոլոր դաշտերը լրացվեցին ինքնաբերաբար։

### Միայն սահմանված դաշտերն են թույլատրված

Struct-ի հիմնական առավելություններից մեկը դաշտերի ստուգումն է։ Ստեղծելիս կարելի է նշել միայն `defstruct`-ում սահմանված դաշտերը։

```elixir
iex> %Product{name: "Keyboard", category: "Accessories"}
** (KeyError) key :category not found expanding struct: Product.__struct__/1
```

`category` դաշտը `Product` struct-ում սահմանված չէ, ուստի struct-ը չի ստեղծվում։ Այս ստուգումը թույլ է տալիս սխալները նկատել անմիջապես։ Սովորական map-ի դեպքում նույն բանալին պարզապես կավելացվեր։

Struct-ը հատկապես օգտակար է այն տվյալների համար, որոնց ձևը հայտնի է նախապես։ Օգտատերը, ապրանքը, պատվերը կամ վճարումը սովորաբար ունեն հստակ դաշտեր։ Դրանց ներկայացման համար struct-ն ավելի անվտանգ է, քան ազատ կառուցվածքով map-ը։

### Դաշտերի ընթերցումը

Struct-ի դաշտերը կարդացվում են atom բանալիներով map-երի կետային գրելաձևով։

```elixir
iex> product = %Product{name: "Mechanical Keyboard", price: 25_000, stock: 12}
%Product{name: "Mechanical Keyboard", price: 25000, stock: 12}

iex> product.name
"Mechanical Keyboard"

iex> product.stock
12
```

Չսահմանված դաշտին դիմելը բերում է `KeyError` սխալի։

```elixir
iex> product.category
** (KeyError) key :category not found in: %Product{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

Այս վարքը օգտակար է, քանի որ վրիպակը առաջացնում է սխալ հենց դիմելու պահին, չի վերածվում լուռ վերադարձված `nil` արժեքի, և դառնում սխալի պատճառ ծրագրի բոլորովին այլ կետում։

### Struct-ի թարմացումը

Գոյություն ունեցող դաշտը թարմացնելու համար կիրառվում է map-ից արդեն ծանոթ `|` գրելաձևը։

```elixir
iex> discounted = %{product | price: 22_500}
%Product{name: "Mechanical Keyboard", price: 22500, stock: 12}

iex> product
%Product{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

Սկզբնական `product` արժեքը մնաց անփոփոխ։ `%{product | price: 22_500}` արտահայտությունը ստեղծեց նոր struct։ Եվս մի անգամ հիշեցնենք, որ Elixir-ում տվյալներն անփոփոխ են։

Թարմացման ժամանակ նույնպես թույլատրված են միայն struct-ում սահմանված դաշտերը։

```elixir
iex> %{product | category: "Accessories"}
** (KeyError) key :category not found in: %Product{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

### Pattern matching-ը struct-ների հետ

Struct-ների հետ հարմար է կիրառել [pattern matching-ը](./pattern_matching.md)։ Դրանով կարելի է միաժամանակ ստուգել struct-ի տիպը և ստանալ անհրաժեշտ դաշտերի արժեքները։

```elixir
iex> %Product{name: product_name, stock: available} = product
%Product{name: "Mechanical Keyboard", price: 25000, stock: 12}

iex> product_name
"Mechanical Keyboard"

iex> available
12
```

Կաղապարում նշվում են միայն անհրաժեշտ դաշտերը։ Մնացած դաշտերը չեն խանգարում համապատասխանեցմանը։

`%Product{}` կաղապարը համընկնում է միայն `Product` struct-ի հետ։ Այն չի համընկնում սովորական map-ի կամ մեկ այլ struct-ի հետ։

```elixir
iex> %Product{} = %{name: "Mechanical Keyboard", price: 25_000, stock: 12}
** (MatchError) no match of right hand side value: %{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

Թեև աջ կողմի map-ն ունի նույն բանալիները, այն `Product` struct չէ։ Struct-ի տիպը տվյալների կառուցվածքի մաս է։

Այս հատկությունը հարմար է կիրառել բազմաճյուղ ֆունկցիաներում։ Յուրաքանչյուր ճյուղ կարող է ընդունել կոնկրետ struct-ի արժեք։

```elixir
defmodule Inventory do
  def status(%Product{stock: 0}), do: :out_of_stock
  def status(%Product{stock: stock}) when stock < 5, do: :low_stock
  def status(%Product{}), do: :available
end
```

```elixir
iex> Inventory.status(%Product{name: "Mouse", stock: 0})
:out_of_stock

iex> Inventory.status(%Product{name: "Keyboard", stock: 3})
:low_stock

iex> Inventory.status(%Product{name: "Monitor", stock: 10})
:available
```

Այս ֆունկցիան ոչ միայն ստուգում է `stock` դաշտը, այլ նաև երաշխավորում է, որ փոխանցված արժեքը `Product` struct է։

### Դինամիկ թարմացումներ `struct!/2`-ով

Նախորդ օրինակներում թարմացվող դաշտի անունը նախապես հայտնի էր։ Սակայն երբեմն փոփոխությունները ստացվում են keyword list-ի կամ map-ի տեսքով։ Օրինակ՝

```elixir
iex> updates = [price: 23_500, stock: 18]
[price: 23500, stock: 18]
```

Այդպիսի տվյալներով struct-ը թարմացնելու համար կիրառվում է `struct!/2` ֆունկցիան։

```elixir
iex> struct!(product, updates)
%Product{name: "Mechanical Keyboard", price: 23500, stock: 18}
```

`struct!/2`-ը որպես փոփոխություններ ընդունում է և՛ keyword list, և՛ map։ Այն պահպանում է struct-ի տիպը և ստուգում է դաշտերի անունները։ Չսահմանված դաշտի դեպքում այն առաջացնում է սխալ։

```elixir
iex> struct!(product, category: "Accessories")
** (KeyError) key :category not found in: %Product{name: "Mechanical Keyboard", price: 25000, stock: 12}
```

Գործնական կանոնը հետևյալն է։ Երբ դաշտերի անունները կոդը գրելիս հայտնի են, կիրառեք `%{struct | field: value}` գրելաձևը։ Երբ թարմացումները գալիս են keyword list-ից կամ map-ից, կիրառեք `struct!/2`։ Struct-ի ամբողջականությունը պահպանելու համար դինամիկ թարմացումների ժամանակ պետք չէ կիրառել `Map.put/3`։

### Struct-ը map է իր հիմքում

Struct-ը ներքին կառուցվածքով map է։ Դա կարելի է ստուգել `is_map/1` ֆունկցիայով։

```elixir
iex> is_map(product)
true
```

Յուրաքանչյուր struct ունի հատուկ `__struct__` դաշտ։ Դրա արժեքը struct-ը սահմանող module-ն է։

```elixir
iex> product.__struct__
Product
```
Սակայն struct-ը չպետք է դիտարկել որպես սովորական map և փոփոխել `Map` module-ի ցանկացած ֆունկցիայով։ Struct-ի համար նախատեսված գրելաձևերն ու `struct!/2`-ը պահպանում են սահմանված կառուցվածքը։

### Struct-ը չի ժառանգում map-ի բոլոր հնարավորությունները

Struct-ը map է իր հիմքում, բայց map-ի բոլոր գործողությունները դրա հետ հասանելի չեն։ Օրինակ՝ struct-ի դաշտը չի կարելի կարդալ `[]` գրելաձևով։

```elixir
iex> product[:name]
** (UndefinedFunctionError) function Product.fetch/2 is undefined (Product does not implement the Access behaviour)
```

Դաշտերը պետք է կարդալ կետային գրելաձևով՝ `product.name`։

Նաև `Enum` module-ի օգնությամբ մենք չենք կարող աշխատել `struct`-ի հետ․

```elixir
iex> Enum.count(product)
** (Protocol.UndefinedError) protocol Enumerable not implemented for %Product{name: "Mechanical Keyboard", price: 25000, stock: 12} of type Product (a struct)
```

`Enumerable`-ը ընդհանուր վարք սահմանող protocol է։ Protocol-ներին առանձին կանդրադառնանք հետագայում։ Կարևոր է հիշել, որ struct-ը չի կարելի անմիջապես փոխանցել `Enum.map/2`, `Enum.each/2` կամ `Enum.count/1` ֆունկցիաներին։

### Նախնական արժեքները

Եթե `defstruct`-ում դաշտի համար արժեք չի նշվում, դրա նախնական արժեքը դառնում է `nil`։

```elixir
defmodule Contact do
  defstruct [:name, :email]
end
```

```elixir
iex> %Contact{}
%Contact{email: nil, name: nil}
```

Կարելի է միավորել առանց բացահայտ արժեքի դաշտերն ու նախնական արժեք ունեցող դաշտերը։ Առանց արժեքի դաշտերը պետք է գրել առաջինը, իսկ keyword list-ի զույգերը՝ վերջում։

```elixir
defmodule Customer do
  defstruct [:email, name: "Unknown", active: true]
end
```

```elixir
iex> %Customer{}
%Customer{active: true, email: nil, name: "Unknown"}
```

Հակառակ հերթականությունը գրելաձևի սխալ է։ Keyword list-ի զույգերը list-ում միշտ պետք է լինեն վերջում։

```elixir
defmodule Customer do
  defstruct [name: "Unknown", active: true, :email]
end
```

```text
** (SyntaxError) unexpected expression after keyword list. Keyword lists must always come last in lists and maps.
```

Այս կանոնը բխում է [նախորդ հոդվածում](./keyword_lists.md)  դիտարկված keyword list-ի գրելաձևից։

### Պարտադիր դաշտերը `@enforce_keys`-ով

Երբ որոշ դաշտեր struct-ը ստեղծելիս պարտադիր են, կարելի է կիրառել `@enforce_keys` մոդուլի հատկությունը։ Այն module-ի սահմանման մեջ գրանցվող հատուկ արժեք է, որը compiler-ը կարդում է struct-ը սահմանելիս։ Այս պահին անհրաժեշտ է իմանալ միայն, որ այն գրվում է `defstruct`-ից առաջ և թվարկում է պարտադիր դաշտերը։ Մոդուլի հատկություններն ավելի մանրամասն մենք կդիտարկենք առաջիկայում։

```elixir
defmodule Order do
  @enforce_keys [:id, :customer_id]
  defstruct [:id, :customer_id, status: :new, total: 0]
end
```

Այժմ `Order` struct-ը հնարավոր չէ ստեղծել առանց `id` և `customer_id` դաշտերի։

```elixir
iex> %Order{}
** (ArgumentError) the following keys must also be given when building struct Order: [:id, :customer_id]
```

Պարտադիր դաշտերը փոխանցելու դեպքում struct-ը ստեղծվում է։

```elixir
iex> %Order{id: 1201, customer_id: 42}
%Order{customer_id: 42, id: 1201, status: :new, total: 0}
```

`@enforce_keys`-ը ստուգում է միայն դաշտի առկայությունը struct-ը ստեղծելիս։ Այն չի ստուգում արժեքի տիպը կամ բովանդակությունը։ Օրինակ՝ այն ինքնուրույն չի կարող ստուգել, որ `id`-ն դրական թիվ է կամ `customer_id`-ն `nil` չէ։ Այդպիսի կանոնները պետք է իրականացնել առանձին ֆունկցիաներով։

Այս պահանջը չի կիրառվում struct-ի հետագա թարմացումների ժամանակ։ Այն օգնում է սկզբնական արժեքը ճիշտ կառուցել, բայց արժեքների ստուգման ամբողջական ու վստահելի համակարգ չէ։

### Map, թե՞ struct

Սովորական map-ը ճիշտ ընտրություն է, երբ բանալիները կարող են ազատորեն ավելանալ կամ տվյալների ձևը նախապես հայտնի չէ։ Օրինակ՝ արտաքին JSON տվյալները հաճախ ներկայացվում են string բանալիներով map-երի տեսքով։

Struct-ը ճիշտ ընտրություն է, երբ ծրագրում կա հստակ հասկացություն՝ օգտատեր, ապրանք, պատվեր կամ վճարում։ Այդպիսի տվյալների դաշտերը նախապես հայտնի են, իսկ վրիպակը կամ ավելորդ բանալին սովորաբար ծրագրային սխալ է։

Հիշենք հիմնական առանձնահատկությունները

* Struct-ը սահմանվում է module-ի ներսում և ստանում է module-ի անունը։
* Դաշտերը նախապես սահմանված են `defstruct/1`-ով։
* Դաշտերը կարող են ունենալ նախնական արժեքներ։
* Չսահմանված դաշտը ստեղծելիս կամ թարմացնելիս առաջացնում է սխալ։
* Struct-ը կարելի է կիրառել pattern matching-ում՝ նաև դրա տիպը ստուգելու համար։
* Դինամիկ թարմացումների համար պետք է կիրառել `struct!/2`։
* `@enforce_keys`-ը կարող է որոշ դաշտեր պարտադիր դարձնել struct-ը ստեղծելիս, բայց արժեքները չի ստուգում։

### Փորձեք ինքներդ

Սահմանեք `Book` module՝ `title`, `author`, `price` և `available` դաշտերով։ `price`-ի նախնական արժեքը թող լինի `0`, իսկ `available`-ինը՝ `true`։ Ստեղծեք struct միայն վերնագրով և հեղինակով, ապա կարդացեք դրա դաշտերը կետային գրելաձևով։

Թարմացրեք գինը `%{book | price: ...}` գրելաձևով և ստուգեք, որ սկզբնական struct-ը մնացել է անփոփոխ։ Այնուհետև փորձեք ավելացնել չսահմանված `pages` դաշտ և դիտարկեք ստացված սխալը։

Ստեղծեք `[price: 4500, available: false]` keyword list և փոխանցեք այն `struct!/2`-ին։ Ավելացրեք չսահմանված բանալի և համոզվեք, որ `struct!/2`-ը բարձրացնում է `KeyError`։

Վերջում `@enforce_keys`-ով պարտադիր դարձրեք `title` և `author` դաշտերը։ Փորձեք ստեղծել struct առանց դրանցից մեկի, ապա ստեղծեք ամբողջական արժեք։
