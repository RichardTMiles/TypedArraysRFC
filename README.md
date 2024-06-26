Howdy people, 

The pattern-matching RFC inspired me to write this. I have not done any work for this yet; just looking for initial feedback. I think we should have typed arrays in PHP. I propose adding the class SplTypeDefinedArray. A few considerations: A new syntax allowing Array interfaces will be needed. Functions should be allowed to return array interface types. How do we pass the interfaces to the constructor? Do we stick to traditional syntax, creating the object with new, or do we support a new array definition syntax? 

```php
interface iArrayA ['a' => string ]
interface iArrayB implements iArrayA ['b' => string, 'c' => ?string ]

$a = new SplTypeDefinedArray(iArrayB, [  // iArrayB::class is invalid 
  'a' => 'hello',
  'b' => 'world'
]);

// or 

$a : iArrayB = [  // this would implicitly initialize SplTypeDefinedArray
  'a' => 'hello',
  'b' => 'world'
];
```

If a typed array tries to define an index that does not exist it will throw a `RuntimeException: Index invalid`, which is consistant with the current implentation of `SplFixedArray`; If a type is nullable then it is not required to exist during construction.

Standard-type operators should be available. 
```
$a : iArrayA & iArrayB = [  // throws a RuntimeException since B is required in iArrayB
  'a' => 'fail'
];

$a : iArrayA | iArrayB = [
  'a' => 'profit'
];

$a : iArrayA &| iArrayB = [  // throws a RuntimeException since c does not exist in iArrayA and b is required in iArrayB
  'a' => 'fail'
  'c' => ''
];
```

I'm not sure what the best data structure would be. Rob Landers suggested a circular buffer in the pattern-matching email thread, but I'm open to anything. 

Best,

Richard Miles
