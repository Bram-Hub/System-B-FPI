# System B FPI
## Authors
2007:
Jonathan Chang

## About
Introduction
To use this program, open systemb.html in a browser. Firefox and Opera are recommended,
but Internet Explorer will also work.
The help page contains information on how to use the program, so this write-up will be
about how it works and how to modify it.
Parsing
The parser will take a propositional logic statement and create a statement tree. Depending
on the arguments to the function, word strings will be parsed either as atomic propositions or as
statement variables. The parser handles extra sets of parentheses by ignoring them, and it handles
improperly entered parentheses by giving a parse error. The function for parsing is
parseStmtWrapper.
The complement of the parser is the stmtTreeToTextWithEnv function, which takes
a statement tree and returns a logical statement (a string).
Checking Steps
There are three main algorithms/functions involved in checking the validity of steps and in
automatically generating statements: matchTreesExact, matchTreesInfer, and matchTreesEquiv. These
use pattern-matching, searching, and variable bindings. Rules are kept in the variable
source_rules, which contains logical statements that are later parsed into statement trees.
Variable Environment
A variable environment is a collection of variable assignments. A variable assignment is amapping from statement variable identifiers (e.g. "phi") to statement trees. An environment can contain
variables that are unbound. There are functions for e.g. checking whether a variable exists in an
environment, retrieving statement trees, and adding variable assignments.
matchTreesExact
This function takes two statement trees and a reference to a variable environment. It
recursively compares these two trees by comparing corresponding substatements. When comparing a
statement variable to something else, it checks the environment. If the variable is bound, it accesses the
statement that it is bound to and recursively compares. If the variable is not bound, it binds it by adding
an assignment to the variable environment and accepts the matching for that pair of substatements.
Comparisons involving atomic propositions and operators occur as expected. This function is capable
of binding one statement variable to a statement tree that has a statement variable, but this does not
happen. The return value of this function indicates whether the two trees could be matched using only
variable bindings. Since the third argument is a reference, variable assignments added to the
environment can be accessed by the calling function when matchTreesExact returns.
matchTreesInfer
This function takes several statement trees: one for the resulting/right-hand-side statement
entered by the user (which may be empty), one for the rule representation of the resulting/right-hand-
side statement (e.g. phi -> psi in CP), and others for the cited statements and left-hand side rule
statements. It pairs up rule statements and user-entered statements and attempts to match them using
matchTreesExact. This function only supports up to 2 left-hand-side statements in the rule. It will
try all possible pairings of the user-entered and left-hand-side rule statements. For any one possible
pairing, it keeps the variable environment that results from each call to matchTreesExact and
passes it into other such calls. If it can match the cited statement trees to the left-hand-side rule
statement trees and the resulting statement tree to the right-hand-side rule statement tree, it will return asuccess condition.
If the user-entered resulting statement tree is empty, this function will essentially return the
right-hand-side rule tree, along with the relevant variable environment, through references in the
arguments. Calling functions should know beforehand whether matchTreesInfer will
automatically generate a new statement tree for the resulting statement.
This function also contains special cases for CP, generalized simplification, generalized
addition, and conjunction because those rules have added functionality and will not fit into this
framework well. There is also some similar code for other rules, but that code is disabled and may not
work properly.
matchTreesEquiv
This function takes two user-entered statement trees and two rule statement trees. It
recursively compares the two user-entered statement trees in a manner similar to that of
matchTreesExact. If a pair of substatements is identical, then this function returns a condition
indicating that they are exact matches. If it is not (i.e. if the operators, the number of operands, or any
pair of operands don't match), this function will attempt to apply the equivalence rule to whole user-
entered substatements trees. It does this by using matchTreesInfer and taking its return value
(whether the match was successful). If this succeeds, it returns a condition indicating that the
substatements match after applying the rule. Otherwise, it will try applying the equivalence rule in the
other direction. If this also fails, it returns a condition indicating this.
This algorithm decreases the amount of work necessary by applying equivalence rules only
where the two user-entered statements differ. It assumes that there are no nested applications of the
rule, but it can handle multiple non-nested applications. The reasoning is that if a pair of corresponding
substatements differ, the rule must have been applied at this level or above, so it can avoid recursively
matching any of the operands. If a matching fails at any level, the problem is passed back up.A minor problem that occurs because of this algorithm is the case where an application of
an equivalence rule results in no differences. For example, commutation on P /\ P results in P /\ P.
Since the algorithm only applies the rule when there are differences, it will not apply a rule in this case.
The function that handles rule checking will know that these two statements are identical but will not
know if the application of commutation is valid. The program handles this problem by changing the
selected rule to reit and then accepting it.
Adding Rules
It is possible to easily add rules through the source code. The variable source_rules, near
the top of the file, is an array of uncompiled rules. Each source rule should have the form
["(display name)", "(internal rule identifier)", [(array of rule
forms)]], where each rule form is a string of the form "(statement) <=> (statement)"
or "{(statements separated by commas)}|-(statement)". Each source rule
contains an array of rule forms so that syntactically different rules can be grouped together under
common names. There are some restrictions on source rules:
* The forms of any given source rule should be only equivalences or only inferences
* The number of left-hand-side statements in an inference should be 0, 1, or 2
* A rule form should have one statement on the right-hand side
* Rules can't have sub-proofs, e.g. as in CP
* Internal rule identifiers should be unique and can't be "invalid"
* The number of left-hand-side statements in all forms of an inference rules should be the same
Source rules that adhere to these restrictions will be compiled the next time the page is loaded. The new
rules should appear in the rule drop-down menu. The program can check these rule applications for
validity and can automatically generate statements for these rules.
Other ModificationsOther modifications of this program won't be as easy as adding rules, but there are some
features that may make them easier. All functions have javadoc-style comments. There are also
comments that describe the program's low-level data structures. There is a debug function that outputs
messages to the page. This is disabled by default but can be enabled by setting the variable
enableDebugMessages to true.
Firefox has a debugger that gives helpful JavaScript error messages, and it also has a good
DOM inspector.
