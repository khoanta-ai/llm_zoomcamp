## HW2
### Q1 Running Ollama with Docker
#### Docker
```
docker run -it \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```
#### Access ollama container
```
docker exec -it ollama bash
```
#### Question: What's the version of ollama client?
```
ollama -v
```
#### Answer
```
ollama version is 0.1.44
```

### Q2. Downloading an LLM
We will donwload a smaller LLM - gemma:2b.

Again let's enter the container and pull the model:

```
ollama pull gemma:2b
```
In docker, it saved the results into `/root/.ollama`

We're interested in the metadata about this model. You can find it in `models/manifests/registry.ollama.ai/library`

#### What's the content of the file related to gemma?
```
cat  /root/.ollama/models/manifests/registry.ollama.ai/library/gemma/2b
```
#### Answer
see `Q2_answer.json`

## Q3. Running the LLM
Test the following prompt: "10 * 10". 

## What's the answer?
```
curl http://localhost:11434/api/chat -d '{
  "model": "gemma:2b",
  "messages": [
    { "role": "user", "content": "10 * 10" }
  ]
}'
```
```
from openai import OpenAI
def build_llm(base_url, api_key):
    client = OpenAI(
        base_url=base_url,
        api_key=api_key
    )
    return client


def query_llm(prompt, client, model_name):
    response = client.chat.completions.create(
        model=model_name,
        messages=[{'role':'user', 'content':prompt}]
    )
    return response.choices[0].message.content

base_url = 'http://localhost:11434/v1/'
api_key = 'ollama'
model_name = 'gemma:2b'

prompt = '10 * 10'
phi3_client = build_llm(base_url, api_key)
response_res = query_llm(prompt=prompt, client=phi3_client, model_name=model_name)
print(response_res)
```
#### Answer
```
'Sure. Here is the answer to the question:\n\n10 * 10 = 100.'
```

## Q4. Donwloading the weights
We don't want to pull the weights every time we run a docker container. Let's do it once and have them available every time we start a container.

First, we will need to change how we run the container.

Instead of mapping the `/root/.ollama` folder to a named volume, let's map it to a local directory:

```
mkdir ollama_files

docker run -it \
    --rm \
    -v ./ollama_files:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```
Now pull the model:

```
docker exec -it ollama ollama pull gemma:2b 
```
#### What's the size of the ollama_files/models folder?
- 0.6G
- 1.2G
- 1.7G
- 2.2G

Hint: on linux, you can use `du -h` for that.

```
8.0K    ./manifests/registry.ollama.ai/library/gemma
12K     ./manifests/registry.ollama.ai/library
16K     ./manifests/registry.ollama.ai
20K     ./manifests
1.6G    ./blobs
1.6G    .
```
#### Answer
```
1.7GB
```
## Q5. Adding the weights
Let's now stop the container and add the weights to a new image

For that, let's create a `Dockerfile`:
```dockerfile
FROM ollama/ollama

COPY ...
```
#### What do you put after `COPY`?
```
FROM ollama/ollama

COPY ./ollama_files/models /root/.ollama/models 
``` 
#### Answer
```
./ollama_files/models /root/.ollama/models 
```

## Q6. Serving it
Let's build it:

```
docker build -t ollama-gemma2b .
```
And run it:

```
docker run -it --rm -p 11434:11434 ollama-gemma2b
```
We can connect to it using the OpenAI client

Let's test it with the following prompt:
```
prompt = "What's the formula for energy?"
```
Also, to make results reproducible, set the `temperature` parameter to 0:
```
response = client.chat.completions.create(
    #...
    temperature=0.0
)
```
#### How many completion tokens did you get in response?
- 304
- 604
- 904
- 1204

```
def query_llm(prompt, client, model_name):
    response = client.chat.completions.create(
        model=model_name,
        messages=[{'role':'user', 'content':prompt}],
        temperature=0.0
    )
    return response.choices[0].message.content, response

response_res, raw_response = query_llm(prompt=prompt, client=phi3_client, model_name=model_name)

raw_response.usage
```
`CompletionUsage(completion_tokens=277, prompt_tokens=0, total_tokens=277)`

#### Answer
```304```







