## Executing Code in E2B Sandbox 

I guess this is not exactly a review because I tested only a few features of the **E2B Sandbox**. I was just curious how to implement a solution that **interprets LLM-generated Python code** and returns its result in a custom environment. By now, I have generated chat responses using LLMs in text format, so it would be a new experience if I could generate not just the code but also invoke its return value.

Although my original plan was to implement this inside a web application and write a post about that, I would rather choose CLI on desktop for now.

The CLI looks usually boring, but **using the Rich Python library** can enhance the experience. By the way, I am also using Rich for the first time; previously, I used Colorama if I wanted to add color to the black and white console window.

### What is Sandbox?

Sandbox originally refers to a safe playground for children. The technical concept is to provide a safe environment by evaluating code without the risk of prompt injection coming from outside the sandbox.

### Setting up E2B Sandbox

#### Prerequisites
- Python >=3.11.5
- Claude API-key

I will use E2B's sandbox. You can learn from them and their goals [here](https://e2b.dev/).

After signing up find and copy the API-key at dashboard and paste it to an `.env` file.
```
E2B_API_KEY=e2b_***
```
First step is to install the E2B package, which is **pip** in my case.

```
pip install e2b-code-interpreter python-dotenv
```
