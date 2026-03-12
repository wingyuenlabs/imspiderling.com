---
Title: GLM-5, NVIDIA 없이 오픈소스 1위 달성 — Phi-4, Qwen3.5까지, 오픈소스 LLM 경쟁이 뜨겁다
Description: 
Author: jidong
Date: 2026-03-12T21:29:55.000Z
Robots: noindex,nofollow
Template: index
---
<h2>
  
  
  무슨 일이 있었나
</h2>

<p>오픈소스 LLM 진영에서 한 주 만에 세 가지 주요 모델이 공개됐습니다.</p>

<h3>
  
  
  GLM-5 — MIT 라이센스, NVIDIA 없이 훈련
</h3>

<p>중국 Zhipu AI가 <strong>GLM-5</strong>를 <strong>MIT 라이센스</strong>로 공개했습니다.</p>

<ul>
<li>
<strong>744B 파라미터</strong> (40B active, MoE 구조)</li>
<li>
<strong>SWE-bench Verified 77.8%</strong> — 오픈소스 모델 중 1위</li>
<li>
<strong>Huawei Ascend 칩</strong>으로만 훈련 (NVIDIA GPU 미사용)</li>
<li>
<strong>API 가격</strong>: $1.00/1M input, $3.20/1M output</li>
</ul>

<p><small><a href="https://www.buildfastwithai.com/blogs/glm-5-released-open-source-model-2026" rel="noopener noreferrer">BuildFast — GLM-5 Released: 744B Open-Source Model</a></small></p>

<h3>
  
  
  Phi-4-Reasoning-Vision-15B — 작지만 강한 멀티모달
</h3>

<p>Microsoft가 <strong>Phi-4-reasoning-vision-15B</strong>를 오픈소스로 공개했습니다.</p>

<ul>
<li>
<strong>15B 파라미터</strong> — consumer GPU에서 로컬 실행 가능 (RTX 4090, M4 Max MacBook 등)</li>
<li>
<strong>멀티모달 reasoning</strong> — 이미지를 보고 논리적으로 추론</li>
<li>
<strong>Adaptive chain-of-thought</strong> — 간단한 질문에는 바로 답하고, 복잡한 문제에만 step-by-step 추론을 자동 활성화</li>
<li>
<strong>240 NVIDIA B200 GPU, 4일 훈련</strong> — 극적인 훈련 효율성</li>
</ul>

<p><small><a href="https://siliconangle.com/2026/03/04/microsoft-open-sources-multimodal-reasoning-model-15b-parameters/" rel="noopener noreferrer">SiliconANGLE — Microsoft open-sources multimodal reasoning model</a></small></p>

<h3>
  
  
  Qwen3.5-397B-A17B — Alibaba의 최신 플래그십
</h3>

<p>Alibaba가 <strong>Qwen3.5-397B-A17B</strong>를 공개했습니다.</p>

<ul>
<li>
<strong>397B 파라미터</strong> (17B active, MoE 구조)</li>
<li>멀티모달 추론 + 초장문 컨텍스트 지원</li>
<li>이전 세대 대비 디코딩 처리량 <strong>8.6x~19x</strong> 향상</li>
</ul>

<h2>
  
  
  관련 소식
</h2>

<p><strong>GitHub 트렌딩에서 AI/LLM 프로젝트가 상위 독식</strong></p>

<p>오픈소스 LLM 생태계를 보여주는 지표로, 이번 주 GitHub 트렌딩 상위에 AI 관련 프로젝트가 대거 포진해 있습니다.</p>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>프로젝트</th>
<th>Stars</th>
<th>설명</th>
</tr>
</thead>
<tbody>
<tr>
<td><a href="https://github.com/ollama/ollama" rel="noopener noreferrer">ollama/ollama</a></td>
<td>164K+</td>
<td>LLM 로컬 실행 도구. GLM-5, DeepSeek, Qwen 등 지원</td>
</tr>
<tr>
<td><a href="https://github.com/langgenius/dify" rel="noopener noreferrer">langgenius/dify</a></td>
<td>131K+</td>
<td>Agentic workflow 빌더</td>
</tr>
<tr>
<td><a href="https://github.com/firecrawl/firecrawl" rel="noopener noreferrer">firecrawl/firecrawl</a></td>
<td>89K+</td>
<td>웹 → LLM-ready markdown 변환</td>
</tr>
<tr>
<td><a href="https://github.com/vllm-project/vllm" rel="noopener noreferrer">vllm-project/vllm</a></td>
<td>72K+</td>
<td>LLM 서빙 엔진 (프로덕션 표준)</td>
</tr>
<tr>
<td><a href="https://github.com/OpenHands/OpenHands" rel="noopener noreferrer">OpenHands/OpenHands</a></td>
<td>68K+</td>
<td>AI 기반 소프트웨어 개발 플랫폼</td>
</tr>
</tbody>
</table></div>

<p><strong>미국 칩 규제의 역설</strong></p>

<p>GLM-5가 NVIDIA 없이 Huawei Ascend 칩만으로 frontier급 성능을 달성한 것은, 미국의 대중국 AI 칩 수출 규제가 의도하지 않은 결과를 낳고 있다는 증거입니다. 단기적으로는 개발 속도를 늦췄을 수 있지만, 장기적으로는 대안 하드웨어 기반의 독자 생태계를 가속화하는 결과로 이어지고 있습니다.</p>

<h2>
  
  
  수치로 보기
</h2>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>모델</th>
<th>파라미터</th>
<th>Active</th>
<th>SWE-bench</th>
<th>라이센스</th>
</tr>
</thead>
<tbody>
<tr>
<td>GLM-5</td>
<td>744B</td>
<td>40B</td>
<td>77.8%</td>
<td>MIT</td>
</tr>
<tr>
<td>Phi-4-Vision</td>
<td>15B</td>
<td>15B</td>
<td>—</td>
<td>오픈소스</td>
</tr>
<tr>
<td>Qwen3.5</td>
<td>397B</td>
<td>17B</td>
<td>—</td>
<td>오픈소스</td>
</tr>
</tbody>
</table></div>

<div class="table-wrapper-paragraph"><table>
<thead>
<tr>
<th>항목</th>
<th>수치</th>
</tr>
</thead>
<tbody>
<tr>
<td>GLM-5 API (input)</td>
<td>$1.00 / 1M tokens</td>
</tr>
<tr>
<td>GLM-5 API (output)</td>
<td>$3.20 / 1M tokens</td>
</tr>
<tr>
<td>Phi-4 훈련 기간</td>
<td>4일 (240x B200 GPU)</td>
</tr>
<tr>
<td>Qwen3.5 처리량 향상</td>
<td>8.6x ~ 19x</td>
</tr>
</tbody>
</table></div>

<h2>
  
  
  정리
</h2>

<p>오픈소스 LLM 경쟁이 새로운 국면에 들어서고 있습니다.</p>

<p><strong>GLM-5</strong>는 두 가지 면에서 의미가 있습니다. 첫째, NVIDIA 독점 체제에 대한 실질적인 대안이 존재한다는 것을 보여줬습니다. 둘째, MIT 라이센스로 상업적 사용에 아무 제한이 없습니다. Meta의 Llama가 열었던 오픈소스 시장에 강력한 경쟁자가 추가된 셈입니다.</p>

<p><strong>Phi-4</strong>의 adaptive chain-of-thought도 주목할 만합니다. 모든 질문에 길게 사고하는 기존 reasoning 모델의 비효율을 해결하는 접근입니다. 이 기법이 범용 모델에 적용되면, 추론 비용과 지연 시간을 크게 줄일 수 있습니다. 그리고 15B 규모이기 때문에 개인 개발자나 소규모 팀이 로컬에서 바로 활용할 수 있다는 점이 실용적입니다.</p>

<p>전체적으로, "크고 비싼 모델이 최고"라는 공식이 점점 무너지고 있습니다. 오픈소스 진영은 MoE 아키텍처를 통해 큰 모델을 효율적으로 돌리거나, 작지만 강한 모델로 실질적인 접근성을 높이는 두 가지 전략을 동시에 추구하고 있습니다.</p>

<p><small>출처: <a href="https://www.buildfastwithai.com/blogs/glm-5-released-open-source-model-2026" rel="noopener noreferrer">BuildFast</a> · <a href="https://siliconangle.com/2026/03/04/microsoft-open-sources-multimodal-reasoning-model-15b-parameters/" rel="noopener noreferrer">SiliconANGLE</a></small></p>




<blockquote>
<p>📌 원문: <a href="https://jidonglab.com/ai-news/2026-03-07-glm5-phi4-opensource" rel="noopener noreferrer">Jidong Lab</a><br>
AI 소식과 개발 로그 → <a href="https://jidonglab.com" rel="noopener noreferrer">jidonglab.com</a></p>
</blockquote>

