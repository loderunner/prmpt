# Typelit

> "Type-safe template string literals"

[![NPM Version](https://img.shields.io/npm/v/typelit?logo=npm)](https://www.npmjs.com/package/typelit)
[![npm bundle size](https://img.shields.io/bundlephobia/minzip/typelit) ](https://bundlephobia.com/package/typelit)
[![dependency count](https://img.shields.io/badge/dynamic/json?url=https%3A%2F%2Fbundlephobia.com%2Fapi%2Fsize%3Fpackage%3Dtypelit&query=dependencyCount&label=dependencies)](https://bundlephobia.com/package/typelit)
[![GitHub Actions Workflow Status](https://img.shields.io/github/actions/workflow/status/loderunner/typelit/lint-test-build.yml?logo=github)](https://github.com/loderunner/typelit/actions/workflows/lint-test-build.yml?query=branch%3Amain)
[![GitHub License](https://img.shields.io/github/license/loderunner/typelit)](./LICENSE)

A type-safe string templating library for TypeScript that provides
strongly-typed variable references with support for nested paths. Create
template strings with compile-time type checking and automatic context type
inference.

Typelit allows you to write template strings that are both type-safe and easy to
read. Variables are referenced using a familiar template literal syntax while
maintaining full type information about the required context object structure.

## Table of contents

- [Getting Started](#getting-started)
  - [Installation](#installation)
  - [Basic Usage](#basic-usage)
  - [Nested Context Objects](#nested-context-objects)
- [Template Function](#template-function)
  - [Features](#features)
- [Variable Creators](#variable-creators)
  - [String](#string)
  - [Number](#number)
  - [Boolean](#boolean)
  - [BigInt](#bigint)
  - [Date](#date)
  - [JSON](#json)
- [Custom Variable Creators](#custom-variable-creators)
  - [Customizing String Conversion](#customizing-string-conversion)
- [Contributing](#contributing)
- [License](#license)

## Getting Started

### Installation

Install using npm:

```bash
npm install typelit
```

Or using yarn:

```bash
yarn add typelit
```

The library is available in multiple module formats to support different
environments:

- CommonJS (Node.js):

```typescript
const typelit = require('typelit');
const greeting = typelit`Hello ${typelit.string('name')}!`;
```

- ESM (Modern JavaScript):

```typescript
import typelit from 'typelit';
const greeting = typelit`Hello ${typelit.string('name')}!`;
```

- UMD (Browser):

```html
<script src="https://unpkg.com/typelit"></script>
<script>
  const greeting = typelit`Hello ${typelit.string('name')}!`;
</script>
```

Each format provides identical functionality - pick the one that best suits your
environment. The library includes TypeScript type definitions that work
automatically with all formats.

### Basic Usage

First, import the library:

```typescript
import typelit from 'typelit';
```

Let's start with a simple greeting template:

```typescript
// Create a simple template with one variable
const greeting = typelit`Hello ${typelit.string('name')}!`;

// Use the template with a context object
const result = greeting({ name: 'Alice' }); // "Hello Alice!"
```

### Nested Context Objects

You can also create templates with nested variable paths:

```typescript
// Create a template with nested variables
const template = typelit`Hello ${typelit.string('user', 'name')}! You are ${typelit.number('user', 'age')} years old.`;

// TypeScript knows exactly what shape the context object needs to have
const result = template({
  user: {
    name: 'Alice',
    age: 30,
  },
}); // "Hello Alice! You are 30 years old."

// This would cause a type error:
template({
  user: {
    name: 'Bob',
    // Error: missing required property 'age'
  },
});
```

## Template Function

The `typelit` tag function creates template functions that evaluate a string
template using values from a context object. The template function takes a
context object as input and returns the evaluated string with all variables
replaced by their values.

When creating a template, TypeScript infers the required shape of the context
object from the variables used in the template.

```typescript
// Example showing type inference
const welcome = typelit`Welcome back, ${typelit.string('username')}!`;

// TypeScript infers that the context must have a 'username' property
welcome({ username: 'alice' }); // OK
welcome({ user: 'bob' }); // Type error: missing username
welcome({ username: 123 }); // Type error: number is not assignable to string
```

### Features

The `typelit` function serves a dual purpose:

1. As a template tag function that creates typed template functions
2. As a namespace that provides variable creators (`string`, `number`, etc.)

Template functions created with `typelit` offer:

- **Type Safety**: All variables are fully typed, ensuring you can't pass the
  wrong type of value.
- **Path Inference**: TypeScript automatically infers the required structure of
  your context object.
- **Composition**: Templates can be composed to build more complex strings:

```typescript
const firstName = typelit`${typelit.string('user', 'firstName')}`;
const lastName = typelit`${typelit.string('user', 'lastName')}`;
const fullName = typelit`${firstName} ${lastName}`;

// Both templates require the same context structure
firstName({ user: { firstName: 'John' } });
fullName({ user: { firstName: 'John', lastName: 'Doe' } });
```

- **Compile-Time Validation**: TypeScript catches errors before runtime:
  - Missing or misspelled variable paths
  - Incorrect value types
  - Missing context properties
  - Invalid template syntax

## Variable Creators

Typelit provides built-in variable creators for common data types. These are
used to define variables in your templates with type-safe paths.

### String

Use `typelit.string()` to create string variables in your templates:

```typescript
// Simple string variable
const greeting = typelit`Hello ${typelit.string('name')}!`;
greeting({ name: 'Alice' }); // "Hello Alice!"

// Nested string variable
const userEmail = typelit`Contact: ${typelit.string('user', 'email')}`;
userEmail({ user: { email: 'alice@example.com' } }); // "Contact: alice@example.com"
```

### Number

Use `typelit.number()` to create number variables:

```typescript
// Simple number variable
const age = typelit`Age: ${typelit.number('age')} years old`;
age({ age: 25 }); // "Age: 25 years old"

// Nested number variable
const score = typelit`Score: ${typelit.number('game', 'score')} points`;
score({ game: { score: 100 } }); // "Score: 100 points"
```

### Boolean

Use `typelit.boolean()` to create boolean variables:

```typescript
// Simple boolean variable
const status = typelit`Status: ${typelit.boolean('isActive')}`;
status({ isActive: true }); // "Status: true"

// Nested boolean variable
const accountStatus = typelit`Account active: ${typelit.boolean('user', 'account', 'enabled')}`;
accountStatus({ user: { account: { enabled: false } } }); // "Account active: false"
```

### BigInt

Use `typelit.bigint()` to create bigint variables:

```typescript
// Simple bigint variable
const id = typelit`ID: ${typelit.bigint('userId')}`;
id({ userId: 9007199254740991n }); // "ID: 9007199254740991"

// Nested bigint variable
const transactionId = typelit`Transaction: ${typelit.bigint('payment', 'transactionId')}`;
transactionId({ payment: { transactionId: 123456789n } }); // "Transaction: 123456789"
```

### Date

Use `typelit.date()` to create Date variables that automatically convert
JavaScript Date objects to strings:

```typescript
// Simple date variable
const eventDate = typelit`Event date: ${typelit.date('date')}`;
eventDate({ date: new Date('2024-12-25') }); // "Event date: Wed Dec 25 2024 00:00:00 GMT+0000"

// Nested date variable
const appointmentTime = typelit`Appointment scheduled for: ${typelit.date('calendar', 'appointment')}`;
appointmentTime({
  calendar: { appointment: new Date('2024-12-25T15:30:00Z') },
}); // "Appointment scheduled for: Wed Dec 25 2024 15:30:00 GMT+0000"
```

### JSON

Use `typelit.json()` to create variables that automatically stringify any value
to JSON with proper formatting:

```typescript
// Simple JSON variable
const data = typelit`Data: ${typelit.json('config')}`;
data({ config: { enabled: true, count: 42 } });
// "Data: {
//   "enabled": true,
//   "count": 42
// }"

// Nested JSON variable
const userProfile = typelit`Profile: ${typelit.json('user', 'profile')}`;
userProfile({
  user: {
    profile: {
      name: 'Alice',
      preferences: {
        theme: 'dark',
        notifications: true,
      },
    },
  },
});
// "Profile: {
//   "name": "Alice",
//   "preferences": {
//     "theme": "dark",
//     "notifications": true
//   }
// }"
```

## Custom Variable Creators

You can create your own variable creators for any type using the `createType`
function. Here's a basic example creating a variable creator for JavaScript's
`Date` type:

```typescript
import { createType } from 'typelit';

// Create a variable creator for Date
const typelitDate = createType<Date>();

// Use it in a template
const template = typelit`Event starts at ${typelitDate('event', 'startTime')}`;

// The context object requires a Date instance
const result = template({
  event: {
    startTime: new Date('2024-12-25T10:00:00Z'),
  },
}); // "Event starts at Wed Dec 25 2024 10:00:00 GMT+0000"

// Type error: string is not assignable to Date
template({
  event: {
    startTime: '2024-12-25', // Error!
  },
});
```

Like built-in variable creators, custom ones:

- Support nested paths
- Provide full type inference for the context object
- Enforce the correct type at compile time

### Customizing String Conversion

The `createType` function accepts an optional `options`. The `options.stringify`
parameter is a function that takes a value of your type and returns a string.

Here are some examples of customizing string conversion:

```typescript
// Custom date formatting
const typelitDate = createType<Date>({
  stringify: (date) =>
    date.toLocaleDateString('en-US', {
      weekday: 'short',
      month: 'short',
      day: 'numeric',
      year: 'numeric',
    }),
});

const event = typelit`Event: ${typelitDate('date')}`;
event({ date: new Date('2024-12-25') }); // "Event: Wed, Dec 25, 2024"

// Currency formatting
const typelitPrice = createType<number>({
  stringify: (price) =>
    new Intl.NumberFormat('en-US', {
      style: 'currency',
      currency: 'USD',
    }).format(price),
});

const price = typelit`Total: ${typelitPrice('amount')}`;
price({ amount: 42.99 }); // "Total: $42.99"

// Custom object formatting
type User = { id: number; name: string };
const typelitUser = createType<User>({
  stringify: (user) => `#${user.id} ${user.name}`,
});

const user = typelit`Created by: ${typelitUser('author')}`;
user({ author: { id: 123, name: 'Alice' } }); // "Created by: #123 Alice"
```

Without a custom `stringify` function, `createType` uses JavaScript's built-in
`String()` function to convert values to strings. This is equivalent to:

```typescript
createType<T>({ stringify: String });
```

You might want to provide a custom `stringify` function when:

- Formatting dates in a specific way
- Formatting numbers (currency, percentages, fixed decimal places)
- Creating a custom string representation for objects
- Adding prefixes, suffixes, or other decorations to values
- Internationalizing or localizing output

## Contributing

We welcome contributions! Please see our [Contributing Guide](CONTRIBUTING.md)
for details on how to set up the development environment and our contribution
process.

## License

Copyright 2024 Charles Francoise

Licensed under the Apache License, Version 2.0 (the "License"); you may not use
this file except in compliance with the License. You may obtain a copy of the
License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed
under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR
CONDITIONS OF ANY KIND, either express or implied. See the License for the
specific language governing permissions and limitations under the License.
