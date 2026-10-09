# 서버비 없이 애드센스 사이트 운영하기: 플랫폼 비교 (2026년 10월)

> 상황: 애드센스 승인을 받은 워드프레스 사이트를 Amazon Lightsail에서 운영하다 서버비 부담으로 중단. 도메인은 유지하고 싶음.
> 이 문서는 웹 검색 기반이다. 가격과 정책은 바뀔 수 있으니 실행 전에 공식 페이지를 확인할 것.

---

## 0. 먼저 확인할 것 (돈이 새고 있을 수 있음)

1. **Lightsail 인스턴스를 "중지"만 했다면 요금은 계속 나간다.** 중지 상태에서도 과금이 멈추지 않는다는 보고가 있다. 고정 IP, 스냅샷, 디스크 요금도 따로 붙는다. 아래 순서로 정리한다.
   - 인스턴스를 잠깐 켜서 **데이터부터 백업**한다. 방법은 둘 중 하나다.
     - 워드프레스 관리자 → 도구 → 내보내기(XML)
     - All-in-One WP Migration 같은 플러그인으로 전체 백업
   - `wp-content/uploads` 폴더(이미지)를 따로 내려받는다.
   - 백업을 확인한 뒤 인스턴스, 고정 IP, 스냅샷을 **삭제**한다.
2. **도메인 DNS가 어디에 있는지 확인한다.**
   - Route 53 호스팅 영역은 월 사용료가 붙는다.
   - DNS를 **Cloudflare 무료 DNS**로 옮기면 남는 비용은 도메인 연장비(보통 연 1~2만원대)뿐이다.
3. **애드센스 콘솔에서 사이트 상태를 확인한다.** 사이트가 오래 꺼져 있었다면 "주의 필요"로 바뀌었을 수 있다. 플랫폼을 옮겨도 **같은 도메인이면 보통 재승인 없이 이어진다는 경험담이 많다.** 다만 구글 공식 문서로 확인하지는 못했다. 이전 후 상태가 "준비됨"으로 남는지 꼭 확인한다.

---

## 1. 선택지 한눈에 비교

| 선택지 | 월 비용 | 기존 도메인 | 애드센스 | 난이도 | 기존 글 URL 유지 | 한 줄 평 |
|---|---|---|---|---|---|---|
| **A. 블로그스팟(Blogger) + 기존 도메인** | 0원 | ✅ | ✅ 기본 연동, ads.txt 설정 가능 | 쉬움 | ❌ 구조 변경(맞춤 리디렉션으로 일부 보완) | **가장 무난한 무료 대안** |
| **B. 워드프레스 → 정적 사이트 + Cloudflare Pages** | 0원 | ✅ | ✅ HTML에 광고 코드 포함 | 중간~어려움 | ✅ 그대로 | **기존 자산을 가장 잘 지키는 방법** |
| C. 티스토리 + 기존 도메인 | 0원 | ✅ | ⚠️ 가능하지만 제약 있음 | 쉬움 | ❌ | 카카오 광고가 함께 붙고 정책이 자주 바뀜 |
| D. 오라클 클라우드 무료 서버 | 0원 | ✅ | ✅ | 어려움 | ✅ | 무료 사양이 축소됐다는 보도가 있고, 유휴 인스턴스 회수 위험이 있음 |
| E. 저가 공유호스팅 | 월 수천 원 | ✅ | ✅ | 쉬움 | ✅ | "0원"은 아니지만 Lightsail보다 싸고 관리가 편함 |
| F. 유튜브 | 0원 | ❌ (도메인 불필요) | ⚠️ 수익화 문턱이 높음 | 어려움(영상 제작) | — | 블로그의 대체재가 아니라 별개 사업 |

---

## 2. 선택지별 상세

### A. 블로그스팟 + 기존 도메인 — 추천 ①

**장점**
- 구글이 운영해서 서버비가 0원이고, 트래픽이 몰려도 안 터진다.
- 개인 도메인 연결과 HTTPS가 무료다.
- 애드센스와 기본 연동되고, 설정에서 맞춤 ads.txt를 넣을 수 있다.
- 플랫폼 운영사가 광고를 강제로 끼워 넣지 않는다(티스토리와 다른 점).

**단점**
- 테마와 기능이 워드프레스보다 제한적이다. 플러그인이 없다.
- 워드프레스 XML을 바로 가져올 수 없어서 변환 도구를 쓰거나 글을 수동으로 옮겨야 한다.
- 글 주소 형식이 `/2026/10/글제목.html`로 바뀐다. 이미 구글에서 순위가 잡힌 글이 있다면 **맞춤 리디렉션**으로 주요 글만 옛 주소에서 새 주소로 연결해야 한다.

**어울리는 경우**: 기존 글이 많지 않거나, 글보다 앞으로 쓸 글이 더 중요하거나, 기술 관리를 최소화하고 싶을 때.

### B. 정적 워드프레스 + Cloudflare Pages — 추천 ②

**구조**: 워드프레스는 내 PC에서만 실행하는 편집기로 쓴다(Local 같은 도구 사용). 글을 쓰면 플러그인이 정적 HTML로 변환해 Cloudflare Pages(무료)에 올린다. 방문자는 Cloudflare에 있는 HTML만 보게 된다.

- Cloudflare는 공식 가이드에서 이 방식을 안내한다.
- `StaticForge for Cloudflare Pages`처럼 발행할 때마다 자동 업로드하는 플러그인도 있다. 설치 전에 유지보수 상태를 확인한다.

**장점**
- 0원이다.
- **기존 URL, 디자인, 글을 그대로 유지**해서 검색 순위 손실이 가장 적다.
- 매우 빠르다(Core Web Vitals에 유리). 서버가 없어서 해킹 위험도 거의 없다.

**단점**
- 댓글, 문의 폼, 사이트 검색 같은 동적 기능은 작동하지 않는다. 외부 서비스로 대체해야 한다.
- PC 환경 세팅이 필요하다. 글을 쓰는 PC에 워드프레스가 있어야 한다.
- 변환할 때 **광고 코드와 `ads.txt`가 결과물에 포함되는지** 직접 확인해야 한다.

**어울리는 경우**: 기존 글 중 구글 유입이 있던 글이 꽤 있고, 약간의 기술 작업을 감수할 수 있을 때.

### C. 티스토리

- 개인 도메인 연결은 가능하고, 다음(Daum) 검색 유입이 생긴다.
- 다만 2023년 약관 개정으로 **광고의 위치·형태를 티스토리가 정할 수 있게 됐고** 카카오 광고가 함께 붙는다. 수익 일부가 플랫폼으로 간다.
- 2025~2026년에는 애드센스 **앵커 광고와 오퍼월 광고가 금지**됐다.
- 내 사이트의 광고 정책을 남이 바꿀 수 있다는 점이 핵심 리스크다. 이미 승인받은 도메인을 굳이 여기로 옮길 이유는 약하다.

### D. 오라클 클라우드 무료 서버

- 워드프레스를 그대로 돌릴 수 있지만 다음 문제가 있다.
  - 2026년 무료 사양이 4코어/24GB에서 2코어/12GB로 줄었다는 보도가 있다(공식 확인 못 함).
  - 사용률이 낮으면 인스턴스를 회수한다는 보고가 있다.
  - 지역에 따라 무료 인스턴스 자체를 받기 어렵다.
- 서버 관리를 Lightsail 때와 똑같이 해야 하고 언제 사라질지 모른다. 수익 사이트의 본진으로는 비추천한다.

### E. 저가 공유호스팅

- 0원은 아니지만 Lightsail 최소 실사용 사양(1GB 플랜, 월 7달러 안팎에 스냅샷 등 추가 비용)보다 대체로 싸다. 서버 관리도 호스팅 업체가 해 준다.
- 워드프레스 기능(플러그인, 댓글)을 포기하기 싫다면 현실적인 타협안이다.

### F. 유튜브

- **광고 수익 조건**: 구독자 1,000명에 긴 영상 시청 4,000시간(12개월) 또는 쇼츠 조회수 1,000만 회(90일).
- **2027년 2월 1일부터 신규 신청자는 시청 8,000시간 또는 쇼츠 2,000만 회로 두 배가 된다**(2026년 8월 발표, 여러 매체 보도).
- 지금 시작하면 넉 달 안에 4,000시간을 채우기 어렵다. 사실상 새 기준으로 준비해야 한다.
- 서버비는 0원이지만 영상 제작 시간이라는 비용이 크다. 블로그를 대신하는 수단이 아니라 별개 사업으로 봐야 한다.
- 지금 다루려는 주제(세금, 지원금, 계산형 정보)는 **검색 → 글 → 표·계산 예시** 흐름이 강해서 텍스트 블로그가 더 잘 맞는다. 유튜브는 나중에 블로그 글을 쇼츠로 요약해 블로그로 유입시키는 보조 채널로 고려한다.

---

## 3. 결론: 어떻게 고를까

```
기존 글 중 구글 유입이 있던 글이 많다?
 ├─ 예 → 기술 작업 감수 가능?
 │        ├─ 예 → B. 정적 워드프레스 + Cloudflare Pages
 │        └─ 아니오 → E. 저가 공유호스팅 (URL 그대로 이전)
 └─ 아니오(글이 적거나 유입이 거의 없었음)
          → A. 블로그스팟 + 기존 도메인  ← 대부분의 초보에게 정답
```

**공통 원칙**: 어떤 플랫폼으로 가든 **도메인은 반드시 유지**한다. 도메인 나이, 애드센스 승인 이력, 백링크가 모두 도메인에 붙어 있다. 도메인이 내 것이면 나중에 플랫폼을 또 바꿔도 손실이 작다. 티스토리 기본 주소(xxx.tistory.com)나 blogspot.com 주소로 글을 쌓으면 그 자산은 플랫폼 소유가 된다.

---

## 4. 블로그스팟 이전 체크리스트 (A안 선택 시)

1. [ ] Lightsail에서 백업(XML과 이미지)을 받은 뒤 인스턴스, 고정 IP, 스냅샷을 삭제한다
2. [ ] DNS를 Cloudflare(무료)로 옮긴다
3. [ ] 블로그스팟 블로그를 만든다 → 설정 → 맞춤 도메인에 `www.내도메인` 입력 → 안내된 CNAME 2개를 DNS에 추가한다
   - Cloudflare 사용 시 이 레코드는 프록시 끄기(회색 구름)로 둔다
4. [ ] 루트 도메인(내도메인.com)을 www로 리디렉션하도록 설정한다
5. [ ] 설정 → HTTPS 사용 켜기
6. [ ] 설정 → 수익 창출 → 맞춤 ads.txt 사용 → 애드센스 ads.txt 내용을 붙여넣는다
7. [ ] 글을 옮긴다. 유입이 있던 상위 글부터 옮기고, 설정 → 맞춤 리디렉션으로 옛 URL을 새 URL에 연결한다
8. [ ] 구글 서치 콘솔에 사이트맵(`/sitemap.xml`)을 다시 제출한다
9. [ ] 애드센스 → 사이트 메뉴에서 상태가 "준비됨"인지 확인한다. 검토가 필요하다고 나오면 글 10~20개를 채운 뒤 검토를 요청한다

---

## 출처

- [Amazon Lightsail Pricing 2026 (cloudburn)](https://cloudburn.io/blog/amazon-lightsail-pricing)
- [AWS Lightsail Pricing 2026 (onedollarvps)](https://onedollarvps.com/pricing/aws-lightsail-pricing)
- [Cloudflare: Deploy a WordPress site to Pages](https://developers.cloudflare.com/pages/how-to/deploy-a-wordpress-site/)
- [Cloudflare: Hosting static WordPress sites](https://developers.cloudflare.com/workers/tutorials/hosting-static-wordpress-sites/)
- [StaticForge for Cloudflare Pages (WordPress.org)](https://en-gb.wordpress.org/plugins/?p=315673)
- [새해 티스토리 광고정책 변경 (머니투데이, 2023)](https://news.mt.co.kr/mtview.php?no=2023010612540096600)
- [티스토리 애드센스 광고 설정 정책 변경 (회사원K)](https://productowner.blog/article/tistory-adsense-policy-change-2025/)
- [티스토리 정책 변경, 애드센스 광고 설정 (emotionte)](https://emotionte.com/%ED%8B%B0%EC%8A%A4%ED%86%A0%EB%A6%AC-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EC%95%A0%EB%93%9C%EC%84%BC%EC%8A%A4-%EA%B4%91%EA%B3%A0-%EC%84%A4%EC%A0%95/)
- [Oracle Cloud Always Free Tier Cut 2026 (braindetox)](https://braindetox.kr/en/posts/oracle_always_free_tier_reduced_2026.html)
- [Oracle Cloud Free Tier: what's actually free in 2026](https://macmyths.com/oracle-cloud-free-tier-whats-actually-free-in-2026/)
- [YouTube Partner Program Requirements 2026 (AIR)](https://air.io/en/monetization/youtube-partner-program-requirements-2026-the-complete-guide)
- [New YouTube creators face 8,000-hour bar from February 2027 (PPC Land)](https://ppc.land/new-youtube-creators-face-8-000-hour-bar-for-ad-revenue-from-february-2027/)
- [YouTube raises YPP minimums for new applicants (Shacknews)](https://www.shacknews.com/article/150324/youtube-partner-program-minimum-qualified-public-watch-hours-shorts-views)
- [티스토리 승인 도메인을 워드프레스로 이전한 사례 (월부 커뮤니티)](https://weolbu.com/community/3607493)
