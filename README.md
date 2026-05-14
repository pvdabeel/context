# context.pl

**Contextual object-oriented logic programming for SWI-Prolog.** A
single-file library that turns the Prolog module system into a
namespace-aware OO meta-protocol with classes, instances, multiple
inheritance, public / protected / private guards, optional declarative
typing, and a small set of operators for state mutation and remote
invocation.

The runtime approach (vs. compile-time translation as in
[Logtalk](https://logtalk.org/)) means contexts come into existence as
they are referenced and instances are dynamically generated; no
preprocessing step, no extra build dependency.

```prolog
:- use_module(library(context)).

% see examples/person.pl for a worked example
:- consult(examples/person).

?- alice:newinstance(person).        % create instance 'alice' of class 'person'
Person constructor - alice
?- alice:set_name(alice).
?- alice:set_age(34).
?- alice:get_name(N).
N = alice.
?- alice:verify_age(34).
true.
?- alice:verify_age(-1).
false.
```

## What it gives you

- **Contexts (namespaces)**. Clauses are local to their context unless
  exported. Referencing a context name is enough to create one.
- **Classes and instances** via `:- class.` / `:- class([Parents]).`
  and `Inst:newinstance(Class)`.
- **Visibility guards**: `dpublic/1`, `dprotected/1`, `dprivate/1`,
  `dstatic/1`, `dmutex/1`. Predicates declared in one tier are not
  callable from the next.
- **Multiple inheritance** with conflict resolution; deep clones via
  `inherit/1`.
- **State mutation operators** for instance data members:
  `<= Fact` (assert / replace), `<+ Fact` (add), `<- Fact` (retract),
  `:: Fact` (read), `: Fact` (verify in current context).
- **Cross-context invocation**: `Inst::Goal`, `Class:Inst://Goal`.
- **Thread-aware token passing** so the same class can serve multiple
  threads concurrently without leaking access tokens.
- **Reflection**: `instances/2`, `declared/1`, `implemented/1`.

## Files

```text
context.pl                  the library (single file, ~900 lines)
examples/person.pl          worked example: class with public/protected/
                            private members, constructor, destructor,
                            type-checked setters, list members
examples/README.md          one-line index of the examples directory
LICENSE                     2-clause BSD-style license
```

## Install

```sh
git clone https://github.com/pvdabeel/context.git ~/lib/context
swipl -p library=~/lib/context context-driven-program.pl
```

Or copy `context.pl` into your project tree and `:- use_module(context).`
The library has no external dependencies beyond a current SWI-Prolog
(7.x or newer).

## Operators (quick reference)

| Operator | Position | Meaning                                                |
|----------|----------|--------------------------------------------------------|
| `::-`    | 1200 xfx | Class clause head: defines a context-aware predicate.  |
| `<=`     |  600 fx/xfx | Assert / replace a (single) fact in the instance.   |
| `<+`     |  600 fx/xfx | Add a fact to a multi-valued instance member.       |
| `<-`     |  600 fx/xfx | Retract a fact from an instance member.             |
| `::`     |  601 xfx | Cross-context call (e.g. `alice::get_age(A)`).         |
| `:`      |  600 fx   | Verify a fact holds in the current context.           |
| `://`    |  603 xfx | Execute a goal in a given context (`alice://G`).       |

## Module declaration shape

```prolog
:- module(my_class, []).      % empty export list; visibility is via dpublic
:- class.                     % or :- class([parent_a, parent_b]).
:- dpublic(method/1).
:- dprotected(helper/1).
:- dprivate(state/1).

method(X) ::-
    :verify_state(X),         % verify in 'this' context
    <=state(X).               % replace stored state
```

## Origin

`context.pl` was extracted from the
[`portage-ng`](https://github.com/pvdabeel/portage-ng) repository,
where it underpins the runtime OO layer of the resolver
(`Source/Knowledge/repository.pl`, `knowledgebase.pl`, etc.). It
predates the rest of portage-ng and stands alone — extracted here so
other Prolog projects can use it without pulling in a 30k-ebuild
package manager. Full git history is preserved (since the original
`Initial import` commit).

This library was originally inspired by Logtalk's contextual logic
programming, but takes a runtime rather than compile-time approach.

## License

2-clause BSD-style. See [LICENSE](LICENSE).
