# macOS Objective-C

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>

## Objective-C

{% hint style="danger" %}
Imajte na umu da programi napisani u Objective-C **zadržavaju** svoje deklaracije klasa **kada** **kompiliraju** u [Mach-O binarne datoteke](macos-files-folders-and-binaries/universal-binaries-and-mach-o-format.md). Takve deklaracije klasa **uključuju** ime i tip:
{% endhint %}

* Klasa
* Metode klase
* Instancne varijable klase

Ove informacije možete dobiti koristeći [**class-dump**](https://github.com/nygard/class-dump):

```bash
class-dump Kindle.app
```

Napomena da bi ova imena mogla biti prikrivena kako bi se otežalo preokretanje binarnog koda.

## Klase, Metode i Objekti

### Interfejs, Svojstva i Metode

```objectivec
// Declare the interface of the class
@interface MyVehicle : NSObject

// Declare the properties
@property NSString *vehicleType;
@property int numberOfWheels;

// Declare the methods
- (void)startEngine;
- (void)addWheels:(int)value;

@end
```

### **Klasa**

```objectivec
@implementation MyVehicle : NSObject

// No need to indicate the properties, only define methods

- (void)startEngine {
NSLog(@"Engine started");
}

- (void)addWheels:(int)value {
self.numberOfWheels += value;
}

@end
```

### **Objekat i poziv metode**

Da biste kreirali instancu klase, koristi se metoda **`alloc`** koja **alocira memoriju** za svako **svojstvo** i **postavlja na nulu** te alokacije. Zatim se poziva metoda **`init`**, koja **inicijalizuje svojstva** na **potrebne vrednosti**.

```objectivec
// Something like this:
MyVehicle *newVehicle = [[MyVehicle alloc] init];

// Which is usually expressed as:
MyVehicle *newVehicle = [MyVehicle new];

// To call a method
// [myClassInstance nameOfTheMethodFirstParam:param1 secondParam:param2]
[newVehicle addWheels:4];
```

### **Metode klase**

Metode klase se definišu sa **plus znakom** (+), a ne sa crticom (-) koja se koristi kod instancnih metoda. Na primer, metoda klase **`stringWithString`** klase **NSString**:

```objectivec
+ (id)stringWithString:(NSString *)aString;
```

### Setter i Getter

Da biste postavili i dobili vrednosti svojstava, to možete uraditi pomoću **tačkaste notacije** ili kao da pozivate **metodu**:

```objectivec
// Set
newVehicle.numberOfWheels = 2;
[newVehicle setNumberOfWheels:3];

// Get
NSLog(@"Number of wheels: %i", newVehicle.numberOfWheels);
NSLog(@"Number of wheels: %i", [newVehicle numberOfWheels]);
```

### **Instance Variables**

Alternativno, umesto metoda za postavljanje i dobavljanje vrednosti, možete koristiti instance varijable. Ove varijable imaju isto ime kao i svojstva, ali počinju sa "\_":

```objectivec
- (void)makeLongTruck {
_numberOfWheels = +10000;
NSLog(@"Number of wheels: %i", self.numberOfLeaves);
}
```

### Protokoli

Protokoli su skup deklaracija metoda (bez svojstava). Klasa koja implementira protokol implementira deklarisane metode.

Postoje 2 vrste metoda: **obavezni** i **opcioni**. Po **podrazumevanju**, metoda je **obavezna** (ali možete je označiti i sa **`@required`** oznakom). Da biste označili da je metoda opciona, koristite **`@optional`**.

```objectivec
@protocol myNewProtocol
- (void) method1; //mandatory
@required
- (void) method2; //mandatory
@optional
- (void) method3; //optional
@end
```

### Sve zajedno

U ovom poglavlju ćemo se baviti osnovama Objective-C jezika i kako ga koristiti za hakiranje macOS sistema. Objective-C je objektno orijentisani jezik koji se često koristi za razvoj aplikacija na macOS platformi. Razumevanje osnovnih koncepata i sintakse Objective-C jezika je ključno za razumevanje i izvođenje određenih hakirajućih tehnika na macOS sistemu.

#### Osnovni koncepti Objective-C jezika

Objective-C je jezik koji kombinuje sintaksu C jezika sa dodatnim objektno orijentisanim konceptima. Osnovni koncepti koje treba razumeti uključuju:

* Klase: Klase su temeljni elementi Objective-C jezika. One definišu objekte i njihove osobine i ponašanje.
* Objekti: Objekti su instance klasa i predstavljaju konkretne entitete sa svojim stanjem i ponašanjem.
* Metode: Metode su funkcije koje se izvršavaju nad objektima i definišu njihovo ponašanje.
* Poruke: Poruke su način komunikacije između objekata. Objekti šalju poruke jedni drugima kako bi izvršili određene akcije.
* Nasleđivanje: Nasleđivanje omogućava kreiranje novih klasa na osnovu postojećih klasa, čime se omogućava ponovno korišćenje koda i proširivanje funkcionalnosti.

#### Hakiranje macOS sistema korišćenjem Objective-C jezika

Objective-C jezik može se koristiti za izvođenje različitih hakirajućih tehnika na macOS sistemu. Neki od najčešće korišćenih tehnika uključuju:

* Privilegija eskalacija: Korišćenje Objective-C jezika omogućava hakere da pristupe privilegijama koje su im inače nedostupne. To se može postići korišćenjem ranjivosti u macOS sistemu ili manipulacijom objekata i metoda.
* Injekcija koda: Objective-C jezik omogućava hakere da ubace zlonamerni kod u postojeće aplikacije ili sistem, čime mogu izvršiti različite napade kao što su krađa podataka ili daljinsko izvršavanje koda.
* Reverse engineering: Objective-C jezik olakšava analizu i dekompilaciju aplikacija kako bi se otkrile ranjivosti ili pronašle tajne funkcionalnosti.
* Sniffing komunikacije: Korišćenjem Objective-C jezika, hakere je moguće izvršiti snimanje i analizu komunikacije između aplikacija i sistema radi otkrivanja osetljivih informacija.

Razumevanje osnovnih koncepata Objective-C jezika i njegova primena u hakiranju macOS sistema omogućava hakerima da izvrše različite napade i postignu svoje ciljeve. Važno je napomenuti da je hakiranje nelegalno i da se ove tehnike trebaju koristiti samo u etičke svrhe, kao što je testiranje sigurnosti sistema ili otkrivanje ranjivosti.

```objectivec
// gcc -framework Foundation test_obj.m -o test_obj
#import <Foundation/Foundation.h>

@protocol myVehicleProtocol
- (void) startEngine; //mandatory
@required
- (void) addWheels:(int)value; //mandatory
@optional
- (void) makeLongTruck; //optional
@end

@interface MyVehicle : NSObject <myVehicleProtocol>

@property int numberOfWheels;

- (void)startEngine;
- (void)addWheels:(int)value;
- (void)makeLongTruck;

@end

@implementation MyVehicle : NSObject

- (void)startEngine {
NSLog(@"Engine started");
}

- (void)addWheels:(int)value {
self.numberOfWheels += value;
}

- (void)makeLongTruck {
_numberOfWheels = +10000;
NSLog(@"Number of wheels: %i", self.numberOfWheels);
}

@end

int main() {
MyVehicle* mySuperCar = [MyVehicle new];
[mySuperCar startEngine];
mySuperCar.numberOfWheels = 4;
NSLog(@"Number of wheels: %i", mySuperCar.numberOfWheels);
[mySuperCar setNumberOfWheels:3];
NSLog(@"Number of wheels: %i", mySuperCar.numberOfWheels);
[mySuperCar makeLongTruck];
}
```

### Osnovne klase

#### String

{% code overflow="wrap" %}
```objectivec
// NSString
NSString *bookTitle = @"The Catcher in the Rye";
NSString *bookAuthor = [[NSString alloc] initWithCString:"J.D. Salinger" encoding:NSUTF8StringEncoding];
NSString *bookPublicationYear = [NSString stringWithCString:"1951" encoding:NSUTF8StringEncoding];
```
{% endcode %}

Osnovne klase su **nepromenljive**, tako da bi se dodao string postojećem, potrebno je **kreirati novi NSString**.

{% code overflow="wrap" %}
```objectivec
NSString *bookDescription = [NSString stringWithFormat:@"%@ by %@ was published in %@", bookTitle, bookAuthor, bookPublicationYear];
```
{% endcode %}

Ili možete koristiti i **mutable** klasu stringova:

{% code overflow="wrap" %}
```objectivec
NSMutableString *mutableString = [NSMutableString stringWithString:@"The book "];
[mutableString appendString:bookTitle];
[mutableString appendString:@" was written by "];
[mutableString appendString:bookAuthor];
[mutableString appendString:@" and published in "];
[mutableString appendString:bookPublicationYear];
```
{% endcode %}

#### Broj

{% code overflow="wrap" %}
```objectivec
// character literals.
NSNumber *theLetterZ = @'Z'; // equivalent to [NSNumber numberWithChar:'Z']

// integral literals.
NSNumber *fortyTwo = @42; // equivalent to [NSNumber numberWithInt:42]
NSNumber *fortyTwoUnsigned = @42U; // equivalent to [NSNumber numberWithUnsignedInt:42U]
NSNumber *fortyTwoLong = @42L; // equivalent to [NSNumber numberWithLong:42L]
NSNumber *fortyTwoLongLong = @42LL; // equivalent to [NSNumber numberWithLongLong:42LL]

// floating point literals.
NSNumber *piFloat = @3.141592654F; // equivalent to [NSNumber numberWithFloat:3.141592654F]
NSNumber *piDouble = @3.1415926535; // equivalent to [NSNumber numberWithDouble:3.1415926535]

// BOOL literals.
NSNumber *yesNumber = @YES; // equivalent to [NSNumber numberWithBool:YES]
NSNumber *noNumber = @NO; // equivalent to [NSNumber numberWithBool:NO]
```
{% endcode %}

#### Nizovi, skupovi i rečnici

{% code overflow="wrap" %}
```objectivec
// Inmutable arrays
NSArray *colorsArray1 = [NSArray arrayWithObjects:@"red", @"green", @"blue", nil];
NSArray *colorsArray2 = @[@"yellow", @"cyan", @"magenta"];
NSArray *colorsArray3 = @[firstColor, secondColor, thirdColor];

// Mutable arrays
NSMutableArray *mutColorsArray = [NSMutableArray array];
[mutColorsArray addObject:@"red"];
[mutColorsArray addObject:@"green"];
[mutColorsArray addObject:@"blue"];
[mutColorsArray addObject:@"yellow"];
[mutColorsArray replaceObjectAtIndex:0 withObject:@"purple"];

// Inmutable Sets
NSSet *fruitsSet1 = [NSSet setWithObjects:@"apple", @"banana", @"orange", nil];
NSSet *fruitsSet2 = [NSSet setWithArray:@[@"apple", @"banana", @"orange"]];

// Mutable sets
NSMutableSet *mutFruitsSet = [NSMutableSet setWithObjects:@"apple", @"banana", @"orange", nil];
[mutFruitsSet addObject:@"grape"];
[mutFruitsSet removeObject:@"apple"];


// Dictionary
NSDictionary *fruitColorsDictionary = @{
@"apple" : @"red",
@"banana" : @"yellow",
@"orange" : @"orange",
@"grape" : @"purple"
};

// In dictionaryWithObjectsAndKeys you specify the value and then the key:
NSDictionary *fruitColorsDictionary2 = [NSDictionary dictionaryWithObjectsAndKeys:
@"red", @"apple",
@"yellow", @"banana",
@"orange", @"orange",
@"purple", @"grape",
nil];

// Mutable dictionary
NSMutableDictionary *mutFruitColorsDictionary = [NSMutableDictionary dictionaryWithDictionary:fruitColorsDictionary];
[mutFruitColorsDictionary setObject:@"green" forKey:@"apple"];
[mutFruitColorsDictionary removeObjectForKey:@"grape"];
```
{% endcode %}

### Blokovi

Blokovi su **funkcije koje se ponašaju kao objekti**, tako da se mogu proslediti funkcijama ili **čuvati** u **nizovima** ili **rečnicima**. Takođe, mogu **predstavljati vrednost ako im se dodeljuju vrednosti**, pa su slični lambda funkcijama.

```objectivec
returnType (^blockName)(argumentType1, argumentType2, ...) = ^(argumentType1 param1, argumentType2 param2, ...){
//Perform operations here
};

// For example

int (^suma)(int, int) = ^(int a, int b){
return a+b;
};
NSLog(@"3+4 = %d", suma(3,4));
```

Takođe je moguće **definisati tip bloka koji će se koristiti kao parametar** u funkcijama:

```objectivec
// Define the block type
typedef void (^callbackLogger)(void);

// Create a bloack with the block type
callbackLogger myLogger = ^{
NSLog(@"%@", @"This is my block");
};

// Use it inside a function as a param
void genericLogger(callbackLogger blockParam) {
NSLog(@"%@", @"This is my function");
blockParam();
}
genericLogger(myLogger);

// Call it inline
genericLogger(^{
NSLog(@"%@", @"This is my second block");
});
```

### Fajlovi

{% code overflow="wrap" %}
```objectivec
// Manager to manage files
NSFileManager *fileManager = [NSFileManager defaultManager];

// Check if file exists:
if ([fileManager fileExistsAtPath:@"/path/to/file.txt" ] == YES) {
NSLog (@"File exists");
}

// copy files
if ([fileManager copyItemAtPath: @"/path/to/file1.txt" toPath: @"/path/to/file2.txt" error:nil] == YES) {
NSLog (@"Copy successful");
}

// Check if the content of 2 files match
if ([fileManager contentsEqualAtPath:@"/path/to/file1.txt" andPath:@"/path/to/file2.txt"] == YES) {
NSLog (@"File contents match");
}

// Delete file
if ([fileManager removeItemAtPath:@"/path/to/file1.txt" error:nil]) {
NSLog(@"Removed successfully");
}
```
{% endcode %}

Takođe je moguće upravljati datotekama **koristeći `NSURL` objekte umesto `NSString` objekata**. Imena metoda su slična, ali **umesto `Path` koristimo `URL`**.

```objectivec
NSURL *fileSrc = [NSURL fileURLWithPath:@"/path/to/file1.txt"];
NSURL *fileDst = [NSURL fileURLWithPath:@"/path/to/file2.txt"];
[fileManager moveItemAtURL:fileSrc toURL:fileDst error: nil];
```

Većina osnovnih klasa ima definisanu metodu `writeToFile:<path> atomically:<YES> encoding:<encoding> error:nil` koja im omogućava da budu direktno upisane u fajl:

{% code overflow="wrap" %}
```objectivec
NSString* tmp = @"something temporary";
[tmp writeToFile:@"/tmp/tmp1.txt" atomically:YES encoding:NSASCIIStringEncoding error:nil];
```
{% endcode %}

<details>

<summary><strong>Naučite hakovanje AWS-a od nule do heroja sa</strong> <a href="https://training.hacktricks.xyz/courses/arte"><strong>htARTE (HackTricks AWS Red Team Expert)</strong></a><strong>!</strong></summary>

Drugi načini podrške HackTricks-u:

* Ako želite da vidite **vašu kompaniju reklamiranu na HackTricks-u** ili **preuzmete HackTricks u PDF formatu** proverite [**SUBSCRIPTION PLANS**](https://github.com/sponsors/carlospolop)!
* Nabavite [**zvanični PEASS & HackTricks swag**](https://peass.creator-spring.com)
* Otkrijte [**The PEASS Family**](https://opensea.io/collection/the-peass-family), našu kolekciju ekskluzivnih [**NFT-ova**](https://opensea.io/collection/the-peass-family)
* **Pridružite se** 💬 [**Discord grupi**](https://discord.gg/hRep4RUj7f) ili [**telegram grupi**](https://t.me/peass) ili nas **pratite** na **Twitter-u** 🐦 [**@carlospolopm**](https://twitter.com/hacktricks\_live)**.**
* **Podelite svoje hakovanje trikove slanjem PR-ova na** [**HackTricks**](https://github.com/carlospolop/hacktricks) i [**HackTricks Cloud**](https://github.com/carlospolop/hacktricks-cloud) github repozitorijume.

</details>
