# SOLID Design Principles

Set of principles to generate quality object-oriented code, originally introduced by Robert C. Martin.

+ Single Responsibility Principle
  + A class should only have one reason to change
  + Separation of concerns: Different classes handling different, independents tasks/problems
+ Open-Closed Principle
  + Classes should be **Open** for extension but **Closed** for modification
+ Liskov Substitution Principle
  + You should always be able to substitute a base type for a subtype
+ Interface Segregation Principle
  + Don't put too much into an interface; split it into separate interfaces
  + *YAGNI* - You ain't gonna need it
+ Dependency Inversion Principle
  + High-level modules should not depend upon low-level ones; use abstractions and don't couple high-level modules to internal implementation details of low-level modules.

## Single Responsibility Principle

A class should have only one responsibility and therefore, it should only have one reason to change that is somehow related to its responsibility.

For example, this class manages a journal with entries.

```javascript
class Journal {
  constructor() {
    this.entires = {};
  }

  addEntry(text) {
    let c = ++Journal.count;
    let entry = `${c}: ${text}`
    this.entries[c] = entry
    return c
  }

  removeEntry(index) {
    delete this.entries;
  }

  toString() {
    return Object.values(this.entries).join('\n');
  }
}

Journal.count = 0;

let j = new Journal();
j.addEntry('I cried today');
j.addEntry('I ate a bug');
console.log(j.toString())
```

Now imagine that we want to add functionalities that save and load the journal to and from files in the file system, or from external URLs, it might be tempting to use this same class to manage that.

```javascript
import fs from 'fs';

class Journal {
  constructor() {
    this.entries = {};
  }

  addEntry(text) {
    let c = ++Journal.count;
    let entry = `${c}: ${text}`
    this.entries[c] = entry
    return c
  }

  removeEntry(index) {
    delete this.entries;
  }

  toString() {
    return Object.values(this.entries).join('\n');
  }

  save(filename) {
    fs.writeFileSync(filename, this.toString())
  }

  load(filename) {
    // ...
  }

  loadFromUrl(url) {
    // ...
  }
}

Journal.count = 0;

let j = new Journal();
j.addEntry('I cried today');
j.addEntry('I ate a bug');
console.log(j.toString())
```

We might have other classes that might want to use the saving and loading functionality, this will lead to having duplicate responsibilities, it is a better idea to extract this behavior and generalize it to handle different types of objects.

Let's create a class called `PersistenceManager`

```javascript
class PersistenceManager {
  saveToFile(entity, filenmae) {
    fs.writeFileSync(filename, entity.toString())
  }
}
```
 
 This way we keep functionalities and responsibilities where they belong, if we need to change or fix something, it's easier to find the place where we need to look. For example, if files aren't being stored correctly, we know that we have to checkout the `PersistenceManager` class rather than go through several other unrelated files.

 ```javascript
import Journal from 'journal';
import PersistenceManager from 'persistenceManager';

 let j = new Journal();
 let p = new PersistenceManager;

 j.addEntry('I cried today');
 j.addEntry('I ate a bug');

 let p = new PersistenceManager();
 let filename = '/home/pankas87/Documents/journal.txt'

 p.saveToFile(j, filename)
 ```

 There is an antipattern called "The God Object", a huge massive class that has many responsibilities, lots of spaghetti code that's difficult to figure out. The single reponsibility principle is the opposite of that antipattern.

 Another term we use constantly is "Separation of Concerns", the idea is to split a large set of behaviors into smaller, more manageable modules.

 ## Open-Closed Principle

The Open-Closed Principle states that objects are **open** for extension but **closed** for modification.

Let's look at it through an example.

 ```javascript
let Color = Object.freeze({
  red: 'red',
  green: 'green',
  blue: 'blue'
})

let Size = Object.freeze({
  small: 'small',
  medium: 'medium',
  large: 'large'
})

class Product {
  constructor(name, color, size) {
    this.name = name;
    this.color = color;
    this.size = size;
  }
}

class ProductFilter {
  filtereByColor(products, color) {
    return products.filter(p => p.color === color);
  }
}

let apple = new Product('Apple', Color.green, Size.small);
let tree = new Product('Tree', Color.green, Size.large);
let house = new Product('House', Color.blue, Size.large);

let products = [apple, tree, house];

let pf = new ProductFilter = new ProductFilter();
console.log(`Green Products (old)`);

for (let p of pf.filtereByColor(products, Color.green)) {
  console.log(`* ${p.name} is green`);
}
```

Now imagine that they ask us to also filter by size, we can go back to the ProductFilter class and a new method.

```
class ProductFilter {
  filtereByColor(products, color) {
    return products.filter(p => p.color === color);
  }

  filterBySize(products, size) {
    return products.filter(p => p.size === size);    
  }
}
```

Extension, in the context of the Open-Closed principle refers usually to inheritance, a class inheriting from another, and acquiring some of its methods and attributes. 

The whole idea of OCP is that once we have the original functionality of the class we don't modify it anymore.

We are breaking the OCP by adding another filtering method.

```javascript
class ProductFilter {
  filtereByColor(products, color) {
    return products.filter(p => p.color === color);
  }

  filterBySize(products, size) {
    return products.filter(p => p.size === size);    
  }

  filterBySizeAndColor(products, size, color) {
    return products.filter(p => p.size === size && p.color === color);
  }
}
```

Space is not infinite and this will not scale at some point if we keep getting new requests to filter products by certain criteria, we will get a massive and unamangeable class.

This take us to the specification pattern that allows to write something that's modular and easy to work with.

Let's write a specification class

```javascript
class ColorSpecification {
  constructor(color) {
    this.color = color;
  }

  isSatisfied(item) {
    return item.color == color;
  }
}

class SizeSpecification {
  constructor(size) {
    this.size = size;
  }

  isSatisfied(item) {
    return item.size == size;
  }
}
```

This might be a bit of an overkill for this case, but we now have independent filters, not tied one to another, that we can use and combine independently.

Now, if we have a new criteria we just create a new specification class.

Let's build a new filter that's based on the specifications.

```javascript
class BetterFilter {
  filter(items, spec) {
    return items.filter(x => spec.isSatisfied(x));
  }
}

let bf = new BetterFilter();
let greenColorSpecification = new ColorSpecification(Color.green)
console.log(`Green products (new): `);

for (let p of bf.filter(products, greenColorSpecification)) {
  console.log(`${p.name} is green`);
}
```

Now let's create a combinator specification that allows to combine multiple specifications into one.

```javascript
class AndSspecification {
  constructor(...specs) {
    this.specs = specs;
  }

  isSatisfied(item) {
    return this.specs.every(x => x.isSatisfied(item))
  }
}
```

Let's create a combined specification that filters products that are large and green.

```javascript
let spec = new AndSpecification(
  new ColorSpecification(Color.green),
  nee SizeSpecification(Size.large)
)

for (let p of bf.filter(products, spec)) {
  console.log(`${p.name} is large and green`);
}
```

We could keep expanding this by adding an `OrSpecification` or even an `XorSpecification` and similar stuff.

To summarize, the idea is that classes should be **OPEN** for extension but **CLOSED** for modification, we should never jump into an existing class and start to modify it unless we really have to, because of a bug for example; but rather, we should extend this class.

There are several reasons for this, when we modify a class we are changing the expected and tested behaviors that are used all along our codebase, therefore introducing unexpected behaviors and errors in unrelated parts of our program. On the other hand, if we extend the original class, we are not affecting the original implementation used by other people or even ourselves in other parts of the program.

## Liskov Substitution Principle

Named after american computer scientist Barbara Liskov.

This principle states that if a method or function takes a base type as an argument it should also be able to take a derived type.

```javascript
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  getArea() {
    return this.width * this.height;
  }

  toString() {
    return `${this.width}x${this.height}`
  }
}

class Square extends Rectangle {
  constructor(size) {
    super(size, size)
  }
}

let rc = new Rectangle(2, 3);
console.log(rc.toString());

let sq = new Square();
console.log(sq.toString());
sq.width = 10;
sq.width = 5;
```

In this example, we could manually modify the width and the height of the square, making it possible to turn it into a rectangle rather than a true square, we need to control that the width and height of the squares are the same.

It might be tempting to use getters and setters to control the ratio of width to height in the squares but this could lead to having functions that work with a Rectangle but fail completely with a derived class like a Square.

```javascript
class Rectangle {
  constructor(width, height) {
    this._width = width;
    this._height = height;
  }

  get width() { return this._width; }
  get height() { return this._height; }

  set width(value) { this._width = value; }
  set height(value) { this._height = value; }

  get area() {
    return this._width * this._height;
  }

  toString() {
    return `${this._width}×${this._height}`;
  }
}

class Square extends Rectangle {
  constructor(size) {
    super(size, size);
  }

  set width(value) {
    this._width = this._height = value;
  }

  set height(value) {
    this._width = this._height = value;
  }
}

let useIt = (rc) => {
  let width = rc._width;
  rc.height = 10;
  console.log(
    `Expected area of ${10*width}, ` +
    `got ${rc.area}`
  )
};

let rc = new Rectangle(2,3);
let sq = new Square(5);

useIt(rc) // Expected area of 20, got 20
useIt(sq) // Expected area of 50, got 100
```

This implementation violates the Liskov Substitution Principle because the `useIt` function does not works in one way with a `Rectangle` object and works in a different way for the `Square` object.

May be using the `Rectangle` class as the base for the `Square` class we could get rid of the getters and setters and may be have a *Factory* method that builds the squares correctly.

// TODO: In the first part, add a summary on each list item, describing every principle

## Interface Segregation Principle

To understand the Interface Segregation Principle let's take a look at this example where we have a `Document` class, a `Machine` abstract class from which we derive two classes, a `MultiFunctionPrinter` and an `OldFashionedPrinter`

```javascript
class Document {

}

class Machine {
  constructor() {
    if (this.constructor.name === 'Machine') {
      throw new Error('Machine is abstract!');
    }
  }

  print(doc) {}

  fax(doc) {}

  scan(doc) {}
}

class MultiFunctionPrinter extends Machine {
  print(doc) {
    //
  }

  fax(doc) {
    //
  }

  scan(doc) {
    //
  }
}

class OldFashionedPrinter extends Machine {
  print(doc) {
    // ok
  }

  fax(doc) {
    // do nothing
  }

  scan(doc) {
    // do nothing 
  }
}
```

The problem with this approach comes when we start implementing the `OldFashionedPrinter` class which cannot scan or fax documents, we can leve the methods blank but this violates the *Principle of Least Surprise* that states that people using our APIs they should not be surprised, they should not see some magic, unexpected behavior or a lack of behavior; we don't want users of our code to be surprised by getting some weird data or behavior, we want them to get predictable results.

By leaving the `fax` and `scan` methods empty we are violating said principle, producing unexpected behavior.

We could throw an exception when the non implemented methods are invoked.

```javascript
class OldFashionedPrinter extends Machine {
  print(doc) {
    // ok
  }

  fax(doc) {
    throw new Error('not implemented!');
  }

  scan(doc) {
    throw new Error('not implemented!');
  }
}
```

But this approach is still unfriendly, and the reason is that by defininf the interface of the `Machine` abstract class to be full of stuff we don't need we are forcing the clients to leave the methods blank or to throw errors, neither is a particualrly good approach. This is where the **Interface Segregation Principle** comes in.

The **Interface Segregation Principle** means that you have to segregate interfaces into different parts so people don't implement more than what they need. 

```javascript
var aggregation = (baseClass, ...mixins) => {
  class base extends baseClass {
    constructor (...args) {
      super(...args);
      mixins.forEach((mixin) => {
        copyProps(this,(new mixin));
      });
    }
  }
  let copyProps = (target, source) => {  // this function copies all properties and symbols, filtering out some special ones
    Object.getOwnPropertyNames(source)
      .concat(Object.getOwnPropertySymbols(source))
      .forEach((prop) => {
        if (!prop.match(/^(?:constructor|prototype|arguments|caller|name|bind|call|apply|toString|length)$/))
          Object.defineProperty(target, prop, Object.getOwnPropertyDescriptor(source, prop));
      })
  };
  mixins.forEach((mixin) => {
    // outside constructor() to allow aggregation(A,B,C).staticFunction() to be called etc.
    copyProps(base.prototype, mixin.prototype);
    copyProps(base, mixin);
  });
  return base;
};

class Printer
{
  constructor()
  {
    if (this.constructor.name === 'Printer')
      throw new Error('Printer is abstract!');
  }

  print(){}
}

class Scanner
{
  constructor()
  {
    if (this.constructor.name === 'Scanner')
      throw new Error('Scanner is abstract!');
  }

  scan(){}
}

class Photocopier extends aggregation(Printer, Scanner)
{
  print()
  {
    // IDE won't help you here
  }

  scan()
  {
    //
  }
}

// we don't allow this!
// let m = new Machine();

let printer = new OldFashionedPrinter();
printer.fax(); // nothing happens
//printer.scan();
```

## Dependency Inversion Principle

The *Dependency Inversion Principle* defines a relationship that we should have between *low level* and *high level* modules.

The *dependency inversion principles* states that high-level modules should not directly depend on low-level modules, specifically, they should not depend on internal implementation details that should be private and they should instead depend on abstractions. 

By abstractions we mean abstract classes or interfaces, in typed languages, or in specific duck types, in the case of Javascript and other dynamic languages.

Let's look at it through an example.

```javascript
let Relationship = Object.freeze({
  parent: 0,
  child: 1,
  sibling: 2
});

class Person {
  constructor(name) {
    this.name = name;
  }
}

// LOW LEVEL MODULE: Close to the metal, for example, related to data storage
class Relationships {
  constructor() {
    this.data = [];
  }

  addParentAndChild(parent, child) {
    this.data.push({
      from: parent,
      type: Relationship.parent,
      to: child
    });
  }
}

let parent = new Person('John');
let child1 = new Person('Chris');
let child2 = new Person('Matt');

// HIGH LEVEL MODULE: Related to working with and processing data, for example
class Research {
  constructor(relationships) {
    // Find all children of John
    let relations = relationships.data;
    let childrenOfJohn = relations.filter(r => r.from.name === 'John' && r.type === Relationship.parent)

    for (let rel of childrenOfJohn) {
      console.log(`John has a child named ${rel.to.name}`)
    }
  }
}

let rels = new Relationships();
rels.addParentAndChild(parent, child1);
rels.addParentAndChild(parent, child2);

new Research(rels)
```

In the case of the `Research` class we are violating the principle by making it depend on the data storage of the `Relationships`, when we access directly to the `relationships.data` array rather than using some intermediate abstraction. This is not a good idea because we are binding our high-level module to an internal implementation detail of the low-level module that could change in the future; we could change the way that we store the data in `Relationships` and break the inner workings of the `Research` class.

The *Dependency Inversion Principle* aims to reduce strong coupling between low and high level modules.

```javascript
let Relationship = Object.freeze({
  parent: 0,
  child: 1,
  sibling: 2
});

class Person {
  constructor(name) {
    this.name = name;
  }
}

class RelationshipBrowser {
  constructor() {
    if (this.constructor.name === 'RelationshipBrowser')
      throw new Error('RelationshipBrowser is abstract');    
  }

  findAllChildrenOf(name) {}
}

// LOW LEVEL MODULE: Close to the metal, for example, related to data storage
class Relationships extends RelationshipBrowser {
  constructor() {
    super()
    this.data = [];
  }

  addParentAndChild(parent, child) {
    this.data.push({
      from: parent,
      type: Relationship.parent,
      to: child
    });
  }

  findAllChildrenOf(name) {
    return this.data.filter(
      r => r.from.name === 'John' && r.type === Relationship.parent
    ).map(r => r.to);
  }
}

// HIGH LEVEL MODULE: Related to working with and processing data, for example
class Research {
  constructor(browser) {
    // Find all children of John
    for (let p of browser.findAllChildrenOf('John')) {
      console.log(`John has a child named ${p.name}`)
    }
  }
}


let parent = new Person('John');
let child1 = new Person('Chris');
let child2 = new Person('Matt');

let rels = new Relationships();
rels.addParentAndChild(parent, child1);
rels.addParentAndChild(parent, child2);

new Research(rels);
```