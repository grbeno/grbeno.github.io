## Deploying Async LLM-Chat with Memory-layer

### 1. Redis Memory Channel Layer

There is a Django web application with ReactJS as the frontend, which presents a chatbot with LLM using OpenAI's API.

The chat has a memory layer as well, allowing the user to have longer conversations with the chatbot.

The main tools of the application are [`channels`](https://channels.readthedocs.io/en/stable/index.html) in Django and `WebSocket` in React, which are the key components for implementing asynchroniuos communication.

This application works well on localhost, but if we want to deploy it, we need to change the channel layer from In-memory to Redis, as suggested by the [documentation](https://channels.readthedocs.io/en/stable/topics/channel_layers.html#redis-channel-layer).

[GitHub repository is available here](https://github.com/grbeno)

### Change the In-memory to Redis Channel Layer

As the first step, we need to install [`channels_redis`](https://github.com/django/channels_redis/?tab=readme-ov-file#channels_redis)
```
pip install channels_redis
```

In order to use Redis for the channel layer, we need to modify the code in several places. First of all, we need to update the Django file that contains the class for handling the chat.

```python
from openai import OpenAI
from environs import Env
import redis  # Redis Channel Layer
import json  # Redis Channel Layer

# Load the environment variables
env = Env()
env.read_env()

client = OpenAI()
client.api_key=env.str("OPENAI_API_KEY")

class AiChat():

    _role = "You are helpful and friendly. Be short but concise as you can!"
    
    # Redis Channel Layer
    _redis_client = redis.Redis(host='redis', port=6379, db=0)  

    def __init__(self, prompt: str, model: str, channel: str) -> None:
        self.prompt = prompt
        self.model = model
        self.channel = channel

        ## Redis Channel Layer
        
        # Check if the channel exists in Redis
        if not self._redis_client.exists(channel):
            initial_data = [{"role": "system", "content": self._role}]
            self._redis_client.set(channel, json.dumps(initial_data))
        
        # Retrieve the conversation from Redis
        conversation_data = self._redis_client.get(channel)
        self.conversation = json.loads(conversation_data) if conversation_data else initial_data


    def chat(self) -> str:
        if self.prompt:
            # The conversation is going on ...
            # Adding prompt to chat history
            self.conversation.append({"role": "user", "content": self.prompt})
            # Redis Channel Layer
            self._redis_client.set(self.channel, json.dumps(self.conversation))
            # The OpenAI's chat completion generates answers to your prompts.
            completion = client.chat.completions.create(
                model=self.model,
                messages=self.conversation
            )
            answer = completion.choices[0].message.content
            # Adding answer to chat history
            self.conversation.append({"role": "assistant", "content": answer})
            # Redis Channel Layer
            self._redis_client.set(self.channel, json.dumps(self.conversation))
            return answer

```

The places where I modified the original code are marked with the _*# Redis Channel Layer*_ comment.

Next, we need to change the channel layer in `./config/settings.py` from In-memory to Redis Channel Layer and set the host to the Redis port.

```python
# Channels

CHANNEL_LAYERS = {
    "default": {
        # "BACKEND": "channels.layers.InMemoryChannelLayer",
        "BACKEND": "channels_redis.core.RedisChannelLayer",
        "CONFIG": {
            "hosts": [(env.str('REDISHOST', default="redis"), 6379)],
        },           
     }    
 }

```

The "REDISHOST" provided from the cloud platform, while "redis" is the default for local development, as you will see in the next section.

### Deploying