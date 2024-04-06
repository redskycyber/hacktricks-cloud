# Rust Basics

### Generički tipovi

Kreirajte strukturu gde jedna od njihovih vrednosti može biti bilo koji tip

```rust
struct Wrapper<T> {
value: T,
}

impl<T> Wrapper<T> {
pub fn new(value: T) -> Self {
Wrapper { value }
}
}

Wrapper::new(42).value
Wrapper::new("Foo").value, "Foo"
```

### Option, Some & None

Tip Option znači da vrednost može biti tipa Some (postoji nešto) ili None:

```rust
pub enum Option<T> {
None,
Some(T),
}
```

Možete koristiti funkcije poput `is_some()` ili `is_none()` da proverite vrednost Option-a.

### Makroi

Makroi su moćniji od funkcija jer se proširuju da bi proizveli više koda od koda koji ste ručno napisali. Na primer, potpis funkcije mora da deklariše broj i tip parametara koje funkcija ima. Makroi, s druge strane, mogu da prime promenljiv broj parametara: možemo pozvati `println!("hello")` sa jednim argumentom ili `println!("hello {}", name)` sa dva argumenta. Takođe, makroi se proširuju pre nego što kompajler tumači značenje koda, pa makro može, na primer, da implementira trait na datom tipu. Funkcija ne može, jer se poziva u vreme izvršavanja, a trait treba da bude implementiran u vreme kompilacije.

```rust
macro_rules! my_macro {
() => {
println!("Check out my macro!");
};
($val:expr) => {
println!("Look at this other macro: {}", $val);
}
}
fn main() {
my_macro!();
my_macro!(7777);
}

// Export a macro from a module
mod macros {
#[macro_export]
macro_rules! my_macro {
() => {
println!("Check out my macro!");
};
}
}
```

### Iterirajte

Iteracija je proces ponavljanja određenog bloka koda više puta. U programiranju, iteracija se koristi za izvršavanje istog koda više puta, obično na različitim ulaznim podacima. To omogućava efikasno rukovanje velikim skupovima podataka ili izvršavanje istih operacija na različitim elementima.

U Rustu, iteracija se može postići korišćenjem petlji. Postoje dve osnovne vrste petlji u Rustu: `for` petlja i `while` petlja.

#### `for` petlja

`for` petlja se koristi kada želite da iterirate kroz kolekciju elemenata. Može se koristiti sa različitim tipovima kolekcija, kao što su vektori, nizovi ili opsezi.

```rust
let numbers = vec![1, 2, 3, 4, 5];

for number in numbers {
    println!("Broj: {}", number);
}
```

#### `while` petlja

`while` petlja se koristi kada želite da iterirate kroz blok koda dok je određeni uslov ispunjen. Uslov se proverava pre svake iteracije petlje.

```rust
let mut count = 0;

while count < 5 {
    println!("Brojač: {}", count);
    count += 1;
}
```

#### `loop` petlja

`loop` petlja se koristi kada želite da iterirate kroz blok koda beskonačno mnogo puta, ili dok se ne ispuni određeni uslov za prekid petlje.

```rust
let mut count = 0;

loop {
    println!("Brojač: {}", count);
    count += 1;

    if count == 5 {
        break;
    }
}
```

Korišćenje odgovarajuće petlje zavisi od specifičnih zahteva vašeg koda.

```rust
// Iterate through a vector
let my_fav_fruits = vec!["banana", "raspberry"];
let mut my_iterable_fav_fruits = my_fav_fruits.iter();
assert_eq!(my_iterable_fav_fruits.next(), Some(&"banana"));
assert_eq!(my_iterable_fav_fruits.next(), Some(&"raspberry"));
assert_eq!(my_iterable_fav_fruits.next(), None); // When it's over, it's none

// One line iteration with action
my_fav_fruits.iter().map(|x| capitalize_first(x)).collect()

// Hashmap iteration
for (key, hashvalue) in &*map {
for key in map.keys() {
for value in map.values() {
```

### Rekurzivna kutija

A recursive box is a data structure in Rust that allows for the creation of self-referential types. It is commonly used when dealing with data structures that have a recursive nature, such as linked lists or trees.

To create a recursive box, you can use the `Box` type provided by Rust. The `Box` type is a smart pointer that provides heap allocation and ownership of the data it points to.

Here is an example of how to create a recursive box:

```rust
struct Node {
    value: i32,
    next: Option<Box<Node>>,
}

fn main() {
    let node1 = Node {
        value: 1,
        next: Some(Box::new(Node {
            value: 2,
            next: Some(Box::new(Node {
                value: 3,
                next: None,
            })),
        })),
    };
}
```

In this example, the `Node` struct contains a `value` field of type `i32` and a `next` field of type `Option<Box<Node>>`. The `next` field is an `Option` type because it can either contain a `Box` pointing to the next `Node`, or `None` to indicate the end of the list.

By using `Box`, we can allocate memory on the heap for each `Node` and create a recursive structure. The `Box` type ensures that the memory is deallocated correctly when it goes out of scope.

Recursive boxes are a powerful tool in Rust for creating complex data structures that require self-referentiality. They provide a safe and efficient way to handle recursive data without causing memory leaks or undefined behavior.

```rust
enum List {
Cons(i32, List),
Nil,
}

let list = Cons(1, Cons(2, Cons(3, Nil)));
```

#### ako

The `if` statement is used to execute a block of code only if a certain condition is true. It has the following syntax:

```rust
if condition {
    // code to be executed if the condition is true
}
```

The `condition` is an expression that evaluates to either `true` or `false`. If the condition is `true`, the code block inside the `if` statement will be executed. If the condition is `false`, the code block will be skipped.

Here's an example:

```rust
let number = 5;

if number > 0 {
    println!("The number is positive");
}
```

In this example, the code inside the `if` statement will be executed because the condition `number > 0` is true. The output will be `The number is positive`.

#### ako

Naredba `ako` se koristi da izvrši blok koda samo ako je određeni uslov tačan. Ima sledeću sintaksu:

```rust
ako uslov {
    // kod koji će se izvršiti ako je uslov tačan
}
```

`Uslov` je izraz koji se vrednuje kao `tačno` ili `netačno`. Ako je uslov `tačan`, blok koda unutar naredbe `ako` će se izvršiti. Ako je uslov `netačan`, blok koda će biti preskočen.

Evo primera:

```rust
let broj = 5;

ako broj > 0 {
    println!("Broj je pozitivan");
}
```

U ovom primeru, kod unutar naredbe `ako` će se izvršiti jer je uslov `broj > 0` tačan. Ispis će biti `Broj je pozitivan`.

```rust
let n = 5;
if n < 0 {
print!("{} is negative", n);
} else if n > 0 {
print!("{} is positive", n);
} else {
print!("{} is zero", n);
}
```

#### poklapanje

`match` je konstrukcija jezika Rust koja omogućava upoređivanje vrednosti sa različitim oblicima i izvršavanje odgovarajućeg koda na osnovu rezultata upoređivanja. Ova konstrukcija je veoma korisna za obradu različitih slučajeva ili opcija.

Evo osnovnog sintaksnog obrasca `match` izraza:

```rust
match vrednost {
    vrednost1 => {
        // Izvršava se kod ako vrednost odgovara vrednosti1
    },
    vrednost2 => {
        // Izvršava se kod ako vrednost odgovara vrednosti2
    },
    _ => {
        // Izvršava se kod ako vrednost ne odgovara ni jednoj od prethodnih vrednosti
    }
}
```

U ovom primeru, `vrednost` se upoređuje sa `vrednost1`, `vrednost2` i `_`. Ako `vrednost` odgovara `vrednost1`, izvršava se kod unutar bloka `vrednost1`. Ako `vrednost` odgovara `vrednost2`, izvršava se kod unutar bloka `vrednost2`. Ako `vrednost` ne odgovara ni jednoj od prethodnih vrednosti, izvršava se kod unutar bloka `_`.

Ova konstrukcija omogućava efikasno upravljanje različitim slučajevima i olakšava čitanje i pisanje čistog i jasnog koda.

```rust
match number {
// Match a single value
1 => println!("One!"),
// Match several values
2 | 3 | 5 | 7 | 11 => println!("This is a prime"),
// TODO ^ Try adding 13 to the list of prime values
// Match an inclusive range
13..=19 => println!("A teen"),
// Handle the rest of cases
_ => println!("Ain't special"),
}

let boolean = true;
// Match is an expression too
let binary = match boolean {
// The arms of a match must cover all the possible values
false => 0,
true => 1,
// TODO ^ Try commenting out one of these arms
};
```

#### petlja (beskonačna)

Kada želite da izvršite određeni blok koda beskonačan broj puta, možete koristiti petlju. Petlja se sastoji od uslova koji se proverava na početku svake iteracije. Ako je uslov ispunjen, blok koda se izvršava, a zatim se petlja ponavlja. U suprotnom, petlja se prekida i izvršavanje se nastavlja sa sledećim delom koda.

Da biste napravili beskonačnu petlju u Rustu, možete koristiti ključnu reč `loop`. Ova petlja će se izvršavati sve dok se ne prekine ručno ili dok se ne naiđe na `break` izjavu unutar petlje.

```rust
loop {
    // Blok koda koji se izvršava beskonačno
    // ...
}
```

Ova petlja je korisna kada želite da izvršite određene zadatke koji zahtevaju neprekidno izvršavanje, kao što je čitanje podataka sa senzora ili obrada zahteva u serverskoj aplikaciji.

```rust
loop {
count += 1;
if count == 3 {
println!("three");
continue;
}
println!("{}", count);
if count == 5 {
println!("OK, that's enough");
break;
}
}
```

#### dok

`while` petlja se koristi za ponavljanje određenog bloka koda sve dok je određeni uslov ispunjen.

```rust
while uslov {
    // Izvršavanje koda
}
```

Ovde je `uslov` izraz koji se evaluira kao `true` ili `false`. Ako je `uslov` `true`, blok koda unutar `while` petlje će se izvršavati. Kada se `uslov` evaluira kao `false`, izvršavanje petlje se zaustavlja i program nastavlja sa izvršavanjem koda nakon petlje.

Na primer:

```rust
let mut brojac = 0;

while brojac < 5 {
    println!("Vrednost brojaca je: {}", brojac);
    brojac += 1;
}
```

Ovde će se blok koda unutar `while` petlje izvršiti pet puta, jer će se `brojac` povećavati za jedan u svakoj iteraciji.

```rust
let mut n = 1;
while n < 101 {
if n % 15 == 0 {
println!("fizzbuzz");
} else if n % 5 == 0 {
println!("buzz");
} else {
println!("{}", n);
}
n += 1;
}
```

#### za

```rust
for n in 1..101 {
if n % 15 == 0 {
println!("fizzbuzz");
} else {
println!("{}", n);
}
}

// Use "..=" to make inclusive both ends
for n in 1..=100 {
if n % 15 == 0 {
println!("fizzbuzz");
} else if n % 3 == 0 {
println!("fizz");
} else if n % 5 == 0 {
println!("buzz");
} else {
println!("{}", n);
}
}

// ITERATIONS

let names = vec!["Bob", "Frank", "Ferris"];
//iter - Doesn't consume the collection
for name in names.iter() {
match name {
&"Ferris" => println!("There is a rustacean among us!"),
_ => println!("Hello {}", name),
}
}
//into_iter - COnsumes the collection
for name in names.into_iter() {
match name {
"Ferris" => println!("There is a rustacean among us!"),
_ => println!("Hello {}", name),
}
}
//iter_mut - This mutably borrows each element of the collection
for name in names.iter_mut() {
*name = match name {
&mut "Ferris" => "There is a rustacean among us!",
_ => "Hello",
}
}
```

#### ako je

`if let` je skraćenica za `if let Some(x) = y`. Ova konstrukcija se koristi za pattern matching i omogućava da se izvrši određeni kod samo ako je `y` jednak `Some(x)`. Ako `y` nije jednak `Some(x)`, kod se ne izvršava.

```rust
let option = Some(5);

if let Some(x) = option {
    println!("Vrednost je {}", x);
} else {
    println!("Nema vrednosti");
}
```

U ovom primeru, `if let` proverava da li je `option` jednak `Some(x)`. Ako jeste, vrednost `x` se koristi u bloku `if` i ispisuje se "Vrednost je 5". Ako `option` nije jednak `Some(x)`, izvršava se blok `else` i ispisuje se "Nema vrednosti".

```rust
let optional_word = Some(String::from("rustlings"));
if let word = optional_word {
println!("The word is: {}", word);
} else {
println!("The optional word doesn't contain anything");
}
```

#### while let

`while let` petlja je korisna kada želite da izvršavate određene radnje dok se određeni obrazac podudara sa vrednošću. Ova petlja se koristi u Rust programiranju.

```rust
while let Some(value) = iterator.next() {
    // Izvršavanje koda dok se obrazac podudara
}
```

Ova petlja će se izvršavati sve dok `iterator.next()` vraća `Some(value)`. Kada `iterator.next()` vrati `None`, petlja se prekida.

Ova petlja je korisna kada radite sa iteratorima i želite da izvršavate određene radnje za svaku vrednost koju iterator vraća.

```rust
let mut optional = Some(0);
// This reads: "while `let` destructures `optional` into
// `Some(i)`, evaluate the block (`{}`). Else `break`.
while let Some(i) = optional {
if i > 9 {
println!("Greater than 9, quit!");
optional = None;
} else {
println!("`i` is `{:?}`. Try again.", i);
optional = Some(i + 1);
}
// ^ Less rightward drift and doesn't require
// explicitly handling the failing case.
}
```

### Traitovi

Kreiranje nove metode za tip

```rust
trait AppendBar {
fn append_bar(self) -> Self;
}

impl AppendBar for String {
fn append_bar(self) -> Self{
format!("{}Bar", self)
}
}

let s = String::from("Foo");
let s = s.append_bar();
println!("s: {}", s);
```

### Testovi

Testovi su ključni deo procesa razvoja softvera. Oni se koriste za proveru ispravnosti funkcionalnosti i otkrivanje grešaka u kodu. Testovi se mogu pisati na različite načine, kao što su jedinični testovi, integracioni testovi i sistemski testovi.

Jedinični testovi se fokusiraju na proveru ispravnosti pojedinačnih delova koda, kao što su funkcije ili metode. Oni se izvršavaju izolovano, bez zavisnosti od drugih delova sistema. Integracioni testovi se koriste za proveru ispravnosti interakcije između različitih delova sistema. Oni se izvršavaju na nivou modula ili komponenti. Sistemski testovi se koriste za proveru ispravnosti celokupnog sistema, uključujući sve njegove delove i interakcije.

Testovi se mogu pisati ručno ili automatski. Ručno pisanje testova može biti vremenski zahtevno i podložno greškama. Automatsko pisanje testova koristi alate i biblioteke za generisanje i izvršavanje testova. Automatski testovi su efikasniji i pouzdaniji, jer se mogu lako ponavljati i ažurirati.

Prilikom pisanja testova, važno je definisati očekivane rezultate i uslove za prolazak testa. Ovo omogućava jasnoću i objektivnost u oceni ispravnosti koda. Takođe je važno pokriti različite scenarije i granice kako bi se osigurala potpuna provera funkcionalnosti.

Testiranje je iterativan proces koji se obavlja tokom celog razvojnog ciklusa. Redovno testiranje pomaže u otkrivanju grešaka i poboljšanju kvaliteta softvera.

```rust
#[cfg(test)]
mod tests {
#[test]
fn you_can_assert() {
assert!(true);
assert_eq!(true, true);
assert_ne!(true, false);
}
}
```

### Threadovanje

#### Arc

Arc može koristiti Clone da bi kreirao više referenci ka objektu kako bi ih prosledio threadovima. Kada poslednja referenca koja pokazuje na vrednost izađe iz opsega, promenljiva se briše.

```rust
use std::sync::Arc;
let apple = Arc::new("the same apple");
for _ in 0..10 {
let apple = Arc::clone(&apple);
thread::spawn(move || {
println!("{:?}", apple);
});
}
```

#### Niti

U ovom slučaju ćemo niti proslediti promenljivu koju će moći da izmeni.

```rust
fn main() {
let status = Arc::new(Mutex::new(JobStatus { jobs_completed: 0 }));
let status_shared = Arc::clone(&status);
thread::spawn(move || {
for _ in 0..10 {
thread::sleep(Duration::from_millis(250));
let mut status = status_shared.lock().unwrap();
status.jobs_completed += 1;
}
});
while status.lock().unwrap().jobs_completed < 10 {
println!("waiting... ");
thread::sleep(Duration::from_millis(500));
}
}
```
