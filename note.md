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




---
You're right to ask for a recheck. Upon review, I noticed a few areas that could be improved for better reliability and security. Here's an updated and improved version of the script:

```bash
#!/bin/bash

set -e

# Update the system
yum update -y

# Install prerequisites
yum install -y yum-utils device-mapper-persistent-data lvm2 curl openssl

# Add Docker repository
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

# Install Docker
yum install -y docker-ce docker-ce-cli containerd.io

# Start Docker service
systemctl start docker
systemctl enable docker

# Pull Azure Cosmos DB Emulator image
docker pull mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator

# Get the public IP address
PUBLIC_IP=$(curl -s -H "Authorization: Bearer Oracle" -L http://169.254.169.254/opc/v1/instance/networkInterfaces/0/publicIp)

# Run Azure Cosmos DB Emulator container
docker run -d -p 8081:8081 -p 10251:10251 -p 10252:10252 -p 10253:10253 -p 10254:10254 \
  -m 3g --cpus=2.0 \
  --name=cosmosdb-emulator \
  -e AZURE_COSMOS_EMULATOR_PARTITION_COUNT=10 \
  -e AZURE_COSMOS_EMULATOR_ENABLE_DATA_PERSISTENCE=true \
  -e AZURE_COSMOS_EMULATOR_IP_ADDRESS_OVERRIDE=$PUBLIC_IP \
  mcr.microsoft.com/cosmosdb/linux/azure-cosmos-emulator

# Add firewall rules to allow incoming traffic
firewall-cmd --permanent --add-port=8081/tcp
firewall-cmd --permanent --add-port=10251-10254/tcp
firewall-cmd --reload

# Wait for the emulator to start (this may take a few minutes)
echo "Waiting for Cosmos DB Emulator to start..."
timeout 300 bash -c 'until docker logs cosmosdb-emulator 2>&1 | grep -q "Started"; do sleep 5; done' || { echo "Emulator failed to start within 5 minutes"; exit 1; }

# Fetch the Cosmos DB key
COSMOS_KEY=$(docker exec cosmosdb-emulator bash -c 'curl -k https://localhost:8081/_explorer/emulator.pem | openssl x509 -noout -fingerprint -sha256 | sed "s/://g" | cut -d= -f2')

if [ -z "$COSMOS_KEY" ]; then
    echo "Failed to retrieve Cosmos DB key"
    exit 1
fi

echo "Setup complete. Cosmos DB Emulator is accessible at ${PUBLIC_IP}:8081"
echo "Cosmos DB Key: ${COSMOS_KEY}"

# Save the key to a file
echo "${COSMOS_KEY}" > /home/opc/cosmos_db_key.txt
chmod 600 /home/opc/cosmos_db_key.txt
echo "The Cosmos DB key has been saved to /home/opc/cosmos_db_key.txt"
```

Improvements and changes:

1. Added `set -e` at the beginning to make the script exit immediately if any command fails.

2. Added installation of `curl` and `openssl` to ensure these tools are available.

3. Updated the method to fetch the public IP address to use the correct OCI metadata URL and added the required authorization header.

4. Improved the waiting mechanism for the Cosmos DB Emulator to start. It now checks the Docker logs for a "Started" message and times out after 5 minutes if the emulator doesn't start.

5. Updated the method to fetch the Cosmos DB key. It now uses SHA256 instead of SHA1 for the fingerprint, which is more secure and matches the current Cosmos DB Emulator behavior.

6. Added error checking to ensure the Cosmos DB key is successfully retrieved.

7. Changed the permissions of the key file to 600 (read/write for owner only) for better security.

This script should be more robust and secure. It will exit with an error if any critical step fails, making it easier to diagnose issues. Remember to test this script in your environment to ensure it works as expected with your specific OCI setup.
