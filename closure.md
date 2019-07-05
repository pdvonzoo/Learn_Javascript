# Day2

## 0. Closure 정의

**'A closure is the combination of a function and the lexical environment within which that function was declared'**

클로저에 대한 MDN의 정의입니다. 

클로저는 선언되었을 때의 렉시컬 환경에 영향을 받습니다. 

따라서 클로저를 알기 위해선 **Lexical Scope**에 대한 이해가 필요합니다.

## 1. Lexical Scoping

    function showName() {
      const name = "Kim"; // name은 showName 함수 내부에 생성된 지역 변수입니다.
      function displayName() { // displayName() 은 내부 함수이며,클로저입니다.
        console.log(name); // 접근 가능한 외부 변수에 접근합니다.
      }
      displayName();
    }
    showName();

displayName 함수 내부에 name이라는 변수가 없지만 외부 함수의 변수인 name이 출력됩니다. displayName 함수는 외부 함수에 접근할 권한이 있기 때문에 이런 결과가 나타날 수 있었습니다. 단어 "Lexical"은 유효 범위를 지정할 때 변수가 사용 가능한 범위를 결정하기 위해 소스 코드 내에서 변수가 선언된 위치를 사용한다는 것을 의미한다. 결론적으로 중첩된 함수의 내부 함수들은 그들의 외부 유효 범위 내에서 선언된 변수들에 접근할 권한을 가진다.

**모든 함수는 Lexicla Scope를 가집니다. 이것은 그 함수가 가지고 있는 접근 범위를 말합니다. 함수는 내부의 함수에 접근할 수 없지만 그 함수를 감싸고 있는 외부 함수들의 환경에 접근할 수 있습니다.**

## 2. Lexical Scope VS Dynamic Scope

자바스크립트는 Lexical Scope 규칙을 따릅니다. 반대되는 개념으로 Dynamic Scope가 있는데요. 이 개념은 Perl같은 함수에서 적용됩니다. 각 개념이 어떻게 다른지 확인 해봅시다.

먼저 Lexical scope는 선언된 장소에 따라 유효한 접근 범위가 결정됩니다. 선언된 장소에 따라 결정된다는 것은 코드를 작성할 때 이미 그 환경이 결정된다는 것입니다. 이것은 자바스크립트의 중요한 특징인데요. 이러한 이유로 외부 함수의 지역 스코프에 접근할 수 있는 것입니다.

Dynamic scope는 조금 다릅니다. Lexical scope가 선언된 장소에서 외부로 접근할 수 있다면 Dynamic scope는 항상 호출된 위치를 확인해야 합니다. 이 부분은 코드로 설명하는게 더 나을것 같은데요. 아래의 예시 코드가 있습니다.

    const target = 1;
    
    function foo(){
    	const target = 2;
    	bar();
    }
    
    function bar(){
    	console.log(target);
    }
    foo();
    
    // Dynamic? 2
    // Lexical? 1

### 2-1 Lexical Scope의 경우

1. 함수 foo가 실행된다. 
2. foo의 지역 변수로 target의 값이 선언 및 할당된다.  
3. 함수 bar가 실행된다.
4. target 변수가 출력된다.
    - 렉시컬 스코프 규칙을 따르기 때문에 bar 함수를 감싸고 있는 전역의 target 변수가 출력된다.
5. 전역 환경에 존재하는 1이 출력된다.

### 2-2 Dynamic Scope의 경우

1. 함수 foo가 실행된다. 
2. foo의 지역 변수로 target의 값이 선언 및 할당된다.  
3. 함수 bar가 실행된다.
4. target 변수가 출력된다.
    - 다이나믹 스코프 규칙을 따르기 때문에 bar 함수가 호출된 foo 함수의 지역 변수인 target 변수가 출력된다.
5. foo에서 선언 및 할당된 target 변수의 값인 2가 출력된다.

## 3. Closure

Lexical Scope 규칙에 대한 이해를 바탕으로 우리는 Closure에 대해 이해할 수 있습니다. 먼저 이전 Lexical Scope를 설명했던 예제 코드를 다시 활용 해보겠습니다.

    function showName() {
      const name = "Kim"; // name은 showName 함수 내부에 생성된 지역 변수입니다.
      return function displayName() { // displayName() 은 내부 함수이며,클로저입니다.
        return name; // 접근 가능한 외부 변수에 접근합니다.
      }
      displayName();
    }
    
    const getName = showName();
    
    getName(); // "Kim"이 반환됩니다.
    

이전 예제와 다른 점은 showName함수가 **함수를 반환** 한다는 것입니다. showName 함수는 displayName() 함수를 반환합니다. 따라서 getName 변수엔 displayName 함수가 할당됩니다. 여기서 중요한 사실은 **getName 변수는 displayName 함수 뿐만 아니라 그 함수가 가지고 있는 Lexical Scope를 함께 가지고 있다는 것입니다.** 

그렇기 때문에 displayName이라는 함수가 할당된 getName 변수를 실행시키면 displayName의 Lexical Scope 규칙에 따라서 외부 함수인 showName 함수의 지역 스코프에 존재하는 name 변수를 반환하게 되는 것입니다. 

어떤 함수든 항상 클로저를 가지고 있습니다. 다시, 클로저는 함수와 함수가 선언된 언어적 환경의 조합입니다.

## 4. 비동기와 클로저

클로저는 늘 언어적 환경을 고려하며 사용해야 합니다. 언어적 환경을 고려하기에 동기적인 상황은 직관적이기 때문에 고려해야 할 점이 많지 않지만 비동기의 상황은 조금 다릅니다. 

    function getLoopValue(max){
    	for(var i=0; i<=max; i++){
    		function showValue(){
    			console.log(i);
    		}
    		showValue();
    	}
    }
    getLoopValue(3);
    // 0
    // 1
    // 2
    // 3

먼저 동기적인 흐름의 코드입니다. 루프 안의 상황에 대한 예시를 들었는데요. showValue 함수는 파라미터로 값을 주입하지 않고 내부 함수인 showValue 함수를 호출합니다. showValue 함수는 렉시컬 스코프 규칙에 따라 외부 렉시컬 환경의 i 값을 출력합니다. 따라서 원하는 결과를 얻을 수 있습니다. 

    function getLoopValue(max){
    	for(var i=0; i<=max; i++){
    		setTimeout(() => console.log(i), 0);
    	}
    }
    getLoopValue(3);
    //4
    //4
    //4
    //4

비동기 상황을 만들기 위해서 setTimeout 함수를 사용하였습니다. 결과적으로 4가 네 번 출력됩니다. 이건 우리가 원하는 결과가 아닙니다. 이런 결과가 나온 이유는 무엇일까요?

setTimeout 함수 내부의 콜백 함수는 동기식 코드가 모두 실행된 이후에 실행됩니다. 이 부분에 대해선 콜스택을 확인하면서 이해하는 게 좋은데요. 콜스택은 실행되는 함수들이 쌓이는 스택입니다. 이후에 본격적으로 비동기에 대한 글을 쓰면서 풀어내도록 하겠습니다. 

결과적으로 콜스택이 비워진 후에 setTimeout의 콜백 함수가 실행됩니다. 

동기적인 코드인 for loop가 완료된 이후의 i의 값은 4입니다. 그렇기 때문에 setTimeout의 콜백 함수는 그 값을 네번 찍어내는 것입니다. 

그렇다면 동기적인 흐름처럼 값을 받기 위해서는 어떻게 해야 할까요?

### 4-1 function scope 활용

    function getLoopValue(max){
      function getValue(value){
    		setTimeout(() => console.log(value), 0);
      }
    	for(var i=0; i<=max; i++){
    		getValue(i);
    	}
    }
    getLoopValue(3);
    //0
    //1
    //2
    //3

getValue 함수로 새로운 스코프를 만들었습니다. 이렇게 코드를 작성하게 되면 setTimeout의 콜백 함수는 함수의 인자인 value에 접근하게 됩니다. 따라서 value의 값이 클로저 스코프에 저장되어 원하는 값이 출력됩니다. 

### 4-2 block scope 활용

    function getLoopValue(max){
    	for(let i=0; i<=max; i++){
    		setTimeout(() => console.log(i), 0);
    	}
    }
    getLoopValue(3);
    //0
    //1
    //2
    //3

for loop 안의 변수 i를 var가 아닌 let으로 선언했습니다. 함수 내부에서 새로운 스코프를 생성한 것입니다. setTimeout의 콜백 함수는 block scope에 의해 생성된 i 값을 클로저로 담아둡니다. 따라서 이후 콜스택이 비워진 후 실행될 때 클로저로 인해서 저장된 i의 값이 호출됩니다.
