# OllamaGUI
Docker-based setup mixing Ollama and Open WebUI. This setup is recommended for lab use only, **DO NOT** use it in production!

This repository is heavily inspired by John Capobianco's series of [YouTube videos](https://www.youtube.com/@johncapobianco2527) and [GitHub repositories](https://github.com/automateyournetwork).

---

### Install
Prerequisites:
- [Docker engine](https://docs.docker.com/engine/install/)

#### Setup
First off, let's download (that is, `pull`) the official Ollama container image and create a couple of directories to store common files:
**WARNING**: the next step is likely to require a long time, depending on your network speed.
```
$ docker pull ollama/ollama:latest

$ sudo mkdir /usr/share/ollama /usr/share/open-webui
```

---

### Run (manual)
The container can be run manually, that is, through CLI and the `docker` command:
```
$ docker run -d \
  -v /usr/share/ollama:/root/.ollama \
  [--gpus all] \
  -p 11434:11434 \
  --name ollama \
  ollama/ollama
```
**NOTE**: `--gpus` will only work on specific hardware + software.

```
$ docker exec -it ollama bash
root@0a438b17cc3b:/# ollama -v
ollama version is 0.8.0
```

**WARNING**: the next step is likely to require a long time, depending on your network speed.
Users can pick and choose any models, this example is run with `llama3.2:3b`:
```
root@0a438b17cc3b:/# ollama pull llama3.2:latest
```

**NOTE**: models can be searched at https://ollama.com/library

Once the pull is complete, here are some commands to verify and display what we just did:
```
root@0a438b17cc3b:/# ollama list
NAME               ID              SIZE      MODIFIED       
llama3.2:latest    a80c4f17acd5    2.0 GB    27 seconds ago

root@0a438b17cc3b:/# ollama show llama3.2
  Model
    architecture        llama
    parameters          3.2B
    context length      131072
    embedding length    3072
    quantization        Q4_K_M
...
```

Inside the container, models are stored under `/root/.ollama/models/blobs/` by default. On the host, the corresponding directory is `/usr/share/ollama/models/blobs`.

**NOTE**: One way to control where models are stored is to set the `OLLAMA_MODELS` environment variable.
```
root@0a438b17cc3b:/# ls -lh /root/.ollama/models/blobs/
total 4.4G
-rw-r--r--. 1 root root  485 Oct  9 12:53 sha256-3f8eb...
-rw-r--r--. 1 root root  13K Oct  9 12:53 sha256-4fa55...
-rw-r--r--. 1 root root  110 Oct  9 12:53 sha256-57707...
-rw-r--r--. 1 root root 4.4G Oct  9 12:53 sha256-6a074...
-rw-r--r--. 1 root root  254 Oct  9 12:53 sha256-8ab48...

$ ls -lh /usr/share/ollama/models/blobs
total 4.4G
-rw-r--r--. 1 root root  485 Oct  9 12:53 sha256-3f8eb...
-rw-r--r--. 1 root root  13K Oct  9 12:53 sha256-4fa55...
-rw-r--r--. 1 root root  110 Oct  9 12:53 sha256-57707...
-rw-r--r--. 1 root root 4.4G Oct  9 12:53 sha256-6a074...
-rw-r--r--. 1 root root  254 Oct  9 12:53 sha256-8ab48...
```

Ollama exposes an API on port 11434. The service can be reached externally (e.g. `0.0.0.0`) as such:
```
% curl -s http://<ip_address>:11434/api/generate \
  -d '{"model": "llama3.2", "prompt": "Which is the Capital city of Italy?", "stream": false}' \
  | jq '.'
{
  "model": "llama3.2",
  "created_at": "2025-05-29T09:58:52.043602763Z",
  "response": "The capital city of Italy is Rome (Italian: Roma).",
  "done": true,
  "done_reason": "stop",
  "context": [
    128006,
    9125,
    ...
  ],
  "total_duration": 214982742,
  "load_duration": 21489818,
  "prompt_eval_count": 33,
  "prompt_eval_duration": 3931930,
  "eval_count": 13,
  "eval_duration": 189099164
}
```

Before moving to the next step, terminate the container and clean up the environment:
```
$ docker container stop ollama

$ docker container rm $(docker container ls -aq -f "status=exited" -f "status=created")
```

---

### Run (compose)
Docker Compose offers a way to start/stop containers without getting too involved with the details. Any settings are stored in a YAML file which is then run as such:
- Step 1: run the Ollama container in _standalone_ mode
```
$ docker compose --file dc1_standalone.yaml up -d
[+] Running 2/2
 ✔ Network ollamagui_default  Created
 ✔ Container ollama           Started
```
**NOTE**: `-d` runs in `detached` mode. To show the container logs run `docker compose --file <yaml> logs [--follow]`.<br/>

Additional checks:<br/>
```
$ docker compose ls
NAME                STATUS              CONFIG FILES
ollamagui           running(1)          /home/toor/github/OllamaGUI/dc1_standalone.yaml


$ docker compose -f dc1_standalone.yaml ps
NAME      IMAGE           COMMAND               SERVICE   CREATED         STATUS         PORTS
ollama    ollama/ollama   "/bin/ollama serve"   ollama    2 minutes ago   Up 2 minutes   0.0.0.0:11434->11434/tcp, :::11434->11434/tcp
```

The container can still be accessed directly through its name, `docker exec`, or it can be accessed externally through its API. There's no difference in comparison with the previous "manual" approach.

To terminate the container, run `docker compose --file <yaml> down`. There is no need to remove the _exited_ container this time.<br/>

- Step 2: run the Ollama and Open WebUI containers by means of Docker Compose
```
$ docker compose --file dc2_openwebui.yaml up -d
```

This time, two separate containers will be spun up:
```
$ docker compose ls
NAME                STATUS              CONFIG FILES
ollamagui           running(2)          /home/toor/github/OllamaGUI/dc2_openwebui.yaml


$ docker compose -f dc2_openwebui.yaml ps
NAME         IMAGE                                COMMAND               SERVICE        CREATED          STATUS                    PORTS
ollama       ollama/ollama                        "/bin/ollama serve"   ollama         23 seconds ago   Up 22 seconds             0.0.0.0:11434->11434/tcp, :::11434->11434/tcp
open-webui   ghcr.io/open-webui/open-webui:main   "bash start.sh"       ollama-webui   23 seconds ago   Up 22 seconds (healthy)   0.0.0.0:8090->8080/tcp, :::8090->8080/tcp
```

Finally, access the GUI through `http://<ip_address>:8090/`, authentication is not required (hint: see `WEBUI_AUTH=False` in `dc2_openwebui.yaml`)<br/>

![Sample GUI screenshot](./assets/GUI.png)

