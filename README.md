mad-at-x
========

miniKanren/cKanren take on adatx (Automatic Design of Algorithms Through X) main example (at https://github.com/LudoTheHUN/adatx)

Alas, I'm using a somewhat hacked, older version of cKanren in Racket, so this code may not run under the current release of cKanren.  The equivalent code under core.logic seems to have problems---I suspect there is a bug in the core.logic finite domain solver.

Under Racket I run the examples from the command line:

`raco test mad-at-x.rkt`

Here is the adatx Clojure code to synthesize (fn [x1 x2] (+ x1 x1 x2)) from example inputs/outputs:

(def workings
 (adatx/prob-solve
  {
  :symvec        ['+ '- 'x1 'x2]
  :prog-holder   '(fn [x1 x2] :adatx.prog-hold/prog)
  :in-out-pairs  [{:in [1 2] :out 4}
                  {:in [1 3] :out 5}
                  {:in [2 3] :out 7}
                  {:in [4 3] :out 11}]
  :sandbox :none}))   ;;In general, this may take some time...

(adatx/get-solution workings)    ; => (fn [x1 x2] (+ x1 x1 x2))

((adatx/solution_fn workings) 4 3)     ; => 11



Equivalent miniKanren/cKanren code, using curried functions, binary addition and subtraction operators, and CLP(fd) [constraint logic programming over finite domains] with a minimal domain of 0 to 11:

(run 1 (exp)
  (fresh (x y body)
    (== `(lambda (,x) (lambda (,y) ,body)) exp)
    (evalo `((,exp 1) 2) 4)
    (evalo `((,exp 1) 3) 5)
    (evalo `((,exp 2) 3) 7)
    (evalo `((,exp 4) 3) 11)))

=>

(((lambda (_.0) (lambda (_.1) (+ _.0 (+ _.1 _.0))))
  : (=/= ((_.0 . _.1)) ((_.0 . closure)) ((_.1 . closure))) (sym _.0 _.1)))

It takes cKanren just under 8 seconds to produce the answer under my Mac Book Pro.

The lambda term

(lambda (_.0) (lambda (_.1) (+ _.0 (+ _.1 _.0))))

is alpha equivalent to

(lambda (x) (lambda (y) (+ x (+ y x))))

The constraints after the colon (:) ensure that the _.0 and _.1 represent distinct symbols, neither of which is the "reserved" symbol `closure`.

run 10 produces other lambda expressions, including:

(lambda (_.0) (lambda (_.1) (+ _.0 (+ _.1 _.0))))

(lambda (_.0) (lambda (_.1) (+ _.1 (+ _.0 _.0))))

(lambda (_.0) (lambda (_.1) (+ _.0 (+ _.0 _.1))))

(lambda (_.0) (lambda (_.1) (+ _.0 (+ _.1 ((lambda (_.2) _.0) 0)))))


We don't actually need to specify the lambda expressions---we can just specify the curried applications:

(run 1 (exp)
  (evalo `((,exp 1) 2) 4)
  (evalo `((,exp 1) 3) 5)
  (evalo `((,exp 2) 3) 7)
  (evalo `((,exp 4) 3) 11))

=>

(((lambda (_.0) (lambda (_.1) (+ _.0 (+ _.1 _.0))))
  : (=/= ((_.0 . _.1)) ((_.0 . closure)) ((_.1 . closure))) (sym _.0 _.1)))


It would be very nice for the reifier to represent the range of possible values of the finite domain values as a constraint, instead of instantiating every possible combination of numberic values.  This would lead to much more interesting answers.
