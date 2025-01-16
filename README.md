# llama.cpp cheat sheet



# Install
```shell
git clone https://github.com/ggerganov/llama.cpp
cd llama.cpp


# https://github.com/ggerganov/llama.cpp/blob/master/docs/build.md
# CUDA BUILD
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release

cd build/bin
./llama-cli -m "/home/user/test/xxxxxxxxxx.gguf"
```














<br><br>
<br><br>





# Files

## GGUF (.gguf)
- https://huggingface.co/bartowski/DeepSeek-Coder-V2-Instruct-GGUF

### Shared Model
- Most of the time the .gguf file is splitted in multiple files because of the size so you can do following:


Option 1: Run shared model:
- llama.cpp unterstützt mittlerweile Modelle, die über mehrere Dateien verteilt sind (z. B. GGUF-Shards). Du musst lediglich eine der Dateien (meist die erste) angeben, und llama.cpp erkennt automatisch, dass es sich um ein sharded Modell handelt.
```shell
./llama-cli -m DeepSeek-Coder-V2-Instruct-Q4_K_M-00001-of-00004.gguf
```


Option 2: Merge into one file
- Das Tool erkennt automatisch, dass es die Dateien 00002-of-00004, 00003-of-00004, und 00004-of-00004 zusammenfügen muss.
- Alle Dateien müssen sich im selben Verzeichnis befinden.
```shell
cd /home/UserName/Projects/ai/resources/models/llm/deepseek/DeepSeek-Coder-V2-Instruct-Q4_K_M.gguf/
/home/UserName/Projects/ai/LLM/RUNTIME/llama.cpp/build/bin/llama-gguf-split --merge DeepSeek-Coder-V2-Instruct-Q4_K_M-00001-of-00004.gguf DeepSeek-Coder-V2-Instruct-Q4_K_M.gguf
```
























<br><br>
___
___
<br><br>





# CLI
```shell
llama-cli -m model.gguf

# > hi, who are you?
# Hi there! I'm your helpful assistant! I'm an AI-powered chatbot designed to assist and provide information to users like you. I'm here to help answer your questions, provide guidance, and offer support on a wide range of topics. I'm a friendly and knowledgeable AI, and I'm always happy to help with anything you need. What's on your mind, and how can I assist you today?
#
# > what is 1+1?
# Easy peasy! The answer to 1+1 is... 2!
```

<br><br>

## Run in conversation mode with custom chat template
```shell
# use the "chatml" template (use -h to see the list of supported templates)
llama-cli -m model.gguf -cnv --chat-template chatml

# use a custom template
llama-cli -m model.gguf -cnv --in-prefix 'User: ' --reverse-prompt 'User:'
```

<br><br>

## Run simple text completion
- To disable conversation mode explicitly, use -no-cnv
```shell
llama-cli -m model.gguf -p "I believe the meaning of life is" -n 128 -no-cnv

# I believe the meaning of life is to find your own truth and to live in accordance with it. For me, this means being true to myself and following my passions, even if they don't align with societal expectations. I think that's what I love about yoga – it's not just a physical practice, but a spiritual one too. It's about connecting with yourself, listening to your inner voice, and honoring your own unique journey.
```


<br><br>

## Constrain the output with a custom grammar
- To disable conversation mode explicitly, use -no-cnv
```shell
llama-cli -m model.gguf -n 256 --grammar-file grammars/json.gbnf -p 'Request: schedule a call at 8pm; Command:'

# {"appointmentTime": "8pm", "appointmentDetails": "schedule a a call"}
```










<br><br>
<br><br>
___
___
<br><br>
<br><br>




## [`llama-server`](examples/server)

#### A lightweight, [OpenAI API](https://github.com/openai/openai-openapi) compatible, HTTP server for serving LLMs.

- <details open>
    <summary>Start a local HTTP server with default configuration on port 8080</summary>

    ```bash
    llama-server -m model.gguf --port 8080

    # Basic web UI can be accessed via browser: http://localhost:8080
    # Chat completion endpoint: http://localhost:8080/v1/chat/completions
    ```

    </details>

- <details>
    <summary>Support multiple-users and parallel decoding</summary>

    ```bash
    # up to 4 concurrent requests, each with 4096 max context
    llama-server -m model.gguf -c 16384 -np 4
    ```

    </details>

- <details>
    <summary>Enable speculative decoding</summary>

    ```bash
    # the draft.gguf model should be a small variant of the target model.gguf
    llama-server -m model.gguf -md draft.gguf
    ```

    </details>

- <details>
    <summary>Serve an embedding model</summary>

    ```bash
    # use the /embedding endpoint
    llama-server -m model.gguf --embedding --pooling cls -ub 8192
    ```

    </details>

- <details>
    <summary>Serve a reranking model</summary>

    ```bash
    # use the /reranking endpoint
    llama-server -m model.gguf --reranking
    ```

    </details>

- <details>
    <summary>Constrain all outputs with a grammar</summary>

    ```bash
    # custom grammar
    llama-server -m model.gguf --grammar-file grammar.gbnf

    # JSON
    llama-server -m model.gguf --grammar-file grammars/json.gbnf
    ```

    </details>


## [`llama-perplexity`](examples/perplexity)

#### A tool for measuring the perplexity [^1][^2] (and other quality metrics) of a model over a given text.

- <details open>
    <summary>Measure the perplexity over a text file</summary>

    ```bash
    llama-perplexity -m model.gguf -f file.txt

    # [1]15.2701,[2]5.4007,[3]5.3073,[4]6.2965,[5]5.8940,[6]5.6096,[7]5.7942,[8]4.9297, ...
    # Final estimate: PPL = 5.4007 +/- 0.67339
    ```

    </details>

- <details>
    <summary>Measure KL divergence</summary>

    ```bash
    # TODO
    ```

    </details>

[^1]: [examples/perplexity/README.md](examples/perplexity/README.md)
[^2]: [https://huggingface.co/docs/transformers/perplexity](https://huggingface.co/docs/transformers/perplexity)
