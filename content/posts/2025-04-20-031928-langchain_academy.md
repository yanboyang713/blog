---
title: "LangChain Academy"
date: 2025-04-20
draft: false
---

## Setup {#setup}


### Clone repo {#clone-repo}

```bash
git clone https://github.com/langchain-ai/langchain-academy.git
cd langchain-academy
```


### Create an environment and install dependencies {#create-an-environment-and-install-dependencies}

```bash
python3 -m venv lc-academy-env
source lc-academy-env/bin/activate
pip install -r requirements.txt
```


### Running [Jupyter]({{< relref "20230307124643-jupyter.md" >}}) notebooks {#running-jupyter--20230307124643-jupyter-dot-md--notebooks}

```bash
jupyter notebook
```


### Sign up for [LangSmith]({{< relref "2025-04-20-033220-langsmith.md" >}}) {#sign-up-for-langsmith--2025-04-20-033220-langsmith-dot-md}

Sign up here. You can reference LangSmith docs here.

Then, set

```bash
LANGCHAIN_API_KEY
LANGCHAIN_TRACING_V2=true
```

in your environment.


### Set up [OpenAI API]({{< relref "2023-10-23-193127-openai_api.md" >}}) key {#set-up-openai-api--2023-10-23-193127-openai-api-dot-md--key}

If you don’t have an OpenAI API key, you can [sign up](https://openai.com/index/openai-api/) here.
Then, set

```bash
OPENAI_API_KEY
```

in your environment.


### Tavily for web search {#tavily-for-web-search}

[Tavily Search]({{< relref "2025-04-20-034501-tavily_search.md" >}}) API is a search engine optimized for LLMs and RAG, aimed at efficient, quick, and persistent search results. You can sign up for an API key here. It’s easy to sign up and offers a generous free tier. Some lessons in Module 4 will use Tavily.
Then, set

```bash
TAVILY_API_KEY
```

in your environment.


### Set up LangGraph Studio {#set-up-langgraph-studio}

-   LangGraph Studio is a custom IDE for viewing and testing agents.
-   Studio can be run locally and opened in your browser on Mac, Windows, and Linux.
-   See documentation [here](https://langchain-ai.github.io/langgraph/concepts/langgraph_studio/#deployed-application) on the local Studio development server and [here](https://langchain-ai.github.io/langgraph/how-tos/local-studio/#run-the-development-server).

Graphs for LangGraph Studio are in the module-x/studio/ folders.
To start the local development server, run the following command in your terminal in the /studio directory each module:

```bash
langgraph dev
```

You should see the following output:

-   🚀 API: <http://127.0.0.1:2024>
-   🎨 Studio UI: <https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024>
-   📚 API Docs: <http://127.0.0.1:2024/docs>

Open your browser and navigate to the Studio UI: <https://smith.langchain.com/studio/?baseUrl=http://127.0.0.1:2024>

-   To use Studio, you will need to create a .env file with the relevant API keys
-   Run this from the command line to create these files for module 1 to 6, as an example:

<!--listend-->

```bash
for i in {1..6}; do
  cp module-$i/studio/.env.example module-$i/studio/.env
  echo "OPENAI_API_KEY=\"$OPENAI_API_KEY\"" > module-$i/studio/.env
done
echo "TAVILY_API_KEY=\"$TAVILY_API_KEY\"" >> module-4/studio/.env
```


## Reference List {#reference-list}

1.  <https://academy.langchain.com/>
