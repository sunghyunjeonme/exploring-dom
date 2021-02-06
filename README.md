# DOM을 깨우치다

- 코디 린들리 지음 / 안재우 옮김
- 해당 저서를 읽고 주요 내용을 정리하는 저장소

<center>

![dom](images/dom.png)

</center>

브라우저가 페이지를 렌더링하려면 먼저 DOM 및 CSSSOM 트리를 생성해야 한다.

# 1. 노드 개요 및 DOM

**브라우저에 의해 문서가 Parse 되는 과정**

- 바이트 -> 문자 -> 토큰 -> 노드 -> 객체 모델 (DOM/CSSOM)
- HTML 마크업은 DOM, CSS 마크업은 CSSOM으로 변환되는 것이다.
- DOM과 CSSOM은 서로 독립적인 데이터 구조이다.

아래의 HTML 문서는 다음의 과정을 거쳐 DOM으로 형성된다.

```html
<!DOCTYPE html>
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1" />
    <link href="style.css" rel="stylesheet" />
    <title>Critical Path</title>
  </head>
  <body>
    <p>Hello <span>web performance</span> students!</p>
    <div><img src="awesome-photo.jpg" /></div>
  </body>
</html>
```

![dom-process](./images/dom-process.png)

1. **변환** : 브라우저는 서버에서 HTML의 원시 바이트를 읽고 지정된 파일 인코딩(예 : UTF-8)에 따라 개별 문자로 변환한다.
2. **토큰화** : 브라우저는 W3C HTML5 표준에 따라서 변환된 문자를 의미 있는 단위(예 : `"<html>"`, `"<body>"`)로 나누고 고유한 의미를 갖는 토큰으로 변환한다.
3. **Lexing** : 변환된 토큰은 프로퍼티와 메서드를 갖는 객체로 변환된다.
4. **DOM 형성** : 객체로 변환되기 전 HTML 파일은 들여쓰기를 통해 시각적으로 태그 간의 계층 구조를 표현했다. 하지만 객체 형태로 HTML 파일이 변환됨에 따라서 해당 계층 간의 구조를 Tree 자료 구조로 표현한다.

![dom-tree](images/dom-tree.png)

## DOM

- DOM은 HTML을 의미 있는 객체, 즉 Node 객체 형태로 바꿔서 자바스크립트가 추가적인 기능을 할 수 있도록 제공하는 녀석.
- DOM은 자바스크립트 Node 객체의 계층화된 트리이다.
- 브라우저는 HTML 코드를 해석해서 트리 형태로 구조화된 노드들을 가지고 있는 문서(DOM)을 생성한다.
- DOM을 객체라는 동물이 사는 생태계 환경에 비유해서 생각했다.

## DOM 목적

- JavaScript를 사용해서 각 요소를 핸들링하기 위한 인터페이스를 제공하는 것이다. (DOM API)
  - document 객체는 DOM의 트리의 진입점과 동시에 문서 전체를 의미하는 Document 노드이다.
  - document 객체를 통해 DOM에서 제공하는 인터페이스로 각 요소를 interactive 하도록 만들 수 있다.

## DOM 트리

- Node 객체로 이루어진 트리 구조의 형태를 DOM 트리라고 한다.

## Node

![node-process](images/node-process.png)

- Node 객체는 DOM에서 시조와 같은 역할을 한다.
- 즉, 모든 DOM은 Node로부터 속성과 메서드를 상속받는다.
  - DOM에서는 Node에서 제공하는 모든 [프로퍼티](https://developer.mozilla.org/ko/docs/Web/API/Node)를 사용할 수 있다.

아래는 모든 노드 유형을 출력하는 코드이다.

```js
for (let key in Node) {
  console.log(key, " = " + Node[key]);
}
// NodeType
ELEMENT_NODE = 1;
ATTRIBUTE_NODE = 2;
TEXT_NODE = 3;
CDATA_SECTION_NODE = 4;
ENTITY_REFERENCE_NODE = 5;
ENTITY_NODE = 6;
PROCESSING_INSTRUCTION_NODE = 7;
COMMENT_NODE = 8;
DOCUMENT_NODE = 9;
DOCUMENT_TYPE_NODE = 10;
DOCUMENT_FRAGMENT_NODE = 11;
NOTATION_NODE = 12;
```

## DOM 상속 관계

객체 상속의 시작은 최상위 조상인 Object.prototype로부터 시작된다.

```js
Object.prototype;
console.dir(EventTarget); // __proto__: Object
console.dir(Node); // __proto__: EventTarget
console.dir(Document); // __proto__: Node
```

- EventTarget 객체는 Object 객체를 상속받아 확장한다.
- Node 객체는 EventTarget 객체를 상속받아 확장한다.
- Document 객체는 Node 객체를 상속받아 확장한다.

cf. Object < EventTarget < Node < Element < HTMLElement < HTMLAnchorElement

cf. Object < EventTarget < Node < Document < HTMLDocument

## Element 노드 및 Text 노드 생성하기

브라우저는 문서를 초기 load 시 HTML 문서를 기반으로 노드와 트리를 생성한다. 하지만 아래와 같은 방법으로도 노드를 생성할 수 있다.

**document 객체 메서드 활용하기**

- createElement (tagName)
  - document.createElement('div')
- createTextNode (text)
  - document.createTextNode('안녕하세요.')

**문자열을 사용하여 DOM에 Element 및 Text 노드를 생성 및 추가**

Element 노드에서 제공하는 프로퍼티와 메서드이다.

- innerHTML
  - innerHTML은 무겁고 비싼 대가를 치르는 HTML 파서를 호출하므로 innerHTML 계열의 사용을 삼가해야 한다.
  - innerHTML 프로퍼티를 사용할 때, 복합 대입 연산자 사용을 지양해야 한다. [Why is "element.innerHTML" += bad code?](https://stackoverflow.com/questions/11515383/why-is-element-innerhtml-bad-code)
- outerHTML
  - outerHTML는 지정한 노드의 바깥 쪽의 Element를 명시하는 것 같지만 실질적으로는 해당 노드를 나타낸다.
- textContent
  - document.getElementById('para').textContent = 'developer';
- insertAdjacentHTML(position, string)
  - 세밀하게 특정 위치에 DOM tree 안에 원하는 node들을 추가할 수 있다.
  - beforbegin
  - afterbegin
  - beforeend
  - afterend

위의 속성을 사용하여 JavaScript 문자열로부터 노드를 생성한 다음, 바로 DOM에 추가한다.

## 노드 개체를 DOM에 추가하기

- appendChild(Node)
  - 인자로 전달한 노드를 마지막 자식 요소로 DOM 트리에 추가한다.
- insertBefore(newNode, referenceNode)
  - 첫 번째 인자에 삽입될 노드와 두 번째 인자에는 해당 노드를 삽입하고자 하는 문서 내의 참조 노드이다.

## 노드 개체를 DOM에서 제거하기

- removeChild()

```js
let divA = document.getElementById("a");
divA.parentNode.removeChild(divA);
```

- replaceChild()

```js
let divA = document.getElementById("b");
let newSapn = document.createElement("span");
divA.parentNode.replaceChild(newSpan, divA);
```

- remove ()
  - [DOM 4](https://www.w3.org/TR/2015/REC-dom-20151119/#childnode)에서 새롭게 지원되는 메서드
  - parentNode를 지정하지 않고도 해당 Element를 제거할 수 있다.

제거하거나 바꾸는 대상이 무엇인지에 따라 innerHTML, outerHTML, textContent 속성에 빈 문자열을 주는 것이 더 빠르고 쉬울 수도 있다. 하지만 존재하지 않는 요소에 값을 할당하지 않는 것이므로, 브라우저의 `메모리 누수`가 발생할 수 있으므로 조심해야 한다.

### cf. innerHTML과 appendChild ()의 차이

- innerHTML은 요소 안의 tag가 교체되는 것이고
- appendChild ()는 요소 안의 tag를 그대로 두고 맨 뒤에 추가되는 것이다.

## HTMLCollection과 NodeList (유사 배열)

HTMLCollection과 NodeList는 유사 배열이라는 공통점을 갖고 있는 반면에 차이점도 존재한다.

- HTMLCollection
  - document.scripts
  - document.body.children
  - docment.getElementsByTagName
  - document.getElementsByClassName

위의 메서드를 사용하면 HTMLCollection을 return한다.

- NodeList
  - document.querySelectorAll
  - element.childNodes

위의 메서드를 사용하면 NodeList를 return한다. (노드의 모음)

- forEach 메서드를 지원한다. (브라우저의 지원 여부 확인해야 함)
  - [Why doesn`t nodelist have forEach ?](https://stackoverflow.com/questions/13433799/why-doesnt-nodelist-have-foreach/39133264#39133264)

## DOM 내의 노드 탐색

- parentNode
- firstChild
- lastChild
- nextSibling

위의 프로퍼티를 사용해서 DOM을 탐색하게 되면 Text 노드, Comment 노드 등이 관계에 포함되기 때문에 요소 탐색에 어려움을 겪는다.

- firstElementChild
- lastElementChild
- nextElementSibling
- previousElementSibling
- children
- parentElement

다음과 같은 프로퍼티를 사용하면 Element 노드만 탐색할 수 있다.

# 2. Document 노드

HTMLDocument 생성자(document로부터 상속)는 DOM 내에 DOUCUMENT_NODE(에:window.document)를 생석한다.

```js
console.log(document.constructor); // HTMLDocument
console.log(document.nodeType); // 9
```
