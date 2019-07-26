# 비동기 - Call Back & Promise

## 1. javascript는 동기식 언어입니다.

  javascript는 싱글 스레드이며 동기식 언어입니다. 스레드가 하나이기 때문에 한가지의 프로세스를 이어갈 수 있으며 현재 일이 끝나지 않는다면 프로그램은 blocking됩니다. 즉, 한가지 일만을 수행할 수 있으며 현재 일이 끝나지 않은 경우에 다음 작업을 할 수 없기 때문에 프로그램은 중지된 것처럼 보입니다.

  웹 사이트에서 얼마간의 대기 시간을 필요로 하는 경우는 생각보다 많습니다. 네트워크 응답 대기, 사용자 입력 대기 등... 기약없는 응답을 기다리는 상황은 언제든 필요할 수 있는데요.  이런 상황에서도 프로그램은 다음 작업을 처리하여 매끄럽게 진행돼야 합니다. 

  비동기 처리는 네트워크를 사용하는 프로그램에 반드시 필요한 기술이며 다른 언어들도 각자 자신만의 고유한 처리 방식이 있습니다. 몇가지 멀티 스레드를 지원하는 경우엔 스레드를 활용하여 비동기를 처리하기도 합니다. 자바스크립트도 싱글스레드로서 자신만의 고유한 비동기 상황에 따른 대처 방법이 존재해 왔었고 특히 ES6이후 promise 도입을 계기로 큰 변화가 일어났습니다.

## 2. Call Stack

  자바스크립트의 대표적인 엔진은 V8입니다. node 환경에서도 chrome에서도 V8엔진을 사용하는데요. 이 엔진은 기본적으로 Memory Heap과 Call Stack으로 구성됩니다. 자바스크립트는 싱글 스레드로 동작합니다. 한가지 일만 할 수 있기 때문에 Call 스택의 최상위 계층의 함수가 현재 실행되고 있는 함수입니다. 

  자바스크립트 엔진은 싱글 스레드로 동작합니다. 즉, 하나의 콜스택만을 가지고 있으며 한번에 한가지 일만 처리할 수 있습니다. 콜스택은 기본적으로 현재 프로그램에서 실행되고 있는 작업을 가리키는 데이터 구조로 설계되어 있습니다. 

    function c(z) {
        console.log(new Error().stack); // (A)
    }
    function b(y) {
        c(y); // (B)
    }
    function a(x) {
        b(x); // (C)
    }
    a(3); // (D)
    
    //Console
    //Error
    //    at c (ArrayPrototype.js:2)
    //    at b (ArrayPrototype.js:5)
    //    at a (ArrayPrototype.js:8)
    //    at ArrayPrototype.js:10
    

> Error.prototype.stack은 가장 최근의 호출부터 시작되어 이전 호출 스택을 타고 나아가 전역 스택까지 도달하여 Call Stack에 쌓인 스택들을 확인할 수 있는 메소드입니다. 위 코드의 실행은 다음과 같습니다.

1. 전역 공간에서 a함수가 호출된다.
2. a 함수가 실행되고 b 함수가 호출된다. (Call Stack에 a 함수가 쌓인다.)
3. b 함수가 실행되고 c 함수가 호출된다. (a함수 위에 b 함수가 쌓인다.)
4. c 함수가 실행되고 Error.prototype.stack 메소드가 실행되고 호출 스택들이 표시된다. (b함수 위에 c 함수가 쌓인다.)

## 3. javascript의 비동기 처리(ES6이전의 이벤트 루프 큐 관리 방식)

  javascript 코드는 싱글 스레드로 처리되지만 node나 chrome은 멀티 스레드로 작업을 처리합니다. 브라우저에서 비동기 처리는 다음과 같습니다.

- 비동기 코드 확인 → web api → 이벤트 큐 —(이벤트 루프)—> 실행

  사실 ES6이전에 자바스크립트에 비동기라는 개념은 없었습니다. 자바스크립트 엔진은 싱글 스레드로서 그저 어떤 시점에서 처리해야 할 작업들을 하나씩 실행 했을 뿐입니다. 주어진 작업들을 동기적으로 처리하는 자바스크립트 엔진이 어떻게 비동기 작업들을 동시에 처리할 수 있을까요? 또, 프로그램에는 DOM, AJAX, setTimeout 등.. 처리해야 할 작업들이 많지만 자바스크립트 엔진은 이것들을 처리할 수 있는 api를 가지고 있지 않습니다.

  어떻게 setTimeout함수나 ajax 작업을 처리할 수 있을까요? 그 답은 호스팅 환경에 있습니다. 자바스크립트 엔진은 싱글 스레드이지만 호스팅 환경(node, 브라우저 등...)은 멀티 스레드로 동작합니다. 자바스크립트 엔진이 싱글 스레드로 작업들을 처리하고 있을때 호스팅 환경의 background 스레드가 자바스크립트 엔진을 통해 수행할 수 없는 작업들이 web api를 통해 처리할 수 있게 됩니다.  콜스택이 비워 졌을 때 호스팅 환경에서 api를 이용한 과정을 통해 생성된 새로운 일감이 이벤트 루프를 통해서 스택이 생성되고 실행됩니다.

  다시 말해, 이벤트 스케줄링은 호스팅 환경에서 일어납니다. ajax로 데이터 요청을 할 때에 호스팅 환경은 네트워크를 통해 응답을 기다립니다. 응답이 이루어진 데이터는 이벤트 루프에 추가되고 콜스택이 비워지면 실행됩니다.

## 4. 이벤트 루프

  모든 웹 페이지에는 메인 스레드가 있습니다. 메인 스레드는 콜 스택의 작업들을 하나씩 처리합니다. 동기 코드로 작성한 웹 페이지라면 이 작업이 어떠한 방해 없이 자연스럽게 일어날 것입니다. 하지만 비동기 코드가 추가된다면 조금 다른 프로세스를 가지게 됩니다. 비동기 코드의 콜백 함수는 web api를 통해 어떠한 처리 과정을 거쳐 이벤트 큐에 자리 잡게 되고 이벤트 루프를 통해 콜 스택에 새로운 작업을 추가합니다. 

  이벤트 루프는 항상 콜 스택을 주시합니다. 이벤트 루프는 큐에서 가장 앞에 있는 이벤트를 꺼내고 콜 스택이 비워지면 새로운 이벤트를 실행하는데요. 이것을 '틱'이라고 합니다.

    while(true){
    	task = eventLoop.nextTask();
    	if(task){
    		task.excute();
    	}
    	eventLoop.excuteMicrotasks();
    	if(eventLoop.needsRendering())
    		eventLoop.render();
    }

## 5. 콜백

  자바스크립트에서 비동기를 처리하는 가장 기본적인 처리 패턴입니다. 다시 말해, 콜백을 사용하면 비동기 함수를 만들 수 있습니다.

    const getName = () => {
    	callback();
    };
    
    getName(() => console.log(name));
    
    let name = 'B-SENS';

  코드의 실행 결과는 무엇일까요? 

  'Uncaught ReferenceError: name is not defined' 에러가 발생합니다. 위 코드는 오직 동기적인 흐름으로 실행됩니다. 에러가 발생된 이유는 변수가 불려지는 곳보다 아래에서 선언되었기 때문입니다. 자바스크립트 엔진은 첫번째 라인부터 주어진 작업을 수행했습니다. 하지만 변수 name는 호이스팅 과정 중 초기화도 이루어지지 않은 TDZ 단계에 있었기 때문에 해당 에러가 발생했습니다. 데이터를 얻기위해 비동기로 무언가를 요청하는 코드에서도 같은 문제가 발생할 수 있습니다. 그렇다면 이 문제를 어떤 방식으로 해결해야 할까요?

    const getName = (callback) => {
        setTimeout(() => {
            callback();
        },0);
    };
    
    getName(() => console.log(name));
    
    let name = 'B-SENS';

  

  위의 코드를 실행시키면 원하는 대로 'B-SENS'이 출력됩니다. 함수 호출이 일어나고 동기 코드 흐름에 따라 콜스택이 비워집니다. 이후 setTimeout의 콜백함수가 web의 타이머 api를 통해 처리됩니다. 이후 이벤트 루프를 통해 콜 스택에 쌓이고 실행이 되는데요. 이때 getName 함수 스코프 내의 렉시컬 환경에 있던 callback이 setTimeout 콜백 함수의 클로저 스코프에 남아있어서 정상적으로 동작하는 것입니다. 이처럼 ES6 이전에도 비동기는 콜백 함수를 통하여 처리할 수 있었습니다. 다만, 이 과정에는 다음과 같은 몇가지 문제점이 있었습니다. 

## 5.1 Callback Hell

    function getAsyncValue(cb) {
      setImmediate(() => {
        console.log("Async Task Calling Callback");
        cb();
      });
    }
    
    getAsyncValue('something', (err, param) => {
      getAsyncParam('something' + param, (err, result) => {
        getAsyncResult('something/' + result, (err) => {
          getAsyncValue(..., (result) => {
            getAsyncValue(..., (result) => {
            });
          });
        });
      });
    });

  콜백을 이용하여 비동기를 처리할 때 접하는 문제입니다. 비동기 콜백 함수들 간의 관계에 의한 체인이 여러 개 생성되는 것입니다. callback hell은 가독성도 좋지 않고 에러를 처리하는 데에도 큰 문제가 있습니다. 이 문제를 해결하기 위해 콜백 함수를 통한 여러가지 방법들이 있지만 promise가 만들어진 이후로는 더 효과적인 개선이 이루어졌습니다. 이후 promise 설명에서 이 부분을 다룹니다.

  

## 5.2 Error Handling

    동기 코드로 작성된 프로그램이라면 try... catch...를 통하여 명시적으로 에러 처리를 할 수 있습니다. 하지만 비동기 함수에서는 불가능합니다. 비동기 함수는 반환값을 즉시 계산할 수 없기 때문에 에러도 콜백 함수를 통해 처리해야 합니다. 

    function response(err, data){
    	if(err){
    		console.error(err)
    	} else {
    		console.log(data)
    	}
    }

  비동기 코드에서 콜백의 에러 처리 문제는 콜백 헬과 관련 있습니다. 콜백 헬이 발생되어 가독성이 좋지 않은 수많은 코드들이 있을 때 오류 검사를 건너 뛰는 경우는 생각보다 많습니다. 

## 5.3 제어권 문제

    function doAsyncTask(){
    	something...
    }
    
    thirdPartyController(doAsyncTask);

  콜백을 사용한다는 것은 제 3자에게 제어권을 준다는 이야기입니다. 때때로 코드에서 콜백 함수를 사용자와 상호 작용하는 방식으로 구현할 때가 있습니다. 제어권을 벗어난 함수가 발생하는 시점마저 확인할 수 없을때 의도치 않은 버그가 발생할 수 있습니다. 따라서 정확한 프로세스에 따른 콜백함수의 작동이 프로그램의 버그를 줄일 수 있습니다. 

## 6. Promise

  Promise은 비동기를 처리하기 위한 콜백의 대안입니다. 비동기 처리와 에러 핸들링을 더욱 직관적이고 명료하게 처리할 수 있습니다. 다음 함수는 Promise을 통해 비동기 결과를 reject, resolve로 반환합니다.

- promise의 결과가 성공이라면 promise.then((resolve, reject))에서 resolve의 콜백 함수는 즉시 큐에 추가됩니다.
- promise의 결과가 성공이라면 promise.then((resolve, reject))에서 reject의 콜백 함수는 즉시 큐에 추가됩니다.

    function asyncFunc() {
        return new Promise(
            function (resolve, reject) {
                ···
                resolve(success);
                ···
                reject(error);
            });
    }
    
    let promise = asyncFunc();
    
    promise
    	.then(result => { ··· })
    	.catch(error => { ··· });

## 6.1 ES6 이후 javascript의 비동기 처리

    setTimeout(_ => console.log('timer1'), 0);
    
    Promise.resolve('promise1').then(result => console.log(result));
        
    setTimeout(_ => console.log('timer2'), 0);
        
    Promise.resolve('promise2').then(result => console.log(result));
        
    // 출력
    // promise1
    // promise2
    // timer1
    // timer2

  setTimeout 타이머 함수와 promise 함수를 통해 비동기 코드를 작성했습니다. setTimeout의 타이머를 0으로 설정 하였는데도 promise의 값이 먼저 출력되었습니다. 이 결과를 설명하기에 필요한 개념들을 정리한 후 분석해 보겠습니다. 

## 6.1.1 Job queue, microTask

  기존의 call stack과 event queue 로 구성 되었던 비동기 처리 계층에 Job queue가 새롭게 추가되었습니다. Job queue에는 추가되는 작업들을 microTask라고 부르는 데요. 이 글에서 설명하고 있는 promise에 의한 작업이 microTask로 Job queue에 추가됩니다. 

  Job queue에 작업들이 있다면 콜 스택이 비어 있을 때 즉시 실행됩니다. 또한 현재 처리해야 할 작업이 있다고 한다면 우선권을 가지고 작업의 바로 뒤에서 바로 실행이 됩니다. promise가 확정 되면 콜백 함수는 microTask로 Job queue에 추가됩니다. 또한 이것은 우선순위를 가지고 가장 event queue의 작업들 보다 빠르게 실행됩니다. 

  이 개념을 가지고 다시 한 번 위의 코드를 분석해 보겠습니다. 

    setTimeout(_ => console.log('timer1'), 0);
    
    Promise.resolve('promise1').then(result => console.log(result));
    
    setTimeout(_ => console.log('timer2'), 0);
    
    Promise.resolve('promise2').then(result => console.log(result));
    
    // 출력
    // promise1
    // promise2
    // timer1
    // timer2

- 코드 실행 절차

1. 순서대로 setTimeout, promise가 교차적으로 실행 되며 콜 스택에 쌓이고 pop over됩니다.
2. setTimeout의 콜백 함수는 일반 Task로 event queue에 추가됩니다.
    - 비동기 작업 실행 전까지 event queue에는 총 두 개의 Task가 추가됩니다.
3. Promise의 콜백 함수는 microTask로 Job queue에 추가됩니다. 
    - 비동기 작업 실행 전까지 Job queue에는 총 두 개의 microTask가 추가됩니다.
4. 동기 적인 코드의 실행이 종료되면 Job queue의 micro task가 바로 실행됩니다. 
5. Job queue와 콜 스택이 비워진 후 event queue의 Task들이 이벤트 루프를 통해서 실행됩니다. 

## 6.2 Promise Status

  프로미스는 세가지 단계가 있습니다. 

- Pending: 비동기 처리가 아직 완료되지 않은 상태를 의미합니다.
- resolve: 비동기 처리 결과 정상적으로 나타난 상태입니다.
- reject: 비동기 처리가 실패 하였을 때의 상태입니다.

## 6.2.1 Pending(대기)

    new Promise((reject, resolve) => {})

  pending(대기) 상태입니다. 처음 promise가 생성될 때이며 비동기 처리가 일어나지 않은 상태입니다. 

## 6.2.2 Resolve(성공)

    let promise = new Promise( (resolve, reject) => resolve('success'));
    
    promise.then(result => console.log(result));
    
    // success

  Pending 상태의 promise 내부 콜백 함수의 resolve 함수가 정상 작동하면 promise는 fulfilled 상태가 됩니다. 또한 then을 통해 이 결과를 확인할 수 있는데요. promise 내부에서 resolve의 인자가 then의 첫번째 콜백 함수의 인자로 실행됩니다. 

## 6.2.3 Reject

    let promise = new Promise( (resolve, reject) => reject('err'));
    
    promise.then(result => console.log(result), err => console.log(err));
    
    promise.then(result => console.log(result))
    	   .catch(err => console.log(err));
    

  Pending 상태의 promise 내부 콜백 함수의 reject 함수가 작동하면 promise는 rejected 상태가 됩니다. resolve에서는 결과 값을 then의 첫번째 콜백 함수를 이 결과를 확인할 수 있었는데요. rejected 상태의 반환은 then의 두번째 콜백 함수의 인자가 됩니다. 또한 catch를 통하여 error 결과를 확인할 수 있습니다.

## 6.2.4 Resolve, Reject 즉시 생성

    let promise = Promise.resolve("done");
    let promise = Promise.reject("fail");

  Promise는 resolve와 reject를 즉시 생성할 수도 있습니다. Promise.resolve(), Promise.reject()를 통해 프로미스의 상태를 조정할 수 있습니다. 

## **6.3 프로미스는 항상 비동기입니다.**

  콜백 함수의 비동기 처리에 대한 예제를 통해 설명하겠습니다. 

    const getName = () => {
    	callback();
    };
    
    getName(() => console.log(name));
    
    let name = 'B-SENS';

  콜백 함수의 첫번째 예제입니다. 자바스크립트 엔진은 맨 첫 줄에서부터 한 라인 씩 작업을 처리합니다. 그렇기 때문에 선언된 곳보다 먼저 불려진 name에 의한 에러가 발생했습니다. 기존에 이 문제를 해결하기 위해 setTimeout을 이용한 비동기 방식을 사용하였습니다.  

    function asyncFunc() {
    	return Promise.resolve();
    }
    
    asyncFunc().then(_ => console.log(name));
    let name = "B-SENS";

  이 코드에 대한 결과를 어떻게 예측하시나요?

  원하는 대로 name에 할당된 'B-SENS'가 출력됩니다. 프로미스를 통해 만들어진 코드의 결과는 항상 비동기로 출력됩니다. 

## 6.4 **then 체이닝**

    const resolveProm = Promise.resolve("success");
    
    resolveProm
    	.then(result => {
    		console.log(result);
    		return "value";
    	})
    	.then(val => console.log(val));
    
    // 'success'
    // 'value'

  콜백으로 비동기를 작성하였을 때의 큰 단점은 callback hell이 생성될 수 있다는 점이었습니다. 이 문제는 promise에서 then chaning을 통해 완화되었습니다. 

    const resolveProm = Promise.resolve("success");
    
    resolveProm
    	.then(result => {
    		console.log(result);
    	})
    	.then(val => console.log(val));
    
    // 'success'
    // 'undefined'

  이전 then의 반환값이 없다면 체이닝 된 then의 콜백 함수 인자는 undefined이 됩니다. 

    const resolveProm = Promise.resolve("success");
    
    resolveProm.then(val => {
    	console.log(val);
    	return "value";
    });
    
    resolveProm.then(val => console.log(val));
    
    // 'success'
    // 'success'

  then의 인자와 반환 값은 모두 독립적 입니다. 위의 코드에서 첫번째 then에서 새로운 값을 반환 하였는데도 두번째 then의 인자에 영향을 끼치지 못하였습니다. promise의 값이 다음 실행 함수에 영향을 주려면 체이닝을 사용해야 합니다. 

    Promise.resolve("success")
    	.then(val => {
    		console.log(val);
    		return new Promise(resolve => {
    			setTimeout(() => resolve("value"), 1000);
    		});
    	})
    	.then(val => console.log(val));

  위 코드의 동작 결과는 어떻게 될까요?

  먼저 "success"가 출력되고 1초 뒤에 "value"가 출력됩니다. 또 다른 비동기 함수를 반환할 수도 있습니다. 

## 6.5 **Finally**

    Promise.resolve("success")
    	.then(result => {
    		throw new Error("fail");
    	})
    	.then(result => console.log(result))
    	.catch(err => console.error(err))
    	.finally(_ => console.log("finally!"));
    
    // Error: fail
    // finally!

  promise가 성공 하였는지 실패 하였는지에 관계없이 무조건 한 번은 실행이 되는 메서드입니다. 위의 코드 실행 시 에러가 출력되는데요. 그 이루에 finally를 통해 실행된 콜백 함수도 실행되어 'finally'도 실행되었습니다. 성공, 실패 여부에 관계 없이 프로미스가 처리된 이후 처리해야 할 작업이 있을때 유용하게 사용할 수 있습니다. 

## 6.6 **Promise.all(iterable)**

  iterable 인자들이 모두 이행될 때나 비어 있을 때, Array로 promise 값을 반환합니다. 만약 promise의 결과가 실패한다면 가장 먼저 실패한 인자의 promise가 반환됩니다. 

    let test1 = Promise.resolve(3);
    let test2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve("foo");
      }, 100);
    });
    
    Promise.all([test1, test2]).then(values => {
      console.log(values);
    }, err => console.log(err));
    
    // (2) [3, "foo"]

  Promise.all의 모든 인자가 이행된다면 각 인자들의 promise 결과가 배열로 반환되어 then의 콜백의 첫번째 인자로 실행됩니다. 

    let test1 = Promise.resolve(3);
    let test2 = new Promise((resolve, reject) => {
      setTimeout(() => {
        resolve("foo");
      }, 100);
    }); 
    
    Promise.all([test1, test2, Promise.reject('error')]).then(values => { 
      console.log(values); 
    }, err => console.log(err));
    
    // error

  Promise.all의 인자 중 하나라도 실패하는 경우가 있다면 reject로 처리됩니다. 또한 다수의 인자의 promise 결과가 reject라면 가장 먼저 실패한 error가 then의 두번째 콜백이나 catch에 의해 반환됩니다. 

## **6.7 Promise.race(iterable)**

  iterable의 Promise 결과값 중 가장 먼저 반환된 결과가 반환됩니다. 결과가 resolve, reject 중 무엇인지는 상관없습니다. 

    let test = new Promise(resolve => setTimeout(resolve, 2000, "foo"));
    let test2 = new Promise(resolve => setTimeout(resolve, 1000, "bar"));
    
    
    Promise.race([test, test2]).then(value => {
    	console.log("first promise result : ", value);
    });
    
    // first promise result :  bar

  setTimeout의 타이머에 따라서 test2의 결과가 출력됩니다. Promise.race는 먼저 실행된 결과만을 반환하기 때문에 test는 반환되지 않습니다. 

    let test = new Promise(resolve => setTimeout(resolve, 2000, "foo"));
    let test2 = new Promise((resolve, reject) => setTimeout(reject, 1000, "error"));
    
    
    Promise.race([test, test2]).then(value => {
    	console.log("first promise result : ", value);
    }).catch(err => console.log("first promise result : ", err));
    
    // first promise result :  error

  위의 코드에서도 test2의 결과만 출력됩니다. Promise의 결과가 실패라고 하더라도 그 에러를 처리하는 함수만 실행되게 됩니다.
