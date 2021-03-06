# 소개

[JSX](https://facebook.github.io/jsx/)는 XML 같은 구문이  포함 가능합니다.  
의미는 구현에 따라 다르지만 유효한 JavaScript로 변형되어야 합니다.  
JSX는 [React](http://facebook.github.io/react/)에서 인기를 얻었으나 이후 다른 애플리케이션도 볼 수 있습니다.  
TypeScript는 JSX를 직접 JavaScript에 포함, 타입 검사 및 컴파일할 수 있도록 지원합니다.
# 기본 사용 방법

JSX를 사용하려면 두 가지 작업을 해야 합니다.

1. 파일의 이름을 `.tsx` 확장자로 지정하세요
2. `jsx` 옵션을 활성화하세요

TypeScript에는 세 가지 JSX 모드가 있습니다: `preserve`, `react`, 그리고 `react-native`.  
이 모드는 방출 단계에만 영향을 미칩니다 - 타입 검사에는 영향받지 않습니다.  
`preserve` 모드는 다른 변환 단계 (예: [Babel](https://babeljs.io/))에서 더 사용되도록 출력의 일부로 JSX를 계속 유지합니다.  
추가적으로 출력에는 `.jsx` 파일 확장자가 지정되어 있습니다.  
`react` 모드는 `React.createElement`를 내보내고 사용하기 전에 JSX 변환을 거칠 필요가 없으며 출력은 `.js` 파일 확장자를 갖습니다.  
`react-native` 모드는 모든 JSX를 유지하고 있다는 점에서 `preserve`와 같지만 대신 출력은 `.js` 파일 확장자를 갖습니다.

모드           | 입력      | 출력                         | 출력 파일 확장자
---------------|-----------|------------------------------|----------------------
`preserve`     | `<div />` | `<div />`                    | `.jsx`
`react`        | `<div />` | `React.createElement("div")` | `.js`
`react-native` | `<div />` | `<div />`                    | `.js`

이 모드는 커맨드 라인의 `--jsx` 명령 또는 [tsconfig.json](./tsconfig.json.md) 파일의 해당 옵션을 사용하여 지정할 수 있습니다.

> *주의사항: `React` 식별자는 하드 코딩되어 있으므로 대문자 R.*로 React를 사용할 수 있도록 해야 합니다.

# `as` 연산자 (The `as` operator)

타입 표명 작성 방법을 회상해봅시다.

```ts
var foo = <foo>bar;
```

여기서 변수 `bar`의 타입을 `foo`라고 주장하고 있습니다.  
TypeScript도 타입 표명을 위해 꺾쇠 괄호를 사용하기 때문에 JSX의 특정 구문 파싱에는 몇가지 어려움이 있습니다.   
결과적으로 TypeScript는 `.tsx` 파일에 꺾쇠 괄호 타입 표명을 허용하지 않습니다.

이러한 `.tsx` 파일의 기능 손실을 채우기 위해 새로운 타입의 표명 연산자가  추가되었습니다: `as`.  
위 예제는 쉽게 `as` 연산자로 다시 작성할 수 있습니다.

```ts
var foo = bar as foo;
```

`as` 연산자는 `.ts`와 `.tsx` 파일 모두에서 사용할 수 있으며 다른 타입 표명 스타일과 똑같이 동작합니다.

# 타입 검사 (Type Checking)

JSX 타입 검사를 이해하기 위해서는 먼저 내장 요소와 값-기반 요소 사이의 차이를 이해해야 합니다.  
JSX 표현식 `<expr />`이 주어지면 `expr`은 원래 환경에 내장된 것을 참조할 수 있습니다 (예: DOM 환경의 `div` 또는 `span`) 또는 직접 작성한 사용자 정의 구성 요소를 참조할 수 있습니다.

이것이 중요한 두 가지 이유가 있습니다:

1. React의 경우, 내장 요소는 문자열 (`React.createElement("div")`)로 생성되는 반면 생성한 컴포넌트는 (`React.createElement(MyComponent)`)가 아닙니다.
2. JSX 요소에서 전달되는 속성의 타입은 다르게 보여야합니다.  
  내장 요소 속성은 *본질적으로* 알려져야 하는 반면에 컴포넌트는 자체 속성 집합을 지정하기를 원할 수 있습니다.

TypeScript는 이러한 것들을 구분하기 위해 [React와 같은 컨벤션](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components)을 사용합니다.  
내장 요소는 항상 소문자로 시작하고 값-기반 요소는 항상 대문자로 시작합니다.

## 내장 요소 (Intrinsic elements)

내장 요소는 `JSX.IntrinsicElements`라는 특수한 인터페이스에서 볼 수 있습니다.
기본적으로 이 인터페이스가 지정되지 않으면 모든 내장 요소에 타입 검사는 하지 않습니다.  
다만 이 인터페이스가 *존재*하는 경우, 내부 요소의 이름은 `JSX.IntrinsicElements` 인터페이스의 프로퍼티로서 참조됩니다.

예를 들어:

```ts
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // 좋아요
<bar />; // 오류
```

위의 예제에서 `<foo />`는 잘 동작하지만 `<bar />`는`JSX.IntrinsicElements`에 지정되지 않았기 때문에 오류가 발생합니다.

> 참고: `JSX.IntrinsicElements`에서 다음과 같이 catch-all 문자열 indexer를 지정할 수도 있습니다:
>```ts
>declare namespace JSX {
>    interface IntrinsicElements {
>        [elemName: string]: any;
>    }
>}
>```

## 값-기반 요소 (Value-based elements)

값 기반 요소는 스코프에 있는 식별자로 간단히 조회됩니다.

```ts
import MyComponent from "./myComponent";

<MyComponent />; // 좋아요
<SomeOtherComponent />; // 오류
```

값 기반 요소를 정의하는 방법에는 두 가지가 있습니다:

1. 무상태 함수형 컴포넌트 (SFC)
2. 클래스 컴포넌트

이 두 가지 타입의 값 기반 요소는 JSX 표현식에서 구분할 수 없기 때문에 일단 오버로드 해석을 사용하여 무상태 함수형 컴포넌트로 표현식을 해결하려고 합니다.    
프로세스가 성공하면 선언에 대한 표현식을 해결합니다.  
만약 SFC로 해결하지 못한다면 클래스 컴포넌트로 해결합니다.  
만약 실패한다면 오류를 보고합니다.

### 무상태 함수형 컴포넌트(Stateless Functional Component)

이름에서 알 수 있듯이 컴포넌트는 첫 번째 인수가 `props` 객체인 JavaScript 함수로 정의됩니다.  
반환 타입은 `JSX.Element`에 할당할 수 있도록 강제합니다.

```ts
interface FooProp {
  name: string;
  X: number;
  Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name=prop.name />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

SFC는 단순히 JavaScript 함수이기 때문에 여기서도 함수 오버로드를 활용할 수 있습니다.

```ts
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element;
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  ...
}
```

### 클래스 컴포넌트 (Class Component)

클래스 컴포넌트의 타입을 제한할 수 있습니다.  
하지만 이를 위해 새로운 두 가지를 도입해야 합니다: *요소 클래스 타입*과 *요소 인스턴스 타입*

`<Expr />`에 주어진 *요소 클래스 타입*은 `Expr`입니다.  
따라서 위 예제의 `MyComponent`가 ES6 클래스라면 이 클래스가 그 클래스 타입이 될 것입니다.  
만일 `MyComponent`가 팩토리 함수라면 클래스 타입이 그 함수가 될 것입니다.

한 번 클래스 타입이 설정되면 인스턴스 타입은 클래스 타입의 호출 서명과 구조 서명의 반환 타입 유니온에 따라 결정됩니다.  
다시 ES6 클래스의 경우, 인스턴스 타입은 해당 클래스의 인스턴스 타입이 되고 팩토리 함수의 경우 해당 함수에서 반환되는 값의 타입이 됩니다.

```ts
class MyComponent {
  render() {}
}

// 구조 서명 사용
var myComponent = new MyComponent();

// 요소 클래스 타입 => MyComponent
// 요소 인스턴스 타입 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// 호출 서명 사용
var myComponent = MyFactoryFunction();

// 요소 클래스 타입 => 팩토리 함수
// 요소 인스턴스 타입 => { render: () => void }
```

요소 인스턴스 타입이 흥미로운 이유는 `JSX.ElementClass`에 할당되어야 하며 그렇지 않을 경우 오류가 발생하기 때문입니다.  
기본적으로 `JSX.ElementClass`는 `{}`이지만 JSX의 사용을 적절한 인터페이스에 맞는 타입으로 제한하도록 확장할 수 있습니다.

```ts
declare namespace JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

<MyComponent />; // 좋아요
<MyFactoryFunction />; // 좋아요

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // 오류
<NotAValidFactoryFunction />; // 오류
```

## 속성 타입 검사 (Attribute type checking)

속성 타입 검사를 하는 첫 번째 단계는 *요소 속성 타입*을 결정하는 것입니다.  
이것은 내장 요소와 값 기반 요소 간에 약간의 차이가 있습니다.

내장 요소의 경우 `JSX.IntrinsicElements` 프로퍼티의 타입입니다.

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// 'foo'에 대한 요소 속성 타입은 '{bar?: boolean}'입니다.
<foo bar />;
```

값 기반 요소의 경우 조금 더 복잡합니다.  
이전에 결정된 *요소 인스턴스 타입*의 프로퍼티 타입에 따라 결정됩니다.  
사용할 프로퍼티는 `JSX.ElementAttributesProperty`에 의해 결정됩니다.  
단일 프로퍼티로 선언해야 합니다.  
그런 다음 해당 프로퍼티의 이름이 사용됩니다.

```ts
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // 사용할 프로퍼티 이름을 지정합니다
  }
}

class MyComponent {
  // 요소 인스턴스 타입에 대한 프로퍼티를 지정합니다
  props: {
    foo?: string;
  }
}

// 'MyComponent'의 요소 속성 타입은 '{foo?: string}'입니다
<MyComponent foo="bar" />
```

요소 속성 타입은 JSX에서 속성을 타입을 확인하는 데 사용됩니다.  
선택적 프로퍼티와 필수 프로퍼티가 지원됩니다.

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // 좋아요
<foo requiredProp="bar" optionalProp={0} />; // 좋아요
<foo />; // 오류, requiredProp이 없습니다
<foo requiredProp={0} />; // 오류, requiredProp이 문자열이어야 합니다.
<foo requiredProp="bar" unknownProp />; // 오류, unknownProp이 존재하지 않습니다.
<foo requiredProp="bar" some-unknown-prop />; // 좋아요, 'some-unknown-prop' 은 유효한 식별자가 아니기 때문입니다.
```


> 참고: 속성 이름이 유효한 JS 식별자 (예 :`data- *`속성)가 아닌 경우 요소 속성 타입에서 속성 이름을 찾을 수 없으면 오류로 간주되지 않습니다.

전개 연산자도 작동합니다:

```JSX
var props = { requiredProp: "bar" };
<foo {...props} />; // 좋아요

var badProps = {};
<foo {...badProps} />; // 오류
```

## 하위 타입 검사 (Children Type Checking)

2.3 버전에서 *하위* 타입 검사를 도입했습니다.  
*하위(children)*는 요소 타입 검사에서 결정된 *요소 속성 타입*의 프로퍼티 입니다.  
`JSX.ElementAttributesProperty`를 사용하여 *props* 이름을 결정하는 것과 마찬가지로 `JSX.ElementChildrenAttribute`를 사용하여 하위 이름을 결정합니다.  
`JSX.ElementChildrenAttribute`는 단일 프로퍼티로 선언해야합니다.

```ts
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {};  // 사용할 하위 이름을 지정하세요
  }
}
```

하위의 타입을 명시적으로 지정하지 않는다면 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)의 기본 타입을 사용합니다.

```ts
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>props.children</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

다른 속성과 마찬가지로 *하위* 타입을 지정할 수 있습니다.  
이렇게 하면 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)에서 기본 타입을 덮어쓰게 됩니다.

```ts
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        this.props.children
      </h2>
    )
  }
}

// 좋아요
<Component>
  <h1>Hello World</h1>
</Component>

// 오류: 하위 타입은 JSX.Element의 배열이 아닌 JSX.Element 타입입니다.
<Component>
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// 오류: 하위 타입은 JSX.Element 또는 string의 배열이 아닌 JSX.Element 타입입니다.
<Component>
  <h1>Hello</h1>
  World
</Component>
```

# JSX 결과 타입 (The JSX result type)

기본적으로 JSX 표현식의 결과 타입은 `any`로 분류됩니다.  
`JSX.Element` 인터페이스를 지정하여 사용자 정의 타입을 사용할 수 있습니다.  
그러나 이 인터페이스에서 JSX의 요소, 속성 또는 하위 항목에 대한 타입 정보를 찾을 수 없습니다.  
이것은 블랙박스 입니다.

# 표현식 포함하기 (Embedding Expressions)

JSX는 중괄호 (`{ }`)로 표현식을 감싸고 태그 사이에 표현식을 삽입할 수 있게합니다.

```JSX
var a = <div>
  {["foo", "bar"].map(i => <span>{i / 2}</span>)}
</div>
```

위의 코드는 문자열을 숫자로 나눌 수 없으므로 오류가 발생합니다.

출력은 `preserve` 옵션을 사용할 때 다음과 같습니다:

```JSX
var a = <div>
  {["foo", "bar"].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

# 리액트 통합 (React integration)

React와 함께 JSX를 사용하려면 [React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)을 사용해야 합니다.  
이러한 typings은 React와 함께 사용하기에 적합하도록 `JSX` 네임스페이스를 적절하게 정의합니다.

```ts
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // 좋아요
<MyComponent foo={0} />; // 오류
```
