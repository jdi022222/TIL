# Regular Expressions

<figure><img src="../../.gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

##

## 🔍 정규식(Regular expressions)

**특정 검색 패턴**(ASCII 또는 유니코드 문자의 시퀀스)에 대한 하나 이상의 일치 항목을 검색하여 **텍스트에서 정보를 추출**하는데 유용한 기술



### 1. Quantifier

<table><thead><tr><th width="190">Syntax</th><th width="251.33333333333331">Description</th><th>Example</th></tr></thead><tbody><tr><td>a<mark style="color:red;">|</mark>b</td><td>or</td><td>a, b</td></tr><tr><td>a<mark style="color:red;">?</mark></td><td>0 or 1</td><td></td></tr><tr><td>a<mark style="color:red;">*</mark></td><td>0 or more</td><td></td></tr><tr><td>a<mark style="color:red;">+</mark></td><td>1 or more</td><td></td></tr><tr><td>{N}</td><td>N번 반복</td><td>abc{3} -> abccc</td></tr><tr><td>{N, }</td><td>N번 이상 반복</td><td>abc{3,} -> abccc, abcccc</td></tr><tr><td>{N, M}</td><td>N번 이상 M번 이하 반복</td><td>abc{1,3} -> abc, abcc, abccc</td></tr><tr><td>(abc)*</td><td>abc의 0번 이상 반복</td><td>(abc)*  -> abc, abcabc</td></tr><tr><td>(abc){N,M}</td><td>abc의 N번 이상 M번 이하 반복</td><td>abc{1,2} -> abc, abcabc</td></tr></tbody></table>



### 2. General Token

<table><thead><tr><th width="191">Syntax</th><th width="249">Description</th><th>Example</th></tr></thead><tbody><tr><td>.</td><td>모든 글자</td><td></td></tr><tr><td>\w</td><td>대소문자 + 숫자 + _ </td><td></td></tr><tr><td>\W</td><td>not 대소문자 + 숫자 + _ </td><td></td></tr><tr><td>\d</td><td>숫자</td><td></td></tr><tr><td>\D</td><td>not 숫자</td><td></td></tr><tr><td>\t</td><td>탭</td><td></td></tr><tr><td>\n</td><td>엔터</td><td></td></tr><tr><td>\s</td><td>\t + \n</td><td></td></tr><tr><td>\b</td><td>경계 -> 알파벳 문자나 숫자와 공백 사이의 경계를 매치</td><td><mark style="color:red;">\b</mark>abc<mark style="color:red;">\b</mark> -> <mark style="color:red;">abc</mark> abctestabc <mark style="color:red;">abc</mark> <mark style="color:red;">abc</mark>.</td></tr><tr><td>\B</td><td>경계 -> 단어가 시작 또는 끝나는 경계가 아닌 경우를 매치</td><td><mark style="color:red;">\B</mark>abc<mark style="color:red;">\B</mark> -> test<mark style="color:red;">abc</mark>test</td></tr></tbody></table>



### 3. Group

<table><thead><tr><th width="194">Syntax</th><th width="247">Description</th><th>Example</th></tr></thead><tbody><tr><td>()</td><td>캡쳐 그룹</td><td></td></tr><tr><td>(?:)</td><td>캡쳐 그룹 무시</td><td></td></tr><tr><td>(?&#x3C;name>)</td><td>캡쳐 그룹에 이름 지정</td><td></td></tr><tr><td>\n</td><td>n번 째 캡쳐 그룹 선택</td><td></td></tr><tr><td>\n&#x3C;name></td><td>name이라는 캡쳐 그룹 선택</td><td></td></tr></tbody></table>



### 4. Pattern

<table><thead><tr><th width="200">Syntax</th><th width="238">Description</th><th>Example</th></tr></thead><tbody><tr><td>[]</td><td>패턴 매칭</td><td>[A-z0-9_] -> 대소문자, 숫자, _</td></tr><tr><td>[^abc]</td><td>패턴 제외</td><td>[^0-9] -> 숫자 제외</td></tr></tbody></table>





## 🚀 Java Practice

### 1. 모음 찾기

```java
String input = "Hello World!";

Pattern p = Pattern.compile("[AEIOUaeiou]");
Matcher matcher = p.matcher(input);

while (matcher.find()) { // boolean
    System.out.print(matcher.group() + " "); // value return
}

// e o o
```

* **`pattern`**과 **`matcher`** 이용



### 2. 이메일 찾아서 괄호 삽입

```java
String input = "Hello comibird! your email is comibird@test.com!";

String output = input.replaceAll("(\\w+@\\w+.\\w+)", "($1)");

System.out.println(output);

// Hello comibird! your email is (comibird@test.com)!
```

* **`$1`**를 통해 1번째 캡쳐 그룹을 조회, 괄호 삽입 후 대체



### 3. 동일한 글자 반복 제거

```java
String input = "Heellooo Wooooorld!";

String output = input.replaceAll("(.)\\1+", "$1");

System.out.println(output);		

//Helo World!
```







