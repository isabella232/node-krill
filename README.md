# krill: simple boolean filter language

Krill provides functions for validating and evaluating boolean filters (also
called *predicates*) expressed in a simple JSON language that's intended to be
easy to incorporate into JSON APIs.


## Synopsis

The basic idea is that you construct a *predicate* as a boolean expression that
uses variables (called *fields*).  You can then evaluate the predicate with a
particular assignment of variables.

You can specify types for each field, in which case the expression itself will
be type-checked when you create it.


```javascript
/*
 * Example user input.  There are two fields: "hostname", a string, and
 * "latency", a number.  This predicate will be true if the "hostname" value is
 * "spike" and the "latency" variable is a number greater than 300.
 */
var types = {
    'hostname': 'string',
    'latency': 'number'
};

var input = {
    'or': [
	{ 'eq': [ 'hostname', 'spike' ] },
	{ 'gt': [ 'latency', 300 ] }
    ]
};


/* Validate predicate syntax and types and throw on error. */
console.log(input);
var predicate = krill.createPredicate(input, types);
```

A *trivial* predicate is one that's just "true":

```javascript
/* Check whether this predicate is trivial (always returns true) */
console.log('trivial? ', predicate.trivial());
```

You can print out the fields (variables) used in this predicate:

```javascript
/* Enumerate the fields contained in this predicate. */
console.log('fields: ', predicate.fields().join(', '));
```

You can also print a C-syntax expression for this predicate, which you can
actually plug directly into a C-like language (like JavaScript) to evaluate it:

```javascript
/* Print a DTrace-like representation of the predicate. */
console.log('DTrace format: ', predicate.toCStyleString());
```

You can also evaluate the predicate for a specific set of values:

```javascript
/* Should print "true".  */
var value = { 'hostname': 'spike', 'latency': 12 };
console.log(value, predicate.eval(value));

/* Should print "true".  */
value = { 'hostname': 'sharptooth', 'latency': 400 };
console.log(value, predicate.eval(value));

/* Should print "false".  */
value = { 'hostname': 'sharptooth', 'latency': 12 };
console.log(value, predicate.eval(value));
```


## JSON input format

All predicates can be represented as JSON objects, and you typically pass such
an object into `createPredicate` to work with them.  The simplest predicate is:

```javascript
{}					/* always evaluates to "true" */
```

The general pattern for relational operators is:

```javascript
{ 'OPERATOR': [ 'VARNAME', 'VALUE' ] }	
```

In all of these cases, OPERATOR must be one of the built-in operators, VARNAME
can be any string, and VALUE should be either a specific string or numeric
value.

The built-in operators are:

* `'eq'`: is-equal-to (strings and numbers)
* `'ne'`: is-not-equal-to (strings and numbers)
* `'lt'`: is-less-than (numbers only)
* `'le'`: is-less-than-or-equal-to (numbers only)
* `'ge'`: is-greater-than-or-equal-to (numbers only)
* `'gt'`: is-greater-than (numbers only)

For examples:

```javascript
{ 'eq': [ 'hostname', 'spike' ] }	/* "hostname" variable == "spike" */
{ 'lt': [ 'count',    15      ] }	/* "count" variable <= 15 */
```

You can also use "and" and "or", which have the form:

```javascript
{ 'or':  [ expr1, expr2, ... ] }    /* any of "expr1", "expr2", ... is true */
{ 'and': [ expr1, expr2, ... ] }    /* all of "expr1", "expr2", ... are true */
```

where `expr1`, `expr2`, and so on are any other predicate.  For example:

```
{
    'or': [
	{ 'eq': [ 'hostname', 'spike' ] },
	{ 'gt': [ 'latency', 300 ] }
    ]
};
```

is logically equivalent to the C expression:

    `hostname == "spike" || latency > 300`
