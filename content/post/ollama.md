---
date: "2024-04-19"
title: "In-Depth with Ollama"
author : "Nanik Tolaram (nanikjava@gmail.com)" 
---
# Table of Contents  
- [Ollama](#ollama)  
- [Architecture](#architecture)  
  - [llama.cpp or llama](#llamacpp-or-llama)  
  - [ollama or llama.go](#ollama-or-llamago)  
- [Source Layout](#source-layout)  
  - [Building Source](#building-source)  
  - [Running Ollama](#running-ollama)
  - [Packaging](#packaging)
- [Ollama Internals](#ollama-internals)  
  - [Debugging](#debugging)  
  - [Ollama to llama](#ollama-to-llama)  
    - [Llama Endpoints](#llama-endpoints)  
    - [Model](#model)  
    - [GGUF](#gguf)  

## Ollama

The [Ollama project](https://github.com/ollama/ollama) is a Go project that has gained a lot of traction with 52,000 stars and forked more than 3600 times. The project can be used as a standalone application to interact with different large language models.. Not only that but the project makes it easy to interface for using the LLM functionality (eg: chat) by exposing REST API.

![figure1](/static/media/ollama/figure1.png)


### Architecture

The diagram shows the high level architecture between the application (`ollama` and `llama`) with the large language model. The `ollama` project works like a thin wrapper to the `llama` project which is responsible for interacting with the model.

![figure2](/static/media/ollama/figure2.png)

We will make reference in article to both `llama` and `ollama` as they both serve different purposes even working together as a single unit.

#### llama.cpp or llama

In Figure-1 we can see that the ollama (Go) is run as a normal process. The llama.cpp will be run as a child process of ollama when a request comes in to load the model. The interaction with llama.cpp (https://github.com/ggerganov/llama.cpp) project is via http. The llama.cpp project is an inference project for LLAMA (and other models). 

An inference project is simply a project that involves working with the Large Language Model (LLM). Each model in the LLM has its own way of interacting with systems. So, in an inference project, we use the LLM to do different tasks, like understanding text or generating new content. It's like using a tool for specific jobs, but each tool works a bit differently.

The interaction that is happening internally inside llama.cpp for example,  is not by calling functions to a large language model, but rather through a series of calculations that are fed into memory to be calculated. These calculations can nest deep in several layers of calculation. 

To get a better visualization of how models works here is a very good interactive website that walked through the input/output parameters performed internally by llm https://bbycroft.net/llm

#### ollama or llama.go

The llama.go project simplifies working with Large Language Models (LLMs) by offering the following features:

* **Local Model Execution:** Easily download and run LLM models on your computer's CPU or GPU. The project supports a variety of models tailored to different tasks.

* **REST API Access:** Access the models through a user-friendly REST API, making it simple to integrate them into your applications. This allows for features like chat functionality and natural language processing.


* **Compatibility Across Systems:** llama.go works seamlessly across different operating systems and hardware architectures, ensuring compatibility regardless of your setup.

* **Mac Applications:** Specific applications are available for macOS users, providing a local interface to interact with LLM models. These applications offer similar features to ChatGPT but for local use on Mac devices.

With `llama.go`, users can effortlessly harness the capabilities of LLM models for various tasks, with flexibility, ease of use, and compatibility in mind.

Complete list of the models can be found in this link https://github.com/ollama/ollama?tab=readme-ov-file#model-library


## Source Layout

The following **main** branch shows the directories of the `ollama` source code

![figure3](/static/media/ollama/figure3.png)

We will take a look at some of the directories to explain what kind of source code it host:

| Directory | Description |
|----------|----------|
| app | Build app for Windows platform |
| auth | Supporting package for authorization key | 
| cmd | CLI handlers for the different options available by ollama | 
| convert | Conversion function to create model based on Gemma and Mistral | 
| gpu | GPU interfaces for different graphic cards and operating system | 
| integration | Integration tests | 
| llm | Native integration layer between `llama.cpp` and `ollama`. This directory contains the crux of the logic that llama provides by interacting with the `llama.cpp` project.<br></br>The `llama.cpp` source code lives inside this directory as can be seen in the screenshot below<br></br> ![figure4](/static/media/ollama/figure4.png) | 
| macapp | Electron based app for Mac platform | 
| openai | Provides middleware for partial compatibility with the OpenAI REST API  | 
| parser | HTTP handler and functions for creating model | 
| scripts | Different build scripts for operating systems | 
| server | Contains server related function relevant to models | 

## Building Source

To build the source code we will need to use the latest Go compiler, in this article we are using `1.22.1`. The build and packaging process is done by executing the following command in the root directory of the project:

```
go generate ./...
```

The generate command perform the following steps:

* Execute operating system specific script. This article uses Linux OS so it will use the `gen_linux.sh` script which is specified inside generate_linux.go source:

    ```
    package generate


    //go:generate bash ./gen_linux.sh
    ```
* The bash script will checkout `llama.cpp` submodule from https://github.com/ggerganov/llama.cpp.git into `llm/llama.cpp` directory as shown in the screenshot

    ![figure5](/static/media/ollama/figure5.png)

* Once successfully checked out the next step will be to perform file patching using the patch file inside **ollama/llm/patches** directory.

Following are the log snippets from the output of running go generate command:

```
+ set -o pipefail
+ echo 'Starting linux generate script'
Starting linux generate script
...


-- Build files have been written to: /home/nanik/Downloads/GoProjects/ollama/llm/build/linux/x86_64_static
...
-- Build files have been written to: /home/nanik/Downloads/GoProjects/ollama/llm/build/linux/x86_64/cpu
...
[100%] Built target ollama_llama_server
...
+ gzip -n --best -f ../build/linux/x86_64/cpu/bin/ollama_llama_server
...
+ gzip -n --best -f ../build/linux/x86_64/cpu_avx/bin/ollama_llama_server
...
+ gzip -n --best -f ../build/linux/x86_64/cpu_avx2/bin/ollama_llama_server
...
+ cd ../llama.cpp/
+ git checkout CMakeLists.txt
Updated 1 path from the index
++ cd ../build/linux/x86_64/cpu_avx2/..
++ echo cpu cpu_avx cpu_avx2
+ echo 'go generate completed.  LLM runners: cpu cpu_avx cpu_avx2'
go generate completed.  LLM runners: cpu cpu_avx cpu_avx2
```

On completion the **build** directory will look like the following on a Linux machine


![figure6](/static/media/ollama/figure6.png)

The next command to run is now to build the Go binary using the following command from root directory:

```
go build .
```

The build command will produce an executable file called `ollama` that also contains the `.gz` files produced earlier by the **go generate** command which contains the `llama.cpp` project.

### Running Ollama

The `ollama` project runs as an http server that accepts requests for a number of things related to the models. To get started open a terminal window and run it using the following command:

```
ollama serve
```

The application will start up printing the following log in the console:

```
time=2024-04-08T23:51:13.450+10:00 level=INFO source=images.go:793 msg="total blobs: 6"
time=2024-04-08T23:51:13.451+10:00 level=INFO source=images.go:800 msg="total unused blobs removed: 0"
[GIN-debug] [WARNING] Creating an Engine instance with the Logger and Recovery middleware already attached.

[GIN-debug] [WARNING] Running in "debug" mode. Switch to "release" mode in production.
 - using env:	export GIN_MODE=release
 - using code:	gin.SetMode(gin.ReleaseMode)

[GIN-debug] POST   /api/pull                 --> github.com/ollama/ollama/server.PullModelHandler (5 handlers)
…
[GIN-debug] HEAD   /api/version              --> github.com/ollama/ollama/server.(*Server).GenerateRoutes.func2 (5 handlers)
…
time=2024-04-08T23:51:13.480+10:00 level=WARN source=amd_linux.go:367 msg="amdgpu detected, but no compatible rocm library found.  Either install rocm v6, or follow manual install instructions at https://github.com/ollama/ollama/blob/main/docs/linux.md#manual-install"
time=2024-04-08T23:51:13.480+10:00 level=WARN source=amd_linux.go:99 msg="unable to verify rocm library, will use cpu: no suitable rocm found, falling back to CPU"
time=2024-04-08T23:51:13.480+10:00 level=INFO source=routes.go:1144 msg="no GPU detected"
```

The server is now ready to accept requests. The same <b>ollama</b> application is also used to communicate with the server. Open another terminal window and run the following command:

```
ollama run llama2
```

The command will communicate with the server and download the `llama2` model to be run locally, when the application successfully download and initialize the model it will give you a prompt that you can type to asked any question as shown below:

![figure7](/static/media/ollama/figure7.png)

The `ollama` application provide several commands that you can use shown below:


| Commands | Description |
|----------|----------|
| serve | Start ollama |
| create | Create a model from a Modelfile |
| show | Show information for a model |
| run | Run a model |
| pull | Pull a model from a registry |
| push | Push a model to a registry |
| list | List models |
| cp | Copy a model |
| rm | Remove a model |


### Packaging

As mentioned previously the ollama binary file is packaged with the llama.cpp binary. The build process embeds these binary files into the final binary ollama file. The following screenshot shows the different generated files for an AMD processor with AVX support.


|  |  | |
|----------|----------| -- |
| ![figure8](/static/media/ollama/figure8.png) | ![figure9](/static/media/ollama/figure9.png) | ![figure10](/static/media/ollama/figure10.png) |
|  |  | |

The above screenshot shows 3 different `.gz` files generated for normal CPU, AVX and AVX2, and all of them are named `ollama_llama_server.gz`. These files are packaged together into the final ollama binary file. This is done by using the `embed` standard library as shown in the code snippet below.

```
package llm


import "embed"


//go:embed build/linux/*/*/bin/*
var libEmbed embed.FS
```

The `go:embed` specifies the files that will be packaged which in this case is under `build/` directory. 

## Ollama Internals

As explained previously the `ollama` application is run as an http server that accepts request, the following tables shows the different endpoints that are available:

| HTTP Method | Url |
|----------|----------|
| POST | /api/pull |
| POST | /api/generate |
| POST | /api/chat |
| POST | /api/embeddings |
| POST | /api/create |
| POST | /api/push |
| POST | /api/copy |
| DELETE | /api/delete |
| POST | /api/show |
| POST | /api/blobs/:digest |
| HEAD | /api/blobs/:digest |
| POST | /v1/chat/completions |
| GET | / |
| GET | /api/tags |
| GET | /api/version |
| HEAD | / |
| HEAD | /api/tags |
| HEAD | /api/version |

The commands that are available when running `ollama` use the above url endpoints, for example: running `ollama run llama2` will call the the _/api/pull_ endpoint to download the model and then it uses the _/api/chat_ to accept chat requests and respond to it.

With the availability of the different endpoints, `ollama` gives the flexibility to develop tools of our own using simple tools like cURL or any programming language.

### Debugging

Ollama provides a debugging flag that can be turned on when running it. These flags provide verbose log information on certain internal parts of the application. The following table outlined some of the available flags:

| Flag | Value | Description |
|----------|----------|----------|
| OLLAMA_DEBUG | true | Provide debug log information for ollama application |
| LLAMA_TRACE | 1 | Provide debug log information for llama |
| GGML_DEBUG | 1 | |

For example if we want to debug some issues in ollama we can turn on the debug flag by using the following command:

```
OLLAMA_DEBUG=true ollama serve
```

You will get more debug log information as shown below:

```
...
time=2024-04-09T00:06:25.637+10:00 level=DEBUG source=payload.go:160 msg=extracting variant=cpu file=build/linux/x86_64/cpu/bin/ollama_llama_server.gz
time=2024-04-09T00:06:25.637+10:00 level=DEBUG source=payload.go:160 msg=extracting variant=cpu_avx file=build/linux/x86_64/cpu_avx/bin/ollama_llama_server.gz
time=2024-04-09T00:06:25.637+10:00 level=DEBUG source=payload.go:160 msg=extracting variant=cpu_avx2 file=build/linux/x86_64/cpu_avx2/bin/ollama_llama_server.gz
time=2024-04-09T00:06:25.658+10:00 level=DEBUG source=payload.go:68 msg="availableServers : found" file=/tmp/ollama3078763984/runners/cpu
time=2024-04-09T00:06:25.658+10:00 level=DEBUG source=payload.go:68 msg="availableServers : found" file=/tmp/ollama3078763984/runners/cpu_avx
time=2024-04-09T00:06:25.658+10:00 level=DEBUG source=payload.go:68 msg="availableServers : found" file=/tmp/ollama3078763984/runners/cpu_avx2
time=2024-04-09T00:06:25.658+10:00 level=INFO source=payload.go:41 msg="Dynamic LLM libraries [cpu cpu_avx cpu_avx2]"
time=2024-04-09T00:06:25.658+10:00 level=DEBUG source=payload.go:42 msg="Override detection logic by setting OLLAMA_LLM_LIBRARY"
time=2024-04-09T00:06:25.658+10:00 level=INFO source=gpu.go:121 msg="Detecting GPU type"
time=2024-04-09T00:06:25.658+10:00 level=INFO source=gpu.go:268 msg="Searching for GPU management library libcudart.so*"
time=2024-04-09T00:06:25.658+10:00 level=DEBUG source=gpu.go:286 msg="gpu management search paths: [/tmp/ollama3078763984/runners/cuda*/libcudart.so* … /home/nanik/Downloads/GoProjects/ollama/libcudart.so**]"
time=2024-04-09T00:06:25.667+10:00 level=INFO source=gpu.go:314 msg="Discovered GPU libraries: []"
...
```

To enable more verbose logging from the `llama` project we need to pass in the following flag before running the `go generate` command to build the native application:

```
export CGO_CFLAGS="-g"
```

Must be wondering how does the log looks like when we use the -g above, below can see the log snippets when the `llama` application is processing a chat request:

```
time=2024-04-09T08:34:19.787+10:00 level=DEBUG source=server.go:432 msg="llama runner started in 24.340054 seconds"
time=2024-04-09T08:34:19.789+10:00 level=DEBUG source=prompt.go:172 msg="prompt now fits in context window" required=1 window=2048
[GIN] 2024/04/09 - 08:34:19 | 200 | 24.578298317s |       127.0.0.1 | POST     "/api/chat"
{"function":"get_new_id","level":"VERB","line":254,"msg":"new task id","new_id":7,"tid":"133124421846592","timestamp":1712615664}
{"function":"add_waiting_task_id","level":"VERB","line":397,"msg":"waiting for task id","task_id":7,"tid":"133124421846592","timestamp":1712615664}
{"function":"start_loop","level":"VERB","line":302,"msg":"new task may arrive","tid":"133129545471872","timestamp":1712615664}
{"function":"start_loop","level":"VERB","line":314,"msg":"callback_new_task","task_id":7,"tid":"133129545471872","timestamp":1712615664}
{"function":"process_single_task","level":"INFO","line":1502,"msg":"slot data","n_idle_slots":1,"n_processing_slots":0,"task_id":7,"tid":"133129545471872","timestamp":1712615664}
{"function":"process_single_task","level":"VERB","line":1507,"msg":"slot data","n_idle_slots":1,"n_processing_slots":0,"slots":[{"dynatemp_exponent":1.0,"dynatemp_range":0.0,"frequency_penalty":0.0,"grammar":"","id":0,"ignore_eos":false,"logit_bias":[],"min_keep":0,"min_p":0.05000000074505806,"mirostat":0,"mirostat_eta":0.10000000149011612,"mirostat_tau":5.0,"model":"/home/nanik/.ollama/models/blobs/sha256-8934d96d3f08982e95922b2b7a2c626a1fe873d7c3b06e8e56d7bc0a1fef9246","n_ctx":2048,"n_keep":0,"n_predict":-1,"n_probs":0,"next_token":{"has_next_token":true,"n_remain":-1,"num_tokens_predicted":0,"stopped_eos":false,"stopped_limit":false,"stopped_word":false,"stopping_word":""},"penalize_nl":false,"penalty_prompt_tokens":[],"presence_penalty":0.0,"prompt":null,"repeat_last_n":64,"repeat_penalty":1.0,"samplers":["top_k","tfs_z","typical_p","top_p","min_p","temperature"],"seed":4294967295,"state":0,"stop":[],"stream":true,"task_id":-1,"temperature":0.800000011920929,"tfs_z":1.0,"top_k":40,"top_p":0.949999988079071,"typical_p":1.0,"use_penalty_prompt_tokens":false}],"task_id":7,"tid":"133129545471872","timestamp":1712615664}
...
{"function":"log_server_request","level":"INFO","line":2730,"method":"GET","msg":"request","params":{},"path":"/health","remote_addr":"127.0.0.1","remote_port":37158,"status":200,"tid":"133124421846592","timestamp":1712615664}
{"function":"log_server_request","level":"VERB","line":2739,"msg":"request","request":"","response":"{\"slots_idle\":1,\"slots_processing\":0,\"status\":\"ok\"}","tid":"133124421846592","timestamp":1712615664}
{"function":"get_new_id","level":"VERB","line":254,"msg":"new task id","new_id":8,"tid":"133124421846592","timestamp":1712615664}
{"function":"add_waiting_task_id","level":"VERB","line":397,"msg":"waiting for task id","task_id":8,"tid":"133124421846592","timestamp":1712615664}
...
{"function":"start_loop","level":"VERB","line":317,"msg":"update_multitasks","tid":"133129545471872","timestamp":1712615664}
{"function":"start_loop","level":"VERB","line":336,"msg":"callback_run_slots","tid":"133129545471872","timestamp":1712615664}
{"function":"start_loop","level":"VERB","line":339,"msg":"wait for new task","tid":"133129545471872","timestamp":1712615664}
{"function":"log_server_request","level":"INFO","line":2730,"method":"GET","msg":"request","params":{},"path":"/health","remote_addr":"127.0.0.1","remote_port":37158,"status":200,"tid":"133124421846592","timestamp":1712615664}
...
```

The log looks like gibberish as there are a lot of messages, but these messages are very useful when troubleshooting or learning how the interaction with the models works.

### Ollama to Llama

So far we have explored the `ollama` source code and looked at compiling and running it locally. In this section we will go deeper on how it works internally.

We know now that the way the application is architected is to communicate via http to the `llama` project that is run as a child process as shown in the diagram below

![figure11](/static/media/ollama/figure11.png)

So what does the actual process look like ?, first let’s use the command `ps aux | grep -i ollama` to get the PID of the `ollama` running in the local machine which is shown in the screenshot below. The PID shown is `3180960`

![figure12](/static/media/ollama/figure12.png)

Using the command `pstree -a 3180960` we will be able to see the tree structure of the process as shown below

![figure13](/static/media/ollama/figure13.png)

The diagram shows the `ollama` spawning a child process executing the `llama` project which is packaged with the file name `ollama_llama_server`. The child process is executed by passing several command line parameters used to initialize the LLM models, logging, etc.

Fascinating to observe how the Llama application spawns multiple threads upon receiving a chat request. The threads are used for managing the processing of data to and from the model, as shown in the screenshot provided.


![figure14](/static/media/ollama/figure14.png)

How do we know which port `llama` is running on ?, this is shown in the log when you access the model by running _ollama run <model_name>_. 

Below is a sample log on how it looks like on my local machine where it shows that the `llama` server is running on port `4767`.

```
...
{"function":"initialize","level":"INFO","line":448,"msg":"initializing slots","n_slots":1,"tid":"137254119753600","timestamp":1713529242}
{"function":"initialize","level":"INFO","line":457,"msg":"new slot","n_ctx_slot":2048,"slot_id":0,"tid":"137254119753600","timestamp":1713529242}
{"function":"main","level":"INFO","line":3064,"msg":"model loaded","tid":"137254119753600","timestamp":1713529242}
{"function":"main","hostname":"127.0.0.1","level":"INFO","line":3267,"msg":"HTTP server listening","n_threads_http":"15","port":"4767","tid":"137254119753600","timestamp":1713529242}
...
```

#### Llama Endpoints


When the `llama` application is launched, it operates as a server capable of handling standard HTTP requests. In this scenario, it awaits connections from `ollama`. Following are the endpoints that are exposed by `llama`:

| HTTP Method | Url | Description |
|----------|----------|----------|
| GET | /health  | Check health status of `llama` |
| GET | /slots  | Obtain information about the slots initialized internally |
| GET | /metrics  | Used to get metric information about processed tokens |
| GET | /completion  | Used for chat functionality  |
| GET | /tokenize  | Used to tokenize the information input by the user |
| GET | /detokenize  |  |
| GET | /embdedding  | Used for RAG based applications  |

#### Slots

Internally `llama` processes token by the number of allocated slots available, for eg - if there is 1 slot allocated then `llama` will process 1 token at a time. 

The diagram below shows the high level flow when the _/completions_ endpoint is called, this endpoint is used for chat. The slots are initialized when the endpoint is called, and once it is initialized it will go through the `Decoding Process`.


![figure15](/static/media/ollama/figure15.png)


The **Decoding Process** is the brain of the operation performed inside the llama process. Basically what it does is it prepares the model which is now in memory to perform calculation in different layers until it gets an output. 

Once output is obtained it will send the output to the caller, in this case the caller of the /completions endpoint.
The **Decoding Process** will keep on going until there are no more output returned from the model or it is said that the process has stopped processing token, which means it has reached the end. 

Accessing the llama endpoint using the following cURL command:

```
curl http://localhost:<llama_port>/slots
```

Will show you information about the slot information:

```
[
  {
    "dynatemp_exponent": 1,
    "dynatemp_range": 0,
    "frequency_penalty": 0,
    "grammar": "",
    "id": 0,
    "ignore_eos": false,
    "logit_bias": [],
    "min_keep": 0,
    "min_p": 0.05000000074505806,
    "mirostat": 0,
    "mirostat_eta": 0.10000000149011612,
    "mirostat_tau": 5,
    "model": "/home/nanik/.ollama/models/blobs/sha256-8934d96d3f08982e95922b2b7a2c626a1fe873d7c3b06e8e56d7bc0a1fef9246",
    "n_ctx": 2048,
    "n_keep": 0,
    "n_predict": -1,
    "n_probs": 0,
    "next_token": {
      "has_next_token": true,
      "n_remain": -1,
      "num_tokens_predicted": 0,
      "stopped_eos": false,
      "stopped_limit": false,
      "stopped_word": false,
      "stopping_word": ""
    },
    "penalize_nl": false,
    "penalty_prompt_tokens": [],
    "presence_penalty": 0,
    "prompt": null,
    "repeat_last_n": 64,
    "repeat_penalty": 1,
    "samplers": [
      "top_k",
      "tfs_z",
      "typical_p",
      "top_p",
      "min_p",
      "temperature"
    ],
    "seed": 4294967295,
    "state": 0,
    "stop": [],
    "stream": true,
    "task_id": -1,
    "temperature": 0.800000011920929,
    "tfs_z": 1,
    "top_k": 40,
    "top_p": 0.949999988079071,
    "typical_p": 1,
    "use_penalty_prompt_tokens": false
  }
]
```

#### Model

The diagram shows how the model looks like internally as it contains different metadata information describing the model such as license, weight, tensors, etc. 

![figure16](/static/media/ollama/figure16.png)

In simple terms `Tensors` are mathematical formulas that are used to process input to derive the required output. Every model have layers of this tensors built-in internally that are used to perform complex calculation. Data that need to be processed are converted into tokens which will be feed into the model to extract the relevant value which are again re-converted to text.

In the example of the model used by `ollama` the llama2 model have in total 291 tensors. The following `ollama` example log shows the tensor information read from llama2 model

```
...
{"function":"load_model","level":"INFO","line":403,"msg":"llama_init_from_gpt_params","tid":"136660906297216","timestamp":1713528385}
llama_model_loader: loaded meta data with 23 key-value pairs and 291 tensors from /home/nanik/.ollama/models/blobs/sha256-8934d96d3f08982e95922b2b7a2c626a1fe873d7c3b06e8e56d7bc0a1fef9246 (version GGUF V3 (latest))
llama_model_loader: - tensor    0, split  0:                token_embd.weight q4_0     [  4096, 32000,     1,     1 ]
llama_model_loader: - tensor    1, split  0:           blk.0.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor    2, split  0:            blk.0.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
...
llama_model_loader: - tensor  164, split  0:            blk.4.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  165, split  0:            blk.4.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  166, split  0:              blk.4.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  167, split  0:            blk.4.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  168, split  0:              blk.4.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  169, split  0:         blk.4.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  170, split  0:              blk.4.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  171, split  0:              blk.4.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  172, split  0:           blk.5.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  173, split  0:            blk.5.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  174, split  0:            blk.5.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  175, split  0:              blk.5.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  176, split  0:            blk.5.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  177, split  0:              blk.5.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  178, split  0:         blk.5.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  179, split  0:              blk.5.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  180, split  0:              blk.5.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  181, split  0:           blk.6.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  182, split  0:            blk.6.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  183, split  0:            blk.6.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  184, split  0:              blk.6.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  185, split  0:            blk.6.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  186, split  0:              blk.6.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  187, split  0:         blk.6.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  188, split  0:              blk.6.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  189, split  0:              blk.6.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  190, split  0:           blk.7.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  191, split  0:            blk.7.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  192, split  0:            blk.7.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  193, split  0:              blk.7.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  194, split  0:            blk.7.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  195, split  0:              blk.7.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  196, split  0:         blk.7.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  197, split  0:              blk.7.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  198, split  0:              blk.7.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  199, split  0:           blk.8.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  200, split  0:            blk.8.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  201, split  0:            blk.8.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  202, split  0:              blk.8.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  203, split  0:            blk.8.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  204, split  0:              blk.8.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  205, split  0:         blk.8.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  206, split  0:              blk.8.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  207, split  0:              blk.8.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  208, split  0:           blk.9.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  209, split  0:            blk.9.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  210, split  0:            blk.9.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  211, split  0:              blk.9.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  212, split  0:            blk.9.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  213, split  0:              blk.9.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  214, split  0:         blk.9.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  215, split  0:              blk.9.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  216, split  0:              blk.9.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  217, split  0:                    output.weight q6_K     [  4096, 32000,     1,     1 ]
llama_model_loader: - tensor  218, split  0:          blk.24.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  219, split  0:           blk.24.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  220, split  0:           blk.24.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  221, split  0:             blk.24.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  222, split  0:           blk.24.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  223, split  0:             blk.24.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  224, split  0:        blk.24.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  225, split  0:             blk.24.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  226, split  0:             blk.24.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  227, split  0:          blk.25.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  228, split  0:           blk.25.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  229, split  0:           blk.25.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  230, split  0:             blk.25.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  231, split  0:           blk.25.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  232, split  0:             blk.25.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  233, split  0:        blk.25.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  234, split  0:             blk.25.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  235, split  0:             blk.25.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  236, split  0:          blk.26.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  237, split  0:           blk.26.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  238, split  0:           blk.26.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  239, split  0:             blk.26.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  240, split  0:           blk.26.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  241, split  0:             blk.26.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  242, split  0:        blk.26.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  243, split  0:             blk.26.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  244, split  0:             blk.26.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  245, split  0:          blk.27.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  246, split  0:           blk.27.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
...
llama_model_loader: - tensor  269, split  0:        blk.29.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  270, split  0:             blk.29.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  271, split  0:             blk.29.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  272, split  0:          blk.30.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  273, split  0:           blk.30.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  274, split  0:           blk.30.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  275, split  0:             blk.30.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  276, split  0:           blk.30.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  277, split  0:             blk.30.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  278, split  0:        blk.30.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  279, split  0:             blk.30.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  280, split  0:             blk.30.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  281, split  0:          blk.31.attn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  282, split  0:           blk.31.ffn_down.weight q4_0     [ 11008,  4096,     1,     1 ]
llama_model_loader: - tensor  283, split  0:           blk.31.ffn_gate.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  284, split  0:             blk.31.ffn_up.weight q4_0     [  4096, 11008,     1,     1 ]
llama_model_loader: - tensor  285, split  0:           blk.31.ffn_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: - tensor  286, split  0:             blk.31.attn_k.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  287, split  0:        blk.31.attn_output.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  288, split  0:             blk.31.attn_q.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  289, split  0:             blk.31.attn_v.weight q4_0     [  4096,  4096,     1,     1 ]
llama_model_loader: - tensor  290, split  0:               output_norm.weight f32      [  4096,     1,     1,     1 ]
llama_model_loader: Dumping metadata keys/values. Note: KV overrides do not apply in this output.
...
```

Models that are compatible with `llama.cpp` must be in GGUF format and can be searched using the following link https://huggingface.co/models?sort=trending&search=gguf

#### GGUF

We know now that model that can be used by `llama` have to be in GGUF format, so what is it ?. According to the author https://github.com/ggerganov/ggml/blob/master/docs/gguf.md

```
GGUF is a file format for storing models for inference with GGML and executors based on GGML. GGUF is a binary format that is designed for fast loading and saving of models, and for ease of reading. Models are traditionally developed using PyTorch or another framework, and then converted to GGUF for use in GGML.
````

![figure17](/static/media/ollama/figure17.png)


The following website provide an in-depth explanation about the format and how the model works internally https://vickiboykis.com/2024/02/28/gguf-the-long-way-around/


