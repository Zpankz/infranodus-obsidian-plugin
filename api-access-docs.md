---
title: "InfraNodus API Access Points"
source: "https://support.noduslabs.com/hc/en-us/articles/13605983537692-InfraNodus-API-Access-Points"
author:
  - "[[Nodus Labs Support Center]]"
published: 2025-08-30
created: 2025-09-10
description: "All InfraNodus subscribers can use the InfraNodus API to generate AI advice and a knowledge graph from any text. The insights include..."
---
## Authorization

To authorize, you need to obtain your API token at [https://infranodus.com/api-access](https://infranodus.com/api-access) 

You will then need to add this API token to your request header using 

Authorization: Bearer your\_api\_token\_here

For example, if you're using Python, you need to make a request like this one below:

headers = {  
'Content-Type': 'application/json',  
'Authorization': 'Bearer your\_api\_token\_here'  
}

## 1\. Obtain Graph, Statements, and Graph Summary (optional: Save Graph to InfraNodus)

This API endpoint lets you submit a text as a string or as an array of statements (via POST request) and obtain a graph and statements JSON object as a response, as well as a graph summary and insights about the main topical clusters, main concepts, gaps, and conceptual gateways in the text.

Alternatively, it can be used to specify the name of the existing user's graph in InfraNodus and to generate a knowledge graph JSON and summary, which can be used to enhance AI RAG workflow responses.

**POST request URL:**

[https://infranodus.com/api/v1/graphAndStatements](https://infranodus.com/api/v1/graphAndStatements)

**POST request parameters** (added to the URL query):

doNotSave: boolean (default: true)  
addStats: boolean (default: true)  
includeStatements: \[optional\], boolean, true by default  
includeGraphSummary: \[optional\], boolean, false by default  
extendedGraphSummary: \[optional\], boolean, true by default // returns information about content gaps, main topics, concepts  
includeGraph: \[optional\], boolean, true by default  
gapDepth: \[optional\], number (0 by default) // increase up to 3 iterate through gaps

**Privacy notice:**

Note that by default (when you're not setting the **\`doNotSave\`** flag to false), **your text will NOT be saved** to your graphs or in our database. We will also **keep no log of your request,** and it will only be processed on the InfraNodus servers in the **EU** (AWS Ireland cluster), according to the sctrict EU privacy rules. The graph object and the statements will be sent back. No trace of the processing will be kept in our system, which means that every time you make a request, the processing will be performed again. If you're using AI features with the request (the *aiTopics* parameter in the POST request body below that generates AI names for the topics), the concepts used in the top clusters (but not your actual content) will be sent to the OpenAI servers for processing. Their API terms of use state that they don't use this data for training and will only store it for 6 months. 

**POST request body:**

{   
   name: (optional) string (e.g. "name\_of\_your\_graph"),   
   text: string (e.g. "text\_string\_to\_process") // if not specified, the system will retrieve the graph with the name specified above from the database  
   statements: \[array\], optional (e.g. \['saas companies','saas blocks'\])   
   timestamps: \[array\], optional (in Date format)  
   aiTopics: (optional), false // if true, will generate topic cluster names using AI  
   modifyAnalyzedText: \[optional\], string, 'detectEntities' or 'extractEntitiesOnly' - this will detect entities in original text and produce a sparser graph. If empty, the analysis is happening at the level of granular concepts.  
   replaceEntities: \[optional, boolean, false by default\] // if true, will replace entities with the most general ontological definition  
   userName: \[optinal, empty by default\], string // if provided, request a public graph of this user  
}

If **\`statements\`** is submitted, it will override the **\`text\`** parameter.

If **no \`text\` or \`statements\` parameter is specified**, InfraNodus will **retrieve the graph of the context specified in the "name" field, if it exists.**

Additionally, you can also use the following POST request parameters to change the way your text is processed. It may be useful if you use special syntax (like \[\[wikilinks\]\] or #hashtags).

"contextSettings": {  
   "partOfSpeechToProcess": "HASHTAGS\_AND\_WORDS",  
   "doubleSquarebracketsProcessing": "PROCESS\_AS\_MENTIONS",  
   "squareBracketsProcessing": "IGNORE\_BRACKETS",  
   "mentionsProcessing": "CONNECT\_TO\_ALL\_CONCEPTS"  
}

You can learn more about it here: [Changing Text Processing Settings in InfraNodus](https://support.noduslabs.com/hc/en-us/articles/12627684289564-Changing-the-Text-Processing-Settings-in-InfraNodus)

**InfraNodus Response:**

The returned object will be in the JSON format, following the [schema detailed here](https://support.noduslabs.com/hc/en-us/articles/13605191136924). If you do not add \`includeGraph=false\` to your request, the response will contain a JSON graph object. Note that we use a modified version of the [Graphology JSON graph](https://graphology.github.io/) with some additional metrics and gaps. This means you can use Graphology and related libraries, such as Gephi or Sigma.Js to manipulate / visualize this data.

If you specify \`includeStatements\`, the return object will also include the statement chunks with meta data (e.g. the clusters they belong to).

If you specify \`includeGraphSummary\`, the return object will include a string \`graphSummary\` that will contain general information about the main clusters, concepts, and gaps in the graph, which you can use to augment your AI prompts. 

## 2\. Obtain AI-Generated Advice for a Text

This API endpoint lets you submit plain text and obtain an AI-generated question, statement, or summary that takes the underlying graph structure into account to generate precise results. This endpoint also returns the graph, so you can use it with other requests to produce different insights.

**POST request URL:**

[https://infranodus.com/api/v1/graphAndAdvice](https://infranodus.com/api/v1/graphAndStatements)

**POST request parameters** (added to the URL query):

doNotSave: boolean (default: true)  
addStats: boolean (default: true)  
optimize: \[optional\], string (develop (default) | reinforce | gap | imagine)   
transcend: \[optional\], if present, do not include graph structure into prompt  
includeStatements: \[optional\], boolean, false by default  
includeGraphSummary: \[optional\], boolean, false by default  
extendedGraphSummary: \[optional\], boolean, true by default // returns information about content gaps, main topics, concepts  
includeGraph: \[optional\], boolean, true by default  
gapDepth: \[optional\], integer, 0 by default (which gap to use for for advice)  
extendedAdvice: \[optiona\], boolean, false by default // when true, will generate advice for 3 different gaps (starting from gapDepth param)  
useContextDescription: \[optional\], boolean, false by default // when true will use the context description as system prompt (attention! this may decrease the quality of responses)

When using \`**optimize=develop**\` the algorithm will get the top nodes from all the clusters of the graph, helping develop the current discourse. Otherwise, it focuses on pinnedNodes and topicsToProcess

When using \`**optimize=reinforce**\` the algorithm will get the top nodes from the top clusters of the graph, reinforcing the current discourse. 

When using \`**optimize=gaps**\` the algorithm will identify the gaps in the discourse graph structure and direct the prompt to bridge those gaps. When the graph has a diverse topical structure, the focus will be made on the most commonly appearing gap. When the graph is highly connected, the algorithm will encourage to explore the least connected topics in order to shift attention towards the periphery.

You can also communicate the depth of the gap that you want to go to with the additional **gapDepth** parameter. The higher the number, the further gap will be selected, the smaller topics will come into focus. Good for reiteratively revealing non-obvious relations. If left empty, a random gap will be chosen.

When using \`**optimize=latent**\` the algorithm will identify the ideal points of entry into the graph or a discourse and that can also potentially connect it to other discourses, thus helping explore beyond the periphery.

**NOTE:** To learn more about the "optimize" parameter, you can check the [portable GraphRAG article](https://support.noduslabs.com/hc/en-us/articles/19226701201436) where we explain how those modes work using a visualization.

**POST request body:**

{  
  name: string, required (e.g. "name\_of\_your\_graph"), // if you're using doNotSave=true mode, you can provide any value here  
  text: string, required (e.g. "text string to process"), // if empty, use existing graph if found using the name field above   
  requestMode: '', // string, optional. Options: 'question' | 'idea' | 'fact' | 'continue' | 'challege' | 'response' | 'gptchat' | 'summary' | 'graph summary' | 'reprompt'. If none provided, the prompt is sent as it is, the responses will be short  
  modelToUse 'gpt-4o', // (string, optional, options: 'gpt-4o' | 'gpt-4o-mini' | 'gpt-5' | 'gpt-5-mini' | 'claude-opus-4.1' | 'claude-sonnet-4' | 'gemini-2.5-pro' | 'gemini-2.5-flash' | 'gemini-2.5-flash-lite')  
  pinnedNodes: \['node name 1', node name 2'\], // array, optional, focus on specific nodes in the graph  
  prompt: string, optional (e.g. "what is this text about?")  
  aiTopics: false // boolean, optional, default: false — add AI topic names to top\_clusters  
  modifyAnalyzedText: (optional) '' | 'detectEntities' | 'extractEntitiesOnly'  
  replaceEntities: false // boolean, optional  
  stopwords: \[optional\] array \["kind", "lot"\] // words to exclude from results. use -word to override default stopwords list for the language  
  systemPrompt: \[optional\] // will override the system prompt (and the useContextDescription query parameter)  
  userName: \[optinal, empty by default\], string // if provided, request a public graph of this user  
} 

**\- \`requestMode**\` — the type of AI advice you require. Use "question" to generate a research question, "response" to get a long-form response, "summary", to get a summary based on the underlying knowledge graph structure, and "graph summary" to get list of the main topical clusters identified in the graph and their summaries. Leave empty to get a generic short response.

\- \`**text\`** — note that your text will be broken into statements. If you'd like to enforce this, separate the statements in your string with a line break \`\\n\` symbol

\- **\`modifyAnalyzedText\`** — if you leave it empty or don't specify, no modification is made. If \`detectEntities\` is specified, detected entities will have \[\[wiki links\]\] syntax, if \`extractEntitiesOnly\` then the graph will be built only from detected entities

**\- \`replaceEntities\`** —  if specified and true then detected entities will be rewritten to their root form (suitable when you want to harmonize across multiple databases)

**Response:**

The returned object will be in the JSON format and contain AI-generated content in a text strings array as well as the graph produced from the content.

For \`**includeGraphSummary=false**\` (default), the return object contains the graph

{   
    "aiAdvice": \[  
       {  
          "text": "How does the perception of deliciousness in fruits influence consumer choices, and what role does the sensory coherence between fruits like apples, oranges, and lemons play in the creation of fruit blends that make sense to the palate?",  
          "finish\_reason": "stop"  
       }  
    \],  
    "graph": {  
        "graphologyGraph": {}  
    },   
    "statements": \[\],  
    "usage": integer (tokens used)  
    "created\_timestamp": timestamp (date created),  
    "userName": name of the user who generated the response,  
    "graphUrl": link to the graph url (if doNotSave query parameter is false),  
    "graphName": name of the graph (if doNotSave query parameter is false)  
}

the field \`**aiAdvice**\` will contain up to 3 options. 

the field \`**graph**\` will contain the \`graphologyGraph\` JSON object with all the graph stats

Note, that if you use **\`aiTopics\` POST body** parameter, you will get a list of AI-generated topics along with your graph object. You can use them to categorize your data or to augment your existing AI workflows with a better holistic understanding of the knowledge base context.

the field \`**statements**\` will be added if \`includeStatemnets\` request is added to the URL

the field \`**usage**\` and \`**created\_timestamp**\` contain meta information about the generated response

For **\`includeGraphSummary=true&includeGraph=false\`** only the AI advice and the summary of the graph is returned. Both can be used for augmenting the prompts in your AI RAG workflows:

"aiAdvice": \[  
   {  
     "text": "How does the perception of deliciousness in fruits influence consumer choices, and what role does the sensory coherence between fruits like apples, oranges, and lemons play in the creation of fruit blends that make sense to the palate?",  
     "finish\_reason": "stop"  
  }  
\],  
graphSummary: '' // string with the summary of the main clusters and concepts, for prompt enhancement,

## 3\. Generate DOT Graph and GraphSummary from the Graphology JSON Graph or Plain Text

This API endpoint ingests plain text or a graph in Graphology JSON format and returns a modified version of Graphviz' **DOT graph** back, which is more compact than JSON and suitable for forwarding to an LLM model for adding additional context to a query.

This endpoint also returns the **graphSummary** string, which is a more human-readable format of the main insights obtained from the full knowledge graph that can be useful for providing a more general overview of the context to an LLM model via a prompt.

The output of this endpoint allows you not only to reduce the prompt size but also to condense the most important information about your knowledge graph based on the criteria you choose in order to help the model generate a more relevant response to your query. We use it in the backend to provide additional context to the AI queries made using InfraNodus.

**POST request URL to use for sending the graph:**

https://infranodus.com/api/v1/dotGraph

**POST request URL to use for sending plain text:**

https://infranodus.com/api/v1/dotGraphFromText

**POST request parameters (added to the URL query):**

optimize: \[optional\] string (auto (default) | reinforce | develop)   
includeGraph: \[optional\] boolean // default: false, will not include JSON graph in response  
includeGraphSummary: \[optional\] boolean // default: true, will also include a string with graph summary for prompt enhancement  
extendedGraphSummary: \[optional\], boolean, true by default // will return an array with main topics, gaps, concepts

When using \`**optimize=develop**\` and when no pinnedNodes or topicsToProcess are provided , the algorithm will get the top nodes from all the clusters of the graph, helping develop the current discourse

When using \`**optimize=reinforce**\` and when no pinnedNodes or topicsToProcess are provided, the algorithm will get the top nodes from the top clusters of the graph, reinforcing the current discourse.

When using \`**optimize=gaps**\` and when no pinnedNodes or topicsToProcess are provided, the algorithm will focus on the structural gaps to help bridge the content

When using \`**optimize=latent**\` the algorithm will identify the ideal points of entry into the graph or a discourse and that can also potentially connect it to other discourses, thus helping explore beyond the periphery

**POST request body for sending a graph:**

 {   
    pinnedNodes: \[\],   
    topicsToProcess: \[\],   
    name: (required for /dotGraphFromText endpoint) 'text name',   
    text: 'plain text string', // if empty, will use the graph found with the name parameter above  
    graph: (required for /dotGraph endpoint) object {   
       nodes: \[\],   
       edges: \[\],   
       graph: { top\_nodes: \[\], top\_clusters: \[\], gaps: \[\], statementHashtags: \[\] }   
    },   
    statements: \[  
      {  
       content: 'content of statement',  
       topStatementCommunity: '3', // most keywords in the statement are in that community  
       topStatementOfCommunity: '2', // this statement is the top statement of this community  
       statementHashtags: \['word1', 'word2'\] // words identified in this statement   
      }  
    \], // array of objects, optional. use to give additional context to the graph  
   }

\- \`**graph**\` (required) should contain the nodes and edges in Graphology JSON format

– \`**pinnedNodes**\` is an optional array listing the nodes that should be extracted from their graphs (and their relations)

— \`**topicsToProcess**\` is an optional array of cluster ids (integers) to be extracted from the graph

If no pinnedNodes and  topicsToProcess are provided, InfraNodus will perform automatic extraction of the most suitable parts of the graph based on the structure of your graph ([read more about the logic here](https://support.noduslabs.com/hc/en-us/articles/360013777139)).

\- **\`modifyAnalyzedText\`** — if you leave it empty or don't specify, no modification is made. If \`detectEntities\` is specified, detected entities will have \[\[wiki links\]\] syntax, if \`extractEntitiesOnly\` then the graph will be built only from detected entities

**\- \`replaceEntities\`** —  if specified and true then detected entities will be rewritten to their root form (suitable when you want to harmonize across multiple databases)

**POST request body for sending a text:**

 {   
    name: "name of the text",  
    text: "apple and oranges are delicious fruits \\n however, oranges are delicious and lemons also make sense",  
    stopwords: \["car", "tool"\],  
    aiTopics: true // optional, show AI-generated names for topical clusters  
  }

\- **\`name\`** — required, the name of the text graph created (can be a random string in case the graph is not saved)

\- \`**text\`** — required, a string with text. Note that your text will be broken into statements. If you'd like to enforce this, separate the statements in your string with a line break \`\\n\` symbol

\- **'stopwords'** — optional, an array that can be used to exclude some words from analysis. Useful for processing the results of search queries, which are biased towards the search terms.

\- **\`aiTopics\`** — optional, boolean, whether AI topic names should be added to the clusters (default: false)

**Response:**

The returned value will be in the string format and contain information about the main relations identified in your original JSON graph object:

{  
    graphKeywords: "apple <-> orange \[label="delicious, fruit"\]" and "lemon <-> make \[label="sense"\]   
and orange <-> lemon \[label="make, sense"\], apple <-> orange \[label="delicious, fruit"\]",  
    clusterIds: \[\],  
    clusterDistance: 0,  
    clusterKeywords:  "\\"delicious orange apple fruit\\" and \\"lemon make sense\\"",  
    allClusters: \[  
       {  
          "text": "delicious orange apple fruit"  
       },  
       {   
           "text": "lemon make sense"  
       }  
    \],  
    convertedGraph: // graphObject (see above),  
    graphSummary: '' // string with the summary of the main clusters and concepts, for prompt enhancement,  
    bigrams:  \[  
       "orange <-> delicious",  
       "apple <-> orange",  
       "delicious <-> fruit",  
       "apple <-> delicious",  
       "orange <-> fruit",  
       "apple <-> fruit",  
       "lemon <-> make",  
       "make <-> sense",  
       "lemon <-> sense"  
    \]  
}

## 4\. Obtain AI-Generated Advice Based on the Knowledge Graph 

This API endpoint lets you submit a knowledge graph in InfraNodus format and obtain an AI-generated question or idea that takes the structural and semantic information into account to generate a response. 

**POST request URL:**

[https://infranodus.com/api/v1/graphAiAdvice](https://infranodus.com/api/v1/graphAndStatements)

**POST request parameters** (added to the URL query):

optimize: \[optional\], string (gaps (default) | reinforce | develop | imagine)   
transcend: \[optional\], if present, go beyond the graph structure

When using \`**optimize=develop**\` the algorithm will get the top nodes from all the clusters of the graph, helping develop the current discourse. Otherwise, it focuses on pinnedNodes and topicsToProcess

When using \`**optimize=reinforce**\` the algorithm will get the top nodes from the top clusters of the graph, reinforcing the current discourse. 

When using \`**optimize=gaps**\` the algorithm will identify the gaps in the discourse graph structure and direct the prompt to bridge those gaps. When the graph has a diverse topical structure, the focus will be made on the most commonly appearing gap. When the graph is highly connected, the algorithm will encourage to explore the least connected topics in order to shift attention towards the periphery. You can also communicate the depth of the gap that you want to go to with the additional gapDepth parameter. The higher the number, the further gap will be selected, the smaller topics will come into focus. Good for reiteratively revealing non-obvious relations. If left empty, a random gap will be chosen.

When using \`**optimize=latent**\` the algorithm will identify the ideal points of entry into the graph or a discourse and that can also potentially connect it to other discourses, thus helping explore beyond the periphery.

**NOTE:** To learn more about the "optimize" parameter, you can check the [portable GraphRAG article](https://support.noduslabs.com/hc/en-us/articles/19226701201436) where we explain how those modes work using a visualization.

**POST request body:**

{  
  prompt: '', // string, required if userPrompt not present. Prompt to send to the graph  
  userPrompt: \[  
          {  
             role: 'user',  
             content: '' // string, prompt to send to the graph  
          }  
  \], // array, required if prompt not present. User prompt for a chat-based model (when gpt-4 is used)  
  promptContext: '', // string, optional. Text context to send with the graph (max 36Kb)  
  promptChatContext: \[  
          {  
             role: 'user',   
             content: '' // string with the context of previous messages  
          },  
  \], // array, optional. Previous chat messages for the context, optional, max 56Kb  
  requestMode: '', // string, optional. Options: 'question' | 'idea' | 'fact' | 'continue' | 'challege' | 'response' | 'gptchat' | 'summary' | 'graph summary'. When empty, generic short response is provided.  
  promptGraph: '', // string, optional. Will be automatically generated if none is provided  
  language: '', // string, optional, options: 'EN', 'FR', 'DE', 'CN', 'TW', 'JP', 'ES', 'PT'  
  modelToUse 'gpt-4o', (string, optional, options: 'gpt-4o' | 'gpt-4o-mini' | 'gpt-5' | 'gpt-5-mini' | 'claude-opus-4.1' | 'claude-sonnet-4' | 'gemini-2.5-pro' | 'gemini-2.5-flash' | 'gemini-2.5-flash-lite')  
  pinnedNodes: \['node name 1', node name 2'\], // array, optional, focus on specific nodes in the graph  
  topicsToProcess: \['2', '4'\], // array, optional, focus on specific topics  
  graph: {   
    nodes: \[\],   
    edges: \[\],   
    attributes: {   
      top\_nodes: \[\],   
      top\_clusters: \[\],   
      gaps: \[\],   
      statementHashtags: \[\]   
    }  
  }, // object, required. Graph in Graphology JSON structure to generate insights from  
  statements: \[  
      {  
         content: 'content of statement',  
         topStatementCommunity: '3', // most keywords in the statement are in that community  
         topStatementOfCommunity: '2', // this statement is the top statement of this community  
         statementHashtags: \['word1', 'word2'\] // words identified in this statement   
      }  
   \], // array of objects, optional. use to give additional context to the graph  
}

Note, that the following parameters are required, the rest can be auto-generated:

\- graph — the JSON of the graph to query in graphology format

\- prompt (string) or userPrompt (array for chat-based models) — the LLM model prompt

\- requestMode — what should be generated (e.g. a question or a response)

\- language — the language that should be used for responding

**Response:**

The returned object will be in the JSON format and contain AI-generated content in a text strings array. 

{  
    "aiAdvice": \[  
        {  
            "text": "How does the sensory perception of \\"deliciousness\\" differ between fruits with similar acidity levels, like oranges and lemons, compared to sweetness-dominant fruits like apples?",  
             "finish\_reason": "stop"  
        }  
     \],  
    "usage": 338,  
    "created\_timestamp": 1722261741  
}

## 5\. Build a Graph from Statements Containing a Search Term (WIP)

This API endpoint lets you submit a search term and get all the statements of a user containing all of those search terms and generate a graph from them. 

**POST request URL:**

[https://infranodus.com/api/v1/search](https://infranodus.com/api/v1/graphAndStatements)

**POST request body:**

{  
  query: 'graphrag', // required, search string  
  contextNames: '', // optional specify context names to search in (comma separated), leave empty to search all user's statements  
  maxNodes: 150, // optional integer, max number of nodes to show in the graph (default 150)  
  showContexts: false // optional boolean, default: false, whether to show the graphs connected to nodes  
}

**Response:**

The returned object will be a list of statements and a graph in JSON format, following the [InfraNodus return object schema](https://support.noduslabs.com/hc/en-us/articles/13605191136924).

{  
  "entriesAdded": {  
    "ids": \[\],  
    "texts": \[\],  
    "statementHashtags": {},  
    "names": {},  
    "tags": \[\],  
    "timestamps": \[\]  
},  
  "graph": {  
    "nodes": \[\],  
    "edges": \[  
         {  
             "source": "b2f375b0-69a4-5050-a4bc-8e82b59baf8a",  
             "target": "b62c828d-901a-5ccd-b2c0-a165923f9ec6",  
             "id": "c8128ddd-f55d-5a17-a341-bf176f101b3b",  
             "weight": 3,  
             "context\_matrix": {  
                  "equipment": {  
                      "184713244": 3  
                   }  
              }  
         }  
     \],  
    "graph": {  
        "nodes\_to\_statements\_map": {}  
    }  
  }  
}

...

### Appendix 1: AI Request Types

The parameter \`requestType\` is used to tell the model what kind of insight it should provide. It influences the final prompt that is sent to an LLM to produce an answer.

Possible values are:

\- '**summary**' — generates a summary of the text provided, augmented with the graph structure and your custom prompt, if provided

**\- \`graph summary\`** — generates summaries for each topical cluster identified in original text. Use it with the \`aiTopics=true\` query parameter for better results

\- '**question**' — generates a question that bridges a gap between structural gaps found in a graph (can be modified or amplified in a certain direction with a prompt and the previous context, if provided

\- '**paraphrase**' — extracts the graph's structure and paraphrases the main topics in one paragraph (like a summary but generated taking the existing context less into account and focusing more on the graph's structure and the main topics and concepts detected)

\- '**outline**' — generates a title and an outline for an article based on the graph's structure

\- '**continue**' — generates a statement that connects the concepts in the prompt taking the graph structure into account

\- '**response**' — generates a direct response to the prompt, taking the graph's structure and previous context into account (difference to "continue" — stronger connection to the graph structure and context text and conversation if sent)

\- '**idea**' — generates an innovative idea that bridges a gap between structural gaps found in a graph (can be modified or amplified with a prompt and the previous context, if provided)

\- '**fact**' — generates a factual statement that bridges a gap between structural gaps found in a graph (can be modified or amplified with a prompt and the previous context, if provided)

\- '**challenge**' — generates a challenging  statement that bridges a gap between structural gaps found in a graph (can be modified or amplified with a prompt and the previous context, if provided)

\- **'reprompt'** — will rewrite your original prompt using the information from the context — useful for prompt augmentation with GraphRAG data

\- '**custom**' — your own prompt provided in the prompt parameter value (note, that if you decide to add the graph data to it, your prompt will be followed by something like "\\n that is also related to ...", followed by a list of graph clusters and concepts — so you might want to adjust your custom prompt to fit into the style of the final instruction that will be submitted to the model)