# Design Notes

## Crosstabs
The cross tab data strucuture is a very important data structure in the microlab engine as it is intended to be used to model an experiment design and the collation of experimental results.

There is some inspiration taken from the Tensor paradigm from pyTorch and Tensorflow libraries, but they aim is to represent large numbers of parameters found in neural networks. The cross tab seeks to represent the small datasets found in scientific experiments. 

The core similarity is that aim to model a multi-dimensional array of data, which is the heart of designing and execting an experiment.

Lets try and build up some intution about the crosstab:

### Coin Tossing - Part 1

Consider the very basic scenario of _tossing a coin_ - where we aim to see if a coin toss is fair. We may want to follow an experimental procedure that looks something like:

1. Experiment Design
    1. Select a coin to test
    2. Choose the number of coin tosses to perform to test accurately
2. Run the experiment and collect the data
3. Analyise the results to see if the coin is fair

#### Experiment Design

To record this experiment, we use a table and prior to the expriment we predefine the things we have controls over. These are the _Independent_ variables that we control. With Coin "C1" and 100 tosses we have an experiment design that looks like:

| Coin        | Trial       |
| ----------- | ----------: |
| C1 | 1 |
| C1 | 2 |
| C1 | ... |
| C1 | 99 |
| C1 | 100 |

#### Collect Results

Running the experiment and recording the results is Step 2 of our procedure and this adds a new column to the table:

| Coin        | Trial       | Result      |
| ----------- | ----------: | ----------: |
| C1 | 1 | H |
| C1 | 2 | T |
| C1 | ... | ... |
| C1 | 99 | T |
| C1 | 100 | T |


> In reality there may be reason why a trial fails or data cannot be collected fairly, so the data table has to be permissive to the fac that _Dependent_ data that we do not control completly may be missing.

#### Results Analysis

Then we can move onto Step 3 and the analysis of data. Here we wish to find the ratio of heads to tails in the data set with the hypothisis that a fair coin will have a ration of close to _1_.

Computationally, we aiming to count 4 sums, The number of 'H's the number of 'T's, the number of 'N' and the Total number of trials. Then using replacing the trial column with the the count and the percentage of results. So that it looks something like:

| Coin  | Result    | Count | Percentage of Total   |
|---	|---        |---:	|---:                   |
|C1     | H         | 50    | 50%  	|
|C1     | T     	| 48  	| 48%  	|
|C1     | Null      | 2  	| 2%  	|

There are already a number of intresting data structure features that need to be taken into account to go through the most simple of experiments.

We have introduced:
* The concepts of independent and dependents variables i.e the variables that you can control and the variables that you collect.
* Creating an initial design of the data with the independent variables and the number of trials that will be run.
* The table has mutated in shape accross the experiment. First we added extra columns. Then aggregated that data reducing the number of rows, eliminating a column and adding two new ones.
* Application of mathmathical procedures that aggregate that data finding the count and percentage of number of items in the dataset.
* Handleing of Null values that comes from a trial failing so that there is missing data that was not defined in the inital plan.

### Coin Tossing - Part 2
We can build on this by expanding the questions from "Is this coin fair?" to "Which of these three coins is the fairest?" 

Following a similar process, we start with an experimental procedure: 

1. Experiment Design
    1. Select 3 coins to test
    2. Choose the number of coin tosses to perform on each coin to test fairness
2. Run the experiment and collect the data
3. Analyise the results to see if the coin is fair

How does this change the data structure:

#### Experiment Design
Our experimental design grid remains the same, but now we have three values in the coin column each with an assocaited 100 trials.

| Coin        | Trial       |
| ----------- | ----------: |
| C1 | 1 |
| C1 | 2 |
| C1 | ... |
| C1 | 99 |
| C1 | 100 |
| C2 | 1 |
| C2 | 2 |
| C2 | ... |
| C2 | 99 |
| C2 | 100 |
| C3 | 1 |
| C3 | 2 |
| C3 | ... |
| C3 | 99 |
| C3 | 100 |

#### Collect Results

The results collections follow the design grid, and now we collect data for all three coins. 

| Coin        | Trial       | Result      |
| ----------- | ----------: | ----------: |
| C1 | 1 | H |
| C1 | 2 | T |
| C1 | ... | ... |
| C1 | 99 | T |
| C1 | 100 | T |
| C2 | 1 | H |
| C2 | 2 | T |
| C2 | ... | ... |
| C2 | 99 | T |
| C2 | 100 | T |
| C3 | 1 | H |
| C3 | 2 | T |
| C3 | ... | ... |
| C3 | 99 | T |
| C3 | 100 | T |


#### Results Analysis

Finally our results table has to compute the counts and percentages for each variable in the Coin column, leading to a table like this:

| Coin  | Result    | Count | Percentage of Total   |
|---	|---        |---:	|---:                   |
| C1    | H         | 50    | 50%                   |
| C1    | T         | 48  	| 48%                   |
| C1    | Null      | 2     | 2%                    |
| C2    | H         | 76    | 76%                   |
| C2    | T         | 24  	| 24%                   |
| C3    | H         | 47    | 47%                   |
| C3    | T         | 48  	| 48%                   |
| C3    | Null      | 5     | 5%                    |

But is this the best view to analyse the data. Probably not. We can go better and think about trying to reduce this data down to a single statistic Ratio H/T that represents fairness. We also keep track of the success rate to evalute if there is a worrying amount of missing data. 

| Coin  | Success Rate      | Ratio H/T  |
|---	|---                |---:	     |
| C1    | 98%               | 1.04       |
| C2    | 100%              | 3.16  	 |
| C3    | 95%               | 0.97       |


This scenario has further built out the intuition of CrossTab structure. What have we observed: 

* The coin column went from single valued to multi valued
* Initally this acted as a multiplier as we repeated the trials for each of the new coins. This followe through the results collection and the analysis section. Although it is import that the analysis functions ran over the coin variables as a group. They could have ignored the group and run over the entire results.
* There was a better way to perform the multi-valued analysis that enabled better intepretability of the results, but required a more bespoke formula. 
* For the engine, we need to decide on practicality of understanding patterns an not trying to build a full statistical engine.
* There is the option of bringing the Percentage dimension into column so it looks like the following. 

| Coin  | H %   | T %   | Null %    |
|---	|---:   |---:   |---:       |
| C1    | 50    | 48    | 2         |
| C1    | 76    | 24    | 0         |
| C1    | 47    | 48    | 5         |

Finally lets consider one further extension

### Coin Tossing - Part 3

Were going to build this one step further and by asking the quesion: "What if the coin or the thrower is the source of unfairness?" And expand analysis to look at a joint analysis of thrower and coin. 

#### Experiment Design

So first of in our experiment design, we add a new independent variable of Thrower, this is represented as a new column in the cross tab. It has two values in this experiment T1 and T2:

As previously this multiplies number of rows in the table so now we're looking at Thrower # x Coin # x Trial # is a cross tab size.

|Thrower    | Coin        | Trial       |
|---        | ----------- | ----------: |
| T1 | C1 | 1 |
| T1 | C1 | ... |
| T1 | C1 | 100 |
| T1 | C2 | 1 |
| T1 | C2 | ... |
| T1 | C2 | 100 |
| T2 | C1 | 1 |
| T2 | C1 | ... |
| T2 | C1 | 100 |
| T2 | C2 | 1 |
| T2 | C2 | ... |
| T2 | C2 | 100 |

#### Collect Results

This propogates though to into the raw results so that we fill in the results dimension for each of the resuls 

|Thrower    | Coin        | Trial       | Result |
|---        | ----------- | ----------: |---|
| T1 | C1 | 1 | T |  
| T1 | C1 | ... | ... |
| T1 | C1 | 100 | T |
| T1 | C2 | 1 | T |
| T1 | C2 | ... | ... |
| T1 | C2 | 100 | H |
| T2 | C1 | 1 | T |
| T2 | C1 | ... | ... |
| T2 | C1 | 100 | T |
| T2 | C2 | 1 | T |
| T2 | C2 | ... | ... |
| T2 | C2 | 100 | H |


#### Results Analysis

Finally we need to consider how we present the results. We saw previously we can start consider how we stack the dimensions in the view.

| Thrower   | Coin  | H %   | T %   | Null %    |
|---	|---	|---:   |---:   |---:       |
| T1    | C1    | 50    | 48    | 2         |
| T1    | C2    | 76    | 24    | 0         |
| T2    | C1    | 50    | 48    | 2         |
| T2    | C2    | 76    | 24    | 0         |

There are a large number of artifacts in the data here.

### Reflections
The cross tab dataset needs to reflect all of this