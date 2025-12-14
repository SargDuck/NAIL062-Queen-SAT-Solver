# Peaceably Co-existing Armies of Queens SAT Solver

## The Problem: Peaceably Co-existing Armies of Queens

We have a nxn chessboard. We want to place two equally sized armies of queens:
- one k sized army of white queens
- one k sized army of black queens

We want them to co-exist peacefully, meaning no white queen attacks any black queen and vice versa. Queens in chess attack along rows, columns, and diagonals. The sum of the queens on the board is 2k.

The decision we need to make is given n and k, can we place k white queens and k black queens on the board such that no white queen attacks any black queen and vice versa?

## CNF Encoding:

We want to encode this problem into a boolean formula in CNF form.

### Variables:

We will introduce boolean variables for each potential queen placement:

- Let W_{i,j} be a boolean variable that is true if and only if a white queen is placed on cell (i,j) of the board.
- Let B_{i,j} be a boolean variable that is true if and only if a black queen is placed on cell (i,j) of the board.

We have 2n^2 variables in total. (This will grow very quickly for larger n which will be relevant later).

### Constraints:

1. Cardinality constraints for placement:
   - Exactly k white queens on the board.
   - Exactly k black queens on the board.

We used a naive subset encoding:
- AtMost(k) for white: For every set S of (k + 1) distinct cells, add a clause: (OR_{(i, j) ∈ S} NOT W_{i, j}). Meaning not all k + 1 of them can be white simultaneously.
- AtLeast(k) for white: Let N be the number of board cells (n^2). For every set T of (N - k + 1) distinct cells, add a clause: (OR_{(i, j) ∈ T} W_{i, j}). Meaning at least one of those cells must contain a white queen.

Together, AtMost(k) and AtLeast(k) enforce Exactly(k).

The same is added for black queens using B_{i, j} vars.

2. Non-attacking constraints between armies:
   We dont need to worry about same color attacks.
   A white queen at (i,j) attacks a black queen at (i',j') if:
- They share the same row: i=i'
- They share the same column: j=j'
- They share the same diagonal: i-j=i'-j' or i+j=i'+j'
- 
  For each pair of squares that lie in attack range, we add a constraint that both W_{i,j} and B_{i',j'} cannot both be true simultaneously. This can be encoded as a clause:
  
  (NOT W_{i,j} OR NOT B_{i',j'})
  
We must add such clause for every pair in attacking positions.

3. No overlap of armies in the same square:

   A single cell can't hold both a white and black queen. For each (i,j):
  (NOT W_{i,j} OR NOT B_{i,j})

## Program Documentation:
### Input Format:
- --n <int>: size of the chessboard (n x n).
- --k <int>: number of queens per army.
- --cnf <path> (optional): path to output the CNF formula in DIMACS format.
- --instance <path> (optional): load instance parameters from a file with two lines: n and k.
- --solve: run the Glucose solver on the generated CNF.
- --stats: print solver statistics (must run with --solve).

### Output Format:

If SAT:
- Prints "SAT"
- Prints a board configuration showing the placement of white (W) and black (B) queens, and empty cells (.).

If UNSAT:
- Prints "UNSAT"

If --stats is given:
- Prints solver statistics reported by Glucose after the solution.

## Instances:

### Small positive (satisfiable) humanly readable instance:
- n = 4, k = 2.

### Small negative (unsatisfiable) humanly readable instance:
- k = n = 2

### Nontrivial satisfiable instance: unconfirmable
I tried n = 7, k = 5 and it took 6.31s which is under the minimum requirement. However, I tried more complex instances such as n = 8, 8, 7, 7 and k = 8, 7, 7, 6 respectively and they all timed out. 

## Experiment report:

The n = 7, k = 5 instance was solve with Glucose CPU time ≈ 5.1s as reported in the solver statistics. In reality, my script's python CNF generation time took much longer since the method that I collect CNF clauses with grows really fast with n and k. For n = 7, k = 5, the generated formula has 98 variables and about 28.4 million clauses, and the resulting DIMACS file size is roughly 0.76 GB. This tells us that the that the naive encoding I used leads to an enormous number of clauses, and better encoding methods must be used if we seek to scale the variables on this problem further.

The CNF files and solver statistics for each instances are in the "instances" folder. 

(NOTE: I wasn't able to upload the n7k5.cnf (760 MB) file to Github as they have a 100MB file limit. I replaced the file with screenshots of the first few lines, last few lines, and size of the file. You can generate the cnf with "python QueenSolver.py --n 7 --k 5 --cnf n7k5.cnf" but it will take some time)