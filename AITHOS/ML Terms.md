### Inference
Inference in machine learning refers to the process of making predictions by applying the trained model to new, unseen data


### Pipeline
The [pipeline()](https://huggingface.co/docs/transformers/v4.34.0/en/main_classes/pipelines#transformers.pipeline) is the easiest and fastest way to use a pretrained model for inference
- Assign a label to a given sequence of text
	- `pipeline(task=“sentiment-analysis”)`
- Generate text given a prompt
	- `pipeline(task=“text-generation”)`

##### Usage
Start by creating an instance of `pipeline()` and specifying a task you want to use it for
```python
from transformers import pipeline
classifier = pipeline("sentiment-analysis")
```
`pipeline()` downloads and caches a default pretrained model and tokenizer for sentiment analysis 
Now you can use the classifier on your target text:
```python
classifier("We are very happy to show you the 🤗 Transformers library.")
# Output:
[{'label': 'POSITIVE', 'score': 0.9998}]
```



### Transformers
- NLP model
- Highly scalable / parallellizable 
- 