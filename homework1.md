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
1. For two sets of generations, discuss the impact choice of decoding strategy has on diversity in the generated stories. Compute a lexical diversity metric such as type-token ratio (the total number of unique words divided by the total number of words) to support your answer.
2. Regarding your second prompt, did the language model generate the correct continuation of the book/speech? Provide reasoning as to why it may or may not have done so.
3. When `top_p`=0, does adjusting the `frequency_penalty` increase the lexical diversity of the stories? Explain why or why not this is the case.

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
1. Add a typo to each line of the poem. Does the perplexity go up or down? Give a reason for why this might have happened.
2. Shuffle the order of the stanzas in the poem. Does the perplexity go up or down? Give a reason for why this might have happened.
3. Taking inspiration from the concept of [mimic poems](https://penandthepad.com/directions-writing-mimic-poem-3668.html), swap several of the words in your poem to alternative words which still make sense in context. Does the perplexity go up or down?  ([Here](https://www.teenink.com/poetry/all/article/13900/Oh-Homework-Oh-Homework) is an example of a mimic poem of Walt Whitman's ["O Captain! My Captain!"](https://www.poetrysoup.com/famous/poem/o_captain!_my_captain!_198), though you don't need to do anything this complex or fancy.) Give a reason for why this might have happened.
3. Famous poems like the one you are using are very likely to be in GPT-3's training set. How might this affect the poem's perplexity compared to a very new poem 
which is not yet in any training set? 

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
2. What prompting formats did you experiment with? What worked well and what didn’t work?
3. What factors do you think most affect the model's performance?
4. For your best prompt, perform an error analysis of the test set examples it gets wrong. Qualitivaly, do you notice any patterns in the examples the model fails to classify correctly? 
5. Try our your best few-shot prompt from Question with the three smaller model sizes: ``curie``, ``babbage`` and ``ada``. How much does test set accuracy degrade on the smaller models? What is the smallest model size you can get away with and have good accuracy?

## 3.2 Few-shot Learning for Generation Tasks
Few-shot learning techniques can also be used fo tasks that require generation.
Choose one of the following sentence manipulation tasks, and try to write a few-shot prompt to get the model to do the task.

a. Convert a sentence to [pig latin](https://www.wikihow.com/Speak-Pig-Latin).
b. Add a space between each character in the sentence.
c. Apply a specified Caesar cipher[https://en.wikipedia.org/wiki/Caesar_cipher]

**Analysis Questions**
1. What few-shot prompt did the pick and how did you decide on it?
2. The three tasks listed above all require character-level manipulations. Why might such tasks be challenging for many modern large language models?


# 4. Investigating Knowledge Across Different Model Sizes

Pick a Wikipedia article on a person or place of your choice, and write a prompt containing the start of the first sentence in the article.
You may choose to omit the pronunciation and other parenthetical details).
For example, if you choose [Andrew Carnegie](https://en.wikipedia.org/wiki/Andrew_Carnegie), you should prompt with either `Andrew Carnegie (Scots: [kɑrˈnɛːɡi], English: /kɑːrˈnɛɡi/ kar-NEG-ee;[2][3][note 1] November 25, 1835 – August 11, 1919) was` or `Andrew Carnegie was an` or similar.
Use this prompt with both `davinci` (the largest version of GPT-3) and `ada` (the smallest version of GPT-3), each time generating 300 tokens.

**Analysis Questions**

1. Paste each generation into your report, and use the Wikipedia article as well as any knowledge you may have of the person/place to fact-check both generations.
Highlight any parts of the generation that are incorrect or contradictory.
For example, if the model generates `Andrew Carnegie was an American composer whose music has been described as America’s great signature.` then you should highlight this sentence.
2. What are some trends you notice about the kinds of errors found in the two model sizes?
3. Experiment with modifying your prompt to improve model accuracy. What extra information can you put in the prompt to make the model do better?
4. Create a sentence-long Wikipedia-sounding prompt that is completely wrong or else about a fictional person or place, for example `Bruce Lee was the 44th president of the United States from 2009 to 2017`. Prompt both model sizes with the sentence. Do both models write continuations in the same style?
5. Discuss the challenges in building an LLM that can simultaneously respond well to factual prompts as well as fantastical ones.  What kind of training data do you think, if initially given to the model at training time, would have made it better at supporting both use cases? 

# 5. Comparing Pre-Trained and Fine-tuned Models

The model ``text-davinci-003`` is a variant of GPT-3 which was finetuned for instruction following, using the methods described in the paper ["Training language models to follow instructions with human feedback"](https://arxiv.org/abs/2203.02155). Experiment with writing prompts for the following tasks using both models.

- Writing a recipe for a food of your choice.
- Explaining the rules for a sport or game of your choice.
- Continuing a Wikipedia article of your choice, conditioned on the first sentence in the article. (You may also choose to re-use your fantastical prompt from Question 4.)

**Analysis Questions**
1. Provide the prompts you decided on and their generations.
2. What do you notice about the differences in the behaviour between the two models? Summarize the advantages and disadvantages of using a model finetuned for instruction following.

# 6. Acknowledgment of AI Tools

If you used ChatGPT or another AI to write any portion of your answers, please use this section to describe the prompts you employed and your methodology for developing them. You do not need to write anything here if you only used LLMs to run experiments as specified in the homework problem instructions.

# 7. Optional: Give us Feedback
Was this homework enjoyable? Was it too easy or too hard? Do you have any suggestions for making the homework run more smoothly? Giving us feedback is compeltely optional and will not factor into your grade.
