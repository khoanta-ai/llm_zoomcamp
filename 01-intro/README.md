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

