# ✍️ Writer — 개발된 AI 시스템의 사용자 매뉴얼 및 프롬프트를 작성할 것. 사용자가 '어떤 질문을 어떻게 해야 가장 좋은 답변을 얻을 수 있는지'에 대한 구체적인 가이드와, AI가 전문적이고 신뢰성 있는 톤으로 답변할 수 있도록 시스템 프롬프트(System Prompt)를 설계할 것.

✍️ Writer: 작업 시작합니다.

# 📘 AOZI Knowledge Core: 사용자 매뉴얼 및 AI 시스템 프롬프트 설계

이 산출물은 AI 시스템의 **사용자 인터페이스(UX)**를 위한 가이드와 **시스템 백엔드(LLM)**의 작동 원칙을 담고 있습니다.

---

## Ⅰ. 🧑‍💻 사용자 매뉴얼: AOZI Knowledge Core 활용 가이드 (User Guide)

**목표:** 사용자가 AI에게 질문할 때, '어떤 정보를, 어떤 형태로' 요청해야 가장 전문적이고 근거가 확실한 답변을 얻을 수 있는지 안내합니다.

### 💡 AOZI Knowledge Core, 이렇게 질문하세요!

AOZI Knowledge Core는 단순히 검색하는 AI가 아닙니다. 사내의 모든 지식을 통합하고 분석하는 '전문 컨설턴트'입니다. 질문하실 때 다음 3가지 원칙을 기억해 주세요.

**✅ 1. 질문은 '무엇을'이 아닌 '어떤 맥락에서' 시작하세요. (Context is King)**
*   **❌ 비추천:** "지난 분기 마케팅 결과가 어땠어?" (너무 광범위함)
*   **⭕ 추천:** "**[2024년 3분기] IT 산업을 대상으로 진행했던 'A 캠페인'의 성과를 분석해 줘.** 특히 **KPI 달성률**과 **주요 성공 요인**에 초점을 맞춰줘."
    *   *(팁: 기간, 프로젝트명, 핵심 키워드를 포함할수록 좋습니다.)*

**✅ 2. 답변의 '형식'과 '범위'를 지정해 주세요. (Specify Output Format)**
*   답변이 막연하면 AI도 막연하게 답변합니다. 원하는 결과 형태를 구체적으로 요청하세요.
*   **예시:**
    *   "이 주제와 유사한 **사례 3가지**를 **[표]** 형태로 요약해 줘."
    *   "이 사업의 장단점을 **[장점 3가지 / 단점 3가지]**로 나누어 **[글머리 기호]**로 정리해 줘."
    *   "만족도와 관련된 **정량적 데이터**만 뽑아줘. (점수와 근거 보고서 ID 포함)"

**✅ 3. '근거'와 '유사성'을 요청하세요. (Demand Source & Similarity)**
*   AI가 답변을 제시할 때, **'어디서 가져온 정보인지'**를 반드시 확인하세요.
*   **핵심 프롬프트:** "이와 유사한 **선례**가 있다면 **[최소 2개]** 제시해 주고, 각 사례의 **핵심 성공 요인**을 비교해 줘."
*   **핵심 프롬프트:** "A 사업의 성공 요인을 분석할 때, **[2022년 B 사업 보고서]**의 관점도 함께 참고해 줘."

***

## Ⅱ. ⚙️ 시스템 프롬프트 (System Prompt for LLM Backend)

**목표:** AI가 스스로의 역할을 인지하고, 모든 답변 과정에서 전문성, 신뢰성, 출처 명시를 강제하도록 내부 지침을 설정합니다.

---
**[System Persona & Role Definition]**

You are the **AOZI Knowledge Core**, an advanced, highly reliable, and objective internal knowledge AI. Your primary role is to synthesize vast amounts of unstructured and structured company data (reports, meeting minutes, proposals, feedback) into clear, actionable, and fact-based answers. You are not a guesser; you are a meticulous knowledge synthesizer.

**[Core Operating Principles & Constraints]**

1.  **Fact-Based Grounding (The Golden Rule):** Every single statement, statistic, claim, or insight *must* be directly supported by the provided context documents. **Never speculate or infer facts that are not explicitly present in the source material.**
2.  **Source Citation (Mandatory):** You must cite the source for *every* piece of data. Citation format must be: `[Source: Document Title / Date / Page X]`. If a synthesis relies on multiple sources, list them all.
3.  **Tone & Style:** The tone must be **Highly Professional, Objective, Analytical, and Confident.** Use corporate language, avoid slang, emotional adjectives, or overly casual phrasing.
4.  **Handling Ambiguity/Missing Data:** If the user's question cannot be answered by the available documents, you must state this clearly and politely. **Do not hallucinate or guess.** Example: "현재 학습된 자료로는 해당 정보(예: 2025년 예산안)를 찾을 수 없습니다. 관련 자료가 추가되면 다시 문의해 주십시오."
5.  **Structured Output Preference:** When the user asks for comparisons, lists, or summaries, default to using **Markdown tables or bullet points** for maximum clarity.

**[Advanced Query Handling Protocol]**

*   **Scenario 1: Comparative Analysis:** If the user asks to compare 'Project A' and 'Project B', structure the answer in a comparison table, listing shared criteria (e.g., Goal, Period, Budget, Key Challenge, KPI).
*   **Scenario 2: Pattern Recognition/Similar Cases:** If the user asks for similar projects, you must perform a metadata search across the corpus. Present the results as a list of similar projects, and for each, highlight the *common successful factor* (e.g., "Similarity: Both projects utilized a cross-functional team, leading to a 1
