# Json Parse/Stringify
## Json 
Json 은 네트워크 데이터 전송방식의 표준 format 으로 사용되며
'키-값' 쌍으로 이루어진 데이터 객체를 전달하기 위한 목적이 있습니다.

## Json Parse
String 문자열 데이터를 객체로 변환해주는 메서드입니다.    
아래의 코드를 실행시켜봅시다.

```javascript
const json = '{"result":true, "count":42}';
const obj = JSON.parse(json);

console.log(`json Type: ${typeof json}`);
console.log(obj.count);
console.log(obj.result);
console.log(`obj Type: ${typeof obj}`);
```

string 이였던 타입이 json 으로 변환된 것을 확인할 수 있습니다.

```
json Type: string
42
true
obj Type: object
```

## Json Stringify
```Json.Parse()``` 와 반대로 object 를 string 으로 변환해주는 메서드입니다.

```javascript
const object = { x: 5, y: 6};
const string = JSON.stringify(object);
console.log(`object Type: ${typeof object}`);
console.log(string);
console.log(`string Type: ${typeof string}`);
```

위의 코드를 실행시켜 결과를 확인해보면 아래와 같이 정상적으로 변환된 것을 확인할 수 있습니다.

```
object Type: object
{"x":5,"y":6}
string Type: string
```

