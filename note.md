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


## spaCy + Lanchain

To achieve this, you can combine spaCy for sentence tokenization and Langchain's `CharacterTextSplitter` with a custom length function to ensure that complete sentences are included within each chunk. Here's an example of how you can implement this:

```python
import spacy
from langchain.text_splitter import CharacterTextSplitter

# Load the spaCy language model
nlp = spacy.load("en_core_web_sm")

# Define the function to get the length of a sentence
def get_sentence_length(sentence):
    return len(sentence.text)

# Read the contents from a file
with open("input_file.txt", "r") as file:
    document = file.read()

# Process the document with spaCy
doc = nlp(document)

# Store sentence-level details
sentence_details = []
for sent in doc.sents:
    sentence_details.append({
        "text": sent.text,
        "start": sent.start_char,
        "end": sent.end_char
    })

# Define the maximum context size
max_context_size = 5000

# Create a custom text splitter
text_splitter = CharacterTextSplitter(
    chunk_size=max_context_size,
    chunk_overlap=0,
    length_function=get_sentence_length,
    separators=["\n\n", "\n", " ", ""]  # Sentence boundaries
)

# Split the document into chunks, ensuring complete sentences
chunks = text_splitter.split_text(document)

# Process each chunk with LLM
for chunk in chunks:
    # Send the chunk to LLM for processing
    print(f"Chunk: {chunk}")
```

Here's a breakdown of the code:

1. Load the spaCy language model using `spacy.load("en_core_web_sm")`.
2. Define a custom function `get_sentence_length` that takes a spaCy `Span` object (representing a sentence) and returns the length of the sentence's text.
3. Read the contents from a file named `input_file.txt`.
4. Process the document with spaCy using `nlp(document)` to obtain the `doc` object.
5. Store sentence-level details (text, start offset, end offset) in a list called `sentence_details`.
6. Define the maximum context size (`max_context_size`) for the LLM.
7. Create a custom `CharacterTextSplitter` instance from Langchain, specifying the `chunk_size` as `max_context_size`, `chunk_overlap` as 0 (no overlap), and the custom `length_function` as `get_sentence_length`. Additionally, provide a list of `separators` that include common sentence boundaries like newlines and spaces.
8. Split the document into chunks using `text_splitter.split_text(document)`. This will ensure that each chunk contains complete sentences and does not exceed the specified `max_context_size`.
9. Process each chunk by sending it to the LLM. In the example, we simply print the chunk text.

By using the custom `length_function` and providing sentence boundaries as `separators`, the `CharacterTextSplitter` will split the document into chunks that contain complete sentences while respecting the specified context size limit.

The `sentence_details` list can be used to map the sentence-level information (text, start offset, end offset) back to the original document, if needed.

Note that this approach assumes that the input document is well-formatted with proper sentence boundaries. If the document has irregular or inconsistent sentence structures, you may need to adjust the `separators` list or consider using a different text splitter or approach.
