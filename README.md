
Tokenizr
========

String Tokenization Library for JavaScript

<p/>
<img src="https://nodei.co/npm/tokenizr.png?downloads=true&stars=true" alt=""/>

<p/>
<img src="https://david-dm.org/rse/tokenizr.png" alt=""/>

About
-----

Tokenizr is a small but flexible JavaScript library, providing string
tokenization functionality. It is intended to be be used as the
underlying "lexical scanner" in a Recursive Descent based "syntax
parser". Its distinct features are:

- **Efficient Iteration**:<br/>
  It iterates over the input character string in a read-only and copy-less fashion.

- **Stacked States**:<br/>
  Its tokenization is based on stacked states for determining rules which can be applied.
  Each rule can be enabled for one or more particular states only.

- **Regular Expression Matching**:<br/>
  Its tokenization is based on Regular Expressions for matching the input string.

- **Match Repeating**:<br/>
  Rule actions can (change the state and then) force the repeating of
  the matching process from scratch at the current input position.

- **Match Rejecting**:<br/>
  Rule actions can reject their matching at the current input position
  and let subsequent rules to still match.

- **Match Ignoring**:<br/>
  Rule actions can force the matched input to be ignored (without
  generating a token at all).

- **Match Accepting**:<br/>
  Rule actions can accept the matched input and provide one *or even more* resulting tokens.

- **Shared Context Data**:<br/>
  Rule actions (during tokenization) can optionally store and retrieve arbitrary
  values to/from their tokenization context to share data between rules.

- **Token Text and Value**:<br/>
  Tokens provide information about their matched input text and can provide
  a different corresponding (pre-parsed) value, too.

- **Debug Mode**:<br/>
  The tokenization process can be debugged through optional detailed
  logging of the internal processing.

- **Nestable Transactions**:<br/>
  The tokenization can be split into distinct (and nestable)
  transactions which can be committed or rolled back. This way the
  tokenization can be incrementally stepped back and this way support
  the attempt of parsing alternatives.

- **Token Look-Ahead**:<br/>
  The forthcoming tokens can be inspected to support alternative decisions
  from within the parser based on look-ahead tokens.

Installation
------------

#### Node environments (with NPM package manager):

```shell
$ npm install tokenizr
```

#### Browser environments (with Bower package manager):

```shell
$ bower install tokenizr
```

Usage
-----

Suppose we have a configuration file `sample.cfg`:

```
foo {
    baz = 1 // sample comment
    bar {
        quux = 42
        hello = "hello \"world\"!"
    }
    quux = 7
}
```

Then we can write a lexical scanner in ECMAScript 6 (under Node.js) for the tokens like this:

```js
import fs       from "fs"
import Tokenizr from "tokenizr"

let lexer = new Tokenizr()

lexer.rule(/[a-zA-Z_][a-zA-Z0-9_]*/, (ctx, match) => {
    ctx.accept("id")
})
lexer.rule(/[+-]?[0-9]+/, function (ctx, match) => {
    ctx.accept("number", parseInt(match[0]))
})
lexer.rule(/"((?:\\\"|[^\r\n]+)+)"/, (ctx, match) => {
    ctx.accept("string", match[1].replace(/\\"/g, "\""))
})
lexer.rule(/\/\/[^\r\n]+\r?\n/, (ctx, match) => {
    ctx.ignore()
})
lexer.rule(/[ \t\r\n]+/, (ctx, match) => {
    ctx.ignore()
})
lexer.rule(/./, (ctx, match) => {
    ctx.accept("char")
})

let cfg = fs.readFileSync("sample.cfg", "utf8")

lexer.input(cfg)
lexer.debug(true)
lexer.tokens().forEach((token) => {
    console.log(token.toString())
})
```

The output of running this sample program is:

```
<type: id, value: "foo", text: "foo", pos: 5, line: 2, column: 5>
<type: char, value: "{", text: "{", pos: 9, line: 2, column: 9>
<type: id, value: "baz", text: "baz", pos: 19, line: 3, column: 9>
<type: char, value: "=", text: "=", pos: 23, line: 3, column: 13>
<type: number, value: 1, text: "1", pos: 25, line: 3, column: 15>
<type: id, value: "bar", text: "bar", pos: 53, line: 4, column: 9>
<type: char, value: "{", text: "{", pos: 57, line: 4, column: 13>
<type: id, value: "quux", text: "quux", pos: 71, line: 5, column: 13>
<type: char, value: "=", text: "=", pos: 76, line: 5, column: 18>
<type: number, value: 42, text: "42", pos: 78, line: 5, column: 20>
<type: id, value: "hello", text: "hello", pos: 93, line: 6, column: 13>
<type: char, value: "=", text: "=", pos: 99, line: 6, column: 19>
<type: string, value: "hello \"world\"!", text: "hello "world"!"", pos: 101, line: 6, column: 21>
<type: char, value: "}", text: "}", pos: 126, line: 7, column: 9>
<type: id, value: "quux", text: "quux", pos: 136, line: 8, column: 9>
<type: char, value: "=", text: "=", pos: 141, line: 8, column: 14>
<type: number, value: 7, text: "7", pos: 143, line: 8, column: 16>
<type: char, value: "}", text: "}", pos: 149, line: 9, column: 5>
```

With the help of an Abstract Syntax Tree (AST) library like
[ASTy](https://github.com/rse/asty) and a query library like [ASTq](https://github.com/rse/astq)
you can [write powerful Recursive Descent based parsers](https://github.com/rse/parsing-techniques/blob/master/cfg2kv-3-ls-rdp-ast/cfg2kv.js).

Application Programming Interface (API)
---------------------------------------

### Class `Tokenizr`

This is the main API class for establishing a lexical scanner.

- Constructor: `Tokenizr(): Tokenizr`<br/>
  Create a new tokenization instance.

- Method: `Tokenizr#reset(): Tokenizr`<br/>
  Reset the tokenization instance to a fresh one.

- Method: `Tokenizr#debug(enable: Boolean): Tokenizr`<br/>
  Enable (or disable) verbose logging for debugging purposes.

- Method: `Tokenizr#input(input: String): Tokenizr`<br/>
  Set the input string to tokenize.
  This implicitly performs a `reset()` operation beforehand.

- Method: `Tokenizr#rule(state?: String, pattern: RegExp, action: (ctx: TokenizerContext, match: Array[String]) => Void): Tokenizr`<br/>
  Configure a token matching rule which executes its `action` in case the
  current tokenization state is one of the states in the comma-separated `state` (by default
  the rule matches all states if `state` is not specified) and the
  next input characters match against the `pattern`. The `ctx` argument provides
  a context object for token repeating/rejecting/ignoring/accepting, the
  `match` argument is the result of the underlying `RegExp#exec` call.

- Method: `Tokenizr#token(): Tokenizr.Token`<br/>
  Get the next token from the input. Internally, the
  current position of the input is matched against the
  patterns of all rules (in rule configuration order). The first rule
  action which accepts the matching leads to the token.

- Method: `Tokenizr#tokens(): Array[Tokenizr.Token]`<br/>
  Tokenizes the entire input and returns all the corresponding tokens.
  This is a convenience method only. Usually one takes single
  tokens at a time.

- Method: `Tokenizr#peek(offset?: Number): Tokenizr.Token`<br/>
  Peek at the following token at the (0-based) offset without consuming
  the token. This is the secondary function used in Recursive Descent
  parsers.

- Method: `Tokenizr#skip(next?: Number): Tokenizr`<br/>
  Consume the `next` number of following tokens.

- Method: `Tokenizr#consume(type: String, value?: String): Tokenizr`<br/>
  Match (with `Tokenizr.Token#isA`) the next token. If it matches,
  consume it. If it does not match, throw a `Tokenizr.ParsingError`.
  This is the primary function used in Recursive Descent parsers.

- Method: `Tokenizr#begin(): Tokenizr`<br/>
  Begin a transaction. Until `commit` or `rollback` are called, all
  consumed tokens will be internally remembered and be either thrown
  away (on `commit`) or pushed back (on `rollback`). This can be used
  multiple times and supports nested transactions.

- Method: `Tokenizr#depth(): Number`<br/>
  Return the number of already consumed tokens in the currently
  active transaction. This is useful if multiple alternatives
  are parsed and in case all failed, to report the error for
  the most specific, i.e., the one which consumed most tokens.

- Method: `Tokenizr#commit(): Tokenizr`<br/>
  End a transaction successfully. All consumed tokens are thrown away.

- Method: `Tokenizr#rollback(): Tokenizr`<br/>
  End a transaction unsuccessfully. All consumed tokens are pushed back
  and can be consumed again.

- Method: `Tokenizr#alternatives(...alternatives: Array[() => any]): any`<br/>
  Utility method for parsing alternatives. It internally executes the
  supplied callback functions in sequence, each wrapped into its own
  transaction. The first one which succeeds (does not throw an exception
  and returns a value) leads to the successful result. In case all
  alternatives failed (all throw an exception), the exception of the
  most-specific alterative (the one with the largest transaction depth)
  is re-thrown.

- Method: `Tokenizr#error(message: String): Tokenizr.ParsingError`<br/>
  Returns a new instance of `Tokenizr.ParsingError`, based on the
  current input character stream position.

### Class `Tokenizr.Token`

This is the class of all returned tokens.

- Property: `Tokenizr.Token#type: String`<br/>
  The type of the token.

- Property: `Tokenizr.Token#value: any`<br/>
  The value of the token. By default this is the same as
  `Tokenizr.Token#text`, but can be any pre-processed value.

- Property: `Tokenizr.Token#text: String`<br/>
  The corresponding input text of this token.

- Property: `Tokenizr.Token#pos: Number`<br/>
  The (0-based) position in the input.

- Property: `Tokenizr.Token#line: Number`<br/>
  The (1-based) line number in the input.

- Property: `Tokenizr.Token#column: Number`<br/>
  The (1-based) column number in the input.

- Method: `Tokenizr.Token#toString(): String`<br/>
  Returns a formatted representation of the token.

- Method: `Tokenizr.Token#isA(type: String, value?: any): String`<br/>
  Checks whether token matches against a particular type and optionally
  a particular value.

### Class `Tokenizr.ParsingError`

This is the class of all thrown exceptions related to parsing.

- Property: `Tokenizr.ParsingError#name: String`<br/>
  Always just the string `ParsingError` to be complaint to
  the JavaScript `Error` class specification.

- Property: `Tokenizr.ParsingError#message: String`<br/>
  The particular error message.

- Property: `Tokenizr.ParsingError#pos: Number`<br/>
  The (0-based) position in the input.

- Property: `Tokenizr.ParsingError#line: Number`<br/>
  The (1-based) line number in the input.

- Property: `Tokenizr.ParsingError#column: Number`<br/>
  The (1-based) column number in the input.

- Property: `Tokenizr.ParsingError#input: String`<br/>
  The input itself.

- Method: `Tokenizr.ParsingError#toString(): String`<br/>
  Returns a formatted representation of the error.

### Class `Tokenizr.ActionContext`

This is the class of all rule action contexts.

- Method: `Tokenizr.ActionContext#data(key: String, value?: any): any`<br/>
  Store or retrieve any user data (indexed by `key`) to the action
  context for sharing data between rules.

- Method: `Tokenizr.ActionContext#state(new?: String): String`<br/>
  Push or pop a state to/from the current tokenization state stack.

- Method: `Tokenizr.ActionContext#repeat(): Tokenizr.ActionContext`<br/>
  Mark the tokenization process to repeat the matching at the current
  input position from scratch. You first have to switch to a different
  state or this will lead to an endless loop, of course.

- Method: `Tokenizr.ActionContext#reject(): Tokenizr.ActionContext`<br/>
  Mark the current matching to be rejected. The tokenization process
  will continue matching following rules.

- Method: `Tokenizr.ActionContext#ignore(): Tokenizr.ActionContext`<br/>
  Mark the current matching to be just ignored. This is usually
  used for skipping whitespaces.

- Method: `Tokenizr.ActionContext#accept(type: String, value?: any): Tokenizr.ActionContext`<br/>
  Accept the current matching and produce a token of `type` and
  optionally with a different `value` (usually a pre-processed variant
  of the matched text). This function can be called multiple times to
  produce one or more tokens.

Implementation Notice
---------------------

Although Tokenizr is written in ECMAScript 6, it is transpiled to ECMAScript
5 and this way runs in really all(!) current (as of 2015) JavaScript
environments, of course.

Internally, Tokenizr scans the input string in a read-only fashion
by leveraging `RegExp`'s `g` (global) flag in combination with the
`lastIndex` field, the best one can do on ECMAScript 5 runtime.
For ECMAScript 6 runtimes we will switch to `RegExp`'s new `y` (sticky)
flag in the future as it is even more efficient.

License
-------

Copyright (c) 2015 Ralf S. Engelschall (http://engelschall.com/)

Permission is hereby granted, free of charge, to any person obtaining
a copy of this software and associated documentation files (the
"Software"), to deal in the Software without restriction, including
without limitation the rights to use, copy, modify, merge, publish,
distribute, sublicense, and/or sell copies of the Software, and to
permit persons to whom the Software is furnished to do so, subject to
the following conditions:

The above copyright notice and this permission notice shall be included
in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY
CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT,
TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

