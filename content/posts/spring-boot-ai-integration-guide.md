---
title: "Spring Boot AI Integration: Complete Guide to Adding LLM Capabilities to Your Application"
date: 2026-06-26
draft: false
tags: ["spring-boot", "ai-integration", "langchain4j", "llm", "tutorial"]
description: "Learn how to integrate AI/LLM capabilities into existing Spring Boot applications. Covers chat, RAG, tool calling, prompt engineering, and production best practices with LangChain4j."
---

Adding AI capabilities to your Spring Boot application doesn't require rebuilding from scratch. In this comprehensive guide, you'll learn how to enhance an existing Spring Boot application with LLM-powered features using **LangChain4j**.

## What You'll Build

A customer support assistant that can:
- Answer questions about your product using documentation (RAG)
- Process natural language commands via tool calling
- Maintain conversation context across requests

## Step 1: Add Dependencies

```xml
<dependencies>
    <!-- LangChain4j with OpenAI -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-open-ai-spring-boot-starter</artifactId>
        <version>0.36.2</version>
    </dependency>
    
    <!-- For RAG with PgVector -->
    <dependency>
        <groupId>dev.langchain4j</groupId>
        <artifactId>langchain4j-pgvector</artifactId>
        <version>0.36.2</version>
    </dependency>
</dependencies>
```

## Step 2: Configure LLM Connection

```yaml
# application.yml
langchain4j:
  open-ai:
    chat-model:
      model-name: gpt-4o-mini
      temperature: 0.3
      max-tokens: 2000
      timeout: 30s
      log-requests: true
      log-responses: true
```

For cost optimization, use `gpt-4o-mini` for most tasks and `gpt-4o` only for complex reasoning.

## Step 3: Create Your First AI Service

```java
@AiService
public interface CustomerSupportAssistant {
    
    @SystemMessage("""
        You are a customer support assistant for our SaaS product.
        Be helpful, concise, and professional.
        If you don't know the answer, say so honestly.
        Always provide specific steps when giving instructions.
        """)
    String chat(@UserMessage String userMessage,
                @MemoryId String sessionId);
}
```

The `@MemoryId` parameter enables automatic conversation memory per session.

## Step 4: Add RAG (Knowledge Base)

Load your documentation into a vector store:

```java
@Configuration
public class RagConfig {
    
    @Bean
    public EmbeddingModel embeddingModel() {
        return OpenAiEmbeddingModel.builder()
            .apiKey(System.getenv("OPENAI_API_KEY"))
            .modelName("text-embedding-3-small")
            .build();
    }
    
    @Bean
    public EmbeddingStore<TextSegment> embeddingStore(DataSource dataSource) {
        return PgVectorEmbeddingStore.builder()
            .dataSource(dataSource)
            .table("document_embeddings")
            .dimension(1536)
            .build();
    }
    
    @Bean
    public ContentRetriever contentRetriever(
            EmbeddingStore<TextSegment> store,
            EmbeddingModel model) {
        return EmbeddingStoreContentRetriever.builder()
            .embeddingStore(store)
            .embeddingModel(model)
            .maxResults(5)
            .minScore(0.7)
            .build();
    }
}
```

Update your AI service to use RAG:

```java
@AiService
public interface CustomerSupportAssistant {
    
    @SystemMessage("""
        Answer questions based on the provided documentation.
        If the documentation doesn't contain the answer, say so.
        Always cite the source document name in your response.
        """)
    String chat(@UserMessage String question,
                @MemoryId String sessionId,
                @Embedding Match documentation);
}
```

## Step 5: Index Your Documents

```java
@Service
public class DocumentIndexer {
    
    private final EmbeddingModel embeddingModel;
    private final EmbeddingStore<TextSegment> embeddingStore;
    
    public void indexDocuments(Path docsDir) {
        DocumentLoader loader = DocumentLoaders.fromFileSystem(docsDir);
        List<Document> documents = loader.load();
        
        // Split into chunks
        TextSplitter splitter = DocumentByParagraphSplitter.builder()
            .maxSegmentSize(500)
            .maxOverlapSize(100)
            .build();
        
        List<TextSegment> segments = splitter.splitAll(documents);
        
        // Store with embeddings
        embeddingStore.addAll(
            embeddingModel.embedAll(
                segments.stream()
                    .map(TextSegment::text)
                    .toList()
            ).content().vectorStoreEmbeddings(),
            segments
        );
    }
}
```

## Step 6: Add Tool Calling

Let your AI assistant perform actions:

```java
public class SupportTools {
    
    private final OrderService orderService;
    private final TicketService ticketService;
    
    @Tool("Look up an order by order ID. Returns order status and details.")
    public OrderInfo lookupOrder(@P("orderId") String orderId) {
        return orderService.getOrderInfo(orderId);
    }
    
    @Tool("Create a support ticket with the given description.")
    public String createTicket(@P("subject") String subject,
                               @P("description") String description,
                               @P("priority") String priority) {
        return ticketService.create(subject, description, priority);
    }
    
    @Tool("Check the current system status and any ongoing incidents.")
    public SystemStatus checkSystemStatus() {
        return statusService.getCurrentStatus();
    }
}
```

Wire tools into your assistant:

```java
@AiService
public interface CustomerSupportAssistant {
    
    String chat(@UserMessage String question,
                @MemoryId String sessionId,
                @Embedding Match documentation,
                @ToolSet SupportTools tools);
}
```

## Step 7: REST API

```java
@RestController
@RequestMapping("/api/chat")
public class ChatController {
    
    private final CustomerSupportAssistant assistant;
    
    @PostMapping
    public ChatResponse chat(@RequestBody ChatRequest request) {
        String response = assistant.chat(
            request.getMessage(),
            request.getSessionId()
        );
        return new ChatResponse(response);
    }
    
    @PostMapping("/stream")
    public Flux<String> chatStream(@RequestBody ChatRequest request) {
        // For streaming responses
        return Flux.create(sink -> {
            String response = assistant.chat(
                request.getMessage(),
                request.getSessionId()
            );
            sink.next(response);
            sink.complete();
        });
    }
}
```

## Production Best Practices

### 1. Prompt Versioning

```java
@AiService
public interface CustomerSupportAssistant {
    
    // Use externalized prompts
    @SystemMessage(fromResource = "prompts/support-v2.txt")
    String chat(@UserMessage String question,
                @MemoryId String sessionId);
}
```

### 2. Cost Monitoring

```java
@Aspect
@Component
public class LlmCostMonitor {
    
    @Around("@annotation(dev.langchain4j.service.AiService)")
    public Object monitorCost(ProceedingJoinPoint joinPoint) throws Throwable {
        long start = System.currentTimeMillis();
        Object result = joinPoint.proceed();
        long duration = System.currentTimeMillis() - start;
        
        // Log and alert on expensive calls
        if (duration > 5000) {
            log.warn("Slow LLM call: {}ms", duration);
        }
        
        metricsService.recordLlmCall(duration);
        return result;
    }
}
```

### 3. Caching

```java
@Bean
public ChatLanguageModel cachedModel(ChatLanguageModel delegate) {
    return CachingChatLanguageModel.builder()
        .delegate(delegate)
        .expireAfterWrite(Duration.ofMinutes(10))
        .maximumSize(1000)
        .build();
}
```

### 4. Graceful Degradation

```java
@Service
public class SmartAssistant {
    
    private final CustomerSupportAssistant aiAssistant;
    private final FaqService faqService;
    
    public String respond(String question, String sessionId) {
        try {
            return aiAssistant.chat(question, sessionId);
        } catch (Exception e) {
            log.warn("AI service unavailable, falling back to FAQ", e);
            return faqService.findBestMatch(question)
                .orElse("I'm currently unable to process your request. " +
                       "A human agent will be with you shortly.");
        }
    }
}
```

## Performance Tips

1. **Use smaller models for simple tasks**: `gpt-4o-mini` handles most customer queries fine
2. **Cache frequent answers**: Questions like "What are your hours?" shouldn't hit the API every time
3. **Async processing**: Use `CompletableFuture` for non-interactive AI tasks
4. **Batch embeddings**: When indexing documents, batch your embedding API calls

## What's Next?

- Add evaluation metrics (answer quality, latency)
- Implement A/B testing for prompts
- Add support for multiple languages
- Build an admin dashboard for monitoring

## Conclusion

Adding AI to a Spring Boot application is surprisingly straightforward with LangChain4j. The key is starting simple — a basic chat endpoint — and iteratively adding RAG, tools, and production hardening.

The complete working example is available on [GitHub](https://github.com/yourusername/spring-boot-ai-guide).

*Questions? Drop them in the comments!*
