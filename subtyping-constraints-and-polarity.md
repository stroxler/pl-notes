# Subtyping and polarity

I started reading [a blog by Henri
Tuhola](https://boxbase.org/entries/2017/oct/16/mlsub-with-ssa-ir/) on
MLSub subtyping, which described constraint solving in terms of
polarity of types.

Unfortunately I found it a little hard to wrap my head around the actual
meaning of the polarity and how it is used, so I wanted to take some
expanded notes on this topic.

Note: I'm not sure whether there's any relationship between this notion
of polarity (which is focused on subtyping and is related to
covariance/contravariance) and the notion of polarity that Robert Harper
sometimes brings up, as related to introduction vs elimination rules
and described in [this nlab article](https://ncatlab.org/nlab/show/polarity+in+type+theory).

## About polarity notation.

In the section just before discussing type polarity, Tuhola talks about
- The join operator and lower-bound constraints: a constraint of the
  form `[type_term] <= A` can be transformed into an equality constraint
  (which Tuhola calls a "substitution", which I think is acknowledging that
  equality constraints are easier to work with because they are substitutable)
  by making it a (recursive) join type `A := [type_term] v A`.
  - Note that the operator looks like a union - join operations correspond
    to union types in type systems that have unions, but the join operation
	can exist in an implementation / theory even if union types don't
	exist in the underlying language.
- The meet operator and upper-bound constraints: a constraint of the form
  `A <= [type_term]` can be transformed into an equality constraint /
  substitution as `A := [type_term] ^ A`, which also corresponds to an
  intersection type if our type system has these.
  
He notes that
- Lower bounds are implied when we pass data "into" some term - for
  example an assignment.
- Upper bounds are implied when we make use of some term - for example,
  accessing an attribute implies an upper bound on the structural type
  because we restrict to only types that have that attribute defined.
  
Polarity notation is then used to relate upper abnd lower bounds in
complex types. Tuhola doesn't say it explicitly, but the reason this
is needed is really function types: function types are the type-level
representation of operations that are, at runtime, equivalent to term
substitutions or "assignments".

We've seen before that this leads to a flipping of argument type direction,
where decomposing a constraint of the form `A -> B <= C -> D` leads to
`B <= D` (i.e. the same direction), but `A >= C` (a reversed direction).

The polarity notation Tuhola introduces makes this explicit by introducing
+ and - unary operators for type notation, where inside of constraint solving:
  * a new upper boundary constraint is introduced when we encounter a
    "positive" position type
  * a new lower boundary constraint is introduced when we encounter a
    "negative" position types.
	
The negative polar types are:
```
-a                  variables
int                 integers
(+a, +b) -> -c      functions
{n: -a}             records
-a ∧ -b             meet
top
```
and the positive polar types are:
```
+a                  variables
int                 integers
(-a, -b) -> +c      functions
{n: +a}             records
+a ∨ +b             join
bot
```

Any time we see an assignment, we treat the variable we are assigning
*to* as a negative type and the term we are assigning as a positive type.

Any time we see a use of a variable in a term, we treat the variable itself
as a positive type and the term using it as a negative type.

Inequalities always have the positive type on the "less than" side and the
negative type on the "greater than side", because the positive type is intended
to represent an upper bound and the negative type a lower bound.

We can flip a polarity of a single type variable when we convert a
constraint to a substitution.

So, for example:

(A) Eliminating
```
{n: +z} <= -x
```
gives us
```
+x = {n: +z} ∨ +x 
```

(B) Eliminating
```
+z <= g
```
gives us
```
+g = +z V +g
```

(C) Eliminating
```
+x <= {n: -g}
```
produces
```
-x = {n -g} ^ -x
```

(D) Elminating
```
+z <= int
```
gives
```
-z = int ^ z
```


I'm still fuzzy on what exactly the polarities are doing here, but they
seem to help with bookeeping. At the very end, when we have a type
signature, we can eliminate them.


## More resources

I don't think the blog is enough for me to understand this properly.

Resources I could consider:
- Stephen Dolan's 2016 thesis [Algebraic
  Subtyping](https://www.cs.tufts.edu/~nr/cs257/archive/stephen-dolan/thesis.pdf),
  which was the motivation for the blog
- [The simple Essence of Algebraic Subtyping](https://lptk.github.io/programming/2020/03/26/demystifying-mlsub.html),
  which attempts to take that dissertation and explain the key ideas
  more compactly.
  - The [code](https://github.com/LPTK/simple-sub) and the
    [playground](https://lptk.github.io/simple-sub/) for the simple-mlsub paper

Robert Harper's notion of polarity, which may or may not be related (I need
to read more to understand this!) is described
- In the nlab pages I linked above, but TBH I couldn't follow those
- In a [blog post by Harper](https://existentialtype.wordpress.com/2012/08/25/polarity-in-type-theory/)
- In Noam Zeilberger's work (which I think generally focuses more on the logical
  point of view than the operational PL point of view)
  - his [dissertatian](https://noamz.org/thesis.pdf)
  - A [couple](http://noamz.org/papers/polpol.pdf) of
    [papers](https://noamz.org/talks/logpolpro.pdf)
	
I do think the two ideas of polarity are related because the type theorists
talk about positive types involving "structure" and negative types "behavior"
which roughly corresponds to the same ideas as polarity in the context of
subtyping.
