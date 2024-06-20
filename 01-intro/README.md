## Preparing the Environment
```
pip install tqdm notebook==7.1.2 openai elasticsearch pandas scikit-learn
```

## Opensource can be replaced for OpenAI API
### Ollama - Running LLMs on a CPU
#### Docker
```
docker run -it \
    -v ollama:/root/.ollama \
    -p 11434:11434 \
    --name ollama \
    ollama/ollama
```

#### Forward a port
```
- Check the port in '-p 11434:11434', we get port after ':'.
- In Visual Studio Code, in the terminal, choose 'PORTS' tag, click 'Forward a Port' then add the '11434' port.
- Use command "docker ps" to find 'NAMES' of the ollama container.
```

#### Pulling the model
```
docker exec -it ollama bash
ollama pull phi3
```

#### Testing
```bash
curl http://localhost:11434/api/chat -d '{
  "model": "phi3",
  "messages": [
    { "role": "user", "content": "why is the sky blue?" }
  ]
}'
```

## Search
### ElasticSearch
#### Run ElasticSearch with Docker
```
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```
#### Index settings:
```
{
    "settings": {
        "number_of_shards": 1,
        "number_of_replicas": 0
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"} 
        }
    }
}
```
#### Query:
```
{
    "size": 5,
    "query": {
        "bool": {
            "must": {
                "multi_match": {
                    "query": query,
                    "fields": ["question^3", "text", "section"],
                    "type": "best_fields"
                }
            },
            "filter": {
                "term": {
                    "course": "data-engineering-zoomcamp"
                }
            }
        }
    }
}
```

We use `"type"`: `"best_fields"`. You can read more about different types of multi_match search in elastic-search.md.