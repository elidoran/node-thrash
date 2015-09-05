# Thrash

Tests functions via generated inputs and validating outputs.

## Install

```sh
npm install thrash --save-dev
```

## Usage

```coffeescript
thrash = require 'thrash'
fn     = require 'yourExportedFunction'

# fn = () -> 'hello'

thrash fn,
  input: [ # specify valid input. use array element per argument
    { # argument one
      type: 'string'
      pattern: /something/ # regex allowed for string
      min: 5  # minimum length
      max: 72 # maximum length
    }
    { # argument two
      optional: true # optional arguments may be null/undefined
      #'null': false # if it may be either null or undefined, but not both, use individual properties.
      #'undefined': false
    }
    {
      optional: true
      type: 'function'
      callback: true # standard (error, result) function signature for callback
    }
  ]    
  'this': # the context it's executed in.
    before: (inputs) -> # thrash tells us the inputs it's going to use, we configure the 'this'
    after: (inputs) -> # thrash again gives us the inputs, we verify the `this` is correct
  output: # specify how to validate the output. works for return/throw and callback's error/result
    type: 'string' # the result of calling typeof on the output
    pattern: /^hello$/ # strings can use patterns for validation
    validate: (output) -> output is 'hello' # can validate via a function
    valid: # can specify a map of valid/invalid values
      hello: true
      goodbye: false
```

This does more than simply call the function and test the result. We could do
that much more simply.

Thrash will generate inputs for the function and then validate the results.
It's going to pass in null, undefined, strings, Strings, Number, Objects, and
ensure they don't mess with its ability to produce the correct output.

It uses generators to produce inputs. For example, StringGenerator produces
random strings as well as dictionary words depending on its configuration. The
regular expression pattern specified as a valid pattern will affect it. It will
produce both valid and invalid values. When the values are invalid, Thrash will
ensure the function either throws an error, provides an error to the callback,
or does what you specify it will do.

## What Happens

Overview:

1. accepts the function and the thrash specification
2. reads the input specifications and gets input generators for them
3. tests all permutations of the default inputs
4. begins generating random values which match valid inputs
5. generates invalid values. it uses permutations of valid/invalid together
6. reports all results, statistics, and closes

Single Test:

1. gather the inputs to use for the Test
2. call the 'this'.before function to get what the this context should have based on those inputs
3. call the function with that this context like: fn.apply theThis, inputsArray
4. catch any thrown error (sync), or record error given to the callback (async)
5. store the returned result (sync), or the result given to the callback (async)
6. call the 'this'.after function to valid the `this` context is correct
7. use outputs specification to validate the result object or error object.
8. record success/failure and detail messages


### Default Inputs

Some values should always be passed as arguments to test the outcome. These are
the ones Thrash always generates based on argument type.

1. null/undefined - each permutation of arguments will be called with null/undefined
2. string: '' - all permutations of string arguments will be called with the empty string
3. Number: min/max/NaN/0/-1/1 - all permutations of number arguments will be called with these values
4. Object: {} - empty object as well as null/undefined mentioned in #1.

### Generators

1. StringGenerator
2. NumberGenerator
3. ObjectGenerator

#### MIT License
