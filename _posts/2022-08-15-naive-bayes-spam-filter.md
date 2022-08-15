# Building a Spam Filter with Naive Bayes

The full Jupyter Notebook can be found [on GitHub](https://github.com/joshfuchs/DataScience_projects/blob/master/Bayes_Spam_Filter.ipynb).

In this project, we're going to build a spam filter for use on text messages using Naive Bayes. To classify messages as spam or non-spam, the Naive Bayes algorithm follows these steps:

- Learns how humans classify messages.
- Uses that human knowledge to estimate probabilities for new messages — probabilities for spam and non-spam.
- Classifies a new message based on these probability values — if the probability for spam is greater, then it classifies the message as spam. Otherwise, it classifies it as non-spam (if the two probability values are equal, then we may need a human to classify the message).


## Dataset
To teach the algorithm how to classify messages, we'll use a dataset of 5,572 SMS messages that have been classified by humans. 

The dataset was put together by Tiago A. Almeida and José María Gómez Hidalgo, and it can be downloaded from the [The UCI Machine Learning Repository](https://archive.ics.uci.edu/ml/datasets/sms+spam+collection). 

The data is unlabelled, so we'll name the two columns:
- ```Label```: whether the message is spam (```spam```) or not spam (```ham```).
- ```SMS```: the actual text message content

## Goal
Build a multinomial Naive Bayes algorithm to classify SMS messages as spam or non-spam. We want to achieve at least 80% accuracy.

## Training and Testing Set

When loading the dataset, we see that about 86% of the SMS messages are not spam while the remaining 14% are spam. Let's split the data into a training set and a testing set so that at the end we can evaluate the success of our spam filter. We'll use 80% for training and 20% for testing. 

```
# randomize the entire dataset
random_sms = sms_messages.sample(frac=1,random_state=42)
```

```
# Calculate index for split
training_test_index = round(len(random_sms) * 0.8)
```

```
# split into training and testing
training = random_sms[:training_test_index].reset_index(drop=True)
testing = random_sms[training_test_index:].reset_index(drop=True)
```

We see that we have approximately the same percentage of spam and non-spam messages in our training and our testing datasets as in the full dataset. This is exactly what we want.

## Letter Case and Punctuation

Let's begin the data cleaning process by removing the punctuation and bringing all the words to lower case. In the end, the algorithm will look calculate probabilities word-by-word, so we want to standardize the formatting.

```
training['SMS'] = training['SMS'].str.replace('\W', ' ')
training['SMS'] = training['SMS'].str.lower()
```

We also go ahead and do the same thing with the testing set. 

## Creating the Vocabulary

In order to run the algorithm, we need to create a vocabulary of every word that is included in each message. We'll also want to know if words appear multiple times in the same message. To get this information in an easy to use way, let's add a column for every unique word in the messages. The row values will then reflect the number of times that word appears in that message. For example, if the word *computer* does not appear in a message, the value will be 0. But if the word *computer* appears twice in a different message, the value will be 2. 

To do this, let's first split each message into a list, then create a list of all unique words that appear. We'll see that we end up with 7816 unique words.

```
training['SMS'] = training['SMS'].str.split()

vocabulary = []

for message in training['SMS']:
    for word in message:
        vocabulary.append(word)
        
vocabulary = set(vocabulary)

vocabulary = list(vocabulary)
```

## The Final Training Set

Now that we have our vocabulary, we need to use it to populate our dataset for analysis. To do this, we will first build a dictionary that we will then convert to a new DataFrame. The dictionary will include each unique word as keys, and the values will be a list of frequencies by message. 

```
# initialize the dictionary
word_counts_per_sms = {unique_word: [0] * len(training['SMS'])
                      for unique_word in vocabulary}

# loop over training['SMS'] using the enumerate function

for index, sms in enumerate(training['SMS']):
    for word in sms:
        word_counts_per_sms[word][index] += 1
```

To make the final dataframe, we concatenate this dataframe with the ```training``` dataframe.

## Calculating Constants

We now have our training data that we are ready to work with. As a reminder, we will use the following two equations to calculate the probabilities of the messages being spam or not-spam. 

$P(Spam|w_1, w_2, ..., w_n) \propto P(Spam) \cdot \prod_{i=1}^{n} P(w_i | Spam)$

$P(Ham|w_1, w_2, ..., w_n) \propto P(Ham) \cdot \prod_{i=1}^{n} P(w_i | Ham)$

And to calculate $P(w_i | Spam)$ and $P(w_i | Ham)$ we will use

$ P(w_i | Spam) = \frac{N_{w_i | Spam} + \alpha}{N_{Spam} + \alpha \cdot N_{vocabulary}} $

$ P(w_i | Ham) = \frac{N_{w_i | Ham} + \alpha}{N_{Ham} + \alpha \cdot N_{vocabulary}} $

Some of these values will be constant each message. Let's calculate these constants first:

- $P(Spam)$
- $P(Ham)$
- $N_{spam}$ - number of words in all spam messages
- $N_{ham}$ - number of words in all ham messages
- $N_{vocabulary}$

For now, we will let alpha = 1 for Laplace smoothing.

## Calculating Parameters

Now, let's calculate $P(w_i|Spam)$ and $P(w_i|Ham)$ for each word. These values will be constant for the whole training set, so if we calculate these parameters now, we will save computational time in the future. 

```
p_spam_dict = {unique_word: 0
                      for unique_word in vocabulary}
p_ham_dict = {unique_word: 0
                      for unique_word in vocabulary}
```

```
for word in vocabulary:
    
    # calculate number of times the word appears
    # in spam and ham messages
    n_word_given_spam = spam_messages[word].sum()
    n_word_given_ham = ham_messages[word].sum()
    
    
    p_word_given_spam = (n_word_given_spam + alpha) / (n_spam + alpha*n_vocabulary)
    p_word_given_ham = (n_word_given_ham + alpha) / (n_ham + alpha*n_vocabulary)
    
    p_spam_dict[word] = p_word_given_spam
    p_ham_dict[word] = p_word_given_ham
```    

## Classifying a New Message

Now that we have calculated all the constants and parameters, we can create the spam filter. This filter will:

- take in as input a new message
- Calculates P(Spam|w1, w2, ..., wn) and P(Ham|w1, w2, ..., wn)
- Compares the values of P(Spam|w1, w2, ..., wn) and P(Ham|w1, w2, ..., wn), and:
    - If P(Ham|w1, w2, ..., wn) > P(Spam|w1, w2, ..., wn), then the message is classified as ham.
    - If P(Ham|w1, w2, ..., wn) < P(Spam|w1, w2, ..., wn), then the message is classified as spam.
    - If P(Ham|w1, w2, ..., wn) = P(Spam|w1, w2, ..., wn), then the algorithm may request human help.

```
def classify(message):

    message = re.sub('\W', ' ', message)
    message = message.lower()
    message = message.split()

    
    p_spam_given_message = p_spam
    p_ham_given_message = p_ham
    
    for word in message:
        if word in p_spam_dict:
            p_spam_given_message *= p_spam_dict[word]
            
        if word in p_ham_dict:
            p_ham_given_message *= p_ham_dict[word]

    print('P(Spam|message):', p_spam_given_message)
    print('P(Ham|message):', p_ham_given_message)

    if p_ham_given_message > p_spam_given_message:
        print('Label: Ham')
    elif p_ham_given_message < p_spam_given_message:
        print('Label: Spam')
    else:
        print('Equal proabilities, have a human classify this!')
```

We can test a few messages and find that *secret money prince code* is most likely a Spam message, while *Sounds good, see you then* is mostly likely not a spam message. 

## Measuring the Spam Filter's Accuracy

Now that we have created our spam filter, let's test it on our testing set of 1,114 messages. We will modify the ```classify``` function we wrote last time. 

```
def classify_test_set(message):

    message = re.sub('\W', ' ', message)
    message = message.lower()
    message = message.split()

    p_spam_given_message = p_spam
    p_ham_given_message = p_ham

    for word in message:
        if word in p_spam_dict:
            p_spam_given_message *= p_spam_dict[word]

        if word in p_ham_dict:
            p_ham_given_message *= p_spam_dict[word]

    if p_ham_given_message > p_spam_given_message:
        return 'ham'
    elif p_spam_given_message > p_ham_given_message:
        return 'spam'
    else:
        return 'needs human classification'
```

When we apply this to our ```testing``` dataframe, we see that our filter correctly classified 85% of messages. While this meets our goal, mislabeling 15% of messages is too high for deploying into a production environment. If we wanted to improve this filter, we could consider the following adjustments:

- Changing the alpha value
- Expanding the training dataset. Approximately 18.7 billion SMS messages are sent each day. Accessing them is difficult, but a larger training dataset could better encompass the probability set.
- We made every word lowercase. We could consider accounting for letter case differences. 


