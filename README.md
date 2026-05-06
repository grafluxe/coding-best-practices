# Coding Best Practices

It's invaluable to have proven and agreed upon standards when working on a
codebase. Whether in a team or working solo on a project, an applications
architecture and the design patterns used to create it are key to a successful
execution. This document was created to serve as a language agnostic set of
rules to follow when developing. Note that most code examples are in written in
TypeScript.

## Contents

- [Respect the Best Practices of the Language You're Writing
  In](#respect-the-best-practices-of-the-language-youre-writing-in)
- [Be Consistent](#be-consistent)
- [Favor Readability](#favor-readability)
- [More on Readability](#more-on-readbility)
    - [A Note on Column Length](#a-note-on-column-length)
- [Favor Simplicity](#favor-simplicity)
- [Favor Immutability](#favor-immutability)
- [Favor Reusability](#favor-reusability)
- [Don't Repeat Yourself](#dont-repeat-yourself)
- [Document Your Code](#document-your-code)
- [Lint Your Code](#lint-your-code)
- [Use Test Driven Development](#use-test-driven-development)
- [Learn Functional Programming](#learn-functional-programming)
- [Use Pure Functions Whenever Possible](#use-pure-functions-whenever-possible)
- [More on Function Purity](#more-on-function-purity)
- [Inject Dependencies](#inject-dependencies)
- [Additional Considerations](#additional-considerations)
    - [Avoid Loops](#avoid-loops)
    - [Annotate Appropriately](#annotate-appropriately)
    - [Comment Your Regular Expressions](#comment-your-regular-expressions)
    - [Indent Properly](#indent-properly)
    - [Decrease Method Arguments](#decrease-method-arugments)
    - [Avoid Code Smells](#avoid-code-smells)
    - [Remember Conway's Law](#remember-conways-law)
- [Continue Learning](#continue-learning)
- [Finally](#finally)

***

## Respect the Best Practices of the Language You're Writing In

If the language you are writing in has a style guide, familiarize yourself with
it. If there's no official guide, learn about the common standards for your
language. While you shouldn't blindly follow a set of rules, you should
understand why they're in place and consider updating your style to match them.
Doing so makes it easier for other developers to follow your logic. Furthermore,
having such knowledge will give you confidence to break a rule if desired.

We often switch between languages throughout a project, so once you switch,
change gears so that you're coding in the style of the current language. Doing
so will go a long way in helping your team have unified code.

As a simple example, here's how you can create variables in:

#### TypeScript

```ts
const theGoodFoot: string[] = [];
const isOnIt = true;
```

The variable names are in camelCase and the optional semi-colons are included.
Notice that members with obvious types don't need their types annotated, as they
are inferred.

#### JavaScript

```js
const theGoodFoot = [];
const isOnIt = true;
```

The variable names are in camelCase and the optional [semi-colons are
included][js-semicolon].

#### Scala

```scala
val theGoodFoot: Array[String] = Array.empty[String]
private val isOnIt = true
```

The variable names are in camelCase and the optional semi-colons are excluded.
Notice that private members with obvious types don't need their
[types annotated][scala-types].

#### PHP

```php
$the_good_foot = [];
$is_on_it = true;
```

The variable names are in snake_case.

## Be Consistent

Consistency cements your coding style and greatly improves readability.

## Favor Readability

Pay attention to how you name your members and the length of your lines. Just
because you can convert a 10 line command into a single line, doesn't mean you
should. Remember that your team, and the future you, may need to revisit the
logic.

If you find yourself nesting multiple expressions, review your logic, as there's
probably a better way to write it (e.g. separate your logic into [small
meaningful methods][cfowler]). Follow the single responsibility principle and
strive to create atomic components (each doing as few things as possible).

For example, all three of the below code-blocks do the same thing:

#### Good

```ts
const isLegalAge = (age: number, hasPermit: boolean): boolean =>
  age >= 18 || (age >= 16 && hasPermit);

const hasGoodSight = (vision: [number, number]): boolean => vision[1] < 50;

const canUserDrive = (user: User): boolean => {
  const { licenseSuspended, age, hasLearnersPermit, vision } = user;

  return licenseSuspended
    ? false
    : isLegalAge(age, hasLearnersPermit) && hasGoodSight(vision);
};
```

The above logic is good because:

- The method names clearly express what they do.
- You can quickly understand what's happening due to separating concerns into
  individual methods.
- Each method is small and handles a single responsibility.
  - Having atomic methods promotes reusability.
- As a bonus, I use [destructuring] to remove the need for
  prepending `user` to the relevant properties.

#### Bad

```ts
const drive = (user: User): boolean => {
  if (user.licenseSuspended) {
    return false;
  } else {
    if (user.age >= 18 || (user.age >= 16 && user.hasLearnersPermit)) {
      if (user.vision[1] < 50) {
        return true;
      } else {
        return false;
      }
    } else {
      return false;
    }
  }
};
```

The above logic is bad because:

- The method name does not clearly express what it does (and is acutally
  misleading in this example).
- There are redundant booleans/returns.
- There's a lot of nesting, making it harder to quickly understand the logic.

#### Bad

```ts
const driveable = (user: User): boolean =>
  !user.licenseSuspended &&
  (user.age >= 18 || (user.age >= 16 && user.hasLearnersPermit)) &&
  user.vision[1] < 50;
```

The above logic is bad because:

- The method name does not clearly express what it does.
- Too much is crammed into one line.
- The logic is hard to understand.

## More on Readability

As Jessica Kerr states in her [FP presentation][jkerr], "familiar code doesn't
equal readable code." Just because you can make sense of logic due to being used
to how it's written, doesn't mean that it's actually easy to read.

Also:

- Use verbs to help clarify a methods action.
- Declare variables where you need them (in the proper scope and next to its
  first use).
- Keep each method as short and focused as possible.
- Set a max column length that aligns with your team and the language you're
  using.

### A Note on Column Length

Setting a narrow column is valuable to enforcing short and clean code.
Additionally, it improves readability -- this is why newspapers to this day
break a page into multiple narrow columns. I find it best to follow the rule
of [80 characters per line][max-chars]; it not only pushes me to write better
code, but easily allows for split screen development. As developers, we spend
most of our time reading code, so it's good practice to set a max length to one
that's optimized for ease of readability.

## Favor Simplicity

Follow [KISS principles][kiss] (Keep It Simple) for improved understanding and
debugging. If you can simplify complex logic, even if it takes a few more lines
of code, do so; your team and future self will appreciate it. Additionally, the
QA and test processes will be smoother if any bugs are found.

## Favor Immutability

Whenever possible, keep your code immutable. Doing so improves testability,
readability, and forces you to really think about each line of code you write.
It makes your code more [predictable][aruhaak] by preventing errors caused by
cases where a variable changes unexpectedly in the middle of running a process.
The more mutable code you have, the higher the risk of errors in your
application.

Example 1:

#### Good

```ts
const greenColor = "green";

// greenColor = "red"; // Will error (since using 'const')
```

#### Bad

```ts
let greenColor = "green";

greenColor = "red"; // Will pass (since using 'let')
```

--

Example 2:

#### Good

```js
const colors = Object.freeze(["red", "green", "blue"]);

colors = []; // Will fail (since using 'const')
colors[0] = "purple"; // Will fail (since array is frozen)

const newColors = colors.concat("yellow"); // Doesn't mutate original variable
```

#### Bad

```js
let colors = ["red", "green", "blue"];

colors = []; // Will pass (since using 'let')
colors[0] = "purple"; // Will pass (since array is mutable)

arr.push(4); // Mutates original variable
```

--

Don't merely make your code immutable because people say it's good practice,
make it immutable so that you (and your team) [can better understand and change
it][khenney].

## Favor Reusability

Keep your methods short and strive to increase modularity by making functions
reusable. Many of us are accustomed to passing values into a method (via
parameters) in order to make them more reusable, consider also [passing in
behaviors][swlaschin].

For example, we rarely hard-code methods like this:

```ts
const countToTen = () => {
  for (let i = 1; i <= 10; i++) {
    console.log("#" + i);
  }
};

// Usage
countToTen();
```

We often create parameters to make methods more dynamic, like this:

```ts
const countTo = (n: number) => {
  for (let i = 1; i <= n; i++) {
    console.log("#" + i);
  }
};

// Usage
countTo(10);
```

We should pass in the behaviors too:

```ts
const countTo = (n: number, fn: (n: number) => void) => {
  for (let i = 1; i <= n; i++) {
    fn("#" + i);
  }
};

// Usage
countTo(10, console.log);
```

Many languages have methods like this built in: just think of a `map` and
`filter`.

## Don't Repeat Yourself

Being [DRY][dry] appropriately condenses code, promotes modularity, and
simplifies future updates (while decreasing chances for bugs caused by updating
logic in one place and missing it in another).

For example, if logic is needed more than once in a single file, create a method
*in that file* so that it can be executed repeatedly. If logic is needed across
multiple files or is complex (defined as a series of related methods), move it
into its own file/class/module.

## Document Your Code

Add inline documentation for all public methods. When it comes to comments,
make them count and make sure to maintain them as your code changes. If your
comment can be replaced with a proper code refactor, do so (e.g. improve a
method name or refactor an if-statement to clearly express what the logic does
without needing to add a comment). When using well named members, your logic
becomes self documenting, and you'll notice that comments become less important.
As [John Papa][jpapa] says, "explain in code, not in comments."

## Lint Your Code

If the language you're writing in has a linter, use it. Commit linting rules
into your project repo and add a linting step into your build scripts. Also,
consider implementing a [code formatter][prettier] if necessary.

## Use Test Driven Development

Consider a [test-driven approach][tdd] to fail fast and decrease chances of bugs
later in your application. Moreover, writing tests first forces you to better
create pure and independent methods -- using dependency injection to create
autonomous and more clear code (with less mocks). Start with unit tests, then
move to integration, feature, et al.

Here are some valuable points from [Randy Shoup][rshoup] in regards to TDD:

- The upfront investment increases the final outputs chance of success.
- The more constrained in time and resources, the more important it is to build
  it right the first time. It's better to do it once really well, then do it
  twice half-baked.
- Tests make better code; they give you the confidence to break things and the
  courage to refactor.

## Learn Functional Programming

Alvin Alexander lists [general benefits][fp-benefits] as:

- Pure functions are easier to reason about.
- Testing is easier, and pure functions lend themselves well to techniques like
  property-based testing.
- Debugging is easier.
- Programs are more bulletproof.
- Programs are written at a higher level, and are therefore easier to
  comprehend.
- Function signatures are more meaningful.
- Parallel/concurrent programming is easier.

If coming from an [imperative][vsubramaniam] or [OOP background][fp2], put your
current knowledge in the back of your head as you familiarize yourself with
Functional Programming. It [solves problems using a different set of rules][fp].

## Use Pure Functions Whenever Possible

A method/function is pure when:

- It has no [side effects][side-effects] (is stateless).
  - It doesn't affect the outside world (e.g. no global variables, no database
  calls, no API requests, no file access, no logic directly tied to user input).
  - It handles exceptions gracefully (no THROWing).
- [Its output only depends on its input.][aalexander]
  - It doesn't hide dependencies nor mutate data.
- It's [referentially transparent.][ref-trans]
  - Its output is always the same when given the same arguments.
  - Its output can replace the actual function call without changing behavior.

Note that methods which interact with a data/time object or ones that print to
the screen are considered impure. So are singleton class variables, since they
[act like global objects][jspahr].

## More on Function Purity

Be idempotent whenever possible -- making the same method call multiple times
should always produce the same result. Additionally, if your method has multiple
paths of executions (if/switch statements), consider splitting your method.

#### Good

```ts
class Preview {
  respond = (catId: string): string => getPreview(catId).body;

  respondWithMetrics = (catId: string, metrics: Metrics, log: LogR): string => {
    metrics.createMeter();
    log.init();

    const preview = this.respond(catId);

    log.complete();
    metrics.exit();

    return preview;
  };
}

// Usage
if (logger) {
  new Preview().respondWithMetrics("vpaid", new Metrics(), logger);
} else {
  new Preview().respond("vpaid");
}
```

The above logic is good because:

- It separates different paths of execution.
- It's easy to read.
- It more clearly defines different use cases.

#### Bad

```ts
class Preview {
  respond = (catId: string, metrics?: Metrics, log?: LogR): string => {
    if (metrics && log) {
      metrics.createMeter();
      log.init();

      const preview = getPreview(catId).body;

      log.complete();
      metrics.exit();

      return preview;
    } else {
      return getPreview(catId).body;
    }
  };
}

// Usage
if (logger) {
  new Preview().respond("vpaid", new Metrics(), logger);
} else {
  new Preview().respond("vpaid");
}
```

The above logic is bad because:

- I combines 2 paths of execution into one (requires optional params).
- It's harder to read.
  - If the names weren't self documenting, it wouldn't be as clear which
    parameters are needed for preview generation versus which are needed for
    collecting metrics.
- It's signature doesn't make it clear that *both* the `metrics` and `log`
  parameters are needed for metrics to be calculated.

## Inject Dependencies

Dependency injection improves your codes purity and makes it easier to
understand. Strive to create methods that you [can understand][fp3] based on its
name and parameters. When calling a method, you shouldn't have to care about the
implementation and should know what its output will be based on how it's called.

Furthermore, only pass in data that's needed; do not send in an entire object as
an argument when only one property (from that object) is needed.

#### Good

```ts
class Team {
  gender = (genderId: number): string => (genderId === 0 ? "female" : "male");
}

// Usage
new Team().gender(database.genderId);
```

The above logic is good because:

- Its output solely depends on its input.
- It does not affect the outside world.
- Based on its signature, it's clear what the method does and what data it needs
  to do it.

#### Bad

```ts
class Team {
  private database = new Database();

  gender = (): string => (this.database.genderId === 0 ? "female" : "male");
}

// Usage
new Team().gender();
```

The above logic is bad because:

- It affects the outside world (by accessing the database internally).
- It's signature does not provide enough detail to understand the
  implementation.

#### Bad

```ts
class Team {
  gender = (db: Database): string => (db.genderId === 0 ? "female" : "male");
}

// Usage
new Team().gender(database);
```

The above logic is bad because:

- It's expecting the entire database object when only one property from it is
  needed.

## Additional Considerations

### Avoid Loops
Avoid loops of any kind (while, for, et. al) and replace with built-in pure
equivalents (or write [recursive methods][recursive]).

#### Good

```ts
const addDollarSign = (nums: number[]): string[] => nums.map((n) => "$" + n);

// Usage
addDollarSign([3, 7]); // ["$3", "$7"]
```

#### Bad

```ts
const addDollarSign = (numbers: number[]): string[] => {
  const updated: string[] = [];

  for (const n of numbers) {
    updated.push("$" + n);
  }

  return updated;
};

// Usage
addDollarSign([3, 7]); // ["$3", "$7"]
```

### Annotate Appropriately

If using a statically typed language, use types in all public members and when
they help document the code, but not when they're obvious.

```ts
// Annotations helps understandability
const relatedPages: number[] = pages.getRelevant(book.currentPage);
const userNode: NodeAsset | null = user.getNode(placement);
const reportByLines = (reportId: number): Promise<string[]> => { ... }

// No annotation needed since the types are clear
const name = "James";
const age = 85;
const add5 = (n: number) => n + 5;
```

### Comment Your Regular Expressions

Regex can be hard to read, so add comments to help explain what they do (and/or
use _named capturing groups_ if supported).

```ts
const subject = "He's not too fancy but his line is pretty clean?";
const encoded = subject
  .replaceAll("[?.]", "!") // Replaces all '?' or '.' with '!'
  .replaceAll("(not\\s+too|pretty)\\s+", "") // Removes all blacklisted words
  .replaceAll("but|or|if", "and"); // Replaces all conjunctions with 'and'
```

### Indent Properly

Understand the rules around indentation and whitespace and follow common
practices where present.

#### Good

```ts
const name: string = "James";
const calculateAge = (dob: User["birthday"]): number => dob.toYears;
const getLunarPhase = (
  month: number,
  date: number,
  year: number,
  phase: Phase,
): string => {
  return phase.calc(month, day, year).pull({ mode: 1 });
};

if (!baz) {
  foo("bar");
}
```

#### Bad

```
const name:string = "James";
const name :string = "James";
const name : string  =  "James";
const calculateAge =( dob: User [ "birthday" ] ) :number=>dob.toYears;
const getLunarPhase = (month: number,
  date: number, year: number,
  phase: Phase): string => {
  phase.calc(month,day,year).pull({mode:1})
};

if(!baz){
  foo("bar")
}
if(! baz){
    foo("bar")
}
```

Use two space indentation over four, and no tabs.

### Decrease Method Arguments

If your method has many arguments, it's probably a sign of code smell, so
revisit its implementation. If you still need many arguments in your method
after a proper refactor, consider grouping related objects together.

Note that if your arguments extend pass your max character limit, break them
into individual lines for improved readability.

#### Good

```ts
const asset: PlacementAsset = placement.getAsset(
  projectScope,
  requestParams(Params.id, Params.age, Params.gender),
  metaData,
);
```

#### Bad

```ts
const asset: PlacementAsset = placement.getAsset(
  projectScope,
  Params.id,
  Params.age,
  Params.gender,
  metaData.time,
  metaData.geo,
  metaData.areaCode,
);
```

### Avoid Code Smells

Familiarize yourself with this [list of code smells][code-smell] and aim to
write code in way that avoids them.

### Remember Conway's Law

Although more tightly coupled to an applications architecture, it's worth being
aware of your teams structure as you work on your project.
[Conway's law][conway] is defined as:

"Organizations which design systems are constrained to produce designs which are
copies of the communication structures of these organizations."

## Continue Learning

Be open to gaining knowledge from languages that you may not be well versed in.
For example, if you're a [JavaScript developer][js-fp] who wants to learn more
about Functional Programming, don't be intimidated to read up on how
[Haskell solves problems][haskell].

## Finally

Care about the code you write... and choose a set of styles that you can stick
with to create an environment you enjoying working in... all while using
patterns and principles which help you, your team, and your projects flourish.


[conway]: https://www.thoughtworks.com/insights/blog/applying-conways-law-improve-your-software-development
[aalexander]: https://alvinalexander.com/scala/fp-book/pure-functions-and-io-input-output
[ref-trans]: https://en.wikipedia.org/wiki/Referential_transparency
[side-effects]: https://dzone.com/articles/side-effects-1
[prettier]: https://prettier.io/
[scala-style-guide]: https://docs.scala-lang.org/style/
[scala-types]: https://docs.scala-lang.org/style/types.html
[js-semicolon]: http://2ality.com/2011/05/semicolon-insertion.html
[kiss]: https://en.wikipedia.org/wiki/KISS_principle
[dry]: https://en.wikipedia.org/wiki/Don%27t_repeat_yourself
[jkerr]: https://youtu.be/pMGY9ViIGNU?t=2186
[jpapa]: https://youtu.be/56mETnrByBM?t=642
[max-chars]: https://en.wikipedia.org/wiki/Characters_per_line#In_programming
[tdd]: https://blog.testlodge.com/what-is-tdd/
[rshoup]: https://youtu.be/E8-e-3fRHBw?t=528
[jspahr]: https://youtu.be/7AqXBuJOJkY?t=1403
[swlaschin]: https://youtu.be/srQt1NAHYC0?t=1276
[fp-benefits]: https://alvinalexander.com/scala/fp-book/benefits-of-functional-programming
[fp]: https://medium.com/@cscalfani/so-you-want-to-be-a-functional-programmer-part-1-1f15e387e536#59c5
[fp2]: https://sidburn.github.io/blog/2016/03/14/immutability-and-pure-functions
[fp3]: https://arnhem.luminis.eu/pure-bliss-with-pure-functions-in-java/
[cfowler]: https://youtu.be/rNQR1HqfEl0?t=342
[recursive]: https://www.safaribooksonline.com/library/view/learning-scala/9781449368814/ch04.html#recursive_functions_section
[code-smell]: https://en.wikipedia.org/wiki/Code_smell#Common_code_smells
[method-line-breaks]: https://docs.scala-lang.org/style/indentation.html#methods-with-numerous-arguments
[curly-braces]: https://docs.scala-lang.org/style/control-structures.html#curly-braces
[destructuring]: http://twitter.github.io/effectivescala/#Functional%20programming-Destructuring%20bindings
[khenney]: https://youtu.be/APUCMSPiNh4?t=3683
[aruhaak]: https://youtu.be/6f5dt923FmQ?t=459
[vsubramaniam]: https://youtu.be/RRcnFd9TbjI?t=871
[haskell]: http://learnyouahaskell.com
[js-fp]: https://mostly-adequate.gitbooks.io/mostly-adequate-guide
