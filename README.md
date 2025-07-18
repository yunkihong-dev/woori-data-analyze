# 📦 ELK Stack 기반 카드 소비 분석 프로젝트
> ElasticSearch, Logstash, Kibana, Filebeat 실습을 중심으로 한 데이터 분석 및 디지털 금융 컨설팅 프로젝트

## 👥팀원
<table>
  <tr>
    <td align="center">
      <a href="https://github.com/menzzi">
        <img src="https://github.com/menzzi.png" width="100px;" alt="서민지"/><br />
        <sub><b>서민지</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/0-zoo">
        <img src="https://github.com/0-zoo.png" width="100px;" alt="이영주"/><br />
        <sub><b>이영주</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/yunkihong-dev">
        <img src="https://github.com/yunkihong-dev.png" width="100px;" alt="홍윤기"/><br />
        <sub><b>홍윤기</b></sub>
      </a>
    </td>
    <td align="center">
      <a href="https://github.com/jihwan77">
        <img src="https://github.com/jihwan77.png" width="100px;" alt="황지환"/><br />
        <sub><b>황지환</b></sub>
      </a>
    </td>
  </tr>
</table>

## 🎯 프로젝트 목적
Elastic Stack(ELK)의 구조와 활용을 실습하는 것을 1차 목표로 하며,
이를 위해 실제 카드 소비 데이터를 분석하고 은행의 디지털 전략 및 맞춤형 카드 기획 방향을 제시하는 컨설팅을 수행하였습니다.

- ElasticSearch: 비정형 데이터를 효율적으로 색인하고 분석

- Logstash & Filebeat: 데이터 수집 및 파이프라인 구축

- Kibana: 대시보드를 활용한 시각화 및 패턴 도출

## ✅ 프로젝트 개요
- 학습목표: ELK Stack을 이용한 데이터 수집, 저장, 분석 및 시각화 실습

- 분석목표: 연령대별/라이프사이클별 카드 소비 특징 분석, 디지털 채널 이용 행태 분석

- 결과목표: 은행 디지털 전략에 대한 인사이트 도출 및 카드 추천 방향 제시


## 🪐 Filebeat + Logstash + ElasticSearch + Kibana데이터 파이프라인

---

## 1️⃣ Filebeat 역할

- **CSV 파일 변경 감지 → Logstash 전송**
- 경로: `C:\ce5\dataset\carddata.csv`
- 포트: `localhost:5044`

```yaml
# filebeat.yml

filebeat.inputs:
  - type: log
    enabled: true
    paths:
      - C:\ce5\dataset\carddata.csv

...

output.logstash:
  hosts: ["localhost:5044"]
```

---

## 2️⃣ Logstash 역할

- CSV 데이터를 `,` 기준으로 컬럼별 분리
- message에서 필요한 데이터만 수집
- `YYYYMM` → `MM` 부분만 추출하여 날짜 필드에 덮어씀
- 연령, 거래금액 등을 `integer`로 변환
- Elasticsearch로 전송해 Kibana 시각화 가능

```ruby
# carddata.conf
mutate {
  split => ["message", ","]
  add_field => {
    "■■■" => "%{[message][2]}"
    "■■■" => "%{[message][3]}"
    ...
  }
}

# 형변환
  mutate {
    convert => {
      "■■■"     => "integer"
      "■■■"     => "integer"
    }
    remove_field => ["@timestamp"]
  }
  
# 월 추출 예시
ruby {
  code => "
     if event.get('■■■■')
        ytm = event.get('■■■■').to_s
        if ytm.length == 6
          mm = ytm[4,2]
          event.set('■■■■', mm)
        end
      end
  "
}

```

<details>
<summary>🖼️ 필터링 전/후 비교 </summary>

- **필터링 적용 전**  
  <img width="600" height="550" alt="filtering-before" src="https://github.com/user-attachments/assets/d2eedc06-c888-4546-b726-15d8ed392e76" />

---

- **필터링 적용 후**  
  <img width="600" height="550" alt="filtering-after" src="https://github.com/user-attachments/assets/9b7cb199-5b57-435d-b06f-cf62fd6366a0" />

</details>

---

## 3️⃣ ElasticSearch 역할

- Logstash에서 전송된 **필터링한** JSON 데이터를 인덱싱
- 인덱스 이름: `carddata`
- 다양한 필드를 기준으로 Kibana에서 분석 가능
- 연령별, 성별, 월별 소비 패턴 등 대시보드 시각화 지원
- 색인 구조 자동 생성 또는 매핑 템플릿 지정 가능

---

## 4️⃣ Kibana 역할

- **Elasticsearch에 저장된 데이터 시각화 및 분석**
- 인덱스 패턴: `carddata`
- Discover 탭에서 데이터 탐색 가능
- Lens 및 Visualize 탭을 통해 다음과 같은 분석 가능

