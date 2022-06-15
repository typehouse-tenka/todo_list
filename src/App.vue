<template>
  <div id="app">
    <h2>新しい作業の追加</h2>
    <form class="add-form" v-on:submit.prevent="doAdd">
      <!-- コメント入力フォーム -->
      コメント <input type="text" ref="comment">
      <!-- 追加ボタンのモック -->
      <button type="submit">追加</button>
    </form>
    <label v-for="(label, key) in options" :key="key">
      <input type="radio"
        v-model="current"
        v-bind:value="label.value">{{ label.label }}
    </label>
    <br>
    <br>
    {{ computedTodos.length }} 件を表示中
    <table>
      <!-- テーブルヘッダー -->
      <thead>
        <tr>
          <th class="id">ID</th>
          <th class="comment">コメント</th>
          <th class="state">状態</th>
          <th class="button">-</th>
        </tr>
      </thead>
      <tbody>
        <!-- ここに <tr> で ToDo の要素を1行づつ繰り返し表示 -->
        <tr v-for="item in computedTodos" v-bind:key="item.id">
          <th>{{ item.id }}</th>
          <td>{{ item.comment }}</td>
          <td class="state">
            <!-- 状態変更ボタンのモック -->
            <button v-on:click="doChangeState(item)">
              {{ labels[item.state] }}
            </button>
          </td>
          <td class="button">
            <!-- 削除ボタンのモック -->
            <button v-on:click.ctrl="doRemove(item)">
              削除
            </button>
          </td>
        </tr>
      </tbody>
    </table>

    
  </div>
</template>

<script>
  var STORAGE_KEY = 'todos-vuejs-demo'
  var todoStorage = {
    fetch: function() {
      var todos = JSON.parse(
        localStorage.getItem(STORAGE_KEY) || '[]'
      )
      todos.forEach(function(todo, index) {
        todo.id = index
      })
      todoStorage.uid = todos.length
      return todos
    },
    save: function(todos) {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
    }
  }

  export default {
    data: function ()  {
      return {
        todos: [],
        options: [
          { value: -1, label: 'すべて' },
          { value: 0,  label: '作業中' },
          { value: 1,  label: '完了' }
        ],
        // 選択している options の value を記憶するためのデータ
        // 初期値を「-1」つまり「すべて」にする
        current: -1
      }
    },
    methods: {
      doAdd: function() {
        // ref で名前を付けておいた要素を参照
        var comment = this.$refs.comment
        // 入力がなければ何もしないで return
        if (!comment.value.length) {
          return
        }
        // { 新しいID, コメント, 作業状態 }
        // というオブジェクトを現在の todos リストへ push
        // 作業状態「state」はデフォルト「作業中=0」で作成
        this.todos.push({
          id: todoStorage.uid++,
          comment: comment.value,
          state: 0
        })
        // フォーム要素を空にする
        comment.value = ''
      },
      doChangeState: function(item) {
        item.state = item.state ? 0 : 1
      },
      // 削除の処理
      doRemove: function(item) {
        var index = this.todos.indexOf(item)
        this.todos.splice(index, 1)
      }
    },
    watch: {
      // オプションを使う場合はオブジェクト形式にする
      todos: {
        // 引数はウォッチしているプロパティの変更後の値
        handler: function(todos) {
          todoStorage.save(todos)
        },
        // deep オプションでネストしているデータも監視できる
        deep: true
      }
    },
    created() {
      // インスタンス作成時に自動的に fetch() する
      this.todos = todoStorage.fetch()
    },
    computed: {
      computedTodos: function() {
        // データ current が -1 ならすべて
        // それ以外なら current と state が一致するものだけに絞り込む
        return this.todos.filter(function(el) {
          return this.current < 0 ? true : this.current === el.state
        }, this)
      },
      labels() {
        return this.options.reduce(function(a, b) {
          return Object.assign(a, { [b.value]: b.label })
        }, {})
        // キーから見つけやすいように、次のように加工したデータを作成
        // {0: '作業中', 1: '完了', -1: 'すべて'}
      }
    }
  }
</script>

<style>
  * {
    box-sizing: border-box;
  }
  #app {
    max-width: 640px;
    margin: 0 auto;
  }
  table {
    width: 100%;
    border-collapse: collapse;
  }
  thead th {
    border-bottom: 2px solid #0099e4; /*#d31c4a */
    color: #0099e4;
  }
  th,
  th {
    padding: 0 8px;
    line-height: 40px;
  }
  thead th.id {
    width: 50px;
  }
  thead th.state {
    width: 100px;
  }
  thead th.button {
    width: 60px;
  }
  tbody td.button, tbody td.state {
    text-align: center;
  }
  tbody tr td,
  tbody tr th {
    border-bottom: 1px solid #ccc;
    transition: all 0.4s;
  }
  tbody tr.done td,
  tbody tr.done th {
    background: #f8f8f8;
    color: #bbb;
  }
  tbody tr:hover td,
  tbody tr:hover th {
    background: #f4fbff;
  }
  button {
    border: none;
    border-radius: 20px;
    line-height: 24px;
    padding: 0 8px;
    background: #0099e4;
    color: #fff;
    cursor: pointer;
  }
</style>
