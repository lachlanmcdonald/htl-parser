# DEPRECATED in favour of contributing to Adobe's official project instead [htlengine](https://github.com/adobe/htlengine)

The core features from this project that weren't already in the official project have been merged in already.

---

![Deloitte Digital](https://raw.githubusercontent.com/DeloitteDigitalAPAC/eslint-config-deloitte/master/dd-logo.png)

# htl-parser

[![npm version](https://badge.fury.io/js/%40deloitte-digital-au%2Fhtl-parser.svg)](https://badge.fury.io/js/%40deloitte-digital-au%2Fhtl-parser) [![Build Status](https://travis-ci.org/DeloitteDigitalAPAC/htl-parser.svg?branch=master)](https://travis-ci.org/DeloitteDigitalAPAC/htl-parser)

> **htl-parser** is a PEG.js grammar and parser for Adobe's *HTML Template Language* (HTL).

## Installation

Versions of this package mirror the [HTL Specification](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md).
This package matches is 1.3.1 of the specification. 

```bash
npm install @deloitte-digital-au/htl-parser@1.3
```

## Usage

#### Parsing a statement

```js
const HTLParser = require('htl-parser');
const tokens = HTLParser.parse('${ a || b }');
```

**Notes:**

- The parser only processes expressions between `${` and `}`. Parsing will fail if there are any characters before or after the expression.
- The parser attempts to process HTL in their corresponding JavaScript type. For example, `int` and `float` types become a Number, etc.
- Superfluous whitespace characters are not tokenised.
- The parser returns an AST-like result. This is not guaranteed to be a proper abstract syntax tree.

#### Generating a parser

This repository comes with a pre-compiled parser. However, you can re-generate the parser with:

```bash
npm run generate
```

The parser will be output to `dist/parser.js`.

## Grammar

The PEG.js grammar is contained in `src/htl.pegjs`

## Parsing results

Each token is an array in the format:

```js
[identifier, value]
```

Where `identifier` is the token identifier (i.e. the type of token), and `value` is either an object describing the token, or a child token itself.

### Processing tokens

#### `["expression", `*object*`]`

This token will appear only once per tree.

**object**:

- `expression` Child token, or `null` for [parametric expressions](https://github.com/Adobe-Marketing-Cloud/htl-spec/blob/master/SPECIFICATION.md#21-syntax)
- `options` Tokens; arguments passed to the expression. An empty array when no arguments are present.
- `location` Location of the expression in the input.

#### `["option", `*object*`]`

Arguments passed to an expression.

**object**:

- `option` Option name
- `value` Option value; either a child token or `null` when a value was not provided.

#### `["factor", `*object*`]`

**object**:

- `negative` Whether the term should be negated, i.e. `${ !a }`
- `term` Child token

#### `["property", `*object*`]`

**object**:

- `atom` Child token, some atomic value or other expression.
- `accessors` Accessors; an array of tokens in order that they are used.

For example:

```
${ a.b.c[2] }
```

- `atom` will be `['variable', 'a']`, and
- `accessors` would be an array of:
  - `['variable', 'b']`
  - `['variable', 'c']`
  - `['int', '2']`

### Logic tokens

#### `["ternary", `*object*`]`

Represents a ternary operation, i.e. `${ a ? b : c }`.

**object**:

- `comparison` Token; expression to evaluate
- `left` Token; expression when comparison evaluates to true
- `right` Token; expression when comparison evaluates to false

#### `["comparison", `*object*`]`

Represents a comparison or logical operation. i.e `${ a || b }` or `${ a < 5 }`

**object**:

- `operator` As listed below
- `left` Token; before operand
- `right` Token; after operand

| Operator    | Expression                    |
| ----------- | ----------------------------- |
| `and`       | <code>&amp;&amp;</code>       |
| `or`        | <code>&#x007C;&#x007C;</code> |
| `lte`       | <code>&lt;=</code>            |
| `lt`        | <code>&lt;</code>             |
| `gt`        | <code>&gt;</code>             |
| `gte`       | <code>&gt;=</code>            |
| `equal`     | <code>==</code>               |
| `not_equal` | <code>!=</code>               |

### Type tokens

#### `["array", `*tokens*`]`

Represents an array type. **tokens** will be an JavaScript Array of each token in the array.

#### `["variable", `*String*`]`

Represents a variable (or identifier) in the model. **String** will be an JavaScript String, representing the value in the model. How **String** should be used to retrieve a value from a model is at the implementation's discretion.

#### `["integer", `*Number*`]`

Represents an integer.

**Number** will be a JavaScript Number, parsed using `parseInt(x, 10);`.

#### `["float", `*Number*`]`

Represents an floating-point number.

**Number** will be a JavaScript Number, parsed using `parseFloat(x);`.

#### `["boolean", `*Boolean*`]`

Represents an boolean value.

**Boolean** will be a JavaScript Boolean, equalling `true` when the value is `"true"`, or `false` otherwise.

#### `["string", `*String*`]`

Represents a string.

**String** will be an JavaScript String.

## Specification notes

Ambiguities in the specifications grammar have been resolved as below:

- Added a declarative array type, as it is present throughout the documentation, but not the grammar. ([see issue #60](https://github.com/Adobe-Marketing-Cloud/htl-spec/issues/60))
- Updated the `propertyAccess` to prevent the erroneous expression `${ a."string" }` from validating. ([see issue #57](https://github.com/Adobe-Marketing-Cloud/htl-spec/issues/57))
- Updated grammar to include missing `== != < <= > >=` operators. ([see issue #58](https://github.com/Adobe-Marketing-Cloud/htl-spec/issues/58))
- Updated grammar to allow complex nodes in option definitions.
- Updated grammar to allow negative `integers` and `floats`.  ([see issue #44](https://github.com/Adobe-Marketing-Cloud/htl-spec/issues/44))

#### Limitations

This implementation does not correctly parse concurrent `&&` or `||` without parentheses. For example:

```
${ a || b || c }
```

Must be written as:

```
${ (a || b) || c }
```

HTL 1.3 treats both `&&` and `||` with the same precedence:

```
true || false && false
```

Evaluates as `false` in HTL 1.3, and `true` is most other programming languages. For this reason, it is recommended to surround each operation with parentheses.

## Tests

Tests can be run with:

```bash
npm run test
```

## License

**BSD 3-Clause License**

> Copyright (c) 2018. All rights reserved.
> 
> htl-parser can be downloaded from: https://github.com/DeloitteDigitalAPAC/htl-parser
> 
> Redistribution and use in source and binary forms, with or without modification, are permitted provided that the following conditions are met:
> 
> - Redistributions of source code must retain the above copyright notice, this list of conditions and the following disclaimer.
> - Redistributions in binary form must reproduce the above copyright notice, this list of conditions and the following disclaimer in the documentation and/or other materials provided with the distribution.
> - Neither the name of the copyright holder nor the names of its contributors may be used to endorse or promote products derived from this software without specific prior written permission.
> 
> THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE DISCLAIMED. 
> 
> IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
