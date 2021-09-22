원본: Hello Python

뒤집기: nohtyP olleH

### 방법 1 - reversed()

reversed()는 **반대방향으로 순회하는 객체를 리턴**합니다. join()을 통해 리턴된 객체의 데이터를 하나의 string으로 만들어주면 됩니다.

```python
string = 'Hello Python'
reversed_string = ''.join(reversed(string))
print(reversed_string)
```

### 방법 2 - slice()

slice 에서 각각의 항목은 **\[start:stop:end\]** 를 의미합니다.

```python
string = 'Hello Python'
reversed_string = string[::-1]
print(reversed_string)
```