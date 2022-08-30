## class Diagram
The class diagram is the main building block of object-oriented modeling. It is used for general conceptual modeling of the structure of the application, and for detailed modeling translating the models into programming code. Class diagrams can also be used for data modeling. The classes in a class diagram represent both the main elements, interactions in the application, and the classes to be programmed.

Mermaid can render class diagrams.
```mermaid
classDiagram
    Animal <|-- Duck
    Animal <|-- Fish
    Animal <|-- Zebra
    Animal : +int age
    Animal : +String gender
    Animal: +isMammal()
    Animal: +mate()
    class Duck{
        +String beakColor
        +swim()
        +quack()
    }
    class Fish{
        -int sizeInFeet
        -canEat()
    }
    class Zebra{
        +bool is_wild
        +run()
    }
```
## class
UML provides mechanisms to represent class members, such as attributes and methods, and additional information about them. A single instance of a class in the diagram contains three compartments:

    The top compartment contains the name of the class. It is printed in bold and centered, and the first letter is capitalized. It may also contain optional annotation text describing the nature of the class.
    The middle compartment contains the attributes of the class. They are left-aligned and the first letter is lowercase.
    The bottom compartment contains the operations the class can execute. They are also left-aligned and the first letter is lowercase.
```mermaid
classDiagram
    class BankAccount
    BankAccount : +String owner
    BankAccount : +Bigdecimal balance
    BankAccount : +deposit(amount)
    BankAccount : +withdrawal(amount)
```

## Define a class

There are two ways to define a class:

- Explicitly defining a class using keyword class like class Animal. This defines the Animal class
- Define two classes via a relationship between them Vehicle <|-- Car. This defines two classes Vehicle and Car along with their relationship.

```mermaid
classDiagram
    class Animal
    Vehicle <|-- Car
```

## Defining Members of a class

UML provides mechanisms to represent class members, such as attributes and methods, and additional information about them.

Mermaid distinguishes between attributes and functions/methods based on if the parenthesis () are present or not. The ones with () are treated as functions/methods, and others as attributes.

There are two ways to define the members of a class, and regardless of whichever syntax is used to define the members, the output will still be same. The two different ways are :

 - Associate a member of a class using : (colon) followed by member name, useful to define one member at a time. For example:

```mermaid
classDiagram
class BankAccount
BankAccount : +String owner
BankAccount : +BigDecimal balance
BankAccount : +deposit(amount)
BankAccount : +withdrawal(amount)
```
- Associate members of a class using {} brackets, where members are grouped within curly brackets. Suitable for defining multiple members at once. For example:

```mermaid
classDiagram
class BankAccount{
    +String owner
    +BigDecimal balance
    +deposit(amount)
    +withdrawal(amount)
}
```
## Return Type
Optionally you can end the method/function definition with the data type that will be returned (note: there must be a space between the final ) of the method definition and return type example:

```mermaid
classDiagram
class BankAccount{
    +String owner
    +BigDecimal balance
    +deposit(amount) bool
    +withdrawal(amount) int
}
```
## Generic Types

Members can be defined using generic types, such as **List<int>**, for fields, parameters and return types by enclosing the type within ~ (tilde). Note: nested type declarations (such as **List<List<int>>**) are not currently supported

This can be done as part of either class definition method:
```mermaid
classDiagram
class Square~Shape~{
    int id
    List~int~ position
    setPoints(List~int~ points)
    getPoints() List~int~
}

Square : -List~string~ messages
Square : +setMessages(List~string~ messages)
Square : +getMessages() List~string~
```
## Return Type

Optionally you can end the method/function definition with the data type that will be returned
## Visibility

To specify the visibility of a class member (i.e. any attribute or method), these notations may be placed before the member's name, but it is optional:

-  <font color="orange"> + </font>Public
-  <font color="orange"> - </font>Private
-  <font color="orange"> # </font>Protected
-  <font color="orange"> ~ </font>Package/Internal



note you can also include additional classifiers to a method definition by adding the following notations to the end of the method, i.e.: after the ():

- <font color="orange"> * </font>Abstract e.g.: someAbstractMethod()*
- <font color="orange"> $ </font>Static e.g.: someStaticMethod()

note you can also include additional classifiers to a field definition by adding the following notations to the end of the field name:

- <font color="orange"> $ </font>Static e.g.: String someField

## Defining Relationship

A relationship is a general term covering the specific types of logical connections found on class and object diagrams.

Type|Description
--|:--:|
<|-- |	Inheritance
*-- |	Composition
o-- |	Aggregation
--> |	Association
-- 	 |   Link (Solid)
..> |	Dependency
..|> |	Realization
.. 	 |   Link (Dashed)

```mermaid
classDiagram
classA <|-- classB
classC *-- classD
classE o-- classF
classG <-- classH
classI -- classJ
classK <.. classL
classM <|.. classN
classO .. classP
```

We can use the labels to describe nature of relation between two classes. Also, arrowheads can be used in opposite directions as well :

```mermaid
classDiagram
classA --|> classB : Inheritance
classC --* classD : Composition
classE --o classF : Aggregation
classG --> classH : Association
classI -- classJ : Link(Solid)
classK ..> classL : Dependency
classM ..|> classN : Realization
classO .. classP : Link(Dashed)
```
##Labels on Relations

It is possible to add a label text to a relation:

 - [classA][Arrow][ClassB]:LabelText
```mermaid
classDiagram
classA <|-- classB : implements
classC *-- classD : composition
classE o-- classF : aggregation
```
## Two-way relations

Relations can go in multiple ways:
Where Relation Type can be one of:
Type|Description
--|:--:|
<\| |Inheritance
* | Composition
o 	|Aggregation
> 	|Association
< 	|Association
|> 	|Realization

And <font color="orange"> Link </font> can be one of:
Type|Description
--|:--:|
-- |	Solid
.. |	Dashed

Cardinality / Multiplicity on relations

Multiplicity or cardinality in class diagrams indicates the number of instances of one class linked to one instance of the other class. For example, one company will have one or more employees, but each employee works for just one company.

Multiplicity notations are placed near the ends of an association.

The different cardinality options are :

- <font color="orange">1</font> Only 1
- <font color="orange">0..1</font> Zero or One
- <font color="orange">1..*</font> One or more
- <font color="orange">\*</font> Many
- <font color="orange">n n</font> {where n>1}
- <font color="orange">0..n</font> zero to n {where n>1}
- <font color="orange">1..n</font> one to n {where n>1}
Cardinality can be easily defined by placing cardinality text within quotes " before(optional) and after(optional) a given arrow.
```mermaid
classDiagram
    Customer "1" --> "*" Ticket
    Student "1" --> "1..*" Course
    Galaxy --> "many" Star : Contains
```

## Annotations on classes

It is possible to annotate classes with a specific marker text which is like meta-data for the class, giving a clear indication about its nature. Some common annotations examples could be:

- <font color="orange">\<\<Interface\>\></font> To represent an Interface class
- <font color="orange">\<\<abstract\>\></font> To represent an abstract class
- <font color="orange">\<\<Service\>\></font> To represent a service class
- <font color="orange">\<\<enumeration\>\></font> To represent an enum

Annotations are defined within the opening << and closing >>. There are two ways to add an annotation to a class and regardless of the syntax used output will be same. The two ways are :

- In a ***separate line*** after a class is defined. For example:
```mermaid
classDiagram
class Shape{
    <<interface>>
    noOfVertices
    draw()
}
class Color{
    <<enumeration>>
    RED
    BLUE
    GREEN
    WHITE
    BLACK
}
```

## Comments

Comments can be entered within a class diagram, which will be ignored by the parser. Comments need to be on their own line, and must be prefaced with %% (double percent signs). Any text after the start of the comment to the next newline will be treated as a comment, including any class diagram syntax
```mermaid
classDiagram
%% This whole line is a comment classDiagram class Shape <<interface>>
class Shape{
    <<interface>>
    noOfVertices
    draw()
}
```
## Setting the direction of the diagram

With class diagrams you can use the direction statement to set the direction which the diagram will render like in this example.
```mermaid
classDiagram
  direction RL
  class Student {
    -idCard : IdCard
  }
  class IdCard{
    -id : int
    -name : string
  }
  class Bike{
    -id : int
    -name : string
  }
  Student "1" --o "1" IdCard : carries
  Student "1" --o "1" Bike : rides
```

