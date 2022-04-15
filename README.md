# Hindley-Milner type inference in Scheme, for Scheme

[C# port here](https://github.com/zdimension/hm-infer-cs), [Rust port here](https://github.com/zdimension/hm-infer-rs).

This is a type inference engine built on the Hindley-Milner algorithm, that analyzes, type-checks and computes the type of expressions written in an extended subset of Scheme.

## Supported Scheme constructs

- `(let ([name val] ...) body)`: basic `let`
- `(let* ([name val] ...) body)`: ordered `let` (each symbol can access the symbols defined before it)
- `(letrec ([name val] ...) body)`: `let` with recursion support (symbols can access themselves inside a lambda definition)
- `(lambda (a b ...) body)`: lambda definition
- `(f a b ...)`: procedure application (**difference from Scheme**: implicit partial application; such that `(pair a b)` is strictly equivalent to `((pair a) b)`)

## Built-ins

Types:
- `int` (Scheme `number?`)
- `bool` (Scheme `boolean?`)
- `str` (Scheme `string?`)
- `unit` (void/unit type)
- `bottom` (Haskell `forall a. a`, Rust `!`)

Symbols:
<table>
<thead>
  <tr><th></th><th>Name</th><th>Signature (n-ary syntax)</th></tr>
</thead>
<tbody>
  <tr><td rowspan="3">Numbers</td><td><code>+</code>, <code>-</code>, <code>*</code>, <code>/</code>, <code>modulo</code>, <code>succ</code>, <code>pred</code></td><td><code>[int int] -&gt; int</code></td></tr>
<tr><td><code>=</code></td><td><code>[int int] -&gt; bool</code></td></tr>
<tr><td><code>zero</code></td><td><code>int -&gt; bool</code></td></tr>
  <tr><td>Booleans</td><td><code>and</code>, <code>or</code></td><td><code>[bool bool] -&gt; bool</code></td></tr>
  <tr><td rowspan="2">Utilities</td><td><code>error</code></td><td><code>str -&gt; never</code></td></tr>
<tr><td><code>if</code></td><td><code>[bool a a] -&gt; a</code></td></tr>
  <tr><td rowspan="3">Pairs</td><td><code>pair</code></td><td><code>[a b] -&gt; (a * b)</code></td></tr>
<tr><td><code>car</code></td><td><code>(a * b) -&gt; a</code></td></tr>
<tr><td><code>cdr</code></td><td><code>(a * b) -&gt; b</code></td></tr>
<tr><td rowspan="5">Lists</td><td><code>nil</code></td><td><code>(list a)</code></td></tr>
<tr><td><code>cons</code></td><td><code>[a (list a)] -&gt; (list a)</code></td></tr>
<tr><td><code>hd</code></td><td><code>(list a) -&gt; a</code></td></tr>
<tr><td><code>tl</code></td><td><code>(list a) -&gt; (list a)</code></td></tr>
<tr><td><code>null?</code></td><td><code>(list a) -&gt; bool</code></td></tr>
<tr><td rowspan="3">Either</td><td><code>left</code></td><td><code>a -&gt; (either a b)</code></td></tr>
<tr><td><code>right</code></td><td><code>b -&gt; (either a b)</code></td></tr>
<tr><td><code>either</code></td><td><code>[(either a b) (a -&gt; x) (b -&gt; y)] -&gt; (either x y)</code></td></tr>
<tr><td rowspan="3">Maybe</td><td><code>just</code></td><td><code>a -&gt; (option a)</code></td></tr>
<tr><td><code>nothing</code></td><td><code>(option a)</code></td></tr>
<tr><td><code>maybe</code></td><td><code>[(option a) (a -&gt; x)] -&gt; (option x)</code></td></tr>
</tbody>
</table>

## Internals 

All functions are internally curried, as in Haskell, e.g. the type of `pair` is `(a -> (b -> (a * b)))`. 
This allows for built-in support for partial application (cf. above) in the HM framework, while still allowing to define n-ary functions.

In effect, `(lambda (a b) body)` is syntactic sugar for `(lambda (a) (lambda (b) body))`.
