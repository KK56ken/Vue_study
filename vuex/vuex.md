###### tags: `Vue` `Vuex`

# Vuexについて

## 必要性
コンポーネントが多くなり末端のコンポーネントどうしでデータをやり取りするとき単調かしてしまうのでそれを防ぐために作られた。

vuexが無いと$emitとpropsでデータを渡すしかない
![](https://i.imgur.com/xNSwFdC.png)

**グローバル変数**を定義するものだと思えばOK

## 使い方
### 導入
```shell=
$ npm install vuex
```

### 定義とstate
store.jsの中身
```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    }
})

```
**stateはグローバル変数を定義する場所**

### 呼び出し方
test.vue
```javascript=

<script>
export default {
 computed:{
     count(){
         return this.$store.state.count
     }
 }
}
</script>

```
stateの値をstore.js以外からアクセスする場合は、**this.$store.state** まで同じ


### getters
vuex用の算出プロパティ(vueのcomputedプロパティと同じ)

store.jsの中身
```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    },
    getters:{
        doubleCount: state => state.count * 2;
        tripleCount: state => state.count * 2;
    }
})

```
test.vue
```javascript=

<script>
export default {
 computed:{
     doubleCount(){
         return this.$store.getters.doubleCount; 
     },
     tripleCount(){
         return this.$store.getters.tripleCount
     }
 }
}
</script>

```

このように定義するとvuex側で処理を定義できる
複数のページやコンポーネントで呼び出せるのがメリット
さらに、stateを参照しないことによりバグを起こりにくくしている

#### mapGettersヘルパー

##### 配列の場合
test.vue
```javascript=

<template>
    <div>
        {{ doubleCount }}
        {{ tripleCount }}
    </div>

</template>

<script>
import { mapGetters } from "vuex";

export default {
    computed: mapGetters(["doubleCount","tripleCount"])
}
</script>

```

##### オブジェクトの場合
あまり使わないので覚えなくていい

test.vue
```javascript=
<template>
    <div>
        {{ myComponentDoubleCount }}
    </div>

</template>


<script>
import { mapGetters } from "vuex";

export default {
    computed: myComponentDoubleCount: "doubleCount"
}
</script>

```
オブジェクトの場合は呼び出す名前を自分で決めることができる

##### computedにmapGetters以外を定義するには
スプレッド演算子を使うことによって定義することができる(vueの機能ではなくES6の機能)
mapGettersの前に.を三つ付ける

test.vue
```javascript=

<template>
    <div>
        {{ doubleCount }}
    </div>

</template>

<script>
import { mapGetters } from "vuex";

export default {
    computed:{
        ...mapGetters(["doubleCount","tripleCount"])
    }
}
</script>

```
### mutations
mutationsでしかstateの内容を変更しないようにする（ただし、stateを参照すれば変更できてしまう）
stateを変える場所を一つに絞る
そうすることで、データを追跡しやすくし、予測しやすくするために使う
store.jsの中身
```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    },
    getters:{
        doubleCount: state => state.count * 2;
        tripleCount: state => state.count * 2;
    },
    mutations: {
        increment(state, number){
            state.count += number
        }
    }
})

```
mutationsに定義するメソッドの第一引数は**state**を指定する
test.vue
```javascript=

<template>
    <div>
        {{ $store.state.count }}
        <button @click="increment">+1</button>
    </div>
    

</template>

<script>
export default {
    methods:{
       increment(){
           this.$store.commit('increment', 2)
       }
    }
}
</script>

```
mutationは**commit**で呼び出す

#### mapMutationsヘルパー
mapGettersヘルパーのmapMutations版
違いはメソッドに書くことと引数はtemplate側に記入すること

test.vue
```javascript=
<template>
    <div>
        {{ $store.state.count }}
        <button @click="increment(2)">+1</button>
    </div>
    

</template>

<script>
export default {
    methods:{
        ...mapMutations(["increment", "decrement"])
    }
}
</script>

```

```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    },
    getters:{
        doubleCount: state => state.count * 2;
        tripleCount: state => state.count * 2;
    },
    mutations: {
        increment(state, number){
            state.count += number
        },
        decrement(state, number){
            state.count -= number
        }
    }
})

```
#### mutationは同期的処理しか書けない
使えない理由は、プログラムの動きを予測できなくなってしまうから
mutationは予測可能にする必要がある

### action
actionsは非同期処理をするときに使用する（同期的処理も書ける）

test.vue
```javascript=
<template>
    <div>
        {{ $store.state.count }}
        <button @click="increment">+1</button>
    </div>
    

</template>

<script>
export default {
    methods:{
        increment(){
           this.$store.dispatch("increment", 2);     
        }
    }
}
</script>

```
mutationはcommitだったがactionは**dispatch**なので注意

```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    },
    getters:{
        doubleCount: state => state.count * 2;
        tripleCount: state => state.count * 2;
    },
    mutations: {
        increment(state, number){
            state.count += number;
        },
        decrement(state, number){
            state.count -= number;
        }
    },
    actions:{
        increment(context, number){
            context.commit('increment', number);
        },
        incrementAsync(context){
            setTimeout(()=>{
                context.commit('increment', number);
            },1000)
        }
    }
})

```
contextには様々なものが入っている(commit,state,getters,dispatch)など
```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2
    },
    mutations: {
        increment(state, number){
            state.count += number;
        },
        decrement(state, number){
            state.count -= number;
        }
    },
    actions:{
        increment({ commit }, number){ 　 //変更点
            commit('increment', number);　　//変更点
        },
    }
})
```
省略して書くこともできる

全部actionでtemplateの方はdipatchしか使わないのがベストプラクティスの可能性がある

#### mapActionsヘルパー


test.vue
```javascript=
<template>
    <div>
        {{ $store.state.count }}
        <button @click="increment(2)">+1</button>
        <button @click="decrement(2)">-1</button>
    </div>
    

</template>

<script>
import { mapActions } from "vuex";

export default {
    methods:{
        ...mapActions(["increment","decrement"])
    }
}
</script>

```

### 全体の流れ
componentsでdispatchを呼びactionsに行く
actionsでcommitを呼び出しmutationsに渡す
mutationsでstateの値を変える
gttersやそのまま渡したりすることでcomponentsに反映させる
![](https://i.imgur.com/QpfdxuW.png)

### 双方向バインディング(v-model)をvuexで使う場合

#### v-modelを使わない場合
```javascript=
import Vue from "vue";
import Vuex from "vuex";

Vue.use(Vuex);

new Vuex.Store({
    state:{
        count:2,
        message: ''
    },
    getters:{
        message: state => state.message 
    }
    mutations: {
        increment(state, number){
            state.count += number;
        },
        decrement(state, number){
            state.count -= number;
        },
        updateMessage(state, newMessage){
            state.message = newMessage;
        }
    },
    actions:{
        increment({ commit }, number){ 　 //変更点
            commit('increment', number);　　//変更点
        },
        updateMessage({ commit }, newMessage){
            commit("updateMessage", newMessage)
        }
    }
})
```

test.vue
```javascript=
<template>
    <div>
        {{ $store.state.count }}
        <input type="text" :value="message" @input="updateMessage">
        <p>{{ message }}</p>
    </div>
</template>

<script>
import { mapActions } from "vuex";

export default {
    computed:{
        ...mapActions(["increment","decrement"])
        message(){
            retun this.$store.getters.message;
        }
    },
    methods:{
        updateMessage(e){
            this.$store.dispatch("updateMessage", e.target.value)
        }
    }
}
</script>

```
v-modelを分解して:valueと@inputを使う

#### v-modelを使う場合
computedプロパティはsetter的な使い方もできる
test.vue
```javascript=
<template>
    <div>
        {{ $store.state.count }}
        <input type="text" v-model="">
        <p>{{ message }}</p>
    </div>
</template>

<script>
import { mapActions } from "vuex";

export default {
    computed:{
        ...mapActions(["increment","decrement"])
        message: {
            get(){
                return this.$store.getters.message;
            },
            set(value){
                tihs.$store.dispatch("updateMessage", value);
            }
        }
    }
}
</script>
```

### vuexを機能ごとにファイルを分ける
vuexの肥大化を避けるためにファイルを分ける
今回はcountの機能を別のファイルにする
```
store/
    ┝ modules/
    │     └count.js
    │     
    │
    └ store.js

```
count.js
```javascript=
const state = {
    count: 2
};

const getters = {
    doubleCount: state => state.count * 2,
    tripleCount: state => state.count * 3
}
const mutations = {
    increment(state, number){
        state.count += number;
    },
    decrement(state, number){
        state.count -= number;
    }
}
const actions = {
    increment({ commit }, number){
        commit("increment", number);
    },
    decrement({ commit }, number){
        commit('decrement', number)
    }
};
export default{
    state,
    getters,
    mutations,
    actions
}
```
store.js
```javascript=
import Vue from "vue";
import Vuex from "vuex";
import count from "./modules/count"

Vue.use(Vuex);

export default new Vuex.Store({
    modules:{
        count
    }
})
```
#### 複数のmodulesに対応するには
count.js
```javascript=
const state = {
    count: 2
};
const getters = {
    doubleCount: state => state.count * 2,
    tripleCount: state => state.count * 3
};
const mutations = {
    increment(state, number){
        state.count += number;
    },
    decrement(state, number){
        state.count -= number;
    }
};
const actions = {
    increment({ commit }, number){
        commit("increment", number);
    },
    decrement({ commit }, number){
        commit('decrement', number)
    }
};
export default{
    namespaced: true, //変更点
    state,
    getters,
    mutations,
    actions
};
```
namespacedをtrueにすることによりスラッシュが付くようになる
例　count/doubleCount

この場合参照先が変わったのでエラーになる

##### 対処法(mapGettersヘルパーの場合)

test.vue
```javascript=

<script>
import { mapGetters } from "vuex";

export default {
 computed:{
     ...mapGetter("count",["doubleCount", "tripleCount"])
 }
}

</script>
```
mapGetterに第一引数を加えてあげることでエラーを回避できる
##### 対処法(mapGettersヘルパーでない場合)

test.vue
```javascript=

<script>
import { mapGetters } from "vuex";

export default {
 computed:{
     douleCount(){
         return this.$store.getters["count/doubleCount"]
     }
 }
}

</script>
```
### vue devtools
クロームの拡張機能
デバッグを容易にするので入れておくと便利
