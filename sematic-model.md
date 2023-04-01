# استنتاج مدل معنایی
زمانی که در فرایند تجزیه یک سند، گره‌های AST ساخته می‌شوند، به آن‌ها یک نوع اختصاص داده می‌شود. دستور زبان شکل این نوع و رابطه آن با سایر نوع‌ها را تعیین می‌کند و تمام انواع مدل معنایی زبان شما را تشکیل می‌دهند. دو روش برای استخراج انواع مدل معنایی زبان از دستور زبان وجود دارد که به ترتیب به صورت استنباطی و اعلانی عمل می‌کنند.


همچنین "Inference" یکی از رفتارهای پیش‌فرض در Langium است. در طی ایجاد انواع مدل‌های معنایی، Langium انواع ممکن را مستقیماً از قوانین گرامری استخراج می‌کند. این رویکرد برای زبان‌های ساده و نمونه‌ها بسیار قدرتمند است، اما برای زبان‌های بالغ‌تر توصیه نمی‌شود چرا که تغییرات کوچک در گرامر به سهولت می‌تواند منجر به تغییرات ناسازگار شود.


برای کاهش احتمال تغییرات ناسازگار، Langium انواع معرفی شده را با معرفی انواع اعلام شده (declared types) تعریف می‌کند که توسط کاربر در گرامر با استفاده از یک نحو مانند TypeScript صراحتاً تعریف می‌شوند

در ادامه، جزئیاتی راجع به اینکه چگونه قوانین گرامری با استفاده از استنتاج و اعلام، شکل مدل معنایی را تعیین می‌کنند، توضیح داده می‌شود

## انواع استنتاجی
انواع استنتاجی نتیجه گیری از استفاده Langium از نوع گذاری از قوانین گرامری هستند. بیایید به بررسی اینکه چگونه قوانین مختلف، تعریف این انواع را شکل می‌دهند، نگاهی بیاندازیم:

### قوانین پارسر

ساده‌ترین روش برای نوشتن یک قانون پارسر به شکل زیر است:
```antlr
X: name=ID;
```
با این دستور، Langium نوع گذاری را در زمان پارس کردن قانون استخراج خواهد کرد. به طور معمول، نوع گره بعد از نام قانون نامگذاری می‌شود، که باعث ایجاد این رابط TypeScript در مدل معنایی می‌شود:
```ts
interface X extends AstNode {
    name: string
}
```
همچنین، با استفاده از دستور زیر می‌توان نام رابط را کنترل کرد:
```antlr
X infers MyType: name=ID;
```
که در نتیجه، رابط زیر در مدل معنایی ساخته می‌شود:
```ts
interface MyType extends AstNode {
    name: string
}
```
لطفا توجه کنید که یک interface X دیگر در مدل معنایی وجود ندارد.

مهم است که بفهمید نام قانون بررسی‌کننده و نام نوعی که آن استخراج می‌کند، در دو سطح انتزاعی جداگانه کار می‌کنند. نام قانون بررسی‌کننده در سطح تجزیه استفاده می‌شود که در آن، انواع نادیده گرفته می‌شوند و تنها قانون بررسی‌کننده در نظر گرفته می‌شود، در حالی که نام نوع در سطح انواع استفاده می‌شود که در آن، هم نوع و هم قانون بررسی‌کننده نقش دارند. این بدان معناست که نام نوع می‌تواند بدون تأثیر بر سلسله مراتب قوانین بررسی‌کننده تغییر کند و نام قانون بررسی‌کننده نیز می‌تواند تغییر کند - اگر به طور صریح یک نوع خاص را بررسی یا برگرداند - بدون تأثیر بر مدل زبان برنامه نویسی.

با استخراج انواع داخل دستور زبان، امکان تعریف چندین قانون بررسی‌کننده برای ایجاد یک نوع مدل زبانی مشابه نیز وجود دارد. به عنوان مثال، در گرامر زیر، دو قانون X و Y با استخراج یک نوع مدل زبانی واحد MyType وجود دارد:
```antlr
X infers MyType: name=ID;
Y infers MyType: name=ID count=INT;
```
این باعث ایجاد یک رابطه واحد در مدل زبانی می‌شود که دو قانون بررسی‌کننده را با خصوصیات غیرمشترک و اختیاری با یکدیگر 'ادغام' می‌کند:
```ts
interface MyType extends AstNode {
    count?: number
    name: string
}
```

### قوانین ترمینال
قوانین ترمینال با نوع داخلی در مدل زبانی مرتبط هستند. این قوانین به تنهایی منجر به تولید نوع مدل زبانی نمی‌شوند، اما نوع خصوصیات در نوع مدل زبانی که از یک قانون بررسی‌کننده:
```antlr
terminal INT returns number: /[0-9]+/;
terminal ID returns string: /[a-zA-Z_][a-zA-Z0-9_]*/;

X: name=ID count=INT;
```

```ts
// generated interface
interface X extends AstNode {
    name: string
    count: number
}
```

خصوصیت name از نوع string است زیرا قانون پایانی ID به نوع داخلی string مرتبط شده است و خصوصیت count از نوع number است زیرا قانون پایانی INT به نوع داخلی number مرتبط شده است.

### قوانین نوع داده‌ای
قوانین نوع داده‌ای مشابه قوانین پایانی هستند اما در این حالت منجر به ایجاد نوع متغیر برای نوع داخلی در مدل زبانی می‌شوند. به عبارت دیگر، آنها نوع خصوصیات را در نوع مدل زبانی که از یک قانون بررسی‌کننده استخراج می‌شود، تعیین می‌کنند. اما متفاوت از قوانین پایانی، منجر به ایجاد نوع‌های متغیر برای نوع‌های داخلی در مدل زبانی می‌شوند:
```antlr
QualifiedName returns string: ID '.' ID;

X: name=QualifiedName;
```

```ts
// generated types
type QualifiedName = string;

interface X extends AstNode {
    name: string
}
```

### انواع اختصاص‌دهی
سه نوع اختصاص‌دهی در یک قانون بررسی‌کننده در دسترس است:

* = برای اختصاص دادن یک مقدار تکی به یک خصوصیت استفاده می‌شود و منجر به استخراج نوع خصوصیت از سمت راست عملگر شده در اختصاص‌دهی می‌شود.
* += برای اختصاص دادن چند مقدار به یک خصوصیت استفاده می‌شود و منجر به استخراج نوع خصوصیت به عنوان یک آرایه از سمت راست عملگر شده در اختصاص‌دهی می‌شود.
* ?= برای اختصاص دادن یک مقدار منطقی (بولین) به یک خصوصیت استفاده می‌شود و منجر به استخراج نوع خصوصیت به عنوان یک boolean می‌شود.

```antlr
X: name=ID numbers+=INT (numbers+=INT)* isValid?='valid'?;
```

```ts
// generated interface
interface X extends AstNode {
    name: string
    numbers: Array<number>
    isValid: boolean
}
```

در سمت راست یک عبارت از یکی از موارد زیر می‌تواند باشد:

* یک قاعده ترمینالی یا قاعده نوع داده‌ای، که باعث می‌شود نوع ویژگی از نوع داخلی تعیین شود.
* یک قاعده پارسر که باعث می‌شود نوع ویژگی برابر با نوع قاعده پارسر شود.
* یک مرجع متقابل که باعث می‌شود نوع ویژگی به نوع مرجع متقابل اشاره کند.
* یک گزینه، که باعث می‌شود نوع ویژگی برابر با یک اتحاد نوع از تمام انواع موجود در گزینه شود.

```antlr
X: 'x' name=ID;

Y: crossValue=[X:ID] alt=(INT | X | [X:ID]);
```

```ts
// generated types
interface X extends AstNode {
    name: string
}

interface Y extends AstNode {
    crossValue: Reference<X>
    alt: number | X | Reference<X>
}
```

### فراخوانی قوانین اختصاص نیافته

یک قانون پارسر نیازی به داشتن نشانی‌ها ندارد. ممکن است تنها شامل تماس های قانون نشانی نشده باشد. این نوع قوانین می‌توانند برای تغییر سلسله مراتب نوع‌ها استفاده شوند.

```antlr
X: A | B;

A: 'A' name=ID;
B: 'B' name=ID count=INT;

```

```ts
// generated types
type X = A | B;

interface A extends AstNode {
    name: string
}

interface B extends AstNode {
    name: string
    count: number
}
```

### اقدامات ساده

اکشن ها می‌توانند برای تغییر نوع یک گره درون یک قانون پارسر به یک نوع مدل سمتی دیگر استفاده شوند. به عنوان مثال، آنها به شما اجازه می دهند قوانین پارسر را که باید به چندین قانون تقسیم شوند را ساده کنید.

```antlr
X: 
    {infer A} 'A' name=ID 
  | {infer B} 'B' name=ID count=INT;

// is equivalent to:
X: A | B;
A: 'A' name=ID;
B: 'B' name=ID count=INT;
```

```ts
// generated types
type X = A | B;

interface A extends AstNode {
    name: string
}

interface B extends AstNode {
    name: string
    count: number
}
```

### اقدامات تعیین شده

عملیات (Action) ها نیز می توانند برای کنترل ساختار نوع مدل نهایی (Semantic Model) استفاده شوند. در این حالت، از گرامر برای ایجاد نوع مدل نهایی استفاده نمی شود بلکه عملیات متناسب با گرامر تعریف شده، این کار را برای ما انجام می دهد. به عبارت دیگر، از این عملیات برای ایجاد ساختار پیچیده تری در نوع مدل نهایی استفاده می شود که با استفاده از گرامر ممکن نبود آن را توصیف کرد. لذا، بهتر است پیش از ورود به این قسمت، با مستندات دیگر آشنا شوید.

بیایید دو گرامر متفاوت از مثال حسابی [Arithmetics example](https://github.com/langium/langium/blob/main/examples/arithmetics/src/language-server/arithmetics.langium) را مورد بررسی قرار دهیم. این دو گرامر برای پارس کردن یک سند شامل یک تعریف واحد از یک نام و یک مقدار انتساب عبارت هستند، که عبارت می تواند شامل هر تعداد اضافات یا یک مقدار عددی باشد.

اولین مورد از اقدامات اختصاص داده شده استفاده نمی کند:

```antlr
Definition: 
    'def' name=ID ':' expr=Expression;
Expression:
    Addition;
Addition infers Expression:
    left=Value ('+' right=Expression)?;
    
Primary infers Expression:
    '(' Expression ')' | {Literal} value=NUMBER;
```
وقتی تجزیه یک سند به شکل `def x: (1 + 2) + 3` باشد، شکل گره مدل معنایی آن به صورت زیر می باشد:

<img src="https://mermaid.ink/svg/pako:eNptUEsKwjAQvUqYVQLtQt1FcNUb6DJQQjP9YJOWmIpScneTWgxVNzPz5n0Wb4ZqUAgchGmsHFtyKY7C4GO0lMbJGMnzE-mxdpTGydibXt62a9rwX1YkouCjL7emLfftXI6U-cdb_oaX93nnVzYFJFjOe79GJ3eCwX4IPGSg0WrZqVDDLAwhAlyLGgXwcCppryLU44NOTm44P00F3NkJM5hGJR0WnQzdaeC17G_oXwXrdGU" align="center" alt="My Flowchart"  width="300" height="400" />


می‌توانیم ببینیم که گره‌های تودرتو `right -> left` در درخت غیر ضروری هستند و می‌خواهیم یک سطح تودرتو از درخت را حذف کنیم.
این کار را می توان با تغییر شکل دستور زبان و اضافه کردن یک عمل اختصاص داده شده انجام داد:

```langium
Definition: 
    'def' name=ID ':' expr=Addition ';';
Expression:
    Addition;
Addition infers Expression:
    Primary ({infer Addition.left=current} '+' right=Primary)*;
    
Primary infers Expression:
    '(' Expression ')' | {Literal} value=NUMBER;
```

اکنون تجزیه همان سند به این مدل معنایی تبدیل می شود:

<img src="https://mermaid.ink/svg/pako:eNplkMsKwjAQRX8lzCqBdqHuIrjqH-gyEEIzfWCTljQVJfTfTVq0VDfzOvcOzAQoe43AoXZqaMitOAuLz8FRmiJjJM8vpMPKU5oiYytexq6tmzhfUgJJ8NXLvWnPfp1Lse2Uj3CaV4_8Xxrp4UM349bKcJwhA4POqFbH04KwhAjwDRoUwGOplbsLEDbp1OT768uWwL2bMINp0Mpj0ar4EQO8Ut2I8xsgfmSI" align="center" alt="My Flowchart"  width="300" height="400" />

با اینکه این یک مثال نسبتاً پیش پا افتاده است، اضافه کردن لایه‌های بیشتری از انواع عبارت در گرامر، کیفیت درخت نحو شما را به شدت کاهش می‌دهد، زیرا هر لایه ویژگی خالی `right` دیگری را به درخت اضافه می‌کند. اقدامات تعیین شده این موضوع را به طور کامل برطرف می کند.
## انواع اعلام شده(Declared Types)
این مهم است که به خاطر داشته باشید که با اینکه که انواع اعلام شده می توانند گرامرهای شما را بهبود بخشند، آنها یک ویژگی *bleeding edge هستند و هنوز در حال توسعه می باشند*.
از آنجا که استنتاج نوع هر موجودیت یک قانون را در نظر می گیرد، حتی کوچکترین تغییرات می تواند انواع استنتاج شده شما را به روز کند. این می تواند منجر به تغییرات ناخواسته در مدل معنایی شما و رفتار نادرست خدمات وابسته به آن شود. برای کاهش احتمال تغییرات ناسازگار هنگام اصلاح دستور زبان، *انواع اعلام شده* را به عنوان یک ویژگی جدید معرفی کرده ایم.
در بیشتر موارد، به‌ویژه برای طراحی‌های اولیه زبان، استفاده از استنتاج نوع برای تولید انواع بهترین انتخاب خواهد بود. همانطور که زبان شما شروع به رشد می کند، بهتر است که بخش هایی از مدل معنایی خود را با استفاده از انواع اعلام شده اصلاح کنید.
با این وجود، انواع اعلام شده می تواند *به خصوص* برای زبان های بالغ تر و پیچیده تر مفید باشد، جایی که یک مدل معنایی پایدار کلیدی است و تغییرات ناسازگار ایجاد شده توسط انواع استنباط شده می تواند خدمات زبان شما را خراب کند. انواع اعلام شده به کاربر این امکان را می دهد که نوع قوانین تجزیه کننده خود را **درست کند** و برای شناسایی تغییرات ناسازگار بر قدرت خطاهای اعتبارسنجی تکیه کند.


بیایید به مثال بخش قبلی بپردازیم:

```langium
X infers MyType: name=ID;
Y infers MyType: name=ID count=INT;

// should be replaced by:
interface MyType {
    name: string
    count?: number
}

X returns MyType: name=ID;
Y returns MyType: name=ID count=INT;
```

اکنون به صراحت "MyType" را مستقیماً در گرامر با کلمه کلیدی `interface` اعلام می کنیم. قوانین تجزیه کننده `X` و `Y` که گره‌هایی از نوع `MyType` ایجاد می‌کنند، باید به صراحت نوع گره‌ای را که ایجاد می‌کنند با کلمه کلیدی `returns` اعلام کنند.

بر خلاف [inferred types](#inferred-types), همه ویژگی ها باید به ترتیب اعلام شوند تا درون یک قانون تجزیه معتبر باشند. دستور زیر:

```langium
Z returns MyType: name=ID age=INT;
```

خطای اعتبارسنجی زیر را نشان می‌دهد `A property 'age' is not expected` زیرا اعلان `MyType` شامل ویژگی `age` نمی‌شود. به طور خلاصه، *انواع اعلام شده* یک لایه حفاظتی از طریق اعتبارسنجی به دستور زبان اضافه می کند که از عدم تطابق بین انواع مدل معنایی مورد انتظار و شکل گره های تجزیه شده جلوگیری می کند.


یک نوع اعلام شده همچنین می تواند انواع را گسترش دهد، مانند انواع دیگر اعلام شده یا انواع استنتاج شده از قوانین تجزیه کننده:

```langium
interface MyType {
    name: string
}

interface MyOtherType extends MyType {
    count: number
}

Y returns MyOtherType: name=ID count=INT;
```

اعلان صریح انواع اتحاد در گرامر با کلمه کلیدی `type` به دست می آید:
```ts
type X = A | B;

// generates:
type X = A | B;
```

<!-- Please note that it is not allowed to use an alias type as a return type in a parser rule. The following syntax is invalid:
```
type X = A | B;

Y returns X: name=ID;
``` -->

استفاده از `return` همیشه انتظار ارجاع به نوع موجود را دارد. برای ایجاد یک نوع جدید برای قانون خود، از کلمه کلیدی `infers` استفاده کنید یا به صراحت یک رابط را اعلام کنید.
### ارجاعات متقابل، آرایه ها و جایگزین ها


انواع اعلان شده دارای دستور خاصی برای اعلام ارجاعات متقابل، آرایه ها و جایگزین ها هستند:


```langium
interface A {
    reference: @B
    array: B[]
    alternative: B | C
}

interface B {
    name: string
}

interface C {
    name: string
    count: number
}

X returns A: reference=[B:ID] array+=Y (array+=Y)* alternative=(Y | Z);

Y returns B: 'Y' name=ID;

Z returns C: 'Z' name=ID count=INT;
```

### اقدامات(Actions)


اقدامات مربوط به یک نوع اعلام شده دارای دستور زیر هستند:

```langium
interface A {
    name: string
}

interface B {
    name: string
    count: number
}

X: 
    {A} 'A' name=ID 
  | {B} 'B' name=ID count=INT;
```

به عدم وجود کلمه کلیدی `infer` در مقایسه با [actions which infer a type](#simple-actions) توجه داشته باشید.

## اتحادهای مرجع


تلاش برای ارجاع به انواع مختلف عناصر می تواند فرآیندی مستعد خطا باشد. به قانون زیر نگاهی بیندازید که سعی می‌کند به یک `Function` یا `Variable` ارجاع دهد:


```langium
MemberCall: (element=[Function:ID] | element=[Variable:ID]);
```

از آنجایی که هر دو گزینه از نظر تجزیه‌کننده فقط یک `ID` هستند، این دستور زبان قابل تصمیم‌گیری نیست و سند `CLI `langium در طول تولید با خطا مواجه می‌شود. خوشبختانه، ما می‌توانیم با افزودن یک لایه غیرمستقیم با استفاده از یک قانون تجزیه‌کننده اضافی، این مورد را بهبود بخشیم:
```langium
NamedElement: Function | Variable;

MemberCall: element=[NamedElement:ID];
```

این به ما این امکان را می‌دهد با استفاده از قانون رایج `NamedElement` به `Function` یا `Variable` ارجاع دهیم. هرچند، اکنون قانونی را معرفی کرده‌ایم که هرگز تجزیه نمی‌شود، بلکه فقط برای هدف سیستم نوع وجود دارد تا انواع هدف صحیح مرجع را انتخاب کند. با استفاده از انواع اعلام شده، می‌توانیم این قانون استفاده‌ نشده را اصلاح کنیم و گرامر خود را در این فرآیند انعطاف‌ پذیرتر کنیم:
```langium
// Note the `type` prefix here
type NamedElement = Function | Variable;

MemberCall: element=[NamedElement:ID];
```

همچنین می‌توانیم از رابط‌ها به جای انواع اتحاد با نتایج مشابه استفاده کنیم:


```langium
interface NamedElement {
    name: string
}

// Infers an interface `Function` that extends `NamedElement`
Function returns NamedElement: {infer Function} "function" name=ID ...;

// This also picks up on the `Function` elements
MemberCall: element=[NamedElement:ID];
```
