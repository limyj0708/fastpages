---
title: Crontab으로 Python 스크립트 주기적으로 실행하기
categories: Linux
layout: post
---

주기적으로 외부 API를 통해 데이터를 수집하여, Bigquery에 적재하고 싶다.
CentOS 서버에서, 주기적으로 Python 스크립트를 실행하여 해결해 보자.

1. `sudo crontab -e`  : crontab 설정 오픈. 자동으로 root가 작업하는 것으로 인지됨
2. 설정
   1. 경로는 절대경로를 입력해야 제대로 작동
      * Python 경로도 절대경로로 입력해 줘야 함
   2. 시간설정은 아래 링크에서 직관적으로 확인 가능
      * [Crontab.guru - The cron schedule expression editor](https://crontab.guru/#30_8_*_*_*)

```shell
30 8 * * * /usr/local/bin/python3.9 /home/limyj0708/cw_daily_bigquery/cw_daily.py
```

3. cron 재시작
   1. 재시작해야 적용됨
   2. `service cron restart`
   3. CentOS일 경우, `service crond restart`

