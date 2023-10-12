Copyright (c) 2023 Mildanach (Michal Rudnicki)

# Syntax
The basis of syntax is the **pair** expression - `subject:definition`
## Comments
Single backtick for one line comments, end-of-line terminates automatically.
Double backtick for multi-line comment.
```perfect
a:int `one-line comment
`NOTICE THAT` a = 6
``multi
    line
       comment``
```
## Identifier
A viable identifier for a variable, function, etc. consists of letters - lowercase or uppercase, digits, but not underscore - it is reserved as operator.
## Variable
```perfect
id:type = initValue
```
Not initializing a variable will yield a compilation error.
## Constant
```perfect
!constID:int = 1234
`is this confusing?`
!constBool:bool = !isThisConfusing()
```
## Array
`id:type[optsize] = { initValueList }`

**Example:**
```perfect
numbers:int[] = {1,2,3} `size deduced from list
names:string[2] = {"Joe","Mary"}
fill:int[5] = {0,1} `missing entries will be filled with default value (zero for integer)
bigNames:string[2048] = {"Francescino"} `empty strings in this case
```
## Operators
==TODO==
## Pointer/Reference
```perfect
alfa:int = 9
beta:@int = alfa `address-of operator is redundant here
beta = 3  `alfa is also 3 now!
```
## Function
`funcName:(paramList)(resultList) { funcBody }`
Function without parameters and results must be declared with one set of empty brackets to be explicit for compiler: `fn:() {}`
(Syntax `id:{}` may be reserved for something else)
You can use *result* like any other variable, and the last value assigned to it will be returned when function ends.
To return results immediately use explicit `out(result1:value1, result2:value2)` .
Including any of the results in return statement is optional. Omitted ones will hold last value assigned to them. Using non-result identifiers in the out-list is an error. Result identifiers may be omitted if all result values are listed in the out-list in correct order: `out(value1, value2)` .
Results must be initialized in one of possible ways or compilation would fail.

**Example:**
```perfect
swap:(param1:int, param2:int)(result1:int, result2:int)
{
	result1 = param2; result2 = param1
'OR'  out(result1:param2, result2: param1)
'OR'  out(param2, param1)
}
(a, b) = swap(5, 6)
```
### Default parameter value
```perfect
myFunction:(a:int=7, b:string="foo") {...}
```

### Function calling
```perfect
myFunction:(a:int, b:string) {...}
`call using only values
myFunction(7, "foo")
`call using only values
myFunction(a:7, b:"foo")
```
## Lambda
```perfect
lambdaFn = (input:int)(output:string) { out("<"_input_">") }
result:string = lambdaFn(6)
```
## Structure
Has only fields
```perfect
Box:Container { edge:real }
```
## Trait
It's just fn without body?
```perfect
computeArea:()(area:real)

Box.computeArea:()(area:real) { out(edge*edge) }
`if we do that without trait declaration, it's possible and is just method injection
```
## Class
```perfect
id:parentClass `parentClass is optional
{
	create:() {}
	destroy:() {}
	publicFieldsAndMethods
	.protectedFaM  `private but overridable
	..privateFaM   `strictly private
}
```

**Example:**
```perfect
Animal:
{
	species:string `public field
	
	create:(species0:string) `constructor
	{ species = species0 }
	
	destroy:() `destructor
	{}
	
	howl:() {...}
		
	..mass:real `private field
	
	.gainMass:(change:real) `protected method
	{...}	
}

`inheritance access modifier
Fox:.Animal {}
Furry:..Fox {}
```
**Final class:**
```
!A : .B {}

C : A {} `cannot inherit final class
```
## Interface
Has only methods declarations. May refer to Traits.
Maybe let's force interfaces as only way to publish methods?
```perfect
ICanineKind
{
	howl:()
	yap:()
}

Barkley:Animal
{
	ICanineKind.howl:() {...}
	ICanineKind.yap:()  {...} `repetition, but you see what interface it's from!
}

ICanineKind.howl:() {...} `error: can't implement interface method beyond a class
```

## Template
`templateID(templateParamsList):optParent {...}`
**Example:**
```perfect
Gun(AmmoType):Weapon
{
	.ammo:AmmoType
	getClip:()(clip:Clip(AmmoType)) {...}
}

fenfal:Gun(Winchester308)
```

```
`shared pointer template
sharp(type):
{
	..raw:@type
	..counter:@integer
	
	create:(initptr:@type)
	{
		raw = initptr
		counter = integer.new(1)
	}
	create:(src:@sharp(type))
	{
		this=src
	}
	downAndCheck:()
	{
		if(counter)
		{
			@counter--
			if(@counter==0)
			{
				counter.destroy()
				raw.destroy()
			}
		}
	}
	destroy:()
	{
		downAndCheck()
	}
	operator(=):(src:@sharp(type))(dest:@sharp(type))
	{
		if(raw!=in.raw)
		{
			downAndCheck()
			raw=in.raw
			counter=in.counter
			@counter++
		}
		out(this)
	}
}
```