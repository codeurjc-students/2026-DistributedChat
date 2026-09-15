## Phase 1:

#### Objective: Plan ahead how to set up fuzzy search in messages while keeping the content encripted.
##### Date: 
14/09/2026
##### Tool: 
Claude Chat, Sonnet 5 Medium
##### Problem to sovle:
We want to encript the content of the messages at rest, and we want the user to be able to do a fuzzy text search on the contents of older messages (in a Direct Message or in a Server). We first planned to use Elasticsearch for the text search, but Elasticsearch does not support search over encrypted text. Searching older messages is an "uncommon" operation but required.
##### Usage: 
Conversation with Claude about how to setup fuzzy text search on the content of the messages after encripting the messages. We started from the scenario of having a PostgreSQL database where we want to store the encripted messages, and Elasticsearch for the fuzzy search. We would have a global key for encripting the messages. After a conversation, we landed on the following design:
- Blind indexing to support searching encripted data. When the server receives a message, it separates the content of the message on tokens using n-grams, and we encript them. We store them on Elasticsearch and when the user searchs we tokenize and encript the search and we do a query to Elasticsearch.
- We will have two global encription keys. One key for encrypting the messages and another for encrypting the tokens. At the time of encryption, we derive a new key from the global key and the conversation id. We do that for the message and token encription. For token encription we will use HMAC-SHA256.
- The server that receives the message will encript the message and the tokens. It will save the encrypted message to PostgreSQL and add a message in a RabbitMQ queue with the encrypted tokens. A different service will then read that message from the queue and save them into Elasticsearch. This is done to offset some load since this can be done later in time without affecting the functionality.
- When the user does a Search, the main server will do it since it has the encryption keys.
- We landed on the conclusion that this is how other systems that implements an "Enterprise Key Management” like Slack may work.
- There is a possible risk of the Elasticsearch index growing too much over time. We can have a TTL on older messages, and sharding by the conversation id or date.
- Since we have Elasticsearch on the application stack, we can use it for another non-cyphered searchs, such as members of a Server or Users in a rol.
