# 타입스크립트와 자바스크립의 관계

* 타입스크립는 자바스크립의 상위 집합
* 아래 코드는 유효한 타입스크립 코드 자바스크립에서는 `who:string`타입 구문으로 인해 오류를 출력한다.
```typescript
function greeting(who: string) {
    console.log('Hello ' + who);
}
```
* 타입스크립는 초기값으로부터 타입을 추론한다.
```typescript
let city = 'Seoul';
console.log(city.toUppercase());
```
* 위 코드는 자바스크립에서는 오류를 출력하지 않지만 타입스크립에서는 오류를 출력한다.
> // 'toUppercase' does not exist on type 'string'. <br/>
> // Did you mean 'toUpperCase'?
* 타입시스템의 목표 중 하나는 런타에 발생할 수 있는 오류를 미리 잡는 것이다.
```typescript
const states = [
  {name: 'Alabama', capital: 'Montgomery'},
]
console.log(states[0].capitol)
```
* 해당 코드의 실행 결과는 undefined 이다. 타입스크립는 이를 오류로 인식한다.
> capitol does not exist on type '{name: string; capital: string;}'.
* 타입 구문을 추가 한다면 더 많은 오류를 찾아낼 수 있다. capitol 와 capital의 위치를 바꾸었을 때 어느쪽이 오타인지 알 수 없다.
* 타입 구문을 통해 코드의 '의도'를 추가여 코드'의도'를 알려줄 수 있다.
```typescript
interface State{
  name: string;
  capital: string;
}
const states: State[] = [
  {name: 'Alabama', capital: 'Montgomery'},
]
```
> Property 'capitol' does not exist on type 'State'. <br/>
> Did you mean 'capital'?
* 타입스크립타입 시스템은 자바스크립트의 런타임 동작을 '모델링'함
* 아래 코드는 다른 런타임 타입 체커와 다르게 타입스크립의 타입체커는 문자열 "23"이 되는 자바스크립트 런타임 동작으로 모델링 됨
* 자바스크립에서 정상 작동을 하지만 타입스크립에서는 오류를 출력하는 경우도 있음.
```typescript
const x = 3 + '3'; // 정상
const y = '3' + 2; // 정상
const z = null + 7; // javascript: 7, 오류
const a = [] + 11; // javascript: 11, 오류
```
* 작성된 프로그램이 타입체커를 통과하여도 런타임에서 오류가 발생할 수 있음
```typescript
const x = ['a', 'b'];
console.log(x[3].toUpperCase());
```
* 타입스크립가 이해하는 타입값과 실제 값에 차이가 있기 때문


# 코드 생성과 타입이 관계없음을 이해하기
* 타입스크립컴파일러는 2가지 역할을 수행
  * 최신 타입/자바 스크립트 코드를 브라우저에서 동작할 수 있게 구버전으로 트렌스파일
  * 코드의 타입 오류를 체크
* 이 두가지 동작은 완벽히 독립적 타입 -> 자바로 변환/샐행될 때 타입의 영향을 주지 않음
### 타입 오류가 있는 코드 컴파일이 가능
* 컴파일과 타입 체크는 독립적으로 동작하기에 타입에 오류가 있어도 컴파일이 가능
### 런타임에는 타입 체크가 불가능
```typescript
interface Square {
  width: number;
}
interface Rectangle extends Square {
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape instanceof Rectangle) {
                       // Rectangle은 형식을 참조하지만
                       // 여기선 값으로 사용되고 있음
    return shape.width * shape.height;
  } else {
    return shape.width * shape.width;
  }
}
```
* `instanceof` 연산자는 런타임에 체크하기 때문에 타입스크립트에서는 오류를 출력하지 않음
* 타입스크립트의 타입은 '제거 가능'함
* 실제로 자바스크립로 컴파일되는 과정에서 모든 인터페이스, 타입, 타입 구문은 제거됨
* 위의 `Shape`타입을 명확하게 하려면 런타임에 타입 정보를 유지해야함
* 아래는 `height` 속성이 존재하는지 체크해 보는 코드
```typescript
function calculateArea(shape: Shape) {
  if ('height' in Shape) {
    // shape은 여기서 Rectangle로 추론됨
    return shape.width * shape.height;
  } else {
    // shape은 여기서 Square로 추론됨
    return shape.width * shape.width;
  }
}
```
* 타입 정보를 유지하는 방법으로는 런타임에 접근 가능한 타입 정보를 명시적으로 저장하는 '태그'기법이 존재
```typescript
interface Square {
  kind: 'square';
  width: number;
}
interface Rectangle {
  kind: 'rectangle';
  width: number;
  height: number;
}
type Shape = Square | Rectangle;

function calculateArea(shape: Shape) {
  if (shape.kind === 'rectangle') {
    // shape은 여기서 Rectangle로 추론됨
    return shape.width * shape.height;
  } else {
    // shape은 여기서 Square로 추론됨
    return shape.width * shape.width;
  }
}
```
## 타입 연산은 런타임에 영향을 주지 않음
* `string` 또는 `number` 타입인 값을 항상 `number`로 정하는 경우를 가정
```typescript
function asNumber(val: string | number): number {
  return val as number;
}
```
* 위 코드는 타입체커를 통과하지만 런타임에는 영향을 미치지 않음
* 값의 정제를 위해서는 런타임의 타입을 체크하고 자바스크립트 연산을 통해 변환을 수행해야 함
```typescript
function asNumber(val: string | number): number {
  return typeof val === 'string' ? Number(val) : val;
}
```
## 런타임 타입은 선언된 타입과 다를 수 있음
```typescript
function setLightSwitch(value: boolean) {
  switch (value) {
    case true:
      console.log('ON');
      break;
    case false:
      console.log('OFF');
      break;
    default:
      console.log('UNKNOWN');
  }
}
```
* 타입스크립는 일반적으로 죽은 코드를 찾아내지만 여기선 `strict가` 적용되어 있어도 못 찾음
* 마지막줄이 실행가능한 이유는 `value가` `boolean이` 아닌 경우가 있기 때문
## 타입스크립트 타입으로는 함수를 오버로드할 수 없다
* 타입스크립트에서는 `C++`와 같은 언어와 다르게 타입과 런타입의 동작이 무관하기에 함수 오버로딩은 불가능
* 타입스크립트가 오버로딩 기능을 지원하기는 하지만 온전히 타입수준에서만 동작
* 하나의 함수에 대해 여러가지 선언문을 작성 가능하지만, 구현체는 하나
```typescript
function add(a: number, b: number): number;
function add(a: string, b: string): string;

function add(a, b) {
  return a + b;
}

const x = add(1, 2); // number
const y = add("1", "2"); // string
```
* `add`에 대한 처음 두 개의 선언문은 타입 보를 제공할 뿐
* 두 선언문은 자바스크립트를 
## 타입스크립 타입은 런타임 성능에 영향을 주지 않음
* 타입과 타입 연산자는 자바스크립트 변환 시점에 제거되기 때문에 런타임 성능에 영향을 주지 않음
  * 런타임 오버헤드가 없는 대신 컴파일 타임 오버헤드가 존재
## 구조적 타이핑에 익숙해지기
* 타입스크립트는 본질적으로 `덕 타이핑`기반
  >  덕 타이핑이란 객체가 어떤 타입에 부합하는 변수와 메서드를 가질 경우 객체를 해당 타입에 속하는걸로 간주하는 방식
* 어떤 함수의 매개변수 값이 모두 제대로 주어진다면, 그 값이 어떻게 만들어졌는지 신경쓰지 않고 사용
* 매개변수 값이 요구사항을 충족하면 타입이 뭔지 신경안쓰고 동작을 모델링함
* 타입 체커의 이해가 사람과 다르기에 예상치 못한 결과가 나올 수 있고 이는 구조적 타이핑을 이해하면 해결할 수 있음
```typescript
interface Vector2D{
  x: number;
  y: number;
}

function calculateLength(v: Vector2D) {
  return Math.sqrt(v.x * v.x + v.y * v.y);
}

interface NamedVector {
  name: string;
  x: number;
  y: number;
}

const v: NamedVector = { x: 3, y: 4, name: 'Zee' };
cancelIdleCallback(v); // 정상 : 5
```
* `Vector2D`와 `NamedVector`는 관계를 선언하지 않아도 결과가 정상적으로 나온다.
* `NamedVector`를 위한 `calculateLength` 함수를 따로 만들 필요가 없다.
* `NamedVector`의 구조가 `Vector2D`와 호환되기 때문에 `calculateLength` 함수의 호출이 가능
* 여기서 `덕 타이핑`이라는 용어가 사용됨
* 구조적 타이핑 때문에 문제가 발생할 수 있음
```typescript
interface Vector3D {
  x: number;
  y: number;
  z: number;
}

function nomalize(v: Vector3D) {
  const length = calculateLength(v);
  return {
    x: v.x / length,
    y: v.y / length,
    z: v.z / length,
  };
}
nomalize({ x: 3, y: 4, z: 5 }); // 비정상 : { x: 0.4242640687119285, y: 0.565685424949238, z: 0.7071067811865475 }
```
* `calculateLength` 함수는 `Vector2D`를 매개변수로 받기 때문에 `Vector3D`를 매개변수로 받는 `nomalize` 함수에서 `z`가 정규화 과정에서 무시됨
* `Vector3D`와 호환되는 `{x, y, z}`로 `calculateLength` 함수를 호출하면 구조적 타이핑으로 `Vector2D`와 호환이 되어버림
* 따라서 오류가 발생하지않고 타입체거가 문제로 인식하지 않음
* 타입스크립트의 타입은 타입의 확장에 `열려`있음
```typescript
function calculateLength1(v: Vector3D) {
  let length = 0;
  for (const axis of Object.keys(v)) {
    const coord = v[axis]; // string은 vector3d의 인덱로 사용될 수 없음
                           // 엘리먼트는 암시적으로 any 타입
    length += coord * coord;
  }
  return Math.sqrt(length);
}
```
* `calculateLength1` 함수는 `Vector3D`를 매개변수로 받기 때문에 `Vector2D`를 매개변수로 받는 `calculateLength` 함수에서 `z`가 정규화 과정에서 무시됨
* 키들은 `number`로 예측이 되지만 그렇지 않음
```typescript
const v3d = { x: 1, y: 2, z: 3, h: '응가' };
calculateLength1(v3d); // 비정상
```
* v는 어떤 속성이든 가질 수 있기 때문에 number로 확정할 수 없음
* 루프보다는 모든 속성을 각각 더하는 구현이 더 나음
````typescript
function calculateLength1(v: Vector3D) {
  return Math.sqrt(v.x * v.x + v.y * v.y + v.z * v.z);
}
````
* 구조적 타이핑은 클래스와 관련되 할당문에서도 문제를 일으킬 수 있음
```typescript
class C{
    foo: string;
    constructor(foo: string) {
        this.foo = foo;
    }
}

const c = new C('instance of C');
const d: C = { foo: 'object literal' }; // 정상
```
* 여기서 `d`에 `C`가 할당되는 이유는 `C`의 인스턴스가 `foo` 속성을 가지고 있고 `Object.prototype`에서 비롯된 생성자를 가지고 있기 때문
* 만약 `C`에 단순할당이 아닌 계산이 있다면 문제가 발생
# any 타입 지양하기
* 타입스크립트의 타입은 점진적이고 선택적임
* 코드에 타입을 조금씩 추가할 수 있어서 점진적이고 언제든 타입체킹이 가능해 선택적임
* 이 기능들의 핵심은 `any`
## any 타입에는 타입 안정성이 없음
```typescript
let age:number;
age = '12'; // 비정상
age = '12' as any; // 정상

age += 1; // 런타임에 정상 : 121
```
## any는 함수 그니처를 무시함
* 함수를 작성할 때 시그니처를 명시해야 함
* 호출쪽은 약속된 타입의 입력을 제공하고, 함수는 약속된 타입의 출력을 반환함
* `any`타입은 이런 약속을 무시할 수 있음
## any 타입에는 언어 서비스가 적용되지 않음
* `any`타입은 타입스크립트의 자동완성과 적절한 도움말 기능을 제공받지 못 함
* 타입스크립트의 모토는 확장 가능한 자바스크립트
* 확장의 중요한 부분은 타입스크립트 경험의 핵심인 언어 서비스
## any 타입은 코드 리팩터링 때 버그를 감춤
## any 타입은 설계를 감춤
* 어플리케이션의 상태같은 객체의 정의는 복잡하지만 `any`타입을 사용하면 간단하게 정의할 수 있음
* 하지만 객체의 설계를 감추어 설계가 불분명해지고, 코드의 유지보수가 어려워짐
## any 타입은 타입시스템의 신뢰도를 떨어트림
* 휴먼 에러를 타입체킹을 통해 줄이고 코드의 신뢰도를 높여야 함
* `any`타입을 사용하면 런타임에서 발생하는 오류를 미리 알 수 없고 신뢰도가 떨어짐
