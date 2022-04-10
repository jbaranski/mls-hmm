# Hidden Markov Model for predicting MLS results
This repository contains a Jupyter notebook for predicting the outcome of MLS match fixtures using a Hidden Markov Model.

## States
The hidden states of the model is the result of a game: win, loss, tie.

## Observations
The observations of the model is where a game was played: home, away.

## Input data
The input csv contains the following columns:
- `State` - Win, loss, tie.
- `Opponent` - If the opponent starts with `@` character, it's an away game, otherwise home game.
- `Score` - The score of the game (not currently used in the model).
- `Season` - The season the game was played (not currently used in the model).

The dataset only goes back to the 2021 season to keep the results relevant (since player transfers, coaching changes, etc... affect results).

## Example
DC United entire 2021 season, and 2022 season up to 04/10/2022, 40 games total in the data set. The result data was obtained from https://www.mlssoccer.com/standings/form-guide/.

### States
The states below represent the counts of 39 transitions from 40 games. The last game doesn't have a transition yet so not included in the states count (this missing transition is what we're going to predict).

`W` = Win, `L` = Loss, `T` = Tie

```
W      W            5
       L            8
       T            3
L      W            7
       L            8
       T            2
T      W            3
       L            2
       T            0
```
### Transition matrix
The probabilities of transitioning from one hidden state to another.

`W` = Win, `L` = Loss, `T` = Tie

```
       W             L             T
 W     0.3125        0.5           0.1875
 L     0.41176471    0.47058824    0.11764706
 T     0.6           0.4           0
```
### Observations
The states below represent the counts of 40 observations from 40 games.

`W` = Win, `L` = Loss, `T` = Tie

`H` = Home, `A` = Away

```
W      A               4
       H              12
L      A              11
       H               7
T      A               4
       H               1
```
### Observation transition matrix
The probabilities of a hidden state based on the observation

`W` = Win, `L` = Loss, `T` = Tie

`H` = Home, `A` = Away

```
       H             A
 W     0.75          0.25
 L     0.38888889    0.61111111
 T     0.2           0.0
```

### Markov Chain Visualization
Here is a Markov Chain visualization (based on the given hidden state transition matrix and observation transition matrix) of DC United's historical results.

![DCUNITED-MARKOV-CHAIN](https://user-images.githubusercontent.com/10889950/162631461-3a734d2a-c1e4-4b74-824c-767527cc2b99.png)


### Prediction probabilities
We input a single item sequence (whether the next game is home or away). The output is the posterior probability for each hidden state in the model.

Here are the following posterior probabilities for each hidden state based on the fact that DC United's next game is at home:

`W` = Win, `L` = Loss, `T` = Tie

```
W               L               T
0.56016598      0.29045643      0.14937759
```

This can be interpreted as a `56%` probability of winning, `29%` chance of losing, and `14%` chance of tieing.

Now we repeat this process for DC United's opponent, Austin FC (they have their own hidden state transition matrix, observation transition matrix, etc... based on their input csv data). Here are the following posterior probabilities for each hidden state based on the fact that Austin FC's next game is NOT at home:

`W` = Win, `L` = Loss, `T` = Tie

```
W               L               T
0.15050167      0.36789298      0.48160535
```

To finalize the prediction, I take the max of the following:
1. Sum home team `W` posterior probability with away team `L` posterior probability, divide by 2
```
= (.56016598 + .36789298) / 2
= 0.46402948
```
2. Sum away team `W` posterior probability with home team `L` posterior probability, divide by 2
```
= (.15050167 + .29045643) / 2
= 0.22047905
```
3. Sum home and away team `T` posterior probability, divide by 2
```
= (0.14937759 + .48160535) / 2
= 0.31549147
```

So as you can see, there is a 46% chance DC United will win their next home game, a 22% chance DC United lose their next home game, and a 31% chance DC United tie their next home game.

### Prediction results on 4/10/2022
The model heavily favors home teams, and I need to tweak it with more observations to make it more reliable. With that said, it was a pretty good predictor of results for this round of games:

```
Game                          Prediction            Actual Result      Correct
Orlando vs. Chicago           Orlando W 41%         Orlando W          Yes
Miami vs. New England         Miami W 37%           Miami W            Yes
LA Galaxy vs. LAFC            LA Galaxy W 40%       LA Galaxy W        Yes
Kansas City vs. Nashville     Kansas City W 46%     Kansas City L      No
Charlotte vs. Atlanta         Charlotte W 59%       Charlotte W        Yes
Austin FC vs. Minnesota       Austin W 48%          Austin
```
