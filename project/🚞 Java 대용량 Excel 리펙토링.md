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
    }
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

<br>

# <u><font color="#76923c">엑셀 내보내기 서버 사이드 구현</font></u>

<br>

## `SimpleExcelFile`

- 엑셀 파일 처리 클래스
- 엑셀파일 생성, 데이터 추가, 스타일 적용 등
- **SXSSFWorkbook**방식으로 스트리밍 처리
	- 대용량 처리에 적합
	- 일정 개수 이상의 row를 디스크로 flush
	- <u>OutOfMemory</u>방지
```java
for (T t : data) {  
    renderBody(t, rowNum, bodyStyle, totalStyle, accumStyle);  
    if (rowNum % 10000 == 0 || rowNum == data.size()) {   // 10,000건 마다 flush
	    
	    try {  
            // 마지막 데이터의 경우, 남는 데이터 만큼만 flush, 아닌경우 10,000건씩 플러쉬  
            workbook.getSheet(sheetName).flushRows(rowNum == data.size() ? data.size() % 10000 : 10000);  
        } catch (IOException e) {  
            throw new BadRequestException(e.getMessage());  
        }  
    }  
    rowNum++;  
}
```
- Excel Sheet정보 파라미터로 받아서 초기화
- `SimpleExcelMetaDataFactory`를 이용하여 엑셀 메타데이터를 생성
- **전체적인 엑셀 다운로드까지의 모든 단계를 포함하고 실제 랜더링해서 셀을 생성하는것**


<br>

## `SimpleExcelMetaDataFactory`

- **싱글톤 객체**로 생성
- 엑셀로 출력할 DTO객체의 어노테이션을 파악해 메타데이터 정리
- `SimpleExcelMetadata`객체를 생성하기 위한 *기본 틀 제공*(헤더, 스타일, 필드 목록 등)
- <u>CellStyleMap을 사용하여 각 필드의 스타일을 미리 캐싱 ( 스타일 중복 방지 )</u>
```java
private void applyCellStyle(CellStyleMap cellStyleMap, ExcelColumnStyle fieldStyle, ExcelColumnStyle classDefaultStyle, String fieldName, CellPart part, Workbook workbook) {  
    /* dto 의 field 값에 스타일이 설정되어 있는지 체크 */    
    boolean styleCheck = fieldStyle.excelCellStyleClass() != NullStyle.class;  
  
    /* dto 의 field 에 스타일 존재 유무에 따라, ExcelCellKey 의 fieldName 지정 */    
    String fieldNameKey = styleCheck ? fieldName : "DEFAULT";  
  
    /* dto 의 field 에 스타일 존재 유무에 따라 스타일 설정 */    
    ExcelColumnStyle style = styleCheck ? fieldStyle : classDefaultStyle;  
  
    ExcelCellKey excelCellKey = ExcelCellKey.of(fieldNameKey, part);  
  
    /* 해당 키값과 같은 키값을 가진 데이터가 있는 경우 styleMap 에 추가하지 않음 */    
    if (!cellStyleMap.valueCheck(excelCellKey)) {  
        cellStyleMap.put(decideAppliedStyle(style, workbook),  
                excelCellKey,  
                workbook);  
    }  
}
```
- **스타일, 정보등 dto 어노테이션 필드들을 읽어와서 파악하고, 가공하여 SimpleExcelFile에서 사용하기 쉽게 만드는 역할**

<br>

## `CustomExcelDto`

- `@DefaultExcelHeaderStyle`: 엑셀 헤더에 기본 스타일 적용
    - 스타일: `HeaderStyle.class`
- `@DefaultExcelBodyStyle`: 엑셀 데이터 행에 기본 스타일 적용
    - 스타일: `BodyStyle.class`
- `@DefaultExcelTotalRow`: 합계 행에 기본 스타일 적용
```java
@DefaultExcelHeaderStyle(style = @ExcelColumnStyle(excelCellStyleClass = HeaderStyle.class))  
@DefaultExcelBodyStyle(style = @ExcelColumnStyle(excelCellStyleClass = BodyStyle.class))  
@DefaultExcelTotalRow(style = @ExcelColumnStyle(excelCellStyleClass = TotalRowStyle.class))  
public class PgDepositListExcelDto {  
    @ExcelColumn(headerName = "No")  
    private String rowNum;  
  
    @ExcelColumn(headerName = "회원번호")  
    private String mbNo;  
  
    @ExcelColumn(headerName = "아이디")  
    private String mbId;  
  
    @ExcelColumn(  
            headerName = "금액",  
            bodyStyle = @ExcelColumnStyle(excelCellStyleClass = AmountStyle.class),  
            totalRowStyle = @ExcelColumnStyle(excelCellStyleClass = TotalAmountStyle.class)  
    )  
    private long amt;
```
- **실제 객체와 맵핑될 excelDTO객체**

### 사용부

**SimpleExcelMetaDataFactory**에서 `@ExcelColumn` 어노테이션이 붙은 필드를 수집하여 리스트에 저장
```java
public SimpleExcelMetadata createSimpleExcelMetaData(
        Class<?> type, Workbook workbook, SheetType sheetType, boolean hasGroupHeader) {

    List<Field> fields = getExcelAnnotatedFields(type);
    List<String> headerNames = new ArrayList<>();

    for (Field field : fields) {
        ExcelColumn excelColumn = field.getAnnotation(ExcelColumn.class); // 어노테이션 체크
        String headerName = excelColumn.headerName();
        headerNames.add(headerName);

        applyCellStyle(cellStyleMap, excelColumn.headerStyle(), null, field.getName(), HEADER, workbook);
    }
    return new SimpleExcelMetadata(headerNames, fields, cellStyleMap, groups);
}

```


<br>

## `Service`

- 엑셀로 출력할 기존 객체 → `CustomExcelDto`로 파싱 후 `simpleExcelWrite`로 엑셀 출력
- 쿼리 데이터 조회시 페이징으로 메모리 관리
```java
// 1) 엑셀 파일 생성 (데이터 -> ExcelFile)
SimpleExcelFile<E> excelFile = new SimpleExcelFile<>(  
        simpleExcelWriteDto.getData(),  
        simpleExcelWriteDto.getType(),  
        simpleExcelWriteDto.getSheetName(),  
        simpleExcelWriteDto.getSheetType()  
);  
  
// 2) ExcelSetUpDto 간단히 만들어서, 기존 write(...) 메서드 사용  
ExcelSetUpDto excelSetUpDto = ExcelSetUpDto.builder()  
        .response(simpleExcelWriteDto.getResponse())  
        .excel(excelFile.getWorkbook())  
        .excelPreFileTitle(simpleExcelWriteDto.getPreFileTitle())  
        .excelFilePath(simpleExcelWriteDto.getFilePath())  
        .build();  
  
// 3) 한 번에 write
excelFile.write(excelSetUpDto);
```
<br>

---

<br>

# <font color="#c0504d">메모리 비교 모니터링</font>

<br>

## <u><font color="#76923c">기존 로직</font></u>

<br>

### **STG서버** Excel Export 로직 실행시

case 1
```json
입력 :
	예치금 입금내역
	셀 개수 : 40,000건 
	
출력 :
	실패
```

case 2
```json
입력 :
	예치금 입금내역
	셀 개수 : 5,000건 
	
출력 :
	성공
```

![[Pasted image 20250516163401.png]]
case1(4만건) - 메모리 사용율 *80%* 초과로 인한 순단 발생
![[Pasted image 20250516163522.png]]
이후, case2(5천건) - 메모리 사용율 *51%*


#### 예상 최대 수용 가능 셀 
- **예치금 입금내역(9개의 속성 X 8,000필드)** → <u>약 7만 셀</u>

<br>

---

<br>

# <u><font color="#76923c">리펙토링 로직</font></u>

<br>

### **Dev서버** Excel Export 로직 실행시

test case
```json
입력 :
	예치금 입금내역
	셀 개수 : 70,000건 
	
출력 :
	성공
```
- cpu 최대 사용율 17.2%

![[Pasted image 20250513101628.png]]
- 메모리 469MB

![[Pasted image 20250513101743.png]]
![[Pasted image 20250516154726.png|725]]


---

### **Stg서버** Excel Export 로직 실행시

case1
```json
입력 :
	예치금 입금내역
	셀 개수 : 70,000건 
	
출력 :
	성공
```
![[Pasted image 20250516164727.png]]
![[Pasted image 20250516164550.png]]
![[Pasted image 20250516164537.png]]
![[Pasted image 20250516165030.png]]

- 메모리 사용율 약 36%

<br>

case2
```json
입력 :
	예치금 입금내역
	셀 개수 : 140,000건 
	
출력 :
	성공
```

![[Pasted image 20250516165307.png]]
![[Pasted image 20250516165250.png]]
![[Pasted image 20250516165344.png]]

- 메모리 사용율 약 38%

<br>

case3
```json
입력 :
	예치금 입금내역
	셀 개수 : 240,000건 
	
출력 :
	성공
```

![[Pasted image 20250516165523.png]]
![[Pasted image 20250516165617.png]]
![[Pasted image 20250516165627.png]]
- 메모리 사용율 약 40%


#### 예상 최대 수용 가능 셀 
- **예치금 입금내역(9개의 속성 X 666,000필드)** → <u>약 600만 셀</u>
---

# <u><font color="#76923c">결론</font></u>

<br>


- 최대 엑셀 셀 수용량 약 100배 증가.

