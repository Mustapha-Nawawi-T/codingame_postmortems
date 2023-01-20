# postmortem for Fall Challenge 2022

link : https://www.codingame.com/contests/fall-challenge-2022

duration : 25 days

## Result :

Score : 28.15

League : Gold

language : c++

|  Scope  |  Rank   |
| ------- | ------- |
| world   | 84/4577 |
| Morocco |  4/131  |
| school  |  4/104  |

## The Game :
### Statement for the game :

Each player controls a team of robots.

Robots are deployed in a field of abandoned electronics, their purpose is to refurbish patches of this field into functional tech.

The robots are also capable of self-disassembly and self-replication, but they need raw materials from structures called Recyclers which the robots can build.

The structures will recycle everything around them into raw matter, essentially removing the patches of electronics and revealing the Grass below.

Players control a team of these robots in the midst of a playful competition to see which team can control the most patches of a given scrap field. They do so by marking patches with their team's color, all with the following constraints:

- If robots of both teams end up on the same patch, they must disassemble themselves one for one. 
The robots are therefore removed from the game, only leaving at most one team on that patch.
- The robots may not cross the grass, robots that are still on a patch when it is completely recycled must therefore disassemble themselves too.
Once the games are over, the robots will dutifully re-assemble and go back to work as normal.

### How the game looks like :

<img src="https://github.com/Mustapha-Nawawi-T/codingame_postmortems/blob/master/img/Fall_challenge_2022/looks.png" alt="image looks.png">

# The approach :

first of all when I saw the game I thought it was an easy game to simulate because there is no hidden information and everything is given to you in the input.
but soon I realized that the game is not easy to simulate at all, because the new thing for me is that the fact that each player can make multiple action in the same turn, 
and that make the branching factor of the game very high, because the player have at list in the mid game 10 actions to perform .
so to get all possible combinations of 10 actions is : 10! = 3628800, and that is a lot of combinations to simulate.

PS : wi have just 50ms in each turn (1000ms in the first turn);

so first thing Is come to my mind is (genetic algorithm) to solve this problem, probably because its one of th fewest algorithms that i now how to implement it.
but where a minute i am not the only one that want to win the game, what about the enemy, here where i give up the idea and went to heretics approach.

after i reach the point that my code was like a mess, because with all the if and else and the loops and the functions, i was not able to understand what i was doing.
her my one of my colleagues bring this idea to use genetic algorithm with two of population instead of one, that gives the algorithm the chance to develop my moves against best enemy move and vice versa.

so i started to implement this idea, first i need to code the simulation of the game thi game was strayfoward so i didn't take mach time to code it.

after that this pseudo code do all the work for me :
``` cpp
initPopulations();
bool turn = ME; 
while (/*continue while i still have time*/) {
    turn = turn == ME ? ENEMY : ME;
    fitness()
    selection()
    crossover()
    mutation()
    swap(population[turn], newPopulation[turn]);
}
getBestMoves();
```
#### initPopulations() :
this function is used to initialize the two populations, the first population is the population of my moves, and the second population is the population of the enemy moves.
all the moves are generated randomly.

#### turn :
is used to switch between the two population of my moves and the enemy moves.
is get switched each iteration of the while loop.

#### fitness() :
this function its the backbone of the algorithm, it evaluate the population of the player against the best chromosome of the anther player.
so here it comes the part where i need to simulate the game, and here is where i need to use the time wisely, because i have only 50ms in each turn.
so after i simulate the two chromosomes of the two population, i call the getScore() function that evaluate the position for the player (more details in the next section).
a didn't need to undo the moves because i was copying the game state each time.

#### selection() :
first sort the population from grater score to the lowest score, and pass 10% to the next generation.

#### crossover() :
choose two chromosomes from the population given the prayority to ownes with heighest score, and get a random point to crossover the two chromosomes.
acording of this point generate two new chromosomes and pass thim to the next generation.

#### mutation() :
iterate over the new chromosomes and mutate them with a probability of 5%.

#### swap() :
swap the new population with the old population.
and repeat the process until the time is over.

#### note:
for more details about the algorithm you can check this link : https://towardsdatascience.com/introduction-to-genetic-algorithms-including-example-code-e396e98d8bf3

# Score function (the most important part) :

