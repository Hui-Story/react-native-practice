[TOC]

# 할일 목록 만들기 II

> TodoApp 프로젝트를 진행하면서 새롭게 알게된 지식 정리 2



## FlatList 를 사용해 여러 항목 렌더링

> data Props 를 활용하여 renderItem 으로 각 원소들 데이터를 가리키는 뷰를 보여줌

**components/TodoList.js**

```js
import React from 'react';
import {FlatList, View, Text, StyleSheet} from 'react-native';
import TodoItem from './TodoItem';

function TodoList({todos}) {
  return (
    <FlatList
      ItemSeparatorComponent={() => <View style={styles.separator} />}
      style={styles.list}
      data={todos}
      renderItem={({item}) => (
        <TodoItem id={item.id} text={item.text} done={item.done} />
      )}
      keyExtractor={item => item.id.toString()}
    />
  );
}

const styles = StyleSheet.create({
  list: {
    flex: 1,
  },
  separator: {
    backgroundColor: '#e0e0e0',
    height: 1,
  },
});

export default TodoList;
```

- **ItemSeparatorComponent** : 각 아이템 간의 커스텀 (사이에 구분선 보여주기)
- **keyExtractor** : 각 항목의 고유 값을 추출해주는 함수 (데이터의 id 값, 문자열 타입)



## react-native-vector-icons (벡터 아이콘 사용하기)

> 폰트 또는 SVG 를 사용해 크기가 조정돼도 아이콘이 흐려지거나 깨지지 않음

### react-native-vector-icons 라이브러리 설치

```bash
$ npm install react-native-vector-icons
```

### 안드로이드에 react-native-vector-icons 적용하기

**android/app/build.gradle** 맨 아래에 추가

```
(...)
apply from: file("../../node_modules/react-native-vector-icons/fonts.gradle")
```

### 아이콘 검색 및 적용

> **아이콘 검색** : https://oblador.github.io/react-native-vector-icons/

```js
import Icon from 'react-native-vector-icons/MaterialIcons';

<Icon name="delete" size={32} color="red" />
```

**components/TodoItem.js**

```js
import React from 'react';
import {View, Text, StyleSheet, Image, TouchableOpacity} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';

function TodoItem({id, text, done, onToggle}) {
  return (
    <View style={styles.item}>
      <TouchableOpacity onPress={() => onToggle(id)}>
        <View style={[styles.circle, done && styles.filled]}>
          {done && (
            <Image
              source={require('../assets/icons/check_white/check_white.png')}
            />
          )}
        </View>
      </TouchableOpacity>
      <Text style={[styles.text, done && styles.lineThrough]}>{text}</Text>
      {done ? (
        <Icon name="delete" size={32} color="red" />
      ) : (
        <View style={styles.removePlaceholder} />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  (...)
});

export default TodoItem;
```



## Alert 창 띄우기

> 리액트 네이티브에 내장되어 있는 Alert 라는 API 를 통해 구현

- **Alert.alert** 함수의 파라미터는 '제목, 내용, 버튼 배열, 옵션 객체' 순서
- **style** 은 'cancel, default, destructive' 값을 설정할 수 있는데 iOS 에서만 작동
- 안드로이드는 버튼에 스타일이 적용되지 않음 (Alert 처럼 보이는 컴포넌트를 직접 제작해야 함)
- **cancelable** 값을 통해 안드로이드에서 박스 바깥이나 Back 버튼을 눌렀을 때 Alert 가 닫히도록 설정
- **onDismiss** : Alert 가 닫힐 때 호출되는 함수

**components/TodoItem.js**

```js
import React from 'react';
import {
  View,
  Text,
  StyleSheet,
  Image,
  TouchableOpacity,
  Alert,
} from 'react-native';
import Icon from 'react-native-vector-icons/MaterialIcons';

function TodoItem({id, text, done, onToggle, onRemove}) {
  const remove = () => {
    Alert.alert(
      '삭제',
      '정말로 삭제하시겠어요?',
      [
        {text: '취소', onPress: () => {}, style: 'cancel'},
        {
          text: '삭제',
          onPress: () => {
            onRemove(id);
          },
          style: 'destructive',
        },
      ],
      {
        cancelable: true,
        onDismiss: () => {},
      },
    );
  };

  return (
    <View style={styles.item}>
      <TouchableOpacity onPress={() => onToggle(id)}>
        <View style={[styles.circle, done && styles.filled]}>
          {done && (
            <Image
              source={require('../assets/icons/check_white/check_white.png')}
            />
          )}
        </View>
      </TouchableOpacity>
      <Text style={[styles.text, done && styles.lineThrough]}>{text}</Text>
      {done ? (
        <TouchableOpacity onPress={remove}>
          <Icon name="delete" size={32} color="red" />
        </TouchableOpacity>
      ) : (
        <View style={styles.removePlaceholder} />
      )}
    </View>
  );
}

const styles = StyleSheet.create({
  (...)
});

export default TodoItem;
```



## AsyncStorage 로 앱이 꺼져도 데이터 유지하기

> 리액트 네이티브에서 사용할 수 있는 key-value 형식의 저장소 (localStorage 와 비슷하며, 비동기로 작동)

### AsyncStorage 설치

```bash
$ npm install @react-native-async-storage/async-storage
```

### AsyncStorage 적용

> https://react-native-async-storage.github.io/async-storage/docs/install/

#### 1. 불러오기

```js
import AsyncStorage from '@react-native-async-storage/async-storage';
```

#### 2. 저장하기

> setItem 메서드 사용

```js
const save = async () => {
  try {
    await AsyncStorage.setItem('key', 'value');
  } catch (e) {
    // 오류 예외 처리
  }
}
```

> 값을 저장할 때는 문자열 타입. 객체 및 배열 타입을 저장하려면 JSON.stringify 함수를 사용

```js
await AsyncStorage.setItem('todos', JSON.stringify(todos));
```

#### 3. 불러오기

> getItem 메서드 사용

```js
const load = async () => {
  try {
    const value = await AsyncStorage.getItem('key');
    // value를 사용하는 코드
  } catch (e) {
    // 오류 예외 처리
  }
}
```

> 객체 및 배열을 불러오려면 JSON.parse 함수를 사용해 문자열을 JSON 으로 변환해야 함

```js
const rawTodos = await AsyncStorage.getItem('todos');
const todos = JSON.parse(rawTodos);
```

#### 4. 초기화하기

> clear 메서드 사용

```js
const clearAll = async () => {
  try {
    await AsyncStorage.clear();
  } catch (e) {
    // 오류 예외 처리
  }
}
```

### AsyncStorage 코드 추상화 및 useEffect 활용

**storages/todosStorage.js**

```js
import AsyncStorage from '@react-native-async-storage/async-storage';

const key = 'todos';

const todosStorage = {
  async get() {
    try {
      const rawTodos = await AsyncStorage.getItem(key);

      if (!rawTodos) {
        // 저장된 데이터가 없으면 사용하지 않음
        throw new Error('No saved todos');
      }

      const savedTodos = JSON.parse(rawTodos);
      return savedTodos;
    } catch (e) {
      throw new Error('Failed to load todos');
    }
  },
  async set(data) {
    try {
      await AsyncStorage.setItem(key, JSON.stringify(data));
    } catch (e) {
      throw new Error('Failed to save todos');
    }
  },
};

export default todosStorage;
```

**App.js**

```js
(...)
import todosStorage from './storages/todosStorage';
 
function App() {
  const today = new Date();
  const [todos, setTodos] = useState([
    {id: 1, text: '작업환경 설정', done: true},
    {id: 2, text: '리액트 네이티브 기초 공부', done: false},
    {id: 3, text: '투두리스트 만들어보기', done: false},
  ]);

  // AsyncStorage 로 todos 불러오기
  useEffect(() => {
    todosStorage.get().then(setTodos).catch(console.error);
  }, []);

  // AsyncStorage 로 todos 저장
  useEffect(() => {
    todosStorage.set(todos).catch(console.error);
  }, [todos]);
  
  (...)
```

### 안드로이드에서 AsyncStorage 최대 용량 설정

> 안드로이드는 기본적으로 6MB 로 설정되어 있으며, iOS 는 최대 용량이 지정되어 있지 않음

**android/gradle.properties** 에 코드 추가

```
(...)
AsyncStorage_db_size_in_MB=10
```

### AsyncStorage 의 제한

> AsyncStorage 는 설정이 매우 간편하고 사용법도 쉽지만, 단점도 있음

- 데이터의 규모가 커지면 성능이 떨어지며, 문자열 타입으로만 저장할 수 있어 속도가 느려짐

  > 대안으로는 realm 와 react-native-sqlite-storage 가 있음

  - 안드로이드의 AsyncStorage 는 이미 SQLite 를 사용하긴 하지만, **인덱싱 기능을 지원**받을 수 있고 **더욱 다양한 방식으로 데이터를 저장하고 조회**할 수 있음

- 검색 또는 정렬 기능이 지원되지 않음
