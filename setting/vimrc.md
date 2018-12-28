```
set nocompatible " VI 오리지널과 호환하는 모드를 사용하지 않음(VIM확장)
set number " 라인번호를 붙임
set backspace=indent,eol,start " BS로 라인끝과 처음 자동들여쓰기한 부분을 지날수 있음
set tabstop=4 " 탭문자는 4컬럼 크기로 보여주기
set shiftwidth=4 " 문단이나 라인을 쉬프트할 때 4컬럼씩 하기
set autoindent " 자동 들여쓰기
set visualbell " Alert 음을 화면 깜박임으로 바꿔보여주기
set laststatus=2 " 최종상태 2개 기억하기
set statusline=%h%F%m%r%=[%l:%c(%p%%)] " 상태표시줄 포맷팅
syntax on " 적절히 Syntax에 따라 하일라이팅 해주기
set background=dark " 이건 터미널 모드에선 영향이 없다.
set paste
set cindent " C 언어 자동 들여쓰기
set showmatch       " 매치되는 괄호의 반대쪽을 보여줌
set autowrite       " :next 나 :make 같은 명령를 입력하면 자동으로 저장
set title           " 타이틀바에 현재 편집중인 파일을 표시
set fileencodings=utf-8,cp949
color desert " 컬러스킴
```
