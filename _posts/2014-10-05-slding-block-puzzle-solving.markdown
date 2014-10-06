---
layout: post
title:  "Sliding Block Puzzle Solver Algorithm"
date:   2014-10-05 00:00:00
categories: ['groovy','java','algorithms']
---

### The Puzzle

This was a fun exercise to write a solver for an NxN sliding block puzzle.  This is also known as a 16-puzzle.  

A board consists of NxN spaces, with numbers 1 to (N^2-1).  In a 4x4 puzzle, this tiles are numbered 1-15.   One space is empty, and is used to shuffle the tiles around the board.
  
The goal of this exercise is to write a solving algorithm to solve the NxN puzzle.
  
Notes:
- A locked spot/row/column is ineligible to be visited by the empty spot.
- I think of a board as a grid, from (0,0) in the top left to (N,N) at the lower right.  
- Moves are in relation to the empty spot.  For instance, moving *up* would move the tile above the empty spot *down*.
- This does not find the shortest set of moves, but trades this for a reasonable complexity against large puzzles (e.g., 10x10)
  
### The Algorithm - high-level:
1. Starting in the top left corner of the board (0,0)
2. Solve the row 
3. Solve the column
4. Advance diagonally to the next corner (1,1) repeat steps 2-4 until there are only 3 left in the bottom square.
5. Crunch Last Square: Solve last 3 tiles: until solved, or too many attempts are made, move empty space clockwise: left,up,right,down
6. Optimize: Remove every no-op move pairs: (up,down), (down,up), (left,right), (right,left)

{% highlight groovy %}

    def solve(){
        (0..(edgeSize-3)).each{cornerIndex->
            solveEdges(cornerIndex)
        }
        if(!isSolved()) crunchLastSquare()
        optimizeMoves()
        return this;
    }
    
{% endhighlight %}

### Deeper: To solve a row:
1. For each number except the last two in the row, move the target value into its spot and lock it
2. Move the *last* value in this row to the bottom right spot in the board
3. Stage the *second to last* value in the *last* spot in the row and lock it.
4. Stage the *last* value beneath the *second to last* value and lock it
5. Unlock both values
6. Sweep both values into place via the second-to last spot (moves: right,down).  Lock the row.

{% highlight groovy %}

private def solveRow(def edge){
        //solve all spots except last two entries
        (edge..edgeSize-3).each{col->
            def val = solvedPuzzle[edge][col]
            moveValToSpot(val, edge, col)
            lockSpot(edge,col)
        }

        //dig last one out of there, in 1,2,3,4 - this is 4
        def lastValInRow = solvedPuzzle[edge][edgeSize - 1]
        moveValToSpot(lastValInRow, edgeSize - 1, edgeSize - 1)

        //stage second to last, in 1,2,3,4 - this is 3
        def val = solvedPuzzle[edge][edgeSize - 2]
        moveValToSpot(val, edge, edgeSize-1)
        lockSpot(edge,edgeSize-1)

        //stage last val (4)
        moveValToSpot(lastValInRow, edge+1, edgeSize-1)
        lockSpot(edge+1,edgeSize-1)

        //sweep them in
        gotoSpot(edge, edgeSize-2)
        unlockSpot(edge,edgeSize-1)
        unlockSpot(edge+1,edgeSize-1)
        right()
        down()

        lockedRows.add(edge)
        lockedSpots.clear()
    }
    
{% endhighlight %}

(The column algorithm is the same, except rotated vertically)

### Yet Deeper: To move a target value from it's current location (A) to a particular spot (B):
1. Find the location of the value on the board
2. Find the path between its location and the target location (x,y) as a set of nodes with moves (e.g, `up`, `down`, `left`, `right`)
3. Execute that set of moves *against* the target value (e.g., `up` means "move the empty space above the value, then move down")

High-Level Algorithm to move a target value `val` to (x,y).

{% highlight groovy %}

    private def moveValToSpot(def val, x, y){
        def location = findVal(val)
        def path = findNodePath(location[X], location[Y], x, y);
        path.each{node->
            moveVal(val, node.moveName)
        }
    }
    
{% endhighlight %}

`moveVal(val, moveName)` essentially moves a target value in a particular direction.  It locks the spot to avoid passing over it.

Take note of the `DELTA_MAP` which knows where the empty space should be to execute an operation.  For instance, to move something *up*, the empty space has to be ABOVE the target value, then MOVE DOWN.

{% highlight groovy %}

    static def DELTA_MAP = [
                up : [delta: [-1, 0], opposite: DIRECTION_DOWN],
                down : [delta: [1, 0], opposite: DIRECTION_UP],
                left : [delta: [0, -1], opposite: DIRECTION_RIGHT],
                right : [delta: [0, 1], opposite: DIRECTION_LEFT]
    ]

    private def moveVal(def val, def direction){
        def location = findVal(val)
        def x = location[X]
        def y = location[Y]
        lockSpot(x,y)
        def delta = DELTA_MAP.get(direction)
        def possible = gotoSpot(x+delta.delta[X],y+delta.delta[Y])
        unlockSpot(x,y)
        if(possible) move(delta.opposite)
    }
{% endhighlight %}

Goto is implemented as finding the path between two spots, and moving there.

Note: This is different from `moveValToSpot` because it is an operation on the empty space itself.

{% highlight groovy %}

    private def gotoSpot(x,y){
        if(x == emptyX() && y == emptyY()){ return true; }
        def path = findNodePath(emptyX(), emptyY(), x, y)
        if(path == null){
            return false
        }
        path.each{
            it.move()
        }
        return true;
    }
{% endhighlight %}

### Even Yet Deeper: To find a path between two spots (x1,y1) to (x2,y2):

This is a recursive algorithm for finding the path between two spots.  The result is a list of anonymous objects that each have a functional reference to the move operation against the empty space.

This algorithm is used for two purposes: moving a target value from A to B, and also moving the empty space around the board (to accomplish the former).

To find path from (x1,y1) to (x2,y2), while avoiding obstacles
1. If already at the target spot, return the current path (YAY!)
2. Otherwise, find all *eligible next spots* that from (x1,y1) that have *NOT* been visited before and SORT this list by distance to the TARGET spot (closest first)
3. For each move, put this move in a copy of the 'current path', and try to calculate the path from this node to the TARGET (x2,y2).
4. If path is found, return the path (YAY!)
5. If path is null, then remove entry from the list and try next value in list.

Algorithm in groovy:

{% highlight groovy %}

    private def getPath(x1, y1, x2, y2, currentPath){
        if([x1,y1] == [x2, y2]){
            return currentPath //SUCCESS
        }

        def nextSteps = getAdjacentNodes(x1, y1).findAll{adjacentSpot->
            !currentPath.any{it.spot == adjacentSpot.spot}
        }?.sort{
            getDistance(x2, y2, it.spot[X], it.spot[Y])
        }

        for(def node : nextSteps){
            currentPath.add(node)
            def tempPath = currentPath.clone()
            def path = getPath(node.spot[X], node.spot[Y], x2, y2, tempPath)
            if(path != null){
                return path  //SUCCESS
            }else{
                currentPath.remove(currentPath.last())
            }
        }
    }
{% endhighlight %}

Distance calculation (Remember from Algebra?):
{% highlight groovy %}
    private static def getDistance(x1, y1, x2, y2){
            Math.sqrt(Math.pow((y2-y1),2) + Math.pow((x2-x1),2))
    }
{% endhighlight %}

Eligible values next to (x,y) (I thought this was cool):
{% highlight groovy %}
    private def getAdjacentNodes(x,y){
           [ [spot: [x, y+1], moveName: DIRECTION_RIGHT, move: {right()}],
             [spot: [x, y-1], moveName: DIRECTION_LEFT, move: {left()}],
             [spot: [x+1, y], moveName: DIRECTION_DOWN, move: {down()}],
             [spot: [x-1, y], moveName: DIRECTION_UP, move: {up()}]].findAll{
               isLegalLocation(it.spot[X], it.spot[Y])
           }
    }
{% endhighlight %}

Is legal location is defined as "on the board" AND "not locked"
(It's strucutred this way for readability...but I could be talked out of it)
{% highlight groovy %}
    private def isLegalLocation(x,y){
            if(lockedRows.contains(x) || lockedColumns.contains(y)){
                return false;
            }
            if(lockedSpots.contains([x,y])){
                return false;
            }
            if(x > (edgeSize-1) || x < 0){
                return false;
            }
            if(y > (edgeSize-1) || y < 0){
                return false
            }
            return true
        }
{% endhighlight %}

###Get the source code, if you're interested in trying it out:
[Get the Source](https://github.com/stevenlanders/block-puzzle-solver)
[Example GUI (knockoutjs)](https://blockpuzzle-stevenlanders.rhcloud.com/puzzle)

