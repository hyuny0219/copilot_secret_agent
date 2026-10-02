# 공식 Copilot 로고 넣는 곳

이 폴더에 사내 브랜드 포털이나 Microsoft 브랜드 자료에서 받은 **공식 Copilot 로고 SVG**를
`copilot-logo.svg` 라는 이름으로 넣으면, 시안의 모든 Copilot 엠블럼(호출 버튼, 호출 화면, Copilot 창, "Copilot으로 작성됨" 표시)이
자동으로 공식 로고로 바뀝니다. 파일이 없으면 임시 그라데이션 엠블럼이 표시됩니다.

- 파일명: `copilot-logo.svg` (PNG를 쓰려면 `station.html`의 `LOGO_PATH` 값을 바꾸세요)
- 정사각형 비율 권장, 여백 없는 아이콘형 로고

# 음성·BGM 파일 넣는 곳 (선택)

아래 이름으로 파일을 넣으면 내장 음성(브라우저 TTS)과 자체 합성 BGM 대신 이 파일이 재생됩니다. 없으면 기본 음성·BGM이 나옵니다.

| 파일명 | 용도 |
|---|---|
| `voice-intro.mp3` | START 후 본부 지령 음성 (중저음 남성 성우 녹음 권장) |
| `bgm-m1.mp3` | 미션 01 투자운영파트 BGM (반복 재생) |
| `bgm-m2.mp3` | 미션 02 글로벌U/W기획 BGM (반복 재생) |
| `bgm-m3.mp3` | 미션 03 기획파트 BGM (반복 재생) |
