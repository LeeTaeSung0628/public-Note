# 🚞 Java 대용량 Excel 리펙토링

#프로젝트 #개발 #JAVA #Excel #엑셀 #성능개선 #리펙토링

---
<br>

# <u><font color="#76923c">개요</font></u>

<br>

- 기존 DataTables기반의 Excel export 기능의 성능 부진으로 인한 리펙토링
- `클라이언트 → 서버사이드` 로의 로직 변경

>[!note] DataTables 기반의 기존 처리
>** DataTables란? <u>HTML 테이블을 동적으로 처리하기 위한 jQuery 플러그인이다.</u>
>
> DataTables의 기능 중 Export 기능이 있는데, 클립보드 복사와 인쇄 등 의 기능도 지원한다.
><u>기존의 Excel출력 기능이 이 DataTables의 Export 기능으로 만들어져 있었다.</u>

---

<br>

# <u><font color="#76923c">목적</font></u>

<br>
DataTables는 <u>클라이언트 측에서 브라우저 메모리</u>를 사용하여 엑셀 파일을 생성한다.
웹 HTML기반의 `DataTables Buttons`를 사용하여 데이터와 스타일을 엑셀 파일로 변환한다.


## 기존로직의 한계

1. **브라우저 메모리 한계**
	- 브라우저는 서버보다 메모리와 CPU 성능이 낮다.
	- 대용량 데이터를 처리할 때 브라우저가 멈추거나 충돌할 가능성이 크다.
    - DataTables는 **전체 데이터를 메모리에 적재**한 후 엑셀로 변환한다.
    - Ajax로 부분 데이터를 불러오는 서버사이드 모드에서는 **현재 페이지 데이터**만 엑셀로 변환됩니다.
    - 모든 데이터를 한꺼번에 가져와 처리하면 처리 한계에 도달할 가능성이 더 커진다.

<u>ex) 10만 개 이상의 행을 엑셀로 내보내면 브라우저의 메모리 한계를 초과하여 강제로 종료되는 경우가 많다.</u>

<br>

2. **파일 변환 속도**
    - JavaScript 기반으로 파일을 생성하는 데 시간이 많이 소요된다.
    - 데이터 변환과 파일 생성이 모두 **싱글 스레드**로 이루어져 병렬 처리의 이점을 활용하지 못한다.

<br>

3. **스타일 커스터마이징의 복잡성**
    - 엑셀 스타일을 커스터마이징하는 과정이 복잡하며, XML 직접 수정 방식은 성능 저하를 초래한다.
    - 파일의 구조와 스타일을 모두 제어하려면 JavaScript 메모리 부담이 더욱 커진다.

<br>

>[!warning] 정리
> 소규모 데이터의 Export에는 간단한 설정으로 빠른 구현이 가능하나.
> 10만개 이상의 대용량 데이터에서는 한계가 명확하다.

---

## 기존 Excel 내보내기 (Client / JS)

```js
...

buttons: [
    {
        extend: 'copyHtml5',
        name: 'Copy',
        text: 'Copy',
        exportOptions: {
            columns: [0, ':visible']
        }
    },
    {
        extend: 'excel',
        name: 'Excel',
        text: 'Excel',
        filename: '엑셀출력_' + moment().format('YYYYMMDDhhmm'),
        title: '',
        action: serverSideButtonAction,
        customize: function(xlsx) {
            var sSh = xlsx.xl['styles.xml'];
            var lastXfIndex = $('cellXfs xf', sSh).length - 1;

            var sheet = xlsx.xl.worksheets['sheet1.xml'];

			// 스타일 적용
            var n1 = '<xf numFmtId="0" fontId="2" fillId="2" borderId="1" applyFont="1" applyFill="1" applyBorder="1" xfId="0" applyAlignment="1">' +
                     '<alignment horizontal="center"/></xf>';
            var n2 = '<xf numFmtId="0" fontId="0" fillId="0" borderId="1" applyFont="0" applyFill="0" applyBorder="0" xfId="0" applyAlignment="0">' +
                     '<alignment horizontal="right"/></xf>';

            sSh.childNodes[0].childNodes[5].innerHTML += n1 + n2;

            var greyBoldCentered = lastXfIndex + 1;
            var value = lastXfIndex + 2;

            $('c', sheet).attr('s', value);
            $('row:first c', sheet).attr('s', greyBoldCentered);
        }
    },
    {
        extend: 'print',
        name: 'Print',
        text: 'Print',
        action: serverSideButtonAction
    },
    'colvis'
]

```

---

<br>

## 서버사이드의 장점

<br>

1. **서버의 자원 활용**
    - 서버의 메모리와 CPU는 클라이언트보다 월등히 높아 대량 데이터 처리에 유리하다.
    - 서버가 Excel 파일을 직접 생성하여 브라우저에 전송하므로 클라이언트의 부담이 줄어든다.

<br>

2. **대용량 데이터 처리**
    - 수십만 건 이상의 데이터를 메모리 효율적으로 처리 가능
    - <font color="#76923c">Apache POI</font>와 같은 라이브러리는 **스트리밍 방식**으로 데이터를 파일에 직접 기록하여 메모리 과부하를 방지한다.
    - `SXSSFWorkbook`를 사용하여 **매우 큰 데이터**를 처리할 수 있습니다.

>[!tip] `SXSSFWorkbook`란?
>
> <u>Apache POI</u> 라이브러리에서 제공하는 <u>대용량 Excel 파일 생성용 클래스</u>
> `SXSSFWorkbook`은 메모리 절약을 위해 **디스크 기반 스트리밍 방식**을 사용하여 메모리에 모든 데이터를 올리지 않고, 필요한 부분만 메모리에 유지한다.

<br>

3. **병렬 처리**
    - 멀티스레딩을 통해 데이터 수집과 파일 생성을 병렬로 수행할 수 있다.
    - 서버 자원을 최대로 활용하여 성능을 극대화할 수 있다.

<br>

4. **직접 파일 다운로드**
    - 엑셀 파일을 서버에서 생성하고, URL을 통해 클라이언트가 다운로드 받도록 처리하여 **브라우저 부담 최소화**.
    - 응답을 **스트리밍 방식**으로 처리하여 중간에 데이터가 소실되지 않도록 보장한다.

---
