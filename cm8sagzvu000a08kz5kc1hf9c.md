---
title: "Claude Desktop으로 MCP를 연결해보자"
datePublished: Fri Mar 28 2025 04:35:10 GMT+0000 (Coordinated Universal Time)
cuid: cm8sagzvu000a08kz5kc1hf9c
slug: claude-desktop-mcp
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1743135727482/4fc26e76-3ffd-4701-8d28-06c3a1b04bb7.webp
tags: claudeai, mcp

---

요새는 누구나 ChatGPT나 Claude 같은 **AI 툴들**에 대해 한 번쯤은 들어봤을 것이다

개발자들은 거의 필수적으로 LLM을 활용해 생산성을 극대화하고 있고,

일반 사용자들도 다양한 작업을 모델에 **대리 위임**하고 있다

복사-붙여넣기 반복에 지쳐

*“아… 이것도 AI가 해줬으면…”*

하는 생각, 한 번쯤 해봤을 거인데

📣 **하지만 이제는 더 이상 상상이 아님!**

👉 바로 MCP를 활용하면 가능함!

## **🧩 1. MCP란?**

**MCP**는 **Model Context Protocol**의 약자이다

📚 위키독스에서 정의된 내용을 먼저 살펴보자:

<div data-node-type="callout">
<div data-node-type="callout-emoji">💡</div>
<div data-node-type="callout-text">Model Context Protocol(MCP)은 LLM(Large Language Model) 애플리케이션과 외부 데이터 소스 및 도구들 간의 원활한 통합을 가능하게 하는 개방형 프로토콜입니다. AI 기반 IDE를 구축하든, 채팅 인터페이스를 개선하든, 혹은 커스텀 AI 워크플로우를 만들든 관계없이, MCP는 LLM이 필요로 하는 컨텍스트와 연결하기 위한 표준화된 방법을 제공합니다.</div>
</div>

🌀 설명이 조금 복잡하다…

📌 쉽게 말해, MCP는 **맥북에 연결하는 USB 허브처럼** 작동을 한다!

![How MCP works: The architecture](https://norahsakal.com/assets/images/mcp_overview-641a298352ff835488af36be3d8eee52.png align="left")

MCP는 맥북에 사용하는 포트 허브라고 생각하면 편하다

기존 MCP host인 LLM 모델들과의 통신을 직접적으로 MCP가 담당하고 여러 서비스들을 이 MCP로 연결만 해주면 해당 기능들을 간단하게 LLM을 통해 대리 수행이 가능한 것이다

예를 들면 아래와 같은 명령어를 MCP client를 통해 수행이 가능하다

**📎 MCP를 통해 가능한 예시들**

* 📆 구글 캘린더에 내일 11시 점심 약속 등록해줘
    
* 📝 노션 표 데이터 수정해줘
    
* ✉️ 지메일 최근 메일 10건 요약해줘
    
* 🧾 [README.md](http://README.md)를 git repo에 커밋해줘
    
* 🔍 Brave Search로 어제 산불 뉴스 검색해줘
    

👉 이런 작업들을 이제는 **명령어 한 줄로 자동화** 가능!

기존에 수동으로 해야했던 작업들을 이제는 MCP라는 에이전트를 통해서 대리 수행하는 것으로 이해하면 쉽다!

확장성이 무궁무진하고 점차 많은 서비스들이 MCP 연결부 개발에 열을 올리고 있다

우리도 Claude Desktop을 이용해서 간편하게 세팅을 해보자

> Pro 버전만 가능한가요??

무료 버전에서도 기본적인 세팅 및 설정은 다 가능하다!

하지만 MCP가 제공하는 기능을 100% 다 끌어낼려면 아무래도 토큰이나 대화 길이 제한이 없는 Pro 버전을 구독하는것을 추천한다

## **⚙️ 2. MCP 설치 및 실행**

1. ### **📥 Claude Desktop 설치**
    

우선 Claude Desktop을 설치를 해보자

👉 [https://claude.ai/download](https://claude.ai/download)

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743125492590/bf7cd559-519e-4ecc-9e99-decc02a5c69a.png align="center")

2. ### [**config 파일 구성**](https://claude.ai/download)
    

[clau](https://claude.ai/download)de\_desktop\_config.json 에 mcpServer 구성을 해야한다

처음부터 파일이 있는것이 아니라 초기 생성이 필요하다

우선 설정부터 가보자 (앱 내의 계정 설정이 아니라 Claude 자체의 Settings으로 가야한다)

![](https://mintlify.s3.us-west-1.amazonaws.com/mcp/images/quickstart-menu.png align="left")

Edit Config를 통해서 claude\_desktop\_config.json을 생성한다

![](https://mintlify.s3.us-west-1.amazonaws.com/mcp/images/quickstart-developer.png align="left")

이를 통해 아래 경로에 정상적으로 desktop config이 생성된것을 확인할 수 있다

* macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`
    
* Windows: `%APPDATA%\Claude\claude_desktop_config.json`
    

그 다음에 아래와 같이 파일시스템 mcp 설정을 추가해보자 (username은 본인 환경에 맞게 수정 필요)

```json
{
  "mcpServers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/Users/username/Desktop",
        "/Users/username/Downloads"
      ]
    }
  }
}
```

3. ### **🔁 Claude 재실행**
    

그리고 Claude Desktop app을 끄고서 재실행시키면 정상적으로 mcp들이 로드되면 아래와 같이 변경된다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743124090617/fb926946-88cf-4fe9-8bcb-aa24555d4f92.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743124110491/cb858f7d-87d5-41b3-830d-30e960883858.png align="center")

UI 상에 **🔨 망치 아이콘**으로 표시되는게 현재 등록된 mcp 서비스 개수이다

클릭해서 보면 팝업으로 리스트업된 도구들이 보인다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743124130412/444457e1-6a85-4ce7-810c-07aa9b9b1737.png align="center")

4. ### **MCP 실행**
    

이제 클로드를 통해서 mcp를 실행시켜보자

간단하게 Downloads 경로에 임시 text 파일 생성을 요청해보자!

✅ Downloads 경로에 test.txt 파일을 만들어줘!

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743124163696/bbd7e3ba-7b8d-4857-b708-3523b89090bd.png align="center")

실제로 Finder를 통해서도 MCP를 통해 파일이 생성된 것을 확인 가능하다

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1743125863386/69c3d385-2bc7-4803-9d37-297abd49478b.png align="center")

## **🚀** 3\. MCP의 확장성

그러면 파일시스템 외에 어떠한 mcp 서비스들을 추가할 수 있는지 살펴보자

| [**AWS KB Retrieval**](https://github.com/modelcontextprotocol/servers/blob/main/src/aws-kb-retrieval-server) | Bedrock Agent Runtime을 사용한 AWS 지식 베이스 검색 |
| --- | --- |
| [**Brave Search**](https://github.com/modelcontextprotocol/servers/blob/main/src/brave-search) | Brave의 검색 API를 활용한 웹 및 로컬 검색 |
| [**EverArt**](https://github.com/modelcontextprotocol/servers/blob/main/src/everart) | 다양한 모델을 사용한 AI 이미지 생성 |
| [**Everything**](https://github.com/modelcontextprotocol/servers/blob/main/src/everything) | 프롬프트, 리소스, 도구를 포함한 참조 / 테스트 서버 |
| [**Fetch**](https://github.com/modelcontextprotocol/servers/blob/main/src/fetch) | 효율적인 LLM 사용을 위한 웹 콘텐츠 가져오기 및 변환 |
| [**Filesystem**](https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem) | 구성 가능한 접근 제어가 포함된 안전한 파일 작업 |
| [**Git**](https://github.com/modelcontextprotocol/servers/blob/main/src/git) | Git 리포지토리 읽기, 검색, 조작 도구 |
| [**GitHub**](https://github.com/modelcontextprotocol/servers/blob/main/src/github) | 레포지토리 관리, 파일 작업, GitHub API 통합 |
| [**GitLab**](https://github.com/modelcontextprotocol/servers/blob/main/src/gitlab) | 프로젝트 관리를 위한 GitLab API |
| [**Google Drive**](https://github.com/modelcontextprotocol/servers/blob/main/src/gdrive) | Google Drive 파일 접근 및 검색 기능 |
| [**Google Maps**](https://github.com/modelcontextprotocol/servers/blob/main/src/google-maps) | 위치 서비스, 길찾기, 장소 정보 제공 |
| [**Memory**](https://github.com/modelcontextprotocol/servers/blob/main/src/memory) | 지식 그래프 기반의 영속적인 메모리 시스템 |
| [**PostgreSQL**](https://github.com/modelcontextprotocol/servers/blob/main/src/postgres) | 스키마 조회가 가능한 읽기 전용 데이터베이스 접근 |
| [**Puppeteer**](https://github.com/modelcontextprotocol/servers/blob/main/src/puppeteer) | 브라우저 자동화 및 웹 스크래핑 |
| [**Redis**](https://github.com/modelcontextprotocol/servers/blob/main/src/redis) | Redis 키-값 저장소와의 상호작용 |
| [**Sentry**](https://github.com/modelcontextprotocol/servers/blob/main/src/sentry) | [Sentry.io](http://Sentry.io)에서 이슈 검색 및 분석 |
| [**Sequential Thinking**](https://github.com/modelcontextprotocol/servers/blob/main/src/sequentialthinking) | 사고 과정을 통한 동적이고 반영적인 문제 해결 |
| [**Slack**](https://github.com/modelcontextprotocol/servers/blob/main/src/slack) | 채널 관리 및 메시지 전송 기능 |
| [**Sqlite**](https://github.com/modelcontextprotocol/servers/blob/main/src/sqlite) | 데이터베이스 상호작용 및 비즈니스 인텔리전스 기능 |
| [**Time**](https://github.com/modelcontextprotocol/servers/blob/main/src/time) | 시간 및 시간대 변환 기능 |

이 외에도 [Cloudflare](https://github.com/cloudflare/mcp-server-cloudflare), [Grafana](https://github.com/grafana/mcp-grafana), [Jetbrains](https://github.com/JetBrains/mcp-jetbrains), [Milvus](https://github.com/zilliztech/mcp-server-milvus) 등등 다양한 Third-party와 커뮤니티 서버들이 추가되고 있다

앞으로는 어떠한 서비스들까지 MCP를 활용하여 서비스 개발이 가능할지 기대가 되는 상황이다

미리 세팅을 해서 필요한 MCP 서버들을 연결을 해놓자!

## **✅ 정리**

• 🧠 MCP는 LLM이 외부 세계와 통신하게 해주는 **표준 통로**

• 🧰 복잡한 작업을 **명령어 한 줄로 자동화**

• 🔧 설치도 간단하며 **무료 버전으로도 충분히 테스트 가능**

• 🧩 앞으로 **더 많은 도구들이 MCP 기반으로 통합될 예정**

### 참고자료

1. “Introducing the Model Context Protocol”, Anthropic, 2024.11.26, [https://www.anthropic.com/news/model-context-protocol](https://www.anthropic.com/news/model-context-protocol)
    
2. “MCP Introduction”, MCP, [https://www.claudemcp.com/docs](https://www.claudemcp.com/docs)
    
3. “Model Context Protocol” GitHub, [https://github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
    
4. “punkpeye / awesome-mcp-servers”, GitHub, [https://github.com/punkpeye/awesome-mcp-servers?tab=readme-ov-file](https://github.com/punkpeye/awesome-mcp-servers?tab=readme-ov-file)