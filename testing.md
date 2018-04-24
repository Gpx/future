# Testing

We have been testing our React components with `enzyme`. This has a few issues, manily the fact that it tests the internal workings of a component (e.g. `expect().toHaveState()`) instead of its output and tests only a mocked version of the components (when using shallow rendering).

A more complete approach to testing in react is provided by [`react-testing-library`](https://github.com/kentcdodds/react-testing-library). The core principle for this library is the following:

> The more your tests resemble the way your software is used, the more confidence they can give you

We propose to replace `enzyme` with `react-testing-library` in order to have more robust tests.

## Fixture data

An issue we encounter often while testing—and that we'll see more with `react-testing-libraray`—is the fact that components sometimes take complex objects. Imagine a `UserCard` profile that expects a `User` has its prop.

Traditionally we would have created an user object right in the text:

_Note: we're going to use `react-testing-library`'s syntax_

```jsx
it("should show the User's name", () => {
  const user = { id: 1, name: 'John Doe' }
  const { getByText } = render(<UserCard user={user} />)
  expect(getByText(user.name}).toBeInTheDOM()
})
```

This approach works but has a problem: we're not passing a "real" user to our component but a subset. Although the test still passes we want it to be as close as possible to a real life application.

Writing all the properties for a user can get tedious though, and if some properties change it will take a lot of time to update all our tests.

For this we propose an approach that is a porting of [Fabrication](https://www.fabricationgem.org/)—a Ruby library for testing—called `fabricator`. The `fabricator` library has two main components: `Fabricator` to define how objects are shaped and `Fabricate` to create new objects.

Here's how you would define the shape of a user:

```js
import { Fabricator, faker } from 'fabricator'

Fabricator('user', {
  id: () => faker.random.id(),
  name: () => faker.name.firstName() + ' ' + faker.name.lastName(),
  is_admin: false,
  // the other props...
})
```

And this is our test updated to take advantage of it:

```jsx
import { Fabricate } from 'fabricator'

it("should show the User's name", () => {
  const user = Fabricate('user')
  const { getByText } = render(<UserCard user={user} />)
  expect(getByText(user.name}).toBeInTheDOM()
})
```

## Snapshot testing

Note that this won't work with snapshot testing because the generated user is everytime different. An easy fix is to set a seed at the beginning of each test file:

```js
import { faker } from 'fabricator'
beforeEach(() => faker.seed(1))
```

For an example of how to test with `fabricator` check out [travelperk/web #1276](https://github.com/travelperk/web/pull/1276).