const SILICONSTORM_API_URL = "https://api.siliconstorm.ai/aigc/chat/completions";
const SILICONSTORM_CREATE_CHAT_URL = "https://api.siliconstorm.ai/aigc/chat/create";

const conversationIdCache = new Map<string, string>();

interface OpenAIRequest {
  model: string;
  messages: Array<{role: string; content: string}>;
  stream?: boolean;
  temperature?: number;
  max_tokens?: number;
  top_p?: number;
  frequency_penalty?: number;
  presence_penalty?: number;
}

interface SiliconStormRequest {
  chatId: string;
  conversationId: string;
  appId: null;
  messages: Array<{role: string; message: string}>;
  modelId: string;
  model: string;
  modelProvider: string;
  maxTokens: number;
  temperature: number;
  topP: number;
  topK: number;
  frequencyPenalty: number;
  type: number;
  knowledgeIds: Array<any>;
}

interface CreateChatRequest {
  type: number;
  title: string;
}

interface CreateChatResponse {
  code: number;
  message: string;
  result: string;
}

const modelMapping: Record<string, { modelId: string, model: string, modelProvider: string }> = {
  "gpt-4o": { modelId: "7", model: "gpt-4o", modelProvider: "OPENAI" },
  "grok-3-think": { modelId: "9", model: "grok-3-reasoning", modelProvider: "GROK" },
};

function generateUUID(): string {
  return crypto.randomUUID();
}

async function createChat(authToken: string, title: string = "新对话"): Promise<string> {
  const createRequest: CreateChatRequest = {
    type: 0,
    title: title
  };
  
  const response = await fetch(SILICONSTORM_CREATE_CHAT_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json;charset=UTF-8",
      "Authorization": authToken,
      "Origin": "https://chat.siliconstorm.ai",
      "Referer": "https://chat.siliconstorm.ai/",
      "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36 Edg/137.0.0.0"
    },
    body: JSON.stringify(createRequest)
  });
  
  if (!response.ok) {
    const errorText = await response.text();
    throw new Error(`创建会话失败: ${errorText}`);
  }
  
  const data = await response.json() as CreateChatResponse;
  
  if (data.code !== 200) {
    throw new Error(`创建会话失败: ${data.message}`);
  }
  
  const conversationId = data.result;
  
  return conversationId;
}

async function getOrCreateConversationId(authToken: string, userContent: string): Promise<string> {
  if (conversationIdCache.has(authToken)) {
    return conversationIdCache.get(authToken)!;
  }
  
  const title = userContent.length > 20 ? userContent.substring(0, 20) + "..." : userContent;
  const conversationId = await createChat(authToken, title);
  
  conversationIdCache.set(authToken, conversationId);
  
  return conversationId;
}

async function convertToSiliconStormRequest(openaiRequest: OpenAIRequest, authToken: string): Promise<SiliconStormRequest> {
  const modelInfo = modelMapping[openaiRequest.model] || { 
    modelId: "9", 
    model: "grok-3-reasoning", 
    modelProvider: "GROK" 
  };

  const messages = openaiRequest.messages.map(msg => ({
    role: msg.role,
    message: msg.content
  }));

  const userMessage = openaiRequest.messages.find(msg => msg.role === "user");
  const userContent = userMessage ? userMessage.content : "新对话";
  
  const conversationId = await getOrCreateConversationId(authToken, userContent);
  const chatId = generateUUID();

  return {
    chatId: chatId,
    conversationId: conversationId,
    appId: null,
    messages,
    modelId: modelInfo.modelId,
    model: modelInfo.model,
    modelProvider: modelInfo.modelProvider,
    maxTokens: openaiRequest.max_tokens || 4000,
    temperature: openaiRequest.temperature || 0.7,
    topP: openaiRequest.top_p || 0.7,
    topK: 50,
    frequencyPenalty: openaiRequest.frequency_penalty || 0,
    type: 0,
    knowledgeIds: []
  };
}

function convertStreamingResponse(
  siliconstormChunk: string,
  id: string,
  created: number,
  model: string
): string {
  try {
    if (!siliconstormChunk.trim()) {
      return "";
    }

    const jsonStr = siliconstormChunk.startsWith("data:") 
      ? siliconstormChunk.slice(5).trim()
      : siliconstormChunk.trim();
    
    if (jsonStr === "[DONE]") {
      return "data: [DONE]\n\n";
    }

    if (jsonStr.startsWith("Error:")) {
      const errorResponse = {
        id,
        object: "chat.completion.chunk",
        created,
        model,
        choices: [{
          index: 0,
          delta: {
            content: jsonStr,
            role: "assistant"
          },
          finish_reason: "error"
        }],
        system_fingerprint: "",
        usage: {
          prompt_tokens: 6,
          completion_tokens: 0,
          total_tokens: 6
        }
      };
      
      return `data: ${JSON.stringify(errorResponse)}\n\n`;
    }
    
    try {
      const data = JSON.parse(jsonStr);
      
      const openaiResponse = {
        id,
        object: "chat.completion.chunk",
        created,
        model,
        choices: [{
          index: 0,
          delta: {
            content: data.message,
            reasoning_content: data.reasoning,
            role: "assistant"
          },
          finish_reason: data.done ? "stop" : null
        }],
        system_fingerprint: "",
        usage: {
          prompt_tokens: 6,
          completion_tokens: data.usedToken || 0,
          total_tokens: (data.usedToken || 0) + 6,
          completion_tokens_details: {
            reasoning_tokens: data.reasoning ? 1 : 0
          }
        }
      };

      return `data: ${JSON.stringify(openaiResponse)}\n\n`;
    } catch (parseError) {
      const errorResponse = {
        id,
        object: "chat.completion.chunk",
        created,
        model,
        choices: [{
          index: 0,
          delta: {
            content: `解析错误: ${parseError.message}. 原始数据: ${jsonStr.substring(0, 200)}...`,
            role: "assistant"
          },
          finish_reason: null
        }],
        system_fingerprint: "",
        usage: {
          prompt_tokens: 6,
          completion_tokens: 0,
          total_tokens: 6
        }
      };
      
      return `data: ${JSON.stringify(errorResponse)}\n\n`;
    }
  } catch (error) {
    const errorResponse = {
      id,
      object: "chat.completion.chunk",
      created,
      model,
      choices: [{
        index: 0,
        delta: {
          content: `处理错误: ${error.message}`,
          role: "assistant"
        },
        finish_reason: null
      }],
      system_fingerprint: "",
      usage: {
        prompt_tokens: 6,
        completion_tokens: 0,
        total_tokens: 6
      }
    };
    
    return `data: ${JSON.stringify(errorResponse)}\n\n`;
  }
}

async function handler(req: Request): Promise<Response> {
  const url = new URL(req.url);
  if (req.method === "GET" && url.pathname === "/") {
    return new Response("SiliconStorm 转 OpenAI API 代理服务正在运行", {
      headers: { "Content-Type": "text/plain;charset=UTF-8" }
    });
  }
  
  if (req.method !== "POST" || url.pathname !== "/v1/chat/completions") {
    return new Response(`未找到路径: ${url.pathname}`, { status: 404 });
  }

  try {
    const authToken = req.headers.get("Authorization") || "";
    if (!authToken) {
      return new Response(
        JSON.stringify({ error: { message: "缺少授权令牌" } }),
        { status: 401, headers: { "Content-Type": "application/json" } }
      );
    }
    
    const openaiRequest = await req.json() as OpenAIRequest;
    const isStreaming = openaiRequest.stream === true;
    const siliconstormRequest = await convertToSiliconStormRequest(openaiRequest, authToken);
    
    const response = await fetch(SILICONSTORM_API_URL, {
      method: "POST",
      headers: {
        "Content-Type": "application/json;charset=UTF-8",
        "Authorization": authToken,
        "Origin": "https://chat.siliconstorm.ai",
        "Referer": "https://chat.siliconstorm.ai/",
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/137.0.0.0 Safari/537.36 Edg/137.0.0.0"
      },
      body: JSON.stringify(siliconstormRequest)
    });

    if (!response.ok) {
      const errorText = await response.text();
      
      let errorInfo = { message: errorText };
      try {
        const errorJson = JSON.parse(errorText);
        if (errorJson.error) {
          errorInfo = errorJson.error;
        }
      } catch (e) {
      }
      
      return new Response(
        JSON.stringify({ 
          error: { 
            message: `SiliconStorm API 错误: ${errorInfo.message || errorText}`,
            status: response.status,
            statusText: response.statusText,
            details: errorInfo
          } 
        }), 
        { status: response.status, headers: { "Content-Type": "application/json" } }
      );
    }

    if (isStreaming) {
      const { readable, writable } = new TransformStream();
      const id = crypto.randomUUID();
      const created = Math.floor(Date.now() / 1000);
      const model = openaiRequest.model;
      
      (async () => {
        const writer = writable.getWriter();
        const encoder = new TextEncoder();
        const decoder = new TextDecoder();
        
        try {
          const reader = response.body!.getReader();
          let buffer = "";
          let chunkCount = 0;
          
          while (true) {
            const { done, value } = await reader.read();
            if (done) {
              break;
            }
            
            const decodedValue = decoder.decode(value, { stream: true });
            buffer += decodedValue;
            
            const chunks = buffer.split("\n\n");
            buffer = chunks.pop() || "";
            
            for (const chunk of chunks) {
              if (chunk.trim()) {
                const openaiChunk = convertStreamingResponse(chunk, id, created, model);
                if (openaiChunk) {
                  await writer.write(encoder.encode(openaiChunk));
                }
              }
            }
          }
          
          if (buffer.trim()) {
            const openaiChunk = convertStreamingResponse(buffer, id, created, model);
            if (openaiChunk) {
              await writer.write(encoder.encode(openaiChunk));
            }
          }
          
          await writer.write(encoder.encode("data: [DONE]\n\n"));
        } catch (error) {
          const errorResponse = {
            id,
            object: "chat.completion.chunk",
            created,
            model,
            choices: [{
              index: 0,
              delta: {
                content: `流处理错误: ${error.message}`,
                role: "assistant"
              },
              finish_reason: "error"
            }],
            system_fingerprint: "",
            usage: {
              prompt_tokens: 6,
              completion_tokens: 0,
              total_tokens: 6
            }
          };
          
          try {
            await writer.write(encoder.encode(`data: ${JSON.stringify(errorResponse)}\n\n`));
            await writer.write(encoder.encode("data: [DONE]\n\n"));
          } catch (writeError) {
          }
        } finally {
          try {
            await writer.close();
          } catch (closeError) {
          }
        }
      })();
      
      return new Response(readable, {
        headers: {
          "Content-Type": "text/event-stream",
          "Cache-Control": "no-cache",
          "Connection": "keep-alive",
          "Access-Control-Allow-Origin": "*"
        }
      });
    } else {
      const data = await response.json();
      
      const openaiResponse = {
        id: crypto.randomUUID(),
        object: "chat.completion",
        created: Math.floor(Date.now() / 1000),
        model: openaiRequest.model,
        choices: [{
          index: 0,
          message: {
            role: "assistant",
            content: data.message || ""
          },
          finish_reason: "stop"
        }],
        usage: {
          prompt_tokens: 0,
          completion_tokens: 0,
          total_tokens: 0
        }
      };
      
      return new Response(JSON.stringify(openaiResponse), {
        headers: { "Content-Type": "application/json" }
      });
    }
  } catch (error) {
    return new Response(
      JSON.stringify({ error: { message: `错误: ${error.message}` } }), 
      { status: 500, headers: { "Content-Type": "application/json" } }
    );
  }
}

function handleCORS(req: Request): Response | null {
  if (req.method === "OPTIONS") {
    return new Response(null, {
      headers: {
        "Access-Control-Allow-Origin": "*",
        "Access-Control-Allow-Methods": "POST, GET, OPTIONS",
        "Access-Control-Allow-Headers": "Content-Type, Authorization",
        "Access-Control-Max-Age": "86400"
      }
    });
  }
  return null;
}

async function handleRequest(req: Request): Promise<Response> {
  const corsResponse = handleCORS(req);
  if (corsResponse) return corsResponse;
  
  const response = await handler(req);
  
  const headers = new Headers(response.headers);
  headers.set("Access-Control-Allow-Origin", "*");
  
  return new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers
  });
}

Deno.serve(handleRequest);