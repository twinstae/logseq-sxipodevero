- 프로젝트 구조
	- 폴더 구조가 지나치게 복잡하진 않은지?
	- 예를 들어 컴포넌트 하나를 만들려면...
		- 폴더
			- style
			- 컴포넌트
			- index.ts
		- 파일 4개가 필요한데. 하나로 합쳐도 이상하지 않을듯함.
		- 특히 components에 style이 없는 경우에도 폴더로 만든 이유는 무엇인지? (common)
- 페이지 라우팅
	- QuizView와 ResultView 는 하나의 quiz 라는 루트로 되어 있는데. 하나의 url 루트에서 삼항연산자로 처리되어서 위화감이 있음.
	- 각 문제 step마다 별개의 route가 있어야 할듯.
	- 문제를 풀던 중에 뒤로가기를 누르면, 난이도 선택 메뉴로 돌아가서 위화감이 있음
	- 그래도 전역 상태에 풀이 기록이 남아 있어서, 풀던 곳부터 다시 시작할 수 있는 점은 좋은듯
- 상태 관리
	- SWR을 사용하고 있는데, 데이터를 관리하는 게 fetcher로만 사용하고 있어서, useEffect를 쓰는 것과 차이를 모르겠음.
	- 대부분의 로직을 setter로 노출하고 있는데...
		- 상태 로직이 변하면 유지보수하기 어려울 것 같음.
		- 보일러 플레이트가 많아짐
		- 각 컴포넌트에서 너무 많은 메서드를 가져오게 됨
		- 예를 들어 handleRestart나 handleReadMode 등은 상태 라이브러리에 메서드로 구현해서 공개하면 좋을듯
		-
	- 다음 로직은 quizzes[page]로 가져올 거면 전용 selector나 getter를 만들면 좋았을듯.
		- ```js
		  const currentQuiz = quizzes[page];
		  const answer = quizzes[page].correct_answer;
		  const selected = quizzes[page].selected_answer;
		  ```
		-