---
title: 3. 이노시스 과제
layout: post
date: 2023-01-18
tag:
- 연습
- 게시판
projects: true
hidden: true # don't count this post in blog pagination
description: My ToyProject_errand 
category: projects
author: GiSeok
externalLink: false
---

## 과제 순서
---
> 1. [개발 환경 구성](#1-개발-환경-구성)
> 2. [화면정의서 참고 과제 시](#2-화면정의서-참고-과제-시작)
>       - [화면정의서](../assets/ppt/Inosys%20%26%20Seowoo%20%ED%99%94%EB%A9%B4%EC%A0%95%EC%9D%98%EC%84%9C.pptx)

## 1. 개발 환경 구성
---
1. JDK 1.8 설치
2. 이클립스 설치
3. Oracle 11g XE 설치
4. sql developer 설치
5. DB 실습용 테이블 생성 및 더미데이터 생성

## 2. 화면정의서 참고 과제 시작
---

### 1. 타입에 데이터 출력하기

``` jsx
 //el 태그의 변수명이 잘못되어 있었음 아래와 같이 수정 후 정상 출력
 ${dto.codeType } 
```

### 2. 01 > 자유, 02 > 익명, 03 > QnA 함수는 2가지  한가지는 mapper에 다른 한가지는 sqldeveloper에 적용해서 확인할 것.

``` sql
1. DECODE(컬럼명,바꿀데이터,바뀔데이터)
select 
    DECODE(CODE_TYPE, '01', '자유', '02', '익명','03', 'QnA') CODE_TYPE,
    NUM, NAME, TITLE, CONTENT, TO_CHAR(REGDATE , 'YYYY/MM/DD') AS   REGDATE 
FROM FREEBOARD

2. case when 조건 then 바꿀데이터
SELECT
    CASE
        WHEN code_type = '01' THEN
            '자유'
        WHEN code_type = '02' THEN
            '익명'
        WHEN code_type = '03' THEN
            'QnA'
    END                            AS code_type,
    num,
    name,
    title,
    content,
    to_char(regdate, 'YYYY/MM/DD') AS regdate
FROM
    freeboard
```
### 3. freeBoardInsert의 글쓰기 버튼 완성

* Ajax 방식으로 변경
* 게시글의 값들 비어있을때 유효성 검증
* 글쓰기 시 try-catch 문으로 Exception 처리(catch일때도 sucess로 처리됨)

>VIEW

``` jsx
function formSubmit() {
    const $queryString = $("form[name=freeBoard]").serialize();
    const $name = $("input[name=name]");
    const $title = $("input[name=title]");
    const $content = $("input[name=content]");
    if ($name.val() == "") {
        alert("이름은 필수값 입니다.");
        $name.focus();
        return false;
    }
    if ($title.val() == "") {
        alert("제목은 필수값 입니다.");
        $title.focus();
        return false;
    }
    if ($content == "") {
        alert("내용은 필수값 입니다.");
        $content.focus();
        return false;
    }

    if (confirm("글을 등록하시겠습니까?")) {
        $.ajax({
            type : "POST",
            url : "./freeBoardInsertPro.ino",
            data : $queryString,
            contentType: "application/x-www-form-urlencoded; charset=UTF-8",
            success : function(res) {
                if (res.state == "sucess") {
                    console.log("등록 성공");
                    if (confirm("main으로 돌아가시겠습니까?")) {
                        location.href = "./main.ino";
                    } else {
                        location.href = "./freeBoardDetail.ino?num="
                                + res.newNum;
                    }
                } else {
                    alert("예외가 발생하였습니다. 다시 시도해주십시오.\n예외내용: "
                            + res.state);
                }
            },
            error : function(request, error) {
                alert("fail");
                // error 발생 이유를 알려준다.
                alert("code:" + request.status + "\n" + "message:"
                        + request.responseText + "\n" + "error:"
                        + error);
            }
        });
    } 
}
```
> Controller

``` java
@ResponseBody // Ajax
@RequestMapping("/freeBoardInsertPro.ino")
public Map<String, String> freeBoardInsertPro(HttpServletRequest request, FreeBoardDto dto){
    Map<String, String> map = new HashMap<String,String>();
    try {
        freeBoardService.freeBoardInsertPro(dto);
        // 글생성후 새 글번호 
        map.put("newNum", Integer.toString(freeBoardService.getNewNum()) ) ;
        map.put("state", "sucess");
        // 예외 처리 확인용
//			throw new Exception("Exception Insert!!");
        return map;
    } catch (Exception e) {
        System.out.println("예외 내용: "+e.getMessage());
        map.put("state", e.getMessage());
        return map;
    }
}
```

### 4. 디테일 창 완성

* Ajax 방식으로 변경
* Insert 참고하여 수정,삭제 기능 구현

``` java
@ResponseBody
@RequestMapping("/freeBoardModify.ino")
public String freeBoardModify(HttpServletRequest request, FreeBoardDto dto){
    try {
        // 수정 기능
        freeBoardService.freeBoardModify(dto);
//		throw new Exception("Exception Modify!!");
        return "sucess";
    } catch (Exception e) {
        System.out.println("예외 내용: "+e.getMessage());
        return e.getMessage();
    }
}

@ResponseBody
@RequestMapping("/freeBoardDelete.ino")
public String FreeBoardDelete(@RequestParam int num  ){
    try {
        // 삭제 기능 num 받아서 삭제 
        freeBoardService.FreeBoardDelete(num);	
        return "sucess";
    } catch (Exception e) {
        System.out.println("예외 내용: "+e.getMessage());
        return e.getMessage();
    }
}
```


### 5. 다중 삭제 구현
    
* DB에서 한번에 처리되도록
* 자바에서 for문 사용 금지

>View

``` jsx
$("#selectAll").click(function() { // 게시글 전체 체크박스 이벤트
    if ($("#selectAll").is(":checked")) {
        $(".selectOne").prop("checked", true);
    } else {
        $(".selectOne").prop("checked", false);
    }
});

function selectDelete() { // 게시글 다중 삭제 함수
    alert("삭제버튼 클릭");
    /* 삭제 버튼 클릭시 체크되어있는 요소의 num 값 컨트롤러로 보내줘야됨 */
    
    let chk_arr = [];
    
    $(".selectOne:checked").each(function() {
        var chk = $(this).val();
        chk_arr.push(chk);
    });
    
    if(chk_arr.length == 0){
        alert("삭제할 게시글을 선택해주세요.")
        return false;
    }

    $.ajax({
        type : "get",
        url : "./freeBoardsDelete.ino",
        traditional : true, // 배열 넘기기
        data : {
            num : chk_arr
        },
        success : function(res) {
            if (res == "sucess") {
                console.log("삭제 성공");
                location.href = "./main.ino";
            } else {
                alert("예외가 발생하였습니다. 다시 시도해주십시오.\n예외내용: " + res);
            }
        },
        error : function(request, error) {
            alert("fail");
            // error 발생 이유를 알려준다.
            alert("code:" + request.status + "\n" + "message:"
                    + request.responseText + "\n" + "error:" + error);
        }
    });
}
```

>Controller

``` java
@ResponseBody
@RequestMapping("/freeBoardsDelete.ino")
public String FreeBoardsDelete(@RequestParam(value="num") List<Integer> num  ){
    try {
        // 삭제 기능 num 받아서 삭제 
        freeBoardService.FreeBoardsDelete(num);	
        return "sucess";
    } catch (Exception e) {
        System.out.println("예외 내용: "+e.getMessage());
        return e.getMessage();
    }
}
```
>SQL

``` xml
<update id="freeBoardsDelete" parameterType="list">
    DELETE FROM FREEBOARD
    WHERE NUM
    IN 
    <foreach collection="list" item="num" index="i" separator=","  open="(" close=")">#{num}</foreach>
</update>
```


### 6. 검색 기능 구현

* 전체, 타입, 번호, 내용, 제목, 이름, 기간 
* 유효성 검증
* 모두 Input tag로

> view

``` html
    <div style="width: 650px; margin-left: 130px;">
		<form id="searchForm" onsubmit="return false;">
			<select id="searchType" name="searchType">
				<option value="0">전체</option>
				<option value="1">타입</option>
				<option value="2">번호</option>
				<option value="3">내용</option>
				<option value="4">제목</option>
				<option value="5">이름</option>
				<option value="6">기간</option>
			</select> 
			<input type="button" id="searchBtn" value="검색" onclick="search()" />
		</form>
	</div>
```

> JavaScript

``` jsx
function search() { // 검색 함수
    console.log("======== 검색 ========");

    const $searchInput = $(".searchInput");
    const $queryString = $("#searchForm").serialize();
    const $searchKeyword = $("input[name=searchKeyword]").val();
    const $startDateValue = $("#startDate").val();
    const $endDateValue = $("#endDate").val();
    let dateRegex = /^\d{4}(0[1-9]|1[012])(0[1-9]|[12][0-9]|3[01])$/;
    let numberRegex = /[^0-9]/gi;
    
    console.log( $queryString);
    // 유효성 검증
    switch (Number($searchType)) {
    case 0:
        break;
    case 2:
        if($searchInput.val().trim() == ""){
            alert("검색어를 입력해주세요");
            return false;
        }
        if(numberRegex.test($searchInput.val())) {
            alert("숫자만 입력가능합니다.");
            $searchInput.val($searchInput.val().replace(numberRegex,''));
            return false;
        }
        break; 
    case 6:
        if(!dateRegex.test($startDateValue)
                    || !dateRegex.test($endDateValue)){
            $("#startDate").val($startDateValue.replace(numberRegex,''));
            $("#endDate").val($endDateValue.replace(numberRegex,''));
            alert("날짜를 YYYYMMDD 형식으로 입력해주세요.");
            
            if($startDateValue =="" && $startDateValue == ""){
                alert("날짜를 입력해주세요");
                return false;
            } 
            if($startDateValue ==""){
                alert("시작일을 입력해주세요");
                return false;
            }
            if(getToday() < Number($startDateValue)){ // 시작일이 오늘보다 미래일때
                alert("오늘 이후의 날짜는 조회가 불가능합니다.");
                return false;
            } 
            if($endDateValue ==""){
                alert("종료일을 입력해주세요");
                return false;
            }	
            return false;
        } else {
            if(getToday() < Number($startDateValue)){ // 시작일이 오늘보다 미래일때
                alert("오늘 이후의 날짜는 조회가 불가능합니다.");
                return false;
            } 
            if(Number($startDateValue) > Number($endDateValue)){ // 시작일이 종료일보다 미래일때 9 > 6
                alert("해당 기간의 조회가 불가능합니다.");
                return false;
            } 
        }
        break;
    default:
        if($searchInput.val().trim() == ""){
            alert("검색어를 입력해주세요.");
            return false;
        }
        break;
        }
    
    $.ajax({
        url : './freeBoardSearch.ino',
        type : 'GET',
        data : $queryString,
        contentType : "application/x-www-form-urlencoded; charset=UTF-8",
        dataType : 'json',
        success : function(res) {
            if(res.state=="sucess"){
                console.log(res);
                
                $("#tb").children().html("");
                const list = res.list;
                let str = "";
                
                for(let i=0; i< list.length; i++) {
                    str +=`
                        <tr>
                            <td align="center"><input type="checkbox" class="selectOne"
                                value="`+ list[i].num +`" /></td>
                            <td style="width: 55px; padding-left: 30px;" align="center">
                                `+list[i].codeType+`</td>
                            <td style="width: 50px; padding-left: 10px;" align="center">`+list[i].num +`</td>
                            <td style="width: 125px;" " align="center"><a
                                href="./freeBoardDetail.ino?num=`+list[i].num +`">`+ list[i].title +`</a></td>
                            <td style="width: 48px; padding-left: 50px;" align="center">`+ list[i].name +`</td>
                            <td style="width: 100px; padding-left: 95px;" align="center">`+ list[i].regdate+`</td>
                        <tr>
                    `;
                    $("#tb").html(str);
                }
                
                let pageStr = "";
                for (var i = res.startPage; i <= res.endPage; i++) {
                    pageStr += `
                        <li style="float: left;"><a href="javascript:void(0);"
                        onclick="pagination(this);">`+i+`</a></li>
                    `
                }
                console.log(pageStr);
                
                $("#pagination").html(pageStr);
                
                
                
            } else {
                alert(res.state);
            }
        },
        error : function(request, error) {
            alert("fail");
            // error 발생 이유를 알려준다.
            alert("code:" + request.status + "\n" + "message:"
                    + request.responseText + "\n" + "error:"
                    + error);
        }
    });
}
```

> Controller

``` java
@ResponseBody
@RequestMapping(value = "/freeBoardSearch.ino", produces = "application/json; charset=utf-8")
public Map<String, Object> FreeBoardSearch(@RequestParam Map<String, Object> map) {
    System.out.println(map);
    
    // 페이징 page , totalCount
    Paging pagination = new Paging(1, freeBoardService.freeBoardGetTotalCount(map));
            
    map.putAll(pagination.getPagination());
    System.out.println(map);
    // 페이징
    
    List<FreeBoardDto> list = freeBoardService.freeBoardList(map);
    if(list.isEmpty()) {
        map.put("state", "게시글이 없습니다.");
        return map;
    }
    map.put("list", list);
    map.put("state", "sucess");
    return map;
}
```

> SQL
``` sql
<select id="freeBoardGetList" resultType="freeBoardDto" parameterType="hashmap" > 
    select ROWNUM as RNUM, A.* from (
    SELECT  DECODE(CODE_TYPE, '01', '자유', '02', '익명',
        '03', 'QnA') CODE_TYPE, NUM, NAME, TITLE, CONTENT, TO_CHAR(REGDATE , 'YYYY/MM/DD') AS REGDATE FROM FREEBOARD
        <if test="searchType == 1 ">
        WHERE CODE_TYPE LIKE #{codeType} 
        </if>
        <if test="searchType == 2 ">
        WHERE NUM = #{searchKeyword} 
        </if>
        <if test="searchType == 3 ">
        WHERE CONTENT LIKE '%' || #{searchKeyword} || '%' 
        </if>
        <if test="searchType == 4 ">
        WHERE TITLE LIKE '%' || #{searchKeyword} || '%' 
        </if>
        <if test="searchType == 5 ">
        WHERE NAME LIKE '%' || #{searchKeyword} || '%' 
        </if>
        <if test="searchType == 6 " >
        WHERE TO_CHAR(REGDATE, 'YYYYMMDD') BETWEEN #{startDate} AND #{endDate}
        </if>
        ORDER BY NUM DESC
    ) A WHERE rownum between #{startCount} and #{endCount}  
</select>
```

### 7. 페이징 구현

* Ajax 방식으로 구현
* 이전 다음 버튼 X
* 현재 게시글 기준 앞뒤로 리프레시

> view

``` html
<ul id="pagination" style="list-style: none;">
    <c:forEach begin="${pagination.startPage }" end="${pagination.endPage }" var="i" varStatus="status">
        <!-- page 시작번호 부터 끝번호 만큼 페이지 번호의 숫자 생성  -->
        <c:choose>
            <c:when test="${page.page == status.count }">
                <!-- 선택된 페이지 -->
                <li style="float: left;"><a href="javascript:void(0);"
                    onclick="pagination(this);">${i}</a></li>
            </c:when>
            <c:otherwise>
                <li style="float: left;"><a href="javascript:void(0);"
                    onclick="pagination(this);">${i}</a></li>
            </c:otherwise>
        </c:choose>
    </c:forEach>
</ul>
```

> Controller

검색기능 참고 

> SQL

``` sql
<select id="freeBoardGetTotalCount" resultType="int"  parameterType="hashmap">
    SELECT COUNT(*) AS totalCount FROM FREEBOARD
        <if test="searchType == 1 ">
            WHERE CODE_TYPE LIKE #{codeType}
        </if>
        <if test="searchType == 2 ">
            WHERE NUM = #{searchKeyword}
        </if>
        <if test="searchType == 3 ">
            WHERE CONTENT LIKE '%' || #{searchKeyword} || '%'
        </if>
        <if test="searchType == 4 ">
            WHERE TITLE LIKE '%' || #{searchKeyword} || '%'
        </if>
        <if test="searchType == 5 ">
            WHERE NAME LIKE '%' || #{searchKeyword} || '%'
        </if>
        <if test="searchType == 6 ">
            WHERE TO_CHAR(REGDATE, 'YYYYMMDD') BETWEEN #{startDate} AND #{endDate}
        </if>
</select>
```


