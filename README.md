Eto.Parse
=========
### A recursive descent LL(k) parser framework for .NET

Discussion
----------

* Chat in [#eto.parse](http://webchat.freenode.net/?channels=eto.parse) on freenode

Description
-----------

Eto.Parse is a highly optimized recursive decent parser framework that can be used to create parsers for complex grammars that go beyond the capability of regular expressions.

You can use BNF, EBNF, or Gold parser grammars to define your parser, code them directly using a fluent API, or use shorthand operators (or a mix of each).

### Why not use RegEx?

This is a very valid question. Regular Expressions work great when the syntax is not complex, but fall short especially when dealing with any recursive syntax using some form of brackets or grouping concepts. 

For example, creating a math parser using RegEx cannot validate (directly) that there are matching brackets.  E.g. "((1+2)*3)", or "{ 'my': 'value', 'is' : {'recursive': true } }"

What's so great about Eto.Parse?
--------------------------------

### Matching

The framework has been put together to get at the relevant values as easily as possible.  Each parser can be *named*, which then builds a tree of named matches that represent the interesting sections of the parsed input. You can use events on the named sections to perform logic when they match, or just parse the match tree directly.

### Left Recursion

One rather cumbersome issue to deal with using recursive descent parsers is [left recursion](http://en.wikipedia.org/wiki/Left_recursion). Eto.Parse automatically identifies left recursive grammars and will parse them with no issues by transforming into a repeating pattern.

Performance
-----------

Eto.Parse has been highly optimized for performance and memory usage. For example, here's a comparison parsing a large JSON string 100 times (times in seconds):

### Speed

Framework       | Run 1  | Run 2  | Run 3  | Average | Diff 
--------------- | ------ | ------ | ------ | ------- | ------------
Eto.Parse       | 0.352s | 0.350s | 0.351s | 0.351s  | 1x
Newtonsoft.Json | 0.278s | 0.268s | 0.268s | 0.271s  | 1.29x Faster
Irony           | 2.514s | 2.513s | 2.557s | 2.528s  | 7.2x Slower

### Memory

Framework       | Allocated | Diff       | # Objects | Diff
--------------- | --------- | ---------- | --------- | ----------
Eto.Parse       | 47.22 MB  | 1x         | 1470682   | 1x
Newtonsoft.Json | 109.39 MB | 2.32x More | 2176326   | 1.48x More
Irony           | 440.21 MB | 9.32x More | 9572011   | 6.51x More

Example
-------

For example, the following defines a simple hello world parser in fluent API:

	// optional repeating whitespace
	var ws = Terminals.WhiteSpace.Repeat(0);

	// parse a value with or without brackets
	var valueParser = Terminals.Set('(')
		.Then(Terminals.AnyChar.Repeat().Until(ws.Then(')')).Named("value"))
		.Then(Terminals.Set(')'))
		.SeparatedBy(ws)
		.Or(Terminals.WhiteSpace.Inverse().Repeat().Named("value"));

	// our grammar
	var grammar = new Grammar(
		ws
		.Then(valueParser.Named("first"))
		.Then(valueParser.Named("second"))
		.Then(Terminals.End)
		.SeparatedBy(ws)
	);

Or using shorthand operators:

	// optional repeating whitespace
	var ws = -Terminals.WhiteSpace;

	// parse a value with or without brackets
	Parser valueParser = 
		('(' & ws & (+(Terminals.AnyChar) - (ws & ')')).Named("value") & ws & ')')
		| (+!Terminals.WhiteSpace).Named("value");

	// our grammar
	var grammar = new Grammar(
		ws & valueParser.Named("first") & 
		ws & valueParser.Named("second") & 
		ws & Terminals.End
	);

Or, using EBNF:

	var grammar = new EbnfGrammar().Build(@"
	(* := is an extension to define a literal with no whitespace between repeats and sequences *)
	ws := {? Terminals.WhiteSpace ?};
	
	letter or digit := ? Terminals.LetterOrDigit ?;
	
	simple value := letter or digit, {letter or digit};
	
	bracket value = simple value, {simple value};
	
	optional bracket = '(', bracket value, ')' | simple value;
	
	first = optional bracket;
	
	second = optional bracket;
	
	grammar = ws, first, second, ws;
	", "grammar");

These can parse the following text input:

	var input = "  hello ( parsing world )  ";
	var match = grammar.Match(input);
	
	var firstValue = match["first"]["value"].Value;
	var secondValue = match["second"]["value"].Value;

**firstValue** will equal "hello", and **secondValue** will equal "parsing world".


License
-------

Licensed under [MIT](http://opensource.org/licenses/MIT).

See LICENSE file for full license.