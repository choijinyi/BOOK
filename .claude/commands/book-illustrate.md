---
description: 편집부 그림 작업 — Higgsfield MCP로 표지·삽화 생성
---

# /book-illustrate

인자: $ARGUMENTS (형식: `<책이름> [cover | figure <설명>]`)

편집 단계의 이미지 생성. Higgsfield MCP 도구가 로드되지 않았으면 먼저 ToolSearch로 한 번에 로드한다:
`select:mcp__claude_ai_Higgsfield__generate_image,mcp__claude_ai_Higgsfield__upscale_image,mcp__claude_ai_Higgsfield__outpaint_image,mcp__claude_ai_Higgsfield__remove_background,mcp__claude_ai_Higgsfield__job_status`

## 원칙

- **도식·차트·다이어그램은 여기서 만들지 않는다** — 집필 단계의 Mermaid `<!-- fig: -->` 마커 사용 (수정 가능·결정적).
- Higgsfield 담당: 표지 원화, 장 간지 일러스트, 분위기 컷, 사진풍 삽화.
- 생성 이미지는 반드시 로컬에 저장 후 사용: 표지 → `BookTemplate/images/`, 본문 → `BookTemplate/images/figures/`.

## 표지 (`cover`)

0. **표지 샘플에서 시작한다.** `BookTemplate/images/cover-samples/README.md`를 읽고, 책 주제·톤에 맞는 샘플 2~3개를 골라 이유와 함께 보여준다(이미지 파일도 함께 제시). 사용자가 하나를 고르면 경로가 정해진다:
   - **09 녹색·금색(`canva-green-gold/09_*`)** 또는 "짙은 녹색과 금색" 요청 → 이미지 생성 없이 `books.py`에 아래처럼 설정하고 바로 `/book-build`로 확인한다 (게이트 — 사용자 승인 후 수정):
     ```python
     "cover": {
         "mode": "green-gold",
         "kicker": "(선택) 영문 소제목",
         "badge": "(선택) 원 위 숫자·기호",
         "blurb": "(선택) 하단 띠 소개문<br>두 번째 줄",
         "laurels": ["(선택) 월계관<br>문구", "..."],   # 최대 3개
         "publisher": "(선택) 출판사명",
     }
     ```
     제목 강조는 title에 `<span class="accent">…</span>`. 원 안에 들어가므로 제목은 줄당 5~6자, 3줄 이내로 `<br>`을 나눈다.
   - **그 밖의 샘플** → 아래 1~4단계(Higgsfield)로 가되, 고른 샘플의 구도·팔레트·여백을 프롬프트에 구체적으로 서술하고, 가능하면 샘플 파일을 `media_upload`로 올려 참조 이미지로 넘긴다. 샘플 속 글자·저자·출판사(`Jakkawa`)는 따라 그리지 않게 "no text"를 명시한다.
   - **Canva 마감을 원하면** → 같은 Canva 템플릿에서 제목·저자를 바꿔 판형 비율로 PNG export, `"mode": "image", "overlay_text": False`.
1. (Higgsfield 경로) `books.py`의 title/subtitle/author, 책 주제, 고른 샘플을 바탕으로 표지 컨셉 2~3안을 프롬프트로 제시하고 사용자가 고른다.
2. `generate_image`로 시안 생성. 판형 비율에 맞춘다 (신국판 152:225 ≈ 2:3, B5 188:257). 비율이 안 맞으면 `outpaint_image`로 확장.
3. 선택된 시안을 `upscale_image`(2K 이상, 인쇄 대비)로 올린 뒤 `BookTemplate/images/<책>-cover.png`로 저장.
4. `books.py`의 해당 책에 cover 설정 반영 (게이트 — 사용자 승인 후 수정):
   ```python
   "cover": {"mode": "image", "image": "<책>-cover.png", "overlay_text": True}
   ```
   - 제목·저자를 이미지에 직접 그려 넣으려면 Canva MCP로 원화 위에 타이포를 얹어 export하고 `overlay_text: False`로 설정.

## 본문 삽화 (`figure <설명>`)

1. 어느 장의 어느 위치에 넣을지 확인.
2. `generate_image` → 필요 시 `remove_background`/`upscale_image` → `BookTemplate/images/figures/<책>-<번호>-<슬러그>.png` 저장.
3. `content/<책>/NN_*.md`의 해당 위치에 삽입:
   ```markdown
   ![캡션](../images/figures/<파일명>.png)
   ```
   단, 이 삽입은 이관 시 덮어써질 수 있으므로 **오래 유지할 삽화는 manuscripts/ 원고의 figures 참조로 넣고 재이관**하는 것을 권장. 일회성 편집 컷만 content/에 직접 넣는다.
4. 같은 책의 삽화는 화풍·팔레트를 통일한다 (첫 삽화의 스타일 서술을 프롬프트에 재사용).
