---
date: 2021-11-08
title: "Python Mock을 사용한  TestCase 작성"
categories: 
  - Python
tags:
  - Python
  - TestCase
---

## TestCase 클래스

### uniittest 내장 모듈을 사용한 Python test code 작성

Test를 작성할 때, 파이썬에서 테스트를 작성하는 표준적인 방법은 unittest 내장 모듈을 쓰는 것이다.

예를 들어, utils.py의 to_str을 테스트하는 코드를 작성하고 싶으면

util_test.py를 만들고 해당 모듈을 import한다. 

```python
from unittest import TestCase, main
from utils import to_str
```

그 후 명령줄에서 테스트 파일을 실행한다. 

```python
if __ name __ =='__main__':
	main() 
```

테스트는 아까 import했던 TestCase의 하위 클래스로 구성된다. 

각각의 **test case**는 **test**라는 단어로 시작하는 **method**들에 매핑되는데, 어떤 test method도 exception을 발생시키지 않고 실행이 끝나면 테스트가 성공한 것으로 간주한다.

Test 중에 일부가 실패하더라도, TestCase 클래스는 최초로 문제가 발생한 지점에서 실행을 중단하지 않고, 나머지 test method를 실행해서 test 전체에 대한 전반적인 그림을 그릴 수 있게 해준다

### Helper method들

TestCase 클래스는 test에 assertion(이건 이거다.. 단언하는 단언문) 을 만들 때 도움이 되는 여러 helper methods를 제공한다. 예를 들어서, assertTrue는 주어진 bool 이 참인가? 검증, assertEqual은 두 값이 같은지 비교하는 것을 검증한다. 

```python
class AssertTestCase(TestCase):
	def test_assert_helper(self):
			expected = 12
			found = 3 * 5
			self.assertEqual(expected, found)

if __name__ '__main__':
	main()
```

### Helper method 튜닝하기

직접 TestCase를  작성할 수도 있다. 근데 이제 helper method는 test로 시작하지 않아야함. 왜냐면 test로 시작하는 케이스를 작성해버리면 helper method가 아니고 test case로 취급되기 때문임

### tearDown, setUp, setUpModule, tearDownModule을 사용해 각각의 테스트를 격리하기

테스트를 제대로 진행하기 위해서는 각각의 테스트들을 격리해야하고, test 환경을 구축하는 과정에서 TestCase를 상속받은 자식클래스 안에서 setUp과 tearDown 메서드를 override해야한다.

setUp 클래스 : test method를 실행하기 전에 호출됨

tearDown 클래스 : test method를 실행한 후에 호출됨

2**개 method를 활용하면 각각 테스트를 서로 격리된 상황에서 실행할 수 있도록 해준다 오옹..**

### 통합테스트 할 때 돈/자원 아끼기 : setUpModule() / tearDownModule()을 이용해서

프로그램이 복잡해지면 코드를 독립적으로 실행할 mock 등의 도구를 사용하는데, 그 전에 만약에 통합테스트를 진행하고 싶을 때 혹시 테스트 환경을 구축할 때 비용이 비싸지거나 (클러스터에 서버 띄우고 그 위에서 테스트 돌리거나 DB 접근하거나..) 할 수 있다. unittest 모듈은 비싼 자원을 단 한번만 초기화하고, 초기화를 반복하지 않고도 모든 TestCase의 클래스랑 test method를 실행할 수 있도록 한다. 

```python
@classmethod
    def tearDownClass(cls) -> None:
        super().tearDownClass()
        disconnect()

    def tearDown(self, *args) -> None:
        print()
        print('(tearDown) ==> Delete all data_sources')
        data_source_vos = DataSource.objects.filter()
        data_source_vos.delete()
```

## Mock을 사용해 dependancy가 복잡한 코드 테스트하기

사용하기에 너무 무거운 함수들은 mock을 만들어서 사용하는 기능이 있다. 

ex. 동물원에서 먹이 주는 시간 관리하는 프로그램 → Database에서 특정 종에 속하는 모든 동물 조회해서.. 최근에 먹이 준 시간을 query해서... (복잡)

정말로 database 서버를 기동하고 그 서버에 연결해서 테스트하면, 단순히 단위테스트 진행하는 주제에 schema 설정하고, data 채워넣고... 너무 많은 리소스 작업이 필요하다.

그래서 Mock을 씀

mock : 자신이 흉내 내려는 대상에 dependancy를 갖고 있는 다른 함수들이 어떤 요청을 보내면 어떤 응답을 보내야 할지 알고, 요청에 따라서 적절한 response를 돌려준다. 

### How to use

Mock class의 return_value라는 attribute에다가  mock이 호출되었을 때 돌려줄 값을 적어준다. spec인자에는 mock을 호출할 애를 적는다.

```python
from unittest.mock import Mock

mock = Mock(spec=get_animals)

expected = [

('점박이', datetime(2021,6,5,11,15)),
('털보', datetime(2021,6,5,11,15))
]
mock.return_value = expected
```

SpaceONE에서 쓰는 예시

```python
mock_list_secrets.return_value = {
            'results': [{
                'secret_id': secret_id,
                'schema': 'aws_access_key'
            }],
            'total_count': 1
        }
```

**assert_called_once_with**

mock을 호출한 코드가 mock 에게 제대로 된 인자를 전달했는지 확인할 수 있도록 해주는 함수.

mock.assert_called_once_with(database, '기린') 잘못된 파라미터를 전달하면 예외가 발생하고 이 assertion을 사용한 TestCase는 실패한다. 

### unittest.mock.patch 관련 함수들

patch 함수는 임시로 모듈이나 클래스의 attribute에 다른 값을 대입해준다.

ex. database에 접근하는 다른 함수들을 임시로 다른 함수로 대치해줌. 

- 함수 decorator 로 써도 되고
- with문 내에서 사용할 수도 있고
- TestCase 클래스 안의 SetUp이나 tearDown안에서 사용할 수도 있음

SpaceONE 사용 예시

```python
@patch.object(PluginManager, 'get_plugin_endpoint', return_value={'endpoint': 'grpc://plugin.spaceone.dev:50051', 'updated_version': '1.2'})
```

_End of docs_