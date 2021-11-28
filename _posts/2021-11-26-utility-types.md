---
layout: post
title: "Record<string, any>"
subtitle: "뭔데 이게"
date: 2021-10-10 12:30:15 +0900
background: '/img/posts/05.jpg'
---

타입스크립트에서 지원하는 유틸리티 타입에 대해서 정리해보았다.
한번 정리 해놓고 계속 활용할 수 있었으면 좋겠다.

대부분의 예제와 텍스트들은 해당 링크를 참조하였다 [링크](https://www.typescriptlang.org/docs/handbook/utility-types.html)



### Record<K, T>
프로퍼키 키가 k이고 프로퍼티 값이 T 형식인 객체의 형식을 정의한다.

``` typescript
interface CatInfo {
  age: number;
  breed: string;
}
 
type CatName = "miffy" | "boris" | "mordred";
 
const cats: Record<CatName, CatInfo> = {
  miffy: { age: 10, breed: "Persian" },
  boris: { age: 5, breed: "Maine Coon" },
  mordred: { age: 16, breed: "British Shorthair" },
};
```

이 유틸리티를 가장 많이 본 경우는, Request 등 에서 하부 프로퍼티로 객체를 가지는 경우에 형태를 정의하기 위함이였다.

예를 들어, 
``` typescript

Record<sting, any> // is strict way of decare {}

const example: Record<CatName, CatInfo> = {
  "aa": "aa Text", // Record<string, string>
  "bb": 10000, // Record<string, number>
  "cc": { "dd" : "dd Text" }, // Record<string, any>, and // Record<string, string>
  "ee": { // Record<string, any>
	  "ff": "ff Text", // Record<string, string>
	  "gg": "gg Text" // Record<string, string>
  }
};
```
와 같은 형태로 사용할 수 있게 된다.

이게 {} 와 무엇이 다른가 라고 한다면, 해당 [링크](https://stackoverflow.com/questions/60996831/differences-between-recordstring-any-and)의 글을 참조해 보면

``` typescript
let a:{} = {"aa": "a text"} 
console.log(a) // {"aa": "a text"}

a = 1;

console.log(a); // 
```

a 는 json 형태로 잡고 싶은데 1을 넣어도 에러가 발생하지 않는다!

문제가 생길 냄새가 스멀스멀 올라온다.
이러한 이유 때문에 Record<string, any>와 같은 형태로 사용하게 되는 것이ㅏㄷ.


### Exclude<T, U>
Exclude 타입은 2개의 제네릭 타입을 받을 수 있으며,
첫번째 제네릭 타입 T 중 두번째 제네릭 타입 U와 겹치는 타입을 제외한 타입을 반환한다.

타입 T가 타입 U를 상속하거나 동일 타입이라면 무시(never)하고 아닐 경우 타입 값을 리턴한다.

``` typescript
type T0 = Exclude<"a" | "b" | "c", "a">; // type T0 = "b" | "c"
     

type T1 = Exclude<"a" | "b" | "c", "a" | "b">; // type T1 = "c"
     

type T2 = Exclude<string | number | (() => void), Function>; // type T2 = string | number
```

다시 말해 두번째 제네릭 타입 U에 대해
첫번째 제네릭 타입 T가 할당 가능한 타입인지를 판단하고,
할당 가능한 타입을 제외한 나머지 타입들을 이용하여 타입을 정의한다.

### Extract<T, U>
Exclude와 반대되는 개념이다. T 타입에서 U 타입과 겹치는 타입만을 추출한다.

``` typescript
type T0 = Extract<"a" | "b" | "c", "a" | "f">; // type T0 = "a"

type T1 = Extract<string | number | (() => void), Function>;  // type T1 = () => void
```

Exclude 타입과는 다르게 유니온 타입 Event와 Point에 대하여 Point 할당 타입을 반환하기 때문에 PointInfo 타입은 Point 타입과 같아진다. 내가 원하는 프로퍼티로 구성된 타입이 이미 있을 때 사용할 수 있을거 같다.

### Pick<T, K>
T 타입으로부터 K 프로퍼티만 추출한다.

``` typescript
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
};
```

주어진 첫번째 제네릭 타입 T 내에서 Union 타입 K라는 프로퍼티에 대한 타입들을 뽑아낸다.
필요한 타입만 추출해서 원하는 타입을 만들 수 있다.

``` typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
}
 
type TodoPreview = Pick<Todo, "title" | "completed">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
};
 
```

### Partial<T>
Partial은 제네릭 타입 T에 대해서 모든 프로퍼티들을 Optional하게 변경한다.

제네릭 타입의 프로퍼티들에 대해 기존의 타입은 유지하되, 각각의 프로퍼티들을 Optional 타입으로 변경해준다. 필수 타입과 Optional 타입을 구분해서 사용해야 하는 경우 아래와 같이 쓸 수 있다.

``` typescript
interface Todo {
  title: string;
  description: string;
}
 
function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
  return { ...todo, ...fieldsToUpdate };
}
 
const todo1 = {
  title: "organize desk",
  description: "clear clutter",
};
 
const todo2 = updateTodo(todo1, {
  description: "throw out trash",
});

console.log(todo2) // { "title": "organize desk", "description": "throw out trash" } 
```

### Required<T>
Required는 위의 Partial과 반대되는 개념이다. 제네릭 타입 T의 모든 프로퍼티에 대해 Required 속성으로 만들어준다.

마이너스 연산자는 Optional을 제거해준다는 의미의 연산자이다.
Partial 타입과 동일하게 기존의 타입은 유지된 상태에서 Required 타입으로 변경된다.

``` typescript
interface Props {
  a?: number;
  b?: string;
}
 
const obj: Props = { a: 5 };
 
const obj2: Required<Props> = { a: 5 }; // Error: Property 'b' is missing in type '{ a: number; }' but required in type 'Required<Props>'.
```

### ReadOnly<T>
T의 모든 프로퍼티를 읽기 전용(readOnly)으로 설정한 타입을 구성한다. 즉 모든 프로퍼티의 값을 변경할 수 없고 참조만 할 수 있도록 만든다.

ReadOnly로 생성된 타입에 값을 재할당하려고 하면 에러가 난다.

``` typescript
interface eventState {
  title: string;
  id: number;
}

const event: ReadOnly<eventState> = {
  title: 'new event',
  id: 123,
};

event.title = 'Hello'; // 에러 발생!
```

### Omit<T, K>
Omit 타입은 두개의 제네릭 타입을 받으며 T에서 모든 프로퍼티를 선택한 다음 K를 제거한 타입을 구성한다.


``` typescript
interface Todo {
  title: string;
  description: string;
  completed: boolean;
  createdAt: number;
}
 
type TodoPreview = Omit<Todo, "description">;
 
const todo: TodoPreview = {
  title: "Clean room",
  completed: false,
  createdAt: 1615544252770,
};
 
 
type TodoInfo = Omit<Todo, "completed" | "createdAt">;
 
const todoInfo: TodoInfo = {
  title: "Pick up kids",
  description: "Kindergarten closes at 5pm",
};
 
console.log(todoInfo); // {
//  "title": "Pick up kids",
//  "description": "Kindergarten closes at 5pm"
//} 
```

보다보니까 Exclude와 다른점이 뭘까라는 생각이 들었는데, Exclude는 두번째 제네릭 타입에 타입이 들어가고 Omit은 키값으로 넣어준다는 게 차이점인 것 같다.
Exclude<T, U> : T에서 U에 할당할 수 있는 타입을 제외한 타입 반환
Omit<T, U> : T의 모든 프로퍼티 중 U가 제거된 프로퍼티들의 타입 반환

### NonNullable<T>
주어진 제네릭 타입 T에서 null과 undefined를 제외한 타입을 구성한다.

``` typescript
type T0 = NonNullable<string | number | undefined>; // type T0 = string | number
type T1 = NonNullable<string[] | null | undefined>; // type T1 = string[]
```

### ReturnType<T>

ReturnType 타입은 주어진 제네릭 타입 T의 return type을 할당한다.


``` typescript
declare function f1(): { a: number; b: string };
 
type T0 = ReturnType<() => string>; // type T0 = string
     

type T1 = ReturnType<(s: string) => void>; // type T1 = void
     

type T2 = ReturnType<<T>() => T>; // type T2 = unknown
     

type T3 = ReturnType<<T extends U, U extends number[]>() => T>; // type T3 = number[]
     

type T4 = ReturnType<typeof f1>; // type T4 = { a: number; b: string; }
     

type T5 = ReturnType<any>; // type T5 = any
     

type T6 = ReturnType<never>; // type T6 = never
     

type T7 = ReturnType<string>;
`
Type 'string' does not satisfy the constraint '(...args: any) => any'.
`    


type T8 = ReturnType<Function>;
`
Type 'Function' does not satisfy the constraint '(...args: any) => any'.
  Type 'Function' provides no match for the signature '(...args: any): any'.
`    

```


이 이외에도 Uppercase, Lowercase, Capitalize, Uncapitalize 등의 유틸리티도 있지만 이는 따로 정리하지 않았다. 

앞으로도 이러한 유틸리티로 정의된 입력 형식과 출력형식에서 안쫄아도 되겠다는 생각이 들고, 앞으로도 안쫄았으면 좋겠다...