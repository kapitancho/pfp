# pfp
PFP (PHP with Functional Programming) is a language that is aimed to transpile to PHP, bringing some nice FP features closer to the programmer.

All top level statements are just type definitions.

The basic scalar types are Int, Float, String and Bool.

Sample values:

a: Int = 1;
b: Float = 3.14;
c: String = "Hello";
d: Bool = true;

Additional types may be built as follows:

c: Int* = Int[1, 2, 3]; // an array of Ints
d: Float# = Float{"a": 3.14, "b": 5}; // a map of Floats
e: String? = "I am there";
f: String? = []; // a value simiar to NULL; see below

Product and sum types:

g: [Int, Float] = [1, 3.14]; //Tuple
h: {a: Int, b: Float} = {a: 1, b: 3.14}; //Record
i: Int|String = 7; //Union
j: Int|String = "I am a string";
k: [] = []; //The FP Unit type, also used as NULL
l: Any = "This could be of any type"

So T? is a shorthand of T|[].

The functions are also types:

m: (a: Int, b: Float) => String = { ... };

Single expression functions body can be also written without brackets and return keyword;
sum: (a: Int, b: Int) => Int = a + b;

All existing types may be used to build other types:

IntAndFloat = [Int, Float];
Cash = [];
Card = { cardNumber: CardNumber, cardType: CardType };
PaymentMethod = Cash|Card;
StringLengthFinder = (str: String) => Int;

Their values are constructed using their basic types:
n: IntAndFloat = IntAndFloat([1, 3.14]);
o: Cash = Cash();
p: Card = Card(
	cardNumber: CardNumber(12345), 
	cardType: CardType(Visa)
);
q: PaymentMethod = PaymentMethod(o);
r: StringLengthFinder = StringLengthFinder(str.length());


The enums are also types:

Suit = :{ Spades, Hearts, Clubs, Diamonds };
s: Suit = Suit(Spades);

Exception types are defined almost the same way as regular types.
Only exception type can be used with "throw":

ValueOutOfRange = @[Int]; //@ marks the type as "exception type".


Every type may have a validation constructor.

Nat = Int :> { 
	if (value < 0) throw ValueOutOfRange(value);
};

Note: ALL VALUES ARE IMMUTABLE!

Along with data types there are also service types:

- Interfaces are defined in the following way:

ProductRepository = !{ 
	ofId(productId: ProductId): Product?;
	persist(product: Product): [];
	remove(productId: ProductId): [];
};

- Classes are defined in the following way:

RedisProductRepository = {
	implements ProductRepository;

	redisConntecor: RedisConnector!;
	
	~mutableProperty: String#;

	ofId = { /*TODO*/ };
	persist = { /*TODO*/ };
	remove = { /*TODO*/ };
}

Classes can implement interfaces but cannot extend other classes.
All properties that are not a part of an implemented interface are not accessible from outside of the class.
RedisConnector! means that the value of this property will be automatically injected by a DI container.
RedisConnector is supposed to be an interface.
The only way to define a mutable property is by prefixing it with ~. It is still mutable only inside the instance.

The syntax for if, while, etc. is the same as in the other C-language families.

There is an additional construct used for the union and optional types (+ Any). Example:

Success = [];
Failure = {message: String};
Result = Success|Failure;

asText: (result: Result) => String = 
	result.value is {
		Success => "OK";
		f: Failure => f.message
	};

Another example:

asText: (value: Any) => String = 
	value is {
		i: Int => "Int(" . String(i) . ")";
		f: Float => "Float(" . String(f) . ")";
		s: String => "String(" . s . ")";
		b: Bool => "Bool(" . (b ? "true" : "false") . ")";
		Any => "Other Type"
	}

Immutability:

Since (almost) all values are immutable, all modifying operations on arrays and maps yield a new copy of the value.

oldList: Int* = Int[1, 2, 3];
newList: Int* = oldList.add(5); //[1,2,3,5]

oldList == newList; //false

oldMap: String# = String{"a": "Bull", "b": "Cow"};
newMap: String# = oldMap.remove("b"); //{"a": "Bull"}

All list, map and optional types are implicitly generic:

ints: Int* = Int[3, 5];
firstInt: Int? = ints[0]; //3
fifthInt: Int? = ints[4]; //[]

Random examples:

//Enum
CardType = :{Visa|MasterCard};

Cash = {};
Paypal = {emailAddress:EmailAddress};
CreditCard = {cardType:CardType, cardNumber: CardNumber};
Money = [Int];

PaymentMethod = Cash|Paypal|CreditCard;
Currency = :{EUR|USD|JPY};
PaymentAmount = [Money];
Payment = {amount: PaymentAmount; currency: Currency; method: PaymentMethod;};

//
NonNaturalInteger = Exception;

Natural = [Int] :> { if (value < 0) throw NonNaturalInteger(); }
EmailAddress = [String] :> { if (!value.match()) throw InvalidEmail(value); };
CardNumber = [String] :> { /* TODO - validate card number */ };

p: Payment = Payment(amount: PaymentAmount(120), currency: EUR, method: Cash());

generateInvoice: (payment: Payment) => Invoice = {
	return Invoice(/*TODO*/);
};

intArray: Int* = Int[1, 2, 3];
numericArray: (Int|Float)* = (Int|Float)[1, 3.14];
mixedArray: Any* = [1, 3.14, true];

floatMap: Float# = Float{"x": 1, "y": 2};

person1: [String, Int] = ["John", 31];
person2: {name: String, age: Int} = {name:"John", age: 31};

option1: String? = "John";
option2: String? = ();

saveInvoice: (invoice: Invoice) => () = { /*TODO*/ };
sumNumbers: (a: Int, b: Int) => Int = a + b;




