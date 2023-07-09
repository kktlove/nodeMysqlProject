
## Crontab을 이용한 노드 API 호출 [develop-crontab-api]

### Crontab 간단 설명
Crontab은 스케줄링을 관리해주는 프로그램으로 특정 시간에 사용자가 직성한 스크립트나 명령어을 간단하게 실행시킬수 있습니다. 시간 설정이 매우 간단하고 직관적이라서 쉽게 사용 가능한게 가장큰 장점입니다.

    *    *    *    *    *  수행할 명령어
    ┬   ┬   ┬   ┬   ┬
    │   │   │   │   │
    │   │   │   │   │
    │   │   │   │   └───────── 요일 (0 - 6) (0 =일요일)
    │   │   │   └────────── 월 (1 - 12)
    │   │   └─────────── 일 (1 - 31)
    │   └──────────── 시 (0 - 23)
    └───────────── 분 (0 - 59)

### Crontab 등록

  Crontab 설치는 여기서 다루지 않고 진행하겠습니다. 등록하기 전에 crontab에서 실행될 쉘스크립트 파일을 생성합니다. 해당 스크립트 작성은 Crontab등록이후 진행하겠습니다. 저 같은 경우는 crontab.sh로 생성했습니다.

```bash
crontab -e
5 0 * * * /path/../crontab.sh
```
crontab -e 명령어를 입력하게 되면 VI 모드로 반복될 시간 설정과 해당 스크립프 파일 경로를 설정합니다. 위에서 설명 했다 싶이 매일 12시간 05분에 crontab.sh를 호출하게 설정하였습니다.

```bash
crontab -l
```
위의 명령어로 crontab 등록을 확인합니다.

`crontab.sh` 스크립트 파일 작성

```bash
#!/bin/bash

echo 'start delete table info...';

curl -H "Content-Type: application/json" -X DELETE -d '
  {"sql":"delete from `user` where  `user_id` is not null;"}
  ' http://localhost:3500/api/v1/crontab

echo 'success...';
```

사실 게임데이터는 레디스에 저장하고 있어 셈플 예제로는 다소 복잡해서 이전에 설명했던 예제로 간단한 user 테이블의 모든 항목의 user 칼럼을 지우는 예제로 대체 하겠습니다. "DELETE"을 PUT, POST로 변경하셔도 그대로 작성 가능합니다

### Node API 설정

```javascript
router.delete('/crontab', function (req, res) {
  var sql = req.body.sql;
  connection.query(sql, function (err, result) {
    if (err) {
      res.json({
        success: false,
        err: err
      });
    } else {
      res.json({
        success: true,
        msg: 'Delete Success'
      })
    }
  });
})
```
req.body.sql 객체로 쿼리문을 받고 해당 쿼리문을 실행하는 간단한 예제입니다.
### Crontab 실행 확인

![EC2 Innound](https://i.imgur.com/BFB2EKG.png)


위의 예제는 12시 05분에 실행되니 그 시간까지 기다릴 수는 없죠…. 그래도 1분에 한 번 실행되는 crontab 설정으로 실행이 되나 테스트를 진행하는 것도 권장합니다. 또한, 스크립트 파일을 명령어로 직접 실행시켜 일단 스크립트 파일이 제대로 작성되었나부터 확인하시는 것이 더욱 더 권장합니다.

