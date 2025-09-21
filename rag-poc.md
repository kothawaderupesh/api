package com.example.oracle_rag_poc;

import java.util.*;
import io.github.cdimascio.dotenv.Dotenv;

// Step 1: Load env variables and init clients
// Copilot: write static variables for Dotenv dotenv, Oracle DataSource, ElasticsearchClient esClient

public class App {

    // Step 2: Fetch sample data from Oracle
    // Copilot: write method fetchCustomers(int limit) using JDBC
    //   - connect with ORACLE_USER, ORACLE_PASSWORD, ORACLE_DSN from dotenv
    //   - run "SELECT id, name, description FROM CUSTOMERS FETCH FIRST ? ROWS ONLY"
    //   - return List<Map<String,Object>>

    // Step 3: Generate embeddings (via REST API)
    // Copilot: write method generateEmbedding(String text)
    //   - POST { "text": "<text>" } to EMBEDDING_API_URL
    //   - parse { "embedding": [ ... ] } into List<Float>

    // Step 4: Create an Elasticsearch index with a dense_vector field
    // Copilot: write method createIndexIfNotExists(String indexName)
    //   - fields: id (keyword), name (text), description (text), embedding (dense_vector, dims=1536)

    // Step 5: Index Oracle data into Elasticsearch
    // Copilot: write method indexDocuments(String indexName, List<Map<String,Object>> docs)
    //   - for each doc: call generateEmbedding(doc.description)
    //   - index with fields id, name, description, embedding

    // Step 6: Semantic search
    // Copilot: write method semanticSearch(String indexName, String query, int k)
    //   - generate embedding for query
    //   - run knn search on "embedding"
    //   - return top hits with id, name, description

    // Step 7: Ask the self-hosted LLM
    // Copilot: write method answerQuestion(String indexName, String question)
    //   - call semanticSearch
    //   - build context string: "Context:\n<doc1>\n...\nQuestion: <question>"
    //   - POST { "prompt": "<context>" } to LLM_API_URL
    //   - parse { "answer": "..." } and return

    public static void main(String[] args) throws Exception {
        // Step 8: Main orchestration
        // Copilot: load env, create index, fetch some customers, index them,
        // then run Scanner loop: read user question, call answerQuestion, print result
    }
}



Static env & clients setup → Oracle DataSource + ElasticsearchClient + Dotenv.
	2.	fetchCustomers → JDBC query into List<Map<String,Object>>.
	3.	generateEmbedding → HTTP POST to your self-hosted embedding API.
	4.	createIndexIfNotExists → ES index with dense_vector.
	5.	indexDocuments → Loop docs → embed + index in ES.
	6.	semanticSearch → Query embedding + knn search in ES.
	7.	answerQuestion → Build context + POST to your LLM /generate endpoint.
	8.	main() → Glue code + user loop.


// Step 3: Generate embeddings (via REST API)
// Copilot: write method generateEmbedding(String text) that:
//   - uses Apache HttpClient5
//   - sends a POST request to EMBEDDING_API_URL (from .env)
//   - request body: { "text": "<the input text>" }
//   - parses JSON response with Jackson
//   - response looks like: { "embedding": [0.12, -0.34, 0.98, ...] }
//   - return the embedding as List<Float>



public static List<Float> generateEmbedding(String text) throws Exception {
    String url = dotenv.get("EMBEDDING_API_URL");

    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(Map.of("text", text));

    try (CloseableHttpClient client = HttpClients.createDefault()) {
        HttpPost request = new HttpPost(url);
        request.setEntity(new StringEntity(json, ContentType.APPLICATION_JSON));

        try (CloseableHttpResponse response = client.execute(request)) {
            String responseBody = EntityUtils.toString(response.getEntity());
            JsonNode node = mapper.readTree(responseBody);
            List<Float> embedding = new ArrayList<>();
            for (JsonNode v : node.get("embedding")) {
                embedding.add((float) v.asDouble());
            }
            return embedding;
        }
    }
}


// Step 7: Ask the self-hosted LLM (via REST API)
// Copilot: write method answerQuestion(String indexName, String question) that:
//   - calls semanticSearch(indexName, question, 5) to get top docs
//   - builds a context string like: "Context:\n<doc1>\n<doc2>\n...\nQuestion: <question>"
//   - sends POST request to LLM_API_URL (from .env) using Apache HttpClient5
//   - request body: { "prompt": "<context and question>" }
//   - response JSON looks like: { "answer": "..." }
//   - parses JSON with Jackson and returns the answer string


public static String answerQuestion(String indexName, String question) throws Exception {
    // Get relevant docs
    List<Map<String, Object>> docs = semanticSearch(indexName, question, 5);

    // Build context
    StringBuilder context = new StringBuilder("Context:\n");
    for (Map<String, Object> doc : docs) {
        context.append(doc.get("description")).append("\n");
    }
    context.append("Question: ").append(question);

    // Prepare request
    String url = dotenv.get("LLM_API_URL");
    ObjectMapper mapper = new ObjectMapper();
    String json = mapper.writeValueAsString(Map.of("prompt", context.toString()));

    try (CloseableHttpClient client = HttpClients.createDefault()) {
        HttpPost request = new HttpPost(url);
        request.setEntity(new StringEntity(json, ContentType.APPLICATION_JSON));

        try (CloseableHttpResponse response = client.execute(request)) {
            String responseBody = EntityUtils.toString(response.getEntity());
            JsonNode node = mapper.readTree(responseBody);
            return node.get("answer").asText();
        }
    }
}


// Step 6: Semantic search (query → embedding → vector search)
// Copilot: write method semanticSearch(String indexName, String query, int k) that:
//   - calls generateEmbedding(query) to get the query vector
//   - uses Elasticsearch Java client (co.elastic.clients.elasticsearch.ElasticsearchClient)
//   - runs a knn search on the "embedding" field with top k results
//   - retrieves id, name, description from the hits
//   - returns a List<Map<String,Object>> of results


public static List<Map<String, Object>> semanticSearch(String indexName, String query, int k) throws Exception {
    List<Float> queryVector = generateEmbedding(query);

    SearchResponse<Map> response = esClient.search(s -> s
            .index(indexName)
            .knn(knn -> knn
                .field("embedding")
                .queryVector(queryVector)
                .k(k)
                .numCandidates(100)
            )
            .source(src -> src
                .filter(f -> f.includes(Arrays.asList("id", "name", "description")))
            ),
        Map.class
    );

    List<Map<String, Object>> results = new ArrayList<>();
    for (Hit<Map> hit : response.hits().hits()) {
        results.add(hit.source());
    }
    return results;
}


With this, your Copilot-assisted Java PoC has all critical pieces:
	1.	Oracle fetch (fetchCustomers)
	2.	Embeddings via REST (generateEmbedding)
	3.	Indexing into Elasticsearch (indexDocuments)
	4.	Vector search (semanticSearch)
	5.	Ask LLM (answerQuestion)
	6.	Main orchestration
