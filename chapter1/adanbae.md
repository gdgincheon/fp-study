[함수형 프로그래밍]

# Chapter 1. 함수형 길들이기

1. 함수형 사고방식
- 애플리케이션의 설계 요소
	- 확장성 : 추가 기능을 지원하기 위해 계속 코드를 리랙터링 해야 하는가?
	- 모듈화 용이성 : 파일 하나를 고치면 다른 파일도 영향을 받는가?
	- 재사용성 : 중복이 많은가?
	- 테스트성 : 함수를 단위 테스트하기 어려운가?
	- 헤아리기 쉬움 : 체계도 없고 따라가기 어려운 코드인가?
- 목표
	- 애플리케이션의 부수효과(side effect)를 방지하고 상태변이(mutation of state) 를 감소하기 위해 데이터의 제어 흐름과 연상을 추상(abstract) 하는 것
	- 무상태성(statelessness), 불변성(immutatbility)를 지향 -> 순수함수(pure function)로 구서된 불변 프로그램 구축을 전체로 함
- Code
	- 1단계 : 일반적인 
		~~~
		document.querySelector(‘#msg’).innerHTML = ‘<h1>Hello World</h1>’;
		~~~
	- 2단계 : 함수화 - 달라지는 부분만 매개변수로 —> 객체지향형 프로그래밍에서 일반적으로 생각하는 함수의 사용
		~~~
		function printMessage(elementId, format, message) {
			document.querySelector(`#${elementId}`).innerHTML = `<${format}>${message|</${format}>`;
		}
		printMessage(‘msg’, ‘h1’, ‘hello World’);
		~~~
	- 3단계 : 함수를 매개변수화(parameterize) —> 함수형 패러다임으로의 접근
		~~~
		var printMessage = run(addToDom(‘msg’), h1, echo);
		printMessage(‘Hello World’);
		
		// run 함수
		var run = function(f, g, h) {
			return function(x) {
				return f(g(h(x)));
			};
		};

		// run for X functions
		var run = (…functions) => x => {
			functions.reverse().forEach(func => x = func(x));
				return x;
		};
		~~~
	- 4단계 : 확장
		~~~
		var printMessage = run(console.log, repeat(2), h2, echo);
		printMessage(‘Get Functional’);
		~~~
- 기본개념
	- 선언적(declarative) 프로그래밍
		- 정의
			- 내부적으로 코드를 어떻게 구현했는지, 데이터는 어떻게 흘러가는지 밝히지 않은 채 연산/작업을 표현하는 사상
			- 프로그램의 서술부(description)와 평가부(evaluation)를 분리, 제어흐름이나 상태 변화를 특정하지 않고도 프로그램 로직이 무엇인지를 표현식(expression)으로 나타냄
				<-> 명령형(imperative), 절차적(procedural) : 자바, c#, c++ 등
			- 어떤 결과를 내기 위해 시스템의 상태를 변경하는 구문을 위에서 아래로 죽 늘어놓는 순차열(Sequence, 수열)임
		- Code
			- 명령형 코드
				~~~
				var array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9];
				for(let i = 0; i < array.length;  I++) {
					array[I] = Math.pow(array[I], 2);
				}
				array; //-> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
				~~~
				- 선언적 코드
				~~~
					[0, 1, 2, 3, 4, 5, 6, 7, 8, 9].map(
					function(num) {
						return Math.pow(num, 2);
					}
				);
				~~~
				- 선언적 코드 : 람다표현식(lambda expression), 화살표 함수(arrow function) 
				~~~
				[0, 1, 2, 3, 4, 5, 6, 7, 8, 9].map(num => Math.pow(num, 2));
				~~~
			- 루프 제거 이유
				- 루프는 재사용하기도 어렵거니와 다른 연산에 끼워넣기도 어려운 명령형 제어 구조물
		- 순수함수
			- 특성 정리
				- 주어진 입력에만 의존할 뿐, 평가 도중 또는 호출 간 변경될 수 있는 숨겨진 값이나 외부 상태와 무관하게 작동
				- 전역 객체나 레퍼런스로 전달된 매개변수를 수정하는 등 함수 스코프 밖에서 어떠한 변경도 일으키지 않습니다. 			b. Code
				- 불순(impure)한 함수
					~~~
					var counter = 0;		// 전역변수
					function increment() {
						return ++counter;
					}
					~~~
					> ! 자신의 스코프에 없는 외부 변수 Counter 를 읽고 수정하므로 불순 !
					> ! 부수효과 : 전역 레퍼런스가 변경됨 - 호출 도중에 counter 값은 언제라도 변할 수 있어 어떤값이 반환될지 알 수 없음
			- 부수효과
				- 발생 상황
					- 전역 범위에서 변수, 속성, 자료구조를 변경
					- 함수의 원래 인수 값을 변경
					- 사용자 입력을 처리
					- 예외를 일으킨 해당 함수가 붙잡지 않고(catch) 그대로 예외를 던짐(throw)
					- 화면 또는 로그 파일에 출력
					- HTML 문서, 브라우저 쿠키, DB에 질의
				- Code [1-3]
					~~~
					function showStudent(ssn) {
						let student = db.find(San);
						if(student !== null) {
							document.querySelector(`#${elementId}`).innerHTML = `${student.ssn|, ${student.firstname}, ${student.lastname}`;
						}
						else {
							throw new Error(‘학생을 찾을 수 없습니다!’);
						}
					}
					showStudent(‘1234-12-1234’);
					~~~
				- Code [1-3] 의 부수효과
					- 변수 db 는 외부변수, 실행중 null 참조, 호출 단계마다 상이한 값 가리켜 결괏값이 달라짐 -> 프로그램 무결성 깨짐
					- elementId 가 전역 변수
					- HTML 요소를 직접 수정 (HTML 문서(DOM)은 가변적인, 전역 공유자원)
					- 학생 레코드를 찾지 못해 예외를 던지면 전체 프로그램 스택이 풀리면서 종료
				- Code [1-3] 개선
					- 개선 1 : 하나의 목적을 가진 짧은 함수로 분리
					- 개선 2 : 함수에 인수를 모두 명시하여 부수효과 개수를 줄임
					~~~
					var find = curry((db, id) => {
						let obj = db.find(id);
						if(obj === null) {
							throw new Error(‘객체를 찾을 수 없습니다!’);
						}
						return obj;
					});
					var csv = student => ‘${student.ssn} , ${student.firstname}, ${student,lastname}`;
					var append = curry((selector, info) => {
						document.querySelector(selector).innerHTML = info;
					});
					~~~
				- 커링(currying) 기법
		- 참조 투명성 = 등식 정합성(equational correctness)
			- 순수함수 본연의 특징 : 함수가 일관된 반환값을 보장하도록 해서 전체 함수 결과를 예측 가능한 방향으로 유도하는 것
			- code
				~~~
				var increment = counter => counter + 1;
				~~~
				- 명령형
					~~~
					increment();
					increment();
					print(counter);
					~~~
				- 함수형
					~~~
					var plus2 = run(increment, increment)(;
					print(plus2(0);
					~~~
		- 불변성
			- 기본형(primitive type - 원시 자료형) : 불면
			- 배열 등의 객체는 불면이 아님 -> 부수효과 발생 가능성 
			- code
				~~~
				var sortDesc = arr => {
					arr.sort(
						(a, b) => b - a
					);
				};
				~~~
			- 문제점 : 원본 레퍼런스가 가리키는 배열의 원소를 정렬함!

2. 함수형 프로그래밍의 정의와 필요성
	- 정의 
		- 외부에서 관찰 가능한 부수효과가 제거된 불변 프로그램을 작성하기 위해 순수함수를 선언적으로 평가하는 것
	- 필요성
		- 함수를 순수 연산의 관점에서 데이터를 절대 변경하지 않는 고정된 작업단위(unit of work) 로 바라봄 => 잠재적인 버그는 줄어듬
	- 함수형 인지력을 향상 시키는 기법
		- 복잡한 작업을 분해
			분해(프로그램을 작은 조각들로 쪼갬) 와 합성(작은 조각들을 다시 합침)
			- showStudent -> find, csv, append
			- 단일성 : 함수는 저마다 한가지 목표만 바라봐야 한다는 사상
			- 합성(composition) 기법 
				- f  합성 g = f  ・ g = f(g(x))
					~~~
      				var showStudent = compose(append(‘#student-info’), csv, find(db));
					showStudent(‘123-33-1234’);
					~~~
		- 흐름 체인(fluent chain)으로 데이터를 처리함
			- Data
			~~~
			let enrollment = [
				{enrolled:2, grade: 100},
				{enrolled:2, grade: 80},
				{enrolled:1, grade: 89}
			];
			~~~
			- 명령형으로 처리
			~~~
			var totalGrades = 0;
			var totalStudentsFound = 0;
			for(let i = 0; i < enrollment.length ; I++) {
				let student = enrollment[I];
				if(student !== null) {
					if(student.enrolled > 1) {
						totalGrades += student.grade;
						totalStudentsFound++;
					}
				}
			};
			~~~
			var average = totalGrades / totalStudentsFound;
			- 함수 체인
				~~~
				_.chain(enrollment)
					.filter(student => student.enrolled > 1)
					.pluck(‘grade’)
					.average()
					.value();
				~~~
		- 리액티브 패러다임
			- 복잡한 비동기 애플리케이션에서도 신속하게 반응
			- Code 일반
				~~~
				var valid = false;
				var elem = document.querySelector(‘#student-ssn’);
				elem.onkeyup = function(event) {
					var val = elem.value;
					if(val !== null && val.length !== 0) {
						val = val.replace(/^\s*|\s*$|\-s/g, ‘’);
						if(val.length === 9) {
							console.log(`올바른 SSN : ${val}!`);
							valid = true;
						}
					}
					else {
						console.log(`잘못된 SSN : ${val}!`);
					}
				};
				~~~
			- Code 리액티브 패러다임 
				~~~
				Rx.Observable.fromEvent(document.querySelector(‘#student-ssn’), ‘keyup’)
					.plunk(’srcElement’, ‘value’)
					.map(ssn => ssn.replace(/^\s*|\s*$|\-s/g, ‘’)
					.filter(ssn => ssn !== null && ssn.length === 9)
					.subscribe(validSsn => {
						console.log(`올바른 SSN : ${val}!`);
					});
				~~~
			- 함수형 리액티브 프로그래밍(functional reactive programming)
3. 불변성, 순수함수 원리
4. 함수형 프로그래밍 기법 및 그것이 설계 전반에 미치는 영향

