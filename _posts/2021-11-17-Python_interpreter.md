---
date: 2021-11-17
title: "Python Interpreter의 작동방식"
categories: 
  - Python
tags:
  - Python
---
![interpreter](https://user-images.githubusercontent.com/25656426/140708704-45c09f51-a974-40da-b688-49bf0f76494e.png)


### 들어가기 전 전 상식

**Compile** : 프로그래밍 언어를 Runtime 이전에 기계어로 해석하는 작업 (원시 코드 → object code)

**Interprete** : Runtime 이후에 Row단위로 해석하며 프로그램을 구동시키는 방식. 프로그래밍 언어를 기계어로 바로 바꾸지 않고 중간 단계를 거친 뒤에 런타임에 직접 코드를 구동시킴. 

### Python interpreter를 trigger하고 작동하는 방식

**python 명령자를 전달**함으로써 python interpreter가 trigger되고, source code가 전달된다.

`ex. python abc.py`

Python interpreter의 작동 stages는

1) Lexing 

전달된 code를 가지고 line을 쪼개 코드를 Token으로 generate한다. 

2) Parsing

Generated token이 parser에게 전달되어 파싱됨. token을 가지고 Abstract Syntax Tree라는 token(code들)사이의 관계를 나타내는 트리 구조를 만듦.

![image](https://user-images.githubusercontent.com/25656426/140708836-98949bbe-dafb-4260-92ea-4135934bd075.png)


3) Compiling 

Parsed code(Abstract Syntax tree)가 compiler에게 전달되고, compiler가 이 Abstract Syntax Tree를 가지고 **Byte code** 라는  `.pyc` 형태의 intermediate language code를 생성됨

- 자바 소스코드와의 차이점? 자바코드는 interpreter에게 소스가 전달되기 전에  컴파일러에 의해 컴파일됨. (compiler가 interpreter outside에 존재함. 파이썬은 인터프리터/컴파일러가 인터프리터 안에 함께 존재해서 performance lag를 줄임)

4) Virtual Machine으로 .pyc가 전달되어 Interpreting 작업 수행

실제로 interpreting하는 stage. operating system을 simulate함. 

실제 interpreting 작업 (Row 단위로 해석하며 프로그램을 구동하는 방식)을 담당하며, byte code를 machine code로 변환함

python interpreter 작동방식까지 공부하게 된 이유...Exception이 raised된 다음에 내 화면에.작업 console에. 프린트문으로. 찍히기까지 일어나는 일들이 너무 궁금해서...

### Exception이 발생되면 파이썬 내부에서는 무슨일이 일어나니?

Exception이 raised되면 일어나는 일

1.  exception이 handled되기 전까지는 현재 block의 어떤 statements나 코드도 실행되지 않는다. 
2. interpreter는 print-loop를 return 시켜주며, file argument로 파이썬이 시작된 경우에는? terminate한다. 
3. python interpreter는 stack backtrace를 출력한다. stack-backtrace는 explanation이 담긴 텍스트 블록 집합으로, 예외가 발생한 실행분기에서 활성화되었던 function들의 call을 나타내는 branch 텍스트 블록 집합이다. 
    
    
```
    Traceback (most recent call last):
      File "/usr/local/lib/python3.8/site-packages/spaceone_inventory-1.8.5-py3.8.egg/spaceone/inventory/api/v1/server.py", line 46, in list
        server_vos, total_count = server_service.list(params)
      File "/usr/local/lib/python3.8/site-packages/spaceone/core/service/service.py", line 62, in wrapped_func
        return _pipeline(func, self, params, append_meta)
      File "/usr/local/lib/python3.8/site-packages/spaceone/core/service/service.py", line 121, in _pipeline
        raise e
      File "/usr/local/lib/python3.8/site-packages/spaceone/core/service/service.py", line 98, in _pipeline
        handler.verify(params)
      File "/usr/local/lib/python3.8/site-packages/spaceone/core/handler/authorization_handler.py", line 37, in verify
        self._verify_auth(params, scope)
      File "/usr/local/lib/python3.8/site-packages/spaceone/core/handler/authorization_handler.py", line 75, in _verify_auth
        raise ERROR_PERMISSION_DENIED()
    spaceone.core.error.ERROR_PERMISSION_DENIED: 
    	error_code = ERROR_PERMISSION_DENIED
    	status_code = PERMISSION_DENIED
    	message = Permission denied.
```
    
    

### 출력은 어떻게 하는데?

traceback 이라는 python의 모듈이 있음. 파이썬 프로그램의 StackTrace를 추출, format, print하는 표준 인터페이스를 정의함!

traceback module은 traceback이라는 Object를 사용하는데, sys.last_traceback이라는 변수에 저장되고 sys.exc_info()라는 함수의 인자로 사용된다고 한다..

https://docs.python.org/ko/3.7/library/traceback.html#module-traceback


### sys module

파이썬의 sys module은 파이썬 런타임에 interpreter의 구성을 확인하고, 변경할 수 있는 서비스를 제공하는 모듈임. 프로그램 외부의 작업환경과 상호작용할 수 있는 서비스를 제공한다. 

ex. build time 버전 정보를 설정한다거나, 인터프리터를 빌드하는데 사용되는 운영체제 플랫폼을 정의한다던가.. 

정리하자면 Traceback 로그가 출력되는 과정은,

런타임환경 외부의 작업환경인 콘솔의 print와 상호작용하기 위해 sys.exc_info() 라는 함수의 인자에다가 traceback object로 생성된 객체를 전달해 내눈까지 로그가 전달되는 기나긴 여정.. 이었다고 한다..! 궁금증 해결

[https://docs.python.org/ko/3.7/library/traceback.html](https://docs.python.org/ko/3.7/library/traceback.html)