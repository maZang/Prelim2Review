
# Imperative Features

For this section, just remember how to use imperative features (such as the ref keyword, defining a variable as mutable in records) and also know how they "work". The ref variables are still immutable, but it is the location of the variable in memory that is immutable in this case, rather than the value itself.

### Question 1
Implement the mutable stack signature:


```ocaml
module type Mutable_Stack = sig
    type 'a mstack
    val empty : unit -> 'a mstack
    val push : 'a mstack -> 'a -> unit
    val pop : 'a mstack -> 'a option
end
```




<pre style="color:slategray;max-height:100px;overflow:hidden" 
onclick="
if (this.style.maxHeight === 'none') 
    this.style.maxHeight = '100px';
else
    this.style.maxHeight = 'none'; 
">module type Mutable_Stack =
  sig
    type 'a mstack
    val empty : unit -&gt; 'a mstack
    val push : 'a mstack -&gt; 'a -&gt; unit
    val pop : 'a mstack -&gt; 'a option
  end
</pre>



#### Answer


```ocaml
module MutStack : Mutable_Stack = struct 
    type 'a mstack = ('a list) ref
    
    let empty () = ref []
    
    let push stack element = 
        stack := (element::!stack)
        
    let pop stack = 
        match (!stack) with 
            | [] -> None 
            | h::t -> stack := t; Some h
end
```




<pre style="color:slategray;max-height:100px;overflow:hidden" 
onclick="
if (this.style.maxHeight === 'none') 
    this.style.maxHeight = '100px';
else
    this.style.maxHeight = 'none'; 
">module MutStack : Mutable_Stack
</pre>



To see it in action


```ocaml
let s = MutStack.empty () in 
let () = MutStack.push s 5 in 
let () = MutStack.push s 6 in 
MutStack.pop s
```




<pre style="color:slategray;max-height:100px;overflow:hidden" 
onclick="
if (this.style.maxHeight === 'none') 
    this.style.maxHeight = '100px';
else
    this.style.maxHeight = 'none'; 
">- : int option = Some 6
</pre>



### Question 2
Define the factorial function without using any loops and without the rec keyword.

#### Answer


```ocaml
let fact = 
let f0 = ref(fun x -> x) in 
let f x = if x <= 1 then 1 else x * !f0 (x - 1) in 
let () = f0 := f in 
!f0

let x = fact 3
```




<pre style="color:slategray;max-height:100px;overflow:hidden" 
onclick="
if (this.style.maxHeight === 'none') 
    this.style.maxHeight = '100px';
else
    this.style.maxHeight = 'none'; 
">val fact : int -&gt; int = &lt;fun&gt;
</pre>






<pre style="color:slategray;max-height:100px;overflow:hidden" 
onclick="
if (this.style.maxHeight === 'none') 
    this.style.maxHeight = '100px';
else
    this.style.maxHeight = 'none'; 
">val x : int = 6
</pre>



# Substitution Model

Key things for this section -- just remember the rules and how to apply them. Also, if you were given a completely new rule-set, know how to use those rules

### Question 3

Evaluate the following using the substitution model:

```OCaml
let f = fun x -> x + x in let x = 1 in let g = fun y -> x + f y in g 3
```

#### Answer

Just following the rules in the notes:

```OCaml
let f = (fun x -> fun y -> x + y) in let g = f 3 in g 1 + (f 2 3)
--> let g = f 3 in g 1 + (f 2 3) {fun x -> fun y -> x + y / f} = 
    let g = (fun x -> fun y -> x + y) in g 1 + ((fun x -> fun y -> x + y) 2 3)
--> let g = fun y -> x + y {3 / x} in g 1 + ((fun x -> fun y -> x + y) 2 3) = let g -> fun y -> 3 + y in g 1 + ((fun x -> fun y -> x + y) 2 3)
--> g 1 + ((fun x -> fun y -> x + y) 2 3) {fun y -> 3 + y / g } = (fun y -> 3 + y) 1 + ((fun x -> fun y + x + y) 2 3)
--> (3 + y {1/y}) + ((fun x -> fun y + x + y) 2 3) = 3 + 1 + ((fun x -> fun y + x + y) 2 3)
--> 4 + ((fun x -> fun y -> x + y) 2 3)
--> 4 + ((fun y -> x + y) {2 / x} 3) = 4 + ((fun y -> 2 + y) 3)
--> 4 + (2 + y) {3 / y} = 4 + (2 + 3)
--> 4 + 5
--> 9
```

# Environment Model

Same as above. Know the rules, how to apply them, and the different steps

### Question 4
Evaluate the following using the environment model:
```OCaml
let f = fun x -> fun y -> x + y in let g = f 3 in g 2
```

#### Answer
Just following the rules in the notes:
```OCaml
<[], let f = fun x -> fun y -> x + y in let g = f 3 in g 2> ==> 5
		because <[], fun x -> fun y -> x + y> ==> {fun x -> fun y -> x + y | []}
		and <[f->{fun x -> fun y -> x + y | []}], let g = f 3 in g 2> ==> 5
			because <[f -> {fun x -> fun y -> x + y | []}], f 3> ==> {fun y -> x + y | [x->3]}
				because <[f->{fun x -> fun y -> x + y | []}], f> ==> {fun x -> fun y -> x + y | []}
				and <[f->{fun x -> fun y -> x + y | []}], 3> ==> 3
				and <[x->3], fun y -> x + y> ==> {fun y -> x + y | [x->3]}
			and <[f->{fun x -> fun y -> x + y | []};g->{fun y-> x + y | [x->3]], g 2> ==> 5
				because < [f->{fun x -> fun y -> x + y | []};g->{fun y-> x + y | [x->3]], g> ==> {fun y -> x + y | [x->3]}
				and <[f->{fun x -> fun y -> x + y | []};g->{fun y-> x + y | [x->3]], 2> ==> 2
				and <[y->2;x->3], x + y> ==> 5
					because <[y->2;x->3], x> ==> 3
						because env(x) ==> 3
					and <[y->2;x->3],y> ==> 2
						because env(y) ==> 2
					and 2 + 3 is 5
```

### Question 5
Evaluate the following using the environment model:

```OCaml
let f = fst( let x = 3 in fun y -> x, 2) in f 0
```

#### Answer

```OCaml
 <[], let f = fst (let x = 3 in fun y -> x, 2) in f 0> ==> 3
		because <[], fst (let x = 3 in fun y -> x, 2) ==> {fun y -> x | [x -> 3]}
			because <[], (let x = 3 in fun y -> x, 2)> ==> ({fun y -> x | [x -> 3]}, 2)
				because <[], let x = 3 in fun y -> x> ==> {fun y -> x | [x -> 3]}
					because <[], 3> ==> 3
					and <[x -> 3, fun y -> x> ==> {fun y -> x | [x->3]}
				and <[], 2> ==> 2
		and <[f->{fun y -> x | [x -> 3]}], f 0> ==> 3
			because <[f->{fun y -> x | [x -> 3]}], f> ==> {fun y -> x | [x -> 3]}
			and <[f->{fun y->x | [x->3]}], 0> ==> 0
			and <[y->0;x->3], x> ==> 3
				becuase env(x) ==> 3
```

### Question 6

Evaluate the following with dynamic scope:

```OCaml
let f y = x + y in 
let x = 3 in 
let y = 4 in 
f 2
```

#### Answer 

The answer is 5.

For dynamic scope, remember to update the variables everytime you see them. At the end, x = 3 and y = 2 since when 2 is applied to f, y gets overwritten.

# Induction

Induction is pretty straight forward. Remember to state your inductive hypothesis and your base case and use them to prove the inductive case. The proofs all follow pretty much the same template. If you feel stuck, recheck your inductive hypothesis as it might be incorrect.

### Question 7

Prove that for all integers a and natural numbers x, 
```OCaml
exp a x ~ a ** x
```
 and where exp is defined as:
 ```OCaml
 let rec exp a x = 
     if x = 0 then 1 
     else a * (exp a (x - 1)
```

#### Answer

Theorem: for all ints a, for all naturals x,
```OCaml
exp a x ~ a ** x
```

Proof: by induction on x

Base Case : x = 0
```OCaml
exp a x
~ 1 -- eval of exp
~ a ** 0 -- math
```

Inductive Case: x = k + 1, where k >= 0
Inductive Hypothesis: Assume for all a, and for all i <= k, exp a i ~ a \*\* i

```OCaml
exp a (k + 1)
~ a * (exp a (k + 1 - 1)) -- eval of exp
~ a * (exp a k) -- math, congruence
~ a * a ** k -- I.H.
~ a ** (k + 1) -- math
```

Good Luck!
