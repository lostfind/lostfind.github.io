---
layout: post
title:  "가계부 자산관리 모델"
date:   2019-05-03 15:20
author: lostfind
categories: Development
tags: PHP 가계부
cover:  "/assets/instacode.png"
---

'자산관리'의 모델 부분 개발의 현재까지 진행상황 정리

## 개발 과정
### 일단 스파게티 코드
```PHP
class Account
{
  // 데이터 항목
  public $id = '';
  public $accountName = '';
  public $balance = 0;
  public $sort = 0;
  public $accountType = '';
  public $accountTypeName = '';

  function getAll()
  {
    $mysqli = new mysqli('localhost', 'dwkim', 'dwkim', 'accountbook');
    if (mysqli_connect_errno()) {
      printf("Connect failed: %s\n", mysqli_connect_error());
      exit;
    }

    $query = "쿼리생략";
    $result = mysqli_query($mysqli, $query);
    $mysqli->close();

    return $result;
  }

  function getAccount($id)
  {
    // ...생략
    if ($result) {
      $this->id = $result->id;
      // ...생략
    }
  }
}
```
#### 현재 코드의 문제점
+ DB커넥션이 각 메소드에서 반복되며 되고 관리가 불편함
+ `mysqli`를 사용함으로 특정 DB에 의존

#### 해결방안
+ DB커넥션을 `Model`클래스에 구현하고 각 모델에서 상속
+ [PDO](https://www.php.net/manual/en/class.pdo.php)로 DB의존해소


### DB연결 부분 분리
```PHP
class Model
{
  public $db;

  function __construct() {
    $dbServer = 'localhost';
    $dbUser = 'dwkim';
    $dbPass = 'dwkim';
    $dbName = 'accountbook';

    $dsn = "mysql:host={$dbServer};dbname={$dbName};charset=utf8";

    try {
      $this->db = new PDO($dsn, $dbUser, $dbPass);
    } catch (PDOException $e) {
      echo '접속실패' . h($e->getMessage());
    }
  }
}
```

```PHP
class Account extends Model
{
  // ...생략
  function getAll()
  {
    $sql = "쿼리생략";
    $prepare = $this->db->prepare($sql);
    $prepare->execute();
    $result = $prepare->fetchAll(PDO::FETCH_ASSOC);

    return $result;
  }
}
```
#### 현재 코드의 문제점
+ 전체 레코드를 조회하는 `getAll()`메소드는 데이터 항목 변수를 사용하지 못함
+ 한 레코드 조회용 `getAccount()`와 `getALL()`이 반환하는 형식이 달라 컨트롤러에서 각각 대응이 필요

#### 해결방안
+ 데이터 객체를 별로도 작성하여 컨트롤러와는 데이터 객체를 통해 데이터를 주고 받는다

### 데이터 객체 생성
테이블 구조와 거의 동일한 데이터 클래스 `AccountData` 작성
```PHP
class AccountData
{
  public $id = '';
  public $accountName = '';
  public $balance = 0;
  public $sort = 0;
  public $accountType = '';
  public $accountTypeName = '';

  function __construct($data = array())
  {
    foreach ($data as $key => $value) {
      switch ($key) {
        case 'id':
          $this->id = $data[$key];
          break;
        case 'account_name':
        case 'accountName':
          $this->accountName = $data[$key];
          break;
        case 'balance':
          $this->balance = $data[$key];
          break;
        case 'sort':
          $this->sort = $data[$key];
          break;
        case 'account_type':
        case 'accountType':
          $this->accountType = $data[$key];
          break;
        case 'account_type_nm':
        case 'accountTypeName':
          $this->accountTypeName = $data[$key];
          break;
      }
    }
  }
}
```

```PHP
class Account extends Model
{
  // ...생략
  function getAll()
  {
    $sql = "쿼리생략";
    $prepare = $this->db->prepare($sql);
    $prepare->execute();
    $result = $prepare->fetchAll(PDO::FETCH_ASSOC);

    $accounts = array();

    foreach ($result as $row) {
      $account = new AccountData($row);
      $accounts[] = $account;
    }

    return $accounts;
  }

  function getAccount($id)
  {
    $sql = "쿼리생략";
    $prepare = $this->db->prepare($sql);
    $prepare->bindValue(':id', $id, PDO::PARAM_INT);
    $prepare->execute();
    $result = $prepare->fetch(PDO::FETCH_ASSOC|PDO::FETCH_UNIQUE);

    $account = new AccountData($result);

    return $account;
  }
  // ...생략
}
```

## 남은 과제
+ DB커넥션에 대한 정보가 Model에서 Config로 옮기기
  + (Model에 있는 것보다 Config에서 관리하는게 타당할 경우)
+ 데이터객체의 get, set기능
  + DB의 스네이크 표기법과 컨트롤러, 뷰의 카멜 표기법 자동 매칭
  `account_name` ↔ `accountName`
