<h1 align="center">
  Ultimate Tic-Tac-Toe AI
</h1>

<p align="center">
  A fully functional Ultimate Tic-Tac-Toe game including AI agents utilizing adverserial searching algorithms and a PyGame GUI.
  <br>
  This project was developed in collaboration with <a href="https://github.com/ilina-d">zhap</a> 🙂
</p>

<p align="center">
  <a href="#how-to-play">How to Play</a> •
  <a href="#usage">Usage</a> •
  <a href="#implemented-algorithms">Implemented Algorithms</a> •
  <a href="#heuristic-evaluation">Heuristic Evaluation</a> •
  <a href="#results-notes-fun-facts">Results, Notes, Fun Facts</a>
</p>

<p align="center"><img src="https://github.com/user-attachments/assets/87a477ab-7b0f-4e8b-96a7-f4689ba843c8"></p>

<br>

## How to Play
Ultimate Tic-Tac-Toe (UTTT) is a more complex version of the classic Tic-Tac-Toe game where instead of a single **3x3** board, each space in the main board contains another smaller **3x3** board.

Play starts with the first player placing an **X** in any of the 81 available spaces, after which player two places an **O** in the small board relative to the previous move. Players take turns placing **X** or **O**, with each move limiting the other player's legal moves to the corresponding small board.
> **Example:** If player one places an **X** on the middle board, on the bottom-right square; then player two must place an **O** in any of the unoccupied spaces of the bottom-right board.

If a player wins a small board (makes 3-in-a-row), that same board takes on their sign (**X** or **O**). If a small board is completed with a tie, that board belongs to no one. The game ends in a **win** for the player that makes 3-in-a-row on the big board or in a **draw** when all squares are occupied.

**Free Moves:** If one player's move limits the other player's options to an already completed board (or a board with no free spaces), that player gets a free move and may place their sign on any uncaptured space. Play continues normally afterwards.

<br>

## Usage
### Installation
```bash
# Clone the repository
git clone https://github.com/zerofyy/Ultimate-Tic-Tac-Toe-AI.git
cd Ultimate-Tic-Tac-Toe-AI

# Install dependencies
pip install -r requirements.txt

# Run
python main.py
```

### Configuration
```py
if __name__ == '__main__':
    # Instance the game evaluator (required if use_eval_bar = True)
    # Currently the GameEvaluator only works with MiniMax, best leave it as is
    GameEvaluator(
        algorithm = MiniMaxPlayer(target_depth = 5, use_randomness = True)
    )

    # Choose player agents and configure the game
    # Check out the GameUI class docstring for more information.
    game = GameUI(
        UserPlayer(),  # Player 1 (X)
        MiniMaxPlayer(target_depth = 5),  # Player 2 (O)
        printing = False,
        wait_after_move = None,
        use_eval_bar = True
    )

    # Start the game
    game.play()

    # Prevent the program from closing by waiting for input
    input('... waiting for input ...')
```

<br>

## Implemented Algorithms
| Algorithm               | Description                                                                                | Implemented? |
|-------------------------|--------------------------------------------------------------------------------------------|--------------|
| MiniMax                 | Makes moves that prevent worst-case outcomes, assumming that the opponent plays perfectly. | ✅          |
| ExpectiMax              | Makes moves with the highest expected outcome, assumming that the opponent plays randomly. | ✅          |
| Monte Carlo Tree Search | Runs many random simulations and picks moves that perform the best on average.             | ❌          |
| Random Player           | Makes completely random moves.                                                             | ✅          |

<br>

## Heuristic Evaluation
The AI players decide which moves to make with the help of a heuristic function that evaluates a given state by calculating a **reward score** within range of `[-1000, +1000]` for each of the small boards, taking into consideration the following formations:

1. `SCORE_WIN` If the small board is won, return score of **1000**. No need to check other formations in this case.
2. `SCORE_TIE` If the small board is completed and isn't won by either player, return score of **0**. No need to check other formations in this case.
3. `SCORE_CENTER` Moves at the center of the small board are worth **40** because the center space has the highest potential for 3-in-a-row.
4. `SCORE_CORNER` Moves in the corners of the small board are worth **16** because they have the potential for creating forks.
5. `SCORE_TWO_IN_ROW` Formations where 2/3 spaces have identical signs (**X** or **O**) in a potentially winning line are worth **56**.
6. `SCORE_FORK` Formations where 3 spaces have identical signs (**X** or **O**) such that they create 2 or 3 potentially winning lines are worth **80**. Forks are usually made by occupying 3 corners or 2 corners and the middle.
7. `SCORE_BLOCKING` Moves that block the opponent's two-in-a-row formations are worth **40**.

The scores for each of the formations are taken arbitrarily, making sure that there is no combination of moves that is scored equal to or greater than the score for winning a board.

The evaluation for a small board is done for both signs such that for each formation **X** adds to the score and **O** subtracts from it.

Finally, each of the small boards' scores are scaled depending on where they are on the big board: The middle board contributes 20% to the final score, the corner boards contribute 15% each, and the rest contribute 5% each. This is done because it is difficult to plan out moves on the big board (such as forks, two-in-a-row, and blocking moves) and the scaling helps prioritize favorable positions without needlessly evaluating the big board.

> We also tried adding a penalty worth **100** for moves that allow the opponent to play anywhere on the board, but this actually led to worse performance overall, so we removed it.

The code for the heuristic evaluation can be found in [this file](utils/helpers/state_evaluator.py).

<br>

## Results, Optimizations, Fun Facts
### Results
As of January 2025 the MiniMax agent (with dynamic depth, explained below) was tested against 9331 other bots at [codingame](https://www.codingame.com/multiplayer/bot-programming/tic-tac-toe) and ranked among the top 6.6%. [This script](codingames.py) contains the compiled code for running the agent on the codingame platform.

### Optimizations
During development we noticed the algorithms were very slow when making decisions. Through profiling we identified the bottlenecks in the code and implemented several optimizations:

1. **Alpha-Beta Pruning:** Implementing this for the MiniMax algorithm, we expected a drastic increase in efficiency, but in reality it didn't contribute as much as the other optimizations.
3. **Limited Depth:** This is an obvious optimization for both MiniMax and ExpectiMax; just lower the algorithm's foresight to something like 5 moves into the future. However, this is a compromise between speed and performance which we weren't too willing to make.
4. **Dynamic Depth:** This works for both MiniMax and ExpectiMax. The idea is that as the game goes on, there are less and less available spaces and thus less possible futures the algorithm has to take into consideration. To take advantage of this, the algorithm starts by looking 5 moves into the future and looks deeper and deeper as the game goes on (depth increases inversely proportional to the breadth of the search tree).
5. **Predefined Moves:** During testing and researching, we noticed that several existing algorithms showcase similar behavior in certain situations. Examples of this are starting the game by playing in the exact middle of the board and when sent to an empty board, keep the opponent on that same board. These moves are predefined to save on computation time.
6. **Evaluation Caching:** Beyond optimizing the algorithms themselves, the implementation of caching when evaluating small boards led to a speedup of over 90%.

### Fun Fact
When checking if a given board is won, an obvious solution is to check for the same sign (**X** or **O**) in every row, column, and diagonal. However, **did you know** that this can also be done in a much cooler way while not gaining or losing on performance? 😎 Through the use of a magic square and its properties\* we can check for a winning combination by summing up all combinations of 3 occupied spaces for a given sign.

\* A magic square is an **NxN** square containing numbers in each space such that the sum of each row, column, and diagonal are equal!
