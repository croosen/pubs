# Typing in Python
Recently I started building some code for a program executed on AWS Lambda. Due to the nature of the program I ended up choosing Python as my language. 
As I am a real TypeScript enthousiast and a bit of a readable code addict, I noticed how much I missed types when writing the code. So as many of us I started 
Googling on how to use types in Python. 

## Creating types by creating classes
In my first attempt to create types, I created classes and stored them in a separate module for my project.
Like below example:

``` python
class FancyToteBagClass:
    def __init__(self, id: int, title: str, color: str, size: str):
        self.id: int = id
        self.title: str = title
        self.color: str = color
        self.size: str = size
```


This approach felt a little bit off. It's just not really typing and it seems like a lot of boiler plating.

## Using the typing module
I think Python agrees with that, and released a typing module with Python 3.5. And yay I was using Python 3.5+ so I immediately looked up the documentation to find out 
what this module does and if it would let me create custom types.

I did find a lot of different types. There is a whole list of predefined types and the trick is to leverage an existing type and make it your own. For example, you can 
use the `NewType` helper class like this:

``` python
from typing import NewType

UserId = NewType('UserId', int)
some_id = UserId(524313) # validates for 'int'
```

This simple use of typing will check if your newly created custom type is indeed an int. With NewType you derive from existing types. I wanted to create the FancyToteBag
and found out I can leverage the NamedTuple type like this:

``` python
class FancyToteBag(NamedTuple):
    id: int
    title: str
    color: str
    size: str
```    

This feels a lot cleaner to me. This way you can make use of decent type checking and the functions related to the NamedTuple type. You do not need to specify all args
in the init.

## Tradeoffs
However I think there might be a downside to this. After creating both custom types I checked the size of the objects created. 
The old school version is about 48 bytes and the new version uses 72 bytes... ouch. It seems we have stumbled on a tradeoff between ease of use/readability and performance here.

Link: [https://docs.python.org/3/library/typing.html](https://docs.python.org/3/library/typing.html)
