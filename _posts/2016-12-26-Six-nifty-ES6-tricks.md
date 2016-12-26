---
layout: post
title: "Six nifty ES6 tricks"
date: 2016-12-26
tags: [ES6]
categories: Front-End
---

In this blog post, I show six tricks enabled by new ES6 features. At the end of each section, I point to related material in my book “Exploring ES6” (which is free to read online).

Enforcing mandatory parameters via parameter default values
ES6 parameter default values are only evaluated when they are actually used. That lets you enforce that a given parameter be provided:

    /**
     * Called if a parameter is missing and
     * the default value is evaluated.
     */
    function mandatory() {
        throw new Error('Missing parameter');
    }
    function foo(mustBeProvided = mandatory()) {
        return mustBeProvided;
    }
The function call mandatory() is only made if the parameter mustBeProvided is missing.

Interaction:

    > foo()
    Error: Missing parameter
    > foo(123)
    123
More information:

Sect. “Required parameters” in “Exploring ES6”
Iterating over Array indices and elements via the for-of loop
Method forEach() lets you iterate over the elements of an Array. It also gives you each element’s index, should you want it:

    var arr = ['a', 'b', 'c'];
    arr.forEach(function (elem, index) {
        console.log('index = '+index+', elem = '+elem);
    });
    // Output:
    // index = 0, elem = a
    // index = 1, elem = b
    // index = 2, elem = c
The ES6 for-of loop is a loop that supports ES6 iteration (via iterables and iterators) and destructuring. If you combine destructuring with the new Array method entries(), you get:

    const arr = ['a', 'b', 'c'];
    for (const [index, elem] of arr.entries()) {
        console.log(`index = ${index}, elem = ${elem}`);
    }
arr.entries() returns an iterable over index-element pairs. The destructuring pattern [index, elem] gives us direct access to both components of each pair. The parameter of console.log() is a so-called template literal, which brings string interpolation to JavaScript.

More information:

Chap. “Destructuring” in “Exploring ES6”
Chap. “Iterables and iterators” in “Exploring ES6”
Sect. “Iterating with a destructuring pattern” in “Exploring ES6”
Chap. “Template literals” in “Exploring ES6”
Iterating over Unicode code points
Some Unicode code points (roughly, characters) comprise two JavaScript characters. For example, emojis:


Strings implement ES6 iteration. If you iterate over them, you get encoded code points (one or two JavaScript characters). For example:

    for (const ch of 'x\uD83D\uDE80y') {
        console.log(ch.length);
    }
    // Output:
    // 1
    // 2
    // 1
That gives you a way to count the number of code points in a string:

    > [...'x\uD83D\uDE80y'].length
    3
The spread operator (...) inserts the items “in” its operand into an Array.

More information:

Chap. “Unicode in ES6” in “Exploring ES6”
Sect. “The spread operator (...)” in “Exploring ES6”
Swapping variable values via destructuring
If you put two variables into an Array and then destructure that Array “into” the same variables, you can swap their values without needing an intermediate variable:

    [a, b] = [b, a];
It is conceivable that JavaScript engines will optimize this pattern in the future so that no Array is created.

More information:

Chap. “Destructuring” in “Exploring ES6”
Simple templating via template literals
ES6 template literals are more like string literals than like traditional text templates. But you can use them for templating if you return them from functions:

    const tmpl = addrs => `
        <table>
        ${addrs.map(addr => `
            <tr><td>${addr.first}</td></tr>
            <tr><td>${addr.last}</td></tr>
        `).join('')}
        </table>
    `;
The function tmpl (an arrow function) maps the Array addrs to a string. Let’s use tmpl() on the Array data:

    const data = [
        { first: '<Jane>', last: 'Bond' },
        { first: 'Lars', last: '<Croft>' },
    ];
    console.log(tmpl(data));
    // Output:
    // <table>
    //
    //     <tr><td><Jane></td></tr>
    //     <tr><td>Bond</td></tr>
    //
    //     <tr><td>Lars</td></tr>
    //     <tr><td><Croft></td></tr>
    //
    // </table>
More information:

Blog post “Handling whitespace in ES6 template literals”
Sect. “Text templating via untagged template literals” in “Exploring ES6”
Chap. “Arrow functions” in “Exploring ES6”
Simple mixins via subclass factories
If an ES6 class extends another class, that class is specified dynamically, via an arbitrary expression (not statically via an identifier):

    // Function id() simply returns its parameter
    const id = x => x;

    class Foo extends id(Object) {}
That allows you to implement a mixin as a function that maps a class C to a new class (with the mixin methods) whose superclass is C. For example, the following two functions Storage and Validation are mixins:

    const Storage = Sup => class extends Sup {
        save(database) { ··· }
    };
    const Validation = Sup => class extends Sup {
        validate(schema) { ··· }
    };
You can use them to compose a class Employee as follows.

    class Person { ··· }
    class Employee extends Storage(Validation(Person)) { ··· }
More information:

Sect. “Simple mixins” in “Exploring ES6”
Further reading
Two chapters of “Exploring ES6” give a good overview of ECMAScript 6:

An overview of what’s new in ES6
First steps with ECMAScript 6 [features that are easy to adopt]