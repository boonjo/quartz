---
title: Sudoku
enableToc: true
draft: false
---

I have been playing Sudoku lately and wanted to code a solver. At first, I thought it would be a little difficult because I considered it an NP-complete problem. But I quickly realized it only applies to a nxn board and not a standard 9x9 board. 

However, that did not stop me from implementing it. At least it was somewhat fun. 

---

## Visualizing the Board
The board that I used to solve was this:
~~~ py
board = [
            [8, 0, 0, 0, 0, 0, 0, 0, 0],
            [0, 0, 3, 6, 0, 0, 0, 0, 0],
            [0, 7, 0, 0, 9, 0, 2, 0, 0],
            [0, 5, 0, 0, 0, 7, 0, 0, 0],
            [0, 0, 0, 0, 4, 5, 7, 0, 0],
            [0, 0, 0, 1, 0, 0, 0, 3, 0],
            [0, 0, 1, 0, 0, 0, 0, 6, 8],
            [0, 0, 8, 5, 0, 0, 0, 1, 0],
            [0, 9, 0, 0, 0, 0, 4, 0, 0]
        ]
~~~

I first wanted to visualize the board and then think about an approach to solving it. So I used a nested for loop to format the Sudoku board. 

~~~ py
for i in range(len(board)):
    for i % 3  == 0 and i != 0:
        print("---------------------")
    for j in range(len(board[i])):
            if j % 3 == 0 and j != 0:
                print("| ", end = "")
            if j == 8:
                print(board[i][j])
            else:
                print(str(board[i][j]) + " ", end = "")
~~~

To catch every 3x3 grid within the board, I added the modulo operation to add sectional characters when printing the board, which will make the board look something like this:
~~~ py
8 0 0 | 0 0 0 | 0 0 0
0 0 3 | 6 0 0 | 0 0 0
0 7 0 | 0 9 0 | 2 0 0
---------------------
0 5 0 | 0 0 7 | 0 0 0
0 0 0 | 0 4 5 | 7 0 0
0 0 0 | 1 0 0 | 0 3 0
---------------------
0 0 1 | 0 0 0 | 0 6 8
0 0 8 | 5 0 0 | 0 1 0
0 9 0 | 0 0 0 | 4 0 0
~~~

## Looking for Empty Cells
Without knowing what algorithm to use to solve Sudoku, I was pretty sure that I would need to know which cells were empty. At first, I thought of implementing a boolean function that checks whether there are any empty cells on the board. However, with more time, I thought that would be less effective since I wouldn't know which cell was empty. 

So, I decided to write a function called `isEmpty` that will print the first empty cell or return None. This will mean that I can take that empty cell and fill in the correct value and move on to other empty cells until complete. 

~~~ py
for i in range(len(board)):
        for j in range(len(board[i])):
            if board[i][j] == 0:
                return i, j  
    return None
~~~

For the board that I am working with, it will return `0 1`.

## Checking if Sudoku is Completed
The last implementation is a boolean function that is essential would be checking if the value that was inputted to an empty cell is valid. 

So to check if a cell is valid in the board, there are three conditions that need to match. 
1. Unique value in its *row*
2. Unique value in its *column*
3. Unique value in its *3x3 grid* 

### Checking the row and column
I used for loops to see if there is any duplicate value from the cell value that is being checked. 
~~~ py
crow = cell[0]
ccol = cell[1]

for i in range(len(board)):
    if number == board[i][ccol]:
        return False
for j in range(len(board[crow])):
    if number == board[crow][j]:
        return False
~~~

### Checking the 3x3 grid
Now to check the 3x3 grid, I wanted to find how big the grid has to be to which sides. So based on the current cell index, I used the modulo operation to figure out where the cell is located within the grid. Then, compare the values to the cell value to check if it's unique. 
~~~ py
row_mod = crow % 3
col_mod = ccol % 3

if row_mod == 0:
    row_rangel = crow
    row_rangeh = crow + 2
elif row_mod == 1:
    row_rangel = crow - 1
    row_rangeh = crow + 1
elif row_mod == 2:
    row_rangel = crow - 2
    row_rangeh = crow
    
if col_mod == 0:
    col_rangel = ccol
    col_rangeh = ccol + 2
elif col_mod == 1:
    col_rangel = ccol - 1
    col_rangeh = ccol + 1
elif col_mod == 2:
    col_rangel = ccol - 2
    col_rangeh = ccol

for i in range(row_rangel, row_rangeh+1):
    for j in range(col_rangel, col_rangeh+1):
        if number == board[i][j]:
            return False
~~~

## Solving Sudoku
To solve a puzzle, one must know the rules to it. So to list out some for Sudoku:
- Each cell must contain a number ranging 1-9
- Each row must contain each number
- Each column must contain each number

After giving some rigorous thought, I came to the conclusion to implement a Backtracking Algorithm.

### Backtracking Algorithm
- Though it sounds fancy, it's really just a brute-force approach to finding the solution.
- The basic principle is that if the current solution is incorrect, then it will backtrack and try another solution through recursion.
- It can be argued that this approach will not always find the most optimal solution. However, I'd say a good Sudoku board will always have one unique solution.
- This algorithm performs a Depth-First Search through the possible solutions. Hence the worst-case time complexity will be **$O(n^m)$** 
  - $n$ is the number of possible values for each cell, in this case it will be 9
  - $m$ is the number of spaces that are blank
- Looking at the time complexity, it can be concluded that the brute force approach will complete in a rather fast time.

To translate that into code, I first wanted the algorithm to visit the empty cell.
~~~ py
empty_cell = isEmpty(board)
if empty_cell is None:
    return True
~~~
I used the `isEmpty` function to find the first empty cell. Then I threw in an if statement to check if there are any, and the solver function will return `True` which would stop the solving process. 

With an existing empty cell, it will now insert a value sequentially until it is a valid value.
~~~ py
else:
    row, col = empty_cell
    for number in range(1, 10):
        if isValid(number, empty_cell, board):
            board[row][col] = number

            if solveBoard(board):
                return True
            
            board[row][col] = 0
    
return False
~~~

This is the main part of the algorithm. What's happening here is that I get the location of the empty cell, then find a valid number to place in the cell. Then the recursion will take place by calling the function again with the updated board. This will keep going until it reaches a point where it will have all valid values or have a value that was incorrect at some point and it will backtrack to that spot and go again.

Some might think that this algorithm code is too simple. But that was one of the reasons why I chose this approach. It will guarantee a solution and the solving time is unrelated to the difficulty of the board. 

The solution comes out to be this:
~~~ py
8 1 2 | 7 5 3 | 6 4 9
9 4 3 | 6 8 2 | 1 7 5
6 7 5 | 4 9 1 | 2 8 3
---------------------
1 5 4 | 2 3 7 | 8 9 6
3 6 9 | 8 4 5 | 7 2 1
2 8 7 | 1 6 9 | 5 3 4
---------------------
5 2 1 | 9 7 4 | 3 6 8
4 3 8 | 5 2 6 | 9 1 7
7 9 6 | 3 1 8 | 4 5 2
~~~

## Things to Improve on
Although this simple approach does a very good job on most occasions, some Sudokus can work against this backtracking algorithm. For example, take a look at this board:
~~~ py
0 0 0 | 0 0 0 | 0 0 0
0 0 0 | 0 0 3 | 0 8 5
0 0 1 | 0 2 0 | 0 0 0
---------------------
0 0 0 | 5 0 7 | 0 0 0
0 0 4 | 0 0 0 | 1 0 0
0 9 0 | 0 0 0 | 0 0 0
---------------------
5 0 0 | 0 0 0 | 0 7 3
0 0 2 | 0 1 0 | 0 0 0
0 0 0 | 0 4 0 | 0 0 9
~~~
As you can see, there are no clues in the top row. This is not great for the backtracking algorithm since it works from top to bottom and increments the values. Ultimately means to solve this board, it will take significant time. From my testing environment, the algorithm was able to solve this board in about 8 minutes. Comparing that to less than 1 second for the first example board that I solved, it is evident there are weaknesses in some cases. 

Here is what the solved board would look like:
~~~ py
9 8 7 | 6 5 4 | 3 2 1
2 4 6 | 1 7 3 | 9 8 5
3 5 1 | 9 2 8 | 7 4 6
---------------------
1 2 8 | 5 3 7 | 6 9 4
6 3 4 | 8 9 2 | 1 5 7
7 9 5 | 4 6 1 | 8 3 2
---------------------
5 1 9 | 2 8 6 | 4 7 3
4 7 2 | 3 1 9 | 5 6 8
8 6 3 | 7 4 5 | 2 1 9
~~~

Although the speed of completion can be aided with faster processors, it is rather wiser to improve the algorithm. Some ideas that I will try later are listed below.
- Try implementing Crook's Pencil and Paper Algorithm.