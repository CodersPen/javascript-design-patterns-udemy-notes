# The Builder Pattern

## Gamma Categorization

+ Design patterns are split into three categories (Creational, Structural, Behavioral)
+ This is called *Gamma Categorization* after Erich Gamma, part of the GoF authors
+ **Creational Patterns**
  + Deal with the creation (construction) of objects
  + Explicit (constructor) vs. implicit (DI, reflection, etc)
  + Wholesale (single statement) vs. piecewise (step by step)
+ **Structural Patterns**
  + Concerned with the structure (e.g. class members)
  + Many patterns are wrappers that mimic the underlying class' interface
  + Stress the importance of good API design
+ **Behavioral Patterns**
  + They are all different, no central theme

## Overview

When construction gets a little bit too complicated

### Motivation

+ Some objects are simple an can be created in a single initializer call
+ Other objects require a lot of ceremony to create, the object has to be created in stages
+ Having an object with 10 initializer arguments is not productive
+ For complex initialization routines we opt for piecewise construction rather than having a large and complex constructor method
+ A *Builder* provides an API for constructing an object step-by-step

> Builder: When piecewise object construction is complicated, provide an API for doing it succintly

## Builder

A builder helps you to construct an object. This pattern is specifically used for really complicated objects that cannot be built in a single instantiation step. It helps us make the process of creating an object understandable, convenient and easy to use.

```javascript
const hello = 'hello';
let html = [];

html.push('<p>');
html.push(hello);
html.push('</p>');

console.log(html.join(''));
```

This is a bit too complex for the specific case of a paragraph but let's imagine that we now want to build a list of items.

```javascript
const words = ['hello', 'world'];
let html = []

html.push('<ul>\n');
for (let word of words) {
    html.push(`<li>${word}</li>\n`)
}
html.push('<(ul>');
console.log(html.join(''))
```

```javascript
const words = ['hello', 'world'];

class Tag {
  static get indentSize() { return 2; }

  constructor(name='', text='') {
    this.name = name;
    this.text = text;
    this.children = [];
  }

  toStringImpl(indent) {
    let html = [];
    let i = ' '.repeat(indent * Tag.indentSize);
    html.push(`${i}<${this.name}>\n`);
    if (this.text.length > 0) {
      html.push(' '.repeat(Tag.indentSize * (indent+1)));
      html.push(this.text);
      html.push('\n');
    }

    for (let child of this.children)
      html.push(child.toStringImpl(indent+1));

    html.push(`${i}</${this.name}>\n`);
    return html.join('');
  }

  toString() {
    return this.toStringImpl(0);
  }

  static create(name) {
    return new HtmlBuilder(name);
  }
}

class HtmlBuilder {
  constructor(rootName) {
    this.root = new Tag(rootName);
    this.rootName = rootName;
  }

  addChild(childName, childText) {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
  }

  toString() {
    return this.root.toString();
  }

  build() {
    return this.root;
  }
}

// Building manually
let builder = new HtmlBuilder('ul');
for (let word of words) {
  builder.addChild('li', word);
}
console.log(builder.toString());

// Bulilding with the static Tag#create method
let builder = Tag.build('ul');
```

```javascript
const words = ['hello', 'world'];

class Tag {
  static get indentSize() { return 2; }

  constructor(name='', text='') {
    this.name = name;
    this.text = text;
    this.children = [];
  }

  toStringImpl(indent) {
    let html = [];
    let i = ' '.repeat(indent * Tag.indentSize);
    html.push(`${i}<${this.name}>\n`);
    if (this.text.length > 0) {
      html.push(' '.repeat(Tag.indentSize * (indent+1)));
      html.push(this.text);
      html.push('\n');
    }

    for (let child of this.children)
      html.push(child.toStringImpl(indent+1));

    html.push(`${i}</${this.name}>\n`);
    return html.join('');
  }

  toString() {
    return this.toStringImpl(0);
  }

  static create(name) {
    return new HtmlBuilder(name);
  }
}

class HtmlBuilder {
  constructor(rootName) {
    this.root = new Tag(rootName);
    this.rootName = rootName;
  }

  addChildFluent(childName, childText) {
    let child = new Tag(childName, childText);
    this.root.children.push(child);
    return this;
  }

  toString() {
    return this.root.toString();
  }

  build() {
    return this.root;
  }

  clear() {
    this.root = new Tag(this.rootName);
  }
}

let builder = Tag.create('ul');
for (let word of words) {
  builder.addChildFluent('li', word);
}
console.log(builder.toString());

builder.clear();

builder
  .addChildFluent('li', 'foo')
  .addChildFluent('li', 'bar')
  .addChildFluent('li', 'baz');

console.log(builder.toString());
```

## Builder Facets

Sometimes we want to have different aspects of an object built by different builders.

```javascript
class Person {
  constructor() {
    // Address
    this.streetAddress = this.postCode = this.city = '';

    // Employment info
    this.companyName = this.position = '';
    this.annualIncome = 0;
  }

  toString() {
    return `Person lives at ${this.streetAddress}, ${this.city}, ${this.postcode}\n`
      + `and works at ${this.companyName} as a ${this.position} earning ${this.annualIncome}`;
  }
}

class PersonBuilder {
  constructor(person=new Person()) {
    this.person = person;
  }

  get lives() {
    return new PersonAddressBuilder(this.person);
  }

  get works() {
    return new PersonJobBuilder(this.person);
  }

  build() {
    return this.person;
  }
}

class PersonJobBuilder extends PersonBuilder {
  constructor(person) {
    super(person);
  }

  at(companyName) {
    this.person.companyName = companyName;
    return this;
  }

  asA(position) {
    this.person.position = position;
    return this;
  }

  earning(annualIncome) {
    this.person.annualIncome = annualIncome;
    return this;
  }
}

class PersonAddressBuilder extends PersonBuilder {
  constructor(person) {
    super(person);
  }

  at(streetAddress) {
    this.person.streetAddress = streetAddress;
    return this;
  }

  withPostcode(postcode) {
    this.person.postcode = postcode;
    return this;
  }

  in(city) {
    this.person.city = city;
    return this;
  }
}

let pb = new PersonBuilder();
let person = pb
  .lives.at('123 London Road').in('London').withPostCode('SW12BC')
  .works.at('Fabrikam').asA('Engineer').earning(123000)
.build();

console.log(person.toString());
```

## Summary

+ A builder is a separate component for building an object
+ Can either give builder an initializer or return it via a static function
+ To make a fluent builder, return self from every single method of the builder to chain method calls
+ Different facets of an object can be built up using different builders working in tandem via a base class