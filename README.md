# OllamaGUI
Docker-based setup mixing Ollama and Open WebUI

### Install
```
$ docker pull ollama/ollama:latest

$ sudo mkdir /usr/share/ollama /usr/share/open-webui
```

### Run (manual)
```
$ docker run -d -v /usr/share/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama

$ docker exec -it ollama bash
root@0a438b17cc3b:/# ollama -v
ollama version is 0.3.9
```

**WARNING**: the next step is likely to require a long time, depending on your network speed.
```
root@0a438b17cc3b:/# ollama pull llama3
```

```
root@0a438b17cc3b:/# ollama list
NAME         	ID          	SIZE  	MODIFIED
llama3:latest	365c0bd3c000	4.7 GB	About a minute ago
```

### Run (compose)
- step 1
Run the Ollama container in _standalone_ mode.
```
$ docker run -d -v /usr/share/ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama
```

The container can be accessed directly by means of:
```
$ docker exec -it ollama bash
```

Or, it can be accessed externally through its API (port 11434), as such:
```
$ curl http://<ip_address>:11434/api/generate -d '{"model": "llama3", "prompt": "Which is the Capital city of Italy?", "stream": false}'
{"model":"llama3","created_at":"2024-09-03T06:05:53.997420619Z","response":"The capital city of Italy is Rome (Italian: Roma).","done":true,"done_reason":"stop","context":[128006,882,128007,271,23956,374,279,18880,3363,315,15704,30,128009,128006,78191,128007,271,791,6864,3363,315,15704,374,22463,320,70211,25,46601,570],"total_duration":30927400777,"load_duration":15572939780,"prompt_eval_count":18,"prompt_eval_duration":8153160000,"eval_count":13,"eval_duration":7142811000}%    
```

- step 2
Run the Ollama and Open WebUI containers by means of Docker Compose.
```
$ docker compose -f dc2_standalone.yaml
```

Finally, access the GUI through `http://<ip_address>:8090/`.

**NOTE**: direct access to Ollama API is still globally (i.e. 0.0.0.0) available on port 11434.

