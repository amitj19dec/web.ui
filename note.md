Sure, here's a Jupyter Notebook with the entire code, including pip installs and the logic for document processing using spaCy:

```python
# Install spaCy and the English language model
import sys
!{sys.executable} -m pip install spacy
!python -m spacy download en_core_web_sm
```

```python
# Import spaCy
import spacy

# Load the English language model
nlp = spacy.load("en_core_web_sm")

# Sample document
document = "This is the first sentence. And this is another sentence with multiple words."

# Process the document
doc = nlp(document)

# Get word positions
print("Word Positions:")
for token in doc:
    print(f"Token: {token.text}, Start: {token.idx}, End: {token.idx + len(token.text)}")

print("\nSentences:")
# Get sentences
for sent in doc.sents:
    print(sent.text)
```

Here's a breakdown of the code:

1. The first code cell installs the spaCy library and the English language model (`en_core_web_sm`) using pip and the spaCy command-line tool. This step is necessary to ensure that spaCy and the required language model are available in the Jupyter Notebook environment.

2. The second code cell imports the spaCy library and loads the English language model using `spacy.load("en_core_web_sm")`.

3. A sample document is defined as a string.

4. The document is processed using the spaCy pipeline: `doc = nlp(document)`.

5. The code then prints the header "Word Positions:" and iterates over each token (word) in the document, printing the token text, start offset, and end offset.

6. After that, the code prints the header "Sentences:" and iterates over each sentence in the document, printing the text of the sentence.

When you run this Jupyter Notebook, it will first install spaCy and the English language model if they are not already installed. Then, it will process the sample document and output the word positions and sentences.

Note that the first code cell (`!{sys.executable} -m pip install spacy` and `!python -m spacy download en_core_web_sm`) uses the exclamation mark (`!`) to run system commands within the Jupyter Notebook. These commands are specific to the Jupyter Notebook environment and may not work in a regular Python script.

If you're running this code in a different environment, such as a Python script or an Azure Function, you would replace the first code cell with the appropriate pip install commands or include the spaCy library and language model in your deployment package.
