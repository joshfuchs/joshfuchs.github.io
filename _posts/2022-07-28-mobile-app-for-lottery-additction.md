# Mobile App for Lottery Addiction

You can find the full Jupter notebook on [GitHub](https://github.com/joshfuchs/DataScience_projects/blob/master/mobile-app-lottery-addiction.ipynb). This is a fairly simple project writing some probability functions. 


Many people start playing the lottery for fun, but for some this activity turns into a habit which eventually escalates into addiction. Like other compulsive gamblers, lottery addicts soon begin spending from their savings and loans, they start to accumulate debts, and eventually engage in desperate behaviors like theft.

A medical institute that aims to prevent and treat gambling addictions wants to build a dedicated mobile app to help lottery addicts better estimate their chances of winning. The institute has a team of engineers that will build the app, but they need us to create the logical core of the app and calculate probabilities.

For the first version of the app, they want us to focus on the [6/49 lottery](https://en.wikipedia.org/wiki/Lotto_6/49) and build functions that enable users to answer questions like:

- What is the probability of winning the big prize with a single ticket?
- What is the probability of winning the big prize if we play 40 different tickets (or any other number)?
- What is the probability of having at least five (or four, or three, or two) winning numbers on a single ticket?

The institute also wants us to consider historical data coming from the national 6/49 lottery game in Canada. The [data set](https://www.kaggle.com/datasets/datascienceai/lottery-dataset) has data for 3,665 drawings, dating from 1982 to 2018 (we'll come back to this).

## Goal
Write a few functions that will enable users to answer probability questions about playing the lottery. 

## Core Functions

We will frequently use these functions to calculate factorials and combinations. Let's start by defining these functions.

```
def factorial(n):
    result = 1
    for x in range(n):
        result *= (n-x)
    return result
```

```
def combinations(n,k):
    return (factorial(n) / (factorial(k) * factorial(n-k)) )
```

## One-ticket Probability

Next, let's calculate the probability of winning the big prize. 

In the 6/49 lottery, six numbers are drawn from a set of 49 numbers that range from 1 to 49. A player wins the big prize if the six numbers on their tickets match all the six numbers drawn. If a player has a ticket with the numbers {13, 22, 24, 27, 42, 44}, he only wins the big prize if the numbers drawn are {13, 22, 24, 27, 42, 44}. If only one number differs, he doesn't win.

For the first version of the app, we want players to be able to calculate the probability of winning the big prize with the various numbers they play on a single ticket (for each ticket a player chooses six numbers out of 49). So, we'll start by building a function that calculates the probability of winning the big prize for any given ticket.

We discussed with the engineering team of the medical institute, and they told us we need to be aware of the following details when we write the function:

- Inside the app, the user inputs six different numbers from 1 to 49.
- Under the hood, the six numbers will come as a Python list, which will serve as the single input to our function.
- The engineering team wants the function to print the probability value in a friendly way — in a way that people without any probability training are able to understand.

```
def one_ticket_probability(x):
   total_outcomes = combinations(49,6) # 49 choose 6
    probability = 1 / total_outcomes
    percentage = probability * 100
    print('''Your chances to win the big prize with the numbers {} are {:.7f}%.
In other words, you have a 1 in {:,} chance of winning.'''.format(x,
                    percentage, int(total_outcomes)))    
```    

## Historical Data Check for Canada Lottery
Our ```one_ticket_probability``` function tells users  the probability of winning the big prize with a single ticket. However, we also want users to be able to compare their ticket against the historical lottery data in Canada and determine whether they would have ever won by now. 

Let's explore the historical data from the Canada 6/49 lottery. The data set contains historical data for 3,665 drawings (each row shows data for a single drawing), dating from 1982 to 2018. For each drawing, we can find the six numbers drawn in the following six columns:

- ```NUMBER DRAWN 1```
- ```NUMBER DRAWN 2```
- ```NUMBER DRAWN 3```
- ```NUMBER DRAWN 4```
- ```NUMBER DRAWN 5```
- ```NUMBER DRAWN 6```
 
## Function for Historical Data Check
Let's write a function that will enable users to compare their tickets agaisnt the historical lottery data in Canada and determine whether they would have ever won by now.

The engineering team told us that we need to be aware of the following details:

- Inside the app, the user inputs six different numbers from 1 to 49.
- Under the hood, the six numbers will come as a Python list and serve as an input to our function.
- The engineering team wants us to write a function that prints:
    - the number of times the combination selected occurred in the Canada data set; and
    - the probability of winning the big prize in the next drawing with that combination.

```
def extract_numbers(row):
    row = row[4:10]
    row = set(row.values)
    return row
```    
```
winning_numbers = lottery.apply(extract_numbers, axis=1)
```    

```
def check_historical_occurent(user_numbers,winning_numbers):
    user_numbers = set(user_numbers)
    comparison = user_numbers == winning_numbers
    number_of_winning_times = comparison.sum()
    if number_of_winning_times == 0:
        print('''If you played the numbers {}, historically, you would have won {} times.'''.format(user_numbers,number_of_winning_times))
    elif number_of_winning_times == 1:
        print('''If you played the numbers {}, historically, you would have won {} time.'''.format(user_numbers,number_of_winning_times))
    else:
        print('''If you played the numbers {}, historically, you would have won {} times.'''.format(user_numbers,number_of_winning_times))
    print('')
    print('''If you played these numbers now, you would have a 0.0000072% chance of winning. In other words, you have a 1 in 13,983,816 chance of winning. ''')
```

## Multi-ticket Probability
Lottery addicts usually play more than one ticket on a single drawing, thinking that this might increase their chances of winning significantly. Our purpose is to help them better estimate their chances of winning. So let's now write a function that will allow users to calculate the chances of winning for any number of different tickets. 

We've talked with the engineering team and they gave us the following information:

- The user will input the number of different tickets they want to play (without inputting the specific combinations they intend to play).
- Our function will see an integer between 1 and 13,983,816 (the maximum number of different tickets).
- The function should print information about the probability of winning the big prize depending on the number of different tickets played.

```
def multi_ticket_probability(number_of_tickets):
    total_number_of_outcomes = combinations(49,6)
    probability_of_winning = number_of_tickets / total_number_of_outcomes
    percentage = probability_of_winning * 100 
    print('''Your chances to win the big prize by playing {} tickets are {:.7f}%.'''.format(number_of_tickets,
                    percentage ))
```

## Less Winning Numbers
To wrap up, let's write a function to allow users to calculate the probabilities for two, three, four, or five winning numbers. 

For extra context, in most 6/49 lotteries there are smaller prizes if a player's ticket match two, three, four, or five of the six numbers drawn. As a consequence, the users might be interested in knowing the probability of having two, three, four, or five winning numbers.

These are the engineering details we'll need to be aware of. Inside the app, the user inputs:

- six different numbers from 1 to 49; and
- an integer between 2 and 5 that represents the number of winning numbers expected
- Our function prints information about the probability of having the inputted number of winning numbers.

```
def probability_less_6(x):
    n_combinations_ticket = combinations(6,x)
    n_combinations_remaining = combinations(43, 6 - x)
    successful_outcomes = n_combinations_ticket * n_combinations_remaining
    
    total_outcomes = combinations(49,6)
    probability = successful_outcomes / total_outcomes
    percentage = probability * 100
    
    combinations_simplified = round(total_outcomes/successful_outcomes)    
    print('''Your chances of having {} winning numbers with this ticket are {:.6f}%.
In other words, you have a 1 in {:,} chances to win.'''.format(x, percentage,
                                                               int(combinations_simplified)))
```    

