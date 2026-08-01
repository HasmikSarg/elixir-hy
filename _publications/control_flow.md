---
layout: publication
title: "Պայմանական ճյուղավորում՝ case, cond և if"
description: "Պայմանական ճյուղավորումը Elixir-ում։ case, cond և if կառուցվածքների կիրառությունը"
lang: hy
date: 2026-07-30 14:51:47 +0400
---

## Պայմանական ճյուղավորում՝ case, cond և if

Այս հոդվածում կդիտարկենք `case`, `cond` և `if` կառուցվածքները։

### case

`case`-ը համեմատում է արժեքը մի քանի ճյուղերի հետ, մինչև գտնի համընկնողը։

Ենթադրենք՝ ունենք վճարման ֆունկցիա, որը վերադարձնում է `{:ok, ...}` կամ `{:error, ...}`.

```elixir
defmodule Payment do
  def process(balance, amount) when amount > balance do
    {:error, :insufficient_funds}
  end

  def process(balance, amount) do
    {:ok, balance - amount}
  end
end
```

```elixir
iex> Payment.process(10000, 3000)
{:ok, 7000}
iex> Payment.process(10000, 50000)
{:error, :insufficient_funds}
```

Այժմ հայտարարենք ֆունկցիա, որը վճարումից հետո վերադարձնում է մնացած գումարը.

```elixir
defmodule Wallet do
  def pay(balance, amount) do
    case Payment.process(balance, amount) do
      {:ok, new_balance} -> new_balance
      {:error, :insufficient_funds} -> balance
    end
  end
end
```

```elixir
iex> Wallet.pay(10000, 3000)
7000
iex> Wallet.pay(10000, 50000)
10000
```

Առաջին ճյուղը ոչ միայն ստուգում է արժեքը, այլև վերադարձնում է այն `new_balance` անվան տակ։ Ի դեպ այդ անունը գոյություն ունի միայն իր ճյուղի ներսում։

Եթե անհրաժեշտ է ճյուղ, որը համընկնում է ցանկացած արժեքի հետ, կիրառվում է `_`-ը՝ բոլորին համընկնող ճյուղը։ Այն գրվում է վերջում, քանի որ իրենից հետո գրված ճյուղերը երբեք չեն ստուգվի.

```elixir
def pay(balance, amount) do
  case Payment.process(balance, amount) do
    {:ok, new_balance} -> new_balance
    _ -> balance
  end
end
```

Արդեն գոյություն ունեցող փոփոխականի հետ համեմատելու համար անհրաժեշտ է `^` operator-ը։ Առանց դրա ճյուղում գրված անունը նոր կապ է ստեղծում և համընկնում ցանկացած արժեքի հետ.

```elixir
defmodule Order do
  def matches_status?(order_status, expected_status) do
    case order_status do
      ^expected_status -> true
      _ -> false
    end
  end
end
```

```elixir
iex> Order.matches_status?(:paid, :paid)
true
iex> Order.matches_status?(:pending, :paid)
false
```

Ճյուղերին կարելի է ավելացնել լրացուցիչ պայմաններ՝ guard-երի միջոցով.

```elixir
defmodule Receipt do
  def label(balance, amount) do
    case Payment.process(balance, amount) do
      {:ok, new_balance} when new_balance < 1000 -> "low"
      {:ok, _new_balance} -> "ok"
      {:error, _reason} -> "failed"
    end
  end
end
```

```elixir
iex> Receipt.label(10000, 9500)
"low"
iex> Receipt.label(10000, 3000)
"ok"
iex> Receipt.label(10000, 50000)
"failed"
```

Առաջին երկու ճյուղերի կառուցվածքը նույնն է և տարբերվում է միայն guard-ով, ուստի հերթականությունը կարևոր է՝ ավելի խիստ ճյուղը գրվում է վերևում։

Կարևոր առանձնահատկություն. guard-ի ներսում առաջացած սխալը պարզապես ձախողում է տվյալ ճյուղը, և ստուգումն անցնում է հաջորդին.

```elixir
defmodule Cart do
  def first_item_label(items) do
    case items do
      list when hd(list) > 1000 -> "expensive"
      _ -> "normal"
    end
  end
end
```

```elixir
iex> Cart.first_item_label([2000, 300])
"expensive"
iex> Cart.first_item_label([500])
"normal"
iex> Cart.first_item_label([])
"normal"
```

Դատարկ ցուցակի դեպքում `hd([])` ֆունկցիայի կանչը սխալ կառաջացներ։ Guard-ի ներսում այն միայն ձախողում է առաջին ճյուղը, և ֆունկցիան աշխատում է առանց ընդհատման։

Եթե ոչ մի ճյուղ չի համընկնում, գործարկման ժամանակ առաջանում է `CaseClauseError` սխալը.

```elixir
iex> order_status = :pending
:pending
iex> case order_status do
...>   :paid -> "done"
...>   :failed -> "failed"
...> end
** (CaseClauseError) no case clause matching: :pending
```

Guard-երում օգտագործելու համար թույլատրվող բոլոր ֆունկցիաների ցանկը բերված է `Kernel` module-ի փաստաթղթերում։

### if

`case`-ը կառուցված է pattern matching-ի և guard-երի վրա, իսկ դրանք սահմանափակված են արտահայտությունների որոշակի շարքով, որոնք օպտիմալացված են կոմպիլյատորի կողմից։ Երբ պայմանը դուրս է այդ շարքից, կիրառվում է `if`-ը։

Ենթադրենք՝ խանութն ունի առաքման հետևյալ կանոնը. եթե պատվերի գումարը 10000 դրամ է կամ ավելի, առաքումն անվճար է։ Հակառակ դեպքում առաքման արժեքը 500 դրամ է։

`Delivery` module-ը պարունակում է մեկ ֆունկցիա՝ `price/1`։ Այն ստանում է պատվերի ընդհանուր գումարը և վերադարձնում առաքման արժեքը.

```elixir
defmodule Delivery do
  def price(total) do
    if total >= 10000 do
      0
    else
      500
    end
  end
end
```

```elixir
iex> Delivery.price(12000)
0
iex> Delivery.price(3000)
500
```

Առաջին կանչում պատվերի գումարը գերազանցում է շեմը, ուստի առաքման արժեքը `0` է։ Երկրորդում՝ `500`։

Այժմ դիտարկենք այլ խնդիր։ Պատվերի էջում պետք է ցուցադրվի «անվճար առաքում» հաղորդագրությունը, սակայն միայն այն ժամանակ, երբ պատվերը բավարարում է պայմանին։ Մնացած դեպքերում հաղորդագրություն ցուցադրելու կարիք չկա։

`Notice.free_delivery/1`-ը վերադարձնում է հաղորդագրության տեքստը կամ ոչինչ.

```elixir
defmodule Notice do
  def free_delivery(total) do
    if total >= 10000 do
      "free delivery"
    end
  end
end
```

```elixir
iex> Notice.free_delivery(12000)
"free delivery"
iex> Notice.free_delivery(3000)
nil
```

Երբ `else` ճյուղը բացակայում է, և պայմանը չի կատարվում, արդյունքը `nil` է։ Այս խնդրում `nil`-ը հենց այն է, ինչ անհրաժեշտ է՝ ցուցադրելու բան չկա։

Պայմանի տեղում կարող է լինել ոչ միայն համեմատություն, այլև ցանկացած արժեք։ Այդ դեպքում գործում է մեզ արդեն ծանոթ կանոնը՝ բացասական են համարվում `false`-ը և `nil`-ը, մնացած բոլոր արժեքները՝ դրական են։

Սա հատկապես օգտակար է, երբ արժեքը կարող է բացակայել.

```elixir
defmodule Profile do
  def greeting(user) do
    name = Map.get(user, :name)

    if name do
      name
    else
      "guest"
    end
  end
end
```

```elixir
iex> Profile.greeting(%{name: "Anna"})
"Anna"
iex> Profile.greeting(%{})
"guest"
```

Երկրորդ դեպքում `Map.get/2`-ը վերադարձնում է `nil`, ուստի կատարվում է `else` ճյուղը։

Կարճ պայմանների համար կա մեկտողանոց գրելաձև՝ արդեն ծանոթ `do:`-ի միջոցով.

```elixir
defmodule Delivery do
  def price(total) do
    if total >= 10000, do: 0, else: 500
  end
end
```

### Ամեն ինչ արտահայտություն է

Ծրագրավորման լեզուների մեծամասնությունը տարբերում են արտահայտությունները (կոդ, որը վերադարձնում է արժեք) և հրահանգները՝ statement-ները (կոդ, որը արժեք չի վերադարձնում)։ Elixir-ում կան միայն արտահայտություններ։ Կոդի ցանկացած կառուցվածք վերադարձնում է որևէ արժեք։

Java-ում, C#-ում կամ JS-ում `if`-ը հրահանգ է, իսկ արժեք ստանալու համար կիրառվում է "ternary" `condition ? a : b` օպերատորը։ Elixir-ում այդպիսի օպերատորի կարիք չկա, քանի որ `if`-ն ինքնին վերադարձնում է արժեք։ Վերևի օրինակներում հենց `if`-ի վերադարձրած արժեքն է դառնում ֆունկցիայի արդյունքը։

Այս հատկությունից բխում է ևս մեկ հետևանք. բլոկի ներսում ստեղծված կապը դուրս չի գալիս այդ բլոկից.

```elixir
iex> total = 3000
3000
iex> discount = 500
500
iex> if discount > 0 do
...>   total = total - discount
...> end
2500
iex> total
3000
```

`if`-ի արդյունքը `2500` է, սակայն բլոկից դուրս գտնվող `total`-ը մնում է `3000`։ Փոփոխությունը պահպանելու համար անհրաժեշտ է կապը ստեղծել բլոկից դուրս.

```elixir
iex> total =
...>   if discount > 0 do
...>     total - discount
...>   else
...>     total
...>   end
2500
iex> total
2500
```

Ուշադրություն դարձրեք, որ `else` ճյուղն այստեղ պարտադիր է։ Առանց դրա չկատարվող պայմանի դեպքում `total`-ը կդառնար `nil`։

### if-ը մակրո է

`if`-ը լեզվի հատուկ կառուցվածք չէ, ինչպես շատ այլ լեզուներում։ Այն մակրո է, որը վերածվում է `case`-ի։

### cond

`case`-ն ընտրում է մի քանի ճյուղերից՝ կախված արժեքից։ `if`-ը ստուգում է մեկ պայման։ Իսկ երբ անհրաժեշտ է ստուգել մի քանի տարբեր պայմաններ և գտնել առաջին համընկնողը, կիրառվում է `cond`-ը։

Ենթադրենք՝ զեղչի տոկոսը կախված է պատվերի գումարից.

```elixir
defmodule Discount do
  def rate(total) do
    cond do
      total >= 50000 -> 20
      total >= 20000 -> 10
      total >= 10000 -> 5
      true -> 0
    end
  end
end
```

```elixir
iex> Discount.rate(60000)
20
iex> Discount.rate(25000)
10
iex> Discount.rate(1000)
0
```

Java-ում, C#-ում և JavaScript-ում սրան համապատասխանում է `if`, `else if` շղթան։ Elixir-ում `cond`-ը:

Երբ ոչ մի պայման չի համընկնում, առաջանում է `CondClauseError` սխալը։ Դրա համար հաճախ վերջում ավելացվում է `true ->` ճյուղը, որը կատարում է `else`-ի դերը.

```elixir
iex> total = 1000
1000
iex> cond do
...>   total >= 50000 -> 20
...>   total >= 20000 -> 10
...> end
** (CondClauseError) no cond clause evaluated to a truthy value
```

Ինչպես և `if`-ում, `cond`-ի պայմանները նույնպես կարող են լինել ոչ միայն համեմատություններ, այլև ուղղակի արժեքներ.

```elixir
defmodule Contact do
  def channel(user) do
    cond do
      Map.get(user, :phone) -> "phone"
      Map.get(user, :email) -> "email"
      true -> "none"
    end
  end
end
```

```elixir
iex> Contact.channel(%{phone: "+37410000000"})
"phone"
iex> Contact.channel(%{email: "anna@example.com"})
"email"
iex> Contact.channel(%{})
"none"
```

### Ամփոփում

Elixir-ում ծրագրի ընթացքը պայմանից կախված կառավարելու համար նախապատվությունը տրվում է pattern matching-ին և guard-երին՝ ֆունկցիայի ճյուղերի կամ `case`-ի միջոցով, քանի որ դրանք ավելի հակիրճ և ճշգրիտ են։ Երբ տրամաբանությունը հնարավոր չէ արտահայտել ճյուղերով և guard-երով, կիրառվում է `if`-ը, իսկ մի քանի պայման ստուգելու դեպքում՝ `cond`-ը։

### Փորձեք ինքներդ

Բացեք `iex`-ը և կատարեք հետևյալ առաջադրանքները։

1. Ստեղծեք `Payment` և `Wallet` module-ները և կանչեք `Wallet.pay/2`-ը տարբեր արժեքներով։
2. `Wallet.pay/2`-ում փոխարինեք `{:error, :insufficient_funds}` ճյուղը `_`-ով և համոզվեք, որ արդյունքը չի փոխվում։
3. `Receipt.label/2`-ում փոխեք guard-ը այնպես, որ `"low"` վերադարձվի 5000-ից փոքր մնացորդի դեպքում։
4. Կանչեք `Cart.first_item_label([])`-ը և համոզվեք, որ guard-ի սխալը չի ընդհատում ֆունկցիան։
5. `Discount.rate/1`-ից հեռացրեք `true ->` ճյուղը և կանչեք այն `1000` արժեքով։ Դիտեք սխալը։
6. `Profile.greeting/1`-ը վերաշարադրեք `case`-ով՝ `nil` և մնացած դեպքերի համար առանձին ճյուղերով։

### Աղբյուրներ

- [Elixir — պաշտոնական կայք](https://elixir-lang.org)
