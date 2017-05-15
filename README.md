# mimolette

[](dependency)
```clojure
[plumula/mimolette "0.1.0-SNAPSHOT"] ;; latest release
```
[](/dependency)

Check your [specs](https://clojure.org/guides/spec) from `clojure.test` and
`cljs.test`. 


## Usage
In your test namespace, require Mimolette. This works both for Clojure and
ClojureScript.

```clj
(ns whatever.my-test
  (:require [plumula.mimolette :refer [defspec-test]]))
```

### Generating tests for your function
Use Mimolette to create a test named `test-foo-spec`, that will check
`a-function` and `another-functnion` against their specs.

#### Testing functions
```clj
(defspec-test test-foo-spec `[a-function another-function])
```

The functions-to-be-tested must be named by fully-qualified symbols. Often, the
most convenient way of getting them is to use the syntax quote as in the example
above.

#### Testing a single function
It is not necessary to use a collection when checking a single function:

```clj
(defspec-test test-foo-spec `yet-another-function)
```

#### Testing whole namespaces
You might find the [`enumerate-namespace`](https://clojure.github.io/clojure/branch-master/clojure.spec-api.html#clojure.spec.test/enumerate-namespace)
functions useful for checking all the functions of one or more namespaces against their respective specs.

```clj
(defspec-test test-namespaces (stest/enumerate-namespace 'my.namespace))
(defspec-test test-namespaces (stest/enumerate-namespace '[my.other-namespace
                                                           yet-another.namespace]))
```

(assuming `stest` is an alias for `clojure.spec.test.alpha` or `cljs.spec.test`).

#### Limiting the number of tests
If your test suite is getting too slow because of spec checks, you might want to
restrict the number of values tested. For instance, this code would test
`my-function` for 10 different inputs.
```clj
(defspec-test test-foo-spec `my-function {:opts {:num-tests 10}})
```

The underlying mechanism is a little convoluted. If you pass a map as third parameter to
`defspec-test`, that parameter will get passed on as the opts to 
[`stest/check`](https://clojure.github.io/clojure/branch-master/clojure.spec-api.html#clojure.spec.test/check).

`stest/check` will in turn pass a special parameter on to
[`quick-check`](https://clojure.github.io/test.check/clojure.test.check.html#var-quick-check).
However, for some crazy reason, that parameter is named differently in Clojure
(`:clojure.spec.test.check/opts`) and ClojureScript (`:clojure.test.check/opts`).

To remedy this, Mimolette will translate the namespace-less `:opts` key to
whichever key the underlying specs implementation understands.

Finally, the `:num-tests` key adds another round of magic: it isn’t actually
interpreted by `quick-check`. Instead, `stest/check` passes it as the `num-tests`
parameter to `quick-check`.

## Troubleshooting
###No tests are generated
`defspec-test` will only generate tests for specs that it knows about, so you
want to make sur that your specs are actually loaded. If your specs reside in
the same namespace as your functions, then that’s already taken care of. If your
specs have a namespace of their own, then make sure to require that from your
tests. This is especially true in ClojureScript, Clojure seems to be more
forgiving here.

## Known limitations
- There is no test suite. Now that side-effect-free functions have been broken
  out of the original macro, they should be easier to test.
- The handling of options is incredibly convoluted and ought to be straightened
  out.

## Project history
I ([Frederic Merizen](https://www.linkedin.com/in/fredericmerizen)) am still
trying to get get attribution right for this library. As far as I can tell,
here’s how it goes. I would like to thank Timothy Washington for pointing me to
the clojurians slack source.

On the November 4th 2016, [Kenny Williams](https://clojurians.slack.com/team/kenny)
published his ‘[solution](https://clojurians.slack.com/files/kenny/F2XV8TRC3/clojure_spec_test___clojure_test.clj)
to integrating `clojure.spec.test/check` with `clojure.test`’ on the
[\#clojure-spec](https://clojurians.slack.com/messages/C1B1BB2Q3)
channel of the [clojurians](https://clojurians.slack.com) slack.

On November 19th 2016, [Timothy Washington](http://stackoverflow.com/users/375616/nutritioustim) 
[asked](http://stackoverflow.com/questions/40697841/howto-include-clojure-specd-functions-in-a-test-suite)
on Stack Overflow how to run spec checks from `clojure.test`. On November 21st
he answered his own question with the `defspec-test` macro from the clojurians
slack.

This is were I found the macro. I wanted to use it for my own projects. I’m no
great fan of reuse by cut and paste, and so I decided to give it a home on
Clojars,  added ClojureScript support, and broke the macro down into simpler
components, and thus Mimolette was born.

## License
Distributed under the [MIT license](LICENSE.txt).
Copyright &copy; 2017 Frederic Merizen & Kenny Williams.
