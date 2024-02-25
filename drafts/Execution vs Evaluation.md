Execution vs Evaluation

For most software engineers and, I dare say, even for most computer scientists and mathematicians, the difference between *evaluation* and *execution* mostly comes down to how the problem is presented, the "shape" of the problem. If a problem looks like an *expression*—it has a natural tree structure either syntactically as with a mathematical expression or conceptually in the sense of the recursive fibonacci definition—then we *evaluate*. If, on the other hand, a problem is presented as or is most naturally is expressed by a sequence of imperative instructions, then we *execute*. However, apart from syntax, the difference between evaluation and execution is virtually entirely psychological.  With a little creativity it is not too hard to prove for yourself why this is so: Try to come up with a way of transforming, say, an algorithm written in C into a representation as an expression that can be evaluated. Rewriting the program in Lisp is one way of doing it. Rewriting it in Prolog is another. 

My own personal psychology places functional programming languages like Haskell and OCaml somewhere in between programs as lists of instructions and programs as expressions to be evaluated. Both the structure of a program and the data upon which the program operates are most naturally understood as having a tree structure in these languages, and the operations generally [construct a new immutable data structure from an existing immutable data structure](https://cs3110.github.io/textbook/chapters/modules/functional_data_structures.html). 

Another programming paradigm called term rewriting considers data as expressions and functions as rewrite rules that describe how to transform an expression (a *term*) matching a particular pattern into another expression. Term rewriting by design maps extremely well onto how humans apply mathematical identities to transform mathematical expression during evaluation. Let's use a notation similar to Wolfram Language (Mathematica) to express some of the rules we might use in a term rewriting system by using the following conventions:

* `myFunction[somevalue]` evaluates the (possibly built-in) function `myFunction ` with the argument `somevalue`. (I called it a function, but we will see shortly that this expression is also data, and a "function" is executed by applying rewrite rules to it.) We use square brackets for function application and reserve parentheses only for grouping. We cannot use underscores in names, because…
* `x_` (the letter x adorned with a trailing underscore) is a variable named `x`. To avoid confusion with *symbols* (what mathematicians call variables), we will call them *metavariables*. You can think of metavariables as higher order variables that can have regular variables as their *values*.
* Patterns are built up from terms and metavariables. You are allowed to have a pattern that is itself a concrete literal value, in which case the pattern would only match that same literal value. Or you can have a pattern that is an arbitrarily complex expression with metavariables and non variables nested as deeply as you like.
* A rewrite rule is written as $pattern \to new\; expression$. We might have the rule `f[x_, g[1, 3, x_], y_] -> h[x_, y_]*f[1, 3]`. In this example, the functions `f`, `g`, and `h` are symbols which might or might not have their own definitions, while `x_` and `y_` are metavariables. Note that `x_` must match the same value wherever it appears. So the pattern `f[x_, x_]` *cannot* match with `f[1, 2]` but *does* match `f[7, 7]` (by binding `x_` to 7).

Say we want to come up with a term rewriting system to evaluate the quadratic function $f(x)=x^2-2x+1$ at the point $x=2$. Let's side-step the issue of constructing the arithmetic operations themselves ([even though it is easy to do](https://en.wikipedia.org/wiki/Peano_axioms#Defining_arithmetic_operations_and_relations)) by pretending there already exist built-in functions like `add[…]` and `multiply[…]`, and let's also assume the system understands operator precedence, which is a parse-time issue and not an issue of how to evaluate an expression once the parsing routine determines what expression is being—er, expressed. We might come up with the following rewrite rules:

1. `a_ + b_` $\to$ `add[a_, b_]`  (converting the syntactic form into the built-in operation)

2. `a_ * b_` $\to$ `multiply[a_, b_]`  (converting the syntactic form into the built-in operation)

3. `a_ - b_` $\to$ `a_ + (-1)*b_`  (since addition and multiplication already exist)

4. `a_^n_ `  $\to$ `a_*a_^(n_-1)` if `n>1` (conditions are allowed)

5. `a_^1` $\to$ `a_` (base case for the previous rule)

To this system we add one last rule: `x` $\to$ `2` (rule 6). The `x` in this rule is not a metavariable. It is a symbol (a plain old variable in the language of high school math—no "meta"). Finally, we feed the expression `x^2-2x+1` into the system, flip the "evaluate" switch to the "on" position, and let the machine do its thing. 

An interesting feature of term rewriting systems is that the order in which the rules are applied are not explicitly specified. There is a whole fascinating theory of term rewriting systems that specifies conditions under which different application orders result in the same answer—or even terminates versus rewriting without end. It turns out that for the most common cases encountered in practice, evaluation order doesn't matter!  So let's just pick rules arbitrarily and try to apply them. 

$x^2-2x+1 \to x*x^1-2*x + 1$ (rule 4)

$x*x^1-2*x + 1 \to x*x-2*x + 1$ (rule 5)

$x*x-2*x + 1 \to x*x+(-1)*2*x + 1$ (rule 3)

$x*x+(-1)*2*x + 1 \to … \to 2*2+(-1)*2*2 + 1$  (repeated applications of rule 6)

$2*2+(-1)*2*2 + 1 \to … \to 4 + (-4) + 1$  (repeated applications of rule 2)

$4 + (-4) + 1 \to … \to 1$ (two applications of rule 1)

No more rules apply, so we halt.

If this process feels inefficient to you, that's because it is. In particular:

* Many of the sub-problems, that is, subexpressions that needed to be evaluated, are independent of one another, and whenever this occurs rule applications can be done concurrently.
* The same sub-problem can come up multiple times, and solving the same problem over and over again makes little sense. For example, the subexpression $2*2$ appears twice in the derivation above. The evaluation sequence $2*2\to multiply(2, 2) \to 4$ could be computed just once and the answer distributed to all instances of that one expression. 
* Related to the previous, having multiple instances of the same subexpression is a waste of memory resources. If subexpressions could somehow be shared so that only a single copy needs to be in memory