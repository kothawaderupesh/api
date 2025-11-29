package com.example.esllm;

import java.util.*;
import java.io.*;
import io.github.cdimascio.dotenv.Dotenv;
import com.fasterxml.jackson.databind.*;
import com.fasterxml.jackson.core.type.TypeReference;
import org.apache.hc.client5.http.impl.classic.CloseableHttpClient;
import org.apache.hc.client5.http.classic.methods.HttpPost;
import org.apache.hc.client5.http.impl.classic.HttpClients;
import org.apache.hc.core5.http.io.entity.StringEntity;
import org.apache.hc.core5.http.ContentType;
import co.elastic.clients.elasticsearch.ElasticsearchClient;
import co.elastic.clients.transport.rest_client.RestClientTransport;
import org.elasticsearch.client.RestClient;
import org.elasticsearch.client.RestClientBuilder;
import co.elastic.clients.json.jackson.JacksonJsonpMapper;
import co.elastic.clients.transport.Transport;

// Copilot: write class App that:
//  - loads dotenv into a static Dotenv dotenv
//  - creates ElasticsearchClient esClient using ELASTICSEARCH_URL from env
//  - has ObjectMapper mapper = new ObjectMapper()
//  - has main(String[] args) that reads lines from stdin and for each line:
//      1) calls parseIntentWithLLM(userQuestion) to get intent JSON as Map
//      2) validates & sanitizes the intent
//      3) converts intent -> Elasticsearch DSL JSON using buildDslFromIntent(intentMap)
//      4) executes DSL using executeEsQuery(dslJson) and returns the ES response as JsonNode
//      5) formats the ES response into a natural language summary using formatAnswer(esResult, intentMap)
//  - implement the following methods with Copilot:
//      - parseIntentWithLLM(String userQuestion) : Map<String,Object>
//      - buildDslFromIntent(Map<String,Object> intent) : String (returns DSL as JSON string)
//      - executeEsQuery(String dslJson) : JsonNode
//      - formatAnswer(JsonNode esResult, Map<String,Object> intent) : String
//  - IMPORTANT: add a method validateIntent(Map<String,Object> intent) that ensures allowed fields only,
//    enforces limits (e.g., limit <= 10000), and rejects dangerous fields
public class App {

    static Dotenv dotenv;
    static ElasticsearchClient esClient;
    static ObjectMapper mapper = new ObjectMapper();

    // Copilot: write static initializer to load dotenv and initialize esClient using RestClient and JacksonJsonpMapper
    // - Use ELASTICSEARCH_URL from env

    // Copilot: write method parseIntentWithLLM(String userQuestion) that:
    //   - builds the prompt using the template provided earlier (replace <<USER_QUESTION>>)
    //   - POSTs JSON { "prompt": "<prompt>" } to LLM_API_URL
    //   - includes optional LLM_API_KEY header if present
    //   - parses the LLM response body to JSON and returns it as Map<String,Object>
    //   - handles errors and returns a safe default intent if parsing fails

    // Copilot: write method validateIntent(Map<String,Object> intent) that:
    //   - ensures "action" is one of allowed values
    //   - ensures dates are in YYYY-MM-DD
    //   - ensures aggregation.type is allowed
    //   - ensures limit <= 10000, otherwise set to 10000
    //   - removes/ignores unknown keys

    // Copilot: write method buildDslFromIntent(Map<String,Object> intent) that:
    //   - converts the intent map into an Elasticsearch DSL JSON string
    //   - when action == "aggregate", produce an ES aggregation query (size: intent.limit or 0 if aggregations)
    //   - include filters under "bool.filter"
    //   - support aggregation types: count (value_count), terms, date_histogram, sum, avg
    //   - if intent.aggregation.type == "terms" use size from aggregation or default 10

    // Copilot: write method executeEsQuery(String dslJson) that:
    //   - uses esClient._transport().restClient() or the high-level client to execute the raw DSL JSON
    //   - returns response as JsonNode using mapper.readTree(responseBody)

    // Copilot: write method formatAnswer(JsonNode esResult, Map<String,Object> intent) that:
    //   - if aggregate/terms: extract buckets and format "Impact counts: high=12, medium=5..."
    //   - if count/value_count: return "Count: X"
    //   - else return a short JSON summary + top hit preview

    public static void main(String[] args) throws Exception {
        // Copilot: implement main:
        //  - load dotenv & init esClient (use static initializer)
        //  - create a Scanner(System.in) to read questions
        //  - loop: prompt "Question> ", read line, if line is blank continue, "exit" to quit
        //  - call parseIntentWithLLM(), validateIntent(), buildDslFromIntent(), executeEsQuery(), formatAnswer()
        //  - print answer
    }
}
