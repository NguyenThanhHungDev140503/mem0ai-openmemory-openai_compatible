# Hướng dẫn cấu hình OpenAI-Compatible API cho OpenMemory

OpenMemory hỗ trợ sử dụng bất kỳ API nào tương thích với chuẩn OpenAI (OpenAI-Compatible) cho cả **LLM** và **Embedder**. Điều này cho phép bạn sử dụng các nhà cung cấp như Chutes, OpenRouter, Together AI, Groq, Minimax, v.v. thay vì OpenAI trực tiếp.

## Yêu cầu

- OpenMemory đang chạy (qua Docker hoặc local)
- API key từ nhà cung cấp bạn muốn sử dụng
- Biết model name và base URL của nhà cung cấp

## Cấu hình qua API

Gửi request `PUT` tới endpoint `/api/v1/config/` với payload JSON chứa thông tin LLM và Embedder.

### Cấu trúc payload

```json
{
  "mem0": {
    "llm": {
      "provider": "openai",
      "config": {
        "model": "<MODEL_NAME>",
        "api_key": "<API_KEY>",
        "openai_base_url": "<BASE_URL>"
      }
    },
    "embedder": {
      "provider": "openai",
      "config": {
        "model": "<EMBEDDING_MODEL_NAME>",
        "api_key": "<EMBEDDING_API_KEY>",
        "openai_base_url": "<EMBEDDING_BASE_URL>"
      }
    }
  }
}
```

**Lưu ý quan trọng:**
- `provider` luôn để là `"openai"` vì các API này tương thích chuẩn OpenAI
- `openai_base_url` là trường bắt buộc để trỏ tới endpoint của nhà cung cấp
- `temperature` và `max_tokens` là tuỳ chọn, nếu không truyền sẽ dùng giá trị mặc định
- LLM và Embedder có thể dùng **khác nhau** nhà cung cấp

---

## Ví dụ cụ thể

### 1. Chutes (LLM) + OpenRouter (Embedder)

```bash
curl -X PUT "http://localhost:8765/api/v1/config/" \
     -H "Content-Type: application/json" \
     -d '{
  "mem0": {
    "llm": {
      "provider": "openai",
      "config": {
        "model": "MiniMaxAI/MiniMax-M2.5-TEE",
        "api_key": "<CHUTES_API_KEY>",
        "openai_base_url": "https://llm.chutes.ai/v1"
      }
    },
    "embedder": {
      "provider": "openai",
      "config": {
        "model": "qwen/qwen3-embedding-8b",
        "api_key": "<OPENROUTER_API_KEY>",
        "openai_base_url": "https://openrouter.ai/api/v1"
      }
    }
  }
}'
```

### 2. Together AI (cả LLM và Embedder)

```bash
curl -X PUT "http://localhost:8765/api/v1/config/" \
     -H "Content-Type: application/json" \
     -d '{
  "mem0": {
    "llm": {
      "provider": "openai",
      "config": {
        "model": "meta-llama/Llama-3.3-70B-Instruct-Turbo",
        "api_key": "<TOGETHER_API_KEY>",
        "openai_base_url": "https://api.together.xyz/v1"
      }
    },
    "embedder": {
      "provider": "openai",
      "config": {
        "model": "togethercomputer/m2-bert-80M-8k-retrieval",
        "api_key": "<TOGETHER_API_KEY>",
        "openai_base_url": "https://api.together.xyz/v1"
      }
    }
  }
}'
```

### 3. Groq (LLM) + OpenAI (Embedder)

```bash
curl -X PUT "http://localhost:8765/api/v1/config/" \
     -H "Content-Type: application/json" \
     -d '{
  "mem0": {
    "llm": {
      "provider": "openai",
      "config": {
        "model": "llama-3.3-70b-versatile",
        "api_key": "<GROQ_API_KEY>",
        "openai_base_url": "https://api.groq.com/openai/v1"
      }
    },
    "embedder": {
      "provider": "openai",
      "config": {
        "model": "text-embedding-3-small",
        "api_key": "<OPENAI_API_KEY>"
      }
    }
  }
}'
```

**Ghi chú:** Khi dùng OpenAI gốc cho Embedder, không cần truyền `openai_base_url`.

### 4. Minimax trực tiếp

```bash
curl -X PUT "http://localhost:8765/api/v1/config/" \
     -H "Content-Type: application/json" \
     -d '{
  "mem0": {
    "llm": {
      "provider": "openai",
      "config": {
        "model": "MiniMax-M2.5",
        "api_key": "<MINIMAX_API_KEY>",
        "openai_base_url": "https://api.minimax.io/v1"
      }
    },
    "embedder": {
      "provider": "openai",
      "config": {
        "model": "embo-01",
        "api_key": "<MINIMAX_API_KEY>",
        "openai_base_url": "https://api.minimax.io/v1"
      }
    }
  }
}'
```

---

## Xác nhận cấu hình

Sau khi gửi lệnh cấu hình, kiểm tra lại bằng:

```bash
curl -s "http://localhost:8765/api/v1/config/" | python3 -m json.tool
```

Kết quả trả về sẽ hiển thị đầy đủ cấu hình hiện tại bao gồm `openai_base_url`.

## Reset về mặc định (OpenAI)

Nếu muốn quay lại sử dụng OpenAI gốc:

```bash
curl -X POST "http://localhost:8765/api/v1/config/reset"
```

**Lưu ý:** Khi reset, bạn cần đảm bảo biến môi trường `OPENAI_API_KEY` đã được thiết lập trong file `api/.env`.

## Cấu hình qua giao diện Settings

Bạn cũng có thể cấu hình thông qua giao diện Settings trên UI tại `http://localhost:3000`. Tuy nhiên, giao diện UI có thể chưa hiển thị trường `openai_base_url` nếu chưa được cập nhật. Trong trường hợp đó, hãy sử dụng lệnh `curl` như hướng dẫn phía trên.

---

## Danh sách nhà cung cấp phổ biến

| Nhà cung cấp | Base URL | Ghi chú |
|---|---|---|
| OpenAI | `https://api.openai.com/v1` | Mặc định, không cần `openai_base_url` |
| Chutes | `https://llm.chutes.ai/v1` | Hỗ trợ nhiều model open-source |
| OpenRouter | `https://openrouter.ai/api/v1` | Aggregator cho nhiều model |
| Together AI | `https://api.together.xyz/v1` | Cả LLM và Embedding |
| Groq | `https://api.groq.com/openai/v1` | Tốc độ inference nhanh |
| Minimax | `https://api.minimax.io/v1` | MiniMax-M2.5 |
| DeepSeek | `https://api.deepseek.com/v1` | DeepSeek-V3, R1 |
| Fireworks | `https://api.fireworks.ai/inference/v1` | Tốc độ cao |
