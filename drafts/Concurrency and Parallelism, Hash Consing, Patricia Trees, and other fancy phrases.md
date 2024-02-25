<style type="text/css">
   .tg {
    border:none;
    border-collapse:collapse;
    border-spacing:0;
    margin:0px auto;
}
 .tg td{
    border-style:solid;
    border-width:0px;
    font-family:Arial, sans-serif;
    font-size:14px;
    overflow:hidden;
     padding:10px 5px;
    word-break:normal;
}
 .tg th{
    border-style:solid;
    border-width:0px;
    font-family:Arial, sans-serif;
    font-size:14px;
    font-weight:normal;
     overflow:hidden;
    padding:10px 5px;
    word-break:normal;
}
 .tg .tg-0pky{
    border-color:inherit;
    text-align:left;
    vertical-align:top
}
 .tg .tg-am1n{
    background-color:#fee9bc;
    border-color:inherit;
    text-align:left;
    vertical-align:top
}
 @media screen and (max-width: 767px) {
    .tg {
        width: auto !important;
    }
    .tg col {
        width: auto !important;
    }
    .tg-wrap {
        overflow-x: auto;
        -webkit-overflow-scrolling: touch;
        margin: auto 0px;
    }
}

.tg-wrap{
    counter-reset:fig-counter
}
figure{
    margin:0 auto;
    text-align:center
}
figcaption{
    counter-increment:fig-counter;
    padding-bottom:1rem;
    font-size:.9rem;
    font-style:italic;
    color:#666
}
figcaption::before{
    content:"Fig. " counter(fig-counter) ": ";
    font-weight:700;
    color:#3d703d
}

</style>

<hr>

<style type="text/css">
  /* It's supposed to look like a tree diagram */
.tree, .tree ul, .tree li {
    list-style: none;
    margin: 0;
    padding: 0;
    position: relative;
}

.tree {
    margin: 0 0 1em;
    text-align: center;
}
.tree, .tree ul {
    display: table;
}
.tree ul {
  width: 100%;
}
    .tree li {
        display: table-cell;
        padding: .5em 0;
        vertical-align: top;
    }
        /* _________ */
        .tree li:before {
            outline: solid 1px #666;
            content: "";
            left: 0;
            position: absolute;
            right: 0;
            top: 0;
        }
        .tree li:first-child:before {left: 50%;}
        .tree li:last-child:before {right: 50%;}

        .tree code, .tree span {
            border: solid .1em #666;
            border-radius: .2em;
            display: inline-block;
            margin: 0 .2em .5em;
            padding: .2em .5em;
            position: relative;
        }
        /* If the tree represents DOM structure */
        .tree code {
            font-family: monaco, Consolas, 'Lucida Console', monospace;
        }

            /* | */
            .tree ul:before,
            .tree code:before,
            .tree span:before {
                outline: solid 1px #666;
                content: "";
                height: .5em;
                left: 50%;
                position: absolute;
            }
            .tree ul:before {
                top: -.5em;
            }
            .tree code:before,
            .tree span:before {
                top: -.55em;
            }

/* The root node doesn't connect upwards */
.tree > li {margin-top: 0;}
    .tree > li:before,
    .tree > li:after,
    .tree > li > code:before,
    .tree > li > span:before {
      outline: none;
    }
</style>



<hr>



# Concurrency and Parallelism, Hash Consing, Patricia Trees, and Other Fancy Words

<div class="tg-wrap"><table class="tg">
<tbody>
  <tr>
    <td class="tg-0pky"><p>
    <h2>Expressions == Functions</h2>
One of the joys of compiler construction is the way in which so many disparate ideas from across computer science are brought together in the same place, but today I want to flip this on its head and look at a small set of related ideas from computer science that find application in many different ways. This set of ideas have been incredibly useful for a wide range of applications from graph databases to computer algebra systems "but perhaps nowhere more so than in compilers and other language processors"[^1]. Our motivating problem will be that of evaluating *expressions*, mathematical expressions, for example. This example, while simple, is extremely powerful, because expression evaluation forms the basis for several full-fledged programming languages such as Lisp, Prolog, and Haskell.  It turns out that the difference between *evaluation of expressions* and *execution of instructions* is almost entirely psychological rather than mathematical in the sense that each can be converted to the other!</p>
    </td>
    <td class="tg-am1n">
    &nbsp;
    </td>
  </tr>
  <tr>
    <td class="tg-0pky"><p>
To illustrate the connection between functional programming and expression evaluation, consider the mathematical expression $x^2 +3x + 2$. If we were to write an interpreter for expressions like this, our interpreter would probably first construct an abstract syntax tree (AST, see fig. 1). The AST itself informs us of how the expression must be evaluated. If we implemented the expression as function calls, the implementation would look something like fig. 2. Indeed, every expression could be thought of as the composition of function calls. This isn't exactly rocket science, right?</p>
<p>
The punchline is that expression evaluation and functional programming can be thought of as the same thing. But there is still a wrinkle to iron out: strict evaluation versus lazy evaluation.
</p>
<h2>Strict vs Lazy Evaluation</h2>
<p>
The OCaml programming language uses a _strict_ evaluation discipline by default, which means that the arguments to functions are evaluated eagerly, and the results of the evaluation of the arguments are passed in to the function. This is sometimes called _call by value_, and it is how most imperative programming languages work.

In contrast, the Haskell programming language uses a _lazy_ evaluation discipline by default, which means that arguments
to functions are not evaluated unless they are used, and even then evaluation occurs only at the moment the value is
needed. For this reason, lazy evaluation is sometimes referred to as _call by need_. For someone coming from imperative
languages like C++, lazy evaluation can seem bizarre. 
</p>
    </td>
    <td class="tg-am1n">
<figure>
  <figcaption>An AST for $x^2 +3x + 2$.</figcaption>
  <ul class="tree">
    <li><code>+</code>
      <ul>
        <li><code>^</code>
          <ul>
            <li><code>$x$</code></li>
            <li> <code>2</code></li>
          </ul>
        </li>
        <li><code>&times;</code>
          <ul>
            <li><code>3</code></li>
            <li><code>$x$</code></li>
          </ul>
        </li>
        <li><code>1</code></li>
      </ul>
    </li>
  </ul>
</figure>

<figure>
  <figcaption>An implementation of the expression using function calls.</figcaption>
  <code><pre>add(
  [
    power(x, 2),
    multiply(3, x),
    2
  ]
)</pre></code>
</figure>


  </td>
  </tr>

   <tr>
    <td class="tg-0pky">
column1
    </td>
    <td class="tg-am1n">
column2
    </td>
  </tr>

  </tbody>
</table></div>










[^1]:Okasaki, Chris & Gill, Andy. (2000). Fast Mergeable Integer Maps.
[^2]:  The theoretical details are as deep as you'd like.

[Gabriel Lebec has a wonderful and exceedingly understandable explanation](https://github.com/glebec/lambda-talk) of how
arithmetic, and boolean logic can be encoded as the composition of very simple functions in his talk on the
unnecessarily scary-sounding $\lambda$-calculus. I recommend watching the [first video, part
1](https://www.youtube.com/watch?v=3VQ382QG-y4&t=0s), after which you will certainly be hooked and want to continue on
to [the second video](https://www.youtube.com/watch?v=pAnLQ9jwN-E&t=0s). He also has [his slide
deck](https://speakerdeck.com/glebec/lambda-as-js-or-a-flock-of-functions-combinators-lambda-calculus-and-church-encodings-in-javascript),
[formatted code with notes](https://glebec.github.io/lambda-talk), and of course [the runnable
code](https://replit.com/@glebec/lambda-talk).
