---
layout: default
title: Homework 6 (a.k.a. Project 3)
nav: assignments
hwpath: hw6
---
## Homework 6 (Part 3 of Project)

+ Due: Friday, November 11, 11:59pm
+ Directory name in your github repository for this homework (case sensitive): `{{page.hwpath}}`.
+ Provide a `README.md` file.

### Overview

This is the third part of your project. As mentioned in the context of HW4, it requires you to implement a Scrabble AI so that you can play against computer-controlled opponents. You should implement at least two AIs, one that always makes a move of maximum immediate score, and one that always uses as many letters as possible. (The latter will be enjoyable for a human to play against.) If you want to write additional AIs, you are free to come up with additional ones beyond that. You will notice that much of the functionality between different AIs will be common, so this would be a fantastic time to use inheritance and quite possibly abstract classes.

For this part of the project, you have three choices regarding what codebase you want to work off:

1. Extend your HW4 submission (or a debugged version), and obtain a console version.
2. Extend your HW5 submission (or a debugged version), and obtain a Qt version.
3. Use the code base we provide. In the homework-resources repository, we are providing .h and .o files of all the functional classes of the game (Board, Bag, Dictionary, Player, Move, Tile, Square, Exceptions, Game, Util). Pretty much the only thing missing is the interaction with the players, which you can add via console or Qt.

You can choose one of the codebases based on any criteria you like, in particular, which one you think will be easiest for you to work with. The only exception is that if you want to enter your AI into the Scrabble AI tournament (below), you will need to use our codebase.

If you use our codebase, you should be aware that we only provide .h and .o files, so you cannot modify it. We hope that our code is mostly (or completely?) bug-free, but if you use it, you should keep a close eye on Piazza and changes in the repository, in case you or we find bugs and fix them. Please report any bugs you find quickly to us via Piazza.

Our recommendation is that if your own version is completely or mostly working, then unless you plan to enter the tournament, your own codebase is probably easier to work with. On the other hand, if your version is missing a bunch of functionality or constantly teetering on collapse, it may be faster to learn how our classes work and interact and build your game around them.


### Player Types

When the user supplies a player name, if the user enters a string whose first four letters are `CPUS` (in upper or lower case), then this should be a computer player, whose moves are entirely dictated by the AI you write in this project, specifically an AI that always chooses the maximum score at each move.

If the user enters a string whose first four letters are `CPUL` (in upper or lower case), then this should be a computer player that always chooses to play the maximum possible number of letters.

If you write multiple AIs, you can invent ways in which the name indicates which one to use (e.g., `CPUFriendly`, `CPUGreedy`, `CPUKickAss`). Just document it in your README so we know how to use it.

### Configuration File

The configuration file may now specify one additional input file, to initialize the board with some tiles. (We want to use this for testing, to give you a half-filled board to add words into.) The config file will now look something like this:

```
NUMBER:   7
TILES:    english-tileset.txt
DICTIONARY: ./dictionaries/english-dictionary.txt
BOARD:  ./boards/standard-scrabble.txt
INIT:   init-state.txt
```

The "INIT" parameter is optional. If it is not provided, then the game just starts with an empty board. If there is an init file, then the format is given below, and the tiles are placed on the board before the first move. After this, the player does not have to use the start square any more (even if the init file happens to be empty).

### Initialization State File

The initialization state file will be given in a format similar to what we (and most of you) used for outputting the board state in HW4. A 5x5 board could be described as follows:

```
.........W04...
.........O03...
.........r10...
H03e01L02L04O01
.........D03...
```

The file will have as many lines as the board has rows. For each tile, there are three characters. If the position is empty, then these three characters are all dots. Otherwise, the first character is a letter (no blanks), while the next two characters give the score. Single digit scores are written as `01`, `02`, etc. Letters could be uppercase or lowercase. Scores even for the same letter could be different, and also could be different from scores for the letter in the bag. A [15x15 example](init.txt) for you to start from is in the repo, and also linked here.

### Backtracking

You will need to write code to determine moves for computer-controlled players.  This code will use a form of backtracking to determine what move to make.

On a computer-controlled player's turn, your AI should consider all possible starting coordinates and directions for the move.  The AI should then consider all possible subsets of tiles and permutations of those tiles, using that starting location and direction, to make a move.  If one or more tiles are blank tiles, the AI will also want to consider all possible letters to assign the blank. This exhaustive search will likely involve backtracking.  While it sounds like this is a lot of possibilities to consider, you can do a back-of-the-envelope calculation and see that in "computer quantities", it's actually not too much for small-ish hand sizes. 

By running backtracking search, your AI will produce a collection of legal moves. From among these moves, it should then choose the move to actually play, either along the way or afterwards. (You can probably reuse more code with the latter option.) As we mentioned above, you should produce at least one AI that always plays the immediately highest-scoring move, and another that always chooses a longest move (placing the most tiles). You can break ties between equally long moves, or equally highly scoring moves, arbitrarily.

Your AI does not need to look ahead and optimize several moves in advance, though if you want to enter the tournament, you may want to consider this. As far as regular homework scores, this will not affect your grade.

For large hand sizes, the backtracking search will take a very long time, so we want to add some additional run-time improvements, via the use of an additional data structure described below. Another thing that would slow down the search a lot is having a lot of blanks. Here, we promise you that you will never have more than two blanks.

### Word Prefixes

You should create a way to terminate a search when you've assembled a prefix that cannot be completed into a word. For instance, if you're trying a branch that starts with "xz", it doesnt matter what letters you add later - you won't be able to form a word. So you need a data structure that lets you find out whether a prefix can be completed into a word.

The most natural candidate for this is a `set<string>`, containing exactly the strings that are prefixes of words in the dictionary. The best time to construct this set would be either when you read in the dictionary or when you initialize your computer player. In that case, you need to give your computer player a list/set of all words in the dictionary. (If you use our codebase, you cannot alter the `Dictionary` class, so you will have to go down this route; for this purpose, our `Dictionary` class provides a function `allWords()` that returns a set of all words in the dictionary.)

Just to illustrate what prefixes mean, for the word "pokemon", the prefixes are "" (the empty string), "p", "po", "pok", "poke", "pokem", "pokemo", and "pokemon". 

You can use this set to cut short a path in your backtracking algorithm.  Here is an example of how this might work:

When you are considering all permutations of a given set of letters, your algorithm should choose a letter to go first. After considering the letters that are immediately before and after it on the board, you can see if there is a word that starts with that letter (or sequence of letters). Only if so, you should choose a second letter to place, etc.  If at any point, there are no words that start with the sequence of letters so far, then you don't need to add more letters, since you know that you won't find a legal move. Similarly, you can consider all the crosswords that your move has formed so far - if any of them are illegal, you can curtail the search. 

If you use our codebase, you do not have access to the internals of the board class, so this may appear to be difficult to implement.  There are a couple ways you could tackle this:

1. You can propose a move of placing one tile, and see what words are formed by that move. If there is a way to continue the word formed so far, then you next try a move of placing two letters, etc. Our Board class returns the formed words as a vector, and always puts the "main" word last in the vector, so you can distinguish the crosswords (which must be correct as is) from the main word (which you may add more letters to).
2. If the first option is too slow for your tastes, then you can use the `getSquare(x,y)` function to determine the tile contents of each square.  You will find you have to re-implement some of the logic from HW4 to make this work, but it should run more quickly.

You should time your AI to see if you implemented this appropriately.  Your AI should be able to find the best move for hands of size about 10 in less than a minute.

If you only implement the exhaustive search without the map-based backtracking (which should work for hands of size up to about 6 or 7), you will receive about half credit on this assignment. If your solution works for hands of size about 10, you will get full credit.

If you want to get more fancy, instead of just a set of prefixes, you could construct a map from prefixes to the set of all words starting with those prefixes. Then, in your search, when a prefix is only completed by a small number of words, you can check whether you have the necessary letters, and terminate the search otherwise. For instance, if you have already started with "pokem", and don't have an 'o' and 'n' in your hand (or in the right position on the board), you don't need to search further: you can save yourself the 26 trials of the next letter to add. But this idea is optional.

###Scrabble Tournament
We intend to run a Scrabble AI tournament for students who want to enter it. Participating in the tournament will not affect your grade. If there are enough participants, we will get prizes for the programmers of the top AIs. (The whole tournament is subject to us being able to set up the infrastructure; if we fail, it might be cancelled.)

Since we will have to have code from multiple students interoperate, the interface for your code will need to be standardized. To achieve this, your Scrabble AI will need to inherit from the class `AbstractAI` we provide in the repo and as the file [AbstractAI.h](AbstractAI.h). Because this class also depends on many other classes (Move, Bag, Tile, Board, Dictionary, ...), if you want to enter the tournament, you will need to use our codebase of .h and .o files.

Since we intend to have each pair of AIs play dozens of games against each other (to correct for luck of the tile draw), your AI needs to be fast if you want to enter the tournament. Finding a good move with a 7-letter hand should not take your AI more than 1/100th of a second most of the time.

Beyond these constraints, feel free to go wild in your AI design. Here are some ideas you may want to consider:

+ build heuristics to evaluate how good certain hands are. You may try to do things such as balance vowels and consonants, or avoid having too many duplicates of the same letter, or value blanks highly, or observe useful combinations (`Q` and `U`, etc.) Your moves may then balance the score earned with the quality of the tiles you will have left.
+ consider the opportunities you are giving your opponent. For instance, making it possible for your opponent to use a triple-word score tends to be dangerous, and you may want to penalize moves that do that.
+ try to plan a few moves ahead. Here, you may want to look into minimax trees for games, which is basically backtracking where you take turns choosing moves that are best for the opposing players. In Scrabble, the difficulty is that you don't know what tiles you will receive next. To deal with this, you could try and simulate a bunch of random draws of the remaining tiles. If you want to really go down this route, our codebase provides you with two functions: `Bag::initialTileCount()` and `Game::initialTileCount()` tell you how many copies of each tile were in the bag initially, and `Board::getTileCount()` returns how many copies of each tile have been placed on the board so far. Between those and your own hand, you can reconstruct what tiles remain in the bag (or the other players' hands).
+ And if you have a few months to spend, you could try to program a neural network that will learn how to play by playing many games itself. Probably not for this assignment, though.

If you decide to enter the Scrabble AI tournament, please place a file called `tournament.txt` in your `hw6` folder. The contents of the file don't matter - we will use the presence or absence as indication of whether to include your code in the tournament.

###Submission Link
You can submit your homework [here](http://bits.usc.edu/codedrop/?course=cs104-fa16&assignment=hw6&auth=Google). Please make sure you have read and understood the [submission instructions]({{ site.url }}/assignments/submission-instructions.html).

{% include commit-reclone.md %}
