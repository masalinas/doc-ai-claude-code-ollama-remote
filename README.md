# Description
Configure ollama to be accesed in remote mode from claude code and ollama directly

## Steps

Follow these steps:

- **STEP01**: install ollama
Install from ollama portal:

```shell
$ curl -fsSL https://ollama.com/install.sh | sh
```

- **STEP02**: configure remote mode
Edit the ollama service configuration to access ollama remotely:

```shell
$ sudo systemctl edit ollama.service
```

Add these env variables: `OLLAMA_HOST` and `OLLAMA_ORIGINS` like this:
```
### Editing /etc/systemd/system/ollama.service.d/override.conf
### Anything between here and the comment below will become the contents of the drop-in file

[Service]
Environment="OLLAMA_HOST=0.0.0.0"
Environment="OLLAMA_ORIGINS=*"

### Edits below this comment will be discarded

### /etc/systemd/system/ollama.service
# [Unit]
# Description=Ollama Service
# After=network-online.target
# 
# [Service]
# ExecStart=/usr/local/bin/ollama serve
# User=ollama
# Group=ollama
# Restart=always
# RestartSec=3
# Environment="PATH=/home/simur/.sdkman/candidates/java/current/bin:/home/simur/.nvm/versions/node/v24.14.1/bin:/home/simur/.local/bin:/usr/lib/nvidia-cuda-toolkit/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin:/home/simur/.lmstudio/bin:/home/simur/.lmstudio/bin"
# 
# [Install]
# WantedBy=default.target
```

- **STEP03**: restart ollama service
```shell
$ sudo systemctl restart ollama
```

- **STEP04**: claude code connected directly to ollama
Create a `settings.json` file under `.claude` folder in your project, and add these lines:

```
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://aburrido.edv.uniovi.es:11434",
    "ANTHROPIC_AUTH_TOKEN": "ollama",
    "ANTHROPIC_MODEL": "qwen3:14b"
  }
}
```

- **STEP05**: claude code connected to ollama throw liteLLM
In this case we are using liteLLM as proxy AI to connect to ollama

First install liteLLM and configure

```shell
$ mkdir liteLLM
$ cd liteLLM
$ python3.12 -m venv .venv
$ pip install 'litellm[proxy]'
```

configure liteLLM with out model:
```
model_list:
  - model_name: qwen3-14b
    litellm_params:
      model: ollama_chat/qwen3:14b
      api_base: 'http://localhost:11434'
      num_ctx: 60000
```

```shell
$ litellm --config config.yaml --port 4000
```

Create a `settings.json` file under `.claude` folder in your project, and add these lines:

```
{
  "env": {
    "ANTHROPIC_BASE_URL": "http://aburrido.edv.uniovi.es:4000",
    "ANTHROPIC_AUTH_TOKEN": "liteLLM",
    "ANTHROPIC_MODEL": "qwen3-14b"
  }
}
```

