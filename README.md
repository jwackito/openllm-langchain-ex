# Minimal Guide for setting up a local LLM and integrating it with LangChain.

## Setup FastChat

[FastChat](https://github.com/lm-sys/FastChat) is an open platform for training, serving, and evaluating large language model based chatbots.
Also FastChat provides an API compatible to openai api making integration with langchain easy.

First install fastchat. Use a virtual environment is recommended.

```bash
conda create -n fastchat python=3.10.9
conda activate fastchat
```

```
pip install fastchat
```

Setup the API (see the [docs](https://github.com/lm-sys/FastChat/blob/main/docs/openai_api.md)] for more info)

There are 3 basic steps:

First, launch the controller
```bash
python3 -m fastchat.serve.controller
```

Then, launch the model worker(s) # that will install the vicuna model in 8bit
```bash
python3 -m fastchat.serve.model_worker --model-path lmsys/vicuna-7b-v1.3 --load-8bits --use-cpu-offload
``` 

Note: It is a good idea to download the model to a local directory and the use the local path instead.


Finally, launch the RESTful API server
```bash
python3 -m fastchat.serve.openai_api_server --host localhost --port 8000
```
Now, let us test the API server.

OpenAI Official SDK
The goal of openai_api_server.py is to implement a fully OpenAI-compatible API server, so the models can be used directly with openai-python library.

First, install `openai-python`:
```bash
pip install --upgrade openai
```
Then, interact with model vicuna:

```python
import openai
openai.api_key = "EMPTY" # Not support yet
openai.api_base = "http://localhost:8000/v1"

model = "vicuna-7b-v1.3"
prompt = "Once upon a time"

# create a completion
completion = openai.Completion.create(model=model, prompt=prompt, max_tokens=64)
# print the completion
print(prompt + completion.choices[0].text)

# create a chat completion
completion = openai.ChatCompletion.create(
  model=model,
  messages=[{"role": "user", "content": "Hello! What is your name?"}]
)
# print the completion
print(completion.choices[0].message.content)
```

You can also execute a gradio interface
```bash
python3 -m fastchat.serve.gradio_web_server
```

You can test the model using a nice [gradio](https://gradio.app/) interface.

## Setup LangChain

[LangChain](https://github.com/hwchase17/langchain) is an open source framework that allows AI developers to combine Large Language Models (LLMs) like GPT-4 with external data. 

install langchain,  Use a virtual environment is recommended. I use one for fastchat and another for langchain. I had some problem with python enviroment and pip install. So I used a conda enviroment.

```bash
conda create -n langchain python=3.10.9
conda activate langchain
```

Then install langchain.

```bash
conda install langchain
```

### Integration with FastChat

LangChain uses OpenAI model names by default, so we need to assign some faux OpenAI model names to our local model. Here, we use Vicuna as an example and use it for three endpoints: chat completion, completion, and embedding. --model-path can be a local folder or a Hugging Face repo name. See a full list of supported models here.

```bash
python3 -m fastchat.serve.model_worker --model-names "gpt-3.5-turbo,text-davinci-003,text-embedding-ada-002" --model-path lmsys/vicuna-7b-v1.3
```
You will need to  relaunch the RESTful API server

```bash
python3 -m fastchat.serve.openai_api_server --host localhost --port 8000
```
That's all. Now you can try the notebook `langchain-vicuna.ipynb` for an example 





