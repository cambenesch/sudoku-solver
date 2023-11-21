Input format: 
	User specifies the board size (e.g. 4, 9) and the constraints in the "User inputs" section of the Jupyter notebook. The constraints are specified as a list of (row, column, value) tuples, so if for example we want to solve a puzzle where the top leftmost box is given to be 3, then we would add (0,0,3) to the constraints list. User may also set display_all_solutions = True to display all solutions as they are found, up to a max of 100 solutions. 



Running the code: 
	Simply run the notebook in a Jupyter environment. I'm using Python 3.9.2 to produce the output shown. In my terminal, I type jupyter notebook, then I navigate to the notebook, and hit the menu option Restart Kernel and Run All Cells. 




Discussion of techniques: 
	First we construct the board based on the constraints, stored as `puzzle`. This fills in the constraints and sets all other cells to zero, then subtracts 1 from all cells to switch from 1-indexing to 0-indexing. 
	Next, we instantiate a Solver, to which we will add clauses that must be satisfied in any solution to the puzzle. 
	Then we create the Boolean operators as an N x N pandas DataFrame of N-length Boolean vectors. bools.iloc[i,j] contains the vector indicating the value of cell i, j, such that if board[i,j] == k in a solution, we have bools.iloc[i,j][k-1] as True and all other entries in bools.iloc[i,j] as False. Immediately after we construct this Bool vector, if the initial puzzle already assigned a value to puzzle[i,j], then we force board[i,j] to take that value by adding a constraint. 
	Then we add in the generic constraints. Each cell must contain at least 1 value, so we add an Or clause for the Bool vector at bools.iloc[i,j]. Each row must contain at least one of each value 0..N-1, so we add another Or clause for each possible value, for each row. We do the same for columns. Then for boxes, with a bit more complicated indexing, we add a similar "at least one cell in the box must have value k" clause. The "at most one" aspect is implictly enforced for these constraints, by the pigeonhole principle. 
	Finally, we ensure that no one cell is assigned multiple values. Hence for each possible value pair (l, k) with l != k, we add a constraint that C_ijk and C_ijl are not both True. 

	Then we look for a solution. If we don't find one, then we say so. If we do, then we look for another solution. We do this by adding a constraint that the board may not be identical to this solution. In particular, looking at the True Bools in the most recent solution, we ensure that at least one of those is set to False in any future solutions. Every time we encounter another solution, we add this new constraint, so that no new solution is a duplicate of any previously found solution. We finally stop after finding 100 solutions, and we print out the last solution found.
