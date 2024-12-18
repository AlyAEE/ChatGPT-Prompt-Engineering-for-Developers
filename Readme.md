# ChatGPT Prompt Engineering for Developers

## 1.Guidelines for Prompting
In this lesson, you'll practice two prompting principles and their related tactics in order to write effective prompts for large language models.

### Setup
#### Load the API key and relevant Python libaries.

In this course, we've provided some code that loads the OpenAI API key for you.


```python
import openai
import os

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())

openai.api_key  = os.getenv('OPENAI_API_KEY')
```

#### helper function
Throughout this course, we will use OpenAI's `gpt-3.5-turbo` model and the [chat completions endpoint](https://platform.openai.com/docs/guides/chat). 

This helper function will make it easier to use prompts and look at the generated outputs.  
**Note**: In June 2023, OpenAI updated gpt-3.5-turbo. The results you see in the notebook may be slightly different than those in the video. Some of the prompts have also been slightly modified to product the desired results.


```python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]
```

**Note:** This and all other lab notebooks of this course use OpenAI library version `0.27.0`. 

In order to use the OpenAI library version `1.0.0`, here is the code that you would use instead for the `get_completion` function:

```python
client = openai.OpenAI()

def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = client.chat.completions.create(
        model=model,
        messages=messages,
        temperature=0
    )
    return response.choices[0].message.content
```

### Prompting Principles
- **Principle 1: Write clear and specific instructions**
- **Principle 2: Give the model time to “think”**

### Tactics

#### Tactic 1: Use delimiters to clearly indicate distinct parts of the input
- Delimiters can be anything like: ```, """, < >, `<tag> </tag>`, `:`
- Delimiters can be any kind of clear punctuation that separates specific text from the rest of the prompt

```python
text = f"""
You should express what you want a model to do by \ 
providing instructions that are as clear and \ 
specific as you can possibly make them. \ 
This will guide the model towards the desired output, \ 
and reduce the chances of receiving irrelevant \ 
or incorrect responses. Don't confuse writing a \ 
clear prompt with writing a short prompt. \ 
In many cases, longer prompts provide more clarity \ 
and context for the model, which can lead to \ 
more detailed and relevant outputs.
"""
prompt = f"""
Summarize the text delimited by triple backticks \ 
into a single sentence.
```{text}```
"""
response = get_completion(prompt)
print(response)
```

    Providing clear and specific instructions to a model is essential for guiding it towards the desired output and reducing the chances of irrelevant or incorrect responses, with longer prompts often providing more clarity and context for more detailed and relevant outputs.

- Using Delimiters is also a good technique to try avoid prompt injections, A prompt injection is if user is allowed to put some input into your prompt, they might give kinfd of conflicting instructions to the model.
![alt text](image.png)


#### Tactic 2: Ask for a structured output
- JSON, HTML


```python
prompt = f"""
Generate a list of three made-up book titles along \ 
with their authors and genres. 
Provide them in JSON format with the following keys: 
book_id, title, author, genre.
"""
response = get_completion(prompt)
print(response)
```

    [
        {
            "book_id": 1,
            "title": "The Midnight Garden",
            "author": "Elena Rivers",
            "genre": "Fantasy"
        },
        {
            "book_id": 2,
            "title": "Echoes of the Past",
            "author": "Nathan Black",
            "genre": "Mystery"
        },
        {
            "book_id": 3,
            "title": "Whispers in the Wind",
            "author": "Samantha Reed",
            "genre": "Romance"
        }
    ]


#### Tactic 3: Ask the model to check whether conditions are satisfied
- You might also consider potential edge cases and how the model should handle them to avoid unexpected errors or result.

```python
text_1 = f"""
Making a cup of tea is easy! First, you need to get some \ 
water boiling. While that's happening, \ 
grab a cup and put a tea bag in it. Once the water is \ 
hot enough, just pour it over the tea bag. \ 
Let it sit for a bit so the tea can steep. After a \ 
few minutes, take out the tea bag. If you \ 
like, you can add some sugar or milk to taste. \ 
And that's it! You've got yourself a delicious \ 
cup of tea to enjoy.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \ 
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \ 
then simply write \"No steps provided.\"

\"\"\"{text_1}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 1:")
print(response)
```

    Completion for Text 1:
    Step 1 - Get some water boiling.
    Step 2 - Grab a cup and put a tea bag in it.
    Step 3 - Pour the hot water over the tea bag.
    Step 4 - Let the tea steep for a few minutes.
    Step 5 - Remove the tea bag.
    Step 6 - Add sugar or milk to taste.
    Step 7 - Enjoy your delicious cup of tea.



```python
text_2 = f"""
The sun is shining brightly today, and the birds are \
singing. It's a beautiful day to go for a \ 
walk in the park. The flowers are blooming, and the \ 
trees are swaying gently in the breeze. People \ 
are out and about, enjoying the lovely weather. \ 
Some are having picnics, while others are playing \ 
games or simply relaxing on the grass. It's a \ 
perfect day to spend time outdoors and appreciate the \ 
beauty of nature.
"""
prompt = f"""
You will be provided with text delimited by triple quotes. 
If it contains a sequence of instructions, \ 
re-write those instructions in the following format:

Step 1 - ...
Step 2 - …
…
Step N - …

If the text does not contain a sequence of instructions, \ 
then simply write \"No steps provided.\"

\"\"\"{text_2}\"\"\"
"""
response = get_completion(prompt)
print("Completion for Text 2:")
print(response)
```

    Completion for Text 2:
    No steps provided.


#### Tactic 4: "Few-shot" prompting
- Give successful examples of completing tasks, Then ask model to perform the task.

```python
prompt = f"""
Your task is to answer in a consistent style.

<child>: Teach me about patience.

<grandparent>: The river that carves the deepest \ 
valley flows from a modest spring; the \ 
grandest symphony originates from a single note; \ 
the most intricate tapestry begins with a solitary thread.

<child>: Teach me about resilience.
"""
response = get_completion(prompt)
print(response)
```

    <grandparent>: The tallest trees weather the strongest storms; the brightest stars shine in the darkest nights; the strongest hearts are forged in the hottest fires.


### Principle 2: Give the model time to “think” 
- If a model is making reasoning errors by rushing to an incorrect conclusion, You should try reframing the query to request a chain or series of relevant reasoning before the model provides its final answer.

#### Tactic 1: Specify the steps required to complete a task

```python
text = f"""
In a charming village, siblings Jack and Jill set out on \ 
a quest to fetch water from a hilltop \ 
well. As they climbed, singing joyfully, misfortune \ 
struck—Jack tripped on a stone and tumbled \ 
down the hill, with Jill following suit. \ 
Though slightly battered, the pair returned home to \ 
comforting embraces. Despite the mishap, \ 
their adventurous spirits remained undimmed, and they \ 
continued exploring with delight.
"""
# example 1
prompt_1 = f"""
Perform the following actions: 
1 - Summarize the following text delimited by triple \
backticks with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the following \
keys: french_summary, num_names.

Separate your answers with line breaks.

Text:
```{text}```
"""
response = get_completion(prompt_1)
print("Completion for prompt 1:")
print(response)
```

    Completion for prompt 1:
    1 - Jack and Jill, siblings from a charming village, go on a quest to fetch water from a hilltop well, but encounter misfortune along the way.
    
    2 - Jack et Jill, frère et sœur d'un charmant village, partent en quête d'eau d'un puits au sommet d'une colline, mais rencontrent des malheurs en chemin.
    
    3 - Jack, Jill
    
    4 - 
    {
      "french_summary": "Jack et Jill, frère et sœur d'un charmant village, partent en quête d'eau d'un puits au sommet d'une colline, mais rencontrent des malheurs en chemin.",
      "num_names": 2
    }


#### Ask for output in a specified format


```python
prompt_2 = f"""
Your task is to perform the following actions: 
1 - Summarize the following text delimited by 
  <> with 1 sentence.
2 - Translate the summary into French.
3 - List each name in the French summary.
4 - Output a json object that contains the 
  following keys: french_summary, num_names.

Use the following format:
Text: <text to summarize>
Summary: <summary>
Translation: <summary translation>
Names: <list of names in summary>
Output JSON: <json with summary and num_names>

Text: <{text}>
"""
response = get_completion(prompt_2)
print("\nCompletion for prompt 2:")
print(response)
```

    
    Completion for prompt 2:
    Summary: Jack and Jill, siblings, go on a quest to fetch water from a hilltop well, but encounter misfortune along the way.
    
    Translation: Jack et Jill, frère et sœur, partent en quête d'eau d'un puits au sommet d'une colline, mais rencontrent des malheurs en chemin.
    
    Names: Jack, Jill
    
    Output JSON: 
    {
      "french_summary": "Jack et Jill, frère et sœur, partent en quête d'eau d'un puits au sommet d'une colline, mais rencontrent des malheurs en chemin.",
      "num_names": 2
    }


#### Tactic 2: Instruct the model to work out its own solution before rushing to a conclusion


```python
prompt = f"""
Determine if the student's solution is correct or not.

Question:
I'm building a solar power installation and I need \
 help working out the financials. 
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \ 
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations 
as a function of the number of square feet.

Student's Solution:
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
"""
response = get_completion(prompt)
print(response)
```

    The student's solution is correct. The total cost for the first year of operations as a function of the number of square feet is indeed 450x + 100,000.


#### Note that the student's solution is actually not correct.
#### We can fix this by instructing the model to work out its own solution first.


```python
prompt = f"""
Your task is to determine if the student's solution \
is correct or not.
To solve the problem do the following:
- First, work out your own solution to the problem including the final total. 
- Then compare your solution to the student's solution \ 
and evaluate if the student's solution is correct or not. 
Don't decide if the student's solution is correct until 
you have done the problem yourself.

Use the following format:
Question:
```
question here
```
Student's solution:
```
student's solution here
```
Actual solution:
```
steps to work out the solution and your solution here
```
Is the student's solution the same as actual solution \
just calculated:
```
yes or no
```
Student grade:
```
correct or incorrect
```

Question:
```
I'm building a solar power installation and I need help \
working out the financials. 
- Land costs $100 / square foot
- I can buy solar panels for $250 / square foot
- I negotiated a contract for maintenance that will cost \
me a flat $100k per year, and an additional $10 / square \
foot
What is the total cost for the first year of operations \
as a function of the number of square feet.
``` 
Student's solution:
```
Let x be the size of the installation in square feet.
Costs:
1. Land cost: 100x
2. Solar panel cost: 250x
3. Maintenance cost: 100,000 + 100x
Total cost: 100x + 250x + 100,000 + 100x = 450x + 100,000
```
Actual solution:
"""
response = get_completion(prompt)
print(response)
```

    Let x be the size of the installation in square feet.
    
    Costs:
    1. Land cost: $100 * x
    2. Solar panel cost: $250 * x
    3. Maintenance cost: $100,000 + $10 * x
    
    Total cost: $100 * x + $250 * x + $100,000 + $10 * x = $360 * x + $100,000
    ```
    The total cost for the first year of operations as a function of the number of square feet is $360 * x + $100,000.
    ```
    Is the student's solution the same as actual solution just calculated:
    ```
    Yes
    ```
    Student grade:
    ```
    correct
    ```


### Model Limitations: Hallucinations
- It can make statements that sound plausible but are not true.


- Boie is a real company, the product name is not real.

```python
prompt = f"""
Tell me about AeroGlide UltraSlim Smart Toothbrush by Boie
"""
response = get_completion(prompt)
print(response)
```

    The AeroGlide UltraSlim Smart Toothbrush by Boie is a high-tech toothbrush designed to provide a superior cleaning experience. It features ultra-soft bristles that are gentle on the gums and teeth, while still effectively removing plaque and debris. The toothbrush also has a slim design that makes it easy to maneuver and reach all areas of the mouth.
    
    One of the standout features of the AeroGlide UltraSlim Smart Toothbrush is its smart technology. It connects to a mobile app that tracks your brushing habits and provides personalized recommendations for improving your oral hygiene routine. The app also includes a timer to ensure you are brushing for the recommended two minutes.
    
    The toothbrush is made from durable, antimicrobial materials that resist bacteria growth and can be easily cleaned and sanitized. It is also eco-friendly, as the brush head is replaceable and the handle is made from recyclable materials.
    
    Overall, the AeroGlide UltraSlim Smart Toothbrush by Boie is a sleek and innovative toothbrush that offers a thorough and personalized cleaning experience for users.

- One additional tactic to reduce hallucinations is to ask the model to first find any relevant information about the text, then answer the question based on the relevant information.

#### Notes on using the OpenAI API outside of this classroom

To install the OpenAI Python library:
```
!pip install openai
```

The library needs to be configured with your account's secret key, which is available on the [website](https://platform.openai.com/account/api-keys). 

You can either set it as the `OPENAI_API_KEY` environment variable before using the library:
 ```
 !export OPENAI_API_KEY='sk-...'
 ```

Or, set `openai.api_key` to its value:

```
import openai
openai.api_key = "sk-..."
```

#### A note about the backslash
- In the course, we are using a backslash `\` to make the text fit on the screen without inserting newline '\n' characters.
- GPT-3 isn't really affected whether you insert newline characters or not.  But when working with LLMs in general, you may consider whether newline characters in your prompt may affect the model's performance.

## 2.Iterative Prompt Development
In this lesson, you'll iteratively analyze and refine your prompts to generate marketing copy from a product fact sheet.
![alt text](image-1.png)

### Generate a marketing product description from a product fact sheet


```python
fact_sheet_chair = """
OVERVIEW
- Part of a beautiful family of mid-century inspired office furniture, 
including filing cabinets, desks, bookcases, meeting tables, and more.
- Several options of shell color and base finishes.
- Available with plastic back and front upholstery (SWC-100) 
or full upholstery (SWC-110) in 10 fabric and 6 leather options.
- Base finish options are: stainless steel, matte black, 
gloss white, or chrome.
- Chair is available with or without armrests.
- Suitable for home or business settings.
- Qualified for contract use.

CONSTRUCTION
- 5-wheel plastic coated aluminum base.
- Pneumatic chair adjust for easy raise/lower action.

DIMENSIONS
- WIDTH 53 CM | 20.87”
- DEPTH 51 CM | 20.08”
- HEIGHT 80 CM | 31.50”
- SEAT HEIGHT 44 CM | 17.32”
- SEAT DEPTH 41 CM | 16.14”

OPTIONS
- Soft or hard-floor caster options.
- Two choices of seat foam densities: 
 medium (1.8 lb/ft3) or high (2.8 lb/ft3)
- Armless or 8 position PU armrests 

MATERIALS
SHELL BASE GLIDER
- Cast Aluminum with modified nylon PA6/PA66 coating.
- Shell thickness: 10 mm.
SEAT
- HD36 foam

COUNTRY OF ORIGIN
- Italy
"""
```


```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Technical specifications: ```{fact_sheet_chair}```
"""
response = get_completion(prompt)
print(response)

```

    Introducing our stylish and versatile mid-century inspired office chair, perfect for both home and business settings. This chair is part of a beautiful family of office furniture that includes filing cabinets, desks, bookcases, meeting tables, and more.
    
    Customize your chair with several options of shell color and base finishes to suit your personal style. Choose between plastic back and front upholstery or full upholstery in a variety of fabric and leather options. The base finish options include stainless steel, matte black, gloss white, or chrome. You can also choose to have armrests or go for a sleek armless design.
    
    Constructed with a 5-wheel plastic coated aluminum base, this chair features a pneumatic chair adjust for easy raise/lower action. The dimensions of the chair are as follows: width 53 cm, depth 51 cm, height 80 cm, seat height 44 cm, and seat depth 41 cm.
    
    Personalize your chair even further with options such as soft or hard-floor caster options, two choices of seat foam densities (medium or high), and the choice between armless or 8 position PU armrests.
    
    Made with high-quality materials, including a cast aluminum shell with modified nylon coating and HD36 foam seat, this chair is designed for durability and comfort. Plus, it is qualified for contract use, making it a reliable choice for any workspace.
    
    Add a touch of Italian elegance to your office with this stylish and functional office chair. Elevate your workspace with our mid-century inspired chair that combines style, comfort, and functionality seamlessly.


### Issue 1: The text is too long 
- Limit the number of words/sentences/characters.


```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""
response = get_completion(prompt)
print(response)

```

    Introducing our versatile and stylish mid-century office chair, available in a range of colors and finishes. With adjustable height and comfortable upholstery options, this chair is perfect for both home and business use. Made in Italy with quality materials, it's a perfect blend of form and function.



```python
len(response.split())
```




    47



### Issue 2. Text focuses on the wrong details
- Ask it to focus on the aspects that are relevant to the intended audience.


```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""
response = get_completion(prompt)
print(response)
```

    Introducing our versatile and stylish office chair, part of a mid-century inspired furniture collection. Choose from a variety of shell colors and base finishes to suit your space. Constructed with durable materials like cast aluminum and HD36 foam, this chair is perfect for both home and business settings. Made in Italy.



```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

At the end of the description, include every 7-character 
Product ID in the technical specification.

Use at most 50 words.

Technical specifications: ```{fact_sheet_chair}```
"""
response = get_completion(prompt)
print(response)
```

    Introducing our versatile and stylish office chair, part of a mid-century inspired furniture collection. Choose from a variety of shell colors and base finishes to suit your space. Constructed with a durable aluminum base and high-density foam seat for comfort. Perfect for home or business use. 
    
    Product IDs: SWC-100, SWC-110


### Issue 3. Description needs a table of dimensions
- Ask it to extract information and organize it in a table.


```python
prompt = f"""
Your task is to help a marketing team create a 
description for a retail website of a product based 
on a technical fact sheet.

Write a product description based on the information 
provided in the technical specifications delimited by 
triple backticks.

The description is intended for furniture retailers, 
so should be technical in nature and focus on the 
materials the product is constructed from.

At the end of the description, include every 7-character 
Product ID in the technical specification.

After the description, include a table that gives the 
product's dimensions. The table should have two columns.
In the first column include the name of the dimension. 
In the second column include the measurements in inches only.

Give the table the title 'Product Dimensions'.

Format everything as HTML that can be used in a website. 
Place the description in a <div> element.

Technical specifications: ```{fact_sheet_chair}```
"""

response = get_completion(prompt)
print(response)
```

    <div>
      <p>Introducing our latest addition to the mid-century inspired office furniture collection - the versatile and stylish office chair. This chair is designed to elevate any workspace, whether it be at home or in a business setting. With a variety of shell colors and base finishes to choose from, you can customize this chair to fit your aesthetic perfectly.</p>
      
      <p>The construction of this chair is top-notch, featuring a 5-wheel plastic coated aluminum base for stability and mobility. The pneumatic chair adjust allows for easy raise and lower action, ensuring maximum comfort throughout the day.</p>
      
      <p>Constructed with high-quality materials, the shell base glider is made of cast aluminum with a modified nylon PA6/PA66 coating, providing durability and style. The seat is filled with HD36 foam, offering a comfortable seating experience.</p>
      
      <p>Product ID: SWC-100</p>
      <p>Product Dimensions:</p>
      <table>
        <tr>
          <td>WIDTH</td>
          <td>20.87"</td>
        </tr>
        <tr>
          <td>DEPTH</td>
          <td>20.08"</td>
        </tr>
        <tr>
          <td>HEIGHT</td>
          <td>31.50"</td>
        </tr>
        <tr>
          <td>SEAT HEIGHT</td>
          <td>17.32"</td>
        </tr>
        <tr>
          <td>SEAT DEPTH</td>
          <td>16.14"</td>
        </tr>
      </table>
    </div>


### Load Python libraries to view HTML


```python
from IPython.display import display, HTML
```


```python
display(HTML(response))
```


<div>
  <p>Introducing our latest addition to the mid-century inspired office furniture collection - the versatile and stylish office chair. This chair is designed to elevate any workspace, whether it be at home or in a business setting. With a variety of shell colors and base finishes to choose from, you can customize this chair to fit your aesthetic perfectly.</p>

  <p>The construction of this chair is top-notch, featuring a 5-wheel plastic coated aluminum base for stability and mobility. The pneumatic chair adjust allows for easy raise and lower action, ensuring maximum comfort throughout the day.</p>

  <p>Constructed with high-quality materials, the shell base glider is made of cast aluminum with a modified nylon PA6/PA66 coating, providing durability and style. The seat is filled with HD36 foam, offering a comfortable seating experience.</p>

  <p>Product ID: SWC-100</p>
  <p>Product Dimensions:</p>
  <table>
    <tr>
      <td>WIDTH</td>
      <td>20.87"</td>
    </tr>
    <tr>
      <td>DEPTH</td>
      <td>20.08"</td>
    </tr>
    <tr>
      <td>HEIGHT</td>
      <td>31.50"</td>
    </tr>
    <tr>
      <td>SEAT HEIGHT</td>
      <td>17.32"</td>
    </tr>
    <tr>
      <td>SEAT DEPTH</td>
      <td>16.14"</td>
    </tr>
  </table>
</div>

![alt text](image-2.png)
## 3.Summarizing
In this lesson, you will summarize text with a focus on specific topics.


### Text to summarize

```python
prod_review = """
Got this panda plush toy for my daughter's birthday, \
who loves it and takes it everywhere. It's soft and \ 
super cute, and its face has a friendly look. It's \ 
a bit small for what I paid though. I think there \ 
might be other options that are bigger for the \ 
same price. It arrived a day earlier than expected, \ 
so I got to play with it myself before I gave it \ 
to her.
"""
```

### Summarize with a word/sentence/character limit


```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site. 

Summarize the review below, delimited by triple 
backticks, in at most 30 words. 

Review: ```{prod_review}```
"""

response = get_completion(prompt)
print(response)

```

    Soft, cute panda plush toy loved by daughter, but smaller than expected for the price. Arrived early, friendly face.


### Summarize with a focus on shipping and delivery


```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site to give feedback to the \
Shipping deparmtment. 

Summarize the review below, delimited by triple 
backticks, in at most 30 words, and focusing on any aspects \
that mention shipping and delivery of the product. 

Review: ```{prod_review}```
"""

response = get_completion(prompt)
print(response)

```

    The customer was pleased with the early delivery of the panda plush toy, but felt it was slightly small for the price paid.


### Summarize with a focus on price and value


```python
prompt = f"""
Your task is to generate a short summary of a product \
review from an ecommerce site to give feedback to the \
pricing deparmtment, responsible for determining the \
price of the product.  

Summarize the review below, delimited by triple 
backticks, in at most 30 words, and focusing on any aspects \
that are relevant to the price and perceived value. 

Review: ```{prod_review}```
"""

response = get_completion(prompt)
print(response)

```

    The panda plush toy is loved for its softness and cuteness, but some customers feel it's a bit small for the price.


#### Comment
- Summaries include topics that are not related to the topic of focus.

### Try "extract" instead of "summarize"


```python
prompt = f"""
Your task is to extract relevant information from \ 
a product review from an ecommerce site to give \
feedback to the Shipping department. 

From the review below, delimited by triple quotes \
extract the information relevant to shipping and \ 
delivery. Limit to 30 words. 

Review: ```{prod_review}```
"""

response = get_completion(prompt)
print(response)
```

    Feedback: The product arrived a day earlier than expected, which was a pleasant surprise. Consider offering larger options for the same price to improve customer satisfaction.


### Summarize multiple product reviews


```python

review_1 = prod_review 

# review for a standing lamp
review_2 = """
Needed a nice lamp for my bedroom, and this one \
had additional storage and not too high of a price \
point. Got it fast - arrived in 2 days. The string \
to the lamp broke during the transit and the company \
happily sent over a new one. Came within a few days \
as well. It was easy to put together. Then I had a \
missing part, so I contacted their support and they \
very quickly got me the missing piece! Seems to me \
to be a great company that cares about their customers \
and products. 
"""

# review for an electric toothbrush
review_3 = """
My dental hygienist recommended an electric toothbrush, \
which is why I got this. The battery life seems to be \
pretty impressive so far. After initial charging and \
leaving the charger plugged in for the first week to \
condition the battery, I've unplugged the charger and \
been using it for twice daily brushing for the last \
3 weeks all on the same charge. But the toothbrush head \
is too small. I’ve seen baby toothbrushes bigger than \
this one. I wish the head was bigger with different \
length bristles to get between teeth better because \
this one doesn’t.  Overall if you can get this one \
around the $50 mark, it's a good deal. The manufactuer's \
replacements heads are pretty expensive, but you can \
get generic ones that're more reasonably priced. This \
toothbrush makes me feel like I've been to the dentist \
every day. My teeth feel sparkly clean! 
"""

# review for a blender
review_4 = """
So, they still had the 17 piece system on seasonal \
sale for around $49 in the month of November, about \
half off, but for some reason (call it price gouging) \
around the second week of December the prices all went \
up to about anywhere from between $70-$89 for the same \
system. And the 11 piece system went up around $10 or \
so in price also from the earlier sale price of $29. \
So it looks okay, but if you look at the base, the part \
where the blade locks into place doesn’t look as good \
as in previous editions from a few years ago, but I \
plan to be very gentle with it (example, I crush \
very hard items like beans, ice, rice, etc. in the \ 
blender first then pulverize them in the serving size \
I want in the blender then switch to the whipping \
blade for a finer flour, and use the cross cutting blade \
first when making smoothies, then use the flat blade \
if I need them finer/less pulpy). Special tip when making \
smoothies, finely cut and freeze the fruits and \
vegetables (if using spinach-lightly stew soften the \ 
spinach then freeze until ready for use-and if making \
sorbet, use a small to medium sized food processor) \ 
that you plan to use that way you can avoid adding so \
much ice if at all-when making your smoothie. \
After about a year, the motor was making a funny noise. \
I called customer service but the warranty expired \
already, so I had to buy another one. FYI: The overall \
quality has gone done in these types of products, so \
they are kind of counting on brand recognition and \
consumer loyalty to maintain sales. Got it in about \
two days.
"""

reviews = [review_1, review_2, review_3, review_4]


```


```python
for i in range(len(reviews)):
    prompt = f"""
    Your task is to generate a short summary of a product \ 
    review from an ecommerce site. 

    Summarize the review below, delimited by triple \
    backticks in at most 20 words. 

    Review: ```{reviews[i]}```
    """

    response = get_completion(prompt)
    print(i, response, "\n")

```

    0 Summary: 
    Adorable panda plush loved by daughter, but small for price. Arrived early, soft and cute. 
    
    1 Great lamp with storage, fast delivery, excellent customer service for missing parts. Company cares about customers. 
    
    2 Impressive battery life, small brush head, good deal for $50, generic replacement heads available, leaves teeth feeling clean. 
    
    3 17-piece system on sale for $49, prices increased later. Base quality not as good, motor issues after a year. 
    
## 3.Inferring
In this lesson, you will infer sentiment and topics from product reviews and news articles.

### Product review text


```python
lamp_review = """
Needed a nice lamp for my bedroom, and this one had \
additional storage and not too high of a price point. \
Got it fast.  The string to our lamp broke during the \
transit and the company happily sent over a new one. \
Came within a few days as well. It was easy to put \
together.  I had a missing part, so I contacted their \
support and they very quickly got me the missing piece! \
Lumina seems to me to be a great company that cares \
about their customers and products!!
"""
```

### Sentiment (positive/negative)


```python
prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    The sentiment of the review is positive. The reviewer is satisfied with the lamp they purchased, mentioning the additional storage, reasonable price, fast delivery, good customer service, and ease of assembly. They also praise the company for caring about their customers and products.



```python
prompt = f"""
What is the sentiment of the following product review, 
which is delimited with triple backticks?

Give your answer as a single word, either "positive" \
or "negative".

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    Positive


### Identify types of emotions


```python
prompt = f"""
Identify a list of emotions that the writer of the \
following review is expressing. Include no more than \
five items in the list. Format your answer as a list of \
lower-case words separated by commas.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    happy, satisfied, grateful, impressed, content


### Identify anger


```python
prompt = f"""
Is the writer of the following review expressing anger?\
The review is delimited with triple backticks. \
Give your answer as either yes or no.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    No


### Extract product and company name from customer reviews


```python
prompt = f"""
Identify the following items from the review text: 
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Item" and "Brand" as the keys. 
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
  
Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    {
      "Item": "lamp",
      "Brand": "Lumina"
    }


### Doing multiple tasks at once


```python
prompt = f"""
Identify the following items from the review text: 
- Sentiment (positive or negative)
- Is the reviewer expressing anger? (true or false)
- Item purchased by reviewer
- Company that made the item

The review is delimited with triple backticks. \
Format your response as a JSON object with \
"Sentiment", "Anger", "Item" and "Brand" as the keys.
If the information isn't present, use "unknown" \
as the value.
Make your response as short as possible.
Format the Anger value as a boolean.

Review text: '''{lamp_review}'''
"""
response = get_completion(prompt)
print(response)
```

    {
        "Sentiment": "positive",
        "Anger": false,
        "Item": "lamp",
        "Brand": "Lumina"
    }


### Inferring topics


```python
story = """
In a recent survey conducted by the government, 
public sector employees were asked to rate their level 
of satisfaction with the department they work at. 
The results revealed that NASA was the most popular 
department with a satisfaction rating of 95%.

One NASA employee, John Smith, commented on the findings, 
stating, "I'm not surprised that NASA came out on top. 
It's a great place to work with amazing people and 
incredible opportunities. I'm proud to be a part of 
such an innovative organization."

The results were also welcomed by NASA's management team, 
with Director Tom Johnson stating, "We are thrilled to 
hear that our employees are satisfied with their work at NASA. 
We have a talented and dedicated team who work tirelessly 
to achieve our goals, and it's fantastic to see that their 
hard work is paying off."

The survey also revealed that the 
Social Security Administration had the lowest satisfaction 
rating, with only 45% of employees indicating they were 
satisfied with their job. The government has pledged to 
address the concerns raised by employees in the survey and 
work towards improving job satisfaction across all departments.
"""
```

### Infer 5 topics


```python
prompt = f"""
Determine five topics that are being discussed in the \
following text, which is delimited by triple backticks.

Make each item one or two words long. 

Format your response as a list of items separated by commas.

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```

    1. Survey
    2. Job satisfaction
    3. NASA
    4. Social Security Administration
    5. Government pledge



```python
response.split(sep=',')
```




    ['1. Survey\n2. Job satisfaction\n3. NASA\n4. Social Security Administration\n5. Government pledge']




```python
topic_list = [
    "nasa", "local government", "engineering", 
    "employee satisfaction", "federal government"
]
```

### Make a news alert for certain topics


```python
prompt = f"""
Determine whether each item in the following list of \
topics is a topic in the text below, which
is delimited with triple backticks.

Give your answer as follows:
item from the list: 0 or 1

List of topics: {", ".join(topic_list)}

Text sample: '''{story}'''
"""
response = get_completion(prompt)
print(response)
```

    nasa: 1
    local government: 0
    engineering: 0
    employee satisfaction: 1
    federal government: 1

- In machine learning, This is sometimes called a Zero Shot Learning , Because we didn't give it any training data that was labeled.

```python
topic_dict = {i.split(': ')[0]: int(i.split(': ')[1]) for i in response.split(sep='\n')}
if topic_dict['nasa'] == 1:
    print("ALERT: New NASA story!")
```

    ALERT: New NASA story!

## 4.Transforming

In this notebook, we will explore how to use Large Language Models for text transformation tasks such as language translation, spelling and grammar checking, tone adjustment, and format conversion.

### Translation

ChatGPT is trained with sources in many languages. This gives the model the ability to do translation. Here are some examples of how to use this capability.


```python
prompt = f"""
Translate the following English text to Spanish: \ 
```Hi, I would like to order a blender```
"""
response = get_completion(prompt)
print(response)
```

    Hola, me gustaría ordenar una licuadora.



```python
prompt = f"""
Tell me which language this is: 
```Combien coûte le lampadaire?```
"""
response = get_completion(prompt)
print(response)
```

    This is French.



```python
prompt = f"""
Translate the following  text to French and Spanish
and English pirate: \
```I want to order a basketball```
"""
response = get_completion(prompt)
print(response)
```

    French: Je veux commander un ballon de basket
    Spanish: Quiero ordenar un balón de baloncesto
    English: I want to order a basketball



```python
prompt = f"""
Translate the following text to Spanish in both the \
formal and informal forms: 
'Would you like to order a pillow?'
"""
response = get_completion(prompt)
print(response)
```

    Formal: ¿Le gustaría ordenar una almohada?
    Informal: ¿Te gustaría ordenar una almohada?


### Universal Translator
Imagine you are in charge of IT at a large multinational e-commerce company. Users are messaging you with IT issues in all their native languages. Your staff is from all over the world and speaks only their native languages. You need a universal translator!


```python
user_messages = [
  "La performance du système est plus lente que d'habitude.",  # System performance is slower than normal         
  "Mi monitor tiene píxeles que no se iluminan.",              # My monitor has pixels that are not lighting
  "Il mio mouse non funziona",                                 # My mouse is not working
  "Mój klawisz Ctrl jest zepsuty",                             # My keyboard has a broken control key
  "我的屏幕在闪烁"                                               # My screen is flashing
] 
```


```python
for issue in user_messages:
    prompt = f"Tell me what language this is: ```{issue}```"
    lang = get_completion(prompt)
    print(f"Original message ({lang}): {issue}")

    prompt = f"""
    Translate the following  text to English \
    and Korean: ```{issue}```
    """
    response = get_completion(prompt)
    print(response, "\n")
```

    Original message (This is French.): La performance du système est plus lente que d'habitude.
    English: "The system performance is slower than usual."
    
    Korean: "시스템 성능이 평소보다 느립니다." 
    
    Original message (This is Spanish.): Mi monitor tiene píxeles que no se iluminan.
    English: "My monitor has pixels that do not light up."
    
    Korean: "내 모니터에는 빛나지 않는 픽셀이 있습니다." 
    
    Original message (Italian): Il mio mouse non funziona
    English: My mouse is not working
    Korean: 내 마우스가 작동하지 않습니다 
    
    Original message (Polish): Mój klawisz Ctrl jest zepsuty
    English: My Ctrl key is broken
    Korean: 제 Ctrl 키가 고장 났어요 
    
    Original message (This is Chinese.): 我的屏幕在闪烁
    English: My screen is flickering
    Korean: 내 화면이 깜박거립니다 
    

### Tone Transformation
Writing can vary based on the intended audience. ChatGPT can produce different tones.



```python
prompt = f"""
Translate the following from slang to a business letter: 
'Dude, This is Joe, check out this spec on this standing lamp.'
"""
response = get_completion(prompt)
print(response)
```

    Dear Sir/Madam,
    
    I am writing to bring to your attention the specifications of a standing lamp that I believe may be of interest to you. 
    
    Sincerely,
    Joe


### Format Conversion
ChatGPT can translate between formats. The prompt should describe the input and output formats.


```python
data_json = { "resturant employees" :[ 
    {"name":"Shyam", "email":"shyamjaiswal@gmail.com"},
    {"name":"Bob", "email":"bob32@gmail.com"},
    {"name":"Jai", "email":"jai87@gmail.com"}
]}

prompt = f"""
Translate the following python dictionary from JSON to an HTML \
table with column headers and title: {data_json}
"""
response = get_completion(prompt)
print(response)
```

    <html>
    <head>
        <title>Restaurant Employees</title>
    </head>
    <body>
        <table>
            <tr>
                <th>Name</th>
                <th>Email</th>
            </tr>
            <tr>
                <td>Shyam</td>
                <td>shyamjaiswal@gmail.com</td>
            </tr>
            <tr>
                <td>Bob</td>
                <td>bob32@gmail.com</td>
            </tr>
            <tr>
                <td>Jai</td>
                <td>jai87@gmail.com</td>
            </tr>
        </table>
    </body>
    </html>



```python
from IPython.display import display, Markdown, Latex, HTML, JSON
display(HTML(response))
```


<html>
<head>
    <title>Restaurant Employees</title>
</head>
<body>
    <table>
        <tr>
            <th>Name</th>
            <th>Email</th>
        </tr>
        <tr>
            <td>Shyam</td>
            <td>shyamjaiswal@gmail.com</td>
        </tr>
        <tr>
            <td>Bob</td>
            <td>bob32@gmail.com</td>
        </tr>
        <tr>
            <td>Jai</td>
            <td>jai87@gmail.com</td>
        </tr>
    </table>
</body>
</html>


### Spellcheck/Grammar check.

Here are some examples of common grammar and spelling problems and the LLM's response. 

To signal to the LLM that you want it to proofread your text, you instruct the model to 'proofread' or 'proofread and correct'.


```python
text = [ 
  "The girl with the black and white puppies have a ball.",  # The girl has a ball.
  "Yolanda has her notebook.", # ok
  "Its going to be a long day. Does the car need it’s oil changed?",  # Homonyms
  "Their goes my freedom. There going to bring they’re suitcases.",  # Homonyms
  "Your going to need you’re notebook.",  # Homonyms
  "That medicine effects my ability to sleep. Have you heard of the butterfly affect?", # Homonyms
  "This phrase is to cherck chatGPT for speling abilitty"  # spelling
]
for t in text:
    prompt = f"""Proofread and correct the following text
    and rewrite the corrected version. If you don't find
    and errors, just say "No errors found". Don't use 
    any punctuation around the text:
    ```{t}```"""
    response = get_completion(prompt)
    print(response)
```

    The girl with the black and white puppies has a ball.
    No errors found
    No errors found.
    Their goes my freedom. There going to bring they’re suitcases.
    
    No errors found.
    
    Rewritten:
    Their goes my freedom. There going to bring their suitcases.
    You're going to need your notebook.
    No errors found.
    No errors found



```python
text = f"""
Got this for my daughter for her birthday cuz she keeps taking \
mine from my room.  Yes, adults also like pandas too.  She takes \
it everywhere with her, and it's super soft and cute.  One of the \
ears is a bit lower than the other, and I don't think that was \
designed to be asymmetrical. It's a bit small for what I paid for it \
though. I think there might be other options that are bigger for \
the same price.  It arrived a day earlier than expected, so I got \
to play with it myself before I gave it to my daughter.
"""
prompt = f"proofread and correct this review: ```{text}```"
response = get_completion(prompt)
print(response)
```

    I got this for my daughter for her birthday because she keeps taking mine from my room. Yes, adults also like pandas too. She takes it everywhere with her, and it's super soft and cute. One of the ears is a bit lower than the other, and I don't think that was designed to be asymmetrical. It's a bit small for what I paid for it though. I think there might be other options that are bigger for the same price. It arrived a day earlier than expected, so I got to play with it myself before I gave it to my daughter.



```python
from redlines import Redlines

diff = Redlines(text,response)
display(Markdown(diff.output_markdown))
```


<span style="color:red;font-weight:700;text-decoration:line-through;">Got </span><span style="color:red;font-weight:700;">I got </span>this for my daughter for her birthday <span style="color:red;font-weight:700;text-decoration:line-through;">cuz </span><span style="color:red;font-weight:700;">because </span>she keeps taking mine from my <span style="color:red;font-weight:700;text-decoration:line-through;">room.  </span><span style="color:red;font-weight:700;">room. </span>Yes, adults also like pandas <span style="color:red;font-weight:700;text-decoration:line-through;">too.  </span><span style="color:red;font-weight:700;">too. </span>She takes it everywhere with her, and it's super soft and <span style="color:red;font-weight:700;text-decoration:line-through;">cute.  </span><span style="color:red;font-weight:700;">cute. </span>One of the ears is a bit lower than the other, and I don't think that was designed to be asymmetrical. It's a bit small for what I paid for it though. I think there might be other options that are bigger for the same <span style="color:red;font-weight:700;text-decoration:line-through;">price.  </span><span style="color:red;font-weight:700;">price. </span>It arrived a day earlier than expected, so I got to play with it myself before I gave it to my <span style="color:red;font-weight:700;text-decoration:line-through;">daughter.
</span><span style="color:red;font-weight:700;">daughter.</span>



```python
prompt = f"""
proofread and correct this review. Make it more compelling. 
Ensure it follows APA style guide and targets an advanced reader. 
Output in markdown format.
Text: ```{text}```
"""
response = get_completion(prompt)
display(Markdown(response))
```


I purchased this adorable panda plush as a birthday gift for my daughter, as she kept borrowing mine from my room. It's not just for kids - even adults can appreciate the charm of this cuddly companion. My daughter absolutely adores it and takes it everywhere with her, thanks to its irresistibly soft and cute design. However, I did notice a slight asymmetry in the ears, which I believe was not intentional. Additionally, considering the price, I found the size to be a bit smaller than expected. I suspect there are larger options available for the same cost. Despite this, I was pleasantly surprised when it arrived a day earlier than anticipated, allowing me to enjoy its company before passing it on to my daughter. This panda plush is a delightful gift choice, perfect for anyone who appreciates quality and cuteness in a stuffed animal.

## 5.Expanding
In this lesson, you will generate customer service emails that are tailored to each customer's review.

### Customize the automated reply to a customer email


```python
# given the sentiment from the lesson on "inferring",
# and the original customer message, customize the email
sentiment = "negative"

# review for a blender
review = f"""
So, they still had the 17 piece system on seasonal \
sale for around $49 in the month of November, about \
half off, but for some reason (call it price gouging) \
around the second week of December the prices all went \
up to about anywhere from between $70-$89 for the same \
system. And the 11 piece system went up around $10 or \
so in price also from the earlier sale price of $29. \
So it looks okay, but if you look at the base, the part \
where the blade locks into place doesn’t look as good \
as in previous editions from a few years ago, but I \
plan to be very gentle with it (example, I crush \
very hard items like beans, ice, rice, etc. in the \ 
blender first then pulverize them in the serving size \
I want in the blender then switch to the whipping \
blade for a finer flour, and use the cross cutting blade \
first when making smoothies, then use the flat blade \
if I need them finer/less pulpy). Special tip when making \
smoothies, finely cut and freeze the fruits and \
vegetables (if using spinach-lightly stew soften the \ 
spinach then freeze until ready for use-and if making \
sorbet, use a small to medium sized food processor) \ 
that you plan to use that way you can avoid adding so \
much ice if at all-when making your smoothie. \
After about a year, the motor was making a funny noise. \
I called customer service but the warranty expired \
already, so I had to buy another one. FYI: The overall \
quality has gone done in these types of products, so \
they are kind of counting on brand recognition and \
consumer loyalty to maintain sales. Got it in about \
two days.
"""
```

```python
prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt)
print(response)
```

    Dear valued customer,

    Thank you for taking the time to share your detailed feedback with us. We are truly sorry to hear about the issues you experienced with the pricing changes and the quality of the product. We apologize for any inconvenience this may have caused you.

    If you have any further concerns or would like to discuss this matter further, please feel free to reach out to our customer service team for assistance. We are here to help address any issues you may have encountered.

    Thank you again for your feedback and for being a loyal customer. We appreciate your support.

    AI customer agent

### Temperature
- Temperature is a parameter of a language model, That allows us to change the kind of variety of the model's responses.
- You can think of it as a degree of exploration or randomness of the model.

![alt text](image-3.png)

### Remind the model to use details from the customer's email


```python
prompt = f"""
You are a customer service AI assistant.
Your task is to send an email reply to a valued customer.
Given the customer email delimited by ```, \
Generate a reply to thank the customer for their review.
If the sentiment is positive or neutral, thank them for \
their review.
If the sentiment is negative, apologize and suggest that \
they can reach out to customer service. 
Make sure to use specific details from the review.
Write in a concise and professional tone.
Sign the email as `AI customer agent`.
Customer review: ```{review}```
Review sentiment: {sentiment}
"""
response = get_completion(prompt, temperature=0.7)
print(response)
```

    Dear valued customer,

    Thank you for sharing your detailed feedback with us regarding your recent purchase. We are sorry to hear about the issues you have experienced with the pricing changes, product quality, and the motor noise after a year of use. We apologize for any inconvenience this may have caused you.

    If you have any further concerns or would like to discuss this matter further, please feel free to reach out to our customer service team. We are here to assist you in any way we can.

    Thank you for your loyalty and for taking the time to provide us with your feedback.

    AI customer agent

## 6. The Chat Format

In this notebook, you will explore how you can utilize the chat format to have extended conversations with chatbots personalized or specialized for specific tasks or behaviors.

### Setup


![alt text](image-4.png)

- We are going to use a different helper function, Instead of putting a single prompt as input and getting a single completion.
- We are going to pass in a list of messages and these messages can be from a variety of different roles 

```python
def get_completion(prompt, model="gpt-3.5-turbo"):
    messages = [{"role": "user", "content": prompt}]
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=0, # this is the degree of randomness of the model's output
    )
    return response.choices[0].message["content"]

def get_completion_from_messages(messages, model="gpt-3.5-turbo", temperature=0):
    response = openai.ChatCompletion.create(
        model=model,
        messages=messages,
        temperature=temperature, # this is the degree of randomness of the model's output
    )
#     print(str(response.choices[0].message))
    return response.choices[0].message["content"]
```
- The benefit of a system message, is that it provides a way to frame the conversation without making the request itself part of the conversation.
![alt text](image-5.png)

```python
messages =  [  
{'role':'system', 'content':'You are an assistant that speaks like Shakespeare.'},    
{'role':'user', 'content':'tell me a joke'},   
{'role':'assistant', 'content':'Why did the chicken cross the road'},   
{'role':'user', 'content':'I don\'t know'}  ]
```


```python
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    To get to the other side, perchance! A jest both old and oft heard, but a merry one nonetheless.



```python
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},    
{'role':'user', 'content':'Hi, my name is Isa'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    Hello Isa! It's nice to meet you. How are you doing today?



```python
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},    
{'role':'user', 'content':'Yes,  can you remind me, What is my name?'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    I'm sorry, but I can't remember your name. If you tell me your name, I will do my best to remember it for future reference.



```python
messages =  [  
{'role':'system', 'content':'You are friendly chatbot.'},
{'role':'user', 'content':'Hi, my name is Isa'},
{'role':'assistant', 'content': "Hi Isa! It's nice to meet you. \
Is there anything I can help you with today?"},
{'role':'user', 'content':'Yes, you can remind me, What is my name?'}  ]
response = get_completion_from_messages(messages, temperature=1)
print(response)
```

    Your name is Isa.


### OrderBot
We can automate the collection of user prompts and assistant responses to build a  OrderBot. The OrderBot will take orders at a pizza restaurant. 

![alt text](image-6.png)

```python
def collect_messages(_):
    prompt = inp.value_input
    inp.value = ''
    context.append({'role':'user', 'content':f"{prompt}"})
    response = get_completion_from_messages(context) 
    context.append({'role':'assistant', 'content':f"{response}"})
    panels.append(
        pn.Row('User:', pn.pane.Markdown(prompt, width=600)))
    panels.append(
        pn.Row('Assistant:', pn.pane.Markdown(response, width=600, style={'background-color': '#F6F6F6'})))
 
    return pn.Column(*panels)

```


```python
import panel as pn  # GUI
pn.extension()

panels = [] # collect display 

context = [ {'role':'system', 'content':"""
You are OrderBot, an automated service to collect orders for a pizza restaurant. \
You first greet the customer, then collects the order, \
and then asks if it's a pickup or delivery. \
You wait to collect the entire order, then summarize it and check for a final \
time if the customer wants to add anything else. \
If it's a delivery, you ask for an address. \
Finally you collect the payment.\
Make sure to clarify all options, extras and sizes to uniquely \
identify the item from the menu.\
You respond in a short, very conversational friendly style. \
The menu includes \
pepperoni pizza  12.95, 10.00, 7.00 \
cheese pizza   10.95, 9.25, 6.50 \
eggplant pizza   11.95, 9.75, 6.75 \
fries 4.50, 3.50 \
greek salad 7.25 \
Toppings: \
extra cheese 2.00, \
mushrooms 1.50 \
sausage 3.00 \
canadian bacon 3.50 \
AI sauce 1.50 \
peppers 1.00 \
Drinks: \
coke 3.00, 2.00, 1.00 \
sprite 3.00, 2.00, 1.00 \
bottled water 5.00 \
"""} ]  # accumulate messages


inp = pn.widgets.TextInput(value="Hi", placeholder='Enter text here…')
button_conversation = pn.widgets.Button(name="Chat!")

interactive_conversation = pn.bind(collect_messages, button_conversation)

dashboard = pn.Column(
    inp,
    pn.Row(button_conversation),
    pn.panel(interactive_conversation, loading_indicator=True, height=300),
)

dashboard
```

![alt text](image-7.png)


```python
messages =  context.copy()
messages.append(
{'role':'system', 'content':'create a json summary of the previous food order. Itemize the price for each item\
 The fields should be 1) pizza, include size 2) list of toppings 3) list of drinks, include size   4) list of sides include size  5)total price '},    
)
 #The fields should be 1) pizza, price 2) list of toppings 3) list of drinks, include size include price  4) list of sides include size include price, 5)total price '},    

response = get_completion_from_messages(messages, temperature=0)
print(response)
```

    {
      "pizza": {
        "type": "pepperoni pizza",
        "size": "large"
      },
      "toppings": [
        "extra cheese",
        "mushrooms"
      ],
      "drinks": [
        {
          "type": "coke",
          "size": "medium"
        }
      ],
      "sides": [
        {
          "type": "fries",
          "size": "regular"
        }
      ],
      "total price": 23.45
    }

