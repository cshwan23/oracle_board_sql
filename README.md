# oracle_board_sql
oracle 게시판 설계 및 sql 구문




    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- 게시판 분석과 SQL 구문
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.151 게시판이란?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- 글쓴이의 주장, 의견 등의 의사표현을 웹상에서 구현한 것을 말한다.
    -- 예>
    --              [게시판 글 목록]           [6개 검색] [새글쓰기]
    --++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    --  번호              제목             이름         작성일       조회수
    --++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
    --   6      청년 실직 문제는 대기업 문제   설현       2011.12.06      0
    ----------------------------------------------------------------
    --   5      경제 침체의 원인            사오정      2011.12.01      2
    ----------------------------------------------------------------
    --   4          ㄴ 안보문제            하하       2011.12.03      1
    ----------------------------------------------------------------
    --   3            ㄴ 북한 핵 문제       전지현      2011.12.05     0
    ----------------------------------------------------------------
    --   2          ㄴ 나라 빚 관리 부실     저팔계      2011.12.02      1
    ----------------------------------------------------------------
    --   1            ㄴ 엔화 하락 문제     김태희      2011.12.04      0
    ----------------------------------------------------------------


    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.152 게시판 관련 테이블을 만들면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm

    create table board (

         b_no           number                              --글번호
        ,subject        varchar2(50)    not null            --글 제목
        ,wirter         varchar2(30)    not null            --글쓴이명
        ,reg_date       date            default sysdate     --등록일
        ,readcount      number(5)       default 0           --조회수
        -----------------------------------------------------------------------------
        ,content        varchar2(2000)  not null            --글 내용
        ,pwd            varchar2(12)    not null            --암호
        ,email          varchar2(30)                        --이메일
        -----------------------------------------------------------------------------
        ,group_no       number(5)       not null            --입력 글의 소속 그룹번호
        ,print_no       number(5)       not null            --같은 그룹 내에 출력 순서 번호
        ,print_level    number(5)       not null            --같은 그룹 내에 댓글 레벨 단계
        -----------------------------------------------------------------------------
        ,primary key (b_no)
    );

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.153 게시판에 처음으로 사오정이 아래와 같은 글을 올린다면 이때 실행되는 SQL구문 모두 쓰면?
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm

    --제목 : 경제 침체의 원인
    --이름 : 사오정
    --내용 : 어쩌구 저쩌구
    --암호 : 1234
    --이메일 : a@naver.com

    --insert into board (???) values (???)
    -------------------------------------------------------------
    insert into board (b_no, subject, wirter, content, pwd, email, group_no, print_no, print_level)
    values (

        (select nvl(max(b_no),0)+1 from board)       --글번호
        ,'경제 침체의 원인'                              --글제목
        ,'사오정'                                      --글쓴이
        -------------------------------------------------
        ,'어쩌구 저쩌구'--글내용
        ,'1234'--암호
        ,'a@naver.com'--이메일
        -------------------------------------------------
        ,(select nvl(max(b_no),0)+1 from board)     --입력 글의 소속 그룹번호
        ,0                                          --같은 그룹 내에 출력 순서번호
        ,0                                          --같은 그룹 내에 댓글 레벨단계 번호

    );

    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm
    -- Q.154 저팔계가 사오정의 댓글로 아래와 같은 글을 올린다면 이때 실행되는 SQL구문 모두 쓰면?
    --       <단, 사오정의 PK,즉 글번호가 1이다>
    --mmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmmm

    --제목 : 나라 빚 관리 부실
    --이름 : 저팔계
    --내용 : 어쩌구 저쩌구 내용물
    --암호 : 2345
    --이메일 : b@naver.com
    --------------------------------------------------------------
    --1. 엄마 조회수 1 증가
    --------------------------------------------------------------
    update board set readcount = readcount +1 where b_no=1;
    --------------------------------------------------------------
    --2. 엄마의 후손 글들의 출력번호를 1씩 증가시키기
    --   그래야 내가 엄마 바로 밑으로 들어갈 수 있다.
    --------------------------------------------------------------
    update board set readcount = readcount +1 where b_no=1;
    --------------------------------------------------------------
    --3.
    --------------------------------------------------------------
    insert into board (b_no, subject, wirter, content, pwd, email, group_no, print_no, print_level)
        values(
               (select nvl(max(b_no),0)+1 from board)
               ,'나라 빚 관리 부실'
               ,'저팔계'
               ---------------------
               ,'어쩌구 저쩌구 내용물'
               ,'2345'
               ,'b@naver.com'
               ---------------------
               ,(select group_no from board where b_no=1)       --엄마글의 그룹 번호를 검색하는 select 가 들어가는 쿼리
               ,(select print_no +1 from board where b_no=1)    --엄마글의 출력순서 번호를 검색한 후 1 더하는 select 가 들어가는 쿼리
               ,(select print_level +1 from board where b_no=1) --엄마글의 댓글레벨 단계를 검색한 후 1 더하는 select 가 들어가는 쿼리

              )
