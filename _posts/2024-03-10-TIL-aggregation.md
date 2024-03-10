---
title: "MongoDB에서의 aggregation"
excerpt: " "
categories: 
  - Programming Language
tags:
  - MongoDB
toc: true
toc_sticky: true
# last_modified_at : 2022-04-12T00:30:00
---


## Aggregation?

MongoDB에서 지원하는 Aggregation은 많은 document들을 특정한 pipeline을 통해 한 번에 처리할 수 있게 해 주는 기능이다. Aggregation을 이용하면 여러 document들을 특정 기준으로 묶거나, 묶인 document들을 대상으로 연산을 수행해 결과를 도출하는 작업 등이 가능하다.

## Aggregation Pipeline
Aggregation Pipeline은 특정 collection 내의 document들에 대해 시행할 연산을 차례대로 정의한 것이다. Pipeline 내의 각 과정을 stage라고 부르고, 각 stage는 document들을 필터링하거나 그룹으로 묶고, 그룹 내 document들의 특정 field의 합이나 평균을 구하는 등의 연산을 수행한다. 한 stage의 연산을 수행하고 나면 그 결과가 다음 stage로 전달되고 모든 stage의 연산을 수행하고 나면 Aggregation 과정이 종료된다.

### Aggregation Pipeline의 사용 방법

> *예시로 사용한 DB와 Aggregation Pipeline은 MongoDB Docs에서 발췌하였다.*

다음과 같은 Document들로 이루어진 collection이 있다고 하자.

```javascript
db.orders.insertMany( [
   { _id: 0, name: "Pepperoni", size: "small", price: 19,
     quantity: 10, date: ISODate( "2021-03-13T08:14:30Z" ) },
   { _id: 1, name: "Pepperoni", size: "medium", price: 20,
     quantity: 20, date : ISODate( "2021-03-13T09:13:24Z" ) },
   { _id: 2, name: "Pepperoni", size: "large", price: 21,
     quantity: 30, date : ISODate( "2021-03-17T09:22:12Z" ) },
   { _id: 3, name: "Cheese", size: "small", price: 12,
     quantity: 15, date : ISODate( "2021-03-13T11:21:39.736Z" ) },
   { _id: 4, name: "Cheese", size: "medium", price: 13,
     quantity:50, date : ISODate( "2022-01-12T21:23:13.331Z" ) },
   { _id: 5, name: "Cheese", size: "large", price: 14,
     quantity: 10, date : ISODate( "2022-01-12T05:08:13Z" ) },
   { _id: 6, name: "Vegan", size: "small", price: 17,
     quantity: 10, date : ISODate( "2021-01-13T05:08:13Z" ) },
   { _id: 7, name: "Vegan", size: "medium", price: 18,
     quantity: 10, date : ISODate( "2021-01-13T05:10:13Z" ) }
] )
```

Aggregation Pipeline은 `db.collection.aggregate()` 메소드로 사용할 수 있고, 인자로 Aggregation Pipeline을 받는다. Aggregation Pipeline은 각각의 stage에서 수행할 연산 작업들의 배열로 정의된다.

```javascript
db.orders.aggregate( [
  // stage 1
  {
    $match: { size: "medium" }
  },
  
  // stage 2
  {
    $group: { _id: "$name", totalQuantity: { $sum: "$quantity" } }
  }
] )
```

### Aggregation Operator

Aggregation Pipeline의 각 stage에서 수행할 연산을 정의하는 연산자를 Aggregation Operator라고 한다. 매우 다양한 종류의 연산자를 지원하지만, 대표적으로 사용할 수 있는 몇 가지만 소개해 보려고 한다.

1. `$match`
collection 내에서 주어진 query의 조건에 맞는 document들을 반환한다. 기존의 `collection.find()` 메소드와 거의 동일한 방법으로 사용할 수 있다. 

```javascript
// result of stage 1:
[
   { _id: 1, name: "Pepperoni", size: "medium", price: 20,
     quantity: 20, date : ISODate( "2021-03-13T09:13:24Z" ) },
   { _id: 4, name: "Cheese", size: "medium", price: 13,
     quantity:50, date : ISODate( "2022-01-12T21:23:13.331Z" ) },
   { _id: 7, name: "Vegan", size: "medium", price: 18,
     quantity: 10, date : ISODate( "2021-01-13T05:10:13Z" ) }
]
```

2. `$group`
`$group` 연산자는 document들을 특정한 기준에 따라 그룹화하고, 각각의 그룹에 대해 정해진 연산을 수행해 그 결과를 반환한다. `$group` 연산자는 다음과 같은 형태로 사용한다.

```javascript
{
 $group:
   {
     _id: <expression>, // Group key
     <field1>: { <accumulator1> : <expression1> },
     ...
   }
 }
```

`_id` 필드는 group key라고 불리며, **document들을 그룹화하는 기준**이 된다. 예를 들어, 위의 예시에서 `_id`를 `$name`으로 설정하면 같은 `$name`끼리 그룹화된 결과를 반환할 것이다. `_id`를 `null`로 설정하면 input으로 주어진 모든 document들에 대해 연산을 수행한다.

`<field>` 필드에서는 그룹화된 document들에 대해 수행할 연산을 정의한다. `accumulator`와 `expression`으로 연산을 정의하면, 그 수행 결과가 `$group` 연산자의 결과 document에 `field` 이름으로 저장된다.

예를 들어, 위의 collection에서 `name`이 `Pepperoni`인 document들의 `price`의 평균을 계산할 수 있다.

```javascript
// 주어진 document들을 name 필드를 기준으로 그룹화하고, 각 그룹의 document들의 price 필드의 평균을 averagePrice 필드에 저장해 반환한다.
db.orders.aggregate([
  {
    $group: {
      _id: "$name",
      averagePrice: { $avg: "price" }
    }
  }
])
```

3. `$project`
`$project` 연산자는 다음 stage로 넘어갈 때 document 내의 지정된 field만 반환하도록 하는 연산자이다. `{ 필드명 : 0 or 1 }`의 구조로 사용하며, 0으로 지정한 field는 제외되고 1로 지정한 field는 포함된다. `$project` 연산자를 통해 특정 field를 명시적으로 output에 포함시킬 경우 연산자에 선언되지 않은 field들은 자동으로 output에서 제외된다.

4. `$unwind`
`$unwind` 연산자는 배열 타입 필드를 deconstruct하는 데 사용한다. input document에 배열 타입인 필드가 있다면, 배열의 각각의 element를 값으로 갖는 field를 가진 document들을 만들어 반환한다.

```javascript
const input_document =
  { "_id" : 1, "item" : "ABC1", sizes: [ "S", "M", "L"] }

// sizes 배열을 deconstruct해서 각각의 element를 output document들의 sizes field에 할당한다.
db.collection.aggregate([
  {
    $unwind: "sizes"
  }
])

const output_documents = [
  { "_id" : 1, "item" : "ABC1", sizes: "S" },
  { "_id" : 1, "item" : "ABC1", sizes: "M" },
  { "_id" : 1, "item" : "ABC1", sizes: "L" },
]
```

5. 기타 유용한 연산자들

- `$sort` 연산자는 주어진 document들을 특정 field를 기준으로 오름차순 또는 내림차순 정렬한다. `{ 필드명 : 1 or -1 }`의 형태로 사용하며, 1일 경우 오름차순, -1일 경우 내림차순으로 정렬한다.
- `$skip` 연산자는 주어진 document들을 인자로 주어진 개수만큼 skip한 결과를 반환한다.
- `$limit` 연산자는 주어진 document들 중 인자로 주어진 개수만큼의 document를 반환한다.


## Reference

- [Aggregation Operations - MongoDB Docs](https://www.mongodb.com/docs/manual/aggregation/)