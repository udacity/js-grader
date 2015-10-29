# Grader
Small library for intelligently grading student code and providing lots of feedback

## Installation
Just copy grader.js into whatever project/quiz you want to grade.

HTML/CSS grading depends on [jQuery](https://jquery.com/).

## Grader Philosophy

The grader helps you intelligently grade complex problems. In real life, teachers use branching logic to grade student work because even the simplest questions reflect a complex tree of dependent concepts.

When a teacher examines student work, she starts by grading the highest level aspects of a student's work and then works down to the details. If a student fails to understand a high level concept then there's usually little to gain by traversing down the tree to grade details based upon the higher level concept.

Teachers also make decisions while they grade. Based on the student's choices, they may want to examine different aspects of their work. For more complex questions, teachers may also grade against multiple questions simultaneously and compare multiple lines of evidence to determine student understanding.

Most importantly, **great teachers provide as much feedback as possible**. Teachers congratulate students on their successes and point them in the right direction when their understanding strays.

The grader exists to provide easy branching, checkpoint and feedback logic so you can easily grade student work the same way that classroom teachers grade their students.

See the [tips and tricks](#tips-n-tricks) section for grading strategies.

## API

### Instantiate a new Grader

```javascript
var grader = new Grader([default messages]);
```

#### Default Messages

```javascript
{category: "message string"}
```

Use `category` feedback to set shared feedback that multiple tests may output. This basically exists for the lazy who, when tasked with writing lots of tests with nearly identical feedback for wrong answers, decide to just write the feedback once in the constructor. In the actual tests, just set the `category` in the feedback object to the category you define here.

### Adding New Tests

```javascript
grader.addTest(callback, feedback object, [keepGoing boolean]);
```

New tests are added to an asynchronous queue that executes each tests's callback when the previous test's callback resolves to either `true` or `false`. As the queue is asynchronous, tests can take any amount of time to resolve.

#### `callback`
`callback` must return `true` or `false` indicating whether or not the test passed. You can determine `true` or `false` however you'd like, though there are some [helper functions](#helpers) in `Grader` that exist to make your life a little easier.

#### `feedback`

```javascript
{
	wrongMessage: "mandatory message for a failing test",
	comment: "optional string for a passing test",
	category: "optional cateogry name"
}
```

`wrongMessage` is mandatory and displayed if the test fails. `comment` is optional and displayed if the test passes. You can set the optional `category` equal to a category set in the Grader constructor in order to use a general message as feedback for a failing test.

#### `keepGoing`

If you want the grader to stop grading when a test fails, set `keepGoing` to `false`. It defaults to `true`.

Being able to stop the `Grader` gives you the ability to maintain a clear picture of the state of student code as you grade. Set `keepGoing` to `false` on any tests examining information on which later tests depend.

(Why call it `keepGoing`? I initially called it `checkpoint` and set this flag to `true` if I wanted the test to stop if it failed. That felt weird because I was using `true` to say "stop running." I switched to a terminology where I could set the flag equal to `false`, because `false` seems like a better way to say "stop here if something goes wrong!")

### Running Tests

```javascript
grader.runTests([options]);
```

Tests do not run automatically. Use `.runTests()` to get them started.

#### `options`

```javascript
{ignoreCheckpoints: true}
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

As mentioned earlier, tests execute in an asynchronous queue. Set an `onresult` handler to be called when the queue empties, whether it's because all of the tests executed or the queue was stopped at a checkpoint.

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

This one is easy to screw up. The `expectedInstance` should be an actual instance of the `value`'s prototype. The most common use is for determining if a variable is an array, like so:

```javascript
grader.isInstance([1,2,3,4], Array);		// true
grader.isInstance("array", Array);			// false
grader.isInstance([1,2,3,4], 'array');	// TypeError
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

#### For HTML/CSS

For these methods, assume that `elem` can be either a jQuery object or a string selector unless specified otherwise.

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
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

```javascript
grader.
```

## <a name="tips-n-tricks"></a> Tips and Tricks

* Have multiple ideas you want to grade independently? Just instantiate multiple `Grader`s!
* You can set up a series of tests by repeatedly adding tests then running them.



