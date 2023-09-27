# 1. Observing the Impact of Decoding Strategy

## 1.1 Rolling a Twenty-Sided Die

**Analysis Questions**
1. **When using full random sampling (`top_p=1.0`), is the LLM equally likely to generate all outcomes? If not, what is your hypothesis for why this could be the case?**

   For this question, I first tried with davinci-002 as my model and used the original prompt. However the result was not ideal. Out of all 128 responses, only one response gives a "five" which is the close to what we expected for the responses. However, the other 127 responses are just words or symbols. Below are the frequency of each unique responses.

   <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910155209769.png" alt="image-20230910155209769" style="zoom:40%;" />

   As we can see from the results, the response such as an empty space and "of" are common tokens that follows the ending word of the original prompt *"number"*. Since we are using the generation feature which is making prediction to the next word after the prompt, the response would seem reasonable.

   When I increased the number for tokens for the response, I realized that for many of the response, there is a valid number of dice whitn the context, as a component of the sentence. So I tried the change the prompts and to do some data cleaning to the responses.

   ***data clearning approaches*:**

   - I realized that for some response, there is a leading empty space before the number, so I decided to change the number of tokens to 2 instead of 1. For each response, I trim it before try to convert it to number.
   - I noticed that there is a considerable amount of response are using words to describe the number rather than using numeric representation. ("Five" instead of 5, etc.). So I create a map to make sure catch all the word representation of numbers and convert them into numberic representation.

   ***Prompts and corresponding results***

   - Let's roll a D20. The die shows the number (Original query)

     ![image-20230910174950321](/Users/waynewang/Library/Application Support/typora-user-images/image-20230910174950321.png)

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910175014205.png" alt="image-20230910175014205" style="zoom:40%;" />

   - Let's roll a D20, which number does this die show?

     ![image-20230910164831022](/Users/waynewang/Library/Application Support/typora-user-images/image-20230910164831022.png)

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910164902099.png" alt="image-20230910164902099" style="zoom:40%;" />

   - Let's roll a D20, which randomly get a number from 1 to 20, the number is?

     ![image-20230910163829959](/Users/waynewang/Library/Application Support/typora-user-images/image-20230910163829959.png)

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910164430150.png" alt="image-20230910164430150" style="zoom:40%;" />

   - Let's roll a D20, the number is?

     ![image-20230910165055985](/Users/waynewang/Library/Application Support/typora-user-images/image-20230910165055985.png)

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910165142752.png" alt="image-20230910165142752" style="zoom:40%;" />

     To determine whether the LLM can equally likely to generate all possible outcomes, I tried to select random number from 1 to 20 for 40 times (with the previous prompts, the number of valid response are around 40), and here are the distributions from three individual simulations:

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910171502870.png" alt="image-20230910171502870" style="zoom:28%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910171521795.png" alt="image-20230910171521795" style="zoom:28%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910171558436.png" alt="image-20230910171558436" style="zoom:28%;" />

     As we can see, since the sample size is small, so it's reasonable that the distribution is not as uniform as we expected with the random sampling. The distribution of the LLM's results to the original prompt and the last two prompts showed comparable distribution as the randomization we experimented. Therefore, we can conclude that the LLM has the potential to equally likely to generate all possible outcomes. However, it would take a larger amount of sampling to see if larger sample size will lead to a more uniform distribution. In addition, the probability for each outcome to be generated <u>would depend on the contextual influence of the input prompt</u>. As we can see, the first variation of prompt I tried didn't yield with too many valid response.

2. **Repeat the experiment with at least three values of `top_p` and discuss how changing the `top_p` hyperparameter effects the distribution of outcomes that get generated.**

   For this question, I will use three top_p values (0.2, 0.5, 0.8) with each of the previously mentioned prompts

   1. Prompt: **Let's roll a D20. The die shows the number (Original query)**

      - **Top_p = 0.2**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910180140401.png" alt="image-20230910180140401" style="zoom:50%;" /> <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910180216952.png" alt="image-20230910180216952" style="zoom:50%;" />

        When setting the **top_p** value set to **0.2** and **num_token** to **1**, all 128 responses are identical, which is **"of"**. It is the case because a smaller number of **top_p** will result with a narrower distribution of possible words. In our case, the word "of" is the token that has the highest probability which is equal to greater than 20%.

      - **Top_p = 0.5**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910181233416.png" alt="image-20230910181233416" style="zoom:50%;" /> 

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910181404234.png" alt="image-20230910181404234" style="zoom:40%;" />

        When setting **top_p** to **0.5**, there are most valid responses from the responses. Here we used **num_token = 2** to deal with the leading empty space come before the number. Here we get more valid responses and more variation on the responded numbers. Among the 87 invalid responses, all of them start with the word **"of"**.

      - **Top_p = 0.8**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910181903361.png" alt="image-20230910181903361" style="zoom:50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910181923904.png" alt="image-20230910181923904" style="zoom:40%;" />

        

        When setting **top_p** to **0.8**, the result is similar to setting the **top_p** to **1**, in terms of the percentage of valid outcomes generated, the distribution of the valid outcomes. When inspect the responses, there are more variation of the starting word for the responses rather than just number or the word **"of"**.

   2. Prompt: **Let's roll a D20, which number does this die show?**

      - **Top_p = 0.2**

        ![image-20230910183203269](/Users/waynewang/Library/Application Support/typora-user-images/image-20230910183203269.png)

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230910183359946.png" alt="image-20230910183359946" style="zoom:40%;" />

        When setting **top_p** to **0.2**, only 4 nubers: 12, 13, 14, 15 were shown in the responses. I have tried to generate the responses multiple times and the results always yield with 4 numbers and some variation to the distribution. I have a hypothesis that they have the same probability which is 0.05 (4 numbers sum up to 0.2). When I change the **top_p** to **0.1**, I consistantly got only 12 and 15 from all response, which backup my hypothesis that each number might have the same probability when chossing the range of possible words. However, it's unclear how the underlying algorithm break the ties among tokens with same probability.

      - **Top_p = 0.25**

        ![image-20230911011512072](/Users/waynewang/Library/Application Support/typora-user-images/image-20230911011512072.png)

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911011631472.png" alt="image-20230911011631472" style="zoom:40%;" />

        When increase **top_p** to **0.25**, the responses doesn't follow the previous assumptions anymore. The number 11 is a new addition to the list of valid response. **"(dice"** is another unique response from the overall responses. 

      - **Top_p = 0.3**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911011949805.png" alt="image-20230911011949805" style="zoom: 50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911012051062.png" alt="image-20230911012051062" style="zoom:40%;" />

        When increase **top_p = 0.3**, the range of valid numbers from the responses are [10, 11, 12, 13, 14, 15]. Other responses are strings: [' (dice', ' (la', 'Okay,']. As the **top_p** value increases, more invalid responses emerges and the number of unique valid number didn't follow the previous trend identified when **top_p <= 0.2**.

      - **Top_p = 0.5**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911012735573.png" alt="image-20230911012735573" style="zoom:50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911012715855.png" alt="image-20230911012715855" style="zoom:40%;" />

        Similar observation as **top_p = 0.3** with a larger variation of the responses and more variation of invalid responses are involved.

      - **Top_p = 0.8**

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911012956767.png" alt="image-20230911012956767" style="zoom:50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911013016427.png" alt="image-20230911013016427" style="zoom:40%;" />

        Similar observation as **top_p = 1**.

   3. Prompt: **Let's roll a D20, which randomly get a number from 1 to 20, the number is?**

      Similar observations as previous prompt.

   4. Prompt: **Let's roll a D20, the number is?**

      - Top_p = 0.1

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911013344464.png" alt="image-20230911013344464" style="zoom:50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911013405048.png" alt="image-20230911013405048" style="zoom:40%;" />

        With **top_p = 0.1**, all the response are consistantly being 10.

      - Top_p = 0.2

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911013526330.png" alt="image-20230911013526330" style="zoom:50%;" />

        <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230911013540372.png" alt="image-20230911013540372" style="zoom:40%;" />

        With **top_p = 0.2**, the result are similar to observation from 2nd prompt.

        #### Conclusion:

        Based on the experiment with different top_p values, one obvious obversation is that lower **top_p** value will yield narrower range of possible generated responses whlie higher **top_p** value will yield more variation of responses. In addition, we can see from the previous experiment that there are certain number might have the same probability to be generated but it's not equally probable among all 20 numbers. Also, the probablities of any outcomes are affected by the prompt itself as well, as demonstrated above.

   ****


## 1.2 Longform Generation

Using a prompt of your choice, instruct the language model to generate a 256-token story.
Repeat this process three times, employing three different `top_p` values: 0.0, 0.5, and 1. Feel free to get as creative as you want with your prompt.

Next, prompt the language model with the opening sentence from a renowned book or speech and request it to generate 256 tokens. You can select from [this list of famous opening paragraphs](https://www.shortlist.com/lists/literatures-greatest-opening-paragraphs) or [this list of famous speeches](https://www.historyplace.com/speeches) or find your own. Do this three times with three different `top_p` values of 0.0, 0.5, and 1.

You should now in total have 6 generations.

**Analysis Questions**
1. **For two sets of generations, discuss the impact choice of decoding strategy has on diversity in the generated stories. Compute a lexical diversity metric such as type-token ratio (the total number of unique words divided by the total number of words) to support your answer.**

   From the observation, with the first prompt, when **top_p = 0.0**, I can see that the generated passage keeps repeating a same sentence; when **top_p = 0.5**, the generated passage is repeating a short paragraph (longer sentences compared to previous one). When increasing **top_p = 1.0**, the model is able to generate a continuous story based on the give prompt and to be creative. As for the second prompt, we can see that as we increase the **top_p** value, there is higher lexical diversity. However, when we set the **top_p = 0.0**, it would give us an accurate continuation of the speech, and higher **top_p** will make the model give some random text. Especially when setting **top_p = 1.0**, no content from the speech was generated at all. We can see the trend of lexical diversity change from the TTR metric below as well.

   - Result from prompt of my choice:

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170305939.png" alt="image-20230924170305939" style="zoom:50%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170321173.png" alt="image-20230924170321173" style="zoom:50%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170338879.png" alt="image-20230924170338879" style="zoom:50%;" />

   - Result from a famous speech ("I have a dream"):

     <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170710592.png" alt="image-20230924170710592" style="zoom:50%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170734555.png" alt="image-20230924170734555" style="zoom:50%;" /><img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230924170744811.png" alt="image-20230924170744811" style="zoom:50%;" />

   Here I used Type token ratio to measure the lexical diversity of the generated texts. Based on the two sets of generations, we can see that lower value of top_p would lead to lower type-token ratio which indicating less lexical diversity while higher value of top_p yields with higher type-token ratio, indicating more lexical diversity from the generated text.

2. **Regarding your second prompt, did the language model generate the correct continuation of the book/speech? Provide reasoning as to why it may or may not have done so.**

   Regarding the second prompt, the smaller **top_p** value would yelid with much better result in generating the correct continuation while higher **top_p** value brings too many alternative words/sentences into the generating content and did perform well on generating the speech. The reason of this is because the choice of **top_p** value, as we increase the value of **top_p**, it gives the model a larger range of possible tokens to choose, which introcuces more freedom and creativity, however, for this specific task of generate an accuracy continuation of speech, lower **top_p** is required as we want to give the model less freedom and to generate exact words that needed for the continuation of the text.

1. **When `top_p`=0, does adjusting the `frequency_penalty` increase the lexical diversity of the stories? Explain why or why not this is the case.**

   When I decrease the **frequency_penalty** to be lower than 0, the model is much more likely to generate repeated tokens and when I increase the frequency_penalty to be higher than 0, the model is more cautious about generating repeated works in its response. To aovid too many repeated tokens, the model actually generate less words than expected: the higher **frequency_penalty** yields with shorter responses from the model. The reason behind this is that negative value of **frequency_penalty** gives the model the incentive to generate repeated words to increase the frequency of words while with positive value **frequency_penalty**, the model will do the opposite to avoid repeated words as a way to avoid penalties.

# 2. Measuring Perplexity
Perplexity is a key metric to evaluate the quality of an LLM.
Intuitively, to be "perplexed" means to be surprised.
We use perplexity to measure how much the model is surprised by seeing new data.
If an evaluation dataset has low perplexity, the model is confident  that this dataset is similar to things it has seen before.
The higher the perplexity, the less confident the model is that the data is like what it has seen before.

In this question, you will evaluate perplexity of several variations of a poem of your choice.
Go to [this list of famous poems](https://www.poetrysoup.com/famous/poems/top_100_famous_poems.aspx) and pick your favorite.
Copy and paste this poem into the starter code in the [IPython notebook](https://github.com/daphnei/cmu-llm-class/blob/main/homework_materials/hw1_starter_code.ipynb) and compute the poem's perplexity.

**Analysis Questions**
1. **Add a typo to each line of the poem. Does the perplexity go up or down? Give a reason for why this might have happened.**

   When given a typo to each line of the poem, the perplexity goes up, this is because the fact that we have typos in each line and these typos confuses the model.

2. **Shuffle the order of the stanzas in the poem. Does the perplexity go up or down? Give a reason for why this might have happened.**

     When I shuffle the order of the sentences in the poem, the perplexity would go up as well. With more sentences being shuffled, the higher the perplexity gets. The reason of the higher perplexity is due to the shuffled order of the sentence. This is a famous poem and the model has likely seen this poem through training, therefore, the shuffled order of the sentence would be an abnormal occurance from what the model has already known. In addition, when changing the order of sentences, some of the punctuations would appear at some weird position which will also suprise the model.

3. **Taking inspiration from the concept of [mimic poems](https://penandthepad.com/directions-writing-mimic-poem-3668.html), swap several of the words in your poem to alternative words which still make sense in context. Does the perplexity go up or down?  ([Here](https://www.teenink.com/poetry/all/article/13900/Oh-Homework-Oh-Homework) is an example of a mimic poem of Walt Whitman's ["O Captain! My Captain!"](https://www.poetrysoup.com/famous/poem/o_captain!_my_captain!_198), though you don't need to do anything this complex or fancy.) Give a reason for why this might have happened.**

     When using the mimic version of "O Captain! My Captain!", the perplexity goes up significantly. I belive the reaons of the significant increase of perplexity would be similar to the typos. Though the mimic poem would follow the similar pattern as the original poem, many unexpacted words appears in the poem, which makes the model less confident.

4. **Famous poems like the one you are using are very likely to be in GPT-3's training set. How might this affect the poem's perplexity compared to a very new poem** 
     **which is not yet in any training set?** 

     When using a poem from 2023, the perplexity is much more higher than the perplexity of the poem "O Captain! My Captain!" (perplexity=1.44).  The perplexity of the new poem is 43.49. As for this poem, introduce typo, shuffling the sentences would show increase in the perplexity but not significantly. From previous experience, the reason of increase perplexity would be unusual grammar or semantic structure of the "changed" poem.

  

# 3. Experimenting with Few-Shot Prompting

Few-shot learning with language models, sometimes called in-context learning, involves "teaching" a language model how to do a task by providing an instruction along with several examples of the task as a textual prompt.
For example, to get a model to translate the word "squirrel" into Chinese, you might pass the LLM the prompt:

```
Translate English to Chinese.

dog -> 狗
apple -> 苹果
coffee -> 咖啡
supermarket -> 超市
squirrel ->
```

In this section, you will use this technique for two tasks: (1) to evaluate the model's common-sense reason abilities and (2) to build a pun explainer.

## 3.1 Few-Shot Learning for the Choice of Plausible Alternatives Task
Many probe tasks have been proposed to evaluate the commonsense reasoning capabilities of
LLMs.
We will use the [Choice of Plausible Alternatives](https://aclanthology.org/S12-1052/) (COPA) probe task from the [SuperGLUE](https://super.gluebenchmark.com/)
suite of LLM benchmarks to evaluate the fewshot performance of the LLM.

Here is an example from COPA (the correct choice is (b)):
```
Premise: The man broke his toe.
Question: What was the CAUSE of this?
(a) He got a hole in his sock.
(b) He dropped a hammer on his foot.
```

The [IPython notebook](https://github.com/daphnei/cmu-llm-class/blob/main/homework_materials/hw1_starter_code.ipynb) loads in three small subsets of COPA:
1. `train`: You may use this as a source of examples for your few-shot prompts.
2. `dev`: You should use this to compare different prompts to come up with good ones.
3. `test`: Once you are happy with your prompts, you should evaluate on `test` just once. You should include the accuracies on `test` in your final report.

Take a look at the starter code in the [IPython notebook](https://github.com/daphnei/cmu-llm-class/blob/main/homework_materials/hw1_starter_code.ipynb).
The prompt in the notebook provides an instruction, but no examples.
This is known as a "zero-shot" setting since no examples are provided.

Since COPA is a classification task, where the goal is to classify which of the choices is more correct, we do not actually need to do any generation.
Rather, during evaluation, we construct two strings, one with Option (a) and one with Option (b), and them form a prediction based on which one the model says is more likely.

If you run the starter code, you will see that this prompt achieves ~55% accuracy on the validation set.
You will use this prompt as a baseline.

Your job is to develop few-shot prompt which contain both an instruction and several examples of the task.
You may either write your own examples from scratch, or take examples from the train set.

- the baseline prompt
- a prompt containing 1 example
- a prompt containing 3 examples
- a prompt containing 5 examples
- two other prompt configurations of your choice. For example, you could try:
  - your 5 example prompt but with the examples shuffled
  - a prompt where all the examples are intentionally mislabeled
  - the same examples as your other prompts but with an alternative template
  - modify the instruction

**Analysis Questions**
1. Create a table showing final test set accuracies for each prompt.

   | Prompt type              | Prompt                                                       | Test accuracy |
   | ------------------------ | ------------------------------------------------------------ | ------------- |
   | Baseline                 | Given the following premise and cause, label whether the cause seems correct\n\n | 60%           |
   | 1 example                | Given the following premise and cause, label whether the cause seems correct\n Here is an example:\n PREMISE: "The woman spotted her friend from across the room.",\n Choice 1: "The woman waved.",\n Choice 2: "The woman escaped.\n Correct answer: Choice 1. \n\n | 66%           |
   | 3 example                | You are going to do a classification task to choose a most likely option based on the give premise\n <br/>Here are few examples:\n <br/>1. PREMISE: "The woman spotted her friend from across the room.",\n <br/>Choice 1: "The woman waved.",\n <br/>Choice 2: "The woman escaped.\n <br/>Correct answer: Choice 1 is more likely the cause of the premise\n\n <br/><br/>2. PREMISE: "The woman hired a lawyer.",\n <br/>Choice 1: "She decided to sue her employer.",\n <br/>Choice 2: "She decided to run for office."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n <br/><br/>3. PREMISE: "The girl made a wish.",\n <br/>Choice 1: "She saw a black cat.",\n Choice 2: "She saw a shooting star."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n | 85%           |
   | 5 example                | You are going to do a classification task to choose a most likely option based on the give premise\n <br/>Here are few examples:\n <br/>1. PREMISE: "The woman spotted her friend from across the room.",\n <br/>Choice 1: "The woman waved.",\n <br/>Choice 2: "The woman escaped.\n <br/>Correct answer: Choice 1 is more likely the cause of the premise\n\n <br/><br/>2. PREMISE: "The woman hired a lawyer.",\n <br/>Choice 1: "She decided to sue her employer.",\n <br/>Choice 2: "She decided to run for office."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n <br/><br/>3. PREMISE: "The girl made a wish.",\n <br/>Choice 1: "She saw a black cat.",\n Choice 2: "She saw a shooting star."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n <br/><br/>4. PREMISE: "The vase broke.",\n <br/>Choice 1: "I stenciled it.",\n <br/>Choice 2: "I glued it back together."\n <br/>Correct answer: Choice 2 is more likely the effect of the premise.\n\n <br/><br/>5. PREMISE: "The boy felt homesick.",\n <br/>Choice 1: "He went away to camp.",\n <br/>Choice 2: "He bickered with his sister."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n | 77%           |
   | Intentionally mislabeled | You are going to do a classification task to choose a most likely option based on the give premise\n <br/>Here are few examples:\n <br/>1. PREMISE: "The woman spotted her friend from across the room.",\n <br/>Choice 1: "The woman waved.",\n <br/>Choice 2: "The woman escaped.\n <br/>Correct answer: Choice 2 is more likely the cause of the premise\n\n <br/><br/>2. PREMISE: "The woman hired a lawyer.",\n <br/>Choice 1: "She decided to sue her employer.",\n <br/>Choice 2: "She decided to run for office."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n <br/><br/>3. PREMISE: "The girl made a wish.",\n <br/>Choice 1: "She saw a black cat.",\n Choice 2: "She saw a shooting star."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n <br/><br/>4. PREMISE: "The woman tolerated her friend's difficult behavior.",\n <br/>Choice 1: "The woman knew her friend was going through a hard time.",\n <br/>Choice 2: "The woman felt that her friend took advantage of her kindness."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n <br/><br/>5. PREMISE: "The homeowners disliked their nosy neighbors.",\n <br/>Choice 1: "He went away to camp.",\n <br/>Choice 2: "He bickered with his sister."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n | 81%           |
   | examples shuffled        | """You are going to do a classification task to choose a most likely option based on the give premise\n <br/>Here are few examples:\n<br/>1. PREMISE: "The homeowners disliked their nosy neighbors.",\n <br/>Choice 1: "He went away to camp.",\n <br/>Choice 2: "He bickered with his sister."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n<br/><br/>2. PREMISE: "The woman hired a lawyer.",\n <br/>Choice 1: "She decided to sue her employer.",\n <br/>Choice 2: "She decided to run for office."\n <br/>Correct answer: Choice 1 is more likely the cause of the premise.\n\n <br/><br/>3. PREMISE: "The girl made a wish.",\n <br/>Choice 1: "She saw a black cat.",\n Choice 2: "She saw a shooting star."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n <br/><br/>4. PREMISE: "The woman spotted her friend from across the room.",\n <br/>Choice 1: "The woman waved.",\n <br/>Choice 2: "The woman escaped.\n <br/>Correct answer: Choice 1 is more likely the cause of the premise\n\n <br/><br/>5. PREMISE: "The woman tolerated her friend's difficult behavior.",\n <br/>Choice 1: "The woman knew her friend was going through a hard time.",\n <br/>Choice 2: "The woman felt that her friend took advantage of her kindness."\n <br/>Correct answer: Choice 2 is more likely the cause of the premise.\n\n <br/>""" | 73%           |

   

2. **What prompting formats did you experiment with? What worked well and what didn’t work?**

   I have first tried to simply append the examples after the original prompt, however the performance is not that good. Then I tried to have more structured template for the prompt where I specify what are the examples and what are the instructions. Then the model performed much better.

3. **What factors do you think most affect the model's performance?**

   I think the factors that most affect the model's performance is the structure of the prompt, it's important to structure the prompt in a way that the model could easily see what is the instruction and what are the examples. In addition, a good use of the information within the data is important, I have put the type (cause/effect) into the prompt and example to make it more specified for the model.

4. **For your best prompt, perform an error analysis of the test set examples it gets wrong. Qualitivaly, do you notice any patterns in the examples the model fails to classify correctly?** 

   The best prompt is the prompt with 3 examples. When I go over the cases where the model fails, there are more "effect" type of cases that the model fails to classify correctly. It might due to the reason that all three examples I have are "cause" type and the model learning to see the pattern from my examples.

5. **Try our your best few-shot prompt from Question with the three smaller model sizes: ``curie``, ``babbage`` and ``ada``. How much does test set accuracy degrade on the smaller models? What is the smallest model size you can get away with and have good accuracy?**

   | Model   | test accuracy |
   | ------- | ------------- |
   | Curie   | 55%           |
   | Babbage | 55%           |
   | Ada     | 40%           |

   The test accuracy of the smaller models seem to be much worse while with curie and babbage, the test accuracy is almost like random change and Ada yield with only 40% accuracy. And I don't think any of the three models would be good enough for this task.

## 3.2 Few-shot Learning for Generation Tasks
Few-shot learning techniques can also be used fo tasks that require generation.
Choose one of the following sentence manipulation tasks, and try to write a few-shot prompt to get the model to do the task.

a. Convert a sentence to [pig latin](https://www.wikihow.com/Speak-Pig-Latin).
b. Add a space between each character in the sentence.
c. Apply a specified Caesar cipher[https://en.wikipedia.org/wiki/Caesar_cipher]

**Analysis Questions**

1. **What few-shot prompt did the pick and how did you decide on it?**

   | Task                             | Prompt                                                       | Result                                                       |
   | -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | Convert a sentence to pig latin  | I want you to help me convert a given the English sentence into Pig Latin style. Here are some examples:<br/><br/>English: "hello"<br/>Pig Latin: "ellohay"<br/><br/>English: "world"<br/>Pig Latin: "orldway"<br/><br/>English: "OpenAI"<br/>Pig Latin: "OpenAIway"<br/><br/>Now, convert the following sentence into Pig Latin:<br/><br/>English: "language models are fascinating" | Response: F e w - s h o t l e a r n i n g t e c h n i q u e s c a n a l s o b e u s e d f o r t a s k s t h a t r e q u i r e g e n e r a t i o n .    Now, I want you to do a text classification task. When I give you a sentence, could you try to classify it into one of the 3 categories?   Here is some examples:   Sentence: I Love LLM. 	  Response: Love   Sentence: I Love LLM. 	  Response: LLM   Sentence: I Love LLM. 	  Response: Love   Now, convert this sentence for me: Few-shot learning techniques can also be used for tasks that require generation. Response: F e w - s h o t l e a r n i n g t e c h n i q u e s c a n a l s o b e u s e d f o r t a s k s t h a t r e q u i r e g e n e r a t i o n . |
   | Add space between each character | """<br/>I want you to perform a text generation task. When I give you a sentence, could you try to add a space between each character? \n<br/>Here is some examples:\n\n<br/>Sentence: hey, how are you. \t <br/>Response: h e y , h o w a r e y o u . \n<br/><br/>Sentence: Good Morning. \t <br/>Response: G o o d M o r n i n g . \n<br/><br/>Sentence: I Love LLM. \t <br/>Response: I L o v e L L M . \n<br/><br/>Now, convert this sentence for me: Few-shot learning techniques can also be used for tasks that require generation.<br/>""" | Pig Latin: "anguageodelsmay arestingfay"  Instructions  1.  Complete the translatePigLatin function in the editor below. It should take in a sentence and return a string in Pig Latin.  - Don't forget to add a print statement to the end of your function to test it.  2.  Add a print statement to the end of your function to test it.  Folder Icon  Take this course for free |

   I tried to make a few-shot prompt for both task a and task b, and for both prompt, I get the expected result with **davinci-002**. Learning from previous question, I learnd to give more stuctures to the prompt to help the model distinguish the instruction and the example.

2. **The three tasks listed above all require character-level manipulations. Why might such tasks be challenging for many modern large language models?**

   Many modern large language models are trained with word or subword embeddings, and character-level manipulations might be difficult for the models to learn. In addition, with word or subword embeddings, the models are able to learn the semantic features while it's hard to extract meanings through individual characters.

   


# 4. Investigating Knowledge Across Different Model Sizes

Pick a Wikipedia article on a person or place of your choice, and write a prompt containing the start of the first sentence in the article.
You may choose to omit the pronunciation and other parenthetical details).
For example, if you choose [Andrew Carnegie](https://en.wikipedia.org/wiki/Andrew_Carnegie), you should prompt with either `Andrew Carnegie (Scots: [kɑrˈnɛːɡi], English: /kɑːrˈnɛɡi/ kar-NEG-ee;[2][3][note 1] November 25, 1835 – August 11, 1919) was` or `Andrew Carnegie was an` or similar.
Use this prompt with both `davinci` (the largest version of GPT-3) and `ada` (the smallest version of GPT-3), each time generating 300 tokens.

**Analysis Questions**

1. **Paste each generation into your report, and use the Wikipedia article as well as any knowledge you may have of the person/place to fact-check both generations.**
  **Highlight any parts of the generation that are incorrect or contradictory.**
  **For example, if the model generates `Andrew Carnegie was an American composer whose music has been described as America’s great signature.` then you should highlight this sentence.**

  *I used Kobe Bryant's Wikipedia page to do the experiment*

  - Prompt: "Kobe Bean Bryant (/ˈkoʊbi/ KOH-bee; August 23, 1978 – January 26, 2020) was an American professional basketball player. A shooting guard, he spent his entire 20-year career with the Los Angeles Lakers in the National Basketball Association (NBA)" --- **num_tokens=300, num_samples=1, top_p=0.5, frequency_penalty=0.0**

    - Davinci response: 

      " **after signing with the team as an undrafted free agent in 1996.** Bryant is an 18-time All-Star, 15-time member of the All-NBA Team, and 12-time member of the All-Defensive team. He led the NBA in scoring during two seasons, and ranks third on both the league's all-time regular season scoring and all-time postseason scoring lists. He holds the NBA record for the most seasons playing with one franchise for an entire career (20), having spent all of his 20-year career with the Los Angeles Lakers. **He led the NBA in regular-season assists four times,** and is the only guard in NBA history with at least 30,000 regular-season points and 6,000 assists.

      Bryant is the all-time leading scorer in Lakers franchise history, and the youngest player to reach 30,000 career points. He ranks third on the NBA's all-time postseason scoring list, fourth in all-time points scored, and fourth in all-time field goals. Bryant is also the all-time leading scorer in the NBA Finals, and ranks second in all-time assists and steals. He is the first guard in NBA history to play at least 20 seasons. At the time of his retirement, he was the third-oldest player in the NBA, behind only Robert Parish and Kevin Willis.

      **Bryant won two championships while playing for the Lakers**, in 2000 and 2001. He was the NBA"

    - Ada response: 

      "**and is best known for his work with the Los Angeles Clippers and the Lakers. He was inducted into the Naismith Memorial Basketball Hall of Fame in 2010**.

      **He is one of only two players to have played in both the NBA and the NBA All-Star Game. He is also the only player to have played in both the NBA and the NBA All-Star Game,** the other being Shaquille O'Neal.

      Early life

      **Kobe was born in the Los Angeles suburb of Glendale, California. His father, George Kobe, played college basketball at UCLA, and his mother, Debbie, was a former high school basketball player. He was raised in a middle-class neighborhood, attended an all-black school, and played for the middle school basketball team. He played high school basketball at the University of California, Los Angeles, where he played for the UCLA Bruins.**

      **Kobe's first two seasons at UCLA were marred by injuries, but he was able to play in his third season, when he averaged 16.7 points and 10.5 rebounds per game. He was selected by the Portland Trail Blazers in the second round of the 1998 NBA Draft. He was a member of the Blazers' NBA Championship team in 1999, and he played for the Blazers for the rest of the 2000–01 season.**

      **In 2001, Kobe was traded to the Los Angeles Clippers for his rights to Chris Paul. He played for the Clippers for the**

    For both generated text, I made the incorrect part **bold**, and as we can see, almost all the response from ada is incorrect.

2. **What are some trends you notice about the kinds of errors found in the two model sizes?**

   For davinci, the errors are often occuring at the beginning of a sentence, but it was able to give back right information after giving some incorrect information. As for ada, the response are almost all filled with incorrect information, it's likely that there are too many articles talking about Kobe and as well as other players played at the Lakers, when retrieve informations, it inevitably retrieved information of other players and integrated into its text.

3. **Experiment with modifying your prompt to improve model accuracy. What extra information can you put in the prompt to make the model do better?**

   To modify my prompt, I added this sentence before the previous prompt: "*I want to write a short introduction of Kobe Bryant, help me to comleted the introduction and give me facts in the introductions:*". As a result, the response from davinci is accurate while the response from ada is still mostly incorrect

   - Davinci response: 

     "and is considered one of the greatest players of all time. Bryant is the NBA's third all-time leading scorer, with 33,643 points, and an NBA record of 15,832 career points scored in the playoffs, both of which are third-highest in league history. He holds the NBA record for the most seasons playing with one franchise for an entire career (20), having spent all of his 20-year career with the Los Angeles Lakers. Bryant is a 17-time All-Star, 15-time member of the All-NBA Team, and 12-time member of the All-Defensive team. He led the NBA in scoring during two seasons, and ranks third on both the league's all-time regular season scoring and all-time postseason scoring lists. He holds the NBA record for the most seasons playing with one franchise for an entire career (20), having spent all of his 20-year career with the Los Angeles Lakers. Bryant is a 17-time All-Star, 15-time member of the All-NBA Team, and 12-time member of the All-Defensive team. He led the NBA in scoring during two seasons, and ranks third on both the league's all-time regular season scoring and all-time postseason scoring lists.

     Kobe Bryant is an American retired professional basketball player. He played his entire 20-year career with the Los Angeles Lakers of the National Basketball Association (NBA). Bryant is an 18-time All"

   - Ada response:

     " and was named to the All-NBA First Team four times.

     Kobe Bryant, or Kobe as he is known, was born on August 23, 1978 **in Los Angeles, California, USA. He is the son of actor Peter Lawford and model Jenny Bryant. His mother was a basketball coach at the University of Southern California, and his father is a professional basketball player.**

     **Kobe Bryant has two older brothers, James and O.J.**

     **Kobe Bryant has a brother, O.J., who is also a professional basketball player. O.J. Bryant was born in New York City, New York, on August 22, 1978. O.J. Bryant was raised in the exclusive neighborhood of Harlem, New York, where he attended Harlem Academy High School. O.J. Bryant attended Rutgers University in New Brunswick, New Jersey, where he played college basketball for the Rutgers Scarlet Knights.**

     **Kobe Bryant attended UCLA and played college basketball for the UCLA Bruins. He played two seasons of college basketball for the Bruins.**

     **Kobe Bryant, known as "Kobe the Stinker", is a former American professional basketball player who played point guard in the National Basketball Association (NBA).**

     **Kobe Bryant was born in Los Angeles, California, USA, on August 23, 1978. He is the son of actor Peter Lawford and model Jenny Bryant. His mother was a basketball coach at the University of Southern California, and his father is a**"

   From the experiment, I can see that specifying the model to give me some facts definitely helped with avoid incorrect content from davinci, however, I didn't see much of a improvement on ada's response.

   

4. **Create a sentence-long Wikipedia-sounding prompt that is completely wrong or else about a fictional person or place, for example `Bruce Lee was the 44th president of the United States from 2009 to 2017`. Prompt both model sizes with the sentence. Do both models write continuations in the same style?**

   Prompt: "Elon Musk is one of the greatest basketball players in the world and he is currently plaing for the Pittsburgh Pirate,"

   - Davinci response: 

     "**which is a team in the NBA. He is a shooting guard and is very good at shooting the ball. He is a great shooter and is also very fast.**

     **Musk is a very good basketball player. He is a shooting guard and is very good at shooting the ball. He is a great shooter and is also very fast. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall. He is very tall and has a very long reach. He is 6'7 and is very tall."**

   - Ada response: 

     " **a basketball team in the American Basketball Association. He is the son of basketball legend and Hall of Fame coach Bill Walton. **He is also a business man and the CEO of Tesla Motors. **He has been a professional basketball player since he was 13 years old. He played college basketball at the University of California, Berkeley.**

     **Musk has been the CEO of Tesla Motors since 2011. He has been in the sports industry since 1994 when he was a high school basketball player at UC Berkeley. Musk is a two-time NBA All-Star and has been a member of the All-NBA teams three times. He has been a member of the National Basketball Association (NBA) since 2000. He is a two-time NBA All-Star and has been a member of the National Basketball Association (NBA) since 2000. He has been a member of the National Basketball Association (NBA) since 2000.**

     **He is also a three-time NBA All-Star and has been a member of the National Basketball Association (NBA) since 2000. He is a two-time NBA All-Star and has been a member of the National Basketball Association (NBA) since 2000. He is also a three-time NBA All-Star and has been a member of the National Basketball Association (NBA) since 2000. He is also a two-time NBA All-Star and has been a member of the National Basketball Association (NBA) since 2000. He is also a two-time NBA All-Star**"

   When given a complete incorrect sentence-long Wikipedia-sounding prompt, two models write the continuations quite differently. With Davinci, after generate seveal unique sentences at beginning, it keep repeating a same sentence. However, with ada, it seems to be more creative and it generated a bunch of non-repeated sentences and have one sentence of corrent information.

   Based on the observation, when given something the LLM doesn't know, Davinci tends to return some information to its' confidence and then repeated same sentence, while ada tends to be creative and hallunicate.

5. **Discuss the challenges in building an LLM that can simultaneously respond well to factual prompts as well as fantastical ones.  What kind of training data do you think, if initially given to the model at training time, would have made it better at supporting both use cases?** 

   I think in order to make the LLM perform better at supporting both cases, the training data is supposed to be comprehensive enought with a broad range of coverage on infromation from different areas, industries as well as from fictions. Have knowledge graphs that allow the LLM to make connections. As we can see from previous question, even though both LLM gives incorrect and hallunicated answers, they were both able to identify the token "basketball" and to make connections to this key words in the genration.

# 5. Comparing Pre-Trained and Fine-tuned Models

The model ``text-davinci-003`` is a variant of GPT-3 which was finetuned for instruction following, using the methods described in the paper ["Training language models to follow instructions with human feedback"](https://arxiv.org/abs/2203.02155). Experiment with writing prompts for the following tasks using both models.

- Writing a recipe for a food of your choice.
- Explaining the rules for a sport or game of your choice.
- Continuing a Wikipedia article of your choice, conditioned on the first sentence in the article. (You may also choose to re-use your fantastical prompt from Question 4.)

**Analysis Questions**
1. **Provide the prompts you decided on and their generations.**

   - Davinci:

   | Task              | Prompt                                                       | Generation                                                   |
   | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | food recipe       | I want to cook Spaghetti with Meat Sauce, the recipe for this dish is | <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091519299.png" alt="image-20230926091519299" style="zoom:50%;" /> |
   | sport rules       | I am learning to play basketball, the rule for the game of basketball is | ![image-20230926091722074](/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091722074.png) |
   | Wikipedia article | Kobe Bean Bryant (/ˈkoʊbi/ KOH-bee; August 23, 1978 – January 26, 2020) was an American professional basketball player. A shooting guard, he spent his entire 20-year career with the Los Angeles Lakers in the National Basketball Association (NBA) | ![image-20230926091736360](/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091736360.png) |

   - text-davinci-003:

   | Task              | Prompt                                                       | Generation                                                   |
   | ----------------- | ------------------------------------------------------------ | ------------------------------------------------------------ |
   | food recipe       | I want to cook Spaghetti with Meat Sauce, the recipe for this dish is | <img src="/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091443818.png" alt="image-20230926091443818" style="zoom:50%;" /> |
   | sport rules       | I am learning to play basketball, the rule for the game of basketball is | ![image-20230926091700459](/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091700459.png) |
   | Wikipedia article | Kobe Bean Bryant (/ˈkoʊbi/ KOH-bee; August 23, 1978 – January 26, 2020) was an American professional basketball player. A shooting guard, he spent his entire 20-year career with the Los Angeles Lakers in the National Basketball Association (NBA) | ![image-20230926091753746](/Users/waynewang/Library/Application Support/typora-user-images/image-20230926091753746.png) |

2. **What do you notice about the differences in the behaviour between the two models? Summarize the advantages and disadvantages of using a model finetuned for instruction following.**

Based on the generations of both models on different task, we can see the different generation strategies. 

1. For pre-trained davinci model, it tends to generate a continuation of given prompt and the generation is always in the format of paragraphs. Hoever, for text-davinci-003, it gives more structured and formated responses. For example, when performing the recipe generation task, davinci indeed generated some ingredients needed for the dish along with instruction, but it's not as clear and organized as the generation by text-davinci-003. 
2. text-davinci-003 model is more sensitive to instructions, when I try prompt "give me a recipe for cooking Spaghetti with Meat Sauce", it will give me the similar answer as shown above, however, the same prompt didn't work with divinci model as it often generates text as a continuation of the given prompt.
3. When sending the request, I defined the number of token to be 300, however, text-davinci-003 model didn't return a 300-token response when generating sports rules, on the other hand, davinci always generate response to its token limits even when it is unsure about the generated content.

Advantage of finetuned for instruction models:

- Improved accuracy in generation, task specific.
- More cautiousness and less hallucination
- Structured and organized content

Disadvantage:

- Less creativity 



# 6. Acknowledgment of AI Tools

If you used ChatGPT or another AI to write any portion of your answers, please use this section to describe the prompts you employed and your methodology for developing them. You do not need to write anything here if you only used LLMs to run experiments as specified in the homework problem instructions.

- For the tasks from question 3 and question 4, when I try to run the experiment, I tried the same approach with ChatGPT to see how it would perform, and the result is pretty good and I can clearly see the performance gap between the well-trained LLM product and the smaller models we are using.

# 7. Optional: Give us Feedback
Was this homework enjoyable? Was it too easy or too hard? Do you have any suggestions for making the homework run more smoothly? Giving us feedback is compeltely optional and will not factor into your grade.
