<script setup>
import { ref } from 'vue'

const articles = ref([])

wikipediaWhatIsToday()

function wikipediaWhatIsToday() {
  const now = new Date()
  const month = now.getMonth()+1
  const day = now.getDate()
  const monthDay = month+"月"+day+"日 "
  const url = 'https://ja.wikipedia.org/w/api.php?format=json&utf8&origin=*&action=query&prop=revisions&rvprop=content&titles=' + monthDay

  fetch(url,{ mode: 'cors' })
    .then(res => res.json())
    .then(data => {
      const pageid = Object.keys(data.query.pages)[0]
      const content = data.query.pages[ pageid ].revisions[0]["*"]
      // 「できごと」は文字列「== できごと ==」と「== 誕生日 ==」の間にある
      const dekigoto = content.match(/== できごと ==([\s\S]*)== 誕生日 ==/)[1]  // 改行文字を含む文字列には [\s\S]* を使う
      // 「できごと」の中にさらに一覧があるのでそれを取り出す
      // アスタリスクで始まり改行文字で終わる文字列を取り出す
      articles.value = dekigoto.match(/\*.+\n/g);  // *の前に\が必要
    })
}
</script>

<template>
  <ul>
    <li>今日は山の日です</li>
    {{articles}}
  </ul>
</template>
