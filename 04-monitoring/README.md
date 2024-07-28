# Homework: Evaluation and Monitoring
## Getting the data

Let's start by getting the dataset. We will use the data we generated in the module.

In particular, we'll evaluate the quality of our RAG system
with [gpt-4o-mini](https://github.com/DataTalksClub/llm-zoomcamp/blob/main/04-monitoring/data/results-gpt4o-mini.csv)


Read it:

```python
url = f'{github_url}?raw=1'
df = pd.read_csv(url)
```

We will use only the first 300 documents:


```python
df = df.iloc[:300]
```

## 