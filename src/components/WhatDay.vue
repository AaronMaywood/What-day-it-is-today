<script setup>
import { ref } from 'vue'
const articles = ref([])

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
    // articles.value = dekigoto.match(/\*.+\n/g);  // *の前に\が必要
    const tmp = dekigoto.match(/\*.+\n/g);  // *の前に\が必要
    for(let i=0;i<tmp.length;i++){ 
      const regex = /\[\[([^\]]+)\]\]/g
      tmp[i] = tmp[i].replaceAll(regex,replacerfn)
    }
    articles.value = tmp
  })

// Wikipedia のリンクにする
// [[1787年]] → <a href="https://ja.wikipedia.org/wiki/1787%E5%B9%B4" target="_blank">1787年</a>
// TODO 「貞観 (日本)|貞観」の場合には「貞観_日本」にする
function replacerfn(match, p1){
  console.log({
    match: match,
    patten:p1
  })
  const url = encodeURI(`https://ja.wikipedia.org/wiki/${p1}`)
  return `<a href="${url}" target="_blank">${p1}</a>`
}


</script>

<template>
  <ul>
    <li v-for="item in articles">
      <span v-html="item"></span>
    </li>
  </ul>
</template>
