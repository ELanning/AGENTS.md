# AGENTS.md

Follow the following recommendations.

<table align="center">
  <tr>
    <td><strong>Code Design</strong></td>
  </tr>
</table>

**DO** use guard clauses and early returns.
```tsx
// GOOD.
if (!condition1)
    return;

if (!condition2)
    return;

[...]
```
```tsx
// BAD.
if (condition1) {
    if (condition2) {
        [...]
    }
}
```

**DO** extract deeply indented code into functions.
```tsx
// GOOD.
while (item = getNextItem()) {
    processItem(item);
}
```
```tsx
// BAD.
while (item = getNextItem()) {
    if (item.hasData) {
        ...
    }

    if (item.isAvailable) {
        ...
    }

    [...]
}
```

**DO** combine conditionals.
```tsx
// GOOD.
if (condition1 && condition2) {
    return result;
}
```
```tsx
// BAD.
if (condition1) {
    if (condition2) {
        return result;
    }
}
```

**DO** use map, filter, reduce.
```tsx
// GOOD.
const names = users.filter(u => u.active).map(u => u.name);
```
```tsx
// BAD.
const names = [];
for (const u of users) {
    if (u.active)
        names.append(u.name);
}
```

**CONSIDER** using functionals and dependency injection.
```tsx
// GOOD.
function writeTestOutput(format) {
    [...]
    return format(result);
}
```
```tsx
// BAD.
function writeTestOutput(formatType) {
    [...]
    if (formatType == 'markdown') {
        return formatAsMarkdown(result);
    } else if (formatType == 'xml') {
        return formatAsXml(result);
    }
    [...]
}
```

**DO** dissolve unnecessary conditionals and edge cases.
```tsx
// GOOD.
function distance(a, b) {
    return Math.hypot(b.x - a.x, b.y - a.y);
}
```
```tsx
//BAD.
function distance(a, b) {
    if (b.x == a.x && b.y == a.y) {
        return 0;
    }
    return Math.hypot(b.x - a.x, b.y - a.y);
}
```

**CONSIDER** adding concise summary comments over sections of code.
```tsx
// GOOD.
function makeRecipe() {
    // Set up the kitchen.
    turnOnOven(350);
    gatherIngredients(bread, mayo, salt, pepper, turkey, lettuce);

    // Make the sandwich.
    waitForOven().thenAdd(bread).thenWaitMinutes(5);
    const sandwich = combineIngredients(bread, mayo, salt, pepper, turkey, lettuce);

    // Deliver the meal.
    handOffToRobot(sandwich);
    sendToRoom(robot);
}
```
```tsx
// BAD.
function makeRecipe() {
    turnOnOven(350);
    gatherIngredients(bread, mayo, salt, pepper, turkey, lettuce);
    waitForOven().thenAdd(bread).thenWaitMinutes(5);
    const sandwich = combineIngredients(bread, mayo, salt, pepper, turkey, lettuce);
    handOffToRobot(sandwich);
    sendToRoom(robot);
}
```

**DO** document implicit assumptions and reasoning if it's not obvious in the code itself.
```tsx
// GOOD.
// This works because the user is in EST time, otherwise [...]
// This calculation works because the primes are [...]
```

**DO** structure code in an optimal way to reduce the total conditional count and indentation.
- This could result in moving `if` or `switch` blocks up or down the call stack.
- This could result in moving `try-catch` blocks up or down the call stack.

<table align="center">
  <tr>
    <td><strong>Function Design</strong></td>
  </tr>
</table>

**DO** write typed functions.
```tsx
// GOOD.
type Point = { x: number; y: number };

function distance(a: Point, b: Point): number {
    return Math.hypot(b.x - a.x, b.y - a.y);
}
```
```tsx
// BAD.
function distance(a, b) {
  return Math.hypot(b.x - a.x, b.y - a.y);
}
```

**CONSIDER** pure functions or functions with no observable side-effects. Push mutations to the top of the program.
```tsx
// GOOD.
function createItem(date) {
    [...]
    return Item(..., date: date);
}
```
```tsx
// BAD.
function createItem() {
    [...]
    return Item(..., date: now());
}
```

**DO** write simple functions.  
- Generally have 4±2 chunks of information.
- Use the right level of abstraction.
- Do not unnecessarily mix domains.
- Avoid boolean parameters.
```tsx
// GOOD.
function createHeading(fontSize, fontStyle, indentation) { [...] }
```
```tsx
// BAD.
function createHeading(isLargeHeading, isFancyFont, hasDeepIndentation, isAuthorizedUser, currentDate, specialCase, ...) { [...] }
```

**DO** validate inputs with types or checks in publicly exported functions.
```tsx
// GOOD.
function addFutureEvent(date) {
    assert date > now();
    [...]
}
```
```tsx
// BAD
function addFutureEvent(date) {
    // Assumes date is correct.
    [...]
}
```

<div style="text-align: center; border: 1px solid;">
  <div style="display: inline-block; padding: 8px; font-weight: bold;">
    Program Design
  </div>
</div>

**DO** use Functional Core, Imperative Shell designs.
- Push state and side effects up to the top level of the program.
- Keep the core logic pure and clean.

```tsx
// GOOD.
function main() {
    // Pure functional block.
    const user = createUser(...);
    // [...]

    // Mutations at the top, in a clear segment.
    saveUser(user);
    publishUserCreatedEvent(user);
}
```
```tsx
// BAD.
function main() {
    // Side effects merged and mixed throughout.
    const user = createAndSaveAndPublish(...);
    // [...]
}
```

**DO** partition the program into responsibilities with as clear boundaries as possible.

- In a video game, the render engine has no knowledge of the game's business logic.
- Model, view, controller is a standard separation of concern for UI.
- Accounting software might have a general math library that the tax portion uses, but does not mix with tax specific domain logic.

**DO** decide the main entities of the program and heavily document their properties and constraints.

<table align="center">
  <tr>
    <td><strong>Test Design</strong></td>
  </tr>
</table>

**AVOID** redundant tests
```tsx
// GOOD.
function testUserCalendarWorkflow() {
    const user = createUser(...);
    const calendarItem = createCalendarItem(...);
    
    addCalendarItem(user, calendarItem);

    assert user.getCalendarItems().contains(calendarItem);
}
```
```tsx
// BAD.
function testCreateUser() {
    const user = createUser(...);
    assert user != null;
}

function testCreateCalendarItem() {
    const calendarItem = createCalendarItem(...);
    assert calendarItem != null;
}

function testUserCalendarWorkflow() {
    // Same as above.
    [...]
}
```

**DO** decouple tests from test utilities and ensure the test is responsible for setting up what it is asserting.
```tsx
// GOOD.
function testUserHasPhoneNumber() {
    const user = getMockUser();

    setUserPhoneNumber(user, "(555) 555-5555");

    assert user.phoneNumber == "(555) 555-5555";
}
```
```tsx
// BAD.
function testUserHasPhoneNumber() {
    const user = getMockUser();
    assert user.phoneNumber == "(555) 555-5555";
}
```

**CONSIDER** structuring tests in a consistent manner, such as `Arrange-Act-Assert`. In a longer workflow, this may entail multiple `Arrange-Act-Assert` blocks in a row.
```tsx
// Good.
function testEmailWorkflow() {
    // Verify email.
    const inbox = getMockInbox("emailWorkflowTest@example.com");
    const user = getMockUser(email="emailWorkflowTest@example.com");

    sendEmailValidation(user);

    assert inbox.hasEmail(subject="Validate Your Account");

    // Verify inbox cleared.
    inbox.clear();

    assert !inbox.hasEmail(subject="Validate Your Account");
}
```
