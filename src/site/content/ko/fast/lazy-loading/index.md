---
layout: post-old
title: 지연 로드를 사용하여 로드 속도 향상
authors:
  - jlwagner
  - rachelandrew
date: 2019-08-16
updated: 2020-06-09
description: 이 게시물은 지연 로드와 사이트에서 요소를 지연 로드하고 싶어 할 수 있는 이유를 설명합니다.
tags:
  - performance
  - images
---

웹 사이트의 일반적인 페이로드에서 [이미지](http://beta.httparchive.org/reports/state-of-images?start=earliest&end=latest) 및 [동영상](http://beta.httparchive.org/reports/page-weight#bytesVideo)이 차지하는 부분이 상당할 수 있습니다. 불행히도 프로젝트 이해 관계자는 기존 응용 프로그램에서 미디어 리소스를 줄이는 것을 꺼려할 수 있습니다. 이러한 교착 상태에서는, 특히 관련된 모든 당사자가 사이트 성능을 개선하기를 원하지만 거기에 도달하는 방법에 동의할 수 없는 경우 좌절감을 느낄 수 있습니다. 다행히 지연 로드는 초기 페이지 페이로드 *및* 로드 시간은 낮추지만 콘텐츠는 아끼지 않는 솔루션입니다.

## 지연 로드란? {: #what }

지연 로드는 페이지 로드 시 중요하지 않은 리소스의 로드를 연기하는 기술입니다. 대신 이러한 중요하지 않은 리소스는 필요한 순간에 로드됩니다. 이미지와 관련하여 "비중요"는 종종 "오프스크린"과 거의 같은 의미를 갖습니다. 여러분이 Lighthouse를 사용하며 몇 가지 개선 가능성을 조사했다면 [오프스크린 이미지 지연 감사](/offscreen-images/) 형식으로 이 영역에 있는 몇 가지 지침을 보았을 것입니다.

<figure class="w-figure">{% Img src="image/admin/63NnMISWUUWD3mvAliwe.png", alt="Lighthouse의 오프스크린 이미지 지연 감사의 스크린샷.", width="800", height="102", class="w-screenshot" %} <figcaption class="w-figcaption">Lighthouse의 성능 감사 중 하나는 지연 로드 후보인 오프스크린 이미지를 식별하는 것입니다.</figcaption></figure>

아마도 지연 로드가 작동하는 것을 이미 보았을 것이며 다음과 같이 진행됩니다.

- 여러분은 페이지에 도착하면 콘텐츠를 읽으면서 스크롤을 시작합니다.
- 어느 시점에 자리 표시자 이미지를 뷰포트로 스크롤합니다.
- 자리 표시자 이미지가 갑자기 최종 이미지로 교체됩니다.

이미지 지연 로드의 예는 인기 퍼블리싱 플랫폼인 [Medium](https://medium.com/)에서 확인할 수 있습니다. 이 플랫폼은 페이지 로드 시 가벼운 자리 표시자 이미지를 로드하고 뷰포트로 스크롤할 때 지연 로드 이미지로 대체합니다.트로 스크롤할 때 지연 로드 이미지로 대체합니다.

<figure class="w-figure">{% Img src="image/admin/p5ahQ67QtZ20bgto7Kpy.jpg", alt="탐색 중인 웹 사이트 Medium의 스크린샷으로 지연 로드가 작동하는 모습을 보여줍니다. 흐릿한 자리 표시자는 왼쪽에 있고 로드된 리소스는 오른쪽에 있습니다.", width="800", height="493" %} <figcaption class="w-figcaption">작동 중인 이미지 지연 로드의 예. 자리 표시자 이미지는 페이지 로드 시 로드되며(왼쪽), 뷰포트로 스크롤하면 최종 이미지가 필요할 때 로드됩니다.</figcaption></figure>

지연 로드에 익숙하지 않다면 이 기술이 얼마나 유용한지, 그리고 그 이점이 무엇인지 궁금할 것입니다. 계속 읽어나가며 알아보십시오.

## 이미지나 동영상을 그냥 *로드*하지 않고 지연 로드하는 이유는 무엇인가요? {: #why }

사용자가 볼 수 없는 내용을 로드할 수 있기 때문입니다. 이것은 다음과 같은 몇 가지 이유로 문제가 됩니다.

- 데이터를 낭비합니다. 무과금 연결의 경우 이는 일어날 수 있는 최악의 상황이 아닙니다(단, 사용자가 실제로 보게 될 다른 리소스를 다운로드하는 데 귀중한 대역폭을 사용할 수는 있음). 그러나 약정 데이터 요금제에서는 사용자가 전혀 볼 수 없는 항목을 로드하는 것은 사실상 비용 낭비가 될 수 있습니다.
- 처리 시간, 배터리 및 기타 시스템 리소스를 낭비합니다. 미디어 리소스를 다운로드한 후 브라우저는 이를 디코딩하고 뷰포트에서 콘텐츠를 렌더링해야 합니다.

이미지 및 동영상을 지연 로드하면 초기 페이지 로드 시간, 초기 페이지 무게 및 시스템 리소스 사용량이 줄어들며 이 모든 것이 성능에 긍정적인 영향을 미칩니다.

## 지연 로드 구현 {: #implementing }

지연 로드를 구현하는 방법에는 여러 가지가 있습니다. 솔루션 선택 시 지원하는 브라우저와 지연 로드를 고려해야 합니다.

최신 브라우저는 이미지 및 iframe에서 `loading` 속성을 사용하여 사용 설정할 수 있는 [브라우저 수준 지연 로드](/browser-level-image-lazy-loading/)를 구현합니다. 구형 브라우저와의 호환성을 제공하거나 내장 지연 로드 없이 요소에서 지연 로드를 수행하기 위해 자체 JavaScript로 솔루션을 구현할 수 있습니다. 이 작업을 수행하는 데 도움이 되는 기존 라이브러리도 많이 있습니다. 이러한 모든 접근 방식에 대한 자세한 내용은 이 사이트의 게시물을 참조하십시오.

- [이미지 지연 로드](/lazy-loading-images/)
- [동영상 지연 로드](/lazy-loading-video/)

또한, 우리는 [지연 로드의 잠재적 문제](/lazy-loading-best-practices) 목록과 구현 시 주의해야 할 사항을 정리했습니다.

## 결론

주의해서 사용하며 이미지와 동영상을 지연 로드하면 사이트의 초기 로드 시간과 페이지 페이로드를 상당히 줄일 수 있습니다. 사용자는 볼 수 없는 미디어 리소스의 불필요한 네트워크 활동 및 처리 비용을 발생시키지 않게 되며, 다만 원하는 경우 해당 리소스를 계속 볼 수 있습니다.

성능 개선 기술의 관점에서 지연 로드는 합리적인 선택이며 논쟁의 여지가 없습니다. 사이트에 인라인 이미지가 많은 경우 불필요한 다운로드를 줄이는 아주 좋은 방법입니다. 여러분의 사이트 사용자와 프로젝트 이해 관계자는 이를 높이 평가할 것입니다!