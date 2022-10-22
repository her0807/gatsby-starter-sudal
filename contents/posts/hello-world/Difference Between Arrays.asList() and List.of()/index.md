---
title: "Difference Between Arrays.asList() and List.of() 요약"
description: ""
date: "2022-10-22"
update: "2022-10-22"
tags:
  - java
  - List.of
  - Arrays.asList

---

> 해당 내용은  **Baeldung 사이트를 번역한 후, 제 생각을 정리한 글입니다.**

<br/>
<br/>

# **1. 개요**

때때로 Java에서는 편의를 위해 작은 목록을 만들거나 배열을 목록으로 변환해야 합니다. Java는 이를 위한 몇 가지 도우미 메서드를 제공합니다. 이 자습서에서는 작은 임시 배열을 초기화하는 두 가지 주요 방법인 *List.of()* 와  *Array.asList()를 비교합니다.*

<br/>


# **2. *Arrays.asList() 사용하기***

*[Java 1.2에 도입된 Arrays.asList()](https://www.baeldung.com/java-arraylist) 는 Java Collections Framework* 의 일부인 *List* 객체 생성을 단순화합니다. 배열을 입력으로 받아 제공된 배열 *의 List 객체를 생성할 수 있습니다.*

```java
Integer[] array =newInteger[]{1, 2, 3, 4};
List<Integer> list = Arrays.asList(array);
assertThat(list).containsExactly(1,2,3,4);복사
```

*보시다시피 간단한 List* of *Integers* 를 만드는 것은 매우 쉽습니다 .

<br/>


# **2.1. *Arrays.asList() 사용해서 반환받은 List 는 삽입, 삭제가 불가능합니다.  (수정은 가능)***

*asList()* 메서드 는 고정 크기 목록을 반환합니다. 따라서 새 요소를 추가 및 제거하면 *`UnsupportedOperationException`* 이 발생합니다.

<br/>

# **2.2. 배열 작업**

배열을 List 로 변환 했다면, 참조가 끊기지 않았으니 배열에 대한 변경 사항이 목록에도 반영 됩니다. 
결국 이는 원하지 않는 부작용으로 이어져 찾기 어려운 버그를 유발할 수 있습니다. 

```java
Integer[] array =newInteger[]{1,2,3};
List<Integer> list = Arrays.asList(array);
array[0] = 1000;
assertThat(list.get(0)).isEqualTo(1000);
```

<br/>

---

<br/>

# **3. *List.of() 사용***

*배열* 과 대조적 입니다. *asList ()* , Java 9에서는 보다 편리한 메서드인 *[List.of()](https://www.baeldung.com/java-init-list-one-line#factory-methods-java-9)* 를 도입했습니다 . `이렇게 하면 수정할 수 없는 List 개체의 인스턴스가 생성`됩니다.

```
String[] array =newString[]{"one", "two", "three"};
List<String> list = List.of(array);
assertThat(list).containsExactly("two", "two", "three");복사
```

<br/>

# **3.1. *Arrays.asList()와의* 차이점**

*Arrays.asList()* 와의 주요 차이점은 `List.of()가 제공된 입력 배열 의 복사본인 변경할 수 없는 목록을 반환 한다`는 것입니다. 이러한 이유로 원래 배열에 대한 변경 사항은 반환된 목록에 반영되지 않습니다.

```java
String[] array =newString[]{"one", "two", "three"};
List<String> list = List.of(array);
array[0] = "thousand";
assertThat(list.get(0)).isEqualTo("one");
```

그렇기 때문에 목록의 요소를 수정할 수 없습니다. 시도하면 *`UnsupportedOperationException`* 이 발생합니다 .

```java
List<String> list = List.of("one", "two", "three");
assertThrows(UnsupportedOperationException.class, () -> list.set(1, "four"));
```

<br/>


# **3.2. 널 값**

또한 *List.of() 는 null* 값을 입력으로 허용하지 않으며 *`NullPointerException`* 을 throw합니다 .

```
assertThrows(NullPointerException.class, () -> List.of("one", null, "two"));복사
```

<br/>

> ### 참고 자료

- [baeldung](https://www.baeldung.com/java-arrays-aslist-vs-list-of)