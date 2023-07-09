# Node.JS + Mysql 연동 [develop-mysql]

### mysql 모듈설치

*우선 필요 모듈부터 설치를 진행합니다. 데이터베이스 설치 및 설정은 생략하겠습니다.*

```bash
npm install mysql --save
```

`db_info.js`

```javascript
module.exports = (function () {
  return {
    local: { // localhost
      host: 'localhost',
      port: '3306',
      user: 'root',
      password: '',
      database: ''
    },
    real: { // real server db info
      host: '',
      port: '',
      user: '',
      password: '!',
      database: ''
    },
    dev: { // dev server db info
      host: '',
      port: '',
      user: '',
      password: '',
      database: ''
    }
  }
})();
```

데이터베이스의 정보가 저장돼있는 곳입니다. 로컬 환경, 실제 서버 환경, 테스트 서버 환경에 필요한 정보를 입력해주세요.

`db_con.js`

```javascript
var mysql = require('mysql');
var config = require('../db/db_info').local;

module.exports = function () {
  return {
    init: function () {
      return mysql.createConnection({
        host: config.host,
        port: config.port,
        user: config.user,
        password: config.password,
        database: config.database
      })
    },

    test_open: function (con) {
      con.connect(function (err) {
        if (err) {
          console.error('mysql connection error :' + err);
        } else {
          console.info('mysql is connected successfully.');
        }
      })
    }
  }
};
```

데이터베이에 커넥트 하는 부분을 담당합니다. config 변수를 보시면 local 프로퍼티를 이용해서 db_info.js에 있는 local 객체를 config 변수에 바인딩시킵니다.

init 메서드를 보시면 방금 바인딩시킨 config 값의 기반으로  mysql connection을 생성이 가능합니다. 또 `var config = require('../db/db_info').(XXX)` 이런 형태로 `db_info.js`에 저장돼있는 데이터베이스의 정보를 손쉽게 변경이 가능합니다.

**router에서 사용하기 (1)**

```javascript
var mysql_dbc = require('../db/db_con')();
var connection = mysql_dbc.init();
mysql_dbc.test_open(connection);
```
mysql_dbc 변수에  db_con의 값을 require을 시키고, 위에서 설명한 init 메서드를 통해서 생성된 Mysql Connnection을 변수 connection에 저장시킵니다


**router에서 사용하기 (2)**


```javascript
router.get('/mysql/test', function (req, res) {
  var stmt = 'select *from ....';
  connection.query(stmt, function (err, result) {
    .....
  })
});
```

위에서 설명한 connection 객체를 이용해서 쿼리 작업을 진행하시면 됩니다.

![EC2 Innound](https://i.imgur.com/yloTaE9.png)

서버를 실행시키면 입력된 데이터베이스와 연결이 되고 `test_open` 메서드를 통해서 데이터베이스의 컨넥션이 제대로 생성이 됐나 간단한 로그로 출력됩니다. 이처럼 서버를 실행시켰을 때 데이터베이스와 제대로 연결되었는지 정보를 출력하시는 것을 권장합니다.

