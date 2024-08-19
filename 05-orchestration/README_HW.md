## Homework: LLM Orchestration and Ingestion

> It's possible that your answers won't match exactly. If it's the case, select the closest one.

Our FAQ documents change with time: students add more records
and edit existing ones. We need to keep our index in sync.

There are two ways of doing it:

1. Incremental: you only update records that got changed, created or deleted
2. Full update: you recreate the entire index from scratch

In this homework, we'll look at full update. We will run our 
indexing pipeline daily and re-create the index from scracth 
each time we run. 


For that, we created two FAQ documents for LLM Zoomcamp

* [version 1](https://docs.google.com/document/d/1qZjwHkvP0lXHiE4zdbWyUXSVfmVGzougDD6N37bat3E/edit)
* [version 2](https://docs.google.com/document/d/1T3MdwUvqCL3jrh3d3VCXQ8xE0UqRzI3bfgpfBq3ZWG0/edit)

First, we will run our ingestion pipeline with version 1 
and then with version 2.

## Q1. Running Mage

Clone the same repo we used in the module and run mage:


```bash
git clone https://github.com/mage-ai/rag-project
```

Add the following libraries to the requirements document:

```
python-docx
elasticsearch
```

Make sure you use the latest version of mage:

```bash
docker pull mageai/mageai:llm
```

Start it:

```bash
./scripts/start.sh
```

Now mage is running on [http://localhost:6789/](http://localhost:6789/)

What's the version of mage? 
### Answer
```
v0.9.72
```



## Creating a RAG pipeline

Create a RAG pipeline


## Q2. Reading the documents

Now we can ingest the documents. Create a custom code ingestion
block 

Let's read the documents. We will use the same code we used
for parsing FAQ: [parse-faq-llm.ipynb](parse-faq-llm.ipynb)


Use the following document_id: 1qZjwHkvP0lXHiE4zdbWyUXSVfmVGzougDD6N37bat3E

Which is the document ID of
[LLM FAQ version 1](https://docs.google.com/document/d/1qZjwHkvP0lXHiE4zdbWyUXSVfmVGzougDD6N37bat3E/edit)

Copy the code to the editor
How many FAQ documents we processed?

* 1
* 2
* 3
* 4
### Answer
```
1
```
## Q3. Chunking

We don't really need to do any chuncking because our documents
already have well-specified boundaries. So we just need
to return the documents without any changes.

So let's go to the transformation part and add a custom code
chunking block:

```python
documents = []

for doc in data['documents']:
    doc['course'] = data['course']
    # previously we used just "id" for document ID
    doc['document_id'] = generate_document_id(doc)
    documents.append(doc)

print(len(documents))

return documents
```


Where `data` is the input parameter to the transformer.

And the `generate_document_id` is defined in the same way
as in module 4:

```python
import hashlib

def generate_document_id(doc):
    combined = f"{doc['course']}-{doc['question']}-{doc['text'][:10]}"
    hash_object = hashlib.md5(combined.encode())
    hash_hex = hash_object.hexdigest()
    document_id = hash_hex[:8]
    return document_id
```

Note: if instead of a single dictionary you get a list, 
add a for loop:

```python
for course_dict in data:
    ...
```

You can check the type of `data` with this code:

```python
print(type(data))
```

How many documents (chunks) do we have in the output?

* 66
* 76
* 86
* 96

### Answer
```
86
```


## Tokenization and embeddings

We don't need any tokenization, so we skip it.

Because currently it's required in mage, we can create 
a dummy code block:

* Create a custom code block
* Don't change it

Because we will use text search, we also don't need embeddings,
so skip it too.

If you want to use sentence transformers - the ones from module
3 - you don't need tokenization, but need embeddings
(you don't need it for this homework)


## Q4. Export

Now we're ready to index the data with elasticsearch. For that,
we use the Export part of the pipeline

* Go to the Export part
* Select vector databases -> Elasticsearch
* Open the code for editing

Because we won't use vector search, but usual text search, we
will need to adjust the code.

First, let's change the line where we read the index name:

```python
index_name = kwargs.get('index_name', 'documents')
``` 

To `index_name_prefix` - we will parametrize it with the day
and time we run the pipeline

```python
from datetime import datetime

index_name_prefix = kwargs.get('index_name', 'documents')
current_time = datetime.now().strftime("%Y%m%d_%M%S")
index_name = f"{index_name_prefix}_{current_time}"
print("index name:", index_name)
```


We will need to save the name in a global variable, so it can be accessible in other code blocks

```python
from mage_ai.data_preparation.variable_manager import set_global_variable

set_global_variable('YOUR_PIPELINE_NAME', 'index_name', index_name)
```

Where your pipeline name is the name of the pipeline, e.g.
`transcendent_nexus` (replace the space with underscore `_`)



Replace index settings with the settings we used previously:

```python
index_settings = {
    "settings": {
        "number_of_shards": number_of_shards,
        "number_of_replicas": number_of_replicas
    },
    "mappings": {
        "properties": {
            "text": {"type": "text"},
            "section": {"type": "text"},
            "question": {"type": "text"},
            "course": {"type": "keyword"},
            "document_id": {"type": "keyword"}
        }
    }
}
```

Remove the embeddings line:

```python
if isinstance(document[vector_column_name], np.ndarray):
    document[vector_column_name] = document[vector_column_name].tolist()
```

At the end (outside of the indexing for loop), print the last document:

```python
print(document)
```

Now execute the block.

What's the last document id?

Also note the index name.

### Answer
```
index name: documents_20240819_1027
index name: documents_20240819_1027
Connecting to Elasticsearch at https://studious-memory-7qp5gq9qqp627w-9200.app.github.dev/
Connecting to Elasticsearch at https://studious-memory-7qp5gq9qqp627w-9200.app.github.dev/
Index created with properties: {'settings': {'number_of_shards': 1, 'number_of_replicas': 0}, 'mappings': {'properties': {'text': {'type': 'text'}, 'section': {'type': 'text'}, 'question': {'type': 'text'}, 'course': {'type': 'keyword'}, 'document_id': {'type': 'keyword'}}}}
Indexing 86 documents to Elasticsearch index documents_20240819_1027
Indexing document 97872393
Index created with properties: {'settings': {'number_of_shards': 1, 'number_of_replicas': 0}, 'mappings': {'properties': {'text': {'type': 'text'}, 'section': {'type': 'text'}, 'question': {'type': 'text'}, 'course': {'type': 'keyword'}, 'document_id': {'type': 'keyword'}}}}
Indexing 86 documents to Elasticsearch index documents_20240819_1027
Indexing document 97872393
Indexing document a57f9581
Indexing document a57f9581
Indexing document 18a32cec
Indexing document 764f2789
Indexing document 18a32cec
Indexing document 764f2789
Indexing document fb81c6ff
Indexing document fb81c6ff
Indexing document baae926f
Indexing document 190fc999
Indexing document baae926f
Indexing document 190fc999
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 1a9b8b53
Indexing document a310259a
Indexing document 1a9b8b53
Indexing document a310259a
Indexing document 5a995cf3
Indexing document 5a995cf3
Indexing document 67da0fb5
Indexing document 67da0fb5
Indexing document 6d61aae2
Indexing document a1ec19a2
Indexing document 6d61aae2
Indexing document a1ec19a2
Indexing document d710981f
Indexing document d710981f
Indexing document 19811ec0
Indexing document bf024675
Indexing document 19811ec0
Indexing document bf024675
Indexing document aa8f7017
Indexing document aa8f7017
Indexing document 48312d67
Indexing document 1c862647
Indexing document 48312d67
Indexing document 1c862647
Indexing document fd874951
Indexing document fd874951
Indexing document 0536ca0b
Indexing document 8ac0422d
Indexing document 0536ca0b
Indexing document 8ac0422d
Indexing document cbe66cfe
Indexing document cbe66cfe
Indexing document 6ef32048
Indexing document 3ffb9e62
Indexing document 6ef32048
Indexing document 3ffb9e62
Indexing document 8efc052a
Indexing document 8efc052a
Indexing document 7b87b859
Indexing document 7b87b859
Indexing document e0d2caf7
Indexing document 5734b048
Indexing document e0d2caf7
Indexing document 5734b048
Indexing document 1804f538
Indexing document 1804f538
Indexing document f8f7469d
Indexing document 4b95ba51
Indexing document f8f7469d
Indexing document 4b95ba51
Indexing document 12f1a26a
Indexing document 12f1a26a
Indexing document a5301a1f
Indexing document aace1f4a
Indexing document a5301a1f
Indexing document aace1f4a
Indexing document db816465
Indexing document db816465
Indexing document eb00a0c9
Indexing document eb00a0c9
Indexing document 1eb85e18
Indexing document 54dd72ba
Indexing document 1eb85e18
Indexing document 54dd72ba
Indexing document 7bd989aa
Indexing document 7bd989aa
Indexing document 464b4d9c
Indexing document 464b4d9c
Indexing document 2806a1c1
Indexing document 9068bbd5
Indexing document 2806a1c1
Indexing document 9068bbd5
Indexing document ee355823
Indexing document ee355823
Indexing document 9816f1ae
Indexing document 0a101a81
Indexing document 9816f1ae
Indexing document 0a101a81
Indexing document 84ef78df
Indexing document 84ef78df
Indexing document a1419bf6
Indexing document a1419bf6
Indexing document 5f8fd79d
Indexing document 0deabb27
Indexing document 5f8fd79d
Indexing document 0deabb27
Indexing document a2dca2e2
Indexing document a2dca2e2
Indexing document 1c96a1fb
Indexing document a262c532
Indexing document 1c96a1fb
Indexing document a262c532
Indexing document 8912e711
Indexing document 8912e711
Indexing document 005ecede
Indexing document 98c1bc60
Indexing document 005ecede
Indexing document 98c1bc60
Indexing document fe48ad62
Indexing document fe48ad62
Indexing document c13c26c8
Indexing document c13c26c8
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 258a03fe
Indexing document d8c4c7bb
Indexing document 258a03fe
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 794ed89c
Indexing document 01cb301e
Indexing document 794ed89c
Indexing document 01cb301e
Indexing document e9107390
Indexing document e9107390
Indexing document 43b399a8
Indexing document befeedef
Indexing document 43b399a8
Indexing document befeedef
Indexing document 534f8148
Indexing document 534f8148
Indexing document 79f67e08
Indexing document d8c4c7bb
Indexing document 79f67e08
Indexing document d8c4c7bb
Indexing document 1fc5e366
Indexing document 1fc5e366
Indexing document 6cf805ca
Indexing document e18124d4
Indexing document 6cf805ca
Indexing document e18124d4
Indexing document a705279d
Indexing document a705279d
Indexing document f5f83001
Indexing document f5f83001
Indexing document ca68d283
Indexing document db752798
Indexing document ca68d283
Indexing document db752798
Indexing document baea0a66
Indexing document e2433e15
Indexing document baea0a66
Indexing document e2433e15
Indexing document 99ab2f5d
Indexing document 99ab2f5d
Indexing document f250bb18
Indexing document f250bb18
Indexing document d8c4c7bb
Indexing document fa136280
Indexing document d8c4c7bb
Indexing document fa136280
Indexing document 6fc3236a
Indexing document 6fc3236a
Indexing document a976d6e7
Indexing document a976d6e7


{'text': 'Prior to using Ollama models in llm-zoomcamp tasks, you need to have ollama installed on your pc and the relevant LLM model downloaded with ollama from https://www.ollama.com\nTo download ollama for Ubuntu:\n``` curl -fsSL https://ollama.com/install.sh | sh ```\nTo download ollama for Mac and Windows, follow the guide on this link:\nhttps://ollama.com/download/\nOllama a number of open-source LLMs like:\nLlama3\nPhi3\nMistral and Mixtral\nGemma\nQwen\nYou can explore more models on https://ollama.com/library/\nTo download a model in Ollama, simply open command prompt and type:\n``` ollama run model_name ```\ne.g.\n``` ollama run phi3 ```\nIt will automatically download the model and you can use it same way as above for later time.\nTo use Ollama models for inference and llm-zoomcamp tasks, use the following function:\nimport ollama\ndef llm(prompt):\nresponse = ollama.chat(\nmodel="llama3",\nmessages=[{"role": "user", "content": prompt}]\n)\nreturn response[\'message\'][\'content\']\nFor example, we can use it in the following way:\nprompt = "When does the llm-zoomcamp course start?"\nanswer = llm(prompt)\nprint(answer)', 'section': 'Module 1: Introduction', 'question': 'OpenSource: How can I use Ollama open-source models locally on my pc without using any API?', 'course': 'llm-zoomcamp', 'document_id': 'a976d6e7'}
{'text': 'Prior to using Ollama models in llm-zoomcamp tasks, you need to have ollama installed on your pc and the relevant LLM model downloaded with ollama from https://www.ollama.com\nTo download ollama for Ubuntu:\n``` curl -fsSL https://ollama.com/install.sh | sh ```\nTo download ollama for Mac and Windows, follow the guide on this link:\nhttps://ollama.com/download/\nOllama a number of open-source LLMs like:\nLlama3\nPhi3\nMistral and Mixtral\nGemma\nQwen\nYou can explore more models on https://ollama.com/library/\nTo download a model in Ollama, simply open command prompt and type:\n``` ollama run model_name ```\ne.g.\n``` ollama run phi3 ```\nIt will automatically download the model and you can use it same way as above for later time.\nTo use Ollama models for inference and llm-zoomcamp tasks, use the following function:\nimport ollama\ndef llm(prompt):\nresponse = ollama.chat(\nmodel="llama3",\nmessages=[{"role": "user", "content": prompt}]\n)\nreturn response[\'message\'][\'content\']\nFor example, we can use it in the following way:\nprompt = "When does the llm-zoomcamp course start?"\nanswer = llm(prompt)\nprint(answer)', 'section': 'Module 1: Introduction', 'question': 'OpenSource: How can I use Ollama open-source models locally on my pc without using any API?', 'course': 'llm-zoomcamp', 'document_id': 'a976d6e7'}

documents_20240819_1027, document_id': 'a976d6e7'


```

## Q5. Testing the retrieval

Now let's test the retrieval. Use mage or jupyter notebook to
test it.

Let's use the following query: "When is the next cohort?"

What's the ID of the top matching result?

### Answer
```
[{'text': 'Summer 2025 (via Alexey).', 'section': 'General course-related questions', 'question': 'When will the course be offered next?', 'course': 'llm-zoomcamp', 'document_id': 'bf024675'}, {'text': 'Cosine similarity is a measure used to calculate the similarity between two non-zero vectors, often used in text analysis to determine how similar two documents are based on their content. This metric computes the cosine of the angle between two vectors, which are typically word counts or TF-IDF values of the documents. The cosine similarity value ranges from -1 to 1, where 1 indicates that the vectors are identical, 0 indicates that the vectors are orthogonal (no similarity), and -1 represents completely opposite vectors.', 'section': 'Module 3: X', 'question': 'What is the cosine similarity?', 'course': 'llm-zoomcamp', 'document_id': 'ee355823'}, {'text': 'The error indicates that you have not changed all instances of employee_handbook to homework in your pipeline settings', 'section': 'Workshops: dlthub', 'question': 'There is an error when opening the table using dbtable = db.open_table("notion_pages___homework"): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': '6cf805ca'}, {'text': 'Make sure you open the correct table in line 3: dbtable = db.open_table("notion_pages___homework")', 'section': 'Workshops: dlthub', 'question': 'There is an error when running main(): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': 'e18124d4'}, {'text': 'This course is being offered for the first time, and things will keep changing until a given module is ready, at which point it shall be announced. Working on the material/homework in advance will be at your own risk, as the final version could be different.', 'section': 'General course-related questions', 'question': 'I was working on next weeks homework/content - why does it keep changing?', 'course': 'llm-zoomcamp', 'document_id': 'fb81c6ff'}]
[{'text': 'Summer 2025 (via Alexey).', 'section': 'General course-related questions', 'question': 'When will the course be offered next?', 'course': 'llm-zoomcamp', 'document_id': 'bf024675'}, {'text': 'Cosine similarity is a measure used to calculate the similarity between two non-zero vectors, often used in text analysis to determine how similar two documents are based on their content. This metric computes the cosine of the angle between two vectors, which are typically word counts or TF-IDF values of the documents. The cosine similarity value ranges from -1 to 1, where 1 indicates that the vectors are identical, 0 indicates that the vectors are orthogonal (no similarity), and -1 represents completely opposite vectors.', 'section': 'Module 3: X', 'question': 'What is the cosine similarity?', 'course': 'llm-zoomcamp', 'document_id': 'ee355823'}, {'text': 'The error indicates that you have not changed all instances of employee_handbook to homework in your pipeline settings', 'section': 'Workshops: dlthub', 'question': 'There is an error when opening the table using dbtable = db.open_table("notion_pages___homework"): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': '6cf805ca'}, {'text': 'Make sure you open the correct table in line 3: dbtable = db.open_table("notion_pages___homework")', 'section': 'Workshops: dlthub', 'question': 'There is an error when running main(): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': 'e18124d4'}, {'text': 'This course is being offered for the first time, and things will keep changing until a given module is ready, at which point it shall be announced. Working on the material/homework in advance will be at your own risk, as the final version could be different.', 'section': 'General course-related questions', 'question': 'I was working on next weeks homework/content - why does it keep changing?', 'course': 'llm-zoomcamp', 'document_id': 'fb81c6ff'}]

'document_id': 'bf024675'
```

## Q6. Reindexing

Our FAQ document changes: every day course participants add
new records or improve existing ones.

Imagine some time passed and the document changed. For that we have another version of the FAQ document: [version 2](https://docs.google.com/document/d/1T3MdwUvqCL3jrh3d3VCXQ8xE0UqRzI3bfgpfBq3ZWG0/edit).

The ID of this document is `1T3MdwUvqCL3jrh3d3VCXQ8xE0UqRzI3bfgpfBq3ZWG0`.

Let's re-execute the entire pipeline with the updated data.

For the same query "When is the next cohort?". What's the ID of the top matching result?

### Answer
```










index name: aak__documents_20240819_0503
index name: aak__documents_20240819_0503
Connecting to Elasticsearch at https://studious-memory-7qp5gq9qqp627w-9200.app.github.dev/
Connecting to Elasticsearch at https://studious-memory-7qp5gq9qqp627w-9200.app.github.dev/
Index created with properties: {'settings': {'number_of_shards': 1, 'number_of_replicas': 0}, 'mappings': {'properties': {'text': {'type': 'text'}, 'section': {'type': 'text'}, 'question': {'type': 'text'}, 'course': {'type': 'keyword'}, 'document_id': {'type': 'keyword'}}}}
Indexing 86 documents to Elasticsearch index aak__documents_20240819_0503
Indexing document 97872393
Index created with properties: {'settings': {'number_of_shards': 1, 'number_of_replicas': 0}, 'mappings': {'properties': {'text': {'type': 'text'}, 'section': {'type': 'text'}, 'question': {'type': 'text'}, 'course': {'type': 'keyword'}, 'document_id': {'type': 'keyword'}}}}
Indexing 86 documents to Elasticsearch index aak__documents_20240819_0503
Indexing document 97872393
Indexing document a57f9581
Indexing document a57f9581
Indexing document 18a32cec
Indexing document 764f2789
Indexing document 18a32cec
Indexing document 764f2789
Indexing document fb81c6ff
Indexing document fb81c6ff
Indexing document baae926f
Indexing document 190fc999
Indexing document baae926f
Indexing document 190fc999
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 1a9b8b53
Indexing document a310259a
Indexing document 1a9b8b53
Indexing document a310259a
Indexing document 5a995cf3
Indexing document 5a995cf3
Indexing document 67da0fb5
Indexing document 6d61aae2
Indexing document 67da0fb5
Indexing document 6d61aae2
Indexing document a1ec19a2
Indexing document a1ec19a2
Indexing document d710981f
Indexing document 19811ec0
Indexing document d710981f
Indexing document 19811ec0
Indexing document b6fa77f3
Indexing document aa8f7017
Indexing document b6fa77f3
Indexing document aa8f7017
Indexing document 48312d67
Indexing document 48312d67
Indexing document 1c862647
Indexing document 1c862647
Indexing document fd874951
Indexing document 0536ca0b
Indexing document fd874951
Indexing document 0536ca0b
Indexing document 8ac0422d
Indexing document cbe66cfe
Indexing document 8ac0422d
Indexing document cbe66cfe
Indexing document 6ef32048
Indexing document 6ef32048
Indexing document 3ffb9e62
Indexing document 3ffb9e62
Indexing document 8efc052a
Indexing document 7b87b859
Indexing document 8efc052a
Indexing document 7b87b859
Indexing document e0d2caf7
Indexing document 5734b048
Indexing document e0d2caf7
Indexing document 5734b048
Indexing document 1804f538
Indexing document 1804f538
Indexing document f8f7469d
Indexing document 4b95ba51
Indexing document f8f7469d
Indexing document 4b95ba51
Indexing document 12f1a26a
Indexing document 12f1a26a
Indexing document a5301a1f
Indexing document aace1f4a
Indexing document a5301a1f
Indexing document aace1f4a
Indexing document db816465
Indexing document db816465
Indexing document eb00a0c9
Indexing document 1eb85e18
Indexing document eb00a0c9
Indexing document 1eb85e18
Indexing document 54dd72ba
Indexing document 54dd72ba
Indexing document 7bd989aa
Indexing document 464b4d9c
Indexing document 7bd989aa
Indexing document 464b4d9c
Indexing document 2806a1c1
Indexing document 9068bbd5
Indexing document 2806a1c1
Indexing document 9068bbd5
Indexing document ee355823
Indexing document ee355823
Indexing document 9816f1ae
Indexing document 9816f1ae
Indexing document 0a101a81
Indexing document 0a101a81
Indexing document 84ef78df
Indexing document a1419bf6
Indexing document 84ef78df
Indexing document a1419bf6
Indexing document 5f8fd79d
Indexing document 5f8fd79d
Indexing document 0deabb27
Indexing document a2dca2e2
Indexing document 0deabb27
Indexing document a2dca2e2
Indexing document 1c96a1fb
Indexing document 1c96a1fb
Indexing document a262c532
Indexing document 8912e711
Indexing document a262c532
Indexing document 8912e711
Indexing document 005ecede
Indexing document 005ecede
Indexing document 98c1bc60
Indexing document fe48ad62
Indexing document 98c1bc60
Indexing document fe48ad62
Indexing document c13c26c8
Indexing document c13c26c8
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 258a03fe
Indexing document 258a03fe
Indexing document d8c4c7bb
Indexing document d8c4c7bb
Indexing document 794ed89c
Indexing document 01cb301e
Indexing document 794ed89c
Indexing document 01cb301e
Indexing document e9107390
Indexing document e9107390
Indexing document 43b399a8
Indexing document 43b399a8
Indexing document befeedef
Indexing document 534f8148
Indexing document befeedef
Indexing document 534f8148
Indexing document 79f67e08
Indexing document d8c4c7bb
Indexing document 79f67e08
Indexing document d8c4c7bb
Indexing document 1fc5e366
Indexing document 1fc5e366
Indexing document 6cf805ca
Indexing document e18124d4
Indexing document 6cf805ca
Indexing document e18124d4
Indexing document a705279d
Indexing document a705279d
Indexing document f5f83001
Indexing document ca68d283
Indexing document f5f83001
Indexing document ca68d283
Indexing document db752798
Indexing document db752798
Indexing document baea0a66
Indexing document e2433e15
Indexing document baea0a66
Indexing document e2433e15
Indexing document 99ab2f5d
Indexing document 99ab2f5d
Indexing document f250bb18
Indexing document f250bb18
Indexing document d8c4c7bb
Indexing document fa136280
Indexing document d8c4c7bb
Indexing document fa136280
Indexing document 6fc3236a
Indexing document 6fc3236a


Indexing document a976d6e7
{'text': 'Prior to using Ollama models in llm-zoomcamp tasks, you need to have ollama installed on your pc and the relevant LLM model downloaded with ollama from https://www.ollama.com\nTo download ollama for Ubuntu:\n``` curl -fsSL https://ollama.com/install.sh | sh ```\nTo download ollama for Mac and Windows, follow the guide on this link:\nhttps://ollama.com/download/\nOllama a number of open-source LLMs like:\nLlama3\nPhi3\nMistral and Mixtral\nGemma\nQwen\nYou can explore more models on https://ollama.com/library/\nTo download a model in Ollama, simply open command prompt and type:\n``` ollama run model_name ```\ne.g.\n``` ollama run phi3 ```\nIt will automatically download the model and you can use it same way as above for later time.\nTo use Ollama models for inference and llm-zoomcamp tasks, use the following function:\nimport ollama\ndef llm(prompt):\nresponse = ollama.chat(\nmodel="llama3",\nmessages=[{"role": "user", "content": prompt}]\n)\nreturn response[\'message\'][\'content\']\nFor example, we can use it in the following way:\nprompt = "When does the llm-zoomcamp course start?"\nanswer = llm(prompt)\nprint(answer)', 'section': 'Module 1: Introduction', 'question': 'OpenSource: How can I use Ollama open-source models locally on my pc without using any API?', 'course': 'llm-zoomcamp', 'document_id': 'a976d6e7'}
Indexing document a976d6e7
{'text': 'Prior to using Ollama models in llm-zoomcamp tasks, you need to have ollama installed on your pc and the relevant LLM model downloaded with ollama from https://www.ollama.com\nTo download ollama for Ubuntu:\n``` curl -fsSL https://ollama.com/install.sh | sh ```\nTo download ollama for Mac and Windows, follow the guide on this link:\nhttps://ollama.com/download/\nOllama a number of open-source LLMs like:\nLlama3\nPhi3\nMistral and Mixtral\nGemma\nQwen\nYou can explore more models on https://ollama.com/library/\nTo download a model in Ollama, simply open command prompt and type:\n``` ollama run model_name ```\ne.g.\n``` ollama run phi3 ```\nIt will automatically download the model and you can use it same way as above for later time.\nTo use Ollama models for inference and llm-zoomcamp tasks, use the following function:\nimport ollama\ndef llm(prompt):\nresponse = ollama.chat(\nmodel="llama3",\nmessages=[{"role": "user", "content": prompt}]\n)\nreturn response[\'message\'][\'content\']\nFor example, we can use it in the following way:\nprompt = "When does the llm-zoomcamp course start?"\nanswer = llm(prompt)\nprint(answer)', 'section': 'Module 1: Introduction', 'question': 'OpenSource: How can I use Ollama open-source models locally on my pc without using any API?', 'course': 'llm-zoomcamp', 'document_id': 'a976d6e7'}







[{'text': 'Summer 2026.', 'section': 'General course-related questions', 'question': 'When is the next cohort?', 'course': 'llm-zoomcamp', 'document_id': 'b6fa77f3'}, {'text': 'Cosine similarity is a measure used to calculate the similarity between two non-zero vectors, often used in text analysis to determine how similar two documents are based on their content. This metric computes the cosine of the angle between two vectors, which are typically word counts or TF-IDF values of the documents. The cosine similarity value ranges from -1 to 1, where 1 indicates that the vectors are identical, 0 indicates that the vectors are orthogonal (no similarity), and -1 represents completely opposite vectors.', 'section': 'Module 3: X', 'question': 'What is the cosine similarity?', 'course': 'llm-zoomcamp', 'document_id': 'ee355823'}, {'text': 'The error indicates that you have not changed all instances of employee_handbook to homework in your pipeline settings', 'section': 'Workshops: dlthub', 'question': 'There is an error when opening the table using dbtable = db.open_table("notion_pages___homework"): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': '6cf805ca'}, {'text': 'Make sure you open the correct table in line 3: dbtable = db.open_table("notion_pages___homework")', 'section': 'Workshops: dlthub', 'question': 'There is an error when running main(): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': 'e18124d4'}, {'text': 'This course is being offered for the first time, and things will keep changing until a given module is ready, at which point it shall be announced. Working on the material/homework in advance will be at your own risk, as the final version could be different.', 'section': 'General course-related questions', 'question': 'I was working on next weeks homework/content - why does it keep changing?', 'course': 'llm-zoomcamp', 'document_id': 'fb81c6ff'}]
[{'text': 'Summer 2026.', 'section': 'General course-related questions', 'question': 'When is the next cohort?', 'course': 'llm-zoomcamp', 'document_id': 'b6fa77f3'}, {'text': 'Cosine similarity is a measure used to calculate the similarity between two non-zero vectors, often used in text analysis to determine how similar two documents are based on their content. This metric computes the cosine of the angle between two vectors, which are typically word counts or TF-IDF values of the documents. The cosine similarity value ranges from -1 to 1, where 1 indicates that the vectors are identical, 0 indicates that the vectors are orthogonal (no similarity), and -1 represents completely opposite vectors.', 'section': 'Module 3: X', 'question': 'What is the cosine similarity?', 'course': 'llm-zoomcamp', 'document_id': 'ee355823'}, {'text': 'The error indicates that you have not changed all instances of employee_handbook to homework in your pipeline settings', 'section': 'Workshops: dlthub', 'question': 'There is an error when opening the table using dbtable = db.open_table("notion_pages___homework"): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': '6cf805ca'}, {'text': 'Make sure you open the correct table in line 3: dbtable = db.open_table("notion_pages___homework")', 'section': 'Workshops: dlthub', 'question': 'There is an error when running main(): FileNotFoundError: Table notion_pages___homework does not exist.Please first call db.create_table(notion_pages___homework, data)', 'course': 'llm-zoomcamp', 'document_id': 'e18124d4'}, {'text': 'This course is being offered for the first time, and things will keep changing until a given module is ready, at which point it shall be announced. Working on the material/homework in advance will be at your own risk, as the final version could be different.', 'section': 'General course-related questions', 'question': 'I was working on next weeks homework/content - why does it keep changing?', 'course': 'llm-zoomcamp', 'document_id': 'fb81c6ff'}]





'document_id': 'b6fa77f3'


```




## Submit the results

* Submit your results here: https://courses.datatalks.club/llm-zoomcamp-2024/homework/hw5
* It's possible that your answers won't match exactly. If it's the case, select the closest one.