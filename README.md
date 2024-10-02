# Embedded C implementation patterns for object oriented designs

## Introduction

The implementation patterns presented in this document do not constitute an
absolute truth but are the results of choices among several possibilities.  
These patterns can be taylored to fit a specific project needs.

The reader shall keep in mind that the choices were motivated by two
principles :
* Objects have to be robust.
* Objects are static and created at compilation time (no dynamic RAM allocation).

This document should be read one chapter after one other. The reader needs a
solid kwnoledge regarding C and OOD.

## Basis

This chapter deals with the basic patterns regarding a simple class having
attributes and public and private methods.

```
|-------------------------------|
|            My_Class           |
|-------------------------------|
| + Do_Something(               |
|       IN param_x:type_x,      |
|       OUT param_y:type_y )    |
| - Do_Something_Private(       |
|       IN param_z:type_z )     |
|-------------------------------|
| - attribute_1:type_1          |
| - attribute_2:type_2          |
|-------------------------------|
```

This class can be instanciated :
```
|-------------------------------|
|       My_Object:My_Class      |
|-------------------------------|
| - attribute_1 = 5             |
| - attribute_2 = 13            |
|-------------------------------|
```

Every class is implemented using a header file and a source file.

Every instance of the class (object) is implemented using a header file and a
source file.

### Attributes

The class itself is declared as a structure.  
Every attributes of the class are the fields of this structure.  
The structure is defined in the header file of the class.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Class declaration */
typedef struct {
    type_1 attribute_1;
    type_2 attribute_2;
} My_Class;

#endif
/*******************************************/
```

The order of the field is important to avoid the padding. The fields shall be
added to the structure from the larger to the smaller size.

### Public methods

When a method of a class is called, the code of the method expects a reference
to the instance to treat.  
This reference is a pointer to the object, transmitted as first parameter of
each operation of the class, with a name fixed by convention : _Me_.

The input parameters of the methods are passed by value while the output
parameters are passed by reference.

The public methods are :
* declared in the class header file,
* defined in the class source file.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Class declaration */
typedef struct {
    type_1 attribute_1;
    type_2 attribute_2;
} My_Class;

/* Public methods declaration */
void Do_Something(
    const My_Class* Me,
    type_x param_x,
    type_y* param_y );

#endif
/*******************************************/
```

```C
/*******************************************/
/* My_Class.c */
/*******************************************/
#include "My_Class.h"

/* Public methods definition */
void Do_Something(
    const My_Class* Me,
    type_x param_x,
    type_y* param_y )
{
    ...
}
/*******************************************/
```

### Private methods

Private methods are declared and defined in the source file of the class.  
They are _static_.

```C
/*******************************************/
/* My_Class.c */
/*******************************************/
#include "My_Class.h"

/* Private methods declaration */
static void Do_Something_Private(
    const My_Class* Me,
    type_z param_z );


/* Private methods definition */
static void Do_Something_Private(
    const My_Class* Me,
    type_z param_z )
{
    ...
}

/* Public methods definition */
...

/*******************************************/
```

### Objects

An object is declared in its header file and defined in its source file.

```C
/*******************************************/
/* My_Object.h */
/*******************************************/
#include "My_Class.h"

/* Object */
My_Class My_Object;
/*******************************************/
```

In the source file, the attributes are initialized.

```C
/*******************************************/
/* My_Object.c */
/*******************************************/
#include "My_Object.h"

/* Object */
My_Class My_Object = {
    .attribute_1 = 5,
    .attribute_2 = 13
};
/*******************************************/
```

## Constant and variable parts of a class

Actually all the attributes of a class are not variable. Some of them can be
constant.  
For robustness purpose, it is interesting to split the class in two parts : the variable and the constant.  
The variable part of the class is allocated in RAM while the constant part is
allocated in ROM.
The constant part has a reference to the variable part.

The split is done in the header file of the class.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Class variables */
typedef struct {
    type_1 attribute_1;
} My_Class_Var;

/* Class declaration */
typedef struct {
    const My_Class_Var* var_attr;
    const type_2 attribute_2;
} My_Class;

/* Public methods */
...

#endif
/*******************************************/
```

Obviously the definition of the object is also split.

The object itself is now constant.

```C
/*******************************************/
/* My_Object.h */
/*******************************************/

#include "My_Class.h"

/* Object */
const My_Class My_Object;
/*******************************************/
```

```C
/*******************************************/
/* My_Object.c */
/*******************************************/
#include "My_Object.h"

/* Object variables */
My_Class_Var My_Object_Var = {
    .attribute_1 = 5
};

/* Object */
const My_Class My_Object = {
    .var_attr = &My_Object_Var,
    .attribute_2 = 13
};
/*******************************************/
```

## Associations

A class can be associated to an other class.
It means that each instances of the class (object) will be linked to an
instance of the associated class (linked object).

```
|-------------------------------|           |-------------------------------|
|            My_Class           |           |         My_Other_Class        |
|-------------------------------|           |-------------------------------|
| + Do_Something( ... )         |  a_friend | + Do_Something_Else( ... )    |
|-------------------------------|---------->|-------------------------------|
| - ...                         |           | - ...                         |
| - ...                         |           | - ...                         |
|-------------------------------|           |-------------------------------|
```

_Note : to ease readability, parameters of the operations and attributes are
skipped._

The association is implemented by a constant reference to the linked object.  
The symbol of the referenced object is the name of the association.  
This reference belong to the constant part of the class.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Associated class inclusion */
#include "My_Other_Class.h"

/* Class declaration */
typedef struct {
    ...
    /* Linked object */
    const My_Other_Class* a_friend;
    ...
} My_Class;

#endif
/*******************************************/
```

The public methods of the linked object can be called by the methods of the
class. The first argument is the reference to the linked object.

```C
/*******************************************/
/* My_Class.c */
/*******************************************/
#include "My_Class.h"

/* ... */
void <Any_Method>(
    const My_Class* Me,
    ... )
{
    ...
    Do_Something_Else( Me->a_friend, ... );
    ...
}
/*******************************************/
```

When the object is defined, the adress of the linked object (declared in its
own file) is given.


```C
/*******************************************/
/* My_Other_Object.h */
/*******************************************/

#include "My_Other_Class.h"

/* Object */
const My_Other_Class My_Other_Object;
/*******************************************/
```

```C
/*******************************************/
/* My_Object.c */
/*******************************************/
#include "My_Object.h"

/* Linked object inclusion */
#include "My_Other_Object.h"

/* Object */
const My_Class My_Object = {
    ...
    /* Linked object */
    .a_friend = &My_Other_Object,
    ...
};
/*******************************************/
```

## Interfaces

An interface is a fully virtual class (abstract) that has no attributes. It
only defines a behaviour through operations.

```
|-------------------------------|
|         <<interface>>         |
|          An_Interface         |
|-------------------------------|
| + Do_Better( ... )            |
| + Do_More( ... )              |
|-------------------------------|
```

_Note : to ease readability, parameters of the operations as skipped._

An interface is declared in a specific header file as a structure of pointer to
functions.

As for the methods of a class, the input parameters are passed by value while
the output parameters are passed by reference.

```C
/*******************************************/
/* An_Interface.h */
/*******************************************/
#ifndef AN_INTERFACE_H
#define AN_INTERFACE_H

typedef struct {
    void (*Do_Better) ( ... );
    void (*Do_More) ( ... );
} An_Interface;

#endif
/*******************************************/
```

### Realization

An interface is realized by a class.

```
|-------------------------------|
|         <<interface>>         |
|          An_Interface         |
|-------------------------------|
| + Do_Better( ... )            |
| + Do_More( ... )              |
|-------------------------------|
              / \
               |

               |

               |
|-------------------------------|
|            My_Class           |
|-------------------------------|
| + Do_Something( ... )         |
|-------------------------------|
| - ...                         |
| - ...                         |
|-------------------------------|
```

The operations of an interface realized by a class are implemented as the
public methods of the class.  
The reference to the class is added as the first parameter on the function.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Class declaration */
typedef struct {
    ...
} My_Class;

/* Public methods */
...

/* Realized interfaces */
/* An_Interface */
void Do_Better(
    const My_Class* Me,
    ... );
void Do_More(
    const My_Class* Me,
    ... );

#endif
/*******************************************/
```

```C
/*******************************************/
/* My_Class.c */
/*******************************************/
#include "My_Class.h"

/* Public methods */
...

/* Realized interfaces */
/* An_Interface */
void Do_Better(
    const My_Class* Me,
    ... )
{
    ...
}

void Do_More(
    const My_Class* Me,
    ... )
{
    ...
}
/*******************************************/
```

Every object instanciated from a class that realizes an interface shall provide
an access to the realized operations.

```C
/*******************************************/
/* My_Object.h */
/*******************************************/
#include "My_Class.h"

/* Object */
const My_Class My_Object;

/* Realized interface */
const An_Interface An_Interface__My_Object;
/*******************************************/
```

```C
/*******************************************/
/* My_Object.c */
/*******************************************/
#include "My_Object.h"

/* Object */
const My_Class My_Object = {
    ...
};

/* Realized interface */
static void do_better( ... )
{
    Do_Better( &My_Object, ... );
}

static void do_more( ... )
{
    Do_More( &My_Object, ... );
}

const An_Interface An_Interface__My_Object = {
    .Do_Better = do_better,
    .Do_More = do_more
};
/*******************************************/
```

### Association

A class can be associated to an interface.

```
|-------------------------------|           |-------------------------------|
|            My_Class           |           |         <<interface>>         |
|-------------------------------|           |          An_Interface         |
| + Do_Something( ... )         |   an_ally |-------------------------------|
|-------------------------------|---------->| + Do_Better( ... )            |
| - ...                         |           | + Do_More( ... )              |
| - ...                         |           |-------------------------------|
|-------------------------------|
```

The association is implemented by a constant reference to the associated
interface.  
The symbol of the referenced interface is the name of the association.  
This reference belong to the constant part of the class.

```C
/*******************************************/
/* My_Class.h */
/*******************************************/
#ifndef MY_CLASS_H
#define MY_CLASS_H

/* Associated interface inclusion */
#include "An_Interface.h"

...

/* Class declaration */
typedef struct {
    ...
    /* Associated interface */
    const An_Interface* an_ally;
    ...
} My_Class;

#endif
/*******************************************/
```

The operations defined by the associated interface can be called by the methods
of the class.

```C
/*******************************************/
/* My_Class.c */
/*******************************************/
#include "My_Class.h"

/* ... */
void <Any_Method>(
    const My_Class* Me,
    ... )
{
    ...
    Me->an_ally->Do_Better( ... );
    ...
}
/*******************************************/
```

When the object is defined, the adress of the realized operations (realized by
an other object) is given.  

```C
/*******************************************/
/* Realizing_Object.h */
/*******************************************/
#include "Realizing_Class.h"

/* Object */
const Realizing_Class Realizing_Object;

/* Realized interface */
const An_Interface An_If__Realizing_Object;
/*******************************************/
```

```C
/*******************************************/
/* My_Object.c */
/*******************************************/
#include "My_Object.h"

/* Object (realizing interface) inclusion */
#include "Realizing_Object.h"

/* Object */
const My_Class My_Object = {
    ...
    /* Linked interface */
    .a_friend = &An_If__Realizing_Object,
    ...
};
/*******************************************/
```

_Note : these patterns can also be used if provider and requirer ports are used
in the design.  
Provider ports are equivalent to realized interfaces while requirer ports are
equivalent to associated interfaces._

## Events

An event is a signal that that is broadcast by an object to one or several
other objects.  
It can have some parameters.

```
|-------------------------------|
|           <<event>>           |
|        Something_Occured      |
|-------------------------------|
| + Param_1:type_1              | 
| + Param_2:type_2              | 
|-------------------------------|
```

An event signature is declared in a specific header file as a pointer of
function which arguments are the parameters of the event.  
The event parameters are passed by value.

```C
/*******************************************/
/* Something_Occured.h */
/*******************************************/
#ifndef SOMETHING_OCCURED_H
#define SOMETHING_OCCURED_H

typedef (void)(*Something_Occured)( type_1, type_2 );

#endif
/*******************************************/
```

### Event sending

An event is sent by a class.  

```
|-------------------------------|           |-------------------------------|
|         Sending_Class         |           |           <<event>>           |
|-------------------------------|           |        Something_Occured      |
| ...                           |   <<send>>|-------------------------------|
|-------------------------------|- - - - - >| + Param_1:type_1              | 
| ...                           |           | + Param_2:type_2              | 
|-------------------------------|           |-------------------------------|
```
The sending is implemented in the constant part of the class.

```C
/*******************************************/
/* Sending_Class.h */
/*******************************************/
#ifndef SENDING_CLASS_H
#define SENDING_CLASS_H

/* Sent event inclusion */
#include "Something_Occured.h"

...

/* Class declaration */
typedef struct {
    ...
    /* Sent event */
    Something_Occured Stg_Occured;
    ...
} Sending_Class;

#endif
/*******************************************/
```

The sending can be used by any method of the class.

```C
/*******************************************/
/* Sending_Class.c */
/*******************************************/
#include "Sending_Class.h"

/* ... */
void <Any_Method>(
    const Sending_Class* Me,
    ... )
{
    ...
    Me->Stg_Occured( value_1, value_2 );
    ...
}
/*******************************************/
```

An object send the actual event.

```
|-------------------------------|           |-------------------------------|
|  Sender_Object:Sending_Class  |           |           <<event>>           |
|-------------------------------|           |    Fired:Something_Occured    |
| ...                           |   <<send>>|-------------------------------|
|-------------------------------|- - - - - >| + Param_1:type_1              | 
| ...                           |           | + Param_2:type_2              | 
|-------------------------------|           |-------------------------------|
```

The event is defined in the source file of the object.  
When the event is sent, entry points provided by the objects receiving the
event are called (see Event reception).

```C
/*******************************************/
/* Sender_Object.c */
/*******************************************/
#include "Sender_Object.h"

/* Sent event signature inclusion */
#include "Something_Occured.h"

/* Sent event declaration */
static void Fired(
    type_1 param_1,
    type_2 param_2 );

/* Object */
const Sending_Class Sender_Object = {
    ...
    /* Sent event */
    .Stg_Occured = Fired,
    ...
};

/* Sent event definition */
static void Fired(
    type_1 param_1,
    type_2 param_2 )
{
    /* Receptor entry point */
}
/*******************************************/
```

### Event reception

An event is received by a class.

```
|-------------------------------|             |-------------------------------|
|        Receiving_Class        |             |           <<event>>           |
|-------------------------------|             |        Something_Occured      |
| ...                           |  <<receive>>|-------------------------------|
|-------------------------------|- - - - - - >| + Param_1:type_1              | 
| ...                           |             | + Param_2:type_2              | 
|-------------------------------|             |-------------------------------|
```

The event reception is implemented by a public function provided by the class
that has the same prototype as the event signature definition plus a constant
reference to the object as first parameter.

```C
/*******************************************/
/* Receiving_Class.h */
/*******************************************/
#ifndef RECEIVING_CLASS_H
#define RECEIVING_CLASS_H

/* Received event signature inclusion */
#include "Something_Occured.h"

...

/* Class declaration */
typedef struct {
    ...
} Receiving_Class;

/* Received event */
void Receiving_Class_Something_Occured(
    const Receiving_Class* Me,
    type_1 param_1,
    type_2 param_2 )

#endif
/*******************************************/
```

An object receive the actual event.

```
|-------------------------------|             |-------------------------------|
|Receiver_Object:Receiving_Class|             |           <<event>>           |
|-------------------------------|             |    Fired:Something_Occured    |
| ...                           |  <<receive>>|-------------------------------|
|-------------------------------|- - - - - - >| + Param_1:type_1              | 
| ...                           |             | + Param_2:type_2              | 
|-------------------------------|             |-------------------------------|
```

The actual event reception is implemented by a function declared in the object
header file and defined in the object source file.

```C
/*******************************************/
/* Receiver_Object.h */
/*******************************************/

/* Received event signature inclusion */
#include "Something_Occured.h"

/* Object */
const Receiving_Class Receiver_Object;

/* Event reception */
void Receiver_Object__Fired(
    type_1 param_1,
    type_2 param_2 );

/*******************************************/


```C
/*******************************************/
/* Receiver_Object.c */
/*******************************************/
#include "Receiver_Object.h"


/* Object */
const Receiving_Class Receiver_Object = {
    ...
};

/* Received event definition */
void Receiver_Object__Fired(
    type_1 param_1,
    type_2 param_2 )
{
    Receiving_Class_Something_Occured(
        &Receiver_Object,
        param_1,
        param_2);
}
/*******************************************/
```

## Specialization

## Polymorphism
