Howdy people, 

It was brought up to me that I should start a new revision of the discussion.

I'm looking for PHP Wiki Karma: wookieetyler

Should we have a syntax RFC or should I get started on the POC?

In the quest for typed arrays in PHP I beleive it is time to solidify the currently posed syntax:

https://github.com/php/php-src/compare/master...RichardTMiles:php-src:arrayTypes

```php
interface iArrayA ['a' => string ]
interface iArrayB extends iArrayA ['b' => string, 'c' => ?string, ‘d’ =>  SomeClass, ‘e’=>  ?iArrayA, ‘f’ => mixed ]

$array = ( iArrayA &| iArrayB ) [
  'a' => 'Hello'
];

class D {
	public ?iArrayB $exampleA;	// Array<iArrayA>
	public ?iArrayB[] $exampleB;	// Array<iArrayA>[]

  public function definedReturn(): iArrayA {
    return [
      'a' => 'World'
    ]
  }
}
```

Under the hood this could be a new implementation like SplObjectStorage and SplFixedArray. It may be easier to implement the type cast just set a property in the hash based array, and speed might be worth exploring both implementations as time complexity may exist casting a typed array back to a "normal" hash array. This is worth it's own discussion, so we will focus purely on syntctic capibillity, and/or lack their-of. 

https://www.php.net/manual/en/class.splfixedarray.php
https://www.php.net/manual/en/class.splobjectstorage.php

If a typed array tries to define an index that does not exist it will throw a `RuntimeException: Index invalid`, which is consistant with the current implentation of `SplFixedArray`; If a type is nullable then it is not required to exist during construction.

Standard-type operators should be available. 
```php
$a = iArrayA [
  'a' => 'Hello World'
];

$a = ( iArrayA & iArrayB ) [  // throws a RuntimeException since b is required in iArrayB
  'a' => 'Fail'
];

$a = ( iArrayA | iArrayB ) [
  'a' => 'Profit'
];

$a = ( iArrayA &| iArrayB ) [  // throws a RuntimeException since c does not exist in iArrayA and b is required in iArrayB
  'a' => 'fail'
  'c' => ''
];
```

Why have typed arrays at all? Array access is faster than object access:
https://github.com/EFTEC/php-benchmarks/blob/master/benchmark_array_vs_object.php
https://medium.com/cook-php/php-benchmark-time-fc19d813aa98

Array numeric no factory: 			        0%   
Array no factory 					              0.95%
Array numeric factory: 				          566.1%
Array factory: 						              650.07%
Object Constructor: 				            609.03%
Object no constructor 				          82.77%
Object no constructor setter/getter: 	  2058.43%
Object no constructor magic methods:	  2273.91%
Object no constructor stdClass: 		    112.53%

Considering a typed array could benifit from knowing the index positions ahead of time, this could mean a faster implemention (other than a hash table) could be appropreate. I see this becomming difficult with a complex type definitions e.g. ( iArrayA &| iArrayB | ( iArrayC & iArrayD ))

How does this differ from generics? Interfaces will be used with generics, but is just another brick in the wall. If anything this would be a building block twords generics. I pose an example:

```php
class A <T extends iArrayA>{
  public T $array = [
		‘a’ => ‘hello’        // this works
	];
}

class A <T extends iArrayB>{
  public T $array = [
		‘a’ => ‘hello’        // this fails with a RuntimeException
	];
}

interface iColorCode { 
  public const string RED = 'red';
  public const array PRINTF_ANSI_COLOR = [
    self::RED => "\033[31m%s\033[0m",
  ];
  public static function colorCode(string $message, string $color = self::RED): iArrayA;
}

class B <T extends iColorCode>{
  public ?iArrayA $arrayA;
  public function __construct(public T $config) {
    $this->arrayA = $config->colorCode('Hello World');
  }
}

``` 

Levi Morrison brought up that some work has been done parameterizing traits for anyone whoes intrested:
https://github.com/php/php-src/compare/master...morrisonlevi:php-src:parameterized_traits
I think this has solid work for that direction. I could see a world where PHP traits, interfaces, and classes could all be made generic. 

```php
interface iArrayA<T> [ 'a' => T ]

class A <T extends iArrayA<string>>{
  public T $array = [
		‘a’ => ‘hello’        // this works
	];
}
```

Casper Langemeijer said, "I cannot stand sitting through conference talks on 'generics' that only talk about 'collections'. This could be solved if we had typed arrays. If anything we would get better talks on Generics. :-)"

Thus, I think that the current feedback requires me bring genreics up specifically but I think that it is in fact, a seperate discussion with a seperate scope of work. Even if the type checking is marginally slower than the checks above, then we should still see performance over typed object properties. 

I'd like remind everyone that while generics would be cool, it is probably off topic. Issues relating to how this scopes to future work on generics is appropriate. Generally, points should be clear about syntax and how it could present problems or could be made better.

I think im ready to get started on the POC, but would like some feedback specifically from a karma granter as to how I should continue. 

___
I do have a working POC of Apache PHP-CGI Websockets (new function apache_connection_stream), so hopefully I can be granted karma for at least that :) 
https://github.com/php/php-src/compare/master...RichardTMiles:php-src:apache_connection_stream

Best,

Richard Miles
