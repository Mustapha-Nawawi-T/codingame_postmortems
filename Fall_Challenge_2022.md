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
In this game, players control teams of robots in a field of abandoned electronics. The robots' goal is to refurbish the field by using raw materials from structures called Recyclers, which they can build. The Recyclers convert everything around them into raw matter, revealing the Grass below. The competition is to see which team can control the most patches of the field by marking them with their team's color. However, there are constraints: if robots from both teams are on the same patch, they must disassemble themselves one for one, and robots may not cross the grass. If a robot is still on a patch when it is completely recycled, it must also disassemble itself.

### How the game looks like :
<img src="https://github.com/Mustapha-Nawawi-T/codingame_postmortems/blob/master/img/Fall_challenge_2022/looks.png" alt="image looks.png">

# The approach :
When I first saw the game, I thought it would be easy to simulate because all the information is provided in the input. However, I soon realized that it is not easy to simulate at all because each player can make multiple actions in a single turn, which increases the number of possible combinations of actions to simulate. I calculated that getting all possible combinations of 10 actions is 10! = 3628800, which is a lot to simulate.

I only have 50ms per turn (1000ms in the first turn) to simulate these combinations. At first, I thought of using a genetic algorithm to solve this problem because it is one of the few algorithms I know how to implement. But then I realized that the enemy is also trying to win the game, and so I abandoned that idea and went for a different approach.

Eventually, my code became a mess because of all the if-else statements, loops, and functions. I couldn't understand what I was doing anymore. Then, one of my colleagues suggested using a genetic algorithm with two populations instead of one. This would allow the algorithm to develop my moves against the enemy's best moves, and vice versa.

I started to implement this idea and first had to code the simulation of the game. The game was straightforward so it didn't take much time to code.

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
The function "initPopulations()" is used to initialize two populations. The first population is a population of the player's moves and the second population is a population of the enemy's moves. All the moves in both populations are generated randomly. This function sets the initial starting point for the genetic algorithm by creating a pool of possible moves for the player and the enemy to select from.

#### turn :
The "turn" variable is used to switch between the two populations of moves, the player's moves and the enemy's moves. It is used in the iteration of the while loop, so each iteration the "turn" variable switches between the two populations, allowing for the genetic algorithm to compare and evolve the moves of both the player and the enemy in each iteration. This allows the algorithm to select the best moves for both the player and the enemy, so it can develop a strategy that takes into account both players' actions.

#### fitness() :
The "fitness()" function is the backbone of the genetic algorithm. It evaluates the population of the player against the best chromosome of the other player (the enemy). This is where the game simulation takes place and it's crucial to use the time wisely as there is only 50ms for each turn.

In this function, the game simulation is done for the two chromosomes of the two populations. Then, the "getScore()" function is called to evaluate the position of the player. The getScore() function will be used to evaluate the player's position, and it will give more information in the next section.
It is not necessary to undo the moves because the game state is copied each time before the simulation starts.

The fitness function serves as the evaluation function that assigns a fitness value to each chromosome in the population, indicating its relative fitness. The better the chromosome, the higher the fitness value. The fitness function compares the player's moves to the enemy's best move, and assigns a score based on the outcome of the game simulation.

#### selection() :
The "selection()" function is used to select the best chromosomes from the current population to be carried forward to the next generation. It works by sorting the population by their fitness scores, with the highest score at the top and the lowest score at the bottom. Then, the top 10% of the population is selected to move on to the next generation. This process is called "elitist selection" because it ensures that the best chromosomes are passed on to the next generation, thus increasing the chances of producing better solutions in the next iteration.

#### crossover() :
The "crossover()" function is used to generate new chromosomes by combining the genetic information of two existing chromosomes. It works by choosing two chromosomes from the population, giving priority to the ones with the highest scores. Then, a random point is chosen to crossover the two chromosomes. The genetic information on the left side of this point is taken from one chromosome and the genetic information on the right side of this point is taken from the other chromosome.

This creates two new chromosomes, which are then passed on to the next generation. The purpose of crossover is to combine the best traits of the parent chromosomes to create offspring that are potentially better than either parent. This is an important step in the genetic algorithm as it allows for the exploration of new combinations of moves and strategies .

#### mutation() :
The "mutation()" function is used to introduce random changes to the chromosomes in the population. It works by iterating over the new chromosomes and applying a random mutation to a small percentage of the genes. The probability of mutation is usually set at a low value, such as 5%, to ensure that the majority of the genetic information remains unchanged but also to introduce new genetic variations. The purpose of mutation is to introduce new genetic variations that can potentially lead to better solutions, preventing the algorithm from getting stuck in a local optimum.

#### swap() :
The "swap()" function is used to replace the old population with the new population, discarding the old chromosomes and replacing them with the new ones that were generated through the genetic algorithm's operations (selection, crossover, mutation). This is done to prepare for the next iteration of the algorithm, where the new population will be evaluated, undergo selection, crossover, and mutation again.

The process of the genetic algorithm is repeated until a stopping criterion is met, such as reaching a maximum number of iterations or a certain level of fitness. The ultimate goal is to find the best solution, or the best set of moves, that maximizes the player's chances of winning the game.

# Score function (the most important part) :
The score function is an important part of the algorithm because it determines the outcome of the game. The goal of each player is to control the maximum area, as the player with the most controlled area at the end of the game wins.

In this implementation, the score function uses a Voronoi diagram to calculate the maximum area controlled. It also takes into account the distance to the line connecting the players' areas to solve the problem of robots that are behind the line not contributing to the Voronoi diagram score.

Players try to control the maximum area but also expand in a straight line towards the enemy's area. They start expanding when they reach the enemy's area but it's also good to expand a little in the start of the game, which gives more options to build or spawn robots.

The score function also takes into account the number of cells controlled by the player. This encourages the robots to take more cells when they are on the way to the enemy's area and makes the fight more interesting. This score also prevents losing the player's cells.

The final score is calculated by combining three factors: the voronoi score, the distance score, and the control score, each with a different weight. The voronoi score is the most important factor, followed by the control score, and then the distance score. These weights are set as follows:

- VORONOI_SCORE_WEIGHT = 1000000.0
- DISTANCE_SCORE_WEIGHT = 1.0
- CONTROL_SCORE_WEIGHT = 2.0

The final score is calculated by taking the product of each of these factors and their corresponding weights, and then summing them up.

``` cpp
return voronoiScore  * VORONOI_SCORE_WEIGHT 
         + distanceScore * DISTANCE_SCORE_WEIGHT 
         + controlScore  * CONTROL_SCORE_WEIGHT;
```

This score function allows the algorithm to evaluate the performance of each chromosome and determine the best moves for the player to make. The combination of these factors gives the player the best chance of winning the game by maximizing the area controlled, while also taking into account the distance to the enemy's area and the number of cells controlled by the player.

# Space for the next contests :
- Search and research extensively before starting to code. This would allow for a better understanding of the problem and the available solutions, leading to a more efficient and effective implementation.
- Use the time wisely. With the limited time available for each turn, it's important to prioritize the most important tasks and optimize the code for speed.
- Create a playground with all the tools that help you to debug your code. This could include test cases, visualization tools, and debuggers, which can make it easier to identify and fix bugs in the code.
- Use Rust. because why not ?