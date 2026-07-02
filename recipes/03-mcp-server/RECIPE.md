# Recipe 03 — MCP 서버로 노출

**조합**: Recipe 02의 DB(표현·연결) + MCP 툴콜(접근)
**잘 답하는 질문**: Recipe 02와 동일하되, **어떤 세션·어떤 클라이언트에서든** 검색 가능
**선행 조건**: `recipes/02-sqlite-db` 완료 (wiki.db 존재)
**필요 도구**: Node.js + @modelcontextprotocol/sdk 또는 Python + FastMCP

---

## 왜 MCP인가

Recipe 01·02는 에이전트가 위키 폴더/DB가 있는 로컬 세션에서만 작동한다.
MCP로 감싸면 Claude Desktop, Claude Code, 다른 프로젝트 어디서든
`/wiki` 검색처럼 호출된다. **위키가 나를 따라다니게 되는 것.**

## 툴 설계 — 3개면 충분하다

과설계 금지. 검색 계약에서 나온 질문에 답하는 최소 툴만 만든다.

| 툴 | 입력 | 동작 |
|---|---|---|
| `wiki_search` | query (자연어/키워드) | FTS5 + concepts 조인 검색 → 요약·축·source_path 반환 |
| `wiki_browse` | axis, value (예: org="삼성전자") | 축 기준 필터 목록 |
| `wiki_read_source` | doc_id | source_path의 원문 텍스트 추출해 반환 |

`wiki_read_source`가 연결층의 완성이다 — 요약 검색 후 원문까지
툴콜 하나로 도달한다.

## 에이전트 실행 절차 (Python FastMCP 기준)

### 1. 서버 작성

```python
# server.py
from fastmcp import FastMCP
import sqlite3

mcp = FastMCP("llm-wiki")
DB = "wiki.db"

@mcp.tool()
def wiki_search(query: str, limit: int = 10) -> list[dict]:
    """위키에서 요약을 검색한다. 결과의 source_path로 원문 접근 가능."""
    con = sqlite3.connect(DB); con.row_factory = sqlite3.Row
    rows = con.execute("""
      SELECT d.id, d.title, d.org, d.topic, d.summary, d.source_path
      FROM doc_fts f JOIN documents d ON d.id = f.rowid
      WHERE doc_fts MATCH ? LIMIT ?""", (query, limit)).fetchall()
    return [dict(r) for r in rows]

@mcp.tool()
def wiki_read_source(doc_id: int) -> str:
    """문서 원문 전체를 읽는다."""
    con = sqlite3.connect(DB)
    path = con.execute("SELECT source_path FROM documents WHERE id=?", (doc_id,)).fetchone()[0]
    # pdf/pptx/docx → 텍스트 추출 후 반환 (extract_text 유틸 사용)
    return extract_text(path)

if __name__ == "__main__":
    mcp.run()
```

`wiki_browse`도 같은 패턴으로 추가한다.

### 2. 클라이언트 등록

```bash
# Claude Code
claude mcp add llm-wiki -- python /path/to/server.py

# Claude Desktop: claude_desktop_config.json의 mcpServers에 동일 커맨드 등록
```

원격 배포가 필요하면 Next.js + mcp-handler로 Vercel에 올리는 변형도 가능하다
(단, 이 경우 source_path가 로컬 경로면 wiki_read_source는 작동하지 않으므로
원문도 클라우드 스토리지/URL로 올려야 한다 — 연결층이 URL로 바뀌는 것).

### 3. 검증

새 세션을 열고 (위키 폴더 밖에서) `questions` 3개를 질문한다.
에이전트가 `wiki_search` → `wiki_read_source` 순으로 호출해
원문 기반 답을 내면 완료.
