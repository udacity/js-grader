# Grader
Small library for intelligently grading student code and providing lots of feedback

## Installation
Just copy grader.js into whatever project/quiz you want to grade.

HTML/CSS grading depends on [jQuery](https://jquery.com/). Asynchronous testing depends on [Promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) (you'll need a [polyfill](https://github.com/jakearchibald/es6-promise/) for IE and other unsupported browsers).

## Grader Philosophy

The grader helps you intelligently grade complex problems. In real life, teachers use branching logic to grade student work because even the simplest questions reflect a complex tree of dependent concepts.

When a teacher examines student work, she starts by grading the highest level aspects of a student's work and then works down to the details. If a student fails to understand a high level concept then there's usually little to gain by traversing down the tree to grade details based upon the higher level concept.

Teachers also make decisions while they grade. Based on the student's choices, they may want to examine different aspects of their work. For more complex questions, teachers may also grade against multiple questions simultaneously and compare multiple lines of evidence to determine student understanding.

Most importantly, **great teachers provide as much feedback as possible**. Teachers congratulate students on their successes and point them in the right direction when their understanding strays.

The grader exists to provide easy branching, checkpoint and feedback logic so you can easily grade student work the same way that classroom teachers grade.

See the [tips and tricks](#tips-n-tricks) section for grading strategies.

## API

### Instantiate a new Grader

```javascript
var grader = new Grader([type string], [categories object]);
```

There are two options for `type`: `'sync'` or `'async'`. The Grader will run tests synchronously by default. If you have some tests that rely on network requests or other asynchronous actions, set type to `'async'`. Async tests are executed as a series of JavaScript [promises](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).


```javascript
categories = {category: "message string"}
```

Use the `categories` feedback to set shared feedback that multiple tests may output. This basically exists for the lazy who, when tasked with writing lots of tests with nearly identical feedback for wrong answers, decide to just write the feedback once in the constructor. In the actual tests, just set the `category` in the feedback object to the category you define here.

### Adding New Tests

```javascript
grader.addTest(callback, feedback object, [keepGoing boolean]);
```

New tests are added to the queue that executes each tests's callback when the previous test's callback resolves to either `true` or `false`. If you are using an async grader, tests can take any amount of time to resolve.

`callback` must return `true` or `false` indicating whether or not the test passed. You can determine `true` or `false` however you'd like, though there are some [helper functions](#helpers) in `Grader` that exist to make your life a little easier.

```javascript
feedback = {
	wrongMessage: "mandatory message for a failing test",
	comment: "optional string for a passing test",
	category: "optional cateogry name"
}
```

`wrongMessage` is mandatory and displayed if the test fails. `comment` is optional and displayed if the test passes. You can set the optional `category` equal to a category set in the Grader constructor if you want to use a general message as feedback for a failing test. If you have both a `category` and a `wrongMessage`, both will be displayed if a test fails.

If you want the grader to stop grading when a test fails, set `keepGoing` to `false`. It defaults to `true`.

Being able to stop the `Grader` gives you the ability to maintain a clear picture of the state of the execution environment as you grade. Set `keepGoing` to `false` on any tests examining information on which later tests depend.

(Why call it `keepGoing`? I initially called it `checkpoint` and set this flag to `true` if I wanted the test to stop if it failed. That felt weird because I was using `true` to say "stop running." I switched to a terminology where I could set the flag equal to `false`, because `false` seems like a better way to say "stop here if something goes wrong!")

### Running Tests

```javascript
grader.runTests([options object]);
```

Tests do not run automatically. Use `.runTests()` to get them started.

```javascript
options = {ignoreCheckpoints: boolean}
```

At the moment, there's only one option, `ignoreCheckpoints`.

This is purely a development feature. Use `ignoreCheckpoints` to run through every test regardless of any `keepGoing` flags. Great for checking feedback of all of your tests at once.

### Pulling Test Results

```javascript
grader.onresult = function (result) { ... };
```

where

```javascript
result = {
  isCorrect: boolean,
  testFeedback: ["array of", "feedback messages", "for wrong answers"],
  testComments: ["array of", "feedback messages", "for right answers"]
};
```

All of the tests must have passed in order for `isCorrect` to be `true`.

As mentioned earlier, tests execute in either a synchronous or asynchronous queue. Set an `onresult` handler to be called when the queue empties, whether it's because all of the tests executed or the queue was stopped at a checkpoint.

### <a name="helpers"></a> Helper Functions

All helper functions return `true` if conditions are met and `false` otherwise.

#### For JavaScript

```javascript
grader.isType(value any, expectedType string);
```

`isType` is just a wrapper around `typeof`.

---

```javascript
grader.isInstance(value any, expectedInstance prototype);
```

This one is easy to confuse. The `expectedInstance` should be an actual instance of the `value`'s prototype. The most common use is for determining if a variable is an array, like so:

```javascript
grader.isInstance([1,2,3,4], Array);		// true
grader.isInstance("array", Array);			// false
grader.isInstance([1,2,3,4], 'array');		// TypeError
```

---

```javascript
grader.isValue(value1 any, value2 any);
```

This helper runs a deep comparison of `value1` and `value2`.

---

```javascript
grader.isSet(value any);
```

Just checks to make sure the value is not `undefined`.

---

#### For HTML/CSS

For these methods, assume that `elem` can be either a jQuery element, regular DOM node, or a string selector unless otherwise specified.

---

```javascript
grader.hasCorrectTag(elem, tag string);
```

Always good to make sure you're looking at the right kind of element.

---

```javascript
grader.hasCorrectClass(elem, className string);
```

Gotta stay classy ;)

---

```javascript
grader.hasCorrectId(elem, id string);
```

Does pretty much what you'd expect it to.

---

```javascript
grader.hasAttr(elem, attrName string, [correctAttr string]);
```

This is called `hasAttr` and not `hasCorrectAttr` because the "correct" aspect is optional. Not all attributes have content, and as such the `correctAttr` is optional. Without the `correctAttr`, the test will pass if the attribute is found, regardless of whether or not it has content. With it, the attribute's content must match `correctAttr`.

---

```javascript
grader.hasCorrectStyle(elem, cssProperty string, [correctStyle string]);
```

This test pulls a CSS property from an element and compares the style to one or more `correctStyle`.

---

```javascript
grader.propertyIsLessThan(elem, cssProperty string, value int);
```

This test pulls a CSS property from an element and tests to see if the value of the property is less than the value specified.

---

```javascript
grader.propertyIsGreaterThan(elem, cssProperty string, value int);
```

This test pulls a CSS property from an element and tests to see if the value of the property is greater than than the value specified.

---

```javascript
grader.hasCorrectText(elem, text regex);
```

Runs a regex match against the `elem`'s text. If one or more match groups are returned, the test passes.

---

```javascript
grader.isCorrectElem(elem, correctElem $(element));
```

This test uses jQuery `.is()` to compare two elements. If they are the same, the test passes.

---

```javascript
grader.isCorrectCollection(collection, correctCollection $(collection));
```

Like `isCorrectElem`, this test compares two collections of elements. If they are the same, the test passes.

---

```javascript
grader.hasCorrectLength(elems, length number);
```

Compares the length of a collection of elements to a number. If the numbers match, the test passes.

---

```javascript
grader.elemDoesExist(elem);
```

`elem` is generally a string selector with this method. Simply checks to make sure that the selector returns one or more elements.

---

```javascript
grader.doesExistInParent(elem, parentElem);
```

Both `elem` and `parentElem` can be either strings or jQuery elements. This is a deep child search, meaning the children may be more than one level below the parent.

---

```javascript
grader.areSiblings(elem1, elem2);
```

If the two elements are siblings, the test passes. This test takes advantage of jQuery's `.siblings()`.

---

```javascript
grader.isImmediateChild(elem string, parentElem);
```

This is one of two methods where the `elem` needs to be a CSS selector, not a jQuery element. If the `elem` is a child of the `parentElem`, the test passes.

---

```javascript
grader.hasParent(elem string, parentElem);
```

Like `isImmediateChild`, this method expects a CSS selector for `elem`. This method uses jQuery's `.closest()`, which traverses up the DOM tree until it finds a parent that matches `parentElem`. If if finds a parent, the test passes.

---

```javascript
grader.sendResultsToExecutor();
```

Udacity internal only.

Packages up all test feedback to send to grading code. Use with programming quizzes running on PhantomJS with Karma.

---

## Examples:

(Note, these examples may be using older versions of the library so you may see some slight differences in the code. Namely, the `onresult` handler is brand new so you probably won't see it.)

* [Autocomplete quiz](https://github.com/udacity/course-web-forms/blob/gh-pages/lesson2/quizAutocomplete/grader/execution_files/unit_tests.js)
* [Datalists quiz](https://github.com/udacity/course-web-forms/blob/gh-pages/lesson1/quizDatalists/grader/execution_files/unit_tests.js)

## <a name="tips-n-tricks"></a> Tips and Tricks

* Remember, this is just a library. You're writing code, so use whatever logic and constructs you'd like to determine how tests get added, when tests get added, and what kind of feedback students should receive.
* Are there multiple ideas you want to grade independently? Just instantiate multiple Graders! Just make sure you aggregate the feedback before sending it off to grading code.
* Don't be afraid to set `keepGoing` to `false`. Checkpoints exist to help you run tests only when the conditions in the testing environment are correct. This also helps you provide targeted, focused feedback that addresses the current state of the student's code.
* Want to create an optional question? Use if/else logic to decide when to add a test and make sure the test returns true!
* A Grader hasn't disappeared after its queue empties. You can add more tests and run again! (Only for synchronous tests.)
