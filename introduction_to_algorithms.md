# O-notation, -notation, and ‚-notation
O-notation characterizes an upper bound on the asymptotic behavior of a function. In other words, it says that a function grows no faster than a certain rate, based on the highest-order term. Consider, for example, the function 7n 3 C 100n 2  20n C 6.
Its highest-order term is 7n 3 , and so we say that this function’s rate of growth is n 3 . Because this function grows no faster than n 3 , we can write that it is O.n 3 /.

Y-notation characterizes a lower bound on the asymptotic behavior of a function. In other words, it says that a function grows at least as fast as a certain rate, based 4as in O-notation4on the highest-order term. Because the highest-order term
in the function 7n 3 C 100n 2  20n C 6 grows at least as fast as n 3 , this function is Y.n 3 /. This function is also Y.n 2 / and Y.n/. More generally, it is Y.n c / for any constant c හ 3.

‚-notation characterizes a tight bound on the asymptotic behavior of a function. It says that a function grows precisely at a certain rate, based4once again4on the highest-order term. Put another way, ‚-notation characterizes the rate of growth of the function to within a constant factor from above and to within a constant factor
from below. These two constant factors need not be equal.

# Divide-and-Conquer
- Divide the problem into one or more subproblems that are smaller instances of the same problem.
- Conquer the subproblems by solving them recursively.
- Combine the subproblem solutions to form a solution to the original problem.
